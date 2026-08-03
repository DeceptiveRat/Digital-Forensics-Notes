## 1. network artifacts
- 2 primary types: 
	- sockets
	- connections
- methods for creating sockets:
	- direct from user mode: `socket` function from Winsock2 API (`ws2_32.dll`)
	- indirect from user mode: functions in libraries such as WinINet (`wininet.dll`) that provide wrappers around Winsock2 functions
	- direct from kernel mode: use of *Transport Driver Interface(TDI)*, primary interface to transport stack

## 2. Windows Sockets API (Winsock)
- `_ADDRESS_OBJECT` represents socket objects and is allocated after call to `bind` or `connect`
- API calls to create TCP server:
	1. server and client call `socket`:
		- calling process opens handle to `\Device\Afd\Endpoint`
		- handle enables user mode process to communicate with `Afd.sys` (*Auxiliary Function Driver(AFD)* for Winsock2) in kernel mode
		- handle must remain open for duration of socket's life time
	2. server calls `bind`:
		- calling process opens handle to `\Device\Tcp`, `\Device\Udp`, or `\Device\Ip` depending on protocol
		- memory is allocated in kernel for `_ADDRESS_OBJECT` and members are filled according to parameters to `socket` and `bind`
		- `listen` does not create new artifacts
	3. client calls `connect`: 
		- same artifacts as `bind` + `_TCPT_OBJECT` (connection object)
		- for every connection established with a client (when `accept` returns), server process is also associated with `_TCPT_OBJECT` and a new set of handles
		- artifacts exist until applications call `closesocket`

![[tcp_server_API_artifacts.png]]
![[tcp_client_API_artifacts.png]]

## 3. network analysis objectives
- identify rogue listeners
- reveal suspicious remote connections
- locate systems with promiscuous network cards: may be machines used to sniff traffic or perform *Man-In-The-Middle(MITM)* attacks
- detect hidden ports on live systems
- reconstruct browser history

## 4. active sockets and connections
- OS maintains active sockets and connections by using chained overflow hash table; walk entries in hash table to find active sockets and connections
- `sockets` and `connections` plugins steps:
	1. locate `tcpip.sys` module in kernel memory
	2. locate non-exported global variable in data section, `_AddrObjTable` for sockets and `_TCBTable` for connections

![[locating_socket_connection_objects.png]]
- `sockets` plugin example output:
	``` sh
	$ python vol.py sockets -f zeus.bin --profile=WinXPSP3x86
	Volatility Foundation Volatility Framework 2.4
	PID Port Proto Protocol Address Create Time
	-------- ------ ------ --------------- --------------- -----------
	1156 1900 17 UDP 192.168.128.128 2008-12-11 20:51:52
	892 19705 6 TCP 0.0.0.0 2009-02-12 03:38:14
	740 500 17 UDP 0.0.0.0 2008-09-18 05:33:19
	4 139 6 TCP 192.168.128.128 2008-12-11 20:51:51
	4 445 6 TCP 0.0.0.0 2008-09-18 05:32:51
	972 135 6 TCP 0.0.0.0 2008-09-18 05:32:59
	4 137 17 UDP 192.168.128.128 2008-12-11 20:51:51
	1320 1029 6 TCP 127.0.0.1 2008-09-18 05:33:29
	1064 123 17 UDP 127.0.0.1 2008-12-11 20:51:52
	740 0 255 Reserved 0.0.0.0 2008-09-18 05:33:19
	1112 1025 17 UDP 0.0.0.0 2008-09-18 05:33:28
	1112 1033 17 UDP 0.0.0.0 2008-09-18 05:42:19
	4 138 17 UDP 192.168.128.128 2008-12-11 20:51:51
	892 35335 6 TCP 0.0.0.0 2009-02-12 03:38:14
	1112 1115 17 UDP 0.0.0.0 2008-12-11 18:54:24
	1064 123 17 UDP 192.168.128.128 2008-12-11 20:51:52
	892 1277 6 TCP 0.0.0.0 2009-02-12 03:38:15
	1156 1900 17 UDP 127.0.0.1 2008-12-11 20:51:52
	740 4500 17 UDP 0.0.0.0 2008-09-18 05:33:19
	1064 1276 17 UDP 127.0.0.1 2009-02-12 03:38:12
	1064 1275 17 UDP 192.168.128.128 2009-02-12 03:38:12
	4 445 17 UDP 0.0.0.0 2008-09-18 05:32:51
	```
	- ports 1275, 1276, and 1277 are assigned in quick succession; likely ephemeral client sockets making ports 19705, 35335 server sockets
