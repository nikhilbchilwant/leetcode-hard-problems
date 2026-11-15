# 992. Subarrays with K Different Integers

> Source: https://algo.monster/liteproblems/992

## Problem Statement

You are given an integer array `nums` and an integer `k`. Your task is to find the total number of **good subarrays** in `nums`.

A **good array** is defined as an array that contains exactly `k` different integers. For instance, the array `[1,2,3,1,2]` contains exactly 3 different integers: `1`, `2`, and `3`.

A **subarray** is a contiguous sequence of elements from the original array. This means the elements must appear consecutively in the same order as they appear in the original array.

Your goal is to count all possible subarrays of `nums` that have exactly `k` distinct integers.

For example:
- If `nums = [1,2,3,1,2]` and `k = 3`, you need to find all contiguous subarrays that contain exactly 3 different integers
- The subarray `[1,2,3]` would be valid because it has exactly 3 distinct integers
- The subarray `[1,2]` would not be valid because it only has 2 distinct integers
- The subarray `[1,2,3,1]` would be valid because it has exactly 3 distinct integers (1, 2, and 3)

The function should return the total count of such good subarrays.

## Intuition

The key insight is that directly counting subarrays with exactly `k` distinct integers is challenging. However, we can transform this problem using a clever observation: the number of subarrays with exactly `k` distinct integers equals the number of subarrays with at most `k` distinct integers minus the number of subarrays with at most `k-1` distinct integers.

Why does this work? Think about it this way:
- Subarrays with at most `k` distinct integers include those with 1, 2, 3, ..., up to `k` distinct integers
- Subarrays with at most `k-1` distinct integers include those with 1, 2, 3, ..., up to `k-1` distinct integers
- When we subtract the second from the first, we're left with only those subarrays that have exactly `k` distinct integers

This transformation is powerful because counting subarrays with "at most `k` distinct integers" is much easier using a [sliding window](/problems/sliding_window_maximum) approach.

For the [sliding window](/problems/sliding_window_maximum) technique, we can use two pointers. For each ending position `i`, we find the leftmost starting position `j` such that the subarray from `j` to `i` contains at most `k` distinct integers. All subarrays ending at position `i` and starting from any position between `j` and `i` will have at most `k` distinct integers.

The solution cleverly stores these leftmost valid positions in an array `pos`. For each ending position `i`, `pos[i]` stores the leftmost starting position where the subarray still has at most `k` distinct integers. The difference between positions when we compute for `k-1` and `k` gives us the count of subarrays with exactly `k` distinct integers ending at each position.

This approach reduces a complex counting problem into two simpler [sliding window](/problems/sliding_window_maximum) problems, making the solution both elegant and efficient.

**Learn more about [Sliding Window](/problems/sliding_window_maximum) patterns.**

## Solution Approach

In the code, the helper `f` is named `find_left_boundaries`, `pos` is `left_boundaries`, `cnt` is `frequency_map`, `j` is `left_pointer`, and `i` is `right_pointer`.

The implementation uses a helper function `f(k)` that calculates, for each position in the array, the leftmost starting position of a subarray ending at that position with at most `k` distinct integers.

**Helper Function `f(k)` Implementation:**

1. **Initialize data structures:**
   - `pos`: An array to store the leftmost valid starting position for each ending position
   - `cnt`: A Counter (hash map) to track the frequency of each integer in the current window
   - `j`: A pointer representing the left boundary of the [sliding window](/problems/sliding_window_maximum), initialized to 0

2. **[Sliding window](/problems/sliding_window_maximum) process:**
   - Iterate through the array with index `i` and value `x`
   - Add the current element to the window: `cnt[x] += 1`
   - **Shrink window when needed:** While the number of distinct integers exceeds `k`:
     - Decrement the count of the leftmost element: `cnt[nums[j]] -= 1`
     - If the count becomes 0, remove that element from the counter entirely: `cnt.pop(nums[j])`
     - Move the left pointer right: `j += 1`
   - Store the current left boundary for position `i`: `pos[i] = j`

3. **Return the position array** which contains the leftmost valid starting positions for subarrays with at most `k` distinct integers

**Main Algorithm:**

1. Call `f(k-1)` to get positions for subarrays with at most `k-1` distinct integers
2. Call `f(k)` to get positions for subarrays with at most `k` distinct integers
3. **Calculate the result:** For each ending position, subtract the position from `f(k)` from the position from `f(k-1)`
   - This difference gives the number of subarrays ending at that position with exactly `k` distinct integers
   - Sum all these differences: `sum(a - b for a, b in zip(f(k - 1), f(k)))`

