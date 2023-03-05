# Processes and Threads

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
10. **`fork()`** (3): < 0 (error); == 0 (child process); > 0 (parent process; pid = child's process ID)
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
        /* Parent; pid = process ID of its child */
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
11. **Fork bomb**: A malicious program calls `fork` repeatedly, and the number of processes spawned is too high for the system to manage. To defend, we can set limit to (1) total number of processes a user may create, and (2) the rate a use spawns a new process.

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
6. Thread attributes: **joinable**; **detachable**; scheduling policy. To prevent a thread from being joined, we can call `pthread_detach`. When a joinable thread is finished, it cannot be cleaned up (its resources deallocated) until it is joined and the return value collected. A detached thread, when it finishes, is cleaned up automatically.
7. Thread cancellation (2): **asynchronous cancellation** (one thread immediately terminates the target); **deferred cancellation** (default) (target is informed that it is cancelled and the target is responsible for cleaning itself up)
```c
/* type: PTHREAD_CANCEL_DEFERRED or PTHREAD_CANCEL_ASYNCHRONOUS */
pthread_setcanceltype( int type, int *oldtype );
```
8. A large number of functions are **cancellation points**: the POSIX specification requires an implicit check for cancellation when calling one of those functions
9. Sometimes a thread can be terminated before it has cleaned up some resources. **Cleanup handler** can help with it!
```c
pthread_cleanup_push( void (*routine)(void*), void *argument ); /* Register cleanup handler, with argument */
pthread_cleanup_pop( int execute ); /* Run if execute is non-zero */

void cleanup( void* mem ) {
    if ( *( ( void ** ) mem) != NULL ) {
        free( *( ( void ** ) mem) );
    }
}

void* do_work( void* argument ) {
    struct job * j = malloc( sizeof( struct job ) );
    pthread_cleanup_push( cleanup, &j );
    /* Do something */
    free( j );
    pthread_cleanup_pop( 0 ); /* Don’t run */
    pthread_exit( NULL );
}

/* Alternative implementation */
void* do_work( void* argument ) {
    struct job * j = malloc( sizeof( struct job ) );
    pthread_cleanup_push( cleanup, &j );
    /* Do something */
    pthread_cleanup_pop( 1 ); /* Run */
    pthread_exit( NULL );
}
```
