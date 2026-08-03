## 1. physical memory maps
- mapping of which devices occupy regions of physical memory

## 2. physical memory map analysis objectives
- detect hardware manipulation
- verify memory captures

## 3. physical memory map DS
- `resource`:
	``` python
	'resource' (56 bytes)
	0x0 : start ['unsigned long long']
	0x8 : end ['unsigned long long']
	0x10 : name ['pointer', ['char']]
	0x18 : flags ['unsigned long']
	0x20 : parent ['pointer', ['resource']]
	0x28 : sibling ['pointer', ['resource']]
	0x30 : child ['pointer', ['resource']]
	```
	- `start`: starting physical address
	- `end`: ending physical address
	- `name`: name of region
	- `sibling`: pointer to next `resource` within same level
	- `child`: pointer to first child

## 4. *linux_iomem* plugin
- displays memory regions in similar manner as `cat /proc/iomem`
- begins at root of device tree (global variable named `iomem_resource`) and enumerates all nodes by recursively processing `children` and `sibling`
- example output:
	``` data
	$ python vol.py --profile=LinuxDebian-3_2x64 -f debian.lime linux_iomem
	Volatility Foundation Volatility Framework 2.4
	reserved 0x0 0xFFFF
	System RAM 0x10000 0x9EFFF
	reserved 0x9F000 0x9FFFF
	PCI Bus 0000:00 0xA0000 0xBFFFF
	Video ROM 0xC0000 0xC7FFF
	reserved 0xCA000 0xCBFFF
	Adapter ROM 0xCA000 0xCAFFF
	PCI Bus 0000:00 0xCC000 0xCFFFF
	PCI Bus 0000:00 0xD0000 0xD3FFF
	PCI Bus 0000:00 0xD4000 0xD7FFF
	PCI Bus 0000:00 0xD8000 0xDBFFF
	reserved 0xDC000 0xFFFFF
	System ROM 0xF0000 0xFFFFF
	System RAM 0x100000 0x3FEDFFFF
	Kernel code 0x1000000 0x1358B25
	Kernel data 0x1358B26 0x1694D7F
	Kernel bss 0x1729000 0x1806FFF
	ACPI Tables 0x3FEE0000 0x3FEFEFFF
	ACPI Non-volatile Storage 0x3FEFF000 0x3FEFFFFF
	System RAM 0x3FF00000 0x3FFFFFFF
	PCI Bus 0000:00 0xC0000000 0xFEBFFFFF
	0000:00:0f.0 0xC0000000 0xC0007FFF
	0000:00:10.0 0xC0008000 0xC000BFFF
	0000:00:07.7 0xC8000000 0xC8001FFF
	0000:00:10.0 0xC8020000 0xC803FFFF
	[snip]
	IOAPIC 0 0xFEC00000 0xFEC003FF
	HPET 0 0xFED00000 0xFED003FF
	pnp 00:08 0xFED00000 0xFED003FF
	Local APIC 0xFEE00000 0xFEE00FFF
	reserved 0xFEE00000 0xFEE00FFF
	[snip]
	```
- computers with multicore chips use APIC architecture to handle hardware interrupts and other actions; APIC DS can be gooked to redirect control flow of interrupts
- video cards and PIC network cards can be targeted by malware as well

## 5.*limeinfo* plugin 
- example output:
	``` data
	$ python vol.py --profile=LinuxDebian-3_2x64 -f debian.lime limeinfo
	Volatility Foundation Volatility Framework 2.4
	Memory Start Memory End Size
	------------------ ------------------ ------------------
	0x0000000000010000 0x000000000009efff 0x000000000008f000
	0x0000000000100000 0x000000003fedffff 0x000000003fde0000
	0x000000003ff00000 0x000000003fffffff 0x0000000000100000
	```
	- can be compared with *linux_iomem* to verify acquisition:
		``` data
		$ python vol.py --profile=LinuxDebian-3_2x64 -f debian.lime linux_iomem | grep "System RAM"
		Volatility Foundation Volatility Framework 2.4
		Resource Name Start Address End Address
		System RAM 0x10000 0x9EFFF
		System RAM 0x100000 0x3FEDFFFF
		System RAM 0x3FF00000 0x3FFFFFFF
		```

