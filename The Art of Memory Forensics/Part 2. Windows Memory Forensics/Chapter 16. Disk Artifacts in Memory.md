## 1. *Master File Table(MFT)*
- special file at root of FS (`\$Mft`)
- stores critical information about all other files on partition
- all or part can be found in memory

## 2. MFT analysis objectives
- find and parse MFT entries
- investigate removable media: MFT entries describing files from removable media can be found as well
- recover *Alternate Data Streams(ADS)*
- recover attacker scripts: batch scripts often fit in MFT entry
- reconstruct events
- prove code execution: analyze prefetch files
- track user activity

## 3. `mftparser` plugin
- extracts MFT entries by scanning for `FILE` and `BAAD` signatures
- attributes supported:
	- `$FILE_NAME`
	- `$STANDARD_INFORMATION`
	- `$DATA`
- example output:
	``` data
	$ python vol.py –f Win7SP1x64.dmp --profile=Win7SP1x64 mftparser --output-file=mftverbose.txt
	Volatility Foundation Volatility Framework 2.4
	[snip]
	*******************************************************
	MFT entry found at offset 0x2a416000
	Attribute: In Use & File
	Record Number: 22052
	Link count: 1
	$STANDARD_INFORMATION
	Creation: 2013-03-10 23:24:45 UTC+0000
	Modified: 2013-03-10 23:28:49 UTC+0000
	MFT Altered: 2013-03-10 23:28:49 UTC+0000
	Access: 2013-03-10 23:24:45 UTC+0000
	Type: Archive
	$FILE_NAME
	Creation: 2013-03-10 23:24:45 UTC+0000
	Modified: 2013-03-10 23:24:45 UTC+0000
	MFT Altered: 2013-03-10 23:24:45 UTC+0000
	Access: 2013-03-10 23:24:45 UTC+0000
	Name/Path: Users\Andrew\Desktop\log.txt
	$DATA
	0000000000: 3c3f786d6c2076657273696f6e3d2231 <?xml.version="1
	0000000010: 2e30223f3e0a3c656e7472793e3c7469 .0"?>.<entry><ti
	0000000020: 6d653e332f31302f3230313320363a32 me>3/10/2013.6:2
	0000000030: 353a333520504d3c2f74696d653e3c6b 5:35.PM</time><k
	0000000040: 6579733e623352714f446c7664476f34 eys>b3RqODlvdGo4
	0000000050: 4f54466f63334d756148527949436858 OTFoc3MuaHRyIChX
	0000000060: 616e6c3664334d70494368485a6e4e77 anl6d3MpIChHZnNw
	0000000070: 4948527249455a79616e647561475967 IHRrIEZyanduaGYg
	0000000080: 664342556333467563326f6752325a7a fCBUc3Fuc2ogR2Zz
	0000000090: 6347357a624342384946687562484d67 cG5zbCB8IFhubHMg
	00000000a0: 546e4d67664342556333467563326f67 TnMgfCBUc3Fuc2og
	00000000b0: 546b6b674c53424362696b3d3c2f6b65 TkkgLSBCbik=</ke
	00000000c0: 79733e3c2f656e7472793e0d0a3c656e ys></entry>..<en
	00000000d0: 7472793e3c74696d653e332f31302f32 try><time>3/10/2
	00000000e0: 30313320363a32383a343920504d3c2f 013.6:28:49.PM</
	00000000f0: 74696d653e3c6b6579733e5a33526e4d time><keys>Z3RnM
	0000000100: 54497a5a33526e4d585530654867794d TIzZ3RnMXU0eHgyM
	0000000110: 48647064327070615735354c6d683063 Hdpd2ppaW55Lmh0c
	0000000120: 6938766479397a616e6c34616d67674b i8vdy9zanl4amggK
	0000000130: 4664716558703363796b674b464a6d63 FdqeXp3cykgKFJmc
	0000000140: 325a73616942485a6e4e77626e4e7349 2ZsaiBHZnNwbnNsI
	0000000150: 4359675557707a6157357a6243424761 CYgUWpzaW5zbCBGa
	0000000160: 476830656e4e35654342384945686d64 Gh0enN5eCB8IEhmd
	0000000170: 5735355a6e456756484e714946527a63 W55ZnEgVHNqIFRzc
	0000000180: 57357a616942485a6e4e774b513d3d3c W5zaiBHZnNwKQ==<
	0000000190: 2f6b6579733e3c2f656e7472793e0d0a /keys></entry>..
	********************************************************
	```
