### windows.info 
``` sh
$ vol -f gotham.raw windows.info
Volatility 3 Framework 2.27.0
Progress:  100.00		PDB scanning finished
Variable	Value

Kernel Base	0xf8000281e000
DTB	0x187000
Symbols	file:///home/sansforensics/venv/vol/lib/python3.12/site-packages/volatility3/symbols/windows/ntkrnlmp.pdb/0D850C4D902640DFB487885688EA4213-1.json.xz
Is64Bit	True
IsPAE	False
layer_name	0 WindowsIntel32e
memory_layer	1 FileLayer
KdVersionBlock	0xf80002a000e8
Major/Minor	0.1515
MachineType	17607
KeNumberProcessors	0
SystemTime	2024-08-06 18:37:19+00:00
NtSystemRoot	C:\Windows
NtProductType	NtProductWinNt
NtMajorVersion	6
NtMinorVersion	1
PE MajorOperatingSystemVersion	6
PE MinorOperatingSystemVersion	1
PE Machine	34404
PE TimeDateStamp	Sun Nov 11 01:25:22 2018
```

### windows.pslist
``` sh
$ vol -f gotham.raw windows.pslist
Volatility 3 Framework 2.27.0
Progress:  100.00		PDB scanning finished
PID	PPID	ImageFileName	Offset(V)	Threads	Handles	SessionId	Wow64	CreateTime	ExitTime	File output
```
- all processes seem to have been unlinked from the process chain

### windows.psscan
``` sh
$ vol -r pretty -f gotham.raw windows.psscan > psscan.txt
Formatting...0.00		PDB scanning finished
$ cat psscan.txt
Volatility 3 Framework 2.27.0
  |  PID |            PPID |  ImageFileName |   Offset(V) | Threads | Handles | SessionId | Wow64 |                     CreateTime |                       ExitTime | File output
* |    4 |               0 |         System |   0x70d4040 |     113 |     573 |       N/A | False | 2024-08-07 04:19:36.000000 UTC |                            N/A |    Disabled
* | 8192 | 255662200399056 |              l |  0xa9658ef8 | 6422577 |       - |         - |  True | 2013-02-01 18:37:15.000000 UTC | 2024-08-07 04:55:16.000000 UTC |    Disabled
* | 2240 |            1172 |   VBoxTray.exe | 0x11ca5f780 |      17 |     165 |         1 | False | 2024-08-06 15:50:00.000000 UTC |                            N/A |    Disabled
* | 4188 |             460 |    conhost.exe | 0x11cb58060 |       2 |      53 |         1 | False | 2024-08-06 16:45:56.000000 UTC |                            N/A |    Disabled
* | 2888 |             552 |    svchost.exe | 0x11cb90b00 |      10 |     372 |         0 | False | 2024-08-06 15:50:33.000000 UTC |                            N/A |    Disabled
* | 1172 |            2024 |   explorer.exe | 0x11cdbeb00 |      32 |    1017 |         1 | False | 2024-08-06 15:49:58.000000 UTC |                            N/A |    Disabled
* | 2460 |             552 | SearchIndexer. | 0x11cdc0b00 |      13 |     664 |         0 | False | 2024-08-06 15:50:06.000000 UTC |                            N/A |    Disabled
* | 1064 |             552 |    svchost.exe | 0x11ce16b00 |      18 |     494 |         0 | False | 2024-08-06 15:49:47.000000 UTC |                            N/A |    Disabled
* | 1400 |             552 |    svchost.exe | 0x11ce8cb00 |      10 |     156 |         0 | False | 2024-08-06 15:49:49.000000 UTC |                            N/A |    Disabled
* | 1280 |             552 |    spoolsv.exe | 0x11cf05590 |      14 |     323 |         0 | False | 2024-08-06 15:49:48.000000 UTC |                            N/A |    Disabled
* | 1308 |             552 |    svchost.exe | 0x11cf1a280 |      16 |     320 |         0 | False | 2024-08-06 15:49:48.000000 UTC |                            N/A |    Disabled
* | 1428 |             552 |    svchost.exe | 0x11cfa8b00 |      20 |     366 |         0 | False | 2024-08-06 15:49:50.000000 UTC |                            N/A |    Disabled
* |  552 |             452 |   services.exe | 0x11d082b00 |       7 |     228 |         0 | False | 2024-08-07 04:19:43.000000 UTC |                            N/A |    Disabled
* |  516 |             444 |   winlogon.exe | 0x11d085280 |       5 |     123 |         1 | False | 2024-08-07 04:19:43.000000 UTC |                            N/A |    Disabled
* |  576 |             452 |        lsm.exe | 0x11d0d5b00 |      10 |     154 |         0 | False | 2024-08-07 04:19:44.000000 UTC |                            N/A |    Disabled
* |  568 |             452 |      lsass.exe | 0x11d0d7060 |       9 |     745 |         0 | False | 2024-08-07 04:19:44.000000 UTC |                            N/A |    Disabled
* |  680 |             552 |    svchost.exe | 0x11d114b00 |      11 |     375 |         0 | False | 2024-08-07 04:19:45.000000 UTC |                            N/A |    Disabled
* | 1080 |             552 |    svchost.exe | 0x11d13bb00 |      13 |     385 |         0 | False | 2024-08-06 15:52:01.000000 UTC |                            N/A |    Disabled
* | 1328 |             552 |   taskhost.exe | 0x11d1b0b00 |       8 |     216 |         1 | False | 2024-08-06 15:49:58.000000 UTC |                            N/A |    Disabled
* |  936 |             552 |    svchost.exe | 0x11d1c4860 |      27 |     610 |         0 | False | 2024-08-06 15:49:46.000000 UTC |                            N/A |    Disabled
* |  968 |             552 |    svchost.exe | 0x11d1d0b00 |      20 |     651 |         0 | False | 2024-08-06 15:49:46.000000 UTC |                            N/A |    Disabled
* | 1004 |             552 |    svchost.exe | 0x11d1e43e0 |      36 |    1041 |         0 | False | 2024-08-06 15:49:46.000000 UTC |                            N/A |    Disabled
* |  452 |             388 |    wininit.exe | 0x11d233360 |       3 |      81 |         0 | False | 2024-08-07 04:19:42.000000 UTC |                            N/A |    Disabled
* |  804 |             552 |    svchost.exe | 0x11d2de5c0 |       8 |     305 |         0 | False | 2024-08-06 15:49:46.000000 UTC |                            N/A |    Disabled
* |  400 |             388 |      csrss.exe | 0x11d2e2b00 |       9 |     524 |         0 | False | 2024-08-07 04:19:42.000000 UTC |                            N/A |    Disabled
* |  896 |             552 |    svchost.exe | 0x11d3c8750 |      23 |     593 |         0 | False | 2024-08-06 15:49:46.000000 UTC |                            N/A |    Disabled
* |  460 |             444 |      csrss.exe | 0x11d5c7b00 |      12 |     535 |         1 | False | 2024-08-07 04:19:42.000000 UTC |                            N/A |    Disabled
* | 3808 |            4456 |     chrome.exe | 0x11ec7fb00 |      21 |     254 |         1 | False | 2024-08-06 18:15:29.000000 UTC |                            N/A |    Disabled
* | 3620 |             552 |   taskhost.exe | 0x11ee46060 |       5 |      96 |         1 | False | 2024-08-06 16:37:06.000000 UTC |                            N/A |    Disabled
* |  912 |             552 |     sppsvc.exe | 0x11ee87410 |       7 |     157 |         0 | False | 2024-08-06 15:52:00.000000 UTC |                            N/A |    Disabled
* |  744 |             552 | VBoxService.ex | 0x11eeeb8e0 |      13 |     124 |         0 | False | 2024-08-07 04:19:45.000000 UTC |                            N/A |    Disabled
* | 4028 |             896 |    audiodg.exe | 0x11ef1c600 |       6 |     132 |         0 | False | 2024-08-06 18:37:14.000000 UTC |                            N/A |    Disabled
* | 4208 |            4456 |     chrome.exe | 0x11ef45600 |       0 |       - |         1 | False | 2024-08-06 16:37:23.000000 UTC | 2024-08-06 16:37:47.000000 UTC |    Disabled
* |  312 |               4 |       smss.exe | 0x11ef69040 |       2 |      34 |       N/A | False | 2024-08-07 04:19:36.000000 UTC |                            N/A |    Disabled
* | 4308 |             680 |    dllhost.exe | 0x11eff6600 |       0 |       - |         1 | False | 2024-08-06 18:37:20.000000 UTC | 2024-08-06 18:37:24.000000 UTC |    Disabled
* | 4452 |            4456 |     chrome.exe | 0x11f1c5060 |      17 |     270 |         1 | False | 2024-08-06 17:25:33.000000 UTC |                            N/A |    Disabled
* | 3944 |            1172 |        cmd.exe | 0x11f203b00 |       1 |      20 |         1 | False | 2024-08-06 16:45:56.000000 UTC |                            N/A |    Disabled
* | 4204 |            4456 |     chrome.exe | 0x11f212060 |      11 |     191 |         1 | False | 2024-08-06 16:37:16.000000 UTC |                            N/A |    Disabled
* | 3704 |            4456 |     chrome.exe | 0x11f23e060 |      17 |     253 |         1 | False | 2024-08-06 17:24:55.000000 UTC |                            N/A |    Disabled
* | 4432 |            4456 |     chrome.exe | 0x11f263060 |       8 |     119 |         1 | False | 2024-08-06 16:36:45.000000 UTC |                            N/A |    Disabled
* | 3172 |            4456 |     chrome.exe | 0x11f26a9a0 |      17 |     258 |         1 | False | 2024-08-06 17:24:52.000000 UTC |                            N/A |    Disabled
* | 3612 |            4456 |     chrome.exe | 0x11f27c060 |      17 |     253 |         1 | False | 2024-08-06 17:24:48.000000 UTC |                            N/A |    Disabled
* | 4456 |            4464 |     chrome.exe | 0x11f2c3b00 |      32 |    1296 |         1 | False | 2024-08-06 16:36:45.000000 UTC |                            N/A |    Disabled
* | 4436 |            2460 | SearchProtocol | 0x11f2d6700 |       9 |     285 |         0 | False | 2024-08-06 18:36:43.000000 UTC |                            N/A |    Disabled
* | 4872 |            4456 |     chrome.exe | 0x11f311b00 |       8 |     155 |         1 | False | 2024-08-06 16:36:47.000000 UTC |                            N/A |    Disabled
* | 3044 |            4908 | GoogleCrashHan | 0x11f387060 |       5 |      92 |         0 |  True | 2024-08-06 16:36:44.000000 UTC |                            N/A |    Disabled
* | 4140 |             460 |    conhost.exe | 0x11f3dca30 |       2 |      53 |         1 | False | 2024-08-06 18:37:17.000000 UTC |                            N/A |    Disabled
* | 3740 |            4456 |     chrome.exe | 0x11f57b4d0 |      12 |     164 |         1 | False | 2024-08-06 18:32:47.000000 UTC |                            N/A |    Disabled
* | 4836 |            4456 |     chrome.exe | 0x11f631290 |      17 |     241 |         1 | False | 2024-08-06 17:26:01.000000 UTC |                            N/A |    Disabled
* | 1496 |            2460 | SearchFilterHo | 0x11f6403e0 |       6 |     105 |         0 | False | 2024-08-06 18:36:43.000000 UTC |                            N/A |    Disabled
* | 3692 |             680 |    dllhost.exe | 0x11f6b74a0 |       0 |       - |         0 | False | 2024-08-06 18:37:17.000000 UTC | 2024-08-06 18:37:22.000000 UTC |    Disabled
* | 4196 |            1004 |    taskeng.exe | 0x11f886060 |       4 |      89 |         0 | False | 2024-08-06 18:33:18.000000 UTC |                            N/A |    Disabled
* | 3764 |            4456 |     chrome.exe | 0x11f93cb00 |      17 |     253 |         1 | False | 2024-08-06 17:24:44.000000 UTC |                            N/A |    Disabled
* | 4612 |            4456 |     chrome.exe | 0x11f9f9060 |      17 |     229 |         1 | False | 2024-08-06 16:36:55.000000 UTC |                            N/A |    Disabled
* | 2592 |            1172 |    notepad.exe | 0x11fa9c4f0 |       1 |      58 |         1 | False | 2024-08-06 16:47:20.000000 UTC |                            N/A |    Disabled
* | 2608 |            4456 |     chrome.exe | 0x11faebb00 |      17 |     250 |         1 | False | 2024-08-06 17:24:45.000000 UTC |                            N/A |    Disabled
* | 2168 |            4456 |     chrome.exe | 0x11fb47860 |      17 |     231 |         1 | False | 2024-08-06 17:27:52.000000 UTC |                            N/A |    Disabled
* | 1648 |             552 |    svchost.exe | 0x11fb94060 |       7 |     112 |         0 | False | 2024-08-06 18:35:09.000000 UTC |                            N/A |    Disabled
* | 4928 |            4456 |     chrome.exe | 0x11fbaeb00 |      13 |     234 |         1 | False | 2024-08-06 16:36:47.000000 UTC |                            N/A |    Disabled
* | 4960 |            1172 |   DumpItog.exe | 0x11fdcf420 |       5 |      56 |         1 |  True | 2024-08-06 18:37:17.000000 UTC |                            N/A |    Disabled
* | 1556 |             936 |        dwm.exe | 0x11fec4b00 |       3 |     110 |         1 | False | 2024-08-06 15:49:58.000000 UTC |                            N/A |    Disabled
* | 2784 |             552 |   wmpnetwk.exe | 0x11fece060 |      33 |     462 |         0 | False | 2024-08-06 15:50:32.000000 UTC |                            N/A |    Disabled
* | 2516 |            1172 |    mspaint.exe | 0x11ffc2490 |       7 |     142 |         1 | False | 2024-08-06 18:35:09.000000 UTC |                            N/A |    Disabled
* |  408 |            4908 | GoogleCrashHan | 0x11ffc5b00 |       5 |      85 |         0 | False | 2024-08-06 16:36:44.000000 UTC |                            N/A |    Disabled
```
- strange process called "l"; seems to be false positive

