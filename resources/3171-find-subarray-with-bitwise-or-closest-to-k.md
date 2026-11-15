# 3171. Find Subarray With Bitwise OR Closest to K

> Source: https://algo.monster/liteproblems/3171

## Problem Statement

You are given an array `nums` and an integer `k`. Your task is to find a contiguous subarray where the absolute difference between `k` and the bitwise OR of all elements in that subarray is minimized.

Specifically, for a subarray `nums[l..r]` (where `l` and `r` are the left and right indices), you need to:
1. Calculate the bitwise OR of all elements: `nums[l] OR nums[l+1] OR ... OR nums[r]`
2. Find the absolute difference: `|k - (bitwise OR result)|`
3. Find the subarray that gives the minimum possible absolute difference

The subarray must be non-empty and contiguous (elements must be consecutive in the original array).

For example, if you have `nums = [1, 2, 3]` and `k = 5`:
- Subarray `[1]` has OR = 1, difference = |5 - 1| = 4
- Subarray `[2]` has OR = 2, difference = |5 - 2| = 3
- Subarray `[3]` has OR = 3, difference = |5 - 3| = 2
- Subarray `[1, 2]` has OR = 1 OR 2 = 3, difference = |5 - 3| = 2
- Subarray `[2, 3]` has OR = 2 OR 3 = 3, difference = |5 - 3| = 2
- Subarray `[1, 2, 3]` has OR = 1 OR 2 OR 3 = 3, difference = |5 - 3| = 2

The function should return the minimum absolute difference found among all possible subarrays.

## Intuition

The key insight is understanding how the bitwise OR operation behaves: once a bit is set to 1 in any element, it remains 1 in the OR result. This means as we extend a subarray to the right, the OR value can only increase or stay the same - it never decreases.

This monotonic property suggests we can use a two-pointer sliding window approach. We fix the right endpoint and adjust the left endpoint to find optimal subarrays.

Here's the thought process:

1. **Expanding the window**: As we add elements from the right (increasing `j`), the OR value `s` grows or stays constant. We check if the current OR value gives us a better answer.

2. **Shrinking the window**: When `s > k`, we want to try reducing the OR value by removing elements from the left. But here's the challenge - removing an element doesn't simply subtract its value from the OR result.

3. **The bit counting trick**: To properly handle element removal, we need to track how many elements contribute a 1 to each bit position. We maintain a count array `cnt[h]` that tells us how many elements in our current window have bit `h` set to 1.

4. **Removing elements**: When we remove an element from the left:
   - For each bit position `h` where the element has a 1, we decrement `cnt[h]`
   - If `cnt[h]` becomes 0, it means no remaining elements in the window have that bit set, so we can turn off bit `h` in our OR result using XOR: `s ^= 1 << h`

5. **Why this works**: By maintaining bit counts, we can efficiently update the OR value when shrinking the window. We keep shrinking while `s > k` to explore subarrays that might give us values closer to `k`.

The algorithm explores all meaningful subarrays by systematically expanding and contracting the window, ensuring we find the minimum possible difference `|s - k|`.

**Learn more about [Segment Tree](/problems/segment_tree_intro) and [Binary Search](/problems/binary_search_intro) patterns.**

## Solution Approach

The solution uses a two-pointer sliding window technique combined with bit manipulation to efficiently find the minimum difference.

**Initial Setup:**
- Calculate `m = max(nums).bit_length()` to determine the maximum number of bits needed
- Create a bit count array `cnt` of size `m` to track how many elements contribute to each bit position
- Initialize variables: `s = 0` (current OR value), `i = 0` (left pointer), `ans = inf` (minimum difference)

**Main Algorithm:**

1. **Expand the window (right pointer `j`):**
   ```python
   for j, x in enumerate(nums):
       s |= x  # Add current element to OR result
       ans = min(ans, abs(s - k))  # Update minimum difference
   ```
   After adding element `x` to the OR, we update the bit counts:
   ```python
   for h in range(m):
       if x >> h & 1:  # Check if bit h is set in x
           cnt[h] += 1  # Increment count for bit h
   ```

2. **Shrink the window (left pointer `i`):**
   When `s > k`, we try to reduce the OR value by removing elements from the left:
   ```python
   while i < j and s > k:
       y = nums[i]  # Element to remove
       for h in range(m):
           if y >> h & 1:  # If bit h is set in y
               cnt[h] -= 1  # Decrement count
               if cnt[h] == 0:  # No elements have bit h set
                   s ^= 1 << h  # Turn off bit h in s
       i += 1
       ans = min(ans, abs(s - k))  # Update after shrinking
   ```

**Key Operations:**
- `s |= x`: Adds element `x` to the OR result
- `x >> h & 1`: Checks if bit `h` is set in `x`
- `s ^= 1 << h`: Toggles (turns off) bit `h` in `s` when no elements have it set
- `cnt[h]`: Tracks how many elements in the current window `[i, j]` have bit `h` set to 1

