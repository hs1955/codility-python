# 5b-CountDiv
## Notes
- 0 is divisible by all numbers (and the answer is 0 - but that is irrelevant to this question)
## Problem
Write a function:

    def solution(A, B, K)
    content_copy

that, given three integers A, B and K, returns the number of integers within the range [A..B] that are divisible by K, i.e.:

    { i : A ≤ i ≤ B, i mod K = 0 }

For example, for A = 6, B = 11 and K = 2, your function should return 3, because there are three numbers divisible by 2 within the range [6..11], namely 6, 8 and 10.

Write an efficient algorithm for the following assumptions:

        A and B are integers within the range [0..2,000,000,000];
        K is an integer within the range [1..2,000,000,000];
        A ≤ B.

## Code
### Attempt 2
#### Test Inputs
```txt
[0, 0, 1]
[0, 1, 1]
[0, 1, 2]
[0, 1, 2000000000]
[0, 2000000000, 1]
[0, 2000000000, 2000000000]
[10, 20, 5]
[10, 20, 6]
[10, 20, 7]
```
#### Code
```python
# you can write to stdout for debugging purposes, e.g.
# print("this is a debug message")

def solution(A, B, K):
    # 1. Find 1st divisible number in range, or return 0 (0 is divisible by all numbers)
    for i in range(A, B+1):
        if i % K == 0:
            # 2. All divisible numbers can be obtained by just adding K to this i
            # 3. Determine the number of times K can fit between i and B (inclusive)
            return 1 + (B - i) // K
    
    return 0
```
### Attempt 1
- It was quick, but had correctness issues
```python
# you can write to stdout for debugging purposes, e.g.
# print("this is a debug message")

import logging
# logging.basicConfig(level=logging.DEBUG)

def solution(A, B, K):
    # Optimization: B must be bigger than K to have any divisible number
    if B < K:
        return 0
    
    # Anything below K will not be divisible by K
    start = K if (A < K) else A
    # Once we find a divisor. We could just add K, and immediately find the next divisor
    # No need to check all the range
    # In fact, we just need to determine how many times we can add K within the space left
    # An integer divide will suffice
    for i in range(start, B+1):
        if i % K == 0:
            return 1 + ((B - i) // K)
    
    return 0
```
