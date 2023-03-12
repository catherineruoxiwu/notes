# Greedy

## Q1: Job Scheduling
Input: n jobs, with processing time [t(1), ..., t(n)]<br>
Output: an ordering minimizes the sum of the completions times T
### Algorithm
return the jobs in non-decreasing processing times
### Correctness
(Proof by Contrapositive)
1. Let L = [e_1, ..., e_n] be a permutation of [1, ..., n]. Supporse L is not in non-decreasing order of processing times. => There exists e_i and e_i+1 s.t. t(e_i) > t(e_i+1).
2. Then sum of the completion times of L is nt(e_1) + (n-1)t(e_2) + ... + t(e_n). The contribution of e_i and e_i+1 is (n-i+1)t(e_i) + (n-i)t(e_i).
3. Switch e_i and e_i+1 to get a permutation L'. After the switch, the contribution of e_i and e_i+1 becomes (n-i)t(e_i) + (n-i+1)t(e_i+1). Thus, T(L') - T(L) = t(e_i+1) - t(e_i) < 0, T(L') < T(L). => L is not optimal. => Our algorithm is correct.
### Runtime
O(n log n) for sorting

## Q2: Interval Scheduling
Input: n intervals with I_1 = [s_1, f_1], ..., I_n = [s_n, f_n]<br>
Output: a choice of T of intervals that do not overlap and has max cardinality
### Algorithm
```
T <- []
sort I by non-decreasing finish time
for k = 1...n do
    if I_k does not overlap the last entry in T
        append I_k to T
```
### Correctness
(Proof by Contradiction + Proof by Induction)
1. Let T = [x_1 < ... < x_l] be the output of the algorithm and S = [y_1 < ... < y_p] be an optimal solution. Both T and S are sorted by increasing finish time.
2. We first need to prove that, for all 1 <= k <= l, [x_1 < ... < x_k < y_k+1 < ... < y_p] is still an optimal solution and finish(x_k) <= finish(y_k).
    - Base case: Since the algorithm always chooses the interval with the earliest end time, finish(x_1) <= finish(y_1) < start(y_2). y_1 and y_2 are disjoint then x_1 and y_2 must also be disjoint. Therefore, [x_1 < y_2 < ... < y_p] is still a valid solution.
    - Inductive hypothesis: assume the claim is true for all c >= 1.
    - Inductive step: Since [x_1 < ... < x_c < y_c+1 < ... < y_p] is a valid solution with finish(x_c) <= finish(y_c), we have finish(x_c+1) <= finish(y_c+1) as our greedy algorithm always choose the interval with earliest finish time that has no overlaps with the previous selected intervals. Since finish(x_c+1) <= finish(y_c+1), replacing y_c+1 with x_c+1 in [x_1 < ... < x_c < y_c+1 < ... < y_p] still gives a set of non-overlapping intervals.
3. We also need to prove T and S contain the same number of elements. Suppose the greedy solution is not optimal, l < p. Since finish(x_l) <= finish(y_l) < start(y_l+1), y_l+1 does not overlap with the set output by the greedy algorithm, so it should have been added to the solution. By contradiction, l >= p.

## Q3: Minimizing Lateness
Input: 