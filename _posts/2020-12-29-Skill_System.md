---
layout: post
title: "基于Dota2技能系统研究与实践"
date:   2020-12-29 10:39:12
categories: MacroDog
tags:  Lua 
excerpt: Lua
mathjax: true
---
* content  
{:toc}
这个项目是想实现一个类似于dota2一样的易于扩展技能系统，学习一下dota2技能系统设计。   
# 数据驱动类技能
>几乎所有在DOTA2中的技能都是使用C++来编写的，当查看所有这些技能的时候，npc_abilities.txt包含了一些技能元数据的KV(Key-Values)文件，暴露出了一些技能的键值，这里包含了几乎所有的元数据，使用数据驱动类技能系统能够为DOTA2创造一些自定义技能，能够为他们赋予特殊的数值和特定的触发事件。
> 
实际上这是技能编辑器部分玩家可以通过暴露的接口设定的值控制一些技能基本属性，对于不同技能的效果可以通过输入脚本自定义。在dota2中玩家可以编写lua脚本自定义效果。我想设计系统也是如此，基本属性通过json格式保存，自定义效果可以通过脚本输入。当然这个只是编辑器使用部分主要的还是技能系统设计，这个只是在我们设计系统时需要考虑的一点。
# 技能系统
## Buff
#### Buff要考虑的部分    
1.施法者（caster）
2.作用对象（target）
3.持续时间（time）
4.叠加
5.施法技能
6.tag
等等待添加
buff本身是个基类要包含基础信息至于每个技能实际生效的通过buff子类Modifier实现
在dota2的脚本信息里技能信息里可有定义生效的buff和buff触发lua脚本。例子如下:
dota2里技能在Modifiers字段下定义需要使用的buff。在RunScript中传入调用脚本和传入的参数
~~~

	"Modifiers"
	{
		"modifier_doom_datadriven"
		{
			"IsDebuff"			"1"
			"IsPurgable"		"0"
	
			"StatusEffectName" 		"particles/status_fx/status_effect_doom.vpcf" 	   
			"StatusEffectPriority"  "10"
	
			"OnCreated"
			{
				"AttachEffect"
				{
					"EffectName"        "particles/units/heroes/hero_doom_bringer/doom_bringer_doom.vpcf"
					"EffectAttachType"  "follow_origin"
					"Target"
					{
						"Center"	"TARGET"
						"Flags"		"DOTA_UNIT_TARGET_FLAG_MAGIC_IMMUNE_ENEMIES"
					}

					"ControlPoints"
					{
						"04"	"TARGET"
					}
				}
			}

			"OnDestroy"
			{
				"RunScript"
				{
					"ScriptFile"	"heroes/hero_doom_bringer/doom.lua"
					"Function"		"StopSound"
					"sound"			"Hero_DoomBringer.Doom"
				}
			}

			"ThinkInterval"  "1.0"
			"OnIntervalThink"
			{
				"Damage"
				{
					"Target"		"TARGET"
					"Type"			"DAMAGE_TYPE_PURE"
					"Damage"		"%damage"
				}
			}

			"States"
			{
				"MODIFIER_STATE_SILENCED"		"MODIFIER_STATE_VALUE_ENABLED"
			}
		}

		"modifier_doom_deny_check_datadriven"
		{
			"IsHidden"		"1"
			"IsPurgable"	"0"

			"ThinkInterval"  "0.03"
			"OnIntervalThink"
			{
				"RunScript"
				{
					"ScriptFile"	"heroes/hero_doom_bringer/doom.lua"
					"Function"		"DoomDenyCheck"
					"modifier"		"modifier_doom_deny_datadriven"
				}
			}
		}

		"modifier_doom_deny_datadriven"
		{
			"IsHidden"		"1"
			"IsPurgable"	"0"

			"States"
			{
				"MODIFIER_STATE_SPECIALLY_DENIABLE"		"MODIFIER_STATE_VALUE_ENABLED"
			}
		}
	}
~~~
### Modifier&&Buff流程
1.CheckCondition 检查生效条件
2.OnCreat 创造出来
3.Interval 调用
4.OnDestroy 销毁
## Skill
可以有多个buff
### 普通攻击
### 技能施法


## 开发过程
### 执行自定义脚本
遇到问题在编写技能系统的时候想要想dota2一样支持自定义脚本。dota2编辑器中可以通过在特定事件下通过使用"RunScript"字段在触发想要执行的脚本例子如下:
~~~
"OnIntervalThink"
{
	"RunScript"
	{
		"ScriptFile"	"heroes/hero_doom_bringer/doom.lua"
		"Function"		"DoomDenyCheck"
		"modifier"		"modifier_doom_deny_datadriven"
	}
}
~~~~
我项目中技能实体在C#层,lua部分使用xlua

