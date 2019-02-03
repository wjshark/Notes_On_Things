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
  - From XSI Fabricio Chamon
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
   - another
    ```
    int ingroup = inpointgroup(0, "group1", @ptnum);
    if (ingroup == 1) 
    {@Cd = set(1,0,0);} 
    else 
    {@Cd = set(0,0,0);}
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
  - In a Time Shift Node put:
    ```
    $F + 40 * stamp("../copy3", "inst", 0)
    ```
  - In a copy stamp put varible 'inst', add this to value:
    ```
    $CY  
    ```
## 7. Get Attr second input of a wrangle node
- input count starts at 0
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
  - works on primatives in wrangle
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
## 9. Init parameters
- wrangle to add initial translation, rotation and scale
  ```
  @N;
  @up = {0,1,0};

  // Define random position values
  float randPos_X = fit01(rand(@ptnum), -ch('Translate_X'), ch('Translate_X'));
  float randPos_Y = fit01(rand(@ptnum), -ch('Translate_Y'), ch('Translate_Y'));
  float randPos_Z = fit01(rand(@ptnum), -ch('Translate_Z'), ch('Translate_Z'));
  vector randPos = set(randPos_X, randPos_Y, randPos_Z);

  // Define random rotation values
  float randRot_X = fit01(rand(@ptnum), -ch('Rotate_X'), ch('Rotate_X'));
  float randRot_Y = fit01(rand(@ptnum), -ch('Rotate_Y'), ch('Rotate_Y'));
  float randRot_Z = fit01(rand(@ptnum), -ch('Rotate_Z'), ch('Rotate_Z'));

  // Apply random positions
  @P.x += randPos_X; 
  @P += randPos_Y*@N; 
  @P.z += randPos_Z; 

  // Apply random rotations
  @orient = quaternion(maketransform(@N,@up));
  vector4 rotate_X = quaternion(radians(randRot_X),{1,0,0});
  vector4 rotate_Y = quaternion(radians(randRot_Y),{0,1,0});
  vector4 rotate_Z = quaternion(radians(randRot_Z),{0,0,1});
  @orient = qmultiply(@orient, rotate_X);
  @orient = qmultiply(@orient, rotate_Y);
  @orient = qmultiply(@orient, rotate_Z);

  // Apply random scale
  @scale = fit01(rand(@ptnum), chf('Scale_MIN'), chf('Scale_MAX'));
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
- Fade noise on curves with ramp
  - Requires uvtexture SOP in "Pts and Columns" mode before this wrangle
    ```
     // Define UI controls
    float remap_uv = chramp('remap_uv', @uv.x);
    float power = chf('Noise_Power');
    float freq = chf('Noise_Frequency');

    // Create noise
    vector noiseXYZ = noise(@P*freq);
    // Modify noise values
    vector displace = fit(noiseXYZ, 0,1, -1, 1)*power*remap_uv;
    // Apply modified noise to a points position
    @P += displace;
    // Visualize fade ramp on curve
    @Cd = remap_uv;
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
## 13. Randoms instance from string
- This is an instance workflow with objects as separate node in the scene, and an instance node.
- Add a wrangle in the Instance Node after the points to instance to
   ```
  string objList = "/obj/box1 /obj/sphere1 /obj/torus1 /obj/grid1";
  string arrayObjects[] = split(objList);
  int index = (int) rint(fit01(rand(@ptnum+ch("seed")), 0, len(arrayObjects)-1));
  s@instance = arrayObjects[index];
  ```
## 14. Make a spiral
- make a open arc circle and add a wrangle
  ```
  @P *= 0.01 * @ptnum;
  ```
- make another one
  ```
  float angle;
  vector pos = {0,0,0}; 
  int npoints = chi('number_of_points');
  float step = radians(ch('sweep'))/npoints;

  for (int n=0; n<npoints; n++) {
      angle = step * n;

      pos.x = sin(angle) * angle; 
      pos.y = angle;
      pos.z = cos(angle) * angle;

      addpoint(0, pos);
  }
  ```
 - Phylotaxis Spirals, by [https://github.com/kiryha/Houdini]
    ```
    int count = 400;
    float bound = 10.0;
    float tau = 6.28318530; // 2*$PI
    float phi = (1+ sqrt(5))/2; // Golden ratio = 1.618
    float golden_angle = (2 - phi)*tau; // In radians(*tau)
    vector pos = {0,0,0};
    float radius = 1.0;
    float theta = 0;
    int pt;


    vector polar_to_cartesian(float theta; float radius){
        return set(cos(theta)*radius, 0, sin(theta)*radius);
    }

    for (int n=0; n<count; n++){
        radius = bound * pow(float(n)/float(count), ch('power'));
        theta += golden_angle;

        pos = polar_to_cartesian(theta, radius);

        // Create UP, pscale and N attr
        pt = addpoint(0, pos);
        setpointattrib(0, "pscale", pt, pow(radius,0.5));
        setpointattrib(0, "N", pt, normalize(-pos));
        setpointattrib(0, "up", pt, set(0,1,0));
    }
    ```  
## 15. Normals and Tangent
- to get tangent on a curve
  - make a polyframe after the line
  - put N in 'tangent name'
  - you can rotate them by making a point vop, appending a 'make trasform' node and multiplying the normal by it.
- to point normals out from a curve
  - add wrangle after the polyframe
  ```
  v@N = cross(v@N, set(0, 1, 0));
  ```

## 16. Spherical and linear gradients
- modified code from here [http://www.tokeru.com/cgwiki/index.php?title=HoudiniVex#Spherical_and_linear_gradients]
- get a grid, append a wrangle, add two points (2 spheres with a merge) into second wrangle input port
  - this will get you a linear gradient between 2 points
    ```
    vector p1, p2, v1, v2;
    p1 = point(1,'P',0);
    p2 = point(1,'P',1);

    v1 = @P-p1; // treat p1 as origin;
    v2 = normalize(p2-p1);

    float r = distance(p1,p2);
    @Cd = clamp(dot(v1,v2)/r, 0, @Cd);
    ```
  - after you can append another wrangle with a ramp to control it
  - this example will affect P.y
    ```
    @P.y += chramp('myramp',@Cd);
    ```
  - this will get you a spherical gradient between 2 points
    ```
    vector p1 = point(1,'P',0);
    vector p2 = point(1,'P',1);

    float r = distance(p1,p2);
    @Cd = (r-distance(@P, p1))/r;
    ```
## 17. Extrude by colour
- get a grid with a @Cd gradient going through it
- append an attributePromote
  - point to primitives
  - 'Original name' = Cd
- append a wrangle on primitives
  ```
  f@extrudeAmount = chramp('myramp',@Cd);
  ```
- append a polyextrude
  - Divide into 'Individual Element'
  - Distance = 1
  - In Local Control tab, Distance scale = extrudeAmount
## 18. Select mesh border points
- add a wrangle
  ```
  // Get number of connectet points
  int nbPts = neighbourcount(0,@ptnum);
  // Create "border" group with border points
  i@group_border = nbPts == 3 | nbPts == 2; 
  ```
## 19. Width
- after the curve append a polyframe
  -Target name = N
- append a resample node and tick the 'Curve U attribute' = curveu
- in a wrangle
  ```
  @width = fit(chramp("ramp_width", @curveu),0 ,1 , ch("min_width"), ch("max_width"));
  ```
- append a attribute rand to randomise the width if needed
  
## 20. Curve point Spreader
- from howiem on cgwiki discord [http://howiem.com/wordpress/]
- Putting the curve through a Resample SOP and a PolyFrame SOP added curveu, N and tangentu attributes to the curve, and those get passed on to the scattered points. Each point's normal (N) vector pointed straight out from the curve, but all the same way.
 - By rotating the N vector a random amount around the tangentu vector (which points along the curve), you could get that radial spread
 - So now you can scale N by a random amount (perhaps dependent on how far along the curve you are, like I seem to have already done above) and add it to @P to push the points' positions out.
  - The rotating-N-thing is straight out of cgwiki: 
    ```
    The rotating-N-thing is straight outta cgwiki: 
    // rotate N randomly round curve tangent
    matrix3 xform = ident();
    float angle = random(@ptnum+2323) * $PI*2;
    rotate(xform, angle, v@tangentu);
    @N *= xform;
    ```
  - this is the point spreader wrangle
    ```
    // rotate N randomly round curve tangent
    matrix3 xform = ident();
    float angle = random(@ptnum+2323) * $PI*2 * 1 + chf("Rotate");
    rotate(xform, angle, v@tangentu);
    @N *= xform;

    // scale N randomly using ramped random distribution
    @N *= chramp("noise_distribution",random(@ptnum+223));

    // scale N according to distance along the curve
    @N *= chramp("parm_scale_ramp",@curveu);

    // scale N by a global multiplier
    @N *= chf("scale");



    // finally, push position along N
    v@P += @N;
    ```
## 21. Delete By distance
- points in first wrangle input
- geometry in second input
  ```
   i@primid;
   v@uv;
   @dist;
   @dist = xyzdist(1, @P, @primid, @uv);

   if ( @dist> ch('threshold') ) {
     removepoint(0,@ptnum);
  }
  ```
## 22. Packed Prims
- from howiem on cgwiki discord [http://howiem.com/wordpress/]
- rotate by 90 degrees increments * @Time
  ```
  matrix3 m = ident();
  float angle = $PI * 0.5 * floor(@Time);
  vector axis = {0,1,0};
  rotate(m, angle, axis);
  setprimintrinsic(0, "transform", @primnum, m);
  ```
- rotate until 90 degrees * @Time
  ```
  matrix3 m = ident();
  float angle = clamp(@Time, 0.0, 1 * $PI);
  vector axis = {0,1,0};
  rotate(m, angle, axis);
  setprimintrinsic(0, "transform", @primnum, m);
  ```
- rotate by a slider, can be keyed
  ```
  matrix3 m = ident();
  float angle = (chf("AnimRotate") * $PI);
  vector axis = {0,1,0};
  rotate(m, angle, axis);
  setprimintrinsic(0, "transform", @primnum, m);
  ```
