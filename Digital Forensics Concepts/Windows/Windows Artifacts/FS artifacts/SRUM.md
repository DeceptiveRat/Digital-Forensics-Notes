## 1. *System Resource Utilization Monitor(SRUM)*
- intended to track application usage, network utilization, and system energy state
- stored in `C:\Windows\System32\sru\SRUDB.dat`
- artifact categories:
	- application resource usage
	- network connections
	- energy usage
	- network usage
	- energy usage (long term)
	- push notification data

## 2. application resource usage
- tracks every *.exe* executed on system
- store full path file executed from 
- stores SID that executed binary 

## 3. energy usage
- captures statistics related to charge and powerstate

## 4. network connections
- utilized to identify when asset was connected to network
- captures connection start time, interface type, and duration of connection

## 5. network usage
- tracks connections and network SSID (if wireless)
- captures bandwidth usage in bytes sent/received
- include full path of application and SID that executed it

## reference
[1] (2022/10/05) SRUM: Forensic Analysis of Windows System Resource Utilization Monitor, 2026/06/04, https://www.magnetforensics.com/blog/srum-forensic-analysis-of-windows-system-resource-utilization-monitor/