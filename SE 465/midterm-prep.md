# SE 465 Midterm Notes

- MAY MUST examples

## Lecture 1: Introduction
1. Bugs are **prevalent** and **costly**.
2. Recent bugs (4): log4shell (logging library for Java; allows remote code execution) (2021); Heartbleed (no bound check of ; a widely used security protocol library; affected CRA) (2014); Amazon Prime Day Down (2018) (non-functional requirement; scalability); British Airways (2019) (IT outage)
3. How software fails? (6): crash (mobile app aborts, core dump); hang (never terminates); data corruption (corrupted resources, such as ill-formed files, and corrupted file systems); incorrect functionality (does not behave as expected); performance degradation (a long-running server gets slower); security vulnerabilities (buffer overflow, signed integer overflow, null pointer dereference)
4. Why software fails? (4): invalid memory access (segfaults); concurrency bugs (deadlocks, data races, atomicity violation); wrong implementation (does not respect the requirement); memory leaks
5. How to avoid software failures? (7): testing; code review; better design; fewer features; better language and IDE; defensive programming (never trust user input); code reuse
6. Failure is inevitable, so ... (4): disclaim liability; release patches; backup/replicate user data; failure recovery
7. **Testing** (evaluating software by observing its execution) vs. **Debugging** (finding a fault given a failure)

## Lecutre 2: Fault, Error, and Failure
1. **Failure** (symptom): an unacceptable behaviour exhibited by a system (observed **externally**)
    - termination of the ability of a system to perform a required function (crash)
    - or its inability to perform within previously specified limits (hang)
    - an externally visible deviation from the system’s specification
2. **Defect/Bug/Fault** (static)
    - incorrect step, process, or data definition in a computer program (needs to be triggered by certain inputs)
    - defect in a hardware device or component
    - defect in a system or a representation of a system that if executed/activated could potentially result in an error
3. **Error** (runtime state): incorrect **internal state** that is the manifestation of some fault; not necessarily lead to failures
4. RIP Model: three conditions must be present for a failure to occur
    - **reachability**: the lines of the fault should be executed during runtime
    - **infection**: the execution of the fault induces an error state
    - **propagation**: the error state propagates to cuase incorrect observable program output
5. Addressing faults at different stages (3): fault avoidance (better design, better PL; for example, buffer overflow is impossible in Java); fault detection (testing; verification e.g. <ins>**CompCert**</ins>); fault tolerance (redundancy/replication; isolation)

## Lecture 3: Control Flow Graph
1. A **basic block** is a sequence of statements that has only **one entry** and **one exit**.
2. Steps in compilation (6): lexing (characters -> tokens); parsing (-> syntax tree); construct abstract syntax tree (-> better syntax tree); conversion to control flow graph (-> CFG); optimizations (-> better CFG); convert to bytecode/machine code
3. Every executable sequence of statement corresponds to a path in CFG, but not all paths in CFG correspond to executable sequences. These are **infeasible paths**.

## Lecture 4: Structural Coverage
1. A **TR** is the set of specific elements of a software artifact that a test case must stisfy or cover.
2. A **coverage citerion (C)** is a rule or collection or rulles that impose test requirements on a test set.
3. **Coverage**: the test set T satisfies C iff every test requirement tr in TR, at least one t in T satisfies tr.
4. **Coverage level**: the ratio of the number of test requiremnents satisfied by T to the size of TR.
5. Stronger coverage criterion helps find more bugs.

## Lecture 5: Graph Coverage
1. A node is **syntactically** reachable from n_i if there exists a path from n_i to n in the graph.
2. A node n is **semantically** reachable if one of the paths from n_i to n can be reached on some input.
3. **Node Coverage** (NC); **Edge Coverage** (EC); **Edge Pair Coverage** (EPC)
4. A path is **simple** if no nodes appears more than once in the path except that first and last nodes may be the same.
5. A path is **prime** if it is simple and does not appear as a proper subpath of any other simple path.
6. **Prime Path Coverage** (PPC); **Complete Path Coverage** (CPC); **Specified Path Coverage** (SPC)
7. A node n is **syntactically** reachable from ni if there exists a path from ni to n. (BFS, DFS) A node n is **semantically** reachable if one of the paths from ni to n can be reached on some input. (undecidable)


