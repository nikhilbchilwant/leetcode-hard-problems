# 354. Russian Doll Envelopes

> Source: https://algo.monster/liteproblems/354

## Problem Statement

You have a collection of envelopes, each with a width and height. The goal is to find the maximum number of envelopes you can nest inside each other, like Russian dolls.

An envelope `[w1, h1]` can fit inside another envelope `[w2, h2]` only if both conditions are met:
- `w1 < w2` (the first envelope's width is strictly less than the second's)
- `h1 < h2` (the first envelope's height is strictly less than the second's)

You need to determine the longest chain of envelopes where each envelope in the sequence can fit inside the next one. Envelopes cannot be rotated.

For example, if you have envelopes `[[5,4], [6,4], [6,7], [2,3]]`, the maximum number you can nest is 3: `[2,3] → [5,4] → [6,7]`.

The solution uses a clever approach by:
1. Sorting envelopes by width (ascending) and height (descending for same width)
2. Finding the longest increasing subsequence (LIS) based on heights only

The sorting strategy ensures that envelopes with the same width cannot be nested (due to descending height order), while the LIS on heights gives us the maximum nesting sequence. The algorithm uses binary search (`bisect_left`) to efficiently maintain the LIS, achieving `O(n log n)` time complexity.

## Intuition

The key insight is recognizing this problem as a 2D version of the Longest Increasing Subsequence (LIS) problem. In standard LIS, we find the longest sequence where each element is greater than the previous one. Here, we need both dimensions (width and height) to be increasing.

The challenge is handling two dimensions simultaneously. We can't simply apply LIS on pairs because we need both width AND height to increase together. This is where the sorting strategy becomes crucial.

By sorting envelopes by width first, we ensure that as we traverse the array, we only need to worry about one dimension - the height. Once sorted by width, any valid nesting sequence must have increasing heights as well.

But there's a subtle issue: what if multiple envelopes have the same width? For example, `[3,4]` and `[3,5]`. They can't nest inside each other since their widths are equal, but if we only consider heights after sorting, we might incorrectly count both in our sequence.

The clever trick is to sort envelopes with the same width in **descending** order by height. Why? This prevents us from selecting multiple envelopes with the same width in our LIS. Since we're looking for an increasing sequence of heights, and envelopes with the same width are arranged in decreasing height order, we can only pick at most one envelope from each width group.

After this special sorting `(x[0], -x[1])`, the problem reduces to finding LIS on the heights alone. The binary search optimization using `bisect_left` maintains a list `d` where `d[i]` represents the smallest ending height of all increasing subsequences of length `i+1`, allowing us to efficiently build the longest sequence.

## Solution Approach

The implementation follows these steps:

**Step 1: Sort the envelopes**
```python
envelopes.sort(key=lambda x: (x[0], -x[1]))
```
This sorts envelopes by width in ascending order. When widths are equal, it sorts by height in descending order (note the negative sign). This ensures envelopes with the same width won't both be selected in our subsequence.

**Step 2: Initialize the LIS tracker**
```python
d = [envelopes[0][1]]
```
We create a list `d` that will maintain the smallest ending heights for subsequences of different lengths. Initially, it contains just the height of the first envelope.

**Step 3: Process remaining envelopes**
```python
for _, h in envelopes[1:]:
```
We iterate through the remaining envelopes, only caring about their heights (width is already handled by sorting).

**Step 4: Update the LIS tracker**
For each height `h`:
- If `h > d[-1]`: The current height is larger than all heights in `d`, so we can extend our longest subsequence by appending `h`.
  ```python
  d.append(h)
  ```

- Otherwise: We find the leftmost position in `d` where we can place `h` to maintain the sorted order.
  ```python
  idx = bisect_left(d, h)
  d[idx] = h
  ```
  This replacement ensures `d[idx]` stores the smallest possible height for subsequences of length `idx+1`.

**Step 5: Return the result**
```python
return len(d)
```
The length of `d` represents the maximum number of envelopes that can be nested.

The algorithm maintains the invariant that `d[i]` is the minimum ending height among all increasing subsequences of length `i+1`. By using binary search (`bisect_left`), we achieve `O(n log n)` time complexity instead of the `O(n²)` dynamic programming approach.

## Example Walkthrough

Let's walk through the solution with envelopes `[[5,4], [6,4], [6,7], [2,3]]`.

**Step 1: Sort the envelopes**
- Original: `[[5,4], [6,4], [6,7], [2,3]]`
- After sorting by `(width ascending, height descending for same width)`:
  - `[2,3]` (width=2)
  - `[5,4]` (width=5)
  - `[6,7]` (width=6, height=7)
  - `[6,4]` (width=6, height=4)

  Note: The two envelopes with width=6 are sorted by height in descending order (7 before 4).

Sorted array: `[[2,3], [5,4], [6,7], [6,4]]`

**Step 2: Extract heights and find LIS**
Heights sequence: `[3, 4, 7, 4]`

Now we find the longest increasing subsequence on these heights:

- Initialize `d = [3]` (first height)

