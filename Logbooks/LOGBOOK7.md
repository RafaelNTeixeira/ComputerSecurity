# Trabalho realizado na Semana #7 - SEED Labs - Format-String Vulnerability Lab

This week's lab topic revolves around the printf() and other similar functions in C. These functions, when not properly sanitized, can easily be used to:
- crash the program
- read the internal memory of the program
- modify the internal memory of the program
- inject and execute malicious code using the victim program’sprivilege

## Setting up

Disabling address randomization:
`sudo sysctl -w kernel.randomize_va_space=0``

## Task 1: Crashing the Program

![Alt text](/Logbooks/img/Week7/task1_1.png)
![Alt text](/Logbooks/img/Week7/task1_2.png)

## Task 2:  Printing Out the Server Program’s Memory

### Task 2.A: Stack Data

![Alt text](/Logbooks/img/Week7/task2A_1.png)
![Alt text](/Logbooks/img/Week7/task2A_2.png)

The stack is found after 64 characters or 256 bytes.

### Task 2.B: Heap Data

## Task 3:  Modifying the Server Program’s Memory

### Task 3.A: Change the value to a different value.

### Task 3.B: Change the value to 0x5000.

### Task 3.C: Change the value to 0xAABBCCDD.


