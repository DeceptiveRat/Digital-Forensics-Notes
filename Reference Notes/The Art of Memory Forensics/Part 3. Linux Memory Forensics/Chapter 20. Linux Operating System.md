## 1. ELF files
- *Executable and Linking Format(ELF)* is the main executable file format on Linux
- User applications, shared libraries, and the kernel are stored in ELF format

## 2. ELF header
- located at beginning of file
- represented by `Elf32_Ehdr` or `Elf64_Ehdr`
- important structure memebers:
	- `e_ident`:
		- holds file identification information
		- first 4 bytes are `0x7fELF`, 5th byte stores whether 32- or 64-bit system, 6th byte stores whether little or big endian
		- signature used to scan through memory for ELF files
	- `e_type`: tells file type
	- `e_entry`: holds program entry point, address of first instruction
	- `e_phoff`, `e_phentsize`, `e_phnum`: offset, entry size, and number of program header entries
	- `e_shoff`, `e_shentsize`, `e_shnum`: offset, entry size, and number of section header entries
	- `e_shstrndx`: stores index within section header table of strings that map to section names

## 3. ELF sections
- at offset `e_shoff` is an array of `Elf32_Shdr` or `Elf64_Shdr` structures that represent each section within the file
- critical members:
	- `sh_name`: index into string table of section name
	- `sh_addr`: 
		- VA of where section will be mapped
		- same as `sh_offset` for relocatable code because linker does not know where code will load within address space
	- `sh_offset`: offset within file
	- `sh_size`: section size in bytes
- common ELF sections:

	| Section Name | Description |
	| --- | --- |
	| .text | Contains the application’s executable code |
	| .data | Contains the read/write data (variables) |
	| .rdata | Contains read-only data |
	| .bss | Contains variables that are initialized to zero |
	| .got | Contains the global offset table |

- common section types:

	| Section Type | Description |
	| --- | --- |
	| PROGBITS | Section whose contents from disk will be loaded into memory upon execution |
	| NOBITS | Sections that do not have data in the file, but have regions allocated in memory The .bss is typically a NOBITS section because all its memory is initialized to zero upon execution (and there is no need to store zeroes within the file) |
	| STRTAB | Holds a string table of the application |
	| DYNAMIC | Indicates that this is a dynamically linked application and holds the dynamic information |
	| HASH | Contains the hash table of the application’s symbols |

## 4. *readelf*
- `-h` parameter to *readelf* displays header information:
	``` data
	$ readelf -h /bin/ls
	ELF Header:
		Magic: 7f 45 4c 46 01 01 01 00 00 00 00 00 00 00 00 00
		Class: ELF32
		Data: 2's complement, little endian
		Version: 1 (current)
		OS/ABI: UNIX - System V
		ABI Version: 0
		Type: EXEC (Executable file)
		Machine: Intel 80386
		Version: 0x1
		Entry point address: 0x804c1b4
		Start of program headers: 52 (bytes into file)
		Start of section headers: 111580 (bytes into file)
		Flags: 0x0
		Size of this header: 52 (bytes)
		Size of program headers: 32 (bytes)
		Number of program headers: 9
		Size of section headers: 40 (bytes)
		Number of section headers: 28
		Section header string table index: 27
	```
- `-S` parameter displays section headers for an executable:
``` data
$ readelf -S /bin/ls
There are 28 section headers, starting at offset 0x1b3dc:
Section Headers:
	[Nr] Name Type Addr Off Size ES Flg Lk Inf Al
	[ 0] NULL 00000000 000000 000000 00 0 0 0
	[ 1] .interp PROGBITS 08048154 000154 000013 00 A 0 0 1
	[ 2] .note.ABI-tag NOTE 08048168 000168 000020 00 A 0 0 4
	[ 3] .note.gnu.build-i NOTE 08048188 000188 000024 00 A 0 0 4
	[ 4] .hash HASH 080481ac 0001ac 000374 04 A 6 0 4
	<snip>
	[12] .init PROGBITS 08049820 001820 000026 00 AX 0 0 4
	[13] .plt PROGBITS 08049850 001850 0006b0 04 AX 0 0 16
	[14] .text PROGBITS 08049f00 001f00 0118fc 00 AX 0 0 16
	[15] .fini PROGBITS 0805b7fc 0137fc 000017 00 AX 0 0 4
	[16] .rodata PROGBITS 0805b820 013820 0041a4 00 A 0 0 32
	<snip>
	[23] .got PROGBITS 08063fec 01afec 000008 04 WA 0 0 4
	[24] .got.plt PROGBITS 08063ff4 01aff4 0001b4 04 WA 0 0 4
	[25] .data PROGBITS 080641c0 01b1c0 00012c 00 WA 0 0 32
	[26] .bss NOBITS 08064300 01b2ec 000c2c 00 WA 0 0 32
	[27] .shstrtab STRTAB 00000000 01b2ec 0000ed 00 0 0 1
Key to Flags:
	W (write), A (alloc), X (execute), M (merge), S (strings)
	I (info), L (link order), G (group), T (TLS), E (exclude), x (unknown)
	O (extra OS processing required) o (OS specific), p (processor specific)
```

