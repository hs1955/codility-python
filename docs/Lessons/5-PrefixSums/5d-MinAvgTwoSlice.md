# 5d-MinAvgTwoSlice
## Notes
## Problem
A non-empty array A consisting of N integers is given. A pair of integers (P, Q), such that 0 ≤ P < Q < N, is called a slice of array A (notice that the slice contains at least two elements). The average of a slice (P, Q) is the sum of A[P] + A[P + 1] + ... + A[Q] divided by the length of the slice. To be precise, the average equals (A[P] + A[P + 1] + ... + A[Q]) / (Q − P + 1).

For example, array A such that:

    A[0] = 4
    A[1] = 2
    A[2] = 2
    A[3] = 5
    A[4] = 1
    A[5] = 5
    A[6] = 8
content_copy

contains the following example slices:

        slice (1, 2), whose average is (2 + 2) / 2 = 2;
        slice (3, 4), whose average is (5 + 1) / 2 = 3;
        slice (1, 4), whose average is (2 + 2 + 5 + 1) / 4 = 2.5.

The goal is to find the starting position of a slice whose average is minimal.

Write a function:

    class Solution { public int solution(int[] A); }
    content_copy

that, given a non-empty array A consisting of N integers, returns the starting position of the slice with the minimal average. If there is more than one slice with a minimal average, you should return the smallest starting position of such a slice.

For example, given array A such that:

    A[0] = 4
    A[1] = 2
    A[2] = 2
    A[3] = 5
    A[4] = 1
    A[5] = 5
    A[6] = 8
content_copy

the function should return 1, as explained above.

Write an efficient algorithm for the following assumptions:

        N is an integer within the range [2..100,000];
        each element of array A is an integer within the range [−10,000..10,000].

## Code
### Attempt 3
```python
def solution(A):
    min_avg = float("inf")
    ans = -1

    # We only need to calculate the average of pairs and triplets
    # We can do so as we step forward

    for i in range(len(A)-2):
        pair_sum = A[i]+A[i+1]
        pair_avg = pair_sum / 2
        if pair_avg < min_avg:
            min_avg = pair_avg
            ans = i
        
        triplet_sum = pair_sum+A[i+2]
        triplet_avg = triplet_sum / 3
        if triplet_avg < min_avg:
            min_avg = triplet_avg
            ans = i

    final_pair_sum = A[-2]+A[-1]
    final_pair_avg = final_pair_sum / 2
    if final_pair_avg < min_avg:
        min_avg = final_pair_avg
        ans = len(A) - 2

    return ans

```
### Attempt 2
- This fails correctness tests
- So the optimal slice length is not always 2. Turns out its either 2 or 3. So I was halfway there.
> [!important] Key Rule
> The key rule is that an average of many numbers is equivalent to the weighted average of 2 smaller averages within these numbers.
- But you must factor in which averages we are even allowed to break down into. The problem restricts this significantly
	- Obviously an average of just 1 number would yield the minimum number of the set. But the problem doesn't let us use an average of just 1 number.
	- An average between 2 numbers is the smallest size we can use
	- An average between 4 numbers, can be represented with a weighted average between the 1st 2 numbers, and the 2nd 2 numbers (strictly)
		- $$\begin{align}
		\text{Avg}&=\frac{a+b+c+d}{4} \\
		&=\frac{2\frac{a+b}{2}+2\frac{c+d}{2}}{4}
		\end{align}$$
		- It's important to notice that while we can break down this set into 3 pairs:
			- $$(a,b),(b,c),(c,d)$$
			- The rule states that we can only break down the average into smaller averages:
				- $$(a),(b,c),(d)$$
				- Can be used to form the weighted average, but we are not allowed single number averages, so this combo is not allowed. This is important in the case of 3 numbers.
		- Thus, checking a size of 4 is redundant. We only need to check the pairs.
	- An average between 3 numbers also has potential for being the smallest. There isn't any valid way to break 3 numbers down into 2 averages which the problems allows for - which is key:
		- Say:
		- $$\begin{align}
		\text{Avg}&=\frac{a+b+c}{3} \\
		&=\frac{2\frac{a+b}{2}+c}{3} \\
		&=\frac{a+2\frac{b+c}{2}}{3} \\
		&=\frac{b+2\frac{a+c}{2}}{3}
		\end{align}$$
		- Sure these are all equal. But we cannot use an average of just 1 number in the context of the problem (nor is the pair $(a,c)$ even allowed) - meaning we have **no way of breaking down an average of 3 numbers** into 2 averages within the context of the problem.
		- Thus an average between 3 numbers could be our minimum
		- And sure, we can form valid 2 pairs from 3 numbers. But can the averages of these pairs be used to form the weighted average of 3 numbers?
			- $$\begin{align}
			&(a,b), (b,c) \\
			\text{Avg}&=\frac{a+b+c}{3} \\
			&\ne \frac{2\frac{a+b}{2}+2\frac{b+c}{2}}{3} \\
			&= \frac{a+2b+c}{3}
			\end{align}$$
			- You cannot break down 3 numbers into 2 averages that fit the problem spec, and can form the weighted average of those 3 numbers.
			- Thus our min average can be between 3 numbers.
	- An average of 5 numbers can be represented by the weighted average of a pair's average and a triple's average (which are allowed within the context of the problem):
		- $$\begin{align}
		\text{Avg}&=\frac{a+b+c+d+e}{5} \\
		&=\frac{3\frac{a+b+c}{3}+2\frac{d+e}{2}}{5} \\
		&=\frac{2\frac{a+b}{2}+3\frac{c+d+e}{3}}{5}
		\end{align}$$
		- Thus, checking a size of 5 is redundant.
	- A similar argument can be made for any number 6+. They can be broken down into pairs and triples that fit are allowed within the problem. Thus are made redundant and not worth checking.
	- Thus we only need to check pairs and triples.
