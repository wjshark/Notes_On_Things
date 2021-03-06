## Emission
- Add to Emit impulse
    - Total Emission From Grid
    ```
    if($FF ==1,npoints("../grid_Emitter/"),0)
    ```
    - Total Emission From Geo, add to Emit Impulse
    - tick 'all points'
    ```
    $FF ==1
    ```
    - Emit every 30 Frames
    ```
    $F%30==0
    ```
    - Emit For a specific Frame range
    ```
    $F>0 && $F<15
    ```
## Pop Replicate
- emit attribute by age
    ```
    if(f@age <= .5){
    f@emit = 1;
    } else {
    f@emit = 0;
    }
    ```
## Deleting Partcles
- Add POP Wrangle
- Delete by Age Condition
    ```
    if (@age > .1) {
    @dead = 1;
    }
    ```
- Delete by Age Condition Rand
    ```
    if (@age > rand(0.1,1.0)*15) {
      @dead = 1;
    }
    ```
- Delete by ID array In Popnet
    ```
    int idlist[]= {10,5,15,1};
    if(find(idlist,@id)>=0){
      @dead = 1;
    }
    ```
- Delete by ID array Post SIM
    ```
    int idlist[]= {10,15,5,1};
    foreach(@id; idlist)
    {
    removeprim(0, @id, 1);
    }
    ```
- Delete by distance of input mesh:
    ```
    int pts =  npoints(@OpInput2);
    
    if ( @id >pts ) {
     @dead = 1;
    }
    ```
## Activate over time
- POP Wrangle
```
int activation_frame = int(fit01(random(@ptnum),1,100));
if (@Frame >= activation_frame)
  i@active = 1;
else
  i@active = 0;
```
## Age
- set an random age that ticks with time
```
int start = 950;
float tick = (@Frame-start)/24;
@age = fit01(rand(@ptnum),chf("min"),chf("max")) + tick;
@age = rint(1000*@age)/1000;
```
- Random age
  - set age before sim node
    ```
    @age = fit01(rand(@ptnum),0,2);
    ```
  - then in sim add age to group:
    ```
        if(i@group_active02 == 1)
    {
        @age = @age;
    }    
    else
    {

         @age= 0;
    }
    ```
  - Force by age:
    ```
    if(f@age >= 1.5){
    wind = wind;
    } else {
    wind  = set(0, 0, 0);
    }
    ```
## Set Rand Up Orient
- POP Wrangle
    ```
    @up = normalize(set(fit01(rand(@id+455),-1,1),fit01(rand(@id+58),-1,1),fit01(rand(@id+986),-1,1)));
    ```
## Orient
- Random rotation
    - Put this in a POP Spin node as a Vexpression
    ```
    // Pass Through
    // oldspinspeed also exists giving the
    // previous spin in degrees per second
    spinspeed = spinspeed;
    axis = axis;
    
    // The following makes it random:
    axis = rand(@id) - set(0.5, 0.5, 0.5);
    spinspeed *= rand(@id+0.1);
    ```
    - or
    ```
    vector v = rand(@id);
    v = fit01(v, -1,1);
    axis = v;
    amount *= fit01(rand(@id * 23456), 0.5, 1.5);
    ```
- Set Rand Up Orient
    - POP Wrangle
    ```
    @up = normalize(set(fit01(rand(@id+455),-1,1),fit01(rand(@id+58),-1,1),fit01(rand(@id+986),-1,1)));
    ```

## Color based on speed
- In POP network create POPColor
    ```
    ramp = lenght(@v);
    ```
## Slow down particles in Group
- Info from [wirginiaromanowska](https://github.com/wirginiaromanowska/Notes_On_Things/blob/master/Houdini_Particles.md#12-slow-down-particles-in-certain-grouop)
- in POP network create PopWrangle
    ```
    @v = @v*0.8;
    ```
    -20% drop in vel for each frame
  
## Scale by age
- outside of popnet
```
@pscale = chf("init_Scale") * chramp('Scale_by_age',(@age/@life));
```
## Color by normalised life
- outside of popnet, downstream
```
@Cd = vector(chramp("colorRamp",(@age/@life)));
```
## Align particle to velocity
- outside of popnet, downstream
    - append an attribdelete and delete 'orient N' to keep things clean
    ```
    @N = @v;
    ```
## Delete Particle by a ground plane
- in a pop wrangle as we want to remove the point entirely from the sim
```
if(@P.y <= (chf("Ground"))){
  removepoint(0,@ptnum);
  }
```
- Now you transfer attribute as the particle approaches the ground plane outside the sim with attributeTransfer
- example alter scale/rotation as it nears the ground.
## Add velocity for Motion Blur in Maya
- This is necessary if your point count is changing. Its not a limit with alembic, it just doesnt make sense to calculate velocity over a changing point count. How can you track point 5 if it suddenly exists somewhere else on the next frame
- you have to convert attributes to vertex attributes so they are exported correctly.
- create attribpromote
    - Original name = v Cd
    - point to Vertex
- add a wrangle on vertices
    ```
    v@velocity = v@v;
    setattribtypeinfo(0, "vertex", "velocity", "color");
    
    v@color = v@Cd;
    setattribtypeinfo(0, "vertex", "color", "color");
    ```
- trouble shooting:
    - the geo has to be unpacked, abc doesnt know what packed geo is. Check the GeoSpreadsheet to see if velocity is there 
- Youre gonna need to Add a trail sop
    - check  'compute velocity'
- In maya, go to MeshDisplay/ColourSetEditor and see if there are velocities
    - In AttributeEditor/MeshControls/MotionVectorColorSet, put 'velocity'
## Emitting trails on particle with limited life
- as the particles are deleting, we need to do this in the popnet with a pop replicate
    - in attributes or pop rep, detick 'add In attributes
    - this way the new particle inherits the origonal point
- after the popnet, append a delete sop
    - find the group 'stream_popreplicate1' and delete non-selected
    - this deletes the first point so the trail curve isnt closed
- append an add point node
    -in group tab, add by attribute
    -attribute name = id
- add a 'tangent name = N' in polyframe
- resample and tick 'Curve U Attribute = curveu'
- append a smooth
- add a width
- randomise a width
- polywire
## Blend 2 color ramps
- from [SK Particles II](https://vimeo.com/247882445)
```
float u = @age/@life;

vector colour1 = chramp("colour1", u);
vector colour2 = chramp("colour2", u);

@Cd = lerp (colour1, colour2, @bias);
```
## Sphere Sprite
- POP Sprite, sprite map = "sphere_matte.pic"
## Velocity
- random velocity by age
```
float randx = fit01(rand(@id+456),-.01,.01);
float randz = fit01(rand(@id+123),-.01,.01);
vector min = {0,.005,0};
vector max = {0,.01,0};

max.x = randx;
max.z = randz;

v@v = fit01(@age,max,min);
```
## Time in stream
- in a pop wrangle put:
    ```
    if (@enterstream == 0) f@enterstream = @Frame;
    f@timeinstream = @Frame - @enterstream;
    ```
    - branch another stream now that we have a timer:
    ```
    ingroup = @timeinstream > 24;
    ```
    - after 24 frames a particle will enter that stream
