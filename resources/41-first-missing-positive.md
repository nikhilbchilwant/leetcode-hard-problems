# 41. First Missing Positive

> Source: https://algo.monster/liteproblems/41

## Problem Statement

You are given an unsorted integer array `nums`. Your task is to find the smallest positive integer that is not present in the array.

The key requirements are:
- Find the smallest missing positive integer (starting from 1, 2, 3, ...)
- Your algorithm must run in `O(n)` time complexity
- You can only use `O(1)` auxiliary space (constant extra space)

For example:
- If `nums = [1, 2, 0]`, the answer is `3` because 1 and 2 are present, but 3 is missing
- If `nums = [3, 4, -1, 1]`, the answer is `2` because 1 is present, but 2 is missing
- If `nums = [7, 8, 9, 11, 12]`, the answer is `1` because no positive integers starting from 1 are present

The solution uses an in-place swapping technique. Since the answer must be in the range `[1, n+1]` where `n` is the length of the array, we can rearrange the array so that each positive integer `x` within range `[1, n]` is placed at index `x-1`. After this rearrangement, we scan through the array to find the first position where the value doesn't match its expected position plus one. This gives us the smallest missing positive integer.

The algorithm works in two phases:
1. **Rearrangement phase**: For each number in the valid range `[1, n]`, swap it to its correct position repeatedly until the current position holds either an out-of-range number or the correct number
2. **Scanning phase**: Check each position to find where `nums[i] != i+1`, which indicates the first missing positive integer

## Intuition

The key insight comes from recognizing that the smallest missing positive integer must be bounded. If we have an array of length `n`, the smallest missing positive can only be in the range `[1, n+1]`. Why? Because even if the array contains all the smallest positive integers perfectly (like `[1, 2, 3, ..., n]`), the answer would be `n+1`. In any other case, there's a gap somewhere in `[1, n]`, and that gap is our answer.

This bounded range gives us a powerful idea: we can use the array indices themselves as a hash table! Since array indices go from `0` to `n-1`, we can map positive integers `1` to `n` to indices `0` to `n-1` respectively. Each positive integer `x` should ideally be at position `x-1`.

But how can we use the array as its own hash table without using extra space? The trick is to rearrange the elements in-place. We can swap each valid number to its "home" position. For example, if we encounter the number `3`, it should go to index `2` (position `3-1`). 