- Process height 4:
  - 4 > 3 (last element in d)
  - Append 4: `d = [3, 4]`

- Process height 7:
  - 7 > 4 (last element in d)
  - Append 7: `d = [3, 4, 7]`

- Process height 4:
  - 4 < 7 (last element in d)
  - Find position using binary search: `bisect_left([3, 4, 7], 4) = 1`
  - Replace `d[1]` with 4: `d = [3, 4, 7]` (no change since d[1] was already 4)

**Result:** `len(d) = 3`

The maximum nesting sequence has 3 envelopes. One valid sequence is:
- `[2,3]` → `[5,4]` → `[6,7]`

The sorting strategy prevented us from incorrectly selecting both `[6,7]` and `[6,4]` (which have the same width and cannot nest). By placing `[6,7]` before `[6,4]` in our sorted array, the LIS algorithm naturally selected `[6,7]` as part of the increasing sequence, while `[6,4]` couldn't extend it further.

## Python Solution

```python
from typing import List
from bisect import bisect_left

class Solution:
    def maxEnvelopes(self, envelopes: List[List[int]]) -> int:
        # Sort envelopes by width ascending, and if widths are equal, by height descending
        # This ensures we can't use two envelopes with the same width in our sequence
        envelopes.sort(key=lambda envelope: (envelope[0], -envelope[1]))
      
        # Initialize LIS (Longest Increasing Subsequence) array with the first envelope's height
        # This array maintains the smallest tail value for each possible subsequence length
        lis_heights = [envelopes[0][1]]
      
        # Process remaining envelopes
        for width, height in envelopes[1:]:
            if height > lis_heights[-1]:
                # If current height is larger than the largest height in LIS,
                # we can extend the sequence
                lis_heights.append(height)
            else:
                # Find the leftmost position where we can replace an existing height
                # with the current height to maintain the LIS property
                insertion_index = bisect_left(lis_heights, height)
                lis_heights[insertion_index] = height
      
        # The length of the LIS array represents the maximum number of envelopes
        # that can be nested (Russian doll style)
        return len(lis_heights)
```

## Time and Space Complexity

**Time Complexity:** `O(n log n)`

The time complexity breaks down as follows:
- Sorting the envelopes takes `O(n log n)` where `n` is the number of envelopes
- Iterating through the sorted envelopes takes `O(n)` time
- For each envelope, we either:
  - Append to the end of array `d` in `O(1)` time, or
  - Perform binary search using `bisect_left` in `O(log k)` time where `k` is the current size of array `d`, and update the element at that index in `O(1)` time
- Since `k ≤ n`, each binary search operation is at most `O(log n)`
- Overall: `O(n log n) + O(n) * O(log n) = O(n log n)`

**Space Complexity:** `O(n)`

The space complexity analysis:
- The sorting operation may use `O(log n)` space for the recursion stack (depending on the sorting algorithm implementation)
- Array `d` stores at most `n` elements in the worst case, requiring `O(n)` space
- No other significant auxiliary space is used
- Overall: `O(n)` space complexity

## Common Pitfalls

### Pitfall 1: Incorrect Sorting Strategy for Same Width

**The Problem:**
A common mistake is sorting envelopes with the same width in ascending order of height instead of descending order. This would look like:
```python
envelopes.sort(key=lambda x: (x[0], x[1]))  # WRONG!
```

**Why This Fails:**
If you sort heights in ascending order for the same width, the LIS algorithm might incorrectly select multiple envelopes with the same width. For example, with envelopes `[[1,2], [1,3], [1,4]]`, the algorithm would think all three can be nested, but they can't since they have the same width.

**The Solution:**
Always sort by height in descending order when widths are equal:
```python
envelopes.sort(key=lambda x: (x[0], -x[1]))  # CORRECT
```
This ensures that among envelopes with the same width, only one can be selected in the increasing subsequence (since heights would be decreasing).

### Pitfall 2: Using Standard LIS Without Proper Sorting

**The Problem:**
Some might try to apply LIS directly on both dimensions or only sort by one dimension:
```python
# Only sorting by width
envelopes.sort(key=lambda x: x[0])
# Then trying to find LIS on (width, height) pairs
```

**Why This Fails:**
This doesn't handle the strict inequality requirement for both dimensions. You might end up with envelopes that have equal heights or widths in your sequence.

**The Solution:**
The two-step approach is essential:
1. First, handle the width dimension through sorting
2. Then, apply LIS only on the height dimension, with the special descending sort for equal widths

### Pitfall 3: Misunderstanding the `bisect_left` Replacement

**The Problem:**
Some might think we're building the actual sequence of envelopes in the `lis_heights` array and be confused why we're replacing values:
```python
# Misconception: lis_heights contains the actual sequence of envelopes
```

**Why This Is Wrong:**
The `lis_heights` array doesn't store the actual subsequence. Instead, `lis_heights[i]` stores the minimum possible ending height for all subsequences of length `i+1`.

**The Solution:**
Understand that this is a space-optimized approach where we only track the minimum ending values for each possible subsequence length. The actual sequence isn't preserved, but its length is, which is all we need for this problem.

