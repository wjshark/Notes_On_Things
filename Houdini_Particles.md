## Topics
* [1. Emission](#1-emission)
* [2. Deleting Particles](#2-deleting-partcles)
* [3. Activate Over Time](#3-activate-over-time)
* [4 Random Age](#4-random-age-and-force-by-age)
* [5. Set Random Orient](#5-set-rand-up-orient)
* [6. Colour Based On Speed](#6color-based-on-speed)
* [7. Slow Particles In Group](#7-slow-down-particles-in-certain-group)
* [8. Scale By Age](#8-scale-by-age)
* [9. Colour By Normalised Age](#9-color-by-normalised-life)
* [10. Align Particle To Velocity](#10-align-particle-to-velocity)
* [11. Delete Particle By Ground Plane](#11-delete-particle-by-a-ground-plane)
* [12. Add Velocity For Motion Blur](#12-add-velocity-for-motion-blur-in-maya)
* [13. Trails](#13-emitting-trails-on-particle-with-limited-life)
* [14. Blend 2 Colour Ramps](#14-blend-2-color-ramps)


<br>

## 1. Emission
- Add to Emit impulse
  -Total Emission From Grid
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
    

## 2. Deleting Partcles
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
## 3. Activate over time
-POP Wrangle
  ```
  int activation_frame = int(fit01(random(@ptnum),1,100));
  if (@Frame >= activation_frame)
      i@active = 1;
  else
      i@active = 0;
  ```
## 4. Random Age and Force by Age
-POP Wrangle
- random age
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
  - then in force add:
    ```
    if(f@age >= 1.5){
    wind = wind;
    } else {
    wind  = set(0, 0, 0);
    }
    ```
## 5. Set Rand Up Orient
- POP Wrangle
  ```
  @up = normalize(set(fit01(rand(@id+455),-1,1),fit01(rand(@id+58),-1,1),fit01(rand(@id+986),-1,1)));
  ```
## 6.Color based on speed
-in POP network create POPColor
  ```
  ramp = lenght(@v);
  ```
## 7. Slow down particles in Group
- Info from [wirginiaromanowska](https://github.com/wirginiaromanowska/Notes_On_Things)
- in POP network create PopWrangle
  ```
  @v = @v*0.8;
  ```
  -20% drop in vel for each frame
  
## 8. Scale by age
- outside of popnet
```
@pscale = chf("init_Scale") * chramp('Scale_by_age',(@age/@life));
```
## 9. Color by normalised life
- outside of popnet, downstream
```
@Cd = vector(chramp("colorRamp",(@age/@life)));
```
## 10. Align particle to velocity
- outside of popnet, downstream
  - append an attribdelete and delete 'orient N' to keep things clean
  ```
  @N = @v;
  ```
## 11. Delete Particle by a ground plane
 - in a pop wrangle as we want to remove the point entirely from the sim
   ```
   if(@P.y <= (chf("Ground"))){
      removepoint(0,@ptnum);
      }
    ```
  - Now you transfer attribute as the particle approaches the ground plane outside the sim with attributeTransfer
    - example alter scale/rotation as it nears the ground.
## 12. Add velocity for Motion Blur in Maya
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

## 13. Emitting trails on particle with limited life
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
## 14. Blend 2 color ramps
- from [SK Particles II](https://vimeo.com/247882445)
  ```
  float u = @age/@life;

  vector colour1 = chramp("colour1", u);
  vector colour2 = chramp("colour2", u);

  @Cd = lerp (colour1, colour2, @bias);
  ```
