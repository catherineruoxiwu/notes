# SE 350 Midterm Notes

## Processes
1. **PCB** (8): id; state; priority; PC; reg data; memory pointers; I/O status; accounting info
2. **Events lead to process creations** (3): system boot up; user request to start a new proc; one process spawns another
3. **Process destruction** (4): normal exit (v); error exit (v); fatal error (inv); killed by another proc (inv)
4. **Zombie process**: A child process finishes execution, but the parent had not collected the return value yet. The child process is dead but not gone because it has not been cleaned up, and it is still in the PCB list and holds all its resources.
5. **Orphan process**: The child process's parent die before the child does. In UNIX, orphan is automatically adopted by the 'init' process so that the orphans would not become zombies.
6. **Process states** (5): running; ready; blocked; new; terminated 
7. **Process transitions** (8): create (* -> new); admit (new -> ready); dispatch (ready -> running); suspend (running -> ready); exit (running -> terminated); block (running -> blocked); unblock (blocked -> ready); reap (terminated -> *)
8. **Swapping processes to disk**: The OS puts parts of some processes on disk so that it can run more processes at the same time. Swapping is time-consuming.
    - 2 new states: ready/swapped; blocked/swapped
    - admit transition (new -> ready/swapped)
9. **Running multiple processes in shell**: `gcc fork.c &` (start process and return control to the console); `screen` command
10. **`fork()`** (3): < 0 (error); == 0 (child process); > 0 (parent process)
```c
int main( int argc, char** argv ) {
    pid_t pid;
    int child_result;
    int parent_result;
    pid = fork();
    if ( pid < 0 ) { 
        /* Fork Failed */
        return -1;
    } else if ( pid == 0 ) { 
        /* Child */
        return execute_B();
    } else { 
        /* Parent */
        parent_result = execute_A();
        /* child_result stores the exit code of the child process */
        wait( &child_result );
    }
    /* no errors: 0; otherwise: -1 */
    if ( child_result == 0 && parent_result == 0 ) {
        printf( "Completed.\n" );
        return 0;
    }
    return -1;
}
```
11. **Fork bomb**: call `fork` repeatedly until the number of processes spawned is too high for the system to manage. To defend, we can set limit to (1) total number of processes a user may create, and (2) the rate a use spawns a new process.

## Threads
1. **Thread info** (5): state; saved context; execution stack; local variables; access to resources (shared with all threads in the process)
2.  **Motivation of creating a new thread** (5): faster than creating a process; faster terminating and cleaning up; takes less time to context switch; direct communications between threads; make part of a program responsive when part of it is blocked
3. **Examples of the uses of threads** (4): foreground and background work; async processing; speed of execution; modular structure (each thread does a different thing)
4. **Drawbacks of threads** (2): no protection between threads in the same process; if one thread encounters an error, the whole process might terminate
5. **POSIX threads**
```c
/* global variables */
pthread_t thread;
foo* parmas = malloc( sizeof( foo ) );
void *returnValue;
/* helper function */
void *goo( void *args );

pthread_create( pthread_t *thread, const pthread_attr_t *attributes
, void *(*start_routine)( void * ), void *argument );
pthread_create( &thread, NULL, goo, params );

/* Wait for a specific thread to exit */
pthread_join( pthread_t thread, void **returnValue );
/* returnValue can be read through casting to the expected pointer */
pthread_join( thread, &returnValue ); 
free(returnValue);

/* a thread cannot be joined */
pthread_detach( pthread_t thread );
/* singal cancellation to a thread, can be async or deferred */
pthread_cancel( pthread_t thread );
/* returns nonzero if cancelled */
pthread_testcancel( ); 

/* If main calls pthread_exit, it would wait for all unjoined thread to finish */
pthread_exit( void *value );

int *rv = malloc( sizeof( int ) );
*rv = 99;
pthread_exit( rv );
```

## Concurrency & Synchronization (Concepts)
1. **Synchronization** vs. **parallelism**: synchronization (multiple processes and threads making progress at the same time); parallelism (more than one thread or process executing on a CPU at a given instant)
2. **Serialization**: We want to execute events with some particular order. (e.g. merge sort)
3. **Mutual exclusion**: Two events cannot happen at the same time. (e.g. read and write)
4. **Atomic**: operation that cannot be interrupted
5. 

## Concurrency & Synchronization (Examples)

1. 
<table>
<tr>
<th> Thread A </th>
<th> Thread B </th>
</tr>
<tr>
<td>

```
int foo() {
    int result = 4;
    return result;
}
```

</td>
<td>

```
int foo() { 
    int x = 4;
    return x;
}
```

</td>
</tr>
</table>