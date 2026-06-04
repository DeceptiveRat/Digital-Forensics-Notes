## 1. kernel mode rootkits
- have power to add, delete, or modify data that kernel or userland applications request about state of system
- can hook many places to subvert system

## 2. accessing kernel mode
- attackers must first gain root-level privileges
- with root level privilege, attackers can leverage native methods Linux provides for writing into kernel address space:
	- *Loadable Kernel Modules(LKM)*:
		- popular method of getting access to kernel
		- have full access to kernel APIs and DS; makes rootkits portable
	- `/dev/mem` and `/dev/kmem`

## 3. hidden kernel modules
- LKMs generally try to hide themselves; easily done by deleting from module list in kernel memory:
	``` C
	list_del_init(&__this_module.list);
	```
- hiding from sysfs:
	- `/sys/module` directory has subdirectory for each active module
	- can hide from sysfs via:
		``` C
		kobject_del(__this_module.holders_dir->parent);
		```

## 4. *linux_hidden_modules* plugin
- modules found through carving but not linked list of modules are reported
- may include freed modules as well
- uses `module_addr_min` and `module_addr_max` variables in kernel, which indicate range of addresses where kernel module structures are allocated
- example output:
	``` data
	$ python vol.py --profile=LinuxDebian-3_2x64 -f susnf.lime linux_hidden_modules
	Volatility Foundation Volatility Framework 2.4
	Offset (V) Name
	------------------ ----
	0xffffffffa03a15d0 suterusu
	```

## 5. *linux_kernel_opened_files* plugin
- can identify artifacts from malicious LKM; e.g. files open within kernel, except for swap devices, is suspicious
- operates by:
	1. gathering all recently used `dentry` structures from `dentry` cache
	2. comparing with set of files that processes opened (using *linux_lsof* and *linux_proc_maps* plugins)
- example output:
	``` data
	$ python vol.py --profile=LinuxDebian-3_2x64 -f suskl2.lime linux_kernel_opened_files
	Volatility Foundation Volatility Framework 2.4
	Offset (V) Partial File Path
	------------------ -----------------
	0xffff8800307109c0 /media/usb/suskl2.lime
	0xffff8800306e5800 /root/.keylog
	```

## 6. kernel mode code injection
1. rootkit loads kernel module from disk
2. in module `init` function, allocate executable kernel memory by calling functions like `kmalloc` and `vmalloc_exec`
3. rootkit code is copied to new executable region and data copied to read/write region
4. kernel module starts new thread, registers callbacks, or hooks code that points to allocated regions
5. module's `init` function returns error condigion so module is unloaded and metadata structure (`module`) destroyed, hiding from `lsmod` or `sysfs`

## 7. *linux_psxview* plugin
- process listing sources:
	- process list: 
		- list of active processes that `init_task` symbol points to 
		- used by *linux_pslist*
	- PID hash table:
		- populated from *linux_pidhashtable* plugin
		- enumerates threads as well as processes
	- memory cache: populated by finding `task_struct` in kernel memory cache (`kmem_cache`) for systems that use SLAB allocator
	- parents: populated by following `parent` pointers of processes and threads found in PID hash table
	- leaders: populated by gathering thread group leader pointer of each process and thread
- example output:
	``` data
	$ python vol.py --profile=LinuxDebian-3_2x64 -f psrk.lime linux_psxview
	Volatility Foundation Volatility Framework 2.4
	Offset(V) Name PID pslist pid_hash kmem_cache parents leader
	------------------ -------------- ---- ------ -------- ---------- ------- ------
	[snip]
	0xffff88003e2a2100 kintegrityd 19 True True True False True
	0xffff880036d60830 postgres 2514 False False True False True
	0xffff880036cce0c0 scsi_eh_2 155 True True True False True
	[snip]
	```
	- absence of *postgres* from lists is suspicious
- exceptions:
	- threads only appear in `pid_hash` and `kmem_cache`; verify thread by checking for thread ID in output of *linux_thread* plugin
	- `kmem_cache` entries are all false for SLUB systems and all true for SLAB systems; should be all true or all false
	- all other entries should be present in process list except *swapper* process

## 8. privilege escalation analysis objectives
- find processes with hijacked credentials
- inspect rootkit specific DS

