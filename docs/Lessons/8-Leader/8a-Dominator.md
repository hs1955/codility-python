# 8a-Dominator
## Notes
### Dictionary Set a Default Value for a Key
To set the default value of a dictionary `d = {}; d.setdefault("key", 0)`. `setdefault` will return the key value, or set the default.
## Problem

## Test Cases
```txt
[]
[99]
[-99]
[-99, 99]
[99, -99]
[99, 99]
[99, 99, -99]
```
## Code
### Attempt 3:
- Best solution
```python
def solution(A):
    candidate = 0
    size = 0

    for num in A:
        if size == 0:
            candidate = num
            size = 1
        elif candidate == num:
            size += 1
        else:
            size -= 1

    if size == 0: # No majority candidates
	    return -1

	count = 0
	for i, num in enumerate(A):
		if num == candidate:
			count += 1
		if count > len(A) // 2:
			return i

    return -1
```
### Attempt 2
- Counts dict version. Note the first part is the same, eliminating pairs of unique elements. As this has a space complexity of $O(n)$ unnecessarily, this is the worse version
```python
def solution(A):
    counts = dict()
    candidate_idx = -1

    candidate = 0
    size = 0
    for i, num in enumerate(A):
        counts[num] = counts.get(num, 0) + 1
        
        if size == 0:
            candidate = num
            candidate_idx = i
            size = 1
        elif candidate == num:
            size += 1
        else:
            size -= 1

    if size > 0: # Candidate is only valid if size > 0
        if counts[candidate] > len(A) // 2:
            return candidate_idx

    return -1
```

### Attempt 1
- This is my recreation of the optimal leader finding code from [[8-Leader]]. But I bet we could use a counts dict instead of a 2nd pass, to do this in strictly 1 pass.
```python
def solution(A):
    candidate = 0
    size = 0

    for num in A:
        if size == 0:
            candidate = num
            size = 1
        elif candidate == num:
            size += 1
        else:
            size -= 1

    if size > 0: # Candidate is only valid if size > 0
        count = 0
        for i, num in enumerate(A):
            if num == candidate:
                count += 1
            if count > len(A) // 2:
                return i

    return -1
```
