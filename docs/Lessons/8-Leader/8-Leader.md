# Leader
The **leader** is the term in a sequence (of length $n$) that occurs more than $\frac{n}{2}$ times.
$$\text{count}(n_{i})> \frac{n}{2}$$
There can only ever be 1 leader to any sequence by definition. There can never be 2.
## Approaches
### 1. Nested Loop Approach
Count occurrences of every single unique element, like in a dict and check for when a count exceeds $\frac{n}{2}$: $$O(n^2)$$
### 2. Sorting Approach
Sort the sequence, then count which one occurs more than $\frac{n}{2}$ times $$O(n\log n)$$
### 3. Stack-like Approach
If you add elements to a stack in order, and always pop 2 elements when the last 2 on the stack are different, then only a candidate leader elements would remain in the stack (because of this, you don't actually need a stack). 

Then you will need to count the occurrences of that element to determine if there's enough to be considered a leader. $$O(n)$$
