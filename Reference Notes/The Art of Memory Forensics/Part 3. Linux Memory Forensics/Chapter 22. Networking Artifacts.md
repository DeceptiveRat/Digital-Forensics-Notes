## 1. network socket FD analysis objectives
- identify socket FDs
- understand socket operations

## 2. network socket FD DS
- `inet_sock`:
	``` python
	>>> dt("inet_sock")
	'inet_sock' (800 bytes)
	0x0 : sk ['sock']
	0x270 : pinet6 ['pointer', ['ipv6_pinfo']]
	0x278 : inet_dport ['unsigned short']
	0x27a : inet_num ['unsigned short']
	0x27c : inet_saddr ['unsigned int']
	0x280 : uc_ttl ['short']
	0x282 : cmsg_flags ['unsigned short']
	0x284 : inet_sport ['unsigned short']
	[snip]
	```
	- `sk` embedded sock structure
	- `pinet6`: source and destination of IPv6 connections
	- `inet_dport`: destination port
	- `inet_sport`: source port
	- `inet_saddr`: source IP
	- `inet_num`: protocol number

## 3. *linux_netstat* plugin
- leverages *linux_lsof* plugin to enumerate process FD
- filters socket FDs by checking `file_operation` pointer of `file` and `dentry_operation` pointer of `dentry`:
	``` python
	openfiles = linux_lsof.linux_lsof(self._config).calculate()

	fops_addr = self.addr_space.profile.get_symbol("socket_file_ops")
	dops_addr = self.addr_space.profile.get_symbol("sockfs_dentry_operations")
	for (task, filp, i) in openfiles:
		if filp.f_op == fops_addr or filp.dentry.d_op == dops_addr:
	[snip]
	```
	- compare all file operation pointers and dentry operation pointers with socket operations
- once FD is known to be a socket, the `inode` structure can be converted to an `inet_sock` structure:
	``` python
	iaddr = filp.dentry.d_inode
	skt = self.SOCKET_I(iaddr)
	inet_sock = obj.Object("inet_sock", offset = skt.sk, vm = self.addr_space)
	yield task, i, inet_sock
	```

## 4. network connections analysis objectives
- recover connection information
- detect malicious network connections

## 5. network connections DS
- `sock_common`:
	``` python
	>>> dt("sock_common")
	'sock_common' (80 bytes)
	0x0 : skc_daddr ['unsigned int']
	0x4 : skc_rcv_saddr ['unsigned int']
	0x8 : skc_hash ['unsigned int']
	0x8 : skc_u16hashes ['array', 2, ['unsigned short']]
	0xc : skc_family ['unsigned short']
	0xe : skc_state ['unsigned char']
	0xf : skc_reuse ['unsigned char']
	0x10 : skc_bound_dev_if ['int']
	0x18 : skc_bind_node ['hlist_node']
	0x18 : skc_portaddr_node ['hlist_nulls_node']
	0x28 : skc_prot ['pointer', ['proto']]
	0x30 : skc_net ['pointer', ['net']]
	0x38 : skc_dontcopy_begin ['array', 0, ['int']]
	0x38 : skc_node ['hlist_node']
	0x38 : skc_nulls_node ['hlist_nulls_node']
	0x48 : skc_tx_queue_mapping ['int']
	0x4c : skc_refcnt ['__unnamed_0x38e']
	0x50 : skc_dontcopy_end ['array', 0, ['int']]
	```
	- `skc_daddr`: destination address of connection
	- `skc_rcv_saddr`: source address of connection
	- `skc_family`: address family; `AF_INET` for TCP and UDP
	- `skc_state`: 
		- protocol state used to determine TCP connection state
		- connection states defined in `include/net/tcp_states.h` Linux source file
	
## 6. TCP and UDP connections
- `linux_netstat` plugin output:
	``` data
	$ python vol.py --profile=LinuxDebian-3_2x64 -f debian.lime linux_netstat
	Proto Source IP:Port Destination IP:Port State Process
	TCP 192.168.174.169:22 192.168.174.1:56705 ESTABLISHED sshd/2787
	TCP 0.0.0.0:22 0.0.0.0:0 LISTEN sshd/2437
	UDP 0.0.0.0:137 0.0.0.0:0 nmbd/2121
	[snip]
	```
	- `Proto`: populated from `inet_sock.sk.sk_protocol`
	- `Source IP`: populated from `skc_rcv_saddr` of `sock_common`
	- `Source Port`: populated from `sport` or `inet_sport` of `inet_sock` or `skc_port` of `sock_common`
	- `Destination IP`: populated from `skc_daddr` of `sock_common`
	- `Destination Port`: populated from `dport` or `inet_dport` of `inet_sock`
	- `State`: populated from `skc_state` of `sock_common`
	- `Process/PID`: gathered from process `task_struct`

