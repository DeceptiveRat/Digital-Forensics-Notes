## 1. userland analysis objectives
- detect shellcode injection
- find shared library injection
- analyze GOT/PLT overwriting
- trace inline function hooks

## 2. shellcode injection
- shellcode is binary data that contains CPU instructions 
- shellcode injection prerequisites:
	- open handle to target process
	- memory region within target process that is writable and executable must be found/allocated

## 3. shellcode injection process
1. attaching to process:
	- `ptrace` allows process with sufficient privileges to have full control over another process
	- `ptrace` final argument:
		- `PTRACE_ATTACH`: attach to target process and pause it, returning handle if successful
		- `PTRACE_PEEKTEXT`: read data from address space of process
		- `PTRACE_POKETEXT`: write data into address space
		- `PTRACE_GETREGS`: read general purpose registers from process
		- `PTRACE_SETREGS`: set any general purpose registers of process
		- `PTRACE_CONT`: continue paused process
		- `PTRACE_STOP`: pause process 
		- `PTRACE_DETACH`: detach from process
2. finding/allocating memory:
	- write shellcode in slack space of code section
	- overwrite functions that execute once so it doesn't execute again
	- for large payloads, inject small shellcode that uses `mmap` to allocate new executable region
3. writing shellcode into process:
	- use `ptrace` with `PTRACE_POKETEXT` argument or `process_vm_writev` to write shellcode
4. controlling foreign process execution:
	1. call `ptrace` with `PTRACE_GETREGS` 
	2. overwrite instruction pointer register so it points to shellcode
	3. use `PTRACE_SETREGS` to update registers
	4. use `PTRACE_CONT` to resume process

## 4. *linux_malfind* plugin
- walks memory mappings of a process looking for for suspicious protection bits, readable/writable/executable
- example output:
	``` data
	$ python vol.py -f injtarget.lime --profile=LinuxDebian3_2x86 linux_malfind
	Volatility Foundation Volatility Framework 2.4
	Process: target Pid: 16929 Address: 0x1011000 File: Anonymous Mapping
	Protection: VM_READ|VM_WRITE|VM_EXEC
	Flags: VM_READ|VM_WRITE|VM_EXEC|VM_MAYREAD|VM_MAYWRITE|VM_MAYEXEC|VM_ACCOUNT
	0x01011000 31 c0 31 db 31 c9 31 d2 b0 66 b3 01 51 6a 06 6a 1.1.1.1..f..Qj.j
	0x01011010 01 6a 02 89 e1 cd 80 89 c6 b0 66 b3 02 52 66 68 .j........f..Rfh
	0x01011020 7a 69 66 53 89 e1 6a 10 51 56 89 e1 cd 80 b0 66 zifS..j.QV.....f
	0x01011030 b3 04 6a 01 56 89 e1 cd 80 b0 66 b3 05 52 52 56 ..j.V.....f..RRV
	0x1011000 31c0 XOR EAX, EAX
	0x1011002 31db XOR EBX, EBX
	0x1011004 31c9 XOR ECX, ECX
	0x1011006 31d2 XOR EDX, EDX
	[snip]
	```

## 5. process hollowing
- act of overwriting process or portions of process in memory with malicious code
- often used for detection evasion

## 6. *linux_process_hollow* plugin
- relies on having a known good copy of application's instructions to compare with
- kernel's file cache or disk cannot always be trusted to be unmodified
- requires:
	- path to trusted binary
	- PID to operate on
	- address of application mapped into process memory
