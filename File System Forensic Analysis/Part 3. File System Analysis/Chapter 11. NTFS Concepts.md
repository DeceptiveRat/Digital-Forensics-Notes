## 1. *New Technolgies File System(NTFS)*
- default FS for Windows NT, 2000, XP, and server
- more complex than FAT
- scalability is provided by use of generic DS that wrap around DS with content; internal DS can change to meet new demands without changing generic DS
- there is no official documentation
- one dominant application creates FS; difficult to differentiate application specific properties and general properties of FS

## 2. NTFS everything is a file
- important data are allocated to files
- NTFS FS does not have a specific layout like other FS
- only consistent layout is first sectors of volume contain boot sector and boot code

## 3. *Master File Table(MFT)* concepts
- has entry for itself, $MFT
- location given in boot sector

	![[MFT layout.png]]
- contains information about all files/directories
- every file/directory has at least one entry
- entries are given addresses based on location on table
- entry size is usually 1,024 bytes; exact size given in boot sector
- dynamic nature makes it easy to expand FS
- Microsoft does not delete created MFT entries

## 4. MFT entry contents
![[MFT entry.png]]
- first 42 bytes contain 12 fields:
	- first field is signature; FILE, BAAD, etcv
	- flag field identifies if entry is being used and if it is for a directory
- remaining bytes store attributes, small DS with specific purpose

## 5. MFT entry addresses
- called *file number* by MS
- sequentially addressed using 48-bit value, starting with address 0
- max is determined by dividing size of $MFT by size of entry
- each MFT entry has 16-bit sequence number that is incremented when entry is allocated
- sequence number + file number = 64-bit *file reference address*

	![[file reference address.png]]
- sequence number makes it easier to determine when the FS is corrupt; address no longer matches for old file because seq number increased
- helps recover deleted data

## 6. FS metadata files
- stores FS administrative data
- MS reserves first 16 MFT entries for this; in practice 24 MFT entries are reserved
- all listed in root directory, but typically hidden from users
- name begins with $
- FS metadata files list:

	| Entry | File Name | Description | 
	| --- | --- | --- | 
	| 0 | $MFT | The entry for the MFT itself. | 
	| 1 | $MFTMirr | Contains a backup of the first entries in the MFT. See the "File System Category" section in Chapter 12. | 
	| 2 | $LogFile | Contains the journal that records the metadata transactions. See the "Application Category" section in Chapter 12. | 
	| 3 | $Volume | Contains the volume information such as the label, identifier, and version. See the "File System Category" section in Chapter 12. | 
	| 4 | $AttrDef | Contains the attribute information, such as the identifier values, name, and sizes. See the "File System Category" section in Chapter 12. | 
	| 5 | . | Contains the root directory of the file system. See the "File Name Category" section in Chapter 12. | 
	| 6 | $Bitmap | Contains the allocation status of each cluster in the file system. See the "Content Category" section in Chapter 12. | 
	| 7 | $Boot | Contains the boot sector and boot code for the file system. See the "File System Category" section in Chapter 12. | 
	| 8 | $BadClus | Contains the clusters that have bad sectors. See the "Content Category" section in Chapter 12. | 
	| 9 | $Secure | Contains information about the security and access control for the files (Windows 2000 and XP version only). See the "Metadata Category" section in Chapter 12. | 
	| 10 | $Upcase | Contains the uppercase version of every Unicode character. 
	| 11 | $Extend | A directory that contains files for optional extensions. Microsoft does not typically place the files in this directory into the reserved MFT entries. | 

## 7. MFT entry attribute concepts
- there are many types of attributes, each with its own internal structure
- all attributes have a header and contents

![[MFT entry attribute headers and content.png]]

## 8. MFT entry attribute header
- standard for all attributes
- contains:
	- name; stored in UTF-16 Unicode
	- type
	- size
	- identifier value; differentiates attributes of same type in entry
- has flags to identify if value is compressed or encrypted

## 9. MFT entry attribute content
- specific to attribute
- can have any format and size
- *resident* attribute content is stored with header
- *non-resident* attribute content is stored in *clusters runs*, with address given in header

	![[MFT entry with external cluster.png]]
- consecutive clusters, *cluster runs*, are documented using starting cluster address and length
- *Virtual Cluster Number(VCN)*, same as logical file address, to *Logical Cluster Number(LCN)*, same as logical FS address, mapping is used to describe non-resident attribute runs

