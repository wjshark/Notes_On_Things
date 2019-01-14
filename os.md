## dir map
- make sure all forward slashes
- run script in empty scene, then open the project file
- swaps the old driver letter with the new one in memory without change the scene reference paths.
```
dirmap -en true;
dirmap -m "to this path" "swaps this path";
```
```
dirmap -en true;
dirmap -m "C:/PROJECTS/FREELANCE/WORK/JOB" "T:/PROJECTS_2018/STUDIO/JOB";
```
[http://download.autodesk.com/us/maya/2010help/Commands/dirmap.html]
