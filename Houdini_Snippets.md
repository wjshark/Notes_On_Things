## 1. Random pscale with range
- Add Attribute Wrangle
```
@pscale = fit(rand(@ptnum),0 ,1 , ch("min_pscale"), ch("max_pscale"));
```
