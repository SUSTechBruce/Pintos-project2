# Pintos Project2 Report: User programs
:+1: Pintos project_2  - for SUSTech OS  :shipit:

## Qustion and Reflection
- A reflection on the project–what exactly did each member do? What went well, and
what could be improved?
```
11612201 Zhongwei Wan : 
Task1-Argument Passing, Task2-Process Control Syscalls, Final report
11610103 Dexuan Tang : 
Task2-Process Control Syscalls, Task3-File Operation Syscalls, Code annotaion
The task2 and task3 , we did a good job. We can improve the Code readability.
```
- Does your code exhibit any major memory safety problems (especially regarding C
strings), memory leaks, poor error handling, or race conditions?
```
Yes, we have had these problems, but we tried our best to solve these memory safety 
problems in the later testing process.

```
- Did you use consistent code style? Your code should blend in with the existing Pintos
code. Check your use of indentation, your spacing, and your naming conventions.
```
Yes, we use a continuous code style. We have used code refactoring to keep the mix 
code consistent in style.
```
- Is your code simple and easy to understand?
```
Our code is not very simple, and we wrote very detailed comments.
```
- If you have very complex sections of code in your solution, did you add enough comments to explain them?
```
Yes. it is .Yes, I do.
```
- Did you leave commented-out code in your final submission?
```
Yes, I have tried to do that, but some of commented-out codes are also important to 
demonstrate how we modifiy the code.
```
- Did you copy-paste code instead of creating reusable functions?
```
We looked at some code written by other people and some of their ideas. We thought 
about designing and constructing based on combining our thoughts with some source material
. Most of our job is designing and complete the mission in the project and comparing our idea 
with other one's idea. That is a great way to improve our coding ability.
```
- Are your lines of source code excessively long? (more than 100 characters)?
```
Negative, we have solved this problem.
```
- Did you re-implement linked list algorithms instead of using the provided list manipulation?
```
No, we just use the given list manipulation.
```
## Task 1: Argument Passing
## I.Data structures and functions
### thread.h
```c 
struct file *self_file;
```
- the executable file of process.

```c
struct list children_list;
```
- the new creating child process list of parent process.
### process.c
```c
process_execute(const char *file_name);
```
- Starts a new thread running a user program loaded from filename

```c
int process_wait(tid_t child_tid);
```
- Waits for thread TID to die and returns its exit status.

```c
bool load(const char *file_name, void(**eip) (void), void **esp);
```
- Loads an ELF executable from FILE_NAME into the current thread.

