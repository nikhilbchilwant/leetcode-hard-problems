# 798. Smallest Rotation with Highest Score

> Source: https://algo.monster/liteproblems/798

## Problem Statement

You have an array `nums` that can be rotated by a non-negative integer `k`. When you rotate by `k`, the array transforms from its original form to `[nums[k], nums[k + 1], ... nums[nums.length - 1], nums[0], nums[1], ..., nums[k-1]]`. In other words, you take the first `k` elements and move them to the end of the array.

After rotation, you calculate a score based on how many elements satisfy a specific condition: an element at index `i` earns one point if its value is less than or equal to `i`.

For example, given `nums = [2,4,1,3,0]`:
- If you rotate by `k = 2`, the array becomes `[1,3,0,2,4]`
- Now checking each position:
  - Index 0: value 1 > 0 → no point
  - Index 1: value 3 > 1 → no point  
  - Index 2: value 0 ≤ 2 → one point
  - Index 3: value 2 ≤ 3 → one point
  - Index 4: value 4 ≤ 4 → one point
- Total score = 3 points

Your task is to find the rotation value `k` that produces the maximum possible score. If multiple values of `k` give the same maximum score, return the smallest `k`.

## Intuition

The key insight is to think about when each element contributes a point across different rotations, rather than calculating the entire score for each possible rotation.

For an element `nums[i]` with value `v`, we need to determine: for which rotation values `k` will this element contribute a point? After rotation by `k`, the element originally at index `i` moves to position `(i - k + n) % n`. For this element to score a point, we need `v ≤ (i - k + n) % n`.

Instead of checking all rotations directly (which would be O(n²)), we can be smarter. For each element, we can calculate the range of `k` values where it contributes a point. When an element starts contributing and when it stops contributing forms an interval.

The critical observation is that for each element at original position `i` with value `v`:
- It starts contributing a point when rotated to a position where its new index is at least `v`
- The element begins contributing at rotation `k = (i + 1) % n` 
- It stops contributing when the new position becomes less than `v`, which happens at `k = (n + i + 1 - v) % n`

By using a difference array technique, we can efficiently track how the score changes as we increase `k`. We mark the start of each contribution interval with `+1` and the end with `-1`. Then, by scanning through all possible `k` values and maintaining a running sum, we can find the maximum score and the smallest `k` that achieves it.

This transforms the problem from calculating full scores for each rotation to simply tracking score changes, reducing complexity from O(n²) to O(n).

**Learn more about [Prefix Sum](/problems/subarray_sum) patterns.**

## Solution Approach

The solution uses a difference array to efficiently track score changes across all possible rotations. The snippets below use short names; in the full solution, `d` is `diff`, `l` and `r` are `left` and `right`, `s` is `current_score`, `mx` is `max_score`, and `ans` is `best_rotation`.

**Step 1: Initialize the difference array**
```python
d = [0] * n
```
This array tracks the change in score when moving from rotation `k-1` to rotation `k`.

**Step 2: Calculate contribution intervals for each element**
For each element at index `i` with value `v`:
```python
l, r = (i + 1) % n, (n + i + 1 - v) % n
d[l] += 1
d[r] -= 1
```

- `l = (i + 1) % n`: The rotation where element `nums[i]` starts contributing a point
- `r = (n + i + 1 - v) % n`: The rotation where element `nums[i]` stops contributing
- We increment `d[l]` to mark the start of contribution
- We decrement `d[r]` to mark the end of contribution

**Step 3: Find the rotation with maximum score**
```python
s = 0
mx, ans = -1, n
for k, t in enumerate(d):
    s += t
    if s > mx:
        mx = s
        ans = k
```

We iterate through all possible rotations `k` from 0 to n-1:
- `s += t`: Accumulate the score change to get the actual score at rotation `k`
- Track the maximum score `mx` and its corresponding rotation `ans`
- Since we iterate from small to large `k`, we automatically get the smallest `k` when there are ties

**Time Complexity:** O(n) - We iterate through the array twice (once to build the difference array, once to find the maximum)

**Space Complexity:** O(n) - For the difference array

The elegance of this approach lies in converting a seemingly O(n²) problem (checking n rotations with n elements each) into an O(n) solution by focusing on when each element's contribution changes rather than recalculating everything from scratch.

## Example Walkthrough

Let's walk through the solution with `nums = [1, 0, 3, 2]`.

**Step 1: Initialize difference array**
```
d = [0, 0, 0, 0]
```

**Step 2: Process each element to find contribution intervals**