### windows.netscan
``` sh
$ vol -f gotham.raw -r pretty windows.netscan > netscan.txt
Formatting...0.00		PDB scanning finished
$ cat netscan.txt
Volatility 3 Framework 2.27.0
  |      Offset | Proto |                 LocalAddr | LocalPort |     ForeignAddr | ForeignPort |       State |  PID |          Owner |                        Created
* | 0x11ca05b20 | UDPv6 | fe80::4d41:8807:3edf:9b0e |     51662 |               * |           0 |             | 1428 |    svchost.exe | 2024-08-06 17:23:55.000000 UTC
* | 0x11ca71830 | UDPv4 |                 127.0.0.1 |     60096 |               * |           0 |             | 1064 |    svchost.exe | 2024-08-06 15:50:00.000000 UTC
* | 0x11cab7b00 | UDPv4 |                   0.0.0.0 |      3702 |               * |           0 |             | 1428 |    svchost.exe | 2024-08-06 17:23:55.000000 UTC
* | 0x11cab7b00 | UDPv6 |                        :: |      3702 |               * |           0 |             | 1428 |    svchost.exe | 2024-08-06 17:23:55.000000 UTC
* | 0x11cb55510 | UDPv4 |                   0.0.0.0 |         0 |               * |           0 |             | 2888 |    svchost.exe | 2024-08-06 15:50:33.000000 UTC
* | 0x11cb55510 | UDPv6 |                        :: |         0 |               * |           0 |             | 2888 |    svchost.exe | 2024-08-06 15:50:33.000000 UTC
* | 0x11cb8ad60 | UDPv6 |                       ::1 |     51663 |               * |           0 |             | 1428 |    svchost.exe | 2024-08-06 17:23:55.000000 UTC
* | 0x11cbcf150 | UDPv4 |                   0.0.0.0 |         0 |               * |           0 |             | 2888 |    svchost.exe | 2024-08-06 15:50:33.000000 UTC
* | 0x11cbcf150 | UDPv6 |                        :: |         0 |               * |           0 |             | 2888 |    svchost.exe | 2024-08-06 15:50:33.000000 UTC
* | 0x11cd3f5d0 | TCPv4 |                 10.0.2.15 |     49741 | 142.250.205.238 |         443 |      CLOSED |    - |              - |                            N/A
* | 0x11ceb1480 | TCPv4 |                   0.0.0.0 |     49154 |         0.0.0.0 |           0 |   LISTENING | 1004 |    svchost.exe |                              -
* | 0x11cecf8b0 | TCPv4 |                   0.0.0.0 |      5357 |         0.0.0.0 |           0 |   LISTENING |    4 |         System |                              -
* | 0x11cecf8b0 | TCPv6 |                        :: |      5357 |              :: |           0 |   LISTENING |    4 |         System |                              -
* | 0x11cee3ee0 | TCPv4 |                   0.0.0.0 |     49154 |         0.0.0.0 |           0 |   LISTENING | 1004 |    svchost.exe |                              -
* | 0x11cee3ee0 | TCPv6 |                        :: |     49154 |              :: |           0 |   LISTENING | 1004 |    svchost.exe |                              -
* | 0x11d17cee0 | TCPv4 |                   0.0.0.0 |       135 |         0.0.0.0 |           0 |   LISTENING |  804 |    svchost.exe |                              -
* | 0x11d17cee0 | TCPv6 |                        :: |       135 |              :: |           0 |   LISTENING |  804 |    svchost.exe |                              -
* | 0x11d18b930 | TCPv4 |                   0.0.0.0 |     49152 |         0.0.0.0 |           0 |   LISTENING |  452 |    wininit.exe |                              -
* | 0x11d18e160 | TCPv4 |                   0.0.0.0 |     49152 |         0.0.0.0 |           0 |   LISTENING |  452 |    wininit.exe |                              -
* | 0x11d18e160 | TCPv6 |                        :: |     49152 |              :: |           0 |   LISTENING |  452 |    wininit.exe |                              -
* | 0x11d18f9b0 | TCPv4 |                   0.0.0.0 |     49156 |         0.0.0.0 |           0 |   LISTENING |  568 |      lsass.exe |                              -
* | 0x11d1be2f0 | TCPv4 |                   0.0.0.0 |     49153 |         0.0.0.0 |           0 |   LISTENING |  896 |    svchost.exe |                              -
* | 0x11d1c03e0 | TCPv4 |                   0.0.0.0 |     49153 |         0.0.0.0 |           0 |   LISTENING |  896 |    svchost.exe |                              -
* | 0x11d1c03e0 | TCPv6 |                        :: |     49153 |              :: |           0 |   LISTENING |  896 |    svchost.exe |                              -
* | 0x11d2ca340 | UDPv4 |                   0.0.0.0 |      3702 |               * |           0 |             |  968 |    svchost.exe | 2024-08-06 17:24:02.000000 UTC
* | 0x11d2ca340 | UDPv6 |                        :: |      3702 |               * |           0 |             |  968 |    svchost.exe | 2024-08-06 17:24:02.000000 UTC
* | 0x11d327540 | UDPv6 |                       ::1 |      1900 |               * |           0 |             | 1428 |    svchost.exe | 2024-08-06 17:23:23.000000 UTC
* | 0x11d5f5010 | UDPv4 |                   0.0.0.0 |      5004 |               * |           0 |             | 2784 |   wmpnetwk.exe | 2024-08-06 15:50:54.000000 UTC
* | 0x11da9cee0 | TCPv4 |                 10.0.2.15 |       139 |         0.0.0.0 |           0 |   LISTENING |    4 |         System |                              -
* | 0x11ec18cd0 | TCPv4 |                 10.0.2.15 |     49806 |   142.250.182.3 |         443 | ESTABLISHED |    - |              - |                            N/A
* | 0x11ec3e890 | UDPv4 |                   0.0.0.0 |      5005 |               * |           0 |             | 2784 |   wmpnetwk.exe | 2024-08-06 15:50:54.000000 UTC
* | 0x11ec3e890 | UDPv6 |                        :: |      5005 |               * |           0 |             | 2784 |   wmpnetwk.exe | 2024-08-06 15:50:54.000000 UTC
* | 0x11ec52b10 | UDPv4 |                   0.0.0.0 |      5353 |               * |           0 |             | 4456 |     chrome.exe | 2024-08-06 17:23:28.000000 UTC
* | 0x11ec6f9f0 | TCPv4 |                   0.0.0.0 |     10243 |         0.0.0.0 |           0 |   LISTENING |    4 |         System |                              -
* | 0x11ec6f9f0 | TCPv6 |                        :: |     10243 |              :: |           0 |   LISTENING |    4 |         System |                              -
* | 0x11ecb4010 | TCPv4 |                   0.0.0.0 |       554 |         0.0.0.0 |           0 |   LISTENING | 2784 |   wmpnetwk.exe |                              -
* | 0x11ecb4010 | TCPv6 |                        :: |       554 |              :: |           0 |   LISTENING | 2784 |   wmpnetwk.exe |                              -
* | 0x11eccc010 | UDPv4 |                 127.0.0.1 |     51665 |               * |           0 |             | 1428 |    svchost.exe | 2024-08-06 17:23:55.000000 UTC
* | 0x11ecd4910 | UDPv4 |                   0.0.0.0 |      5004 |               * |           0 |             | 2784 |   wmpnetwk.exe | 2024-08-06 15:50:54.000000 UTC
* | 0x11ecd4910 | UDPv6 |                        :: |      5004 |               * |           0 |             | 2784 |   wmpnetwk.exe | 2024-08-06 15:50:54.000000 UTC
* | 0x11ed2eec0 | UDPv4 |                   0.0.0.0 |      5005 |               * |           0 |             | 2784 |   wmpnetwk.exe | 2024-08-06 15:50:54.000000 UTC
* | 0x11ed6c9b0 | TCPv4 |                   0.0.0.0 |       135 |         0.0.0.0 |           0 |   LISTENING |  804 |    svchost.exe |                              -
* | 0x11edd82c0 | TCPv4 |                   0.0.0.0 |     49156 |         0.0.0.0 |           0 |   LISTENING |  568 |      lsass.exe |                              -
* | 0x11edd82c0 | TCPv6 |                        :: |     49156 |              :: |           0 |   LISTENING |  568 |      lsass.exe |                              -
* | 0x11ee62610 | UDPv4 |                 10.0.2.15 |     64369 |               * |           0 |             | 4456 |     chrome.exe | 2024-08-06 16:36:51.000000 UTC
* | 0x11eef2c90 | UDPv4 |                   0.0.0.0 |         0 |               * |           0 |             | 2888 |    svchost.exe | 2024-08-06 15:50:43.000000 UTC
* | 0x11eef2c90 | UDPv6 |                        :: |         0 |               * |           0 |             | 2888 |    svchost.exe | 2024-08-06 15:50:43.000000 UTC
* | 0x11eef4950 | UDPv4 |                   0.0.0.0 |      3540 |               * |           0 |             | 2888 |    svchost.exe | 2024-08-06 15:50:43.000000 UTC
* | 0x11eef4950 | UDPv6 |                        :: |      3540 |               * |           0 |             | 2888 |    svchost.exe | 2024-08-06 15:50:43.000000 UTC
* | 0x11ef04cd0 | TCPv4 |                 10.0.2.15 |     49800 |  216.58.196.163 |         443 |      CLOSED |    - |              - |                              -
* | 0x11ef242d0 | UDPv4 |                   0.0.0.0 |     52481 |               * |           0 |             | 4928 |     chrome.exe | 2024-08-06 18:37:21.000000 UTC
* | 0x11ef5c7d0 | TCPv4 |                 10.0.2.15 |     49802 | 142.250.196.163 |         443 | ESTABLISHED |    - |              - |                            N/A
* | 0x11ef67b10 | UDPv4 |                   0.0.0.0 |         0 |               * |           0 |             | 1004 |    svchost.exe | 2024-08-06 16:40:52.000000 UTC
* | 0x11ef7d9c0 | UDPv4 |                   0.0.0.0 |         0 |               * |           0 |             |  744 | VBoxService.ex | 2024-08-06 18:37:43.000000 UTC
* | 0x11f148c50 | TCPv4 |                   0.0.0.0 |       554 |         0.0.0.0 |           0 |   LISTENING | 2784 |   wmpnetwk.exe |                              -
* | 0x11f258ec0 | UDPv4 |                   0.0.0.0 |      5353 |               * |           0 |             | 4456 |     chrome.exe | 2024-08-06 17:23:28.000000 UTC
* | 0x11f258ec0 | UDPv6 |                        :: |      5353 |               * |           0 |             | 4456 |     chrome.exe | 2024-08-06 17:23:28.000000 UTC
* | 0x11f269da0 | UDPv4 |                   0.0.0.0 |      3702 |               * |           0 |             | 1428 |    svchost.exe | 2024-08-06 17:23:55.000000 UTC
* | 0x11f2da450 | UDPv4 |                   0.0.0.0 |         0 |               * |           0 |             | 1064 |    svchost.exe | 2024-08-06 17:23:24.000000 UTC
* | 0x11f2da450 | UDPv6 |                        :: |         0 |               * |           0 |             | 1064 |    svchost.exe | 2024-08-06 17:23:24.000000 UTC
* | 0x11f30bc30 | UDPv4 |                 10.0.2.15 |       138 |               * |           0 |             |    4 |         System | 2024-08-06 17:23:24.000000 UTC
* | 0x11f352730 | TCPv4 |                 127.0.0.1 |      2869 |       127.0.0.1 |       49791 |      CLOSED |    - |              - |                            N/A
* | 0x11f37ca90 | UDPv4 |                   0.0.0.0 |      3702 |               * |           0 |             | 1428 |    svchost.exe | 2024-08-06 17:23:55.000000 UTC
* | 0x11f37ca90 | UDPv6 |                        :: |      3702 |               * |           0 |             | 1428 |    svchost.exe | 2024-08-06 17:23:55.000000 UTC
* | 0x11f52fc80 | TCPv4 |                   0.0.0.0 |      2869 |         0.0.0.0 |           0 |   LISTENING |    4 |         System |                              -
* | 0x11f52fc80 | TCPv6 |                        :: |      2869 |              :: |           0 |   LISTENING |    4 |         System |                              -
* | 0x11f623c90 | UDPv4 |                   0.0.0.0 |      3702 |               * |           0 |             |  968 |    svchost.exe | 2024-08-06 17:24:02.000000 UTC
* | 0x11f6db130 | UDPv6 | fe80::4d41:8807:3edf:9b0e |      1900 |               * |           0 |             | 1428 |    svchost.exe | 2024-08-06 17:23:23.000000 UTC
* | 0x11f6ea110 | UDPv4 |                   0.0.0.0 |      3702 |               * |           0 |             |  968 |    svchost.exe | 2024-08-06 17:24:02.000000 UTC
* | 0x11f6ea110 | UDPv6 |                        :: |      3702 |               * |           0 |             |  968 |    svchost.exe | 2024-08-06 17:24:02.000000 UTC
* | 0x11f85d7b0 | TCPv4 |                 127.0.0.1 |     49797 |       127.0.0.1 |        2869 |      CLOSED |    - |              - |                              -
* | 0x11f8658b0 | UDPv4 |                 10.0.2.15 |     51664 |               * |           0 |             | 1428 |    svchost.exe | 2024-08-06 17:23:55.000000 UTC
* | 0x11f874cd0 | TCPv4 |                 10.0.2.15 |     49804 | 142.250.186.163 |         443 | ESTABLISHED |    - |              - |                            N/A
* | 0x11f87f720 | TCPv4 |                 10.0.2.15 |     49805 | 142.250.205.238 |         443 | ESTABLISHED |    - |              - |                            N/A
* | 0x11f88ac90 | UDPv4 |                 10.0.2.15 |       137 |               * |           0 |             |    4 |         System | 2024-08-06 17:23:24.000000 UTC
* | 0x11f8d5950 | UDPv4 |                   0.0.0.0 |         0 |               * |           0 |             |  744 | VBoxService.ex | 2024-08-06 18:37:23.000000 UTC
* | 0x11f972620 | UDPv4 |                 10.0.2.15 |      1900 |               * |           0 |             | 1428 |    svchost.exe | 2024-08-06 17:23:23.000000 UTC
* | 0x11f974ec0 | UDPv4 |                   0.0.0.0 |      3702 |               * |           0 |             | 1428 |    svchost.exe | 2024-08-06 17:23:55.000000 UTC
* | 0x11f9c8d60 | UDPv4 |                   0.0.0.0 |     58814 |               * |           0 |             | 4928 |     chrome.exe | 2024-08-06 18:37:21.000000 UTC
* | 0x11fa4b1c0 | UDPv4 |                   0.0.0.0 |     55717 |               * |           0 |             |  968 |    svchost.exe | 2024-08-06 17:24:02.000000 UTC
* | 0x11fa4b1c0 | UDPv6 |                        :: |     55717 |               * |           0 |             |  968 |    svchost.exe | 2024-08-06 17:24:02.000000 UTC
* | 0x11fa68960 | TCPv4 |                 10.0.2.15 |     49801 |  216.58.200.132 |         443 | ESTABLISHED |    - |              - |                            N/A
* | 0x11fbd9240 | TCPv4 |                   0.0.0.0 |      3587 |         0.0.0.0 |           0 |   LISTENING | 2888 |    svchost.exe |                              -
* | 0x11fbd9240 | TCPv6 |                        :: |      3587 |              :: |           0 |   LISTENING | 2888 |    svchost.exe |                              -
* | 0x11fbdf170 | UDPv4 |                   0.0.0.0 |         0 |               * |           0 |             |  744 | VBoxService.ex | 2024-08-06 16:24:29.000000 UTC
* | 0x11fbf2310 | UDPv4 |                 127.0.0.1 |      1900 |               * |           0 |             | 1428 |    svchost.exe | 2024-08-06 17:23:23.000000 UTC
* | 0x11fd39010 | UDPv4 |                   0.0.0.0 |      5355 |               * |           0 |             | 1064 |    svchost.exe | 2024-08-06 17:23:25.000000 UTC
* | 0x11fd39010 | UDPv6 |                        :: |      5355 |               * |           0 |             | 1064 |    svchost.exe | 2024-08-06 17:23:25.000000 UTC
* | 0x11fd8ac50 | UDPv4 |                   0.0.0.0 |      3702 |               * |           0 |             |  968 |    svchost.exe | 2024-08-06 17:24:02.000000 UTC
* | 0x11fdfb8a0 | UDPv4 |                   0.0.0.0 |      5355 |               * |           0 |             | 1064 |    svchost.exe | 2024-08-06 17:23:25.000000 UTC
* | 0x11fe53300 | TCPv4 |                   0.0.0.0 |       445 |         0.0.0.0 |           0 |   LISTENING |    4 |         System |                              -
* | 0x11fe53300 | TCPv6 |                        :: |       445 |              :: |           0 |   LISTENING |    4 |         System |                              -
* | 0x11fe64cd0 | TCPv4 |                         - |         0 |      8.43.104.6 |           0 |      CLOSED |    - |              - |                            N/A
* | 0x11fe6b010 | TCPv4 |                   0.0.0.0 |     49155 |         0.0.0.0 |           0 |   LISTENING |  552 |   services.exe |                              -
* | 0x11fe74280 | UDPv4 |                   0.0.0.0 |     61835 |               * |           0 |             | 1428 |    svchost.exe | 2024-08-06 15:49:53.000000 UTC
* | 0x11fe76d00 | UDPv4 |                   0.0.0.0 |     61836 |               * |           0 |             | 1428 |    svchost.exe | 2024-08-06 15:49:53.000000 UTC
* | 0x11fe76d00 | UDPv6 |                        :: |     61836 |               * |           0 |             | 1428 |    svchost.exe | 2024-08-06 15:49:53.000000 UTC
* | 0x11fe7bc80 | TCPv4 |                   0.0.0.0 |     49155 |         0.0.0.0 |           0 |   LISTENING |  552 |   services.exe |                              -
* | 0x11fe7bc80 | TCPv6 |                        :: |     49155 |              :: |           0 |   LISTENING |  552 |   services.exe |                              -
* | 0x11fea65f0 | UDPv4 |                   0.0.0.0 |     55716 |               * |           0 |             |  968 |    svchost.exe | 2024-08-06 17:24:02.000000 UTC
```
- suspicious foreign IPs found; no owners:
	- 142[.]250[.]205[.]238
	- 142[.]250[.]182[.]3
	- 216[.]58[.]196[.]163
	- 142[.]250[.]196[.]163
	- 142[.]250[.]186[.]163
	- 142[.]250[.]205[.]238
	- 216[.]58[.]200[.]132
	- 8[.]43[.]104[.]6

