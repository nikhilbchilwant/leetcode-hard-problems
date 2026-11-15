# 3445. Maximum Difference Between Even and Odd Frequency II

> Source: https://algo.monster/liteproblems/3445

## Problem Statement

You are given a string `s` consisting of characters from '0' to '4', and an integer `k`. Your task is to find a substring of `s` that maximizes a specific frequency difference.

The substring must satisfy these conditions:
- It has a length of at least `k` characters
- It contains a character `a` that appears an **odd** number of times
- It contains a character `b` that appears an **even** number of times (but not zero times)

Your goal is to find the **maximum** value of `freq[a] - freq[b]`, where `freq[a]` is the number of times character `a` appears in the substring, and `freq[b]` is the number of times character `b` appears in the substring.

Note that:
- Characters `a` and `b` must be different
- The substring can contain other characters besides `a` and `b`
- Character `b` must appear at least once (non-zero even frequency means 2, 4, 6, etc.)

For example, if you have a substring where character '1' appears 5 times (odd) and character '2' appears 2 times (even), the frequency difference would be 5 - 2 = 3.

The function should return the maximum such difference across all valid substrings of `s`.

## Intuition

Since the string only contains 5 distinct characters ('0' to '4'), we can enumerate all possible pairs `(a, b)` where `a` is the character we want to have odd frequency and `b` is the character we want to have even frequency. This gives us at most `5 × 4 = 20` combinations to check.

For each character pair `(a, b)`, we need to find the substring that maximizes `freq[a] - freq[b]` while satisfying the parity constraints. The key insight is that we can use a [sliding window](/problems/sliding_window_maximum) approach combined with prefix state tracking.

The challenge is handling the parity constraints efficiently. We observe that:
- To maximize `freq[a] - freq[b]`, we want `freq[a]` as large as possible and `freq[b]` as small as possible
- But we need `freq[a]` to be odd and `freq[b]` to be non-zero even

Here's where the clever part comes in: instead of checking all possible substrings, we can track the parity states of cumulative counts. For any substring `[i, j]`, the frequency of a character in that substring equals its cumulative count at position `j` minus its cumulative count at position `i-1`.

The parity of this difference depends on the parities of both cumulative counts. If we want the frequency in the substring to be odd, we need the cumulative counts at the two endpoints to have different parities. If we want it to be even, we need them to have the same parity.

By maintaining a 2D array `t[2][2]` that stores the minimum value of `(cumulative_a - cumulative_b)` for each parity combination of cumulative counts seen so far, we can efficiently find the best starting position for our substring. When we're at position `r` with current cumulative counts, we look for a previous position with the right parity combination that minimizes the prefix contribution, thus maximizing the substring's frequency difference.

The [sliding window](/problems/sliding_window_maximum) ensures we maintain the minimum length constraint `k`, while the parity tracking ensures we satisfy the odd/even frequency requirements.

**Learn more about [Prefix Sum](/problems/subarray_sum) and [Sliding Window](/problems/sliding_window_maximum) patterns.**

## Solution Approach

The implementation uses a [sliding window](/problems/sliding_window_maximum) with prefix state compression to efficiently find the optimal substring for each character pair.

**Main Loop Structure:**
We enumerate all possible character pairs `(a, b)` where `a ≠ b`. For each pair, we process the string using a [sliding window](/problems/sliding_window_maximum) approach.

**Key Variables:**
- `curA`, `curB`: Count of characters `a` and `b` in the current window `[l+1, r]`
- `preA`, `preB`: Cumulative count of characters `a` and `b` before position `l` (i.e., in `[0, l]`)
- `l`: Position before the left boundary of the window (initialized to -1)
- `r`: Right boundary of the window
- `t[2][2]`: 2D array storing minimum values of `preA - preB` for each parity combination

**[Sliding Window](/problems/sliding_window_maximum) Logic:**
1. We iterate through the string with `r` as the right pointer
2. For each position `r`, we update `curA` and `curB` based on whether the current character equals `a` or `b`
3. We shrink the window from the left when two conditions are met:
   - Window size is at least `k`: `r - l >= k`
   - Character `b` appears at least twice in the window: `curB - preB >= 2` (ensuring non-zero even frequency)

