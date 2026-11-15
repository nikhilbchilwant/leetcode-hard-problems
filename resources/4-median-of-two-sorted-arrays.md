# 4. Median of Two Sorted Arrays

> Source: https://algo.monster/liteproblems/4

## Problem Statement

You are given two sorted arrays `nums1` and `nums2` with sizes `m` and `n` respectively. Your task is to find **the median** of the combined elements from both arrays.

The median is the middle value in an ordered list of numbers. If the total number of elements is odd, the median is the middle element. If the total number of elements is even, the median is the average of the two middle elements.

For example:
- If the combined sorted array is `[1, 2, 3]`, the median is `2`
- If the combined sorted array is `[1, 2, 3, 4]`, the median is `(2 + 3) / 2 = 2.5`

The key constraint is that your solution must achieve a time complexity of `O(log(m+n))`. This means you cannot simply merge the two arrays and find the median directly, as that would take `O(m+n)` time. Instead, you need to use a more efficient approach like [binary search](/problems/binary_search_intro) or divide-and-conquer.

The solution uses a divide-and-conquer strategy by implementing a helper function `f(i, j, k)` that finds the k-th smallest element across both arrays starting from index `i` in `nums1` and index `j` in `nums2`. The function cleverly eliminates half of the remaining elements in each recursive call by comparing the middle elements of the current search ranges.

To find the median, the solution calculates:
- The `((m+n+1)//2)`-th smallest element (call it `a`)
- The `((m+n+2)//2)`-th smallest element (call it `b`)
- Returns `(a + b) / 2`

This formula works for both odd and even total lengths because when the total length is odd, `a` and `b` will be the same element (the middle one), and when even, they will be the two middle elements.

## Intuition

The key insight begins with recognizing that finding the median is essentially finding the middle element(s) in the combined sorted array. Since we need `O(log(m+n))` time complexity, we can't afford to merge the arrays. This logarithmic requirement immediately suggests [binary search](/problems/binary_search_intro) or divide-and-conquer.

Let's think about what the median represents. For a combined array of size `m+n`:
- If `m+n` is odd, we need the `((m+n+1)/2)`-th smallest element
- If `m+n` is even, we need the average of the `((m+n)/2)`-th and `((m+n)/2 + 1)`-th smallest elements

Here's a clever observation: we can unify both cases! Notice that:
- `((m+n+1)//2)` gives us the lower middle element position
- `((m+n+2)//2)` gives us the upper middle element position
- When `m+n` is odd, these two positions are the same
- When `m+n` is even, these give us the two middle elements

So finding the median reduces to finding the k-th smallest element in two sorted arrays - twice.

Now, how do we find the k-th smallest element efficiently? The breakthrough comes from realizing we can eliminate elements in chunks rather than one by one. If we're looking for the k-th smallest element, we can:

1. Look at the `k/2`-th element in each array
2. Compare these two elements
3. The smaller one (and all elements before it in its array) all rank before the k-th smallest element, so none of them can be the answer
4. We can safely discard these `k/2` elements and now look for the `(k - k/2)`-th smallest element in the remaining portions

Why is this safe? Let `p = k/2` (rounded down) and suppose `nums1`'s `p`-th element `x` is smaller than `nums2`'s `p`-th element `y`. Only the `p - 1` elements before `x` in its own array and at most the `p - 1` elements before `y` in the other array can come before `x`. So `x` is at most the `(p - 1) + (p - 1) + 1 = 2p - 1`-th smallest element, and `2p - 1 <= k - 1`. Every discarded element ranks strictly before position `k`, so the k-th element is still among the remaining ones.

This divide-and-conquer approach eliminates half of the remaining elements to find in each step, giving us the required `O(log(m+n))` complexity. The recursion naturally handles edge cases where one array is exhausted or we've narrowed down to finding the 1st smallest element (which is just the minimum of the two current elements).

**Learn more about [Binary Search](/problems/binary_search_intro) and [Divide and Conquer](/problems/divide_and_conquer_intro) patterns.**

## Solution Approach

The solution implements a divide-and-conquer approach through a recursive helper function `f(i, j, k)` that finds the k-th smallest element between two sorted arrays.

**Main Function Structure:**
```python
def findMedianSortedArrays(self, nums1, nums2):
    m, n = len(nums1), len(nums2)
    a = f(0, 0, (m + n + 1) // 2)  # Find lower middle element
    b = f(0, 0, (m + n + 2) // 2)  # Find upper middle element
    return (a + b) / 2              # Return average
```

