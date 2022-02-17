

# George Papadopoulos - gip200@nyu.edu

LAB 1, PART 1 SUBMISSION
-------------

## **Task 1 (2pts): Getting Familiar with Shellcode**

In this part of the lab, we are asked to review C versions of shellcode and compile (via Makefile) 32 and 64 bit versions. Running make, we get two files, a32.out and a64.out. These two compiled programs can be run from the command line. In both cases, they generate a new shell that can run commands such as ls, who,etc. There is no "visible" difference from 32 to 64 bit binaries when they are run.

![enter image description here](https://github.com/gip200/gip200-appsec1/blob/main/Reports/Artifacts/gip200-lab1part1task1.jpg?raw=true)


## **Task 2 (3pts): Understanding the Vulnerable Program**

In this part of the lab, we are asked to review and compile the vulnerable stack.c program. It is implied that the program has a buffer overflow vulnerabity. We are asked to compile the program, (via Makefile)  which also compiles the debugging versions, as well as turning off the StackGuard and the non-executable stack protections, as well as set owner and mod permissions. We see a number of compiled versions of the stack executable, including numerous -dbg versions.

![enter image description here](https://github.com/gip200/gip200-appsec1/blob/main/Reports/Artifacts/gip200-lab1part1task2.jpg?raw=true)

## **Task 3 (10pts): Launching Attack on 32-bit Program (Level 1)**

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

![enter image description here](https://github.com/gip200/gip200-appsec1/blob/main/Reports/Artifacts/gip200-lab1part1task3.jpg?raw=true)


## **Task 4 - THERE IS NO TASK 4**

**

## **Task 5 (12pts): Launching Attack on 64-bit Program (Level 3)**

**

Not fully completed - we know from discussion the 64-bit registers need to be found using (R) like rbp instead of ebp. Here we comparitively see the offset of 208 for this 64-bit instance.

    nyuappsec@ubuntu:~/AppSec1/Part1/code$ gdb stack-L3-dbg 

    Reading symbols from stack-L3-dbg...
    gdb-peda$ b bof
    Breakpoint 1 at 0x1229: file stack.c, line 16.
    gdb-peda$ r
    Starting program: /home/nyuappsec/AppSec1/Part1/code/stack-L3-dbg 
    gdb-peda$ n
    22	    return 1;
    gdb-peda$ p $rbp
    $1 = (void *) 0x7fffffffd960
    gdb-peda$ p &buffer
    $2 = (char (*)[200]) 0x7fffffffd890
    gdb-peda$ p/d 0x7fffffffd960 - 0x7fffffffd890
    $3 = 208

## Task 6 - THERE IS NO TASK 6


**



## **Task 7 (5pts): Defeating dash's Countermeasure**

In this part, we revert the symbolic link for /bin/sh to dash shell. We then compare compiled binaries where both a32/64 binaries are run with the case without and with setuid enforced.
 
In the first case, without setuid, we see the shell enforces the uid of the logged in user and does not get elevation.

In the latter case, we recompile the code, prepending setuid support and then when the program is run, the elevation is to root(0), as shown, effectively bypassing the dash shell countermeasures. The symbolic links for the shell confirm it is definitely using bash.

![enter image description here](https://github.com/gip200/gip200-appsec1/blob/main/Reports/Artifacts/gip200-lab1part1task7.jpg?raw=true)


## **Task 8 (5pts): Defeating Address Randomization**

The brute force attack was able to be run using randomization. After 5512 iterations and 19 minutes and 22 seconds elapsed, the 32-bit L1 program was able to achieve a shell that could run commands.

![enter image description here](https://github.com/gip200/gip200-appsec1/blob/main/Reports/Artifacts/gip200-lab1part1task8.jpg?raw=true)



## **Tasks 9 (10pts): Experimenting with Other Countermeasures**

**Task 9.a: Turn on the StackGuard Protection**
In this section, we were advised to recompile the stack.c(L1, 32-bit) program that was used previously in our buffer overflow without turning off Stackguard as was defined in the Makefile, and see how this affects the executable.

In comparing the versions, we can see the version with StackGuard disabled was able to execute the buffer overflow, as expected. In the second instance of running the executable with StackGuard enabled (default), we see the error "stack smashing detected", so the StackGuard clearly detected the overflow attempt and terminates the executable.

![enter image description here](https://github.com/gip200/gip200-appsec1/blob/main/Reports/Artifacts/gip200-lab1part1task9a.jpg?raw=true)


**Task 9.b: Turn on the Non-executable Stack Protection**
In this section, we were advised to recompile the shellcode which produces the a32.out and a64.out. stack.c(L1, 32-bit) program that was used previously in our buffer overflow without "-z execstack", such that the OS is not told the binary will not be allowed to execute shelldcode.

In comparing the versions, we can see the version of shellcode.c a32 and a64 code, it executes our expected buffer overflow, as expected. In the second instance of running the executable without execstack bypass, we clearly see both 32/64 executables segfault/dump as they are explicitly not allowed to run shell code, which blocks the overflow attempt by strictly limiting the execution.

![enter image description here](https://github.com/gip200/gip200-appsec1/blob/main/Reports/Artifacts/gip200-lab1part1task9b.jpg?raw=true)

## END OF LAB 1, PART 1 SUBMISSION