## 6. virtual memory maps
- Linux reserves a number of areas in the VA space for storing specific data types
- boundary exists between user and kernel memory:
	![[kernel_memory.png]]
	- kernel address space starts with mapping of kernel
- certain functions are located within kernel code, but overwritten and pointed elsewhere by malware; kernel code segment address can be used to check
- `vmalloc` area is used to store large, virtually contiguous regions; e.g. kernel modules, video card buffers, and swapped pages
- high memory map is limited in size and stores mappings that only exist for a short period of time; located via `PKMAP_BASE` symbol
- mappings reserved by hardware can be found via `FIXADDR_START` symbol

## 7. kernel debug buffer
- stores log messages from drivers and kernel components
- all users can read using *dmesg*

## 8. kernel debug buffer analysis objectives
- recover USB serial numbers: debug buffer contains details about recently inserted removable media devices
- examine network activity: traces of network devices entering promiscuous mode are left
- build timeline of events: entries include timestamp

## 9. kernel debug buffer DS
- `log`:
	``` python
	>>> dt("log")
	'log' (16 bytes)
	0x0 : ts_nsec ['unsigned long long']
	0x8 : len ['unsigned short']
	0xa : text_len ['unsigned short']
	0xc : dict_len ['unsigned short']
	0xe : facility ['unsigned char']
	0xf : flags ['BitField', {'end_bit': 5, 'start_bit': 0}]
	0xf : level ['BitField', {'end_bit': 8, 'start_bit': 5}]
	```
	- `ts_nsec`: timestamp showing number of nanoseconds since machine booted
	- `text_len`: length of text portion
	- `len`: length of text portion plus header info
	- `level`: severity of message

## 10. *linux_dmesg* plugin
- starts by locating addresses of `log_buf` and `log-buf_len` variables
- example output:
	``` data
	$ python vol.py --profile=LinuxDebian-3_2x64 -f debian.lime linux_dmesg
	<6>[ 0.000000] Initializing cgroup subsys cpu
	<5>[ 0.000000] Linux version 3.2.0-4-amd64 (debian-kernel@lists.debian.org) (gcc version 4.6.3 (Debian 4.6.3-14) ) #1 SMP Debian 3.2.51-1
	<6>[ 0.000000] Command line: BOOT_IMAGE=/boot/vmlinuz-3.2.0-4-amd64 root=UUID=b2385703-e550-4736-a19f-e8490e5a570e ro quiet
	[snip]
	<6>[ 0.000000] found SMP MP-table at [ffff8800000f6bf0] f6bf0
	<7>[ 0.000000] initial memory mapped : 0 - 20000000
	<7>[ 0.000000] Base memory trampoline at [ffff88000009a000] 9a000 size 20480
	<6>[ 0.000000] init_memory_mapping: 0000000000000000-0000000040000000
	<7>[ 0.000000] 0000000000 - 0040000000 page 2M
	[snip]
	```
- USB thumb drive output:
	``` data
	<6> [ 143.110316] usb 4-4: new SuperSpeed USB device number 4 using xhci_hcd
	<6> [ 143.126899] usb 4-4: New USB device found, idVendor=125f, idProduct=312b
	<6> [ 143.126908] usb 4-4: New USB device strings: Mfr=1, Product=2, SerialNumber=3
	<6> [ 143.126913] usb 4-4: Product: ADATA USB Flash Drive
	<6> [ 143.126916] usb 4-4: Manufacturer: ADATA
	<6> [ 143.126920] usb 4-4: SerialNumber: 23719051000100F8
	```
- connecting to wireless networks output:
	``` data
	<6> [ 55.947702] wlan0: authenticate with 00:24:9d:c8:a5:42
	<6> [ 55.957212] wlan0: send auth to 00:24:9d:c8:a5:42 (try 1/3)
	<6> [ 55.962997] wlan0: authenticated
	<6> [ 55.963319] iwlwifi 0000:04:00.0 wlan0: disabling HT as WMM/QoS is not supported by the AP
	<6> [ 55.963328] iwlwifi 0000:04:00.0 wlan0: disabling VHT as WMM/QoS is not supported by the AP
	<6> [ 55.964447] wlan0: associate with 00:24:9d:c8:a5:42 (try 1/3)
	<6> [ 55.966767] wlan0: RX AssocResp from 00:24:9d:c8:a5:42 (capab=0x411 status=0 aid=5)
	<6> [ 55.970763] wlan0: associated
	```

