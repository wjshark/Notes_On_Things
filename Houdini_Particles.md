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
- POP Wrangle
- Delete by Age Condition
  ```
  if (@age > .1) {
    dead = 1;
  }
  ```
- Delete by Age Condition
  ```
  if (@age > rand(0.1,1.0)*15) {
      dead = 1;
  }
