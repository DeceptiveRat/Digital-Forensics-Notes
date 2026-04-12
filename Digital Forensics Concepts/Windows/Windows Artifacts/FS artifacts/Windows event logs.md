## 1. Windows event log
- record of events related to system, security, and application
- kept under `C:\WINDOWS\system32\config\folder`

## 2. event log elements
- log name
- event date/time
- task category
- event id
- source
- level
- user 
- computer

## 3. event log categories
- system log
- application log
- security
- setup
- forwarded events

## 4. event log types
- information: app or service working well
- warning: potential issues in the future
- error: significant issue
- success audit: valid attempt of audited security access
- failure audit: failure of audited security access

## 5. sysmon
- addon for windows logging
- capabilities:
	- log process creation with full command line for current and parent process
	- record hash of process image files
	- includes process GUID in process creation events; allows correlation independent of PID
	- inlcudes session GUID
	- logs loading of drivers/DLLs with hash
	- logs opens for raw read access of disks/volumes
	- logs network connections
	- detects changes in file creation time
	- reload configuration if changed in registry
	- generates events from early in boot process; can capture kernel-mode malware

## reference
[1] What is Windows Event Log?, 2026/04/02, https://www.solarwinds.com/resources/it-glossary/windows-event-log
[2] (Mark Sussinovich and Thomas Garnier, 2026/03/26) Sysmon v15.2, 2026/04/02, https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon
