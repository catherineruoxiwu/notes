# Concurrency and Synchronization

## Concepts
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

## Examples
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
<br />
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

## The Producer-Consumer Problem
1. Idea: two threads share a common buffer of the same size; one generates data and the other one reads

2. Producer blocks when the buffer is full and consumer blocks when the buffer has zero element in it.

<table>
<tr>
<th> Producer </th>
<th> Consumer </th>
</tr>
<tr>
<td>

```c
1. [produce item]
2. added = false
3. while added is false
4.     wait( mutex )
5.     if count < BUFFER_SIZE
6.         [add item to buffer]
7.         count++
8.         added = true
9.     end if
10.    post( mutex )
11. end while
```
</td>
<td>

```c
1. removed = false
2. while removed is false
3.     wait( mutex )
4.     if count > 0
5.         [remove item from buffer]
6.         count--
7.         removed = true
8.     end if
9.     post( mutex )
10. end while
11. [consume item]
```
</td>
</tr>
</table>

**CORRECT** But inefficient because of busy waiting.

<table>
<tr>
<th> Producer </th>
<th> Consumer </th>
</tr>
<tr>
<td>

```c
1. [produce item]
2. wait( spaces )
3. [add item to buffer]
4. post( items )
```
</td>
<td>

```c
1. wait( items )
2. [remove item from buffer]
3. post( spaces )
4. [consume item]
```
</td>
</tr>
</table>

There are two general semaphores: `items` (starts at 0, represents how many spaces in the buffer are full); `spaces` (starts at `BUFFER_SIZE`, represents the number of spaces in the buffer that are empty)
<br />
**WRONG** There might be context switches during producer/consumer adding/removing item from buffer.

<table>
<tr>
<th> Producer </th>
<th> Consumer </th>
</tr>
<tr>
<td>

```c
1. [produce item]
2. wait( spaces )
3. wait( mutex )
4. [add item to buffer]
5. post( mutex )
6. post( items )
```
</td>
<td>

```c
1. wait( items )
2. wait( mutex )
3. [remove item from buffer]
4. post( mutex )
5. post( spaces )
6. [consume item]
```
</td>
</tr>
</table>

**CORRECT** We add `mutex`, a binary semaphore that is initialized to 1, to protect the critical section.

<table>
<tr>
<th> Producer </th>
<th> Consumer </th>
</tr>
<tr>
<td>

```c
1. [produce item]
2. wait( mutex )
3. wait( spaces )
4. [add item to buffer]
5. post( items )
6. post( mutex )
```
</td>
<td>

```c
1. wait( mutex )
2. wait( items )
3. [remove item from buffer]
4. post( spaces )
5. post( mutex )
6. [consume item]
```
</td>
</tr>
</table>

**WRONG** A counterexample: a producer might be stuck between 2 and 3, waiting for `spaces` while no consumers can be executed because they are blocked on `mutex`. 