## 9. privilege escalation DS
- process credentials are stored in `cred`:
	``` python
	>>> dt("cred")
	'cred' (152 bytes)
	0x0 : usage ['__unnamed_0x38e']
	0x4 : uid ['unsigned int']
	0x8 : gid ['unsigned int']
	0xc : suid ['unsigned int']
	0x10 : sgid ['unsigned int']
	0x14 : euid ['unsigned int']
	0x18 : egid ['unsigned int']
	0x1c : fsuid ['unsigned int']
	0x20 : fsgid ['unsigned int']
	0x24 : securebits ['unsigned int']
	0x28 : cap_inheritable ['kernel_cap_struct']
	0x30 : cap_permitted ['kernel_cap_struct']
	0x38 : cap_effective ['kernel_cap_struct']
	[snip]
	```
	- `uid`
	- `gid`
	- `euid`: 
		- effective user ID of process
		- when user invokes suid application, uid is that of user and euid is user for which suid is used
	- `egid`: effective group ID of process

## 10. hijacked credentials
- kernel provides APIs, `prepare_creds` and `commit_creds`, for creating `cred`; rootkit can create own credential structures and give themselves root
- can point `cred` to that of existing process, preferably one that is root and doesn't exist

## 11. *linux_check_creds* plugin
- can detect processes sharing `cred` structure
- example output:
	``` data
	$ python vol.py -f avg.hidden-proc.lime --profile=LinuxDebian-3_2x64 linux_check_creds
	Volatility Foundation Volatility Framework 2.4
	PIDs
	--------
	1, 9673
	```

## 12. *linux_check_syscall* plugin
- used to detect hooked system call tables
- locates the system call table and validates each entry
- example output:
	``` data
	$ python vol.py -f kbeast.lime --profile=LinuxDebianx86 linux_check_syscall -i /usr/include/x86_64-linux-gnu/asm/unistd_32.h > ksyscall
	$ head -10 ksyscall
	Table Index System Call Handler Symbol
	----- ----- ---------------- ---------- ----------------------------------
	32bit 0 restart_syscall 0xc103be51 sys_restart_syscall
	32bit 1 exit 0xc1033c85 sys_exit
	32bit 2 fork 0xc100333c ptregs_fork
	32bit 3 read 0xf841998b HOOKED: ipsecs_kbeast_v1/h4x_read
	32bit 4 write 0xf84191a1 HOOKED: ipsecs_kbeast_v1/h4x_write
	32bit 5 open 0xf84194fc HOOKED: ipsecs_kbeast_v1/h4x_open
	32bit 6 close 0xc10b227a sys_close
	32bit 7 waitpid 0xc10334d8 sys_waitpid
	$ grep HOOKED ksyscall
	32bit 3 read 0xf841998b HOOKED: ipsecs_kbeast_v1/h4x_read
	32bit 4 write 0xf84191a1 HOOKED: ipsecs_kbeast_v1/h4x_write
	32bit 5 open 0xf84194fc HOOKED: ipsecs_kbeast_v1/h4x_open
	32bit 10 unlink 0xf841927f HOOKED: ipsecs_kbeast_v1/h4x_unlink
	32bit 37 kill 0xf841900e HOOKED: ipsecs_kbeast_v1/h4x_kill
	32bit 38 rename 0xf841940c HOOKED: ipsecs_kbeast_v1/h4x_rename
	32bit 40 rmdir 0xf8419300 HOOKED: ipsecs_kbeast_v1/h4x_rmdir
	32bit 220 getdents64 0xf84190ef HOOKED: ipsecs_kbeast_v1/h4x_getdents64
	32bit 301 unlinkat 0xf8419381 HOOKED: ipsecs_kbeast_v1/h4x_unlinkat
	```
	- the fact that malicious handlers are so close to each other indicates they point to same kernel module or injected code segment

## 13. keyboard notifier analysis objectives
- locate keyboard notifiers
- determine associated modules and symbols of malicious functions

## 14. keyboard notifier DS
- `notifier_block`:
	``` python
	>>> dt("notifier_block")
	'notifier_block' (24 bytes)
	0x0 : notifier_call ['pointer', ['void']]
	0x8 : next ['pointer', ['notifier_block']]
	0x10 : priority ['int']
	```
	- `notifier_call`: reference to callback function that executes when key is pressed
	- `next`: pointer to list of keyboard notifiers