- `10001` is a poor choice of number. I should've used `float("inf")`
```python
from itertools import islice

def solution(A):
    N = len(A)
    A_sum = [0] * (N + 1)
    for (i, num) in enumerate(A):
        A_sum[i+1] = A_sum[i] + num

    ans = -1
    min_avg = 10001
    
    # We only need to check all consecutive pairs, and nothing else
    # We can do so efficiently with iterators
    A_sum_i = islice(A_sum, N)
    A_sum_j = islice(A_sum, 2, None)
    for idx, (i, j) in enumerate(zip(A_sum_i, A_sum_j)):
        # slice_sum = i - j
        avg = (j - i) # / 2 # Don't actually have to / 2 we just want a minimum nominator as the denominator is always the same
        # print("idx = ", idx)
        # print("i = ", i)
        # print("j = ", j)
        # print("avg = ", avg)
        
        if avg < min_avg:
            min_avg = avg
            ans = idx
            # print("min_avg = ", min_avg)
            # print("ans = ", ans)
    
    return ans
```
### Attempt 1
- The **nested loop is a bad sign**, but how to get rid if it?
	- The question does talk about taking averages across ranges of several sizes
	- Let's say we take the range of 3 numbers, and it's a low number.
	- But would it be the lowest average? Surely if we took 2 numbers in there and did an average of them, it would be lower - right?
	- Well we cannot take the average of numbers 1 and 3 directly.
	- Only 1 and 2, and 2 and 3.
	- And what if we had 5 numbers: `[-10000, 2, 0, 2, -10000]`
		- Well the averages of each pair: `[-4999, 1, 1, -4999]`
			- `-4999` is pretty low
			- But would an average of all 5 numbers be less?
				- No. `−3999.2`
				- Clearly, the solution is to just check all consecutive pairs only, but why?
#### Test Cases
```txt
[-10000, -10000]
[-10000, 10000]
[10000, 10000]
```
#### Code
```python
def solution(A):
    N = len(A)
    A_sum = [0] * (N + 1)
    for (i, num) in enumerate(A):
        A_sum[i+1] = A_sum[i] + num

    # We have several potential slices we'd need to check
    # In fact, we have:
    # For the 1st num, there's N-1 values it can pair with
    # For the 2nd num, there's N-2 values it can pair with (cannot re-pair with an already paired num)
    # So a total of N-1 + N-2 + ... + 1 pairs
    # For any pair of nums, there's at least N-2 values it can form triples with
    #  so a total of sum_N-1(N-i) * sum_N-2(N-i) triples.
    # That's an insane number of combos - too many ranges too check 1 by 1
    # Not sure what to do about that - do we cache calculated averages?
    # Like a cumulative sum AND cumulative average? Maybe...

    # How to calculate the average of a slice with A_avg
    # Can't really do it without turning A_avg back into A_sum
    # A     =    [4, 2,   2,    5,    1,    5,    8]
    # A_Sum = [0, 4, 6,   8,   13,   14,   19,   27]
    # A_Avg = [0, 4, 3, 8/3, 13/4, 14/5, 19/6, 27/7]
    #
    # A[1:2+1] = [2, 2]
    # slice_sum = 4 = 8 - 4 = A_Sum[3] - A_Sum[1]
    # slice_avg = 2 = slice_sum/slice_len
    # A_Avg[3] = 13/4
    # A_Sum[1] = 4
    # There's no simple way to obtain slice_avg here, apart from reverting the averages back into sums

    ans = -1
    min_avg = 10001
    for i in range(N-1):
        for j in range(i+1, N):
            slice_sum = A_sum[j+1] - A_sum[i]
            slice_avg = slice_sum / (j+1 - i)
            
            if slice_avg < min_avg:
                min_avg = slice_avg
                ans = i
    
    return ans
```
