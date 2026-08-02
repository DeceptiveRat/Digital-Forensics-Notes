## inodes
- contains file metadata and location of data content
- consists of:
	- inode core
	- data fork
	- attribute fork

## inode core
- structure:

| Offset | Size | Name              | Description                                                        |
|--------|------|-------------------|--------------------------------------------------------------------|
| 0x00   | 0x02 | Signature         | XFS inode signature (ASCII: IN).                                   |
| 0x02   | 0x02 | Mode              | File type and permissions. Similar to ext, the most                |
|        |      |                   | significant nibble provides the type while the remainder           |
|        |      |                   | provides the permissions.                                          |
| 0x04   | 0x01 | Version           | Inode version (1, 2 or 3). XFS version 5 file systems will         |
|        |      |                   | always have version 3 inodes.                                      |
| 0x05   | 0x01 | Format            | Identifies how information is stored in the data fork. Known       |
|        |      |                   | values include: 1 – Resident data (i.e. the content is present     |
|        |      |                   | in the inode); 2 – Extent array; There are other values but        |
|        |      |                   | their meaning is unclear.                                          |
| 0x06   | 0x02 | # Links (old)     | Unused in version 3 inodes.                                        |
| 0x08   | 0x04 | UID               | User ID of the inode owner.                                        |
| 0x0C   | 0x04 | GID               | Group ID of the inode group.                                       |
| 0x10   | 0x04 | # Links           | Number of links to this file.                                      |
| 0x14   | 0x04 | Project ID        | Files may be grouped by project ID in XFS allowing quotas          |
|        |      |                   | to be set for a particular project.                                |
| 0x18   | 0x06 | Padding           | Padding bytes.                                                     |
| 0x1E   | 0x02 | Flushiter         | Flush counter.                                                     |
| 0x20   | 0x04 | atime             | Last access time.                                                  |
| 0x24   | 0x04 | atime (ns)        | Nanosecond component of atime.                                     |
| 0x28   | 0x04 | mtime             | Last content modification time.                                    |
| 0x2C   | 0x04 | mtime (ns)        | Nanosecond component of mtime.                                     |
| 0x30   | 0x04 | ctime             | Last metadata change time.                                         |
| 0x34   | 0x04 | ctime (ns)        | Nanosecond component of ctime.                                     |
| 0x38   | 0x08 | File Size         | For regular files this is the file size, for directories it is the |
|        |      |                   | size of the directory entries, and for symbolic links it is the    |
|        |      |                   | length of the link.                                                |
| 0x40   | 0x08 | # Blocks          | The number of blocks used to store the inode’s data                |
|        |      |                   | excluding extended attributes.                                     |
| 0x48   | 0x04 | Extent Size       | Extent size hint.                                                  |
| 0x4C   | 0x04 | # Extents         | The number of extents associated with this file.                   |
| 0x50   | 0x02 | # ExtAttr Extents | The number of extents associated with the extended                 |
|        |      |                   | attributes.                                                        |
| Offset | Size | Name              | Description                                                        |
| 0x52   | 0x01 | Fork Offset       | Offset in the inode at which the extended attribute fork           |
|        |      |                   | begins. This number must be multiplied by 8d to get the            |
|        |      |                   | actual byte offset.                                                |
| 0x53   | 0x01 | Attr. Format      | Storage format of the attribute fork. This uses the same           |
|        |      |                   | values as the data fork format.                                    |
| 0x54   | 0x04 | DMAPI Event Mask  | Related to the Data Management API.                                |
| 0x58   | 0x02 | DMAPI State       | Related to the Data Management API.                                |
| 0x5A   | 0x02 | Flags             | Flags.                                                             |
| 0x5C   | 0x04 | Generation        | Generation ID.                                                     |
| 0x60   | 0x04 | Next Unlinked     | Tracking of deleted attributes that are still in use by a          |
|        |      |                   | program.                                                           |
| 0x64   | 0x04 | CRC               | Inode checksum.                                                    |
| 0x68   | 0x08 | Change Count      | Number of changes to the attributes in this inode.                 |
| 0x70   | 0x08 | LSN               | Log sequence number of the last write to this file.                |
| 0x78   | 0x08 | Flags2            | Further inode flags.                                               |
| 0x80   | 0x04 | COW Extent Size   | Copy-on-Write extent size.                                         |
| 0x84   | 0x0C | Padding           | Padding.                                                           |
| 0x90   | 0x04 | btime             | Birth (Creation) time.                                             |
| 0x94   | 0x04 | btime (ns)        | Nanosecond component of btime.                                     |
| 0x98   | 0x08 | Inode #           | Absolute inode number for this inode.                              |
| 0xA0   | 0x10 | UUID              | File system UUID.                                                  |

## inode data fork
- 3 primary formats:
	- local:
		- store data locally in inode
		- only for small data
	- extent list:
		- array of 16 byte extents
	- B+tree:
		- used when file is heavily fragmented
		- data fork becomes root  of extent-indexing B+tree

## inode attribute fork
- attributes are simply name, value pairs
- 255 byte limit on size of any name or value
- attribute set with *attr* command
- long attributes are tracked using extents
- resident attribute information starts at specific byte offset from end of inode core<sup>[2]</sup>
- header structure:
![[extended attribute header.png]]
- example extended attribute:
![[example extended attribute.png]]

## inode addressing
- absolute addressing: 
	- 4 bytes in size
	- $log_2(\textit{inodes per block})$ LSB: inode offset in block ($inode_{offset}$)
	- $log_2(\textit{AG count})$ subsequent bits: block offset in AG ($block_{offset}$)
	- remaining bits: AG number
	- inode address equation: $((AG_{\#} \times AG_{size} + block_{offset}) \times block_{size}) + (inode_{offset} \times inode_{size})$

## reference
[1] (Fergus Toolan, 2025/02/07) File System Forensics, 2026/08/01, -
[2] (Hal Pomeranz, 2018/05/23) # XFS (Part 2) – Inodes, 2026//08/01, https://righteousit.com/2018/05/23/xfs-part-2-inodes/