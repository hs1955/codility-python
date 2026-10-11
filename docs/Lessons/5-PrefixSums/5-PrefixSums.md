# 5-PrefixSums
## What is it?
A technique for quickly calculating the sums/counts of elements within a slice of an array.
The idea is the have a cumulative frequency/sum of your desired elements to count at each index. Then to calculate the count of those elements at each index, you just subtract the counts/sums at the start and end of the slice:
![[IMG-5-PrefixSums-20261007222023077.png]]
So the count/sum of elements in a slice is ($x, y$ inclusive!):
$$
P[x, y] = P[y+1] - P[x]
$$
where
$$
\begin{align}
&len(P) = N+1  \\
&\text{ and } P[0] == 0 \\ \\
&\text{ and } P[n] == a_{n-1} + a_{n-2} + \dots + a_{1} + 0 \\
\end{align}
$$
So,
$$
\begin{align} P[1, 2] &= \\
& P[3] - P[1] =  \\
& (a_{0} + a_{1} + a_{2}) - a_{0} = \\
& a_{1} + a_{2}
\end{align}$$
$$
P[1, 2] = P[3] - P[1]
$$

> [!warning] Why is $len(P) == N+1$ and $P[0] == 0$?
> Why? Because of $P[y+1]$.
> If the sum/counts of index `i`  were directly stored at $P[i]$, we wouldn't be able to access $a_{0}$ as that would always be subtracted off. And trying to include it would be even worse...:
> - Let's say $B[i] = a_{i} + a_{i-1} +\dots+ a_{0}$
> - Then $B[x, y] = B[y] - B[x-1]$
> - So
>   $$\begin{align} B[1, 2] &= \\ & B[2] - B[0] = \\ &(a_{0} + a_{1} + a_{2}) - a_{0} =  \\ & a_{1} + a_{2} \end{align}$$
 > - What if we want $a_{0} + a_{1}$?
> 	- Ironically, we don't need the formula here in this case since $B[1]$ is our answer.
> 	- But if we applied it:
> 		- $B[1] - B[-1] = !!!$
> 			- Oh god, Python actually accepts negative indices too
> 	- Rather than use add an `if x==0:`, we can use $P$ instead:
> 		- $len(P) = N+1$
> 		- $P[0] = 0$
> 	- So the formula then becomes:
> 		- $P[2]-P[0]$ instead of $B[1] - B[-1]$.
> 		- $P[y+1]-P[x]$ instead of $B[y] - B[x-1]$
> 		- and it works for our edge case now
> 	- The Python range/slice which has been counted/summed is $A[0:2]$
> 	- So when creating $P$, we need to put the cumulative sum up to and including $a_{n}$ into $P[n+1]$. NOT $P[n]$

> [!warning] Beware Inclusive/Exclusive Bounds!
> That will change this formula and how we build $P$
> For Python, the start index is inclusive, and the outer bound is exclusive.
> - For example: We want $a_{1} + a_{2} = P[1,2]$, we need to remove $a_{0}$ from our count.
> 	- So we must use $P[3] - P[1]$
> 	- The Python range/slice which has been counted/summed is $A[1:3]$
## How to do it?
Calculate the cumulative frequency/sum of the element(s) at each index of the array.
![[IMG-5-PrefixSums-20261007222023077.png]]
Note that:
$$
\begin{align}
&p_{n+1} \\
&= p_{n}+a_{n+1} \\
&= a_{0}+a_{1}+\dots+a_{n}+a_{n+1}
\end{align}
$$
Meaning you can just calculate the next cumulative frequency/sum just by adding the previous one on, which is a constant time operation $O(1)$.

## Where can I apply this?
You can apply this on an iterable of countable elements, both going forwards, and backwards.
- After 1 pass, you'll have all the information you need anyways, but maybe you will obtain enough info before 1 pass, then direction may be relevant