The beauty of this approach is that after rearrangement:
- Numbers outside the range `[1, n]` stay wherever they end up (we don't care about them)
- Numbers within `[1, n]` are placed at their corresponding positions if possible
- The first position where `nums[i] != i+1` tells us that `i+1` is missing

This works because we're essentially creating a presence indicator using the array itself. If position `i` contains `i+1`, then `i+1` is present in the original array. If not, then `i+1` is missing. The first such missing number is our answer.

The swapping process ensures that each element gets moved at most once to its final position, maintaining `O(n)` time complexity, while using the input array itself as storage maintains `O(1)` space complexity.

## Solution Approach

The implementation follows the in-place swap strategy to rearrange the array such that each number `x` in range `[1, n]` is placed at position `x-1`.

**Phase 1: In-place Rearrangement**

We iterate through each position `i` in the array. For the current `nums[i]`, we check if:
1. It's within the valid range: `1 <= nums[i] <= n`
2. It's not already at its correct position: `nums[i] != nums[nums[i] - 1]`

If both conditions are met, we swap `nums[i]` with the element at its target position `nums[i] - 1`. The key detail is using a `while` loop instead of `if` because after swapping, the new element at position `i` might also need to be moved to its correct position.

```python
for i in range(n):
    while 1 <= nums[i] <= n and nums[i] != nums[nums[i] - 1]:
        j = nums[i] - 1
        nums[i], nums[j] = nums[j], nums[i]
```

The condition `nums[i] != nums[nums[i] - 1]` is crucial - it prevents infinite loops when a duplicate exists. If `nums[i]` equals the value at its target position, we know there's a duplicate and we should stop swapping.

**Phase 2: Finding the First Missing Positive**

After rearrangement, we scan the array linearly. At each position `i`, we check if `nums[i] == i + 1`. The first position where this condition fails indicates that `i + 1` is the smallest missing positive integer.

```python
for i in range(n):
    if nums[i] != i + 1:
        return i + 1
```

If all positions satisfy `nums[i] == i + 1`, it means the array contains all integers from `1` to `n`, so the smallest missing positive is `n + 1`.

```python
return n + 1
```

**Time and Space Complexity:**
- Time: `O(n)` - Although we have nested loops, each element is moved at most once to its final position
- Space: `O(1)` - We only use a constant amount of extra variables for swapping

## Example Walkthrough

Let's walk through the algorithm with `nums = [3, 4, -1, 1]`:

**Initial State:**
- Array: `[3, 4, -1, 1]`
- Length n = 4, so valid range is [1, 4]

**Phase 1: In-place Rearrangement**

*Position i = 0:*
- Current value: `nums[0] = 3`
- Is 3 in range [1, 4]? Yes
- Target position for 3: index 2 (3-1)
- Is `nums[0] != nums[2]`? Compare 3 != -1? Yes
- Swap: `[3, 4, -1, 1]` → `[-1, 4, 3, 1]`
- Check new `nums[0] = -1`: Not in range [1, 4], stop while loop

*Position i = 1:*
- Current value: `nums[1] = 4`
- Is 4 in range [1, 4]? Yes
- Target position for 4: index 3 (4-1)
- Is `nums[1] != nums[3]`? Compare 4 != 1? Yes
- Swap: `[-1, 4, 3, 1]` → `[-1, 1, 3, 4]`
- Check new `nums[1] = 1`: In range [1, 4]
- Target position for 1: index 0 (1-1)
- Is `nums[1] != nums[0]`? Compare 1 != -1? Yes
- Swap: `[-1, 1, 3, 4]` → `[1, -1, 3, 4]`
- Check new `nums[1] = -1`: Not in range [1, 4], stop while loop

*Position i = 2:*
- Current value: `nums[2] = 3`
- Is 3 in range [1, 4]? Yes
- Target position for 3: index 2 (3-1)
- Is `nums[2] != nums[2]`? Compare 3 != 3? No
- Already at correct position, no swap needed

*Position i = 3:*
- Current value: `nums[3] = 4`
- Is 4 in range [1, 4]? Yes
- Target position for 4: index 3 (4-1)
- Is `nums[3] != nums[3]`? Compare 4 != 4? No
- Already at correct position, no swap needed

**After Rearrangement:** `[1, -1, 3, 4]`

**Phase 2: Finding the First Missing Positive**

Check each position:
- Position 0: `nums[0] = 1`, should be 1 ✓
- Position 1: `nums[1] = -1`, should be 2 ✗

The first mismatch is at position 1, where we expected 2 but found -1.

**Answer: 2**

The smallest missing positive integer is 2.

## Python Solution

```python
class Solution:
    def firstMissingPositive(self, nums: List[int]) -> int:
        """
        Find the first missing positive integer in an unsorted array.
        Uses cyclic sort to place each positive integer at its correct position.
        Time: O(n), Space: O(1)
        """
        n = len(nums)
      
        # Phase 1: Place each positive integer at its correct position
        # For a positive integer k (where 1 <= k <= n), place it at index k-1
        for i in range(n):
            # Keep swapping current element to its correct position
            # until current position has correct element or invalid element
            while (1 <= nums[i] <= n and 
                   nums[i] != nums[nums[i] - 1]):
                # Calculate target index for current element
                target_index = nums[i] - 1
                # Swap current element with element at target position
                nums[i], nums[target_index] = nums[target_index], nums[i]
      
        # Phase 2: Find the first position where element doesn't match expected value
        # Position i should contain value i+1
        for i in range(n):
            if nums[i] != i + 1:
                return i + 1
      
        # All positions [0, n-1] contain correct values [1, n]
        # So the first missing positive is n+1
        return n + 1
```

## Time and Space Complexity

**Time Complexity:** `O(n)`, where `n` is the length of the array.

The algorithm consists of two main parts:
1. The first loop with a nested while loop that rearranges elements to their correct positions. Although there's a while loop inside the for loop, each element is moved at most once to its correct position. Once an element is placed correctly (where `nums[i] == i + 1`), it won't be moved again. Therefore, the total number of swaps across all iterations is at most `n`, making this part `O(n)`.

2. The second loop simply iterates through the array once to find the first position where `nums[i] != i + 1`, which takes `O(n)` time.

Overall time complexity: `O(n) + O(n) = O(n)`.

**Space Complexity:** `O(1)`.

The algorithm only uses a constant amount of extra space:
- Variables `i`, `j`, and `n` for iteration and indexing
- The swapping operation is done in-place without requiring additional arrays or data structures

The input array is modified in-place, so no additional space proportional to the input size is used.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Swap Implementation Leading to Infinite Loops**

One of the most common mistakes is incorrectly implementing the swap logic, which can cause infinite loops when duplicates are present.

**Incorrect Implementation:**
```python
while 1 <= nums[i] <= n:
    j = nums[i] - 1
    nums[i], nums[j] = nums[j], nums[i]
```

**Why it fails:** Without checking `nums[i] != nums[nums[i] - 1]`, the code will infinitely swap when encountering duplicates. For example, with `nums = [1, 1]`, position 0 will keep trying to swap 1 to position 0 (where it already is).

**Correct Implementation:**
```python
while 1 <= nums[i] <= n and nums[i] != nums[nums[i] - 1]:
    j = nums[i] - 1
    nums[i], nums[j] = nums[j], nums[i]
```

### 2. **Using Wrong Variable in Swap Operation**

Another frequent error is accidentally using the loop index `i` instead of the calculated target index when swapping.

**Incorrect Implementation:**
```python
while 1 <= nums[i] <= n and nums[i] != nums[nums[i] - 1]:
    nums[i], nums[i - 1] = nums[i - 1], nums[i]  # Wrong! Uses i-1 instead of nums[i]-1
```

**Why it fails:** This swaps with the wrong position. We need to swap with position `nums[i] - 1`, not `i - 1`.

**Correct Implementation:**
```python
while 1 <= nums[i] <= n and nums[i] != nums[nums[i] - 1]:
    target = nums[i] - 1  # Store target index first
    nums[i], nums[target] = nums[target], nums[i]
```

### 3. **Order of Operations in Swap Statement**

The swap operation must be carefully written to avoid overwriting values before they're used.

**Problematic Implementation:**
```python
while 1 <= nums[i] <= n and nums[i] != nums[nums[i] - 1]:
    nums[i], nums[nums[i] - 1] = nums[nums[i] - 1], nums[i]  # Wrong target ordering
```

**Why it fails:** Python evaluates the whole right side first, but then assigns the targets from left to right. `nums[i]` is assigned first, so when the second target `nums[nums[i] - 1]` is evaluated, it uses the *new* `nums[i]` and writes to the wrong index. For `nums = [2, 1]`, this sets `nums[0] = 1` and then writes `2` into `nums[1 - 1] = nums[0]`, leaving `[2, 1]` unchanged, so the `while` loop never ends. (Writing the targets in the opposite order happens to work, which makes this bug easy to introduce by accident.)

**Safe Implementation:**
```python
while 1 <= nums[i] <= n and nums[i] != nums[nums[i] - 1]:
    j = nums[i] - 1  # Calculate index first
    nums[i], nums[j] = nums[j], nums[i]  # Then swap
```

### 4. **Using `if` Instead of `while` for Swapping**

**Incorrect Implementation:**
```python
for i in range(n):
    if 1 <= nums[i] <= n and nums[i] != nums[nums[i] - 1]:
        j = nums[i] - 1
        nums[i], nums[j] = nums[j], nums[i]
```

**Why it fails:** After swapping, the new element at position `i` might also need to be moved to its correct position. Using `if` only performs one swap and moves on, leaving some elements in wrong positions.

**Example where it fails:** `nums = [2, 3, 4, 1]` (the answer is `5`)
- At i=0: Swap 2 to index 1 → `[3, 2, 4, 1]`
- With `if`: Move on to i=1, leaving 3 stranded at index 0. i=2 swaps 4 to index 3 → `[3, 2, 1, 4]`, and the scan returns `1` even though 1 is present
- With `while`: At i=0 keep swapping: 3 to index 2 → `[4, 2, 3, 1]`, then 4 to index 3 → `[1, 2, 3, 4]`, and the scan returns `5`

**Correct Implementation:**
```python
for i in range(n):
    while 1 <= nums[i] <= n and nums[i] != nums[nums[i] - 1]:
        j = nums[i] - 1
        nums[i], nums[j] = nums[j], nums[i]
```