**Window Shrinking:**
When shrinking, we:
1. Record the state at the left boundary: `t[preA & 1][preB & 1] = min(t[preA & 1][preB & 1], preA - preB)`
2. Move `l` forward and update `preA` and `preB` accordingly

**Answer Calculation:**
For each position `r`, we calculate the potential answer using:
```
ans = max(ans, curA - curB - t[(curA & 1) ^ 1][curB & 1])
```

This formula works because:
- `curA - curB` is the total frequency difference from position 0 to `r`
- We subtract `t[(curA & 1) ^ 1][curB & 1]` to get the frequency difference in the substring
- `(curA & 1) ^ 1` ensures we find a prefix with opposite parity for `a` (making the substring count odd)
- `curB & 1` ensures we find a prefix with the same parity for `b` (making the substring count even)

**Why This Works:**
The substring `[l+1, r]` has:
- Frequency of `a` = `curA - preA`
- Frequency of `b` = `curB - preB`
- Frequency difference = `(curA - preA) - (curB - preB) = (curA - curB) - (preA - preB)`

By storing the minimum `preA - preB` for each parity state, we maximize the frequency difference while ensuring the parity constraints are satisfied.

The algorithm processes each character pair in `O(n)` time, giving an overall time complexity of `O(5 × 4 × n) = O(n)`.

## Example Walkthrough

Let's walk through a concrete example with `s = "02211"` and `k = 2`.

We'll focus on one character pair: `a = '1'` (want odd frequency) and `b = '2'` (want even frequency).

**Initial Setup:**
- `l = -1` (position before the window starts)
- `preA = 0, preB = 0` (no characters before position -1)
- `curA = 0, curB = 0` (cumulative counts)
- `t[2][2] = [[INF, INF], [INF, INF]]` (minimum values for each parity state)

**Step-by-step processing:**

**Position r = 0 (char = '0'):**
- Neither 'a' nor 'b', so `curA = 0, curB = 0`
- Window size: `0 - (-1) = 1 < k`, don't shrink
- Skip answer calculation (window too small)

**Position r = 1 (char = '2'):**
- This is 'b', so `curB = 1`
- Window size: `1 - (-1) = 2 >= k`, can potentially shrink
- Check if `curB - preB >= 2`: `1 - 0 = 1 < 2`, don't shrink yet
- Skip answer calculation (need even frequency for 'b', but have odd)

**Position r = 2 (char = '2'):**
- This is 'b', so `curB = 2`
- Window size: `2 - (-1) = 3 >= k`
- Check if `curB - preB >= 2`: `2 - 0 = 2 >= 2`, can shrink!
- **Shrink window:**
  - Record state: `t[preA & 1][preB & 1] = t[0][0] = min(INF, 0 - 0) = 0`
  - Move `l` from -1 to 0
  - Update prefix counts: `preA = 0`, `preB = 0` (char at position 0 is '0')
- **Calculate answer:**
  - Need: `curA` odd (currently 0, even) and `curB` even (currently 2, even ✓)
  - Look for prefix with opposite parity for A: `t[(0 & 1) ^ 1][2 & 1] = t[1][0]`
  - `t[1][0] = INF`, so no valid answer yet

**Position r = 3 (char = '1'):**
- This is 'a', so `curA = 1`
- Window size: `3 - 0 = 3 >= k`
- Check if `curB - preB >= 2`: `2 - 0 = 2 >= 2`, can shrink!
- **Shrink window:**
  - Record state: `t[0][0] = min(0, 0 - 0) = 0`
  - Move `l` from 0 to 1
  - Update prefix counts: `preA = 0`, `preB = 1` (char at position 1 is '2')
- **Calculate answer:**
  - Need: `curA` odd (currently 1, odd ✓) and `curB` even (currently 2, even ✓)
  - Look for prefix with opposite parity for A and same for B: `t[(1 & 1) ^ 1][2 & 1] = t[0][0] = 0`
  - Answer = `(curA - curB) - t[0][0] = (1 - 2) - 0 = -1`

