## 1. acquisition overview
![[memory acquisition decision tree.png]]
- powered down or hibernating systems may have written recent volatile data to persistent storage:
	- hiberation files
	- page files
	- crash dumps
- software based acquisition considerations:
	- remote vs local
	- cost
	- format
	- CLI vs GUI
	- acquisition vs runtime interrogation

## 2. risk of acquisition
- atomicity:
	- *atomic operations* completes without interruption
	- memory acquisition is not atomic
	- at best memory dump will help infer state of system between start and end of acquisition
- device memory:
	- on x86/x64 computers, BIOS provides physical memory map to OS with different regions marked as reserved for use by firmware, PIC buses, etc, called *device-memory regions*
	- reading from device memory can alter state of device, causing system to freeze
	- most devices are not designed to accommodate simultaneous access from more than one processor
	- may contain valuable evidence:
		- real mode *Interrupt Vector Table(IVT)* with artifacts left by firmware-based rootkits
		- BIOS rootkits that inject code at top of real mode memory
	- acquiring range of device memory relies on data in `HARDWARE` registry hive; malicious code can change to hide self
- cache coherency:
	- processors are not designed to accommodate simultaneous mappings of same physical address with multiple cache attributes:
		- *Translation Lookaside Buffer(TLB)* corruption may occur
		- data corruption at specified address may occur

## 3. how to acquire memory
- local acquisition to removable media:
	- ensure drive is formatted with NTFS or other high performance FS
	- never attach same drive to more than one infected computer
	- do not plug infected drive directly to workstation
	- forensically sterilize drive after use; firmware infection means sterilization is not enough
- remote acquisition:
	- create a temporary admin account that only permits access to target system
	- some tools support acquisition of evidence over network using SSL/TLS
	- consider compressing evidence to save bandwidth
	- compute integrity hashes before and after transfer
- runtime interrogation:
	- quickly sweep entire enterprise and check for specific indicators in physical memory

## 4. software acquisition
- map desired physical addresses into virtual address space of task running on system
- methods:
	- use OS API to create page table entry via:
		- `ZwMapViewOfSection`
		- `MmMapIoSpace`
		- `MmMapLockedPagesSpecifyCache`
		- `MmMapMemoryDumpMdl`
	- use OS API to allocate empty page table entry and manually encode physical pages
- certain APIs map physical addres into user mode address space while some don't

## 5. raw memory dump
- most widely supported format
- includes padding for skipped/unreadable memory ranges

## 6. Windows Crash Dump
- designed for debugging purposes
- begin with `_DMP_HEADER` or `_DMP_HEADER64` struct
- header identfies:
	- major/minor OS version
	- kernel *DirectoryTableBase(DTB)*
	- addresses of active process and loaded kernel module list head
	- info on physical memory runs
- generation methods:
	- Blue Screen
	- CrashOnScrollControl
	- Debuggers
- problems:
	- doesn't include device memory region or first physical page
	- can be subverted by malware
- some memory acquisition tools create crash dump files but do not use same kernel facility that other techniques use, avoiding problems
- `_DMP_HEADER` example:
	``` python
	>>> dt("_DMP_HEADER")
	'_DMP_HEADER' (4096 bytes)
	0x0 : Signature 					['array', 4, ['unsigned char']]
	0x4 : ValidDump 					['array', 4, ['unsigned char']]
	0x8 : MajorVersion 					['unsigned long']
	0xc : MinorVersion 					['unsigned long']
	0x10 : DirectoryTableBase 			['unsigned long']
	0x14 : PfnDataBase 					['unsigned long']
	0x18 : PsLoadedModuleList 			['unsigned long']
	0x1c : PsActiveProcessHead 			['unsigned long']
	0x30 : MachineImageType 			['unsigned long']
	0x34 : NumberProcessors 			['unsigned long']
	0x38 : BugCheckCode 				['unsigned long']
	0x40 : BugCheckCodeParameter 		['array', 4, ['unsigned long long']]
	0x80 : KdDebuggerDataBlock 			['unsigned long long']
	0x88 : PhysicalMemoryBlockBuffer 	['_PHYSICAL_MEMORY_DESCRIPTOR']
	[snip]
	```
	- Signature member contains PAGEDUMP or PAGEDU64

