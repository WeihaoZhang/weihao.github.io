# Operating System organization
---
> Three requirements for operating system:multiplexing,isolation and interaction.This chapter provides an overview of how operating systems are origanized to achieve these three requirements.This chapter provides an overview of an xv6 process,and the creation of the first process when xv6 starts.
> Xv6 is written for the support hardware simulated by qemu's  "-machine virt" option.This includes RAM,a ROM containing boot code, a serial connection to the user's keyboard/screen, a disk for storage.

---

## Abstracting physical resources

To ioslate,it's helpful to forbid applications from directly accessing sensitive hardware resource (For efficiency, some operating systems for embedded devices  or real-time systems are orginized in the way that applications could interact with hardware resources)
. Syscall open,read,write and close are examples.


## User mode,supervisor mode,and system calls

CPUs provide hardware support for strong isolation.RISC-V has 3 modes in which CPU can execute instructions:
- machine mode (full privilege,cpu starts)
- supervisor mode(kernel run,enabling or disableing interrupts,reading or writing register that holds the address or a page table)
- user mode()
RISC-V provides ecall function for purpose that switches CPU form user mode to supervisor mode and enters kernel at point **specified by the kernel.**


## Kernel organization

Monolithic kernel:
The entire operating system resides in the kernel, so that the implementations of all system calls run in supervisor mode. 
Microkernel kernel:
OS designers can minimize the amount of operating
system code that runs in supervisor mode, and execute the bulk of the operating system in user mode

## Code:xv6 organization

<img src="C:\Users\WYX\AppData\Roaming\Typora\typora-user-images\image-20220706221915296.png" alt="image-20220706221915296" style="zoom:50%;" />

## Process overview
The unit of isolation in xv6 is a process. Xv6 uses separate page table for each process defines process's address space.Trampoline pae contains the code to transition in and out of the kernel and mapping trapframe is necessary to save and restore the state of user porcess.

<img src="C:\Users\WYX\AppData\Roaming\Typora\typora-user-images\image-20220706224525430.png" alt="image-20220706224525430" style="zoom:50%;" />

Each process has two stacks: a user stack and a kernel stack(p->kstack).
A process can make a system call by executing the RISC-V ecall instruction that rasing the hardware privilege level and changes the program counter to a kernel-defined entry point.And kernel can switch back to user mode by calling the sret instruction.
Generally,a process bundles two design ideas: an address space to give illusion of merory and thread to give the process the illision of its own CPU.