- `-D/--dump-dir` can be used to dump resident files to disk

## 4. ADS
- used to associate security zones with downloaded files
- often exploited to hide files
- ADS not shown in `pslist`:
	``` data
	$ python vol.py –f Win7SP1x64.dmp --profile=Win7SP1x64 pslist
	Volatility Foundation Volatility Framework 2.4
	Name PID PPID Thds Hnds Sess Start
	------------ ------ ------ ------ -------- ------ ------
	[snip]
	1654157019 3596 696 1 5 0 2014-02-18 18:27:29 UTC+0000
	[snip]
	```
	- viewing with `dlllist` shows ADS:
		``` data
		$ python vol.py –f Win7SP1x64.dmp --profile=Win7SP1x64 dlllist -p 3596
		Volatility Foundation Volatility Framework 2.4
		************************************************************************
		1654157019 pid: 3596
		Command line : 1654157019:613509021.exe
		Service Pack 3
		Base Size LoadCount Path
		---------- ---------- ---------- ----
		0x00400000 0x330 0xffff C:\WINDOWS\1654157019:613509021.exe
		0x7c900000 0xaf000 0xffff C:\WINDOWS\system32\ntdll.dll
		0x7c800000 0xf6000 0xffff C:\WINDOWS\system32\kernel32.dll
		```
	
## 5. timestomping
- refers to manipulating timestamps of MFT entries to cover tracks
- detection methods:
	- find MFT entry of timestomping program
	- find prefetch file of timestomping program
	- find shimcache entry of timestomping program
	- cross-reference timestamps from different sources; e.g. shimcache, event log, recent document registry key

## 6. MFT scanning problems
- on a system with multiple NTFS volumes, scanning memory for individual MFT records may cause conflicts
- to avoid, use `dumpfiles` plugin to dump `$Mft` file and process offline; may miss MFT entries no longer referenced in `$Mft` file

## 7. memory-resident file analysis objectives
- extract cached files
- leverage cached file data
- access unencrypted files: files on encrypted media can be cached as well

## 8. Windows cache manager
- subsystem that provides data caching support for FS drivers
- responsible for making sure frequently accessed data is found in phyiscal memory
- accesses data by mapping views of files (within VAS) using memory manager's support for memory-mapped files, aka. section objects
- caches data within *Virtual Address Control Blocks(VACB)*, which correspond to a 256KB view of data mapped in system cache address space

## 9. extracting executable and data files
1. find `_FILE_OBJECT`:
	- pool scanning
	- walking process-handle tables
	- file pointers embedded in process VAD nodes
2. use `SectionObjectPointer` member to find associated `_SECTION_OBJECT_POINTERS`, which is used to store file mapping and cache information for a particular file stream:
	- `ImageSectionObject` and `DataSectionObject` pointers are opaque pointers to control areas (`_CONTROL_AREA`)
3. with `_CONTROL_AREA`, `_SUBSECTION` structures used by memory manager to track regions of memory-mapped file streams can be found:
	- subsequent subsections can be found by traversing singly linked list in `NextSubsection`
	- `SubsectionBase` points to array of page table entry (`_MMPTE`) structures; can be traversed to determine which pages are memory-resident and where
- diagram:
	![[cache_manager_DS.png]]

## 10. shared cache files
- `SharedCacheMap` member of `_SECTION_OBJECT_POINTERS` is an opaque pointer to `_SHARED_CACHE_MAP` structure:
	![[SharedCacheMap_diagram.png]]
	- cache manager uses VACM index arrays to store pointers to VACBs
	- for performance optimization, `_SHARED_CACHE_MAP` contains VACB index array named `InitialVacbs` that consists of four pointers used for files 1MB or less in size
	- if file is larger than 1MB, `Vacbs` member of `_SHARED_CACHE_MAP` is used to store pointer to dynamically allocated VACB index array
	- if file is larger than 32MB, sparse multilevel index array is created where each index array can hold up to 128 entries
	- `_VACB` contains VA of where data is stored in system cache (`BaseAddress`) and offset where data is found within file (`FileOffset`)
- shared cache map is used to track state of cached regions, including 256KB VACBs

## 11. `dumpfiles` plugin
- automates finding and reconstructing memory-resident files
- collects `_FILE_OBJECTS` from process handle tables and VAD trees
- after collection, extracts all memory-mapped and cached regions to designated output directory
- `-S` options saves summary file containing metadata:
	- mapping of original filename and path when extracted to disk
	- regions of file paged out; zero-padded to maintain spatial alignment