## Lecture 6 & 7: Dataflow Coverage
1. `def(n)` or `def(e)`: set of variables that are defined by node n or edge e
2. `use(n)` or `use(e)`: set of variables that are used by node n or edge e
3. **def-clear** with respect to variable v: a path that does not contain `def(v)` other than at the start or end of the path
4. def of v **reaches** l_j: there is a def-clear path from l_i to l_j s.t. the def at l_i reaches the use at l_j
5. **du-path**: a simple def-clear path with respect to v from a def of v to a use of v
    - `du(n_i, n_j, v)`: the set of du-paths from n_i to n_j
    - `du(n_i, n_j)`: the set of du-paths starting n_i
6. Test criteria
    - **All-defs coverage (ADC)**: every def reaches at least one use; for each set of du-paths S = du (n, v), TR contains at least one path d in S
    - **All-uses coverage (AUC)**: every def reaches all possible uses; for each set of du-paths to uses S = du (ni, nj, v), TR contains at least one path d in S
    - **All-du-paths coverage (ADUPC)**: all du-paths between defs and uses; for each set S = du (ni, nj, v), TR contains every path d in S
7. Interprocedural du pairs
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
8. PPC subsumes ADUPC.

## Lecture 8: Automated Testing & Bug Detection Tools
1. Types of test inputs
    - black box: truly random, common/interesting test patterns
    - grey box: guided by coverage, new inputs should cover new code paths
    - white box: new input should cover new paths; testers have access to the underlying framework, design, and structure of the software
2. Software instrumentation
    - adding extra codes to an application for monitoring some program behaviour
    - **static** (compile time): happens without running the program; the binary has instrumented codes
    - **dynamic** (run time): happens at runtime; the binary do not have instrumented codes; codes are instrumented in the process
3. Tools
    - <ins>**Valgrind memcheck**</ins> (dynamic) (10x+ slowdown)
        - checks (5): illegal reads/writes; uninitialized values; illegal frees; overlapping source and destination blocks; memory leaks
        - does not perform bounds checking for stack or global arrays
        - dynamically emulates a CPU and checks during runtime
    - <ins>**Address sanitizer**</ins> (Asan) (static) (2x slowdown)
        - rewrites relevant memory access at compile-time to call a checking library
        - (in virtual memory) mem is the normal application memory; shadow is the memory that keeps track of meta-data (for each byte addr of mem, shadow contains a discriptor); poisoning a byte addr of mem means writing a special value to corresponding place in shadow
        - does not check uninitialized memory read
    - Memory sanitizer (msan)
        - checks uninitialized memory read
    - Thread sanitizer (tsan)
        - detects data races
    - Undefined behaviour sanitizer (ubsan)
        - detects undefined behavious (e.g. integer overflow)
4. **Fuzz testing**: tries to identify abnormal program behaviours by evaluation how the tested program responds to various inputs
    - challenges (4): finding interesting inputs; exploring the whole system, not just individual tools or functions; reducing the size of test cases; reducing duplication
    - naive fuzzer: random data; pros (easy, fast); issues (relies on luck; may run the same things repeatedly) 
    - generation-based fuzzing: **Csmith**, **Funfuzz**
    - tools that automatically discover clean, interesting test cases that trigger new program states in the targeted binary （mutation-based fuzzing)
        - **American fuzzy loop (AFL)**
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
2. **Static analysis**: the examination of a piece of software without running it
3. **SpotBugs (FindBugs)**
    - an open-source static bytecode analyzer
    - looks for defects based on bug patterns -> group patterns into categories -> assign priority
