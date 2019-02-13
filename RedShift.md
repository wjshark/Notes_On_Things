## Unified Sampling & Cleaning Noise
- Need more Local samples than Max samples
  - Local samples = Spec, Ref, lights, A), GI etc
  - the Unified samples should be focusing on AA, Dof, motion blur samples, not everythinhg else
  - most optimal on a per object Material basis (eg reflection sample, refraction etc)
- Adaptive error threshold = how sensitive the engine has to be at sampling



## Houdini Multi Redshift Proxy Instance
- add a wrangle after the scattered points
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
- add a wrangle
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