## 15. *linux_keyboard_notifier* plugin
- walks list of keyboard notifiers and prints each one, along with handler
- example output:
	``` data
	$ python vol.py --profile=LinuxDebian-3_2x64 -f susnf.lime linux_keyboard_notifier
	Volatility Foundation Volatility Framework 2.4
	Address Symbol
	------------------ -------------------
	0xffffffffa039f5d7 HOOKED
	```

## 16. resolving malicious functions
1. use *linux_hidden_modules* plugin to carve hidden modules:
	``` data
	$ python vol.py --profile=LinuxDebian-3_2x64 -f susnf.lime linux_hidden_modules
	Volatility Foundation Volatility Framework 2.4
	Offset (V) Name
	------------------ ----
	0xffffffffa03a15d0 suterusu
	```
2. use *linux_lsmod* plugin to determine where sections are mapped:
	``` data
	$ python vol.py --profile=LinuxDebian-3_2x64 -f susnf.lime linux_lsmod -b 0xffffffffa03a15d0 -S
	Volatility Foundation Volatility Framework 2.4
	ffffffffa03a15d0 suterusu 18928
	.note.gnu.build-id 0xffffffffa03a0000
	.text 0xffffffffa039e000
	.text.unlikely 0xffffffffa039fdcc
	.init.text 0xffffffffa0030000
	.exit.text 0xffffffffa039fdfc
	.rodata 0xffffffffa03a0028
	.rodata.str1.1 0xffffffffa03a00c5
	.parainstructions 0xffffffffa03a0ab0
	.data 0xffffffffa03a1000
	.gnu.linkonce.this_module 0xffffffffa03a15d0
	.bss 0xffffffffa03a1820
	.symtab 0xffffffffa0031000
	.strtab 0xffffffffa00322a8
	```
	- handler at `0xffffffffa039f5d7` is inside `.text` section
3. use *linux_moddump* plugin to extract module to disk and search symbol table:
	``` data
	$ readelf -s -W suterusu.0xffffffffa039e000.lkm | grep 15d7
	28: 00000000000015d7 129 FUNC GLOBAL DEFAULT 2 notify
	```

## 17. TTY handlers
- another keylogging method
- reads inputs on TTY devices such as terminals
- overwrites a function pointer with malware-controlled pointer so it can gain access to each keystroke

## 18. TTY handler analysis objectives
- locate TTY devices in memory
- determine whether malicious handler is installed

## 19. TTY handler DS
- `tty_struct` holds information on a keyboard notifier:
	``` python
	>>> dt("tty_struct")
	'tty_struct' (1280 bytes)
	0x0 : magic ['int']
	0x4 : kref ['kref']
	0x8 : dev ['pointer', ['device']]
	0x10 : driver ['pointer', ['tty_driver']]
	0x18 : ops ['pointer', ['tty_operations']]
	0x20 : index ['int']
	0x28 : ldisc_mutex ['mutex']
	0x48 : ldisc ['pointer', ['tty_ldisc']]
	0x50 : termios_mutex ['mutex']
	0x70 : ctrl_lock ['spinlock']
	0x78 : termios ['pointer', ['ktermios']]
	0x80 : termios_locked ['pointer', ['ktermios']]
	0x88 : termiox ['pointer', ['termiox']]
	0x90 : name ['String', {'length': 64}]
	0xd0 : pgrp ['pointer', ['pid']]
	0xd8 : session ['pointer', ['pid']]
	0xe0 : flags ['unsigned long']
	0xe8 : count ['int']
	<snip>
	```
	- `dev`: reference to device structure
	- `ldisc`: 
		- reference to operations structure for TTY device; receives input and handles read/write operations
		- inspect to ensure no malicious hooking occurred
	- `name`: name of device
	- `session`: 
		- session of device
		- used to associated device with logins and processes
	
## 20. hooking TTY handler
1. locate `file` pointer structure of TTY device:
	- pass name of device to `file_open` function
2. use `private_data` member to get reference to `tty_file_private` structure, which points to `tty_struct`
3. overwrite `tty_struct->ldisc->ops->receive_buf` poiter with address of key logging function

