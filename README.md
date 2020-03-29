# buffer-overflow

## Overview

Buffer overflow errors are characterized by the overwriting of adjacent memory locations of the process, which should have never been modified intentionally or unintentionally. Overwriting values of the IP (Instruction Pointer), BP (Base Pointer) and other registers causes exceptions, segmentation faults, and other errors to occur. Usually these errors end execution of the application in an unexpected way. Buffer overflow errors occur when we operate on buffers of char type.

Buffer overflows can consist of overflowing the stack (Stack overflow) or overflowing the heap (Heap overflow).

## What is a Buffer

A buffer, in terms of a program in execution, can be thought of as a region of computer’s main memory that has certain boundaries in context with the program variable that references this memory.

For example :

    char buff[10]
    
In the above example, ‘buff’ represents an array of 10 bytes where buff[0] is the left boundary and buff[9] is the right boundary of the buffer.


## Why are buffer overflows harmful

Generally, exploitation of these errors may lead to:

application DoS
reordering execution of functions
code execution (if we are able to inject the shellcode, described in the separate document)
How are buffer overflow errors are made?


## Protect against buffer overflow
These kinds of errors are very easy to make. For years they were a programmer’s nightmare. The problem lies in native C functions, which don’t care about doing appropriate buffer length checks. Below is the list of such functions and, if they exist, their safe equivalents:

gets() -> fgets() - read characters
strcpy() -> strncpy() - copy content of the buffer
strcat() -> strncat() - buffer concatenation
sprintf() -> snprintf() - fill buffer with data of different types
(f)scanf() - read from STDIN
getwd() - return working directory
realpath() - return absolute (full) path
Use safe equivalent functions, which check the buffers length, whenever it’s possible. Namely:

gets() -> fgets()
strcpy() -> strncpy()
strcat() -> strncat()
sprintf() -> snprintf()
Those functions which don’t have safe equivalents should be rewritten with safe checks implemented. Time spent on that will benefit in the future. Remember that you have to do it only once.

There are many functions that do the exact same thing—these are known as unbounded functions because developers cannot predict when they will stop reading from or writing to memory. Microsoft even has a web page documenting what it calls “banned” functions, which includes these unbounded functions.

Use compilers, which are able to identify unsafe functions, logic errors and check if the memory is overwritten when and where it shouldn’t be.

## Running 

In order to compile this program run the below line

    gcc -Wall -g -fno-stack-protector buffer_overflow.c -o buffer_overflow
    
You may notice that this line contains a flag
    
    -fno-stack-protector
    
This flag emits extra code to check for buffer overflows, such as stack smashing attacks. If we were to compile the exploit without the
flag, the gcc compiler would in most cases be able to prevent the buffer over.

Now once we've compiled our code we can run the output

    ./buffer_overflow


## Results

We will then be presented with a screen to input our password and depending on what we input the output will either give us root privileges or deny this.

    Enter the password : 
    fdsfds

    Wrong Password 

This seems like our program is working correctly however if we enter a slightly longer password see what happens

    Enter the password : 
    aaaaaaaaaaaaaaaaaaaaaaaaaaaaa

    Wrong Password 

    Root privileges given to the user 
    
Hold on why did the program give root privileges to an incorrect password to understand this we have to enter the memory of our c program.

    root@kali:~/buffer-overflow# gdb buffer_overflow 
    (gdb) list
    3       #include <signal.h>
    4
    5       int main(void)
    6       {
    7           char buff[15];
    8           int pass = 0;
    9
    10          printf("\n Enter the password : \n");
    11          gets(buff);
    12
    
Here we are interested in the variable pass which is set to one, now in order to find the memory location of where this is stored
we can execute the following command
   
    (gdb) break 8
    Breakpoint 1 at 0x116d: file buffer_overflow.c, line 8.
    
This will set up a breakpoint at line 8 so that we can see the memory location of this variable once we run the program

    (gdb) run buffer_overflow
    Starting program: /root/buffer-overflow/buffer_overflow buffer_overflow

    Breakpoint 1, main () at buffer_overflow.c:8
    8           int pass = 0;
    (gdb) p &pass
    $1 = (int *) 0x7fffffffe13c
    
    
Now we know the memory location of the variable pass lets finish the running of this programe

    (gdb) continue
    Continuing.

    Enter the password : 
    aaaaaaaaaaa

    Wrong Password 

    Program received signal SIGINT, Interrupt.
    __GI_raise (sig=<optimized out>) at ../sysdeps/unix/sysv/linux/raise.c:50
    50      ../sysdeps/unix/sysv/linux/raise.c: No such file or directory.
    (gdb) info frame
    Stack level 0, frame at 0x7fffffffe120:
    rip = 0x7ffff7e30761 in __GI_raise (../sysdeps/unix/sysv/linux/raise.c:50); saved rip = 0x5555555551e5
    called by frame at 0x7fffffffe150
    source language c.
    Arglist at 0x7fffffffdff8, args: sig=<optimized out>
    Locals at 0x7fffffffdff8, Previous frame's sp is 0x7fffffffe120
    Saved registers:
    rip at 0x7fffffffe118
    (gdb) x/200x 0x7fffffffdff8
    .....
    0x7fffffffe128: 0x55555080      0x61616155      0x61616161      0x61616161
    0x7fffffffe138: 0x00000000      0x00000000      0x555551f0      0x00005555
    .....


After entering a series of a's (11 in face) we can se this represented between the two memory location as 61 in hex, let's keep increasing this in order to corrupt the value stored at our pass variable. Or an easier solution would be to subtract the two memory locations of the buff array and pass variable.

    (gdb) p &buff
    $8 = (char (*)[15]) 0x7fffffffe12d
    (gdb) p &pass
    $9 = (int *) 0x7fffffffe13c
    0x7fffffffe13c - 0x7fffffffe12d = F
    
The difference between the two locations is 15 bytes or F in Hexadecimal. So in order to change the value of pass we need to enter a input of at least 16 to change the value of pass in order to give us root privileges.

    Enter the password : 
    aaaaaaaaaaaaaaa

    Wrong Password 

    Program received signal SIGINT, Interrupt.
    
It worked and if we inspect the memory of our programme now we can see that the value of pass has changed.
     
    ...
    0x7fffffffe128: 0x55555080      0x61616155      0x61616161      0x61616161
    0x7fffffffe138: 0x61616161      0x00000061      0x555551f0      0x00005555
    ...
    (gdb) x/s 0x7fffffffe13c
    0x7fffffffe13c: "a"


This is because the array "buf" which is limited to 15, since we gave it an input of 16 the extra character a ran over and into the pass variable thus now "pass" contains the value of "a". 


## Reference

1. https://www.thegeekstuff.com/2013/06/buffer-overflow/
2. https://blog.rapid7.com/2019/02/19/stack-based-buffer-overflow-attacks-what-you-need-to-know/
3. https://owasp.org/www-community/attacks/Buffer_overflow_attack


