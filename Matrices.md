## Matrices and Local Space transforms
- can be can be 2x2 or 3x3 or 4x4 matrix

## Identity Matrix (also called unit matrix)
- type of matrix that
- if you a shape multiply by this matrix it will remain the same shape, equilivent of multiplying by 1.
- it outputs the input postions
- example:
  - the diagional line OF 1's indicates this is an Identity Matrix  
    ```
    1 0 0 0
    0 1 0 0
    0 0 1 0
    0 0 0 1
    ```
- a 3x3 matrix is called a Rotation Matrix
  - the x's in this 3x3 matrix define the rotation  
  - the T's define the translation
    ```
    X X X C
    X X X C
    X X X C
    T T T C
    ```
  
- example
    - houdini reads radians so we have to convert degree to radians
    - an euler angle is an angle defined by a 3d vector and direction
    - quaternions are better
    - 'matrix' by default defines a 4x4 matrix 
      ```
      float angle_in_deg = 45; // or add chf("rotate_x")
      float angle_in_rad = radians (angle_in_deg); // houdini reads radians
      vector euler_angle = set(angle_in_deg, 0 , 0);
      vecrot 4 quat =  eulertoquaternion(euler_angle, 0); // (angle to convert, rotation order xyz)
      matrix3 rotM = qconvert(quat); // convert quaternions into a rotation matrix (3x3 matrix)
      matrix m = set(rotM); // sets 3x3 matrx in a 4x4 matrix , puts 0 into translation
      @P *= m; // multiply position by matrix to ge the rotation
      ```