## 5. ELF program headers
- `e_phoff` is an offset to `Elf32_Phdr` or `Elf64_Phdr`
- program headers are used to map file and sections to memory at runtime
- often the only information available to statically analyze binary
- important members:
	- `p_type`: 
		- describes type of segment, portions of file that load into memory
		- common types are:
			- `PT_LOAD`: segment that must load into memory
			- `PT_DYNAMIC`: describes linking information
			- `PT_INTERP`: holds full path to program interpreter
	- `p_vaddr` and `p_offset`: same purpose as `sh_addr` and `sh_offset` within section headers
	- `p_filesz`: size of segment on disk
	- `p_memsz`: size of segment in memory
- listed using *readelf*:
	``` data
	$ readelf -l /bin/ls
	<snip>
	Program Headers:
		Type Offset VirtAddr PhysAddr FileSiz MemSiz Flg Align
		PHDR 0x000034 0x08048034 0x08048034 0x00120 0x00120 R E 0x4
		INTERP 0x000154 0x08048154 0x08048154 0x00013 0x00013 R 0x1
			[Requesting program interpreter: /lib/ld-linux.so.2]
		LOAD 0x000000 0x08048000 0x08048000 0x1a8cc 0x1a8cc R E 0x1000
		LOAD 0x01aed8 0x08063ed8 0x08063ed8 0x00414 0x01054 RW 0x1000
	Section to Segment mapping:
		Segment Sections...
		00
		01 .interp
		02 .interp .note.ABI-tag .note.gnu.build-id .hash .gnu.hash .dynsym .dynstr .gnu.version .gnu.version_r .rel.dyn .rel.plt .init .plt .text .fini .rodata .eh_frame_hdr .eh_frame
		03 .init_array .fini_array .jcr .dynamic .got .got.plt .data .bss
		04 .dynamic
		05 .note.ABI-tag .note.gnu.build-id
		06 .eh_frame_hdr
		07
		08 .init_array .fini_array .jcr .dynamic .got
	```

## 6. packed ELF binaries
- section header tables are often removed by a packer:
	``` sh
	$ cp /bin/ls ls_upx
	$ upx ls_upx
	[REMOVED]
	File size Ratio Format Name
	-------------------- ------ ----------- -----------
	112700 -> 50820 45.09% linux/elf386 ls
	Packed 1 file.
	$ ./ls_upx /etc
	acpi defom hosts localtime passwd- securetty
	<snip>
	$ readelf -h ls_upx
	ELF Header:
		Magic: 7f 45 4c 46 01 01 01 03 00 00 00 00 00 00 00 00
		<snip>
		Size of section headers: 40 (bytes)
		Number of section headers: 0
		Section header string table index: 0
	$ readelf -S ls_upx
	There are no sections in this file.
	```
- no longer have `PHDR` or `INTERP` headers:
	``` data
	$ readelf -l ls_upx
	Elf file type is EXEC (Executable file)
	Entry point 0xc0cbf0
	There are 2 program headers, starting at offset 52
	Program Headers:
		Type Offset VirtAddr PhysAddr FileSiz MemSiz Flg Align
		LOAD 0x000000 0x00c01000 0x00c01000 0x0c3e0 0x0c3e0 R E 0x1000
		LOAD 0x000f2c 0x08064f2c 0x08064f2c 0x00000 0x00000 RW 0x1000
	```
	- executable is now a statically linked binary; dynamic loaders typically require `INTERP` header