**Why it works:**
- The OR value only increases when expanding right, allowing us to use two pointers
- By tracking bit counts, we can correctly update the OR value when removing elements
- We explore all relevant subarrays by systematically expanding and contracting the window
- The algorithm ensures we check both scenarios: when `s ≤ k` (by expanding) and when `s > k` (by shrinking)

**Time Complexity:** `O(n × log M)` where `n` is the array length and `M` is the maximum value in the array
**Space Complexity:** `O(log M)` for the bit count array

## Example Walkthrough

Let's walk through the algorithm with `nums = [3, 5, 1, 6]` and `k = 4`.

**Initial Setup:**
- Maximum value is 6, which needs 3 bits (110 in binary)
- `m = 3`, so we create `cnt = [0, 0, 0]` to track bits at positions 0, 1, 2
- `s = 0` (current OR), `i = 0` (left pointer), `ans = inf`

**Step 1: j = 0, x = 3 (binary: 011)**
- Add to OR: `s = 0 | 3 = 3`
- Update answer: `ans = min(inf, |3 - 4|) = 1`
- Update bit counts for 3:
  - Bit 0: set → `cnt[0] = 1`
  - Bit 1: set → `cnt[1] = 1`
  - Bit 2: not set → `cnt[2] = 0`
- Window: [3], OR = 3, difference = 1

**Step 2: j = 1, x = 5 (binary: 101)**
- Add to OR: `s = 3 | 5 = 7` (binary: 111)
- Update answer: `ans = min(1, |7 - 4|) = 1`
- Update bit counts for 5:
  - Bit 0: set → `cnt[0] = 2`
  - Bit 1: not set → `cnt[1] = 1`
  - Bit 2: set → `cnt[2] = 1`
- Window: [3, 5], OR = 7, difference = 3
- Since `s = 7 > k = 4`, we shrink:
  - Remove nums[0] = 3 from left
  - Update bit counts for removing 3:
    - Bit 0: `cnt[0] = 2 - 1 = 1` (still has elements with bit 0)
    - Bit 1: `cnt[1] = 1 - 1 = 0` (no elements have bit 1 now!)
      - Turn off bit 1: `s = 7 ^ 2 = 5`
    - Bit 2: unchanged
  - Move left pointer: `i = 1`
  - Update answer: `ans = min(1, |5 - 4|) = 1`
- Window: [5], OR = 5, difference = 1

**Step 3: j = 2, x = 1 (binary: 001)**
- Add to OR: `s = 5 | 1 = 5` (no change, bit 0 already set)
- Update answer: `ans = min(1, |5 - 4|) = 1`
- Update bit counts for 1:
  - Bit 0: set → `cnt[0] = 2`
  - Bit 1: not set → `cnt[1] = 0`
  - Bit 2: not set → `cnt[2] = 1`
- Window: [5, 1], OR = 5, difference = 1
- Since `s = 5 > k = 4`, we try to shrink:
  - Remove nums[1] = 5 from left
  - Update bit counts for removing 5:
    - Bit 0: `cnt[0] = 2 - 1 = 1` (still has elements with bit 0)
    - Bit 2: `cnt[2] = 1 - 1 = 0` (no elements have bit 2 now!)
      - Turn off bit 2: `s = 5 ^ 4 = 1`
  - Move left pointer: `i = 2`
  - Update answer: `ans = min(1, |1 - 4|) = 1`
- Window: [1], OR = 1, difference = 3

**Step 4: j = 3, x = 6 (binary: 110)**
- Add to OR: `s = 1 | 6 = 7` (binary: 111)
- Update answer: `ans = min(1, |7 - 4|) = 1`
- Update bit counts for 6:
  - Bit 0: not set → `cnt[0] = 1`
  - Bit 1: set → `cnt[1] = 1`
  - Bit 2: set → `cnt[2] = 1`
- Window: [1, 6], OR = 7, difference = 3
- Since `s = 7 > k = 4`, we shrink:
  - Remove nums[2] = 1 from left
  - Update bit counts for removing 1:
    - Bit 0: `cnt[0] = 1 - 1 = 0` (no elements have bit 0 now!)
      - Turn off bit 0: `s = 7 ^ 1 = 6`
  - Move left pointer: `i = 3`
  - Update answer: `ans = min(1, |6 - 4|) = 1`
- Window: [6], OR = 6, difference = 2

**Final Result:** The minimum difference is 1, achieved by subarrays like [3], [5], and [3,5] after shrinking.

## Python Solution

```python
class Solution:
    def minimumDifference(self, nums: List[int], k: int) -> int:
        # Find the number of bits needed to represent the maximum number
        num_bits = max(nums).bit_length()

        # Count array to track how many numbers in current window have each bit set
        bit_count = [0] * num_bits

        # Variables for sliding window
        current_or = 0  # Current bitwise OR of the window
        left = 0  # Left pointer of the sliding window
        min_difference = float('inf')  # Initialize minimum difference to infinity

        # Iterate through array with right pointer
        for right, current_num in enumerate(nums):
            # Add current number to the OR result
            current_or |= current_num

            # Update minimum difference with current window's OR
            min_difference = min(min_difference, abs(current_or - k))

            # Update bit counts for the current number
            for bit_position in range(num_bits):
                if current_num >> bit_position & 1:
                    bit_count[bit_position] += 1

            # Shrink window from left while OR value is greater than k
            while left < right and current_or > k:
                left_num = nums[left]

                # Update bit counts when removing left number from window
                for bit_position in range(num_bits):
                    if left_num >> bit_position & 1:
                        bit_count[bit_position] -= 1
                        # If no numbers in window have this bit set, remove it from OR
                        if bit_count[bit_position] == 0:
                            current_or ^= 1 << bit_position

                left += 1

                # Update minimum difference after shrinking window
                min_difference = min(min_difference, abs(current_or - k))

        return min_difference
```

