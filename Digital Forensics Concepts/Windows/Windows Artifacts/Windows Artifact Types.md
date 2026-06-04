## 1. Windows Artifact Types
- 4 main categories:
	- registry
	- FS
	- event log
	- memory

## 2. registry artifacts
- found in Windows registry
- loaded into memory while system is in operation, written to disk during shutdown
- stores low-level config settings

## 3. FS artifacts
- created due to operation of FS, NTFS

## 4. event log artifacts
- found in Windows event log
- typically audit logs from OS and applications

## 5. memory artifacts
- must be collected from live system
- page files, hibernation files that consist of memory may be on disk as well

## reference
[1] (psmths, 2024) Windows Forensic Artifacts Guide, 2026/03/12, https://github.com/Psmths/windows-forensic-artifacts