- `connections` plugin example output:
	``` sh
	$ python vol.py -f zeus.vmem --profile=WinXPSP3x86 connections
	Volatility Foundation Volatility Framework 2.4
	Offset(V) Local Address Remote Address Pid
	---------- ------------------------- ------------------------- ---
	0x81eba510 192.168.128.128:1277 XX.XX.117.254:80 892
	```

## 5. attributing connections to code
1. determine if main process executable is malicious:
	1. dump process
	2. reverse engineer
	3. examine *Import Address Table(IAT)* and follow cross references to `socket`, `connect`, and `send` APIs
2. if main process is not malicious, scan process memory for injected code blocks:
	- use `malfind` plugin or `yarascan` plugin with criteria related to connection
	- scan finds associated IP address in memory:
		``` sh
		$ python vol.py -f memory.raw yarascan --profile=WinXPSP3x86
		-p 3060 -W -Y "XX.XXX.5.140"
		Volatility Foundation Volatility Framework 2.4
		Rule: r1
		Owner: Process ab.exe Pid 3060
		0x5500e9ae XX 00 XX 00 2e 00 XX 00 XX 00 XX 00 2e 00 35 00 X.X...X.X.X...5.
		0x5500e9be 2e 00 31 00 34 00 30 00 3a 00 38 00 30 00 38 00 ..1.4.0.:.8.0.8.
		0x5500e9ce 30 00 2f 00 7a 00 62 00 2f 00 76 00 5f 00 30 00 0./.z.b./.v._.0.
		```
		- perform reverse lookup with `dlllist`:
			``` sh
			$ python vol.py -f memory.raw yarascan --profile=WinXPSP3x86 -p 3060 dlllist
			Volatility Foundation Volatility Framework 2.4
			************************************************************************
			ab.exe pid: 3060
			Command line : "C:\WINDOWS\system32\ab.exe"
			Service Pack 2
			Base Size LoadCount Path
			---------- ---------- ---------- ----
			0x00400000 0x21000 0xffff C:\WINDOWS\system32\ab.exe
			0x7c900000 0xb0000 0xffff C:\WINDOWS\system32\ntdll.dll
			0x7c800000 0xf4000 0xffff C:\WINDOWS\system32\kernel32.dll
			0x77d40000 0x90000 0xffff C:\WINDOWS\system32\USER32.dll
			0x77f10000 0x46000 0xffff C:\WINDOWS\system32\GDI32.dll
			0x77c10000 0x58000 0xffff C:\WINDOWS\system32\msvcrt.dll
			0x55000000 0x33000 0x1 C:\WINDOWS\system32\ab.dll
			0x77dd0000 0x9b000 0x14 C:\WINDOWS\system32\advapi32.dll
			0x77e70000 0x91000 0xb C:\WINDOWS\system32\RPCRT4.dll
			0x71ab0000 0x17000 0x1 C:\WINDOWS\system32\WS2_32.dll
			```
			- address range is inside DLL named `ab.dll`
	- scan finds URL but not within dll:
		``` sh
		$ python vol.py -f jack.mem --profile=Win7SP0x86 yarascan -p 3030 --wide -Y "http"
		Volatility Foundation Volatility Framework 2.4
		Rule: r1
		Owner: Process jack.exe Pid 3030
		0x75d82438 68 00 74 00 74 00 70 00 3a 00 2f 00 2f 00 XX 00 h.t.t.p.:././.X.
		0x75d82448 XX 00 XX 00 2e 00 31 00 33 00 34 00 2e 00 31 00 X.X...1.3.4...1.
		0x75d82458 37 00 36 00 2e 00 31 00 32 00 36 00 2f 00 65 00 7.6...1.2.6./.e.
		0x75d82468 78 00 69 00 73 00 74 00 73 00 2f 00 50 00 61 00 x.i.s.t.s./.P.a.
		[snip]
		```
		- search for pointers to referenced address and disassemble:
			``` sh
			$ python vol.py -f jack.mem --profile=Win7SP0x86 yarascan -p 3030 -Y "{38 24 d8 75}"
			Volatility Foundation Volatility Framework 2.4
			Rule: r1
			Owner: Process jack.exe Pid 3030
			0x75d47500 38 24 d8 75 ff b5 64 ff ff ff e8 da 37 00 00 85 8$.u..d.....7...
			0x75d47510 c0 75 27 8b 03 53 ff b5 68 ff ff ff 89 85 60 ff .u'..S..h.....`.
			0x75d47520 ff ff 68 38 24 d8 75 e8 31 c7 fe ff 89 85 6c ff ..h8$.u.1.....l.
			0x75d47530 ff ff 85 c0 0f 8c 4a e2 01 00 6a 00 57 ff d6 83 ......J...j.W...
			$ python vol.py -f jack.mem --profile=Win7SP0x86 volshell -p 3030
			Volatility Foundation Volatility Framework 2.4
			Current context: process jack.exe, pid=3030, ppid=2340 DTB=0x1f441380
			Welcome to volshell!
			To get help, type 'hh()'
			>>> dis(0x75d474f0)
			0x75d474f0 39058422d875 CMP [0x75d82284], EAX
			0x75d474f6 7542 JNZ 0x75d4753a
			0x75d474f8 a900100000 TEST EAX, 0x1000
			0x75d474fd 753b JNZ 0x75d4753a
			0x75d474ff 683824d875 PUSH DWORD 0x75d82438
			0x75d47504 ffb564ffffff PUSH DWORD [EBP-0x9c]
			0x75d4750a e8da370000 CALL 0x75d4ace9
			0x75d4750f 85c0 TEST EAX, EAX
			```
			- URL pointer is being passed to `0x75d4ace9`; can disassemble function for further information
- data might not always be strings; integers of IP addresses in network-byte order should be searched for as well:
	``` python
	$ python
	>>> import socket
	>>> import struct
	>>> struct.unpack(">I", socket.inet_aton("12.34.56.78"))[0]
	203569230
	```

## 6. hidden connections
- common techniques to hide connections:
	- API hooking:
		- APIs such as `DeviceIoControl`, `ZwDeviceIoControlFile`, `GetTcpTable`, and `GetExtendedTcpTable`
		- `apihooks` plugin can detect hooks
		- `netscan` plugin can detect hidden connections
	- hook `IRP_MJ_DEVICE_CONTROL` function of `\Device\Tcp` to filter attempts to gather information using `IOCTL_TCP_QUERY_INFORMATION_EX`:
		- `driverirp` plugin to detect kernel driver that does the hooking
		- `netscan` can still detect connections
	- NDIS driver which operates at a lower level thatn Winsock2 bypasses creation of common artifacts:
		- find loaded driver by scanning driver objects or hidden kernel threads
		- carve IP packets or Ethernet frames from memory dump


## 7. packets in memory
- IP packets and Ethernet frames must be structured; easy to identify
- DNS operation is quick so it's difficult to capture memory while it is active; finding packets in memory is valuable

## 8. DKOM attacks
- not much of a threat against socket/connection objects because overwriting them causes communication to fail

## 9. raw sockets
- enable programs to access underlying transport layer data, allowing system to forge or spoof packets
- raw sockets in promiscuous mode can be used to sniff packets
- processes that open raw sockets will have a socket bound to port 0 of protocol 0 and an open handle to `\Device\RawIp\0`

## 10. `netscan`
- uses pool-scanning approach to lcoate `_TCP_ENDPOINT`, `_TCP_LISTENER`, and `_UDP_ENDPOINT` in memory
	``` sh
	$ python vol.py -f win764bit.raw --profile=Win7SP0x64 netscan
	Volatility Foundation Volatility Framework 2.4
	Proto Local Address Foreign Address State Pid Owner
	----- -------------- ---------------- ----------- ---- ------------
	[snip]
	TCPv4 -:0 232.9.125.0:0 CLOSED 1 ?C?
	TCPv4 -:49227 184.26.31.55:80 CLOSED 2820 iexplore.exe
	TCPv4 -:49359 93.184.220.20:80 CLOSED 2820 iexplore.exe
	TCPv4 10.0.2.15:49363 173.194.35.38:80 ESTABLISHED 2820 iexplore.exe
	TCPv4 -:49341 82.165.218.111:80 CLOSED 2820 iexplore.exe
	```
	- "-" indicates information could not be accessed in memory dump; pointer chain can easily be broken if pages are swapped to disk

## 11. partition tables
- peformance in the TCP/IP stack is enhanced by splitting work between multiple processors
- global variable `PartitionTable` in `tcpip.sys` stores a pointer to `_PARTITION_TABLE`, an array of `_PARTITION`s
- during startup for `tcpip.sys`, `TcpStartPartitionModule` function allocates memory for partition structures and initializes them
- partition table usage:
	![[partition_table.png]]
	- `_PARTITION` contains 3 `_RTL_DYNAMIC_HASH_TABLE` structures; one for each state: established, SYN sent, and time wait (about to close)
	- dynamic hash tables point to doubly linked list of connection structures; e.g. `_TCP_ENDPOINT`

## 12. port pools
- big page tracker tables tell you the exact addresses of pools with `InPP` tag storing `_INET_PORT_POOL` structures
- port pools contain a 65535-bit bitmap, one for each port, and an equal number of pointers to `_PORT_ASSIGNMENT` structures
- index of a bit in the bitmap can be used to compute the address of the corresponding `_TCP_LISTENER`, `_TCP_ENDPOINT`, or `_UDP_ENDPOINT` structure
- port pool:
	![[port_pool.png]]
	- `_PORT_ASSIGNMENT` structures don't directly point to the connection structures; the value is derived from a base address + index of bit in bitmap

## 13. internet history
- *Internet Explorer*'s history file is loaded by all processes that use the WinINet API to access HTTP, HTTPS, or FTP sites
- viewing history in memory:
	``` sh
	$ python vol.py -f win7_x64.dmp --profile=Win7SP0x64 yarascan -Y "/(URL |REDR|LEAK)/" -p 2580,3004
	Volatility Foundation Volatility Framework 2.4
	Rule: r1
	Owner: Process iexplore.exe Pid 3004
	0x026f1600 55 52 4c 20 03 00 00 00 00 99 35 2c 82 43 ca 01 URL.......5,.C..
	0x026f1610 a0 ec 34 cb 34 02 cc 01 00 00 00 00 00 00 00 00 ..4.4...........
	0x026f1620 76 01 00 00 00 00 00 00 00 00 00 00 00 00 00 00 v...............
	0x026f1630 60 00 00 00 68 00 00 00 03 01 10 10 c4 00 00 00 `...h...........
	0x026f1640 41 00 00 00 dc 00 00 00 7d 00 00 00 00 00 00 00 A.......}.......
	0x026f1650 98 3e a3 20 01 00 00 00 00 00 00 00 98 3e a3 20 .>...........>..
	0x026f1660 00 00 00 00 ef be ad de 68 74 74 70 3a 2f 2f 6d ........http://m
	0x026f1670 73 6e 62 63 6d 65 64 69 61 2e 6d 73 6e 2e 63 6f snbcmedia.msn.co
	[snip]
	Rule: r1
	Owner: Process iexplore.exe Pid 3004
	0x026c0b00 4c 45 41 4b 06 00 00 00 00 a6 3b 01 cc 97 cb 01 LEAK......;.....
	0x026c0b10 c0 71 20 14 33 02 cc 01 98 3e 39 1f 00 00 00 00 .q..3....>9.....
	0x026c0b20 f8 cf 00 00 00 00 00 00 00 00 00 00 80 2a 02 00 .............*..
	0x026c0b30 60 00 00 00 68 00 00 00 03 00 10 10 40 02 00 00 `...h.......@...
	0x026c0b40 41 00 00 00 60 02 00 00 9e 00 00 00 00 00 00 00 A...`...........
	0x026c0b50 98 3e 99 1e 01 00 00 00 00 00 00 00 98 3e 99 1e .>...........>..
	0x026c0b60 00 00 00 00 ef be ad de 68 74 74 70 3a 2f 2f 75 ........http://u
	0x026c0b70 73 65 2e 74 79 70 65 6b 69 74 2e 63 6f 6d 2f 6b se.typekit.com/k
	[snip]
	Rule: r1
	Owner: Process iexplore.exe Pid 3004
	0x026e2680 52 45 44 52 02 00 00 00 78 1b 02 00 40 af d3 51 REDR....x...@..Q
	0x026e2690 68 74 74 70 3a 2f 2f 62 73 2e 73 65 72 76 69 6e http://bs.servin
	0x026e26a0 67 2d 73 79 73 2e 63 6f 6d 2f 42 75 72 73 74 69 g-sys.com/Bursti
	0x026e26b0 6e 67 50 69 70 65 2f 61 64 53 65 72 76 65 72 2e ngPipe/adServer.
	```
	- at offset `0x34` from start of URL or LEAK, you can find a 4-byte number specifying offset from start of string to visited location
	- location can be found at offset `0x10` for redirected URLs
