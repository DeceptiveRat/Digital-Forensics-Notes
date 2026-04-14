## 1. Windows registry analysis objectives
- locate registry files
- parse registry data
- discover forensically relevant registry keys

## 2. Windows registry DS
- `_CMHIVE` for hive file on disk:
	``` python
	>>> dt("_CMHIVE")
	'_CMHIVE' (1584 bytes)
	0x0 : Hive ['_HHIVE']
	0x2ec : FileHandles ['array', 6, ['pointer', ['void']]]
	0x304 : NotifyList ['_LIST_ENTRY']
	0x30c : HiveList ['_LIST_ENTRY']
	0x314 : PreloadedHiveList ['_LIST_ENTRY']
	0x31c : HiveRundown ['_EX_RUNDOWN_REF']
	0x320 : ParseCacheEntries ['_LIST_ENTRY']
	0x328 : KcbCacheTable ['pointer', ['_CM_KEY_HASH_TABLE_ENTRY']]
	[snip]
	0x3a0 : FileFullPath ['_UNICODE_STRING']
	0x3a8 : FileUserName ['_UNICODE_STRING']
	0x3b0 : HiveRootPath ['_UNICODE_STRING']
	[snip]
	```
	- `HiveList`: doubly linked list to other `_CMHIVE` structures
	- `FileFullPath`: kernel device path to registry hive; e.g. `\Device\HarddiskVolume1\WINDOWS\system32\config\software`
	- `FileUserName`: path to registry hive on disk; e.g. `\??\C:\Windows\System32\config\SOFTWARE`
	- `HiveRootPath`: registry path; e.g. `\REGISTRY\MACHINE\SOFTWARE`
- `_HHIVE` for hive header:
	``` python
	>>> dt("_HHIVE")
	'_HHIVE' (748 bytes)
	0x0 : Signature ['unsigned long']
	0x4 : GetCellRoutine ['pointer', ['void']]
	0x8 : ReleaseCellRoutine ['pointer', ['void']]
	0xc : Allocate ['pointer', ['void']]
	0x10 : Free ['pointer', ['void']]
	0x14 : FileSetSize ['pointer', ['void']]
	0x18 : FileWrite ['pointer', ['void']]
	0x1c : FileRead ['pointer', ['void']]
	0x20 : FileFlush ['pointer', ['void']]
	0x24 : HiveLoadFailure ['pointer', ['void']]
	0x28 : BaseBlock ['pointer', ['_HBASE_BLOCK']]
	[snip]
	0x58 : Storage ['array', 2, ['_DUAL']]
	```
	- `Signature`: signature of `0xbee0bee0`
	- `BaseBlock`: used to find root key of registry
	- `Storage`: mapping of VA spaces for keys within registry

## 3. data in registry
- auto-start programs
- hardware: external media devices connected to system
- user account information: user passwords, accounts, *Most Recently Used(MRU)* items, and user preferences
- recently run programs: using data from Userassist, Shimcache, and MUICache keys
- system information:
	- system settings
	- installed software
	- security patches 
- malware configurations: anything malicious code writes to registry; e.g. C2 data, infected files, encryption keys

## 4. stable and volatile registry
- some registry keys and hives are found only in memory
- if Windows APIs (`RegCreateKeyEx` and `RegSetValueEx`) are used, data is flushed to disk every 5 seconds
- if registry in memory is manipulated directly, changes do not get flushed to disk

## 5. `hivelist` plugin
- scans for registry hives:
	1. find structure in pool with tag `CM10` for `_CMHIVE`
	2. verify `Signature` 
	3. use `HiveList` to locate all other hives
