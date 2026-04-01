## 1. Registry
- collection of databases that contain system configuration data
- useful data contained:
	- recently used files
	- recently used programs
	- devices connected to system
- consists of keys and values

## 2. hive
- aka. root key
- group of keys, subkeys, and values stored in single file
- 5 root keys:
	- *HKEY_CURRENT_USER*
	- *HKEY_USERS*
	- *HKEY_LOCAL_MACHINE*
	- *HKEY_CLASSES_ROOT*
	- *HKEY_CURRENT_CONFIG*

## 3. *HKEY_CURRENT_USER*
- contains root configuration information for user who is logged on
- controls user-level settings:
	- installed printers
	- desktop wallpaper
	- display settings
	- environment variables
	- keyboard layout
	- mapped network drives
- subkey of *HKEY_USERS*
- notable subkeys:
	- *Software*
	- *Control Panel*
	- *Environment*
	- *Software\Microsoft\Windows\CurrentVersion*
- search for:
	- suspicious applications in *Software\Microsoft\Windows\CurrentVersion\Run*

## 4. *HKEY_USERS*
- contains all actively loaded user profiles on computer
- search for:
	- new users
	- unusual environment variables

## 5. *HKEY_LOCAL_MACHINE*
- aka. HKLM
- configuration of computer for any user
- notable subkeys:
	- *Software*
	- *SYSTEM\CurrentControlSet\Services*
	- *HARDWARE\Description\System*
	- *SOFTWARE\Microsoft\Windows\CurrentVersion*
- search for:
	- suspicious users in *SAM\Domains\Account\Users*
	- suspicious services in *System\CurrentControlSet\Services* 
	- suspicious applications in *Software\Microsoft\Windows\CurrentVersion\Run*
	- disabled/enabled unusual policies in *Software\Microsoft\Windows\CurrentVersion\Policies*
	- unusual entries in *Software\Microsoft\Windows NT\CurrentVersion\Winlogon*
	- unusual values in *SYSTEM\CurrentControlSet\Services\Tcpip\Parameters*

## 6. *HKEY_CLASSES_ROOT*
- aka. HKCR
- ensures correct program opens when opening via `Windows Explorer`
- contains:
	- file extension association information 
	- *Program Identifier(ProgID)*
	- *Class Identifier(CLSID)*
	- *Interface Identifier(IID)*
- notable subkeys:
	- *.exe*
	- *.txt*
	- *.jpg, .png, .gif, etc*
	- *.html, .htm*
	- *Directory*: contains settings related to directories
- search for:
	- suspicious programs associated with file types

## 7. *HKEY_CURRENT_CONFIG*
- contains information about hardware profile that is used by local computer at startup
- search for:
	- unusual values in *System\CurrentControlSet\Services\Tcpip\Parameters*

## reference
[1] (2023/03/05, Ziya DENIZ) Registry Forensic Analysis, 2026/04/01, https://medium.com/@dziyaaa/registry-forensic-analysis-317192c9cf59
[2] (2023/10/29, snoballz) The Heart of Windows: Exploring the Registry, 2026/04/01, https://medium.com/@snoballz_909/the-heart-of-windows-exploring-the-registry-1e9f89dd6274
