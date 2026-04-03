## 1. event logs
- contain details about:
	- application errors
	- interactive/remote logins
	- changes in firewall policy
- can give timeframe to focus efforts on
- many log files are mapped into memory during run time
- sometimes can even extract entries marked for deletion

## 2. event logs analysis objectives
- locate event logs in memory
- process event logs
- detect brute force logins
- identify backdoors: changes to firewall policy and server port conflicts can indicate backdoor activity
- identify cleared event logs

## 3. event logs DS
- file header `EVTLogHeader`:
	``` python
	>>> dt("EVTLogHeader")
	'EVTLogHeader' (48 bytes)
	0x0 : HeaderSize ['unsigned int']
	0x4 : Magic ['int']
	0x10 : OffsetOldest ['unsigned int']
	0x14 : OffsetNextToWrite ['unsigned int']
	0x18 : NextID ['int']
	0x1c : OldestID ['int']
	0x20 : MaxSize ['unsigned int']
	0x28 : RetentionTime ['int']
	0x2c : RecordSize ['unsigned int']
	```
	- `OldestID`: ID of oldest record
	- `Magic`: `LfLe`
	- `NextID`: ID of next record that will be written
- record `EVTRecordStruct`:
	``` python
	>>> dt("EVTRecordStruct")
	'EVTRecordStruct' (56 bytes)
	0x0 : RecordLength ['unsigned int']
	0x4 : Magic ['int']
	0x8 : RecordNumber ['int']
	0xc : TimeGenerated ['UnixTimeStamp', {'is_utc': True}]
	0x10 : TimeWritten ['UnixTimeStamp', {'is_utc': True}]
	0x14 : EventID ['unsigned short']
	0x18 : EventType ['Enumeration', [snip]
	0x1a : NumStrings ['unsigned short']
	[snip]
	```
	- `Magic`: `LfLe`
	- `RecordNumber`: ID of record 
	- `NumStrings`: number of messages included in record that help describe event

## 4. Windows 2000, XP, and 2003 event logs
[skipped]

## 5. Windows Vista, 2008, and 7 event logs
- aka. *Evtx*
- contained in XML binary format
- at `%systemroot%\system32\winevt\Logs`
- extract logs from memory using `dumpfiles`:
	``` sh
	$ python vol.py –f Win7SP1x86.vmem --profile=Win7SP1x86 dumpfiles --regex .evtx$ --ignore-case --dump-dir output
	Volatility Foundation Volatility Framework 2.4
	DataSectionObject 0x8509eba8 756 \Device\HarddiskVolume1\Windows\System32\
	winevt\Logs\Microsoft-Windows-Diagnostics-Performance%4Operational.evtx
	SharedCacheMap 0x8509eba8 756 \Device\HarddiskVolume1\Windows\System32\
	winevt\Logs\Microsoft-Windows-Diagnostics-Performance%4Operational.evtx
	DataSectionObject 0x83eaec48 756 \Device\HarddiskVolume1\Windows\System32\
	winevt\Logs\Microsoft-Windows-Kernel-WHEA%4Errors.evtx
	SharedCacheMap 0x83eaec48 756 \Device\HarddiskVolume1\Windows\System32\
	winevt\Logs\Microsoft-Windows-Kernel-WHEA%4Errors.evtx
	DataSectionObject 0x845bcab0 756
	[snip]
	```
