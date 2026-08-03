## 1. mounted FS
- Linux maintains a list of mounted FS in kernel memory

## 2. mounted FS analysis objectives
- identify reconnaissance and snooping: attackers attempt to mount interesting network shares
- detect data exfiltration: USB sticks may be used to exfiltrate data

## 3. mounted FS DS
- hash table that contains all mounted FS system structures is pointed to by kernel symbol `mount_hashtable`
- elements in hash table are `vfsmount` in newer kernels:
	``` python
	>>> dt("vfsmount")
	'vfsmount' (240 bytes)
	0x0 : mnt_hash ['list_head']
	0x10 : mnt_parent ['pointer', ['vfsmount']]
	0x18 : mnt_mountpoint ['pointer', ['dentry']]
	0x20 : mnt_root ['pointer', ['dentry']]
	0x28 : mnt_sb ['pointer', ['super_block']]
	0x30 : mnt_pcp ['pointer', ['mnt_pcp']]
	0x38 : mnt_longterm ['__unnamed_0x38e']
	0x40 : mnt_mounts ['list_head']
	0x50 : mnt_child ['list_head']
	0x60 : mnt_flags ['int']
	0x64 : mnt_fsnotify_mask ['unsigned int']
	0x68 : mnt_fsnotify_marks ['hlist_head']
	0x70 : mnt_devname ['pointer', ['char']]
	0x78 : mnt_list ['list_head']
	0x88 : mnt_expire ['list_head']
	0x98 : mnt_share ['list_head']
	0xa8 : mnt_slave_list ['list_head']
	0xb8 : mnt_slave ['list_head']
	0xc8 : mnt_master ['pointer', ['vfsmount']]
	0xd0 : mnt_ns ['pointer', ['mnt_namespace']]
	0xd8 : mnt_id ['int']
	0xdc : mnt_group_id ['int']
	0xe0 : mnt_expiry_mark ['int']
	0xe4 : mnt_pinned ['int']
	0xe8 : mnt_ghosts ['int']
	```
	- `mnt_hash`: list of collisions for hash value in `mount_hashtable`
	- `mnt_parent`: reference to parent FS
	- `mnt_mountpoint`: reference to directory entry for mount point 
	- `mnt_root`: reference to directory entry for root
	- `mnt_devname`: name of mounted device
	- `mnt_sb`: 
		- pointer to `super_block` of mount point
		- used by *Volatility* to enumerate files and directories in FS

## 4. *linux_mount* plugin
- recovers each FS and lists the device, mount point, FS type, and mount options
- example output:
	``` data
	$ python vol.py --profile=LinuxDebian-3_2x64 -f debian-mount.lime linux_mount
	Volatility Foundation Volatility Framework 2.4
	Device Mount Point FS Type Mount Options
	sysfs /sys sysfs rw,relatime,nosuid,nodev,noexec
	tmpfs /run/lock tmpfs rw,relatime,nosuid,nodev,noexec
	proc /proc proc rw,relatime,nosuid,nodev,noexec
	tmpfs /run/shm tmpfs rw,relatime,nosuid,nodev,noexec
	devpts /dev/pts devpts rw,relatime,nosuid,noexec
	tmpfs /run tmpfs rw,relatime,nosuid,noexec
	udev /dev devtmpfs rw,relatime
	rpc_pipefs /var/lib/nfs/rpc_pipefs rpc_pipefs rw,relatime
	/dev/disk/by-uuid/b2385703-e550-4736-a19f-e8490e5a570e /
	ext4 rw,relatime
	//192.168.174.1/user/adam/documents /mnt/share
	cifs ro,relatime
	```
	- target machine was connected to remote system 
- data recovery methods:
	- `Device`: from `mnt_devname` member of `vfsmount` 
	- `Mount Point`: directory in which FS is mounted
	- `FS Type`: from `s_type` member of `super_block`
	- `Mount Options`: `s_flags` member of `super_block`

## 5. common FS types
- ext3/4: frequently used on-disk FS
- sysfs: 
	- pseudo FS mounted at `/sys`
	- exports information about hardware devices connected
- proc:
	- pseudo FS mounted at `/proc`
	- exports list of running processes, active network connections, loaded kernel modules, and settings related to the memory manager
	- allows reading of network settings such as MTU for each interface and whether to forward packets
