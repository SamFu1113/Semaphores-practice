# Three processes synchronization using semaphores

***
**Introduction**
---------------------------------------

&nbsp;&nbsp;&nbsp;&nbsp;The goal of this project is make me familiar with semaphores and use them to coordinate and synchronize among processes. The concept introduced in this project is also applied to multithreads.
Every operating systems provides a set of synchronization commands (system calls), called IPC (Inter Process Communication).
Although the functionality and signatures are different in different OS, they are basically similar. Please note that, this project can only be done in Linux. I have to prepare the Linux environment for the labs. 

&nbsp;&nbsp;&nbsp;&nbsp;In the provided files, there are two sample programs, *prog1.c* and *prog2.c*. Please compile them and make them executable in the Linux. *prog1.c* is very simple. It starts by creating a semaphore.
The name of semaphore is composed by two parts. One is path and another is project id. In *prog1.c*, it uses create_sem to create a semaphore with name “.”+”S” and its initial value is 0.

```diff
- IMPORTANT:In principle, a semaphore is created for any other processes (including other users) to access it. 
- Any processes can access the semaphore once they know the path+name.
- So, if I plan to run this 'prog1' and 'prog2' is in the same operating systems with the other, I better clear it before doing so. 
- Another better way is to rename the semaphore so that no one can access the same semaphore with me.
```
&nbsp;&nbsp;&nbsp;&nbsp;After the creation of a semaphore, *prog1* calls *P(semid)*, which in some OS books called *wait(semid)*.
However, the original Linux system calls of semaphore are not *P(s)* nor *wait(s)*. Interested readers can open *sem.c* to see how the real Linux system calls are wrapped into *P(s)* and *V(s)*.  
&nbsp;&nbsp;&nbsp;&nbsp;Because *semid* is initialized as 0, so according to the semantics of a semaphore, *prog1* should block on the *P(semid)*. After *prog1* enters a blocking state, I can run *prog2*. 
The code of *prog2* is very simple too. It uses the name ```“.”+”S”``` to find the semaphore created by *prog1* and then call *V(semid)* to release a blocked process from semaphore *semid*. 
In this case, it is *prog1* to be woken up. This is the *signal(S)* in OS text books. This call will wake up *prog1* and then both *prog1* and *prog2* run concurrently to the end.

## Sample Runs

### To run *prog1* and *prog2*, please compile the program as follows:
```sh
gcc –o  prog1  prog1.c sem.c
```

```sh
gcc –o  prog2  prog2.c sem.c
```

### Please run *prog1* in Linux as follows:
```sh
./prog1 &
```
&nbsp;&nbsp;&nbsp;&nbsp;The purpose of “&” is to tell my shell to run my program in background without blocking my shell. 
So, although *prog1* is immediately blocked by semaphore, my shell is still alive for me to continue entering commands. 
### Next, please run
```sh
./prog2 &
```
&nbsp;&nbsp;&nbsp;&nbsp;*Prog2* should wake up *prog1* and then both run to the end concurrently.  
&nbsp;&nbsp;&nbsp;&nbsp;After *prog1* and *Prog2* finished, remember that semaphore still exist. I can use *ipcs* command to list the IPC resources I own. 
If I want to delete the semaphore manually, I can use *icprm sem semid*, where semid is the semaphore id listed by ipcs.  
&nbsp;&nbsp;&nbsp;&nbsp;I can try a series of runs as follows 
```sh
./prog1 &
```
```sh
./prog1 &
```
```sh
./prog1 &
```
&nbsp;&nbsp;&nbsp;&nbsp;After that, I block three *prog1* on the semaphore. I can use *jobs* to list how many background processes are in the background.
Next, once I execute ```./prog2 &```, it will wake up one earlier blocked *prog1*. So I need to run *prog2* three times to wake up all the blocked *prog1*. 
When *prog1* is woken up, it shall print some message and their process id. So I should observe which process is woken up.

## Project goal
&nbsp;&nbsp;&nbsp;&nbsp;In this project, three files *p1.c* *p2.c* and *p3.c* are provided.  They are incomplete but simple.Each one will print a message. Please assume *p1.c* is always executed first. 

