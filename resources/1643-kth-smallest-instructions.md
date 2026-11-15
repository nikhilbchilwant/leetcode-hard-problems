# 1643. Kth Smallest Instructions

> Source: https://algo.monster/liteproblems/1643

## Problem Statement

Bob starts at position `(0, 0)` on a grid and needs to reach a destination at position `(row, column)`. He can only move in two directions:
- **Right** (represented by 'H' for horizontal)
- **Down** (represented by 'V' for vertical)

The path Bob takes is represented as a string of instructions. For example, if the destination is `(2, 3)`, Bob needs to move down 2 times and right 3 times. Valid instruction strings include `"HHHVV"` (go right 3 times, then down 2 times) or `"HVHVH"` (alternating moves).

Since there are multiple valid paths to reach the destination, all possible instruction strings can be ordered lexicographically (dictionary order, where 'H' comes before 'V'). The problem asks you to find the **k-th smallest instruction string** in this lexicographical ordering.

The solution uses combinatorial mathematics to determine which character ('H' or 'V') to place at each position. At each step, it calculates how many valid paths exist if we choose 'H' next using the combination formula `C(h+v-1, h-1)`. If `k` is greater than this count, it means the k-th path must start with 'V' instead of 'H'. Otherwise, it starts with 'H'. This process continues, building the path character by character until reaching the destination.

## Intuition

The key observation is that we need to make exactly `h` horizontal moves and `v` vertical moves to reach the destination, regardless of the order. This means every valid path is a permutation of `h` 'H's and `v` 'V's.

To find the k-th lexicographically smallest path, we can build it character by character from left to right. At each position, we need to decide whether to place an 'H' or a 'V'.

Since 'H' comes before 'V' lexicographically, we want to place 'H' whenever possible to get smaller strings. But how do we know if we should place 'H' at the current position for the k-th smallest path?

The insight is to count how many valid paths exist if we place 'H' at the current position. If we place 'H' now, we have `h-1` horizontal moves and `v` vertical moves remaining. The number of ways to arrange these remaining moves is `C(h+v-1, h-1)` (choosing `h-1` positions for 'H' from `h+v-1` total positions).

This count tells us something crucial:
- If `k ≤ C(h+v-1, h-1)`, then the k-th path is among those that start with 'H'
- If `k > C(h+v-1, h-1)`, then we've "skipped" all paths starting with 'H', so the k-th path must start with 'V'

When we choose 'V', we need to adjust `k` by subtracting the number of paths we skipped (those starting with 'H'). This way, we maintain the relative position of k among the remaining paths.

By repeating this process for each position, we build the k-th smallest path character by character, always making the locally optimal choice based on combinatorial counting.

**Learn more about [Math](/problems/math-basics), [Dynamic Programming](/problems/dynamic_programming_intro) and [Combinatorics](/problems/backtracking) patterns.**

## Solution Approach

The solution implements a greedy algorithm with combinatorial counting to build the k-th smallest path character by character.

**Algorithm Steps:**

1. **Initialize variables**: Extract `v` (vertical moves) and `h` (horizontal moves) from the destination. Create an empty list `ans` to build the result string.

2. **Iterate through each position**: Loop `h + v` times, as we need exactly that many characters in our path.

3. **Handle edge case**: If `h == 0`, we have no horizontal moves left, so we must append 'V'.

4. **Calculate combinations**: For the general case where `h > 0`, calculate `x = comb(h + v - 1, h - 1)`. This represents the number of valid paths if we place 'H' at the current position.

5. **Make decision based on k**:
   - If `k > x`: The k-th path doesn't start with 'H' at this position. We append 'V', decrement `v` by 1, and update `k -= x` to adjust for the skipped paths.
   - If `k ≤ x`: The k-th path starts with 'H' at this position. We append 'H' and decrement `h` by 1.

6. **Build result**: After processing all positions, join the characters in `ans` to form the final instruction string.

