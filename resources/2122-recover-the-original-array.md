# 2122. Recover the Original Array

> Source: https://algo.monster/liteproblems/2122

## Problem Statement

Alice originally had an array `arr` of `n` positive integers. She picked a positive integer `k` and created two new arrays from it:
- `lower[i] = arr[i] - k` for each element at index `i`
- `higher[i] = arr[i] + k` for each element at index `i`

Now Alice has lost all three arrays (the original `arr`, `lower`, and `higher`). However, she still remembers all the numbers that were in `lower` and `higher` combined, but she doesn't know which number came from which array.

You're given an array `nums` containing `2n` integers - these are all the values from both `lower` and `higher` mixed together. Your task is to recover the original array `arr`.

**Key observations:**
- The `nums` array has exactly `2n` elements (n from `lower`, n from `higher`)
- For each original value `arr[i]`, there exists `arr[i] - k` in `lower` and `arr[i] + k` in `higher`
- This means for each original value, the difference between its corresponding values in `nums` is `2k`
- The original value `arr[i]` is the average of its two corresponding values: `arr[i] = (lower[i] + higher[i]) / 2`

The solution tries different possible values of `2k` by checking pairs of elements in the sorted `nums` array. For each valid `2k` value (must be even and positive), it attempts to pair up elements that differ by exactly `2k`. If exactly `n` valid pairs can be formed, the average of each pair gives us the original array values.

If multiple valid arrays exist, any one of them can be returned. The problem guarantees at least one valid solution exists.

## Intuition

The key insight is understanding the relationship between the original array and the mixed array we're given. Since each original value `arr[i]` produces two values (`arr[i] - k` and `arr[i] + k`), these two values are always exactly `2k` apart.

Think of it this way: if we sort the mixed array `nums`, the smallest element must be some `arr[i] - k` (since subtracting `k` always gives a smaller value). This smallest element needs a partner that is `2k` larger to form a valid pair.

But here's the challenge - we don't know what `k` is! So how do we find it?