**Why the subtraction works:**
- For position `i`, `f(k)[i]` gives the leftmost position where a subarray can start and have at most `k` distinct integers
- `f(k-1)[i]` gives the leftmost position for at most `k-1` distinct integers
- Subarrays starting between `f(k)[i]` and `f(k-1)[i] - 1` will have exactly `k` distinct integers
- The count of such subarrays is `f(k-1)[i] - f(k)[i]`

The time complexity is `O(n)` where `n` is the length of the array, as each element is visited at most twice by the two pointers. The space complexity is `O(n)` for storing the position arrays.

## Example Walkthrough

Let's walk through a small example with `nums = [1,2,1,2,3]` and `k = 2`.

**Goal:** Find all subarrays with exactly 2 distinct integers.

**Step 1: Find subarrays with at most k-1=1 distinct integers**

Using the sliding window with `f(1)`:
- i=0, nums[0]=1: Window [1], distinct=1, j=0, pos[0]=0
- i=1, nums[1]=2: Window [2], distinct=1 (shrink: remove 1), j=1, pos[1]=1  
- i=2, nums[2]=1: Window [1], distinct=1 (shrink: remove 2), j=2, pos[2]=2
- i=3, nums[3]=2: Window [2], distinct=1 (shrink: remove 1), j=3, pos[3]=3
- i=4, nums[4]=3: Window [3], distinct=1 (shrink: remove 2), j=4, pos[4]=4

Result: `f(1) = [0, 1, 2, 3, 4]`

**Step 2: Find subarrays with at most k=2 distinct integers**

Using the sliding window with `f(2)`:
- i=0, nums[0]=1: Window [1], distinct=1, j=0, pos[0]=0
- i=1, nums[1]=2: Window [1,2], distinct=2, j=0, pos[1]=0
- i=2, nums[2]=1: Window [1,2,1], distinct=2, j=0, pos[2]=0
- i=3, nums[3]=2: Window [1,2,1,2], distinct=2, j=0, pos[3]=0
- i=4, nums[4]=3: Window [1,2,1,2,3] has distinct=3, so shrink: remove nums[0]=1, nums[1]=2, nums[2]=1 (the last 1 leaves the map), window [2,3], j=3, pos[4]=3

Result: `f(2) = [0, 0, 0, 0, 3]`

**Step 3: Calculate the difference**

For each position i, the number of subarrays ending at i with exactly 2 distinct integers is:
- i=0: f(1)[0] - f(2)[0] = 0 - 0 = 0 (no subarrays with exactly 2 distinct)
- i=1: f(1)[1] - f(2)[1] = 1 - 0 = 1 (subarray [1,2])
- i=2: f(1)[2] - f(2)[2] = 2 - 0 = 2 (subarrays [1,2,1] and [2,1])
- i=3: f(1)[3] - f(2)[3] = 3 - 0 = 3 (subarrays [1,2,1,2], [2,1,2], and [1,2])
- i=4: f(1)[4] - f(2)[4] = 4 - 3 = 1 (subarray [2,3])

**Total:** 0 + 1 + 2 + 3 + 1 = **7 subarrays with exactly 2 distinct integers**

The subarrays are: [1,2], [1,2,1], [2,1], [1,2,1,2], [2,1,2], [1,2] (at position 2-3), [2,3]

## Python Solution

```python
class Solution:
    def subarraysWithKDistinct(self, nums: List[int], k: int) -> int:
        def find_left_boundaries(max_distinct):
            """
            For each index i, find the leftmost index j such that 
            nums[j:i+1] contains at most max_distinct distinct integers.
          
            Returns a list where left_boundaries[i] represents the leftmost 
            valid starting position for subarrays ending at index i.
            """
            left_boundaries = [0] * len(nums)
            frequency_map = Counter()
            left_pointer = 0
          
            for right_pointer, current_num in enumerate(nums):
                # Add current element to the window
                frequency_map[current_num] += 1
              
                # Shrink window from left while we have too many distinct elements
                while len(frequency_map) > max_distinct:
                    left_num = nums[left_pointer]
                    frequency_map[left_num] -= 1
                  
                    # Remove element from map if its count reaches 0
                    if frequency_map[left_num] == 0:
                        del frequency_map[left_num]
                  
                    left_pointer += 1
              
                # Store the leftmost valid position for current right_pointer
                left_boundaries[right_pointer] = left_pointer
          
            return left_boundaries
      
        # Get left boundaries for at most (k-1) and at most k distinct elements
        boundaries_k_minus_1 = find_left_boundaries(k - 1)
        boundaries_k = find_left_boundaries(k)
      
        # The difference gives us the count of subarrays with exactly k distinct
        # For each position i, boundaries_k_minus_1[i] - boundaries_k[i] gives
        # the number of valid starting positions for subarrays ending at i
        # that have exactly k distinct integers
        return sum(left_k_minus_1 - left_k 
                   for left_k_minus_1, left_k in zip(boundaries_k_minus_1, boundaries_k))
```

