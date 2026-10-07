# 4d-MissingInteger
## Notes
- I decided to try go full in on performance testing within the time
- Took 1hr without knowing the best way to go performance testing on Codify interview questions
- Below, I added debug mode, which MUST BE DISABLED ON REAL RUN
- And a way to pick between solutions

## Code
```python
# you can write to stdout for debugging purposes, e.g.
# print("this is a debug message")

import time
import logging

DEBUG = False
if DEBUG:
    logging.basicConfig(level=logging.DEBUG)
    log = logging.getLogger("test")

def mean(ls):
    return sum(ls) / len(ls)

# print(list(range(1, 501))) # Worst Testcase
NORMAL_TIME = mean([3.338e-06, 4.053e-06, 3.338e-06, 3.576e-06]) # Test [1, 3, 6, 4, 1, 2]
SOL_0_TIME = mean([2.551e-05, 2.551e-05, 2.48e-05, 2.456e-05]) # Worst Case Test Scenario (the max space I was given to print before FILE_TOO_BIG error)
SOL_1_TIME = mean([3.862e-05, 3.862e-05, 3.839e-05, 4.005e-05, 3.91e-05])

def eval_times(t0, t1):
    solution_time = t1-t0
    log.debug(f"Time: {solution_time:.4}s")
    log.debug(f"NORMAL Improvement: {NORMAL_TIME / solution_time:.4}x")
    log.debug(f"SOL_0 Improvement: {SOL_0_TIME / solution_time:.4}x") # About 1.5x quicker than solution 1
    log.debug(f"SOL_1 Improvement: {SOL_1_TIME / solution_time:.4}x")

SOLUTION = 0
def solution(A):
    if DEBUG:
        log.debug("SET DEBUGGING TO FALSE!!")
        t0 = time.time()

    match SOLUTION:
        case 0: # Set
            answer = len(A) + 1
            A_set = set(A) # O(n)
            for i in range(1, len(A)+1): # O(n)
                try:
                    A_set.remove(i)
                except KeyError:
                    answer = i
                    break

            if DEBUG:
                t1 = time.time()
                eval_times(t0, t1)

            return answer # Overall: O(n^2) - Second n may exit early
        
        case 1: # Semi-Ordered List
            answer = len(A) + 1
            B = [0] * len(A)
            for (i, num) in enumerate(A): # O(n)
                try:
                    if num < 1:
                        continue
                    B[num - 1] = num
                except IndexError:
                    continue

            # Find first 0 - O(n)
            for i in range(len(A)):
                if B[i] == 0:
                    answer = i+1
                    break
            
            if DEBUG:
                t1 = time.time()
                eval_times(t0, t1)

            return answer # Overall: O(n^2) - Second n will likely exit early

        case _:
            return -1
```