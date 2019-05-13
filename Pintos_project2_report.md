# Pintos Project2 Report: User programs
:+1: Pintos project_2  - for SUSTech OS  :shipit:
## Team Members and Mission distribution
```
11612201 Zhongwei Wan : 
Task1-Argument Passing, Task2-Process Control Syscalls, Final report
11612201 Xuande Tang : 
Task2-Process Control Syscalls, Task3-File Operation Syscalls, Code annotaion
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
int IWrite(struct intr_frame *f);
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
void put_stack_operation(int nums, char * word, char * file_name_str, char * ok_str, void ** space, char * file_str)
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
void judge_stack_addr(const void *address);
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
void Halt(void);
```
- system call Halt.
```c
int Execute(struct intr_frame *f);
```
- system call Execute.
```c
int Wait(struct intr_frame *f);
```
- system call Wait;
### thread.h
```c
struct child_process * next_c_process; 
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
struct file *exec_file;  
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

void Halt(void) {
	shutdown_power_off();
}
```
- **Step3**:Execute(). In this algorithm, the user can create a child process using this system call. The specific steps are a. First, assign a page and copy a user-provided username. b. Create a child process in the function call process execute function.
