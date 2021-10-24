# Semaphore synchronization

***
**Introduction**
---------------------------------------

> &nbsp;&nbsp;&nbsp;&nbsp;The goal of this lab is make you familiar with semaphores and use them to coordinate and synchronize among processes. The concept introduced in this lab is also applied to multithreads.
Every operating systems provides a set of synchronization commands (system calls), called IPC (Inter Process Communication).
Although the functionality and signatures are different in different O.S., they are basically similar. Please note that, this lab can only be done in Linux. I have to prepare the Linux environment for the labs. 

> &nbsp;&nbsp;&nbsp;&nbsp;In the provided files, there are two sample programs, *prog1.c* and *prog2.c*. Please compile them and make them executable in the Linux. *prog1.c* is very simple. It starts by creating a semaphore.
The name of semaphore is composed by two parts. One is path and another is project id. In prog1.c, it uses create_sem to create a semaphore with name “.”+”S” and its initial value is 0.

```diff
- IMPORTANT:In principle, a semaphore is created for any other processes (including other users) to access it. 
- Any processes can access the semaphore once they know the path+name.
- So, if you plan to run this prog1 and prog2 in the same operating systems with other, you better clear it before doing so. 
- Another better way is to rename the semaphore so that no one can access the same semaphore with you.
```
> &nbsp;&nbsp;&nbsp;&nbsp;After the creation of a semaphore, *prog1* calls *P(semid)*, which in some OS books called *wait(semid)*.
However, the original Linux system calls of semaphore are not *P(s)* nor *wait(s)*. Interested readers can open *sem.c* to see how the real Linux system calls are wrapped into *P(s)* and *V(s)*.
> &nbsp;&nbsp;&nbsp;&nbsp;Because *semid* is initialized as 0, so according to the semantics of a semaphore, *prog1* should block on the *P(semid)*. After *prog1* enters a blocking state, you can run *prog2*. 
The code of *prog2* is very simple too. It uses the name “.”+”S” to find the semaphore created by *prog1* and then call *V(semid)* to release a blocked process from semaphore *semid*. 
In this case, it is *prog1* to be woken up. This is the *signal(S)* in OS text books. This call will wake up *prog1* and then both *prog1* and *prog2* run concurrently to the end.

## Sample Runs
### To run *prog1* and *prog2*, please compile the program as follows:
```sh
$ gcc –o  prog1  prog1.c sem.c
$ gcc –o  prog2  prog2.c sem.c
```

### Please run prog1 in Linux as follows:
```sh
$ prog1 &
```
> &nbsp;&nbsp;&nbsp;&nbsp;The purpose of “&” is to tell your shell to run your program in background without blocking your shell. 
So, although prog1 is immediately blocked by semaphore, your shell is still alive for you to continue entering commands. 

### Next, please run
```sh
$ prog2 &
```

### **Below shows how to compile and execute the programs**


Compile and execute Order : p1 p2 p3


## Compile commands
```sh
$ gcc p1.c sem.c -o p1
$ gcc p2.c sem.c -o p2
$ gcc p3.c sem.c -o p3

```

***
## Execute commands
```sh
$./p1 &
$./p2 &
$./p3 &
```


### **Below shows how to compile and execute the programs**
Compile and execute Order : p1 p3 p2

## Compile commands

>same as previous one.
```sh
$ gcc p1.c sem.c -o p1
$ gcc p2.c sem.c -o p3
$ gcc p3.c sem.c -o p2
```

***
## Execute commands
```sh
$./p1 &
$./p3 &
$./p2 &
```

***
## Check semaphore id and counts
```sh
$ ipcs
```


***
## Remove semaphore id (when execute one round, do this!)
```sh
$ ipcrm sem **id**
```
 - **id** stands for the semaphore id.





