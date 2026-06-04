## 1. Shimcache
- aka AppCompatCache
- provides compatibility for old applications
- record:
	- executable file name
	- file path
	- last modification date/time
- executables on removable media and *Universal Naming Convention(UNC)* paths are also stored 
- size of 1024 entries

## 2. location
- source file
`C:\Windows\System32\config\SYSTEM`
- registry key
`HKLM\SYSTEM\CurrentControlSet\Control\SessionManager\AppCompatCache\AppCompatCache`

## reference
[1] (2022/08/25) Investigating ShimCache with ArtiFast ShimCache Artifact Parser, 2026/03/13, https://forensafe.com/blogs/shimcache.html