4. Bug categories (4): correctness; common practice violations; concurrency; performance; security threats
5. **PMD**
    - finds common programming flaws (unused variables, empty catch blocks, unnecessary object creations, ...)
    - includes CPD that fins duplicated codes
    - searches for patterns on Abstract Syntax Tree (AST)
6. **CheckerFramework**
    - an extension to Java's type system
    - based on data-flow analysis on source code AST
    - runs as a plugin of javac
7. **ErrorProne**
    - hooks into standard builds
    - shows mistakes immediately & suggests fixes
    - plugin of javac, pattern-based on AST
8. Other tools: **Coverity**, **Clang static analyzer**, **Facebook infer**

## Lecture 10: Syntax-Based Testing
1. 

## Lecture 11: Mutation Testing
1. Idea: inserting artificial defects (**mutants**) in the codes -> checks if at least one of the test cases fails (testing the tests)
2. Generating mutants automatically
    - mutation operators: rules to apply syntactic changes to the changes
    - real fault based operators: apply changes very similar to defects seen in the past
    - language-specific operators: check for inferitance in Java, pointers in C, etc.
3. Mutation operator
    - arithmetic operator replacement (AOR): replace (+, -, *, /, %) in the production code with an alternative operator
    - relational operator replacement (ROR): replace (<, >, <=, >=, ==, !=) in the production code with an alternative operator
    - conditional operator replacement (COR): replace (&&, ||, &, |, !, ^) in the production code with an alternative operator
    - assignment operator replacement (AOR): replace (=, +=, -=, *=, ...) in the production code with an alternative operator
    - scalar variable replacement (SVR): replace with another variable reference of the same type that is already declared in code
4. **Equivalent mutant**: a mutant M that is functionally equivalent to the original program P
5. **Program equivalence**: two programs always produce the same output on every input
6. **Strongly mutation coverage**: for each mutant m, TR contains a test which strongly kills m
7. Mutants to avoid (3): cannot compile; killed by almost all test cases; equivalent to the original program
8. Speeding-up mutation testing
    - only run the tests that reach the target mutant
    - if a mutant is already killed, no need to execute the remaining test
    - do a subset of the generated mutants (random sampling)
9. Tools
    - Java (PIT, MuJava, Bacterio, Javalanche, Major, Descardes)
    - PHP (Humbug, Infection PHP)
    - JavaScript (Stryker)
    - C# (Nester, VisualMutator)
    - C/C++ (Dextool Mutate, Mutate.py)
10. **Mutation testing** (assess the quality of codes) vs. **mutation-based testing** (create more test cases)
11. Object-oriented operators (6): access modifier change; hiding variable deletion; hiding variable insertion; overriding method deletion; parent constructor deletoin; declaration type change
12. Uninteresting mutants (3): **stillborn** (cannot compile); **trivial** (killed by almost any test case); **equivalent** (indistinguishable from original program)

## Lecture 12: Beliefs, Bug Finding and Coverity
1. Finding bugs requires specifications, but how to get?
    - programming languages come with specifications
    - developers write specifications
    - automated tools (static and dynamic analysis)
2. **Coverity**: find bugs in large programs; a leading company for buliding bug detection tools
3. Without knowing the truty, we can find erros from:
    - contradiction
    - deviance
4. **MUST beliefs**: inferred from acts that imply beliefs code "must" have -> check using internal consistency (infer beliefs at different locations, then cross-check for contradiction)
5. **MAY beliefs**: inferred from acts that imply beliefs code "may" have -> need many examples to separate fact from coincidence -> rank errors by belief confidence
6. Trivial consistency NULL pointers may cause 3 types of errors:
    - check-then-use
    - use-then-check
    - contradiction/redundant checks
7. **Redundancy checking**
    - identity operations (`x = x`, `1 * y`, `x & x`, `x | x`)
    - assignments never read
