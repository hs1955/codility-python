# 5a-PassingCars
## Notes
- DISABLE THE DEBUGGING
- Read the question very carefully. There does tend to be a specific optimization that can be made in efficiency questions. Like this problems definition of "pair" was specifically anytime `(x,y) == (0,1)`. `(1, 0)` was not allowed, even though in colloquial English it would be considered a pair.
- `itertools` is a standard library and should be used where applicable.
	- Use `itertools.islice` to slice iterators without creating slice copies of the whole list `A`
- We could even just add `A[j]` directly onto pairs, because it is either 0 or 1, and we care about when it's 1. Could remove an if statement
> [!note] Python Code Optimization
> - Python doesn't do much compiler level optimizations like C++ would. This is a deliberate design choice, and it's such that the non-optimized code would only run a few nano-seconds slower.
- We can actually directly see the opcode any python code we want easily. Opcode is kind of an in-between of language and assembly:
> [!def] Opcode
> An **opcode** is a number that identifies one operation in CPython's bytecode instruction set. That's it. It's the "verb" — the thing to do — encoded as an integer.
> - The numeric identifier for an operation, e.g. the number that means "load a local variable" or "call a function." `dis.opmap["LOAD_FAST"]` is literally an integer like `124`
> - It's meant look like a CPU instruction - because opcodes are a data format the C code of Python interprets
> > [!fail] What it's not
> > - Not assembly
> >   - Assembly is the textual form of a _real_ CPU's machine code
> >   - Opcodes here are for a _virtual_ machine with no hardware behind it. There are no real registers, no real program counter, no real decode unit. The "stack" that `LOAD_FAST` pushes onto is a Python list of object pointers managed in C.
> > - Not C
> >   - C is the language that CPython is written in. CPython is the default implementation of Python, which you download from python.org
> >   - ![[IMG-5a-PassingCars-20261006130800684.png]]

> [!def] Instruction
> An opcode plus its argument(s). `LOAD_FAST 3` means "opcode for load-fast, argument 3 (which local slot)."
> - This is what `dis.dis` prints one per line.

> [!def] Bytecode
> The full sequence of instructions that a code object contains. The thing the interpreter walks.
### Seeing how Python Iterators Are Quicker than Index-Loops
```python
import dis
dis.dis("""
for elm in A:
    do_stuff(elm)
""")
```
Representative output (exact names and offsets vary by CPython version):

```text
GET_ITER
FOR_ITER        → end
STORE_FAST      elm
LOAD_FAST       do_stuff
LOAD_FAST       elm
CALL
JUMP_BACKWARD
```
And the index loop:

```python
dis.dis("""
for i in range(n):
    do_stuff(A[i])
""")
```
```text
GET_ITER
FOR_ITER        → end
STORE_FAST      i
LOAD_FAST       do_stuff
LOAD_FAST       A
LOAD_FAST       i
BINARY_SUBSCR
CALL
JUMP_BACKWARD
```
#### What Each One Does
- `GET_ITER` — call `iter()` once, before the loop.
- `FOR_ITER` — advance the iterator; push the next value, or jump past the loop on exhaustion.
- `STORE_FAST i` — pop the top of stack into the local variable `i`.
- `LOAD_FAST A` / `LOAD_FAST i` — push local variables onto the stack.
- `BINARY_SUBSCR` — pop index and container, do `A[i]`, push result.
- `CALL` — call `do_stuff` with the stack arguments.
- `JUMP_BACKWARD` — go back to `FOR_ITER`.

You can see the difference directly: the index loop has `STORE_FAST i`, `LOAD_FAST A`, `LOAD_FAST i`, and `BINARY_SUBSCR` that the iterator loop doesn't. That's the store/reload and the extra subscript you were asking about.
## Problem
A non-empty array A consisting of N integers is given. The consecutive elements of array A represent consecutive cars on a road.

Array A contains only 0s and/or 1s:

> - 0 represents a car traveling east,
> - 1 represents a car traveling west.

The goal is to count passing cars. We say that a pair of cars (P, Q), where 0 ≤ P < Q < N, is passing when P is traveling to the east and Q is traveling to the west.

For example, consider array A such that:

A[0] = 0 A[1] = 1 A[2] = 0 A[3] = 1 A[4] = 1

We have five pairs of passing cars: (0, 1), (0, 3), (0, 4), (2, 3), (2, 4).

Write a function:

> def solution(A)

that, given a non-empty array A of N integers, returns the number of pairs of passing cars.

The function should return −1 if the number of pairs of passing cars exceeds 1,000,000,000.

For example, given:

A[0] = 0 A[1] = 1 A[2] = 0 A[3] = 1 A[4] = 1

the function should return 5, as explained above.

Write an ****efficient**** algorithm for the following assumptions:

> - N is an integer within the range [1..100,000];
> - each element of array A is an integer that can have one of the following values: 0, 1.

## Code
### Attempt 10
```python
# you can write to stdout for debugging purposes, e.g.
# print("this is a debug message")

import math
import itertools

def solution(A):
    # A pair is anywhere where we can make (0, 1) from 2 distinct indicies,
    # where 1 index is always lower than the other
    # For any 0 in A, the number of pairs it can make is the sum of 1s to its right
    # If iterate from right to left, we can keep a running total of pairs

    # For A = [0, 1, 0, 1, 1]
    # For A_rev = iter([1, 1, 0, 1, 0])
    # N A[i] pairs ones
    # 0 1    0     1
    # 1 1    0     2
    # 2 0    2     2
    # 3 1    2     3
    # 4 0    5     3
    # pairs = 5 = sum of 1s at before each 0 in A_rev

    # Optimization: We cannot encounter the exit condition until we've encountered at least sqrt(1e9) numbers
    # Earliest we can encounter max pairs, is a sequence of zeros followed by an equal number of ones
    # Because pairs = zeros * ones = zeros^2. We cannot make any zero worth more than this.
    # Thus, only enact the check after we've encountered SQRT_1e9 values
    SQRT_1e9 = math.ceil(math.sqrt(1e9))
    # SQRT_1e9 = 2 # DEBUG TESTING
    A_rev = None
    A_rev_sqrt1e9 = None
    if len(A) < SQRT_1e9:
        A_rev = reversed(A)
        # A_rev_sqrt1e9 = None
    else:
        A_rev = itertools.islice(reversed(A), SQRT_1e9)
        A_rev_sqrt1e9 = itertools.islice(reversed(A), SQRT_1e9, None)

    pairs = 0
    ones = 0
    # Optimization: While ones == 0, case 0 does nothing, so skip forward to a 1
    for car in A_rev:
        if car == 1:
            ones = 1 # Must set ones here in condition. We may exit this loop without finding a single 1.
            break

    for car in A_rev:
        match car:
            case 0:
                pairs += ones
            case 1:
                ones += 1

    if A_rev_sqrt1e9:
        # Optimization: While ones == 0, case 0 does nothing, so skip forward to a 1
        if ones == 0:
            for car in A_rev_sqrt1e9:
                if car == 1:
                    ones = 1 # Must set ones here in condition. We may exit this loop without finding a single 1.
                    break

        for car in A_rev_sqrt1e9:
            match car:
                case 0:
                    pairs += ones
                    if pairs > 1e9: # Condition from Task Definition
                        return -1
                case 1:
                    ones += 1
    
    return pairs
```
### Attempt 9
- You could've just iterated from right to left instead, and you would have a running total of pairs throughout the run, allowing you to reintegrate the check at large values
- I'm not sure if iterators are actually any more efficient than for loops. But how does an iterator know it's at the end when it doesn't know its length?
	- By putting an END element at the end of the loop