### hivelist
``` sh
$ vol -r csv -f gotham.raw windows.registry.hivelist
Volatility 3 Framework 2.27.0
TreeDepth,Offset,FileFullPath,File outputing finished
```

### hivescan
``` sh
$ vol -r csv -f gotham.raw windows.registry.hivescan
Volatility 3 Framework 2.27.0
TreeDepth,Offset0		PDB scanning finished
0,0x60a71134
0,0x60a7114c
0,0x60a7116c
0,0x6163a24c
0,0x62254608
0,0x62254620
0,0x62254640
0,0x858e0410
0,0x865c7010
0,0x87392010
0,0x9159d010
0,0x91773010
0,0x9221f010
0,0x93223010
0,0x9af62010
0,0xa3b46010
0,0xa3f57410
0,0xa96e4010
0,0xa9812010
0,0xa988f010
0,0xa9962214
0,0xb793a720
```

### yarascan
- search for IPs
``` sh
$ vol -r csv -f gotham.raw yarascan.YaraScan --yara-string "/[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}/" --max-size 4831838208 > ip_scan.csv
Volatility 3 Framework 2.27.0
$ csvtool cols 2,5 ip_scan.csv | csvtool readable - > ipscan.txt
$ vim ipscan.txt
     1 Offset         Value
     2 0xf80003c027d6
     3 31 2e 30 2e 30 2e 30                            1.0.0.0
     4 0xf80003c027d8
     5 30 2e 30 2e 30 2e 30                            0.0.0.0
[REMOVED]
```