- example output:
	``` data
	$ python vol.py -f Win7SP1x64.mem --profile=Win7SP1x64 dumpfiles
	-S summary.json -D output/
	Volatility Foundation Volatility Framework 2.4
	DataSectionObject 0xfffffa800d35c9e0 4 \Device\clfsKtmLog
	SharedCacheMap 0xfffffa800d35c9e0 4 \Device\clfsKtmLog
	DataSectionObject 0xfffffa800d40b7c0 4 \Device\HarddiskVolume1\Windows\System32\LogFiles\WMI\RtBackup\EtwRTDiagLog.etl
	SharedCacheMap 0xfffffa800d40b7c0 4 \Device\HarddiskVolume1\Windows\System32\LogFiles\WMI\RtBackup\EtwRTDiagLog.etl
	DataSectionObject 0xfffffa800d423320 4 \Device\HarddiskVolume1\Windows\System32\LogFiles\WMI\RtBackup\EtwRTEventLog-Application.etl
	[snip]
	```
- output file format:
	``` data
	file.PID.[SCMOffset|CAOffset].[img|dat|vacb]
	```
	- `PID`: process ID where `_FILE_OBJECT` was found
	- `SCMOffset`: VA of `SharedCacheMap` object from which file was extracted
	- `CAOffset`: VA of `_CONTROL_AREA` object from which file was extracted
	- `img`: extracted from `ImageSectionObject`
	- `dat`: extracted from `DataSectionObject`
	- `vacb`: extracted from `SharedCacheMap`
- summary file can be used to map extracted file to original path:
	``` python
	>>> import json as json
	>>> file = open("summary.json", "r")
	>>> for item in file.readlines():
	... info = json.loads(item.strip())
	... print "{0} -> {1}".format(info["ofpath"], info["name"])
	...
	output/file.4.0xfffffa800d3566e0.vacb -> \Device\clfsKtmLog
	output/file.4.0xfffffa800d479280.dat -> \Device\HarddiskVolume1\Windows\System32\LogFiles\WMI\RtBackup\EtwRTDiagLog.etl
	output/file.4.0xfffffa800d46fa10.vacb -> \Device\HarddiskVolume1\Windows\System32\LogFiles\WMI\RtBackup\EtwRTDiagLog.etl
	```
- `-r/--regex` option can be used to filter output:
	``` data
	$ python vol.py -f Win7SP1x64.raw --profile=Win7SP1x64 dumpfiles -D output/ -i -r .evtx$
	Volatility Foundation Volatility Framework 2.4
	DataSectionObject 0xfffffa800e598e20 756 \Device\HarddiskVolume1\Windows\System32\winevt\Logs\System.evtx
	SharedCacheMap 0xfffffa800e598e20 756 \Device\HarddiskVolume1\Windows\System32\winevt\Logs\System.evtx
	[snip]
	```
- malware making changes to memory-resident PE get private version of the page mapped to their address space (copy-on-write); comparing different view can show anomalies:
	- `apihooks` plugin output:
		``` data
		$ python vol.py -f silentbanker.vmem apihooks --profile=WinXPSP3x86
		[snip]
		Hook mode: Usermode
		Hook type: Inline/Trampoline
		Process: 1884 (IEXPLORE.EXE)
		Victim module: WININET.dll (0x771b0000 - 0x77256000)
		Function: WININET.dll!CommitUrlCacheEntryA at 0x771b5319
		Hook address: 0x1080000
		Hooking module: <unknown>
		Disassembly(0):
		0x771b5319 e9e2acec89 JMP 0x1080000
		[snip]
		```
		- data at `0x771b5319` has been modified
	- `dlldump` plugin output:
		``` data
		$ python vol.py dlldump -f silentbanker.vmem --profile=WinXPSP3x86 -p 1884 -i -r wininet.dll -D extracted
		Volatility Foundation Volatility Framework 2.4
		Process(V) Name Module Base Module Name Result
		---------- -------------------- ----------- -------------------- ------
		0x80f1b020 IEXPLORE.EXE 0x0771b0000 WININET.dll OK: module.1884.107e020.771b0000.dll
		$ xxd module.1884.107e020.771b0000.dll | less
		[snip]
		00004710 3BF6 FFFF 9090 9090 90E9 E2AC EC89 83EC ;...............
		00004720 4853 568B 3548 131B 7757 6AFF FF75 08FF HSV.5H..wWj..u..
		```
		- at offset `0x4719` modified 5 bytes appear
	- `dumpfiles` plugin output:
		``` data
		$ python vol.py dumpfiles -f silentbanker.vmem --profile=WinXPSP3x86 -p 1884 -r wininet.dll -D extracted
		Volatility Foundation Volatility Framework 2.4
		ImageSectionObject 0xff3b9130 1884 \Device\HarddiskVolume1\WINDOWS\system32\wininet.dll
		$ xxd file.1884.0x80f04f30.img | less
		[snip]
		00004710 3BF6 FFFF 9090 9090 908B FF55 8BEC 83EC ;..........U....
		00004720 4853 568B 3548 131B 7757 6AFF FF75 08FF HSV.5H..wWj..u.
		```
		- offset `0x4719` contains original bytes