**Position r = 4 (char = '1'):**
- This is 'a', so `curA = 2`
- Window size: `4 - 1 = 3 >= k`
- Check if `curB - preB >= 2`: `2 - 1 = 1 < 2`, don't shrink
- **Calculate answer:**
  - Need: `curA` odd (currently 2, even) and `curB` even (currently 2, even ✓)
  - Look for prefix with opposite parity for A: `t[(2 & 1) ^ 1][2 & 1] = t[1][0] = INF`
  - No valid answer from this position

**Result for this pair (a='1', b='2'):** Maximum difference = -1

The algorithm would continue checking all other character pairs. For instance, with `a = '2'` and `b = '1'`, we might find a substring "2211" where '2' appears 2 times (even, but we want odd) and '1' appears 2 times (even ✓), but this wouldn't be valid since '2' needs odd frequency.

The key insight is how the parity tracking ensures we only consider valid substrings: when we calculate the answer, we specifically look for a prefix state that, when "subtracted" from the current state, gives us the desired parities (odd for 'a', even for 'b').

## Python Solution

```python
class Solution:
    def maxDifference(self, S: str, k: int) -> int:
        # Convert string of digits to list of integers
        digits = list(map(int, S))
        max_diff = float('-inf')
      
        # Try all pairs of different digits (a, b) from 0-4
        for digit_a in range(5):
            for digit_b in range(5):
                # Skip if both digits are the same
                if digit_a == digit_b:
                    continue
              
                # Current counts of digit_a and digit_b in the window
                current_count_a = 0
                current_count_b = 0
              
                # Previous counts (before the window)
                prev_count_a = 0
                prev_count_b = 0
              
                # Store minimum (count_a - count_b) for each parity combination
                # min_diff[parity_a][parity_b] stores the minimum difference
                # for given parities of count_a and count_b
                min_diff = [[float('inf'), float('inf')], 
                           [float('inf'), float('inf')]]
              
                # Left pointer of the sliding window (exclusive)
                left = -1
              
                # Iterate through the string with right pointer
                for right, digit in enumerate(digits):
                    # Update current counts
                    current_count_a += (digit == digit_a)
                    current_count_b += (digit == digit_b)
                  
                    # Shrink window from left while maintaining constraints
                    # Window size must be at least k and we need at least 2 occurrences of digit_b
                    while right - left >= k and current_count_b - prev_count_b >= 2:
                        # Update minimum difference for current parity combination
                        min_diff[prev_count_a & 1][prev_count_b & 1] = min(
                            min_diff[prev_count_a & 1][prev_count_b & 1], 
                            prev_count_a - prev_count_b
                        )
                      
                        # Move left pointer and update previous counts
                        left += 1
                        prev_count_a += (digits[left] == digit_a)
                        prev_count_b += (digits[left] == digit_b)
                  
                    # Calculate maximum difference using opposite parity for count_a
                    # This ensures we're comparing segments with different parity
                    max_diff = max(
                        max_diff, 
                        current_count_a - current_count_b - min_diff[(current_count_a & 1) ^ 1][current_count_b & 1]
                    )
      
        return max_diff
```

## Time and Space Complexity

**Time Complexity:** `O(n × |Σ|²)`, where `n` is the length of string `S` and `|Σ|` is the alphabet size (5 in this problem).

The analysis breaks down as follows:
- The outer two nested loops iterate through all pairs of digits `(a, b)` where `a ≠ b`, giving us `5 × 5 - 5 = 20` iterations, which is `O(|Σ|²)`
- For each pair `(a, b)`, the inner loop iterates through the string once with index `r` from `0` to `n-1`
- The while loop inside moves the left pointer `l` forward, but across all iterations of `r`, the pointer `l` moves at most `n` times total (amortized `O(1)` per iteration)
- All operations inside the loops (comparisons, arithmetic, array access) are `O(1)`
- Therefore, the total time complexity is `O(|Σ|² × n)`

**Space Complexity:** `O(n + 1) = O(n)`

The analysis breaks down as follows:
- The list `s` stores the converted integer values of the input string, requiring `O(n)` space
- The 2D array `t` is of fixed size `2 × 2`, which is `O(1)`
- All other variables (`ans`, `curA`, `curB`, `preA`, `preB`, `l`, `r`, `x`, `a`, `b`) use constant space `O(1)`
- Therefore, the total space complexity is `O(n)`

