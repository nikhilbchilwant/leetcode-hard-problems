# 1703. Minimum Adjacent Swaps for K Consecutive Ones

> Source: https://algo.monster/liteproblems/1703

## Problem Statement

You have an array `nums` that contains only `0`s and `1`s. You can perform moves where each move allows you to swap two **adjacent** elements in the array.

Your goal is to rearrange the array so that there are `k` **consecutive** `1`s somewhere in the array. You need to find the **minimum** number of moves required to achieve this.

For example, if you have `nums = [1,0,0,1,0,1]` and `k = 2`, you want to get at least one position in the array where two `1`s are next to each other, like `[0,1,1,0,0,1]` or `[1,0,0,0,1,1]`, etc. The problem asks for the minimum number of adjacent swaps needed to create such a configuration.

## Intuition

The key observation is that we only care about the positions of `1`s in the array since we need to group `k` of them together. The `0`s are just obstacles that we need to move past.

First, let's extract all positions where `1`s appear. If we have `1`s at positions `[1, 4, 7, 10]` and want to group `k=3` of them, we need to choose which 3 to group and where to place them.

A crucial insight: when we want to group `k` ones together into consecutive positions, the optimal strategy is to move them towards their **median** position. Why? Because the median minimizes the sum of distances. If we pick any `k` consecutive target positions `[p, p+1, p+2, ..., p+k-1]` and want to move `k` ones from positions `[a₁, a₂, ..., aₖ]` to these spots, the total moves needed is the sum of distances each `1` needs to travel.

Consider this: if we have ones at positions `[2, 5, 9]` and want them consecutive, moving them to positions around `5` (the median) like `[4, 5, 6]` requires fewer moves than moving them to positions around `2` or `9`.

The calculation becomes: for each possible group of `k` ones, find their median position. Then calculate how many swaps are needed to bring all ones in this group to consecutive positions centered around the median. The ones to the left of the median need to move to positions `[median-x+1, ..., median]` and the ones to the right need to move to positions `[median+1, ..., median+y]`.

Using prefix sums helps us quickly calculate the sum of positions for any subarray of ones, which we need to determine the total distance all ones must travel. The formula accounts for both the current positions (from prefix sums) and the target positions (arithmetic sequence around the median).

By trying all possible groups of `k` consecutive ones in our extracted positions array and choosing the one with minimum cost, we get our answer.

**Learn more about [Greedy](/problems/greedy_intro), [Prefix Sum](/problems/subarray_sum) and [Sliding Window](/problems/sliding_window_maximum) patterns.**

## Solution Approach

Let's walk through the implementation step by step:

**Step 1: Extract positions of all 1s**
```python
arr = [i for i, x in enumerate(nums) if x]
```
We create an array `arr` containing all indices where `nums[i] = 1`. For example, if `nums = [1,0,0,1,0,1]`, then `arr = [0, 3, 5]`.

**Step 2: Build [prefix sum](/problems/subarray_sum) array**
```python
s = list(accumulate(arr, initial=0))
```
We compute the prefix sum array `s` where `s[i]` represents the sum of the first `i` elements in `arr`. This allows us to quickly calculate the sum of positions for any subarray of ones in O(1) time.

**Step 3: Set up variables for median-based calculation**
```python
x = (k + 1) // 2  # Number of elements on the left (including median)
y = k - x         # Number of elements on the right
```
For a group of `k` ones, we split them around the median: `x` elements on the left (including the median itself) and `y` elements on the right.

**Step 4: Enumerate all possible median positions**
```python
for i in range(x - 1, len(arr) - y):
```
We iterate through all valid median positions. The median must have at least `x-1` elements before it and `y` elements after it in the `arr` array.

**Step 5: Calculate movement costs for each median**
For each median position `i`:
- `j = arr[i]` is the actual index of the median in the original `nums` array
- `ls = s[i+1] - s[i+1-x]` is the sum of positions of the left `x` ones
- `rs = s[i+1+y] - s[i+1]` is the sum of positions of the right `y` ones

The target positions are:
- Left ones should move to positions `[j-x+1, j-x+2, ..., j]`
- Right ones should move to positions `[j+1, j+2, ..., j+y]`

The cost calculations:
- `a = (j + j - x + 1) * x // 2 - ls`: Cost to move left ones
  - `(j + j - x + 1) * x // 2` is the sum of target positions (arithmetic sequence)
  - Subtract `ls` (current positions) to get the total distance

- `b = rs - (j + 1 + j + y) * y // 2`: Cost to move right ones
  - `(j + 1 + j + y) * y // 2` is the sum of target positions
  - `rs` minus this gives the total distance