### modscan
``` sh
$ vol -r csv -f gotham.raw windows.modscan > modscan.csv
Volatility 3 Framework 2.27.0
$ csvtool readable modscan.csv
TreeDepth Offset      Base           Size       Name             Path                                              File output
0         0x28dc3a9   0x24748b483024 0x48cccccc -                -                                                 Disabled
0         0x2c37857   0x6840c70772   0x48206848 -                -                                                 Disabled
0         0x2ca0c05   0x24548d48ffc2 0x448b48ff -                -                                                 Disabled
0         0x2ddbec1   0x304b8b48ffb0 0x37f9e800 -                -                                                 Disabled
0         0x705d070   0xf8000281e000 0x48000    hal.dll          \\SystemRoot\\system32\\hal.dll                   Disabled
0         0x70636c0   0xf88000c42000 0x4f000    mcupdate.dll     \\SystemRoot\\system32\\mcupdate_GenuineIntel.dll Disabled
0         0x70637b0   0xf80000b9a000 0xa000     kdcom.dll        \\SystemRoot\\system32\\kdcom.dll                 Disabled
0         0x7063890   0xf80002866000 0x5dc000   ntoskrnl.exe     \\SystemRoot\\system32\\ntoskrnl.exe              Disabled
0         0x7064010   0xf88000c91000 0x14000    PSHED.dll        \\SystemRoot\\system32\\PSHED.dll                 Disabled
0         0x7064170   0xf88000fde000 0x9000     atapi.sys        \\SystemRoot\\system32\\drivers\\atapi.sys        Disabled
0         0x7064250   0xf88000fc4000 0x1a000    mountmgr.sys     \\SystemRoot\\System32\\drivers\\mountmgr.sys     Disabled
0         0x7064340   0xf88000d77000 0x5c000    volmgrx.sys      \\SystemRoot\\System32\\drivers\\volmgrx.sys      Disabled
0         0x7064430   0xf88000fb0000 0x14000    volmgr.sys       \\SystemRoot\\system32\\drivers\\volmgr.sys       Disabled
0         0x7064510   0xf88000ec8000 0xc000     BATTC.SYS        \\SystemRoot\\system32\\DRIVERS\\BATTC.SYS        Disabled
0         0x70645f0   0xf88000ebf000 0x9000     compbatt.sys     \\SystemRoot\\system32\\DRIVERS\\compbatt.sys     Disabled
0         0x70646e0   0xf88000eaa000 0x15000    partmgr.sys      \\SystemRoot\\System32\\drivers\\partmgr.sys      Disabled
0         0x70647d0   0xf88000e9d000 0xd000     vdrvroot.sys     \\SystemRoot\\system32\\drivers\\vdrvroot.sys     Disabled
0         0x70648c0   0xf88000e6a000 0x33000    pci.sys          \\SystemRoot\\system32\\drivers\\pci.sys          Disabled
0         0x70649a0   0xf88000e60000 0xa000     msisadrv.sys     \\SystemRoot\\system32\\drivers\\msisadrv.sys     Disabled
0         0x7064a90   0xf88000e57000 0x9000     WMILIB.SYS       \\SystemRoot\\system32\\drivers\\WMILIB.SYS       Disabled
0         0x7064b70   0xf88000e00000 0x57000    ACPI.sys         \\SystemRoot\\system32\\drivers\\ACPI.sys         Disabled
0         0x7064c60   0xf88000fa0000 0x10000    WDFLDR.SYS       \\SystemRoot\\system32\\drivers\\WDFLDR.SYS       Disabled
0         0x7064d40   0xf88000ede000 0xc2000    Wdf01000.sys     \\SystemRoot\\system32\\drivers\\Wdf01000.sys     Disabled
0         0x7064e40   0xf88000d04000 0x73000    CI.dll           \\SystemRoot\\system32\\CI.dll                    Disabled
0         0x7064f20   0xf88000ca5000 0x5f000    CLFS.SYS         \\SystemRoot\\system32\\CLFS.SYS                  Disabled
0         0x7065010   0xf88000dd3000 0x2a000    ataport.SYS      \\SystemRoot\\system32\\drivers\\ataport.SYS      Disabled
0         0x7065120   0xf88001400000 0x49000    fwpkclnt.sys     \\SystemRoot\\System32\\drivers\\fwpkclnt.sys     Disabled
0         0x7065210   0xf880015c3000 0x2b000    ksecpkg.sys      \\SystemRoot\\System32\\Drivers\\ksecpkg.sys      Disabled
0         0x7065300   0xf88001563000 0x60000    NETIO.SYS        \\SystemRoot\\system32\\drivers\\NETIO.SYS        Disabled
0         0x70653f0   0xf88001471000 0xf2000    ndis.sys         \\SystemRoot\\system32\\drivers\\ndis.sys         Disabled
0         0x70654f0   0xf88001200000 0xa000     Fs_Rec.sys       \\SystemRoot\\System32\\Drivers\\Fs_Rec.sys       Disabled
0         0x70655d0   0xf880013ef000 0x11000    pcw.sys          \\SystemRoot\\System32\\drivers\\pcw.sys          Disabled
0         0x70656b0   0xf88001000000 0x5c000    VBoxGuest.sys    \\SystemRoot\\system32\\DRIVERS\\VBoxGuest.sys    Disabled
0         0x70657a0   0xf88001176000 0x75000    cng.sys          \\SystemRoot\\System32\\Drivers\\cng.sys          Disabled
0         0x7065890   0xf880013d4000 0x1b000    ksecdd.sys       \\SystemRoot\\System32\\Drivers\\ksecdd.sys       Disabled
0         0x7065970   0xf88001118000 0x5e000    msrpc.sys        \\SystemRoot\\System32\\Drivers\\msrpc.sys        Disabled
0         0x7065a60   0xf8800122d000 0x1a7000   Ntfs.sys         \\SystemRoot\\System32\\Drivers\\Ntfs.sys         Disabled
0         0x7065b70   0xf88001104000 0x14000    fileinfo.sys     \\SystemRoot\\system32\\drivers\\fileinfo.sys     Disabled
0         0x7065c60   0xf880010ba000 0x4a000    fltmgr.sys       \\SystemRoot\\system32\\drivers\\fltmgr.sys       Disabled
0         0x7065d50   0xf88000ff2000 0xb000     amdxata.sys      \\SystemRoot\\system32\\drivers\\amdxata.sys      Disabled
0         0x7065e40   0xf88000c00000 0x10000    PCIIDEX.SYS      \\SystemRoot\\system32\\drivers\\PCIIDEX.SYS      Disabled
0         0x7065f30   0xf88000fe7000 0xb000     msahci.sys       \\SystemRoot\\system32\\drivers\\msahci.sys       Disabled
0         0x7066010   0xf88001604000 0x1fb000   tcpip.sys        \\SystemRoot\\System32\\drivers\\tcpip.sys        Disabled
0         0x70668d0   0xf880018c6000 0x30000    CLASSPNP.SYS     \\SystemRoot\\system32\\drivers\\CLASSPNP.SYS     Disabled
0         0x70669c0   0xf880018b1000 0x15000    disk.sys         \\SystemRoot\\system32\\drivers\\disk.sys         Disabled
0         0x7066aa0   0xf88001877000 0x3a000    fvevol.sys       \\SystemRoot\\System32\\DRIVERS\\fvevol.sys       Disabled
0         0x7066b80   0xf8800186e000 0x9000     hwpolicy.sys     \\SystemRoot\\System32\\drivers\\hwpolicy.sys     Disabled
0         0x7066c70   0xf8800185c000 0x12000    mup.sys          \\SystemRoot\\System32\\Drivers\\mup.sys          Disabled
0         0x7066d50   0xf88001822000 0x3a000    rdyboost.sys     \\SystemRoot\\System32\\drivers\\rdyboost.sys     Disabled
0         0x7066e40   0xf88001449000 0x8000     spldr.sys        \\SystemRoot\\System32\\Drivers\\spldr.sys        Disabled
0         0x7066f20   0xf8800105c000 0x4c000    volsnap.sys      \\SystemRoot\\system32\\drivers\\volsnap.sys      Disabled
0         0x71cf130   0xf880051c2000 0x1d000    cdfs.sys         \\SystemRoot\\system32\\DRIVERS\\cdfs.sys         Disabled
0         0x11cad0570 0xf880061e5000 0x11000    DumpIt.sys       \\??\\C:\\Windows\\SysWOW64\\Drivers\\DumpIt.sys  Disabled
0         0x11cc04670 0xf88005d50000 0x68000    srv2.sys         \\SystemRoot\\System32\\DRIVERS\\srv2.sys         Disabled
0         0x11cc12660 0xf880060cf000 0x97000    srv.sys          \\SystemRoot\\System32\\DRIVERS\\srv.sys          Disabled
0         0x11ce3e980 0xf88004c12000 0x15000    lltdio.sys       \\SystemRoot\\system32\\DRIVERS\\lltdio.sys       Disabled
0         0x11ce52670 0xf88004c27000 0x18000    rspndr.sys       \\SystemRoot\\system32\\DRIVERS\\rspndr.sys       Disabled
0         0x11ceec010 0xf880046c4000 0xc8000    HTTP.sys         \\SystemRoot\\system32\\drivers\\HTTP.sys         Disabled
0         0x11cf36f30 0xf880047a9000 0x18000    mpsdrv.sys       \\SystemRoot\\System32\\drivers\\mpsdrv.sys       Disabled
0         0x11cf58220 0xf8800478c000 0x1d000    bowser.sys       \\SystemRoot\\system32\\DRIVERS\\bowser.sys       Disabled
0         0x11cf65a10 0xf880047c1000 0x2d000    mrxsmb.sys       \\SystemRoot\\system32\\DRIVERS\\mrxsmb.sys       Disabled
0         0x11cf67af0 0xf88004600000 0x4e000    mrxsmb10.sys     \\SystemRoot\\system32\\DRIVERS\\mrxsmb10.sys     Disabled
0         0x11cf7ef20 0xf8800464e000 0x24000    mrxsmb20.sys     \\SystemRoot\\system32\\DRIVERS\\mrxsmb20.sys     Disabled
0         0x11cfa7df0 0xf88005c63000 0xaa000    peauth.sys       \\SystemRoot\\system32\\drivers\\peauth.sys       Disabled
0         0x11cfbbde0 0xf88006166000 0xe000     monitor.sys      \\SystemRoot\\system32\\DRIVERS\\monitor.sys      Disabled
0         0x11cff3bb0 0xf88005d0d000 0x31000    srvnet.sys       \\SystemRoot\\System32\\DRIVERS\\srvnet.sys       Disabled
0         0x11cff5700 0xf88005d3e000 0x12000    tcpipreg.sys     \\SystemRoot\\System32\\drivers\\tcpipreg.sys     Disabled
0         0x11d2291e0 0xf88005191000 0x19000    HIDCLASS.SYS     \\SystemRoot\\system32\\DRIVERS\\HIDCLASS.SYS     Disabled
0         0x11d22f160 0xf88005183000 0xe000     hidusb.sys       \\SystemRoot\\system32\\DRIVERS\\hidusb.sys       Disabled
0         0x11d23c450 0xf880051aa000 0x9000     HIDPARSE.SYS     \\SystemRoot\\system32\\DRIVERS\\HIDPARSE.SYS     Disabled
0         0x11d23e160 0xf880051b3000 0x2000     USBD.SYS         \\SystemRoot\\system32\\DRIVERS\\USBD.SYS         Disabled
0         0x11d23e870 0xf880051b5000 0xd000     mouhid.sys       \\SystemRoot\\system32\\DRIVERS\\mouhid.sys       Disabled
0         0x11d269940 0xf880051df000 0xc000     Dxapi.sys        \\SystemRoot\\System32\\drivers\\Dxapi.sys        Disabled
0         0x11d31d010 0xf880051eb000 0xe000     crashdmp.sys     \\SystemRoot\\System32\\Drivers\\crashdmp.sys     Disabled
0         0x11d31d480 0xf8800500c000 0xb000     dump_msahci.sys  \\SystemRoot\\System32\\Drivers\\dump_msahci.sys  Disabled
0         0x11d328010 0xf88005000000 0xc000     dump_pciidex.sys \\SystemRoot\\System32\\Drivers\\dump_dumpata.sys Disabled
0         0x11d3285e0 0xf88005017000 0x13000    dump_dumpfve.sys \\SystemRoot\\System32\\Drivers\\dump_dumpfve.sys Disabled
0         0x11d4db790 0xf88005053000 0x5a000    usbhub.sys       \\SystemRoot\\system32\\DRIVERS\\usbhub.sys       Disabled
0         0x11d55ab10 0xf8800511e000 0x3d000    portcls.sys      \\SystemRoot\\system32\\drivers\\portcls.sys      Disabled
0         0x11d55d5b0 0xf880050ad000 0x15000    NDProxy.SYS      \\SystemRoot\\System32\\Drivers\\NDProxy.SYS      Disabled
0         0x11daab7b0 0xf88004cae000 0x16000    intelppm.sys     \\SystemRoot\\system32\\DRIVERS\\intelppm.sys     Disabled
0         0x11dace830 0xf88004cc4000 0x10000    CompositeBus.sys \\SystemRoot\\system32\\DRIVERS\\CompositeBus.sys Disabled
0         0x11dad64a0 0xf88004cd4000 0x16000    AgileVpn.sys     \\SystemRoot\\system32\\DRIVERS\\AgileVpn.sys     Disabled
0         0x11dad8a60 0xf88004cea000 0x24000    rasl2tp.sys      \\SystemRoot\\system32\\DRIVERS\\rasl2tp.sys      Disabled
0         0x11dad9310 0xf88004d49000 0x1b000    raspppoe.sys     \\SystemRoot\\system32\\DRIVERS\\raspppoe.sys     Disabled
0         0x11dadab70 0xf88004d0e000 0xc000     ndistapi.sys     \\SystemRoot\\system32\\DRIVERS\\ndistapi.sys     Disabled
0         0x11dadc7f0 0xf88004d1a000 0x2f000    ndiswan.sys      \\SystemRoot\\system32\\DRIVERS\\ndiswan.sys      Disabled
0         0x11daee2f0 0xf88004d64000 0x21000    raspptp.sys      \\SystemRoot\\system32\\DRIVERS\\raspptp.sys      Disabled
0         0x11ec5f9c0 0xf880019c6000 0xb000     Msfs.SYS         \\SystemRoot\\System32\\Drivers\\Msfs.SYS         Disabled
0         0x11ec5fbe0 0xf880019d1000 0x11000    Npfs.SYS         \\SystemRoot\\System32\\Drivers\\Npfs.SYS         Disabled
0         0x11ec69ca0 0xf88001800000 0x22000    tdx.sys          \\SystemRoot\\system32\\DRIVERS\\tdx.sys          Disabled
0         0x11ec6a380 0xf880019e2000 0xd000     TDI.SYS          \\SystemRoot\\system32\\DRIVERS\\TDI.SYS          Disabled
0         0x11ec6eb00 0xf880019b4000 0x9000     rdpencdd.sys     \\SystemRoot\\system32\\drivers\\rdpencdd.sys     Disabled
0         0x11ec70300 0xf88002d82000 0x26000    pacer.sys        \\SystemRoot\\system32\\DRIVERS\\pacer.sys        Disabled
0         0x11ec70760 0xf88002d34000 0x45000    netbt.sys        \\SystemRoot\\System32\\DRIVERS\\netbt.sys        Disabled
0         0x11ec7c500 0xf88002cab000 0x89000    afd.sys          \\SystemRoot\\system32\\drivers\\afd.sys          Disabled
0         0x11ec82e50 0xf88002c00000 0x7e000    VBoxSF.sys       \\SystemRoot\\System32\\drivers\\VBoxSF.sys       Disabled
0         0x11ec83010 0xf88002d79000 0x9000     wfplwf.sys       \\SystemRoot\\system32\\DRIVERS\\wfplwf.sys       Disabled
0         0x11ec8b3e0 0xf88002da8000 0x10000    netbios.sys      \\SystemRoot\\system32\\DRIVERS\\netbios.sys      Disabled
0         0x11ec8f790 0xf88002db8000 0x14000    termdd.sys       \\SystemRoot\\system32\\DRIVERS\\termdd.sys       Disabled
0         0x11ec902e0 0xf88003ed2000 0x53000    rdbss.sys        \\SystemRoot\\system32\\DRIVERS\\rdbss.sys        Disabled
0         0x11ec94580 0xf88002c7e000 0x1b000    wanarp.sys       \\SystemRoot\\system32\\DRIVERS\\wanarp.sys       Disabled
0         0x11ec9d680 0xf88003f25000 0xc000     nsiproxy.sys     \\SystemRoot\\system32\\drivers\\nsiproxy.sys     Disabled
0         0x11ec9da70 0xf88003f31000 0xb000     mssmbios.sys     \\SystemRoot\\system32\\DRIVERS\\mssmbios.sys     Disabled
0         0x11ec9e010 0xf88003f3c000 0xf000     discache.sys     \\SystemRoot\\System32\\drivers\\discache.sys     Disabled
0         0x11ec9e870 0xf88003fd0000 0x21000    dfsc.sys         \\SystemRoot\\System32\\Drivers\\dfsc.sys         Disabled
0         0x11ec9fcf0 0xf88003f4b000 0x85000    csc.sys          \\SystemRoot\\system32\\drivers\\csc.sys          Disabled
0         0x11ec9fdf0 0xf88003e00000 0x11000    blbdrive.sys     \\SystemRoot\\system32\\DRIVERS\\blbdrive.sys     Disabled
0         0x11ecc9c10 0xf88003e37000 0x1e000    i8042prt.sys     \\SystemRoot\\system32\\DRIVERS\\i8042prt.sys     Disabled
0         0x11eccf2e0 0xf88003e55000 0xf000     kbdclass.sys     \\SystemRoot\\system32\\DRIVERS\\kbdclass.sys     Disabled
0         0x11ece0740 0xf88003e64000 0x4d000    VBoxMouse.sys    \\SystemRoot\\system32\\DRIVERS\\VBoxMouse.sys    Disabled
0         0x11ece9650 0xf88003e11000 0x26000    tunnel.sys       \\SystemRoot\\system32\\DRIVERS\\tunnel.sys       Disabled
0         0x11ecf16a0 0xf88003eb1000 0xf000     mouclass.sys     \\SystemRoot\\system32\\DRIVERS\\mouclass.sys     Disabled
0         0x11ecf4b80 0xf8800487d000 0x7f000    VBoxWddm.sys     \\SystemRoot\\system32\\DRIVERS\\VBoxWddm.sys     Disabled
0         0x11ed0f620 0xf96000010000 0x329000   win32k.sys       \\SystemRoot\\System32\\win32k.sys                Disabled
0         0x11ed1d480 0xf960005c0000 0xa000     TSDDD.dll        \\SystemRoot\\System32\\TSDDD.dll                 Disabled
0         0x11ed22210 0xf880048fc000 0xf5000    dxgkrnl.sys      \\SystemRoot\\System32\\drivers\\dxgkrnl.sys      Disabled
0         0x11ed22dd0 0xf96000720000 0x27000    cdd.dll          \\SystemRoot\\System32\\cdd.dll                   Disabled
0         0x11ed28850 0xf88004daa000 0x2000     swenum.sys       \\SystemRoot\\system32\\DRIVERS\\swenum.sys       Disabled
0         0x11ed3ab50 0xf88004c00000 0x12000    umbus.sys        \\SystemRoot\\system32\\DRIVERS\\umbus.sys        Disabled
0         0x11ed3d9c0 0xf88004dac000 0x43000    ks.sys           \\SystemRoot\\system32\\DRIVERS\\ks.sys           Disabled
0         0x11ed591b0 0xf8800486a000 0xb000     usbohci.sys      \\SystemRoot\\system32\\DRIVERS\\usbohci.sys      Disabled
0         0x11ed5e1d0 0xf88004800000 0x46000    dxgmms1.sys      \\SystemRoot\\System32\\drivers\\dxgmms1.sys      Disabled
0         0x11ed6b1d0 0xf88002dcc000 0x24000    HDAudBus.sys     \\SystemRoot\\system32\\DRIVERS\\HDAudBus.sys     Disabled
0         0x11ed761d0 0xf88004846000 0x24000    E1G6032E.sys     \\SystemRoot\\system32\\DRIVERS\\E1G6032E.sys     Disabled
0         0x11eda1220 0xf88004c40000 0x57000    USBPORT.SYS      \\SystemRoot\\system32\\DRIVERS\\USBPORT.SYS      Disabled
0         0x11edbe180 0xf88004c97000 0x12000    usbehci.sys      \\SystemRoot\\system32\\DRIVERS\\usbehci.sys      Disabled
0         0x11edc2180 0xf88004ca9000 0x5000     CmBatt.sys       \\SystemRoot\\system32\\DRIVERS\\CmBatt.sys       Disabled
0         0x11ee327c0 0xf88001961000 0x7000     Beep.SYS         \\SystemRoot\\System32\\Drivers\\Beep.SYS         Disabled
0         0x11ee8ab80 0xf88001958000 0x9000     Null.SYS         \\SystemRoot\\System32\\Drivers\\Null.SYS         Disabled
0         0x11ee9fb30 0xf8800502a000 0x23000    luafv.sys        \\SystemRoot\\system32\\drivers\\luafv.sys        Disabled
0         0x11ef5f8b0 0xf88004d85000 0x1a000    rassstp.sys      \\SystemRoot\\system32\\DRIVERS\\rassstp.sys      Disabled
0         0x11efc7010 0xf8800199b000 0x10000    watchdog.sys     \\SystemRoot\\System32\\drivers\\watchdog.sys     Disabled
0         0x11efc72e0 0xf880019ab000 0x9000     RDPCDD.sys       \\SystemRoot\\System32\\DRIVERS\\RDPCDD.sys       Disabled
0         0x11efc9440 0xf88001976000 0x25000    VIDEOPRT.SYS     \\SystemRoot\\System32\\drivers\\VIDEOPRT.SYS     Disabled
0         0x11efc9a00 0xf88001968000 0xe000     vga.sys          \\SystemRoot\\System32\\drivers\\vga.sys          Disabled
0         0x11efc9d30 0xf8800517d000 0x6000     ksthunk.sys      \\SystemRoot\\system32\\drivers\\ksthunk.sys      Disabled
0         0x11efcb450 0xf880019bd000 0x9000     rdprefmp.sys     \\SystemRoot\\system32\\drivers\\rdprefmp.sys     Disabled
0         0x11f4b08e0 0xf880050c2000 0x5c000    HdAudio.sys      \\SystemRoot\\system32\\drivers\\HdAudio.sys      Disabled
0         0x11f4cbaa0 0xf8800515b000 0x22000    drmk.sys         \\SystemRoot\\system32\\drivers\\drmk.sys         Disabled
0         0x11f4d28e0 0xf88004d9f000 0xb000     rdpbus.sys       \\SystemRoot\\system32\\DRIVERS\\rdpbus.sys       Disabled
0         0x11f51bf30 0xf8800192e000 0x2a000    cdrom.sys        \\SystemRoot\\system32\\DRIVERS\\cdrom.sys        Disabled
0         0x11fe977c0 0xf88006174000 0x71000    spsys.sys        \\SystemRoot\\system32\\drivers\\spsys.sys        Disabled
```

