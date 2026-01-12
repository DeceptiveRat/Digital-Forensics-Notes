## 1. Digital Crime Scene Investigation(CSI) Process
![[Crime Scene Investigation phases.png]]
- system preservation
- evidence searching
- event reconstruction

## 2. system preservation
- reduce amount of evidence that may be overwritten
- techniques:
	- dead analysis: duplicate all data
	- live analysis: terminate suspicious processes and disconnect from network, copy important data
	- calculate cryptographic hash to confirm integrity
	
## 3. evidence searching
- survey common locations based on incident type
- look for evidence to refute hypothesis
- technique:
	- based on name patterns
	- based on keyword in content
	- based on temporal data
	- based on hash

## 4. event reconstruction
- requires knowledge of application and OS capabilities to form hypothesis

## 5. general guidelines - PICL
- preservation:
	- perform analysis on copy
	- hashes for integrity checks
	- write-blocking devices
	- minimize file creation to prevent overwriting evidence in unallocated space
	- opening files could modify data such as last access time
- isolation:
	- isolate suspect data: view using VM
	- isolate from outside world: remove network connection 
- correlate:
	- reduces risk of forged data
	- e.g. file time stamps should be correlated with log entries, network traffic, etc
- log:
	- helps identify which searches are left
	- important in live analysis where system can change

## 6. data analysis
![[analysis layers.png]]![[data analysis process.png]]
- volume: collection of storage locations
- file system: collection of data structures that allow an app to create/read/write files
- application analysis: each file has a different structure internally

## 7. essential/nonessential data
- essential data: 
	- can trust
	- required to read/write data
	- e.g. file name, file location
- nonessential data:
	- can't trust
	- e.g. last access time