## Time and Space Complexity

**Time Complexity:** `O(n * m)` where `n` is the length of the input array `nums` and `m` is the bit length of the maximum element in `nums`.

The analysis breaks down as follows:
- The outer loop iterates through all `n` elements of the array
- For each element, we perform bit operations that iterate through `m` bits (where `m = max(nums).bit_length()`)
- The inner while loop with variable `i` can move from 0 to `n-1` throughout the entire execution, but each element is visited at most twice (once when `j` reaches it, once when `i` leaves it)
- For each movement of `i`, we again perform `m` bit operations
- Therefore, the total operations are bounded by `O(n * m)` for the outer loop and `O(n * m)` for all iterations of the inner loop combined, resulting in `O(n * m)` overall

**Space Complexity:** `O(m)` where `m` is the bit length of the maximum element in `nums`.

The space usage comes from:
- The `cnt` array of size `m` that tracks the count of set bits at each position
- A few integer variables (`s`, `i`, `ans`, `j`, `x`, `y`, `h`) which take `O(1)` space
- No recursive calls or additional data structures that scale with input size
- Therefore, the total space complexity is `O(m)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect OR Value Update When Removing Elements**

**The Pitfall:**
A common mistake is trying to "remove" an element from the OR result by using XOR or subtraction operations directly. For example:
```python
# WRONG: Trying to remove element from OR
current_or ^= nums[left]  # This doesn't work!
# or
current_or -= nums[left]  # This is completely wrong!
```

The bitwise OR operation is not reversible like addition. Once a bit is set to 1 in the OR result, you cannot simply "subtract" an element to undo its contribution. Multiple elements might contribute to the same bit being 1.

**Why it fails:**
- If `nums = [5, 3, 7]` where 5 = 101, 3 = 011, 7 = 111
- OR of [5, 3] = 101 | 011 = 111
- Trying to remove 5 by XOR: 111 ^ 101 = 010 (which is 2, not 3!)
- The correct answer should be 3 (just the OR of [3])

**The Solution:**
Track how many elements contribute to each bit position. Only turn off a bit when its count reaches zero:
```python
for bit_position in range(num_bits):
    if left_num >> bit_position & 1:
        bit_count[bit_position] -= 1
        if bit_count[bit_position] == 0:  # No elements have this bit
            current_or ^= 1 << bit_position  # Safe to turn off this bit
```

### 2. **Not Checking Single-Element Subarrays**

**The Pitfall:**
Starting the window shrinking only when `left < right` might seem logical, but forgetting to check single-element subarrays before shrinking:
```python
# WRONG: Missing single element check
for right, current_num in enumerate(nums):
    current_or |= current_num
    # Forgot to check difference here!

    while left < right and current_or > k:
        # shrink window...
```

**Why it fails:**
If `nums = [10]` and `k = 10`, the answer should be 0. But if you don't check the difference immediately after adding each element, you might miss the optimal single-element subarray.

**The Solution:**
Always update the minimum difference immediately after expanding the window:
```python
for right, current_num in enumerate(nums):
    current_or |= current_num
    min_difference = min(min_difference, abs(current_or - k))  # Check immediately!
    # ... rest of the code
```

### 3. **Incorrect Bit Manipulation Operations**

**The Pitfall:**
Using wrong bit manipulation operations or incorrect order of operations:
```python
# WRONG: Using OR instead of XOR to toggle bit
if bit_count[bit_position] == 0:
    current_or |= 1 << bit_position  # This sets the bit, doesn't toggle!

# WRONG: Incorrect bit checking
if current_num & bit_position:  # Should be (current_num >> bit_position) & 1
```

**The Solution:**
- Use XOR (`^=`) to toggle bits: `current_or ^= 1 << bit_position`
- Check bit correctly: `(current_num >> bit_position) & 1`
- Set bit: `current_or |= 1 << bit_position`
- The expression `1 << bit_position` creates a mask with only that bit set

### 4. **Not Handling Edge Cases with Window Boundaries**

**The Pitfall:**
The condition `left < right` prevents shrinking to empty windows, but you might forget to update the answer after shrinking:
```python
# WRONG: Only updating answer before shrinking
while left < right and current_or > k:
    # ... shrink window logic
    left += 1
    # Forgot to update min_difference here!
```

**The Solution:**
Update the minimum difference both when expanding AND after shrinking the window to ensure all valid subarrays are considered.

