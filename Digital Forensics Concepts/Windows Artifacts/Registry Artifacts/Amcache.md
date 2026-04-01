## 1. Amcache
- managed by OS
- no known way to remove/modify Amcache data<sup>[1]</sup>
- allows searching for patterns, e.g. file names, paths
- stores SHA-1 hashes of executed files

## 2. artifact location
- `C:\Windows\AppCompat\Programs\Amcache.hve`
	- additional log files exist:
		- `C:\Windows\AppCompat\Programs\Amcache.hve.*LOG1`
		- `C:\Windows\AppCompat\Programs\Amcache.hve.*LOG2`

## 3. limitations
- computes SHA1 hash over only first 31,457,280 bytes
- records:<sup>[1]></sup>
	- files in directories scanned by MS compatibility appraiser
	- executables and drives copied during program execution
	- GUI applications that required compatibility shimming

## 4. collection tools
- Aralez
- Velociraptor
- Kape

## 5. analysis tools
- Registry Explorer

## 6. Amcache.hve structure
- registry hive in REGF format
- contains multiple subkeys that store distinct classes of data
- important keys:
	- InventoryApplicationFile
	- InventoryApplication
	- InverntoryDriverBinary
	- InventoryAppliationShortcut

## 7. InventoryApplicationFile
- essential for tracking executable discovered on system
- each executable represented by unique subkey containing:
	- ProgramId
	- FileID: SHA-1 hash of file
	- LowerCaseLongPath
	- Name
	- OriginalFileName: file name specified in PE header's version resource
	- Publisher: usually empty for malware
	- Version
	- BinaryType
	- ProductName
	- LinkDate
	- Size
	- IsOsComponent

## 8. InventoryApplication
- records details about previously installed applications
- entry named by unique ProgramId
- important subkeys:
	- InstallDate
	- MsiInstallDate:
		- present when installed via Windows Installer
		- show time MSI package was applied
	- UninstallString: command line used to uninstall
	- Language
	- Publisher
	- ManifestPath

## 9. InventoryDriverBinary
- records kernel-mode driver system loaded
- important subkeys:
	- FileID
	- LowerCaseLongPath
	- DigitalSignature
	- LastModified
- pair with `InventoryApplicationDriver` which keeps track of drivers installed by specific applications

## 10. InventoryApplicationShortcut
- contains entries for lnk files present in directories like user start menu or desktop
- important subkeys:
	- ShortcutTargetPath
	- ShortcutProgramId

## reference
[1] (Cristian Souza, 2025/10/01) Forensic journey: hunting evil within AmCache, 2026/03/12, https://securelist.com/amcache-forensic-artifact/117622/