## 7. shared library
- reusable pieces of code 
- can be dynamically loaded into application
- stored with *.so* extension
- counter part to DLL files on Windows
- specified within dynamic information section:
	``` data
	$ readelf -d /bin/bash | grep NEEDED
	0x00000001 (NEEDED) Shared library: [libtinfo.so.5]
	0x00000001 (NEEDED) Shared library: [libdl.so.2]
	0x00000001 (NEEDED) Shared library: [libc.so.6]
	```

## 8. *Global Offset Table(GOT)*
- stores runtime address of symbols that cannot be computed at link time
- often stored within shared libraries that can be loaded anywhere within the process' address space
- analysis often lends insight into how a system was compromised as well as how malicious code altered the runtime state
- exploit code can use GOT to find addresses of API functions such as `system`, `strcpy`, `memcpy`, `setuid`, and `setgid`; difficult to locate GOT on *Position Independent Executable(PIE)* applications
- exploit code can overwrite address in GOT as well

## 9. GOT and *Procedure Linkage Table(PLT)* internals
1. view source code of *printtest.c*:
	``` C
	#include <stdio.h>

	extern int test;

	int main(void)
	{
		printf("test is: %d\n", test);
		printf("libfunc result is: %d\n", libfunc());
		return 0;
	}
	```
2. global variable `test` and `libfunc` are defined in *library.c*:
	``` C
	int test = 42;
	int libfunc(void)
	{
		return 12345;
	}
	```
3. compile shared library
	``` sh
	$ gcc -o library.so library.c -shared
	```
4. compile *printtest* and link
	``` sh
	$ gcc -o printtest printtest.c library.so
	```
5. runtime loader must be able to locate `test` and `libfunc`; relocation entries are created by the linker
	``` data
	$ readelf -r ./printtest
	Relocation section '.rel.dyn' at offset 0x418 contains 2 entries:
	Offset Info Type Sym.Value Sym. Name
	08049824 00000406 R_386_GLOB_DAT 00000000 __gmon_start__
	0804984c 00000d05 R_386_COPY 0804984c test
	Relocation section '.rel.plt' at offset 0x428 contains 4 entries:
	Offset Info Type Sym.Value Sym. Name
	08049834 00000207 R_386_JUMP_SLOT 00000000 printf
	08049838 00000307 R_386_JUMP_SLOT 00000000 libfunc
	0804983c 00000407 R_386_JUMP_SLOT 00000000 __gmon_start__
	08049840 00000507 R_386_JUMP_SLOT 00000000 __libc_start_main
	```
	- `test` is within the `rel.dyn` relocation section
	- `libfunc` is within the `rel.plt` section
	- `test` has type `R_386_COPY` and address of `0x804984c`; tells loader to copy value of `test` within *library.so* into `0x804984c` of *printtest*
	- imported functions are given relocation entries of type `R_386_JUMP_SLOT`; tells loader function must be located within one of the specified shared libraries
	- computed addresses of external symbols are stored within the GOT; similar to IAT for PE files

## 10. PLT
- supports calling functions within shared libraries
- disassembly of *printtest* shows `libfunc` within *library.so* is not called directly but redirected through *printtest* PLT entry:
	``` data
	$ gdb ./printtest
	(gdb) disassemble main
	Dump of assembler code for function main:
		<snip>
		0x080485ca <+30>: call 0x8048490 <libfunc@plt>
		0x080485cf <+35>: mov DWORD PTR [esp+0x4],eax
		0x080485d3 <+39>: mov DWORD PTR [esp],0x804868d
		<snip>
	```
