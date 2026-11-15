# 321. Create Maximum Number

> Source: https://algo.monster/liteproblems/321

## Problem Statement

You have two arrays of integers `nums1` and `nums2` with lengths `m` and `n` respectively. Each array represents the digits of a number. Your task is to create the maximum possible number of exactly `k` digits by selecting digits from both arrays.

The key constraints are:
- The total length `k` must satisfy `k <= m + n`
- When selecting digits from each array, you must maintain their original relative order within that array
- You want to maximize the resulting number

For example, if `nums1 = [3, 4, 6, 5]` and `nums2 = [9, 1, 2, 5, 8, 3]`, and `k = 5`, you need to pick a total of 5 digits from both arrays while preserving the relative order of digits from each array, such that the resulting 5-digit number is as large as possible.

The solution involves three main steps:

1. **Extract subsequences**: For each possible split (taking `x` digits from `nums1` and `k-x` digits from `nums2`), extract the maximum subsequence of the required length from each array using a monotonic stack approach.

2. **Merge subsequences**: Combine the two subsequences into a single array while maintaining the maximum value at each position. This requires comparing the remaining elements lexicographically when the current elements are equal.

3. **Find the overall maximum**: Try all valid combinations of how to split `k` between the two arrays and keep track of the maximum result.

The function `f` extracts the maximum subsequence of length `k` from a single array using a stack-based approach that maintains a decreasing monotonic stack while ensuring exactly `k` elements are selected.

The `compare` function performs lexicographic comparison between two arrays starting from given indices, which is crucial for the merge operation when current elements are equal.

The `merge` function combines two arrays into the maximum possible result by always choosing the larger element (or the one that leads to a larger result when elements are equal).

The main algorithm iterates through all valid ways to split `k` digits between the two arrays, generates the maximum subsequences for each split, merges them, and keeps track of the overall maximum.

## Intuition

The core insight is that we need to decompose this problem into smaller, manageable subproblems. Since we're picking digits from two arrays while preserving order, we can think of this as: "How many digits should I take from `nums1` and how many from `nums2`?"

If we decide to take `x` digits from `nums1`, then we must take `k - x` digits from `nums2`. This gives us a finite number of possibilities to check. For each split, we need to solve two independent problems:
1. What's the maximum number I can form by selecting `x` digits from `nums1` while preserving order?
2. What's the maximum number I can form by selecting `k - x` digits from `nums2` while preserving order?

For extracting the maximum subsequence from a single array, we can use a greedy approach with a monotonic stack. The idea is: as we traverse the array, we want to keep larger digits towards the front. If we encounter a digit larger than what's currently in our result, we should consider removing smaller digits before it (as long as we have enough remaining digits to fill our required length). This is similar to removing digits to form the largest number - we maintain a decreasing sequence as much as possible.

Once we have the optimal subsequences from both arrays, merging them is like merging two sorted sequences, but with a twist. At each step, we want to pick the digit that will lead to the larger overall number. When the current digits are equal, we need to look ahead - we should pick from the array whose remaining portion is lexicographically larger.

The beauty of this approach is that it breaks down a complex problem into three simpler problems:
1. Finding the maximum k-length subsequence from a single array (solved with monotonic stack)
2. Merging two sequences optimally (solved with lookahead comparison)
3. Trying all possible splits (solved with enumeration from `max(0, k - n)` to `min(k, m)`)

The bounds for enumeration are important: we can't take more than `m` digits from `nums1` or more than `n` digits from `nums2`, and we need exactly `k` total digits. This gives us the range `[max(0, k - n), min(k, m)]` for the number of digits to take from `nums1`.

## Solution Approach

The solution implements three key functions that work together to solve the problem:

### 1. Extract Maximum Subsequence (`f` function)

This function extracts the maximum subsequence of length `k` from a single array using a monotonic stack approach:

```python
def f(nums: List[int], k: int) -> List[int]:
    n = len(nums)
    stk = [0] * k  # Pre-allocate stack of size k
    top = -1       # Stack pointer
    remain = n - k # Number of elements we can skip
```

The algorithm maintains:
- `stk`: A stack to build our result of exactly `k` digits
- `top`: Current top position in the stack
- `remain`: How many elements we can afford to skip (important for knowing when we can pop elements)

For each element `x` in `nums`:
1. While the stack has elements, the top element is smaller than `x`, and we still have elements to spare (`remain > 0`), we pop from the stack
2. If the stack isn't full (`top + 1 < k`), we push `x` onto the stack
3. Otherwise, we skip `x` and decrement `remain`

This greedy approach ensures we get the lexicographically largest subsequence of length `k`.

### 2. Lexicographic Comparison (`compare` function)

This recursive function compares two arrays starting from indices `i` and `j`:

```python
def compare(nums1: List[int], nums2: List[int], i: int, j: int) -> bool:
```

The logic:
- If `nums1` is exhausted (`i >= len(nums1)`), return `False`
- If `nums2` is exhausted (`j >= len(nums2)`), return `True`
- If `nums1[i] > nums2[j]`, `nums1` is larger, return `True`
- If `nums1[i] < nums2[j]`, `nums2` is larger, return `False`
- If elements are equal, recursively compare the next elements

This lookahead comparison is crucial for the merge operation when current elements are equal.

### 3. Merge Two Subsequences (`merge` function)

This function merges two arrays to create the maximum possible result:

```python
def merge(nums1: List[int], nums2: List[int]) -> List[int]:
    m, n = len(nums1), len(nums2)
    i = j = 0
    ans = [0] * (m + n)
```

At each position `k` in the result:
- Use `compare` to determine which array to pick from
- If `compare(nums1, nums2, i, j)` returns `True`, pick from `nums1` and increment `i`
- Otherwise, pick from `nums2` and increment `j`

### 4. Main Algorithm

The main function ties everything together:

```python
m, n = len(nums1), len(nums2)
l, r = max(0, k - n), min(k, m)  # Valid range for digits from nums1
ans = [0] * k
```

The bounds calculation:
- Minimum from `nums1`: `max(0, k - n)` - we need at least `k - n` from `nums1` if `nums2` can't provide all `k`
- Maximum from `nums1`: `min(k, m)` - we can't take more than `m` (size of `nums1`) or more than `k` total

For each valid split `x` (taking `x` from `nums1` and `k - x` from `nums2`):
1. Extract maximum subsequence of length `x` from `nums1`
2. Extract maximum subsequence of length `k - x` from `nums2`
3. Merge the two subsequences
4. Update `ans` if the merged result is larger (Python lists compare lexicographically by default)

The time complexity is `O(k * (m + n))` for each split, and we try at most `k + 1` splits, giving us `O(k^2 * (m + n))` overall.

## Example Walkthrough

Let's work through a concrete example to understand the solution approach.

**Given:**
- `nums1 = [3, 9]`
- `nums2 = [8, 9]` 
- `k = 3`

We need to select 3 total digits from both arrays while preserving their relative order to form the maximum possible number.

**Step 1: Determine valid splits**