```c
pthread_mutex_init( pthread_mutex_t *mutex, pthread_mutexattr_t *attributes );
pthread_mutex_lock( pthread_mutex_t *mutex ); /* blocking */
pthread_mutex_trylock( pthread_mutex_t *mutex ); /* Returns 0 on success; non-blocking */
pthread_mutex_unlock( pthread_mutex_t *mutex );
pthread_mutex_destroy( pthread_mutex_t *mutex );

int buffer[BUFFER_SIZE];
int pindex = 0;
int cindex = 0;
sem_t spaces;
sem_t items;
pthread_mutex_t mutex;

int produce( int id ) {
    int r = rand();
    printf("Producer %d produced %d.\n", id, r);
    return r;
}

void consume( int id, int number ) {
    printf("Consumer %d consumed %d.\n", id, number);
}

void* producer( void* arg ) {
    int* id = (int*) arg;
    for(int i = 0; i < 10000; ++i) {
        int num = produce(*id);
        sem_wait( &spaces );
        pthread_mutex_lock( &mutex );
        buffer[pindex] = num;
        pindex = (pindex + 1) % BUFFER_SIZE;
        pthread_mutex_unlock( &mutex );
        sem_post( &items );
    }
    free( arg );
    pthread_exit( NULL );
}

void* consumer( void* arg ) {
    int* id = (int*) arg;
    for(int i = 0; i < 10000; ++i) {
        sem_wait( &items );
        pthread_mutex_lock( &mutex );
        int num = buffer[cindex];
        buffer[cindex] = -1;
        cindex = (cindex + 1) % BUFFER_SIZE;
        pthread_mutex_unlock( &mutex );
        sem_post( &spaces );
        consume( *id, num );
    }
    free( id );
    pthread_exit( NULL );
}

int main( int argc, char** argv ) {
    sem_init( &spaces, 0, BUFFER_SIZE );
    sem_init( &items, 0, 0 );
    pthread_mutex_init( &mutex, NULL );
    pthread_t threads[20];
    for( int i = 0; i < 10; i++ ) {
        int* id = malloc(sizeof(int));
        *id = i;
        pthread_create(&threads[i], NULL, producer, id);
    }
    for( int j = 10; j < 20; j++ ) {
        int* jd = malloc(sizeof(int));
        *jd = j-10;
        pthread_create(&threads[j], NULL, consumer, jd);
    }
    for( int k = 0; k < 20; k++ ){
        pthread_join(threads[k], NULL);
    }
    sem_destroy( &spaces );
    sem_destroy( &items );
    pthread_mutex_destroy( &mutex );
    pthread_exit( 0 );
}
```

## The Readers-Writers Problem
1.  Idea: concurrently reading and modification of a data structure or record by more than one thread
2. Any number of readers may enter the critical section simultaneously; only one writer is allowed in the critical section.

<table>
<tr>
<th> Writer </th>
<th> Reader </th>
</tr>
<tr>
<td>

```c
1. wait( roomEmpty )
2. [write data]
3. post( roomEmpty )
```
</td>
<td>

```c
1. wait( mutex )
2. readers++
3. if readers == 1
4.     wait( roomEmpty )
5. end if
6. post( mutex )
7. [read data]
8. wait( mutex )
9. readers--
10. if readers == 0
11.     post( roomEmpty )
12. end if
13. post( mutex )
```
</td>
</tr>
</table>

`mutex` and `roomEmpty` are binary semaphores. This pattern is the **light switch** (the first one into the room turns on the lights and the last one
out turns them off).
<br />
**WRONG** Starvation of the writer thread might occur. 

<table>
<tr>
<th> Writer </th>
<th> Reader </th>
</tr>
<tr>
<td>

```c
1. wait( turnstile )
2. wait( roomEmpty )
3. [write data]
4. post( turnstile )
5. post( roomEmpty )
```
</td>
<td>

```c
1. wait( turnstile )
2. post( turnstile )
3. wait( mutex )
4. readers++
5. if readers == 1
6. wait( roomEmpty )
7. end if
8. post( mutex )
9. [read data]
10. wait( mutex )
11. readers--
12. if readers == 0
13. post( roomEmpty )
14. end if
15. post( mutex )
```
</td>
</tr>
</table>

When a writer arrives, any readers currently reading should finish and no new readers should be allowed to enter.
<br />
**CORRECT** But not optimal. The solution does not give writers any particular priority.

<table>
<tr>
<th> Writer </th>
<th> Reader </th>
</tr>
<tr>
<td>

```c
1. wait( writeMutex )
2. writers++
3. if writers == 1
4.     wait( noReaders )
5. end if
6. post( writeMutex )
7. wait ( noWriters )
8. [write data]
9. post( noWriters )
10. wait( writeMutex )
11. writers--
12. if writers == 0
13.     post( noReaders )
14. end if
15. post( writeMutex )
```
</td>
<td>

