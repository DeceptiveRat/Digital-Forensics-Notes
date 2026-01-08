## 1. data acquisition layers
- at each layer of abstraction, data is lost => acquire data at lowest layer with evidence
- e.g. acquiring a disk at volume level would copy every sector in every partition, losing sectors not allocated to partitions

## 2. acquisition tool testing
- *National Institute of Standards and Technology* (NIST) conducted tests on common acquisition tools
- *Computer Forensic Tool Testing* (CFTT) project developed requirements and test cases for disk-imaging tools
- results can be found on their website

## 3. disk access via BIOS
- may return incorrect information about disk:
	- BIOS is configured for hard disk geometry that is different
	- acquisition tool uses legacy method of requesting size of disk
- should be avoided if alternatives are available

## 4. dead and live acquisition
- live acquisition risks false data
- hardware can be modified to return false data even with a trusted operating system, but it is uncommon

## 5. error handling
- generally bad sector addresses are logged and 0s are written in their place

## 6. HPA handling
- some write blockers may block disk configuration commands
- errors can occur while changing disk configuration to remove HPA, causing data to be lost => disk should be imaged before HPA removal

## 7. DCO handling
- DCO removal commands may be blocked by write blockers 
- safer to create image of disk before removing DCO

## 8. hardware write blockers
- located between computer and storage device
- prevents writing to storage
- simple designs prevent data from being written to command register
- CFTT at NIST published specification for hardware write blockers

## 9. software write blockers
- work by modifying INT13h interrupt table entry to point to write blocker code
- software can bypass BIOS to write directly to controller, BIOS can still write to controller
- hardware write blocking is more effective
- CFTT at NIST published details on software write block devices

## 10. writing output to disk
- destination disk should be wiped with zeroes before a duplicate(clone) copy
- some OS auto mounts disk, potentially changing data
- some data structures rely on geometry to describe locations, so different geometry can cause problems

## 11. writing output to file
- easy to know boundaries of data
- OS will not auto mount
- file is called *image*
- wiping disks before writing can help prove there is no contamination

## 12. image file format
- raw image: contains only data from source
- embedded image: contains additional descriptive data such as hash values, dates, and times

![[image formats.png]]

## 13. compressing image file
- special compression algorithms that allow decompression of parts should be used to save time and space
- compressing images cons:
	- limited by tools that support format
	- acquisition takes longer
	- analysis takes longer
