# 2106. Maximum Fruits Harvested After at Most K Steps

> Source: https://algo.monster/liteproblems/2106

## Problem Statement

You are given an infinite x-axis where fruits are placed at specific positions. The fruit locations and quantities are provided in a 2D array `fruits`, where `fruits[i] = [position_i, amount_i]` means there are `amount_i` fruits at position `position_i`. The array is already sorted by position in ascending order, and each position is unique.

You start at position `startPos` and can walk either left or right along the x-axis. Each step moves you one unit, and you can take at most `k` steps total. Whenever you reach a position with fruits, you automatically harvest all fruits at that position (and they disappear).

The goal is to find the maximum total number of fruits you can harvest given your movement constraint of `k` steps.

For example, if you start at position 5, have 4 steps available, and there are fruits at positions [2, 4, 6, 7], you need to determine the optimal path. You could go left to position 2 (3 steps) then right to position 4 (2 steps back), but that would exceed your step limit. Alternatively, you could go right to positions 6 and 7, using 2 steps total. The key insight is that you can move in one direction, then turn around and go in the opposite direction to maximize fruit collection within your step budget.

The challenge is to efficiently determine which range of fruit positions `[l, r]` you should visit to maximize the total fruits collected, considering that:
- Moving from `startPos` to any range `[l, r]` requires a minimum number of steps
- You might need to backtrack (go in one direction then reverse) to cover a range
- The total steps taken cannot exceed `k`

## Intuition

The key insight is that when we harvest fruits, we're essentially choosing a continuous range of positions `[l, r]` to visit. Why continuous? Because it never makes sense to skip a position with fruits if we're already passing through it - we might as well collect those fruits too since harvesting is free once we're at a position.

Now, for any range `[l, r]` we want to visit starting from `startPos`, we need to figure out the minimum steps required. There are three scenarios:

1. If we start to the right of the range (`startPos >= r`), we simply walk left to `l`, taking `startPos - l` steps
2. If we start to the left of the range (`startPos <= l`), we simply walk right to `r`, taking `r - startPos` steps
3. If we start inside the range (`l < startPos < r`), we have two choices:
   - Go left first to `l`, then turn around and go right to `r`
   - Go right first to `r`, then turn around and go left to `l`

In the third case, we'd choose whichever option requires fewer steps. The total distance is always `r - l` (to cover the range), plus the smaller of the two "extra" distances: either `|startPos - l|` (if we go right first) or `|startPos - r|` (if we go left first).

This gives us a unified formula for minimum steps: `(r - l) + min(|startPos - l|, |startPos - r|)`.

The next realization is that we can use a [sliding window](/problems/sliding_window_maximum) approach. As we fix the right endpoint `r` and move the left endpoint `l` rightward, the minimum steps needed either stays the same or decreases. This monotonic property means we can use two pointers: expand the window by moving the right pointer, and shrink it by moving the left pointer whenever we exceed `k` steps.

By maintaining a running sum of fruits in our current window and updating our maximum whenever we have a valid window (steps <= `k`), we can efficiently find the optimal range in a single pass through the fruits array.

**Learn more about [Binary Search](/problems/binary_search_intro), [Prefix Sum](/problems/subarray_sum) and [Sliding Window](/problems/sliding_window_maximum) patterns.**

## Solution Approach

We implement a two-pointer [sliding window](/problems/sliding_window_maximum) approach to find the optimal range of fruit positions to visit.

**Algorithm Setup:**
- Initialize pointers `i = 0` (left) and `j` iterating through fruits (right)
- Maintain running sum `s` of fruits in current window `[i, j]`
- Track maximum fruits collected in `ans`

**Main Algorithm:**

For each position `j` in the fruits array:

1. **Expand window**: Add `fruits[j]` to our window by updating `s += fruits[j][1]`

2. **Check validity**: Calculate minimum steps needed for current window using the formula:
   ```
   steps = fruits[j][0] - fruits[i][0] + min(|startPos - fruits[i][0]|, |startPos - fruits[j][0]|)
   ```
   This represents the distance to cover the range plus the minimum extra distance based on starting position.

3. **Shrink if needed**: While steps > `k`:
   - Remove leftmost fruits: `s -= fruits[i][1]`
   - Move left pointer right: `i += 1`
   - This maintains the invariant that our window uses at most `k` steps

4. **Update answer**: After ensuring valid window, update `ans = max(ans, s)`

**Why this works:**

The [sliding window](/problems/sliding_window_maximum) is valid because as we fix the right endpoint and move the left endpoint rightward, the minimum steps needed is non-increasing. This monotonic property ensures we never miss the optimal solution.

The algorithm processes each fruit position at most twice (once when entering the window, once when leaving), giving us `O(n)` time complexity where `n` is the number of fruit positions.

**Implementation Details:**

The condition `i <= j` in the while loop ensures we don't have an invalid window. The window shrinking continues until either:
- The window becomes empty (`i > j`), or
- The minimum steps becomes ≤ `k`

