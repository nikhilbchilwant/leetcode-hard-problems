# 801. Minimum Swaps To Make Sequences Increasing

> Source: https://algo.monster/liteproblems/801

## Problem Statement

You have two integer arrays `nums1` and `nums2` of equal length. You can perform swap operations where you exchange `nums1[i]` with `nums2[i]` at any index `i`.

For example, if `nums1 = [1,2,3,8]` and `nums2 = [5,6,7,4]`, swapping at index 3 would give you `nums1 = [1,2,3,4]` and `nums2 = [5,6,7,8]`.

Your goal is to make both arrays **strictly increasing** using the minimum number of swap operations. A strictly increasing array means each element is strictly less than the next one: `arr[0] < arr[1] < arr[2] < ... < arr[arr.length - 1]`.

The problem guarantees that it's always possible to make both arrays strictly increasing with the given input.

You need to return the minimum number of swaps required to achieve this.

## Intuition

When looking at this problem, we need to track the state of our arrays as we process each position. At any index `i`, we have two choices: either swap the elements at position `i` or keep them as they are.

The key insight is that our decision at position `i` depends on what happened at position `i-1`. If we swapped at `i-1`, this affects what we can do at position `i` to maintain the strictly increasing property.

This suggests a [dynamic programming](/problems/dynamic_programming_intro) approach where we track two states:
- `a`: minimum swaps needed if we DON'T swap at position `i`
- `b`: minimum swaps needed if we DO swap at position `i`

For each position, we need to consider the relationship between consecutive elements. There are two main scenarios:

1. **Forced swap scenario**: When `nums1[i-1] >= nums1[i]` or `nums2[i-1] >= nums2[i]`, the current arrangement violates the strictly increasing property. We must change the relative positions - if we swapped at `i-1`, we can't swap at `i`, and vice versa.

2. **Flexible scenario**: When both arrays are already increasing (`nums1[i-1] < nums1[i]` and `nums2[i-1] < nums2[i]`), we have more freedom. We can maintain the same swap pattern from `i-1`. Additionally, if cross-comparisons also work (`nums1[i-1] < nums2[i]` and `nums2[i-1] < nums1[i]`), we can change the swap pattern, giving us the opportunity to choose the minimum cost option.

By building up these minimum costs position by position, we eventually find the global minimum number of swaps needed for the entire arrays.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

We implement a [dynamic programming](/problems/dynamic_programming_intro) solution using two variables `a` and `b` to track the minimum swaps needed (in the code, `a` is `keep`, `b` is `swap`, and the saved copies `x`, `y` are `prev_keep`, `prev_swap`):
- `a`: minimum swaps to make arrays strictly increasing up to index `i` without swapping at position `i`
- `b`: minimum swaps to make arrays strictly increasing up to index `i` with swapping at position `i`

**Initialization:**
- `a = 0`: no swap needed at index 0 if we don't swap
- `b = 1`: one swap needed at index 0 if we do swap

**Processing each position `i` from 1 to n-1:**

1. Save previous states: `x = a` and `y = b` to remember the minimum swaps up to position `i-1`

2. Check if positions `i-1` and `i` violate the strictly increasing property:
   - If `nums1[i-1] >= nums1[i]` or `nums2[i-1] >= nums2[i]`:
     - The relative positions must change between `i-1` and `i`
     - If we swapped at `i-1` (cost `y`), we can't swap at `i`: `a = y`
     - If we didn't swap at `i-1` (cost `x`), we must swap at `i`: `b = x + 1`

3. Otherwise, both arrays are already increasing at these positions:
   - We can maintain the swap pattern: `b = y + 1` (if we swapped at `i-1`, we swap at `i` too)
   - Check if cross-swapping is valid: if `nums1[i-1] < nums2[i]` and `nums2[i-1] < nums1[i]`:
     - We can change the swap pattern between `i-1` and `i`
     - Take the minimum: `a = min(a, y)` (compare not swapping at `i` with different patterns)
     - Take the minimum: `b = min(b, x + 1)` (compare swapping at `i` with different patterns)

**Final Result:**
Return `min(a, b)` - the minimum between not swapping and swapping at the last position.

The algorithm runs in `O(n)` time with `O(1)` space, as we only maintain two variables throughout the iteration.

## Example Walkthrough

Let's walk through the algorithm with `nums1 = [1,3,5,4]` and `nums2 = [1,2,3,7]`.

We need to track two states:
- `a`: minimum swaps if we DON'T swap at current position
- `b`: minimum swaps if we DO swap at current position

**Initial state (i=0):**
- `a = 0` (no swap at position 0)
- `b = 1` (swap at position 0)

**Position i=1:**
- Previous values: `x = 0`, `y = 1`
- Check if arrays violate strictly increasing: `nums1[0] < nums1[1]` (1 < 3 ✓) and `nums2[0] < nums2[1]` (1 < 2 ✓)
- No violation, so we have flexibility
- `a = x = 0` (don't swap at i=1, didn't swap at i=0)
- `b = y + 1 = 2` (swap at i=1, swapped at i=0)
- Check cross-swap validity: `nums1[0] < nums2[1]` (1 < 2 ✓) and `nums2[0] < nums1[1]` (1 < 3 ✓)
- Cross-swap is valid, so we can optimize:
  - `a = min(0, 1) = 0` (compare not swapping with different patterns)
  - `b = min(2, 0+1) = 1` (swapping at i=1 without swapping at i=0 is better)

**Position i=2:**
- Previous values: `x = 0`, `y = 1`
- Check: `nums1[1] < nums1[2]` (3 < 5 ✓) and `nums2[1] < nums2[2]` (2 < 3 ✓)
- No violation, flexibility exists
- `a = x = 0` (maintain no-swap pattern)
- `b = y + 1 = 2` (maintain swap pattern)
- Check cross-swap: `nums1[1] < nums2[2]` (3 < 3 ✗) - fails!
- Cannot optimize with cross-swap, keep current values: `a = 0`, `b = 2`

