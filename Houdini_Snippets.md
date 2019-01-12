## 1. Random pscale with range
- Add Attribute Wrangle
  ```
  @pscale = fit(rand(@ptnum),0 ,1 , ch("min_pscale"), ch("max_pscale"));
  ```
## 2. Orient
- Add Attribute Wrangle
  - needs normals so add a facet with post processing ticked
  ```
  float rand = fit(rand(@ptnum+311),0,1 , ch("min_rot"), ch("max_rot"));
  p@rot = quaternion(radians(rand), v@N);
- Or the cgwiki way [http://www.tokeru.com/cgwiki/index.php?title=HoudiniVex#Normalizing_vectors]
  - "use a dihedral to create a matrix that will rotate the {0,0,1} vector to @N, then rotate that matrix around @N.
    Convert that matrix to @orient, and you're done":
    ```
    matrix3 m = dihedral({0,1,0},@N);
    rotate(m,fit(rand(@ptnum+311),0,1 , ch("min_rot"), ch("max_rot")), @N);
    @orient = quaternion(m);
    ```
- Or the Raph Gadot way [http://www.tokeru.com/cgwiki/index.php?title=HoudiniVex#Normalizing_vectors]
  ```
  @orient = quaternion(maketransform(normalize(-@P),{0,1,0}));
  vector4 pitch = quaternion({1,0,0}*fit01(rand(@ptnum+311), ch("min_rot_x"), ch("max_rot_x")));
  vector4 roll  = quaternion({0,0,1}*fit01(rand(@ptnum+311), ch("min_rot_y"), ch("max_rot_y")));
  vector4 yaw   = quaternion({0,1,0}*fit01(rand(@ptnum+311), ch("min_rot_z"), ch("max_rot_z")));

  @orient = qmultiply(@orient, pitch);
  @orient = qmultiply(@orient, yaw);
  @orient = qmultiply(@orient, roll);
  ```
- Random rotation
  ```
  @orient = rand(@ptnum);
  ```
- More random rotation with sliders
  ```
  float minX = chf("minX");
  float maxX = chf("maxX");
  float minY = chf("minY");
  float maxY = chf("maxY");
  float minZ = chf("minZ");
  float maxZ = chf("maxZ");

  float randomX = fit01(rand(@ptnum+chf("seed")), minX, maxX);
  float randomY = fit01(rand(@ptnum+chf("seed")), minY, maxY);
  float randomZ = fit01(rand(@ptnum+chf("seed")), minZ, maxZ);

  float angleX = radians(randomX);
  float angleY = radians(randomY);
  float angleZ = radians(randomZ);

  vector angle = set(angleX, angleY, angleZ);
  @orient = (eulertoquaternion(angle, 0));
  ```
## 3. Add upVector
  ```
  @up = {0,1,0};
  ```
## 4. Groups
- check if in a group
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
- Sometimes its nice to be explicit with groups when assigning
  ```
  float condition = rand(@ptnum);
  if(condition > 0.5)
    {
      @group_groupA = 1;
    }    
  else
    {
      @group_groupB = 1;
    }
   ```
## 5. Delete point by threshold
  ```
  if ( rand(@ptnum) > ch('threshold') ) {
     removepoint(0,@ptnum);
  }
  ```
## 6. Stamping
- Random instances by id
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
- imput count starts at 0
  ```
  @P = v@opinput1_P;
  ```
## 8. Colour
- Random Cd
  - if instance; Add a wrangle in a Instance Node
    ```
    v@Cd = vector(chramp("Color", rand(@ptnum+ch("seed"))));
    ```
- Add Cd gradient over total num points
    ```
    @Cd = set(relbbox(@P).x, 0, 0);
    ```
- Cd gradient ramp with fit
    ```
    float min = ch('min');
    float max = ch('max');
    float ramp = fit(@P.x, min,max,0,1);
    @Cd = 0;  // lazy way to reset the colour to black before the next step
    @Cd.r = chramp('myramp',ramp);
    ```
  - Colour side on geo/prims
    ```
    @Cd = @N; 
    if(min(@Cd)<0) {
      @Cd = 0.1;
      }
    ```
## 8. Delete half Geo for Mirror
- Add Attribute Wrangle
  ```
  if (@P.x>0)
  removepoint(0,i@ptnum);
  ```
## 9. Rotate prims around an edge
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
  ```
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
- This Noise pattern can be added to any wrangle, by kiryha [https://github.com/kiryha/Houdini/tree/master/hips]
  ```
  // Visualise nose as Black and White values
  // Delete black and white points separatly

  // Default non zero values for 10X10 grid:
  // Noise_size = 1
  // Noise_threshold = 0.5

  // Make geometry white
  @Cd = {1, 1, 1};

  // Setup noise
  float noseValues = noise(@P*(1/chf('Noise_Size')) + chf('Noise_Offset'));

  // Paint-delete points with noise
  if(noseValues > chf('Noise_Threshold')){
      @Cd = 0;
      if(rand(@ptnum) < ch('delete_black')){
          if(chi('del') == 0){
              @Cd = {1,0,0};
              }
          else{
              removepoint(0,@ptnum);
              }
          }
      }

  if(noseValues < chf('Noise_Threshold')){
      if ( rand(@ptnum) < ch('delete_white') ) {       
          if(chi('del') == 0){
              @Cd = {1,0,0};
              }
          else{
              removepoint(0,@ptnum);
              }
          }
      }
  ```
## 12. Export .abc as single transform
- after copy or at end of sim append an 'attribute create'
  - name = path
  - type = string
  - class = primitive
  - string = /SRT/MeshName_GEO
- Then append a ROP Alembic
  - tick 'Build Hierarchy from Attribute'.
    - Path Attribute = path
      [http://willjsharkey.com/wp-content/uploads/2019/01/ABC_Path_BuildingGeo.jpg]
    - Thanks AV!    
## 13. Randoms instance from string
- Add a wrangle in a Instance Node
   ```
  string objList = "/obj/box1 /obj/sphere1 /obj/torus1 /obj/grid1";
  string arrayObjects[] = split(objList);
  int index = (int) rint(fit01(rand(@ptnum+ch("seed")), 0, len(arrayObjects)-1));
  s@instance = arrayObjects[index];
  ```
