# 7a-Brackets
## Notes
### Generating Almost Identical Logic Expect for a Few Parts
> [!question] How to handle making similar logic with slight differences to already existing code
> There's 2 immediate cases to consider here:
> 1. Logic/Execution only differs from the parameters
> 	- In this case, it's best **NOT** to generate another function for each parameter, and rather, just create a dict which can return all parameters related to the key parameter
> 	- `PAIRS = { ")": "(", "]": "[", "}": "{" }`
> 	- `OPENERS = set(PAIRS.values())`
> 	- See the answer. It falls in this case.
> 2. Logic/Execution differs more than simply a change of parameters
> 	- In this case, it is best to use the **Factory** pattern, which can return specialized objects/functions:
> 	- #### When to use a Factory
> 		- You have **multiple concrete classes** that share an interface.
> 		- The client shouldn't know or care which one it gets.
> 		- The choice of class depends on runtime conditions (config, platform, input type).
> 		- The classes have **substantial, distinct behavior**.

```python
# Factory Pattern in Python
from abc import ABC, abstractmethod

class Bracket(ABC):
    @abstractmethod
    def matches(self, other): ...

class Paren(Bracket):
    def matches(self, other): return isinstance(other, Paren)

class Bracket_(Bracket):
    def matches(self, other): return isinstance(other, Bracket_)

class Brace(Bracket):
    def matches(self, other): return isinstance(other, Brace)
```

## Problem
A string S consisting of N characters is considered to be properly nested if any of the following conditions is true:

        S is empty;
        S has the form "(U)
        content_copy
        " or "[U]
        content_copy
        " or "{U}
        content_copy
        " where U is a properly nested string;
        S has the form "VW
        content_copy
        " where V and W are properly nested strings.

For example, the string "{[()()]}
content_copy
" is properly nested but "([)()]
content_copy
" is not.

Write a function:

    def solution(S)
    content_copy

that, given a string S consisting of N characters, returns 1 if S is properly nested and 0 otherwise.

For example, given S = "{[()()]}
content_copy
", the function should return 1 and given S = "([)()]
content_copy
", the function should return 0, as explained above.

Write an efficient algorithm for the following assumptions:

        N is an integer within the range [0..200,000];
        string S is made only of the following characters: '(
        content_copy
        ', '{
        content_copy
        ', '[
        content_copy
        ', ']
        content_copy
        ', '}
        content_copy
        ' and/or ')
        content_copy
        '.
## Test Cases
```txt
''
'[][]{]'
'[][][]}'
')'
'((((((('
')))))))'
'())'
'()('
')())'
'((()))()('
```
## Code
### Attempt 2
```python
def solution(S):
    pairs = {
        ")": "(",
        "}": "{",
        "]": "["
    }
    openers = pairs.values()
    # print("openers: ", openers)
    
    # Order we encounter brackets matters
    # If we clear a bracket, we need to remember the previous one
    # We must remember at least all valid enocuntered openers until
    # they are cleared
    stack = []
    for char in S:
        # print("char: ", char)
        # print("stack: ", stack)
        if char in openers:
            stack.append(char)
            continue

        # If we encounter a closer with no opener
        if len(stack) == 0: # Cannot pop an empty stack
            return 0
        last_opener = stack.pop()
        if pairs[char] != last_opener:
            return 0
        
    return int(len(stack) == 0) # Ensure there's no open brackets left
```
### Attempt 1
- Missed the case where we leave open brackets
	- When we encounter open brackets, we by default just add it to the stack.
	- The stack will only ever contain open brackets
	- But what if we don't close all of them?
```python
def solution(S):
    pairs = {
        ")": "(",
        "}": "{",
        "]": "["
    }
    openers = pairs.values()
    # print("openers: ", openers)
    
    # Order we encounter brackets matters
    # If we clear a bracket, we need to remember the previous one
    # We must remember at least all valid enocuntered openers until
    # they are cleared
    stack = []
    for char in S:
        # print("char: ", char)
        # print("stack: ", stack)
        if char in openers:
            stack.append(char)
            continue

        # If we encounter a closer with no opener
        if len(stack) == 0: # Cannot pop an empty stack
            return 0
        last_opener = stack.pop()
        if pairs[char] != last_opener:
            return 0
        
    return 1
```
