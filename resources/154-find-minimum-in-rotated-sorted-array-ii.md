# 154. Find Minimum in Rotated Sorted Array II

> Source: https://algo.monster/liteproblems/154

## Problem Statement

You are given an array that was originally sorted in ascending order, but has been **rotated** between 1 and n times. A rotation means taking the last element and moving it to the front of the array.

For example, if you start with `[0,1,4,4,5,6,7]`:
- After 1 rotation: `[7,0,1,4,4,5,6]`
- After 4 rotations: `[4,5,6,7,0,1,4]`
- After 7 rotations: `[0,1,4,4,5,6,7]` (back to original)

The array may contain **duplicate** values. Your task is to find and return the **minimum element** in this rotated array.

The challenge is to minimize the number of operations needed to find this minimum value. A naive approach would check every element, but you need to find a more efficient solution.

The solution uses a modified binary search approach. The key insight is comparing the middle element with the rightmost element to determine which half of the array contains the minimum:
- If `nums[mid] > nums[right]`: The minimum must be in the right half (after mid)
- If `nums[mid] < nums[right]`: The minimum is in the left half (including mid)
- If `nums[mid] == nums[right]`: Due to duplicates, we can't determine which half contains the minimum, so we cautiously shrink the search space by moving `right` one position left

This approach handles the complexity introduced by duplicates while maintaining efficiency in most cases.

## Intuition

The key observation is that even though the array is rotated, it maintains a special property: it consists of two sorted portions. When we rotate a sorted array, we essentially split it into two parts - both still sorted internally, but with a "break point" where the minimum element sits.

Think of it like a mountain range that's been cut and shifted. The original sorted array is like a smooth incline from lowest to highest. After rotation, it becomes two inclines with a sudden drop between them - and that drop is where our minimum lives.

In a normal sorted array, we could use binary search because we know exactly how elements relate to each other. Here, we can still use binary search, but we need to be clever about which half to search.

When we pick a middle element, we compare it with the rightmost element. Why the right? Because:
- If `nums[mid] > nums[right]`, something's wrong with the normal sorted order - the minimum must have been rotated into the right portion
- If `nums[mid] < nums[right]`, the right portion is properly sorted, so the minimum must be in the left portion (or could be mid itself)

The tricky part comes with duplicates. When `nums[mid] == nums[right]`, we lose our ability to determine which half is "broken" by the rotation. Consider `[3,3,1,3]` - the middle and right are both 3, but the minimum 1 is between them. We can't tell just from the values.

In this case, we play it safe by shrinking our search space by just one element from the right. This ensures we don't accidentally skip over the minimum, though it means in worst case (many duplicates) we might degrade to linear time.

This approach elegantly balances efficiency with correctness, using the sorted nature of the sub-arrays to guide our search while carefully handling the edge case of duplicates.

## Solution Approach

We implement a modified binary search algorithm to find the minimum element efficiently. Here's the step-by-step breakdown:

**Initialize the search boundaries:**
```python
left, right = 0, len(nums) - 1
```
We start with pointers at both ends of the array.

**Binary search with modifications:**
```python
while left < right:
    mid = (left + right) >> 1
```
The loop continues while we have a search space. We calculate the middle index using bit shift (`>> 1`), which is equivalent to integer division by 2.

**Decision logic based on comparison:**

1. **Case 1: `nums[mid] > nums[right]`**
   ```python
   if nums[mid] > nums[right]:
       left = mid + 1
   ```
   This means the rotation point (and minimum) is in the right half. For example, in `[4,5,6,7,0,1,2]`, if mid points to 6 and right points to 2, we know the minimum must be after position mid.

2. **Case 2: `nums[mid] < nums[right]`**
   ```python
   elif nums[mid] < nums[right]:
       right = mid
   ```
   The right portion is properly sorted, so the minimum is either at mid or to its left. We keep mid in our search space by setting `right = mid` (not `mid - 1`).

