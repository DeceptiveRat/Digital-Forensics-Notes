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

## 4. 
