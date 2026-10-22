# 7b-Fish
## Notes
### Common Codility Coding Patterns
> [!tip] Common Codility Coding Patterns
> Most Codility tricks fall into a few buckets:
> - **Track one thing as you go** (running min/max/sum) → O(n)
> - **Use a stack** for nested or "frontier" problems → O(n)
> - **Use a set or hashmap** for lookup problems → O(n)
> - **Sort first** to turn O(n²) comparisons into O(n log n)
> - **Two pointers** moving from ends or at different speeds
> - **Prefix sums** for range queries
### How to Not Get Confused by Your Own Code
> [!tip] How to Not Get Confused by Your Own Code
> Funnily enough, the trick is to **plan first**. Then write.
> Most candidates' biggest mistake is **writing a solution WHILE trying to determine the optimal solution**
> - Why? Because these candidates are forcing themselves to multitask between 4 things at once:
> 	- Understanding the problem
> 	- Deriving the algorithm
> 	- Tracking code state
> 	- Making readable code
> #### The Workflow that Reduces the Load
> 1. Understand - Don't think about code
> 	1. Restate the problem in your own words.
> 	2. Write the reworded problem down in your notes
> 	3. Identify:
> 		1. Inputs
> 		2. Outputs
> 		3. The Key to the optimal solution
> 			1. Most Codility Questions have a naive slow solution of like $O(n^2)$, like finding all valid pairs
> 			2. and a quick solution. In this exercise, the trick is that only the frontier fish matters, so we need to use a stack data type, which is at worst $O(2n) = O(n)$
> 2. Pseudocode and Dry Runs
> 	1. Work through 2-3 small examples by hand
> 	2. Find the pattern(s)
> 	3. Write a pseudocode algorithm in plain English:
> 		1. ![[7b-Fish#^7886db]]
> 3. Translate Pseudocode into Code
> 	1. If you get confused by your own code - **delete and rewrite it**. This is quicker than debugging a complicated solution
> 4. Test and Verify Code
> 	1. Ensure it works as intended
> 	2. Ensure it meets **ALL** criteria in the question
> 5. Clean Up - Delete notes, prints, draft comments

#### References
```txt
stack of surviving fish
for each new fish:
    while top of stack fights new fish and new fish is bigger:
        pop
    if top still fights new fish:
        new fish dies
    else:
        push new fish
```
^7886db

### Abstraction - How Good is it Really?
> [!danger] Abstraction - How good is it really?
> Abstraction is the process of simplifying complex systems by focusing on essential details while hiding unnecessary complexity, allowing programmers to manage large systems using high-level commands.
> > [!tldr]- Attempt 2
> > ![[7b-Fish#Attempt 2]]
> My 2nd attempt does use some form of abstraction which takes 2 Fish Named Tuples, hides a fair amount of logic, and returns 3 possible Enum result states.
> - Seemingly, abstraction here has allowed for the creation of a single function that can help us compare 2 fish in all situations we could desire...
> 
> But has it actually simplified the system?
> - We created 2 constants: `UPSTREAM` and `DOWNSTREAM`
> - We created a Named Tuple: `Fish`, which has a `size` and `direction`.
> - We created an enum `Fish_Encounter_Result` specifically to represent the 3 possible results it could return.
> - We created a helper function `fish_encounter` that holds **critical** logic to the functioning of `solution`
> 	- The word, `encounter`, is not a term used in the original question, meaning **I've invented a new term** for this solution which will force readers to understanding the code, just to understand the meaning of the term. Not good at all!
> - And while everything is named with its exact purpose, altogether it has produced some very long words and lines
> - And **ALL** of that is before we even reach **solution**...
> 
> No. This abstraction failed to simplify the complexity of the system. So what gives?
>
> #### Key Message
> > **Match the weight/size of the abstraction to the weight/size of the problem.**
> > "Will this abstraction be used, named, or reasoned about outside this function?"
> > - If yes — enum, class, namedtuple, module-level helper: all fair game.
> > - If no — inline it. Let the algorithm be the thing the reader sees.
>
> This problem is:
> - A single stack
> - One kind of interaction (downstream vs upstream)
> - One decision per interaction (who wins)
> 
> That's small. The abstraction should be small too.
> - `Fish` Named Tuple is a perfect example. We could just use a raw tuple instead: `(size, direction)`. So `river[-1].size vs river[-1][0]`. `river[-1].size` is nicer for sure. But is it really worth introducing an extra type for a 10 line Codility answer? No - probably not.

### Enums in Python
```python
from enum import Enum

# class syntax
class Color(Enum):
    RED = 1
    GREEN = 2
    BLUE = 3

# functional syntax
Color = Enum('Color', [('RED', 1), ('GREEN', 2), ('BLUE', 3)])
# or in simple cases
Color = Enum('Color', ['RED', 'GREEN', 'BLUE'])

Color.RED  # returns <Animal.ant: 1>
Color['RED']  # returns <Animal.ant: 1> (string lookup)
Color.RED.name  # returns 'RED' (inverse lookup)
```
- Note that B is 0s and 1s, not a Direction Enum. So better to just use constants for that
### Self-documenting Code
> [!tip] What is Self-documenting Code?
> A piece of code is self-documenting when a reader can understand **what it does and why** without:
> 1. Jumping to another location to resolve a name
> 2. Holding hidden context in their head
> 3. Reconstructing the author's intent from the structure
> 
> That's it. It's a property of the **reader's experience**, not of how many names you invented.
> - Surprisingly, trying to create a more accurate representation of the data, can go against readable data.
>
> #### What Self-documenting code CANNOT do?
> - Code explains "what", not "why". Code cannot provide the **intent**, the **context**, or the "**why**" behind a decision.
> - A comment like: `// Using a binary search because this dataset is expected to have millions of records `
> - Code cannot fully describe **side effects** or **contracts**.
> 	- Side Effect - Code changes something that is not obvious by it's interface/signature: E.g. Modifies a global variable
> 	- Contract - Requires prequisite conditions to be met before it's called.
> 
> #### Bad Example:
> ```python
> fish_clash = lambda: len(river) > 0 and river[-1].drt == 1 and drt == 0
> ```
> To understand fish_clash(), the reader must:
> - Know that `river` is the stack (defined above)
> - Know that `river[-1]` is the top fish
> - Know that `drt` is the current fish's direction, not the top fish's
> - Know that `1` means downstream and `0` means upstream
> - Know that `fish` (the variable) exists and is the current fish
> ##### Verdict
> Five hidden dependencies. The reader still has to open the lambda body and trace every free variable, making it self-obscuring with a friendly label, rather than self-documenting.
> #### Better Example
> ```python
def will_fish_fight(top, incoming):
return top.direction == DOWNSTREAM and incoming.direction == UPSTREAM
> ```
> - Clearly shows it's a function returning a boolean on whether the fish will fight
> - But the name may be a little long. Maybe, maybe not. See next part

### Variable, Object and Function Naming
> [!note] How to make Good, Self-documenting Names
> A good name:
> - Describes **exactly what the thing is**, not the action or feeling:
>	- `surviving_fish = []`or `living_fish = []`
>		- NOT just `alive` or `river`
>- Are **as long as their scope demands**
>	- `surviving_fish`is a function-wide level variable, that is used in the return value. It's pretty important and deserves a long name.
>	- A variable in a nested loop needs to be short - you've already got very little space to work with.
>		- A module/public API variable/function/object? That requires a full name with documentation
>- Are consistent across the codebase
>- Should **not require a comment** to be understood
>	- If a variable is only used in a very small section of the code, then you may get away with a small name.
>		- `idx`, or `x, y`.
>	- Even then, there's standards for abbreviations too:
>		- `row`/`col` — 2D data
>		- `i`, `j`, `k` — loop indices.
>			- Though `i` and `j` are infamously known to be extremely easy to mix-up in nested loops.
>			- Even Google recommends using more descriptive names in loops because of this.
>				- `r` instead of `i` or `row`
>				- `c` instead of `j` or `col`/`column
>		- `n` — count of items
>		- `x`, `y`, `z` — coordinates
>		- `id` — identifier
>		- `url`, `http`, `api` — well-known acronyms
>		- `min`, `max`, `len` — standard
>		- `dpi`, `rgb`, `html` — universally known
>	- Though yes, you can shorten names in very small contexts, where the meaning is still very obvious:
>		- From: `for surviving_fish in surviving_fish_by_size:`
>			- To: `for f in surviving_fish_by_size:`
>		- From: `for row in range(rows):`
>			- `for col in range(cols)`
>			- To:
>	- Below is an example of exactly a bad name that fails to be self-documenting, because it required a comment to be understood:
>		- ```python
def will_fish_fight(stack_fish_drt, new_fish_drt): # drt = DiRecTion ```
>		- The **comment is critical** to understanding what the parameters stand for
>			- You are forcing any reader of this code to learn the meaning of these parameters by jumping back in the code. Not self-documenting at all
>		- `drt` **on it's own is ambiguous**, maybe it's short for `dirt`, the word closest to it in the English Language?
>			- Meanwhile `direction` is completely unambiguous, and not much longer. Plus nowadays, IDEs can autocomplete such names.
>		- These **variable names will NOT appear if someone does a search** for `direction`
>	- And here's the same example improved:
>		- ```python
def will_fish_fight(top_direction, new_direction):```
> 		- Use `direction` - plain, simple, unmistakable english.
> 			- Even better to **use words directly from the question**, so its meaning cannot be misunderstood. No inventing new words
> 		- We can actually shorten the parameter names by removing `fish` from both - because the function name itself already clearly mentions fish.
### Tuples are **Immutable**
Tuples are **immutable**, so best not to use tuples for data that needs to change.
### Named Tuples in Python
```python
from collections import namedtuple

# Note Fish is a special type of class
Fish = namedtuple("Fish", ["size", "direction"])
Fish(5, 0)
```
## Problem
You are given two non-empty arrays A and B consisting of N integers. Arrays A and B represent N voracious fish in a river, ordered downstream along the flow of the river.

The fish are numbered from 0 to N − 1. If P and Q are two fish and P < Q, then fish P is initially upstream of fish Q. Initially, each fish has a unique position.

Fish number P is represented by A[P] and B[P]. Array A contains the sizes of the fish. All its elements are unique. Array B contains the directions of the fish. It contains only 0s and/or 1s, where:

        0 represents a fish flowing upstream,
        1 represents a fish flowing downstream.

If two fish move in opposite directions and there are no other (living) fish between them, they will eventually meet each other. Then only one fish can stay alive − the larger fish eats the smaller one. More precisely, we say that two fish P and Q meet each other when P < Q, B[P] = 1 and B[Q] = 0, and there are no living fish between them. After they meet:

        If A[P] > A[Q] then P eats Q, and P will still be flowing downstream,
        If A[Q] > A[P] then Q eats P, and Q will still be flowing upstream.

We assume that all the fish are flowing at the same speed. That is, fish moving in the same direction never meet. The goal is to calculate the number of fish that will stay alive.

For example, consider arrays A and B such that:

  A[0] = 4 B[0] = 0
  A[1] = 3 B[1] = 1
  A[2] = 2 B[2] = 0
  A[3] = 1 B[3] = 0
  A[4] = 5 B[4] = 0
content_copy

Initially all the fish are alive and all except fish number 1 are moving upstream. Fish number 1 meets fish number 2 and eats it, then it meets fish number 3 and eats it too. Finally, it meets fish number 4 and is eaten by it. The remaining two fish, number 0 and 4, never meet and therefore stay alive.

Write a function:

    class Solution { public int solution(int[] A, int[] B); }
    content_copy

that, given two non-empty arrays A and B consisting of N integers, returns the number of fish that will stay alive.

For example, given the arrays shown above, the function should return 2, as explained above.

Write an efficient algorithm for the following assumptions:

        N is an integer within the range [1..100,000];
        each element of array A is an integer within the range [0..1,000,000,000];
        each element of array B is an integer that can have one of the following values: 0, 1;
        the elements of A are all distinct.

## Test Cases
```txt
([5, 1, 2, 3, 4], [1, 1, 1, 0, 1])
```
## Code
### Attempt 3
- This is a practically perfect answer. All problems here are negligible by comparison to the issues the other solutions had
```python
# UPSTREAM = 0
DOWNSTREAM = 1

def solution(A, B):
    downstream_fish = [] # A stack data-type
    survived_upstream_fish = 0

    for size, direction in zip(A, B):
        if direction == DOWNSTREAM:
            downstream_fish.append(size)
            continue

        while len(downstream_fish) > 0 and size > downstream_fish[-1]:
            downstream_fish.pop()

        if len(downstream_fish) == 0:
            survived_upstream_fish += 1

    return survived_upstream_fish + len(downstream_fish)
```
### Attempt 2
- Apparently, I made the readability even worse by adding an abstraction which handles much logic
```python
from collections import namedtuple
from enum import Enum

Fish = namedtuple("Fish", ["size", "direction"])
UPSTREAM = 0
DOWNSTREAM = 1

Fish_Encounter_Result = Enum("Fish_Encounter_Result", ["NO_FIGHT", "FISH1_WINS", "FISH2_WINS"])
def fish_encounter(fish1: Fish, fish2: Fish) -> Fish_Encounter_Result:
    # print("fish1: ", fish1)
    # print("fish2: ", fish2)
    if not (fish1.direction == DOWNSTREAM and fish2.direction == UPSTREAM):
        # print("NO_FIGHT")
        return Fish_Encounter_Result.NO_FIGHT

    elif fish1.size > fish2.size:
        # print("FISH1_WINS")
        return Fish_Encounter_Result.FISH1_WINS
    else:
        # print("FISH2_WINS")
        return Fish_Encounter_Result.FISH2_WINS

def solution(A: list, B: list):
    if len(A) < 2:
        return len(A)

    surviving_fish = []

    for size, direction in zip(A, B):
        new_fish = Fish(size, direction)
    
        # print("===================================")
        # print("surviving_fish: ", surviving_fish)
        # print("new_fish: ", new_fish)

        while len(surviving_fish) > 0 and fish_encounter(surviving_fish[-1], new_fish) == Fish_Encounter_Result.FISH2_WINS:
            surviving_fish.pop()
            # print("DIED")

        if len(surviving_fish) == 0 or \
            fish_encounter(surviving_fish[-1], new_fish) != Fish_Encounter_Result.FISH1_WINS:
            surviving_fish.append( new_fish )
            # print("surviving_fish new: ", surviving_fish)
        # else: surviving_fish[-1] ate new_fish. It's already in surviving_fish, so just don't add new_fish

    # print("surviving_fish FINAL: ", surviving_fish)
    return len(surviving_fish)
```
### Attempt 1
- This solution works, but it is very messy. Better rewrite this in a readable/simple way
```python
# Not sure what the optimal solution will look like
# So start with the unoptimal one and make realizations
# The unoptimal one is ugly - basically a fancy Bubble Sort on B, affecting A: O(n^2)
# 2 for loops (iterating opposite directions) inside a while loop which checks if B is sorted
# The optimal one will use a stack, because we only care about
#   the active fish, and the top of the stack can append and pop
#   - move both directions. This is likely the one pass solution
#
# Realization 1: Once B is ordered, we can stop

from collections import namedtuple

# Note Fish is a special type of class
# szs = SiZeS
# drts = DiRecTionS
Fish = namedtuple("Fish", ["sz", "drt"])

def solution(A, B):
    if len(A) < 2:
        return len(A)

    river = []
    for sz, drt in zip(A, B):
        fish = Fish(sz, drt)

        fish_clash = lambda: len(river) > 0 and \
                                river[-1].drt == 1 and drt == 0
        
        new_fish_wins = lambda: fish.sz > river[-1].sz

        if not fish_clash():
            river.append(fish)
            continue

        while fish_clash() and new_fish_wins():
            river.pop() # Kill Fish

        if not fish_clash(): # New fish beat all downstream fish
            river.append(fish)
        # else: # New fish died to river[-1]. It's already in river

    return len(river)
```
