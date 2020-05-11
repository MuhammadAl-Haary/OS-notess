# OPERATING SYSTEM
### Operating system is an abstract machine
* Extends the basic hardware with added functionality
* Provides high-level abstractions
	* Programmer friendly
	* Common core for all application
	> E.g. Filesystem instead of just registers on a disk controller
* It hides the details of the hardware
	* Makes application code portable

### Operating system is a resource manager
* Responsible for allocating resources to users and processes
* Must ensure the following:
	* No starvation
	* Progress
	* Allocation is according to some desired policy
	> FIFO, fair share, weighted fair share, limit(quotas), etc..
* Overall, the system is efficiently used

## Operating system kernel 
* Also called *nucleus* or *supervisor*
* Portion of the operating system that is running on *privileged mode*
* Usually resident (stays) in the main memory
* Contains fundamental functionality
	* Whatever required to implement other services
	* Whatever required to provide security
* Contains most-frequently used functions

## Operating system is privileged
* Application should not be able to interfere or bypass the operating system
	* OS can enforce the *extended machine*
	* OS can enforce its resource allocation policies
	* Prevent applications from interfering with each other
> "*Assuming the application is mallicious*"

### Structure of the computer system
* OS interacts via load and store instructions to all memory, CPU and device registers, and interrupts.
* Applications interact with themselves via function calls to library procedures.
* OS and Applications interact via *system calls*

## Previlege-less OS
* Some embedded OSs have no privileged component
	* PalmOS, Mac OS 9, RTEMS
* Can implement OS functionality but cannot enforce it
	* All software runs together
	* No isolation
	* ***One fault potentially brings down entire file system***

> **System libraries** are just libraries of support functions (procedures, subroutines) <br />
> Only a subset of library functions are actually system calls <br />
> *strcmp, memcpy* are examples of *pure library functions* (for convenience) <br />
> **open, close, read, write** are system calls: They cross the user-kernel boundary. Implementation focused on passing request to OS and returning result to application.

## Operating system software
* Fundamentally, OS functions the same way as ordinary computer software
	* It is machine code that is executed (same machine instructions as application)
	* It has more privileges (extra instructions and access)
* Operating system relinquishes control of the processor to execute other programs
	* Reestablishes control after
		* System calls
		* Interrupts (expecially timer intterupts)

## Operating system structure
* File system
* Memory manager 
* Network interface
* Process Scheduler
* Inter-process communication
* Initialization
* Library
> *Spaghetti-ness → all are interconnected*

#Checkpoint
1. What are some differences between a processor running in *privileged mode (kernel mode)* and *user mode*? Why are two modes needed?
	* In user mode:
		* CPU control registers are inaccessible
		* CPU management instructions are inaccessible
		* Parts of the address space (containing kernel code and data) are inaccessible
		* Some device memory and registers (or ports) are inaccessible
	* Two modes of operations are required to ensurerd that applications (running in user-mode) cannot bypass, circumvent, or control the operating system
2. What are the two main roles of the Operating system?
	1. Provie a high-level abstract machine for programmers (hide details of hardware)
	2. Manage resources among competing processes or users according to some policy
3. Given a high-level understanding of fine systems, explain how a file system fulfills the two roles of an operating system
	* At the level of hardware, storage involves low-level controller hardware and storage devices that store blocks of data at many locations in the store. The OS filesystem abstracts above all these details and provides an interface to store, name, and organise arbitrarily unstructured data
	* The filesystem also arbitrates betweem competing processor by managing allocated and free space on the storage device, in addition to enforcing limits on storage consumption
4. Which of the following instructions (or instruction sequences) should only be allowed in kernel mode?
	1. Disable all interrupts.
	2. Read the time of day clock.
	3. Set the time of day clock.
	4. Change the memory map.
	5. Write to the hard disk controller register.
	6. Trigger the write of all buffered blocks associated with a file back to disk (fsync).
	* 1, 3, 4, 5 must be restricted to kernel mode
```c
#include <sys/types.h>
#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>
#define FORK_DEPTH 3
main(){
  int i, r;
  pid_t my_pid;
  my_pid = getpid();
  for (i = 1; i <= FORK_DEPTH; i++) { 
    r = fork();   
    if (r > 0) {
      /* we're in the parent process after
         successfully forking a child */     
      printf("Parent process %d forked child process %d\n",my_pid, r);     
    } else if (r == 0) {   
      /* We're in the child process, so update my_pid */
      my_pid = getpid()   
      /* run /bin/echo if we are at maximum depth, otherwise continue loop */
      if (i == FORK_DEPTH) { 
        r = execl("/bin/echo","/bin/echo","Hello World",NULL);  
        /* we never expect to get here, just bail out */
        exit(1);
      }
    } else { /* r < 0 */
      /* Eek, not expecting to fail, just bail ungracefully */
      exit(1);
    }
  }
}
```
5. The following code contains the use of typical UNIX process management system calls: fork(), execl(), exit() and getpid(). If you are unfamiliar with their function, browse the man pages on a UNIX/Linux machine get an overview, e.g: man fork. Answer the following questions about the code below.
	1. What is the value of i in the parent and child after fork.
		* The child is a new independent process that is a copy of the parent. the *i* in the child will be whatever value the parent will have at the point of forking
	2. What is the value of my_pid in a parent after a child updates it?
		* *my_pid* in parent is not updated by any action performed by the child as they are independent after forking
	3. What is the process id of /bin/echo?
		* *execl* replaces the content of a running proces with a specified executable. The process id does not change
	4. Why is the code after execl not expected to be reached in the normal case?
		* A sucessful *execl* results in replacement of current code. *execl* does not return if it succeeds as there is no previous code to return to.
	5. How many times is Hello World printed when FORK_DEPTH is 3?
		* 4 times
	6. How many processes are created when running the code (including the first process)?
		* 8 Processes
```c
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
char teststr[] = "The quick brown fox jumps over the lazy dog.\n";
main(){
  int fd;
  int len;
  ssize_t r;
  fd = open("testfile", O_WRONLY | O_CREAT, 0600);
  if (fd < 0) {
    /* just ungracefully bail out */
    perror("File open failed");
    exit(1);
  }
  len = strlen(teststr);
  printf("Attempting to write %d bytes\n",len);
  r = write(fd, teststr, len);
  if (r < 0) {
    perror("File write failed");
    exit(1);
  }
  printf("Wrote %d bytes\n", (int) r);
  close(fd);
}
```
6. Answer the following:
	1. What does the following code do?
		* Writes a string to a file. It will create a new file if needed (O_CREAT)
	2. In addition to O_WRONLY, what are the other 2 ways one can open a file?
		* O_RDONLY, O_RDWR
	3. What open return in fd, what is it used for? Consider success and failure in your answer.
		* On failure, fd is set to -1 to signify error. On success, fd is set to an integer and is used to handle the file and identify the file in other file related system cases.
7. What does **strace** do?
	* *strace* prints trace of all system calls invoked by a process, together with the arguments to the system call.
8. *printf* does not appear in the system call trace. Why?
	* *printf* is a library function that creates a buffer based on the string specification that it passed. The buffer is then written to the console using *write()* to fd 1
9. On UNIX, which of the following are considered system calls? Why?
	1. read()
	2. printf()
	3. memcpy()
	4. open()
	5. strncpy()
	* 1 and 4 are system calls; 2 is a *c* library function; 3 and 5 are simple library functions


# Processes and threads
## Major requirements of an operating system
* Interleave the execution of several processes to maximize processor utilization while providing reasonable response time
* Allocate resources to processes
* Support interprocess communication and user creation of processes

## Processes
* Also called *task* or *job*
* Execution of an individual program
* *Owner* of resources allocated for program execution
* Encompasses one or more threads

## Threads 
* Unit of execution
* Can be traced
	* List the sequence of instructions that execute
* Belongs to a process
	* Executes within it

## Process and thread models of selected Operating Systems
* Single process, single thread
	* MSDOS
* Single process, multiple thread
	* OS161 as distributed
* Multiple processes, single thread
	* Traditional UNIX
* Multiple processes, multiple threads
	* Modern Unix (Linux, Solaris), Windows

## Process Creation
Events that cause process creation:
* System initialization
	* Foreground processes (interactive programs)
	* Background processes
		* Email server, web server, print server, etc
		* Called a daemon(unix) or service(window)
* Execution of a process creation system call by running process
	* New login shell for an incoming *SSH* connection
* User request to create a new process
* Initiation of a batch job

## Process Termination
Conditions which terminate processes
* Normal exit (volunary)
* Error exit (volunary)
* Fatal error (involuntary)
* Killed by another process (involuntary)

## Process implementation
* Processes' information is stored in a *Process control block* (PCB)
* The *process control block* can form a process table
	* Reality can be more complex (hashing, chaining, allocation bitmaps)

Process management | Memory Management | File Management 
-------------------|-------------------|-----------------
Registers | Pointer to text segment | Root directory
Program counter | Pointer to data segment | Working directory
Program status word | Pointer to stack segment | File descriptors
Stack pointer | |User ID
Process state | | Group ID
Priority | | 
Scheduling paramenters | | 
Process ID | | 
Parent process | |
Process group | |
Signals | |
Time when process started | |
CPU time used | |
Children's CPU time | |
Time of next alarm | |

> **Transition causing events** <br />
> Running → Ready <br />
> Voluntary yield() <br />
> End of timeslice <br />

> Running → Blocked <br />
> File, network <br />
> Waiting for a timer (alarm signam) <br />
> Waiting for a resource to become available <br />

## Scheduler
* Sometimes called *dispatcher*
* Has to choose a *ready* process to run
	* its inefficient to search through all processes
		* The ready queue
		* Two queues
		* *n* queues

## Threads & The thread model
* Thread model - separating execution from the environment

Per process items | Per thread items
------------------|------------------
Address space | Program counter
Global variables | Registers
Open files | Stack
Child processes | State
Pending alarms |
Signals and signal handlers |
Accounting information |

> Items shared by all threads in a process <br />
> Items private to each thread

**Finite state(event) model**
* State explicitly managed by program

**Thread model**
* State implicitly stored on stack
* Each thread has its own stack
* Local variables are per thread
	* Allocated on the stack
* Global variables are shared between all threads
	* Allocated in data section
	* Concurrency control is an issue
* Dynamically allocated memory (*malloc*) can be global or local
	* Program defined (pointer can be global or local)

**Thread Usage**
Model | Characteristics
------|----------------
Threads | Parallelism, blocking system calls
Single-threaded process | No parallelism, blocking system calls
Finite-state machine | Parallelism, nonblocking system calls, interrupts

## Why threads?
* Simpler to program than state machine
* Less resource are associated with them than a complete processd
	* Cheaper to create and destroy
	* Shares resources (especially memory) between them
* Performance: Threads waiting for I/O can be overlapped with computing threads
	* Note if all threads are *compute bound*, then there is no performace improvement (on a uni processor)
* Threads can take advantage of the parallelism available on machines with more than one CPU (multiprocessor)

# Checkpoint
1. In the *three-state process* model, what do each of the three stages signify? What transitions are possible between each of the states, and what causes a process (or thread) to undertake such a transision?
	* The three states are:
		* Running - process is currently being executed on hte CPU
		* Ready - The process is ready to execute, but yet has been selected for execution by the dispatcher
		* Blocked - process is not runnable as it is waiting for some event prior to continuing execution
	* Possible transisions are
		* Running → Ready
			* Timeslice expired, yield, or higher priority process becomes ready
		* Ready → Running
			* Dispatcher chooses next thread to run
		* Running → Blocked
			* A requested resource (file, disk block, printer, mutex) is unavailable, so the process is waiting for the resource to be available 
		* Blocked → Ready
			* A resource has become available, so all processes blocked waiting for the resource now become ready to continue execition
2. Given *N* threads of uniprocessor system, how many threads can be *running* at the same point in time? How many threads can be *ready* at the same time? How many threads can be *blocked* at the same time?
	* Running → 0 or 1
	* Blocked → *N* - Running - Ready
	* Ready → *N* - Running - Blocked
3. Compare reading a file using single-threaded file server and a multithreaded file server. Within the file server, it takes 15*ms* ti get a request for work and do all necessary processing, assuming the required block is in the main memory disk block cache. A disk operation is required for one third of the requests, which takes additional 75*ms* during which the thread sleeps. How many requests/sec can a server handle if its single threaded? If multithreaded?
	* Single threaded → cache hit takes 15*ms* and cache miss takes 90*ms*; hence the average is 40*ms* so the server can do 25 per second
	* Multithreaded →all the waiting for the disk is overlapped, so every request takes 15*ms* and the server can hande 66.67 per second.


# Concurrency and Synchronisation
Concurrency example
> count is a global variable shared between two threads
``` c
void increment() {
	int t;
	// Race condition
	t = count;
	t = t + 1;
	count = t;
}
void decrement(){
	int t; 
	// Race condition
	t = count;
	t = t - 1;
	count = t;
}
```
> *Result can be either 1, 0, or -1*

## Critical region
* We can access to the shared resource by controlling access to the code that accesses the resource
	* *Critical region* is a region of code where shared recources are accessed
	> Variables, memory, files, etc
* Uncoordinated entry to the critical region results in a race condition
	* Incorrect behaviour, deadlock, lost work
* Regions of code that
	* Access a shared resource
	* Correctness relies on the shared resource not being concurrently modified by another thread/process/entity.

### Critical region solutions
Conditions required of any solution to the critical region problem
* Mutual exclusion
	* No two processes simultaneously in critical region
* No assumptions made about speeds or number of CPUs
* Progress
	* No process running outside its critical region may block another process
* Bounded
	* No process waits forever to enter critical region

## Mutual Exclusion by Taking Turns
``` c
	while(TRUE){
		while(turn!=0)
		critical_region();
		turn = 1;
		non_critical_region()
	}
	while(TRUE){
		while(turn!=1)
		critical_region();
		turn = 0;
		non_critical_region()
	}
```
* Works due to *strict alternation*
	* Each process takes turns
* Cons:
	* Busy waiting
	* Process must wait its turn even while the other process is doing something else
		* With many processes, must wait for everyone to have a turn
			> Doesnt guarantee progress if a process no longer needs a turn
		* Poor solution when processes require critical section at different rates

## Mutual Exclusion by Disabling Interrupts
* Before entering a critical region, disable interrupts
* After leaving a critical region, enable interrupts
* Pros:
	* Simple
* Cons:
	* Only available in kernel 
	* Blocks everybody else, even with no contention
		* Slows interrupt response time
	* Does not work on a multiprocessor

## Hardware Spport for Mutual Exclusion
* Test and set instruction
	* Can be used to implement lock varibales correctly
		* Load value of the rock
		* If lock is 0
			* Set the lock to 1
			* Return the result 0 -- acquire the lock
		* If lock is 1
			* Return 1 -- another thread/process has the lock
	* Hardware guarantees that the instruction executes atomically
	> *Atomically: as an indivisible unit*

## Mutual Exclusion with Test-and-Set
Enter_region | comments
-------------| --------------
TSL REGISTER, LOCK | Copy lock to register and set it to 1 
CMP REGISTER, #0| Was lock zero? 
JNE enter_region | If non zero, lock was set; hence loop 
RET| Return to caller; critical region entered 

Leave region | comments
-------------| ----------
MOVE LOCK, #0 		| Store a 0 in a lock 
RET 				| Return to caller 

* Pros
	* Simple and easy to show its correct
	* Available at user-level
		* To any number of processors
		* To implement any number of lock variables
* Cons
	* Busy awaits (*spinlock*)
		* Consumes CPU
		* Starvation possible when process leaves its critical region and more than one process is waiting

#### Tacking the busy wait problem
* Sleep/Wakeup
	* When process is waiting for an event, it calls sleep to block instead of busy waiting
	* The event happens, the event generator(another process) calls wake up to unblock the sleeping process
	* Waking a ready/running process has no effect

## Producer-Consumer Problem
* Also called the *bounded buffer* problem
* Producer produces data items and stores the items to buffer
* Consumer takes the item out of the buffer and consumes them
* **Issues**
	* We must keep an accurate count of items in buffer

	Producer | Consumer
	---------|---------
	Should sleep when the buffer is full | Should sleep when buffer is empty
	Wake up when there is empty space in buffer | Wake up when there are items available
	Consumer can call wakeup when it consumes the first entry of the full buffer | Producer can call wakeup when it adds the first item to the buffer
* **PROBLEMS**
```c
int count = 0;
#define N 4 // BUF SIZE
prod(){
	while(TRUE){
		item = produce();
		if(count == N) 
			sleep();
		insert_item(); 	// concurrent issue 1
		count ++;		// concurrent issue 2
		if(count == 1)
			wakeup(con);
	}
}
con(){
	while(TRUE){
		if(count == 0)
			sleep();
		remove_item(); 	// concurrent issue 1
		count--; 		// concurrent issue 2
		if(count == N-1)
			wakeup(prod);
	}
}
```
> Concurrent uncontrolled access to the buffer and counter
* The test for *some condition* and actually going to sleep needs to be atomic;
* The following does not work: The lock is held while asleep → count will never change
```c
	acquire_lock();					acquire_lock()
	if(count == N)					if(count == 1)
		sleep();					wakeup();
	release_lock();					release_lock();			
```

## Semaphores
* *Dijkstra* introduced two primitives that are more powerful than simple sleep and wakeup
	* P() *proberen* → test
	* V() *verhogen* → increment
	* Also called *wait & signal* and *down & up*
* **How to semaphores work?**
* If a resource is not available, the corresponding semaphore blocks any process waiting for the resource
* Blocked processes are put into a process queue maintained by the semaphore (avoid busy waiting)
* When a process releases a resource, it signals this by means of the semaphore
* Signalling resumes a blocked process if there is any
* Wait and signal operations cannot be interrupted
* Complex coordination can be implemented by multiple semaphores

#### Implementation
``` c
typedef struct {
	int count;
	struct process *L;
} semaphore;
```
> Assume two simple operations:
> * **sleep** suspends the process that invokes it
> * **wakeup(P)** resumes the execution of blocked process **P**
```c
wait(S):
	S.count--;
	if(S.count<0){
		add this process to S.L;
		sleep;
	}
signal(S):
	S.count++;
	if(S.count <= 0){
		remove process P from S.L;
		wakeup(P);
	}
```

#### Semaphore implementation of a Mutex
> Mutex is short for mutual exclusion
```c
sephamore mutex;
mutex.count = 1;	/* initialise mutex */
wait(mutex);		/* enter the critical region */
do_something();
signal(mutex); 		/* exit the critical region */
```
> Initial count determines how many waits can progress before blocking and requiring a signal → mutex.count initialised as 1
```c
#define N 4
sephamore mutex = 1;
semaphore empty = N; 	/* count empty slots */
sephamore full = 0; 	/* count full slots */
prod() {				con() {
	while(TRUE){				while(TRUE){
		item = produce()			wait(full);
		wait(empty);				wait(mutex);
		wait(mutex)				remove_item();
		insert_item();				signal(mutex);
		signal(mutex);				signal(empty);	
		signal(full);
	}					}
}					}
```

