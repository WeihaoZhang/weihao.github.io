# Operating System organization
---
> Three requirements for operating system:multiplexing,isolation and interaction.This chapter provides an overview of how operating systems are origanized to achieve these three requirements.This chapter provides an overview of an xv6 process,and the creation of the first process when xv6 starts.
> Xv6 is written for the support hardware simulated by qemu's  "-machine virt" option.This includes RAM,a ROM containing boot code, a serial connection to the user's keyboard/screen, a disk for storage.

---

## Abstracting physical resources

To ioslate,it's helpful to forbid applications from directly accessing sensitive hardware resource (For efficiency, some operating systems for embedded devices  or real-time systems are orginized in the way that applications could interact with hardware resources)
. Syscall open,read,write and close are examples.


## User mode,supervisor mode,and system calls
## Kernel organization
## Code:xv6 organization
## 