- brute-force searching for domains:
	``` sh
	$ python vol.py -f win7_x64.dmp --profile=Win7SP0x64 yarascan -p 3004 -Y "/[a-zA-Z0-9\-\.]+\.(com|org|net|mil|edu|biz|name|info)/"
	Volatility Foundation Volatility Framework 2.4
	Rule: r1
	Owner: Process iexplore.exe Pid 3004
	0x003e90dd 77 77 77 2e 72 65 75 74 65 72 73 2e 63 6f 6d 2f www.reuters.com/
	0x003e90ed 61 72 74 69 63 6c 65 2f 32 30 31 31 2f 30 34 2f article/2011/04/
	0x003e90fd 32 34 2f 75 73 2d 73 79 72 69 61 2d 70 72 6f 74 24/us-syria-prot
	0x003e910d 65 73 74 73 2d 69 64 55 53 54 52 45 37 33 4c 31 ests-idUSTRE73L1
	0x003e911d 53 4a 32 30 31 31 30 34 32 34 22 20 69 64 3d 22 SJ20110424".id="
	0x003e912d 4d 41 41 34 41 45 67 42 55 41 4a 67 43 47 6f 43 MAA4AEgBUAJgCGoC
	0x003e913d 64 58 4d 22 3e 3c 73 70 61 6e 20 63 6c 61 73 73 dXM"><span.class
	0x003e914d 3d 22 74 69 74 6c 65 74 65 78 74 22 3e 52 65 75 ="titletext">Reu
	Rule: r1
	Owner: Process iexplore.exe Pid 3004
	0x00490fa0 77 77 77 2e 62 69 6e 67 2e 63 6f 6d 2f 73 65 61 www.bing.com/sea
	0x00490fb0 72 63 68 3f 71 3d 6c 65 61 72 6e 2b 74 6f 2b 70 rch?q=learn+to+p
	0x00490fc0 6c 61 79 2b 68 61 72 6d 2b 31 11 3a 87 26 00 88 lay+harm+1.:.&..
	0x00490fd0 00 00 00 00 00 00 00 00 80 00 00 00 00 00 00 00 ................
	0x00490fe0 d8 50 0b 09 00 00 00 00 00 00 00 00 00 00 00 00 .P..............
	0x00490ff0 00 00 00 00 3e 46 69 6e 5d c7 37 4e 20 00 00 00 ....>Fin].7N....
	0x00491000 40 10 49 00 00 00 00 00 00 00 00 00 00 00 00 00 @.I.............
	0x00491010 01 00 00 00 63 61 3c 2f 63 00 6f 00 6e 00 74 00 ....ca</c.o.n.t.
	```

## 14. DNS
- DNS cache is stored in address space of `svchost.exe` that runs DNS resolver service, specifically heaps
- *hosts* file should be examined for sabotage:
	``` sh
	$ python vol.py -f infectedhosts.dmp filescan | grep -i hosts
	Volatility Foundation Volatility Framework 2.4
	0x0000000002192f90 1 0 R--rw- \Device\HarddiskVolume1\WINDOWS\system32\
	drivers\etc\hosts
	$ python vol.py -f infectedhosts.dmp dumpfiles -Q 0x2192f90 -D OUTDIR --name
	Volatility Foundation Volatility Framework 2.4
	DataSectionObject 0x02192f90 None
	\Device\HarddiskVolume1\WINDOWS\system32\drivers\etc\hosts
	$ strings OUTDIR/file.None.0x8211f1f8.hosts.dat
	# Copyright (c) 1993-1999 Microsoft Corp.
	[snip]
	127.0.0.1 localhost
	127.0.0.1 avp.com
	127.0.0.1 ca.com
	127.0.0.1 customer.symantec.com
	127.0.0.1 dispatch.mcafee.com
	127.0.0.1 f-secure.com
	127.0.0.1 kaspersky.com
	127.0.0.1 liveupdate.symantec.com
	```
