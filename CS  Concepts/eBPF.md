## 1. *Extended Berkeley Packet Filter(eBPF)*
- in-kernel virtual machine
- allows users to run sandboxed programs in kernel space
- can extend capability of kernel at runtime without changes to kernel source code
- guarantees execution efficiency and safety with a JIT compiler and verification engine 

## 2. example program
``` python
from bcc import BPF
program = r"""
	int hello(void *ctx) {
		bpf_trace_printk("Hello World!");
		return 0;
	}
"""
b = BPF(text=program)
syscall = b.get_syscall_fnname("execve")
b.attach_kprobe(event=syscall, fn_name="hello")

b.trace_print()
```

![[eBPF_example.png]]

## 3. eBPF hooks
- eBPF programs are event-driven; run when kernel or application passes a certain hook point
- predefined hooks include:
	- system calls
	- function entry/exit
	- kernel tracepoints
	- network events
- possible to create *kernel probe(kprobe)* or *user probe(uprobe)* to attach eBPF programs almost anywhere

![[eBPF_hooks.png]]

## 4. 

## reference
[1] eBPF, 2026/07/04, https://en.wikipedia.org/w/index.php?title=EBPF&oldid=1361446527
[2] BoB class - 클라우드 컴퓨팅 개요 및 클라우드 기반 인프라, 서비스 보안 기술
[3] eBPF Documentation, 2026/07/04, https://ebpf.io/what-is-ebpf/
