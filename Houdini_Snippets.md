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
## 3. Add upVector
  ```
  @up = {0,1,0};
  ```
## 4. Group if statement
```
  if(i@group_act == 1)
  {
      i@active = 1;
  }    
  else
  {
      i@active = 0;
  }
  ```
## 5. Delete point by threshold
  ```
  if ( rand(@ptnum) > ch('threshold') ) {
     removepoint(0,@ptnum);
  }
  ```
## 6. Stamping
- Random insances by id
  - Put $ID in copy stamp field
  - This uses 3 ids
  ```
  rand(stamp("../copy1", "id", 2))
  ```
- Offset anim by clone
  - In a Time Shift Node put:
    ```
    $F + 40 * stamp("../copy3", "inst", 0)
    ```
  - In a copy stamp put varible 'inst', add this to value:
    ```
    $CY  
    ```
## 7. Get second input of a wrangle node
  ```
  @P = v@opinput1_P;
  ```
## 8. Add Gradient over points
  ```
  @Cd = set(relbbox(@P).x, 0, 0);
  ```
## 8. Delete half Geo for Mirror
- Add Attribute Wrangle
  ```
  if (@P.x>0)
  removepoint(0,i@ptnum);
  ```
