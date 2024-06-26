```jsx
Course: CS-225-L - Operating System Lab
Name : Muhammad Abdullah
Reg. ID: BS-CYS-F22-221546
Lab Number: 05
```

# C Programming and System Calls

This lab contains basic scripts to demonstrate the usage of System Calls.

## File Creation:
> First, create an empty file by name `system_calls.c`

```console
┌──(MnM@kali)-[~/Desktop/OS-L/Tasks]
└─$ touch system_calls.c

┌──(MnM@kali)-[~/Desktop/OS-L/Tasks]
└─$ ls
hello.sh  system_calls.c
```

## Code Compilation

> `Nano` file editor can be used to edit the file and write code in it.

```bash
sudo nano system_calls.c
```

> Creating an executable `exe` file we'll use the `gcc` compiler

```bash
gcc system_calls.c -o output
```

> Runing the code

```bash
./output
```

## Lab Codes

**Task Number: 01**

> Write a code to explain the use of `fork()` system call

**Code:**

```C
#include <stdio.h>
#include <stdlib.h>
#include <sys/wait.h>
#include <unistd.h>

int main()
{
    pid_t cpid;

    if (for() == 0)
    {
        exit(0);
    }
    else
    {
        cpid = wait(NULL);
        printf("Parent pid = %d\n" , getpid());
        printf("Child pid = %d\n" , cpid);
    }

    return 0;
}
```

**Output:**

```console
┌──(MnM@kali)-[~/Desktop/OS-L/Tasks]
└─$ ./output
Parent pid = 8890
Child pid = 8891
```

**Explanation**

This `C` program uses the `fork()` system call to create a child process. In the child process, it immediately exits, terminating the child. The parent process waits for the child to complete using `wait()`, capturing the child's process ID. Finally, both the parent and child processes print their respective process `IDs`, demonstrating the creation and termination of the child process.

**Task Number: 02**

> Expalin the use case of `Zombie`

**Code:**

```C
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/wait.h>

int main() {
    pid_t child_pid;

    child_pid = fork();

    if (child_pid == -1) {
        perror("fork");
        exit(EXIT_FAILURE);
    } else if (child_pid == 0) {
        printf("Child process created. PID: %d\n", getpid());
        exit(EXIT_SUCCESS);
    } else {
        printf("Parent process. PID: %d, Child PID: %d\n", getpid(), child_pid);
        sleep(10);
        exit(EXIT_SUCCESS);
    }

    return 0;
}
```

**Output:**

```console
┌──(MnM@kali)-[~/Desktop/OS-L/Tasks]
└─$ ./output &
[1] 22718

Parent process. PID: 22718, Child PID: 22720
Child process created. PID: 22720
```

> When you append an ampersand `&` at the end of a command in the shell, it runs the command in the background

```console
┌──(MnM@kali)-[~/Desktop/OS-L/Tasks]
└─$ ps -l
F S    UID     PID    PPID  C  PRI  NI  ADDR SZ   WCHAN  TTY            TIME  CMD
0 S   1000   22718    1792  0   85   5 -     616 hrtime  pts/0      00:00:00  output
0 Z   1000   22720   22718  0   85   5 -     0   -       pts/0      00:00:00  output
```

> Now when we use the command `ps -l` it will enlist all the current running processes and we will check our respective `process ID` from the list as shown and highlighted below.

**Explanation:**

- The `C` program initiates a child process using the `fork()` system call, creating a new process in which the child immediately `exits`, becoming a zombie.
- The parent process, upon successfully creating the child, prints information about both the parent and child processes, including their respective process IDs `PIDs`.
- The parent process then sleeps for ***10 seconds*** using the `sleep()` function, allowing the child process to persist as a **zombie** during this period.
- After the sleep duration, the parent process exits, leaving the child process in a zombie state as the parent did not wait for the child's termination using functions like `wait()`.
- When running the program in the background and checking the process status using `ps - l`, the child process with a state of `Z` (zombie) should be visible, confirming its existence in the system

**Task Number: 03**

> Write a code to demonstrate priority scheduling.

```C
#include <stdio.h>

#define MAX_PROCESSES 5

typedef struct {
    int process_id;
    int priority;
    int burst_time;
    int waiting_time;
    int turnaround_time;
} Process;

void swap(Process *a, Process *b) {
    Process temp = *a;
    *a = *b;
    *b = temp;
}

void sort_by_priority(Process processes[], int n) {
    for (int i = 0; i < n - 1; i++) {
        for (int j = 0; j < n - i - 1; j++) {
            if (processes[j].priority > processes[j + 1].priority) {
                swap(&processes[j], &processes[j + 1]);
            }
        }
    }
}

void priority_scheduling(Process processes[], int n) {
    sort_by_priority(processes, n);

    int total_waiting_time = 0;
    int total_turnaround_time = 0;

    printf("Process\tPriority\tBurst Time\tWaiting Time\tTurnaround Time\n");
    for (int i = 0; i < n; i++) {
        if (i == 0) {
            processes[i].waiting_time = 0;
        } else {
            processes[i].waiting_time = processes[i - 1].waiting_time + processes[i - 1].burst_time;
        }
        processes[i].turnaround_time = processes[i].waiting_time + processes[i].burst_time;
        
        total_waiting_time += processes[i].waiting_time;
        total_turnaround_time += processes[i].turnaround_time;

        printf("%d\t%d\t\t%d\t\t%d\t\t%d\n", processes[i].process_id, processes[i].priority, 
               processes[i].burst_time, processes[i].waiting_time, processes[i].turnaround_time);
    }

    double avg_waiting_time = (double)total_waiting_time / n;
    double avg_turnaround_time = (double)total_turnaround_time / n;

    printf("\nTotal Waiting Time: %d\n", total_waiting_time);
    printf("Average Waiting Time: %.2lf\n", avg_waiting_time);
    printf("Total Turnaround Time: %d\n", total_turnaround_time);
    printf("Average Turnaround Time: %.2lf\n", avg_turnaround_time);
}

int main() {
    Process processes[MAX_PROCESSES] = {
        {1, 3, 10, 0, 0},
        {2, 1, 5, 0, 0},
        {3, 4, 7, 0, 0},
        {4, 2, 12, 0, 0},
        {5, 5, 3, 0, 0}
    };

    printf("Before scheduling:\n");
    priority_scheduling(processes, MAX_PROCESSES);

    return 0;
}
```

**Output**

```console
┌──(MnM@kali)-[~/Desktop/OS-L/CPU]
└─$ ./Priority
Process Priority Burst Time  Waiting Time  Turnaround Time
2       1        5            0             5
4       2        12           5             17
1       3        10           17            27
3       4        7            27            34
5       5        3            34            37

Total Waiting Time: 83
Average Waiting Time: 16.60
Total Turnaround Time: 120
Average Turnaround Time: 24.00
```

---
