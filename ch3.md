# Page tables

## paging hardware
>> The machine's RAM is indexed with physical addresses,while RISC-V instructions manipulate virtual addresses.The RISC-V page table hardware connects these two kinds of addresses.
>> xv6 runs on Sv39 RISC-V,only the bottom 39 bits of a 64-bit virtual address are used, a RISC-V page table is logically an array of 2^27^.Each PTE contains a 44-bit physical page number(PPN) and some flag. The paging hardware translates virtual addr by using top27 bits of the 39 bits to index into the page table to find a PTE and making a 56 bit physical address whose top 44 bits come froem PPN in the PTE and bottom 12 bits are copied frome original virtual address.
>>
>> Top 27 bits are used to select page-tables,with 9 bits for top one,....Each page table has 512 entrys.A page-fault exception is raised if an address is not present.Three level page table has a potential downside that PTES have to be loaded or stored from RAM.So RISC-V caches page table entries in a Translation Look-aside Buffer(TLB)
>> 