## 7. queued network packets
- kernel queues packets that are sent or received until the receiving client or server can process them

## 8. queued network packets analysis objectives
- recover queued packets on a per-process basis
- combine recovered packets with network captures
- transfer attribution

## 9. queued network packets DS
- `sk_buff`:
	``` python
	>>> dt("sk_buff")
	'sk_buff' (240 bytes)
	0x0 : next ['pointer', ['sk_buff']]
	0x8 : prev ['pointer', ['sk_buff']]
	0x10 : tstamp ['ktime']
	0x18 : sk ['pointer', ['sock']]
	0x68 : len ['unsigned int']
	0x6c : data_len ['unsigned int']
	0xcc : tail ['unsigned int']
	0xd0 : end ['unsigned int']
	0xd8 : head ['pointer', ['unsigned char']]
	0xe0 : data ['pointer', ['unsigned char']]
	```
	- `next`: reference to next packet of connection
	- `len`: packet content length
	- `head`: reference to beginning of packet protocol headers
	- `data`: reference to packet protocol data

## 10. *linux_packet_queues* plugin
- leverages *linux_netstat* to retrieve `inet_sock` of each socket descriptor and reads `sk` to obtain `sock` of descriptor
- `sock` holds queues in `sk_receive_queue` and `sk_write_queue` members
- each queue is implemented as `sk_buff_head` structures, which hold a pointer to the head of the list of `sk_buff` structures
- example output:
	``` data
	$ python vol.py --profile=LinuxDebian-3_2x64 -f debian.lime linux_pkt_queues -D output
	Volatility Foundation Volatility Framework 2.4
	Wrote 308 bytes to receive.1851.5
	```
- output lists size of each packet recovered along with filename where data is written
- filename is created as `<receive/send>.<PID>.<FD number>`

## 11. network interface analysis objectives
- detect promiscuous mode interfaces
- identify sniffing applications

## 12. network interface DS
- `net_device`:
	``` python
	'net_device' (1856 bytes)
	0x0 : name ['String', {'length': 16}]
	0x10 : pm_qos_req ['pm_qos_request']
	0x40 : name_hlist ['hlist_node']
	0x50 : ifalias ['pointer', ['char']]
	0x58 : mem_end ['unsigned long']
	0x60 : mem_start ['unsigned long']
	0x68 : base_addr ['unsigned long']
	0x70 : irq ['unsigned int']
	0x78 : state ['unsigned long']
	0x80 : dev_list ['list_head']
	0x1b0 : flags ['unsigned int']
	0x1cc : perm_addr ['array', 32, ['unsigned char']]
	[snip]
	```
	- `dev_list`: pointer to list of network devices
	- `perm_addr`: MAC address of interface
	- `flags`: 
		- status information of device
		- promiscuous mode devices will have `IFF_PROMISC`(`0x100`) flag set

## 13. *linux_ifconfig* plugin
- enumerates all active network interfaces
- walks each network namespace and walks list of devices
- to determine all interface names and aliases of a device, `ip_ptr` member in `in_device` type is used
- list of devices is stored in `ifa_list` member
- name of each device or alias can be found in `ifa_lable` member
- IP address can be found in `ifa_address`
- example output:
	``` data
	$ python vol.py --profile=LinuxDebian-3_2x64 -f debian.lime linux_ifconfig
	Volatility Foundation Volatility Framework 2.4
	Interface IP Address MAC Address promiscuous Mode
	---------------- -------------------- ------------------ ---------------
	lo 127.0.0.1 00:00:00:00:00:00 False
	eth0 192.168.174.169 00:0c:29:e5:11:2e False
	```

## 14. network interface naming conventions
- common prefixes for network device names:
	- `lo`: local loopback
	- `eth`: ethernet
	- `wlan`: wireless
	- `usb`: USB network devices

## 15. interface aliases
- aliases can be used to assign multiple IP addresses to a device
- named with base name folowed by colon and index
- *linux_ifconfig* output:
	``` data
	$ python vol.py --profile=LinuxDebian-3_2x64 -f tcpdump.lime linux_ifconfig
	Volatility Foundation Volatility Framework 2.4
	Interface IP Address MAC Address Promiscuous Mode
	---------------- -------------------- ------------------ ---------------
	lo 127.0.0.1 00:00:00:00:00:00 False
	eth0 192.168.174.169 00:0c:29:e5:11:2e True
	eth0:0 192.168.174.200 00:0c:29:e5:11:2e True
	```