## 11. *Loaded Kernel Modules(LKM)*
- enable code to be dynamically inserted into running OS
- ELF files normally stored with *.ko* extensio
- can contain hardware drivers, FS implementations, security extensions and more
- often abused by rootkits

## 12. LKM analysis objectives
- locate kernel modules in memory: Linux keeps linked list of loded kernels
- extract kernel modules to perform analysis

## 13. LKM DS
- `module`:
	``` python
	>>> dt("module")
	'module' (584 bytes)
	0x0 : state ['Enumeration', {'target': 'int', 'choices': {0: 'MODULE_STATE_LIVE', 1: 'MODULE_STATE_COMING', 2: 'MODULE_STATE_GOING'}}]
	0x8 : list ['list_head']
	0x18 : name ['String', {'length': 60}]
	0x50 : mkobj ['module_kobject']
	0xa8 : modinfo_attrs ['pointer', ['module_attribute']]
	[snip]
	0xe0 : kp ['pointer', ['kernel_param']]
	0xe8 : num_kp ['unsigned int']
	[snip]
	0x148 : init ['pointer', ['void']]
	0x150 : module_init ['pointer', ['void']]
	0x158 : module_core ['pointer', ['void']]
	0x160 : init_size ['unsigned int']
	0x164 : core_size ['unsigned int']
	0x168 : init_text_size ['unsigned int']
	0x16c : core_text_size ['unsigned int']
	0x170 : init_ro_size ['unsigned int']
	0x174 : core_ro_size ['unsigned int']
	[snip]
	0x198 : symtab ['pointer', ['elf64_sym']]
	0x1a0 : core_symtab ['pointer', ['elf64_sym']]
	0x1a8 : num_symtab ['unsigned int']
	0x1ac : core_num_syms ['unsigned int']
	0x1b0 : strtab ['pointer', ['char']]
	0x1b8 : core_strtab ['pointer', ['char']]
	0x1c0 : sect_attrs ['pointer', ['module_sect_attrs']]
	0x1c8 : notes_attrs ['pointer', ['module_notes_attrs']]
	0x1d0 : args ['pointer', ['char']]
	[snip]
	```
	- `list`: pointer to linked list of loaded modules
	- `name`: name of module
	- `kp`: pointer to parameters passed to module at load time
	- `num_kp`: number of parameters
	- `module_init`: pointer to initialized code
	- `init_size`: size of initialized code
	- `module_core`: pointer to code used after initialization 
	- `core_size`: size of code used after initialization
	- `sect_attrs`: 
		- array into module's ELF sections
		- used by *Volatility* to reconstruct ELF file
	
## 14. *linux_lsmod* plugin
- enumerates modules by walking global list stored within `modules` variable
- example output:
	``` data
	$ python vol.py --profile=LinuxDebian-3_2x64 -f debian.lime linux_lsmod
	Volatility Foundation Volatility Framework 2.4
	lime 17991
	nfsd 216170
	nfs 308313
	nfs_acl 12511
	auth_rpcgss 37143
	fscache 36739
	lockd 67306
	sunrpc 173730
	loop 22641
	coretemp 12898
	crc32c_intel 12747
	snd_ens1371 23250
	snd_ac97_codec 106942
	snd_rawmidi 23060
	snd_seq_device 13176
	[snip]
	```
- can view parameters passed with `-P/--params` flag

## 15. *linux_moddump* plugin
- capable of extracting memory resident sections of module and creating ELF file
- because the original ELF header is discarded, must re-create ELF header first
- example output:
	``` data
	$ python vol.py --profile=LinuxDebian-3_2x64 -f debian.lime linux_lsmod -S
	Volatility Foundation Volatility Framework 2.4
	lime 17991
	.note.gnu.build-id 0xffffffffa0392000
	.text 0xffffffffa0391000
	.rodata 0xffffffffa0392024
	.rodata.str1.1 0xffffffffa0392034
	__param 0xffffffffa0392058
	.data 0xffffffffa0393000
	.gnu.linkonce.this_module 0xffffffffa0393010
	.bss 0xffffffffa0393260
	.symtab 0xffffffffa0030000
	.strtab 0xffffffffa00303c0
	[snip]
	```
