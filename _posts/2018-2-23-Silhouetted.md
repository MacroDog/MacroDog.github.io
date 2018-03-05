---
layout: post
title:  Unity Silhouetted Shader
---
## Unity Silhouetted Shader
  描边这种风格被广泛应用在卡通风格当中。最近也看了很多，记录一下自己的理解。

#### Edge Detection
这种方法是一种屏幕后处理。在一张2D图像中对于边缘的定义其实就是与附近像素色差极大而产生。这种方法就是对于当前像素周围的像素进行采样然后通过算法检测。我使用是Sobel算子代码如下。

```
Shader "Custom/EdgeDerection" {
      Properties {
        _MainTex("MainTex",2D ) = "white"{}
        _EdgeOnly("EdgeOnly",Float )= 1.0
        _EdgeColor("EdgeColor",Color ) = (0,0,0,1)
        _EdgeSize("EdgeSize",Range(0,5)) = 5
        _BackgroundColor("Background Color",Color ) = (1,1,1,1)
      }
      SubShader {
        Pass {
          ZTest Always
          Cull  Off
          ZWrite Off
          CGPROGRAM
          #pragma vertex vert
          #pragma fragment frag
          #include "UnityCG.cginc"
          sampler2D _MainTex;
          half4 _MainTex_TexelSize;
          float _EdgeOnly;
          float4 _EdgeColor;
          float _EdgeSize;
          float4 _BackgroundColor;
          struct v2f
          {
            float4 pos:SV_POSITION;
            half2 uv[9]:TEXCOORD0 ;
          };
          fixed luminance(fixed4 color){
            return 0.2125 * color.r + 0.7154 * color.g + 0.0721 * color.b;
          }
          half Sobel(v2f i){
            const half Gx[9] = {-1,-2,-1,0,0,0,1,2,1};
            const half Gy[9] = {-1,0,1,-2,0,2,-1,0,1};
            half texColor;
            half edgeX = 0;
            half edgeY = 0;
            for (int it = 0;it  < 9; ++it)
            {
              texColor = luminance(tex2D(_MainTex,i.uv[it]));
              edgeX += texColor * Gx[it];
              edgeY += texColor * Gy[it];
            }
            half edge = 1 - abs(edgeX) - abs(edgeY);
            return edge ;
          }
          v2f vert(appdata_img v){
            v2f o;
            o.pos = UnityObjectToClipPos(v.vertex);
            half2 uv = v.texcoord;
            o.uv[0] = uv + _MainTex_TexelSize.xy * half2(-1,-1) * _EdgeSize;
            o.uv[1] = uv +_MainTex_TexelSize.xy*half2(0,-1) * _EdgeSize;
            o.uv[2] = uv + _MainTex_TexelSize.xy * half2(1,-1) * _EdgeSize;
            o.uv[3] = uv + _MainTex_TexelSize.xy * half2(-1,0) * _EdgeSize;
            o.uv[4] = uv +_MainTex_TexelSize.xy * half2(0,0) * _EdgeSize;
            o.uv[5] = uv + _MainTex_TexelSize.xy * half2(0,1) * _EdgeSize;
            o.uv[6] = uv + _MainTex_TexelSize.xy * half2(-1,1) * _EdgeSize;
            o.uv[7] = uv + _MainTex_TexelSize.xy * half2(0,1) * _EdgeSize;
            o.uv[8] = uv + _MainTex_TexelSize.xy * half2(1,1) * _EdgeSize;
            return o;
          }
          fixed4 frag(v2f i) :SV_Target{
            half edge = Sobel(i);
            fixed4 withEdgeColor = lerp(_EdgeColor,tex2D(_MainTex,i.uv[4]),edge);
            return withEdgeColor;
          }
          ENDCG
        }
      }
    FallBack "Diffuse"
}
```
这种是单纯靠算法分别图像中的边缘然后进行描边。好处性能和场景复杂度无关（只和分辨率有关）而且也会对与图片进行描边，但是不能对特定的物体修改描边，精确度可能会有问题。
#### Edge Detection With Depth And Normal
除了通过屏幕图像色差直接进行边缘检测然后描边，也可以通过z-buffer和normal来对3D物体进行边缘检测。做法的核心思想就是通过读取_CameraDepthNormalsTexture获得顶点的深度信息，通过比对附近的像素深度和法线差异。当差异大于阀值则判断为边缘。
```
Shader "Custom/EdgeDetectNormalsAndDepth" {
  Properties {
  _MainTex("Base(RGB)",2D) = "white"{}
  _EdgeColor("Color",Color) = (1,1,1,1)
  _EdgeOnly("Edge Only", Float) = 1.0
  _BackgroundColor("_BackgroundColor", Color) = (1,1,1,1)
  _SampleDistance("SampleDistance",Float) = 1.0
  _Sensitivity("Sensitivity",Vector) = (1,1,1,1)
  }
  SubShader {
    Tags { "RenderType"="Opaque" }
    LOD 200
    CGINCLUDE
    #include"UnityCG.cginc"
    sampler2D _MainTex;
    float4 _MainTex_TexelSize;
    float4 _EdgeColor;
    float4 _EdgeOnly;
    float4 _BackgroundColor;
    float _SampleDistance;
    float2 _Sensitivity;
    sampler2D _CameraDepthNormalsTexture;
    struct v2f{
      float4 pos : SV_POSITION;
      float2 uv[5] : TEXCOORD0;
    };
    v2f vert (appdata_img v){
      v2f o;
      o.pos = UnityObjectToClipPos(v.vertex);
      float2 uv = v.texcoord;
      o.uv[0] = uv;
      #if UNITY_UV_STARTS_AT_TOP
      if(_MainTex_TexelSize.y<0)
        uv.y=1-uv.y;
      #endif
      o.uv[1] = uv + _MainTex_TexelSize.xy*float2(1,1)*_SampleDistance;
      o.uv[2] = uv + _MainTex_TexelSize.xy*float2(-1,-1)*_SampleDistance;
      o.uv[3] = uv + _MainTex_TexelSize.xy*float2(-1,1)*_SampleDistance;
      o.uv[4] = uv + _MainTex_TexelSize.xy*float2(1,-1)*_SampleDistance;
      return o;
    }

    float CheckSame(float4 center,float4 sample){
      float2 centerNormal = center.xy;
      float centerDepth = DecodeFloatRG(center.zw);
      float2 samplerNormal = sample.xy;
      float sampleDepth = DecodeFloatRG(sample.zw);
      float2 diffNormal =  abs(centerNormal - samplerNormal) * _Sensitivity.x;
      int isSameNormal  =  (diffNormal.x - diffNormal.y) < 0.1;
      float diffDepth =  abs(centerDepth-sampleDepth)* _Sensitivity.y;
      int isSameDepth = diffDepth < 0.1 * centerDepth;
      return (isSameNormal * isSameDepth) ? 1.0f : 0.0f;
    }
    float4 frag(v2f i) : SV_Target{
      float4 sample1 = tex2D(_CameraDepthNormalsTexture,i.uv[1]);
      float4 sample2 = tex2D(_CameraDepthNormalsTexture,i.uv[2]);
      float4 sample3 = tex2D(_CameraDepthNormalsTexture,i.uv[3]);
      float4 sample4 = tex2D(_CameraDepthNormalsTexture,i.uv[4]);
      half edge = 1.0;
      edge *= CheckSame(sample1,sample2);
      edge *= CheckSame(sample3,sample4);
      fixed4 withEdgeColor = lerp(_EdgeColor,tex2D(_MainTex,i.uv[0]),edge);
      fixed4 onlyEdgeColor = lerp(_EdgeColor,_BackgroundColor,edge);
      return lerp(withEdgeColor,onlyEdgeColor,_EdgeOnly);
    }
    ENDCG
    Pass{
        ZTest Always Cull Off ZWrite Off
        CGPROGRAM
        #pragma vertex vert
        #pragma fragment frag
        ENDCG
        }
      }
  FallBack Off
}
```