3. **Case 3: `nums[mid] == nums[right]`**
   ```python
   else:
       right -= 1
   ```
   With duplicates, we can't determine which half contains the minimum. We cautiously shrink the search space by one element from the right. This ensures we don't miss the minimum even if it means slower convergence.

**Return the result:**
```python
return nums[left]
```
When the loop exits, `left` and `right` converge to the minimum element's position.

**Time Complexity:** 
- Best/Average case: `O(log n)` when there are few or no duplicates
- Worst case: `O(n)` when the array contains many duplicates (e.g., `[1,1,1,1,0,1,1]`)

**Space Complexity:** `O(1)` - only using a constant amount of extra space for pointers.

## Example Walkthrough

Let's walk through the algorithm with the array `[4,5,6,7,0,1,2]`:

**Initial state:**
- Array: `[4,5,6,7,0,1,2]`
- `left = 0`, `right = 6`

**Iteration 1:**
- `mid = (0 + 6) >> 1 = 3`
- `nums[mid] = 7`, `nums[right] = 2`
- Since `7 > 2`, the minimum must be in the right half
- Update: `left = mid + 1 = 4`
- Search space: `[0,1,2]`

**Iteration 2:**
- `left = 4`, `right = 6`
- `mid = (4 + 6) >> 1 = 5`
- `nums[mid] = 1`, `nums[right] = 2`
- Since `1 < 2`, the right portion is sorted, minimum is at mid or left of it
- Update: `right = mid = 5`
- Search space: `[0,1]`

**Iteration 3:**
- `left = 4`, `right = 5`
- `mid = (4 + 5) >> 1 = 4`
- `nums[mid] = 0`, `nums[right] = 1`
- Since `0 < 1`, the right portion is sorted, minimum is at mid or left of it
- Update: `right = mid = 4`
- Search space: `[0]`

**Final state:**
- `left = 4`, `right = 4` (loop exits since `left == right`)
- Return `nums[4] = 0`

Now let's see how duplicates are handled with `[2,2,2,0,2,2]`:

**Initial state:**
- Array: `[2,2,2,0,2,2]`
- `left = 0`, `right = 5`

**Iteration 1:**
- `mid = (0 + 5) >> 1 = 2`
- `nums[mid] = 2`, `nums[right] = 2`
- Since they're equal, we can't determine which half has the minimum
- Update: `right = right - 1 = 4`
- Search space: `[2,2,2,0,2]`

**Iteration 2:**
- `left = 0`, `right = 4`
- `mid = (0 + 4) >> 1 = 2`
- `nums[mid] = 2`, `nums[right] = 2`
- Again equal, so decrement right
- Update: `right = 3`
- Search space: `[2,2,2,0]`

**Iteration 3:**
- `left = 0`, `right = 3`
- `mid = (0 + 3) >> 1 = 1`
- `nums[mid] = 2`, `nums[right] = 0`
- Since `2 > 0`, minimum is in the right half
- Update: `left = mid + 1 = 2`
- Search space: `[2,0]`

**Iteration 4:**
- `left = 2`, `right = 3`
- `mid = (2 + 3) >> 1 = 2`
- `nums[mid] = 2`, `nums[right] = 0`
- Since `2 > 0`, minimum is in the right half
- Update: `left = mid + 1 = 3`
- Search space: `[0]`

**Final state:**
- `left = 3`, `right = 3` (loop exits)
- Return `nums[3] = 0`

The algorithm successfully navigates through duplicates by carefully shrinking the search space when ambiguity arises, ensuring the minimum is never skipped.

## Python Solution

