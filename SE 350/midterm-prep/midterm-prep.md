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
11. **Fork bomb**: A malicious program calls `fork` repeatedly, and the number of processes spawned is too high for the system to manage. To defend, we can set limit to (1) total number of processes a user may create, and (2) the rate a use spawns a new process.

## Inter-Process Communication (IPC)
1. **Sync send, sync receive**: sender is blocked when the message is sent, receiver will continue whether or not a message is received (uncommon)
2. **Approaches to accomplish IPC** (4): file system; message passing; netwoking communication; pipes and shared memory
3. **File system**: producer and consumer processes communicate through files in the file system; the system is responsible for file creation and manipulation (e.g. managing permissions)
4. **Message passing using signals**
    - signals are an interrupt with a specified ID and no messages
    - 

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
5. **Interrupts cannot be disabled to protect critical section** (2): interrupts from users or other normal thread switches will not occur either; not sufficient when there are multiple processors
6. **Test-and-Set**: a special machine instruction that is performed in a single instruction cycle (an atomic read and write)
```c
boolean test_and_set( int* i ) {
    if ( *i == 0 ) {
        *i = 1;
        return true;
    } else {
        return false;
    }
}
```
7. **Busy-waiting**
```c
while ( !test_and_set( busy ) ) {
    /* Wait for my turn */
}
/* Critical section */
busy = 0;
```
8. **Binary semaphore**: a variable that has two values 0 and 1
    - `wait` is how a program tries to enter the critical section
        - if the semaphore is 1, set it to 0 and enter the critical section
        - if the semaphore is 0, some other thread is in the critical section then the curren tthread must wait its turn and the current thread is blocked by the OS
    - `post` is how a program sends the message that it is finished with the critical section
        - if the semaphore is 1, do nothing
        - if the semaphore is 0 and there is a task blocked awaiting the semaphore, the task gets unblocked
        - if the semaphore is 0 and there are not any tasks blocked, set the semaphore to 1
9. **Mutext** from (mutual exclusion): a binary semaphore in which only the thread that has called `wait` may `post` to the semaphore
10. No `malloc` in critical sections: the memory allocation can be blocked. The process is currently in the critical section so no other thread can enter critical section, which might result in the system getting totally stuck.
11. **Deadlock** (informal definition): all threads are permanently stuck
12. **Turnstile**: the pattern of a `wait` followed immediately by a `post`; allows one thread at a time to proceed through but allow all processes to proceed through

## Concurrency & Synchronization (Examples)

1. **Mutual exclusion through flags**
<table>
<tr>
<th> Thread A </th>
<th> Thread B </th>
</tr>
<tr>
<td>

```c
A1. while ( turn != 0 ) {
A2.     /* Wait for my turn */
A3. }
A4. /* critical section */
A5. turn = 1;
```
</td>
<td>

```c
B1. while ( turn != 1 ) {
B2.     /* Wait for my turn */
B3. }
B4. /* critical section */
B5. turn = 0;
```
</td>
</tr>
</table>

Thread A runs when `turn = 0`. Thread B runs when `turn = 1`.
**WRONG** (1) strict alternation is needed: A -> B -> A -> B -> ... (2) If thread B is terminated, thread A will be stuck forever.

<table>
<tr>
<th> Thread A </th>
<th> Thread B </th>
</tr>
<tr>
<td>

```c
A1. while ( busy == true ) {
A2.     /* Wait for my turn */
A3. }
A4. busy = true;
A5. /* critical section */
A6. busy = false;
```
</td>
<td>

```c
B1. while ( busy == true ) {
B2.     /* Wait for my turn */
B3. }
B4. busy = true;
B5. /* critical section */
B6. busy = false;
```
</td>
</tr>
</table>

**WRONG** When thread switch happens after A3, both threads are in the critical section at the same time.


<table>
<tr>
<th> Thread A </th>
<th> Thread B </th>
</tr>
<tr>
<td>

```c
A1. flag[0] = true;
A2. while ( flag[1] ) {
A3.     /* Wait for my turn */
A4. }
A5. /* critical section */
A6. flag[0] = false;
```
</td>
<td>

```c
B1. flag[1] = true;
B2. while ( flag[0] ) {
B3.     /* Wait for my turn */
B4. }
B5. /* critical section */
B6. flag[1] = false;
```
</td>
</tr>
</table>

**WRONG** Thread switches after A1 and B1 would lead both threads to stuck by the while loop.

2. **Semaphore syntax**
```c
sem_t sem;

/* shared: 1 if semaphore is to be shared between processes; 0 otherwise */
sem_init( sem_t* semaphore, int shared, int initial_value );
sem_init( &sem, 0, 1 );

sem_destroy( sem_t* semaphore );
sem_wait( sem_t* semaphore );
sem_post( sem_t* semaphore );
sem_wait( &sem ); {
    /* critical section */
} sem_post( &sem );
sem_destroy( &sem );
```

3. **Signalling**: Thread A always executes before Thread B.
<table>
<tr>
<th> Thread A </th>
<th> Thread B </th>
</tr>
<tr>
<td>

```c
1. Statement A1
2. post( sem )
```
</td>
<td>

```c
1. wait( sem )
2. Statement B2
```
</td>
</tr>
</table>

**CORRECT** Semaphore is initialized to 0. In this case, it makes sense for a thread to `post` a semaphore without the `wait`, and the mutex structure is not necessary in every circumstance.

