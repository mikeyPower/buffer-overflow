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


## Protect against buffer overflow

## Examples of buffer overflows

## Reference