**Step 6: Track minimum cost**
```python
ans = min(ans, a + b)
```
We keep track of the minimum total cost across all possible median positions.

The algorithm efficiently finds the optimal grouping of `k` ones by using the median property and prefix sums to achieve O(n) time complexity after the initial setup.

## Example Walkthrough

Let's walk through a concrete example with `nums = [1,0,0,0,1,0,1,1]` and `k = 3`.

**Step 1: Extract positions of 1s**
- The 1s appear at indices: `arr = [0, 4, 6, 7]`
- We have 4 ones total, and need to group 3 of them consecutively

**Step 2: Build prefix sum array**
- `s = [0, 0, 4, 10, 17]` (cumulative sums of positions)
- This helps us quickly calculate sum of positions for any subarray

**Step 3: Set up median split**
- `x = (3 + 1) // 2 = 2` (elements on left including median)
- `y = 3 - 2 = 1` (elements on right of median)

**Step 4 & 5: Try each possible median**

**Option 1: Median at index 1 (position 4 in original array)**
- Group ones at positions `[0, 4, 6]`
- Median position `j = 4`
- Left ones (positions 0, 4) should move to positions [3, 4]
- Right one (position 6) should move to position [5]
- Cost calculation:
  - `ls = s[2] - s[0] = 4 - 0 = 4` (sum of left positions)
  - `rs = s[3] - s[2] = 10 - 4 = 6` (sum of right positions)
  - Left cost: `(4 + 3) * 2 / 2 - 4 = 7 - 4 = 3` moves
  - Right cost: `6 - 5 = 1` move
  - Total: `3 + 1 = 4` moves

**Option 2: Median at index 2 (position 6 in original array)**
- Group ones at positions `[4, 6, 7]`
- Median position `j = 6`
- Left ones (positions 4, 6) should move to positions [5, 6]
- Right one (position 7) should move to position [7]
- Cost calculation:
  - `ls = s[3] - s[1] = 10 - 0 = 10` (sum of left positions)
  - `rs = s[4] - s[3] = 17 - 10 = 7` (sum of right positions)
  - Left cost: `(6 + 5) * 2 / 2 - 10 = 11 - 10 = 1` move
  - Right cost: `7 - 7 = 0` moves
  - Total: `1 + 0 = 1` move

**Step 6: Return minimum**
- Minimum cost is 1 move (Option 2)
- This corresponds to moving the 1 at position 4 to position 5
- Final array: `[1,0,0,0,0,1,1,1]` with 3 consecutive 1s at the end

The algorithm correctly identifies that grouping the last three 1s requires minimal movement, as they're already close together.

## Python Solution

```python
class Solution:
    def minMoves(self, nums: List[int], k: int) -> int:
        # Extract indices where nums[i] == 1
        ones_indices = [i for i, value in enumerate(nums) if value == 1]
      
        # Create prefix sum array for quick range sum calculations
        # prefix_sums[i] contains sum of first i elements from ones_indices
        prefix_sums = list(accumulate(ones_indices, initial=0))
      
        min_moves = float('inf')
      
        # Split k into left and right parts around a center position
        # left_count: number of ones to collect on the left side (including center if k is odd)
        # right_count: number of ones to collect on the right side
        left_count = (k + 1) // 2
        right_count = k - left_count
      
        # Try each possible center position for collecting k ones
        for center_idx in range(left_count - 1, len(ones_indices) - right_count):
            # Get the actual position in nums array for the center
            center_pos = ones_indices[center_idx]
          
            # Calculate sum of positions for left group using prefix sums
            # Elements from index (center_idx - left_count + 1) to center_idx
            left_sum = prefix_sums[center_idx + 1] - prefix_sums[center_idx + 1 - left_count]
          
            # Calculate sum of positions for right group using prefix sums
            # Elements from index (center_idx + 1) to (center_idx + right_count)
            right_sum = prefix_sums[center_idx + 1 + right_count] - prefix_sums[center_idx + 1]
          
            # Calculate moves needed for left group
            # Target positions: center_pos - left_count + 1, ..., center_pos
            # Sum of target positions using arithmetic sequence formula
            left_target_sum = (center_pos + center_pos - left_count + 1) * left_count // 2
            left_moves = left_target_sum - left_sum
          
            # Calculate moves needed for right group
            # Target positions: center_pos + 1, ..., center_pos + right_count
            # Sum of target positions using arithmetic sequence formula
            right_target_sum = (center_pos + 1 + center_pos + right_count) * right_count // 2
            right_moves = right_sum - right_target_sum
          
            # Update minimum moves
            total_moves = left_moves + right_moves
            min_moves = min(min_moves, total_moves)
      
        return min_moves
```