**Example walkthrough**: 
For destination `(2, 3)` and `k = 2`:
- Start with `v=2, h=3`
- Position 1: `x = C(4, 2) = 6`, since `k=2 ≤ 6`, append 'H', now `h=2`
- Position 2: `x = C(3, 1) = 3`, since `k=2 ≤ 3`, append 'H', now `h=1`
- Position 3: `x = C(2, 0) = 1`, since `k=2 > 1`, append 'V', now `v=1, k=1`
- Position 4: `x = C(1, 0) = 1`, since `k=1 ≤ 1`, append 'H', now `h=0`
- Position 5: `h=0`, so append 'V'
- Result: "HHVHV"

The algorithm runs in `O(h + v)` time with `O(h + v)` space for storing the result.

## Example Walkthrough

Let's find the 5th lexicographically smallest path to reach destination `(2, 3)`.

**Initial Setup:**
- Destination: `(2, 3)` means we need `v = 2` vertical moves and `h = 3` horizontal moves
- Total moves needed: `2 + 3 = 5`
- We're looking for `k = 5`

**Step-by-step Process:**

**Position 1:**
- Current state: `v = 2`, `h = 3`, `k = 5`
- Since `h > 0`, calculate combinations: `C(2+3-1, 3-1) = C(4, 2) = 6`
- This means there are 6 valid paths if we place 'H' at position 1
- Decision: Since `k = 5 ≤ 6`, the 5th path is among those starting with 'H'
- Action: Append 'H', update `h = 2`
- Result so far: "H"

**Position 2:**
- Current state: `v = 2`, `h = 2`, `k = 5`
- Calculate: `C(2+2-1, 2-1) = C(3, 1) = 3`
- Decision: Since `k = 5 > 3`, the 5th path does NOT have 'H' at position 2
- Action: Append 'V', update `v = 1`, and adjust `k = 5 - 3 = 2`
- Result so far: "HV"

**Position 3:**
- Current state: `v = 1`, `h = 2`, `k = 2`
- Calculate: `C(1+2-1, 2-1) = C(2, 1) = 2`
- Decision: Since `k = 2 ≤ 2`, place 'H' at position 3
- Action: Append 'H', update `h = 1`
- Result so far: "HVH"

**Position 4:**
- Current state: `v = 1`, `h = 1`, `k = 2`
- Calculate: `C(1+1-1, 1-1) = C(1, 0) = 1`
- Decision: Since `k = 2 > 1`, place 'V' at position 4
- Action: Append 'V', update `v = 0`, and adjust `k = 2 - 1 = 1`
- Result so far: "HVHV"

**Position 5:**
- Current state: `v = 0`, `h = 1`, `k = 1`
- Since `v = 0`, we can only place 'H'
- Action: Append 'H', update `h = 0`
- Final result: "HVHVH"

**Verification:**
The 5th lexicographically smallest path to `(2, 3)` is "HVHVH", which correctly uses 3 'H's (horizontal moves) and 2 'V's (vertical moves) to reach the destination.

## Python Solution

```python
from math import comb
from typing import List

class Solution:
    def kthSmallestPath(self, destination: List[int], k: int) -> str:
        """
        Find the k-th lexicographically smallest path from (0,0) to destination.
        Path consists of 'H' (horizontal right) and 'V' (vertical down) moves.
      
        Args:
            destination: Target coordinates [row, column]
            k: The k-th smallest path to find (1-indexed)
      
        Returns:
            String representing the k-th smallest path
        """
        # Extract vertical and horizontal distances to destination
        vertical_moves, horizontal_moves = destination
        result_path = []
      
        # Build the path character by character
        for _ in range(horizontal_moves + vertical_moves):
            if horizontal_moves == 0:
                # No more horizontal moves left, must go vertical
                result_path.append("V")
            else:
                # Calculate number of paths that start with 'H'
                # This is the number of ways to arrange remaining moves after placing 'H'
                paths_starting_with_h = comb(horizontal_moves + vertical_moves - 1, 
                                            horizontal_moves - 1)
              
                if k > paths_starting_with_h:
                    # k-th path starts with 'V' since it's beyond paths starting with 'H'
                    result_path.append("V")
                    vertical_moves -= 1
                    # Adjust k to find position among paths starting with 'V'
                    k -= paths_starting_with_h
                else:
                    # k-th path starts with 'H'
                    result_path.append("H")
                    horizontal_moves -= 1
      
        return "".join(result_path)
```