- starts by reading symbol table of trusted file to determine where each function is loaded, then compares each function with actual code in memory
- direct relocations, false positives that overwrite function addresses at runtime, are accounted for by parsing the relocation table
- example output:
	``` sh
	# get PID
	$ python vol.py -f sharedlib.lime --profile=LinuxDebian3_2x86 linux_pslist | grep bash
	Volatility Foundation Volatility Framework 2.4
	Offset Name Pid Uid Gid DTB Start Time
	------------------ ----- ----- ---- --------- ----------------------------
	0xf7116140 bash 18550 0 0 0x35acd000 2014-02-25 12:09:19 UTC+0000
	# get address of executable region
	$ python vol.py -f sharedlib.lime --profile=LinuxDebian3_2x86 linux_proc_maps -p 18550 | grep bash
	Volatility Foundation Volatility Framework 2.4
	Pid Start End Flags Pgoff Major Minor Inode File Path
	----- --------- --------- ----- ---- ----- ------ ------ -------------------
	18550 0x8048000 0x8049000 r-x 0x0 8 1 948592 /bin/bash
	18550 0x8049000 0x804a000 rw- 0x0 8 1 948592 /bin/bash
	# search for discrepancy
	$ python vol.py -f sharedlib.lime --profile=LinuxDebian3_2x86 linux_process_hollow -p 18550 -b 0x8048000 –P /mnt/baselines/bin/bash
	Volatility Foundation Volatility Framework 2.4
	Task PID Symbol Name Symbol Address
	-------- ---- ----------- --------------
	bash 18550 main 0x80485bc
	```
	- note there are 2 mappings; one is for executable data and the other is for data mapping

## 7. shared library injection
- writing full-featured software in shellcode is difficult because it must be position-independent and cannot directly rely on API calls
- malware is often implemented as shared libraries instead
- 2 methods exist:
	- loading shared libary stored on disk
	- loading library located in memory

## 8. disk-based library injection
- Linux dynamic loader provides functions `dlopen`, `dlclose`, and `dlsym`; equivalent to `LoadLibrary`, `FreeLibrary`, and `GetProcAddress` from Windows
- forcing target process to call `_dlopen` with first parameter set to path of library injects library

## 9. disk-based library injection detection
- use *linux_proc_maps* plugin:
	``` data
	$ python vol.py -f sharedlib.lime --profile=LinuxDebian3_2x86 linux_proc_maps -p 18550
	Volatility Foundation Volatility Framework 2.4
	Pid Start End Flags Pgoff Major Minor Inode File Path
	-------- -------- ----------- ------ ----- ----- ------ ---------- -------
	<snip>
	18550 0xb77b1000 0xb77b2000 r-x 0x0 8 1 1439419 /tmp/.XICE-unix
	18550 0xb77b2000 0xb77b3000 rw- 0x0 8 1 1439419 /tmp/.XICE-unix
	<snip>
	```
	- suspicious file path
- *linux_library_list* plugin designed to report libraries mapped into process:
	- analyzes list of loaded libraries that dynamic linker in userland maintains; similar to list of DLLs kept in PEB on Windows
	- mappings are represented by `link_map`, which maintains starting address and path of libraries mapped using functions like `dlopen`
	- example output:
		``` data
		$ python vol.py -f sharedlib.lime --profile=LinuxDebian3_2x86 linux_library_list -p 18550
		Volatility Foundation Volatility Framework 2.4
		Task Pid Load Address Path
		---------------- -------- ------------ ----
		bash 18550 0x00b7647000 /lib/i386-linux-gnu/i686/cmov/libc.so.6
		bash 18550 0x00b77b7000 /lib/ld-linux.so.2
		bash 18550 0x00b77b1000 /tmp/.XICE-unix
		[snip]
		```
- *linux_ldrmodules* plugin cross-references libraries found through kernel's list of per-process memory mappings and linked list of libraries kept by the dynamic linker:
	``` data
	$ python vol.py -f sharedlib.lime --profile=LinuxDebian3_2x86 linux_ldrmodules -p 18550
	Volatility Foundation Volatility Framework 2.4
	Pid Name Start File Path Kernel Libc
	----- ------- --------- ------------------- ------ ------
	18550 bash 0x08048000 /bin/bash True False
	18550 bash 0xb7647000 /lib/[snip]/cmov/libc-2.13.so True True
	18550 bash 0xb77b7000 /lib/i386-linux-gnu/ld-2.13.so True True
	18550 bash 0xb77b1000 /tmp/.XICE-unix True True
	[snip]
	```
	- binary for process only found in process mappings because it is not loaded through dynamic linker; `task.mm.start_code` address can be used to determine which entry corresponds to process

