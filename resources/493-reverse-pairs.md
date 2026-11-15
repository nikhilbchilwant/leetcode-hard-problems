# 493. Reverse Pairs

> Source: https://algo.monster/liteproblems/493

## Problem Statement

You are given an integer array `nums`. Your task is to count how many **reverse pairs** exist in the array.

A **reverse pair** is defined as a pair of indices `(i, j)` that satisfies both of these conditions:
- The first index comes before the second index: `0 <= i < j < nums.length`
- The element at position `i` is more than twice the element at position `j`: `nums[i] > 2 * nums[j]`

For example, if `nums = [1, 3, 2, 3, 1]`, the reverse pairs would be:
- `(1, 4)` because `nums[1] = 3 > 2 * nums[4] = 2 * 1 = 2`
- `(3, 4)` because `nums[3] = 3 > 2 * nums[4] = 2 * 1 = 2`

So the answer would be 2.

## Intuition

The naive approach would be to check every pair `(i, j)` where `i < j` and count how many satisfy `nums[i] > 2 * nums[j]`. This would take `O(n²)` time, which is inefficient for large arrays.

The key observation is that this problem is similar to counting inversions in an array, but with a modified condition. In the classic inversion problem, we count pairs where `nums[i] > nums[j]` and `i < j`. Here, we need `nums[i] > 2 * nums[j]`.

Why does merge sort work well here? During merge sort, we divide the array into two halves and recursively sort them. At each merge step, we have two sorted subarrays. The crucial insight is that for any element in the left subarray and any element in the right subarray, we naturally have `i < j` (left indices come before right indices).

When we have two sorted halves, we can efficiently count reverse pairs. If we find that `nums[i] > 2 * nums[j]` for some position `i` in the left half and position `j` in the right half, then due to the sorted property, all elements from position `i` to the end of the left half will also satisfy this condition with `nums[j]`. This means we can add `(mid - i + 1)` pairs at once, rather than checking each one individually.

The beauty of this approach is that we can count the reverse pairs during the merge process without adding extra time complexity. We perform two passes during each merge:
1. First pass: Count reverse pairs by comparing `nums[i]` with `2 * nums[j]`
2. Second pass: Actually merge the two sorted halves to maintain the sorted property for the next level

This way, we use the divide-and-conquer nature of merge sort to reduce the problem from `O(n²)` to `O(n log n)`.

## Solution Approach

The solution implements a modified merge sort algorithm to efficiently count reverse pairs. Let's walk through the implementation step by step:

**Main Structure:**
The solution uses a recursive `merge_sort` function that takes left and right boundaries `(l, r)` of the current subarray being processed.

**Base Case:**
```python
if l >= r:
    return 0
```
When the subarray has one or zero elements, there are no pairs to count.

**Divide Step:**
```python
mid = (l + r) >> 1
ans = merge_sort(l, mid) + merge_sort(mid + 1, r)
```
We divide the array at the midpoint and recursively count reverse pairs in both halves. The bit shift operation `>> 1` is equivalent to integer division by 2.

**Count Reverse Pairs:**
```python
i, j = l, mid + 1
while i <= mid and j <= r:
    if nums[i] <= 2 * nums[j]:
        i += 1
    else:
        ans += mid - i + 1
        j += 1
```
This is the crucial counting phase. We use two pointers:
- `i` traverses the left sorted half `[l, mid]`
- `j` traverses the right sorted half `[mid + 1, r]`

When `nums[i] > 2 * nums[j]`, all elements from index `i` to `mid` in the left half will form reverse pairs with `nums[j]` (since both halves are sorted). So we add `(mid - i + 1)` to our count and move `j` forward.

**Merge Phase:**
```python
t = []
i, j = l, mid + 1
while i <= mid and j <= r:
    if nums[i] <= nums[j]:
        t.append(nums[i])
        i += 1
    else:
        t.append(nums[j])
        j += 1
```
After counting, we perform the standard merge operation to combine the two sorted halves into a single sorted array. We use a temporary array `t` to store the merged result.

**Handle Remaining Elements:**
```python
t.extend(nums[i : mid + 1])
t.extend(nums[j : r + 1])
nums[l : r + 1] = t
```
We append any remaining elements from either half to the temporary array, then copy the sorted result back to the original array.

**Time and Space Complexity:**
- Time: `O(n log n)` - We have `log n` levels of recursion, and at each level, we process all `n` elements once for counting and once for merging.
- Space: `O(n)` - For the temporary array used during merging and the recursion stack.

