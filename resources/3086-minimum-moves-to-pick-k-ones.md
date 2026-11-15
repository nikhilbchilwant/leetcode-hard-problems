# 3086. Minimum Moves to Pick K Ones

> Source: https://algo.monster/liteproblems/3086

## Problem Statement

You are given a binary array `nums` of length `n`, a positive integer `k`, and a non-negative integer `maxChanges`.

Alice plays a game where she needs to collect exactly `k` ones from the array using the minimum number of moves. 

**Game Rules:**
1. Alice starts by choosing any index `aliceIndex` in the range `[0, n-1]` to stand at
2. If `nums[aliceIndex] == 1`, Alice immediately picks up that one (setting `nums[aliceIndex] = 0`) without using any moves

**Available Actions (each counts as one move):**
- **Action 1:** Select any index `j ≠ aliceIndex` where `nums[j] == 0` and change it to `1`. This action can be performed at most `maxChanges` times.
- **Action 2:** Select two adjacent indices `x` and `y` (where `|x - y| == 1`) such that `nums[x] == 1` and `nums[y] == 0`, then swap their values. If after swapping `y == aliceIndex`, Alice automatically picks up the one at position `y` (setting it back to `0`).

The goal is to find the minimum number of moves Alice needs to collect exactly `k` ones.

**Example Strategy:**
- Alice can use Action 1 to create ones at convenient locations (limited by `maxChanges`)
- Alice can use Action 2 to slide existing ones toward her position through adjacent swaps
- The challenge is finding the optimal starting position and combination of actions to minimize total moves

The problem requires determining the best starting position for Alice and the optimal sequence of actions to collect `k` ones with the fewest moves possible.

## Intuition

The key insight is that we should try every possible starting position for Alice and find the one that minimizes the total moves.

For any starting position `i`, we can think about the most efficient order to collect ones:

1. **Free pickup**: If there's already a one at position `i`, we get it for free (0 moves).

2. **Adjacent ones**: The next cheapest ones to collect are those immediately adjacent to Alice (at positions `i-1` and `i+1`). Each of these only costs 1 move since we can directly swap them to Alice's position.