- output example:
	``` sh
	$ python vol.py -f win7.vmem --profile=Win7SP0x86 hivelist
	Volatility Foundation Volatility Framework 2.4
	Virtual Physical Name
	---------- ---------- ----
	0x82b7a140 0x02b7a140 [no name]
	0x820235c8 0x203675c8 \SystemRoot\System32\Config\SAM
	0x87a1a250 0x27eb3250 \REGISTRY\MACHINE\SYSTEM
	0x87a429d0 0x27f9d9d0 \REGISTRY\MACHINE\HARDWARE
	0x87ac34f8 0x135804f8 \SystemRoot\System32\Config\DEFAULT
	0x88603008 0x20d36008 \??\C:\Windows\ServiceProfiles\NetworkService\NTUSER.DAT
	0x88691008 0x1ca1c008 \??\C:\Windows\ServiceProfiles\LocalService\NTUSER.DAT
	0x9141e9d0 0x1dc569d0 \??\C:\Windows\System32\config\COMPONENTS
	[snip]
	```
	- `[no name]` registry contains keys and symbolic links for `REGISTRY\A`, `REGISTRY\MACHINE`, and `REGISTRY\USER`

## 6. *Configuration Manager(CM)*
- registry file format:
	![[registry_file_format.png]]
- kernel component that manages registry and deals with address translation
- creates mappings between cell indexes and VA then stores in `_HHIVE.Storage` as `_DUAL` type:
	``` python
	>>> dt("_DUAL")
	'_DUAL' (220 bytes)
	0x0 : Length ['unsigned long']
	0x4 : Map ['pointer', ['_HMAP_DIRECTORY']]
	0x8 : SmallDir ['pointer', ['_HMAP_TABLE']]
	0xc : Guard ['unsigned long']
	0x10 : FreeDisplay ['array', 24, ['_RTL_BITMAP']]
	[snip]
	```
	- can obtain VA for registry key by following `Map` to `BlockAddress`:
		``` python
		>>> dt("_HMAP_DIRECTORY")
		'_HMAP_DIRECTORY' (4096 bytes)
		0x0 : Directory ['array', 1024, ['pointer', ['_HMAP_TABLE']]]
		>>> dt("_HMAP_TABLE")
		'_HMAP_TABLE' (8192 bytes)
		0x0 : Table ['array', 512, ['_HMAP_ENTRY']]
		>>> dt("_HMAP_ENTRY")
		'_HMAP_ENTRY' (16 bytes)
		0x0 : BlockAddress ['unsigned long']
		0x4 : BinAddress ['unsigned long']
		0x8 : CmView ['pointer', ['_CM_VIEW_OF_FILE']]
		0xc : MemAlloc ['unsigned long']
		```
- every cell index is broken up and used as indices into structures:
	- bit 0: indicates whether key is stable or volatile
	- bits 1~10: index into `Directory`
	- bits 11~19: index into `Table`
	- bits 20~31: offset within `BlockAddress` where key data resides (4 must be added to account for size of `Length` member)
	![[cell_breakdown.png]]

## 7. printing keys and values
- registry keys are stored as a tree-like structure
- to access a registry key and data, tree has to be traversed until leaf node is reached
- `_CM_KEY_NODE` structure:
	``` python
	>>> dt("_CM_KEY_NODE")
	'_CM_KEY_NODE' (80 bytes)
	0x0 : Signature ['String', {'length': 2}]
	0x2 : Flags ['unsigned short']
	0x4 : LastWriteTime ['WinTimeStamp', {}]
	0xc : Spare ['unsigned long']
	0x10 : Parent ['unsigned long']
	0x14 : SubKeyCounts ['array', 2, ['unsigned long']]
	0x1c : ChildHiveReference ['_CM_KEY_REFERENCE']
	0x1c : SubKeyLists ['array', 2, ['unsigned long']]
	0x24 : ValueList ['_CHILD_LIST']
	[snip]
	0x4c : Name ['String', {
	'length': <function <lambda> at 0x1017eb5f0>}]
	```

