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
