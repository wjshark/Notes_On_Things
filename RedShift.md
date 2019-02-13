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
