
# monte Carlo method
# Bidirectional Path Tracing
- traces sub-paths from both the camera and the light
- Connects the end points from both sub-path
# Metroplis Light Transport
- A Markov Chain Monte Carlo(MCMC)application
- Jumping from the current sample to the next with some PDF
# Photon Mapping
biased approach
very good ad handling Specular-Diffuse-Specular(SDS) paths and generationg caustics
- stage 1   
    Emitting photons from the light source,bouncing them around, then recording photons on diffuse surface
-  stage 2  
    Shoot sub-paths from the camera,bouncing them around,until they hit diffuse surfaces
- Calculation - local density estimation    
  idea: sreas with morebphotons should be brighter  
  For each shading point,find the nearest N photons.Take the surface area they over
## Why Biased?
- Local Density estimation
    $$\frac{dN}{dA}!=\frac{\Delta N}{\Delta A}$$
- But in the sense of limit
- More photons emitted ->
- the same N photons coversa smaller $\Delta A$
- $\Delta A$ is closer dA
- biased but consistent
- Understanding bias in rendering   
  Biassed == Blurry
  Consistent == Not blurry with infinite #samples
# Vertex Connection and Merging 
- key idea
  Let1's not waste the sub-paths inBDPT if their end points cannot be connected but can be merged   
  Use photon mapping to handle the merging of neayby "photons"
# Instant Radiosity
-   Key Idea    
   Lit surface can be treated as Light Source
- Approach  
    Shoot light sub-path and assume the end point of each sub-path is a Virtual Point Light(VPL)    
    Render the scene as usual using these VPLs
- Pros:fast and usually gives good results on diffuse scenes
- Cons
  Spikes will emerge when VPLs are close to shading points
  Cannot handle glossy materials  
# Advanced Appearance Modeling
## particiating media
At any point as light travels through a participating medium, it can be absorbed and scattered
-  Use Phase Function to describe the angular distribution of light scattering at any point x within participating media
###  particiating media:rendering
- randomly choose a direction to bunce
- randomly choose a distance to go straight
- At each 'shading point',connect to the light
## Hair Appearance
 Kajiya-Kay Model
 Marschner Model
## Granular Materaial
## Translucent Material
### Subsurface Scattering
Visual characteristics of many surfaces caused by light exiting at different points than it enters
- Violates a fundamental asssumption of the BRDF    
  $$S(x_i,w_i,x_o,w_o)$$
  $$L(x_o,w_o)=\int_A\int_{H^2}S(x_i,w_i,x_o,w_o)L_i(x_i,w_i)\cos\theta dw_idA$$
#### Dipole Approximation
- Approximate light diffusion by introducing two point surces
## Cloth
- Given the weaving pattern,calculate the overall behavior
- Render using a BRDF
### Cloth: Render as Participation Media
- Properties of individual fibers&their distribution->scattering parameters
- Render as participating medium
- ### Cloth: Render as Actual Fibers
  ## Detailed Appearance:Motivation
  ## Procedural Appearance
  noise function