# 995. Minimum Number of K Consecutive Bit Flips

> Source: https://algo.monster/liteproblems/995

## Problem Statement

You have a binary array `nums` containing only 0s and 1s, and an integer `k`.

A **k-bit flip** operation means:
- Select a contiguous subarray of exactly length `k` from `nums`
- Flip all bits in that subarray (change all 0s to 1s and all 1s to 0s)

Your goal is to transform the array so that it contains only 1s (no 0s remain).

Return the minimum number of k-bit flip operations needed to achieve this. If it's impossible to eliminate all 0s, return `-1`.

**Example walkthrough:**
- If `nums = [0,1,0]` and `k = 1`, you can flip each 0 individually (2 flips total)
- If `nums = [1,1,0]` and `k = 2`, you can flip the subarray `[1,0]` to get `[1,0,1]` (1 flip total)
- If `nums = [0,0,0,1,0,1,1,0]` and `k = 3`, you need multiple flips to eliminate all 0s

## Intuition

The key insight is that flipping order doesn't matter - flipping subarray A then B gives the same result as flipping B then A. This means we can use a greedy approach.

Starting from the leftmost position, if we encounter a 0, we **must** flip it at some point. Since flips are expensive (we want minimum flips), we should flip as soon as we encounter a 0. Why? Because delaying the flip doesn't help - we'll still need to flip it eventually, and flipping early might help us fix multiple 0s in one operation.

But here's the challenge: when we flip a subarray, it affects `k` consecutive elements. How do we efficiently track which positions have been affected by previous flips?

Consider position `i`. It might have been affected by flips that started at positions `i-k+1`, `i-k+2`, ..., up to `i`. If position `i` has been flipped an odd number of times, a original 0 becomes 1 and original 1 becomes 0. If flipped an even number of times, it returns to its original value.

To track this efficiently, we use a difference array trick. When we flip at position `i`:
- Mark the start of flip at position `i` with `+1`
- Mark the end of flip at position `i+k` with `-1`

As we scan through the array, we maintain a running sum `s` of these markers. This `s` tells us how many flips are "active" at the current position.

The clever part: if `s % 2 == nums[i]`, then the effective value at position `i` is 0:
- If `nums[i] = 0` and `s` is even (even flips), the position remains 0
- If `nums[i] = 1` and `s` is odd (odd flips), the position becomes 0

In both cases, we need to perform a flip starting at position `i`. This greedy choice is optimal because we process left-to-right and this is the earliest opportunity to fix this 0.

**Learn more about [Queue](/problems/queue_intro), [Prefix Sum](/problems/subarray_sum) and [Sliding Window](/problems/sliding_window_maximum) patterns.**

## Solution Approach

The implementation uses a **difference array** pattern to efficiently track the cumulative effect of flips at each position.

In the code, `d` is named `flip_diff`, `s` is `current_flip_count`, and `ans` is `total_flips`.

**Data Structures:**
- `d`: Difference array of size `n+1` to mark flip boundaries
- `s`: Running sum to track the number of active flips at current position
- `ans`: Counter for total number of flips performed

**Algorithm Steps:**

1. **Initialize variables:**
   - Create difference array `d` with all zeros
   - Set `ans = 0` (flip counter) and `s = 0` (active flips counter)

2. **Process each position from left to right:**
   ```python
   for i, x in enumerate(nums):
       s += d[i]  # Update active flips count
   ```
   - Add `d[i]` to running sum `s` to account for flips starting/ending at position `i`

3. **Check if current position needs flipping:**
   ```python
   if s % 2 == x:
   ```
   - If `s % 2 == x`, the effective value at position `i` is 0
   - This happens when:
     - `x = 0` and `s` is even (no net flip effect)
     - `x = 1` and `s` is odd (flipped to 0)

