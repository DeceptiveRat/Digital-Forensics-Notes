## 1. historical methods
- `/dev/mem`:
	- currently disabled due to security concerns
	- when enabled, exports physical memory
	- allows programs with root privileges to directly read from and write to RAM
	- accessing sensitive regions may cause memory corruption or system instability
	- can only address first 896MB of RAM
- `/dev/kmem`:
	- used to acquire memory on 32-bit systems
	- disabled
	- exports kernel virtual address space
- *ptrace*:
	- userland debugging interface
	- can acquire pages only from running processes; not suitable for robust memory acquisition
	- should be used when interested in code and data of only one process

## 2. modern acquisition
- *fmem*:
	- loads kernel driver that creates character device named `/dev/fmem`
	- device exports physical memory
	- checks if physical pages reside in main memory before accessing; avoids stability issues
	- can access above 896MB boundary
	- requires investigator to inspect `/proc/iomem` to determine where RAM is mapped; necessary for machines that do not map all main memory at physical offset 0:
		``` data
		$ grep "System RAM" /proc/iomem
		00010000-0009f3ff : System RAM
		00100000-bfedffff : System RAM
		bff00000-bfffffff : System RAM
		100000000-100bfffff : System RAM
		```
- *Linux Memory Extractor(LiME)*:
	- operates by loading kernel driver, but acquisition is done within kernel; enhances accuracy be removing context switches
	- automatically determines address ranges that contain main memory; walks `iomem_resource` linked list
	- can choose `lime` acquisition format:
		``` python
		>>> dt("lime_header")
		'lime_header' (32 bytes)
		0x0 : magic ['unsigned int']
		0x4 : version ['unsigned int']
		0x8 : start ['unsigned long long']
		0x10 : end ['unsigned long long']
		0x18 : reversed [‘unsigned long long']
		```
	- examples:
		``` sh
		$ sudo insmod lime.ko "path=/mnt/externaldrive/memdmp.lime format=lime"
		$ sudo insmod lime.ko "path=tcp:4444 format=lime"
		```
- `/proc/kcore`:
	- exports kernel's VAS to userland in form of core dump (ELF)
	- limits acquisition to first 896MB of memory on 32-bit systems
	- requires specialized tool
	- may be disabled on certain distributions
	- malware can hook read function of `/proc/kcore` and filter out data

## 3. Volatility Linux Profiles
[SKIPPED]