- tmpfs: memory-only FS used for `/tmp` and `/dev/shm`
- devpts: 
	- pseudo FS that is mount point for pseudo terminal
	- user that logs in to terminal from remote locatio receives psuedo terminal
- devtmpfs:
	- pseudo FS that manages files and directories under `/dev`
	- stores device nodes for physical devices as well as those that software creates
- cifs:
	- used to mount remote SMB shares
	- seen when Linux wants to mount remote Windows file shares or when NAS devices are in use on network

## 6. mount options
- `atime/noatime`: controls whether access times for files are updated
- `diratime/nodiratime`: controls whether directory access times are updated
- `relatime/norelatime`: updates access times only if access time is older than last modified time
- `dev/nodev`: allows or disallows devices to load under a mountpoint
- `exec/noexec`: 
	- controls whether file are executed inside mountpoint
	- used as security mechanism to prevent attackers from downloading and executing binaries in world-writable directories such as `/tmp`
- `suid/nosuid`:
	- controls whether *Set User ID(SUID)* binaries are allowed within FS
	- security mechanism that prevents users from executing binaries as other users
- `rw/ro`: marks FS as read-write or read-only
- can compare current options with default options in `/etc/fstab` to find attacker modifications

## 7. temporary FS
- only exists in memory
- `/tmp` is used as storage for installations, archive extraction, etc
- `/dev/shm` is for shared memory of applications
- often used by attackers to reduce foot prints
- can be used to provide a mutable FS for live CDs:
	![[live_CD_interactions.png]]
	- writes are stored in tmpfs
	- FS driver handles deletes so processes cannot access them even if they are on the CD

## 8. *Virtual File System(VFS)*
- used to generically handle the wide range of FS
- all FS operations go through same interface

## 9. VFS analysis objectives
- determine files being used
- identify unknown or unexpected files

## 10. VFS DS
- `dentry` holds info on directory entry:
	``` python
	>>> dt("dentry")
	'dentry' (192 bytes)
	0x0 : d_flags ['unsigned int']
	0x4 : d_seq ['seqcount']
	0x8 : d_hash ['hlist_bl_node']
	0x18 : d_parent ['pointer', ['dentry']]
	0x20 : d_name ['qstr']
	0x30 : d_inode ['pointer', ['inode']]
	0x38 : d_iname ['array', 32, ['unsigned char']]
	0x58 : d_count ['unsigned int']
	0x5c : d_lock ['spinlock']
	0x60 : d_op ['pointer', ['dentry_operations']]
	0x68 : d_sb ['pointer', ['super_block']]
	0x70 : d_time ['unsigned long']
	0x78 : d_fsdata ['pointer', ['void']]
	0x80 : d_lru ['list_head']
	0x90 : d_u ['list_head', {}]
	0xa0 : d_subdirs ['list_head']
	0xb0 : d_alias ['list_head']
	```
	- `d_parent`: 
		- pointer to parent directory entry
		- can be used to build the directory structure of FS
	- `d_name`: `qstr` (quick string) structure that holds name of file or directory
	- `d_inode`: 
		- pointer to corresponding `inode` structure
		- used to extract metadata and content associated with file
	- `d_sb`: pointer to `super_block` containing `dentry`
	- `d_u`: union whose `d_child` member is pointer into list of files and directories inside directory
	- `d_subdirs`: 
		- list of subdirectories inside directory
		- combined with `d_u` to produce accurate directory listing

## 11. *linux_enumerate_files* plugin
- uses `s_root` member of `super_block` within `mnt_sb` of `vfsmount` because it stores a reference to the root `dentry` of the FS
- example output:
	``` data
	$ python vol.py --profile=LinuxDebian-3_2x64 -f backdooredrm.lime linux_enumerate_files
	[snip]
	/tmp/rm
	<snip>
	/etc/mtab
	/etc/resolv.conf
	/etc/gcrypt
	/etc/hosts
	/etc/host.conf
	/etc/suid-debug
	/etc/ssh
	/etc/ssh/moduli
	/etc/cron.hourly
	/etc/default
	/etc/default/locale
	/etc/environment
	/etc/security
	/etc/security/limits.d
	/etc/security/limits.conf
	/etc/security/pam_env.conf
	/etc/shadow
	/etc/pam.d
	/etc/pam.d/common-session
	/etc/pam.d/common-password
	/etc/pam.d/other
	/etc/pam.d/common-session-noninteractive
	[snip]
	```
	- shows every file found in memory
	- `/tmp/rm` attacker planted can be seen
	- `/etc/shadow` stores encrypted passwords of users
	- `/etc/hosts` stores hardcoded hostname-to-IP mappings
	- `/etc/resolv.conf` stores DNS settings