4. **Perform flip if needed:**
   ```python
   if i + k > n:
       return -1  # Can't flip - would go out of bounds
   d[i] += 1      # Mark flip start
   d[i + k] -= 1  # Mark flip end
   s += 1         # Update active flips
   ans += 1       # Increment flip counter
   ```
   - First check if flip is possible (won't exceed array bounds)
   - Use difference array technique: increment at start position, decrement at end position
   - Update `s` immediately since we're starting a new flip at current position
   - Increment answer counter

5. **Return result:**
   - After processing all positions, return `ans` as the minimum flips needed

**Why the difference array works:**
- When we set `d[i] += 1` and `d[i+k] -= 1`, we're marking that a flip affects positions `[i, i+k-1]`
- As we scan left to right with running sum `s`, it automatically tracks how many flips are active
- This avoids the O(k) cost of updating each position in the flip range, making the solution O(n) overall

**Time Complexity:** O(n) - single pass through the array
**Space Complexity:** O(n) - for the difference array

## Example Walkthrough

Let's trace through `nums = [0,1,0]` with `k = 2`:

**Initial state:**
- `nums = [0,1,0]`, `n = 3`, `k = 2`
- `d = [0,0,0,0]` (difference array of size n+1)
- `s = 0` (active flips), `ans = 0` (total flips)

**Step 1: Process index 0 (value = 0)**
- Update active flips: `s = s + d[0] = 0 + 0 = 0`
- Check if flip needed: `s % 2 == nums[0]` → `0 == 0` ✓ (position needs flip)
- Can we flip? `i + k = 0 + 2 = 2 ≤ 3` ✓ (within bounds)
- Perform flip:
  - `d[0] += 1` → `d = [1,0,0,0]` (mark flip start)
  - `d[2] -= 1` → `d = [1,0,-1,0]` (mark flip end)
  - `s += 1` → `s = 1` (one flip now active)
  - `ans += 1` → `ans = 1`
- This flip affects positions [0,1], changing them from [0,1] to [1,0]

**Step 2: Process index 1 (value = 1)**
- Update active flips: `s = s + d[1] = 1 + 0 = 1`
- Check if flip needed: `s % 2 == nums[1]` → `1 == 1` ✓ (position needs flip)
  - Original value was 1, but with 1 active flip it's now 0, so we need to flip
- Can we flip? `i + k = 1 + 2 = 3 ≤ 3` ✓ (within bounds)
- Perform flip:
  - `d[1] += 1` → `d = [1,1,-1,0]` (mark flip start)
  - `d[3] -= 1` → `d = [1,1,-1,-1]` (mark flip end)
  - `s += 1` → `s = 2` (two flips now active)
  - `ans += 1` → `ans = 2`
- This flip affects positions [1,2], changing them from [0,0] to [1,1]

**Step 3: Process index 2 (value = 0)**
- Update active flips: `s = s + d[2] = 2 + (-1) = 1`
  - The -1 indicates the first flip (from index 0) has ended
- Check if flip needed: `s % 2 == nums[2]` → `1 == 0` ✗ (position doesn't need flip)
  - Original value was 0, with 1 active flip it's now 1, which is what we want

**Final Result:**
- Return `ans = 2`
- The array after all flips would be [1,1,1]

**Verification of flips:**
1. First flip at index 0: [0,1,0] → [1,0,0]
2. Second flip at index 1: [1,0,0] → [1,1,1] ✓

The difference array efficiently tracked when flips started and ended, allowing us to maintain the count of active flips at each position with just O(1) updates per flip.

## Python Solution

```python
class Solution:
    def minKBitFlips(self, nums: List[int], k: int) -> int:
        n = len(nums)
        # Difference array to track flip operations
        # flip_diff[i] represents the change in flip count at position i
        flip_diff = [0] * (n + 1)
      
        # Total number of flips performed
        total_flips = 0
        # Current cumulative flip count affecting position i
        current_flip_count = 0
      
        for i, num in enumerate(nums):
            # Update current flip count with the difference at position i
            current_flip_count += flip_diff[i]
          
            # Check if current position needs flipping
            # If current_flip_count is even and num is 0, or
            # if current_flip_count is odd and num is 1, we need to flip
            if current_flip_count % 2 == num:
                # Check if we can perform a k-length flip starting at i
                if i + k > n:
                    # Cannot flip: would exceed array bounds
                    return -1
              
                # Mark the start of flip at position i
                flip_diff[i] += 1
                # Mark the end of flip at position i + k
                flip_diff[i + k] -= 1
                # Update current flip count for this new flip
                current_flip_count += 1
                # Increment total number of flips
                total_flips += 1
      
        return total_flips
```

## Time and Space Complexity

**Time Complexity:** `O(n)`

The algorithm iterates through the array `nums` exactly once using a single for loop from index 0 to n-1. Within each iteration, all operations performed are constant time:
- Accessing and updating the difference array `d[i]`: `O(1)`
- Updating the running sum `s`: `O(1)`
- Checking conditions and updating counters: `O(1)`

Since we perform `O(1)` operations for each of the `n` elements, the overall time complexity is `O(n)`.

**Space Complexity:** `O(n)`

The algorithm uses additional space for:
- The difference array `d` of size `n + 1`: `O(n + 1) = O(n)`
- A few integer variables (`ans`, `s`): `O(1)`

The dominant space usage comes from the difference array `d`, which stores the flip boundaries to track where flips start and end. Therefore, the total space complexity is `O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Misunderstanding the Flip Condition Logic**

**Pitfall:** The condition `current_flip_count % 2 == num` is counterintuitive. It compares the parity of the flips with the original value, not the effective value, so it is easy to invert it by mistake and write `current_flip_count % 2 != num`. That version flips positions that are already 1 and skips the ones that are 0. For `nums = [0, 1, 0]` and `k = 1` it returns 1 instead of 2. (The form `(current_flip_count + num) % 2 == 0` is equivalent to the correct condition and also works.)

**Correct Understanding:**
- When `current_flip_count % 2 == 0` (even flips): effective value = original value
- When `current_flip_count % 2 == 1` (odd flips): effective value = 1 - original value
- We need to flip when effective value is 0:
  - If `num = 0` and even flips → effective is 0 → need flip
  - If `num = 1` and odd flips → effective is 0 → need flip
  - Both cases satisfy: `current_flip_count % 2 == num`

**Solution:**
```python
# Add a comment or use a helper variable for clarity
effective_value_is_zero = (current_flip_count % 2 == num)
if effective_value_is_zero:
    # Perform flip
```

### 2. **Off-by-One Error in Difference Array**

**Pitfall:** Using `flip_diff[i + k] += 1` instead of `flip_diff[i + k] -= 1`, or incorrectly sizing the difference array as `[0] * n` instead of `[0] * (n + 1)`.

**Why it happens:** Confusion about how difference arrays work or forgetting that marking the end of a range requires decrementing.

**Solution:**
```python
# Always remember: difference array needs n+1 size
flip_diff = [0] * (n + 1)  # NOT [0] * n

# Marking a range [i, i+k-1]:
flip_diff[i] += 1      # Start of range (increment)
flip_diff[i + k] -= 1  # Position AFTER range ends (decrement)
```

### 3. **Forgetting to Update `current_flip_count` After Starting a New Flip**

**Pitfall:** After marking the flip in the difference array, forgetting to immediately update `current_flip_count += 1` for the current position.

**Why it happens:** The difference array update `flip_diff[i] += 1` doesn't automatically affect `current_flip_count` until the next iteration, but we need the effect immediately at position `i`.

**Incorrect Code:**
```python
if current_flip_count % 2 == num:
    if i + k > n:
        return -1
    flip_diff[i] += 1
    flip_diff[i + k] -= 1
    # MISSING: current_flip_count += 1
    total_flips += 1
```

**Solution:** Always update `current_flip_count` immediately after starting a flip:
```python
flip_diff[i] += 1
flip_diff[i + k] -= 1
current_flip_count += 1  # Don't forget this!
total_flips += 1
```

### 4. **Using Actual Flipping Instead of Virtual Tracking**

**Pitfall:** Actually modifying the array elements during flips, which leads to O(n*k) time complexity.

**Inefficient Approach:**
```python
# DON'T DO THIS - O(n*k) complexity
for i in range(n):
    if nums[i] == 0:
        if i + k > n:
            return -1
        for j in range(i, i + k):
            nums[j] = 1 - nums[j]  # Actually flipping
        total_flips += 1
```

**Solution:** Use the difference array pattern to track flips virtually without modifying the original array, maintaining O(n) complexity.