The elegance of this solution lies in using the sorted property of subarrays during merge sort to count reverse pairs in linear time at each level, avoiding the need for nested loops.

## Example Walkthrough

Let's trace through the algorithm with `nums = [2, 4, 3, 5, 1]`.

**Initial Call:** `merge_sort(0, 4)` on `[2, 4, 3, 5, 1]`

**Level 1 - First Split:**
- Calculate `mid = 2`
- Recursively call `merge_sort(0, 2)` for `[2, 4, 3]`
- Recursively call `merge_sort(3, 4)` for `[5, 1]`

**Level 2 - Left Branch `[2, 4, 3]`:**
- Split into `[2, 4]` and `[3]`
- After recursive calls, we have sorted subarrays: `[2, 4]` and `[3]`
- **Count phase:** Compare elements from left `[2, 4]` with twice the elements from right `[3]`
  - `i=0, j=0`: Is `2 > 2*3`? No (2 ≤ 6), move `i` forward
  - `i=1, j=0`: Is `4 > 2*3`? No (4 ≤ 6), move `i` forward
  - No reverse pairs found
- **Merge phase:** Merge to get `[2, 3, 4]`

**Level 2 - Right Branch `[5, 1]`:**
- Split into `[5]` and `[1]`
- **Count phase:** Compare `[5]` with `[1]`
  - `i=0, j=0`: Is `5 > 2*1`? Yes (5 > 2), add 1 pair
- **Merge phase:** Merge to get `[1, 5]`

**Level 1 - Final Merge:**
Now we merge `[2, 3, 4]` and `[1, 5]`
- **Count phase:** Find reverse pairs between left and right halves
  - `i=0, j=0`: Is `2 > 2*1`? No (2 ≤ 2), move `i` forward
  - `i=1, j=0`: Is `3 > 2*1`? Yes (3 > 2), add `(2-1+1) = 2` pairs
    - The left elements 3 and 4 both pair with the right element 1; in original positions these are the pairs `(2, 4)` and `(1, 4)`
  - `j=1`: Is `3 > 2*5`? No (3 ≤ 10), move `i` forward
  - Continue until both pointers reach the end
- **Merge phase:** Merge to get `[1, 2, 3, 4, 5]`

**Total reverse pairs found:** 0 + 1 + 2 = **3**

The three reverse pairs in the original array are:
1. `(1, 4)`: `nums[1]=4 > 2*nums[4]=2`
2. `(2, 4)`: `nums[2]=3 > 2*nums[4]=2`  
3. `(3, 4)`: `nums[3]=5 > 2*nums[4]=2`

The algorithm efficiently counts these pairs by recognizing that once we find one element in the left sorted half that satisfies the condition, all subsequent elements will too.

## Python Solution

```python
class Solution:
    def reversePairs(self, nums: List[int]) -> int:
        """
        Count reverse pairs where i < j and nums[i] > 2 * nums[j]
        using modified merge sort algorithm.
      
        Args:
            nums: List of integers
          
        Returns:
            Count of reverse pairs
        """
      
        def merge_sort_and_count(left: int, right: int) -> int:
            """
            Recursively divide array and count reverse pairs during merge.
          
            Args:
                left: Left boundary index (inclusive)
                right: Right boundary index (inclusive)
              
            Returns:
                Count of reverse pairs in range [left, right]
            """
            # Base case: single element or invalid range
            if left >= right:
                return 0
          
            # Divide the array into two halves
            mid = (left + right) // 2
          
            # Recursively count pairs in left half, right half, and across halves
            count = merge_sort_and_count(left, mid) + merge_sort_and_count(mid + 1, right)
          
            # Count reverse pairs across the two sorted halves
            # For each element in left half, count elements in right half
            # where nums[i] > 2 * nums[j]
            i, j = left, mid + 1
            while i <= mid and j <= right:
                if nums[i] <= 2 * nums[j]:
                    i += 1
                else:
                    # All elements from i to mid satisfy the condition
                    # with current nums[j]
                    count += mid - i + 1
                    j += 1
          
            # Merge the two sorted halves
            temp = []
            i, j = left, mid + 1
          
            # Merge elements in sorted order
            while i <= mid and j <= right:
                if nums[i] <= nums[j]:
                    temp.append(nums[i])
                    i += 1
                else:
                    temp.append(nums[j])
                    j += 1
          
            # Add remaining elements from left half
            temp.extend(nums[i:mid + 1])
          
            # Add remaining elements from right half
            temp.extend(nums[j:right + 1])
          
            # Copy sorted elements back to original array
            nums[left:right + 1] = temp
          
            return count
      
        # Start merge sort from entire array
        return merge_sort_and_count(0, len(nums) - 1)
```

