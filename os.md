## dir map
- This script will swap the drive letters in memory leaving the maya scene unchanged. It'll keep referenced character paths intact      preventing the need to relink character references.
- Directories must both be absolute paths, and should be separated with forward slashes ('/')
- Run this script in the script editor of an empty scene, then open the project file.
  - The first argument is the directory to map, and the second is the destination directory to map to.
  - The mapping is case-sensitive on all platforms.
  - The directory mappings and enabled state are not preserved between Maya sessions.
```
dirmap -en true;
dirmap -m "C:/PROJECTS/FREELANCE/WORK/JOB" "T:/PROJECTS_2019/STUDIO/JOB";
```
[http://download.autodesk.com/us/maya/2010help/Commands/dirmap.html]
