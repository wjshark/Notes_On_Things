## Multi Redshift Proxy Instance
- add a wrangle after the scattered points
  ```
  int source_count = 4;
  int proxy_id = int(rand(@ptnum+456)*source_count)+1;
  s@instancefile = sprintf("$HIP/../proxy/Grain.%03d.rs",proxy_id);
  ```
