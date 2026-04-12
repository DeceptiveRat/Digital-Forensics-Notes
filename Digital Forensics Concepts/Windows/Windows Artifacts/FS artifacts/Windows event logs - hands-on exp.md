## 1. generating logs
1. install *sysmon*
	![[installing_sysmon.png]]

2. run *injector.exe*
	![[run_injector.png]]
	![[injector_result.png]]

## 2. view sysmon event logs
- *injector.exe* creation event:
	![[injector_creation_event.png]]
	- *ProcessGUID* can be used to identify same program even with a different PID
	- *Image* of process can be located analyzed
	- hash of executable can be used to look up malware databases

- *CreateRemoteThread* event:
	![[RemoteThread_event.png]]
	- time stamp can be used to create timeline
	- *SourceProcessGuid* can be used to correlate program

- *ImageLoad* event:
	![[ImageLoad_event.png]]
	- timestamp shows it happened right after *CreateRemoteThread* event
	- *ProcessID* or *ProcessGuid* shows target to be same process
	- loaded dll file can be located
	- dll file hash can be looked up

## 3. files used
- *injector.exe* and *injectme.dll*: 
	- source: https://github.com/DeceptiveRat/malware-dev/tree/main/windows-dll_injector
	- hashes:
	``` sh
	$ sha256sum injector.exe 
	fbbd2f2fa1e5342653903c03ba2bd36f123cd7c9f2e006b57f273c7a6ccafc87  injector.exe
	$ sha256sum injectme.dll
	57365e08aafe89c58ce0730990ae4f08ee2ab1221bcf8e158c5b330a540f1dec  injectme.dll
	```
- *config.xml* file:
	``` xml
	<Sysmon schemaversion="4.90">
	  <EventFiltering>
		<RuleGroup name="DLL-Logging" groupRelation="or">
		  <ImageLoad onmatch="exclude">
			</ImageLoad>
		  <CreateRemoteThread onmatch="exclude">
		  </CreateRemoteThread>
		</RuleGroup>
	  </EventFiltering>
	</Sysmon>
	```
