## Basics
- point based rendering techniques are not supported in IPR (subsurface scattering, photon mapping), use bucket rendering.
- to get chromatic abberation add a map to the bokeh shader using [this](https://docs.redshift3d.com/display/RSDOCS/Bokeh?product=maya#Bokeh-UseBokehImage/ImageNormalization) map
- AOV stands for arbitrary output variables
- Use Max trace depth on the material for reflection/refraction for efficiency, not globally.
- Volume contribute scale on the light adjusts the volume scattering
- If using a gobo, set the colour balance on the file texture 'default colour' to black. Also turn off wraping texture
  - Spot lights might give you a more expected result when using a gobo

## Unified Sampling & Cleaning Noise
- Info from [Saul Espinosa](https://www.youtube.com/watch?v=25YZ--F1aAQ&t=1s)
- Sampling formula info is [here](https://imgur.com/zqL9BBM)
- Need more Local samples than Max samples
  - Local samples = Spec, Ref, lights, AI, GI etc
  - the Unified samples should be focusing on AA, Dof, motion blur samples, not everythinhg else
  - most optimal on a per object Material basis (eg reflection sample, refraction etc)
- Adaptive error threshold = how sensitive the engine has to be at sampling
- Example:
  - If Unified samples = Min 16 Max 256
  - Put Light 'overide samples' to 512, which is double Max Unified
  - The Unified sampler only has to work on AA, ref etc and not put samples into lights.
- Increse Sample Filtering 'Filter Size' to remove jaggies on lights or hightlights (eg increase from 2 to 3). Creates a soften image but its cheap. Other ways are to increase sampling sampling but expensive.
- Increasing Adaptive error threshold is a faster render but noisier. It tells Redshift to reach sample limit sooner.
  - Adaptive error threshold at .003 would be a good range for a clean final render. (more sensitive to noise detection)
## Houdini Multi Redshift Proxy Instance
- Add a wrangle after the scattered points
  ```
  int source_count = 4;
  int proxy_id = int(rand(@ptnum+456)*source_count)+1;
  s@instancefile = sprintf("$HIP/../proxy/Grain.%03d.rs",proxy_id);
  ```
## Houdini Add velocity for Motion Blur in Maya
- This only necessary if your point count is changing. Its not a limit with alembic, it just doesnt make sense to calculate velocity over a changing point count. How can you track point 5 if it suddenly exists somewhere else on the next frame
  - If it is a static point count then you dont need this.
- create attribpromote
  - Original name = v Cd
  - point to Vertex
- Add a wrangle
  ```
  v@velocity = v@v;
  setattribtypeinfo(0, "vertex", "velocity", "color");

  v@color = v@Cd;
  setattribtypeinfo(0, "vertex", "color", "color");
  ```
- didnt work?
  - the geo has to be unpacked, abc doesnt know what packed geo is. Check the GeoSpreadsheet to see if velocity is there 
- Youre gonna need to Add a trail sop
  - check  'compute velocity'
  - set Velocity scale to .01 since we usually scale the geo down on import to .01 and export at 100
- In maya, go to MeshDisplay/ColourSetEditor and see if there are velocities
  - In AttributeEditor/MeshControls/MotionVectorColorSet, put 'velocity'
- that should work
## GGX vs Beckman
- Beckmann and GGX are two different algorithms for simulating 'microfacets' or micro scratches in dielectrics (typically things with reflection that arent metal) and conductors (metal). These are usually called 'microfacet distribution' and simulate the effect of very small imperfections in a surface - resulting in what appears to be a different falloff.
- So for example when you start to increase your roughness of the shader - we could generalise that the reflection becomes 'blurred'. Beckmann and GGX could be generalised into being called the 'falloff'.
## Subsurface Scattering
- A Transmittance colour of black means all light rays are absorbed resulting in zero scattering, white is full scattering.
- Higher values on Absorption scale helps make an object seem more dense. It affects the Transmittance. 
- Scatter Coeff should be grey values and use colour on the Transmittance to control the effect. Larger values of Scatter Coeff makes the center of the object more cloudy.
- Scatter Scale scales the Scatter Coeff. Higher values pushes the scatter effect making thinner parts scatter more. Try keeping the Scatter Coeff at full and just tweak the Scatter Scale.
- Phase of zero scatters in both directions, a value of 1 scatters away from the light source. 