## 8. `prinkey` plugin
- finds all available registries in memory and accesses `SubKeyLists` and `ValueLists` members to traverse tree
- can print a key, subkeys, and values
- output example:
	``` sh
	$ python vol.py -f win7.vmem --profile=Win7SP1x86 printkey -K "controlset001\control\computername"
	Volatility Foundation Volatility Framework 2.4
	Legend: (S) = Stable (V) = Volatile
	----------------------------
	Registry: \REGISTRY\MACHINE\SYSTEM
	Key name: ComputerName (S)
	Last updated: 2011-10-20 15:25:16
	Subkeys:
	(S) ComputerName
	(V) ActiveComputerName
	Values:
	```
- limited to printing strings and integers; not sufficient for keys that contain binary or embedded data

## 9. detecting persistence
- system startup:
	- `HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\RunOnce`
	- `HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\Explorer\Run`
	- `HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run`
- user logons:
	- `HKCU\Software\Microsoft\Windows NT\CurrentVersion\Windows`
	- `HKCU\Software\Microsoft\Windows NT\CurrentVersion\Windows\Run`
	- `HKCU\Software\Microsoft\Windows\CurrentVersion\Run`
	- `HKCU\Software\Microsoft\Windows\CurrentVersion\RunOnce`
- services:
	- `HKLM\SYSTEM\CurrentControlSet\Services`

## 10. Volatility registry API
- allows easy processing of registry keys
- when `Registry API` object is instantiated, a dictionary of all registry files on memory sample is saved
- initialization:
	``` python
	>>> import volatility.plugins.registry.registryapi as registryapi
	>>> regapi = registryapi.RegistryApi(self._config)
	```
- printing subkeys of designated registry key:
	``` python
	>>> regapi.set_current(hive_name = "NTUSER.DAT", user = "administrator")
	>>> key = "software\\microsoft\\windows\\currentversion\\explorer"
	>>> for subkey in regapi.reg_get_all_subkeys(None, key = key):
	... 	print subkey.Name
	...
	Advanced
	BitBucket
	CabinetState
	CD Burning
	CLSID
	ComDlg32
	[snip]
	```
- getting registry value:
	``` python
	>>> k = "controlset001\\Control\\ComputerName\\ComputerName"
	>>> v = "ComputerName"
	>>> val = regapi.reg_get_value(hive_name = "system", key = k, value = v)
	>>> print val
	BOB-DCADFEDC55C
	```
- printing multiple registry values:
	``` python
	>>> k = "Microsoft\\Windows\\CurrentVersion\\Run"
	>>> regapi.set_current(hive_name = "software")
	>>> for value, data in regapi.reg_yield_values(hive_name = "software", key = k):
	... 	print value, "\n ", data
	...
	Adobe Reader Speed Launcher 
		"C:\Program Files\Adobe\Reader 9.0\Reader\Reader_sl.exe"
	Adobe ARM 
		"C:\Program Files\Common Files\Adobe\ARM\1.0\AdobeARM.exe"
	svchosts 
		C:\WINDOWS\system32\svchosts.exe
	```
- getting last 10 modified keys from admin's `NTUSER.DAT` registry:
	``` python
	>>> hive = "NTUSER.DAT"
	>>> for t, k in regapi.reg_get_last_modified(hive_name = hive, count = 10):
	... 	print t, k
	...
	2012-04-28 02:22:16 UTC+0000 $$$PROTO.HIV\Software\Microsoft\Windows\CurrentVersion\Explorer
	2012-04-28 02:21:41 UTC+0000 $$$PROTO.HIV\Software\Microsoft\Windows\CurrentVersion\Explorer\MountPoints2\##DC01#response
	2012-04-28 02:21:41 UTC+0000 $$$PROTO.HIV\Software\Microsoft\Windows\CurrentVersion\Explorer\MountPoints2
	2012-04-28 02:21:41 UTC+0000 $$$PROTO.HIV\Network\z
	2012-04-28 02:21:41 UTC+0000 $$$PROTO.HIV\Network
	2012-04-28 02:21:41 UTC+0000 $$$PROTO.HIV
	```
