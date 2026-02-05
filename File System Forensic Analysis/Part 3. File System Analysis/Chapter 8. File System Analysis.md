## 1. file system
- consists of structural and user data
- independent from specific computer

## 2. data categories
- file system:
	- general file system information
	- tells where to find certain data structures and how big a data unit is; can be thought of as map for file system
- content:
	- data that make up file
	- typically organized in collection of standard-sized containers; data units
- metadata:
	- data that describe file
	- may not contain name of file
	- examples:
		- FAT directory entries
		- NTFS *Master File Table(MFT)* entries
		- UFS and Ext3 inode structures
- file name:
	- aka human interface category
	- data that assigns a name to each file
	- typically located in contents of directory; list of file names with corresponding metadata address
	- similar to host names in network; helps people locate file metadata without remembering actual address
- application:
	- contains data that provides special features
	- more efficient to implement in file system instead of file
	- examples:
		- user quota stats
		- file system journal
	- not needed to read/write file; more easily forged

![[data category interaction.png]]

## 3. essential and non-essential data
- essential data is necessary to retrieve file; trusted
	- file content address
	- name of file
	- pointer to metadata
- non-essential data is not necessary and not trusted:
	- access time
	- permissions

## 4. file system category analysis techniques
- necessary to find location of data structures in other categories
- typically independent values; not much can be done
- data structures here frequently have unused values; easy to hide data
- compare size of file system with size of volume to identify *volume slack*
- TSK has a tool called *fsstat* that displays file system category data 

## 5. content category
- analysis is conducted to recover deleted data or conduct low-level searches

## 6. logical file system address
- file systems use logical volume address, but also assign *logical file system addresses* 
![[logical file system address demo.png]]
	- each data unit contains 2 sectors
	- sector 16 is volume slack

## 7. allocation strategies
- first available:
	- assigns first empty data unit from start of file system
	- fragmentation is common
	- quicker algorithm
	- deleted content is easier to recover towards end of file system
- next available:
	- assigns first empty data unit from previous allocated data unit
	- more balanced allocation of data units
- best fit:
	- searches for consecutive data units big enough
	- causes fragmentation for files that grow in size
- OS dependent; OS chooses strategy
- application dependent; some applications update file, some delete and create new file 
*must be differentiated from allocation strategy behavior*

## 8. damaged data unit
- easy to mark data unit as damaged and hide data
- bad sector report from acquisition tools can be compared with damaged list to identify manually marked data units

## 9. content category analysis technique
- data unit viewing:
	- used when investigator knows the address of evidence
	- many hex editors and investigation tools can perform this function
	- *dcat* tool in TSK can perform this function
- logical file system-level searching:
	- used when what content evidence is, but not where
	- logical file system search checks each data unit for specific phrase of value
	- does not work if value is split between non-consecutive data units
- data unit allocation status:
	- used if data is unallocated
	- unallocated data definition varies between tools and should be checked
	- *dls* in TSK can be used to extract unallocated data to a file; *dcalc* tool can be used to determine which data unit of the file system it was in
- data unit allocation order:
	- allocation stratgies of OS can be used to determine relative allocation order of data units; very difficult
- consistency checks:
	- allow us to determine if file system is in suspicious state
	- done by verifying each allocated data unit has exactly one allocated metadata entry pointing to it
	- allocated data units without corresponding metadata structures are called *orphan data units*
	- done by checking data units listed as damaged; damaged data units should be zero
- content category wiping techniques:
	- third party wiping tools rely on OS to behave certain way; may not work as intended
	- detection of wiping tools is difficult in content category

## 10. metadata category
- analysis used to gather details or search for files that meet a criteria
- has more non-essential data than other categories

## 11. logical file address
- *logical file address* is relative to start of file
![[logical file address demo.png]]

## 12. slack space
- *slack space* occurs when file size is not multiple of data unit size; refers to unused bytes in last data unit
- slack space can contain data from previous files
- slack space area:
	- end of file ~ end of sector containing file; usually filled with 0s
	- remaining sectors in data unit containing file; some OS do not wipe, leaving data from previous files
- all file systems have slack space
- important to note slack space is considered allocated

## 13. metadata based file recovery
- works when metadata from deleted file still exists; if wiped or reallocated to new file use application-based method
- metadata and data units can become out of sync; exercise caution
- if multiple metadata entries point to same address, the more recent entry has to be determined:
	- use time information from each entry; not reliable
	- compare file type in entry and at address

- sync issue example
![[metadata entry sync issues.png]]
	- 2 unallocated entries point to data unit 1000 but it actually belongs to entry 300
	- no way to know data corresponds to file created after entry was unallocated

## 14. compressed and sparse files
- compression can occur at 3 levels:
	- highest level: data inside file format is compressed
		- e.g. JPEG
	- medium level: external program compresses entire file to create new file
	- lowest level: file system compresses data
		- application does not know file is being compressed
- file system compression techniques:
	- same technique as for files
	- don't allocated a physical data unit for all 0s; *sparse file*
	- *Unix File System(UFS)* implements by writing 0s to address of block

## 15. encrypted files
- encryption methods:
	- application layer encryption
	- external application encryption
	- OS encryption
		- non-content data is typically not encrypted
	- volume encryption
- if only select files/directories are encrypted, unencrypted copies may be found in temporary files or unallocated space

## 16. metedata category analysis techniques
- metadata lookup:
	- view metadata of suspicious file name
	- procedures are file system dependent
	- *istat* tool in TSK shows values from metadata data structure
- logical file viewing:
	- find file via metadata and view contents
	- keep slack space in mind; slack space = size of file % size of data unit
	- *icat* tool in TSK allows viewing of content allocated to metadata structure
		- -r flag for recovery attempt
		- -s flag to view slack space
- logical file searching:
	- used to find file based on content
	- different from logical file system search in that data units are searched in order of file use, not order on volume
	- logical volume search for same value should be conducted on unallocated data
	- **some tools do not search slack space**
- unallocated metadata analysis:
	- it is possible name of deleted file is reused before metadata structure is; metadata entry is hidden because there is no name for it
	- *ils* tool of TSK can list unallocated data structures
- metadata attribute searching and sorting:
	- it is common to search based on metadata such as time slot and user
	- file times can be easily changed; but can provide clues
	- temporal data can also give hints on how the computer was used
	- *mactime* tool of TSK can be used to make a timeline of file activity
	![[mactime output.png]]
		- first column has date stamp
		- second column has file size
		- third column has modification(m) time, access(a) time, and change(c) time
		- last column has file name
	- some time stamps are in UTC; requires knowing timezone offset of computer for actual time value
	- when investigating certain user, owner ID or files they have write access to can be examined
	- searching for data units addressed containing interesting data may show file that allocated said data unit; other data units for same file should be examined
	![[metadata structure parsing for file.png]]
		- *ifind* tool TSK can do this
- data structure allocation order:
	- allocation strategy of OS can be used to determine relative allocation times of two entries; very difficult
- consistency checks:
	- may reveal attempts to hide data or internal error
	- can only draw conclusions about essential data
	- examine each allocated entry and verify data units allocated to it are also allocated
	- verify number of data units allocated matches size of file
	- verify entries for special file types do not have data units allocated to them
	- verify allocated entry has an allocated name that points to it
	- checks for non-essential data such as date can be performed, but are not reliable
- wiping technique:
	- an intelligent wiping tool could fill values with valid data with no relation to original data
	- a more extreme version would shift maining entries so there are no unused entries

## 17. file name category