## Time and Space Complexity

**Time Complexity:** `O((h + v)²)` or `O(n²)` where `n = h + v`

The algorithm iterates through a loop `h + v` times. In each iteration, it computes `comb(h + v - 1, h - 1)`. The combination function `comb(n, k)` has a time complexity of `O(min(k, n-k))`. In the worst case, when `h` and `v` are roughly equal, this becomes `O(h + v)` for each combination calculation. Since we perform this calculation up to `h + v` times, the overall time complexity is `O((h + v)²)`.

**Space Complexity:** `O(h + v)` or `O(n)` where `n = h + v`

The space complexity is determined by:
- The `ans` list which stores exactly `h + v` characters: `O(h + v)`
- The variables `v`, `h`, `k`, and `x` which use constant space: `O(1)`
- The final string created by `"".join(ans)`: `O(h + v)`

Therefore, the total space complexity is `O(h + v)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Off-by-One Error in Combination Calculation**

**Pitfall**: A common mistake is using the wrong parameters for the combination calculation, such as `comb(h + v, h)` instead of `comb(h + v - 1, h - 1)`. This error stems from misunderstanding what we're counting - we need the number of ways to arrange the *remaining* moves after placing the current character.

**Why it happens**: When we place an 'H' at the current position, we're left with `h-1` horizontal moves and `v` vertical moves to arrange. The total positions remaining is `(h-1) + v = h + v - 1`, and we need to choose `h-1` positions for the remaining 'H' characters.

**Solution**: Always remember that after placing a character, you're counting arrangements of the *remaining* characters. Use `comb(h + v - 1, h - 1)` when determining if the current position should be 'H'.

### 2. **Not Updating k When Choosing 'V'**

**Pitfall**: Forgetting to update `k` when placing a 'V' character. Some might only decrement `v` but forget to adjust `k -= paths_starting_with_h`.

**Why it matters**: The value of `k` represents which path we're looking for among the remaining possibilities. When we skip all paths starting with 'H' (by choosing 'V'), we need to adjust our target position among the paths that start with 'V'.

**Solution**: Always update `k` when choosing 'V':
```python
if k > paths_starting_with_h:
    result_path.append("V")
    vertical_moves -= 1
    k -= paths_starting_with_h  # Don't forget this line!
```

### 3. **Integer Overflow in Large Combinations**

**Pitfall**: For large grids, combination values can become extremely large. While Python handles big integers naturally, in other languages this could cause overflow.

**Example**: For a 100x100 grid, `C(199, 99)` is an enormous number that would overflow standard integer types in languages like Java or C++.

**Solution**: 
- In Python: The built-in `math.comb` handles large numbers automatically
- In other languages: Use specialized libraries for big integers or implement a solution that avoids calculating the full combination value (comparing ratios instead)

### 4. **Incorrect Base Case Handling**

**Pitfall**: Not properly handling the edge case when `horizontal_moves == 0`. Some implementations might try to calculate combinations even when no horizontal moves remain, leading to errors or incorrect results.

**Solution**: Always check if `horizontal_moves == 0` before calculating combinations:
```python
if horizontal_moves == 0:
    result_path.append("V")
    # No need to update k or calculate combinations
else:
    # Safe to calculate combinations here
    paths_starting_with_h = comb(horizontal_moves + vertical_moves - 1, 
                                horizontal_moves - 1)
```