```c
void load_split(char * file_str, const char * file_name);
```
- split filename and arguments in load function.
```c
static bool setup_stack(char * file_name_str, void **esp);
```
- Create a minimal stack by mapping a zeroed page at the top of user virtual memory.
### syscall.c
```c
write(int size,void *buffer,int fd);
```
- write mission of system call.
## II. Algorithms and implementations
### Main algorithm
- 1. Separate the file name and parameters passed in from the command line.
- 2. Put the parameters on the stack according to the C function calling convention.
### Implementations
- **Step1**: In the `process_execute()` function, first use the `strtok_r()` function to separate the file name from the parameter, use the global variable to record the file name, and pass the file name to the new thread created by `threadcreate()`. Once the thread is created, let it enter the wait phase.
```c
//切割filename，将参数和名字分离
	char *get_tname;
	get_tname = malloc(strlen(file_name) + 1);
	strlcpy(get_tname, file_name, strlen(file_name) + 1);
	get_tname = strtok_r(get_tname, " ", &string_save);
	//创建新线程，名字叫做get_tname
	tid = thread_create(get_tname, PRI_DEFAULT, start_process, fn_copy);
	free(get_tname);
```
- **Step2**: Pass the separated file name to the `load()` function, assign the value to the user stack pointer `esp` and allocate the stack space.
```c
void load_split(char * file_str, const char * file_name)
{
	strlcpy(file_str, file_name, strlen(file_name) + 1);
	file_str = strtok_r(file_str, " ", &string_save);
}
char *file_str = malloc(strlen(file_name) + 1);
load_split(file_str, file_name);
file = filesys_open(file_str);
```
- **Step3**: In the `setupstack()` function, the parameters separated by `strtok_r` are assigned to the user stack, and the parameter array `arg[]` is set to save the position of these parameters on the stack.
```c
  char * file_str = malloc(strlen(file_name_str) + 1);
	file_str_split(file_str, file_name_str);
	int str_length = strlen(file_name_str);
```
- **Step4**: The stack pointer has not been stored in one parameter mod4 because the stack is four-byte aligned.
```c
*space = *space - ((unsigned)*space % 4);
```
- **Step5**: Follow the saved pointers in step 2 in the stack, put `argc`, `argv`, and let the user stack pointer point to the new stack top.(Put the parameters into the specified memory, and set the parameters into the specified memory)
```c
void put_stack_operation(int nums, char * word, char * file_name_str, char * ok_str, void ** space, 
char * file_str)
{
	int *argument = calloc(nums, sizeof(int));
	int count;
	word = strtok_r(file_name_str, " ", &ok_str);
	push_addrss(count, word, space, argument, ok_str, nums, file_str);
}
void push_addrss(int count, char * word, void ** space, int * argument, char * ok_str, int nums, char * file_str)
{
	for (count = 0; ; count++) {
		if (word) {
			*space -= strlen(word) + 1;
			memcpy(*space, word, strlen(word) + 1);
			argument[count] = *space;
			word = strtok_r(NULL, " ", &ok_str);
		}
		else {
			break;
		}
	}
	*space = *space - ((unsigned)*space % 4);
	*space = *space - sizeof(int);

	for (count = nums - 1; count >= 0; count--)
	{
		*space = *space - sizeof(int);
		memcpy(*space, &argument[count], sizeof(int));
	}

	int change = *space;
	*space = *space - sizeof(int);
	memcpy(*space, &change, sizeof(int));
	*space = *space - sizeof(int);
	memcpy(*space, &nums, sizeof(int));
	*space = *space - sizeof(int);
	memcpy(*space, &argument[nums], sizeof(int));
	free(file_str);
	free(argument);
}
```
## Synchronization
- In order to maintain synchronization, we use the `fileslock` defined in thread.h to ensure that each time a process is executed, no other process will enter the `critical section` of the executable to ensure the synchronization of the process.
```c
lock_acquire(&filesys_lock);
// Loads an ELF executable from FILE_NAME into the current thread.
lock_release(&filesys_lock);
```
## Rationale
- In the task1 passed by this parameter, our main purpose is to separate the file name and various parameters passed in from the command line in process execute, load and setupstack, and assign the parameters to a specific stack in a specific order. Therefore, the method logic for processing parameter passing is relatively clear.

## Task2: Process Control Syscalls
## Data structure and functions
### syscall.c
```c
void check_valid_get_address(const void *address);
```
- Determine whether the address is reasonable, such as whether it exceeds the address of the stack, and whether it does   not match the address format of the stack.
```c
void call_of_stack(int *arg, int *pointer, int offset);
```
- Call the parameters out of the stack and perform the next call to the system call function.
```c
void syscall_init();
```
- Initialize a specific array to NULL；
```c
void syscall_handler();
```
- Call the corresponding code according to the parameter name.
```c
void halt();
```
- system call Halt.
```c
int execute(char *file_name);
```
- system call Execute.
```c
int wait(tid_t tid);
```
- system call Wait;
### thread.h
```c
struct child_process * wait; 
```
- Next child process.
```c
struct list file_list;    
```
- List of all open files.
```c
uint8_t *stack;                   
```
- Save stack pointer
```c
enum process_status check;  
```
- Save the process state, whether it is a successful load or a failure.
```c
struct file *self;  
```
- Determine if this file is an executable file.
```c
 unsigned stackoverflow;               
```
- Determine if a stack overflow will occur