#### Test Inputs
```txt
[0]
[1]
[0, 0]
[0, 1]
[1, 0]
[1, 1]
[0, 0, 1]
[0, 1, 0, 1]
[1, 0, 1, 0]
[1, 0, 0, 1, 1]
```
#### Code
```python
# you can write to stdout for debugging purposes, e.g.
# print("this is a debug message")

def solution(A):
    # Number of unique ways to obtain a pair (0, 1)
    # For any given 0, the number of pairs it can make = sum of 1s to its right
    # But using algebra, we could complete the task in 1 pass,
    # substituting the total 1s at the end of the pass
    # So for [0, 1, 0, 1, 1]. Let the total ones = t
    # 0 pairs=t-0       ones = 0
    # 1 pairs="         ones = 1
    # 2 pairs=t+t-1     ones = 1
    # 3 pairs="         ones = 2
    # 4 pairs="         ones = 3 = t
    # E pairs=2t-1=6-1=5
    # The number of t's at the end is len(A) - t = 5-3 = 2
    # At each 0, pairs+=t-ones at that point
    # At each 1, we increment ones
    # At the end, pairs == zeros * t - many_ones
    #
    # Note we cannot calculate the total pairs in one pass until it is complete,
    # meaning we cannot exit early if our solution exceeds 1e9 pairs

    A_iter = iter(A)
    many_ones = 0
    ones = 0

    # Optimization: While ones==0, case 0 makes no changes
    for car in A_iter:
        if car == 1:
            ones = 1 # We must set ones inside this conditional - we may complete this loop and never encounter 1
            break

    for car in A_iter:
        match car:
            case 0:
                many_ones += ones
            case 1:
                ones += 1

    # Now ones == t (total ones/sum)
    # zeros = len(A) - ones
    # pairs == zeros * t - many_ones
    pairs = (len(A) - ones) * ones - many_ones
    if pairs > 1e9:
        return -1
    return pairs
```

### Attempt 8
- Logic is flawed. You may exit initial skip-0s-until-1 optimization and not encounter a 1, ruining the final calculated pairs in an all zeros array
#### Test Inputs
```txt
[0]
[1]
[0, 0]
[0, 1]
[1, 0]
[1, 1]
[0, 0, 1]
[0, 1, 0, 1]
[1, 0, 1, 0]
[1, 0, 0, 1, 1]
```
#### Code
```python
# you can write to stdout for debugging purposes, e.g.
# print("this is a debug message")

import math

def solution(A):
    # Number of unique ways to obtain a pair (0, 1)
    # For any given 0, the number of pairs it can make = sum of 1s to its right
    # But using algebra, we could complete the task in 1 pass,
    # substituting the total 1s at the end of the pass
    # So for [0, 1, 0, 1, 1]. Let the total ones = t
    # 0 pairs=t-0       ones = 0
    # 1 pairs="         ones = 1
    # 2 pairs=t+t-1     ones = 1
    # 3 pairs="         ones = 2
    # 4 pairs="         ones = 3 = t
    # E pairs=2t-1=6-1=5
    # The number of t's at the end is len(A) - t = 5-3 = 2
    # At each 0, pairs+=t-ones at that point
    # At each 1, we increment ones
    # At the end, pairs == zeros * t - many_ones
    #
    # Note we cannot calculate the total pairs in one pass until it is complete,
    # meaning we cannot exit early if our solution exceeds 1e9 pairs

    A_iter = iter(A)

    # Optimization: While ones==0, case 0 makes no changes
    for car in A_iter:
        if car == 1:
            break

    many_ones = 0
    ones = 1

    for car in A_iter:
        match car:
            case 0:
                many_ones += ones
            case 1:
                ones += 1

    # Now ones == t (total ones/sum)
    # zeros = len(A) - ones
    # pairs == zeros * t - many_ones
    pairs = (len(A) - ones) * ones - many_ones
    if pairs > 1e9:
        return -1
    return pairs
```