For **element at index 0, value 1**:
- Start contributing: `l = (0 + 1) % 4 = 1`
- Stop contributing: `r = (4 + 0 + 1 - 1) % 4 = 0`
- Update: `d[1] += 1`, `d[0] -= 1`
- Result: `d = [-1, 1, 0, 0]`

For **element at index 1, value 0**:
- Start contributing: `l = (1 + 1) % 4 = 2`
- Stop contributing: `r = (4 + 1 + 1 - 0) % 4 = 2`
- Update: `d[2] += 1`, `d[2] -= 1`
- Result: `d = [-1, 1, 0, 0]` (no net change at index 2)

For **element at index 2, value 3**:
- Start contributing: `l = (2 + 1) % 4 = 3`
- Stop contributing: `r = (4 + 2 + 1 - 3) % 4 = 0`
- Update: `d[3] += 1`, `d[0] -= 1`
- Result: `d = [-2, 1, 0, 1]`

For **element at index 3, value 2**:
- Start contributing: `l = (3 + 1) % 4 = 0`
- Stop contributing: `r = (4 + 3 + 1 - 2) % 4 = 2`
- Update: `d[0] += 1`, `d[2] -= 1`
- Result: `d = [-1, 1, -1, 1]`

**Step 3: Find maximum score by scanning difference array**

Starting with `s = 0`, `mx = -1`, `ans = 4`:

- k=0: `s = 0 + (-1) = -1`. Since `-1 ≤ -1`, no update
- k=1: `s = -1 + 1 = 0`. Since `0 > -1`, update `mx = 0`, `ans = 1`
- k=2: `s = 0 + (-1) = -1`. Since `-1 ≤ 0`, no update
- k=3: `s = -1 + 1 = 0`. Since `0 ≤ 0`, no update

**Result:** The highest running sum is 0, first reached at k = 1, so the function returns 1.

**Why the running sums are not the actual scores:** An interval `[left, right)` that wraps around the end of the array (where `right <= left`) is marked only with `+1` at `left` and `-1` at `right`. The prefix sum then misses the part of the interval that lies in `[0, right)`, which lowers every rotation's sum by exactly 1 for that element. Elements 0 and 2 have wrapping intervals, and element 1 (value 0, which scores at every rotation) has `left == right`, so its marks cancel. Each of these three elements lowers every running sum by the same constant 1, so the running sum at each `k` equals the actual score minus 3. A constant shift does not change which `k` has the largest value, so the answer is still correct.

**Verification against actual scores:**
- k=0: `[1, 0, 3, 2]` → indices 1 and 3 score → 2 points (running sum -1, plus 3)
- k=1: `[0, 3, 2, 1]` → indices 0, 2, 3 score → 3 points (running sum 0, plus 3)
- k=2: `[3, 2, 1, 0]` → indices 2 and 3 score → 2 points (running sum -1, plus 3)
- k=3: `[2, 1, 0, 3]` → indices 1, 2, 3 score → 3 points (running sum 0, plus 3)

The maximum score is 3, reached first at k = 1, which matches the returned answer.

## Python Solution

```python
class Solution:
    def bestRotation(self, nums: List[int]) -> int:
        """
        Find the rotation index K that maximizes the score.
        Score is calculated as the count of elements where nums[i] <= i after rotation.
      
        Args:
            nums: List of integers to rotate
          
        Returns:
            The rotation index K that gives the maximum score
        """
        n = len(nums)
      
        # Initialize variables to track the best rotation
        max_score = -1
        best_rotation = n
      
        # Use difference array to efficiently track score changes
        # diff[k] represents the change in score when rotating by k positions
        diff = [0] * n
      
        # For each element, calculate the range of rotations where it contributes to the score
        for index, value in enumerate(nums):
            # Calculate the rotation range where nums[index] contributes 1 point
            # After k rotations, element at index i moves to position (i - k + n) % n
            # We need: value <= (index - k + n) % n, which means k is in range [left, right)
          
            # Start of valid rotation range (inclusive)
            left = (index + 1) % n
          
            # End of valid rotation range (exclusive)
            # Derived from: value <= (index - k + n) % n
            right = (n + index + 1 - value) % n
          
            # Mark the range in the difference array
            diff[left] += 1
            diff[right] -= 1
      
        # Use prefix sum to calculate actual scores for each rotation
        current_score = 0
        for rotation_k, delta in enumerate(diff):
            current_score += delta
          
            # Update the best rotation if current score is better
            if current_score > max_score:
                max_score = current_score
                best_rotation = rotation_k
      
        return best_rotation
```

## Time and Space Complexity

**Time Complexity:** `O(n)`