3. **Creating ones nearby**: After grabbing adjacent ones, we should consider using Action 1 (creating new ones from zeros). The optimal strategy is to create these ones at positions `i-1` or `i+1` (if they're currently 0), then immediately swap them to position `i`. This costs 2 moves per one collected (1 move to create, 1 move to swap).

4. **Distant ones**: Only after exhausting our `maxChanges` should we consider moving ones from farther away. Moving a one from distance `d` costs `d` moves (we need `d` swaps to bring it to Alice).

The crucial realization is that for distant ones, we want to minimize the total distance traveled. This means we should collect the closest ones first. We can use binary search to find the minimum radius around position `i` that contains enough ones to reach our target `k`.

To efficiently calculate the cost of moving distant ones, we use prefix sums:
- `cnt[i]`: count of ones up to position `i` 
- `s[i]`: sum of positions of all ones up to position `i`

The cost to move all ones in range `[l, r]` to position `i` is:
- For ones to the left of `i`: `count * i - sum_of_positions`
- For ones to the right of `i`: `sum_of_positions - count * i`

By trying all starting positions and applying this [greedy](/problems/greedy_intro) strategy at each position, we can find the global minimum number of moves needed.

**Learn more about [Greedy](/problems/greedy_intro), [Prefix Sum](/problems/subarray_sum) and [Sliding Window](/problems/sliding_window_maximum) patterns.**

## Solution Approach

The solution implements the [greedy](/problems/greedy_intro) strategy with prefix sums and binary search for efficiency.

**Step 1: Build Prefix Arrays**
```python
cnt = [0] * (n + 1)  # cnt[i] = count of ones in nums[0:i]
s = [0] * (n + 1)    # s[i] = sum of positions of ones in nums[0:i]
```
We use 1-based indexing for easier calculation. For each position `i`, we track:
- `cnt[i]`: total number of ones up to position `i`
- `s[i]`: sum of all positions where ones appear up to position `i`

**Step 2: Enumerate Each Starting Position**
For each possible starting position `i` (1 to n):

1. **Pick up the free one** if `nums[i-1] == 1`:
   - Set `need = k - x` where `x` is 1 if there's a one at position `i`, else 0
   - Cost: 0 moves

2. **Collect adjacent ones** (positions `i-1` and `i+1`):
   ```python
   for j in (i - 1, i + 1):
       if need > 0 and 1 <= j <= n and nums[j - 1] == 1:
           need -= 1
           t += 1  # Each adjacent one costs 1 move
   ```

3. **Use Action 1 to create ones**:
   ```python
   c = min(need, maxChanges)
   need -= c
   t += c * 2  # Each created one costs 2 moves (create + swap)
   ```
   We create at most `min(need, maxChanges)` ones and place them optimally at adjacent positions.

4. **If still need more ones, use binary search** to find the minimum radius:
   ```python
   l, r = 2, max(i - 1, n - i)
   while l <= r:
       mid = (l + r) >> 1
   ```
   - Search for the smallest distance `mid` such that intervals `[i-mid, i-2]` and `[i+2, i+mid]` contain at least `need` remaining ones
   - Calculate ranges excluding adjacent positions (already processed):
     - Left range: `[max(1, i-mid), max(0, i-2)]`
     - Right range: `[min(n+1, i+2), min(n, i+mid)]`

5. **Calculate the cost using prefix sums**:
   ```python
   c1 = cnt[r1] - cnt[l1 - 1]  # Count of ones on the left
   c2 = cnt[r2] - cnt[l2 - 1]  # Count of ones on the right
 
   if c1 + c2 >= need:
       t1 = c1 * i - (s[r1] - s[l1 - 1])  # Cost for left ones
       t2 = s[r2] - s[l2 - 1] - c2 * i    # Cost for right ones
   ```
   - For ones to the left of `i`: each one at position `j` costs `i - j` moves
   - For ones to the right of `i`: each one at position `j` costs `j - i` moves
   - Total cost = `sum of distances` = `count * i - sum_of_positions` (left) + `sum_of_positions - count * i` (right)

**Step 3: Track Minimum**
After calculating the cost for each starting position, we keep track of the minimum:
```python
ans = min(ans, t + t1 + t2)
```

The algorithm returns the minimum number of moves across all possible starting positions.

## Example Walkthrough

Let's walk through a concrete example to illustrate the solution approach.

**Example:** `nums = [1, 0, 0, 1, 0, 1]`, `k = 2`, `maxChanges = 1`

We need to collect exactly 2 ones using the minimum number of moves.

**Step 1: Build Prefix Arrays**
- Original array (1-indexed): `[1, 0, 0, 1, 0, 1]`
- Ones are at positions: 1, 4, 6
- `cnt = [0, 1, 1, 1, 2, 2, 3]` (cumulative count of ones)
- `s = [0, 1, 1, 1, 5, 5, 11]` (cumulative sum of positions with ones)

**Step 2: Try Each Starting Position**

Let's examine starting position `i = 4` (index 3 in 0-based):

1. **Free pickup**: `nums[3] = 1`, so Alice gets 1 one for free
   - `need = k - 1 = 2 - 1 = 1`
   - Cost so far: 0 moves

2. **Check adjacent positions** (i-1=3 and i+1=5):
   - Position 3: `nums[2] = 0` (no one here)
   - Position 5: `nums[4] = 0` (no one here)
   - Still need: 1 one
   - Cost so far: 0 moves

3. **Use Action 1** (create ones):
   - Can create `min(need, maxChanges) = min(1, 1) = 1` one
   - Create a one at position 3 or 5, then swap to position 4
   - `need = 1 - 1 = 0`
   - Cost: 0 + 1×2 = 2 moves total

4. **Binary search for distant ones**: Not needed since `need = 0`

Total cost for starting at position 4: **2 moves**

Let's also try starting position `i = 1`:

1. **Free pickup**: `nums[0] = 1`, get 1 one for free
   - `need = 2 - 1 = 1`
   - Cost: 0 moves

2. **Check adjacent positions** (only i+1=2 since i-1=0 is out of bounds):
   - Position 2: `nums[1] = 0` (no one here)
   - Still need: 1 one
   - Cost: 0 moves

3. **Use Action 1**:
   - Create `min(1, 1) = 1` one at position 2
   - Swap to position 1
   - `need = 1 - 1 = 0`
   - Cost: 0 + 1×2 = 2 moves

Total cost for starting at position 1: **2 moves**

Let's try starting position `i = 2` (where there's no initial one):

