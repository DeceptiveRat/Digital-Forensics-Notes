## 1. *Representational State Transfer(REST)* 
- software architectural style created to describe the design and guide the development of the architecture for the World Wide Web
- defines a set of constraints for how the architecture of a distributed Internet-scale hypermedia system should behave
- emphasizes:
	- uniform interfaces
	- independent deployment of components
	- scalability of interactions
	- layered architecture to promote caching to reduce latency, enforce security, and encapsulate legacy systems
- REST API: API that conforms to design principles of *REST* 

## 2. design principles
- uniform interface:
	- requests for the same resource should look the same
	- ensure same piece of data belongs to only one *Uniform Resource Identifier(URI)*
	- resources shouldn't be too large
	- resources should contain every information client might need
- client/server decoupling:
	- client and server are independent of each other
	- client should only know the URI of the requested resource
	- server should not modify client application
- statelessness:
	- each request contains all information necessary to process it

## reference
[1] REST, 2026/07/04, https://en.wikipedia.org/w/index.php?title=REST&oldid=1345506837
[2] (Phil Powell and Ian Smalley, 2025/04/24), What is REST API?, 2026/07/04, https://www.ibm.com/think/topics/rest-apis