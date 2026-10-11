# 6a-Distinct
## Notes
- I know the lesson says sorting. But using a set is far more optimal here in Python.
- In C, you would need to sort it. But in Python, it's far more optimal to just let CPython handle it:
	- A Python set ignores duplicates, so we'll be left with a collection of unique elements. Then we just take the set's length
- CPython will have an efficient C, non-Python way of converting a Python list into a Python set, which likely avoids using Python loops for efficient C loops.
## Problem
Write a function

    def solution(A)
    content_copy

that, given an array A consisting of N integers, returns the number of distinct values in array A.

For example, given array A consisting of six elements such that:

 A[0] = 2    A[1] = 1    A[2] = 1
 A[3] = 2    A[4] = 3    A[5] = 1
content_copy

the function should return 3, because there are 3 distinct values appearing in array A, namely 1, 2 and 3.

Write an efficient algorithm for the following assumptions:

        N is an integer within the range [0..100,000];
        each element of array A is an integer within the range [−1,000,000..1,000,000].

## Code
```python
def solution(A):
    # I know the lesson is sorting.
    # But an optimal solution is to use a set, and count its length.
    # Sorting: O(nlogn)
    # Set: O(n)

    return len(set(A))
```