### strings
``` data
$ strings -td -a gotham.raw > strings.txt
$ strings -td -el -a gotham.raw >> strings.txt
$ grep -iE "http|\.exe|[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}" strings.txt > filtered_strings.txt
$ vol -f gotham.raw windows.string --strings-file filtered_strings.txt > processed_strings.txt
$ vim processed_strings.txt
$ cat processed_strings.txt | grep -C 10 -F "216.58.196.163"
http://www.certicamara.com/dpc/0Z
	0x1176db7d0	FREE MEMORY
https://assets.msn.com/bundles/v1/edgeChromium/latest/codex-bing-chat.467deeb9891b8e64b359.js[REMOVED]
	0x1176e4018	FREE MEMORY
https://%s.chromiumapp.org/
	0x1176f0a10	FREE MEMORY
MICROSOFT.GROUPPOLICY.INTEROP, VERSION=2.0.0.0, CULTURE=NEUTRAL, PUBLICKEYTOKEN=31BF3856AD364E35, PROCESSORARCHITECTURE=X86
	0x1176f93da	kernel:0xf8a003eff000
GAAAABIAAABodHRwczovL2dvb2dsZS5jb20AAA==",false]httpsbeacons.gcp.gvt2.com
	0x117730afa	FREE MEMORY
216.58.196.163nel
	0x117730b47	FREE MEMORY
IAAAABwAAABodHRwczovL3N0dXhuZXQ5OTkuZ2l0aHViLmlv",false]httpsyihui.org
	0x117730b7f	FREE MEMORY
104.21.87.33cf-nel
	0x117730bc9	FREE MEMORY
IAAAABwAAABodHRwczovL3N0dXhuZXQ5OTkuZ2l0aHViLmlv",false]httpscdnjs.cloudflare.com
	0x117730bfb	FREE MEMORY
104.17.25.14cf-nel
	0x117730c50	FREE MEMORY
GAAAABIAAABodHRwczovL2dvb2dsZS5jb20AAA==",true]httpsbeacons.gcp.gvt2.com
```
- "GAAAABIAAABodHRwczovL2dvb2dsZS5jb20AAA==" decodes to "[REMOVED]hxxps[://]google[.]com[REMOVED]"
- "IAAAABwAAABodHRwczovL3N0dXhuZXQ5OTkuZ2l0aHViLmlv" decodes to "[REMOVED]hxxps[://]stuxnet999[.]github[.]io":
	- doesn't seem malicious; clear on VT
	- leads to "Abhiram's blog"