## 10. memory-based library injection
- avoids artifacts such as the library file and corresponding `/proc/<pid>/maps` entries
- methods:
	- imitate `execve` in usermode, allowing processes to be created without involving kernel and writing to disk
	- use `mmap` to allocate memory region in target process to store malicious library code and hook dynamic loader's APIs so they work on binaries already loaded, then use `_dl_open` on the library

## 11. memory-based library injection detection
- *linux_malfind* plugin to detect initial shellcode used
- *linux_library_list* plugin to find library loaded with `_dl_open`
- *linux_ldrmodules* plugin will find discrepancy caused by using `execve` method which does not use dynamic linker 

## 12. *linux_procdump* and *linux_librarydump* plugins
- purpose is extracting executables in native ELF form, not regions of disjointed memory
- functionality:
	1. *Volatility* provides function that takes `task_struct` and VA of ELF header and returns instantiated `elf_hdr` object at given address
	2. enumerate program headers of file, dump sections of type `PT_LOAD` 
- example output:
	``` sh
	# find VA
	$ python vol.py -f sharedlib.lime --profile=LinuxDebian3_2x86 linux_proc_maps -p 18550 | grep ICE
	Volatility Foundation Volatility Framework 2.4
	18550 0xb77b1000 0xb77b2000 r-x 0x0 8 1 1439419 /tmp/.XICE-unix
	18550 0xb77b2000 0xb77b3000 rw- 0x0 8 1 1439419 /tmp/.XICE-unix
	# extract ELF
	$ python vol.py -f sharedlib.lime --profile=LinuxDebian3_2x86 linux_librarydump -p 18550 -D outdir -b 0xb77b1000
	Volatility Foundation Volatility Framework 2.4
	Offset Name Pid Address Output File
	---------- ----------- --------- ---------- -----------
	0xf7116140 bash 18550 0xb77b1000 outdir/bash.18550.0xb77b1000
	$ file outdir/bash.18550.0xb77b1000
	outdir/bash.18550.0xb77b1000: ELF 32-bit LSB shared object, Intel 80386,
	version 1 (SYSV), dynamically linked,
	BuildID[sha1]=0x8200585e3443980847707c3f63eaa38f531fb67a, not stripped
	```

## 13. `LD_PRELOAD` rootkits
- `LD_PRELOAD` is an EV that specifies path to shared library
- any program run with `LD_PRELOAD` set has specified library loaded in address space first; easy to hook API functions because preloaded library is used to resolve symbols first
- example library:
	``` C
	#include <stdio.h>
	#include <dlfcn.h>
	#include <unistd.h>
	#include <fcntl.h>

	ssize_t (*orig_write)(int, const void *, size_t);
	int log_fd;

	// fake write function
	ssize_t write(int fd, const void *buf, size_t count)
	{
		void *handle;

		if (!orig_write)
		{
			// resolves original write function
			handle = dlopen("libc.so.6", RTLD_LAZY);
			orig_write = dlsym(handle, "write");
			// create log file
			log_fd = open("/tmp/logfile.txt", O_CREAT|O_WRONLY, 0666);
		}
	
		// data logged and original write function called
		orig_write(log_fd, buf, count);
		return orig_write(fd, buf, count);
	}
	```

## 14. detecting `LD_PRELOAD` rootkits
- if path of library is written to `/etc/ld.so.preload` to load into every process:
	``` sh
	# get file inode
	$ python vol.py --profile=LinuxUbuntu1204x64 -f jynxkit.mem linux_find_file -F /etc/ld.so.preload
	Volatility Foundation Volatility Framework 2.4
	Inode Number Inode
	---------------- ------------------
	263883 0xffff88003be9b440
	# extract file
	$ python vol.py --profile=LinuxUbuntu1204x64 -f jynxkit.mem linux_find_file -i 0xffff88003be9b440 -O ld.so.preload
	Volatility Foundation Volatility Framework 2.4
	$ cat ld.so.preload
	/XxJynx/jynx2.so
	# search all process mappings for malicious library
	$ python vol.py --profile=LinuxUbuntu1204newx64 -f jynxkit.mem linux_proc_maps | grep -c /XxJynx/jynx2.so
	Volatility Foundation Volatility Framework 2.4
	364
	```
