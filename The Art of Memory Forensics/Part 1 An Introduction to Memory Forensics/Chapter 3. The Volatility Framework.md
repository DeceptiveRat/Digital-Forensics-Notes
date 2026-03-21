## 1. what Volatility is
- single, cohesive framework
- open source GPLv2
- written in Python
- runs on Windows, Linux, and Mac
- extensible and scriptable API
- unparalleled feature set
- comprehensive coverage of file formats
- fast and efficient algorithms
- serious community
- focused on forensics, IR, and malware

## 2. what Volatility is not
- memory acquisition tool
- GUI tool
- bug-free

## 3. framework

## 4. VTypes
- Volatility's structure definition and parsing language
- provides a way to represent C DS in Python source files
``` C
struct process {
	int pid;
	int parent_pid;
	char name[10];
	char * command_line;
	void * ptv;
};
```
``` VType
'process' : [ 26, {
	'pid' : [ 0, ['int']],
	'parent_pid' : [ 4, ['int']],
	'name' : [ 8, ['array', 10, ['char']]],
	'command_line' : [ 18, ['pointer', ['char']]],
	'ptv' : [ 22, ['pointer', ['void']]],
}]
```
	- name of structure is the first dictionary key
	- first value is size of structure and dictionary of elements
	- each element name is the key to their offset and list of contained data type

## 5. overlays
- enables you to fix up automatically generated structure definitions
``` python
'process' : [ None, {
	'ptv' : [ None, ['pointer', ['process']]],
}
```
	- changes type of ptv from void* to process*

## 6. objects and classes
- *object* is an instance of a structure that exists at a specific address within an *Address Space(AS)*
- *object class* enables you to extend functionality of an object:
	- can attach methods or properties to an object that becomes accessible to all instances of the object

## 7. profiles
- collection of VTypes, overlays, and object classes for a OS version and architecture
- also includes:
	- metadata
	- system call information
	- constant values
	- native types
	- system map

## 8. address spaces
- an interface that:
	- provides flexible and consistent access to data in RAM
	- handles virtual-to-physical-address transation
	- accounts for differences in memory dump formats

## 9. virtual/paged address spaces
- provide support for reconstructing virtual memory
- deals only with memory that is both allocated and accessible
- contains subset of memory that programs on a system can "see" at acquisition without page fault
- includes kernal and process AS

## 10. physical address spaces
- deals with various file formats that memory acquisition tools use
- often contain memory ranges marked as freed

## 11. plugin system
- allow expansion of existing framework
- *address space plugins* can introduce support for OS that run on new CPU chipsets
- *analysis plygins* are written to find and analyze specific components of the OS, user applications, or malicious code samples

## 12. using volatility

## 13. basic commands
``` sh
$ python vol.py –f <FILENAME> --profile=<PROFILE> <PLUGIN> [ARGS]
```
- execute main Python script and pass:
	- path to memory dump
	- name of profile
	- plugin to execute

## 14. displaying help
- pass `-h` to display main help menu:
	- shows global options
	- lists plugins available to current profile
- pass `-h` with plugin name to view plugin-specific options
	``` sh
	$ python vol.py handles --help
	```

## 15. selecting profile
- default is WinXPSP2x86
- `image info` plugin provides high-level summary of memory sample, helps determine profile for Windows
	``` sh
	$ python vol.py -f memory.raw imageinfo
	```
- `kdbgscan` plugin also helps determine profile for Windows
	``` sh
	$ python vol.py -f memory.raw kdbgscan
	```

## 16. saving command line options
- Volatility can search environment variables and configuration files for options not supplied
- environment variable:
	``` sh
	$ export VOLATILITY_PROFILE=Win7SP0x86
	$ export VOLATILITY_LOCATION=file:///tmp/myimage.img
	```
	- naming convention:
		- variable name set according to original option, but with VOLATILITY prefixed
	- location and filename:
		- path must be prefixed with file:///
	- persistence:
		- add environment variables to .bashrc for persistence
- config files:
	- named .volatilityrc in current directory or home directory
	- can be specified with --conf-file
	``` ini
	[DEFAULT]
	PROFILE=Win7SP0x86
	LOCATION=file:///tmp/myimage.img
	```