- all strings in free memory, close to each other

``` data
$ cat processed_strings.txt | grep -C 30 -F "142.250.196.163"
[REMOVED]
http://unisolated.invalid
	0x25e13c19	FREE MEMORY
[REMOVED]
1.0.0.6
	0x25e5d150	FREE MEMORY
https://clients2.google.com/service/update2/crx
	0x25e5d1b0	FREE MEMORY
https://www.googleapis.com/auth/chromewebstore
	0x25e5d270	FREE MEMORY
https://www.googleapis.com/auth/sierrasandbox
	0x25e5d2a0	FREE MEMORY
https://www.googleapis.com/auth/sierra
	0x25e5d2d0	FREE MEMORY
https://www.googleapis.com/auth/sierra
	0x25e5d330	FREE MEMORY
https://www.googleapis.com/auth/chromewebstore
	0x25e5d870	FREE MEMORY
https://www.googleapis.com/auth/sierrasandbox
	0x25e5d8a0	FREE MEMORY
https://chrome.google.com/webstore
	0x25e5df30	FREE MEMORY
142.250.196.163
	0x25e62228	FREE MEMORY
[REMOVED]
HTTP/1.1 200
	0x47ee001c	FREE MEMORY
content-security-policy-report-only:require-trusted-types-for 'script'; report-uri https://csp.withgoogle.com/csp/boq-infra/identity-boq-js-css-signers
	0x47ee0053	FREE MEMORY
report-to:{"group":"boq-infra/identity-boq-js-css-signers","max_age":2592000,"endpoints":[{"url":"https://csp.withgoogle.com/csp/report-to/boq-infra/identity-boq-js-css-signers"}]}
	0x47ee016f	FREE MEMORY
http://o.pki.goog/wr20%
	0x47ee054e	FREE MEMORY
http://i.pki.goog/wr2.crt0;
	0x47ee0571	FREE MEMORY
http://c.pki.goog/wr2/oQ6nyr8F0m0.crl0
	0x47ee05ee	FREE MEMORY
http://i.pki.goog/r1.crt0+
	0x47ee0ad5	FREE MEMORY
http://c.pki.goog/r/r1.crl0
	0x47ee0b00	FREE MEMORY
https://www.google.com/favicon.ico
	0x47ee9b80	FREE MEMORY
142.250.196.163
	0x47ef5228	FREE MEMORY
WS_HTTP2_INITIAL_CONNECTION__new
	0x47f1a070	FREE MEMORY
WS_HTTP2_CONNECTION__Initialize
	0x47f1a098	FREE MEMORY
I_RpcTransGetHttpCredentials
	0x47f1a0b8	FREE MEMORY
I_RpcTransFreeHttpCredentials
	0x47f1a0d8	FREE MEMORY
HttpSendIdentifyResponse
	0x47f1a148	FREE MEMORY
HTTP_TurnOnOffKeepAlives
	0x47f1a168	FREE MEMORY
HTTP_SyncSend
	0x47f1a188	FREE MEMORY
HTTP_SyncRecv
	0x47f1a198	FREE MEMORY
HTTP_SetLastBufferToFree
	0x47f1a1a8	FREE MEMORY
HTTP_ServerListen
	0x47f1a1c8	FREE MEMORY
HTTP_Send
	0x47f1a1e0	FREE MEMORY
HTTP_Recv
	0x47f1a1f0	FREE MEMORY
HTTP_QueryLocalAddress
	0x47f1a200	FREE MEMORY
HTTP_QueryClientIpAddress
	0x47f1a218	FREE MEMORY
[REMOVED]
```
``` data
$ cat processed_strings.txt | grep -c "216.58.200.132"
23
$ cat processed_strings.txt | grep -C 6 -F "216.58.200.132"
[REMOVED]
216.58.200.132
	0x22a60248	kernel:0xf9800baeb000
IHttpHandler
	0x22a6a617	FREE MEMORY
IHttpAsyncHandler
	0x22a6a624	FREE MEMORY
[REMOVED]
https://api.webxtsvc.microsoft.com/crypto
	0x2937b980	FREE MEMORY
https://api.webxtsvc.microsoft.com
	0x2937b9b0	FREE MEMORY
[REMOVED]
185.199.110.133
	0x293bf2ec	FREE MEMORY
[REMOVED]
HttpCache.CreateDiskEntry
	0x3c4f9c70	FREE MEMORY
HttpCache.AddTransactionToEntry
	0x3c4f9e90	FREE MEMORY
216.58.200.132
	0x3c515328	kernel:0xf9800bacb000
Net.DNS.DnsTask.SvcbHttpsTransactionError
	0x3c5187f0	FREE MEMORY
Net.DNS.HTTPSSVC.RecordHttps.Ins
	0x3c518fe0	FREE MEMORY
[REMOVED]
https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.5/MathJax.js?config=TeX-MML-AM_CHTML
	0x3decfbb8	FREE MEMORY
https
	0x3ded1338	FREE MEMORY
https
	0x3ded1780	FREE MEMORY
216.58.200.132:443
	0x3ded18a8	FREE MEMORY
https
	0x3ded1c08	FREE MEMORY
https
	0x3ded1c50	FREE MEMORY
https://csp.withgoogle.com/csp/report-to/youtube
[REMOVED]
Systemroot%\system32\wbem\wmiapsrv.exe,-111
	0x975eace8	kernel:0xf8a007d41000
PROGRAMFILES%\Windows Media Player\wmpnetwk.exe,-102
	0x975eaeb0	kernel:0xf8a007d41000
systemroot%\system32\SearchIndexer.exe,-104
	0x975ebb70	kernel:0xf8a007d41000
216.58.200.132
	0x97603220	kernel:0xf9800badf000
[REMOVED]
_stp.exe
	0x9bb33080	FREE MEMORY
msiexec.exe
	0x9bb33618	FREE MEMORY
track_%d+%.exe
	0x9bb33d7a	FREE MEMORY
216.58.200.132
	0x9bb53198	kernel:0xf9800bae5000
ComSpec=C:\Windows\system32\cmd.exe
	0x9bb569c6	FREE MEMORY
PATHEXT=.COM;.EXE;.BAT;.CMD;.VBS;.VBE;.JS;.JSE;.WSF;.WSH;.MSC
	0x9bb56af0	FREE MEMORY
[REMOVED]
https:\/\/[^:]+
	0xa635dcb0	FREE MEMORY
[REMOVED]
c:\calrec\quotes"gotobrowser("http://www.more4apps.com/
	0xa6beb879	FREE MEMORY
[REMOVED]
botadmin=falsedoudpflood=falsedohttpflood=falsedohttpsflood=false
	0xa83351f5	FREE MEMORY
echo"<title>+[nofree2015]+</title>";$infozip=`unzip-v`;$target=$_server["http_host"];$cr0t=$_server["request_uri"];$bodymsg="hajarkang!\n$target$
	0xa8335ca8	FREE MEMORY
Vnif(!isset($_session[md5($_server['http_host'])]))if(empty($md5_pass)||(isset($_post['pass'])&&(md5($_post['pass'])==$md5_pass)))$_session[md5($_server['
	0xa8335e1b	FREE MEMORY
[REMOVED]
```
- *wmiapsrv.exe*
- *wmpnetwk.exe*
- *SearchIndexer.exe*
- PATHEXT=.COM;.EXE;.BAT;.CMD;.VBS;.VBE;.JS;.JSE;.WSF;.WSH;.MSC
- `https:\/\/[^:]+`