这种方法也是屏幕后处理所以添加到项目中很简便，但是有很多局限性比如，不能单独修改某一物体的描边大小和颜色，同时对于深度值变化较小的的轮廓就无法检测出来等等 还是要慎用。

#### Procedural Geometry Silhouette
这个方法的核心思想就是用过使用两个Pass分别渲染正面和背面，在渲染背面的Pass中讲背面的沿法线(或者其它方向)方扩展一些。这样就可以在渲染正面后就仍然能看到被扩展出来的那一部分。

```
Shader "Custom/outline"{
  Properties{
  _MainTex("MainTex",2D ) = "white"{}
  _Color("Color",Color) = (1,1,1,1)
  _OutlineSize("OutLineSize",Range(0,0.1)) = 0.01
  _OutlineColor("OutlineColor",Color) = (1,1,1,1)
  }
  SubShader{
    Pass{
      Tags{"RenderType" = "Opaque" "LightMode"="ForwardBase"}
      Cull Back
      CGPROGRAM
      #pragma vertex vert
      #pragma fragment frag
      #pragma multi_compile_fwdbase
      #include "UnityCG.cginc"
      sampler2D _MainTex;
      float4 _Color;
      struct a2v{
        float4 vertex : POSITION;
        float3 normal : NORMAL;
        float4 texcoord : TEXCOORD0;
      };
      struct v2f{
        float4 pos : SV_POSITION;
        float2 uv :TEXCOORD0;
      };
      v2f vert(a2v v){
        v2f o;
        o.pos = UnityObjectToClipPos(v.vertex);
        o.uv =v.texcoord.xy;
        return o;
      }
      float4 frag(v2f i):SV_Target{
        float3 albedo = tex2D(_MainTex,i.uv).rgb*_Color;
        float3 ambient = UNITY_LIGHTMODEL_AMBIENT.rgb * albedo;
        return ambient;
      }
      ENDCG
    }
    Pass{
      NAME "OUTLINE" //用来扩展的Pass放在后面同时开启深度测试可以减少一些性能消耗
      Tags{"RenderType" = "Qpaque"}
      Cull Front
       ZTest Always
      CGPROGRAM
      #pragma vertex vert
      #pragma fragment frag
      #include "UnityCG.cginc"
      float4 _OutlineColor;
      float _OutlineSize;
      struct v2f{
        float4 pos : SV_POSITION;
      };
      v2f vert(appdata_base v){
        v2f o ;
        float4 viewPos = mul(UNITY_MATRIX_MV,v.vertex);
        float3 viewNormal = mul(v.normal,UNITY_MATRIX_IT_MV).xyz;
        viewNormal.z = -0.5;
        viewPos += float4(normalize(viewNormal),0)*_OutlineSize;
        o.pos = mul(UNITY_MATRIX_P,viewPos);
        return o;
      }
      float4 frag(v2f i) :SV_Target{
        return float4(_OutlineColor.rgb,1);
      }
      ENDCG
    }
  }
}
```
这种方法好处就是可控，每一个物体都可以随意调控。但是同时也有一些其它问题 比如：使用了两个Pass要注意性能。而且效果其实只是背面的轮廓线并不是真正的描边。
如果要想到达真正的比较完美的描边，可以考虑从贴图上下手 我想到的是对贴图采样进行Edge Detection 原理其实和之前的对屏幕图像进行修改的Edge Detection shader 一样只不过采样的对象是贴图，然后添加描边效果。这种效果其实就和美术在贴图上进行描边一样。对贴图（美术）有一定要求。
