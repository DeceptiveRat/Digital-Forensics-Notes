## 1. Prefetch
- utilized to improve application performance by pre-loading resources when application is launched
- provides:
	- evidence of execution
	-  list of modules/files accessed by process within 10 seconds following launch
- up to 1024 files for Win8+ <sup>[2]</sup>
- disabled by default in Windows servers<sup>[2]</sup>
- enabled by default in Windows workstations<sup>[2]</sup>

## 2. artifact location
- `C:\Windows\Prefetch\[ApplicationName]-[8-character_hash_of_path]`
	- for hosting applications, hash includes command line used to launch application <sup>[2]</sup>

## 3. prefetch file contents
- hash of original path 
- application name
- application run count
- timestamps for last 8 times application was run

## 4. significance
- multiple prefetch files with same path hash may indicate execution from same directory

## 5. analysis method
- PECmd.exe

## reference
[1] Windows Forensic Handbook, 2026/03/12, https://psmths.gitbook.io/windows-forensics/artifacts-by-type/filesystem-artifacts/prefetch
[2] (omayma, 2025/08/08), Windows Forensics:Prefetch, 2026/03/13, https://medium.com/@omaymaW/windows-forensics-prefetch-8447dbb6cd9b