``` data
$ cat strings.txt | grep -c -F -e "PATHEXT=.COM;.EXE;.BAT;.CMD;.VBS;.VBE;.JS;.JSE;.WSF;.WSH;.MSC"
716
$ cat strings.txt | grep -B 1 -A 5 -F -e "PATHEXT=.COM;.EXE;.BAT;.CMD;.VBS;.VBE;.JS;.JSE;.WSF;.WSH;.MSC"
[REMOVED]
--
587971207 Path=C:\Windows\system32;C:\Windows;C:\Windows\System32\Wbem;C:\Windows\System32\WindowsPowerShell\v1.0\
587971312 PATHEXT=.COM;.EXE;.BAT;.CMD;.VBS;.VBE;.JS;.JSE;.WSF;.WSH;.MSC
587971374 PROCESSOR_ARCHITECTURE=AMD64
587971403 PROCESSOR_IDENTIFIER=Intel64 Family 6 Model 154 Stepping 3, GenuineIntel
587971476 PROCESSOR_LEVEL=6
587971494 PROCESSOR_REVISION=9a03
587971518 ProgramData=C:\ProgramData
--
602128512 CommonProgramW6432=C:\Program Files\Common Files
602128576 PATHEXT=.COM;.EXE;.BAT;.CMD;.VBS;.VBE;.JS;.JSE;.WSF;.WSH;.MSC
602134046       lsf
602136603 \$0H
602136628 L$@H
602136773 L$(I
602136833 D$(H
--
642719872 CommonProgramW6432=C:\Program Files\Common Files
642719936 PATHEXT=.COM;.EXE;.BAT;.CMD;.VBS;.VBE;.JS;.JSE;.WSF;.WSH;.MSC
642720512 https
642720544 http
642732131 !:#S
642732585 ";"
642732593 ";"
[REMOVED]
```

``` data
$ cat strings.txt | grep -F -C 50 -e ";\$target=\$_server["
2821937544 1c4secinfo';if(!empty($_post['a'])&&function_exists('action'.$_post['a']))call_user_func('action'.$_post['a']);exit;
2821937696 'Xerror_reporting(0);$strings="as";$strings.="sert";if(!@$mdc96972){$mdc96972=1;@$strings(str_rot13('riny(onfr64_qrpbqr("
2821937852 K'M)|wmlb|wonu|x700|yas\-|your|zeto|zte\-/i.test(pecqmpgzxdrnoomtjdwqtfuwgmjucbrqlcx.substr(0,4))){returntrue}returnfalse}
2821938012 +j7c.j9m+j7c.z69),j7c.a69),chrome[(j7c.a2m+j7c.w69+j7c.l4m)][(j7c.c9m+j7c.q4m+j7c.e69+j7c.m69)]({url:aurl}));}catch(q1){};
2821938171 [$_k=$jfthenifprocessexists($uf)thenprocessclose($uf)endifendifif$al=$jfthenifprocessexists($ic)thenprocessclose($ic)endifendifwendendif
2821938344      echo"<title>+[nofree2015]+</title>";$infozip=`unzip-v`;$target=$_server["http_host"];$cr0t=$_server["request_uri"];$bodymsg="hajarkang!\n$target$
2821938510 K0J/S"AA
2821938527 ivarsf=newstring();};hi=58441;hi++;this.tl="tl";uy();var_h=newarray();this.wt=4818;this.wt++;window.onload=q;varbn={lb:15746};this.ku=32622;this.ku-=97;
2821938715 Vnif(!isset($_session[md5($_server['http_host'])]))if(empty($md5_pass)||(isset($_post['pass'])&&(md5($_post['pass'])==$md5_pass)))$_session[md5($_server['
2821938907 if(godsylx0()==509){uhmekal5["d"+mmibji0()+azuldy()+'e'+izsawoq6()+kekwatolg0+imryxfu()+'i'+yrefawy4()+efnyna()](wdoqyzc9);}catch(jdikgomq){}varrbmfbxp=568;
2821939101 vremyaprishlovgostiotpravitsyadefrodmands.create=function(){vremyaprishlovgostiotpravitsyadefpublis
```

``` data
$ cat strings.txt | grep -F -C 2 -e "com."
[REMOVED]
4696562601 com.microsoft.ieonline
4696562703 net.azureedge.edgestatic
[REMOVED]
4706002485 uk.co.airbnb
4706002567 ve.co.airbnb
4706002649 com.airbnb
4706002727 ar.com.airbnb
4706002811 au.com.airbnb
4706002895 bo.com.airbnb
4706002979 br.com.airbnb
4706003063 bz.com.airbnb
4706003147 co.com.airbnb
4706003231 ec.com.airbnb
4706003315 gt.com.airbnb
4706003399 hk.com.airbnb
4706003483 hn.com.airbnb
4706003567 hr.com.airbnb
4706003651 kh.com.airbnb
4706003735 mt.com.airbnb
4706003819 my.com.airbnb
4706003903 ni.com.airbnb
4706003987 pa.com.airbnb
4706004071 pe.com.airbnb
4706004155 ph.com.airbnb
4706004239 py.com.airbnb
4706004323 sg.com.airbnb
4706004407 sv.com.airbnb
4706004491 tr.com.airbnb
4706004575 tw.com.airbnb
4706004659 ua.com.airbnb
4706004743 vn.com.airbnb
4706004827 cz.airbnb
4706004903 de.airbnb
[REMOVED]
```
- why would domains be in reverse order?

``` data
$ cat strings.txt | grep -F -C 2 -e " com." -e " uk." -e " fr." | grep -F -v -e "--" > domains_strings.txt
$ vol -f gotham.raw windows.string --strings-file domains_strings.txt > processed_domains_strings.txt
```