## 12. disk encryption
- suspects often protect their data with *Full Disk Encryption(FDE)*
- while system is powered off, whole disk is encrypted
- when system is runnning and disk is mounted, applications can access data because it is decrypted on the fly
- RAM may contain cached volume passwords, master encryption keys, and portions of unencrypted files

## 13. cached password extraction
- disabled by default for *TrueCrypt*, but enabled by many users
- cache password option when mounting volume with *TrueCrypt*:
	![[TrueCrypt_mount.png]]
- cached password can be cleared on demand by *TrueCrypt*
- *TrueCrypt* driver in kernel mode (*truecrypt.sys*) manages caching:
	- when passwords are cached, uses structure in `Common/Password.h` to store passwords:
		``` C
		typedef struct
		{
		// Modifying this structure can
		// introduce incompatibility with previous versions
		unsigned __int32 Length;
		unsigned char Text[MAX_PASSWORD + 1];
		char Pad[3]; // keep 64-bit alignment
		} Password;
		```
		- value of password is stored in `Text` member
- `truecryptpassphrase` plugin scans for instances of the password structure:
	``` data
	$ python vol.py -f Win8SP0x86-Pro.mem --profile=Win8SP0x86 truecryptpassphrase
	Volatility Foundation Volatility Framework 2.4
	Found at 0x9cd8f064 length 31: duplicative30205_nitrobacterium
	```

## 14. finding encrypted volume
- `truecryptsummary` plugin output for encrypted container:
	``` data
	$ python vol.py -f Win8SP0x86-Pro.mem --profile=Win8SP0x86 truecryptsummary
	Volatility Foundation Volatility Framework 2.4
	Registry Version TrueCrypt Version 7.1a
	Process TrueCrypt.exe at 0x85d79880 pid 3796
	Kernel Module truecrypt.sys at 0x9cd5b000 - 0x9cd92000
	Symbolic Link Volume{ad5c0504-eb77-11e2-af9f-8c2daa411e3c} -> \Device\TrueCryptVolumeJ mounted 2013-10-10 22:51:29 UTC+0000
	File Object \Device\TrueCryptVolumeJ\ at 0x6c1a038
	File Object \Device\TrueCryptVolumeJ\Chats\GOOGLE\Query\modernimpact88@gmail.com.xml at 0x25e8e7e8
	File Object \Device\TrueCryptVolumeJ\Pictures\haile.jpg at 0x3d9d0810
	File Object \Device\TrueCryptVolumeJ\Pictures\nishikori.jpg at 0x3e44cc38
	File Object \Device\TrueCryptVolumeJ\$RECYCLE.BIN\desktop.ini at 0x3e45f790
	File Object \Device\TrueCryptVolumeJ\ at 0x3f14b8d0
	File Object \Device\TrueCryptVolumeJ\Chats\GOOGLE\Query\modernimpact88@gmail.com.log at 0x3f3332f0
	Driver \Driver\truecrypt at 0x18c57ea0 range 0x9cd5b000 - 0x9cd91b80
	Device TrueCryptVolumeJ at 0x86bb1728 type FILE_DEVICE_DISK
	Container Path: \??\C:\Users\Mike\Documents\lease.pdf
	Device TrueCrypt at 0x85db6918 type FILE_DEVICE_UNKNOWN
	```
	- querying cached registry hives tells *TrueCrypt* version installed on system
	- based on symbolic link objects, volume was mounted on `J:` at `2013-10-10 22:51:29`
	- various pictures and *Gmail* chat logs exist on *TrueCrypt* volume
	- full path to encrypted file container at `C:\Users\Mike\Documents\lease.pdf`; can extract from disk image and unlock with recovered password
