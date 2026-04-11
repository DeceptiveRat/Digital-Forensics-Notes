## 1. Windows services
- usually non-interactive
- run consistently in background
- often run with higher privileges than most programs
- examples:
	- event-logging facility
	- print spooler
	- host firewall
	- time daemon
	- Windows Defender
	- Security Center
- malicious code often leverage services for persistence

## 2. service architecture
- list of installed services and configurations is stored in `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\services` key:
	- each services has a dedicated subkey with various values describing:
		- how and when service starts
		- whether the service is for a process, DLL, or kernel driver
		- service specific settings
- services implemented as DLLs run from a shared host process (*svchost.exe*)
- Windows service components:
	![[Windows_service_components.png]]
	- upon system boot, *Service Control Manager(SCM)* reads registry to launch services configured for auto-start
	- some services specify dependencies; other services must be online for them to start
	- SCM(*services.exe*) creates linked list of service record structures that store service's current state and associated PIDs in its memory
- only services that run with the same privilege levels should share a host process; otherwise one service can access memory of other services

## 3. installing services
- it is important to be able to identify service installation capabilities 
- methods:
	- manual commands:
		- `sc create` and `sc start` commands in shell can create and start services
		- leaves command history artifacts as well as service artifacts
	- batch scripts:
		- additional disk artifacts are created for script file
		- example:
			``` batch
			@echo off
			set SERVICENAME="MyService"
			set BINPATH="C:\windows\system32\MyService.dll"
			sc create "%SERVICENAME%" binPath= "%SystemRoot%\system32\svchost.exe \
			-k %SERVICENAME%" type= share start= auto
			reg add "HKLM\System\CurrentControlSet\Services\%SERVICENAME%\Parameters" \
			/v ServiceDll /t REG_EXPAND_SZ /d "%BINPATH%" /f
			reg add "HKLM\Software\Microsoft\Windows NT\CurrentVersion\SvcHost" \
			/v %SERVICENAME% /t REG_MULTI_SZ /d "%SERVICENAME%\0" /f
			sc start %SERVICENAME%
			```
			- creates service named `MyService`
			- configures new service to load *MyService.dll* on boot
	- Windows APIs:
		- `CreateService` and `StartService` can be directly imported by malware written in C or C++
		- before `CreateService` returns, a new subkey is added to the registry with configuration
		- `StartService` leaves traces in the event log
	- *Windows Management Instrumentation(WMI)*:
		- high level interface that allows interaction with services
		- exposes `Win32_Service` class with `Create` and `Start` methods

## 4. stealth installation
- can bypass `CreateService` by manually creating registry keys
- `NdrClientCall2` (RPC interface called internally by `StartService`) can be called to start service
- `NtLoadDriver` can start service for kernel driver
- registry keys and values can be deleted to remove evidence:
	![[TDL3_stealth_service.png]]
	- calls `NtLoadDriver` to bypass `StartService`
	- removes evidence in registry via `SHDeleteKeyA`

## 5. stopping services
- certain services are stopped by malware to lower security:
	- Wscsvc (Windows Security Center Service)
	- BITS (Background Intelligent Transfer Service)
	- WinDefend (Windows Defender Service)
- can be done via `ControlService` API function or batch file with commands like `net stop SERVICENAME`
- `TerminateProcess` on service process can lead to stability issues so it is not done often

## 6. live system service investigation
- *Microsoft Management Console(MMC)* can be used to view services on a system
- `Get-Service` powershell command:
	``` powershell
	PS C:\Users\Jake> Get-Service
	Status Name DisplayName
	------ ---- -----------
	Stopped AdobeARMservice Adobe Acrobat Update Service
	Running AeLookupSvc Application Experience
	Stopped ALG Application Layer Gateway Service
	[snip]
	```
- `sc query` command:
	``` cmd
	C:\Users\Jake> sc query
	SERVICE_NAME: Appinfo
	DISPLAY_NAME: Application Information
		TYPE : 20 WIN32_SHARE_PROCESS
		STATE : 4 RUNNING (STOPPABLE, NOT_PAUSABLE, IGNORES_SHUTDOWN)
		WIN32_EXIT_CODE : 0 (0x0)
		SERVICE_EXIT_CODE : 0 (0x0)
		CHECKPOINT : 0x0
		WAIT_HINT : 0x0
	SERVICE_NAME: AudioEndpointBuilder
	DISPLAY_NAME: Windows Audio Endpoint Builder
		TYPE : 20 WIN32_SHARE_PROCESS
		STATE : 4 RUNNING (STOPPABLE, NOT_PAUSABLE, IGNORES_SHUTDOWN)
		WIN32_EXIT_CODE : 0 (0x0)
		SERVICE_EXIT_CODE : 0 (0x0)
		CHECKPOINT : 0x0
		WAIT_HINT : 0x0
	[snip]
	```

