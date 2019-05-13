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
- 
