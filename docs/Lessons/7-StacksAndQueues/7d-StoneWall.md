## Notes
- 
## Problem
You are going to build a stone wall. The wall should be straight and N meters long, and its thickness should be constant; however, it should have different heights in different places. The height of the wall is specified by an array H of N positive integers. H[I] is the height of the wall from I to I+1 meters to the right of its left end. In particular, H[0] is the height of the wall's left end and H[N−1] is the height of the wall's right end.

The wall should be built of cuboid stone blocks (that is, all sides of such blocks are rectangular). Your task is to compute the minimum number of blocks needed to build the wall.

Write a function:

    def solution(H)
    content_copy

that, given an array H of N positive integers specifying the height of the wall, returns the minimum number of blocks needed to build it.

For example, given array H containing N = 9 integers:

  H[0] = 8    H[1] = 8    H[2] = 5
  H[3] = 7    H[4] = 9    H[5] = 8
  H[6] = 7    H[7] = 4    H[8] = 8
content_copy

the function should return 7. The figure shows one possible arrangement of seven blocks.

Write an efficient algorithm for the following assumptions:

        N is an integer within the range [1..100,000];
        each element of array H is an integer within the range [1..1,000,000,000].

## Test Cases
```txt
[1] [7]
```
## Code
- Pretty much nailed the textbook optimal solution, and it's very readable too. All comments provide extremely valuable information here to any reader. 
	- Improvements to readability here make a negligible difference.
```python
# KEY:
# Blocks cannot be reused if there is a smaller block after it
# Keep track of blocks greater than smallest block encountered
# Forget blocks behind a small block

def solution(H):
    blocks = 0
    available_block_heights = []
    for h in H:
        while len(available_block_heights) > 0 and \
				h < available_block_heights[-1]:
            available_block_heights.pop()
        
        if len(available_block_heights) == 0 or \
				available_block_heights[-1] != h:
            blocks += 1
            available_block_heights.append(h)

        # if available_block_heights[-1] == h
        # then we can reuse an existing block

    return blocks
```