## 12. file metadata analysis objectives
- create activity timelines
- distinguish between different users' files

## 13. file metadata DS
- `inode` holds metadata for a file:
	``` python
	>>> dt("inode")
	'inode' (552 bytes)
	0x0 : i_mode ['unsigned short']
	0x2 : i_opflags ['unsigned short']
	0x4 : i_uid ['unsigned int']
	0x8 : i_gid ['unsigned int']
	0xc : i_flags ['unsigned int']
	0x10 : i_acl ['pointer', ['posix_acl']]
	0x18 : i_default_acl ['pointer', ['posix_acl']]
	0x20 : i_op ['pointer', ['inode_operations']]
	0x28 : i_sb ['pointer', ['super_block']]
	0x30 : i_mapping ['pointer', ['address_space']]
	0x38 : i_security ['pointer', ['void']]
	0x40 : i_ino ['unsigned long']
	<snip>
	0x50 : i_atime ['timespec']
	0x60 : i_mtime ['timespec']
	0x70 : i_ctime ['timespec']
	0x80 : i_lock ['spinlock']
	<snip>
	0x90 : i_size ['long long']
	<snip>
	0x108 : i_dentry ['list_head']
	<snip>
	0x130 : i_fop ['pointer', ['file_operations']]
	<snip>
	```
	- `i_mode`: encodes file type as well as permissions
	- `i_uid/i_gid`: 
		- UID and GID of file owner
		- used with `/etc/passwd` and `/etc/group` files to translate to user and group names
	- `i_op/i_fop`: 
		- inode/file operation pointers
		- controls all interactions with inode and FS drivers
		- play a role in reporting directory listings and file contents to userland processes
		- often hijacked by malware
	- `i_mapping`: pointer to `address_space_operations` structure for file data
	- `i_ino`: 
		- inode number uniquely identifying each file and directory in FS
		- appears in the output of *stat* command
	- `i_mtime/i_atime/i_ctime`: MAC time of file
	- `i_size`: 
		- size of file
		- used to recover contents

## 14. *linux_recover_filesystem* plugin
- using metadata from `inode`, dumps FS artifacts to disk
- access and modify times of dumped files are changed to match those in memory sample; create time is not modified
- dumped file permissions are restored via `os.chown` function in Python
- inode number cannot be replicated
- example output:
	``` data
	$ sudo python vol.py --profile=Linuxdfrws2008x86 -f challenge.mem linux_recover_filesystem -D fsout
	Volatility Foundation Volatility Framework 2.4
	Recovered 12463 files
	$ cat fsout/home/stevev/.bash_history
	uname -a
	who
	ll -h
	mkdir temp
	ll -h
	chmod o-xrw temp/
	$ ls -l fsout/mnt/hgfs/Admin_share/
	total 784
	-rw-rw-r-- 1 500 500 141824 Dec 8 2007 acct_prem.xls
	-rw-rw-r-- 1 500 500 100864 Dec 8 2007 domain.xls
	-rwxr-xr-x 1 500 500 2395 Aug 5 2000 ftp.pcap
	-rwxr-xr-x 1 500 500 460288 May 16 2007 intranet.vsd
	-rw-rw-r-- 1 500 500 10376 Nov 26 2007 libfindrtp-0.4b.tar.gz
	-rw-rw-r-- 1 500 500 354 Dec 8 2007 negotiation notes.txt
	-rw-rw-r-- 1 500 500 52493 Nov 26 2007 rtp-stego-code.tgz
	-rwxrw-r-- 1 500 500 3209 Dec 16 2007 xfer.pl
	```
- run as root to fully replicated metadata
- FS recovered may be attacker controlled so caution is advised

