## 1. Operating System
- deals with low-level details of hardware
- implements a set of high-level services and interfaces that define how hardware can be accessed

![[Computer Architecture Overview.png]]

## 2. motherboard
- circuit board that connects various components
- communication channels are called *computer buses*

## 3. *Central Processing Unit(CPU)*
- accesses main memory to obtain instructions then executes instructions
- relies on *Memory MAnagement Unit(MMU)* to find data

## 4. MMU
- translates address that processor requests to address in main memory
- DS for address translation also stored in memory
- special cache *Translation Lookaside Buffer(TLB)* used for MMU translation table; used before MMU for quick lookups

## 5. *memory controller*
- mediates potentially concurrent requests for system memory from processor and devices

## 6. *Direct Memory Access(DMA)*
- CPU initiates data transfer and DMA controller manages data transfer
- can directly access contents of physical memory without involivng untrusted software

## 7. *Random Access Memory(RAM)*
- volatile memory; lost without power
- constant access time regardless of location

## 8. address space
- refers to range of valid addresses used to identify data stored
- *linear addresses*, or *virtual addresses*, are translated into *physical addresses* for accessing physical memory

## 9. IA-32
- x86 architectures that support 32-bit computation
- specifies instruction set and programming environment for Intel's 32-bit processors
- little endian machine that uses byte addressing
- max linear/physical address 4GB
- size of physical memory can be expanded to 64GB using *Physical Address Extension(PAE)* 

## 10. IA-32 reigisters
- processor core contains 8 32-bit general purpose registers for logical and arithmetic operations
- EIP register:
	- aka *program counter*
	- contains linear address of next instruction
- 5 control registers that specify configuration of processor and characteristics of executing task
- CR0:
	- flags controlling operating mode of processor
	- e.g. paging
- CR1:
	- reserved
	- should not be addressed
- CR2:
	- linear addresses that caused page fault
- CR3: 
	- physical address of initial structure used for address translation
- CR4: 
	- enable architectural extensions
	- e.g. PAE

## 11. *segmentation*
- divides 32-bit linear address space into multiple variable-length segments
- all IA-32 memory references are addressed using:
	- 16-bit segment selector to identify *segment descriptor*
	- 32-bit offset into specified segment
- segment descriptor:
	- memory-resident DS
	- defines location, size, type, and permissions for given segment
- processor core contains 2 special registers, GDTR and LDTR:
	- GDTR points to *Global Descriptor Table(GDT)*
	- LDTR points to *Local Descriptor Table(LDT)*
- segmentation registers should contain valid segment selectors:
	- CS: code
	- SS: stack
	- DS: data
	- ES: data
	- FS: data
	- GS: data
- segmented addressing is hidden by defining overlapping segments with base address 0

## 12. *paging*
- provides ability to virtualize linear address space
- large linear space is simulated with physical memory and disk storage
- 32-bit linear address space is broken up into fixed-length sections, *pages*
- pages are mapped into physical memory in arbitrary order
- memory-resident *page directories* and *page tables* are used to translate linear addresses to physical addresses
- translating *virtual address(VA)* to *physical address(PA)*:
	- bits 32:12 of CR3 register + bits 32:22 of VA = *Page Directory Entry(PDE)* address
	- bits 31:12 of PDE + bits 21:12 of VA = *Page Table Entry(PTE)* address
	- bits 31:12 of PTE + bits 11:0 of VA = PA

![[address translation example.png]]
![[paging structure address format.png]]

- example translation:

![[VA to PA.png]]

## 13. PAE
- allows processors to support address spaces greater than 4GB
- linear address divided into 4 indexes:
	- *Page Directory Pointer Table(PDPT)*
	- *Page Directory(PD)*
	- *Page Table(PT)*
	- page offset
