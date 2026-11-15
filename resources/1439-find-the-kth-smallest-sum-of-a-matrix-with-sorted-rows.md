# 1439. Find the Kth Smallest Sum of a Matrix With Sorted Rows

> Source: https://algo.monster/liteproblems/1439

## Problem Statement

You have a matrix `mat` with `m` rows and `n` columns where each row is sorted in non-decreasing order (smallest to largest). You need to select exactly one element from each row to form an array.

For example, if you have a 3×3 matrix, you must pick one element from row 1, one from row 2, and one from row 3. The sum of these selected elements forms your array sum.

Since there are many possible ways to select elements (one from each row), there will be many possible array sums. Your task is to find the `k`-th smallest array sum among all these possibilities.

The solution uses a clever approach by building up the answer row by row. Starting with `pre = [0]`, it processes each row of the matrix. For each row, it generates all possible sums by adding each element in the current row (limited to first `k` elements) to each sum in `pre`. It then sorts these new sums and keeps only the `k` smallest ones. This ensures that at each step, we maintain only the `k` smallest possible sums, which is sufficient to find the final answer.

After processing all rows, `pre` contains the `k` smallest possible array sums, and the last element `pre[-1]` is the `k`-th smallest sum.

## Intuition

The key insight is that we don't need to generate all possible array sums to find the `k`-th smallest. If we have `m` rows with `n` elements each, there are `n^m` possible combinations, which could be enormous.

Think about building the solution incrementally. After processing the first row, we have `n` possible sums (just the elements themselves). When we add the second row, each of these sums can combine with any element from the second row, giving us `n × n` possibilities. This grows exponentially.

However, we only care about the `k`-th smallest sum. This means that after processing each row, we only need to keep track of the `k` smallest sums so far. Why? Because larger sums from earlier rows, when combined with elements from the next row, will only produce even larger sums that definitely won't be among the `k` smallest final results.

The algorithm uses this by maintaining a list `pre` that stores at most `k` smallest sums at each step. When processing a new row, we:
1. Generate all possible new sums by adding each element in the current row to each sum in `pre`
2. Sort these new sums
3. Keep only the `k` smallest