This greedy approach guarantees we always maintain the maximum possible window for each right endpoint, ultimately finding the global maximum.

## Example Walkthrough

Let's walk through a concrete example to illustrate the solution approach.

**Given:**
- `fruits = [[0,9],[4,1],[5,7],[6,2],[7,4],[10,9]]` (positions with fruit amounts)
- `startPos = 5` (starting position)
- `k = 4` (maximum steps allowed)

**Goal:** Find maximum fruits we can collect within 4 steps.

**Step-by-step execution:**

**Initial state:** `i = 0`, `s = 0` (sum), `ans = 0` (max fruits)

**j = 0** (position 0, 9 fruits):
- Add to window: `s = 9`
- Calculate steps: `|5-0| + min(|5-0|, |5-0|) = 5 + 5 = 10` steps
- Since 10 > 4, shrink window: remove position 0, `i = 1`, `s = 0`
- Window is now empty

**j = 1** (position 4, 1 fruit):
- Add to window: `s = 1`
- Calculate steps: `|4-4| + min(|5-4|, |5-4|) = 0 + 1 = 1` step
- Since 1 ≤ 4, window is valid
- Update: `ans = max(0, 1) = 1`

**j = 2** (position 5, 7 fruits):
- Add to window: `s = 1 + 7 = 8`
- Window is [4,5], steps: `|5-4| + min(|5-4|, |5-5|) = 1 + 0 = 1` step
- Since 1 ≤ 4, window is valid
- Update: `ans = max(1, 8) = 8`

**j = 3** (position 6, 2 fruits):
- Add to window: `s = 8 + 2 = 10`
- Window is [4,6], steps: `|6-4| + min(|5-4|, |5-6|) = 2 + 1 = 3` steps
- Since 3 ≤ 4, window is valid
- Update: `ans = max(8, 10) = 10`

**j = 4** (position 7, 4 fruits):
- Add to window: `s = 10 + 4 = 14`
- Window is [4,7], steps: `|7-4| + min(|5-4|, |5-7|) = 3 + 1 = 4` steps
- Since 4 ≤ 4, window is valid
- Update: `ans = max(10, 14) = 14`

**j = 5** (position 10, 9 fruits):
- Add to window: `s = 14 + 9 = 23`
- Window is [4,10], steps: `|10-4| + min(|5-4|, |5-10|) = 6 + 1 = 7` steps
- Since 7 > 4, shrink window:
  - Remove position 4: `s = 23 - 1 = 22`, `i = 2`
  - Window is [5,10], steps: `|10-5| + min(|5-5|, |5-10|) = 5 + 0 = 5` steps
  - Still 5 > 4, continue shrinking:
  - Remove position 5: `s = 22 - 7 = 15`, `i = 3`
  - Window is [6,10], steps: `|10-6| + min(|5-6|, |5-10|) = 4 + 1 = 5` steps
  - Still 5 > 4, continue shrinking:
  - Remove position 6: `s = 15 - 2 = 13`, `i = 4`
  - Window is [7,10], steps: `|10-7| + min(|5-7|, |5-10|) = 3 + 2 = 5` steps
  - Still 5 > 4, continue shrinking:
  - Remove position 7: `s = 13 - 4 = 9`, `i = 5`
  - Window is [10], steps: `|10-10| + min(|5-10|, |5-10|) = 0 + 5 = 5` steps
  - Still 5 > 4, continue shrinking:
  - Remove position 10: `s = 0`, `i = 6`
  - Window is empty

**Result:** Maximum fruits = 14

The optimal strategy is to go left from position 5 to position 4 (1 step), then turn around and go right to position 7 (3 more steps), collecting 1+7+2+4 = 14 fruits total.

## Python Solution

```python
class Solution:
    def maxTotalFruits(self, fruits: List[List[int]], startPos: int, k: int) -> int:
        """
        Find the maximum number of fruits that can be collected within k steps.

        Strategy: Use a sliding window to find valid ranges where we can collect all fruits.
        We can move in one direction then backtrack, so the total distance is:
        - Distance to cover the range + minimum distance from start to either endpoint

        Args:
            fruits: List of [position, amount] pairs representing fruit locations
            startPos: Starting position
            k: Maximum number of steps allowed

        Returns:
            Maximum total fruits that can be collected
        """
        max_fruits = 0
        left_index = 0
        current_sum = 0

        # Iterate through each fruit position as the right boundary of our window
        for right_index, (right_pos, right_fruits) in enumerate(fruits):
            # Add fruits at the current right position to our sum
            current_sum += right_fruits

            # Shrink the window from the left while the range is invalid
            # The range is invalid if the minimum steps needed exceeds k
            while (left_index <= right_index and
                   self._calculate_min_steps(fruits, left_index, right_index, startPos) > k):
                # Remove the leftmost fruits from our sum
                current_sum -= fruits[left_index][1]
                left_index += 1

            # Update the maximum fruits collected
            max_fruits = max(max_fruits, current_sum)

        return max_fruits

    def _calculate_min_steps(self, fruits: List[List[int]], left_idx: int, right_idx: int, start: int) -> int:
        """
        Calculate minimum steps needed to collect all fruits in the range [left_idx, right_idx].

        The optimal strategy is to go to one end first, then backtrack to the other end.
        We choose the direction that minimizes total distance.

        Args:
            fruits: List of fruit positions and amounts
            left_idx: Left boundary index
            right_idx: Right boundary index
            start: Starting position

        Returns:
            Minimum steps needed to cover the range
        """
        left_pos = fruits[left_idx][0]
        right_pos = fruits[right_idx][0]

        # Total range to cover
        range_distance = right_pos - left_pos

        # Minimum distance from start to either endpoint
        # This determines which direction we should go first
        min_endpoint_distance = min(abs(start - left_pos), abs(start - right_pos))

        # Total steps = range distance + backtracking distance
        return range_distance + min_endpoint_distance
```

