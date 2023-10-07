# Trabalho realizado na Semana #4 - Environment Variable and Set-UID Lab

For this week we where proposed to do tasks presented on this link:
https://seedsecuritylabs.org/Labs_20.04/Software/Environment_Variable_and_SetUID/

## Task 1: Manipulating Environment Variables
1. Use printenv/env to print out the environment variables

![PrintenvEnv](/Logbooks/img/Week4/Task1.1.png)

2. Use export and unset to set or unset environment variables

![ExportUnset](/Logbooks/img/Week4/Task1.2.png)


## Task 2: Passing Environment Variables from Parent Process to Child Process
1. We compiled and ran the myprintenv.c file by typping "gcc myprintenv.c". This program could be found in the Labsetup folder when cloning this repository https://github.com/seed-labs/seed-labs. With this, a file a.out was generated and we ran it too and saved its output into a file using "a.out > file".

![Compile](/Logbooks/img/Week4/Task2.1.png)

2. For step 2, we now commented the printenv() statement in the child case and uncommented the printenv() in the parent process case. By compiling and running the code again, we discovered with the help of the diff command that the output was the same for both programs, meaning the environment variables are the same for a parent process and its child process when created through the default configuration of fork

![ResultComparison](/Logbooks/img/Week4/Task2.2.png)


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

1. Compiled and ran myenv.c, program made to print out the environment variables of the current process.

![CompileTask3](/Logbooks/img/Week4/Task3.1.png)

2. There was no output because since execve() receives NULL in the third argument (array of environment variables), the new process has no environment variables 

![Output1](/Logbooks/img/Week4/Task3.2.png)

3. Changed `execve("/usr/bin/env", argv, NULL);` to `execve("/usr/bin/env", argv, environ);` on the file and compiled and ran it. This time there was output since we inserted the pointer to the array of the parent processe's environment variables (environ) in the third argument of the function execve, printing out the environment variables of the parent's process.

![Output2](/main/Logbooks/img/Week4/Task3.3.png)

**myenv.c:**

```c++
#include <unistd.h>

extern char **environ;

int main()
{
  char *argv[2];

  argv[0] = "/usr/bin/env";
  argv[1] = NULL;

  execve("/usr/bin/env", argv, environ);  

  return 0 ;
}
```



