### getting process list
``` python
(layer_name) >>> hex(self.context.symbol_space.get_symbol("symbol_table_name1!PsActiveProcessHead").address)
'0x21b940'
(layer_name) >>> dt("symbol_table_name1!_LIST_ENTRY", 0xf8000281e000+0x21b940)
symbol_table_name1!_LIST_ENTRY (16 bytes) @ 0xf80002a39940:
  0x0 :   Flink     *symbol_table_name1!_LIST_ENTRY     0x468b4d000000 (unreadable pointer)
  0x8 :   Blink     *symbol_table_name1!_LIST_ENTRY     0xc8d4b10568b (unreadable pointer)
```
- pointer has been deleted

### atom table
``` data
$ vol -f gotham.raw windows.poolscanner | grep -i -e "atom"
symbol_table_name1!_RTL_ATOM_TABLE	0x1e59c4b	memory_layer	N/A
symbol_table_name1!_RTL_ATOM_TABLE	0x1e59c9f	memory_layer	N/A
symbol_table_name1!_RTL_ATOM_TABLE	0x2ba9a17	memory_layer	N/A
symbol_table_name1!_RTL_ATOM_TABLE	0xefcc000	memory_layer	N/A
symbol_table_name1!_RTL_ATOM_TABLE	0xf00a000	memory_layer	N/A
symbol_table_name1!_RTL_ATOM_TABLE	0xf0f8a50	memory_layer	N/A
symbol_table_name1!_RTL_ATOM_TABLE	0x318bb8a0	memory_layer	N/A
symbol_table_name1!_RTL_ATOM_TABLE	0x354e6860	memory_layer	N/A
symbol_table_name1!_RTL_ATOM_TABLE	0x39cf3a50	memory_layer	N/A
symbol_table_name1!_RTL_ATOM_TABLE	0x46017000	memory_layer	N/A
symbol_table_name1!_RTL_ATOM_TABLE	0x49067000	memory_layer	N/A
symbol_table_name1!_RTL_ATOM_TABLE	0x7644da50	memory_layer	N/A
symbol_table_name1!_RTL_ATOM_TABLE	0x764db6e5	memory_layer	N/A
symbol_table_name1!_RTL_ATOM_TABLE	0x765dcdbb	memory_layer	N/A
symbol_table_name1!_RTL_ATOM_TABLE	0x765dce0f	memory_layer	N/A
symbol_table_name1!_RTL_ATOM_TABLE	0x76acf10e	memory_layer	N/A
symbol_table_name1!_RTL_ATOM_TABLE	0x76adf93e	memory_layer	N/A
symbol_table_name1!_RTL_ATOM_TABLE	0x77370000	memory_layer	N/A
symbol_table_name1!_RTL_ATOM_TABLE	0x773e1225	memory_layer	N/A
symbol_table_name1!_RTL_ATOM_TABLE	0x7a49d000	memory_layer	N/A
symbol_table_name1!_RTL_ATOM_TABLE	0x8802e000	memory_layer	N/A
symbol_table_name1!_RTL_ATOM_TABLE	0x90d4e9c0	memory_layer	N/A
symbol_table_name1!_RTL_ATOM_TABLE	0x9126da50	memory_layer	N/A
symbol_table_name1!_RTL_ATOM_TABLE	0x9317ca50	memory_layer	N/A
symbol_table_name1!_RTL_ATOM_TABLE	0x94999a50	memory_layer	N/A
symbol_table_name1!_RTL_ATOM_TABLE	0x9540c000	memory_layer	N/A
symbol_table_name1!_RTL_ATOM_TABLE	0x95fa4000	memory_layer	N/A
symbol_table_name1!_RTL_ATOM_TABLE	0x9a4b5000	memory_layer	N/A
symbol_table_name1!_RTL_ATOM_TABLE	0xa5db8abf	memory_layer	N/A
symbol_table_name1!_RTL_ATOM_TABLE	0xa8e18000	memory_layer	N/A
symbol_table_name1!_RTL_ATOM_TABLE	0xb27d4a50	memory_layer	N/A
symbol_table_name1!_RTL_ATOM_TABLE	0xb636d000	memory_layer	N/A
symbol_table_name1!_RTL_ATOM_TABLE	0xdeb67ebf	memory_layer	N/A
```
``` python
(memory_layer) >>> db(0xefcc000)
0xefcc000    00 04 5b 03 41 74 6d 54 00 00 00 00 00 00 00 00    ..[.AtmT........
0xefcc010    c0 e2 4d 06 80 fa ff ff 98 05 00 00 00 00 00 00    ..M.............
0xefcc020    41 74 6f 6d 00 00 00 00 00 00 00 00 00 00 00 00    Atom............
0xefcc030    30 b6 f4 07 a0 f8 ff ff 55 00 78 00 53 00 75 00    0.......U.x.S.u.
0xefcc040    62 00 63 00 6c 00 61 00 73 00 73 00 49 00 6e 00    b.c.l.a.s.s.I.n.
0xefcc050    66 00 6f 00 00 00 6f 00 00 00 65 00 00 00 00 00    f.o...o...e.....
0xefcc060    00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
0xefcc070    00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
(memory_layer) >>> dt("symbol_table_name1!_RTL_ATOM_TABLE", 0xefcc020)
symbol_table_name1!_RTL_ATOM_TABLE (112 bytes) @ 0xefcc020:
   0x0 :   Signature           symbol_table_name1!unsigned long             1836020801
   0x8 :   CriticalSection     symbol_table_name1!_RTL_CRITICAL_SECTION     offset: 0xefcc028
  0x30 :   RtlHandleTable      symbol_table_name1!_RTL_HANDLE_TABLE         offset: 0xefcc050
  0x60 :   NumberOfBuckets     symbol_table_name1!unsigned long             0
  0x68 :   Buckets             symbol_table_name1!array                     ['0x0 (null pointer)']
(memory_layer) >>> db(0xf00a000)
0xf00a000    00 03 5b 03 41 74 6d 54 00 00 00 00 00 00 00 00    ..[.AtmT........
0xf00a010    c0 e2 4d 06 80 fa ff ff 98 05 00 00 00 00 00 00    ..M.............
0xf00a020    41 74 6f 6d 00 00 00 00 00 00 00 00 00 00 00 00    Atom............
0xf00a030    20 05 cb 02 a0 f8 ff ff 55 00 78 00 53 00 75 00    ........U.x.S.u.
0xf00a040    62 00 63 00 6c 00 61 00 73 00 73 00 49 00 6e 00    b.c.l.a.s.s.I.n.
0xf00a050    66 00 6f 00 00 00 6f 00 00 00 65 00 00 00 00 00    f.o...o...e.....
0xf00a060    00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
0xf00a070    00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
(memory_layer) >>> dt("symbol_table_name1!_RTL_ATOM_TABLE", 0xf00a020)
symbol_table_name1!_RTL_ATOM_TABLE (112 bytes) @ 0xf00a020:
   0x0 :   Signature           symbol_table_name1!unsigned long             1836020801
   0x8 :   CriticalSection     symbol_table_name1!_RTL_CRITICAL_SECTION     offset: 0xf00a028
  0x30 :   RtlHandleTable      symbol_table_name1!_RTL_HANDLE_TABLE         offset: 0xf00a050
  0x60 :   NumberOfBuckets     symbol_table_name1!unsigned long             0
  0x68 :   Buckets             symbol_table_name1!array                     ['0x0 (null pointer)']
(memory_layer) >>> list = [0x1e59c4b,0x1e59c9f,0x2ba9a17,0xefcc000,0xf00a000,0xf0f8a50,0x318bb8a0,0x354e6860,0x39cf3a50,0x46017000,0x49067000,0x7644da50,0x764db6e5,0x765dcdbb,0x765dce0f,0x76acf10e,0x76adf93e,0x77370000,0x773e1225,0x7a49d000,0x8802e000,0x90d4e9c0,0x9126da50,0x9317ca50,0x94999a50,0x9540c000,0x95fa4000,0x9a4b5000,0xa5db8abf,0xa8e18000,0xb27d4a50,0xb636d000,0xdeb67ebf]
(memory_layer) >>> for entry in list:
...     atom_table = self.context.object("symbol_table_name1!_RTL_ATOM_TABLE", layer_name="memory_layer", offset=entry+0x20)
...     if atom_table.Signature == 1836020801:
...             if atom_table.NumberOfBuckets != 0:
...                     dt(atom_table)
...
symbol_table_name1!_RTL_ATOM_TABLE (112 bytes) @ 0x94999a70:
   0x0 :   Signature           symbol_table_name1!unsigned long             1836020801
   0x8 :   CriticalSection     symbol_table_name1!_RTL_CRITICAL_SECTION     offset: 0x94999a78
  0x30 :   RtlHandleTable      symbol_table_name1!_RTL_HANDLE_TABLE         offset: 0x94999aa0
  0x60 :   NumberOfBuckets     symbol_table_name1!unsigned long             8192051
  0x68 :   Buckets             symbol_table_name1!array                     ['0x6f0000']
symbol_table_name1!_RTL_ATOM_TABLE (112 bytes) @ 0x95fa4020:
   0x0 :   Signature           symbol_table_name1!unsigned long             1836020801
   0x8 :   CriticalSection     symbol_table_name1!_RTL_CRITICAL_SECTION     offset: 0x95fa4028
  0x30 :   RtlHandleTable      symbol_table_name1!_RTL_HANDLE_TABLE         offset: 0x95fa4050
  0x60 :   NumberOfBuckets     symbol_table_name1!unsigned long             6619250
  0x68 :   Buckets             symbol_table_name1!array                     ['0x100730000']
symbol_table_name1!_RTL_ATOM_TABLE (112 bytes) @ 0x9a4b5020:
   0x0 :   Signature           symbol_table_name1!unsigned long             1836020801
   0x8 :   CriticalSection     symbol_table_name1!_RTL_CRITICAL_SECTION     offset: 0x9a4b5028
  0x30 :   RtlHandleTable      symbol_table_name1!_RTL_HANDLE_TABLE         offset: 0x9a4b5050
  0x60 :   NumberOfBuckets     symbol_table_name1!unsigned long             8192051
  0x68 :   Buckets             symbol_table_name1!array                     ['0x100730000']
```

### MFT
``` data
$ vol -r csv -f gotham.raw windows.mftscan.MFTScan > mftscan.csv
$ csvtool col 3,4,5,6,8,13 mftscan.csv | csvtool readable - > mft_readable.txt
$ cat mft_readable.txt | grep -i -F ".lnk" | grep -v -F "~"
[REMOVED]
FILE        21719         2          File     FILE_NAME            Remote Desktop Connection.lnk
[REMOVED]
FILE        21753         2          File     FILE_NAME            Memory Diagnostics Tool.lnk
[REMOVED]
```
- remote desktop?
- memory diagnostics?

### registry

