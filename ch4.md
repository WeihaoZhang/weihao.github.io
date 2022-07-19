## Traps and system calls
>> There are three kinds of event that causes CPU to set aside the execution of instructions and force a transfer of control to special code that handles the event.
>>> 1.syscall. ecall instruction 
>>> 2.exception. something illegal
>>> 3.device interrupt,need attention
>> We oftern want trap to be transparent.The handle sequence is that :
>>> 1.a trap forces a transfer of control into the kernel
>>> 2.the kernel save registers and other state so that execution can be resumed
>>> 3.kernel executes approriate handler code
>>> 4.kernel restores the saved state and return from the trap
>>> 5.code resumes where it left off.
>>xv6 trap handling proceeds in 4 steps:
>>>1.hardware actions taken by the RISC-V CPU
>>>2.an assember vectorthat prepares the way for kernel C code
>>>3.ctrap that decides waht to do with the trap
>>>4.sytem call or device-driver service routine
## RISC-V trap machinery
>> RISC-V has a set of control registers that the kernel reads to find out abount a trap that has occured and writes to tell CPU how to handle traps.
>>> stvec:the kernel writes the address of its trap handler here.
>>> sepc:when traps occurs,RISC-V saves program counter here.
>>> scause:RISC-V CPU puts a number here that describes the reason for the trap
>>> sscratch:The kernel places a value that comes in handy at very start of a trap handler.
> > sstatus:SIE bit sstatus control whether device interrupts are enabled.If the kernel clears SIE,RISC-V will defer device interruptsuntil the kernel ste SIE.SPP bit indicates whether a trap came from user mode or supervisor mode.
>> > All above registers handled in supervisor mode and can't be read or write in user mode.An equivalent set of control registers for traps handled in machine mode, only timer interrupts handled.
>> > 1.If the trap is a device interrupt,an status SIE bit is clear,don't do any of the following
>> > 2.disable interrupts by clearing SIE
>> > 3.cop pc to sepc
>> > 4.save current mode to SPP bit
>> > 5.set sacause to reflect trap's cause
>> > 6.set mode to supervisor
>> > 7.Copy stvec to pc
>> 
>> Now that CPU doesn't save page table ,stack and any other registers other than pc.Kernel must perform these tasks.

## Traps from user space
>> The high-level path of trap from user space is uservec,then usertrap
and returning usertrapret and userret.Since the RISC-V doesn't switch page tables during a trap,the user page table must include a mapping for uservec. And in order to continue executing instructions after switch,uservec must be mapped at the same address in the kernel page table as in user page table.Xv6 satifies theses constraints with a trampoline page taht contains uservec.
## Code:Calling system call
>> initcode.S places teh arguments for exec in registers a0 and a1,and puts the system call number in a7.Sytem call number match the entries in the syscalls array,a table of function pointers.Then ecall instruction traps into the kernel and causes uservec,usertrap and then syscall to execute.
>> When sys_exec returns,syscall records its return value in p->trapframe->a0.This will casue the original user-space call to exec() to return that value.

## Code: System call arguments
>> Because user code calls system call wrapper functions, the argument are initially where the RISC-V C calling convention places them: in registers.The kernel trap code will save user registers to the current process's trap frame,where kernel code can find them using argint,argaddr and argfd.
>> kernel implements functions that safely transfer data to and from user-supplied addresses.fetchstr is an example, it invokes copyinstr. copyinstr uses walkaddr to look up srcva in pagetable,yielding physical address pa0.The kernel maps each RAM address to the corresponding kernel virtual address,so copyinstr can directly copy string bytes from pa0 to dst.