## Semaphore summary
* Semaphores can be used to solve variety of concurrency problems
* However, programming with then can be error-prone
	* Needs to be placed in the right places
	* E.g Must *singnal* for every *wait* mutexes
		* Too many, or too few signals or waits, or signals and waits in the wrong order, can have catastrophic results.

## Monitors
* To ease concurrent programming, *Hoare* proposed *monitors*
	* A higher level synchronisation primitive
	* Programming language construct
* A set of procedures variables, data types are grouped in a special kind of module, *a monitor*.
* Only one process/thread can be in the monitor at any one time
	* Mutual exclusion is implemented by the compiler
* When a thread calls a monitor procedure that has a thread already inside, it is queueed and it sleeps until the current thread exits the monitor
> *One thread inside at a time*
```c
monitor counter{					// monitor example
	int count;						// integer i
	procedure inc(){					// condition c
		count = count + 1;				// procedure producer();
	}							// ... end
	procedure dec(){					// procedure consumer();
		count = count - 1;				// ... end
	}						// end monitor
}
```
> Compiler guarantees only one thread can be active in the monitor at any one time <br />
> Easy to see this providdes mutual exclusion
> 	* No race condition on count

How is waiting blocked for an event?
## Condition variable
* To allow a process to wait within the monitor, a condition variable must be declared as **condition x,y;**
* Condition variable can only be used with the operations **wait** and **signal**.
* x.wait()
	* process invoking this operation is suspended until another process invokes
	* Another thread can enter the monitor while original is suspended
* x.signal()
	* resumes exactly one suspended process
	* If no process is suspended, then signal operation has no effect

###### Outline of producer-consumer problem with monitors
> Only one monitor procedure active at one time <br />
> Buffer has N slots
```c
monitor ProducerConsumer
	condition full, empty;				procedure producer;
	integer count;						while TRUE
	procedure insert(integer item){					item = produce_item;
		if(count == N) wait(full)				ProducerConsumer.insert(item)
		insert_item(item)
		count = count + 1;
		if(count == 1) signal(empty)
	}
	procedure remove(){				procedure consumer;
		if(count == 0) wait(empty);			while TRUE
		remove = remove_item();					item =ProducerConsumer.remove();
		count = count - 1;					consume_item(item)
		if(count == N - 1) signal(full);
	}
	count = 0;
```

# Checkpoint
```c
x = x + 1
```
1. The code above is a single line of code. How might a race condition occur if it is executed concurrently by multiple threads? Can you give an example of how an incorrect result can be computed for x?
	* The single code statement is compiled to multiple machine instructions. The memory location corresponding to x is loaded into register, increamented, and then stored back to memory. During the interval between the load and store in the first thread, another thread may perform a load, increment, and store, and when control passes back to the first thread, the results of the second are overwritten. 
```c
int i;
void foo(){
    int j;
    /* random stuff*/
    i = i + 1;
    j = j + 1;
    /* more random stuff */
}
```
2. The code above is called by multiple threads(potentially concurrently) in multithreaded program. Identify the critical regions that require mutual exclusion. Describe the race condtion or why no race condition exists.
	* There is no race condition on *j* as it is a local varaible. However, *i* is a variable shared between threads; this *i = i + 1* would form a critical section.
```c
void inc_mem(int *iptr){
    *iptr = *iptr + 1;
}
```
3. The code above is called by threads in a multithreaded program. Under what conditions would it form a critical section?
	* Whether *iptr = *iptr + 1* forms a critical section depends on the scope of the pointer passed to *inc_mem*. If the pointer porints to a local variable, then there is no race. If it points to shared global variable, then there is a potential for a race, thus the increment would become a critical section.
4. What synchronisation mechanism or approach might one take to have one thread wait for another thread to update some state?
	* Semaphore (count initialised as 0):
```c
	wait_for_update(){
	   P(updated)
	}
	signal_update_occurred(){
	   V(updated)
	}
```  
	* CV (flag initialised as 0; with lock l):
```c
wait_for_update(){
   	  lock_acquire(l)
	  while(flag == 0)
	    cv_wait(cv,l)
	  lock_release(l)
	}
signal_update_occurred(){
	lock_acquire(l)
	  flag = 1
	  cv_signal(cv,l)
	lock_release(l)
	}	
```
5. A particular abstraction only allows a maximum of 10 threads to enter a *room* at any point in time. Further threads are attempting to enter the room have to wait at the door for another thread to exit the room. How could one implement such synchronisation approach to enforce the above restriction?
	* Semaphore (count intialised as 10):
```c
enter_room(){
   P(room);
}	
leave_room(){
   V(room);
}
```
	* CV (occupants initialised as 0, lock room_lock)
```c
	enter_room(){
       	  lock_acquire(room_lock);
	  while(occupants == 10)
	    cv_wait(room_cv,room_lock);
	  occupants = occupants + 1;
	  lock_release(room_lock);
	}
	leave_room(){
	  lock_acquire(room_lock);
	    occupants = occupants - 1;
	    if (occupants == 9) 
	      cv_signal(room_cv,room_lock);
	  lock_release(room_lock);
	}
```



# Deadlocks
## Resources
* Examples of computer resources
	* Printers
	* Tape drives
	* Tables in database
* Porcesses need access to resources in reasonable order
* Preemptable resources
	* can be taken away from a process with no ill effects
* Nonpreemptable resources
	* will cause the process to fail if taken away
*Example*
```c
void proc_A(){			void proc_B(){
	down(&res_1);   <-deadlocked->	down(&res_2);
	down(&res_2);			down(&res_1);
	use_both();			use_both();
	up(&res_2);			up(&res_1);
	up(&res_1);			up(&res_2)
}				}
```

## Resources & Deadlocks
* Suppose a process holds resource A and requests resource B
	* at same time another process holds B and requests A
	* both are blocked and remain so → deadlocked
* Deadlocks occur when...
	* processes are granted exclusive access to device, locks, tables, etc..
	* refer to these entities generally as *resources*
* **Resouce Access**
	* Sequence of events required to use a resource
		1. request the resource
		2. use the resource
		3. release the resource
	* Must wait if request is denied
		* requesting process may be blocked
		* may fail with error code

## Deadlocks
* *A set of processes is deadlocked if each process in the set is waiting for an event that only another process in the set can cause*
* Usually the event is release of a current held resource
* None of the process can:
	* run
	* release resources
	* be awakened

#### Condiions for Deadlock
* Mutual exclusion condition
	* Each resource assigned to 1 process or is available
* Hold and wait condition
	* Process holding resources can request additional
* No preemption conditon
	* Previously granted resources cannot be forcibly taken away
* Circular wait condition
	* Must be circular chain of 2 or more processes
	* Each is waiting for resource held by next member of the chain

#### Strategies for dealing with deadlocks
* Just ignore the problem
* Prevention
	* Negating one of the four necessary conditions
* Detection and recovery
* Dynamic avoidance
	* Careful resource allocation

#### Approach one: *The Ostrich Algorithm*
* Pretend there is no problem
* Reasonable if
	* Deadlocks occur very rarely
	* Cost of prevention is high
		* Example of *cost*, only one process runs at a time
* Unix and Windows takes this approach for some of the more complex resource relationships they manage
* Tradeoff between *convenience* and *correctness*

#### Approach two: *Deadlock prevention*
* Resource allocation rules prevent deadlock by preventing one of the four conditions required for deadlock from occurring
	* Mutual exclusion
		* Not feasible in general
			* Some devices/resources are intrinsically not shareable
	* Hold and wait
		* Require processes to request resources before starting
			* a process never has to wait for what it needs
		* Issues
			* may not know required resources at start of run
				* not always possible
			* also ties up resources other processes could be using
			* Prone to *livelock*
		* Variations
			* process must give up all resources if it would block holding a resource
			* then request all immediately needed
			* prone to livelock
		> Livelock
		> Livelocked processes are not blocked, change state regularly, but never make progress <br />
		> Example: Two people passing each other in corridor that attempt to step out of each other's way in same direction, indefinitely
		> * Both are actively changing state
		> * Both never pass each other 
	* No preemption
		* Not viable option
		> Given a printer, take the printer away forcibly halfway through its job ???
	* Circular wait
		* The displayed deadlock cannot happen
			* If A requires *1*, it must require it before acquiring *2*
			> If B has *1*, all higher numbered resources must be free or held by processes who doesn't need *1*
* **Summary**

Condition | Approach
----------|-----------
Mutual Exclusion | Not feasible
Hold and Wait | Request resources intially 
No Preemption | Take resources away
Circular Wait | Order resources

#### Approach three: *Detection and Recovery*
* Need a method to determine if a system is deadlocked
* Assuming deadlock is detected, we need a method of recovery to restore progress to the system
* **Recovery from deadlock**
	* Recovery through preemption
		* take a resource from some other process
		* depends on nature of the resource
	* Recovery through rollback
		* checkpoint process periodically
		* use this saved state
		* restart the process if its found deadlocked
			* no guarantee is won't deadlock again
	* Recovery through killing processes
		* crudest but simplest way to break a deadlock
		* kill one of the processes in the deadlock cycle
		* the other processes get its resources
		* choose process that can be rerun from the beginnning

#### Approach four: *Deadlock Avoidance*
* Instead of detecting deadlock, simply avoid it
	* Only if enough information is available in advance\
	> Maximum number of each resource required
* **Safe and unsafe states**
	* Safe if:
		* System is not deadlocked
		* There exists a scheduling order that results in every process running to completion, *even if they all request their maximum resources immediately*
	* Unsafe states:
		* Not necessarily deadlocked
			* With lucky sequence, all processes may complete
			* but **cannot guarantee** that they will complete
		* Safe states guarantee we will eventually complete all processes
		* Deadlock avoidance algorithm
			* Only grant requests that result in safe states
		###### Banker's Algorithm
		* Modelled on a banker with customers
			* Banker has a limited amount of money to loan customers
				* Limited number of resources
			* Each customer can borrow money up to the customer's credit limit
				* Maximum number of resources required
				<br />
			* Keep the bank in a *safe* state
				* So all customers are happy even if they all request to borrow up to their credit limit at the same time
			* Customers wishing to borrow such that the bank would enter an unsafe state must wait until somebody else repays their loan such that the transaction becomes safe.
		> Not practical <br />
		> Its difficult (sometimes impossible) to know in advance<br />
		> resources process will require<br />
		> the number of processes in a dynamic system<br />

## Starvation
* A process never receives the resource its waiting for, despite the resource (repeatedly) becoming free, the resource is always allocated to another waiting process
	* Example: an algorithm to allocate resource may be to give the resource to the shortest job first
	* Works great for multiple short jobs in a system but causes a long job to wait indefinitely, even though not blocked
	* One simple solution: FIFO (*queue*)

# Process and threads implementation
### MIPS R3000 
* Load/Store architecture
	* No instructions that operate on memory except load and store
	* Simple load/store to/from memory from/to registers
		* Store word: *sw r4, (r5)*
		> Store contents of r4 to memory using address contained in register r5
		* Load word: *lw r3, (r7)*
		> Load the contents of memory into r3 using address contained in r7 <br />
		> Delay one instruction after load before data available in destination reguster
			* Must always an instruction between a load from memory and the subsequent use of register
		> *lw, sw, lb, sb, lh, sh ...*
* Arithmetic and logical operations are register to register opertation
	* *add r3, r2, r1*  →  r3 = r2 + r1
	* No arithmetic operation on memory
	> *add, sub, and, or, xor, sll, srl, move*
* All instructions are encoded in 32-bit
* Some instructions have *immediate* operands
	* immediate values are constants encoded in the instruction itself
	* Only 16 bit value
	> *addi r2, r1, 2048* → r2 = r1 + 2048 <br />
	> *li r2, 1234* →  r2 = 1234
*Example code for a = a + 1*
``` c 
lw r4, 32(r29) 	// r29 = stack pointer <br />
li r5, 1	// offset(address)
add r4, r4, r5
sw r4, 32(r29)
```

#### MIPS registers
* User-mode accessible registers
	* 32 general purpose registers
		* *r0* hardwired to **zero**
		* r31 the *link* register to *jump-and-link(JAL)* instructino
	* *HI/LO*
		* 2 * 32 bits for multiply and divide
	* *PC*
		* not directly visible
		* modified implicitly by jump and branch instructions
* Branching and jumping	
	* Branching and jumping have a *branch delay slot*
		* The instruction following a branch or jump always executed prior to destination of jump
* Jump and Link instruction
	* *JAL* is used to implement function calls
		*r31 = PC + 8*
	* Return address (*RA*) is used to return from function call

### Compiler register conventions
Register No. | Name | Used for
-------------|------|---------
0 | zero | always returns 0
1 | at | (assembler temporary) Reserved for use by assembler
2-3 | v0-v1 | Value (except FP) returned by subroutine
4-7 | a0-a3 | (arguments) First four parameters for a subroutine
8-15 <br /> 24-25 | t0-t7 <br /> t8-t9 | (temporaries) subroutunes may use without saving
16-23 | s0-s7 | Subroutine *register variables*; a subroutine which will write. One of these must save the onld value and restore it before it exits, so the *calling* routine sees their values preserved
26-27 | k0-k1 | Reserved for use by interruupt/trap handler → may change under your feet
28 | gp | global pointer →  some runtime systems maintain this to give easy access to *some* "static" or "extern" variables
29 | sp | stack pointer
30 | s8/fp | 9<sup>th</sup> register variable. Subroutines which need one can use this as a *frame pointer*
31 | ra | Return address for subroutine

## Function Stack Frames
* Each function call allocates a new stack frame for local variables, the return address, previous frame pointer, etc.
	* *Frame pointer* →  start of current stack frame
	* *Stack pointer* →  End of current stack frame

## Process
* Minimally consists of three segments
	* Text
		* Contains the code (Instructions)
	* Data
		* Global variables
	* Stack
		* Activation records of procedure/function/method
		* Local variables
	> Data can dynamically grow up (*malloc()*) <br />
	> Stack can dynamically grow down (increasing function call depth; *recursion*)
* User mode
	* Processes(programs) scheduled by the kernel
	* Isolated from each other
	* No concurrency issues between each other
* System-calls transition into and return from the kernel
* Kernel mode
	* Nearly all activities associated with a process
	* Kernel memory shared between all processes
	* Concurrency issues exist between processes concurrently executing in the system call
* Thread model - separating execution from the environment

Per process items | Per thread items
------------------|------------------
Address space | Program counter
Global variables | Registers
Open files | Stack
Child processes | State
Pending alarms |
Signals and signal handlers |
Accounting information |

#### User level threads
* Implementation at user-level
	* User level thread control block (TCB), ready queue, blocked queue, and dispatcher
	* Kernel has no knowledge of the threads(only sees a single process)
	* If a thread blocks waiting for a resource held by another thread inside the same process, its state is saved and the dispatcher switches to another ready thread
	* Thread management(create, exit yield, wait) are implemented in a runtime support library
* Pros
	* Thread management and switching at user level is faster than doing it in kerel level
		* No need to trap (take syscall exception) into kernel and back to switch
	* Dispatcher algorithm can be turned to the application
		* e.g. Use priorities
	* Can be implemented on any OS (thread or non-thread aware)
	* Can easily support massive numbers of threads on per-application basis
		* Use normal application virtual memory
		* Kernel memory more constrained. Difficult to efficiently support widely differing numbers of threads for different applications
* Cons
	* Thread have to *yield()* manually (no timer interrupt delivery to user-level)
		* Co-operative multithreading
			* A single poorly designed/implemented thread can monopolise the available CPU time
		* There are workarounds (e.g. timer signal per second to enable pre-emptive multithreading), they are course grain and kludge
	* Does not take advantage of multiple CPUs (in reality, we still have a single threaded process as far as the kernel is concerned)
	* If a thread makes a blocking system call (or takes a page fault), the process (and all its internal threads) blocks
		* Can't overlap *I/O* with computation

#### Kernel-provided threads
* Also called kernel-level threads even though they provide threads to applications
* Threads are implemented by the kernel
	* TCBs are stored in the kernel 
		* A subset of information in a traditional PCB
			* The subset related to execution context
		* TCBs have a PCB associated with them
			* Resources associated with the group of threads (the process)
	* Thread management calls are implemented as system calls
		* E.g. create, wait, exit
* Pros
	* Preemptive multi-threading
	* Parallelism
		* Can overlap blocking I/O with computation
		* Can take advantage of a multi-processor
* Cons
	* Thread creation, destruction, and blocking and unblocking threads require kernel-entry and exit
		* More expensive than user-level equivalent

#### Multiprogramming implementation
1. Hardware stacks program counter, etc
2. Hardware loads new program counter from interrupt vector
3. Assembly language procedure saves registers
4. Assembly language procedure sets up new stack
5. C interrupt service runs (typically reads and buffers input)
6. Scheduler decides which process is to run next
7. C procedure returns to assembly code
8. Assembly language procedure starts up new current process
> Skeleton of what lowest level OS does when an interrupt occurs → a context switch

#### Context Switch
* Switch between threds
	* Involving saving and restoring of state associated with a thread
* Switch between processes
	* Involving the above, plus extra state associated with a process
		* e.g. Memory maps
* **Context Switch Occurence**
	* A switch between process/threads can happen anytime the OS is invoked
		* On a system call
			* Mandatory if system call blocks or on exit();
		* On an exception
		 	* Mandatory if offender is killed
		 * On an interrupt
		 	* Triggering a dispatch is the main purpose of the timer interrupt
	* **A thread switch can happen between any two instructions**
	> Instructions do not equal program statements
* Must be transparent for processes/threads
	* When dispatched again, process/thread should not notice that something else was running in the meantime (except for elapsed time) → OS must save all state that affects the thread
* This state is called process/thread context
* Switching between process/threads consequently results in a *context switch*

# System calls
## Interface
* Allow interaction between Kernel mode and User mode
* Can be viewed as special function calls
	* Provides for a controlled entry to the kernel
	* While in kernel, they perform a privileged operation
	* Returns to original caller with the result
* The system call interface represents the abstract machine provided by the operating system
* Process management
	* fork(), waitpid(pid &statloc, options), execve(name, argv, environp), exit(status)
* File Management
	* open(file, how, ...), close(fd), read(fd, buffer, nbytes), write(fd, buffer, nbytes), lseek(fd, offset, whence), stat(name, &buf)
* Directories management
*stripped down shell*
```c
while(TRUE){
	type_prompt();				//display prompt
	read_command(cmd, params);		// input from terminal
	if(fork() != 0){			// fork off child process
		// parent code
		waitpid(-1, &status, 0) 	// wait for child to exit
	} else{
		// child code
		execve(cmd, params, 0) 		// execute cmd
	}
}
```
## Implementation
* **A simple model of CPU Computation**
	* The fetch-execute cycle
		* Load memory contents from address in the *program counter*(PC)
			* The instruction
		* Execute the instruction
		* Increment PC
		* Repeat
		* Stack pointer(SP)
		* Status register
			* Condition codes
				* Positive result
				* Zero result
				* Negative result
		* General purpose registers
			* Holds operands of most instructions
			* Enables programmers(compilers) to minimise memory references