**Position i=3:**
- Previous values: `x = 0`, `y = 2`
- Check: `nums1[2] >= nums1[3]` (5 >= 4 ✓) - violation detected!
- Must change relative positions:
  - `a = y = 2` (if we swapped at i=2, don't swap at i=3)
  - `b = x + 1 = 1` (if we didn't swap at i=2, must swap at i=3)

**Final Result:**
`min(a, b) = min(2, 1) = 1`

The minimum number of swaps needed is **1**. We swap at position 3, transforming:
- `nums1 = [1,3,5,7]` 
- `nums2 = [1,2,3,4]`

Both arrays are now strictly increasing.

## Python Solution

```python
class Solution:
    def minSwap(self, nums1: List[int], nums2: List[int]) -> int:
        # Dynamic Programming approach to find minimum swaps
        # keep: minimum swaps needed if we DON'T swap at current position
        # swap: minimum swaps needed if we DO swap at current position
        keep = 0  # No swap needed at position 0 to keep arrays valid
        swap = 1  # One swap needed if we swap at position 0
      
        # Iterate through arrays starting from index 1
        for i in range(1, len(nums1)):
            # Store previous state values
            prev_keep = keep
            prev_swap = swap
          
            # Check if arrays are NOT naturally increasing without any swap
            if nums1[i - 1] >= nums1[i] or nums2[i - 1] >= nums2[i]:
                # Arrays are not naturally increasing
                # Must swap at exactly one position (either previous or current)
                keep = prev_swap  # Don't swap current, so must have swapped previous
                swap = prev_keep + 1  # Swap current, so previous must not be swapped
            else:
                # Arrays are naturally increasing without swap
                # Default: continue the same state as previous
                swap = prev_swap + 1  # If we swapped previous, swap current too
              
                # Check if cross-comparison also works
                # (previous nums1 with current nums2, and previous nums2 with current nums1)
                if nums1[i - 1] < nums2[i] and nums2[i - 1] < nums1[i]:
                    # Cross-swap is also valid
                    # We can choose to swap or not swap independently
                    keep = min(keep, prev_swap)  # Can keep current regardless of previous
                    swap = min(swap, prev_keep + 1)  # Can swap current regardless of previous
      
        # Return minimum of both final states
        return min(keep, swap)
```

## Time and Space Complexity

The time complexity of this code is `O(n)`, where `n` is the length of the input arrays `nums1` and `nums2`. This is because the algorithm uses a single for loop that iterates through the arrays from index 1 to `len(nums1) - 1`, performing a constant amount of work in each iteration. Each iteration involves only comparisons and assignments of variables, which are all `O(1)` operations.

The space complexity is `O(1)` as the algorithm only uses a fixed number of variables (`keep`, `swap`, `prev_keep`, `prev_swap`) regardless of the input size. These variables store the minimum number of swaps needed for different states at each position, and they are reused throughout the iteration rather than creating new storage that scales with the input size. No additional data structures like arrays or recursion stacks are used that would grow with the input size.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrectly Updating States In-Place**

A critical mistake is updating `a` and `b` (or `keep` and `swap`) directly without preserving their previous values for subsequent calculations within the same iteration.

**Incorrect Implementation:**
```python
for i in range(1, len(nums1)):
    if nums1[i-1] >= nums1[i] or nums2[i-1] >= nums2[i]:
        keep = swap  # Wrong! 'swap' might be modified below
        swap = keep + 1  # Using already modified 'keep'!
```

**Why it's wrong:** When we update `keep = swap`, we lose the original value of `keep`. Then when we calculate `swap = keep + 1`, we're using the new value of `keep` (which is actually the old `swap`), not the original `keep` value from the previous iteration.

**Correct Solution:**
```python
for i in range(1, len(nums1)):
    prev_keep = keep
    prev_swap = swap
  
    if nums1[i-1] >= nums1[i] or nums2[i-1] >= nums2[i]:
        keep = prev_swap
        swap = prev_keep + 1  # Using preserved original value
```

### 2. **Missing the Cross-Swap Validation Check**

Another common mistake is assuming that if the arrays are naturally increasing (`nums1[i-1] < nums1[i]` and `nums2[i-1] < nums2[i]`), we can freely choose to swap or not swap at position `i` regardless of what happened at position `i-1`.

**Incorrect Implementation:**
```python
else:
    # Arrays are naturally increasing
    keep = min(prev_keep, prev_swap)  # Wrong! Not always valid
    swap = min(prev_keep + 1, prev_swap + 1)
```

**Why it's wrong:** Just because the arrays are naturally increasing doesn't mean we can mix and match swap decisions. We need to verify that cross-comparisons are valid.

**Correct Solution:**
```python
else:
    # Default: maintain same swap pattern
    swap = prev_swap + 1
  
    # Only optimize if cross-swap is valid
    if nums1[i-1] < nums2[i] and nums2[i-1] < nums1[i]:
        keep = min(keep, prev_swap)
        swap = min(swap, prev_keep + 1)
```

### 3. **Initializing with Wrong Values**

Some might initialize both states to 0 or both to 1, misunderstanding what they represent.

**Incorrect:**
```python
keep = 0
swap = 0  # Wrong! If we swap at index 0, that's 1 operation
```

**Correct:**
```python
keep = 0  # No swap at index 0 means 0 operations
swap = 1  # Swapping at index 0 means 1 operation
```

These pitfalls can lead to incorrect minimum swap counts or runtime errors, making it crucial to carefully track state transitions and validate all swap possibilities.