## 21. *linux_check_tty* plugin
- detects by:
	1. walks list of TTY drivers stored within `tty_drivers` global variable
	2. within each `tty_driver` structure, access `ttys` member, which holds list of TTY devices managed by driver
	3. for each TTY device, validate `receive_buf` pointer
- example output:
	``` data
	$ python vol.py -f centos.lime --profile=LinuxCentosx64 linux_check_tty
	Volatility Foundation Volatility Framework 2.4
	Name Address Symbol
	---------------- ------------------ ------------------------------
	tty1 0xffffffff8131a0b0 n_tty_receive_buf
	tty2 0xffffffff8131a0b0 n_tty_receive_buf
	tty3 0xffffffff8131a0b0 n_tty_receive_buf
	tty4 0xffffffff8131a0b0 n_tty_receive_buf
	tty5 0xffffffff8131a0b0 n_tty_receive_buf
	tty6 0xffffffff8131a0b0 n_tty_receive_buf
	```
	- clean system; all point to `n_tty_receive_buf`
- infected system output:
	``` data
	$ python vol.py -f tty-hook.lime --profile=LinuxDebian-3_2x64 linux_check_tty
	Volatility Foundation Volatility Framework 2.4
	Name Address Symbol
	---------------- ------------------ ------------------------------
	tty1 0xffffffffa0427016 HOOKED
	tty2 0xffffffffa0427016 HOOKED
	tty3 0xffffffffa0427016 HOOKED
	```
	- all devices have `receive_buf` overwritten with same function; they share a default operations structure
- possible to make a copy of default operations structure and point only desired device to copy

## 22. network protocol
- popular way to hide connection information from userland 
- done by overwriting function pointers associated with `/proc` operation functions

## 23. network protocol analysis objectives
- locate hooked sequence operations structures within network protocols

## 24. network protocol DS
- `seq_operations` stores pointers to handlers of data exported from kernel subsystems:
	``` python
	>>> dt("seq_operations")
	'seq_operations' (32 bytes)
	0x0 : start ['pointer', ['void']]
	0x8 : stop ['pointer', ['void']]
	0x10 : next ['pointer', ['void']]
	0x18 : show ['pointer', ['void']]
	```
	- `start`: retrieves first record of sequence
	- `stop`: determines when sequence ended
	- `next`: retrieves next record of sequence
	- `show`: 
		- shows information of a record such as network connection state or fields of process
		- often targeted to hide data from live forensics

## 25. network sequence operations
- files under `/proc/net/` directory export all network related information to userland:
	``` sh
	# network interface information
	$ cat /proc/net/dev
	Inter-| Receive | Transmit
	face |bytes packets errs drop fifo compressed multicast|bytes packets
	lo: 3396353 10929 0 0 0 0 0 3396353 10929
	eth0: 7412216 69623 0 0 0 0 0 17223107 57677
	# arp cache 
	$ cat /proc/net/arp
	IP address HW type Flags HW address Mask Device
	192.168.174.2 0x1 0x2 00:50:56:fa:ad:55 * eth0
	192.168.174.1 0x1 0x2 00:50:56:c0:00:08 * eth0
	192.168.174.254 0x1 0x2 00:50:56:e7:00:e1 * eth0
	# active network socket
	$ cat /proc/net/tcp
	sl local_address rem_address st uid inode
	0: 00000000:C60E 00000000:0000 0A 102 4965
	1: 00000000:006F 00000000:0000 0A 0 6108
	2: 00000000:0016 00000000:0000 0A 0 6404
	3: 0100007F:1538 00000000:0000 00 104 7585
	4: A9AEA8C0:0016 01AEA8C0:D878 01 0 16238
	5: A9AEA8C0:0016 01AEA8C0:D8B9 01 0 16287
	```

## 26. *linux_check_afinfo* plugin
- validates each handler within file operations and sequence operation structures of each network protocol
- included protocols:
	- tcp4: `tcp4_seq_afinfo`
	- tcp6: `tcp6_seq_afinfo`
	- udp4: `udp4_seq_afinfo`
	- udp6: `udp6_seq_afinfo`
	- udplite4: `udplite4_seq_afinfo`
	- udplite6: `udplite6_seq_afinfo`
