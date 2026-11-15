# 1982. Find Array Given Subset Sums

> Source: https://algo.monster/liteproblems/1982

## Problem Statement

You need to recover an unknown array of length `n` by using information about all its subset sums.

Given:
- An integer `n` representing the length of the unknown array you want to find
- An array `sums` containing all `2^n` possible subset sums of the unknown array (these sums are given in no particular order)

Your task is to determine what the original array was and return it. If there are multiple valid arrays that could produce the given subset sums, you can return any one of them.

Key concepts to understand:
- A **subset** of an array is formed by selecting zero or more elements from the original array
- A **subset sum** is the sum of all elements in a subset
- The empty subset has a sum of `0`
- For an array of length `n`, there are exactly `2^n` possible subsets (including the empty subset)

For example, if the unknown array was `[1, 2]`:
- The subsets would be: `[]`, `[1]`, `[2]`, `[1, 2]`
- The subset sums would be: `0`, `1`, `2`, `3`

The problem guarantees that there will always be at least one valid solution for the given test cases.

The challenge lies in working backwards from the subset sums to figure out what elements must have been in the original array. This requires carefully analyzing the relationships between different subset sums to deduce the individual elements.

## Intuition

The key insight is that we can iteratively extract elements from the subset sums by exploiting a fundamental property: when we know an element `x` is in the original array, we can partition all subset sums into two groups - those that include `x` and those that don't.

Let's think about this step by step:

1. **Handling negative numbers**: Since the array can contain negative numbers, we first shift all subset sums by adding the minimum value's absolute value (`m = -min(sums)`). This ensures all working values are non-negative, making them easier to handle.

2. **Finding the first element**: After shifting and removing `0` (empty subset), the smallest remaining value must be one of the original array elements. Why? Because the smallest non-zero subset sum can only come from a single element (the smallest positive element after shifting).

3. **Finding subsequent elements**: Once we know the first `i` elements, we have `2^i` subset sums formed by these elements. We can calculate and remove all these known subset sums from our list. After removal, the smallest remaining value must be the next element of the original array.

4. **The iterative process**: 
   - Start with element at position 0: `ans[0] = smallest non-zero value`
   - For position `i`, remove all subset sums formed by combinations of `ans[0]` through `ans[i-1]`
   - The smallest remaining value becomes `ans[i]`
   - This works because any remaining subset sum must include at least one element we haven't discovered yet, and the smallest such sum uses only one new element

5. **Restoring original signs**: After finding all elements in their shifted form, we need to determine which elements were originally negative. We find a subset whose sum equals `m` (the shift amount). The elements in this subset were originally negative, so we flip their signs.

This greedy approach works because at each step, we're guaranteed that the smallest unexplained subset sum reveals a new element of the original array.

**Learn more about [Divide and Conquer](/problems/divide_and_conquer_intro) patterns.**

## Solution Approach

Let's walk through the implementation step by step:

**1. Initial Setup and Shifting:**
```python
m = -min(sums)
sl = SortedList(x + m for x in sums)
sl.remove(0)
```
- Calculate `m` as the absolute value of the minimum element in `sums`
- Create a `SortedList` with all values shifted by `m` to make them non-negative
- Remove `0` (representing the empty subset) from the sorted list
- Using `SortedList` allows efficient insertion and removal while maintaining sorted order

**2. Finding the First Element:**
```python
ans = [sl[0]]
```
- After removing `0`, the smallest value in the sorted list must be an element from the original array
- This becomes our first recovered element

**3. Iteratively Finding Remaining Elements:**
```python
for i in range(1, n):
    for j in range(1 << i):
        if j >> (i - 1) & 1:
            s = sum(ans[k] for k in range(i) if j >> k & 1)
            sl.remove(s)
    ans.append(sl[0])
```
- For each position `i` from 1 to n-1:
  - Generate all subsets of the first `i` elements using bit manipulation
  - `1 << i` gives us `2^i` possible subsets
  - The condition `j >> (i - 1) & 1` ensures we only consider subsets that include the most recently added element `ans[i-1]`
  - For each such subset, calculate its sum and remove it from the sorted list
  - After removing all known subset sums, the smallest remaining value is the next element

**4. Restoring Original Signs:**
```python
for i in range(1 << n):
    s = sum(ans[j] for j in range(n) if i >> j & 1)
    if s == m:
        for j in range(n):
            if i >> j & 1:
                ans[j] *= -1
        break
```
- Iterate through all `2^n` possible subsets of the recovered array
- Find the subset whose sum equals `m` (the shift amount)
- The elements in this subset were originally negative before shifting
- Multiply these elements by -1 to restore their original negative values
- Once found, break out of the loop

