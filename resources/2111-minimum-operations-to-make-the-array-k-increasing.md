# 2111. Minimum Operations to Make the Array K-Increasing

> Source: https://algo.monster/liteproblems/2111

## Problem Statement

You have an array `arr` of positive integers (0-indexed) and a positive integer `k`. Your task is to determine the minimum number of operations needed to make the array "K-increasing".

An array is **K-increasing** if for every valid index `i` (where `k ≤ i ≤ n-1`), the condition `arr[i-k] ≤ arr[i]` holds true. In other words, if you look at elements that are exactly `k` positions apart, the later element must be greater than or equal to the earlier one.

To understand this better, imagine dividing the array into `k` subsequences:
- Subsequence 0: elements at indices 0, k, 2k, 3k, ...
- Subsequence 1: elements at indices 1, k+1, 2k+1, 3k+1, ...
- Subsequence 2: elements at indices 2, k+2, 2k+2, 3k+2, ...
- And so on...

For the array to be K-increasing, each of these subsequences must be non-decreasing.

**Example walkthrough:**
Given `arr = [4, 1, 5, 2, 6, 2]` and `k = 2`:
- Elements at distance 2:
  - `arr[0]` and `arr[2]`: 4 ≤ 5 ✓
  - `arr[1]` and `arr[3]`: 1 ≤ 2 ✓
  - `arr[2]` and `arr[4]`: 5 ≤ 6 ✓
  - `arr[3]` and `arr[5]`: 2 ≤ 2 ✓
- This array is K-increasing for `k = 2`

But for `k = 1`, we'd need `arr[0] ≤ arr[1]`, which means 4 ≤ 1, which is false.

In one **operation**, you can select any index and change its value to any positive integer. The goal is to find the minimum number of such operations needed to make the array K-increasing.

## Intuition

The key insight is recognizing that the K-increasing property creates **independent subsequences** within the array. When we look at indices that are `k` apart, they form chains that don't interact with each other.

Consider indices 0, k, 2k, 3k, ... - these form one chain. Similarly, indices 1, k+1, 2k+1, ... form another chain. In total, we have exactly `k` such chains, and each chain must be non-decreasing for the array to be K-increasing.

Here's the crucial observation: **changing an element in one chain doesn't affect the other chains**. This means we can solve the problem independently for each chain and sum up the results.

For each chain, we need to make it non-decreasing with minimum changes. This is a classic problem: given a sequence, what's the minimum number of elements to change to make it non-decreasing?

The answer lies in finding the **longest non-decreasing subsequence** (not necessarily contiguous) within that chain. Why? Because:
- The longest non-decreasing subsequence represents the maximum number of elements we can keep unchanged
- All other elements must be modified to fit into this non-decreasing pattern
- Therefore, minimum changes = total elements - longest non-decreasing subsequence length

For finding the longest non-decreasing subsequence efficiently, we use a clever [binary search](/problems/binary_search_intro) technique. We maintain an array `t` where `t[i]` represents the smallest ending value of all non-decreasing subsequences of length `i+1`. When we encounter a new element `x`:
- We find where `x` fits using `bisect_right` (finding the rightmost position where we can insert `x` while maintaining sorted order)
- If `x` is larger than all elements in `t`, we've found a longer subsequence
- Otherwise, we update `t` to keep track of the best (smallest) ending value for subsequences of that length

The final answer is the sum of minimum changes needed across all `k` chains, which the solution computes as `sum(lis(arr[i::k]) for i in range(k))` where `arr[i::k]` extracts the i-th chain.

**Learn more about [Binary Search](/problems/binary_search_intro) patterns.**

## Solution Approach

The implementation consists of two main parts: a helper function to find the minimum changes needed for a single subsequence, and the main logic that applies this to all `k` subsequences.

**Helper Function - Finding Minimum Changes (`lis` function):**

The `lis` function calculates how many elements need to be changed to make a sequence non-decreasing:

1. **Initialize an empty array `t`**: This array will maintain the smallest ending values of non-decreasing subsequences of different lengths.

2. **Process each element `x` in the array**:
   - Use `bisect_right(t, x)` to find the rightmost position where `x` can be inserted while keeping `t` sorted
   - This position tells us the length of the longest non-decreasing subsequence ending with a value ≤ `x`

3. **Update the tracking array**:
   - If `idx == len(t)`, it means `x` is larger than or equal to all elements in `t`, so we can extend the longest subsequence by appending `x`
   - Otherwise, we update `t[idx] = x` to maintain the invariant that `t[i]` stores the smallest ending value for subsequences of length `i+1`