- if `LD_PRELOAD` EV is set in user's bash preferences:
	``` data
	$ python vol.py -f ncpreload.lime --profile=LinuxDebian3_2x86 linux_psenv -p 14259
	Volatility Foundation Volatility Framework 2.4
	Name Pid Environment
	nc 14259 TERM=xterm SHELL=/bin/bash SSH_CLIENT=192.168.174.1 51514 22
	LD_PRELOAD=/root/ldpre/netlib.so OLDPWD=/root SSH_TTY=/dev/pts/2 USER=root
	MAIL=/var/mail/root PATH=/root/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:
	/usr/bin:/sbin:/bin PWD=/root/ldpre LANG=en_US.UTF-8 SHLVL=1 HOME=/root
	LOGNAME=root SSH_CONNECTION=192.168.174.1 51514 192.168.174.128 22
	_=/bin/nc /bin/nc
	```
- checking GOT/PLT: function pointers within GOT would point to preloaded library
- comparative symbol analysis:
	1. user `dlsym` function to request address of a number of commonly hooked functions
	2. requests same function from `dlsym` with `RTLD_NEXT` flag set, skipping first library found
	3. compare results for mismatch

## 15. GOT/PLT overwrite
- allows redirecting calls to functions
- overwriting GOT requires capabilities:
	- read/write data in foreign process; done with `ptrace`
	- find target GOT entry; query relocation table which contains address of resolved functions within GOT
	- find address of hook function within target process' address space; done with `dlopen` and `dlsym`

## 16. detecting GOT overwrites
1. walk dynamic linking information of all applications and all loaded libraries
2. record resolved address and symbol name for all functions within relocation table:
	- filter out libraries that are lazily loaded; points to within ELF object
3. identify libraries the application needs:
	- these libraries are determined at compile time
	- stored as `DT_NEEDED` entries within dynamic linking information
4. validate each resolved GOT:
	- verify resolved address points to one of the needed libraries; ensures function points to library linked with at compile time

## 17. *linuX_plthook* plugin
- example output:
	``` data
	$ python vol.py --profile=LinuxDebian3_2x86 -f preload.lime
	linux_plthook -p 22996
	Volatility Foundation Volatility Framework 2.4
	Task ELF Start ELF Name Symbol Resolved Address Target Info
	22996 0x08048000 /usr/sbin/sshd __xstat64 0x000000b7743fc9 /root/jynx2.so
	22996 0x08048000 /usr/sbin/sshd write 0x000000b774327a /root/jynx2.so
	22996 0x08048000 /usr/sbin/sshd fopen64 0x000000b7742f32 /root/jynx2.so
	22996 0x08048000 /usr/sbin/sshd __lxstat64 0x000000b7743930 /root/jynx2.so
	22996 0x08048000 /usr/sbin/sshd opendir 0x000000b7744432 /root/jynx2.so
	[snip]
	```

## 18. inline hooking
- overwrites first several bytes of function and replaces them with instructions of malware's choosing
- used frequently because GOT can be marked read-only (RELRO) after process is first loaded

## 19. *linux_apihooks* plugin
- detects inline hooks by:
	1. gathering list of functions an application uses in same manner as *linux_plthook*
	2. disassembling first several instructions looking for control flow transfers
- example output:
	``` data
	$ python vol.py -f hooks.lime --profile=LinuxDebian-3_2x64 linux_apihooks -p 65033
	Volatility Foundation Volatility Framework 2.4
	Hook VMA Hooked Symbol Symbol Address Type Hook Address
	------------------------ ---------- ------------- ----- --------------
	/lib/<snip>/libc-2.13.so write 0x7ff29b512c06 JMP 0x7ff31c069a45
	/lib/<snip>/libc-2.13.so close 0x7ff29b512c46 JMP 0x7ff31c069a96
	/lib/<snip>/libc-2.13.so read 0x7ff29b512c56 JMP 0x7ff31c068e45
	[snip]
	```
	- follow `Hook Address` column to investigate further
