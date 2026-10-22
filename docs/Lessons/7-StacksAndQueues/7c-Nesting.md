## Notes
- 
## Problem

## Test Cases
```txt
''
'()'
'((((((()))))))'
')((((((()))))))('
'))(('
'()('
'()(('
'())(()'
'(((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((())))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))'
```
## Code
### Attempt 2
- We've made the exit conditions in the loop as clear as we possibly can, both the negative open brackets, and the pruning condition.
	- This should make it obvious that the exit condition will always trigger in all cases and return 0 for invalid `S`
```python
def solution(S):
    if len(S) % 2 == 1:
        return 0

    num_open_brackets = 0
    for i, char in enumerate(S):
        match char:
            case "(":
                num_open_brackets += 1
            case ")":
                num_open_brackets -= 1

		remaining_chars = len(S) - (i + 1)
        if num_open_brackets < 0 or num_open_brackets > remaining_chars:
            return 0

    return 1
```

### Attempt 1
- This solution is perfect unit-tests-wise. But it can be unclear why the `return 1` at the end is safe.
```python
def solution(S):
    if len(S) % 2 == 1:
        return 0

    num_open_brackets = 0
    for i, char in enumerate(S):
        match char:
            case "(":
                num_open_brackets += 1
            case ")":
                num_open_brackets -= 1

        if num_open_brackets < 0 or num_open_brackets > (len(S) - (i+1)):
            return 0

    return 1
```