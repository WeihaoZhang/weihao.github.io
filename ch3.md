# Page tables

## paging hardware
>> The machine's RAM is indexed with physical addresses,while RISC-V instructions manipulate virtual addresses.The RISC-V page table hardware connects these two kinds of addresses.
>> xv6 runs on Sv39 RISC-V,only the bottom 39 bits of a 64-bit virtual address are used, a RISC-V page table is logically an array of 2^27^.Each PTE contains a 44-bit physical page number(PPN) and some flag. The paging hardware translates virtual addr by using top27 bits of the 39 bits to index into the page table to find a PTE and making a 56 bit physical address whose top 44 bits come froem PPN in the PTE and bottom 12 bits are copied frome original virtual address.
>
>> Top 27 bits are used to select page-tables,with 9 bits for top one,....Each page table has 512 entrys.A page-fault exception is raised if an address is not present.Three level page table has a potential downside that PTES have to be loaded or stored from RAM.So RISC-V caches page table entries in a Translation Look-aside Buffer(TLB)
>>
>> <img src="D:\zwh\operatingSystem\weihao.github.io\ch3\image-20220712221607489.png" alt="image-20220712221607489" style="zoom: 33%;" />

>>Each PTE contains flag bits that tell the paging hardware how the associated virtual address is allowed to be used.

<img src="C:\Users\WYX\AppData\Roaming\Typora\typora-user-images\image-20220712222349672.png" alt="image-20220712222349672" style="zoom: 33%;" />

>> To tell the hardware to use a page table,the kernel must write the physical address of the root page-table page into the satp register. Each CPU has its own satp so that different CPUs can run different processes.

## Kernel address space

>> Kernel configures the layout of its address space to give itself access to physical memory and various resources at predictable virtual address.
>>
>> <img src="C:\Users\WYX\AppData\Roaming\Typora\typora-user-images\image-20220712225232572.png" alt="image-20220712225232572" style="zoom:50%;" />