This pruning strategy dramatically reduces the space and time complexity. Instead of tracking potentially millions of combinations, we only track `k` at each step. Since each row is already sorted, we can further optimize by only considering the first `k` elements of each row (larger elements would only create larger sums that we'd eventually discard anyway).

The beauty of this approach is that it guarantees we never lose the actual `k`-th smallest sum while avoiding unnecessary computation of larger sums that we know won't be in our final answer.

**Learn more about [Binary Search](/problems/binary_search_intro) and [Heap (Priority Queue)](/problems/heap_intro) patterns.**

## Solution Approach

The implementation uses a dynamic programming approach with pruning to efficiently find the `k`-th smallest array sum.

**Initial Setup:**
- Start with `pre = [0]` as the base case. This represents having selected zero elements with a sum of 0.

**Processing Each Row:**
The algorithm iterates through each row `cur` in the matrix:

1. **Generate All Possible Sums:**
   - For each existing sum `a` in `pre` and each element `b` in the current row (limited to first `k` elements with `cur[:k]`), compute the new sum `a + b`
   - This is done using a generator expression: `(a + b for a in pre for b in cur[:k])`

2. **Sort and Prune:**
   - Sort all the newly generated sums using `sorted()`
   - Keep only the `k` smallest sums by slicing with `[:k]`
   - Update `pre` with these `k` smallest sums

**Why limit to `cur[:k]`?**
Since each row is sorted in non-decreasing order, the first `k` elements are the smallest. Any element beyond index `k-1` would only produce larger sums that would be discarded anyway after sorting and keeping only `k` smallest.

**Final Result:**
After processing all rows, `pre` contains the `k` smallest possible array sums in sorted order. The `k`-th smallest sum is at index `k-1`, which is accessed by `pre[-1]` (the last element in the list of size `k`).

**Time Complexity:**
- For each of `m` rows, we generate at most `k × k` sums
- Sorting these `k²` sums takes `O(k² log k)`
- Overall: `O(m × k² log k)`

**Space Complexity:**
- We maintain at most `k` sums in `pre` at any time
- Space: `O(k)`

This approach elegantly balances between brute force (generating all possible sums) and efficiency (keeping only what's necessary to find the answer).

## Example Walkthrough

Let's walk through a concrete example to illustrate how the algorithm works.

**Given:**
- Matrix `mat = [[1, 3, 11], [2, 4, 6]]` (2 rows, 3 columns each)
- `k = 5` (we want the 5th smallest array sum)

**Step-by-step execution:**

**Initial State:**
- `pre = [0]` (base case: no elements selected yet)

**Processing Row 1: `[1, 3, 11]`**
- Limit row to first k elements: `[1, 3, 11][:5]` = `[1, 3, 11]` (all elements since k=5)
- Generate new sums by adding each element to each sum in `pre`:
  - 0 + 1 = 1
  - 0 + 3 = 3
  - 0 + 11 = 11
- Sort these sums: `[1, 3, 11]`
- Keep only k smallest: `[1, 3, 11][:5]` = `[1, 3, 11]`
- Update `pre = [1, 3, 11]`

**Processing Row 2: `[2, 4, 6]`**
- Limit row to first k elements: `[2, 4, 6][:5]` = `[2, 4, 6]`
- Generate new sums by combining each sum in `pre` with each element:
  - From pre[0]=1: 1+2=3, 1+4=5, 1+6=7
  - From pre[1]=3: 3+2=5, 3+4=7, 3+6=9
  - From pre[2]=11: 11+2=13, 11+4=15, 11+6=17
- All generated sums: `[3, 5, 7, 5, 7, 9, 13, 15, 17]`
- Sort these sums: `[3, 5, 5, 7, 7, 9, 13, 15, 17]`
- Keep only k=5 smallest: `[3, 5, 5, 7, 7]`
- Update `pre = [3, 5, 5, 7, 7]`

**Final Result:**
- The k smallest array sums are `[3, 5, 5, 7, 7]`
- The 5th smallest (k=5) is `pre[-1] = 7`

**Verification:**
The array sum of 7 can be formed by:
- Selecting 1 from row 1 and 6 from row 2: 1+6=7
- Selecting 3 from row 1 and 4 from row 2: 3+4=7

Both combinations give us the 5th smallest possible sum among all ways to select one element from each row.

## Python Solution

```python
class Solution:
    def kthSmallest(self, mat: List[List[int]], k: int) -> int:
        # Initialize with a single element 0, representing the sum before processing any row
        previous_sums = [0]

        # Process each row in the matrix
        for current_row in mat:
            # Generate all possible sums by adding each element from current row
            # to each sum from previous rows
            # Only consider first k elements from current row for optimization
            all_new_sums = []
            for prev_sum in previous_sums:
                for element in current_row[:k]:
                    all_new_sums.append(prev_sum + element)

            # Sort all new sums and keep only the k smallest ones
            # This ensures we maintain at most k candidates for the final answer
            previous_sums = sorted(all_new_sums)[:k]

        # The k-th smallest sum is the last element in our k smallest sums
        return previous_sums[-1]
```

## Time and Space Complexity

**Time Complexity:** `O(m * k^2 * log(k))` where `m` is the number of rows in the matrix and `k` is the parameter.

- The outer loop iterates through each row of the matrix: `m` iterations
- For each row, we generate all combinations of sums between elements in `pre` and elements in `cur[:k]`
  - `pre` contains at most `k` elements
  - `cur[:k]` contains at most `k` elements
  - This generates at most `k * k = k^2` combinations
- We sort these `k^2` combinations: `O(k^2 * log(k^2)) = O(k^2 * log(k))`
- We then keep only the first `k` elements using slicing `[:k]`
- Total: `O(m * k^2 * log(k))`

**Space Complexity:** `O(k^2)`

- The `pre` array stores at most `k` elements: `O(k)`
- During each iteration, the list comprehension `[a + b for a in pre for b in cur[:k]]` creates a temporary list of at most `k^2` elements before sorting: `O(k^2)`
- The sorted operation creates another temporary list of the same size: `O(k^2)`
- The dominant space usage is `O(k^2)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Not Limiting Row Elements to First k**

**Pitfall:** A common mistake is iterating through all elements in each row instead of limiting to the first `k` elements. This causes unnecessary computation and can lead to Time Limit Exceeded (TLE) for large matrices.

```python
# Wrong approach - iterates through entire row
for element in current_row:  # Should be current_row[:k]
    all_new_sums.append(prev_sum + element)
```

**Why it's problematic:** Since rows are sorted, elements beyond index `k-1` will only produce larger sums that get discarded anyway. Processing them wastes time, especially when `n >> k`.

**Solution:** Always slice the row with `[:k]` to consider only the first `k` elements.

### 2. **Forgetting to Sort Before Keeping k Smallest**

**Pitfall:** Some implementations might forget to sort the generated sums before taking the first `k` elements, leading to incorrect results.

```python
# Wrong - taking first k without sorting
previous_sums = all_new_sums[:k]  # Missing sorted()
```

**Solution:** Always sort before slicing:
```python
previous_sums = sorted(all_new_sums)[:k]
```

### 3. **Using a Min-Heap Without Size Control**

**Pitfall:** When optimizing with a heap, forgetting to limit the heap size to `k` can cause memory issues and slower performance.

```python
import heapq
# Wrong - heap grows unbounded
for prev_sum in previous_sums:
    for element in current_row[:k]:
        heapq.heappush(heap, prev_sum + element)
# No size limiting!
```

**Solution:** Use a max-heap of size `k` or pop elements when heap size exceeds `k`:
```python
import heapq
heap = []
for prev_sum in previous_sums:
    for element in current_row[:k]:
        heapq.heappush(heap, prev_sum + element)

# Keep only k smallest
previous_sums = heapq.nsmallest(k, heap)
```

### 4. **Incorrect Index for k-th Smallest**

**Pitfall:** Accessing the wrong index for the final answer, especially confusing 0-based vs 1-based indexing.

```python
# Wrong - returns (k-1)th smallest instead of kth
return previous_sums[k-1]  # Should be previous_sums[-1]
```

**Why `previous_sums[-1]` is correct:** After keeping exactly `k` smallest sums, the list has indices 0 to k-1, where index k-1 (or -1) contains the k-th smallest value.

### 5. **Not Handling Edge Cases**

**Pitfall:** Not considering edge cases like when `k = 1` or when the matrix has only one row.

**Solution:** The current implementation handles these naturally, but be aware:
- When `k = 1`: The algorithm correctly returns the sum of minimum elements from each row
- Single row matrix: Works correctly as the loop executes once