The median is calculated as the average of two positions:
- Position `(m + n + 1) // 2` - the lower middle element
- Position `(m + n + 2) // 2` - the upper middle element

When total length is odd, both positions point to the same element. When even, they point to the two middle elements.

**Helper Function `f(i, j, k)` Implementation:**

The function finds the k-th smallest element starting from index `i` in `nums1` and index `j` in `nums2`.

**Base Cases:**
1. **If `i >= m`**: Array `nums1` is exhausted from position `i`. The k-th element must be in `nums2`:
   ```python
   return nums2[j + k - 1]
   ```

2. **If `j >= n`**: Array `nums2` is exhausted from position `j`. The k-th element must be in `nums1`:
   ```python
   return nums1[i + k - 1]
   ```

3. **If `k == 1`**: We need the smallest element between the current positions:
   ```python
   return min(nums1[i], nums2[j])
   ```

**Recursive Case:**

1. Calculate `p = k // 2` - we'll try to eliminate `p` elements

2. Find the p-th element from current positions in both arrays:
   ```python
   x = nums1[i + p - 1] if i + p - 1 < m else inf
   y = nums2[j + p - 1] if j + p - 1 < n else inf
   ```
   If an array doesn't have enough elements, we use `inf` as a placeholder.

3. Compare and eliminate:
   - **If `x < y`**: Elements `nums1[i]` through `nums1[i + p - 1]` are among the smallest and can be eliminated. We advance pointer `i` by `p` and look for the `(k - p)`-th element:
     ```python
     return f(i + p, j, k - p)
     ```

   - **If `x >= y`**: Elements `nums2[j]` through `nums2[j + p - 1]` are among the smallest and can be eliminated. We advance pointer `j` by `p` and look for the `(k - p)`-th element:
     ```python
     return f(i, j + p, k - p)
     ```

**Why This Works:**

When we compare the `p`-th elements from both arrays (`p = k // 2`), the smaller one (and all elements before it) ranks strictly before the k-th smallest element overall. This is because:
- There are `p - 1` elements before it in its own array
- At most `p - 1` elements of the other array can come before it, because the other array's `p`-th element is not smaller than it
- So it is at most the `(p - 1) + (p - 1) + 1 = 2p - 1`-th smallest element, and `2p - 1 <= k - 1`

Therefore, none of the `p` discarded elements can be the k-th smallest, and we can safely discard them and reduce our problem size by half.

**Time Complexity:** `O(log(m + n))` because we eliminate half of the remaining search space in each recursive call.

**Space Complexity:** `O(log(m + n))` for the recursion stack depth.

## Example Walkthrough

Let's walk through finding the median of `nums1 = [1, 3]` and `nums2 = [2, 4]`.

**Step 1: Initial Setup**
- `m = 2`, `n = 2`, total length = 4 (even)
- We need to find:
  - `a = f(0, 0, (4+1)//2) = f(0, 0, 2)` → 2nd smallest element
  - `b = f(0, 0, (4+2)//2) = f(0, 0, 3)` → 3rd smallest element
  - Median = `(a + b) / 2`

**Step 2: Finding `a = f(0, 0, 2)` (2nd smallest element)**

Call `f(0, 0, 2)`:
- `i = 0`, `j = 0`, `k = 2`
- Not a base case (both arrays have elements, k > 1)
- `p = k // 2 = 1`
- `x = nums1[0 + 1 - 1] = nums1[0] = 1`
- `y = nums2[0 + 1 - 1] = nums2[0] = 2`
- Since `x < y` (1 < 2), eliminate `nums1[0]` and recurse

Call `f(1, 0, 1)`:
- `i = 1`, `j = 0`, `k = 1`
- Base case: `k == 1`
- Return `min(nums1[1], nums2[0]) = min(3, 2) = 2`

So `a = 2`

**Step 3: Finding `b = f(0, 0, 3)` (3rd smallest element)**

Call `f(0, 0, 3)`:
- `i = 0`, `j = 0`, `k = 3`
- Not a base case
- `p = k // 2 = 1`
- `x = nums1[0] = 1`
- `y = nums2[0] = 2`
- Since `x < y` (1 < 2), eliminate `nums1[0]` and recurse

