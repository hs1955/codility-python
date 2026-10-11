# 6b-MaxProductOfThree
## Notes
- In Python 3, `int` is actually unbounded. There are no `INT.MAX` or `INT.MIN`, unlike many other languages.
	- However, you might actually be looking for information about the current interpreter's _[word size](https://en.wikipedia.org/wiki/Word_\(computer_architecture\)#Table_of_word_sizes)_, which will be the same as the machine's word size in most cases. That information is still available in Python 3 as [`sys.maxsize`](https://docs.python.org/3/library/sys.html#sys.maxsize), which is the maximum value representable by a signed word. Equivalently, it's the size of the largest possible list or in-memory sequence.
	- Generally, the maximum value representable by an unsigned word will be `sys.maxsize * 2 + 1`, and the number of bits in a word will be `math.log2(sys.maxsize * 2 + 2)`. See [this answer](https://stackoverflow.com/a/6918334/577088) for more information.
## Problem
A non-empty array A consisting of N integers is given. The product of triplet (P, Q, R) equates to A[P] * A[Q] * A[R] (0 ≤ P < Q < R < N).

For example, array A such that:
  A[0] = -3
  A[1] = 1
  A[2] = 2
  A[3] = -2
  A[4] = 5
  A[5] = 6

contains the following example triplets:

        (0, 1, 2), product is −3 * 1 * 2 = −6
        (1, 2, 4), product is 1 * 2 * 5 = 10
        (2, 4, 5), product is 2 * 5 * 6 = 60

Your goal is to find the maximal product of any triplet.

Write a function:

    def solution(A)

that, given a non-empty array A, returns the value of the maximal product of any triplet.

For example, given array A such that:
  A[0] = -3
  A[1] = 1
  A[2] = 2
  A[3] = -2
  A[4] = 5
  A[5] = 6

the function should return 60, as the product of triplet (2, 4, 5) is maximal.

Write an efficient algorithm for the following assumptions:

        N is an integer within the range [3..100,000];
        each element of array A is an integer within the range [−1,000..1,000].


## Code
### Attempt 3
- How do we know that the largest product is either `largest * largest2 * largest3` or `largest * smallest * smallest2`?
	- Lets forget about signs and positive/negative for a moment. In this case, the largest product we can make, will be the product of the 3 numbers furthest away from 0, which will just be the largest 3 numbers.
	- But when we have negative numbers, then its possible that the product of 2 negative numbers may exceed the product of 2 positive numbers.
	- We have 2 directions in which the absolute value can be high. Though of course, we need 2 negative values to create a positive, so we want the 2 smallest values possible for the best chance of getting a large number.
	- We have calculated the products of 2 sets of numbers at the extremes of the range. Now we just multiply them by the largest number.
	- But if `largest` happens to be negative? Then the sign will invert which pair is the largest.
		- Again, forgetting the signs for a moment, we have 2 sets of numbers with the largest possible absolute values we can get - so it makes sense that multiplying them with our largest number furthest from 0, would get us the largest product furthest away from 0.
		- Including signs again, if `largest` is positive, then `largest * largest2 * largest3` becomes the largest.
		- But if `largest` is negative and our product is positive:
			- Firstly, that means all of our numbers are negative, which means we need the highest negative product
			- Then `smallest * smallest2` multiplies together to give us the largest possible positive number. While `largest2 * largest3` will give us the smallest possible positive number. Then `largest` will be the highest negative number closest to 0, so `largest * largest2 * largest3`  is still our largest product.
		- But what if we have a mix of positive and negative numbers? Like in the case of 4 numbers, `largest3 == smallest2`. Well we can actually just apply the same logic as the bullet point above, but instead of `largest` being positive/negative - `largest` is always positive, and `smallest * smallest2` and `largest2 * largest3` that will be negative and inverting the selected pair. But our answer still lies within this pair.
```python
import sys

def solution(A):
    # I know this problem is about sorting
    # But it would be more efficient to store the 3 largest
    # and 2 most negative/smallest numbers as follows

    # There is no situation where sorting would be quicker than
    # storing just 5 numbers - even in C

    # Int type is unbounded in Python.
    # But these values should be suitable substitutes for int bounds
    # We cannot use an initial element from A as we cannot have a single
    # element of A appear more than once in each largest or smallest sets
    # Likewise, we cannot use the first 3 values, as they need to be sorted
    # once in largest/smallest
    largest = -sys.maxsize
    largest2 = -sys.maxsize
    largest3 = -sys.maxsize

    smallest = sys.maxsize
    smallest2 = sys.maxsize
    
    for n in A:
        if n >= largest: # if n == largest, then we have 2 largest numbers of equal value. We need to run this branch in case of duplicates
            largest3 = largest2
            largest2 = largest
            largest = n
        elif n >= largest2:
            largest3 = largest2
            largest2 = n
        elif n > largest3: # Optimization: if n == largest3, then there's no change
            largest3 = n

        # The 1st number could be the biggest or smallest
        # Ensure n gets a chance at being both. Don't let it get
        # stuck in the largest numbers
        # Because n can enter both smallest and largest, we can
        # expect certain n to appear in both smallest and largest
        # sets when n < 5. But that is fine, we do not double count within
        # each respective set, so we're good
        if n <= smallest:
            smallest2 = smallest
            smallest = n
        elif n < smallest2:
            smallest2 = n

    # print("largest = ", largest)
    # print("largest2 = ", largest2)
    # print("largest3 = ", largest3)
    # print("smallest2 = ", smallest2)
    # print("smallest = ", smallest)
    return max(largest * largest2 * largest3, largest * smallest * smallest2)
```
### Attempt 2
- Somehow, I got full marks on performance, and 1 incorrect correctness test in a simple case. Well that's a first - seems there may be an edge case I haven't accounted for, but overall the logic is sound for large lists.
	- What you return is the bug. What if `largest` is negative? That would flip the ordering of which side is largest/smallest. Include `largest` within the max.
```python
import sys

def solution(A):
    # I know this problem is about sorting
    # But it would be more efficient to store the 3 largest
    # and 2 most negative/smallest numbers as follows

    # There is no situation where sorting would be quicker than
    # storing just 5 numbers - even in C

    # Int type is unbounded in Python.
    # But these values should be suitable substitutes for int bounds
    # We cannot use an initial element from A as we cannot have a single
    # element of A appear more than once in each largest or smallest sets
    # Likewise, we cannot use the first 3 values, as they need to be sorted
    # once in largest/smallest
    largest = -sys.maxsize
    largest2 = -sys.maxsize
    largest3 = -sys.maxsize

    smallest = sys.maxsize
    smallest2 = sys.maxsize
    
    for n in A:
        if n >= largest: # if n == largest, then we have 2 largest numbers of equal value. We need to run this branch in case of duplicates
            largest3 = largest2
            largest2 = largest
            largest = n
        elif n >= largest2:
            largest3 = largest2
            largest2 = n
        elif n > largest3: # if n == largest3, then there's no change
            largest3 = n

        # The 1st number could be the biggest or smallest
        # Ensure n gets a chance at being both. Don't let it get
        # stuck in the largest numbers
        # Because n can enter both smallest and largest, we can
        # expect certain n to appear in both smallest and largest
        # sets when n < 5. But that is fine, we do not double count within
        # each respective set, so we're good
        if n <= smallest:
            smallest2 = smallest
            smallest = n
        elif n < smallest2:
            smallest2 = n

    # print("largest = ", largest)
    # print("largest2 = ", largest2)
    # print("largest3 = ", largest3)

    # print("smallest2 = ", smallest2)
    # print("smallest = ", smallest)
    return largest * max(largest2 * largest3, smallest * smallest2)
```
### Attempt 1
- I didn't consider the cases where the 5 numbers aren't assigned
- And I also incorrectly assign the largest2 and largest3 and smallest2. I assume that the number I encounter next will always bigger/smaller respectively, but there's no reason to believe that
```python
import sys

def solution(A):
    # I know this problem is about sorting
    # But it would be more efficient to store the 3 largest
    # and 2 most negative/smallest numbers as follows

    # There is no situation where sorting would be quicker than
    # storing just 5 numbers - even in C

    # Int type is unbounded in Python
    largest = -sys.maxsize
    largest2 = 0
    largest3 = 0

    smallest = sys.maxsize
    smallest2 = 0
    
    for n in A:
        if n > largest:
            largest3 = largest2
            largest2 = largest
            largest = n

        if n < smallest:
            smallest2 = smallest
            smallest = n

    return largest3 * max(largest2 * largest, smallest2 * smallest)
```
