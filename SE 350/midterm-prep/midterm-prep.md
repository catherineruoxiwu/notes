# SE 350 Midterm Notes

## Processes
1. **PCB**(8): id; state; priority; PC; reg data; memory pointers; I/O status; accounting info
2. **Events lead to process creations**(3): system boot up; user request to start a new proc; one process spawns another
3. **Process destruction**(4): normal exit(v); error exit(v); fatal error(inv); killed by another proc(inv)
4. **Zombie process**: A child process finishes execution, but the parent had not collected the return value yet. The child process is dead but not gone because it has not been cleaned up, and it is still in the PCB list and holds all its resources.
5. **Orphan process**: The child process's parent die before the child does. In UNIX, orphan is automatically adopted by the 'init' process so that the orphans would not become zombies.
6. **Process states**(5): running; ready; blocked; new; terminated 
7. **Process transitions**(8): create(* -> new); admit(new -> ready); dispatch(ready -> running); suspend(running -> ready); exit(running -> terminated); block(running -> blocked); unblock(blocked -> ready); reap(terminated -> *)
8. **Swapping processes to disk**: The OS puts parts of some processes on disk so that it can run more processes at the same time. Swapping is time-consuming.
    - 2 new states: ready/swapped; blocked/swapped
    - admit transition(new -> ready/swapped)
9. 