The algorithm consists of two main parts:
1. First loop iterates through all elements in `nums` to calculate difference array values - `O(n)` operations
2. Second loop iterates through the difference array `diff` to find the maximum prefix sum - `O(n)` operations

Each operation inside both loops (including modulo calculations and array updates) takes `O(1)` time. Therefore, the overall time complexity is `O(n) + O(n) = O(n)`.

**Space Complexity:** `O(n)`

The algorithm uses additional space for:
- Difference array `diff` of size `n` - `O(n)` space
- A few scalar variables (`max_score`, `best_rotation`, `current_score`, `left`, `right`) - `O(1)` space

The total auxiliary space used is `O(n) + O(1) = O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### Pitfall 1: Splitting a Wrapping Range Into Two Ranges by Hand

**The Problem:**
When `right <= left`, the range of good rotations wraps around: it is `[left, n)` together with `[0, right)`. It is tempting to treat this as a bug and add special handling. The extra marking is not needed for correctness, because the missing `[0, right)` part only lowers every running sum by the same constant 1, and a constant shift does not change which `k` has the maximum.

**Equivalent Implementation (extra work, same answer):**
```python
if left <= right:
    diff[left] += 1
    diff[right] -= 1
else:
    # Wrapping range: [left, n) plus [0, right)
    diff[left] += 1
    diff[0] += 1
    diff[right] -= 1
```

This version makes the running sums equal the true scores, which helps when the score itself is needed, but it returns the same `k`.

**Solution Used:**
```python
# Mark only the start and end; wrapping ranges shift all sums by a constant
diff[left] += 1
diff[right] -= 1
```

The real requirement that follows from this is in Pitfall 4: since the running sums can be shifted down, they are relative values and can be zero or negative.

### Pitfall 2: Off-by-One Error in Range Calculation

**The Problem:**
The formula for calculating the right boundary `(n + i + 1 - v) % n` is derived from the inequality `v <= (i - k + n) % n`. A common mistake is forgetting the `+1` or using incorrect boundary logic.

**Incorrect Implementation:**
```python
# Wrong: Missing the +1
right = (n + i - v) % n  # Ends the range one rotation too early

# Or wrong: Using inclusive end instead of exclusive
right = (n + i - v) % n
diff[right + 1] -= 1  # This could cause index out of bounds
```

**Correct Solution:**
```python
# Correct: The range is [left, right) where right is exclusive
left = (i + 1) % n
right = (n + i + 1 - v) % n  # Note the +1 here
diff[left] += 1
diff[right] -= 1  # Mark the end of the range (exclusive)
```

### Pitfall 3: Forgetting to Handle the Case When `v > n`

**The Problem:**
The problem guarantees `0 <= nums[i] < n`, so the solution does not need this check. In a variant where values can exceed the largest index (`v > n - 1`), such an element can never score at any rotation. For `v = n` the formula gives `right == left` and the marks cancel, but for `v > n` it marks a range of rotations that does not exist, so the element must be skipped.

**Incorrect Implementation:**
```python
# Wrong only if values larger than n can appear
for i, v in enumerate(nums):
    left = (i + 1) % n
    right = (n + i + 1 - v) % n  # Could give unexpected results if v > n
    diff[left] += 1
    diff[right] -= 1
```

**Correct Solution:**
```python
for i, v in enumerate(nums):
    # Skip elements that can never contribute
    if v >= n:
        continue
  
    left = (i + 1) % n
    right = (n + i + 1 - v) % n
    diff[left] += 1
    diff[right] -= 1
```

### Pitfall 4: Initializing Best Score Incorrectly

**The Problem:**
The running sums are shifted down by a constant (see Pitfall 1), so they can be zero or negative even when the real scores are positive. Starting with `max_score = 0` and `best_rotation = 0` then skips a rotation whose running sum is 0 and returns the wrong index. For `nums = [2, 5, 3, 4, 5, 2]`, the running sums are `[-2, -1, 0, 0, 0, 0]` (the real scores are `[1, 2, 3, 3, 3, 3]`), so this version never updates and returns 0 instead of 2.

**Incorrect Implementation:**
```python
max_score = 0  # Wrong: running sums can be 0 or negative
best_rotation = 0

for k, delta in enumerate(diff):
    current_score += delta
    if current_score > max_score:  # This might never trigger
        max_score = current_score
        best_rotation = k
```

**Correct Solution:**
```python
max_score = -1  # Start with -1 to ensure first valid score is captured
best_rotation = n  # Or 0, but should be set on first iteration

for k, delta in enumerate(diff):
    current_score += delta
    if current_score > max_score:
        max_score = current_score
        best_rotation = k
```