- printing subkeys and values:
	``` python
	>>> hive = "NTUSER.DAT"
	>>> for t, k in regapi.reg_get_last_modified(hive_name = hive, count = 10):
	...		print "LastWriteTime:", t
	...		print "Key:", k
	...		k = k.replace("$$$PROTO.HIV\\", "")
	...		for subkey in regapi.reg_get_all_subkeys(hive_name = hive, key = k):
	...		print "Subkey: ", subkey.Name
	...		for value, data in regapi.reg_yield_values(hive_name = hive, key = k):
	...		print "Value:", value, data
	...		print "*" * 20
	...
	[snip]
	LastWriteTime: 2012-04-28 02:21:41 UTC+0000
	Key: $$$PROTO.HIV\Software\Microsoft\Windows\CurrentVersion\Explorer\MountPoints2\##DC01#response
	Value: BaseClass Drive
	Value: _CommentFromDesktopINI
	Value: _LabelFromDesktopINI
	[snip]
	********************
	LastWriteTime: 2012-04-28 02:21:41 UTC+0000
	Key: $$$PROTO.HIV\Network\z
	Value: RemotePath \\DC01\response
	Value: UserName
	Value: ProviderName Microsoft Windows Network
	Value: ProviderType 131072
	```

## 11. Userassist keys
- used to determine what programs the user ran and when
- found in `NTUSER.DAT` registry of each user
- key path starting Windows 7:
	- `HKCU\software\microsoft\windows\currentversion\explorer\userassist\{CEBFF5CD-ACE2-4F4F-9178-9926F41749EA}\Count`
	- `HKCU\software\microsoft\windows\currentversion\explorer\userassist\{F4E57C4B-2036-45F0-A9AB-443BCFE33D9F}\Count`
- data is rot13 encoded
- `userassist` plugin prints translated data fit into data structure:
	``` sh
	$ python vol.py –f XPSP3x86.vmem --profile=WinXPSP3x86 userassist
	[snip]
	REG_BINARY UEME_RUNPATH:C:\WINDOWS\system32\cmd.exe :
	ID: 1
	Count: 1
	Last updated: 2010-02-26 03:42:15
	0x00000000 01 00 00 00 06 00 00 00 b0 41 5e b0 95 b6 ca 01
	```

## 12. Shimcache
- part of the *Application Compatibility Database*
- keys contain path for an executable and last modified timestamp from `$STANDARD_INFORMATION`
- key path: `HKLM\SYSTEM\CurrentControlSet\Control\Session Manager\AppCompatCache`
- older entries are overwritten with newer ones
- `shimcache` plugin prints processed data:
	``` sh
	$ python vol.py –f PhysicalMemory.001 --profile=Win2003SP2x86 shimcache
	Volatility Foundation Volatility Framework 2.4
	Last Modified Path
	------------------------------ ----
	[snip]
	2007-02-17 10:19:26 UTC+0000 \??\C:\WINDOWS\system32\inetsrv\iisrstas.exe
	2007-02-17 10:19:26 UTC+0000 \??\C:\WINDOWS\system32\iisreset.exe
	2007-02-17 10:59:04 UTC+0000 \??\C:\Program Files\Outlook Express\setup50.exe
	2009-03-08 11:32:52 UTC+0000 \??\C:\WINDOWS\system32\ieudinit.exe
	2010-07-22 07:47:49 UTC+0000 \??\C:\XXX\nv.exe
	2010-07-22 08:40:57 UTC+0000 \??\C:\XXX\123.exe
	2010-07-22 07:44:57 UTC+0000 \??\C:\XXX\dl.exe
	2010-07-22 07:46:41 UTC+0000 \??\C:\XXX\ow.exe
	[snip]
	2010-02-06 23:45:26 UTC+0000 \??\C:\WINDOWS\PSEXESVC.EXE
	[snip]
	2010-01-19 09:21:41 UTC+0000 \??\E:\XXX\sample.exe
	2010-01-19 09:02:26 UTC+0000 \??\E:\XXX\s.exe
	```