## Time and Space Complexity

**Time Complexity:** `O(n log n)`

The algorithm uses a modified merge sort approach. The merge sort divides the array into two halves recursively until single elements are reached, creating `log n` levels in the recursion tree. At each level:
- The first while loop counts reverse pairs by comparing elements from left and right subarrays, taking `O(n)` time in total across all recursive calls at that level
- The second while loop performs the standard merge operation, also taking `O(n)` time
- The extend operations take `O(n)` time in worst case

Since we have `log n` levels and each level does `O(n)` work, the overall time complexity is `O(n log n)`.

**Space Complexity:** `O(n)`

The space complexity consists of:
- The temporary array `t` used during merging, which can hold at most `n` elements across all recursive calls at the same level: `O(n)`
- The recursion call stack depth, which is `O(log n)` for the divide-and-conquer approach
- Since `O(n) + O(log n) = O(n)`, the overall space complexity is `O(n)`

## Common Pitfalls

### 1. Integer Overflow When Doubling Values

**The Problem:**
When calculating `2 * nums[j]`, integer overflow can occur if `nums[j]` is very large. In Python, this isn't typically an issue due to arbitrary precision integers, but in languages like Java or C++, this can cause incorrect comparisons.

For example, if `nums[j] = 2^30`, then `2 * nums[j] = 2^31` which overflows a 32-bit signed integer, potentially wrapping to a negative value and causing incorrect pair counting.

**Solution:**
Instead of comparing `nums[i] > 2 * nums[j]`, rearrange the comparison to avoid multiplication:
```python
# Instead of:
if nums[i] > 2 * nums[j]:
  
# Use:
if nums[i] / 2.0 > nums[j]:
```

Or use long/64-bit integers in languages with fixed integer sizes:
```python
# For languages with overflow concerns:
if nums[i] > 2 * long(nums[j]):
```

### 2. Off-by-One Errors in Index Management

**The Problem:**
The algorithm uses inclusive boundaries `[left, right]` which can lead to confusion. Common mistakes include:
- Using `mid` instead of `mid + 1` when starting the right half
- Incorrect range calculations when counting pairs
- Wrong slice indices when copying back sorted elements

**Solution:**
Be consistent with boundary conventions and carefully track indices:
```python
# Ensure correct boundaries:
# Left half: [left, mid] (inclusive)
# Right half: [mid + 1, right] (inclusive)

# When counting reverse pairs:
count += mid - i + 1  # "+1" because mid is inclusive

# When extending remaining elements:
temp.extend(nums[i:mid + 1])      # mid + 1 for inclusive mid
temp.extend(nums[j:right + 1])    # right + 1 for inclusive right
```

### 3. Modifying the Counting Logic During Merge

**The Problem:**
A common mistake is trying to count reverse pairs during the actual merge phase instead of in a separate pass. This leads to incorrect counts because elements get moved and the relative positions change.

```python
# WRONG: Trying to count during merge
while i <= mid and j <= right:
    if nums[i] <= nums[j]:
        temp.append(nums[i])
        i += 1
    else:
        # Don't count here! Elements are being reordered
        if nums[i] > 2 * nums[j]:  # This is wrong timing
            count += ...
        temp.append(nums[j])
        j += 1
```

**Solution:**
Always count reverse pairs in a separate pass before the merge operation:
```python
# CORRECT: Count first, then merge
# First pass: count reverse pairs
i, j = left, mid + 1
while i <= mid and j <= right:
    if nums[i] <= 2 * nums[j]:
        i += 1
    else:
        count += mid - i + 1
        j += 1

# Second pass: merge the arrays
i, j = left, mid + 1
# ... merge logic ...
```

### 4. Not Resetting Pointers Between Counting and Merging

**The Problem:**
After counting reverse pairs, the pointers `i` and `j` have moved. Forgetting to reset them before the merge phase causes incorrect merging.

**Solution:**
Always reinitialize pointers before the merge phase:
```python
# After counting phase
i, j = left, mid + 1  # Reset pointers
# Now perform merge
```

### 5. Incorrect Base Case Handling

**The Problem:**
Using `if left > right:` instead of `if left >= right:` in the base case. When `left == right`, we have a single element which cannot form any pairs, so we should return 0.

**Solution:**
Use the correct base case:
```python
if left >= right:  # Handles both empty and single-element cases
    return 0
```