- for each protocol, handlers are checked to be inside the kernel or within a known module
- example output:
	``` data
	$ python vol.py -f kbeast.lime --profile=LinuxDebianx86 linux_check_afinfo
	Volatility Foundation Volatility Framework 2.4
	Symbol Name Member Address
	----------- ------ ----------
	tcp4_seq_afinfo show 0xe0fb9965
	```

## 27. netfilter hook
- packet-filtering engine built into Linux kernel 
- leveraged by *iptables* to implement *Network Address Translation(NAT)* and firewall filtering
- hooks can be inserted to intercept and modify packets entering and leaving system

## 28. netfilter hook analysis objectives
- detect covert *Command and Control(C2)* servers: attackers use Netfilter hooks to send and receive covert messages embedded in normal looking packets
- detect malicious advertisement injection
- reveal network sniffing

## 29. netfilter hook DS
- `nf_hook_ops` holds information on a Netfilter hook:
	``` python
	>>> dt("nf_hook_ops")
	'nf_hook_ops' (48 bytes)
	0x0 : list ['list_head']
	0x10 : hook ['pointer', ['void']]
	0x18 : owner ['pointer', ['module']]
	0x20 : pf ['unsigned char']
	0x24 : hooknum ['unsigned int']
	0x28 : priority ['int']
	```
	- `list`: pointer to list of Netfilter hooks
	- `hook`: pointer to hook function
	- `pf`: protocol family of hook
	- `hooknum`: hook location within network stack

## 30. netfilter subsystem
- enables kernel modules to hook into a number of stages within network stack; inspection at different times during processing:
	- `PRE_ROUTING`: before packet enters routing subsystem of kernel
	- `LOCAL_IN`: 
		- for packets that are sent to computer itself
		- last hook before process receives packet data
	- `FORWARD`: before packet is forwarded to another machine
	- `LOCAL_OUT`: for packets generated on local machine
	- `POST_ROUTING`: right before packet is sent out to network
- options for processing packets:
	- `NF_DROP`: drop
	- `NF_ACCEPT`: allow packet to continue
	- `NF_STOLEN`: 
		- gives control of packet to handler
		- resources are not freed, but not sent through network stack
	- `NF_QUEUE`: queue packets for userland processing
	- `NF_REPEAT`: call hook function again
	- `NF_STOP`: process without calling other Netfilter hooks
- options are declared in `include/linux/netfilter.h` header file

## 31. *linux_netfilter* plugin
- enumerates Netfilter hooks
- works by enumerating all entries within `nf_hooks` global variable:
	``` C
	struct list_head nf_hooks[NFPROTO_NUMPROTO][NF_MAX_HOOKS]
	```
- example output:
	``` data
	$ python vol.py --profile=LinuxDebian-3_2x64 -f susnf.lime linux_netfilter
	Volatility Foundation Volatility Framework 2.4
	Proto Hook Handler Is Hooked
	----- ---------------- ------------------ ---------
	IPV4 PRE_ROUTING 0xffffffffa039fcd4 True
	```

## 32. file operations
- all file related operations go through `file_operations` structure 

## 33. file operations analysis objectives
- analyze anti-forensics techniques: can determine when rootkits are monitoring access to particular files
- detect hidden logged on users

## 34. file operations DS
- `file_operations` holds info on functions that a particular file uses for read, write, and other tasks:
	``` python
	>>> dt("file_operations")
	'file_operations' (208 bytes)
	0x0 : owner ['pointer', ['module']]
	0x8 : llseek ['pointer', ['void']]
	0x10 : read ['pointer', ['void']]
	0x18 : write ['pointer', ['void']]
	0x20 : aio_read ['pointer', ['void']]
	0x28 : aio_write ['pointer', ['void']]
	0x30 : readdir ['pointer', ['void']]
	0x38 : poll ['pointer', ['void']]
	0x40 : unlocked_ioctl ['pointer', ['void']]
	0x48 : compat_ioctl ['pointer', ['void']]
	0x50 : mmap ['pointer', ['void']]
	0x58 : open ['pointer', ['void']]
	0x60 : flush ['pointer', ['void']]
	<snip>
	```

