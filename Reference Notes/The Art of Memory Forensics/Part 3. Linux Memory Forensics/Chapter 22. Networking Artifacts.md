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