## 7. Windows Hibernation File
- hibernation files(`hiberfil.sys`) contain a compressed copy of memory that the system dumps to disk during hibernation
- consists of:
	- standard header(`PO_MEMORY_IMAGE`)
	- set of kernel contexts and registers
	- arrays of compressed data blocks
- hibernation file header example:
	``` python
	>>> dt("PO_MEMORY_IMAGE")
	'PO_MEMORY_IMAGE' (168 bytes)
	0x0 : Signature 		['String', {'length': 4}]
	0x4 : Version 			['unsigned long']
	0x8 : CheckSum 			['unsigned long']
	0xc : LengthSelf 		['unsigned long']
	0x10 : PageSelf 		['unsigned long']
	0x14 : PageSize 		['unsigned long']
	0x18 : ImageType 		['unsigned long']
	0x20 : SystemTime 		['WinTimeStamp', {}]
	[snip]
	```
	- Signature member usually contains hibr, HIBR, wake, or WAKE
- `PO_MEMORY_IMAGE` header may be zeroed out when system resumes
- Volatility decompresses certain segments each time; usually better to decompress entire memory dump once and for all
- to create:
	1. enable hibernation in kernel (`powercfg.exe /hibernate on`)
	2. issue `shutdown /h` command
- any active network connections are terminated before hibernation
- malware may remove itself before hibernation

## 8. *Expert Witness Format(EWF)*
- memory acquired by EnCase is stored in EWF
- methods of analysis:
	- EWFAddressSpace:
		- Volatility includes address space for working with EWF files; requires `libewf` installation
	- mounting with EnCase:
		- works in networked environment that allows sampling
	- mounting with FTK imager

## 9. HPAK format
- allows target system's physical memory and page files to embed in same output
- only FastDump can create this format with `-hpak` option
- structure:
	- 32-byte header:
		- starts with 4 magic bytes, `HPAK`
		``` python
		>>> dt("HPAK_HEADER")
		'HPAK_HEADER' (32 bytes)
		0x0 : Magic 			['String', {'length': 4}]
		```
	- `HPAK_SECTION`:
		``` python
		>>> dt("HPAK_SECTION")
		'HPAK_SECTION' (224 bytes)
		0x0 : Header 			['String', {'length': 32}]
		0x8c : Compressed 		['unsigned int']
		0x98 : Length 			['unsigned long long']
		0xa8 : Offset 			['unsigned long long']
		0xb0 : NextSection 		['unsigned long long']
		0xd4 : Name 			['String', {'length': 12}]
		```
		- `header` ontains string for data type; e.g. `_PHYSDUMP`, `_PAGEDUMP`
		- `offset` and `length` show location of corresponding data in HPAK file
		- if `compressed` is non-zero, data is compressed with `zlib`
- `hpakinfo` can be used to explore contents:
	``` sh
	$ python vol.py -f memdump.hpak hpakinfo
	Header: 		HPAKSECTHPAK_SECTION_PHYSDUMP
	Length: 		0x20000000
	Offset: 		0x4f8
	NextOffset: 	0x200004f8
	Name: 			memdump.bin
	Compressed: 	0

	Header: 		HPAKSECTHPAK_SECTION_PAGEDUMP
	Length: 		0x30000000
	Offset: 		0x200009d0
	NextOffset: 	0x500009d0
	Name: 			dumpfile.sys
	Compressed: 	0
	```

## 10. virtual machine memory
- acquisition can be done in guest OS or by hypervisor
- acquisition by hypervisor is less invasive; harder for malware to detect

## 11. VMware
- acquisition:
	- desktop product, suspend/pause or create snapshot:
		- VM's memory is written to directory on host's FS
	- VMware server or ESX, use vSphere GUI console or `vmrun` command:
		- likely written to *Storage Area Network(SAN)* or *Network File System(NFS)* data store