* **Privileged-mode operation**
	* To protext operating system execution, two or more CPU modes of operation exist
		* Privileged mode(kernel mode)
			* All instructions and registers are available
		* User mode
			* Uses *safe* subset of the instruction set
				* Only affects the state of the application itself
				* They cannot be used to uncontrollably interfere with the OS
			* Only *safe* registers are accessible
	* The accessibility of addresses within an address space changes depending on operating mode
		* To protect kernel code and data
	> The exact memory ranges are usually configurable, and vary between CPU architectures and/or operating systems

	> System call mechanism securely transfers from user execution to kernel execution and back

#### System call mechanism overview
* System call transitions are triggered by special processor instructions
	* User to Kernel → System call instruction
	* Kernel to User → Return from privileged mode instruction
* Processor mode
	* Switched from user mode to kernel mode
		* Switched back when returning to user mode
* Stack pointer (SP)
	* User level SP is saved and a kernel SP is initialised
		* User level SP is restored when returning to user mode
* Program counter (PC)
	* User level PC is saved and PC set to kernel entry point
		* User level PC restored when returning to user level
	* Kernel entry via designated entry point must be strictly enforced
* Registers
	* Set at user level to indicate system call type and its arguments
		* A convention between applications and the kernel
	* Some registers are preserved at user level or kernel level in order to restart user level execution
		* Depends on language calling convention, etc.
	* Result of system call placed in registers when returning to user level
		* Another convention
* **QUESTION: WHY DO WE NEED SYSTEM CALLS?**
	* **WHY NOT JUMP STRAIGHT INTO KERNEL VIA FUNCTION CALL?**
		* Function calls do not
			* Change from user to kernel mode and back
			* Restrict possible entry points to secure locations
				* To prevent entering after any security checks
* Steps in making a system call
*read(fd, buffer, nbytes)*
1. Push *nbytes*
2. Push *&buf*
3. Push *fd*
4. Call *read*
5. Put code for read in register
6. Trap to the kernel → Dispatch 	(kernel mode)
7. → trapframe → syscall handler 	(kernel mode)
9. Return to caller
10. Increment *SP* 

### MIPS R2000/R3000
#### Coprocessor 0
* The processor control registers are located in *CP0*
	* Exception/interrupt management registers
	* Translation management registers
* *CP0* is manipulated using *mtc0* (move to) and *mfc0* (move from) instructions
	* Only accessible in kernel mode

###### CP0 Registers
Exception management | Memory manamgemet | Miscellaneous
---------------------|-------------------|---------------
c0_cause → cause of recent exception | c0_index | c0_prid → process identifier
c0_status → current status of CPU | c0_random |
c0_epc → Address of the instruction that caused exception | c0_entryhi
c0_badvaddr → address accessed that caused exception | c0_entrylo
| | c0_context

## *TODOS HERE... continue some time in the future (SYSCALL)*

# Checkpoint
1. What is *branch delays*
	* The pipeline structure of the MIPS CPU means that when a jump instruction reaches the *execute* phase and a new program counter is generated, the instruction after the jump will already have been decoded. Rather than discard this potentially useful work, the architecture rules state that the instruction after a branch is always executed before the instruction at the target of the branch
2. Why is recursion or large arrays of local variavbles avoided by kernel programmers?
	* The kernel stack is a limited resource. Stack overflow crashes the entire machine.
3. Cooperative multithreading vs Preemptive multithreading
	* Cooperative multithreading
		* Running thrad must explicitly *yield()* the CPU so that the dispatcher can select a ready thread to run next. It relies on the cooperation of the threads to ensure each thread receives regular CPU time. 
	* Preemptive multithreading
		* External event (e.g. regular timer interrupt) causes the dispatcher to be invoked and thus *preempt* the running thread, and select a ready thread to run next. Preemptive multithreading enforces a regular (at least systematic) allocation of CPU time to each thread, even when thread is uncooperative or malicious.
4. Describe *user-level* threads *kernel-level* threads. What are the advantages or disadvantages of each approach?
	* User level threads
		* Implemented in the application(usually in a thread library). The thread management structures (thread control blocks) and scheduler are contained within the application. The kernel has no knowledge of the user-level threads.
		* Faster to create, destroy, manage, block, and activate (no kernel entry required)
		* Dont take advantage of parallelism available on multh-CPU machines. 
		* Ususally cooperative
		> Some user level threads use alarms or timeouts to provide a tick for preemption
		* Can be implemented on OSes without support for kernel threads
	* Kernel threads
		* Implemented in the kernel. The TCBs are managed by the kernel, the thread scheduler is the normal in-thread scheduler
		* If a single user-level thread blocks in the kernel, the whole process is blocked. However, some libraries avoid blocksing in the kernel by using non-blocking system call variants emulate the blocking calls
		* Usually preemptive
5. A web server is constructed such that it is multithreaded. If the only way to read from a file is a normal blocking read system call, do you think user-level threads or kernel-level threads are being used for the web server? Why?
	* A worker thread within the web server will block when it has to read a webpage from the disk. If user thread is being used, the action will block the entire process, destroying the value of multithreading. Thus, it is essential that kernel threads are used to permit some threads to block without affecting others.
6. Assume a multi-process operating system with single-threaded applications. The OS manages the concurrent application requests by having a thread of control within the kernel for each process. Such a OS would have an in-kernel stack assocaited with each process.
<br /> <br/ >
Switching between each process (in-kernel thread) is performed by the function switch_thread(cur_tcb,dst_tcb). What does this function do?
	* The function saves the registers requried to preserve the compiler calling convetion(and registers to return to the caller) on to the current stack
	* The function saves the resulting stack pointer into the thread of control block associated with *cur_tcb*, and sets the stack pointer to the stack pointer stored in destination *tcb*
	* The function then restores the registers that were stored previously on the destination(now current) stack, and returns to the destination thread to conitnue where it left off.
7. What is the *EPC* register? What is it used for?
	* It is a 32-bit register containing the 32-bit address of the return point for the last excetpion. The instruction causing the exception is at *EPC*, unless *BD* is set in *Case*, in which case *EPC* points to previous(branch) instruction
	* It is used by the exception handler to restart exectution at the point where execution was interrupted
8. What happens to the KUc and IEc bits in the STATUS register when an exception occurs? Why? How are they restored?
	* the *c*(current) bits are shifted into the corresponding *p*(previous) bits, after which *KUc = 0, IEc = 0*(kernel mode with interrupts disabled). They are shifted in order to preserve the current state at the point of exception in order to restore that exact state when returning from the exception
	* They are restored via a *rfe* instruction (restore from exception)
9. What is the value of ExcCode in the Cause register immediately after a system call exception occurs?
	* 8	
10. Why must kernel programmers be especially careful when implementing system calls?
	* System calls with poor argument checking or implementation can result in a malicious or buggy program crashing, or compromising the operating system
11. The following questions are focused on the case study of the system call convention used by OS/161 on the MIPS R3000 from the lecture slides.
	1. How does the 'C' function calling convention relate to the system call interface between the application and the kernel?
		* The *C* function calling convention must always apepar to be adhered to after any system-call wrapper function completes. This involves saving and restoring of the *preserved* registers
		* The system call convention also uses the calling convention of the *C-compiler* to pass arguments to OS161. Having the same convention as the compiler means the system call wrapper can avoid moving arguments around the compiler has already placed them where the OS expects to find them.
	2. What does the most work to preserve the compiler calling convention, the system call wrapper, or the OS/161 kernel.
		* The OS161 kernel code does the saving and restoring of preserved registers. The system call wrapper function does very little
	3. At minimum, what additional information is required beyond that passed to the system-call wrapper function?
		* The interface bewteen the system-call wrapper function and the kernel can be defined to provide additional information beyond that passed to the wrapper function. At minimum, the wrapper function must add the system call number to the arguments passed to the wrapper function. It's usually added by setting an agreed-to register to the value of the system call number
12. To a programmer, a system call looks like any other call to a library function. Is it important that a programmer know which library function result in system calls? Under what circumstance and why?
	* As far as program logic is concerned, it does not matter whether a call to libaray function results in a system call. But if performance is an issue, if a task can be accomplished without a system call, the program will run faster. Every system call involves overhead in switching from user context to kernel context. Furthermore, on a multiuser system, the operating system may schedule another process to run when system call completes, further slowing the progress in real time of calling process

# Computer Hardware (Memory Hierarchy)
* **Operating systems**
	* Exploit the hardware available
	* Provide a set of high level service that represent or are implemented by the hardware
	* Manages the hardware reliably and efficienty
	* *Understanding the Operating System requires a basic understanding of the underlying hardware*

##### Memory Hierarchy
* Register → Cache → Main memory → Magnetic disk → Magnetic tape 
> Decreasing cost per bit <br />
> Increasing capacity <br />
> Increasing access time

## Caching
* Given two levels of data storage, small and fast vs large and slow, can speed access to slower storage by using itermediate-speed storage as *cache*
> CPU registers Fast → Cache memory (SRAM) Fast -> Main memory (DRAM) Slow
* **CPU Cache**
	* CPU cache is fast memory placed between CPU and main memory
		* 1 to a few cycles access time compared to RAM access time of tens-hundreds of cycles
	* Holds recently used data or instructions to save memory accesses
	* Matches slow RAM access time to CPU speed if high hit rate
	* Is hardware maintained and (mostly) transparent to software
	* Sizes range from few KB to tens of MB
	* Usually hierarchy of caches(2-5 levels), on and off chip
* *What is the effective access time of memory subsystem?* → It depends on the hit rate in the first level

* **Effective Access Time**
###### <div align="center"> *T<sub>eff</sub> = H x T<sub>1</sub> + (1-H) x T<sub>2</sub>*</div>
###### <div align="center"> T<sub>eff</sub> = Effective access time of system <br /> H = hit rate in memory 1 <br />  T<sub>1</sub> = Access time of memory 1 <br />  T<sub>2</sub> = Access time of memory 2</div>

***How to improve system performance***

OS | Application
---|-------------
Keep a subset of the disk's data in main memory <br /> → OS uses main memory as a *cache* of disk contents | Keep a subset of internet's data on disk <br /> → Application uses disk as a *cache* of the internet

# Checkpoint
1. Describe the memory hierarchy. What types of memory appear in it? What are the characteristics of the memory as one moves through hierarchy? How can memory hierarchies provide both fast access times and large capacity?
	* The memory hierarchy is a hierarchy of memory types composed such that if data is not accessible at the top of the hierarchy, lower levels of hierarchy are accessed until the data is found, upon which a copy (usually) of the data is moved up the hierarchy for access 
	* Registers, cache, main memory, disk, CDROM, tape are all types of memory that can be composed to form a memory hierarchy
	* In going top of the hierarchy to bottom, the memory types feature decreasing cost per bit, increasing capacity, but also increasing access time.
	* As we move down the hierarchy, data is accessed less frequently (*frequently accessed data at the top of the hierarchy*). The phenomenon is called *locality* of access, most accesses are to a small subset of all data.
2. Given that disks can stream data quite fast (1 block in tens of microseconds), why are average access times for a block in milliseconds?
	* Seek times are in milliseconds(*e.g. 0.5 millisecond track to track, 8 millisecond inside to outside*), and rotational latency (1/2 rotation) is in milliseconds (*e.g. 2 milliseconds for 15,000 rpm disk*).
3. You have a choice of buying a 3 Ghz processor with 512K cache, and a 2 GHz processor (of the same type) with a 3 MB cache for the same price. Assuming memory is the same speed in both machines and is much less than 2GHz (say 400MHz). Which would you purchase and why? Hint: You should consider what applications you expect to run on the machine.
	* If you are running a small application, then 3GHz processor would be much faster. If you are running a large application that accesses a larger amount of memory than 512K memory, but generally less than 3MB, then the 2GHz processor would be faster as 3GHz processor will be limited by memory speed.

# File systems
**Overview of the FS abstraction**
User's view | Under the hood (*in practice*)
------------|----------------
Uniform namespace | Heterogeneous collection of storage devices
Hierarchial structure | Flat address space (block numbers)
Arbitrary-sized files | Fixed-size blocks
Symbolic file names | Numeric block addresses
Contiguous address space inside a file | Fragmentation
Access control | No access control
Tools for <br>
→ Formatting <br />
→ Defragmentation <br />
→ Backup <br />
→ Consistency checking | 

## File names
* File system must provide a convenient naming scheme
	* Textual names
	* May have restrictions
		* Only certain characters 
		> e.g. no '/' characters
		* Limited length
		* Only certain format
		> e.g. DOS, 8 + 3
	* Case (in)sensitive → *depends on the system*
	* Names may obey conventions (.c for C files → *nothing to do with OS*)
		* Iterpreted by tools (e.g. UNIX)
		* Iterpreted by the operating systems (e.g. windons 'con')
		> Operating system only recognises one file format → executable file

## File structure
* Sequence of bytes
	* OS considers a file to be unstructured 
	* Applications can impose their own structure
	* Used by UNIX, Windows, most modern OSes

## File types
* Regular files
* Directories
* Device files (*e.g. /dev*)
	* Maybe devieded into
		* Character devices → Stream of bytes
		* Block devices
* Some systems distinguish between regular file types
	* ASCII text files, binary files

## File access types (Patterns)
* Sequential access
	* Read all bytes/records from the beginning
	* Cannot jump around, could rewind or backup
	* Convenient when medium was magnetic tape
* Random access
	* Bytes/records read in any order
	* Essential for database systems
	* Read can be
		* Move file pointer(seek) then read or 
			* lseek(location,...); read(...)
		* Each read specifies the file pointer
			* read(location)

## File attributes
Attribute | Meaning
----------|----------
Protection | Who can access the file and in what way
Password | Password needeed to access the file
Creator | ID of the person who create the file
Owner | Current owner
Read-only flag | 0 for read/write; 1 for read only
Hidden flag | 0 for nomal; 1 for do not display in listings
System flag | 0 for normal files; 1 for system file
Archive flag | 0 for has been backed up; 1 for needs to be backed up
ASCII/Binary flag | 0 for ASCII file; 1 for binary file
Random access flag | 0 for sequenctial access only; 1 for random access
Temporary flag | 0 for normal; 1 for delete file on process exit
Lock flag | 0 for unlocked; nonzero for locked
Record length | Number of bytes in a record
Key position | Offset of the key within each record
Key length | Number of bytes in a key field
Creation time | Date and time the file was created
Time of last access | Date and time the file was last accessed
Time of last change | Date and time the file has last changed
Current size | Number of bytes in the file
Maximum size | Number of bytes the file may grow to

## Typical file operations
* Create, delete, open, close, read, write, append, seek, get attributes, set attributes, rename

## File organisation and access; a programmer's perspective
* Given an operating system supporting unstructured files that are a *stream-of-bytes*, how can one organise the contents of the files?
* Some possible access patterns:
	* Read the whole file 
	* Read individual records from a file
	> *record → sequence of bytes containing the record*
	* Read records preceding or follwoing the current one
	* Retrieve a set of records
	* Write a whole file sequentially
	* Insert/delete/update records in a file
> Programmers are free to structure the file to suit the application

## Criteria for file organization
Things to consider when designing file layout:
* Rapid access 
	* Needed when accessing a single record 
	* Not needed for batch mode
		* Read from start to finish
	* Ease of update
		* File on CD-ROM will not be updated; so this is not a concern
	* Economy of storage
		* Should be minimum redundancy in the data
		* Redundancy can be used to speed access such as an index

## File directories
* Procide mappiong between file names and files themselves
* Contain information about files
	* Attributes
	* Location
	* Ownership
* Directory itself is a file owned by the operating system

#### Hierarchical (Tree structured Directory)
* Files can be located by following a path from the root, or master, directory down to various branches
> *this is the absolute pathname for the file*
* Can have several files with the same file name as long as they have unique path names

#### Current Working Directory
* Always specifying the absolute pathname is tedious
* → Introduce idea of *working directory* 
	* Files are referenced relative to the working directory

#### Relative and absolute pathnames
* Absolute pathname
	* A path specified from the root of the file system to the file
* Relative pathname
	* Pathname specified from *Current Working Directory*
> . → *current directory* .. → *parent directory*

## Typical directory operations
* Create, delete, opendir, closedir, readdir, rename, link, unlink

## Properties of UNIX naming
* Simple, regular format
	* Name referring to different servers, objects, etc have the same syntax
		* Regular tools can be used where speiclaised tool would be otherwise be needed
	* Location independent
		* Objects can be distributed or migrated, and continue with the same names

## File sharing
* In multiuser system, allow files to be shread among users
* Two issues:
	* Access rights
	* Management of simultaneous access

#### Access rights
* None
	* User may not know the result of the file
	* User is not allowed to read the directory that includes the file
* Knowledge
	* User can only determine that the file exists and who the owner is
* Execution
	* The user can load and execute the progarm but cannot copy it
* Reading 
	* The user can read the file for any purpose, including copying and execution
* Appending
	* The suer can add data to the file but cannot modify or delete any of the file's contents
* Updating
	* The user can modify, delete, and add to the file's data. This includes creating the file, rewriting it, and removing all or part of the data
* Changing protection
	* User can change access rights granted to other users
* Deletion
	* User can delete the file
* Owner
	* Has all rights listed
	* May grant access rights to others using the following classes of users
		* Specific users
		* User groups
		* All for public files

#### Simultaneous Access
* Most Operating Systems provide mechanism for users to manage concurrent access to files
	* e.g. flock(), lockf(), system calls
* Typically


# File system internals
**UNIX Storage stack**

Application|&nbsp;
-----------|--------------
&nbsp;|Syscall interface: *create, open, read, write...*
FD Table <br /> OF Table | Keep track of files open by user-level processes<br /> Matches syscall interface to *virtual file system* interface
Virtual File System *VFS* | Unified interface to multiple *file systems*
File system *FS* | Hides physical location of data on the disk<br /> Exposes directory hierarchy, symbolic file names, random-access files, protection
Buffer cache <br /> Disk scheduler | Keep recently accessed disk blocks in memory<br /> Schedule disk accesses from multiple processes for performance and fairness
Device Driver | Hides device-specific protocol<br /> Exposes block-device interface (linear sequence of blocks)
Disk controller | Hides disk geometry, bad sector<br /> Exposes linear sequence of blocks
Hard disk platters | tracks, sectors

* Some popular file systems:
	* FAT16 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; FAT32 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; NTFS &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Ext2 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Ext3 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Ext4 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ReiserFS &nbsp;&nbsp;&nbsp;&nbsp; XFS
	* ISO9660 &nbsp;&nbsp;&nbsp;&nbsp; HFS+ &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; UFS2 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ZFS &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;   OCFS &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Btrfs &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; JFFS2 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ExFAT &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; UBIFS