4. **Calculate minimum changes**: Return `len(arr) - len(t)`, where `len(t)` represents the length of the longest non-decreasing subsequence that can be kept unchanged.

**Main Solution Logic:**

The main function uses the independence of the `k` subsequences:

```python
return sum(lis(arr[i::k]) for i in range(k))
```

This line does the following:
- For each starting position `i` from `0` to `k-1`
- Extract the subsequence using slice notation `arr[i::k]`, which gets elements at indices `i`, `i+k`, `i+2k`, etc.
- Apply the `lis` function to find minimum changes needed for that subsequence
- Sum all the minimum changes across all `k` subsequences

**Time Complexity**: `O(n log n)` where `n` is the length of the array. Each element is processed once, and for each element, we perform a [binary search](/problems/binary_search_intro) operation.

**Space Complexity**: `O(n)` in the worst case for storing the tracking array `t` and the subsequences.

The elegance of this solution lies in decomposing the problem into independent subproblems and using an efficient algorithm (longest non-decreasing subsequence with [binary search](/problems/binary_search_intro)) to solve each subproblem optimally.

## Example Walkthrough

Let's walk through the solution with `arr = [5, 4, 3, 2, 1]` and `k = 2`.

**Step 1: Identify the k subsequences**
With `k = 2`, we have 2 independent subsequences:
- Subsequence 0 (indices 0, 2, 4): `[5, 3, 1]`
- Subsequence 1 (indices 1, 3): `[4, 2]`

**Step 2: Process Subsequence 0: `[5, 3, 1]`**
Using the `lis` function to find the longest non-decreasing subsequence:
- Process element 5: `t = [5]` (start with 5)
- Process element 3: `bisect_right([5], 3) = 0`, so update `t[0] = 3`, giving `t = [3]`
- Process element 1: `bisect_right([3], 1) = 0`, so update `t[0] = 1`, giving `t = [1]`

The longest non-decreasing subsequence has length 1 (we can keep just `[1]`).
Minimum changes needed: `3 - 1 = 2` elements must be changed.

**Step 3: Process Subsequence 1: `[4, 2]`**
Using the `lis` function:
- Process element 4: `t = [4]` (start with 4)
- Process element 2: `bisect_right([4], 2) = 0`, so update `t[0] = 2`, giving `t = [2]`

The longest non-decreasing subsequence has length 1 (we can keep just `[2]`).
Minimum changes needed: `2 - 1 = 1` element must be changed.

**Step 4: Sum the results**
Total minimum operations = `2 + 1 = 3`

**Verification of the result:**
One possible K-increasing array after 3 changes could be `[1, 2, 3, 4, 5]`:
- Changed indices 0, 2, and 3
- Now checking K-increasing property:
  - `arr[0] ≤ arr[2]`: 1 ≤ 3 ✓
  - `arr[1] ≤ arr[3]`: 2 ≤ 4 ✓
  - `arr[2] ≤ arr[4]`: 3 ≤ 5 ✓

The algorithm correctly identifies that we need 3 operations to make the array K-increasing.

## Python Solution

```python
from typing import List
import bisect

class Solution:
    def kIncreasing(self, arr: List[int], k: int) -> int:
        """
        Find minimum operations to make array k-increasing.
        A k-increasing array means arr[i] <= arr[i+k] for all valid i.

        Args:
            arr: Input array of integers
            k: The step size for k-increasing property

        Returns:
            Minimum number of elements to change
        """

        def find_min_changes_to_non_decreasing(subsequence: List[int]) -> int:
            """
            Calculate minimum changes needed to make a sequence non-decreasing.
            Uses Longest Non-Decreasing Subsequence (LIS variant) approach.

            Args:
                subsequence: Array to make non-decreasing

            Returns:
                Number of elements that need to be changed
            """
            # Build the longest non-decreasing subsequence using binary search
            # tails[i] stores the smallest tail element for non-decreasing subsequence of length i+1
            tails = []

            for num in subsequence:
                # Find the rightmost position where num can be placed
                # bisect_right allows equal elements (non-decreasing property)
                insertion_pos = bisect.bisect_right(tails, num)

                if insertion_pos == len(tails):
                    # num is larger than or equal to all elements, extend the sequence
                    tails.append(num)
                else:
                    # Replace the element at insertion_pos to maintain smallest possible tail
                    tails[insertion_pos] = num

            # Return number of elements that need to be changed
            # (total length - longest non-decreasing subsequence length)
            return len(subsequence) - len(tails)

        # Process k independent subsequences
        # Each subsequence consists of elements at positions i, i+k, i+2k, ...
        total_changes = 0
        for start_index in range(k):
            # Extract subsequence starting at start_index with step k
            subsequence = arr[start_index::k]
            # Add minimum changes needed for this subsequence
            total_changes += find_min_changes_to_non_decreasing(subsequence)

        return total_changes
```