**Key Techniques Used:**
- **Bit manipulation** for subset generation: `i >> j & 1` checks if the j-th bit of i is set
- **SortedList data structure** for maintaining sorted order with efficient removals
- **Greedy selection**: Always choosing the smallest unexplained value as the next element
- **Shift-and-restore pattern**: Working with non-negative values then restoring signs at the end

The algorithm has a time complexity of `O(2^n * n)` for generating and processing all subsets, which is acceptable given that the input already contains `2^n` subset sums.

## Example Walkthrough

Let's work through a concrete example where the original array is `[1, -3, 2]`.

**Step 1: Understanding the input**
- Original array: `[1, -3, 2]` (this is what we need to recover)
- All possible subsets and their sums:
  - `[]` → 0
  - `[1]` → 1
  - `[-3]` → -3
  - `[2]` → 2
  - `[1, -3]` → -2
  - `[1, 2]` → 3
  - `[-3, 2]` → -1
  - `[1, -3, 2]` → 0
- Given `sums = [0, 1, -3, 2, -2, 3, -1, 0]` (in no particular order)

**Step 2: Shift all values to make them non-negative**
- Find minimum: `min(sums) = -3`
- Shift amount: `m = -(-3) = 3`
- Shifted sums: `[3, 4, 0, 5, 1, 6, 2, 3]`
- Create SortedList: `[0, 1, 2, 3, 3, 4, 5, 6]`
- Remove 0 (empty subset): `[1, 2, 3, 3, 4, 5, 6]`

**Step 3: Find first element**
- Smallest value in list: `1`
- First element: `ans[0] = 1`
- Current array: `[1]`

**Step 4: Find second element**
- Remove subsets formed by `[1]`:
  - Subset `[1]` has sum `1` → remove from list
- Updated list: `[2, 3, 3, 4, 5, 6]`
- Smallest value: `2`
- Second element: `ans[1] = 2`
- Current array: `[1, 2]`

**Step 5: Find third element**
- Remove subsets formed by `[1, 2]` that include `ans[1] = 2`:
  - Subset `[2]` has sum `2` → remove
  - Subset `[1, 2]` has sum `3` → remove
- Updated list: `[3, 4, 5, 6]`
- Smallest value: `3`
- Third element: `ans[2] = 3`
- Current array: `[1, 2, 3]`

**Step 6: Restore original signs**
- We need to find which subset sums to `m = 3`
- Check all subsets of `[1, 2, 3]`:
  - `[]` → 0 (not 3)
  - `[1]` → 1 (not 3)
  - `[2]` → 2 (not 3)
  - `[3]` → 3 ✓ (equals 3!)
- Elements in this subset were originally negative
- Flip sign of element at index 2: `ans[2] = -3`

**Final Result:** `[1, 2, -3]`

Note that the order might differ from the original `[1, -3, 2]`, but both arrays produce the same set of subset sums, which is what matters for this problem.

## Python Solution

```python
class Solution:
    def recoverArray(self, n: int, sums: List[int]) -> List[int]:
        # Find the sum of all negative elements (minimum subset sum)
        negative_sum_offset = -min(sums)
      
        # Create sorted list with all sums shifted by the offset to make them non-negative
        # This ensures the empty subset sum becomes 0
        sorted_sums = SortedList(subset_sum + negative_sum_offset for subset_sum in sums)
      
        # Remove the empty subset sum (which is 0 after shifting)
        sorted_sums.remove(0)
      
        # The smallest remaining sum must be one of the original array elements
        result = [sorted_sums[0]]
      
        # Find remaining n-1 elements
        for element_index in range(1, n):
            # Remove all subset sums that include the elements found so far
            for subset_mask in range(1 << element_index):
                # Check if this subset includes the most recently found element
                if subset_mask >> (element_index - 1) & 1:
                    # Calculate sum of this subset
                    subset_sum = sum(result[bit_pos] for bit_pos in range(element_index) 
                                    if subset_mask >> bit_pos & 1)
                    sorted_sums.remove(subset_sum)
          
            # After removal, the smallest sum is the next element
            result.append(sorted_sums[0])
      
        # Determine which elements should be negative
        # Try all possible sign combinations
        for sign_mask in range(1 << n):
            # Calculate sum with current sign assignment
            current_sum = sum(result[bit_pos] for bit_pos in range(n) 
                            if sign_mask >> bit_pos & 1)
          
            # If this combination gives us the original negative sum
            if current_sum == negative_sum_offset:
                # Apply negative signs to selected elements
                for bit_pos in range(n):
                    if sign_mask >> bit_pos & 1:
                        result[bit_pos] *= -1
                break
      
        return result
```

## Time and Space Complexity

**Time Complexity:** `O(n * 2^n)`

