# Trabalho realizado na Semana #4 - Environment Variable and Set-UID Lab

For this week we where proposed to do tasks presented on this link:
https://seedsecuritylabs.org/Labs_20.04/Software/Environment_Variable_and_SetUID/

## Task 1: Manipulating Environment Variables
1. Use printenv/env to print out the environment variables

![PrintenvEnv]()

2. Use export and unset to set or unset environment variables

![ExportUnset]()


## Task 2: Passing Environment Variables from Parent Process to Child Process
1. We compiled and ran the myprintenv.c file by typping "gcc myprintenv.c". This program could be found in the Labsetup folder when cloning this repository https://github.com/seed-labs/seed-labs. With this, a file a.out was generated and we ran it too and saved its output into a file using "a.out > file".

![Compile]()

2. For step 2, we now commented the printenv() statement in the child case and uncommented the printenv() in the parent process case. By compiling and running the code again, we discovered with the help of the diff command that the output was the same for both programs, meaning the environment variables are the same for a parent process and its child process when created through the default configuration of fork

![ResultComparison]()


**myprintenv.c:**

```c++
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>

extern char **environ;

void printenv()
{
  int i = 0;
  while (environ[i] != NULL) {
      printf("%s\n", environ[i]);
      i++;
  }
}

void main()
{
  pid_t childPid;
  switch(childPid = fork()) {
    case 0:  /* child process */
      // printenv();          
      exit(0);
    default:  /* parent process */
      printenv();       
      exit(0);
  }
}
```


## Task 3: Environment Variables and execve()























