## 1. creating prefetch files
1. initial state
![[Digital Forensics Concepts/Windows/Windows Artifacts/FS artifacts/Prefetch - images/initial_state.png]]

2. execute *LogfileParser.exe*
![[execute_logfileparser.png]]

3. execute *RegistryExplorer.exe*
![[execute_registryexplorer.png]]

4. execute *cmd.exe* and create and append to *test.txt*
![[execute_cmd.png]]

## 2. parse prefetch files
![[parse_prefetch.png]]

## 3. view results
- *LOGFILEPARSER.EXE-7282EB65.pf*:
	```
	Created on: 2026-04-01 07:08:55
	Modified on: 2026-04-01 07:05:13
	Last accessed on: 2026-04-01 07:11:48

	Executable name: LOGFILEPARSER.EXE
	Hash: 7282EB65
	File size (bytes): 34,588
	Version: Windows 10 or Windows 11

	Run count: 1
	Last run: 2026-04-01 07:05:04

	Volume information:

	#0: Name: \VOLUME{01dcb2fbbef99864-eabf355a} Serial: EABF355A Created: 2026-03-13 15:12:02 Directories: 11 File references: 84

	Directories referenced: 11

	00: \VOLUME{01dcb2fbbef99864-eabf355a}\USERS
	01: \VOLUME{01dcb2fbbef99864-eabf355a}\USERS\HUGO
	02: \VOLUME{01dcb2fbbef99864-eabf355a}\USERS\HUGO\DOWNLOADS
	03: \VOLUME{01dcb2fbbef99864-eabf355a}\USERS\HUGO\DOWNLOADS\LOGFILEPARSER_V2.0.0.53
	[REMOVED]
	Files referenced: 57
	[REMOVED]
	08: \VOLUME{01dcb2fbbef99864-eabf355a}\USERS\HUGO\DOWNLOADS\LOGFILEPARSER_V2.0.0.53\LOGFILEPARSER.EXE (Executable: True)
	[REMOVED]
	11: \VOLUME{01dcb2fbbef99864-eabf355a}\$MFT
	[REMOVED]
	```
- *REGISTRYEXPLORER.EXE-393B4CF6.pf*:
	```
	Created on: 2026-04-01 07:08:55
	Modified on: 2026-04-01 07:06:16
	Last accessed on: 2026-04-01 07:11:48

	Executable name: REGISTRYEXPLORER.EXE
	Hash: 393B4CF6
	File size (bytes): 309,514
	Version: Windows 10 or Windows 11

	Run count: 1
	Last run: 2026-04-01 07:06:09

	Volume information:

	#0: Name: \VOLUME{01dcb2fbbef99864-eabf355a} Serial: EABF355A Created: 2026-03-13 15:12:02 Directories: 38 File references: 609

	Directories referenced: 38
	[REMOVED]
	14: \VOLUME{01dcb2fbbef99864-eabf355a}\USERS
	15: \VOLUME{01dcb2fbbef99864-eabf355a}\USERS\HUGO
	16: \VOLUME{01dcb2fbbef99864-eabf355a}\USERS\HUGO\DOWNLOADS
	17: \VOLUME{01dcb2fbbef99864-eabf355a}\USERS\HUGO\DOWNLOADS\REGISTRYEXPLORER
	[REMOVED]
	01: \VOLUME{01dcb2fbbef99864-eabf355a}\USERS\HUGO\DOWNLOADS\REGISTRYEXPLORER\REGISTRYEXPLORER.EXE (Executable: True)
	[REMOVED]
	275: \VOLUME{01dcb2fbbef99864-eabf355a}\USERS\HUGO\DOWNLOADS\REGISTRYEXPLORER\SETTINGS\GENERAL
	[REMOVED]
	```
- *CMD.EXE-89305D47.pf*:
	```
	Created on: 2026-04-01 07:08:55
	Modified on: 2026-04-01 07:06:54
	Last accessed on: 2026-04-01 07:11:48

	Executable name: CMD.EXE
	Hash: 89305D47
	File size (bytes): 9,228
	Version: Windows 10 or Windows 11

	Run count: 1
	Last run: 2026-04-01 07:06:44

	Volume information:

	#0: Name: \VOLUME{01dcb2fbbef99864-eabf355a} Serial: EABF355A Created: 2026-03-13 15:12:02 Directories: 8 File references: 25
	[REMOVED]
	01: \VOLUME{01dcb2fbbef99864-eabf355a}\WINDOWS\SYSTEM32\CMD.EXE (Executable: True)
	[REMOVED]
	```

- matches run time (+09:00:00 to change to KST)
- can see executable file creation time as well
- files and directories references are listed
- for *cmd.exe* activity creating *test.txt* was not recorded since it happened after 10 seconds have passed
