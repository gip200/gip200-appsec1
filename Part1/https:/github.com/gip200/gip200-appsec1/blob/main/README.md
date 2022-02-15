PART 1  
------

Task 1 (2pts): Getting Familiar with Shellcode

In this part of the lab, we are asked to review C versions of shellcode and compile (via Makefile) 32 and 64 bit versions. Running make, we get two files, a32.out and a64.out. These two compiled programs can be run from the command line. In both cases, they generate a new shell that can run commands such as ls, who,etc. There is no "visible" difference from 32 to 64 bit binaries when they are run.



Task 2 (3pts): Understanding the Vulnerable Program

In this part of the lab, we are asked to review and compile the vulnerable stack.c program. It is implied that the program has a buffer overflow vulnerabity. We are asked to compile the program, (via Makefile)  which also compiles the debugging versions, as well as turning off the StackGuard and the non-executable stack protections, as well as set owner and mod permissions. We see a number of compiled versions of the stack executable, including numerous -dbg versions.


Task 3 (10pts): Launching Attack on 32-bit Program (Level 1)

In this part of the lab, we use gdb to debug the compiled debug version of the stack binary. Running the debugger and setting breakpoint at bof, we run the stack binary and (setting breakpoint at bof as its vulnerable) and then run, we find ebp and start of buffer values. We found values to be $ebp=0xffffcb28, &buffer=0xffffcaac. Calculating the offset, $ebp-&buffer = 0x6c (108 in decimal). 

We set the return address  at 0xffffcb28+120 bytes from the buffer, with an offset of 112 (108+ 4 bytes, the size of frame pointer). Additionally, we need to know the start point is 517-length of shellcode, which is 32bit/x86 specific due to the variance in shell code size. With this knowledge, we can run the python script and generate the badfile, which can cause the buffer to overflow. Upon running the stack-L1, we can now see that we caused buffer overflow and have root and can run root commands such as id and whoami.

Legend: code, data, rodata, value
20	    strcpy(buffer, str);       
gdb-peda$ p $ebp
$1 = (void *) 0xffffcb28
gdb-peda$ p &buffer
$2 = (char (*)[100]) 0xffffcabc
gdb-peda$ p/d 0xffffcb28-0xffffcabc
$3 = 108




Task 5 (12pts): Launching Attack on 64-bit Program (Level 3)

Not fully completed -


Legend: code, data, rodata, value
20	    strcpy(buffer, str);       
gdb-peda$ p $rbp
$1 = (void *) 0x7fffffffd960
gdb-peda$ p &buffer
$2 = (char (*)[200]) 0x7fffffffd890
gdb-peda$ p/d 0x7fffffffd960-0x7fffffffd890
$3 = 208



Task 7 (5pts): Defeating dash's Countermeasure

In this part, we revert the symbolic link for /bin/sh to dash shell. We then compare compiled binaries where both a32/64 binaries are run with the case without and with setuid enforced.
 
In the first case, without setuid, we see the shell enforces the uid of the logged in user and does not get elevation.

In the latter case, we recompile the code, prepending setuid support and then when the program is run, the elevation is to root(0), as shown, effectively bypassing the dash shell countermeasures. The symbolic links for the shell confirm it is definitely using bash.

 







Task 8 (5pts): Defeating Address Randomization
Started 2:10PM 2/14


Tasks 9 (10pts): Experimenting with Other Countermeasures



