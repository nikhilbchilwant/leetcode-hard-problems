# 1388. Pizza With 3n Slices

> Source: https://algo.monster/liteproblems/1388

## Problem Statement

You have a circular pizza divided into `3n` slices, where each slice has a different size represented by an integer value. You're playing a game with two friends (Alice and Bob) to divide the pizza following these rules:

1. You pick any slice of pizza to start
2. Alice immediately takes the slice next to yours in the counter-clockwise direction
3. Bob immediately takes the slice next to yours in the clockwise direction  
4. This process repeats - you pick another available slice, then Alice and Bob take the adjacent slices
5. The game continues until all slices are taken

Given an array `slices` where `slices[i]` represents the size of the i-th slice in clockwise order around the circular pizza, you need to find the maximum total sum of slice sizes you can obtain.

The key insight is that this problem can be transformed into selecting `n` non-adjacent elements from a circular array of size `3n` to maximize their sum. This is because:
- When you pick a slice, Alice and Bob take the adjacent slices, making those three slices unavailable
- You get to pick `n` times total (since `3n` slices divided among 3 people gives `n` slices each)
- Your selections cannot be adjacent in the original circle (since picking one slice removes its neighbors)

The circular nature of the array means if you select the first element, you cannot select the last element (they're adjacent in a circle). Therefore, the solution considers two cases: either exclude the first element or exclude the last element from consideration, then solve for the maximum sum in each linear array case.

## Intuition

The first observation is understanding what happens when you pick a slice. Once you choose a slice, Alice and Bob immediately take the slices on either side, effectively removing three consecutive slices from play. This means you can never pick two adjacent slices in the original circle.

Since there are `3n` total slices and three people, each person gets exactly `n` slices. The challenge is to maximize your `n` selections while ensuring no two of your picks are adjacent in the original circular arrangement.

This constraint - selecting `n` elements from `3n` elements where no two selections are adjacent - is a classic [dynamic programming](/problems/dynamic_programming_intro) pattern similar to the "House Robber" problem. However, the circular nature adds complexity: in a circle, the first and last elements are neighbors.

To handle the circular constraint, we can break it down into two simpler linear problems:
1. Consider slices `[0, 1, 2, ..., 3n-2]` (excluding the last slice)
2. Consider slices `[1, 2, 3, ..., 3n-1]` (excluding the first slice)

By solving both cases separately, we ensure that we never pick both the first and last slices together (which would violate the adjacency rule in a circle). The maximum of these two solutions gives us the answer.

For each linear array, we use [dynamic programming](/problems/dynamic_programming_intro) where `f[i][j]` represents the maximum sum when considering the first `i` elements and selecting exactly `j` non-adjacent elements. At each position, we have two choices:
- Skip the current element: `f[i][j] = f[i-1][j]`
- Take the current element (if we didn't take the previous one): `f[i][j] = f[i-2][j-1] + nums[i-1]`

The maximum of these two choices gives us the optimal solution for that state.

**Learn more about [Greedy](/problems/greedy_intro), [Dynamic Programming](/problems/dynamic_programming_intro) and [Heap (Priority Queue)](/problems/heap_intro) patterns.**

## Solution Approach

The solution implements the [dynamic programming](/problems/dynamic_programming_intro) approach through a helper function `g(nums)` that solves the linear version of the problem: selecting `n` non-adjacent elements from an array to maximize their sum.

**Main Function Structure:**
- Calculate `n = len(slices) // 3` to determine how many slices you get to pick
- Call `g(slices[:-1])` to solve for the case excluding the last element
- Call `g(slices[1:])` to solve for the case excluding the first element  
- Return the maximum of these two results

**Helper Function `g(nums)` Implementation:**

The function uses a 2D DP table `f` where `f[i][j]` represents the maximum sum when considering the first `i` elements and selecting exactly `j` non-adjacent elements.

1. **Initialization:** Create a table `f` of size `(m+1) × (n+1)` initialized with zeros, where `m = len(nums)`

2. **State Transition:** For each position `i` from 1 to `m` and each count `j` from 1 to `n`:
   ```
   f[i][j] = max(
       f[i-1][j],                                    # Don't take element i
       (f[i-2][j-1] if i >= 2 else 0) + nums[i-1]  # Take element i
   )
   ```

   - `f[i-1][j]`: Maximum sum without taking the current element
   - `f[i-2][j-1] + nums[i-1]`: Maximum sum when taking the current element (we must skip the previous element, hence `i-2`, and we've selected one more element, hence `j-1`)

3. **Edge Case Handling:** When `i < 2`, we handle it specially by using 0 for `f[i-2][j-1]` since there's no element two positions back

4. **Result:** Return `f[m][n]`, which represents the maximum sum when considering all `m` elements and selecting exactly `n` non-adjacent ones

**Time Complexity:** `O(n²)` for each call to `g()`, and we call it twice, so overall `O(n²)`

**Space Complexity:** `O(n²)` for the DP table

The elegance of this solution lies in transforming a complex circular selection problem into two simpler linear problems, then applying classical [dynamic programming](/problems/dynamic_programming_intro) to solve each one optimally.

## Example Walkthrough

Let's walk through a small example with `slices = [3, 1, 4, 2, 5, 6]` (6 slices total, so n = 2).

**Step 1: Understanding the Problem**
- We have a circular pizza with 6 slices
- We need to pick 2 slices (n = 6/3 = 2)
- When we pick a slice, our friends take the adjacent ones
- We want to maximize our total

**Step 2: Transform to Linear Problems**
Since the array is circular, slices[0] and slices[5] are adjacent. We handle this by solving two cases:
- Case 1: Consider slices `[3, 1, 4, 2, 5]` (exclude last element 6)
- Case 2: Consider slices `[1, 4, 2, 5, 6]` (exclude first element 3)

**Step 3: Solve Case 1 - `g([3, 1, 4, 2, 5])`**
We need to select 2 non-adjacent elements from `[3, 1, 4, 2, 5]`.

Building our DP table `f[i][j]` where i = elements considered, j = elements selected:

| i\j | 0 | 1 | 2 |
|-----|---|---|---|
| 0   | 0 | 0 | 0 |
| 1   | 0 | 3 | 0 |
| 2   | 0 | 3 | 0 |
| 3   | 0 | 4 | 7 |
| 4   | 0 | 4 | 7 |
| 5   | 0 | 5 | 9 |

Key calculations:
- `f[3][2] = max(f[2][2], f[1][1] + 4) = max(0, 3 + 4) = 7` (picking elements at indices 0 and 2: values 3 and 4)
- `f[5][2] = max(f[4][2], f[3][1] + 5) = max(7, 4 + 5) = 9` (picking elements at indices 2 and 4: values 4 and 5)

Case 1 maximum: **9**

**Step 4: Solve Case 2 - `g([1, 4, 2, 5, 6])`**
We need to select 2 non-adjacent elements from `[1, 4, 2, 5, 6]`.

Building our DP table:

| i\j | 0 | 1 | 2 |
|-----|---|---|---|
| 0   | 0 | 0 | 0 |
| 1   | 0 | 1 | 0 |
| 2   | 0 | 4 | 0 |
| 3   | 0 | 4 | 3 |
| 4   | 0 | 5 | 9 |
| 5   | 0 | 6 | 10 |

Key calculations:
- `f[4][2] = max(f[3][2], f[2][1] + 5) = max(3, 4 + 5) = 9` (picking elements at indices 1 and 3: values 4 and 5)
- `f[5][2] = max(f[4][2], f[3][1] + 6) = max(9, 4 + 6) = 10` (picking elements at indices 1 and 4: values 4 and 6)

Case 2 maximum: **10**

**Step 5: Final Answer**
The maximum of Case 1 (9) and Case 2 (10) is **10**.

This means we should pick slices with values 4 and 6 from the original circular arrangement (indices 2 and 5), which are indeed non-adjacent in the circle, giving us a total of 10.

## Python Solution

```python
class Solution:
    def maxSizeSlices(self, slices: List[int]) -> int:
        """
        Find the maximum sum of n non-adjacent slices from a circular array.
        This is equivalent to choosing n/3 slices from n total slices where no two are adjacent.
      
        Args:
            slices: List of pizza slice sizes arranged in a circle
          
        Returns:
            Maximum sum of sizes of n/3 non-adjacent slices
        """
        def max_sum_non_adjacent(nums: List[int]) -> int:
            """
            Helper function to find maximum sum of selecting 'slices_to_pick' elements
            from a linear array where no two selected elements are adjacent.
          
            Args:
                nums: Linear array of numbers
              
            Returns:
                Maximum sum of non-adjacent elements
            """
            array_length = len(nums)
          
            # dp[i][j] represents max sum when considering first i elements 
            # and selecting exactly j elements
            dp = [[0] * (slices_to_pick + 1) for _ in range(array_length + 1)]
          
            # Fill the DP table
            for i in range(1, array_length + 1):
                for j in range(1, slices_to_pick + 1):
                    # Option 1: Don't take current element
                    dont_take = dp[i - 1][j]
                  
                    # Option 2: Take current element (if possible)
                    # We need at least 2 elements to have a previous non-adjacent element
                    take = (dp[i - 2][j - 1] if i >= 2 else 0) + nums[i - 1]
                  
                    dp[i][j] = max(dont_take, take)
          
            return dp[array_length][slices_to_pick]
      
        # Calculate number of slices we need to pick (n/3 of total)
        slices_to_pick = len(slices) // 3
      
        # Handle circular array by considering two cases:
        # Case 1: Exclude last element (so first and last aren't both selected)
        max_without_last = max_sum_non_adjacent(slices[:-1])
      
        # Case 2: Exclude first element (so first and last aren't both selected)
        max_without_first = max_sum_non_adjacent(slices[1:])
      
        # Return the maximum of both cases
        return max(max_without_last, max_without_first)
```

## Time and Space Complexity

The time complexity is `O(n²)`, where `n` is the length of the array `slices`.

The function `g` is called twice, each processing an array of length `n-1`. Inside function `g`:
- There are two nested loops: the outer loop runs `m` times (where `m = n-1`), and the inner loop runs `n/3` times
- Since `n/3` is proportional to `n`, the nested loops result in `O(m × n/3) = O(n × n) = O(n²)` operations
- Each operation inside the loops takes constant time `O(1)`

The space complexity is `O(n²)`.

The 2D array `f` has dimensions `(m+1) × (n+1)`, where `m = n-1` and `n` in the context of function `g` is `len(slices) // 3`. This creates an array of size approximately `n × (n/3) = O(n²)`. The function `g` is called twice, but the space is not accumulated as each call creates its own local array, so the overall space complexity remains `O(n²)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Handling of the Circular Constraint**

**Pitfall:** A common mistake is trying to solve the circular array problem directly without breaking it into two linear cases. Developers often attempt to modify the DP transition to handle the circular constraint within a single pass, which leads to incorrect results.

**Why it happens:** The circular constraint (first and last elements cannot both be selected) creates a dependency that's difficult to handle in a single DP formulation.

**Solution:** Always break the circular array into two linear arrays:
- Case 1: Consider elements `[0, n-2]` (exclude last)
- Case 2: Consider elements `[1, n-1]` (exclude first)

Then solve both cases independently and take the maximum.

### 2. **Off-by-One Errors in DP Table Indexing**

**Pitfall:** Mixing up 0-based array indexing with 1-based DP table indexing, leading to accessing wrong elements or array out of bounds errors.

```python
# WRONG: Forgetting to subtract 1 when accessing array
take = dp[i-2][j-1] + nums[i]  # nums[i] would be out of bounds!

# CORRECT: Proper index adjustment
take = dp[i-2][j-1] + nums[i-1]  # i is 1-based, nums is 0-based
```

**Solution:** Be consistent with indexing. If using 1-based indexing for DP table (where `dp[i]` represents considering first `i` elements), always use `nums[i-1]` to access the actual array element.

### 3. **Forgetting Edge Cases in DP Transitions**

**Pitfall:** Not handling the case when `i < 2` in the DP transition, causing index errors when trying to access `dp[i-2][j-1]`.

```python
# WRONG: No boundary check
take = dp[i-2][j-1] + nums[i-1]  # Fails when i = 1

# CORRECT: Proper boundary handling
take = (dp[i-2][j-1] if i >= 2 else 0) + nums[i-1]
```

**Solution:** Always check boundaries when accessing previous states in DP. When `i < 2`, there's no valid "two positions back" state, so use 0 as the base case.

### 4. **Incorrect Calculation of Number of Selections**

**Pitfall:** Misunderstanding how many slices you get to pick. Some might think you pick `n` slices from `n` total, but the problem states you have `3n` slices and pick `n` of them.

```python
# WRONG: Using wrong number of selections
slices_to_pick = len(slices)  # This would be 3n, not n!

# CORRECT: Proper calculation
slices_to_pick = len(slices) // 3  # This gives n
```

**Solution:** Carefully read the problem statement. You have `3n` total slices divided among 3 people, so each person gets `n` slices.

### 5. **Space Optimization Confusion**

**Pitfall:** Attempting to optimize space from O(n²) to O(n) without fully understanding the dependencies, leading to incorrect results.

**Why it's tricky:** While the transition only depends on the previous two rows (`dp[i-1]` and `dp[i-2]`), you need to track different values for different `j` values, making the optimization non-trivial.

**Solution:** If space optimization is needed, maintain two arrays representing the previous two rows and carefully update them. However, for interview settings, the O(n²) solution is usually acceptable and less error-prone.