## 16. *linux_list_raw* plugin
- shows which programs are using raw sockets
- example output:
	``` data
	$ python vol.py --profile=LinuxDebian-3_2x64 -f tcpdump.lime linux_list_raw
	Volatility Foundation Volatility Framework 2.4
	Process PID File Descriptor Inode
	---------------- ------ --------------- ------------------
	tcpdump 3796 3 9209
	dhclient 1788 4 6532
	```

## 17. route cache
- removed after 3.6.x series kernel
- stored in `rt_hash_table` global variable:
	- element is `rt_hash_bucket` 
	- `rt_hash_mask` global variable determines size 

## 18. route cache analysis objectives
- map IP addresses to domain names

## 19. route cache DS
- `rtable`:
	``` python
	>>> dt("rtable")
	'rtable' (224 bytes)
	0x0 : dst ['dst_entry']
	0x98 : rt_key_dst ['unsigned int']
	0x9c : rt_key_src ['unsigned int']
	0xa0 : rt_genid ['int']
	0xa4 : rt_flags ['unsigned int']
	0xa8 : rt_type ['unsigned short']
	0xaa : rt_key_tos ['unsigned char']
	0xac : rt_dst ['unsigned int']
	0xb0 : rt_src ['unsigned int']
	0xb4 : rt_route_iif ['int']
	0xb8 : rt_iif ['int']
	0xbc : rt_oif ['int']
	0xc0 : rt_mark ['unsigned int']
	0xc4 : rt_gateway ['unsigned int']
	0xc8 : rt_spec_dst ['unsigned int']
	0xcc : rt_peer_genid ['unsigned int']
	0xd0 : peer ['pointer', ['inet_peer']]
	0xd8 : fi ['pointer', ['fib_info']]
	```
	- `dst`: used to determine which interface the route was active
	- `rt_gateway`: network gateway IP 
	- `rt_dst`: remote destination

## 20. *linux_route_cache* plugin
- finds routing cache entries by enumerating every element of array and attempting to walk collision chain in `chain` member
- example output:
	``` data
	$ python vol.py --profile=LinuxDebian-3_2x64 -f debian.lime linux_route_cache
	Volatility Foundation Volatility Framework 2.4
	Interface Destination Gateway
	---------------- -------------------- -------
	eth0 192.168.174.1 192.168.174.1
	lo 192.168.174.169 192.168.174.169
	eth0 192.168.174.254 192.168.174.254
	eth0 192.168.174.1 192.168.174.1
	eth0 192.168.174.1 192.168.174.1
	lo 192.168.174.255 192.168.174.255
	<snip>
	```

## 21. ARP cache analysis objectives
- detect lateral movement

## 22. ARP cache DS
- `neighbour`: 
	``` data
	'neighbour' (392 bytes)
	0x0 : next ['pointer', ['neighbour']]
	0x8 : tbl ['pointer', ['neigh_table']]
	0x10 : parms ['pointer', ['neigh_parms']]
	0x18 : confirmed ['unsigned long']
	0x20 : updated ['unsigned long']
	0x28 : lock ['__unnamed_0x2ff3']
	0x2c : refcnt ['__unnamed_0x38e']
	0x30 : arp_queue ['sk_buff_head']
	0x48 : timer ['timer_list']
	0x98 : used ['unsigned long']
	0xa0 : probes ['__unnamed_0x38e']
	0xa4 : flags ['unsigned char']
	0xa5 : nud_state ['unsigned char']
	0xa6 : type ['unsigned char']
	0xa7 : dead ['unsigned char']
	0xa8 : ha_lock ['__unnamed_0x3015']
	0xb0 : ha ['array', 32, ['unsigned char']]
	0xd0 : hh ['hh_cache']
	0x160 : output ['pointer', ['void']]
	0x168 : ops ['pointer', ['neigh_ops']]
	0x170 : rcu ['rcu_head']
	0x180 : dev ['pointer', ['net_device']]
	0x188 : primary_key ['array', 0, ['unsigned char']]
	```
	- `next`: next ARP entry in list
	- `tbl`: back-pointer to owning ARP table
	- `primary_key`: IP of machine contacted
	- `ha`: MAC of machine contacted
	- `dev`: `net_device` of interface assiociated with cache entry

## 23. *linux_arp* plugin
- `neigh_tables` global variable stores list of neighbor table:
	- array index stores pointer to `neighbour` structure
- enumerate all neighbors by walking `next` member
- example output:
	``` data
	$ python vol.py --profile=LinuxDebian-3_2x64 -f debian.lime linux_arp
	Volatility Foundation Volatility Framework 2.4
	[:: ] at 00:00:00:00:00:00 on lo
	[192.168.174.1 ] at 00:50:56:c0:00:08 on eth0
	[192.168.174.2 ] at 00:50:56:fa:ad:55 on eth0
	[192.168.174.254 ] at 00:50:56:e3:2e:81 on eth0
	```