## 7. Windows service analysis objectives
- determine recently created services
- detect services in invalid states
- identify hijacked services:
	- path to service binary in registry can be overwritten
	- service binary on disk can be patched
- locate hidden service records

## 8. Windows service DS
- 64-bit Windows 7 `_SERVICE_HEADER`:
	``` python
	>>> dt("_SERVICE_HEADER")
	'_SERVICE_HEADER' (None bytes)
	0x0 : Tag ['array', 4, ['unsigned char']]
	0x10 : ServiceRecord ['pointer', ['_SERVICE_RECORD']]
	```
	- `Tag`: fixed value `sErv` or `serH` that identifies service record headers

- 64-bit Windows 7 `_SERVICE_RECORD`:
	``` python
	>>> dt("_SERVICE_RECORD")
	'_SERVICE_RECORD'
	0x0 : PrevEntry ['pointer', ['_SERVICE_RECORD']]
	0x8 : ServiceName ['pointer', ['String', {'length': 512, encoding': 'utf16'}]]
	0x10 : DisplayName ['pointer', ['String', {'length': 512, 'encoding': 'utf16'}]]
	0x18 : Order ['unsigned int']
	0x20 : Tag ['array', 4, ['unsigned char']]
	0x28 : DriverName ['pointer', ['String', {'length': 256, 'encoding': 'utf16'}]]
	0x28 : ServiceProcess ['pointer', ['_SERVICE_PROCESS']]
	0x30 : Type ['Flags', {'bitmap': svc_types}]
	0x34 : State ['Enumeration', {'target': 'long', 'choices': svc_states}]
	```
	- `ServiceName`: pointer to Unicode string containing service name
	- `DisplayName`: more descriptive name for service
	- `FullServicePath`:
		- Unicode string containing name of driver object for FS/kernel driver service
		- `_SERVICE_PATH` structure containing full path on disk to executable file and current PID for process service
- 64-bit Windows 7 `_SERVICE_PROCESS`:
	``` python
	>>> dt("_SERVICE_PROCESS")
	'_SERVICE_PROCESS'
	0x10 : BinaryPath ['pointer', ['String', {'length': 256, 'encoding': 'utf16'}]]
	0x18 : ProcessId ['unsigned int']
	```
- a service's position in the list is indicative of when SCM read configuration from registry, not order of service start
- services created after system boot are appended to the end of the list

## 9. `svcscan` plugin
- leverages linked list, but also performs brute force scan through memory owned by *services.exe* for `sErv` or `serH`
- can't find services that are started inappropriately; e.g. `NtLoadDriver` function

## 10. service hijacking
- registry-based hijack:
	- change `ImagePath` or `ServiceDll` registry value to point to malicious file
	- hijack example:
		``` sh
		$ python vol.py –f memory.dmp --profile=Win7SP0x64 svcscan --verbose
		Volatility Foundation Volatility Framework 2.4
		[snip]
		Offset: 0x992c30
		Order: 34
		Process ID: 892
		Service Name: ERSvc
		Display Name: Error Reporting Service
		Service Type: SERVICE_WIN32_SHARE_PROCESS
		Service State: SERVICE_RUNNING
		Binary Path: C:\Windows\system32\svchost.exe -k netsvcs
		ServiceDll: %SystemRoot%\system\ersvc.dll
		```
		- `ServiceDll` should be `%SystemRoot%\system32\ersvc.dll`
	- keep track of legitimate paths by building a whitelist of good paths for service binaries from a clean system
- disk-based hijack:
	- query registry to get binary path and replace then restart system
	- to detect, look for artifacts the new DLL creates when it loads, or dump DLLs and compare file size, PE header compilation timestamp, etc to baseline copies; hashes cannot be used as they will never match

## 11. hiding services
- *Blazgel* disassembled:
	![[Blazgel_disassembled.png]]
	- finds service then unlinks it from the list
- unlinking services with *UnlinkServiceRecord.exe*:
	``` cmd
	C:\>UnlinkServiceRecord.exe wscsvc
	[!] Service to hide: wscsvc
	[!] SCM Process ID: 0x28c
	[!] Found PsServiceRecordListHead at 0x6e1e90
	[!] Found a matching SERVICE_RECORD structure at 0x6ea3d0!
	C:\>sc query wscsvc
	[SC] EnumQueryServicesStatus:OpenService FAILED 1060:
	The specified service does not exist as an installed service
	```
	![[unlinked_service.png]]
