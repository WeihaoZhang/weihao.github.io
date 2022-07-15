# Page tables

## paging hardware
>> The machine's RAM is indexed with physical addresses,while RISC-V instructions manipulate virtual addresses.The RISC-V page table hardware connects these two kinds of addresses.
>> xv6 runs on Sv39 RISC-V,only the bottom 39 bits of a 64-bit virtual address are used, a RISC-V page table is logically an array of 2^27^.Each PTE contains a 44-bit physical page number(PPN) and some flag. The paging hardware translates virtual addr by using top27 bits of the 39 bits to index into the page table to find a PTE and making a 56 bit physical address whose top 44 bits come froem PPN in the PTE and bottom 12 bits are copied frome original virtual address.
>
>> Top 27 bits are used to select page-tables,with 9 bits for top one,....Each page table has 512 entrys.A page-fault exception is raised if an address is not present.Three level page table has a potential downside that PTES have to be loaded or stored from RAM.So RISC-V caches page table entries in a Translation Look-aside Buffer(TLB)
>>
>> <img src="https://github.com/WeihaoZhang/weihao.github.io/blob/main/ch3/image-20220712221607489.png" alt="image-20220712221607489" style="zoom: 33%;" />

>>Each PTE contains flag bits that tell the paging hardware how the associated virtual address is allowed to be used.

<img src="https://github.com/WeihaoZhang/weihao.github.io/blob/main/ch3/image-20220712222349672.png" alt="image-20220712222349672" style="zoom: 33%;" />

>> To tell the hardware to use a page table,the kernel must write the physical address of the root page-table page into the satp register. Each CPU has its own satp so that different CPUs can run different processes.

## Kernel address space

>> Kernel configures the layout of its address space to give itself access to physical memory and various resources at predictable virtual address.
>>
>> <img src="https://github.com/WeihaoZhang/weihao.github.io/blob/main/ch3/image-20220712225232572.png" alt="image-20220712225232572" style="zoom:50%;" />
>> The kernel gets at RAM and memory-mapped device registers using "direct mapping",mapping the resources at virtual address equal to the physical address.There are a couple of kernel virtual addresses that aren't direct-mapped:
>>1.trampoline page,at the top of virtual address space.A physical page holding the trampoline code is mapped twice in the virtual address space of the kernel:once at top of virtual address space and once with a direct mapping.
>>2.The kernel stack page, mapped high so that below xv6 can leave an unmapped guard page.The guard page's PTE is invalid,so that kernle will panic if it overflows a kernel stack.  


## Code: creating an address space

>> The center data structure is pagetable_t,which is really a pointer to a RISC-V root page-table:a pagetable_t may be either the kernel page table or per-process page tables.The central functions are walk,which finds the PTE for a virtual address and mappages,which installs PTEs for new mappings.
>> code is showed in vm.c
>> When xv6 changes a page table ,it must tell CPU to invalidate correspoding cached TLB entries.RISC-V has an instruction sfence.vma that flushes the current CPU's TLB.To aoid the complete TLB,RISC-V CPUs may support address space identifiers ASIDs.The kernel can flush the TLB entries for particular address space

## Physical memory allocation

>> xv6 uses physical memory between the end of kernel adn PHYSTOP for run-time allocation.It allocates and frees whole 4096-byets pages at a time.Free pages are threaded by a linked list through the pages themselves.

## Code:Physical memory allocator

>>The allocaor's data structure is a free lsit of physical memory  that are available for allocation.Each free page's list element is a struct run. Thre free list is stored in a free page,protected by a spin lock.

##Process address space
>> We see here a few nice examples of use of page tables:
>> 1.Different processes's page tables translate user address to different pages of physical memory, so that each process has privatre user memory.
>> 2.Each process has continus virtual address,but non-continuous.
>> 3.The kernel maps a page with trampoline code at the top of user address space,thus a single page of pyhsical memory shows up in all address sapce.
>> To detect a user stack overflowing the allocated stack memory,xv6 places an inaccessible guard page right below the stack by clearing PTE_U flag.

## Code:sbrk

>> Sbrk is the system call for a process to shrink or grow its memory.The system call is implemented by the function growproc.Growproc calls uvmalloc or uvmdealloc,depending on whether n is positive or negative.uvmalloc allocates physical memory with kalloc,and adds PTES to the user page table with mappages.uvmdemalloc calls uvmunmap, which uses walk to find PTEs and kfree to free the physical memory they refer to.

<img src="https://github.com/WeihaoZhang/weihao.github.io/blob/main/ch3/image-20220715222004245.png" alt="image-20220715222004245" style="zoom:50%;" />

## Code:exec
>> Exec is the system that creates the user part of an address space.Exec opens the named binary path using namei.Then it reads the ELF header described in the widely-used ELF format,defined in kernel/elf.h.An ELF binary consists of an ELF header,struct elfhdr followed by a sequence of program section headers,struct proghdr.Each proghdr describes a section of application that must be loaded into memory.
>> Exec allocates a new page table with no user mappings with proc_pagetable,allocates memory for each ELF segment with uvmalloc,and loads each segment into memory with loadseg.loadseg uses walkaddr to find the physical address of the allocated memory at which to write each page of the ELF segment,and readi to read from the file.
>>  Now Exec allocates and initializes the user stack.It allocates one stack page.places an inaccessible page below the stack page.If an invalid program segment detected by exec during the preparation of the new memory image, exec frees the new image and returns -1.Execs laod binary bytes form ELF file into memory at address specified by the ELF file,thus exec is risky,because teh address may refer to kernel.Xv6 performs a number of checks to avoid theses risks. But in new xv6 system,kernel has its separate page table.
