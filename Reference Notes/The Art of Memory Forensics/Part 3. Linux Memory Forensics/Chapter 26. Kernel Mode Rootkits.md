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
- 