## 35. *linux_check_fop* plugin
- gathers `file_operations` structures from:
	- opened files: *linux_lsof* plugin
	- proc file system: `/proc` FS is walked and file structures gathered
	- file cache: *linux_find_file* plugin used to find all files within page cache
- for each structure found, checks all funtion pointer members to ensure they are within static kernel or known kernel module
- example output:
	``` data
	$ python vol.py -f avgcoder.mem --profile=LinuxCentOS63x64 linux_check_fop
	Symbol Name Member Address
	-------------------------- --------------- ------------------
	proc_mnt: root readdir 0xffffffffa05ce0e0
	buddyinfo write 0xffffffffa05cf0f0
	modules read 0xffffffffa05ce8a0
	/ readdir 0xffffffffa05ce0e0
	/var/run/utmp read 0xffffffffa05ce4d0
	```
	- first hook allows rootkit to hide processes from running system; e.g. *ps*, *top*
	- second hook allows writing to `/proc/buddyinfo` to elevate privileges:
		``` sh
		$ id
		uid=1000(x) gid=1000(x) groups=1000(x),24(cdrom),25(floppy),29(audio),
		30(dip),44(video),46(plugdev)
		$ echo $$
		9673
		$ echo "root $$" > /proc/buddyinfo
		-bash: echo: write error: Operation not permitted
		$ id
		uid=0(root) gid=0(root) groups=0(root)
		```
	- third hook allows rootkit to hide kernel modules; e.g. *lsmod*
	- fourth hook allows hiding files 
	- final hook allows hiding logged on users; e.g. *w*, *who*

## 36. hiding logged on users
- *w* or *who* commands read binary structure of `/var/run/utmp` and print information; hooking `/var/run/utmp` allows hiding users
- extracting `utmp` file from memory allows bypassing hook:
	``` sh
	# find address of file
	$ python vol.py -f avgcoder.mem --profile=LinuxCentOS63x64 linux_find_file -F "/var/run/utmp"
	Volatility Foundation Volatility Framework 2.4
	Inode Number Inode
	---------------- ------------------
	130564 0x88007a85acc0
	# extract file
	$ python vol.py -f avgcoder.mem --profile=LinuxCentOS63x64 linux_find_file -i 0x88007a85acc0 -O utmp
	# view file
	$ who utmp
	centoslive tty1 2013-08-09 16:26 (:0)
	centoslive pts/0 2013-08-09 16:28 (:0.0)
	```

## 37. *linux_check_inline_kernel* plugin
- leverages other plugins to locate functions and function pointer that may be targeted by malware
- checks functions:
	- `dev_get_flags`: used to hide promiscuous mode setting of network interfaces
	- `ia32_sysenter_target` and `ia32_syscall`: handler functions of system call interrupts
	- `vfs_readdir` and `tcp_sendmsg`: can be hooked by IFRAME injecting rootkit
- types on inline hooks detected:
	- `JMP`
	- `CALL`
	- `RET`
- example output:
	``` data
	$ python vol.py --profile=LinuxDebian-3_2x64 -f susnf.lime linux_check_inline_kernel
	proc_root readdir JMP 0xffffffffa039e06f
	/proc readdir JMP 0xffffffffa039e06f
	/ readdir JMP 0xffffffffa039e06f
	/ readdir JMP 0xffffffffa039e0be
	/home readdir JMP 0xffffffffa039e0be
	/home/x readdir JMP 0xffffffffa039e0be
	/root readdir JMP 0xffffffffa039e0be
	<snip>
	/var readdir JMP 0xffffffffa039e0be
	/var/mail readdir JMP 0xffffffffa039e0be
	/var/www readdir JMP 0xffffffffa039e0be
	/var/cache readdir JMP 0xffffffffa039e0be
	/var/cache/samba readdir JMP 0xffffffffa039e0be
	/var/cache/samba/printing readdir JMP 0xffffffffa039e0be
	/var/spool readdir JMP 0xffffffffa039e0be
	/var/spool/exim4 readdir JMP 0xffffffffa039e0be
	/var/spool/exim4/db readdir JMP 0xffffffffa039e0be
	```
	- hooking `readdir` of root subsequently hooks all files and directories inside
