## 1. character encoding
- ASCII: one byte for each character
- UTF-32: 4 bytes for each character
- UTF-16: 2 bytes for heavily used characters, 4 bytes for other characters
- UTF-8: uses 1, 2, or 4 bytes for each character

## 2. boot process
- power is supplied to CPU 
- CPU reads instructions from specific location in memory; typically ROM
- instructions in ROM forces system to probe for and configure hardware
- CPU searches for a device that may contain additional boot code
- boot code locates and loads specific OS

## 3. Windows boot process
- system powers on
- CPU reads instructions from *Basic Input / Output System* (BIOS)
- searches for hard disks, CD drives, and other hardware to support
- BIOS searches first sector of floppy disks, hard disks, and CDs for boot code
- code in first sector of bootable disk causes CPU to process partition table and locate bootable partition where OS is located
- in first sector of partition is  boot code that locates and loads OS
![[boot code relationships.png]]

## 4. hard disk internals
- circular platters stacked on top of each other
- top and bottom of platter is coated with magnetic media
- head moves back and forth to read and write data
- each track is given address starting with 0 from the outside
- tracks are divided into *sectors*, the smallest addressable storage unit, typically 512 bytes
- heads are given addresses
- a specific sector can be selected with cylinder address, head address, and sector address
- CHS address is no longer used and *Logical Block Address* (LBA) is used instead
![[harddisk internal.png]]

## 5. *AT Attachment* (ATA)/*Integrated Disk Electronics* (IDE) interface
- controller issues commands to one or two ATA disks using ribbon cable
- interface data path between controller and disks is called *channel*

## 6. physical addressing method
- disk geometry and CHS method
	- smallest size for each value is used, limiting disk size
	- BIOS can translate addresses to physical address, but size is still limited
- LBA 
	- uses single number to address each sector
- conversion formula:
```math
LBA = (((CYLINDER * heads_per_cylinder) + HEAD) * sectors_per_track) + SECTOR - 1
```

## 7. disk commands
- commands are issued to both disks on cable
- part of the command is used to identify which disk to target
- controller communicates by writing to disk registers
- command is executed only when command register is written to

## 8. hard disk password
- user password and master password exists
- high security mode: both passwords can unlock disk
- maximum security mode: master password can unlock disk after disk has been wiped

## 9. *host protected area* (HPA)
- special area of disk which casual observer can't see
- size is configurable using ATA commands
- located at end of disk
- accessible by reconfiguring hard disk
- using commands:
	- READ_NATIVE_MAX_ADDRESS: returns maximum physical address
	- IDENTIFY_DEVICE: returns number of sectors a user can access; used to detect HPA
	- SET_MAX_ADDRESS: set max address user has access to, creating/removing HPA
		- has volatility bit for temporary change

![[HPA.png]]

## 10. *Device Configuration Overlay* (DCO)
- allows apparent capability of hard disk to be limited
- e.g. cause IDENTIFY_DEVICE command to show supported features as not supported and show smaller disk size
- DCO commands:
	- DEVICE_CONFIGURATION_IDENTIFY: returns actual features and size of disk; used to detect DCO
	- DEVICE_CONFIGURATION_SET: create or change DCO
		- no volatility bit
	- DEVICE_CONFIGURATION_RESET: remove DCO

![[DCO demo.png]]

## 11. accessing disk via controller
- software needs to know how to issue commands to controller

## 12. accessing disk via BIOS
- software must load data, such as sector address and sizes into CPU registers and execute software interrupt command INT13h

## 13. *Small Computer Systems Interface* (SCSI)
- has numerous connector types
- each device on SCSI cable needs a unique numerical ID
- many disks have a jumper to make disk read only
- doesn't have a controller
- does not suffer from size limitations like ATA disks