We need to decide how many digits to take from each array. The valid range for digits from `nums1` is:
- Minimum: `max(0, k - n) = max(0, 3 - 2) = 1` (we need at least 1 from nums1)
- Maximum: `min(k, m) = min(3, 2) = 2` (we can't take more than 2 from nums1)

So we'll try taking 1 or 2 digits from `nums1`.

**Step 2: Try split 1 - Take 1 from nums1, 2 from nums2**

Extract maximum subsequence of length 1 from `nums1 = [3, 9]`:
- Start with empty stack, `remain = 2 - 1 = 1`
- Process 3: Stack empty, push 3. Stack = [3]
- Process 9: 9 > 3 and remain > 0, so pop 3 and push 9. Stack = [9]
- Result: [9]

Extract maximum subsequence of length 2 from `nums2 = [8, 9]`:
- Start with empty stack, `remain = 2 - 2 = 0`
- Process 8: Stack empty, push 8. Stack = [8]
- Process 9: 9 > 8 but remain = 0 (can't pop), push 9. Stack = [8, 9]
- Result: [8, 9]

Merge [9] and [8, 9]:
- Position 0: Compare [9] vs [8, 9]. Since 9 > 8, pick 9 from nums1
- Position 1: Only nums2 remains, pick 8
- Position 2: Only nums2 remains, pick 9
- Merged result: [9, 8, 9]

**Step 3: Try split 2 - Take 2 from nums1, 1 from nums2**

Extract maximum subsequence of length 2 from `nums1 = [3, 9]`:
- Start with empty stack, `remain = 2 - 2 = 0`
- Process 3: Stack empty, push 3. Stack = [3]
- Process 9: 9 > 3 but remain = 0 (can't pop), push 9. Stack = [3, 9]
- Result: [3, 9]

Extract maximum subsequence of length 1 from `nums2 = [8, 9]`:
- Start with empty stack, `remain = 2 - 1 = 1`
- Process 8: Stack empty, push 8. Stack = [8]
- Process 9: 9 > 8 and remain > 0, so pop 8 and push 9. Stack = [9]
- Result: [9]

Merge [3, 9] and [9]:
- Position 0: Compare [3, 9] vs [9]. Need to check nums1[0]=3 vs nums2[0]=9. Since 3 < 9, pick 9 from nums2
- Position 1: Compare [3, 9] vs [] (nums2 exhausted). Pick 3 from nums1
- Position 2: Only nums1 remains, pick 9
- Merged result: [9, 3, 9]

**Step 4: Compare all results**

- Split 1 gave us: [9, 8, 9]
- Split 2 gave us: [9, 3, 9]

Comparing lexicographically: [9, 8, 9] > [9, 3, 9] (since 8 > 3 at position 1)

**Final answer: [9, 8, 9]**

This example demonstrates how the algorithm:
1. Tries different ways to split k between the two arrays
2. Uses a greedy stack-based approach to extract maximum subsequences
3. Merges subsequences optimally using lookahead comparison
4. Keeps track of the overall maximum across all splits

## Python Solution

```python
class Solution:
    def maxNumber(self, nums1: List[int], nums2: List[int], k: int) -> List[int]:
        def get_max_subsequence(nums: List[int], k: int) -> List[int]:
            """
            Extract the maximum subsequence of length k from nums while maintaining order.
            Uses a monotonic stack approach to greedily select largest elements.
            """
            n = len(nums)
            stack = [0] * k  # Pre-allocate stack of size k
            top = -1  # Stack pointer (top index)
            elements_to_drop = n - k  # Number of elements we can skip
          
            for num in nums:
                # Pop smaller elements from stack if we can still drop elements
                while top >= 0 and stack[top] < num and elements_to_drop > 0:
                    top -= 1
                    elements_to_drop -= 1
              
                # Add current element if stack not full
                if top + 1 < k:
                    top += 1
                    stack[top] = num
                else:
                    # Stack is full, just decrement elements_to_drop
                    elements_to_drop -= 1
          
            return stack
      
        def is_greater(nums1: List[int], nums2: List[int], idx1: int, idx2: int) -> bool:
            """
            Compare two arrays starting from given indices.
            Returns True if nums1[idx1:] is lexicographically greater than nums2[idx2:].
            """
            # If nums1 exhausted, it's not greater
            if idx1 >= len(nums1):
                return False
            # If nums2 exhausted but nums1 has elements, nums1 is greater
            if idx2 >= len(nums2):
                return True
            # Compare current elements
            if nums1[idx1] > nums2[idx2]:
                return True
            if nums1[idx1] < nums2[idx2]:
                return False
            # If equal, recursively compare next elements
            return is_greater(nums1, nums2, idx1 + 1, idx2 + 1)
      
        def merge_arrays(nums1: List[int], nums2: List[int]) -> List[int]:
            """
            Merge two arrays to create the maximum possible array.
            Always picks the lexicographically larger remaining portion.
            """
            m, n = len(nums1), len(nums2)
            idx1 = idx2 = 0
            result = [0] * (m + n)
          
            for pos in range(m + n):
                # Choose from nums1 if it has greater or equal remaining portion
                if is_greater(nums1, nums2, idx1, idx2):
                    result[pos] = nums1[idx1]
                    idx1 += 1
                else:
                    result[pos] = nums2[idx2]
                    idx2 += 1
          
            return result
      
        # Main logic
        m, n = len(nums1), len(nums2)
        # Determine valid range for elements to take from nums1
        min_from_nums1 = max(0, k - n)  # Must take at least k-n from nums1
        max_from_nums1 = min(k, m)      # Can take at most min(k, m) from nums1
      
        max_result = [0] * k
      
        # Try all valid splits between nums1 and nums2
        for take_from_nums1 in range(min_from_nums1, max_from_nums1 + 1):
            take_from_nums2 = k - take_from_nums1
          
            # Get maximum subsequences from each array
            subsequence1 = get_max_subsequence(nums1, take_from_nums1)
            subsequence2 = get_max_subsequence(nums2, take_from_nums2)
          
            # Merge the subsequences to get candidate result
            candidate = merge_arrays(subsequence1, subsequence2)
          
            # Update max_result if candidate is lexicographically larger
            if max_result < candidate:
                max_result = candidate
      
        return max_result
```

## Time and Space Complexity

**Time Complexity:** `O((m + n)³)`

The analysis breaks down as follows:

1. The main loop iterates from `l` to `r`, which is at most `min(k, m)` iterations, so `O(k)` iterations.

2. For each iteration:
   - Function `f(nums1, x)`: Takes `O(m)` time to extract subsequence of length `x` from `nums1`
   - Function `f(nums2, k - x)`: Takes `O(n)` time to extract subsequence of length `k - x` from `nums2`
   - Function `merge(arr1, arr2)`: 
     - Creates an array of size `k` and fills it
     - For each position, calls `compare()` which in worst case compares all remaining elements
     - Worst case for `compare()` is `O(k)` when arrays have many equal prefixes
     - Total merge time: `O(k²)`
   - Array comparison `ans < arr`: Takes `O(k)` time

3. Overall time complexity:
   - Number of iterations: `O(k)`
   - Work per iteration: `O(m) + O(n) + O(k²) + O(k)`
   - Since `k ≤ m + n`, we have `O(k) ≤ O(m + n)`
   - Total: `O(k) × O(k²) = O(k³) = O((m + n)³)` in the worst case

**Space Complexity:** `O(m + n)`

The space analysis:

1. Function `f()`: Creates a stack of size `k`, so `O(k)` space
2. Function `compare()`: Uses recursion with maximum depth `O(min(len(nums1), len(nums2)))`, which is `O(m + n)` in worst case
3. Function `merge()`: Creates an array of size `m + n` for merging, so `O(m + n)` space
4. Main function: Stores `arr1`, `arr2`, `arr`, and `ans`, each of size at most `k`, so `O(k)` space
5. Since `k ≤ m + n`, the dominant space complexity is `O(m + n)`

## Common Pitfalls

### 1. Incorrect Merge Logic When Elements Are Equal

**Pitfall**: A naive merge implementation might simply compare current elements and pick the larger one, without considering what comes after when elements are equal.

```python
# INCORRECT merge approach
def merge_wrong(nums1, nums2):
    result = []
    i = j = 0
    while i < len(nums1) and j < len(nums2):
        if nums1[i] >= nums2[j]:  # Wrong! Doesn't consider future elements
            result.append(nums1[i])
            i += 1
        else:
            result.append(nums2[j])
            j += 1
    # ... append remaining elements
```

**Why it fails**: Consider `nums1 = [6, 0]` and `nums2 = [6, 7]`. The naive approach picks `6` from `nums1` first (since `6 >= 6`), which leaves `[0]` against `[6, 7]` and produces `[6, 6, 7, 0]`. Taking the `6` from `nums2` first exposes the `7` sooner and gives the larger `[6, 7, 6, 0]`. Comparing the remaining suffixes (`[6, 0]` vs `[6, 7]`) shows that `nums2` should go first.

**Solution**: Use lexicographic comparison of the remaining portions:
```python
def is_greater(nums1, nums2, i, j):
    while i < len(nums1) and j < len(nums2):
        if nums1[i] != nums2[j]:
            return nums1[i] > nums2[j]
        i += 1
        j += 1
    return i < len(nums1)  # nums1 has more elements remaining
```

### 2. Off-by-One Errors in Stack Implementation

**Pitfall**: Incorrectly managing the stack pointer or the count of elements that can be dropped.

```python
# Alternative stack implementation (also correct)
def get_max_alt(nums, k):
    stack = []
    drop_count = len(nums) - k
    for num in nums:
        while stack and stack[-1] < num and drop_count > 0:
            stack.pop()
            drop_count -= 1
        stack.append(num)  # may grow beyond k elements
    return stack[:k]  # keep the first k
```

**Why it works, and what to watch for**: Unused drops mean the tail of the stack is non-increasing, so cutting it to the first `k` elements keeps the largest prefix. The version is correct; it just stores up to `len(nums)` elements instead of `k`. The real off-by-one risks are elsewhere: popping without checking `drop_count > 0` (which can leave fewer than `k` elements) or decrementing `drop_count` on a push.

**Fixed-size version** (as in the solution), which never stores more than `k` elements:
```python
def get_max_correct(nums, k):
    n = len(nums)
    stack = [0] * k
    top = -1
    remain = n - k
  
    for num in nums:
        while top >= 0 and stack[top] < num and remain > 0:
            top -= 1
            remain -= 1
        if top + 1 < k:
            top += 1
            stack[top] = num
        else:
            remain -= 1
    return stack
```

### 3. Incorrect Bounds Calculation for Split Range

**Pitfall**: Not correctly calculating the valid range of how many elements to take from each array.

```python
# INCORRECT bounds
for take_from_nums1 in range(k + 1):  # Wrong! Doesn't check array sizes
    take_from_nums2 = k - take_from_nums1
    # This might try to take more elements than available
```

**Why it fails**: If `k = 5`, `len(nums1) = 3`, `len(nums2) = 2`, trying to take 4 from `nums1` is impossible.

**Solution**: Calculate proper bounds:
```python
min_from_nums1 = max(0, k - len(nums2))  # Must take at least this many
max_from_nums1 = min(k, len(nums1))      # Can take at most this many

for take_from_nums1 in range(min_from_nums1, max_from_nums1 + 1):
    take_from_nums2 = k - take_from_nums1
    # Now guaranteed: 0 <= take_from_nums1 <= len(nums1)
    #                 0 <= take_from_nums2 <= len(nums2)
```

### 4. Stack Overflow in Recursive Comparison

**Pitfall**: The recursive `is_greater` function can cause stack overflow for very long arrays.

**Solution**: Use an iterative approach:
```python
def is_greater_iterative(nums1, nums2, i, j):
    while i < len(nums1) and j < len(nums2):
        if nums1[i] != nums2[j]:
            return nums1[i] > nums2[j]
        i += 1
        j += 1
    return i < len(nums1)
```

### 5. Not Handling Edge Cases

**Pitfall**: Failing to handle cases where one array is empty or k equals the total length.

**Solution**: The bounds calculation naturally handles these cases, but always verify:
- Empty array: The algorithm should work with `nums1 = []` or `nums2 = []`
- `k = m + n`: Should return all elements merged optimally
- `k = 1`: Should return the single largest element from either array