- the destination contains information necessary to initialize the GOT entry:
	``` data
	(gdb) disassemble 0x8048490
	Dump of assembler code for function libfunc@plt:
		0x08048490 <+0>: jmp DWORD PTR ds:0x8049838
		0x08048496 <+6>: push 0x8
		0x0804849b <+11>: jmp 0x8048470
	End of assembler dump.
	(gdb) x/x 0x8049838
	0x8049838 <libfunc@got.plt>: 0x08048496
	```
	- function redirects to instruction `push 0x8` at `0x8048496`
	``` data
	(gdb) x/2i 0x8048470
	0x8048470: push DWORD PTR ds:0x804982c
	0x8048476: jmp DWORD PTR ds:0x8049830
	(gdb) x/x 0x8049830
	0x8049830 <_GLOBAL_OFFSET_TABLE_+8>: 0xb7ff59b0
	(gdb) x/8i 0xb7ff59b0
	0xb7ff59b0: push eax
	0xb7ff59b1: push ecx
	0xb7ff59b2: push edx
	0xb7ff59b3: mov edx,DWORD PTR [esp+0x10]
	0xb7ff59b7: mov eax,DWORD PTR [esp+0xc]
	0xb7ff59bb: call 0xb7fefb40
	0xb7ff59c0: pop edx
	0xb7ff59c1: mov ecx,DWORD PTR [esp]
	(gdb) ^Z
	[1]+ Stopped gdb ./printtest
	$ ps aux | grep printtest
	root 9441 0.0 0.2 14812 7696 pts/0 T 01:26 0:00 gdb ./printtest
	root 9443 0.0 0.0 1712 252 pts/0 t 01:26 0:00 /root/lib/printtest
	root 9447 0.0 0.0 3548 796 pts/0 R+ 01:29 0:00 grep printtest
	$ grep ld /proc/9443/maps
	b7fe2000-b7ffe000 r-xp 00000000 08:01 588962 /lib/i386-linux-gnu/ld-2.13.so
	b7ffe000-b7fff000 r--p 0001b000 08:01 588962 /lib/i386-linux-gnu/ld-2.13.so
	b7fff000-b8000000 rw-p 0001c000 08:01 588962 /lib/i386-linux-gnu/ld-2.13.so
	```
	- following code ends up in loader *ld*, which is responsible for patching GOT entry with full address of `libfunc`
- after first call, address of call instruction changes to actual address of `libfunc`:
	``` data
	(gdb) break *0x080485cf
	Breakpoint 1 at 0x80485cf
	(gdb) run
	Starting program: /root/lib/printtest
	test is: 42
	Breakpoint 1, 0x080485cf in main ()
	(gdb) disassemble 0x8048490
	Dump of assembler code for function libfunc@plt:
		0x08048490 <+0>: jmp DWORD PTR ds:0x8049838
		0x08048496 <+6>: push 0x8
		0x0804849b <+11>: jmp 0x8048470
	End of assembler dump.
	(gdb) x/x 0x8049838
	0x8049838 <libfunc@got.plt>: 0xb7fdd530
	(gdb) disassemble 0xb7fdd530
	Dump of assembler code for function libfunc:
		0xb7fdd530 <+0>: push ebp
		0xb7fdd531 <+1>: mov ebp,esp
		0xb7fdd533 <+3>: mov eax,0x3039
		0xb7fdd538 <+8>: pop ebp
		0xb7fdd539 <+9>: ret
	End of assembler dump.
	```
- knowing whether the function was called at least once can be valuable

## 11. Linux DS: lists
- at `include/linux/list.h` of Linux kernel source code
- type-generic implementations of doubly linked lists and hash tables are included
- `list_head` is used for storing lists:
	``` python
	>>> dt("list_head")
	'list_head' (8 bytes)
	0x0 : next ['pointer', ['list_head']]
	0x4 : prev ['pointer', ['list_head']]
	```
- `LIST_HEAD(name)` is a macro that takes the name of the list and declares it
	``` C
	static LIST_HEAD(modules);
	// after macro expansion
	struct list_head module = {&module, &module};
	```
- `INIT_LIST_HEAD(list)` function takes a previously allocated `list_head` structure and sets `prev` and `next` to list address
- list API also supports insertions at beginning(`list_add`) and end(`list_add_tail`):
	``` C
	// race-condition safe version of list_add()
	list_add_rcu(&mod->list, &modules);
	```
- `list_del` can be used to delete an element from a list:
	``` C
	list_del(&mod->list);
	```
- can iterate lists with `list_for_each`, `list_for_each_entry`, and `list_for_each_prev` macros:
	``` C
	struct module *mod;
	list_for_each_entry_rcu(mod, &modules, list) {
		<snip>
	}
	```
	- *Volatility* provides an API that allows walking doubly linked lists in a similar manner; used in `linux_lsmod` plugin:
		``` python
		def calculate(self):
			linux_common.set_plugin_members(self)
			modules_addr = self.addr_space.profile.get_symbol("modules")

			modules = obj.Object("list_head", vm = self.addr_space, offset = modules_addr)

			# walk the modules list
			for module in modules.list_of_type("module", "list"):
				<snip>
		```