Note: The reference answer states `O(1)` space complexity, which would be accurate if the input string could be processed character by character without creating the list `s`. However, the given code creates `s = list(map(int, S))`, which uses `O(n)` additional space.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Parity Matching for Valid Substrings**

**The Pitfall:**
The most common mistake is misunderstanding how the parity matching works when calculating the answer. Developers often incorrectly assume they need to check if the substring itself has odd count for `a` and even count for `b`, but the code actually checks the difference between cumulative counts and stored prefix counts.

**Why it happens:**
The formula `max_diff[(current_count_a & 1) ^ 1][current_count_b & 1]` is counterintuitive. It looks for a prefix with:
- **Opposite** parity for `a` (using XOR with 1)
- **Same** parity for `b`

This ensures that the substring `[left+1, right]` has:
- Odd count for `a`: `(current_count_a - prev_count_a)` is odd when parities differ
- Even count for `b`: `(current_count_b - prev_count_b)` is even when parities are the same

**Solution:**
Add explicit validation to verify the substring meets requirements:

```python
# After calculating the potential answer, validate the substring
if min_diff[(current_count_a & 1) ^ 1][current_count_b & 1] != float('inf'):
    # Calculate actual substring counts
    substring_count_a = current_count_a - stored_prev_count_a
    substring_count_b = current_count_b - stored_prev_count_b
  
    # Verify constraints
    if substring_count_a % 2 == 1 and substring_count_b % 2 == 0 and substring_count_b > 0:
        max_diff = max(max_diff, substring_count_a - substring_count_b)
```

### 2. **Forgetting to Initialize min_diff Array Properly**

**The Pitfall:**
Not handling the case where no valid prefix exists yet. The `min_diff` array starts with `float('inf')` values, but we might try to use these infinite values in calculations before any valid prefix is stored.

**Why it happens:**
The algorithm assumes that once we shrink the window enough times, we'll have valid prefixes stored. However, for small inputs or specific character distributions, we might attempt to calculate an answer before any valid prefix exists.

**Solution:**
Check if a valid prefix exists before using it:

```python
# Before calculating max_diff
if min_diff[(current_count_a & 1) ^ 1][current_count_b & 1] != float('inf'):
    max_diff = max(
        max_diff, 
        current_count_a - current_count_b - min_diff[(current_count_a & 1) ^ 1][current_count_b & 1]
    )
```

### 3. **Misunderstanding the Window Shrinking Condition**

**The Pitfall:**
The condition `current_count_b - prev_count_b >= 2` ensures that character `b` appears at least twice in the window, guaranteeing a non-zero even count. However, developers might mistakenly think this guarantees an even count, forgetting that 3, 5, 7, etc., are also possible.

**Why it happens:**
The condition only ensures a minimum count of 2, not that the count is even. The even count is enforced through the parity matching mechanism, not this condition.

**Solution:**
The window shrinking should continue as long as we can maintain at least 2 occurrences of `b`:

```python
# More robust shrinking that ensures we keep valid windows
while right - left >= k:
    window_b_count = current_count_b - prev_count_b
  
    # Only shrink if we can maintain at least 2 occurrences of b
    if window_b_count >= 2:
        # Store the state
        min_diff[prev_count_a & 1][prev_count_b & 1] = min(
            min_diff[prev_count_a & 1][prev_count_b & 1], 
            prev_count_a - prev_count_b
        )
  
    # Check if we can shrink further without violating constraints
    next_digit = digits[left + 1] if left + 1 < len(digits) else -1
    if next_digit == digit_b and window_b_count <= 2:
        break  # Can't shrink without losing valid b count
  
    left += 1
    prev_count_a += (digits[left] == digit_a)
    prev_count_b += (digits[left] == digit_b)
```

### 4. **Not Handling Edge Cases with Return Value**

**The Pitfall:**
Returning `float('-inf')` when no valid substring exists, which might not be the expected behavior.

**Solution:**
Define clear behavior for when no valid substring exists:

```python
# At the end of the function
return max_diff if max_diff != float('-inf') else -1  # or 0, depending on requirements
```

