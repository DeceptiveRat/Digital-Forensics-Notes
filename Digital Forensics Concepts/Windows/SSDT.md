## 1. *System Service Dispatch Table(SSDT)*
- instance of `SYSTEM_SERVICE_TABLE` inside `SERVICE_DESCRIPTOR_TABLE`

## 2. *System Descriptor Table(SDT)*
- kernel structure containing 4 *System Service Table(SST)* entries
- 2 exist on system,  `nt!KeServiceDescriptorTable` and `nt!KeServiceDescriptorTableShadow` 
- `nt!KeServiceDescriptorTable` uses only 1 available SST entry, which describes SSDT for Windows Native APIs exported by *ntoskrnl.exe*
- `nt!KeServiceDescriptorTableShadow` uses 2 SST entries, copy of `nt!KiServiceTable` from `nt!KeServiceDescriptorTable` and `win32k!W32pServiceTable`, which describes SSDT for user and GDI routines exported by *win32k.sys*

## reference
[1] The Quest for the SSDTs, 2026/04/12, http://www.atelierweb.com/index.php/the-quest-for-the-ssdts/