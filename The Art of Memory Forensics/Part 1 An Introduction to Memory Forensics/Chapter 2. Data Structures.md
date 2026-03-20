## 1. basic data types
- used to specify how a set of bits is utilized within a program
- not defined in terms of other data types in that language
- may map directly to native hardware data types supported by processor architecture
- storage size my change depending on hardware

## 2. C
- C basic data types

	| Type | 32-Bit Storage Size(Bytes) | 64-Bit Storage Size(Bytes) |
	| --- | --- | --- |
	| char | 1 | 1 |
	| unsigned char | 1 | 1 |
	| signed char | 1 | 1 |
	| int | 4 | 4 |
	| unsigned int | 4 | 4 |
	| short | 2 | 2 |
	| unsigned short | 2 | 2 |
	| long | 4 | Windows: 4, Linux/Mac: 8 |
	| unsigned long | 4 | Windows: 4, Linux/Mac: 8 |
	| long long | 8 | 8 |
	| unsigned long long | 8 | 8 |
	| float | 4 | 4 |
	| double | 8 | 8 |
	| pointer | 4 | 8 |
- Windows types

	| Type | 32-Bit Storage Size(Bytes) | 64-Bit Storage Size(Bytes) | Purpose/Native Type |
	| --- | --- | --- | --- |
	| DWORD | 4 | 4 | Unsigned long |
	| HMODULE | 4 | 8 | Pointer/handle to a module |
	| FARPROC | 4 | 8 | Pointer to a function |
	| LPSTR | 4 | 8 | Pointer to a character string |
	| LPCWSTR | 4 | 8 | Pointer to a Unicode string |

## 2. abstract data types
- provide models for data and operations performed on data
- independent of programming language

## 3. arrays
- collection of <index, element> pairs
- elements are of homogeneous type
- size is fixed when instance is created
- *random access* time, meaning it doesn't depend on element number
- elements are accessed using base address + offset
- frequently used by OS due to efficiency:
	- Linux file handles
	- Microsoft **MajorFunction** table
![[MajorFunction table example.png]]

## 4. *bitmaps*
- aka *bit vector* or *bit array*
- elements store a boolean value

## 5. records
- aka *structure*
- heterogeneous elements
- collection of <name, element> pairs
- static; combination of elements and order is fixed when instance is created
- elements are accessed using base address + offset
- padding may be added between fields

## 6. strings
- special case of array storing characters codes
- can contain variable length sequence of elements
- Windows **_UNICODE_STRING** stores characters encoded in UTF-16; characters may be 2 or 4 bytes

	| Byte Range | Name | Type | Description |
	| --- | --- | --- | --- |
	| 0-1 | Length | unsigned short | Current string length |
	| 2-3 | MaximumLength | unsigned short | Maximum string length |
	| 8-15 | Buffer | *unsigned short | String address |
	- only contains metadata for string; VA translation required 

## 7. linked lists
- can efficiently support dynamic updates
- not indexed; sequential access required
- VA translation required
- types:
	- singly linked list: single direction traversal
	- doubly linked list: bi-direction traversal
	- circular linked list: tail points to head

## 8. hash tables
- used in circumstances that require efficient insertions and searches where data is being stored in <key, element> pairs
- used by OS to store info on:
	- active processes
	- network connections
	- mounted FS
	- cached files
- common implementation uses hash tables composed of arrays of linked lists, *chained overflow hash tables*:
	- hash function converts key into array index
	- collisions are stored within linked list

	![[chained overflow hash table example.png]]

## 9. trees
- structured organization of data in storage
- operations can be performed efficiently on elements
- hierarchical trees do not contain cycles
- tree traversal:
	- preorder: current node, subtrees left to right
	- inorder: left subtree, current node, subtrees left to right
	- post order: subtrees left to right, current node
