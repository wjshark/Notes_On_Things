## 1. Random pscale with range
- Add Attribute Wrangle
  ```
  @pscale = fit(rand(@ptnum),0 ,1 , ch("min_pscale"), ch("max_pscale"));
  ```
## 2. Orient around axis
- Add Attribute Wrangle
  - needs normals so add a facet with post processing ticked
  ```
  float rand = fit(rand(@ptnum+311),0,1 , ch("min_rot"), ch("max_rot"));
  p@rot = quaternion(radians(rand), v@N);
  ```
- Random rotation
  ```
  @orient = rand(@ptnum);
  ```