## 15. *linux_dentry_cache* plugin
- can build timelines from FS artifacts on memory
- finds instances of `dentry` within `kmem_cache` of SLAB-based systems and creates a body file from the structures
- example output:
	``` data
	$ python vol.py --profile=Linuxdfrws-profilex86 -f challenge.mem
	linux_dentry_cache > body.txt
	Volatility Foundation Volatility Framework 2.4
	$ grep Admin body.txt
	0|Admin_share/xfer.pl|262633|0|500|500|3209|1197862134|1197858797|1197861669|0
	0|Admin_share/intranet.vsd|262647|0|500|500|460288|1197861668|1179352760|1197186983|0
	0|Admin_share/acct_prem.xls|262646|0|500|500|141824|1197861980|1197119951|1197119951|0
	0|Admin_share/domain.xls|262645|0|500|500|100864|1197861980|1197119385|1197119385|0
	```
	- create timeline with *mactime* from TSK:
		``` data
		$ mactime -b body.txt -d
		-g fsout/etc/group -p fsout/etc/passwd | grep Admin_share
		Wed Dec 31 1969 18:00:00,4096,...b,0,user,user,262632,"Admin_share"
		Wed Dec 31 1969 18:00:00,3209,...b,0,user,user,262633,"Admin_share/xfer.pl"
		Wed Dec 31 1969 18:00:00,52493,...b,0,user,user,262634,"Admin_share/rtp-stego-code.tgz"
		Wed Dec 31 1969 18:00:00,10376,...b,0,user,user,262635,"Admin_share/libfindrtp-0.4b.tar.gz"
		Wed Dec 31 1969 18:00:00,354,...b,0,user,user,262638,"Admin_share/negotiation notes.txt"
		Wed Dec 31 1969 18:00:00,2395,...b,0,user,user,262640,"Admin_share/ftp.pcap"
		Wed Dec 31 1969 18:00:00,100864,...b,0,user,user,262645,"Admin_share/domain.xls"
		Wed Dec 31 1969 18:00:00,141824,...b,0,user,user,262646,"Admin_share/acct_prem.xls"
		[snip]
		```

## 16. page cache
- use by Linux to save information read from disk
- read-ahead cache that loads subsequent portions is implemented as well

## 17. page cache analysis objectives
- recover malware and attacker files from memory
- determine most recently used files on system

## 18. page cache DS
- `address_space` holds information on file mappings within page cache:
	``` python
	>>> dt("address_space")
	'address_space' (168 bytes)
	0x0 : host ['pointer', ['inode']]
	0x8 : page_tree ['radix_tree_root']
	0x18 : tree_lock ['spinlock']
	0x1c : i_mmap_writable ['unsigned int']
	0x20 : i_mmap ['prio_tree_root']
	0x30 : i_mmap_nonlinear ['list_head']
	0x40 : i_mmap_mutex ['mutex']
	0x60 : nrpages ['unsigned long']
	0x68 : writeback_index ['unsigned long']
	0x70 : a_ops ['pointer', ['address_space_operations']]
	0x78 : flags ['unsigned long']
	0x80 : backing_dev_info ['pointer', ['backing_dev_info']]
	0x88 : private_lock ['spinlock']
	0x90 : private_list ['list_head']
	0xa0 : assoc_mapping ['pointer', ['address_space']
	```
	- `page_tree`: 
		- root of radix tree that holds particular file's pages
		- enumerating in proper order reveals all file contents
	- `a_ops`: 
		- pointer to `address_space_operations` structure
		- defines operations on address space; e.g. reading, writing, freeing
	- `backing_dev_info`: pointer to device that holds file

## 19. file extraction algorithm
1. calculate index of each page in tree:
	- index is page number of page of interest, calculated by dividing file offset by page size
	- if index is 0, use constant offset to find `struct page`
	- if index is greater than 0, find corresponding note of tree
2. use `struct page` address in kernel VA space to find physical address of page:
	- index into `mem_map` array

## 20. *linux_find_file* plugin
- operates in 2 modes:
	- use `-F/--find` to determine whether a file is within the page cache
	- use `-i/--inode` and `-O/--outfile` to extract file
- example output:
	``` data
	$ python vol.py --profile=LinuxDebian-3_2x64 -f hiddenargs.lime linux_find_file -F /tmp/backdoor
	Volatility Foundation Volatility Framework 2.4
	Inode Number Inode
	---------------- ------------------
	1059161 0xffff88000b96f4d8
	$ python vol.py --profile=LinuxDebian-3_2x64 -f hiddenargs.lime linux_find_file -i 0xffff88000b96f4d8 -O backdoor.dump
	Volatility Foundation Volatility Framework 2.4
	$ file backdoor.dump
	backdoor-dump: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked (uses shared libs), for GNU/Linux 2.6.26, BuildID[sha1]=0x36c8848c0aa617cbdc6db0a16c91b8bee69f63cb, not stripped
	```