* ***Why are there so many file systems?***
	* Different physical nature of storage devices
		* *Ext3* is optimised for magnetic disks
		* *JFFS2* is optimised for flask memory disks
		* *ISO9660* is optimised for CDROM
	* Different storage capacities
		* *FAT16* does not support drives *greather than* 2GB
		* *FAT32* becomes inefficient on drives *greater than* 32GB
		* *ZFS, Btrfs* are designed to scale to multi-TB disk arrays
	* Different CPU and memory requirements
		* *FAT16* is not suitable for modern PCs but is a good fit for many embedded devices
	* Proprietary standards
		* *NTFS* may be a nice GS, but its specification is closed

> Assumptions <br/> 
> Focus on file systems for magnetic disks: <br/>
> * Seek time → 15ms worst case  <br/> 
> * Rotational delay → 8ms worst case for 7200rpm drive <br/>
> * For comparison, disk-to-buffer transfer speed for modern drive is \~10us per 4k block <br />

> Conclusion: Keep blocks that are likely to be accessed together close to each other.

## Implementing a file system
* The *file system* must map symbolic file names into collection of block addresses
* The *file system* must keep track of 
	* Which blocks belong to which files
	* In what order the blocks form the file
	* Which blocks are free for allocation
* Given a logical region of a file, the FS must track the corresponding block(s) on disk
	* Stored in file system metadata

## File allocation methods
* A file is divided into *blocks*
	* unit of transfer to storage
* Given the logical blocks of file, what method is used to choose where to put the blocks on disk?

##### External and Internal fragmentation
* External Fragmentation
	* The space wasted external to the allocated memory regions
	* Memory space exists to satisfy a request but is unusable as it is not contiguous
	* *Cant allocate even if enough memory; memory is not contiguous*
* Internal Fragmentation
	* The space wasted internal to the allocated memory regions
	* Allocated memory may be slightly larger than requested memory; the size difference is wasted memory internal to a partition
	* *1KB allocation on 512KB block analogy*

#### Contiguous allocation
> ISO9660 (CDROM FS) <br/>

**Pros**
* Easy bookkeeping (need to keep track of the starting block and length of the file)
* Increases performance for sequential operations <br/>

**Cons**
* Need the maximum size for the file at the time of creation
* As files are deleted, free space becomes divided into many small chunks → *external fragmentation* 

#### Dynamic allocation strategies
* Disk space allocated in portions as needed
* Allocation occurs in fixed-size blocks

**Pros**
* No external fragmentation
* Does not require pre-allocating disk space
* All blocks are same sized

**Cons**
* Partially filled blocks → internal fragmentation
* File blocks are scattered across the disk
* Complex metadata management (maintain the collection of blocks for each file)

#### Linked list allocation
* Each block contains a pointer to the next block in the chain. Free blocks are also linked in a chain

**Pros**
* Only single metadata entry per file
* Best for sequential files

**Cons**
* Poor for random access
* Blocks end up stacttered across the disk due to free list eventually becoming randomised

#### File Allocation Table (FAT)
* Keep a map of the entire FS in a separate table
	* A table entry contains the number of the next block of the file
	* The last block in a file and empty blocks are marked using reserved values
* The table is stored on the disk and is replicated in memory
* Random access is fast (following the in memory list)

**Issues**
* Requires alot of memory for large disks
	* 200GB → 200\*10<sup>6</sup>\*1K-blocks → 200\*10<sup>6</sup> FAT entries = 800MB
* Free block lookup is slow
	* Searches for a free entry in table (linear search)

#### Inode based FS Structure
* Separate table(index-node or i-node) for each file
	* only keep table for open files in memory
	* fast random access
* Most popular FS structure today

**Issues**
* i-nodes occupy one or several disk areas
* i-nodes are allocated dynamically, hence free-space management is required for i-nodes
	* use fix-size i-node to simplify dynamic allocation
	* reserve the last i-node entry for a pointer (a block number) to an extension i-node
* Free-space management
	* Approach 1: Linked list of free blocks in free blocks on disk
	* Approach 2: Keep bitmaps of free blocks and free i-nodes on disk

##### Free block list
* List of all unallocated blocks
* Background jobs can re-order list for better contiguity
* Store in free blocks themselves
	* Does not reduce disk capacity
* Only one block of pointers need be kept in the main memory

##### Bit tables
* Individual bits in a bit vector flags used/free blocks
* 16GB disk with 512-byte blocks → 4MB table
* May be too large to hold in main memory
* Expensive search
	* Can optimise *e.g. two level table*
* Concentrating (de)allocations in a portion of the bitmap has desirable effect of concentrating access
* Simple to find contiguous free space

## Implementing directories
* Directories are stored like normal files
	* Directory entries are contained inside data blocks
* The *file system* assigns special meaning to the content of these files
	* A directory file is a list of directory entries
	* A directory entry contains filename, attributes, and the file i-node number
		* Maps human-oriented filename to a system oriented name

#### Fixed size vs Variable size directory entries
* Fixed size directory entries
	* Either too small 
		* DOS 8 + 3
	* Or waste too much space
		* 255 characters per file name
* Variable size directory entries
	* Freeing variable length entries can create external fragmentation in directory blocks
		* Can compact when block is in RAM

#### Searching directory listings
* Locating a file in a directory
	* Linear scan
		* Implement a directory cache in software to speed up search
	* Hash lookup
	* B-tree(100's of thousands entries)

#### Storing file attributes
* Disk addresses and attributes in directory entry → **FAT**
* Directory which each entry refers to an i=node → **UNIX**

### Trade-ff in FS block size
* File system deal with 2 types of blocks
	* Disk blocks or sectors (usually 512 bytes)
	* File system blocks 512 * 2<sup>N</sup> bytes
	* What is optimal *N*?
* Larger blocks requrie less FS metadata but more wasted space
* Smaller blocks waste less disk space(less internal fragmentation)
* Sequential access
	* Larger block size, less I/O operations required
* Random access
	* Larger block size, more unrelated data loaded
	* Spatial locality of access improves the situation
* Choosing an appropriate block size is a compromise

	* User may lock entire file when it is to be updated
	* User may lock the individual records (i.e ranges) during the updated
* Mutual exclusion and deadlock are issues for shared access

# Checkpoint
1. Consider a file currently consisting of 100 records of 400 bytes. The filesystem uses fixed blocking, i.e. one 400 byte record is stored per 512 byte block. Assume that the file control block (and the index block, in the case of indexed allocation) is already in memory. Calculate how many disk I/O operations are required for contiguous, linked, and indexed (single-level) allocation strategies, if, for one record, the following conditions hold. In the contiguous-allocation case, assume that there is no room to grow at the beginning, but there is room to grow at the end of the file. Assume that the record information to be added is stored in memory.
	* Contiguous
	* Linked
	* Indexed
	1. added in beginning
		* 100r/101w
		* 0r/1r
		* 0r/1w
	2. added in middle
		* 50r/51w
		* 50r/2w
		* 0r/1w
	3. added at the end
		* 0r/1w
		* 100r/2w
		* 0r/1w
	4. removed from beginning
		* 0r/0w
		* 1r/0w
		* 0r/0w
	5. removed from middle
		* 49r/49w
		* 50r/1w
		* 0r/0w
	6. removed from end
		* 0r/0w
		* 99r/1w
		* 0r/0w
2. In the previous example, only 400 bytes is stored in each 512 byte block. Is this wasted space due to internal or external fragmentation?
	* Internal fragmentation
3. Older versions of UNIX allowed you to write to directories. Newer ones do not even allow the superuser to write to them. Why?
	* To prevent total corruption of the file system *e.g. cat /dev/zero > /*
4. Given a file which varies in size from 4KiB to 4MiB, which of the three allocation schemes (contiguous, linked-list, or i-node based) would be suitable to store such a file? If the file is access randomly, how would that influence the suitability of the three schemes?
	* Contiguous is not really suitable for a variable size file as it would require 4MiB to be pre-allocated, which would waste a lot of space if the file is generally smaller. Either linkedlist or i-node based allocation would be preferred. Adding random access to the situation (well supported by contiguous or i-node based) which further motivate i-node based allocation to be most appropriate
5. Why is there a VFS layer in UNIX?
	* Provides framework to support multiple file system types without requiring each file system to be aware of other file system types
	* Provides transparent access to all supported file systems including network file system
	* Provides a clean interface between the file system independent kernel code and the file system specific kernel code.
	* Provides support for *special* file system types like /proc
6. How does choice of block size affect file system performance? 
	* Sequential access
		* Larger block size, the fewer I/O operations required and more contiguous disk accesses. Compare loading a single 16K block with loading 32-512 byte blocks
	* Random access
		* Larger the block size, the more unrelated data are loaded. Spatial locality of access can improve the situation.
7. Is *open()* system call in UNIX essential? What would be the consequence of not having it?
	* Not essential
	* Read and write system calls would have to be modified such that
		* The filename is passed on each call to identify the file to operate on
		* With a nfile descriptor to identify the open session that is returened by open, the syscalls would also need to specify the offset into the file that the syscall would need to use.
		* Effectively opening and closing the file on each read or write would reduce performance
8. Some operating system provide a rename system call to give a file a new name. What would be different compared to the approach of simply copying the file to a new name and then deleting the original file?
	* A rename system call would just change the string of characters stored in the directory entry. A copy operation would result in a new directory entry, and more importantly, much more I/O as each block of the original file is copied into a newly allocated block in the file. Additionaly, the original file blocks need de-allocating after the copy finishes, and the original name removed from the directory. A rename is much less work, and thus way more efficient than copy approach
9. In both UNIX and Windows, random file access is performed by having a special system call that moves the current position in the file so the subsequent read or write is performed from the new position. What would be the consequence of not having such a call. How could random access be supported by alternative means?
	* Without being able to move the file pointer, random access would be extremely inefficient, as one would have to read sequentially from the start each time until appropriate offset is arrived at,  or the extra argument would need to be added to *read* or *write* to specify the offset for each operation.
10. Why does Linux pre-allocate up to 8 blocks on a write to a file?
	* Pre allocating provides better locality when writes to independent files are interleaved
11. Linux uses *buffer cache* to improve performance. What is the drawback of such a cache? In what scenario is it problematic? What alternative would be more appropriate where a buffer cache is inappropriate?
	* The buffering writes in the buffer cache provide the opportunity for data to be lost if the system stops prior to cache being flushed
	* Removable storage devices are particularly problematic if users dont *unmount* them first
	* Robustness can be improved by using a write-through cache at the expense of poor wrote performance
12. The UNIX inode structure contains a reference count. What is the reference count for? Why can't we just remove the inode checking the reference count when a file is detected?
	* Inodes contain a reference count due to hard links. The reference count is equal to the number of directory entries that reference the inode. For hard-linked files, multiple directory entries reference a single inode. The inode must not be removed until no directory entries are left (i.e. reference count is 0) to ensure that the filesystems remain consistent
13. Inode-based file systems typically divide a filesystem partition into block groups. Each block group consists of a number of contiguous physical disk blocks. Inodes for a given block group are stored in the same physical location as the block groups. What are the advantages or this scheme? Are there any disadvantages?
	* Each group contains a redundant superblock. This makes the file system more robust to disk block failures
	* Block groups keep inodes physically closer to the files they refer to than they would be (on average) on a system without block groups. Since accessing and updating files also involve accessing or updating its inode, the inode and the file's block close together reduces disk seek time, and thus improves performance. The OS must take care that all blocks remain within the block group of their inode.
14. Assume an inode with 10 direct blocks, as well as single, double and triple indirect block pointers. Taking into account creation and accounting of the indirect blocks themselves, what is the largest possible number of block reads and writes in order to:
	* Read 1 byte
	* Write 1 byte
	1. To write 1 byte in the worst case, 
		* 4 writes: *create single indirect block, create double indirect block, create triple indirect block, write data block*
		* 3 reads, 2 writes: *read single indirect, read double indirect, read triple indirect, write triple indirect, write data block*
		* Other combinations are possoble
	2. To read 1 byte, in the worst case,
		* 4 reads: *read single indirect, read double indirect, read triple indirect, read data block*
15. A typical UNIX inode stores both the file's size and the number of blocks currently used to store the file. Why store both? Should not *blocks = size/block size*?
	* Blocks used to store the file are only indirectly related to file size
		* The blocks used to store a file includes and indirect blocks used by the file system to keep track of the file data blocks themselves
		* File systems only store blocks that actually contain file data. Sparsely populated files can have large regions that are unused within a file.
16. How can deleting a file leave a inode-based file system (like ext2fs in Linux) inconsistent in the presence of a power failure.
	* Deleting a file consists of three separate modifications to the disk:
		* Mark disk blocks as free
		* Remove the directory entry
		* Mark the inodes as free
	* If the system only completes a subset of the operations (due to power failures or the like), the file system is no longer consistent. 
17. How does adding journalling to a file system avoid corruption in the presence of unexpected power failures.
	* Adding journal addresses the issue by grouping file system updates into transactions that should either completely fail or succeed. These transactions are logged prior to manipulating the file system. In the presence of failure, the transaction can be completed by replaying the updates remaining in the log.

# UNIX File management
> Older systems only had single file system
> * They had file system specific open, close, read, write, etc calls.
> * However, modern systems need to support many file system types (*ISO9660, MSDOS, ext2fs, tmpfs*)

**Supporting multiple file systems**
* Change the file system code to understand different file system types
	* Prone to code bloat, complex, non-solution
* Provide a framework that separates file system independent and file system dependent code
	* Allows different file systems to be *plugged in*

## Virtual File System *VFS*
* Provide single system call interface for many file systems 
	* *e.g. UFS, Ext2, XFS, DOS, ISO9660,...*
* Transparent handling of network file systems 
	* *e.g. NFS, AFS, CODA*
* File-based interface to arbitrary device drivers (*/dev*)
* File-based interface to kernel data structures (*/proc*)
* Provides an indirection layer for system calls
	* File operation table set up at file open time
	* Points to actual handling code for particular type
	* Further file operations redirected to those funcitons

### VFS Interface
**Two major data types**
* VFS
	* Represents all file system types
	* Contains pointers to functions to manipulate each file system as a whole (e.g. mount, unmount)
		* Form a standard interface to the file system
* Vnode
	* Represents a file(inode) in the underlyinf filesystem
	* Points to the real inode
	* Contains pointers to functions to manipulate files/inodes (e.g. open, close, read, write, ...)

###### VFS and Vnode structures

struct vnode | &nbsp;
-------------|-------
Generic (FS-independent) fields | → size <br/> → uid, gid <br/> → ctime, atime, mtime <br /> ...
fs_data | → FS-Specific fields (*Block group number, Data block list, ...*)
vode ops | → ext2fs_read (*FS-specific implementation of vnode operations*) <br /> ext2fs_write <br /> ...

struct vfs | &nbsp;
-----------|-------
Generic (FS-independent) fields | → Block size <br /> → Max file size <br /> ...
fs_data | FS-specific fields (*inodes per group, Superblock addresses, ...*)
vfs ops | ext2_unmount (*FS-specific implementation of FS operations*) <br /> ext2_getroot

### OS161's VFS
```c
struct fs {
	int (*fs_sync)(struct fs *); // Force the filesystem to flush its content to disk
	const char *(*fs_getvolname)(struct fs *); // Retrieve the volume name
	struct vnode *(*fs_getroot)(struct fs *); // Retrieve the vnode associated with the root of the file system
	int (*fs_unmount)(struct fs *);	// Unmount the filesystem. Note: mount called via funcion ptr passed to vfs_mount
	void *fs_data; // Private file system specific data
}
```
```c
struct vnode {
	int vn_refcount; // count the number of 'references' to this vnode
	struct spinlock vn_countlock; // Look for mutual exclusive access to counts
	struct fs *vn_fs; // Pointer to FS containing the vnode
	void *vn_data; // Poitner to FS specific vnode data (e.g. in memory copy of inode)

	const struct vnode_ops *vn_ops; // Array of pointers to functions operating on vnodes
};
```

## File descriptor & Open file tables
> System call interface: <br />
> *fd = open('file', ...) <br />
> read(fd,...); wrtite(fd,...); close(fd);*

> VFS interface: <br />
> *vnode = vfs_open('file', ...) <br />
> vop_read(vnode, uio); vop_write(vnode, uio); vop_close(vnode);*

#### File descriptors
* Each open file has a file descriptor
* read/write/lseek/... use them to specify which file to operate on
* **State associated with file descripter**
	* File pointer
		* Determines where in the file the next read or write is performed
	* Mode 
		* Was the file opened read-only?, etc

* Use vnode numbers as file descriptors and add file pointer to the vnode
	* Problems:
		* What happens when we concurrently open the same file twice?
			* We should get two separate file descriptors and file pointers..
	* Single global open file array
		* *fd* is an index into the array
		* Entries contain file pointer and pointer to a vnode
	* Issues:
		* File descriptor *1* is *stdout*
			* *stdout* is 
				* console for some processes
				* A file for others
		* every *1* needs to be different per process
**Per-process file descriptor array**
* Each process has its own open file array
	* Contains fp, v-ptr, etc
	* *Fd 1* can point to any vnode for each process (console, logfile)
* Issue:
	* Fork
		* Fork defines that the child shares the file pointer with the parent
	* Dup2
		* Also defines file descripters share the file pointer
	* With per-process table, we can only have independent file pointers
		* even when accessing the same file
**Per-process *fd* table with global open file table**
* Per process file descriptor array
	* Contains pointers to *open file table entry*
* Open file table array
	* Contain entries with a fp and pointer to a vnode
* Provides
	* Shared file pointers if required
	* Independent file pointers if required
> Used by Linux and most other Unix operating systems

## Buffer Cache
**Buffer**
* Temporary storage used when transferring data between two entities
	* Especially when the entities work at different rates
	* Or when the unit of transfer is incompatible
	* *e.g. between application program and disk*
**Buffering disk blocks**
* Allow applications to work with arbitrarily sized region of a file
	* However, apps can still optimise for a particular block size
* Writes can return immediately after copying to kernel buffer
	* Avoids waiting until write to disk is complete
	* Write is schedutled in the background
* Can implement read-ahead by pre-loading next block on disk into kernel buffer
	* Avoids having to wait until next read is issued

**Cache**
* Fast storage used to temporarily hold data to speed up repeated access to the data
* *e.g. main memory can cache disk blocks*
**Caching disk blocks**
* On access
	* Before loading block from disk, check if it is in cache first
	* Avoids disk accesses
	* Can optimise for repeated access for single or several processes
<hr />

Buffering and caching are related
* Data is read into the buffer; an extra independent cache copy would be wastful
* After use, block should be cached
* Future access may hit cached copy
* Cache utilises unused kernel memory space
	* May have to shrink, depending on memory demand
**UNIX buffer cache**
* On read:
	* Hash the device number, block number
	* Check if match in buffer cache
		* Yes, simply use in memory copy
		* No, follow the collision chain
		* If not found, load block from disk into buffer cache

*What happens when the buffer cache is full and we need to read another block into memory?*
* We must choose an existing entry to replace
* Need a policy to choose a victim
	* Can use FIFO
	* Least recently used
		* Timestamps required for LRU