Call `f(1, 0, 2)`:
- `i = 1`, `j = 0`, `k = 2`
- Not a base case
- `p = k // 2 = 1`
- `x = nums1[1 + 1 - 1] = nums1[1] = 3`
- `y = nums2[0 + 1 - 1] = nums2[0] = 2`
- Since `x >= y` (3 >= 2), eliminate `nums2[0]` and recurse

Call `f(1, 1, 1)`:
- `i = 1`, `j = 1`, `k = 1`
- Base case: `k == 1`
- Return `min(nums1[1], nums2[1]) = min(3, 4) = 3`

So `b = 3`

**Step 4: Calculate Median**
- Median = `(a + b) / 2 = (2 + 3) / 2 = 2.5`

The combined sorted array would be `[1, 2, 3, 4]`, and indeed the median is `(2 + 3) / 2 = 2.5`.

Notice how the algorithm efficiently found the 2nd and 3rd smallest elements without merging the arrays, eliminating half of the remaining search space at each step.

## Python Solution

```python
class Solution:
    def findMedianSortedArrays(self, nums1: List[int], nums2: List[int]) -> float:
        """
        Find the median of two sorted arrays.

        Args:
            nums1: First sorted array
            nums2: Second sorted array

        Returns:
            The median value of the combined sorted arrays
        """

        def find_kth_element(index1: int, index2: int, k: int) -> int:
            """
            Find the k-th smallest element in the union of nums1[index1:] and nums2[index2:].
            Uses binary search by eliminating k/2 elements at each step.

            Args:
                index1: Starting index in nums1
                index2: Starting index in nums2
                k: The k-th element to find (1-indexed)

            Returns:
                The k-th smallest element
            """
            # Base case: if nums1 is exhausted, return from nums2
            if index1 >= len_nums1:
                return nums2[index2 + k - 1]

            # Base case: if nums2 is exhausted, return from nums1
            if index2 >= len_nums2:
                return nums1[index1 + k - 1]

            # Base case: if k is 1, return the minimum of current elements
            if k == 1:
                return min(nums1[index1], nums2[index2])

            # Binary search: compare elements at position k/2 - 1 from current indices
            half_k = k // 2

            # Get the element at position (index + k/2 - 1) or infinity if out of bounds
            nums1_mid_value = nums1[index1 + half_k - 1] if index1 + half_k - 1 < len_nums1 else float('inf')
            nums2_mid_value = nums2[index2 + half_k - 1] if index2 + half_k - 1 < len_nums2 else float('inf')

            # Eliminate the smaller half and recursively find the (k - k/2)-th element
            if nums1_mid_value < nums2_mid_value:
                # Eliminate first half_k elements from nums1
                return find_kth_element(index1 + half_k, index2, k - half_k)
            else:
                # Eliminate first half_k elements from nums2
                return find_kth_element(index1, index2 + half_k, k - half_k)

        # Store array lengths
        len_nums1, len_nums2 = len(nums1), len(nums2)
        total_length = len_nums1 + len_nums2

        # For odd total length: find the middle element
        # For even total length: find the two middle elements and average them
        # Using (total + 1) / 2 and (total + 2) / 2 handles both cases elegantly
        left_median = find_kth_element(0, 0, (total_length + 1) // 2)
        right_median = find_kth_element(0, 0, (total_length + 2) // 2)

        # Return the average of the two middle values
        # For odd length, left_median == right_median
        return (left_median + right_median) / 2.0
```

## Time and Space Complexity

**Time Complexity:** `O(log(m + n))`

The algorithm uses a recursive approach to find the k-th smallest element in two sorted arrays. The key insight is that in each recursive call, the function `f` eliminates half of the remaining elements to search (specifically `k//2` elements).

- In each recursive call, we compare elements at positions `i + p - 1` and `j + p - 1` where `p = k // 2`
- Based on the comparison, we eliminate `p` elements from either `nums1` or `nums2`
- The value of `k` reduces by `p` in each recursive call, effectively halving the search space
- Since we're reducing `k` by half in each iteration and `k` starts at most at `(m + n + 2) // 2`, the maximum recursion depth is `O(log(m + n))`
- Each recursive call performs `O(1)` operations (comparisons and index calculations)

Therefore, the total time complexity is `O(log(m + n))`.

**Space Complexity:** `O(log(m + n))`

The space complexity is determined by the recursion call stack:

