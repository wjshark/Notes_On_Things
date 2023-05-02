## Naming
- Get Layer name below
  ```
  const layerBelow = thisComp.layer(index + 1);
  text.sourceText = layerBelow.toString();
  ```

- Get current layer name
  ```
  "Source : " + thisLayer.effect("SourceLayer")("Layer").name;
  ```
