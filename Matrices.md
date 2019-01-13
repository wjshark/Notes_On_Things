## Matrices
- A matrix is a rectangular array of numbers, symbols, or expressions, arranged in rows and columns.
- A major application of matrices is to represent linear transformations
  - For example, the rotation of vectors in three-dimensional space is a linear transformation
- Two matrices can be multiplied only when the number of columns in the first equals the number of rows in the second
  ```
  AB = ( -1 2 3 ) ( 2  1 )    
       (  4 0 5 ) ( 8 -1 )
                  ( 6  5 )
  ```
  - Dimensions are (2  x **3**)(**3** x 2)
  - The **inner** numbers are the same so it can be multiplied
  - the result of the multiplications will have dimensions equal to the outer numbers (2 x 2)

- Dimensions of a matrix can be 2x2, 3x3, 4x4, 50x50, infinity)X(infinity) matrix. 
- Matrix multiplication is not commutative, changing the order of the operands **does** change the result.

## Identity Matrix (also called unit matrix)
- all the elements on the main diagonal are equal to 1 and all other elements are equal to 0
- It is called an identity matrix because multiplication with it leaves a matrix unchanged
- example:
  - the diagional line of 1's indicates this is an Identity Matrix  
    ```
    1 0 0 0
    0 1 0 0
    0 0 1 0
    0 0 0 1
    ```
- a 3x3 matrix is called a Rotation Matrix
- this is a 4x4:
  - the R's define the rotation  
  - the T's define the translation
  - the D's define the Determinant: it can be viewed as the scaling factor of the linear transformation described by the matrix
    ```
    R R R D
    R R R D
    R R R D
    T T T D
    ```
## Transforming by Matrix 
- example
    - houdini reads radians so we have to convert degree to radians
    - an euler angle is an angle defined by a 3d vector and direction
    - quaternions are better
    - 'matrix' by default defines a 4x4 matrix 
      ```
      float angle_in_deg = 45; // or add chf("rotate_x")
      float angle_in_rad = radians (angle_in_deg); // houdini reads radians
      vector euler_angle = set(angle_in_deg, 0 , 0);
      vector4  quat =  eulertoquaternion(euler_angle, 0); // (angle to convert, rotation order xyz)
      matrix3 rotM = qconvert(quat); // convert quaternions into a rotation matrix (3x3 matrix)
      matrix m = set(rotM); // sets 3x3 matrx in a 4x4 matrix , puts 0 into translation
      @P *= m; // multiply position by matrix to ge the rotation
      ```
    - to translate locally on y axis, you must invert or reset the rotation to world space, move it and multiply it back:
      ```
      float angle_in_deg = chf("rotate_x"); 
      float angle_in_rad = radians (angle_in_deg);
      vector euler_angle = set(angle_in_deg, 0 , 0);
      vector4 quat =  eulertoquaternion(euler_angle, 0);
      matrix3 rotM = qconvert(quat);
      matrix m = set(rotM);
      @P *= m;

      @P *=invert(m); // resets the rotation
      matrix translate_matrix = ident(); 
      translate(translate_matrix, set(0,chf("y_translate"),0));//this function does not return any value(void)
      translate_matrix*=m; // multiplying the translate_matrix buy the Rot matrix
      @P *= translate_matrix;
      ```
    - you can keep layering this example but its long winded
      ```
      float angle_in_deg = chf("rotate_x"); 
      float angle_in_rad = radians (angle_in_deg);
      vector euler_angle = set(angle_in_deg, 0 , 0);
      vector4 quat =  eulertoquaternion(euler_angle, 0);
      matrix3 rotM = qconvert(quat);
      matrix m = set(rotM);
      @P *= m;

      @P *= invert(m); // resets the rotation
      matrix translate_matrix = ident(); 
      translate(translate_matrix, set(0,chf("y_translate"),0)); 
      translate_matrix *= m;
      @P *= translate_matrix;

      @P *= invert(translate_matrix);
      float angle_in_deg_z = chf("rotate_z"); 
      float angle_in_rad_z = radians (angle_in_deg_z);
      vector euler_angle_z = set(0, 0 , angle_in_deg_z);
      vector4 quat_z =  eulertoquaternion(euler_angle_z, 0);
      matrix3 rotM_z = qconvert(quat_z);
      matrix m_z = set(rotM_z);
      m_z *= translate_matrix;    
      @P *= m_z;
      ```

## Transforming by Matrix (the better way)
- we transform the point with a matrix
- before we make another transformation, we check if there is a detail attribute
  - the inputXform is the matrix on the geo
  - we invert the points and the matrix to reset before another rotation
  - then we apply the initial tranformation again and update or set() the "xform" (inputXform) for the matrix
    ```
    float angleX = chf("rotate_x");
    float angleY = chf("rotate_y");
    float angleZ = chf("rotate_z");

    float angleRadX = radians(angleX);
    float angleRadY = radians(angleY);
    float angleRadZ = radians(angleZ);

    vector euler = set(angleRadX, angleRadY, angleRadZ);
    vector4 quat = eulertoquaternion(euler, 0);
    matrix3 m3 = qconvert(quat);
    matrix m = set(m3); // created the transformation 

    // we put the rot in geo as a detail attribute, so we need to check if it exists
    // if exists; we apply the inverse multiplication before we set the rot to the points

    if (hasdetailattrib(0, "xform")){ // input geo and the attribute
        matrix inputXform = detail(0 ,"xform"); // matrix is the detail attribute of input geo
        @P*=invert(inputXform); // multiply points back to the invert of xform
        m*=inputXform;// multiply the matix by the inputXform
    }

    @P*=m;// set back to points
    setdetailattrib(0,"xform", m);
    ```
  - and the same with translation
    ```
    float translateX = chf("translate_x");
    float translateY = chf("translate_y");
    float translateZ = chf("translate_z");

    matrix m = ident();
    translate(m, set(translateX, translateY, translateZ));

    if (hasdetailattrib(0, "xform")){
        matrix inputXform = detail(0 ,"xform");
        @P*=invert(inputXform);
        m*=inputXform;
    }

    @P*=m;// set back to points
    setdetailattrib(0,"xform", m);
    ```