### Attempt 7
- `return -1` condition is wrong. pairs is what needs to be exceeding `1e9`, not ones, which can never exceed `1e9` due to question constraints `N <= 100,000`
- And you cannot calculate a total without counting every single 1, which requires completing 1 pass, which means you never had any basis for an early exit condition, making the double iterator is redundant.
#### Test Inputs
```txt
[0]
[1]
[0, 0]
[0, 1]
[1, 0]
[1, 1]
[0, 0, 1]
[0, 1, 0, 1]
[1, 0, 1, 0]
[1, 0, 0, 1, 1]
```
#### Code
```python
# you can write to stdout for debugging purposes, e.g.
# print("this is a debug message")

import math

def solution(A):
    # Number of unique ways to obtain a pair (0, 1)
    # For any given 0, the number of pairs it can make = sum of 1s to its right
    # But using algebra, we could complete the task in 1 pass,
    # substituting the total 1s at the end of the pass
    # So for [0, 1, 0, 1, 1]. Let the total ones = t
    # 0 pairs=t-0       ones = 0
    # 1 pairs="         ones = 1
    # 2 pairs=t+t-1     ones = 1
    # 3 pairs="         ones = 2
    # 4 pairs="         ones = 3 = t
    # E pairs=2t-1=6-1=5
    # The number of t's at the end is len(A) - t = 5-3 = 2
    # At each 0, pairs+=t-ones at that point
    # At each 1, we increment ones
    # At the end, pairs == zeros * t - many_ones

    # Optimization: We can enact the 1e9 check only after we've encountered the min number of cars needed to exceed this value
    # Min no. is 0s followed by an equal no. of 1s, or zeros * ones - 0 == zeros^2
    # sqrt(1e9) ~= 63246
    SQRT_1e9=math.ceil(math.sqrt(1e9)) # Used for enabling the exit check when N is large
    A_iter = None
    A_iter_large = None
    if len(A) < SQRT_1e9:
        A_iter = iter(A)
        # A_iter_large = None
    else:
        A_iter = iter(A[:SQRT_1e9])
        A_iter_large = iter(A[SQRT_1e9:])

    many_ones = 0
    ones = 0

    # Optimization: While ones==0, case 0 makes no changes
    for car in A_iter:
        if car == 1:
            ones = 1 # Set ones inside conditional loop, to allow A_iter_large to continue ones==0 optimization if none are found in A_iter
            break

    for car in A_iter:
        match car:
            case 0:
                many_ones += ones
            case 1:
                ones += 1
    
    if A_iter_large:
        # Optimization: While ones==0, case 0 makes no changes
        if ones == 0:
            for car in A_iter_large:
                if car == 1:
                    ones = 1
                    break

        for car in A_iter_large:
            match car:
                case 0:
                    many_ones += ones
                case 1:
                    ones += 1
                    if ones > 1e9:
                        return -1

    # Now ones == t (total ones/sum)
    # zeros = len(A) - ones
    # pairs == zeros * t - many_ones
    return (len(A) - ones) * ones - many_ones
```
### Attempt 6
```python
# you can write to stdout for debugging purposes, e.g.
# print("this is a debug message")

import math

def solution(A):
    # Number of unique ways to obtain a pair (0, 1)
    # For any given 0, the number of pairs it can make = sum of 1s to its right
    # But using algebra, we could complete the task in 1 pass,
    # substituting the total 1s at the end of the pass
    # So for [0, 1, 0, 1, 1]. Let the total ones = t
    # 0 pairs=t-0       ones = 0
    # 1 pairs="         ones = 1
    # 2 pairs=t+t-1     ones = 1
    # 3 pairs="         ones = 2
    # 4 pairs="         ones = 3 = t
    # E pairs=2t-1=6-1=5
    # The number of t's at the end is len(A) - t = 5-3 = 2
    # At each 0, pairs+=t-ones at that point
    # At each 1, we increment ones
    # At the end, pairs == zeros * t - many_ones

    # Optimization: We can enact the 1e9 check only after we've encountered the min number of cars needed to exceed this value
    # Min no. is 0s followed by an equal no. of 1s, or zeros * ones - 0 == zeros^2
    # sqrt(1e9) ~= 63246
    SQRT_1e9=math.ceil(math.sqrt(1e9)) # Used for enabling the exit check when N is large
    A_iter = None
    A_iter_large = None
    if len(A) < SQRT_1e9:
        A_iter = iter(A)
        # A_iter_large = None
    else:
        A_iter = iter(A[:SQRT_1e9])
        A_iter_large = iter(A[SQRT_1e9:])

    # Declare variables. They will start in this state
    many_ones = 0
    ones = 1 # We've encountered one 1 already above

    # Optimization: While ones == 0, case 0 makes no changes, so skip to the 1st one
    for car in A_iter:
        if car == 1:
            break

    for car in A_iter:
        match car:
            case 0:
                many_ones += ones
            case 1:
                ones += 1

    # Optimization: While ones == 0, case 0 makes no changes, so skip to the 1st one
    # Catch case of all 0s
    
    if A_iter_large:
        # Optimization: While ones == 0, case 0 makes no changes, so skip to the 1st one
        for car in A_iter:
            if car == 1:
                break

        for car in A_iter:
            match car:
                case 0:
                    many_ones += ones
                case 1:
                    ones += 1
                    if ones > 1e9:
                        return -1

    # Now ones == t (total ones/sum)
    # zeros = len(A) - ones
    # pairs == zeros * t - many_ones
    return (len(A) - ones) * ones - many_ones
```
### Attempt 6 WIP
- try catch is bad for performance, especially when you are using it for input you can actually expect
```python
# you can write to stdout for debugging purposes, e.g.
# print("this is a debug message")

import math

def solution(A):
    # Number of unique ways to obtain a pair (0, 1)
    # For any given 0, the number of pairs it can make = sum of 1s to its right
    # But using algebra, we could complete the task in 1 pass,
    # substituting the total 1s at the end of the pass
    # So for [0, 1, 0, 1, 1]. Let the total ones = t
    # 0 pairs=t-0       ones = 0
    # 1 pairs="         ones = 1
    # 2 pairs=t+t-1     ones = 1
    # 3 pairs="         ones = 2
    # 4 pairs="         ones = 3 = t
    # E pairs=2t-1=6-1=5
    # The number of t's at the end is len(A) - t = 5-3 = 2
    # At each 0, pairs+=t-ones at that point
    # At each 1, we increment ones
    # At the end, pairs == zeros * t - many_ones

    # Optimization: We can enact the 1e9 check only after we've encountered the min number of cars needed to exceed this value
    # Min no. is 0s followed by an equal no. of 1s, or zeros * ones - 0 == zeros^2
    # sqrt(1e9) ~= 63246
    SQRT_1e9=math.ceil(math.sqrt(1e9)) # Used for enabling the exit check when N is large
    A_iter = None
    A_iter_large = None
    if len(A) < SQRT_1e9:
        A_iter = iter(A)
        # A_iter_large = None
    else:
        A_iter = iter(A[:SQRT_1e9])
        A_iter_large = iter(A[SQRT_1e9:])

    # Declare variables. They will start in this state
    many_ones = 0
    ones = 1 # We've encountered one 1 already above

    try:
        # Optimization: While ones == 0, case 0 makes no changes, so skip to the 1st one
        # Catch case of all 0s
        while next(A_iter) == 0:
            pass

        for car in A_iter:
            match car:
                case 0:
                    many_ones += ones
                case 1:
                    ones += 1
    except StopIteration: # All zeros in first SQRT_1e9 of sequence, while loop failed
        if A_iter_large == None:
            return 0
        # else move onto the large_iter and repeat
            
    
    # Optimization: While ones == 0, case 0 makes no changes, so skip to the 1st one
    # Catch case of all 0s
    
    if A_iter_large:
        try:
            # Optimization: While ones == 0, case 0 makes no changes, so skip to the 1st one
            # Catch case of all 0s
            while next(A_iter) == 0:
                pass

            for car in A_iter:
                match car:
                    case 0:
                        many_ones += ones
                    case 1:
                        ones += 1
                        if ones > 1e9:
                            return -1
        except StopIteration: # All zeros in remaining sequence, while loop failed
            return 0

    # Now ones == t (total ones/sum)
    # zeros = len(A) - ones
    # pairs == zeros * t - many_ones
    return (len(A) - ones) * ones - many_ones
```
### Attempt 5
```python
# you can write to stdout for debugging purposes, e.g.
# print("this is a debug message")

def solution(A):
    # Number of unique ways to obtain a pair (0, 1)
    # For any given 0, the number of pairs it can make = sum of 1s to its right
    # But using algebra, we could complete the task in 1 pass,
    # substituting the total 1s at the end of the pass
    # So for [0, 1, 0, 1, 1]. Let the total ones = t
    # 0 pairs=t-0       ones = 0
    # 1 pairs="         ones = 1
    # 2 pairs=t+t-1     ones = 1
    # 3 pairs="         ones = 2
    # 4 pairs="         ones = 3 = t
    # E pairs=2t-1=6-1=5
    # The number of t's at the end is len(A) - t = 5-3 = 2
    # At each 0, pairs+=t-ones at that point
    # At each 1, we increment ones
    # At the end, pairs == zeros * t - many_ones
    pairs = 0
    ones = 0
    for car in A:
        match car:
            case 0:
                pairs -= ones
            case 1:
                ones += 1

    # Now ones == t (total ones/sum)
    zeros = len(A) - ones
    return zeros * ones + pairs
```
### Attempt 4
```python
# you can write to stdout for debugging purposes, e.g.
# print("this is a debug message")

def solution(A):
    pairs = 0
    # i and j need to be different cars, so A[:-1] is the penultimate car, and car1 becomes the ultimate on the final loop
    # We want to eliminate the diagonal (and the bottom triangle) from our table of i and j.
    for i in range(len(A)-1): # O(n)
        if A[i] == 1: # Optimization: Due to (P, Q) restriction: 0 ≤ P < Q < N, P must be 0 to form a pair
            continue
        for j in range(i+1, len(A)): # O(n) # i+1 Avoids double counting pairs and the same car
            pairs += A[j] # If j == 1, we have a pair and add 1 to pairs. Otherwise we add nothing, or 0
        if pairs > 1000000000: # Problem specifically states that this is a requirement
            return -1

    return pairs

```
### Attempt 3
```python
# you can write to stdout for debugging purposes, e.g.
# print("this is a debug message")

import logging

# logging.basicConfig(level=logging.DEBUG)

def solution(A):
    pairs = 0
    # i and j need to be different cars, so A[:-1] is the penultimate car, and car1 becomes the ultimate on the final loop
    # We want to eliminate the diagonal (and the bottom triangle) from our table of i and j.
    for i in range(len(A)-1): # O(n)
        if A[i] == 1: # Optimization: Due to (P, Q) restriction: 0 ≤ P < Q < N, P must be 0 to form a pair
            continue
        for j in range(i+1, len(A)): # O(n) # i+1 Avoids double counting pairs and the same car
            logging.debug(f"Testing: {A[i]}!={A[j]} i={i} j={j}")
            if A[j] == 1: # If i and j make a pair
                pairs += 1
                if pairs > 1000000000: # Problem specifically states that this is a requirement
                    return -1
                logging.debug(f"Pair: ({i}, {j})")

    return pairs

```
### Attempt 2
```python
# you can write to stdout for debugging purposes, e.g.
# print("this is a debug message")

import logging

# logging.basicConfig(level=logging.DEBUG)

def solution(A):
    pairs = 0
    # i and j need to be different cars, so A[:-1] is the penultimate car, and car1 becomes the ultimate on the final loop
    # We want to eliminate the diagonal (and the bottom triangle) from our table of i and j.
    for i in range(len(A)-1): # O(n)
        if A[i] == 1: # Optimization: Due to (P, Q) restriction: 0 ≤ P < Q < N, P must be 0 to form a pair
            continue
        for j in range(i+1, len(A)): # O(n) # i+1 Avoids double counting pairs and the same car
            logging.debug(f"Testing: {A[i]}!={A[j]} i={i} j={j}")
            if A[j] == 1: # If i and j make a pair
                pairs += 1
                logging.debug(f"Pair: ({i}, {j})")

    return pairs
```
### Attempt 1
```python
# you can write to stdout for debugging purposes, e.g.
# print("this is a debug message")

import logging

# logging.basicConfig(level=logging.DEBUG)

def solution(A):
    pairs = 0
    # i and j need to be different cars, so A[:-1] is the penultimate car, and car1 becomes the ultimate on the final loop
    # We want to eliminate the diagonal (and the bottom triangle) from our table of i and j.
    for i in range(len(A)-1):
        car0 = A[i]
        for j in range(i+1, len(A)): # i+1 Avoids double counting pairs and the same car
            car1 = A[j]
            logging.debug(f"Testing: {car0}!={car1} i={i} j={j}")
            if car0 < car1: # If i and j make a pair
                pairs += 1
                logging.debug(f"Pair: ({i}, {j})")

    return pairs
            
```