1. **Free pickup**: `nums[1] = 0`, no free one
   - `need = 2`
   - Cost: 0 moves

2. **Check adjacent positions** (i-1=1 and i+1=3):
   - Position 1: `nums[0] = 1` - can swap this one!
   - `need = 2 - 1 = 1`
   - Cost: 0 + 1 = 1 move
   - Position 3: `nums[2] = 0` (no one here)

3. **Use Action 1**:
   - Create `min(1, 1) = 1` one
   - `need = 1 - 1 = 0`
   - Cost: 1 + 1×2 = 3 moves

Total cost for starting at position 2: **3 moves**

**Step 3: Find Minimum**
After checking all positions, the minimum is **2 moves** (achieved at positions 1 and 4).

The algorithm correctly identifies that starting at a position with an existing one and using our single `maxChanges` to create and collect another one nearby is optimal for this example.

## Python Solution

```python
class Solution:
    def minimumMoves(self, nums: List[int], k: int, maxChanges: int) -> int:
        n = len(nums)
      
        # Prefix arrays for count and weighted sum
        # cnt[i] = count of 1s in nums[0:i]
        # weighted_sum[i] = sum of (index+1) * value for nums[0:i]
        cnt = [0] * (n + 1)
        weighted_sum = [0] * (n + 1)
      
        for i, val in enumerate(nums, 1):
            cnt[i] = cnt[i - 1] + val
            weighted_sum[i] = weighted_sum[i - 1] + i * val
      
        min_moves = float('inf')
      
        # Try each position as the collection point
        for i, val in enumerate(nums, 1):
            moves = 0
            remaining_needed = k - val  # How many more 1s we need after taking current position
          
            # First, try to collect from immediate neighbors (cost = 1 move each)
            for neighbor_pos in (i - 1, i + 1):
                if remaining_needed > 0 and 1 <= neighbor_pos <= n and nums[neighbor_pos - 1] == 1:
                    remaining_needed -= 1
                    moves += 1
          
            # Use maxChanges to create 1s and immediately collect them (cost = 2 moves each)
            changes_to_use = min(remaining_needed, maxChanges)
            remaining_needed -= changes_to_use
            moves += changes_to_use * 2
          
            # If we've collected enough, update answer
            if remaining_needed <= 0:
                min_moves = min(min_moves, moves)
                continue
          
            # Binary search for the minimum radius to collect remaining_needed 1s
            left, right = 2, max(i - 1, n - i)
          
            while left <= right:
                mid_radius = (left + right) >> 1
              
                # Define ranges for collecting (excluding immediate neighbors already considered)
                left_range_start = max(1, i - mid_radius)
                left_range_end = max(0, i - 2)
                right_range_start = min(n + 1, i + 2)
                right_range_end = min(n, i + mid_radius)
              
                # Count 1s in left and right ranges
                left_count = cnt[left_range_end] - cnt[left_range_start - 1]
                right_count = cnt[right_range_end] - cnt[right_range_start - 1]
              
                if left_count + right_count >= remaining_needed:
                    # Calculate cost to move 1s from left range to position i
                    left_cost = left_count * i - (weighted_sum[left_range_end] - weighted_sum[left_range_start - 1])
                    # Calculate cost to move 1s from right range to position i
                    right_cost = weighted_sum[right_range_end] - weighted_sum[right_range_start - 1] - right_count * i
                  
                    min_moves = min(min_moves, moves + left_cost + right_cost)
                    right = mid_radius - 1  # Try smaller radius
                else:
                    left = mid_radius + 1  # Need larger radius
      
        return min_moves
```

## Time and Space Complexity

**Time Complexity: `O(n × log n)`**

The algorithm iterates through each position `i` in the array (outer loop runs `n` times). For each position, it performs a binary search to find the optimal range for collecting ones. The binary search runs in `O(log n)` time since it searches within a range from 2 to `max(i - 1, n - i)`, which is at most `n`. All operations inside the binary search (calculating prefix sums, counting ones) are done in constant time due to preprocessing. Therefore, the overall time complexity is `O(n × log n)`.