&nbsp;&nbsp;&nbsp;&nbsp;That is, *p1.c* is responsible for creating semaphore and *p2*, *p3* are not responsible for creating semaphores. My goal is to use semaphores to coordinate p1,p2,p3 so that p1 prints message once, p2 prints message once, and then p3 prints message twice.

&nbsp;&nbsp;&nbsp;&nbsp;They loop forever until loop exists. That is, suppose I run ```./p1 &``` ```./p2 &``` ```./p3 &```, the program output should be</br>

```sh
P1111111
P2222222
P3333333
P3333333
P1111111
P2222222
P3333333
P3333333
```


### **Below shows how to compile and execute the programs**


Compile and execute Order : ```p1 p2 p3```


## Compile commands
```sh
gcc p1.c sem.c -o p1
```
```sh
gcc p2.c sem.c -o p2
```
```sh
gcc p3.c sem.c -o p3
```

***
## Execute commands
```sh
./p1 &
```
```sh
./p2 &
```
```sh
./p3 &
```

---------------------------------

### **Below shows another way how to compile and execute the programs**

> Test my program by different order such as below explanation:

Compile and execute Order : ```p1 p3 p2```, but the program output should be the same as previous one. 

## Compile commands

>The commands are same as previous one but the order is differnet.
```sh
gcc p1.c sem.c -o p1
```
```sh
gcc p2.c sem.c -o p3
```
```sh
gcc p3.c sem.c -o p2
```

***
## Execute commands
```sh
./p1 &
```
```sh
./p3 &
```
```sh
./p2 &
```

***
## Check semaphore id and counts
```sh
ipcs
```


***
## Remove semaphore id (when execute one round, do this!)
```console
ipcrm sem id
```
 - **id** stands for the semaphore id.
 
 
## Notes
&nbsp;&nbsp;&nbsp;&nbsp;In order to achieve the goal, I need to think of some ways to use semaphore to coordinate the processes. Try to let *p1* prints message once first.  After that, *p2* is allowed to print message. Next, *p3* is allowed to print message twice.

**NOTE THAT**, someone may test my program by different order such as:
```p1 & ; p3 & ; p2 &``` but my program output should be the same.

**NOTE THAT**, when someone tests my program, he(she) will use ipcrm to clear any existing semaphores. So, I should do so as well. Clear all the semaphores by ipcrm before running my program. 

```diff
- IMPORTANT CONSTRAINTS:
To coordinate p1, p2 and p3, **I'm only allowed to use P(), V()** for the lab. I'm not allowed to use any other statements such as IF or use additional variables, etc. 
If I use other commands other than P() and V(), I will get 0 points.  
1. remember to killed bad processes when my experiment fails
2. remember to clear my semaphore by ipcrm. 
```

### HINT
&nbsp;&nbsp;&nbsp;&nbsp;In order to achieve the synchronization, please think how many semaphores should be used. Then use ```create_sem()``` in *p1.c* to create all the semaphores and their initial values. Then, in *p2*, *p3*, before the loop, use ```get_sem()``` to read the semaphores created by *p1*. 
&nbsp;&nbsp;&nbsp;&nbsp;The initial values of semaphores are very important. Semaphore values must be greater than 0. There are no points in setting a negative semaphore value. Before and after the *printf* statements, please use ```P()```, ```V()``` to achieve your goal. Good luck.  
&nbsp;&nbsp;&nbsp;&nbsp;In ```sem.c```, the following two functions are provided for me to observe and debug. I cannot use them in the final program. If I use, 0 point will be graded.  
&nbsp;&nbsp;&nbsp;&nbsp;```get_blocked_no(semid)```: return the number of processes  
&nbsp;&nbsp;&nbsp;&nbsp;```get_sem_val(semid)```: return the semaphore value of semid. In UNIX semaphore, the semaphore value always greater than 0 but in OS text book, the semaphore value can be negative.  

***May force be with you.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;~ Yoda，the Master***