## Algorithm and implenmentation
- **Step1**: Initialize the parameter array in the created `syscall_init()` function, and complete the corresponding parameter call method in the syscallhandler.
- **Step2**: The system calls system call HALT. First, call the `shut_down_poweroff()` function to shut down the operating system. When the user program causes a page fault, it will enter the `pagefault()` function. We need to add the pagefault function to handle the page error.
```c

void halt() {
	shutdown_power_off();
}
```
- **Step3**:Execute(). In this algorithm, the user can create a child process using this system call. The specific steps are : **a.** First, assign a page and copy a user-provided username. **b.** Create a child process in the function call process execute function. **c.** After calling `process_execute()`, you need to wait for `sema_down(sema)` on a semaphore until the user program is successfully loaded in the `start_process()` function.After `semp_up(sema)` activates the parent process, it should immediately hang itself after activating the parent process,`sema_down(sema)`.**d**. Add semaphore `WaitSuccess` to the struct_thread structure, return pid if the child process is created successfully, and return -1 if it fails.
```c
int execute_process(char *file_name);
```
- **Step4**:Wait().First, we made it clear that the parent process may have to wait for the child process to finish after creating the child process. There may be two situations. One is when the parent process calls process_wait() and the child process has not ended.
The parent process will be suspended, and the parent process will wake up after the child process ends, and the parent process will get the return value. The second is that the child process has ended when the parent process calls `process_wait()`. This requires the child process to save the return value to the process control block of the parent process. So we need to add a data structure in `thread.h` to save the return value of the child process. Therefore, our approach is to **a.** Compare the pid implementation by traversing `children_list`. If the process control block of the child process is not found in child_list; indicating that the child process has been closed Bundle, directly return the return value of the child process from its own child_list linked list. **b.** If the child process is still running, execute `sema_down(t->parent->SemaWait)` to suspend itself. After the child process is executed, it is found. In waiting==true, you are waiting, and then release the parent process `sema_up(SemaWait)`; if `waiting==false`, you don't need to wake up the parent process. **c.** At the end of a process, call the`free()` function.
```c
void process_wait(child_tid);
```
## Synchronization
In the implementation of the `execute()` function and the `waiting()` function, it is necessary to call the load function in the function. The specific function of this function is Loads an ELF executable from FILE_NAME into the current thread.
Stores the executable's entry point into *EIP and its initial stack pointer into *ESP.Returns true if successful, false otherwise. And we have modified this function, using `lock_acquire(&filesys_lock)` and `lock_release(&filesys_lock)` to ensure that the process is not interfered by other processes. Therefore, the synchronization and consistency of the system call function is guaranteed.
```c
lock_acquire(&filesys_lock);
// Loads an ELF executable from FILE_NAME into the current thread.
lock_release(&filesys_lock);
```
In addition, when these functions involve the relationship between the child process and the parent process, the synchronization semaphore is used to ensure synchronization.
```c
semp_up(sema);
sema_down(sema);
```
## Rationale
- n implementing these functions, we analyzed the various deep call functions involved in these functions, and used the synchronization `semaphore` to ensure the relationship between the father and child processes.

## Task3: File Operation Syscalls
## Data structure and functions
### syscall.h
```c
void exit(int status);
```
- This system call is called when the user program exits normally.
```c
int create(char *name,off_t initial_size);
```
- System call, create file.
```c
int remove(char *name);
```
- System call, delete file.
```c
int open(char *name);
```
- System call, open file.
```c
int filesize(int fd);
```
- System call to get the file size.
```c
int read(int size,void *buffer,int fd);
```
- System call, read file.
```c
int write(int size,void *buffer,int fd);
```
- The function will call this system call to output to the screen, so without implementing this system call, the user program will not be able to output any characters.
```c
void seek(int fd,int pos);
```
- Moving file pointer.
```c
int tell(int fd);
```
- Returns the current position of the file pointer.
```c
void close(int fd);
```
- Close file.
- Design a struct for record the process of files, including process_r, list_elem elem, index.
### thread.h
```c
enum process_status process_status;
```
- Record the status of process, to check whether the loading is succussful or failure.
```c
struct semaphore is_load;   
```
- The current thread waits until the end of all processes.
```c
int exit_status;    
```
- Status code when exiting.
### process.h
```c
struct process_file {
	struct file* ptr;
	struct list_elem elem;
	int fd;
};
```
## Algorithm and implenmentation