```python
class Solution:
    def findMin(self, nums: List[int]) -> int:
        """
        Find the minimum element in a rotated sorted array that may contain duplicates.
        Uses binary search to achieve O(log n) average time complexity.
      
        Args:
            nums: A rotated sorted array that may contain duplicate elements
          
        Returns:
            The minimum element in the array
        """
        # Initialize binary search boundaries
        left = 0
        right = len(nums) - 1
      
        # Continue searching while the search space has more than one element
        while left < right:
            # Calculate middle index using bit shift (equivalent to // 2)
            mid = (left + right) >> 1
          
            # Case 1: Mid element is greater than right element
            # The minimum must be in the right half (after mid)
            if nums[mid] > nums[right]:
                left = mid + 1
            # Case 2: Mid element is less than right element  
            # The minimum is in the left half (including mid)
            elif nums[mid] < nums[right]:
                right = mid
            # Case 3: Mid element equals right element
            # Cannot determine which side contains minimum, safely shrink from right
            else:
                right -= 1
      
        # When left == right, we've found the minimum element
        return nums[left]
```

## Time and Space Complexity

**Time Complexity:** `O(n)` in the worst case, `O(log n)` in the average case

The algorithm uses binary search to find the minimum element in a rotated sorted array that may contain duplicates. 

- In the best/average case where duplicates are minimal, the search space is halved in each iteration, resulting in `O(log n)` time complexity.
- In the worst case, when the array contains many duplicates (especially when `nums[mid] == nums[right]`), we can only eliminate one element at a time by decrementing `right`. Consider an array like `[2,2,2,2,2,2,2,1,2,2]` - we might need to linearly scan through most elements, resulting in `O(n)` time complexity.

**Space Complexity:** `O(1)`

The algorithm only uses a constant amount of extra space for the variables `left`, `right`, and `mid`, regardless of the input size. No additional data structures or recursive calls are used.

## Common Pitfalls

### 1. **Incorrect Boundary Update in Case 2**
A frequent mistake is setting `right = mid - 1` instead of `right = mid` when `nums[mid] < nums[right]`.

**Why it's wrong:** When the mid element is less than the right element, the mid element itself could be the minimum. By setting `right = mid - 1`, you might exclude the actual minimum from your search space.

**Example causing failure:**
```python
nums = [3, 1, 2]
# left = 0, right = 2, mid = 1
# nums[1] = 1 < nums[2] = 2, so the minimum is at mid or to its left
# right = mid - 1 = 0 drops index 1, and the search returns nums[0] = 3 instead of 1
```

**Correct approach:**
```python
elif nums[mid] < nums[right]:
    right = mid  # Keep mid in search space
```

### 2. **Attempting to Optimize the Duplicate Case Too Aggressively**
Some might try to skip multiple elements or use `left += 1` when handling duplicates.

**Why it's wrong:** When `nums[mid] == nums[right]`, we cannot determine which side contains the minimum. Moving left pointer forward or skipping multiple elements might miss the minimum.

**Example causing failure:**
```python
nums = [0, 1, 1]  # rotated n times, so still sorted
# left = 0, right = 2, mid = 1: nums[1] == nums[2] == 1
# left += 1 moves past index 0, the only 0, and the search returns 1
```

**Correct approach:**
```python
else:
    right -= 1  # Safely shrink from right by one element only
```

### 3. **Using Wrong Comparison Target**
Comparing `nums[mid]` with `nums[left]` instead of `nums[right]` makes the logic more complex and error-prone.

**Why it's problematic:** The relationship between mid and left doesn't consistently indicate where the minimum lies, especially with duplicates. The right comparison provides clearer decision boundaries.

**Correct approach:** Always compare with `nums[right]` for consistent logic.

### 4. **Not Handling Single Element or Already Sorted Arrays**
While the provided solution handles these cases correctly, forgetting to consider them during implementation can lead to issues.

**Edge cases to remember:**
- Single element: `[1]` → Should return 1 immediately
- No rotation: `[1, 2, 3, 4, 5]` → Minimum is at index 0
- Full rotation: Array rotated n times returns to original position

The current solution handles all these cases naturally through the while loop condition and comparison logic.

