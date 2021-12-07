* Process and interrupt contexts
  * most modern monolithic OSes are monolithic in design
  * process context(task): when a process or thread issues a system call, it switches to kernel mode and executes kernel code (synchronous)
  * interrupt context(atomic): HW fires a interrupt and ISR runs (also runs in kernel mode, asynchronous)
  * ![image](https://user-images.githubusercontent.com/24621517/144950059-07b5cd47-9260-439d-9b88-b21162c25c0e.png)
* The traditional UNIX process model: "Everything is a process; if not, it's a file"
  * A thread is merely an execution path within a process
  * Threads share all process resources including the user VAS, **except for the stack**
  * Every thread maps to a kernel metadata structure called the **task structure**
  * **One stack per thread per privilege level supported by the CPU**
    * A user space stack
    * A kernel space stack
      * kernel thread only has a kernel stack
  * ![image](https://user-images.githubusercontent.com/24621517/144950516-872d5f01-1b9f-4409-ad2e-09867a8b2cb7.png)
* User space stack
  * Text/Code
  * Data segments (init/uinit, heap)
  * Library mappings
  * Stack (default limit 8MB)
  * dynamic in size
* Kernel space stack
  * Stack (2 pages on 32-bit, 4pages on 64-bit)
  * **task_struct**
  * fixed in size
* Viewing the stack
  * /proc/<pid>/stack
  * utilities: gstack, bpf
* Understanding and accessing the kernel task structure
  * The task stucture is the root metadata structure for the thread
  * It encapsulates all the information required by the OS for that thread
  * Scheduling, files, credentials, locks, AIO contexts, HW contexts, signaling, IPC objects, ... etc.
  * Certain attributes will be inherited by a child process or thread upon fork(2)
  * ![image](https://user-images.githubusercontent.com/24621517/144951840-bb874d53-9acb-4e64-bd76-13dc39d9fece.png)
* Accessing the task structure with current
  * all the task strucutre objects in kernel memory are chained up on a circular doubly linked list called the task list
  * we can access thread currently running(process context) via current macro
  * current yields the pointer to task_struct
* Determining the context
  * in_task() macro return true if your code is running in process context
* Tips
  * do_each_thread() { ... } while_each_thread()

  
