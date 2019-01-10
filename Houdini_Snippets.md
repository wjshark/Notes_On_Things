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
- Spin point or geo
  ```
  // create a matrix
  matrix3 m = ident();

  // rotate the matrix
  vector axis = {0,0,1};
  float angle = radians(ch('amount'));

  rotate(m, angle, axis);

  // apply the rotation
  @P *= m;
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
  - In a Time Shift Node put (thank you AV!):
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
- over total points
    ```
    @Cd = set(relbbox(@P).x, 0, 0);
    ```
 - Gradient ramp with fit
    ```
    float min = ch('min');
    float max = ch('max');
    float ramp = fit(@P.x, min,max,0,1);
    @Cd = 0;  // lazy way to reset the colour to black before the next step
    @Cd.r = chramp('myramp',ramp);
    ```
## 8. Delete half Geo for Mirror
- Add Attribute Wrangle
```
if (@P.x>0)
removepoint(0,i@ptnum);
```
## 9 .Rotate prims around an edge
- From Matt Estela [http://www.tokeru.com/cgwiki/?title=HoudiniVex#Create_a_new_attribute]
  ```
  int points[] = primpoints(0,@primnum); // list of points in prim

  // get @P of first and second point
  vector p0 = point(0,'P',points[0]);
  vector p1 = point(0,'P',points[1]);

  vector axis = normalize(p0-p1);

  float angle = ch('angle');
  matrix3 rotm = ident();
  rotate(rotm, angle, axis);

  // get midpoint of edge
  vector pivot = (p0+p1)/2;

  // move point to origin, rotate, move back

  @P -= pivot;
  @P *= rotm;
  @P += pivot;
  `
## 10. Probability split range on point
- From S.K Particles III
- Creates prob attribute on percentage points in order to manipulate
- Adds to Attribute Wrangle
  ```
  @prob = fit(@ptnum, 0 ,1 , 0, ch("max_prob"));
  @group_prob = rand(@ptnum + @id*456 + ch("overall_seed")) < @prob;
  ```
## 11. Noise
- Curl noise animated (kill time to remove anim)
  ```
  @Cd = 0;
  @Cd.x = curlnoise(@P*chv('scale')+@Time);
  ```
## 12. Export .abc as single transform
- after copy or at end of sim append an 'attribute create'
  - name = path
  - type = string
  - string = /SRT/MeshName_GEO
- Then append a ROP Alembic
  - tick 'Build Hierarchy from Attribute'
    - Path Attribute = Path