**Space Complexity: `O(n)`**

The algorithm uses two auxiliary arrays: `cnt` (prefix sum array for counting ones) and `s` (weighted prefix sum array), each of size `n + 1`. These arrays require `O(n)` space. The remaining variables (`ans`, `t`, `need`, `l`, `r`, etc.) use constant space. Thus, the total space complexity is `O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Handling of Partial Collection from Ranges**

**The Pitfall:** 
The current implementation calculates the cost assuming we take ALL ones from both the left and right ranges when `left_count + right_count >= remaining_needed`. However, we only need `remaining_needed` ones, not all available ones. This leads to an overestimation of the minimum cost.

**Why It Happens:**
When binary searching for the minimum radius, once we find a radius where enough ones are available, we calculate:
```python
left_cost = left_count * i - (weighted_sum[left_range_end] - weighted_sum[left_range_start - 1])
right_cost = weighted_sum[right_range_end] - weighted_sum[right_range_start - 1] - right_count * i
```
This computes the cost to move ALL `left_count + right_count` ones, but we might only need a subset.

**Example Scenario:**
- Position i = 5, remaining_needed = 3
- Left range has 2 ones at positions [2, 3]
- Right range has 4 ones at positions [7, 8, 9, 10]
- Current code calculates cost for all 6 ones, but we only need 3

**Solution:**
We need to select the closest `remaining_needed` ones from the combined ranges. The optimal strategy is to:
1. Sort all available ones by their distance from position `i`
2. Select the `remaining_needed` closest ones
3. Calculate the actual cost for only those selected ones

**Corrected Code Segment:**
```python
if left_count + right_count >= remaining_needed:
    # Collect all ones with their distances
    ones_with_dist = []
  
    # Add ones from left range
    for j in range(left_range_start, left_range_end + 1):
        if nums[j - 1] == 1:
            ones_with_dist.append(i - j)  # distance to position i
  
    # Add ones from right range  
    for j in range(right_range_start, right_range_end + 1):
        if nums[j - 1] == 1:
            ones_with_dist.append(j - i)  # distance to position i
  
    # Sort by distance and take the closest remaining_needed ones
    ones_with_dist.sort()
    actual_cost = sum(ones_with_dist[:remaining_needed])
  
    min_moves = min(min_moves, moves + actual_cost)
    right = mid_radius - 1
```

### 2. **Alternative Efficient Solution Using Sliding Window**

Instead of iterating through positions in the binary search, a more efficient approach maintains sorted positions of all ones and uses a sliding window:

```python
def minimumMoves(self, nums: List[int], k: int, maxChanges: int) -> int:
    # Collect positions of all ones
    ones_positions = [i for i, val in enumerate(nums) if val == 1]
    m = len(ones_positions)
  
    if m == 0:
        return 2 * k  # All ones must be created using maxChanges
  
    # Prefix sum for efficient range sum calculation
    prefix = [0] * (m + 1)
    for i in range(m):
        prefix[i + 1] = prefix[i] + ones_positions[i]
  
    min_moves = float('inf')
  
    # For each possible collection point
    for alice_pos in range(len(nums)):
        moves = 0
        remaining = k
      
        # Take one at current position if available
        if nums[alice_pos] == 1:
            remaining -= 1
      
        # Use maxChanges optimally (create adjacent ones)
        created = min(remaining, maxChanges)
        remaining -= created
        moves += created * 2
      
        if remaining == 0:
            min_moves = min(min_moves, moves)
            continue
      
        # Find the cost to collect 'remaining' closest ones
        # Use sliding window on ones_positions
        for window_size in range(1, min(remaining, m) + 1):
            for start in range(m - window_size + 1):
                end = start + window_size
                median_idx = (start + end) // 2
                median_pos = ones_positions[median_idx]
              
                # Calculate cost to move all ones in window to median_pos
                cost = 0
                for i in range(start, end):
                    cost += abs(ones_positions[i] - median_pos)
              
                # Then move from median_pos to alice_pos
                cost += abs(median_pos - alice_pos) * window_size
              
                if window_size >= remaining:
                    min_moves = min(min_moves, moves + cost)
  
    return min_moves
```

This approach correctly handles partial collection and provides optimal selection of ones to minimize total movement cost.