The breakthrough comes from realizing that the smallest element in `nums` must belong to the `lower` array (it's some original value minus `k`). Its corresponding element from the `higher` array must be somewhere else in `nums`, exactly `2k` units away.

Since we don't know which element is its partner, we can try pairing the smallest element with each other element in the array. The difference between them would give us a candidate value for `2k`. For this to be valid:
1. The difference must be even (since `2k` is even when `k` is an integer)
2. The difference must be positive

Once we have a candidate `2k`, we can verify if it's correct by trying to pair up all elements in `nums` with this fixed difference. We use a greedy approach: always try to pair the smallest unpaired element with another element that's exactly `2k` larger.

If we can successfully create exactly `n` pairs where each pair has a difference of `2k`, then we've found the correct value of `k`. The original array values are simply the midpoints of each pair: `(lower_value + higher_value) / 2 = (arr[i] - k + arr[i] + k) / 2 = arr[i]`.

The beauty of this approach is that we systematically try all possible values of `2k` by considering each element as a potential partner for the smallest element, guaranteeing we'll find the correct solution.

**Learn more about [Two Pointers](/problems/two_pointers_intro) and [Sorting](/problems/sorting_summary) patterns.**

## Solution Approach

The implementation follows a systematic approach to find the correct value of `k` and reconstruct the original array:

1. **Sort the input array**: First, we sort `nums` to make it easier to find pairs with a specific difference. After [sorting](/problems/sorting_summary), we know the smallest element must be from the `lower` array.

2. **Try different values of 2k**: We iterate through each element `nums[i]` (starting from index 1) and calculate the difference `d = nums[i] - nums[0]`. This difference represents a candidate value for `2k`.
   - Skip if `d == 0` (same element, can't be a valid pair)
   - Skip if `d % 2 == 1` (odd difference means `k` wouldn't be an integer)

3. **Attempt to pair elements with the current difference**: For each valid candidate `d`:
   - Use a boolean array `vis` to track which elements have been paired
   - Mark `nums[i]` as visited (it's paired with `nums[0]`)
   - Calculate the first element of the original array: `(nums[0] + nums[i]) >> 1` (right shift by 1 is equivalent to dividing by 2)

4. **Greedy pairing process**: Use [two pointers](/problems/two_pointers_intro) `l` and `r` to find pairs:
   - `l` starts at 1 (index 0 is already paired)
   - `r` starts at `i + 1`
   - For each unpaired element at position `l`:
     - Skip already visited elements by advancing `l`
     - Move `r` forward until we find an element where `nums[r] - nums[l] >= d`
     - If `nums[r] - nums[l] == d`, we found a valid pair:
       - Mark `nums[r]` as visited
       - Add the midpoint `(nums[l] + nums[r]) >> 1` to the answer
       - Move both pointers forward
     - If `nums[r] - nums[l] > d` or we reach the end of the array, this value of `d` doesn't work

5. **Verify the solution**: If we successfully create exactly `n >> 1` pairs (which is `n/2`), we've found the correct value of `k` and reconstructed the original array. Return the answer.

6. **Continue if unsuccessful**: If the current value of `d` doesn't yield a complete solution, try the next candidate.

The algorithm efficiently explores all possible values of `2k` in `O(n²)` time in the worst case, where for each of the `O(n)` candidates, we perform an `O(n)` pairing operation. The space complexity is `O(n)` for storing the visited array and the answer.

## Example Walkthrough

Let's walk through a concrete example to understand how the solution works.

**Example:** `nums = [1, 5, 3, 7]`

We need to find the original array of size 2 that Alice had. Let's trace through the algorithm:

**Step 1: Sort the array**
- Sorted `nums = [1, 3, 5, 7]`
- We know `nums[0] = 1` must be from the `lower` array (some `arr[i] - k`)

**Step 2: Try different values of 2k**

**Attempt 1:** Pair `nums[0] = 1` with `nums[1] = 3`
- Difference `d = 3 - 1 = 2` (even, valid candidate for `2k`)
- This means `k = 1`
- Mark indices 0 and 1 as paired
- First element of original array: `(1 + 3) / 2 = 2`
- Now try to pair remaining elements: `nums[2] = 5` and `nums[3] = 7`
- Check if `7 - 5 = 2` ✓ (equals our `d`)
- Second element: `(5 + 7) / 2 = 6`
- Successfully paired all elements!
- **Answer: `[2, 6]`**

Let's verify this is correct:
- Original array: `[2, 6]` with `k = 1`
- Lower array: `[2-1, 6-1] = [1, 5]`
- Higher array: `[2+1, 6+1] = [3, 7]`
- Combined: `[1, 5, 3, 7]` ✓ matches our input

**What if Attempt 1 had failed?**

Let's see what would happen if we tried pairing `nums[0] = 1` with `nums[2] = 5`:
- Difference `d = 5 - 1 = 4` (even, valid candidate for `2k`)
- This means `k = 2`
- Mark indices 0 and 2 as paired
- First element: `(1 + 5) / 2 = 3`
- Try to pair remaining: `nums[1] = 3` and `nums[3] = 7`
- Check if `7 - 3 = 4` ✓ (equals our `d`)
- Second element: `(3 + 7) / 2 = 5`
- **Alternative answer: `[3, 5]`**

Verification:
- Original array: `[3, 5]` with `k = 2`
- Lower array: `[3-2, 5-2] = [1, 3]`
- Higher array: `[3+2, 5+2] = [5, 7]`
- Combined: `[1, 3, 5, 7]` ✓ also matches!

This example shows that multiple valid solutions can exist, and the algorithm will find the first valid one based on the order it tries different values of `2k`.

## Python Solution

```python
class Solution:
    def recoverArray(self, nums: List[int]) -> List[int]:
        # Sort the input array to facilitate pairing
        nums.sort()
        n = len(nums)
      
        # Try different possible values of 2k by pairing nums[0] with each other element
        for i in range(1, n):
            # Calculate the difference (which should be 2k)
            difference = nums[i] - nums[0]
          
            # Skip invalid cases: difference must be positive and even
            if difference == 0 or difference % 2 == 1:
                continue
          
            # Track which elements have been used in pairs
            visited = [False] * n
            visited[i] = True
          
            # Calculate the first element of the original array
            # (nums[0] + nums[i]) / 2 = (arr[j] - k + arr[j] + k) / 2 = arr[j]
            result = [(nums[0] + nums[i]) >> 1]
          
            # Two pointers to find matching pairs
            left_pointer = 1
            right_pointer = i + 1
          
            # Try to pair remaining elements
            while right_pointer < n:
                # Skip already visited elements on the left
                while left_pointer < n and visited[left_pointer]:
                    left_pointer += 1
              
                # Find the matching element on the right with exact difference
                while right_pointer < n and nums[right_pointer] - nums[left_pointer] < difference:
                    right_pointer += 1
              
                # If no valid pair found, this k value doesn't work
                if right_pointer == n or nums[right_pointer] - nums[left_pointer] > difference:
                    break
              
                # Mark the right element as used and add the original value to result
                visited[right_pointer] = True
                result.append((nums[left_pointer] + nums[right_pointer]) >> 1)
              
                # Move both pointers forward
                left_pointer += 1
                right_pointer += 1
          
            # If we successfully paired all elements, return the result
            if len(result) == (n >> 1):
                return result
      
        # No valid k value found (shouldn't happen with valid input)
        return []
```

## Time and Space Complexity

**Time Complexity:** `O(n²)`

The analysis breaks down as follows:
- The initial sorting takes `O(n log n)` time
- The outer loop runs at most `n-1` times (iterating through possible values of `d`)
- For each valid difference `d`, the inner while loop processes each element at most once
  - The `l` pointer traverses from index 1 to n, incrementing through the array
  - The `r` pointer also traverses from `i+1` to n
  - Both pointers move forward only, never backward
  - This gives `O(n)` for each iteration of the outer loop
- In the worst case, we might try all possible differences before finding the valid one
- Total time complexity: `O(n log n) + O(n) × O(n) = O(n²)`

**Space Complexity:** `O(n)`

The space usage consists of:
- The `vis` boolean array of size `n`: `O(n)`
- The `ans` array which stores at most `n/2` elements: `O(n/2) = O(n)`
- A few integer variables (`l`, `r`, `d`, etc.): `O(1)`
- Total space complexity: `O(n)`

Note that the sorting operation might use `O(log n)` additional space depending on the implementation (e.g., quicksort's recursion stack), but the dominant factor remains `O(n)` from the auxiliary arrays.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Pairing Logic - Not Resetting the Right Pointer**

One of the most common mistakes is not properly managing the right pointer during the pairing process. Many implementations incorrectly continue from where the right pointer left off for the next left element, which can miss valid pairs.

**Problematic Code:**
```python
# WRONG: right_pointer continues from previous position
left_pointer = 1
right_pointer = i + 1
while right_pointer < n:
    while left_pointer < n and visited[left_pointer]:
        left_pointer += 1
    # right_pointer doesn't reset, might skip valid pairs
    while right_pointer < n and nums[right_pointer] - nums[left_pointer] < difference:
        right_pointer += 1
```

**Solution:**
The right pointer should start searching from positions after the current left pointer to ensure we don't miss valid pairs due to visited elements:

```python
while left_pointer < n:
    # Skip visited elements
    while left_pointer < n and visited[left_pointer]:
        left_pointer += 1
  
    if left_pointer >= n:
        break
  
    # Reset right_pointer to search from left_pointer + 1
    right_pointer = left_pointer + 1
  
    # Find matching pair
    while right_pointer < n and (visited[right_pointer] or nums[right_pointer] - nums[left_pointer] < difference):
        right_pointer += 1
```

### 2. **Off-by-One Error in Result Array Size Check**

Another pitfall is incorrectly checking if we've found all pairs. Since we have `2n` elements that should form `n` pairs, the result array should have exactly `n` elements.

**Problematic Code:**
```python
# WRONG: Checking for n pairs instead of n/2
if len(result) == n:  # This would expect 2n elements in result
    return result
```

**Solution:**
```python
# CORRECT: Check for n/2 elements (since n = len(nums) = 2 * original_array_size)
if len(result) == (n >> 1):  # or n // 2
    return result
```

### 3. **Not Handling Integer Division Properly**

When calculating the original array values, ensure proper integer division:

**Problematic Code:**
```python
# Potential floating point issues
result.append((nums[left] + nums[right]) / 2)
```

**Solution:**
```python
# Use integer division or bit shift for guaranteed integer result
result.append((nums[left] + nums[right]) >> 1)  # or // 2
```

### 4. **Forgetting to Mark Both Elements in Initial Pair**

When pairing `nums[0]` with `nums[i]`, forgetting to mark `nums[0]` as implicitly used can lead to incorrect pairing attempts:

**Enhanced Solution with Explicit Tracking:**
```python
visited = [False] * n
visited[0] = True  # Explicitly mark nums[0] as used
visited[i] = True  # Mark nums[i] as used
```

While the current implementation works because it starts `left_pointer` at 1, explicitly marking both elements makes the logic clearer and prevents bugs if the code is modified later.