- `truecryptsummary` plugin output for encrypted partition:
	``` data
	$ python vol.py -f WIN-QBTA4959AO9.raw --profile=Win2012SP0x64 truecryptsummary
	Volatility Foundation Volatility Framework 2.4
	Process TrueCrypt.exe at 0xfffffa801af43980 pid 2096
	Kernel Module truecrypt.sys at 0xfffff88009200000 - 0xfffff88009241000
	Symbolic Link Volume{52b24c47-eb79-11e2-93eb-000c29e29398} -> \Device\TrueCryptVolumeZ mounted 2013-10-11 03:51:08 UTC+0000
	Symbolic Link Volume{52b24c50-eb79-11e2-93eb-000c29e29398} -> \Device\TrueCryptVolumeR mounted 2013-10-11 03:55:13 UTC+0000
	File Object \Device\TrueCryptVolumeR\$Directory at 0x7c2f7070
	File Object \Device\TrueCryptVolumeR\$LogFile at 0x7c39d750
	File Object \Device\TrueCryptVolumeR\$MftMirr at 0x7c67cd40
	File Object \Device\TrueCryptVolumeR\$Mft at 0x7cf05230
	File Object \Device\TrueCryptVolumeR\$Directory at 0x7cf50330
	File Object \Device\TrueCryptVolumeR\$BitMap at 0x7cfa7a00
	Driver \Driver\truecrypt at 0x7c9c0530 range 0xfffff88009200000 – 0xfffff88009241000
	Device TrueCryptVolumeR at 0xfffffa801b4be080 type FILE_DEVICE_DISK
	Container Path: \Device\Harddisk1\Partition1
	Device TrueCrypt at 0xfffffa801ae3f500 type FILE_DEVICE_UNKNOWN
	```
	- OS caches files accessed on volume; can recover part or all of unencrypted files in memory

## 15. AES master key extraction
- inherent risk of disk encryption is that master keys must remain in RAM while volume is mounted for transparent encryption
- *TrueCrypt*'s default encryption scheme is AES in XTS mode:
	- primary and secondary 256-bit keys are combined to form 512-bit master key
- AES key schedules can be distinguished from random data; can be located in memory dumps, packet captures, etc:
	- *AESKeyFinder*
	- *Bulk Extractor*
- extracting keys from memory:
	``` data
	$ ./aeskeyfind Win8SP0x86.raw
	f12bffe602366806d453b3b290f89429
	e6f5e6511496b3db550cc4a00a4bdb1b
	4d81111573a789169fce790f4f13a7bd
	a2cde593dd1023d89851049b8474b9a0
	[snip]
	269493cfc103ee4ac7cb4dea937abb9b
	4d81111573a789169fce790f4f13a7bd
	0f2eb916e673c76b359a932ef2b81a4b
	7a9df9a5589f1d85fb2dfc62471764ef47d00f35890f1884d87c3a10d9eb5bf4
	e786793c9da3574f63965803a909b8ef40b140b43be062850d5bb95d75273e41
	Keyfind progress: 100%
	```
	- the final 2 keys are 256-bits; can be combined for master key
- *Interrogate* can be used to extract non-default keys (Twofish, Serpent) based on patterns in key schedules

## 16. `truecryptmaster` plugin
- uses structured approach to finding keys in the same way *TrueCrypt* driver finds keys
- can find key regardless of encryption algorithm, mode, key length, etc
- output example:
	``` data
	$ python vol.py -f WIN-QBTA4.raw --profile=Win2012SP0x64 truecryptmaster --dump-dir=OUTPUT
	Volatility Foundation Volatility Framework 2.4
	Container: \Device\Harddisk1\Partition1
	Hidden Volume: No
	Read Only: No
	Disk Length: 7743733760 (bytes)
	Host Length: 7743995904 (bytes)
	Encryption Algorithm: SERPENT
	Mode: XTS
	Master Key
	0xfffffa8018eb71a8 bbe1dc7a8e87e9f1f7eef37e6bb30a25 ...z.......~k..%
	0xfffffa8018eb71b8 90b8948fefee425e5105054e3258b1a7 ......B^Q..N2X..
	0xfffffa8018eb71c8 a76c5e96d67892335008a8c60d09fb69 .l^..x.3P......i
	0xfffffa8018eb71d8 efb0b5fc759d44ec8c057fbc94ec3cc9 ....u.D.......<.
	Dumped 64 bytes to ./0xfffffa8018eb71a8_master.key
	```