## 10. standard MFT entry attribute types
- standard attributes have a default type value assigned
- type value can be redefined in $AttrDef FS metadata file
- each attribute type has a name; all caps and starts with $
- MFT entry attribute types:

	| Type Identifier | Name | Description |
	| --- | --- | --- |
	| 16 | $STANDARD_INFORMATION | General information, such as flags; the last accessed, written, and created times; and the owner and security ID. |
	| 32 | $ATTRIBUTE_LIST | List where other attributes for file can be found. |
	| 48 | $FILE_NAME | Size, file name, in Unicode, and the last accessed, written, and created times. |
	| 64 | $VOLUME_VERSION | Volume information. Exists only in version 1.2 (Windows NT). |
	| 64 | $OBJECT_ID | A 16-byte unique identifier for the file or directory. Exists only in versions 3.0+ and after (Windows 2000+). |
	| 80 | $SECURITY_DESCRIPTOR | The access control and security properties of the file. |
	| 96 | $VOLUME_NAME | Volume name. |
	| 112 | $VOLUME_INFORMATION | File system version and other flags. |
	| 128 | $DATA | File contents. |
	| 144 | $INDEX_ROOT | Root node of an index tree. |
	| 160 | $INDEX_ALLOCATION | Nodes of an index tree rooted in $INDEX_ROOT attribute. |
	| 176 | $BITMAP | A bitmap for the $MFT file and for indexes. |
	| 192 | $SYMBOLIC_LINK | Soft link information. Exists only in version 1.2 (Windows NT). |
	| 192 | $REPARSE_POINT | Contains data about a reparse point, which is used as a soft link in version 3.0+ (Windows 2000+). |
	| 208 | $EA_INFORMATION | Used for backward compatibility with OS/2 applications(HPFS). |
	| 224 | $EA | Used for backward compatibility with OS/2 applications(HPFS). |
	| 256 | $LOGGED_UTILITY_STREAM | Contains keys and information about encrypted attributes in version 3.0+(Windows 2000+) |
- nearly every MFT entry has a $FILE_NAME and a $STANDARD_INFORMATION attribute; only exception is non-base MFT entries
- every file has a $DATA attribute:
	- non-resident when over about 700 bytes
	- additional $DATA are called *Alternate Data Streams(ADS)* and must have a name associated with it
- every directory has an $INDEX_ROOT: contains contains information about files and subdirectories
- $INDEX_ALLOCATION and $BITMAP are also used to store information if directory is large
- it is possible for a directory to have $DATA
- $INDEX_ROOT and $INDEX_ALLOCATION typically have the name $I30

![[MFT entry with attribute type names.png]]

## 11. base MFT entry
- files can have up to 65,536 entries (16-bit identifier); may need more than 1 MFT entry for attribute headers
- refers to first MFT entry 
- subsequent entries have base entry address in their MFT entry fields
- has $ATTRIBUTE_LIST that contains a list with each attribute and its MFT address
- non-base MFT entries do not have $FILE_NAME and $STANDARD_INFORMATION

## 12. sparse attributes
- attribute where clusters that contain all zeros are not written to disk
- sparse run contains only size and not starting location of clusters
- flag indicates whether an attribute is sparse

![[normal and sparse data.png]]

## 13. compressed attributes
- NTFS allows attributes to be written in compressed format
- MS says only $DATA when it is non-resident should be compressed
- used together with sprase runs to reduce amount of space required
- attribute header flags identifies whether it is compressed
- $STANDARD_INFORMATION and $FILE_NAME show if file contains compressed attributes
- before attribute content compression, data is broken up into chunks called *compression units*
- compression unit scenarios:
	- all clusters contain zeros: 
		- sparse data run of compression unit size is created
		- no space is allocated
	- compression results in same amount of clusters:
		- run is made for original data
	- compression results in less clusters:
		- data is compressed and stored in run on disk
		- sparse run follows to make total run length equal to number of clusters in compression unit

![[attribute compression example.png]]
![[attribute decompression example.png]]

## 14. encrypted attributes
- NTFS allows attribute content to be encrypted; not header
- Windows allows only $DATA to be encrypted
- $LOGGED_UTILITY_STREAM containing keys needed to decrypt data is created
- in Windows, a file or directory can be chosen to be encrypted
- encrypted file/directory has a special flag set in $STANDARD_INFORMATION; each encrypted attribute also has special flag set in attribute header

## 15. NTFS encryption implementation
- $DATA contents are encrypted with a symmetric algorithm called DESX
- one random key is generated for each MFT entry with encrypted data, called *File Encryption Key(FEK)*
- multiple $DATA in single MFT entry are encrypted with same FEK
- FEK is stored in an encrypted state in $LOGGED_UTILITY_STREAM
- $LOGGED_UTILITY_STREAM contains list of *Data Decryption Fields(DDF)* and *Data Recovery Fields(DRF)*
- DDF:
	- created for every user with access
	- containing user's *Security ID(SID)*, encryption information, and FEK encrypted with user's public key
- DRF:
	- created for each method of data recovery
	- containing FEK encrypted with data recovery public key 
	- for admin or other authorized user
- temporary file called EFS0.TMP containing plaintext version of file being encrypted is created; later deleted but not wiped from disk, allowing recovery

![[NTFS encryption process.png]]

## 16. NTFS decryption implementation
1. user's encrypted private key is retrieved from registry
2. encrypted key is decrypted with user password
3. $LOGGED_UTILITY_STREAM is processed to locate user DDF entry
4. FEK is decrypted using user key
5. $DATA is decrypted using FEK

![[NTFS decryption process.png]]
