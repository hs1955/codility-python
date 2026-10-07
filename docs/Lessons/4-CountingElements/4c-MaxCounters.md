# 4c-MaxCounters
https://app.codility.com/programmers/lessons/4-counting_elements/
## Notes
- Error Branches are SLOW. Do not use error branches for expected input.
- Function Calls are also slow - avoid max and min in loops
- Store values that are repeatedly used into single variables
- Lazily update values where possible

## Problem
You are given N counters, initially set to 0, and you have two possible operations on them:

> - _increase(X)_ − counter X is increased by 1,
> - _max counter_ − all counters are set to the maximum value of any counter.

A non-empty array A of M integers is given. This array represents consecutive operations:

> - if A[K] = X, such that 1 ≤ X ≤ N, then operation K is increase(X),
> - if A[K] = N + 1 then operation K is max counter.

For example, given integer N = 5 and array A such that:

A[0] = 3 A[1] = 4 A[2] = 4 A[3] = 6 A[4] = 1 A[5] = 4 A[6] = 4

the values of the counters after each consecutive operation will be:

(0, 0, 1, 0, 0) (0, 0, 1, 1, 0) (0, 0, 1, 2, 0) (2, 2, 2, 2, 2) (3, 2, 2, 2, 2) (3, 2, 2, 3, 2) (3, 2, 2, 4, 2)

The goal is to calculate the value of every counter after all operations.

Write a function:

> def solution(N, A)

that, given an integer N and a non-empty array A consisting of M integers, returns a sequence of integers representing the values of the counters.

Result array should be returned as an array of integers.

For example, given:

A[0] = 3 A[1] = 4 A[2] = 4 A[3] = 6 A[4] = 1 A[5] = 4 A[6] = 4

the function should return [3, 2, 2, 4, 2], as explained above.

Write an ****efficient**** algorithm for the following assumptions:

> - N and M are integers within the range [1..100,000];
> - each element of array A is an integer within the range [1..N + 1].

## Code
### Attempt 4:
```python
# you can write to stdout for debugging purposes, e.g.
# print("this is a debug message")

def solution(N, A):
    counters = [0] * N
    max_count = 0
    base = 0 # Lazily update counters once we read them

    for op in A:
        if op <= N:
	        idx = op - 1 # This value is used alot (and only on this branch), so store it

            if counters[idx] > base: # Ensure counter is at least base before incrementing
                counters[idx] += 1
            else:
                counters[idx] = base + 1

            if counters[idx] > max_count: # Update max
                max_count = counters[idx]

        else: # op == N + 1
            base = max_count # Every counter must be at least `base` before it can increment

    for i in range(N): # Update remaining counters below base
        if counters[i] < base:
            counters[i] = base

    return counters
```

### Attempt 3
```python
# you can write to stdout for debugging purposes, e.g.
# print("this is a debug message")

def solution(N, A):
    counters = [0] * N
    max_count = 0
    max_count_old = 0

    for op in A:
        if op <= N:
            counters[op - 1] += 1
            max_count = max(max_count, counters[op - 1])
        elif max_count_old == max_count: # Optimization to skip maxing after consecutive max instructions
            continue
        else: # op == N + 1:
            max_count_old = max_count
            for i in range(len(counters)):
                counters[i] = max_count
    
    return counters
```
### Attempt 2
```python
# you can write to stdout for debugging purposes, e.g.
# print("this is a debug message")

import time
# from random import randint

DEBUG = False

def mean(ls):
    return sum(ls) / len(ls)

NORMAL_TIME = mean([5.48e-06, 6.2e-06, 6.2e-06, 6.2e-06, 8.11e-06])
TESTCASE_TIME = mean([0.000156, 0.000282, 0.000159, 0.000156, 0.000159])

# print([randint(1, 9) for i in range(1000)]) # Worst Case Test Input

def solution(N, A):
    if DEBUG:
        print("DISABLE DEBUG!!")
        t0 = time.time()

    counters = [0] * N
    max_count = 0
    max_count_old = 0

    for op in A:
        try:
            counters[op - 1] += 1
            max_count = max(max_count, counters[op - 1])
        except IndexError:
            if max_count_old == max_count: # Optimization to skip maxing after consecutive max instructions
                continue
            if op == N + 1:
                counters = [max_count] * N # Expensive Operation
                max_count_old = max_count
                continue
            raise IndexError
        
    if DEBUG:
        t1 = time.time()
        solution_time = t1-t0
        print(f"Time: {solution_time:.3}s")
        print(f"NORMAL_TIME: {NORMAL_TIME:.3}s")
        print(f"TESTCASE_TIME: {TESTCASE_TIME:.3}s")

    return counters
```

### Attempt 1
```python
# you can write to stdout for debugging purposes, e.g.
# print("this is a debug message")

import time
DEBUG = False

def mean(ls):
    return sum(ls) / len(ls)

if DEBUG:
    NORMAL_TEST_TIME = mean([6.2e-06, 6.676e-06, 6.199e-06, 6.199e-06, 5.245e-06, 5.007e-06])
    UNOPTIMIZED_TIME = mean([0.237, 0.2315, 0.2513, 0.2493, 0.2511, 0.2324])

# print(f"(1000, {[1001] * 200})") # Expensive Testcase
def solution(N, A):
    if DEBUG:
        t0 = time.time()

    counters = [0] * N
    max_count = 0
    max_count_old = 0
    # max_changed = False

    for op in A: # O(len(A))
        try:
            counters[op - 1] += 1 # Reduce Try code to just this line for max speed
            max_count = max(max_count, counters[op - 1]) # Line Required to determine change to max.
            # max_changed = True
        except IndexError:
            if max_count == max_count_old: # Optimization to skip expensive operation when unchanged
                continue
            if op == N + 1:
                max_count_old = max_count # Reset
                counters = [max_count] * N # New List * N == Expensive Operation
                continue
            raise IndexError

    if DEBUG:
        t1 = time.time()
        solution_time = t1-t0
        print(f"Time: {solution_time:.4}s")
        print(f"Time: {NORMAL_TEST_TIME / solution_time:.4}x improvment from normal time")
        print(f"Improvment: {UNOPTIMIZED_TIME / solution_time:.4}x improvment from unoptimized time")

    return counters
```