```c
1. wait( noReaders )
2. wait( readMutex )
3. readers++
4. if readers == 1
5.     wait( noWriters )
6. end if
7. post( readMutex )
8. post( noReaders )
9. [read data]
10. wait( readMutex )
11. readers--
12. if readers == 0
13.     post( noWriters )
14. end if
15. post( readMutex )
```
</td>
</tr>
</table>

**CORRECT** `roomEmpty` breaks up to `noReaders` and `noWriters`

```c
pthread_rwlock_init( pthread_rwlock_t * rwlock, pthread_rwlockattr_t * attr );
pthread_rwlock_rdlock( pthread_rwlock_t * rwlock );
pthread_rwlock_tryrdlock( pthread_rwlock_t * rwlock );
pthread_rwlock_wrlock( pthread_rwlock_t * rwlock );
pthread_rwlock_trywrlock( pthread_rwlock_t * rwlock );
pthread_rwlock_unlock( pthread_rwlock_t * rwlock );
pthread_rwlock_destroy( pthread_rwlock_t * rwlock );

pthread_rwlock_t rwlock;

void init( ) {
    pthread_rwlock_init( &rwlock, NULL );
}

void cleanup( ) {
    pthread_rwlock_destroy( &rwlock );
}

void* writer( void* arg ) {
    pthread_rwlock_wrlock( &rwlock );
    write_data( arg );
    pthread_rwlock_unlock( &rwlock );
}

void* reader( void* read ) {
    pthread_rwlock_rdlock( &rwlock );
    read_data( arg );
    pthread_rwlock_unlock( &rwlock );
}
```

3. **The Search-Insert-Delete Problem**
    - **searchers** examine the list; can execute concurrently with other serachers
    - **inserters** add new items to the list; can be done in parallel with search; one insertion may take place at a time
    - **deleters** remove items from anywhere in the list; one deletion may take place at a time; no searchers or inserters can run at the same time
```c
pthread_mutex_t searcher_mutex;
pthread_mutex_t inserter_mutex;
pthread_mutex_t perform_insert;
sem_t no_searchers;
sem_t no_inserters;
int searchers;
int inserters;

void init( ) {
    pthread_mutex_init( &searcher_mutex, NULL );
    pthread_mutex_init( &inserter_mutex, NULL );
    pthread_mutex_init( &perform_insert, NULL );
    sem_init( &no_inserters, 0, 1 );
    sem_init( &no_searchers, 0, 1 );
    searchers = 0;
    inserters = 0;
}

void* searcher_thread( void *target ) {
    pthread_mutex_lock( &searcher_mutex );
    searchers++;
    if ( searchers == 1 ) {
        sem_wait( &no_searchers );
    }
    pthread_mutex_unlock( &searcher_mutex );
    search( target );
    pthread_mutex_lock( &searcher_mutex );
    searchers--;
    if ( searchers == 0 ) {
        sem_post( &no_searchers );
    }
    pthread_mutex_unlock( &searcher_mutex );
}

void* deleter_thread( void* to_delete ) {
    sem_wait( &no_searchers );
    sem_wait( &no_inserters );
    delete( to_delete );
    sem_post( &no_inserters );
    sem_post( &no_searchers );
}

void* inserter_thread( void *to_insert ) {
    pthread_mutex_lock( &inserter_mutex );
    inserters++;
    if ( inserters == 1 ) {
        sem_wait( &no_inserters );
    }
    pthread_mutex_unlock( &inserter_mutex );
    node * insert_after = find_insert_location( );
    pthread_mutex_lock( &perform_insert );
    insert( to_insert, insert_after ); /* Can update its position if needed */
    pthread_mutex_unlock( &perform_insert );
    pthread_mutex_lock( &inserter_mutex );
    inserters--;
    if ( inserters == 0 ) {
        sem_post( &no_inserters );
    }
    pthread_mutex_unlock( &inserter_mutex );
}
```