4. **Rendezvous**: Two threads "meet-up" at desired spots before either of them proceeds.
<table>
<tr>
<th> Thread A </th>
<th> Thread B </th>
</tr>
<tr>
<td>

```c
1. Statement A1
2. wait( bArrived )
3. post( aArrived )
4. Statement A2
```
</td>
<td>

```c
1. Statement B1
2. wait( aArrived )
3. post( bArrived )
4. Statement B2
```
</td>
</tr>
</table>

**WRONG** Both threads are blocked on wait.

<table>
<tr>
<th> Thread A </th>
<th> Thread B </th>
</tr>
<tr>
<td>

```c
1. Statement A1
2. post( aArrived )
3. wait( bArrived )
4. Statement A2
```
</td>
<td>

```c
1. Statement B1
2. post( bArrived )
3. wait( aArrived )
4. Statement B2
```
</td>
</tr>
</table>

**CORRECT** aArrived and bArrived are initialized to 0.


<table>
<tr>
<th> Thread A </th>
<th> Thread B </th>
</tr>
<tr>
<td>

```c
1. Statement A1
2. wait( bArrived )
3. post( aArrived )
4. Statement A2
```
</td>
<td>

```c
1. Statement B1
2. post( bArrived )
3. wait( aArrived )
4. Statement B2
```
</td>
</tr>
</table>

**CORRECT** But less efficient than the previous solution because it may require an extra context switch.

5. **Mutal exclusion**: no two threads run critical section at the same time
<table>
<tr>
<th> Thread A </th>
<th> Thread B </th>
</tr>
<tr>
<td>

```c
1. wait( mutex )
2. critical section
3. post( mutex )
```
</td>
<td>

```c
1. wait( mutex )
2. critical section
3. post( mutex )
```
</td>
</tr>
</table>

**CORRECT** The mutex semaphore is initialized to 1. 

6. **Multiplex**: at most n threads running critical section at the same time.
<table>
<tr>
<th> Thread K </th>
</tr>
<tr>
<td>

```c
1. wait( mutex )
2. critical section
3. post( mutex )
```
</td>
</tr>
</table>

**CORRECT** The mutex semaphore is initialized to n. 

7. **Barrier**: generalization of the rendezvous problem; having more than two threads meet up at the smae point before any can proceed
<table>
<tr>
<th> Thread K </th>
</tr>
<tr>
<td>

```c
1. wait( mutex )
2. count++
3. post( mutex )
4. if count == n
5.     post( barrier )
6. end if
7. wait( barrier )
```
</td>
</tr>
</table>

**WRONG** The first `wait` after `post` is called will be unblocked while other threads will be stuck forever.

<table>
<tr>
<th> Thread K </th>
</tr>
<tr>
<td>

```c
1. wait( mutex )
2. count++
3. post( mutex )
4. if count == n
5.     for i from 1 to n
6.         post( barrier )
7.     end for
8. end if
9. wait( barrier )
```
</td>
</tr>
</table>

**CORRECT** But inefficient. In the worst case, there are 2n process switches.

<table>
<tr>
<th> Thread K </th>
</tr>
<tr>
<td>

```c
1. wait( mutex )
2. count++
3. post( mutex )
4. if count == n
5.     post( barrier )
6. end if
7. wait( barrier )
8. post( barrier )
```
</td>
</tr>
</table>

**CORRECT** But it causes some minor distress accessing the value of `count` without a mutex lock.

8. **Reusable Barrier**: decrement `count` variable to 0 when there are n threads reaching the "meet-up" point

<table>
<tr>
<th> Thread K </th>
</tr>
<tr>
<td>

```c
1. wait( mutex )
2. count++
3. post( mutex )
4. if count == n
5.     post( turnstile )
6. end if
7. wait( turnstile )
8. post( turnstile )
9. [critical point]
10. wait( mutex )
11. count--
12. post( mutex )
13. if count == 0
14.     wait( turnstile )
15. end if
```
</td>
</tr>
</table>

**WRONG** (1) Context switch at line 4 when `count == n` may cause a problem. (2) On line 13, more than one thread will be waiting for `turnstile`, which would result in deadlock.

<table>
<tr>
<th> Thread K </th>
</tr>
<tr>
<td>

```c
1. wait( mutex )
2. count++
3. if count == n
4.     post( turnstile )
5. end if
6. post( mutex )
7. wait( turnstile )
8. post( turnstile )
9. [critical point]
10. wait( mutex )
11. count--
12. if count == 0
13.     wait( turnstile )
14. end if
15. post( mutex )
```
</td>
</tr>
</table>

**WRONG** We fixed the problems in the previous case with `mutex`. It is possible for a thread to run before the critical section (e.g. running in a loop) while others have past the critical section.

<table>
<tr>
<th> Thread K </th>
</tr>
<tr>
<td>

```c
1. wait( mutex )
2. count++
3. if count == n
4.     wait( turnstile2 )
5.     post( turnstile1 )
6. end if
7. post( mutex )
8. wait( turnstile1 )
9. post( turnstile1 )
10. [critical point]
11. wait( mutex )
12. count--
13. if count == 0
14.     wait( turnstile1 )
15.     post( turnstile2 )
16. end if
17. post( mutex )
18. wait( turnstile2 )
19. post( turnstile2 )
```
</td>
</tr>
</table>

**CORRECT** It is a **two-phase barrier** because all threads have to wait twice.