The time complexity analysis breaks down as follows:
- The outer loop runs `n` times (from creating the first element to the nth element)
- For each iteration `i`, the inner loop runs `2^i` times to remove subset sums from the sorted list
- The removal operation from SortedList takes `O(log(2^n))` = `O(n)` time
- Total removal operations: `∑(i=1 to n-1) 2^i = 2^n - 2`
- Cost of all removals: `O((2^n - 2) * n)` = `O(n * 2^n)`
- The final loop to find the correct sign assignment runs through all `2^n` subsets once, with each subset sum calculation taking `O(n)` time
- Total time: `O(n * 2^n) + O(n * 2^n)` = `O(n * 2^n)`

**Space Complexity:** `O(2^n)`

The space complexity analysis:
- The SortedList `sl` stores all `2^n` subset sums initially: `O(2^n)`
- The answer array `ans` stores `n` elements: `O(n)`
- The bit manipulation operations use constant extra space: `O(1)`
- Total space: `O(2^n) + O(n)` = `O(2^n)` (since `2^n` dominates `n`)

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Handling of Duplicate Values in Subset Sums**

**The Pitfall:**
When removing subset sums from the sorted list, if there are duplicate values, you might accidentally remove the wrong occurrence. For example, if multiple different subsets produce the same sum, the algorithm needs to handle these duplicates correctly.

**Why It Happens:**
- The input array might contain duplicate elements or elements that combine to produce identical sums
- When we call `sorted_sums.remove(subset_sum)`, it only removes one occurrence
- If the same sum appears multiple times (from different subsets), we need to ensure we're removing the correct number of occurrences

**Solution:**
The current implementation actually handles this correctly by:
1. Processing subsets in a specific order (only those containing the most recent element)
2. Removing each sum exactly once per iteration
3. The SortedList maintains all duplicates and removes them one at a time

However, if you were to modify the algorithm, ensure you track which sums have been accounted for.

### 2. **Bit Manipulation Off-by-One Errors**

**The Pitfall:**
Getting the bit positions wrong when checking subset membership:
```python
# Incorrect: might check wrong bit position
if subset_mask >> element_index & 1:  # Wrong!

# Correct: check the (element_index - 1)th bit
if subset_mask >> (element_index - 1) & 1:  # Right!
```

**Why It Happens:**
- Array indices are 0-based, but we're iterating element_index from 1 to n-1
- The bit position for checking if element at index i-1 is included should be (i-1)

**Solution:**
Always be careful with index calculations in bit manipulation:
- Use parentheses to ensure correct order of operations
- Remember that bit position k corresponds to array index k
- Test with small examples to verify bit patterns

### 3. **Not Handling the Sign Restoration Correctly**

**The Pitfall:**
After finding the absolute values of array elements, incorrectly determining which elements should be negative:
```python
# Wrong approach: assuming any subset that sums to offset works
for sign_mask in range(1 << n):
    current_sum = sum(result[j] for j in range(n) if sign_mask >> j & 1)
    if current_sum == negative_sum_offset:
        # Forgetting to actually apply the negative signs!
        return result  # Wrong - returns all positive values
```

**Why It Happens:**
- The algorithm shifts all values to be non-negative for processing
- After finding the elements, we need to restore the original signs
- Simply finding which subset sums to the offset isn't enough - we must apply the sign changes

**Solution:**
Always complete the sign restoration:
```python
if current_sum == negative_sum_offset:
    for bit_pos in range(n):
        if sign_mask >> bit_pos & 1:
            result[bit_pos] *= -1  # Actually apply the negative sign
    break
```

### 4. **Inefficient Data Structure Choice**

**The Pitfall:**
Using a regular list instead of SortedList and manually sorting after each operation:
```python
# Inefficient approach
sums_list = list(sums)
sums_list.sort()
# ... later in the code
sums_list.remove(subset_sum)
sums_list.sort()  # O(n log n) each time!
```

**Why It Happens:**
- Not being aware of the SortedList data structure from the sortedcontainers library
- Trying to use built-in Python lists which don't maintain sorted order after modifications

**Solution:**
Use SortedList which maintains sorted order automatically:
- O(log n) insertion and removal
- Always maintains sorted order without explicit sorting
- Provides efficient access to elements by index

### 5. **Edge Case: Array Contains Zero**

**The Pitfall:**
If the original array contains 0, the algorithm might get confused since removing subsets with 0 doesn't change the sum:
```python
# If original array is [0, 1, 2], subset sums include:
# [], [0], [1], [2], [0,1], [0,2], [1,2], [0,1,2]
# Sums: 0, 0, 1, 2, 1, 2, 3, 3 (duplicates!)
```

**Why It Happens:**
- Zero doesn't contribute to subset sums
- This creates duplicate values in the sums array
- The algorithm needs to handle these duplicates correctly

**Solution:**
The current algorithm handles this correctly because:
- SortedList maintains all duplicates
- We remove sums one at a time in a controlled manner
- The greedy selection of the smallest remaining value still works