## 13. shellbags
- describes a collection of registry keys that allow Windows to track user window viewing preferences specific to Windows Explorer
- example artifacts:
	- window sizes and preferences
	- icon and folder view settings
	- metadata such as MAC timestamps
	- *Most Recently Used(MRU)* files and file type
	- files, folders, zip files, and installers that existed at one point on system
	- network shares and folders within them
	- information about TrueCrypt volumes
- `shellbags` plugin extracts data from appropriate keys:
	``` sh
	$ python vol.py –f XPSP3.vmem --profile=WinXPSP3x86 shellbags
	Volatility Foundation Volatility Framework 2.4
	Registry: \Device\HarddiskVolume1\Documents and Settings\User\NTUSER.DAT
	Key: Software\Microsoft\Windows\ShellNoRoam\BagMRU\0\0
	Last updated: 2011-06-03 04:24:36
	Value: 1
	Mru: 0
	File Name: DOCUME~1
	Modified Date: 2010-08-22 17:38:04
	Create Date: 2010-08-22 13:32:26
	Access Date: 2010-08-26 01:04:52
	File Attribute: DIR
	Path: C:\Documents and Settings
	------------------------------------------------------------
	Value: 0
	Mru: 1
	File Name: PROGRA~1
	Modified Date: 2010-08-25 23:04:02
	Create Date: 2010-08-22 13:32:48
	Access Date: 2010-08-25 23:04:22
	File Attribute: RO, DIR
	Path: C:\Program Files
	[snip]
	```
- `SHELLITEM` entries remain in registry even after file has been deleted
- timestamps associated with `SHELLITEM` are not updated
- `ITEMPOS` entries are updated if file is moved, deleted, or accessed

## 14. timestomping registry keys
- registry key timestamps can be overwritten
- example of timestomped registry key:
	``` sh
	$ python vol.py –f XPSP3x86.vmem --profile=WinXPSP3x86 shellbags
	[snip]
	Registry: \Device\HarddiskVolume1\Documents and Settings\user\NTUSER.DAT
	Key: Software\Microsoft\Windows\ShellNoRoam\Bags\63\Shell
	Last updated: 3024-05-21 00:00:00
	------------------------------------------------------------
	Value: ItemPos1567x784(1)
	File Name: NEWTEX~1.TXT
	Modified Date: 2012-08-17 14:14:56
	Create Date: 2012-08-17 14:14:50
	Access Date: 2012-09-25 11:49:38
	File Attribute: ARC
	Path: New Text Document.txt
	```

## 15. `hashdump` plugin
- used to dump hashes stored in `SYSTEM` and `SAM` hives
- LM hash and NT hash are stored in `SAM`; LM hash is now obsolete due to design flaw
- output example:
	``` sh
	$ python vol.py -f Bob.vmem --profile=WinXPSP3x86 hashdump
	Volatility Foundation Volatility Framework 2.4
	Administrator:500:e52cac67419a9a2[snip]:8846f7eaee8fb117ad06bdd830b7586c:::
	Guest:501:aad3b435b51404eeaad3b43[snip]:31d6cfe0d16ae931b73c59d7e0c089c0:::
	HelpAssistant:1000:9f8ac2eaebcd2e[snip]:d95e38a172b3ddaa1ce0b63bb1f5e1fb:::
	SUPPORT_388945a0:1002:aad3b435b51[snip]:ad052c1cbab3ec2502df165cd25d95bd::
	$ john hashes.txt
	Loaded 6 password hashes with no different salts (LM DES [128/128 BS SSE2-16])
	(SUPPORT_388945a0)
	(Guest)
	PASSWOR (Administrator:1)
	D (Administrator:2)
	[interrupted]
	$ john --show hashes.txt
	Administrator:PASSWORD:500:8846f7eaee8fb117ad06bdd830b7586c:::
	Guest::501:31d6cfe0d16ae931b73c59d7e0c089c0:::
	SUPPORT_388945a0::1002:ad052c1cbab3ec2502df165cd25d95bd:::
	4 password hashes cracked, 2 left
	```