- Step1: exit().This system call is called when the user exits the program. Therefore, first we should take the return value, save it in the variable of the process control block, and call `thread_exit()` to exit the process. We also need to consider that when the user program exits abnormally, for instance, array out of bounds，it needs to add another function to achieve.
```c
void
thread_exit(void)
{
	ASSERT(!intr_context());
	process_exit();
	intr_disable();
	list_remove(&thread_current()->allelem);
	thread_current()->status = THREAD_DYING;
	schedule();
	NOT_REACHED();
}
```
- Step2: create().For this system call, we need to take the file name separated from the parameter, and then call the filesys_create() function to save the return value.
```c
result = filesys_create(name, initial_size);
```
- Step3: remove(). Remove the file name of the file to be deleted from the user stack. Call filesys_remove() to delete the file.
```c
if (filesys_remove(name) == NULL)
		result= 0;
	else
		result= 1;
```
- Step4: open().Take the file name and call the filesys_open function to open the file. At the same time, in order to maintain an open file table for each process, you need to add `f_num` of the number of records in the struct, an open file `list file_list`, and an allocated `fd`, and make this structure associated with the following structure 
```c
struct process_file {
	struct file* ptr;
	struct list_elem elem;
	int fd;
};
```
Therefore, each time you open a file you need to create a file_node structure, assign a handle, and add file_node to file_list; finally return the file handle.

- Step5: filesize(). Get fd from the user stack and get the file pointer from the process open file table by fd.Call file_len gth to get the file size.
```c
result = file_length(search_fd(&thread_current()->file_list, fd)->ptr);
```
- Step5: read().Get the fd buffer size three parameters from the user stack. If fd is a standard input device, call input_getc(). If fd is a file handle, fd gets the file pointer from the process open file table, and calls file_read() function from the file and reading data in the middle.
```c
struct process_file *pf = search_file_id(&thread_current()->file_list, fd);
if (pf == NULL)
	return -1;
else
{
	lock_acquire(&filesys_lock);
	result = file_read(pf->ptr, buffer, size);
	lock_release(&filesys_lock);
}
```
- Step6: write().1. First take three parameters from the user stack fd, buffer, size as the Step5.
```c
args[0] = *((int *)check_valid_get_address(p + 7));
args[1] = *((int *)check_valid_get_address(p + 6));
args[2] = *((int *)check_valid_get_address(p + 5));
```
If fd is a file handle, first find the file pointer corresponding to the handle from the process open file table and then call the `file_write()` function provided by pintos to write data to the file.
```c
result = file_write(pf->ptr, buffer, size);
```
2. Open the file table will be established when the file is opened, and then the specific implementation when the SYS_OPEN system call is implemented.
3. If fd is the standard output stdout handle, call the putbuf function to output to the terminal.

- Step7: seek().Extract the file sentence from the user stack. The distance to which the fd handle should be moved, convert fd to a file pointer, and call the file_seek() function to move the file pointer.
```c
fd = args[0] = *((int *)check_valid_get_address(p + 5));
pos = args[1] = *((int *)check_valid_get_address(p + 4));
file_seek(search_fd(&thread_current()->file_list, pos)->ptr, fd);
```
- Step8. tell().Remove the file sentence from the user stack. The distance the fd handle should move, turn fd into a file pointer, and call the file_tell() function to get the pointer position.
```c
fd = args[0] = *((int *)check_valid_get_address(p + 1));
result = file_tell(search_fd(&thread_current()->file_list, fd)->ptr);
```
- Step9. close().Get the handle of the file to close from the user stack. Find the corresponding file in the user open file list to get the file pointer. Call the clean_single_file() function to close the file and release the struct file_node.
```c
fd = args[0] = *((int *)check_valid_get_address(p + 1));
clean_single_file(&thread_current()->file_list, fd);
```
## Synchronization
In order to maintain synchronization, we added `lock_acquire(&filesys_lock)` and `lock_release(&filesys_lock)` to all key steps of the system call function to ensure that the process is executed without interference from other processes.For instance:
```c
lock_acquire(&filesys_lock);
result = file_read(pf->ptr, buffer, size);
lock_release(&filesys_lock);
```
## Rationale
In implementing these nine system call functions, we use a lot of functions provided by the source code, such as `file_read (pf->ptr, buffer, size)` or `file_write (pf->ptr, buffer, size)`. Reading these source code can help us with our tasks. In addition, in order to maintain the synchronization of the program, a file lock is added to these key operations.

# Reference
https://github.com/pindexis/pintos-project2  
https://github.com/Wang-GY/pintos-project2  
https://wenku.baidu.com/view/eed6bbdaa48da0116c175f0e7cd184254b351ba8.html  
https://download.csdn.net/download/zhibolau/9275043  