## Time and Space Complexity

**Time Complexity:** `O(n)`, where `n` is the length of the fruits array.

The algorithm uses a two-pointer sliding window approach. The outer loop iterates through each fruit position with pointer `j`, visiting each element exactly once. The inner while loop moves pointer `i` forward to maintain a valid window. Since both pointers `i` and `j` only move forward and never reset, each element is visited at most twice (once by `j` and once by `i`). This results in a linear time complexity of `O(n)`.

**Space Complexity:** `O(1)`

The algorithm only uses a constant amount of extra space for variables `ans`, `i`, `s`, `j`, `pj`, and `fj`. No additional data structures that scale with the input size are created. The space usage remains constant regardless of the input size, resulting in `O(1)` space complexity.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Distance Calculation Formula**

**Pitfall:** A common mistake is incorrectly calculating the minimum steps needed to cover a range. Developers often compute it as simply the distance from start to left plus the range width, or use formulas like:
- `abs(startPos - fruits[i][0]) + abs(fruits[j][0] - fruits[i][0])` (always go left first)
- `abs(startPos - fruits[j][0]) + abs(fruits[j][0] - fruits[i][0])` (always go right first)

**Why it's wrong:** These approaches don't consider that we can choose the optimal direction. The correct strategy is to go to the closer endpoint first, then traverse to the farther one.

**Solution:** Use the correct formula:
```python
steps = (fruits[j][0] - fruits[i][0]) + min(abs(startPos - fruits[i][0]), abs(startPos - fruits[j][0]))
```
This represents: range_width + minimum_distance_to_either_endpoint

### 2. **Off-by-One Errors in Window Boundaries**

**Pitfall:** Using `<` instead of `<=` in the while loop condition:
```python
while left_index < right_index and ...:  # Wrong!
```

**Why it's wrong:** This prevents checking single-fruit windows (when left_index equals right_index), potentially missing valid solutions where visiting just one fruit position is optimal.

**Solution:** Always use `<=` to include single-element windows:
```python
while left_index <= right_index and ...:
```

### 3. **Forgetting to Handle Edge Cases**

**Pitfall:** Not considering scenarios where:
- The starting position is exactly at a fruit location
- All fruits are on one side of the starting position
- k = 0 (no movement allowed)

**Why it's wrong:** The algorithm works correctly for these cases, but developers sometimes add unnecessary special case handling that introduces bugs.

**Solution:** The sliding window approach naturally handles these cases:
- If startPos is at a fruit location, the distance calculation still works correctly
- If all fruits are on one side, the min() in the distance formula ensures optimal path
- If k = 0, the window shrinks to empty for all non-zero distance ranges

### 4. **Premature Optimization with Binary Search**

**Pitfall:** Trying to optimize by using binary search to find the starting position in the fruits array:
```python
# Finding where startPos would fit in the sorted array
left = bisect.bisect_left(fruits, [startPos, 0])
```

**Why it's problematic:** While this seems like an optimization, it adds complexity without improving the overall O(n) time complexity. It can also lead to errors in handling the window initialization.

**Solution:** Start with left_index = 0 and let the sliding window naturally find the optimal range. The algorithm already runs in O(n) time, and the simple approach is more maintainable.

### 5. **Incorrect Window Shrinking Logic**

**Pitfall:** Forgetting to remove fruits from the sum before incrementing the left pointer:
```python
while ...:
    left_index += 1  # Wrong order!
    current_sum -= fruits[left_index][1]  # This now references the wrong fruit
```

**Why it's wrong:** This accesses the wrong fruit amount (the next one instead of the current one being removed), leading to incorrect sum calculations.

**Solution:** Always update the sum before moving the pointer:
```python
while ...:
    current_sum -= fruits[left_index][1]  # Remove current fruit first
    left_index += 1  # Then move pointer
```

