# SE 465 Midterm Notes

## Lecture 6 & 7: Dataflow Coverage
1. `def(n)` or `def(e)`: set of variables that are defined by node n or edge e
2. `use(n)` or `use(e)`: set of variables that are used by node n or edge e
3. **def-clear** with respect to variable v: a path that does not contain `def(v)` other than at the start or end of the path
4. def of v **reaches** l_j: there is a def-clear path from l_i to l_j s.t. the def at l_i reaches the use at l_j
5. **du-path**: a simple def-clear path with respect to v from a def of v to a use of v
    - `du(n_i, n_j, v)`: the set of du-paths from n_i to n_j
    - `du(n_i, n_j)`: the set of du-paths starting n_i
6. test criteria
    - **All-defs coverage (ADC)**: every def reaches at least one use
    - **All-uses coverage (AUC)**: every def reaches all possible uses
    - **All-defs coverage (ADC)**: all du-paths between defs and uses
7. interprocedural du pairs
```c
/* f: caller */
f() {
    x = 14   /* last-def */
    y = g(x) /* x: actual parameter */
             /* g: interface */
    print(y) /* first-use */
}

/* g: callee */
g(a) {
    print(a) /* first-use */
    b = 42   /* last-def */
    return b
}
```

## Lecture 8 Automated Testing & Bug Detection Tools
1. types of test inputs
    - black box: truly random
    - grey box: guided by coverage, new inputs should cover new code paths
    - white box: new input should cover new paths
2. software instrumentation
    - adding extra codes to an application for monitoring some program behaviour
    - **static** (compile-time): happens without running the program; the binary has instrumented codes
    - **dynamic** (runtime): happens at runtime; the binary do not have instrumented codes; codes are instrumented in the process
3. tools
    - **valgrind memcheck** (dynamic) (10x+ slowdown)
        - checks (5): illegal reads/writes; uninitialized values; illegal frees; overlapping source and destination blocks; memory leaks
        - does not perform bounds checking for stack or global arrays
        - dynamically emulates a CPU and checks during runtime
    - **address sanitizer** (asan) (static) (2x slowdown)
        - rewrites relevant memory access at compile-time to call a checking library
        - (in virtual memory) mem is the normal application memory; shadow is the memory that keeps track of meta-data (for each byte addr of mem, shadow contains a discriptor); poisoning a byte addr of mem means writing a special value to corresponding place in shadow
        - does not check uninitialized memory read
    - memory sanitizer (msan)
        - checks uninitialized memory read
    - thread sanitizer (tsan)
        - detects data races
    - undefined behaviour sanitizer (ubsan)
        - detects undefined behavious (e.g. integer overflow)
4. **fuzz testing**: tries to identify abnormal program behaviours by evaluation how the tested program responds to various inputs
    - challenges (4): finding interesting inputs; exploring the whole system, not just individual tools or functions; reducing the size of test cases; reducing duplication
    - naive fuzzer: random data; pros (easy, fast); issues (relies on luck; may run the same things repeatedly)
    - tools that automatically discover clean, interesting test cases that trigger new program states in the targeted binary
        - **american fuzzy loop (AFL)**
            - security-oriented; compile-time instrumentation
            - passes input in a file (between processes)
        - **libFuzzer**
            - fuzzes the program in the same process
            - requires "fuzz targets" (entry points that accept an array of bytes)
            - executes the fuzz target multiple times with different inputs