## Time and Space Complexity

**Time Complexity:** `O(n + m)`, which simplifies to `O(n)` since `m ≤ n`

- Creating the `arr` list requires iterating through all `n` elements of `nums` to find positions of 1s: `O(n)`
- Building the prefix sum array `s` using `accumulate` on `arr` of length `m`: `O(m)`
- The main loop iterates `m - k + 1` times (from `x - 1` to `len(arr) - y - 1`), which is `O(m)`
- Each iteration performs constant time operations: `O(1)`
- Overall: `O(n) + O(m) + O(m) = O(n + m) = O(n)`

**Space Complexity:** `O(m)`

- The `arr` list stores indices of all 1s in `nums`, requiring `O(m)` space where `m` is the number of 1s
- The prefix sum array `s` has length `m + 1`: `O(m)`
- Other variables (`ans`, `x`, `y`, `i`, `j`, etc.) use constant space: `O(1)`
- Overall: `O(m) + O(m) + O(1) = O(m)`

Here, `n` is the length of the array `nums` and `m` is the number of 1s in the array `nums`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Off-by-One Errors in Index Calculations**

One of the most common mistakes is incorrectly calculating the range boundaries when iterating through possible center positions or when computing prefix sum ranges.

**Pitfall Example:**
```python
# Incorrect: Missing the -1 adjustment
for center_idx in range(left_count, len(ones_indices) - right_count):
    # This starts one position too late
```

**Correct Approach:**
```python
# Start from left_count - 1 because we need left_count elements total,
# and the center itself is included in that count
for center_idx in range(left_count - 1, len(ones_indices) - right_count):
```

### 2. **Misunderstanding the Median Split for Odd vs Even k**

When k is odd, the median element is included in the left group, but developers often forget this asymmetry.

**Pitfall Example:**
```python
# Incorrect: Treating left and right symmetrically
left_count = k // 2
right_count = k // 2
if k % 2 == 1:
    # Forgot to handle the middle element
```

**Correct Approach:**
```python
left_count = (k + 1) // 2  # Includes median for odd k
right_count = k - left_count  # Automatically handles both cases
```

### 3. **Incorrect Target Position Calculation**

Calculating the sum of target positions using arithmetic sequences can be error-prone, especially with the boundary values.

**Pitfall Example:**
```python
# Incorrect: Wrong arithmetic sequence formula
left_target_sum = (center_pos - left_count + center_pos) * left_count // 2
# Missing the +1 adjustment in the first term
```

**Correct Approach:**
```python
# For positions [center_pos - left_count + 1, ..., center_pos]
# First term: center_pos - left_count + 1
# Last term: center_pos
left_target_sum = (center_pos - left_count + 1 + center_pos) * left_count // 2
```

### 4. **Not Handling Edge Cases**

Failing to check if there are enough 1s in the array or if k equals the total number of 1s.

**Pitfall Example:**
```python
def minMoves(self, nums: List[int], k: int) -> int:
    ones_indices = [i for i, value in enumerate(nums) if value == 1]
    # Directly proceed without checking if len(ones_indices) >= k
```

**Correct Approach:**
```python
def minMoves(self, nums: List[int], k: int) -> int:
    ones_indices = [i for i, value in enumerate(nums) if value == 1]
  
    # Handle edge cases
    if len(ones_indices) < k:
        return -1  # Or handle according to problem constraints
    if k == 1:
        return 0  # Already have a consecutive group of 1
```

### 5. **Confusion Between Index in ones_indices vs Original Array**

Mixing up the index within the `ones_indices` array and the actual position in the original `nums` array.

**Pitfall Example:**
```python
for center_idx in range(left_count - 1, len(ones_indices) - right_count):
    # Incorrect: Using center_idx directly as position
    left_moves = calculate_moves(center_idx, ...)  # Wrong!
```

**Correct Approach:**
```python
for center_idx in range(left_count - 1, len(ones_indices) - right_count):
    # Get actual position in original array
    center_pos = ones_indices[center_idx]
    # Use center_pos for position calculations
    left_moves = calculate_moves(center_pos, ...)
```

### Prevention Strategy:

1. **Draw out small examples** with k=2, k=3 to verify index calculations
2. **Use descriptive variable names** to distinguish between indices and positions
3. **Add assertions** during development to verify intermediate calculations
4. **Test with edge cases** like k=1, k=len(ones_indices), and arrays with all 1s clustered at boundaries

