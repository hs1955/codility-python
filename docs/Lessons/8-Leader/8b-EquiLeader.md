## Notes


## Problem


## Test Cases
```txt
[5]
[]
[5]
[-5]
[-5, 5]
[5, -5]
[5, 5]
[5, 5, -5]
```
## Code
### Attempt 2
```python
def solution(A):
    n = len(A)
    if n < 2:
        return 0

    candidate = None
    size = 0
    for num in A:
        if size == 0:
            candidate = num
            size = 1
        elif num == candidate:
            size += 1
        else:
            size -= 1

    if size == 0:
        return 0

    total = sum(1 for num in A if num == candidate)
    if total <= n // 2:
        return 0

    equi_leaders = 0
    left_count = 0
    for i in range(n - 1):
        if A[i] == candidate:
            left_count += 1
        left_len = i + 1
        right_len = n - left_len
        right_count = total - left_count
        if left_count > left_len // 2 and right_count > right_len // 2:
            equi_leaders += 1

    return equi_leaders
```
### Attempt 1
- Not sure how this was rated easy. This is $O(n)$ solution, as it does 3 passes to determine number of equi-leaders. But prefix-counts are required to achieve the 3rd pass.
- Apparently, we don't require prefix_counts, as each prefix_counts value is read only once. We can reduce Space Complexity from $O(n)$ to $O(1)$.
```python
def solution(A):
    if len(A) < 2: # Cannot split A when N < 2
        return 0

    # 1. Determine if there is a leader,
    # as equi-leaders slices will have the same leader
    size = 0
    candidate = None
    for num in A:
        if size == 0:
            candidate = num
            size = 1
        elif num == candidate:
            size += 1
        else:
            size -= 1

    # No candidate leader was found, so no equi-leaders can be found either
    if size <= 0:
        return 0

    # 2. Obtain counts of candidate to calculate prefix counts
    prefix_counts = [0] * (len(A) + 1)
    for i, num in enumerate(A, 1):
        if num == candidate:
            prefix_counts[i] = 1 + prefix_counts[i-1]
        else:
            prefix_counts[i] = prefix_counts[i-1]

    # No leader was found, so no equi-leaders can be found either
    if prefix_counts[-1] <= len(A) // 2:
        return 0

    # leader = candidate
    
    # 3. Determine equi-leaders using prefix_counts of leader
    equi_leaders = 0
    for slice1_len in range(1, len(prefix_counts)-1):
        slice2_len = len(A) - slice1_len
        count1 = prefix_counts[slice1_len]
        count2 = prefix_counts[-1] - prefix_counts[slice1_len]
        if count1 > slice1_len // 2 and count2 > slice2_len // 2:
            equi_leaders += 1

    return equi_leaders
```