**File system consistency**
* File data is expected to survive
* Strict LRU could keep modified critical data in memory forever if it is frequently used
* Generally, cached disk blocks are prioritised in terms of how critical they are to file system consistency
	* Directory blocks, inode blocks if lost can corrupt entire file system
		* e.g. imagine losing the root directory
		* These blocks are usually scheduled for immediate write to disk
	* Data blocks if lost corrupt only the file they are associated with
		* These blocks are only scheduled for write back to disk periodically
		* in UNIX, flushd (flush daemon) flushes all modified blocks to disk every 30 seconds
* Alternatively, use a write-through cache
	* All modified blocks are writeen immediately to disk
	* Generates much more disk traffic
		* Temporary files written bacl
		* Multiple updates not combined
	* Used by DOS
* Gave ok consistency when
	* Floppies were removed from drives
	* Users are constantly resetting(or crashing) their machines
* Still used (USB Storage devices)

# Memory management
### Process
* One or more threads of execution
* Resources required for execution
	* Memory (RAM)
		* Program code(*text*)
		* Data (*initialised, uninitialised, stack*)
		* Buffers held in the kernel on behalf of the process
	* Others
		* CPU Time
		* Files, disk space, printers, etc

### Memory Hierarchy
> registers → Cache → main memory → electronic disk → magnetic disk → optical disk → 
magnetic tapes

* Ideally, programmers want memory thats
	* Fast
	* Large
	* Nonvolatile
* Not possible
* Memory management coordinates how memory hierarchy is used
	* Focus on RAM <=> Disk

### OS Memory management
* Keeps track of what memory is in use and what memory is free
* Allocates free memory to process when needed
	* And deallocates it when they don't
* Manages the transfer of memory between RAM and the disk
* Two broad classes of memory management systems
	* Those that transfer processes to and from external storage during execution
		* Called swapping or paging
	* Those that dont
		* Simple
		* Might find this scheme in an embedded device, dumb phone, or smartcard

#### Basic memory management
*Monoprogramming without swapping or paging*
* Three of organising memory
	* User program → Operating system in RAM
	* Operating system in ROM → User program
	* Device drivers in ROM → User in ROM → Operating system in RAM
* **Monoprogramming** 
	* is OK if 
		* only have one thing to do
		* memory available approximately equats to memory required
	* otherwise
		* Poor CPU utilisation in the presence of I/O waiting
		* Poor memory utilisation with a varied job mix

#### Multiprogramming
* OS aims to
	* Maximise memory utilisation
	* Maximise CPU utilization
		* ignore battery/power-management issues
	* Subdivide and run more than one process at once
		* Multiprogramming/Multitasking
**How to divide memory between processes?**
* Given a workload, how do we
	* Keep track of free memory?
	* Locate free memory for a new process?
* Overview of evolution of simple memory management
	* Static (fixed partitioning) approaches:
		* Simple, predicable workloads of early computing
	* Dynamic (partitioning) approaches:
		* More flexiable computing as compute power and complexity increased
* Introduce virtual memory
	* Segmentation and paging
* One approach
	* Divide memory into fixed equal sized partitioned
	* Any process <= partition size cam be loaded into any partition
	* Partitions are free or busy

##### Simple memory management: Fixed, equal sized partition
* Any used space in the partiton is wasted
	* Called internal fragmentation
* Processes smaller than main memory, but larger than a partition cannot run
* Divide memory at boot time into a selection of different sized paritions
	* Can base sizes on expected workload
* Each partition has queue:
	* Place process in queue for smallest partition that it fits in
	* Processes wait for when assigned partition is empty to start
* Issue:
	* Some partitions may be idle
		* Small jobs available, but only large partitions free
		* Workload could be unpredictable
* Alternative queue strategy:
	* Single queue; search for any jobs that fit
		* Small jobs in large partition if necessary
	* Increases internal memory fragmentation
**Fixed partition summary**
* Simple
* Easy to implement
* Can result in poor memory utilisation
	* Due to internal fragmentation
* Used on IBM system  operating system (OS/MFT)
* Still applicable on embedded systems
	* Static workload known in advance

##### Dynamic partitioning
* Partitions are of variable length
	* Allocated on-demand from ranges of free memory
* Proces is allocated exactly what it needs
	* Assumes a process knows what it needs
* Prone to external fragmentation(unusable holes)
	* External Fragmentation
		* Space wasted external to the allocated memory regions
		* Memory space exists to satisfy a request, but it is unusable as it is not contiguous
	* Internal Fragmentation
		* Space wasted internal to the allocated memory regions
		* Allocated memory may be slightly larger than requested memory; this size difference is wasted memory internal to a partition
* Also applicable to *malloc()* like in application allocators
* Given a region of memory, basic requirements are:
	* Quickly locate a free partition satisfying the request
		* Minimise CPU time search
	* Minimise external fragmentation
	* Minimise memory overhead of bookkeeping
	* Efficiently support merging two adjacent free partitions into a larger partition
**Classic approach**
* Represent available memory as a linked list of available *holes* (free memory ranges)
	* Base, size
	* Kept in order of increasing addresses
		* Simplifies merging of adjacent holes into larger holes
	* List nodes be stored in the *holes* themselves
**Dynamic partitioning placement algorithm**
* First-fit algorithm
	* Scan the list for the first entry that fits
		* If greater in size, break it into an allocated and free part
		* Intent: Minimise amount of searching performed
	* Aims to find a match quickly
	* Biases allocation to one end of memory
	* Tends to preserve larger blocks at the end of memory
* Next-fit algorithm
	* Like *first-fit*, except it begins its search from the point in list where the last request succeeded instead of the beginning
		* (Flawed) intuition: spread allocation more uniformly over entire memory to avoid skipping over small holes at start of memory
		* Performs worst tha *first-fit* as it breaks up the large free space at end of memory
* Best-fit algorithm
	* Chooses the block that is closest in size to the request
	* Performs worse than *first-fit*
		* Has to search complete list
				* Does more work than *first-fit*
			* Since smallest block is chosen for process, the smallest amount of external fragmentation is left
			* Create lots of unusable holes
* Worst-fit algorithm
	* Chooses the block that is largest in size (*worst-fit*)
		* (whimsical) idea is to leave usable fragment over
	* Poor performer
		* Has to fo more work (like *best-fit*) to search complete list
		* Does not result in significanly less fragmentation
**Summary**
* First-fit is generally better than the others and easiest to implement
* They are simple solutions to a still-existing OS or application service/function -- memory allocation
> Largely have been superseded by more complex and specific allocation strategies <br/>
> Typical in-kernel allocators used are *lazy buddy* and *slab* allocators
* Issues:
	* Relocation
		* How does a process run in different locations in memory?
	* Protection
		* How do we prevent processes interfering with each other?

###### Compaction
* Can reduce external fragmentation by *compaction*
	* Shuffle memory contents to place all free memory together in one large block
	* Only if we can relocated running programs?
		* Pointers?
	* Generally requires hardware support

**When are memory addresses bound?**
*souce program → compliler/assmebler → object module → linkage editor (with other object modules) → load module → load → in memory binary memory image (execution/run time)*
* Compile/Link time
	* Compiler/Linker binds the addresses
	* Must know *"run"* location at compile time
	* Recompile if location changes
* Load time
	* Compiler generates relocatable code
	* Loader binds the addresses at load time
* Run time
	* Logical compile-time addresses translated to physical addresses by *special hardware*

**Base and Limit registers**
* Also called
	* Base and bound registers
	* Relocation and limit registers
* restrict and relocate currently active process
* must be changed at
	* load time
	* relocation(compaction time)
	* on context switch
* Pro:
	* Supports protected multi-processing/tasking
* Cons:
	* Physical memory allocation must still be contiguous
	* The entire process must be in memory
	* Do not support partial sharing of address spaces
		* No shared code, libraries, or data structures between processes

**Timesharing**
* So far, we have a system suitable for batch system
	* Limited number of dynamically allocated processes
		* Enough to keep CPU utilised
	* Relocated at runtime
	* Protected from each other
* We need more than just a small number of processes running at once
* Need to support a mix of active and inactive processes, of varying longetivity

**Swapping**
* A process can be swapped temporarily out of memory to a *backing* store, and then brought back into memory for continued execution
* *backing store* → fast disk large enough to accomodate copies of all memory images for all users; must provide direct access to these memory images
* *can prioritize* → lower priority process is swapped out so higher priority process can be loaded and executed
* Major part of swap time is transfer time; total transfer time is directly proportional to the *amount* of memory swapped
	* slow

*What if process is larger than main memory?*

### Virtual memory
* Developed to address the issue identified with simple schemes covered thus far.
* Two classic variants:
	* Paging (dominant)
	* Segmentation (not covered in the course)
*Paging overview*
* Partition physical memory into small equal sized chunks → called *frames*
* Divide each process's virtual (logical) address space into same size chunks → called *pages*
	* Virtual memory addresses consist of *page number* and *offset* within the page
* OS maintains a *page table*
	* contains the frame location for each page
	* used by *hardware* to translate each virtual address to physical address
	* The relation between virtual addresses and physical memory addresses is given by page table
* Process's physical memory **does not** have to be contiguous

##### Paging
* No external fragmentation
* Small internal fragmentation (in last page)
* Allows sharing by *mapping* several pages to the same frame
* Abstracts physical organisation
	* Programmer only deal with virtual addresses
* Minimal support for logical organisation
	* Each unit is one or more pages

# Checkpoint
1. Describe internal and external fragmentation
	* External fragmentation - total memory space exists to satisfy a request but is not contiguous
	* Internal fragmentation - Allocated memory may be slightly larger than requested memory; the size difference is memory internal to partition, but is not being used
2. What are the problems with multiprogrammed systems with fixed-partitioning?
	* Internal fragmentation
	* Inability to run processes greater in size than a partition, but smaller then memory
3. Assume a system protected with base-limit registers. What are the advantages and problems with such a protected system (compared to either a unprotected system or a paged VM system)?
	* Advantages:
		* Applications are protected from each other
		* Memory for applications can be allocated dynamically and hardware translates the application (logical) addresses to allocated addresses
		* Multiple concurrent executions of the same application is possible
		* Compaction is also possible
	* Disadvantages
		* Partitions must be contiguous -- external fragmentation
		* Entire process must be in memory
		* Cannot practically memory with other processes
4. A program is to run on a multiprogrammed machine. Describe at which points in time during program development to execution time where addresses within the program can be bound to the actual physical memory it uses for execution? What are the implication of using each of the three binding times?
	* Compile/Link time binding
		* The executable itself contains the actual physical addresses it will use during the execution. It can only run at one location, and only a single copy can run at a time, unless the executable is recompiled/relinked to a new location in physical memory prior to each execution
	* Load time binding
		* Addresses within the executable are annotated such thath when the program is loaded into physical memory, the loader can bind the addresses to the correct locaton within physical memory as the program is loaded (copied) into memory. The process slows loading (increases start up latency), and increases executable file size (to hold annotations, minor point)
	* Run-time binding
		* The compiler/linker generated addresses are bound to a logical address space (an abstract memory layout). The program executes using the logical addresses independent of where it is loaded into physical memory. At run-time, the logical addresses are translated into the appropriate physical addresses by specialised hardware on each memory reference.
		* Runtime binding is the most flexible (depending of the translation hardware available e.g. *page VM MMU*), but it incurs the cost of translation on every memory reference, which can be significant.
5. Describe four algorithms for memory allocation of contiguous memory, and describe their properties
	* First fit
		* Scan memory region list from start for first fit
		* Tends to skip over potentially many regions at the start of list
	* Next fit
		* Scan memory region list from point of last allocation to next fit
		* Braeks up large block at end of memory with any reduction in searching
	* Best fit
		* Picks the closest free region in the entire list
		* Tends to leave small unusable regions and slower due to requirement of search the entire list
	* Worst fit
		* Find the worst fit in the entire list
		* Fragmentation is still an issue as it searches the entire list
6. What is compaction? Why should it be used?
	* Moving all the allocated regions of memory next to each other (e.g. to the bottom of memory) to free up larger contiguous free regions

# Virtual memory
### Paged based virtual memory
* Divided into equal sized *pages*
* A *mapping* is a translation between
	* A page and a frame
	* A page and null
* Mappings defined at runtime
	* They can change
* Address space can have holes
* Process does not have to be contiguous in physical memory

### Typical Address Space Layout
* Stack region is at top and can grow down
* Heap has free space to grow up
* Text is typically read only
* Kernel is in a reserved, protected, shared region
* 0<sup>th</sup> page is typically not used

##### Virtual Address space
*Programmer's perspective: logically present <br />
*System's perspective: Not mapped, data on disk*
* A process may be only partially resident
	* Allows OS to store individual pages on disk
	* Saves memory for infrequently used data & code
* What happens if we access non-resident memory?

### Page faults
* Referencing an invalid page triggers a page fault
	* An exception handled by the OS
* Broadly, two standard page fault types
	* Illegal address (protection error)
		* Signal or kill the process
	* Page not resident
		* Get an empty frame
		* Load page from disk
		* Upate page (translation) table (enter frame number, set valid bit, etc)
		* Restart the faulting instruction
> Page table for resident part of address space

### Shared pages
* Private code and data
	* Each process has own copy of code and data
	* Code and data can appear anywhere in the address space
* Shared code
	* Single copy of code shared between all processes executing it
	* Code must not be self modifying
	* Code must appear at same address in all processes
> Two (or more) processes running the same program and sharing the same text section

### Page table structure
* Page table is (logically) an array of frame numbers
	* Index by page number
* Each page-table entry (PTE) also has other bits

##### PTE Attributes (bits)
* Present/Absent bit
	* Also called *valid bit*, it indicates a valid mapping for the page
* Modified bit
	* Also called *dirty bit*, indicates the page may have been modified in memory
* Reference bit
	* Indicates the page has been accessed
* Protection bits
	* Read permission, write permission, execute permission
	* Or combinations of above
* Caching bit
	* Use to indicate processor should bypass the cache when accessing memory
		* e.g. to access device registers or memory

#### Address Translation
* Every (virtual) memory address issued by the CPU must be translated to physical memory
	* Every load and every store instruction
	* Every instruction fetch
* Need translation hardware
* In paging system, translation involves replace page number with frame number

## Virtutal Memory Summary
*Virtual and physical memory chopped up in pages/frames*
* Programs use **virutal addresses**
* Virtual to physical mapping by **MMU**
	* First check if page present(present/absent bit)
	* if yes → addresses in page table form MSBs in physical addresses
	* if no → bring in the page from disk
		* → page fault

## Page Tables
* Assume we have 
	* 32-bit virtual addresses (4GB address space)
	* 4KB page size
	* How many table entries do we need for one process?
* Assume we have 
	* 64-bit virtual address (*humungous* address space)
	* 4KB page size
	* How many page table entries do we need for one process?
* Problem:
	* Page table is very large
	* Access has to be fast, lookup for every memory reference
	* Where do we store the page table?
		* registerss? 
		* main memory?

* Page tables are implemented as data structures in main memory
* Most processes do not use the full 4GB address space
	* ~ 1 MB text, ~10 MB data, ~ 0.1 MB stack
* We need a compact representation that does not waste space
	* But still fast tp search
* Three basic schemes:
	* Use data structures that adapt to sparsity
	* Use data structures which only respresent resident pages
	* Use VM techiniques for page tables

#### Two level page table
* 2<sup>nd</sup> level page tables representing unmapped pages are not allocated
	* Null in the top-level page table