## Time and Space Complexity

**Time Complexity:** `O(k * (n/k) * log(n/k))` = `O(n * log(n/k))`

The algorithm processes `k` separate subsequences, where each subsequence is formed by taking every k-th element starting from position `i` (for `i` from 0 to k-1). Each subsequence has approximately `n/k` elements.

For each subsequence:
- We iterate through `n/k` elements
- For each element, we perform a binary search using `bisect_right` on array `t`, which takes `O(log(len(t)))` time
- In the worst case, `len(t)` can be up to `n/k`
- Therefore, processing one subsequence takes `O((n/k) * log(n/k))` time

Since we process `k` such subsequences, the total time complexity is:
`k * O((n/k) * log(n/k))` = `O(n * log(n/k))`

**Space Complexity:** `O(n/k)`

The space is dominated by:
- The temporary array `t` in the `lis` function, which in the worst case can store all elements of a subsequence, requiring `O(n/k)` space
- The slicing operation `arr[i::k]` creates a new array of size approximately `n/k`
- Since we process subsequences one at a time (not simultaneously), we only need `O(n/k)` space at any given moment

Therefore, the overall space complexity is `O(n/k)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Using `bisect_left` instead of `bisect_right`**

**The Pitfall:**
A common mistake is using `bisect_left` instead of `bisect_right` when finding the insertion position. This subtle difference can lead to incorrect results because we need to handle equal elements properly for a non-decreasing sequence.

**Why it matters:**
- `bisect_left(tails, x)` returns the leftmost position where `x` can be inserted
- `bisect_right(tails, x)` returns the rightmost position where `x` can be inserted
- For non-decreasing sequences, we need to allow equal consecutive elements

**Example of the bug:**
```python
# Incorrect implementation
def find_min_changes_incorrect(subsequence):
    tails = []
    for num in subsequence:
        # WRONG: Using bisect_left
        insertion_pos = bisect.bisect_left(tails, num)
        if insertion_pos == len(tails):
            tails.append(num)
        else:
            tails[insertion_pos] = num
    return len(subsequence) - len(tails)

# Test case: [1, 2, 2, 3]
# With bisect_left: Would incorrectly replace the first 2 with the second 2
# With bisect_right: Correctly extends the sequence to include both 2s
```

**The Solution:**
Always use `bisect_right` for non-decreasing sequences and `bisect_left` for strictly increasing sequences.

### 2. **Confusing K-increasing with every K-th element**

**The Pitfall:**
Misunderstanding the problem statement and thinking that only every k-th element needs to be checked (like checking arr[0], arr[k], arr[2k] only), rather than understanding that ALL elements at distance k must satisfy the condition.

**Wrong interpretation:**
```python
# INCORRECT: Only checking one subsequence
def wrong_approach(arr, k):
    # This only checks elements at indices 0, k, 2k, ...
    subsequence = arr[::k]
    return find_min_changes(subsequence)
```

**Correct interpretation:**
```python
# CORRECT: Checking all k subsequences
def correct_approach(arr, k):
    total = 0
    for i in range(k):
        # Each subsequence: arr[i], arr[i+k], arr[i+2k], ...
        subsequence = arr[i::k]
        total += find_min_changes(subsequence)
    return total
```

### 3. **Not handling edge cases properly**

**The Pitfall:**
Failing to consider edge cases like:
- When `k >= len(arr)`: No elements need to satisfy the k-increasing property
- When `k = 1`: The entire array must be non-decreasing
- Empty subsequences when array length is small

**The Solution:**
```python
def kIncreasing(self, arr: List[int], k: int) -> int:
    # Edge case handling
    if not arr or k >= len(arr):
        return 0

    total_changes = 0
    for start_index in range(k):
        subsequence = arr[start_index::k]
        if subsequence:  # Only process non-empty subsequences
            total_changes += find_min_changes_to_non_decreasing(subsequence)

    return total_changes
```

### 4. **Memory inefficiency with large arrays**

**The Pitfall:**
Creating all k subsequences at once can consume excessive memory for large arrays.

**Inefficient approach:**
```python
# Creates all subsequences in memory at once
subsequences = [arr[i::k] for i in range(k)]
```

**Memory-efficient solution:**
Process one subsequence at a time as shown in the original solution, or use generators if needed for extremely large datasets.

