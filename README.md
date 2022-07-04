# MIT S081 charpter1 Operating System interface
## job of operating system
>>sharing computer among multi programs
>>operating system providing services to user programs through interface
<img src="https://github.com/WeihaoZhang/weihao.github.io/blob/main/ch1/1.png" alt="kerne program" style="zoom:25%;" />
>> user program can invoke system call to used service provided by kernel.user program can access its own memory,but kernel can implement protections with hardware privileges.shell is a user program read comands from user then execute them using syscall, illustrating the powerful user interface.

---

## processes and memory
>>a process consists of user-space memory (instructions,data and stack) and per-process state private to the kernel.
>>fork: a system call and create a new process same as the origin one.Fork returns new process pid in the origin process and zero in the new process.wait(*status) will return the pid of child process and pass its exit status to wait function. 
>>Alough parent and child process has the same contents initially, they are executing with different memory and registers,so changing parameter in one doesn't affect anothor.
>>
>><img src="https://github.com/WeihaoZhang/weihao.github.io/blob/main/ch1/2.png" alt="image-20220702224242801" style="zoom: 50%;" />
>>
>>The exec system call replaces the calling process's memory with a new image loaded from a file in file system.when exec fucttion successs,it doesn't return;instead,the instructions loaded from the file executing from the entry point of the elf header.It has two paras,one is path to the excuting file,another is an array of arguments.
>>
>><img src="https://github.com/WeihaoZhang/weihao.github.io/blob/main/ch1/image-20220702224336538.png" alt="image-20220702224336538" style="zoom:50%;" />
>>
>>The following pics shows how shell works.
>>
>><img src="https://github.com/WeihaoZhang/weihao.github.io/blob/main/ch1/image-20220702230049979.png" alt="image-20220702230049979" style="zoom: 25%;" />
>>
>><img src="https://github.com/WeihaoZhang/weihao.github.io/blob/main/ch1/image-20220702230126230.png" alt="image-20220702230126230" style="zoom: 25%;" />
>>
>>
>>
>>when fork works,it use cow(copy on write) tech,so only when child process change content,a new physical page will be allocated.But user space is implicity allocated.fork ,exec will prepare enough space ,and when running program need more space (such as malloc),sbrk(n) can grow nbytes and return the address of allocatetd space.

---

## IO and file descriptors

> > File desciptors are obtained by opening:
> > >- file
> > >
> > >- dirtectory
> > >
> > >- pipe
> > >
> > >- device
> > >-  coping a descriptor
> >
> > Process can operating device or read/write file by reading or writing bytes to the descriptor .File desciprtors are the indexs to a per-process table . By convention,a process reads from file descriptor 0(standard input),write out to 1(standard output),write error message to file descriptor2(standard error).
> > Read(fd,buff,n) can read n bytes data to buff from descriptor fd and return the bytes read. And eache file desciptor refer to a file has an offset associated with it ,read function reads bytes from the offset.Read will return 0 when threre are no more bytes to read.Write(fd,buff,n) is similar as read,writes n bytes  from current offset and advance the offset by n bytes.Fowllowing pics are the code fragment of cat program.
> > 
> > <img src="https://github.com/WeihaoZhang/weihao.github.io/blob/main/ch1/image-20220703104650672.png" alt="image-20220703104650672" style="zoom:50%;" />
> > 
> > <img src="https://github.com/WeihaoZhang/weihao.github.io/blob/main/ch1/image-20220703104750069.png" alt="image-20220703104750069" style="zoom:50%;" />
> > 
> > Close op will release the descriptor.And a new allocated file fescriptor is always the lowest numbered unused descriptor of current process.Cat exploits this characteristic to descriptor redirection.So seperating fork and exec , shell has the chance to redirect the child's I/O without distubring I/O setup of main shell.
> >
> > <img src="https://github.com/WeihaoZhang/weihao.github.io/blob/main/ch1/image-20220703110359247.png" alt="image-20220703110359247" style="zoom:50%;" />
> >
> > Descriptor origins from the same descriptor sharing sam offset.Following program will writing hello world to file descriptor.And ls exsitring-file non-exsiting-file \> tmp1 2\>&1. 2\>&1 tells shell sends cmd to descriptor 2 to that is a duplication of descriptor 1,and exsiting file and error message wiil show up in tmp1 file.
> >
> > <img src="https://github.com/WeihaoZhang/weihao.github.io/blob/main/ch1/image-20220703113042189.png" alt="image-20220703113042189" style="zoom:50%;" />

---

## pipes
>>A pipe is a small kernel buffer exposed to processes as a pair of file descriptors.Pipes provide a way for processes to communicate.
>>In next case, leftcmd | rightcmd 's pipeline is implemented.The process create a child process using fork and then runcmd for the left.For right one,the process does similar.And the parent process will wait both to finish.
>>
>><img src="https://github.com/WeihaoZhang/weihao.github.io/blob/main/ch1/image-20220703202623528.png" alt="image-20220703202623528" style="zoom:50%;" />
>
>Alough pipe is no more powerful than using temporary file,there are three advantags over temporary files.
>
>>>- no need to clean files
>>>- passing arbitrarily long streams of data
>>>- pipes allow parallel execution of pipeline satges, so for multiprocess situations,pipes are more efficient.

---

## File system

>> The directories form a tree,starting at a special directory called root. Syscall chdir change the process's current working directory.Syscall open with O_CREATE para will create a file if it doesn't exists.Mknod can create a special file refering to a device ,associated with a major and minor device number.

<img src="https://github.com/WeihaoZhang/weihao.github.io/blob/main/ch1/image-20220703212315514.png" alt="image-20220703212315514" style="zoom:50%;" />

>> The same underling file ,called an inode,can have multiple names,called links.The syscall retrieves information from the inode that a file descriptor refers to.It fills in a struct stat,including dev (File system's disk device)

<img src="https://github.com/WeihaoZhang/weihao.github.io/blob/main/ch1/image-20220703213713629.png" alt="image-20220703213713629" style="zoom:50%;" />

>> The link system call creates another file system name referring to the same inode as an existing file.Such as link("a","b").Unlink syscall will removes a name from the file system, and inode will be deleted when nlink is 0.So using open fuc create a file then unlink its name will be a idiomatic way to create a temporary inode with no name. And this inode will be cleaned up when the process cloeses fd or exits.
>> Mkdir ,rm and etc are the user-level programs callable from the shell.This design allow anyone exten the command-line by adding new user-level program.But cd is an out of expection because it will fork a child process and changing work dir in child process,not in the shell.

---

## Real Word

>> Unix system introducing the idea that combining file descriptors,pipes adn convient shell syntax , which is a major advance in writing general-purpose reusable system. And BSD,Linux,MacOs are derived from Unix. Unix system call interface has been standardized through the POSIX interface. xv6 is not POSIX compliant ,so missing some system call is ok.