- The function `f` is called recursively without using any additional data structures
- The maximum recursion depth is `O(log(m + n))` as explained above
- Each recursive call uses `O(1)` space for its parameters (`i`, `j`, `k`) and local variables (`p`, `x`, `y`)
- The call stack will have at most `O(log(m + n))` frames

Therefore, the space complexity is `O(log(m + n))` due to the recursion stack.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Off-by-One Errors in Index Calculation**

One of the most common mistakes is incorrectly calculating indices when finding the k-th element or when checking bounds.

**Pitfall Example:**
```python
# Incorrect: forgetting that k is 1-indexed
return nums2[j + k]  # Should be nums2[j + k - 1]

# Incorrect: wrong boundary check
if index1 + half_k < len_nums1:  # Should be index1 + half_k - 1
    nums1_mid_value = nums1[index1 + half_k]
```

**Solution:**
Always remember that:
- The k-th element (1-indexed) is at position `index + k - 1` (0-indexed)
- When comparing elements at k/2 position, use `index + k/2 - 1`

### 2. **Choosing a Sentinel That Can Tie With Real Values**

When one array doesn't have `p` elements left, the code substitutes a sentinel so that array is never chosen for elimination. The sentinel must be strictly larger than every value that can appear in the input. If a real element equals the sentinel (for example, an input containing `Integer.MAX_VALUE` when the Java solution uses `Integer.MAX_VALUE` as the sentinel), the `else` branch can discard `p` elements from an array that does not have them.

**Pitfall Example:**
```python
# Breaks if some input value is >= 10**6
nums1_mid_value = 10**6
```

**Solution:**
The LeetCode constraints keep values within `[-10^6, 10^6]`, so any sentinel above `10^6` (such as `Integer.MAX_VALUE` or `1 << 30`) is safe. In Python, `float('inf')` avoids the question entirely:
```python
nums1_mid_value = nums1[index1 + half_k - 1] if index1 + half_k - 1 < len_nums1 else float('inf')
```

### 3. **Incorrect Handling of k = 0 or k/2 = 0**

When k becomes 1, dividing by 2 gives 0, which can lead to infinite recursion if not handled properly.

**Pitfall Example:**
```python
def find_kth_element(index1, index2, k):
    half_k = k // 2  # When k=1, half_k=0
    # Without the k==1 base case, this would try to eliminate 0 elements
    return find_kth_element(index1 + half_k, index2, k - half_k)  # Infinite loop!
```

**Solution:**
Always include the `k == 1` base case before the recursive logic:
```python
if k == 1:
    return min(nums1[index1], nums2[index2])
```

### 4. **Wrong Median Position Calculation**

A subtle but critical error is using the wrong formula for finding median positions.

**Pitfall Example:**
```python
# Incorrect for odd-length arrays
left_median = find_kth_element(0, 0, total_length // 2)
right_median = find_kth_element(0, 0, total_length // 2 + 1)
# For n = 5 this finds positions 2 and 3, but the median is position 3 alone
```

**Solution:**
Use the formula `(n + 1) // 2` and `(n + 2) // 2`:
```python
left_median = find_kth_element(0, 0, (total_length + 1) // 2)
right_median = find_kth_element(0, 0, (total_length + 2) // 2)
```
This correctly handles both odd and even lengths:
- Odd (n=5): positions 3 and 3 (same element)
- Even (n=4): positions 2 and 3 (two middle elements)

### 5. **Not Handling Empty Arrays**

The code might fail if one or both arrays are empty.

**Pitfall Example:**
```python
# Without proper checks, this could fail
if nums1[index1] < nums2[index2]:  # IndexError if either array is empty
```

**Solution:**
The base cases should handle empty arrays naturally:
```python
if index1 >= len_nums1:  # Handles empty nums1 when len_nums1 = 0
    return nums2[index2 + k - 1]
if index2 >= len_nums2:  # Handles empty nums2 when len_nums2 = 0
    return nums1[index1 + k - 1]
```

### 6. **Forgetting to Update k After Elimination**

When eliminating elements, it's crucial to reduce k by the number of eliminated elements.

**Pitfall Example:**
```python
# Incorrect: not updating k
return find_kth_element(index1 + half_k, index2, k)  # Should be k - half_k
```

**Solution:**
Always subtract the number of eliminated elements from k:
```python
return find_kth_element(index1 + half_k, index2, k - half_k)
```

