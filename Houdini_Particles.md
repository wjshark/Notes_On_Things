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
    

## 2. Deleting Partcles
- Add POP Wrangle
- Delete by Age Condition
  ```
  if (@age > .1) {
    dead = 1;
  }
  ```
- Delete by Age Condition Rand
  ```
  if (@age > rand(0.1,1.0)*15) {
      dead = 1;
  }
  ```
- Delete by ID
  ```
  if (@ID == 1) {
    dead = 1;
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
## 4. Age and Force by Age
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
## 7. Slow down particles in certain group
-in POP network create PopWrangle
  ```
  @v = @v*0.8;
  ```
  -20% drop in vel for each frame
