## 1. Random pscale with range
- Attribute Wrangle
  ```
  @pscale = fit(rand(@ptnum),0 ,1 , ch("min_pscale"), ch("max_pscale"));
  ```