## Time and Space Complexity

**Time Complexity:** `O(n)`

The algorithm calls function `f` twice with parameters `k-1` and `k`. Each call to `f` performs:
- A single pass through the array with the outer loop iterating `n` times
- The inner while loop moves pointer `j` from left to right across the array
- Since `j` only moves forward and never resets, each element is visited at most twice (once by `i`, once by `j`)
- Counter operations (add, remove, check size) are `O(1)` on average
- The final zip and sum operation is `O(n)`

Therefore, the total time complexity is `O(n) + O(n) + O(n) = O(n)`.

**Space Complexity:** `O(n + k)`

The space usage includes:
- Two arrays `pos` of size `n` (one for each call to `f`): `O(n)`
- Counter dictionary `cnt` which stores at most `k` distinct elements in each call: `O(k)`
- The zip operation creates pairs but processes them on-the-fly: `O(1)` additional space

Therefore, the total space complexity is `O(n + k)`, which simplifies to `O(n)` when `k ≤ n`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Understanding of the Subtraction Logic**

**Pitfall:** Many developers struggle to understand why subtracting `f(k)` from `f(k-1)` gives subarrays with exactly `k` distinct integers. They might incorrectly think they need to count subarrays directly.

**Why it happens:** The concept that "exactly k" = "at most k" - "at most k-1" is not immediately intuitive.

**Solution:** Visualize with an example:
- If `f(k)[i] = 2`, subarrays starting from indices 2, 3, 4, ..., i have at most k distinct integers
- If `f(k-1)[i] = 5`, subarrays starting from indices 5, 6, 7, ..., i have at most k-1 distinct integers
- Therefore, subarrays starting from indices 2, 3, 4 (positions between f(k)[i] and f(k-1)[i]-1) have exactly k distinct integers

### 2. **Off-by-One Error in Window Management**

**Pitfall:** Incorrectly managing the sliding window boundaries, especially when removing elements from the frequency map.

```python
# WRONG: Removing before decrementing
while len(frequency_map) > max_distinct:
    if frequency_map[nums[left_pointer]] == 1:
        del frequency_map[nums[left_pointer]]
    frequency_map[nums[left_pointer]] -= 1  # This will fail!
    left_pointer += 1
```

**Solution:** Always decrement first, then check if removal is needed:
```python
# CORRECT: Decrement first, then remove if zero
while len(frequency_map) > max_distinct:
    frequency_map[nums[left_pointer]] -= 1
    if frequency_map[nums[left_pointer]] == 0:
        del frequency_map[nums[left_pointer]]
    left_pointer += 1
```

### 3. **Edge Case: k = 0 or k > array length**

**Pitfall:** Worrying that `k` larger than the number of unique elements breaks the method. The constraints guarantee `k >= 1`, and when `k` exceeds the number of unique values, both boundary arrays are all zeros, so the solution already returns 0. An explicit early return is optional and only skips work:
```python
def subarraysWithKDistinct(self, nums: List[int], k: int) -> int:
    if k == 0 or k > len(set(nums)):
        return 0
    # ... rest of the code
```

### 4. **Memory Inefficiency with Position Arrays**

**Pitfall:** Storing entire position arrays when only the differences are needed, which can be memory-intensive for large inputs.

**Solution:** Calculate the result on-the-fly without storing full arrays:
```python
def subarraysWithKDistinct(self, nums: List[int], k: int) -> int:
    def count_at_most_k(max_distinct):
        result = 0
        frequency_map = Counter()
        left_pointer = 0
      
        for right_pointer, current_num in enumerate(nums):
            frequency_map[current_num] += 1
          
            while len(frequency_map) > max_distinct:
                frequency_map[nums[left_pointer]] -= 1
                if frequency_map[nums[left_pointer]] == 0:
                    del frequency_map[nums[left_pointer]]
                left_pointer += 1
          
            # Add count of all subarrays ending at right_pointer
            result += right_pointer - left_pointer + 1
      
        return result
  
    return count_at_most_k(k) - count_at_most_k(k - 1)
```

### 5. **Using Dictionary Instead of Counter**

**Pitfall:** Using a regular dictionary and forgetting to initialize keys before incrementing.

```python
# WRONG: This will raise KeyError
frequency_map = {}
frequency_map[current_num] += 1  # KeyError if current_num not in map
```

**Solution:** Use `Counter` from collections or handle initialization:
```python
# Option 1: Use Counter
from collections import Counter
frequency_map = Counter()

# Option 2: Handle initialization manually
frequency_map = {}
frequency_map[current_num] = frequency_map.get(current_num, 0) + 1
```

