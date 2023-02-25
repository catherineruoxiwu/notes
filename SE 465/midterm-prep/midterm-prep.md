# SE 465 Midterm Notes

## Lecture 1: Introduction

## Lecutre 2: Fault, 

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

## Lecture 8: Automated Testing & Bug Detection Tools
1. types of test inputs
    - black box: truly random
    - grey box: guided by coverage, new inputs should cover new code paths
    - white box: new input should cover new paths; testers have access to the underlying framework, design, and structure of the software
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

## Lecture 9: Static Analysis Tools
1. "goto fail"
    - to detect: compiler warning
    - to avoid: do not use goto; use `{` and `}`; format codes
2. **static analysis**: the examination of a piece of software without running it
3. **spotBugs (findBugs)**
    - an open-source static bytecode analyzer
    - looks for defects based on bug patterns -> group patterns into categories -> assign priority
4. bug categories (4): correctness; common practice violations; concurrency; performance; security threats
5. **PMD**
    - finds common programming flaws (unused variables, empty catch blocks, unnecessary object creations, ...)
    - includes CPD that fins duplicated codes
    - searches for patterns on Abstract Syntax Tree (AST)
6. **checkerFramework**
    - an extension to Java's type system
    - based on data-flow analysis on source code AST
    - runs as a plugin of javac
7. **errorProne**
    - hooks into standard builds
    - shows mistakes immediately & suggests fixes
    - plugin of javac, pattern-based on AST

## Lecture 10: Syntax-Based Testing
1. 

## Lecture 11: Mutation Testing
1. idea: inserting artificial defects (**mutants**) in the codes -> checks if at least one of the test cases fails (testing the tests)
2. generating mutants automatically
    - mutation operators: rules to apply syntactic changes to the changes
    - real fault based operators: apply changes very similar to defects seen in the past
    - language-specific operators: check for inferitance in Java, pointers in C, etc.
3. mutation operator
    - arithmetic operator replacement (AOR)
    - relational operator replacement (ROR)
    - conditional operator replacement (COR)
    - assignment operator replacement (AOR)
    - scalar variable replacement (SVR)
4. **equivalent mutant**: a mutant M that is functionally equivalent to the original program P
5. **program equivalence**: two programs always produce the same output on every input
6. **strongly mutation coverage**: for each mutant m, TR contains a test which strongly kills m
7. mutants to avoid (3): cannot compile; killed by almost all test cases; equivalent to the original program
8. speeding-up mutation testing
    - only run the tests that reach the target mutant
    - if a mutant is already killed, no need to execute the remaining test
    - do a subset of the generated mutants (random sampling)
9. tools
    - Java (PIT, MuJava, Bacterio, Javalanche, Major, Descardes)
    - PHP (HUmbug, Infection PHP)
    - JavaScript (Stryker)
    - C# (Nester, VisualMutator)
    - C/C++ (Dextool Mutate, Mutate.py)
10. **mutation testing** (assess the quality of codes) vs. **mutation-based testing** (create more test cases)