## 16. `lsadump` plugin
- dumps decrypted LSA secrets from registry
- uses both `SYSTEM` and `SECURITY` hives
- items in LSA secrets:
	- `$MACHINE.ACC`: domain authentication
	- `DefaultPassword`: password used to log on when auto-login is enabled
	- `NL$KM`: secret key used to encrypt cached domain passwords
	- `L$RTMTIMEBOMB_*`: timestamp giving date when unactivated copy of Windows will stop working
	- `L$HYDRAENCKEY_*`:
		- private key used for *Remote Desktop Protocol(RDP)*
		- together with client public key from packet capture, can decrypt traffic
- output example:
	``` sh
	$ python vol.py -f XPSP3x86.vmem --profile=WinXPSP3x86 lsadump
	Volatility Foundation Volatility Framework 2.4
	[snip]
	L$HYDRAENCKEY_28ada6da-d622-11d1-9cb9-00c04fb16e75
	0x00000000 52 53 41 32 48 00 00 00 00 02 00 00 3f 00 00 00 RSA2H.......?...
	0x00000010 01 00 01 00 f1 93 70 67 69 62 de d1 aa f0 99 67 ......pgib.....g
	0x00000020 83 bb 95 20 a0 de 05 a7 40 7b 7e 5e a9 d2 f5 bd ........@{~^....
	0x00000030 52 37 18 c2 b5 6d f0 78 b3 cc 7e e0 b8 b7 70 01 R7...m.x..~...p.
	0x00000040 33 bf fb 3d 75 69 d8 e1 84 b4 ab b8 bc 82 63 d9 3..=ui........c.
	0x00000050 17 d3 80 d6 00 00 00 00 00 00 00 00 4d 40 cd 12 ............M@..
	0x00000060 c1 18 93 a6 ec a8 99 03 cb f7 76 ab bb 6d e8 63 ..........v..m.c
	[snip]
	$ python vol.py -f Win7SP1x64.raw --profile=Win7SP1x64 lsadump
	Volatility Foundation Volatility Framework 2.4
	DefaultPassword
	0x00000000 00 00 00 01 7e a3 eb 47 10 31 8b 1f 6b 54 65 5c ....~..G.1..kTe\
	0x00000010 23 67 b1 dd 03 00 00 00 00 00 00 00 3b 7e b7 96 #g..........;~..
	0x00000020 d5 98 fa 71 32 24 24 b5 92 a0 8a cb 40 43 b5 24 ...q2$$.....@C.$
	0x00000030 19 90 dd e3 15 96 f4 34 4e 8b 75 ea a0 49 b4 4f .......4N.u..I.O
	0x00000040 08 eb 90 ec e3 0a 7c 3d c7 87 f7 ef 3f 8a 5f ad ......|=....?._.
	0x00000050 c1 d7 f2 8f 01 99 98 c3 e1 8e 97 c9 ............
	DPAPI_SYSTEM
	0x00000000 00 00 00 01 7e a3 eb 47 10 31 8b 1f 6b 54 65 5c ....~..G.1..kTe\
	0x00000010 23 67 b1 dd 03 00 00 00 00 00 00 00 2b 04 ff 76 #g..........+..v
	0x00000020 30 d3 c5 53 7b 8c 98 15 92 9b ab ec 68 83 7e cd 0..S{.......h.~.
	0x00000030 f8 f8 17 6b ba 6a 68 f2 28 57 17 1a 89 1d f7 fd ...k.jh.(W......
	0x00000040 e9 97 32 fc a3 61 ce bc a1 3c 95 b6 d2 11 9b 98 ..2..a...<......
	0x00000050 77 10 c9 fd 95 86 60 09 68 83 9f b0 38 ff 01 3c w.....`.h...8..<
	0x00000060 30 04 b5 47 8d eb 8c 85 2b 69 03 1b 60 67 9c 34 0..G....+i..`g.4
	0x00000070 fa a5 0d 1f b5 eb 88 ea 82 92 28 40 ..........(@
	```