### Inverted page table (IPT)
* *Inverted page table* is an array of page numbers sorted (indexed) by frame number(it's frame table)
* Algorithm:
	* Compute hash of page number
	* Extract index from hash table
	* Use this to index into inverted page table
	* Match the PID and page number in the IPT entry
	* If match, use index vaoue as frame number for translation
	* If no match, get next candidate IPT entry from chain field 
	* If NULL chain entry → page fault
* Properties
	* IPT grows with size of RAM, not virtual address space
	* Frame table is needed anyway (for page replacement)
	* Need separate data structure for non-resident pages
	* Saves a vast amount of space
	* Used in some IBM and HP workstations

### Improving the IPT: Hashsed page table
* Retain fast lookup of IPT
	* Singe memory reference in best case
* Retain page table size based on physical memory size (not virtual
)
	* Enable efficient frame sharing
	* Support more than one mappong for same frame
* Sizing:
	* HPT size based on physical memory size
	* With sharing
		* Each frame can have more than one PTE
		* More sharing increases number of slots used
			* Increases collision likelihood
	* However, we can tune HPT size based on
		* Physical memory size
		* Expected sharing
		* Hash collision avoidance

### VM implementation issue
* Performance
	* Each virtual memory reference can cause two physical memory access
		* One to fetch the page table entry
		* One to fetch/store the data
		* →Intolerable performance impact
* Solution:
	* High speed cache for page table entries
		* Called *translation look-aside buffer*
		* Contains recently used page table entries
		* Associative, high speed memory, similar to cache memory
		* May be under OS control (unlike memory cache)

### Translation look aside buffer
* Given virtual address, processor examines the TLB
* If matching PTE found (TLB *hit*), the address is translated
* Otherwise (TLB *miss*, the page number is used to index the process's page table
	* If PT contains a valid entry, reload TLB and restart
	* Otherwise (page fault), check if page is on disk
		* If on disk, swap
		* Otherwise, allocate new page or raise an exception
* Properties
	* Page table is (logically) an array of frame numbers
	* TLB holds a (recently used) subset of PT entries
		* Each TLB entry must be identified (tagged) with the page number it translates
		* Access is by associative lookup
			* All TLB entries' tags are concurrently compared to page number
			* TLB is associative (or content-addressable) memory
	* May or may not be under direct OS control
		* Hardware loaded TLB 
			* On miss, hardware performs PT lookup and reloads TLB
			* e.g. x86, ARM
		* Software loaded TLB
			* On miss, hardware generates TLB miss exception, and exception handler reloads TLB
			* e.g. MIPS, itanium
	* TLB size → *typically 64 ~ 128 entries*
	* Can have separate TLB for intruction fetch and data access
	* TLB can also be used with inverted page tables (and others) 
* TLB and context switching
	* TLB is a shared piece of hardware
	* Normal page tables are per process
	* TLB entries are *process specific*
		* On context switch, need to *flush* the TLB (invalidate all entries)
			* high context-switching overhead(intex x86)
		* Or tag entries with *address-space ID* (ASID)
			* called a *tagged* TLB
			* used (in some form) on all modern architectures
			* TLB entry: ASID, page number, frame number, valid and write protect bits
* Effect:
	* Without TLB
		* Average number of physical memory references per virtual reference
			* → 2
	* With TLB (Assume 99% hit ratio)
		* Average number of physical memory references per virtual reference
			* → 0.99 * 1 + 0.01 * 2
			* → 1.01
* TLB Summary:
	* Fast associative cache of page table entries
		* Contains subset of the pagetable
	* May or may not be under OS control
		* Hardware loaded TLB
			* On miss, hardware performs *page table* lookup and reloads TLB
		* Software loaded TLB
			* On miss, hardware generates a TLB miss exception, and exception handler reloads TLB
	* TLB is still a hardware-based translator

#### R3000 TLB Handling
* TLB refill is handled by software
	* An exception handler
* TLB refil exceptions accessing kuseg are expected to be frequent
	* CPU optimised for handling kuseg TLB refils by having a special execption handler for TLB refills

#### Special Exception Vector
* Can be optimised for TLB refill only
	* Does not need to check the exception type
	* Does not need to save any registers
		* It uses a specialised assembly routine that only uses k<sub>0</sub> and k<sub>1</sub>
	* Does not check if PTE exists
		* Assumes virtual linear array
	* With careful data structure choice, exception handler can be made very fast

#### MIPS VM related exceptions
* TLB refill
	* Handled via special exception vector
	* Needs to be very fast
* Others handled by the general exception vector
	* TLB Mod
		* TLB modify exception; attempt to write to a read only page
	* TLB Load
		* Attempt it load from a page with an invalid translation
	* TLB Store
		* Attempt to store to a page with an invalid translation
	> There can be slower as they are mostly caused by an error or non resident pages

### c0 Registers
* c0_EPC
	* Address of where to restart after exception
* c0_status
	* Kernel/User mode bits, interrupt control
* c0_cause
	* What caused the exception
* c0_badvaddr
	* The address of fault

#### The TLB and EntryHi, EntryLo
* c0_EntryHi, c0_EntryLo, c0_Index → used to read and write individual TLB entries
* Each TLB entry contains 
	* EntryHi to match page number and ASID
	* EntryLo which contains frame number and protection

##### c0 Index Register
* Used as an index to TLB entries
	* Single TLB entries are manipulated/viewed through EntryHi and EntryLo registers
	* Index register specifies with TLB entry to change/view

### Special TLB management instructions
* TLBR
	* TLB read
		* EntryHi and EntryLo are loaded from the entry pointer to by the index register
* TLBP
	* TLB probe
		* Set entryHi to the entry you with to match, index register is loaded with the index to the matching entry
* TLBWR
	* TLB write random
		* Write EntryHi and EntryLo to a psuedo-random location in the TLB
* TLBWI
	* TLB wrote indexed
		* Write EntryHi and EntryLo to the location in the TLB pointed to by the index register

##### Cooprocessor 0 registers on a refill exception
* c0.EPC →PC
* c0.cause.ExcCode → TLBL; if read fault
* c0.cause.ExcCode →TLBS; if write fault
* c0.BadVaddr → faulting address
* c0.EntryHi.VPN →page number of faulting adress
* c0.status → kernel mode, interrupts disabled
* c0.PC → 0x8000 0000

### Outline of TLB misshandling
* Software:
	* Look up PTE corresponding to the faulting address
	* If found:
		* load c0_EntryLo with translation
		* load TLB using TLBWR instruction
		* return from exception
	* Otherwise, page fault
* The TLB entry (*i.e c0_entryLo*) can be:
	* (Theoretically) created on the fly or
	* Stored completely in the right format page table
		* more efficient

# Checkpoint
1. What is swapping? What are the benefits? What is the main limitation?
	* Swapping is where a process is brought into main memory in its entirety, run for a while, and then put completely back on disk
	* Swapping allows OS to run more programs than what would fit in memory if all programs remained resident in memory
	* Swwapping is slow as it has to copy the entire program's in-memory image out to disk and back
2. What is paging?
	* Paging is where main memory is divided into equal-sized chunks (frames) and the programs address space (virtual address space) is also divided up into matching-sized chunks (pages). Memory is transferred to and from disk in units of pages
3. Why do all virtual memory system page sizes have to be a power of 2?
	* Lower buts of a virtual address is not translated and passed through the MMU to form a physical address
4. What is a *TLB*? What is its function?
	* A *translation lookaside buffer* is an associative cache of page table entries used to speed up the translation of virtual addresses to physical addresses
5. Given a two-level page table (in physical memory), what is the average number of physical memory accesses per virtual memory access in the case where the TLB has a 100% miss ratio, and the case of a 95% hit ratio
	* 3
	* 1 * .95 + .05 * (1 + 2) = 1.1
6. What are two broad categories of events causing page faults? What other event might cause page faults?
	* Illegal memory references and access to non-resident pages
	* Page faults may be used to set reference and dirty bits on architectures that do not support them in hardware
7. What is the most appropriate page table type for large virtual address spaces that are sparsely populated (e.g. many single pages scattered through memory)?
	* The 2-level suffers from internal fragmentation of page table nodes themselves. The IPT and HPT are best as it is searched via hash, and not based on the structure of virtual address space
8. What is temporal and spatial locality?
	* Temporal locality - recenly accessed items are likely to be accessed in the near future
	* Spatial locality - items with addresses near one another tend to eb referenced close together in time


# Demand paging/Segmentation
* With VM, only parts of the program image need to be resident in memory for execution
* Can transfer presently unused pages/segments to disk
* Reload non-resident pages/segment on *demand*
	* Reload is triggered by a page or segmentation fault
	* Faulting process is blocked and another is scheduled
	* When page/segment is resident, faulting process is restarted
	* May require freeing up memory first
		* Replace current resident page/segment
		* How determine replace *victim?*
	* If *victim* is unmodified(*clean*), can simpliy disregard it
		* This is reason for maintaining *dirty* bit in the Page table
* Why does demand paging/segmentation work?
	* Program executes at full speed only when accessing the resident set
	* TLB misses introdice delays of several microseconds
	* Page/segment faults introduce delays of several milliseconds
	* Why do it?
		* Less physical memory required per process
			* Can fit more processes in memeory
			* Improved change of finding a runnable one
		* Principle of locality

### Princinple of locality
* An important observation comes from empirical studies of properties of programs
	* Programs tend to reuse data and instructions they have used recently
	* 90 / 10 rule → *A program spends 90% of its time in 10% of its code*
* We can exploit this *locality of references*
* An implication of locality is that we can reasonably predict what *instructions* and *data* a program will use in the near future based on its accesses in the recent past
* Two different types of locality:
	* Temporal locality
		* recently accessed items are likely to be accessed in the near future
	* Spatial locality
		* Items whos addresses are near one another tend to be referenced close together in time

### Working set
* Pages/segments required by an application in a time window is called its memory *working set*
* Working set is an approximation of program's locality
	* If too small, will not encompass several locality
	* If too large, will encompass several localities
	* If infinity, will encompass entire program
	* Working set's size is an application specific tradeoff
* System should keep resident at least a process's working set
	* Process executes while it remains in its working set
*  Working set tends to change gradually
	* Get only few page/segment faults during a time window
	* Possible(but hard) to make intelligent guesses about which pieces will be needed in the future
		* May be able to pre-fetch page/segments

### Thrashing
* CPU utilisation tends to increase with the degree of multiprogramming
	* Number of processes in system
* Higher degrees of multiprogramming - less memory available per process
* Some process's working sets may no longer fit in RAM
	* Implies an increasing page fault rate
* Eventually, many processes have insufficient memory
	* Can't always find a runnable process
	* Decreasing CPU utilisation
	* System become I/O limited
* This is called *thrashing*

#### Recovery from thrashing
* In the presence of increasing page fault frequency and decreasing CPU utilisation
	* Suspend a few process to reduce degree of multiprogramming
	* Resident pages of suspended processes will migrate to backing store
	* More physical memory becomes available
		* Less faults, faster progress for runnable processes
	* Resume suspended processes later when memory pressure eases

# VM Management policies
* Operations and performance of VM system is dependent on a number of policies:
	* Page table format (may be dictated by hardware)
		* Multi-level
		* Inverted/Hashed
	* Page size (may be dictated by hardware)
	* Fetch policy
	* Replacement policy
	* Resident set size
		* Minimum allocation
		* Local versus global allocation
	* Page cleaning policy

### Page size
Increasing page size:
* Decreases internal fragmentation
	* Reduces adaptability to working set size
* Decreases number of pages
	* Reduces size of page tables
* Increases TLB coverage
	* Reduces number of TLB misses
* Decreases page fault latency
	* Need to read more from disk before restarting process
* Increases swapping I/O throughput
	* Small I/O are dominated by seek/rotation delays
* Optimal page size is a (work-load dependent) tradeoff

* Multiple page sizes provide flexibity to optimise the use of TLB
* Example:
	* Large page sizes can be used for code
	* Small page size for thread stacks
* Most operating systems support only a single page size
	* Dealing with multiple page sizes is hard!

### Fetch Policy
* Determines *when* a page should be brought into memory
	* *Demand paging* only loads pages in response to page faults
		* Many page faults when a process first starts
	* *Pre-paging* brings in more pages than needed at the moment
		* Improves I/O performance by reading in larger chunks 
		* Pre-fetch when disk is idle
		* Wastes I/O bandwidth if pre-fetched pages aren't used
		* Especially bad if we eject pages in working set in order to pre-fetch unused pages
		* Hard to get right in practice

### Replacement policy
* Which page is chosen to be tossed out?
	* Page removed should be the page least likely to be references in the near future
	* Most policies attempt to predict the future behaviour on the basis of past behaviour
* Contraint: locked frames
	* Kernel code
	* Main kernel data structure
	* I/O buffers
	* Performance-critical user-pages (DBMS)
* Frame table has a lock (or *pinned*) bit

### *Optimal* replacement policy
* Tossed the page that won't be used for longest time
* Impossible to implement
* Only good as theoric reference point:
	* The closer a practical algorithm gets *optimal*, the better

#### FIFO replacement policy
* First in first out; toss the oldest page
	* Easy to implement
	* Age of a page is isn't necessarily related to useage

#### LRU replacement policy
* *Least Recently Used*; Toss the least recently used page
	* Assume that page that has not been referenced for a long time is unlikely to be referenced in the near future
	* Will work if locality holds
	* Implementation requires a timestamp to be kept for each page, updated on *every reference*
	* Impossible to implement efficiently
	* Most practical algorithms are approximations of LRU

#### Clock Page replacement policy
* Clock policy; also called *second change*
	* Employs a usage or reference bit in the frame table
	* Set to *one* when page is used
	* While scanning for a victim, reset all reference bits
	* Toss the first page with zero reference bit

#### Issue
* How do we know when a page is referenced?
* Use the valid bit in the PTE:
	* When a page is mapped (valid bit set), set the reference bit
	* When resetting the reference bit, invalidate the PTE entry
	* On page fault
		* Turn on valid bit in PTE
		* Turn on reference but
* We thus simulate a reference bit in software

#### Performance
* In terms of selecting the most appropriate replacement, they rank as follows:
	1. Optimal
	2. LRU
	3. Clock
	4. FIFO
> There are other algorithms (Working set, WSclock, Ageing, NFU, NRU)

#### Resident Set Size
* How many frames should each process have?
	* Fixed Allocation
		* Gives a process fixed number of pages within which to execute
		* Isolates process memory useage from each other
		* When a page fault occurs, one of the pages of that process must be replaced
		* Achieving high utilisation is an issue
			* Some processes have high fault rate while others don't use theor allocation
	* Variable allocation
		* Number of pages allocated to a process varies over the lifetime of a process

##### Variable allocation
* Global scope
	* Easy to implement
	* Adopted by many Operating Systems
	* Operating System keeps global list of free frames
	* Free frame is added to resident set of process when a page fault occurs
	* If no free frame, replace one from any process
	* Pro/Cons
		* Automatic balancing across system
		* Does not provide guarantees for important activities
* Local scope
	* Allocate number of page frames to a new process based on
		* Application type
		* Program request
		* Other criteria (Priority)
	* When a page fault occurs, select a page from among the resident set of the process that suffers the page fault
	* *Re-evaluate allocation from time to time*

#### Page table frequency scheme
* Establish '*accptable*' page fault rate
	* Too low → process loses frame
	* Too high → process gains frame

### Cleaning Policy
* Observation
	* Clean pages are much cheaper to replace than dirty pages
* Demand cleaning
	* A page is written out only when it has been selected for replacement
	* High latency between the decision to replace and availability of free frame
* Precleaing
	* Pages are written out in batches (in the background, the *pagedaemon*)
	* Increases likelihood of replacing clean frames
	* Overlap I/O with current activity

# Checkpoint
1. What effect does increasing the page size have?
	* Increases potential internal fragmentation, and the unit of allocation is now greater, hence greater potential for unused memory when allocation is rounded up/ Practically speaking, this is not an issue usually, and round up an application to the next page boundary usually results in a relative small amount wasted compared to the application size itself.
	* Decreases the number of pages. For a given amount of memory, larger pages reult in fewer page table entries, thus smaller (and potentially faster lookup) page table. 
	* Increases TLB coverage as the TLB has a fixed number of entries, thus larger entries cover more memory, reducing miss rate
	* Increases page fault latency as a page fault must load the whole page into memory before allowing the process to continue. Large pages have to wait for a longer loading time per fault
	Increases swapping I/O throughput. Given a certain amount of memory, larger pages have higher I/O throughput as the content of a page is stored contiguously on disk, giving sequential access
	* Increases the working set size. Work set is defined as the size of memory associates with all pages accessed within a window of time. With large pages, the more potential for pages to include significant amount of unused data, thus working set size generally increases with page size.
2. Why is demand paging more prevalent than pre-paging
	* Prepaging requires predicting the future (hard) and the penalty for making a mistake may be expensive (may page out a needed page for unneeded page) 
3. Describe four replacement policies and compare them
	* Optimal
		* Toss the page that wont be used for the longest time
		* Impossible to implement
		* Only good as a theoretic point: the closer a practival algorithm gets to optimal, the better
	* FIFO
		* First in, first out: toss the oldest page
		* Easy to implement
		* Age of page is not necessarily related to usage
	* LRU
		* Toss the least recently used page
		* Assumes that page that has not been referenced for a long time is unlikely to be referenced in the near future
		* Will work if locality holds
		* Implementation requires a timestamp to be kept for each page, updated on every reference
		* Impossible ti implement efficiently
	* Clock
		* Employs a usage or reference bit in the frame table
		* Set to one when page is used
		* While scanning for a victim, reset all the reference bits
		* Toss the first page with a zero reference bit
4. What is thrashing? How can it be detected? What can be done to combat it?
	* Thrashing is where the sum of the working set sizes of all processes exceeds available physical memory and the computer spends more and more time waiting for pages to transfer in and out
	* It can be detected by monitoring pagefault frequency
	* Some proecesses can be suspended and swapped out to relieve pressure on memory


# Multiprocessor System
* Shared memory multiprocessors
	* More than one processor sharing same memory
* A single CPU can only go so fast
	* Use more CPU than one CPU to improve performance
	* Assumes
		* Workload can be parallelised
		* Workload is not I/O-bound or memory-bound
* Disk and other hardware can be expensive
	* Can share hardware between CPUs

##### Amdahl's Law
* Given a proportion *P* of a program that can be made parallel, and the remaining serial portion *(1 - P)*, speedup by using *N* processors
######<div align="center">1 / ((1 - P) + P / N)</div>

### Types of Multiprocessors (MPs)
* UMA MP
	* Uniform Memory Access
		* Access to all memory occurs at the same speed for all processors
* NUMA MP
	* Non-uniform memory access
		* Access to some parts of memory is faster for some processors than other parts of memory

### Bus Based UMA
* Simplest MP is more than one processor on a single bus connect to memory
	* Bus bandwidth becomes bottleneck with more than just a few CPUs
* Each processor has a cache to reduce its need for access to memory
	* Hope is most accesses are to the local cache
	* Bus bandwidth still becomes a bottleneck with many CPUs
* **Cache Consistency**
	* What happens if one CPU writes to address 0x1234 (and its stored in its cache) and another CPU reads from the same address (and gets whats in its cache)
	* Cache consistency is usually handled by the hardware
		* Writes to one cache propagate to, or invalidate appropriate entries
		on other caches
		* Cache transactions also consume bus bandwidth
* With only a single shared bus, scalability can be limited by the bus bandwidth of the single bus
	* Caching only helps so much
* Alternative bus architectures exist
	* They improve bandwidth available
	* Don't eliminate constraint that bandwidth is limited

**Summary**
* Multiprocessors can
	* Increase computation power beyond that available from a single CPU
	* Share resources such as disk and memory
* However,
	* Assumes parallelizable workload to be effective
	* Assumes not I/O bound
	* Shared buses (bus bandwidth) limits scalability
		* Can be reduced via hardware design
		* Can be reduced by carefully created software behaviour
			* Good cache locality together with limited data sharing where possible

### Each CPU has its own OS?
* Statistically allocate physical memory to each CPU
* Each CPU run its own independent OS
* Share peripherals
* Each CPU (OS) handles its processes system calls

### Each CPU has its own OS
* Used in early multiprocessor systems to *get them going*
	* Simpler to implement
	* Avoids CPU-based concurrency issues by not sharing
	* Scaled - no shared serial sections
	* Modern analogy, virtualisation in the cloud

**Issues**
* Each processor has its own scheduling queue
	* We can have one processor overloaded, and the rest idle
* Each processor has its own memory partition
	* We can a one processor thrashing, and the others with free memory
		* No way to move free memory from one OS to another

### Symmetric Multiprocessors (SMP)
* OS kernel run on all processors
	* Load and resource are balance between all processors
		* Including kernel execution
* Issue: *Real* concurrency in the kernel
	* Need carefully applied synchronisation primitives to avoid disaster
* One alternative: A single mutex that makes the entire kernel a large critical section
	* Only one CPU can be in the kernel at a time
	* The *big lock* becomes a bottleneck when in-kernel processing exceeds what can be done on a single CPU
* Better alternative: Identify largely independent parts of the kernel and make each of them their own critical section
	* Allows more parallelism in the kernel
* Issue: Difficult task
	* Code is mostly similar to uniprocessor code
	* Hard part is identifying independent parts that don't interfere with each other
		* Remember all the inter-dependencies between OS subsystems
* Example:
	* Associate a mutex with independent parts of the kernel
	* Some kernel activities require more than one part of the kernel
		* Need to acquire more than one mutex
		* Great opportunity to deadlock
	* Results in potentially complex lock ordering schemes that must be adhered to

	* Given a *big lock* kernel, divide the kernel into two independent parts with a lock each
		* Good change that one of those locks will become the next bottleneck
		* Leads to more subdivision, more locks, more complex lock acquisition rules
			* Subsivision in practice is (in reality) making more code multithreaded (parallelised)

### Multiprocessor Synchronisation
* Given we need sysnchronisation, how can we achieve it on multiprocessor machine?
	* Unlike a uniprocessor, disdabling interrupts does not work
		* It does not prevent other CPUs from running in parallel
	* Need special hardware support

**Recall mutual exclusion with test-and-set** <br />
Entering and leaving a critical region using the TSL instruction
```c
enter_region:
    TSL REGISTER, LOCK // Copy lock to register and set lock to 1
    CMP REGISTER, #0 // Was lock zero?
    JNE enter_region // If it was non-zero, lock was set, so loop
    RET // return to caller; critical region entered

leave_region:
    MOVE LOCK, #0 // store a 0 in lock
    RET // return to caller
```
**Test and Set** <br />
* Hardware guarantees that the instruction executes atomically on a CPU
	* Atomically: as an indivisible unit
	* The instruction cannot stop half way through
* It does not work without some extra hardware support
* A solution:
	* Hardware blocks all other CPUs from accessing the bus during the TSL instruction to prevent memory accesses by any other CPU
		* TSL has mutually exclusive access to memory for duration of the instruction

**Test and Set on SMP** <br />
* Test-and-set is a busy-wait synchronisation primitive
	* Called a **spinlock**
* Issue:
	* Lock contention leads to spinning on the lock
		* Spinning on a lock requires blocking the bus which slows all other CPUs down
			* Independent of whether other CPUs need a lock or not
			* Causes bus contention
* Caching does not help reduce bus contention
	* Either TSL still blocks the bus
	* Or TSL requires exclusive access to an entry in the local cache
		* Requires invalidation of same entry in other caches, and loading entry into local machine
		* Many CPUs performing TSL simply bounce a single exclusive entry between all caches using the bus

### Reducing Bus Contention
* Read before TSL
	* Spin reading the lock variable waiting for it to change
	* When it does, use TSL to acquire the lock
* Allows lock to be shared read-only in all caches until its released
	* No bus traffic until actual release
* No race conditions, as acquisition is still with TSL
```c
start:
	while(lock == 1);
	r = TSL(lock);
	if(r == 1)
		goto start;
```

### Benchmark
```c
for i = 1 ... 1,000,000{
	lock(l);
	critical_section();
	unlock();
	compute();
}
```
* Compute chosen from uniform random distribution of mean 5 times critical section
* Measure elapsed time on sequent symmetry(20 CPU 30386, coherent white-back invalidate caches)

### Results
* Test andSet performs poorly once there is enough CPUs to cause contention for lock
	* Expected
* Read before test and set perofrms better
	* Performance less than expected
	* Still significant contention on lock when CPUs notice release and all attempt acquisition
* Critical section performance degenerates
	* Critical section requires bus traffic to modify shared structure
	* Lock holder competes with CPU that's waiting as they test and set, so the lock holder is slower
	* Slower lock holder results in more contention

### Spinning VS Blocking and Switch
* Spinning (busy-waiting) on a lock makes no sense on a uniprocessor
	* The ways no other running process to release lock
	* Blocking and (eventually) switching to the lock holder is the only sensible option
* On SMP systems, the decision to spin or block is not as clear
	* The lock is held by another running CPU and will be freed without necessarili switching away from the requestor

### Spinning VS Switching
* Blocking and switching
	* to another process takes time
		* Save context and restore another
		* Cache contains current process; not new process
			* Adjusting the cache working set also takes time
		* TLB is similar to cache
	* Switching back when the lock is free encounters the same again
* Spinning wastes CPU time directly
* Trade off
	* If lock is held for less time than the the overhead of switching to and back
		* More efficient to spin
	* Spinlocks expect critical sections to be short
		* No waiting for I/O within a spinlock
		* No nesting locks within a spinlock

### Preemption and spinlock
* Critical sections synchronised via spinlocks are expected to be short
	* Avoid other CPUs wastinf cycles spinning
* What heppens if the spinlock holder is preempted at end of holder's timeslice
	* Mutualn exclusion is still guaranteed
	* Other CPUs will spin unitl the holder is scheduled again
* Spinlock implementations disable interrupts in addition to acquiring locks to avoid lock-holder preemption

# Checkpoint
1. What are the advantages and disadvantages of using a global scheduling queue over per-CPU queues? Under which circumstances would you use the one or the other? What features of a system would influence this decision?
	* Global queue is simple and provides automatic load balancing, but it can suffer from contention on the global scheduling queue in the presence of many scheduling events or many CPUs or both. Another disadvantage is that all CPUs are treated equally, so it does not take advantage of hot caches present on the CPU the process was last scheduled on.
	* Per-CPU queues provide CPU affinity, avoid contention on the scheduling queue, however they require more complex schemes to implement load balancing.
2. When does spinning on a lock (busy waiting, as opposed to blocking on the lock, and being woken up when it's free) make sense in a multiprocessor environment?
	* Spinning makes sense when the average spin time spent spinning is less than the time spent blocking the lock requester, switching to another process, switching back, and unblocking the lock requester.
3. Why is preemption an issue with spinlocks?
	* Spinning wastes CPU time and indirectly consumes bus bandwidth. When acquiring a lock, an overall system design should minimise time spent spinning, which implies minimising the time a lock holder holds the lock. Preemption of the lock holder extends lock holding time across potentially many time slices, increasing the spin time of lock aquirers.
4. How does a read-before-test-and-set lock work and why does it improve scalability?
	* See the lecture notes for code. It improves scalability as the spinning is on a read instruction (not a test-and-set instruction), which allows the spinning to occur on a local-to-the-CPU cached copy of the lock's memory location. Only when the lock changes state is cache coherency traffic required across the bus as a result of invalidating the local copy of the lock's memory location, and the test-and-set instruction which require exclusive access to the memory across all CPUs.



# Scheduling
* On a multi-programmed system
	* We may have more than one *Ready* process
* On a batch system
	* We may have many jobs waiting to be run
* On a multi-user system
	* We may have many users concurrently using the system
* The ***Scheduler*** decides who to run next
	* The process of choosing is called *scheduling*

* Importance of Scheduling
	* Is not important on certain scenarios:
		* Early systems
			* Usually batching
			* Scheduling algorithm simple
				* Run next on tape or next on punch tape
		* Only one thing to run
			* Simple PCs
				* Only ran on a word processor, etc
			* Simple Embedded Sysytems
				* TV remote control, washing machine, etc
	* In recent scenarios:
		* Multitasking/Multiuser syste,
			* E.g. Email daemon takes 2 seconds to process an email
			* User takes button on application
				* Scenario 1:
					* Run daemon, then application
					* Systen appears really sluggish to user
				* Scenario 2:
					* Run application, then daemon
					* Application appears really responsive, small email delay is unnoticed
	* Scheduling decisions can have dramatic effect on the perceived performance of the system
		* Can also affect correctness of a system with deadlines

### Application behaviour
* Bursts of CPU usage alternate with periods of I/O wait
	1. CPU-Bound process
		* Spends most of its computing
		* Time to completion largely determined by received CPU time
	2. I/O-Bound process
		* Spend most of its time waiting for I/O to complete
			* Small bursts of CPU to process I/O and request next I/O
		* Time to completion largely determined by I/O request time
	* **Obeservation**
		* We need a mix of CPU-bound and I/O-bound processes to keep both CPU and I/O systems busy
		* Process can go from CPU to I/O bound (or vise-versa) in different phases of execution
	* **Key insights**
		* Choosing to run an I/O-bound process delays a CPU-bound process by very little
		* Choosing to run a CPU-bound process prior to an I/O-bound process delays the next I/O request significantly
			* No overlap of I/O waiting with computation
			* Results in device (disk) not as busy as possible
		* Generally, favour I/O-bound processes over CPU-bound processes

### When is scheduling performed?
* A new process
	* Run the parent or child?
* A process exits
	* Who runs next?
* A process waits for I/O
	* Who runs next?
* A process blocks on a lock
	* Who runs next? The lock holder?
* Al I/O interrupt occurs
	* Who do we resume, the interrupt process or the process that was waiting?
* On timer interrupt?
* Generally, a scheduling decision is required when a process (or thread) can no longer continue, or when an activity results in more than one ready process

### Preemptive vs Non-preemptive scheduling
* Preemptive Scheduling
	* Current thread can be interrupted by OS and moved to *ready* state
	* Usually after a timer interrupt and process has exceeded its maximum run time
		* Can also be as a result of higher priority
		process that has become ready (after I/O interrupt)
	* Ensure fairer service as single thread can't monomolise the system
		* Requires a timer interrupt

### Categories of Scheduling Algorithms
* The choice of scheduling algorithm depends on the goals of the application (or the operating system)
	* No one algorithm suits all environment
* Batch systems
	* No users directly waiting, can optimise for overall machine performance
* Interactive systems
	* Users directly waiting for their results, can optimise for users perceived performance
	* Goals:
		* Minimalise *response* time
			* Response time is the time difference between issuing a command and getting the result
				* *e.g* selecting a menu, and getting the result of that selection
			* Response time is important to the user's perception of the performance of the system
		* Provide *Proportionality*
			* Proportionality is the user exception that short jobs will have a short response time, and long jobs can have a long response time
			* Generally, favour short jobs
* Realtime Systems
	* Jobs have deadlines, must schedule such that all jobs (predictably) meet their deadlines
	* Goals:
		* Must meet deadlines
			* Each job/task has a deadline
			* A missed deadline can result to data loss or catastrpopic failure
				* Aircraft control system missed deadline to apply breaks
		* Provide predictability
			* For some apps, an occasional missed deadline is OK
				* *e.g.* DVD decoder
			* Predictable behaviour allows smooth DVD decoding with only rare skips
**Goals of sceduling algorithms**<br />
* Fairness
	* Give each process a *fair* share of the CPU
* Policy Enforcement
	* What ever policy chosen, the scheduler should ensure it is carried out
* Balance/Efficiency
	* Try to keep all parts of the system busy

# Interactive Scheduling
### Round Robin Scheduling
* Each process is given a *timeslice* to run in
* When the timeslice expires, the next process preempts the current process, and runs for its timeslice and so on
	* The preempted process is placed at the end of the queue
* Implemented with
	* Ready queue
	* Regular timer interrupt
* Pros/Cons
	* Fair, easy to implement
	* Assumes everybody is *equal*
* Issue: *What should the timeslice be?*
	* Too short
		* Waste a lot of time switching between processes
		* *e.g.* time slice of 4 *ms* with 1 *ms* context switch → 20% round robin overhead
	* Too long
		* System is not responsive
		* *e.g.* timeslice of 100ms
			* If 10 people hit enter key simultaneously, the last guy to run will onpy see progress after 1 second
		* Degenerates into FCFS if timeslice longer than burst length

### Priorities
* Each process (or thread) is associated with priority
* Provides basic mechanism to influence a scheduler decision
	* Scheduler will always choose a thread of higher priority over lower priority
* Priorities can be defined internally or externally
	* Internal: e.g. I/O bound or CPU bound
	* External: e.g. Based on importance to the user
* Usually implemented by multiple priority queues, with round robin on each queue
* Con
	* Low priorities can starve
		* Need to adapt priorities periodically
			* Based on ageing or execution history

### Traditional UNIX Scheduler
* Two level scheduler
	* High level scheduler schedules processes between memory and disk
	* Low level scheduler is CPU scheduler
		* Based on multi level queue structure with round robin at each level
* The highest priority (lower number) is scheduled
* Priorities are re-calculated once per second, and re-inserted in appropriate queue
	* Avoids starvation of low priority threads
	* Penalise CPU-bound threads

**Priority = CPU_Usage + *Nice* + base** <br />
* CPU usage → Number of clock ticks 
	* Decays over time to avoid permanently penalisingthe process
* Nice → value given to the process by a user to permenently boost or reduce its priority
	* Reduce priority of background jobs
* Base → set of hardwired, negative values to boost priority of I/O bound system activities
	* Swapper, disk I/O, Character I/O

### Multiprocessor Scheduling
* *Given X processes (or threads), and Y CPUs, how do we allocate them to the CPUs?*
* Single shared only ready queue
	* When a CPU goes idle, it takes the highest priority process from the shared ready queue
	* Pros
		* Simple
		* Automatic balancing
	* Cons
		* Lock contention on the ready queue can be a major bottleneck
			* Due to frequent scheduling or may CPUs or both
		* Not all CPUs are equal
			* The last CPU a process ran on is likely to have more related entries in the cache
* Affinity Scheduling
	* Try hard to run a process on the CPU it ran on last time
	* One approach: *Multiple queue multiprocessor scheduling*
	* **Multiple Queue SMP Scheduling**
		* Each CPU has its own ready queue
		* Coarse-grained algorithm assigns processes to CPUs
			* Defines their affinity and roughly balances the load
		* The bottom-level fine-grained scheduler
			* Is the frequent invoked scheduler (e.g. on blocking I/O, a lock or exhausting a timeslice)
			* Runs on each CPU and selects from its own ready queue
				* Ensures affinity
			* If nothing is available from the local ready queue, it runs a process from another CPUs ready queue rather than go idle
				* Termed *work stealing*
		* Pros
			* No lock contention on per-CPU ready queues in the common case
			* Load balancing to avoid idle queues
			* Automatic affinity to single CPU for more cache friendly behaviour


# I/O Management

## I/O Devices
* There exists a large variety of I/O devices
	* Many of them with different properties
	* They seem to require different interfaces to manipulate and manage them
		* We dont want a new interface for every device
		* Diverse, but similar interfaces lead to code duplication
	* Challenge:
		* Uniform and efficient approach to I/O

### Device drivers
* Drivers (originally) compiled into kernel
	* Including OS/161
	* Device installers were technicians
	* Number and types of devices rarely changed
* Nowadays they are dynamically loaded when needed
	* Linux modules
	* Typical users (device installers) cant build kernels
	* Number and types vary greatly
		* Even while OS is running (e.g. hot-plug USB devices)
* Drivers classified into similar categories
	* Block devices and character (stream of data) device
* OS defines a standard (internal) interface to the different classes of devices
	* Example: USB *Human Input Device* (HID) class specifications
		* Human input device follow a set of rules making if easier to design a standard interface

#### I/O Device handling
* Data rate
	* May be differences of several orders of magnitude between the data transfer rates
	* Example: Assume 1000 cycles/byte I/O
		* Keyboard needs 10KHz processor to keep up
		* Gigabit Ethernet needs 100GHz processor

#### Device Drivers
* Device drivers job
	* Translate request through the device-independent standard interface (open, close, read, write) into appropriate sequence of commands (register manipulations) for the particular hardware
	* Initilise the hardware at boot time, and shut it down cleanly at shutdown
* After issuing the command to the device, the device wither
	* Completes immediately and the driver simply returns to the caller
	* or device must process the request and the driver usually blocks wawiting for an I/O complete interrupt
* Drivers are thread-safe as they can be called by another process while a process is already blocked in the driver
	* Thread-safe: Synchronised

#### Device-Independent I/O Software
* There is commonality between drivers of similar classes
* Divide I/O software into device-dependent and device-independent I/O software
* Device independent software includes
	* Buffler or buffer-cache management
	* TCP/IP stack
	* Managing access to dedicated devices
	* Error reporting

#### Driver <=> Kernel Interface
* Major issue is uniform interfaces to devices and kernel
	* Uniform device interface for kernel code
		* Allows different devices to be used the same way
			* No need to rewrite file-system to switch between SCSI, IDE or RAM disk
		* Allows internal changes to device driver with fear of breaking kernel code
	* Uniform kernel interface for device code
		* Drivers use a defined interface kernel services (e.g. kmalloc, install IRQ handler, etc)
		* Allows kernel to evolve without breaking existing drivers
	* Together both unforim interfaces avoid alot of programming implementing new interface
		* Retains compatibility as drivers and kernel change over time

#### Accessing I/O Controllers
* Separate I/O and memory space
	* I/O Controller registers appear as I/O ports
	* Accessed with special I/O instructions
* Memory-mapped I/O
	* Controller registers appear as memory
	* Use normal load/store instructions to access
* Hybrid
	* x86 has both ports and memory mapped I/O

#### Interrupts
* Devices connected to an *Interrupt controller* via lines on an I/O bus (e.g. PCI)
* Interrupt controller signals interrupt to CPU and is eventually acknowledged
* Exact details are architecture specific

# I/O Management Software
* Operating System design issues
	* Efficiency
		* Most I/O devices slow compared to main memory (and CPU)
			* Use of multiprogramming allows for some processes to be waiting on I/O while other process executes
			* Often I/O still cannot keep up with processor speed
			* Swapping may used to bring in additional ready processes
				* More I/O Operations
	* **Optimise I/O efficiency -- especially disk & network I/O**
	* The quest for generality/uniformity
		* Ideally, handle all I/O devices in the same way
			* Both in the OS and in user applications
		* Problem
			* Diversity of I/O devices
			* Especially, different access methods (random access vs stream based) as well as vastly different data rates
			* Generality often compromises efficiency
		* Hide most of the details of device I/O in lower-level routines so that processes and upper levels see devices in general terms such as read, write, open, close.

### I/O Software Layers
1. User-level I/O software
2. Device-independent operating system software
3. Device drivers
4. Interrupt handlers
	* Can execute at (almost) anytime
		* Raise complex concurrency issues in kernel
		* Can propagate to userspace (signals, upcalls), causing similar issues
		* Generally structured so I/O operations block until interrupts notify them of completion
	1. **Save registers** not already saved by hardware interrupt mechanism
	2. (optionally) set up context for interrupt service procedure
		* Typically runs in the context of the currently running process
			* No expensive context switch
	3. **Set up stack** for interrupt service procedure
		* Handler usually runs on kernel stack of current process
		* Or *nests* if already in kernel more running on kernel stack
	4. **Ack/Mask interrupt controler**, re-enable other interrupts
		* Implies potential for interrupt nesting
	5. Run interrupt service procedure
		* Acknowledges interrupt at device level
		* Figures out what caused the interrupt
			* Received a network packet, disk read finished, UART transmit queue empty
		* If needed, it signals blocked device driver
	6. In some cases, will have woken up a higher priority blocked thread
		* Choose newly worken thread to schedule next
		* Set up MMU context for process to run next
		* What if we are nested?
	7. Load new/orignialm process' registers
	8. Re-enable interrupt; start running the new process
5. Hardware

#### Sleeping in interupts
* An interrupt generally has no context (runs on current kernel stack)
	* Unfair to sleep on interrupt process (deadlock possible)
	* Where to get context for long running operation?
	* What foes into the ready queue?
* What to do?
	* Top and bottom half
		* Top half
			* Interrupt handler
			* Remains short
		* Bottom half
			* Is preemptable by top half (interrupts)
			* Performs deferred work (e.g. IP stack processing)
			* Is checked prior to every kernel exit
			* Signals blocked processes/threads to continue
		* Enables low interrupt latency
		* Bottom half can't block
		* Deferring work on in-kernel threads
			* Interrupt
				* Handler defers work onto in-kernel thread
			* In-kernel thread handles deferred work 
				* Scheduled normally
				* Can block
			* Both low interrupt latency and block operations
	* Linux implements with *tasklets* and *workqueues*
	* Generically, in-kernel thread(s) handle long running kernel operations 

# Buffering
### Device-independent I/O software
* Unbuffered input
* Buffering in user space
* *Single buffering* in the kernel followed by copying to user space
* Double buffering in the kernel

### No buffering
* Process must read/write a device byte/word at a time
	* Each individual system call adds significant overhead
	* Process must wait until each I/O is complete
		* Blocking/interrupt/waking adds to overhead
		* Many short runs of a process is inefficient (poor CPU cache temporal locality)

### User-level buffering
* Process specifies a memory *buffer* that incoming data is placed in until it fills
	* Filling can be done by interrupt service routine
	* Only a single system call, and block/wakeup per data buffer
		* Much more efficient
* Issues
	* What happens if buffer is paged out to disk
		* Could lose data while unavailable buffer is paged in
		* Could lock buffer in memory (needed for DMA), however many processes doing I/O reduce RAM available for paging.
			* Can cause deadlock as RAM is limited resource
	* Consider write case
		* When is buffer available for re-use?
			* Either process must blcok until potential slow device drains buffer
			* Or deal with asynchronous signals indicating buffer drained

### Single buffer
* Operating system assigns a buffer in kernel's memory for an I/O request
* IN a stream-oriented scenario
	* Used a line at time
	* User input from a terminal is one line at a time with carriage return signaling the end of the line
	* Output to the terminal is one line at a time
* Block-oriented
	* Input transfers made to buffer
	* Block copied to user space when needed
	* Another block is written into the buffer
		* Read ahead
* User process can process one block of fata while next block is read in
* Swapping can occur since input is taking place in system memory, not user memory
* Operating system keeps track of assignmnet of system buffers to user processes
* **Single buffer speed up**
	* Assume
		* *T → transfer time for a block from device*
		* *C → computation time to process incoming block*
		* *M → time to copy kernel buffer to user buffer*
	* Computation and transfer can be done in parallel
	* Speed up with buffering
	###### <div align="center">*(T+C)/(max(T,C) + M)*
	</div>
* What happens if kernel buffer is full?
	* user buffer is swapped out or,
	* The application is slow to process previous buffer
	* and more data is received???
	* → we start to lose characters or drop network packets

### Double buffer
* Use two system buffers instead of one
* A process can transfer data to or from one buffer while the operating system empties or fills the other buffer
* **Double buffer speed up**
	* Computation and memory copy can be done i n parallel with transfer
	* Speed up with double buffering
	###### <div align="center">*(T+C)/max(T,C + M)* </div>
	* Ususally M is much more less than T giving a favourable result
	 * May be insufficient for really bursty traffic
	 	* Lots of application writes between long periods of computation
	 	* Long periods of application computation while receiving data
	 	* Might want to read-ahead more than a single block for disk

### Circular buffer
* More than two buffers are used
* Each individual buffer is one unit in a circular buffer
* Used when I/O operation must keep up with process


> Buffering, double buffering, and circular buffering are all **Bounded-Buffer Producer-Consumer problems**

# Sample exam 1
* A smaller page size leads to smaller page tables
	* False – need more entries because we have more pages
* A smaller page size leads to more TLB misses
	* True – less likely that page will encompass address we are after.
* A smaller page size leads to fewer page faults 
	* True – In practice, initial faulting-in is usually less important than page faults generated once the application is up and running. Once the app is up and running, reducing the page size results in a more accurate determination of workset, which is in turn smaller, which requires less memory, which in turn leads to lower page fault rate.
* Smaller page size reduces paging I/O throughput 
	* True 
* Threads are cheaper to create than processes
	* True
* Kernel implemented threads are cheaper to create than user level implemented threads
	* False
* A blocking kernel implemented thread blocks all threads in the process
	* False –  True for user level threads
* Threads are cheaper to context switch than processes
	* True – Dont have to save address space
* A blocking user level implemented thread blocks the process
	* True
* Different user level threads of the same process can have different scheduling priorities in the kernel
	* False – User level threads are not scheduled; they *yield()* the CPU
* All kernel scheduled threads of a process share the same virtual address space
	* True
* The *optimal* page replacement is best choice in practice
	* False – imposisble to implement
* The OS is not responsible for resource allocation between competing processes
	* False – is responsible
* System calls do not change privilege mode of the processor
	* False – We trap into the kernel so we do change the privilege mode
* A scheduler favouring I/O bound processes ususally does not significantly delay the completion of CPU bound processes
	* True – the I/O processes get the I/O and then yield CPU again

* Consider a demand-paging system with a paging disk that has an average access and transfer time of 5 milliseconds for a single page. Addresses are translated through a page table in main memory, with an access time of 100 nanoseconds per memory access. Thus, each memory reference through the page table takes two accesses. The system has a 48-entry TLB to speed up memory accesses. Assume that 99% of memory accesses result in a TLB hit, and of the remaining 1%, 5 percent (or 0.05% of the total) cause page faults. What is the effective memory access time?
	* 0.99 TLB hits, which needs only a single memory access
	* 0.0095 TLB miss, one read from memory to get the page table entry to load TLB, then one read to read actual memory
	* 0.0005 pagefault plus eventual read from memory plus something like one of the following:
		* Memory reference to update the page table
		* Memory reference to update the page table and memory reference for the hardware to re-read the same entry to refill the TLB
		* Any reasonable scenario that results in valid pagetable and refilled TLB
	* Assuming the second, <br />
	0.99 * 100 ns + 0.0095 * 2 * 100 ns + 0.0005(3 * 100ns + 5ms)

# Sample exam 2
* The role of the operating system is to present all the low level details of a computer hardware without any abstraction
	* False – The role of the OS is to abstract low level details
* The operating system runs in privileged mode of the microprocessor
	* True – If the processor features a privileged mode, them the operating system will use it
* An application running in user mode shares teh same stack with the operating system when it runs in kernel mode
	* False – kernel has its own stack
* An operating system can enforce security by putting checks in standard C library
	* False – Operating system is language independent and has no control over the C library which is user code
* Arguments to system calls are placed with registers (or on the stack) based on a mutually defined convention between the OS and user level application
	* True – otherwise system calls wouldnt work
* In the three-state process model, a common transition is from *ready to blocked*
	* False – A process can go from blocked to ready but not the other way; it needs to run to be blocked
* Co-operative(non preemptive) multitasking can result in a nonresponsive system if an application has an endless loop
	* True – In co-operative multitasking, a thread must *yield()* in order for anotehr thread to take over
* A Threading library implementation at user level does not expose the concurrency available in the application to toe operating system
	* True – Kernel does not know about user level threads
* Application threading supported by kernel implemented threads (i.e. kernel level threads) can take advantage of multiple processors if available
	* True – OS can schedule kernel level threads
* Applications can synchronise to avoid race condtions by disabling and enabling interrupts
	* False – user level applications cannot disable interrupts as this requires kernel mode
* Semaphores can be used to implement mutual exclusion primitives
	* True – if initialised to one
* Condition variables are used together with semaphores to manage blocking and waking within the semaphore
	* False – Condition variables are used in conjuction with *monitors* or monitor like lock use
* Bankers algorithm can avoid deadlock if the maximum resource requirements of thread are known in advance
	* True – Although its not realistic in pracice
* Hold and wait is a practical deadlock prevention strategy
	* False – hold and wait is one of the preconditions that lead to deadlock
* Sparse files save disk space by not storing the parts of the file that are not written to
	* True – metadata representing the empty blocks can be stored instead
* The buffer cache improves write performance by buffering writes. Without extra application effort, the performance increase comes at the expense of reliability and consistency in the presence of failures
	* True – even if the FS remains in consistent (or recoverable) state, data is still lost
* Contiguous file allocation is an allocation strategy suitable for read only media
	* True – if the file increases in size, reallocation will be required
* Chained (linked list) file allocation is desireable for file systems that support random access workloads
	* False – Random access in linked list is slow
* Best fit memory allocation gives the best result in terms of memory fragmentation
	* True – best fit results in least external fragmentation
* Swapping allows applications larger than physical memory to execute
	* False – swapping *swaps* the entire application, so it must fit entirely in memory to be present. (swapping vs paging https://stackoverflow.com/questions/4415254/difference-swapping-and-paging)
* Overlays allow applications larger than physical memory to execute
	* True – The application controls the overlays
* With appropriate language and OS support, segmentation can be used for bounds checking array access
	* True – The OS checks that memory access falls within segments boundaries
* Virtual memory thrashing is where virtual memory translation happens too fast
	* False – they are unrelated
* Optimal is the best practical page replacement algorithm
	* False – its impossible to implement
* When choosing a victim for page replacement, a clean page is faster to replace than a dirty page
	* True – it doesnt have to be written to disk
* Increasing the pagesize generally increases the working set size of an application
	* True – the larger granularity of inclusion results in more virtual memory being included in the working set when only part of the page is actually accessed
* Spatial locality contributes to VM system efficiency, but temporal does not
	* False – the TLB hitrate if entries are reused before they are replaced
* Adding more levels of I/O buffering always improves performance
	* False – buffering adds more memory copy overhead, which can be significant for fast I/O devices
* *Shortest seek time first* is a disk scheduling algorithm that is always fair
	* False – it can cause starvation
* Compared to polled I/O, interrupt driven I/O is preferrable when there is significant delay between a device request and the device response
	* True – otherwise CPU time is wasted polling
* Favoring CPU bound applications over I/O bound applications generally improves overall system performance 
	* False – A combination of I/O bound and CPU bound is optimal
* Rate monotonic scheduling can always schedule a task set if the total utilisation is less or equal to 1
	* False – but *earliest deadline first* does
* Round robin scheduling can be tuned by varying the time slice length to tradeoff responsiveness for decreased CPU overhead
	* True
* Spinlocks can never be more efficient to use than blocking locks, even on a multiprocessor machine
	* False – They can avoid the overhead of context switching to scheduler then another process on a multiprocessor
* On a multiprocessor machine, a single ready queue provides automatic load balancing between CPUs
	* True – Ready processes are variable to all CPUs
* A *read before test and set* implementation of a spinlock just adds extra overhead to the lock implementation by adding superfluous read
	* False – reduces bus traffic since the test & set instructions are only applied when they might succeed
* A multiprocessor machine only speeds up parallelisable workloads
	* True
* On a processor with hardware filled TLB, the OS is free to implement the most efficient page table data structure for the expected workload
	* False – the PT structure must be readable by the hardware refilling the TLB
* Memory compaction can be performed transparently to an application on a machine with *base* and *limit* registers 
	* True – The while segment can be copied higher or lower in physical memory
* Page sizes are always a power of 2
	* True
* Because the operating system runs in privileged mode, the operating system can safely use the memory pointers provided by applications (via the arguments to system calls) just like in-kernel memory pointers.
	* False
* Page-based virtual memory using a single page size is likely to suffer from external fragmentation in main memory.
	* False
* The size of an inverted page table is determined by the virtual address space size of a process, and is unrelated to the size of physical memory.
	* False
* Internal fragmentation is space wasted in main memory. External fragmentation is space wasted on disk.
	* False
* The minimum virtual-memory page size available for use by the operating system is determined by the properties of the underlying hardware architecture.
	* True
* It is possible to construct a time-sharing system able to withstand malicious applications on a computer architecture possessing only base and limit registers for memory management.
	* True

# OS CHEATSHEET

* Internal fragmentation: Space wasted due to space needed being smaller than space allocated.
* External fragmentation: Space wasted due to space needed being larger than space available between existing allocations.

Kernel mode | Both | User mode 
-----------| ---------| ------------
The application's page table | Application memory | x
A TLB management instruction | An add instruction | x
Hard disk drive controller registers |The application's memory containing the standard C library | x
An interrupt disabling instruction | The registers used to transfer system call arguments. | x

Bill and Ben are two threads that execute the functions of the same name below respectively. Bill and Ben are prone to deadlock. Identify the function calls in Bill and Ben where they end up deadlocked if they deadlock. 

Each line of code in each function has a line number to help you identify the location of deadlock. 
> Bill line 3, Ben line 6
```c
void bill() 
{
	1:    lock_acq(&file2)
	2:    lock_acq(&file3)
	3:    lock_acq(&file1)
	/* access all three files mutually exclusively */
	4:    lock_rel(&file3)
	5:    lock_rel(&file1)
	/* do nothing */
	6:    lock_acq(&file1)
	/* access file1 and file2 mutually exclusively */
	7:    lock_rel(&file1)
	8:    lock_rel(&file2)
}
void ben() 
{
	1:     lock_acq(&file2)
	2:     lock_acq(&file3)
	3:     lock_acq(&file1)
	/* access all three files mutually exclusively */
	4:    lock_rel(&file3)
	5:    lock_rel(&file2)
	/* do nothing */
	6:    lock_acq(&file3)
	/* write to file1 and file3 */
	7:    lock_rel(&file1)
	8:    lock_rel(&file3)
}
```
* Hold and Wait → Allocate all required resources initially
* No Preemption → Nearly always impractical to remove a resource
* Circular Wait → Use a single defined order of resource acquistion
* Mutual exclusion → Always infeasible


Per process | Per thread
------------|-----------
Open file descriptors |General purpose registers
Virtual address space |The execution state, e.g. blocked, ready, running
Global variables |Local variables
x |Stack 
x |Program counter 



## TRUE
* One deadlock prevention method is to release all locks held by a thread when the thread finds a lock that it requires is held by another thread. This method prevents deadlock, but not necessarily livelock.
* Sparse files save disk space by not storing the blocks of the file that are not written to.
* Using a bitmap to manage free blocks on a disk reduces the usable disk capacity compared to the approach of storing the addresses of free blocks in the free blocks themselves.
* Every access to an application virtual address is translated to a physical address using the information in a page table. The processor itself may cache a subset of the translation information in the translation look-aside buffer (TLB).
* In the user mode of the microprocessor, only the application's virtual memory is accessible to the processor.
* A buffer cache improves performance by buffering writes to disk, allowing the application to continue computation while buffers are written back to the disk in the background. This increases the chance of losing data during a power failure.
* It is possible to construct a time-sharing system able to withstand malicious applications on a computer architecture possessing only base and limit registers for memory management.
* Dining philosophers can avoid deadlock using a single lock for the entire table.
* A computer has a 32-bit virtual address space and uses a two-level page table. The top-level of the page table has 256 entries, and the second-level page tables have 1024 entries. That implies the page size on the computer must be 16 KiB.
* Compared to polled I/O, interrupt-driven I/O is preferable when there is a significant delay between requesting something from a device and the device providing the response.
* File systems based on contiguous allocation are suitable for read-only media such as optical disks (DVDs and CDs).
* A hypothetical i-node-based file system consists of i-nodes with only 12 direct addresses of blocks, and no addresses of indirect blocks. Each direct address is a 32-bit integer, and the block size is 2KiB. This file system only supports files with a maximum size of 24KiB.
* A file system using linked-list disk allocation (also termed chained allocation) provides poor random access performance.
* On a uniprocessor machine, it is more efficient to block and context switch than to spin (busy-wait) on a lock, even if the context switching overhead is very high.
* When choosing a victim for page replacement, a clean page is faster to replace than a dirty page.
* A page table implemented as a simple array in physical memory that is indexed by page number would provide fast access to page table entries with a single memory reference. However, it is impractical and inefficient to allocate such an array for each virtual address space.
* An operating system provides a high-level programming interface to the low-level computer hardware for applications.
* An operating system implements deadlock detection. Upon detecting deadlock, it notifies the system administrator of the set of deadlocked processes that it has detected. A practical method of deadlock recovery is for the administrator to kill one of the processes in the set, thus allowing the others to continue.
* On a uniprocessor machine, disabling interrupts prior to entering short critical sections, and re-enabling interrupts on exiting is a reasonable approach to achieving mutual exclusion within critical sections inside the operating system.
* Using a bitmap to manage free blocks on a disk reduces the usable disk capacity compared to the approach of storing the addresses of free blocks in the free blocks themselves.
* When an application performs a series of small file system writes, having the operating system contiguously allocate multiple blocks on the initial write (and occasionally on subsequent writes) can improve file system performance compared to allocating a block at a time.
* On a microprocessor with no privileged mode, an operating system can implement resource allocation policies, but not enforce them.
* In the user mode of the microprocessor, only the application's virtual memory is accessible to the processor.
* On a multiprocessor machine, a single ready queue in the operating system provides automatic load balancing across CPUs.




## FALSE 
* A read before test and set implementation of a spinlock (also called a test and test and set) just adds extra overhead to the lock implementation by adding a superfluous read.
* In a typical UNIX file system, such as ext2fs, the attributes of the file (e.g., file owner, size, and creation time) are stored in the directory entry.
* A buffer cache improves performance by buffering writes to disk, allowing the application to continue computation while buffers are written to the disk in the background. Meta-data and data writes are generally treated equally and simultaneously flushed to disk periodically
* Multi-threaded applications (i.e. user-level code) can avoid race conditions by disabling and enabling interrupts
* The translation lookaside buffer (TLB) that translates virtual addresses to physical addresses for every CPU instruction is sometimes implemented in software. 
* In a system using page-based virtual memory, increasing the page size will generally increase the TLB miss rate.
* All application calls to the standard C library result in a transition into kernel mode to process the call, after which the processor returns to the application in user mode.
* Because the operating system runs in privileged mode, the operating system can safely use the memory pointers provided by applications (via the arguments to system calls) just like in-kernel memory pointers.
* Swapping allows an application larger than physical memory to execute.
* A typical i-node-based UNIX file system features differing levels of indirection (levels of indirect blocks) depending on the offset of access to the file. File access performance would be improved for small files if the file system was simplified to use three levels of indirection for all offsets in the file.
* Condition variables are used to manage blocking and waking within critical sections protected by  semaphores.
* Internal fragmentation is space wasted in main memory. External fragmentation is space wasted on disk.
* An inverted page table is a single data structure shared between all processes, therefore, it is efficient at enabling the sharing of virtual memory between multiple processes.
* A process and the operating system kernel share the same stack so they both can execute a high-level language like the C programming language.
* In a multi-threaded process, every thread has its own virtual address space.
* Multi-threaded applications (i.e. user-level code) can avoid race conditions by disabling and enabling interrupts.
* Page-based virtual memory using a single page size is likely to suffer from external fragmentation in main memory.
* A system uses a resource management policy that is potentially prone to starvation (e.g. shortest job first scheduling). When starvation occurs, the whole system makes no progress. 
* An I/O-bound process spends a significant fraction of its execution time blocked waiting for I/O. One can reduce the overall execution time of the application by using user-level threads to overlap the application's blocking I/O with its execution.
* Semaphores can be used to implement mutually exclusive execution in critical sections in a multi-threaded program by initialising the semaphore count to zero.
* Application multi-threading supported by user-level threads features a per-thread user-level stack and a 1-to-1 corresponding per-thread in-kernel stack.
* Using an operating system is always more efficient than programming the computer hardware directly, even for a single simple application.
* The size of an inverted page table is determined by the virtual address space size of a process, and is unrelated to the size of physical memory.
* When the amount of virtual memory allocated in a system exceeds the physical memory size, thrashing always occurs.
* Optimal is the best practical page replacement algorithm.
* A FAT file system gets its name as it is designed for really big disks.
* Spatial locality contributes to virtual memory system efficiency, but temporal locality does not.
* Deadlock prevention can be achieved by mitigating one of the four conditions required for deadlock. Each one of the four conditions are as practical a candidate to mitigate as any of the others.