## 12. Linux DS: hash tables
- similar structure and API compared to lists
- main DS ,`hlist_head` and `hlist_node`:
	``` python
	>>> dt("hlist_head")
	'hlist_head' (4 bytes)
	0x0 : first ['pointer', ['hlist_node']]
	>>> dt("hlist_node")
	'hlist_node' (8 bytes)
	0x0 : next ['pointer', ['hlist_node']]
	0x4 : pprev ['pointer', ['pointer', ['hlist_node']]]
	```
- hash tables are initialized using `HLIST_HEAD(name)` and `INIT_HLIST_HEAD(ptr)` macros
- `hlist_node` is initialized with `INIT_HLIST_NODE`
- `hlist_add_head`, `hlist_add_before`, and `hlist_add_after` can add elements
- `hlist_for_each` and `hlist_for_each_entry` enumerates a hash table:
	- `tpos` argument to `hlist_for_each_entry` is a variable of hash table type
	- `pos` is a variable of type `hlist_node` to use for looping

## 13. Linux DS: trees
- used when many elements must be tracked and searched efficiently; e.g. memory range of a process
- supported implementations of trees:
	- radix tree
	- red-black tree
- red-black trees are implemented in `include/linux/rbtree.h` and `lib/rbtree.c`
- red-black tree implementation in kernel requires users of the API to implement their own insertion and search functions; increases performance and allows generic types

## 14. handling embedded structures
- `container of` macro allows simple retrieval of structure members within embedded structures:
	``` C
	// macro definition
	#define container_of(ptr, type, member) ({\
		const typeof( ((type *)0)->member ) *__mptr = (ptr); // type safety check
		(type *)( (char *)__mptr - offsetof(type,member) );}) // get address of container
	```
	- example usage:
		``` C
		// socket_alloc structure
		struct socket_alloc {
			struct socket socket;
			struct inode vfs_inode;
		};

		static inline struct socket *SOCKET_I(struct inode *inode)
		{
			return &container_of(inode, struct socket_alloc, vfs_inode)->socket;
		}
		```

## 15. Linux address translation - kernel identity paging
- Linux identity maps the kernel's code and data
- identity paging is when VA is same as or at a constant offset from the corresponding physical offset
- leveraging identity paging to locate data associated with process in memory:
	1. use `linux_volshell` to get offset of name member(`comm`) within `task_struct`:
		``` python
		>>> addrspace().profile.get_obj_offset("task_struct", "comm")
		516
		```
	2. use *System.map* to find VA of process list head(`init_task`):
		``` sh
		$ grep -w init_task /boot/System.map-3.2.0-4-686-pae
		c13defe0 D init_task
		```
	3. subtracting `0xc0000000`(constant offset of kernel code and data pages on 32-bit Linux) and adding offset 516 gives 20836836
	4. use dd to recover process name:
		``` sh
		$ dd if=/dev/fmem bs=1 count=16 skip=20836836 | xxd
		16+0 records in
		16+0 records out
		16 bytes (16 B) copied, 0.000139388 s, 115 kB/s
		0000000: 7377 6170 7065 722f 3000 0000 0000 0000 swapper/0.......
		```
- *Volatility* uses this to quickly find the kernel's *Directory Table Base(DTB)* value
- can be used to determine whether memory acquisition was successful

## 16. Linux address translation - DTB
- identity paging doesn't work for all regions of memory
- address of initial DTB(`swapper_pg_dir`) is stored in both the *System.map* file and within the identity mapped region of the kernel
- finding DTB on 32-bit systems:
	``` python
	class VolatilityDTB(obj.VolatilityMagic):
		"""A scanner for DTB values."""

		def generate_suggestions(self):
			"""Tries to locate the DTB."""
			shift = 0xc0000000
			yield self.obj_vm.profile.get_symbol("swapper_pg_dir") - shift
	```
	- works the same on 64-bit systems, except symbol used is `init_level4_pgt` and shift value is `0xffffffff80000000`