- important to recover all files with `.vmem`, `.vmsn`, `.vmss` extensions
- `_VMWARE_HEADER` example:
	``` python
	>>> dt("_VMWARE_HEADER")
	'_VMWARE_HEADER' (12 bytes)
	0x0 : Magic 			['unsigned int']
	0x8 : GroupCount 		['unsigned int']
	0xc : Groups 			['array', lambda x : x.GroupCount, ['_VMWARE_GROUP']]
	```
	- `Magic` must be one of: 0xbed2bed0, 0xbad1bad1, 0xbed2bed2, 0xbed3bed3
	- `Groups` specifies array of `_VMWARE_GROUP` structures
- `_VMWARE_GROUP` example:
	``` python
	>>> dt("_VMWARE_GROUP")
	'_VMWARE_GROUP' (80 bytes)
	0x0 : Name 				['String', {'length': 64, 'encoding': 'utf8'}]
	0x40 : TagsOffset 		['unsigned long long']
	```
	- `Name` allows metadata components to be categorized
	- `TagsOffset` specifies list of `_VMWARE_TAG` structures
- `_VMWARE_TAG` example:
	``` python
	>>> dt("_VMWARE_TAG")
	'_VMWARE_TAG' (None bytes)
	0x0 : Flags 			['unsigned char']
	0x1 : NameLength 		['unsigned char']
	0x2 : Name 				['String',{'length': lambda x : x.NameLength, 'encoding': 'utf8'}]
	```

## 12. VirtualBox
- methods:
	- `vboxmanage debugvm` commands:
		- creates ELF64 core dump binary with custom sections that represent guest memory
	- `--dbg` switch when starting VM and `.pgmphystofile` command:
		- outputs raw memory dump
	- VirtualBox Python API to create memory dumping utility
- ELF64 note segment contains `DBGFCOREDESCRIPTOR` structure:
	``` python
	>>> dt("DBGFCOREDESCRIPTOR")
	'DBGFCOREDESCRIPTOR' (24 bytes)
	0x0 : u32Magic 			['unsigned int']
	0x4 : u32FmtVersion 	['unsigned int']
	0x8 : cbSelf 			['unsigned int']
	0xc : u32VBoxVersion 	['unsigned int']
	0x10 : u32VBoxRevision 	['unsigned int']
	0x14 : cCpus 			['unsigned int']
	```
	- contains VitualBox magic signature, 0xc01ac0de
	- version information
	- number of CPUs
- ELF64 `PT_LOAD` segments contain:
	- `p_paddr`: starting physical memory address
	- `p_offset`: offset of physical memory chunk in ELF64 file
	- `p_memsz`: size of memory chunk

## 13. QEMU
- saves VM memory in ELF64 core dump format
- dumps created with `virsh`, a CLI to libvirt

## 14. Xen/KVM
- supported by LibVMI project; real-time analysis of running VMs without running code inside VM is possible

## 15. Microsoft Hyper-V
- acquisition:
	1. save VM state or create snapshot
	2. recover `.bin` and `.vsv` files 
- use `vm2dmp.exe` to convert `.bin` and `.vsv` files into Windows crash dump for analysis with Volatility

## 16. Actaeon
- enables analysis of guest OSs in virtualization environments using Intel VT-x given physical memory dump of a host system
- inclues abilities to locate memory resident hypervisors and nested virtualization
- allows VM instrospection of 32-bit Windows guests under:
	- KVM
	- Xen
	- VMware workstation
	- VirtualBox
	- HyperDbg

## 17. converting memory dumps
- `imagecopy` to convert any format into raw memory dump
	``` sh
	$ python vol.py -f win7x64.dmp --profile=Win7SP0x64 imagecopy -O copy.raw
	Volatility Foundation Volatility Framework 2.4
	Writing data (5.00 MB chunks): |........[snip]........................|
	```
- `raw2dmp` to convert raw memory dump to crash dump
	``` sh
	$ python vol.py -f memory.raw --profile=Win8SP0x64 raw2dmp -O win8.dmp
	Volatility Foundation Volatility Framework 2.4
	Writing data (5.00 MB chunks): |........[snip]........................|
	```

## 18. volatile memory on disk

