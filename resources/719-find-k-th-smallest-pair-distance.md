# 719. Find K-th Smallest Pair Distance

> Source: https://algo.monster/liteproblems/719

## Problem Statement

You are given an integer array `nums` and an integer `k`. The problem asks you to find the `k`-th smallest distance among all possible pairs of elements in the array.

A **distance of a pair** is defined as the absolute difference between two elements. For example, if you have elements `a` and `b`, their distance is `|a - b|`.

You need to consider all possible pairs `(nums[i], nums[j])` where `i < j`. This means you're looking at all combinations of two different elements from the array where the first element comes before the second element in terms of index position.

Among all these pairs and their corresponding distances, you need to return the `k`-th smallest distance value.

**Example walkthrough:**
- If `nums = [1, 3, 1]` and `k = 1`
- All possible pairs are: `(1,3)`, `(1,1)`, `(3,1)`
- Their distances are: `|1-3| = 2`, `|1-1| = 0`, `|3-1| = 2`
- Sorted distances: `[0, 2, 2]`
- The 1st smallest distance is `0`

## Intuition

The key insight is recognizing that this is a "find the k-th smallest value" problem where we're not looking for the k-th smallest element, but rather the k-th smallest distance. Since we need the k-th smallest among potentially millions of distances (up to `n*(n-1)/2` pairs), computing and [sorting](/problems/sorting_summary) all distances would be inefficient.

Instead of generating all distances explicitly, we can use [binary search](/problems/binary_search_intro) on the answer. The possible distance values range from `0` (when two elements are equal) to `max(nums) - min(nums)` (the maximum possible difference).

The crucial observation is: if we pick any distance value `d`, we can efficiently count how many pairs have a distance less than or equal to `d`. If this count is less than `k`, then our k-th smallest distance must be larger than `d`. If the count is at least `k`, then our answer is at most `d`.

To count pairs efficiently, we first sort the array. For each element `nums[i]`, we want to find how many elements before it have a distance of at most `d` from `nums[i]`. This means finding elements `nums[j]` where `j < i` and `nums[i] - nums[j] <= d`, which translates to `nums[j] >= nums[i] - d`. Since the array is sorted, we can use [binary search](/problems/binary_search_intro) to find the leftmost position where this condition holds, giving us the count in logarithmic time.

This transforms the problem from "find the k-th smallest distance" to "find the smallest distance `d` such that at least `k` pairs have distance <= `d`", which is perfectly suited for [binary search](/problems/binary_search_intro) on the answer space.

**Learn more about [Two Pointers](/problems/two_pointers_intro), [Binary Search](/problems/binary_search_intro) and [Sorting](/problems/sorting_summary) patterns.**

## Solution Approach

The implementation uses [binary search](/problems/binary_search_intro) on the answer space combined with a counting helper function. Here's how the solution works step by step:

**1. Sort the array:**
```python
nums.sort()
```
[Sorting](/problems/sorting_summary) is crucial as it allows us to efficiently count pairs using [binary search](/problems/binary_search_intro) later.

**2. Define the counting function:**
```python
def count_pairs_with_distance_at_most(max_distance):
    pair_count = 0
    for i, current_num in enumerate(nums):
        min_valid_value = current_num - max_distance
        left_bound_index = bisect_left(nums, min_valid_value, 0, i)
        pair_count += i - left_bound_index
    return pair_count
```

This function counts how many pairs have a distance less than or equal to `max_distance`. For each element `nums[i]` (called `current_num` in the code):
- We want to find pairs `(nums[j], nums[i])` where `j < i` and `nums[i] - nums[j] <= max_distance`
- This means `nums[j] >= nums[i] - max_distance` (which is stored as `min_valid_value`)
- `bisect_left(nums, min_valid_value, 0, i)` finds the leftmost index `left_bound_index` in the range `[0, i)` where `nums[left_bound_index] >= min_valid_value`
- The count of valid pairs with `nums[i]` as the larger element is `i - left_bound_index`

**3. [Binary search](/problems/binary_search_intro) on the answer:**
```python
left, right = 0, nums[-1] - nums[0]
first_true_index = -1
while left <= right:
    mid = (left + right) // 2
    if count_pairs_with_distance_at_most(mid) >= k:
        first_true_index = mid
        right = mid - 1
    else:
        left = mid + 1
return first_true_index
```

This loop performs [binary search](/problems/binary_search_intro) on the closed range `[0, nums[-1] - nums[0]]`:
- The search space is all possible distances from `0` up to and including the maximum possible distance
- The condition `count_pairs_with_distance_at_most(mid) >= k` is false for small distances and true from some point on
- When the condition holds, `mid` is recorded in `first_true_index` and the search continues to the left; otherwise it moves right
- The loop ends with `first_true_index` holding the smallest distance `d` such that the number of pairs with distance `<= d` is at least `k`

**Time Complexity:** `O(n log n * log(max-min))` where:
- `O(n log n)` for initial [sorting](/problems/sorting_summary)
- [Binary search](/problems/binary_search_intro) runs `O(log(max-min))` iterations
- Each iteration calls `count_pairs_with_distance_at_most()` which takes `O(n log n)` time (n iterations, each with a binary search)

**Space Complexity:** `O(n)` for Python's sort (Timsort can use up to `n/2` extra slots). Apart from sorting, only a constant number of variables are used.

The elegance of this solution lies in avoiding the explicit generation of all `n*(n-1)/2` distances, instead using [binary search](/problems/binary_search_intro) to directly find the k-th smallest value.

## Example Walkthrough

Let's walk through the solution with `nums = [1, 6, 1]` and `k = 3`.

**Step 1: Sort the array**
- Original: `[1, 6, 1]`
- Sorted: `[1, 1, 6]`

**Step 2: Determine search range**
- Minimum possible distance: `0`
- Maximum possible distance: `6 - 1 = 5`
- Search range: `[0, 5]`

**Step 3: Binary search on distances**

First, let's see what distances actually exist:
- Pairs: `(1,1)`, `(1,6)`, `(1,6)`
- Distances: `0`, `5`, `5`
- Sorted: `[0, 5, 5]`
- We want the 3rd smallest, which is `5`

Now let's trace the binary search:

**Iteration 1:** Try `mid = 2`
- Count pairs with distance ≤ 2:
  - For `nums[0]=1`: No elements before it → 0 pairs
  - For `nums[1]=1`: Check `nums[j] ≥ 1-2 = -1`. All elements before index 1 satisfy this (index 0), so 1 pair
  - For `nums[2]=6`: Check `nums[j] ≥ 6-2 = 4`. No elements before index 2 satisfy this, so 0 pairs
  - Total: 1 pair
- Since 1 < 3, we need a larger distance. Search in `[3, 5]`

**Iteration 2:** Try `mid = 4`
- Count pairs with distance ≤ 4:
  - For `nums[0]=1`: 0 pairs
  - For `nums[1]=1`: Check `nums[j] ≥ 1-4 = -3`. Element at index 0 satisfies this, so 1 pair
  - For `nums[2]=6`: Check `nums[j] ≥ 6-4 = 2`. No elements before index 2 satisfy this, so 0 pairs
  - Total: 1 pair
- Since 1 < 3, we need a larger distance. Search in `[5, 5]`

**Iteration 3:** Try `mid = 5`
- Count pairs with distance ≤ 5:
  - For `nums[0]=1`: 0 pairs
  - For `nums[1]=1`: Check `nums[j] ≥ 1-5 = -4`. Element at index 0 satisfies this, so 1 pair
  - For `nums[2]=6`: Check `nums[j] ≥ 6-5 = 1`. Elements at indices 0 and 1 both equal 1, so 2 pairs
  - Total: 3 pairs
- Since 3 ≥ 3, this distance works!

The binary search finds that `5` is the smallest distance where at least 3 pairs have distance ≤ that value. Therefore, the 3rd smallest distance is `5`.

## Python Solution

```python
class Solution:
    def smallestDistancePair(self, nums: List[int], k: int) -> int:
        def count_pairs_with_distance_at_most(max_distance):
            """
            Count the number of pairs with distance <= max_distance.
            For each element, find how many elements before it have
            distance <= max_distance from it.
            """
            pair_count = 0

            # For each element at position i
            for i, current_num in enumerate(nums):
                # Find the smallest element that satisfies:
                # current_num - element <= max_distance
                # which means: element >= current_num - max_distance
                min_valid_value = current_num - max_distance

                # Binary search for the leftmost position where
                # nums[j] >= min_valid_value in range [0, i)
                left_bound_index = bisect_left(nums, min_valid_value, 0, i)

                # All elements from left_bound_index to i-1 form valid pairs with current_num
                pair_count += i - left_bound_index

            return pair_count

        # Sort the array to enable binary search
        nums.sort()

        # Binary search on the answer space [0, max_difference]
        # Find the smallest distance where count_pairs_with_distance_at_most(distance) >= k
        # Using the template: feasible(mid) = count >= k
        left, right = 0, nums[-1] - nums[0]
        first_true_index = -1

        while left <= right:
            mid = (left + right) // 2

            if count_pairs_with_distance_at_most(mid) >= k:
                first_true_index = mid
                right = mid - 1
            else:
                left = mid + 1

        return first_true_index
```

## Time and Space Complexity

**Time Complexity:** `O(n log n × log(max_diff))`

The time complexity consists of two main parts:
- Sorting the array takes `O(n log n)` where `n` is the length of `nums`
- The binary search on the range `[0, nums[-1] - nums[0]]` takes `O(log(max_diff))` iterations, where `max_diff = nums[-1] - nums[0]` is the maximum possible distance
- For each binary search iteration, the `count_pairs_with_distance_at_most` function is called, which:
  - Iterates through all `n` elements
  - For each element, performs a `bisect_left` operation which takes `O(log n)`
  - Therefore, the counting function takes `O(n log n)` time
- Total for binary search part: `O(n log n × log(max_diff))`

Since the sorting dominates when `max_diff` is small, and the binary search dominates when `max_diff` is large, the overall complexity is `O(n log n + n log n × log(max_diff))` = `O(n log n × log(max_diff))`.

However, note that `max_diff` is bounded by the range of values in the array, not by `n`. If we consider the maximum value in the array as `M`, then the complexity can also be written as `O(n log n × log M)`.

**Space Complexity:** `O(n)`

- Python's `list.sort()` sorts in place, but Timsort can allocate up to `n/2` temporary slots while merging, which is `O(n)` extra space
- The binary search and the counting function use only a constant number of variables (`left`, `right`, `mid`, `pair_count`, `left_bound_index`)

Therefore, the space complexity is `O(n)`, coming from the sort. Excluding the sort, the extra space is `O(1)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. Off-by-one error in binary search range
**Pitfall:** When writing the search with `bisect_left(range(...), k, key=...)`, using `range(nums[-1] - nums[0])` instead of `range(nums[-1] - nums[0] + 1)` can miss the maximum possible distance as a valid answer.

**Why it happens:** The maximum distance between any two elements is exactly `nums[-1] - nums[0]`, but `range(n)` generates values from `0` to `n-1`, excluding `n`.

**Example scenario:**
```python
nums = [1, 100], k = 1
# Only one pair (1, 100) with distance 99
# If we use range(99), bisect_left searches [0, 98] and misses 99
```

**Solution:** Always use `range(nums[-1] - nums[0] + 1)` to include the maximum distance. The `while left <= right` loop in the solution avoids this by searching the closed range `[0, nums[-1] - nums[0]]`.

### 2. Incorrect counting logic for pairs
**Pitfall:** Counting pairs where `nums[i] - nums[j] == dist` instead of `nums[i] - nums[j] <= dist`.

**Why it happens:** Misunderstanding that we need cumulative count (pairs with distance at most `dist`) for binary search to work correctly, not exact count.

**Incorrect implementation:**
```python
def count(dist):
    cnt = 0
    for i in range(len(nums)):
        # Wrong: looking for exact distance
        target = nums[i] - dist
        if target in nums[:i]:
            cnt += 1
    return cnt
```

**Solution:** Use `bisect_left` to find all elements where the difference is `<= dist`:
```python
def count(dist):
    cnt = 0
    for i in range(len(nums)):
        j = bisect_left(nums, nums[i] - dist, 0, i)
        cnt += i - j  # All elements from j to i-1 satisfy the condition
    return cnt
```

### 3. Forgetting to sort the array
**Pitfall:** Applying binary search operations on an unsorted array leads to incorrect results.

**Why it happens:** The counting function relies on the sorted property to use `bisect_left` effectively.

**Solution:** Always sort the array first: `nums.sort()`

### 4. Using bisect_right instead of bisect_left in the final search
**Pitfall:** Using `bisect_right` finds the insertion point after the last occurrence, potentially returning a distance larger than necessary.

**Why it happens:** Confusion about which bisect function to use for finding the k-th smallest.

**Example:**
```python
# If distances [0, 1, 1, 2, 3] and k = 3
# count(0) = 1, count(1) = 3, count(2) = 4
# bisect_left returns 1 (correct: 3rd smallest is 1)
# bisect_right would return 2 (incorrect)
```

**Solution:** Use `bisect_left` to find the smallest distance where the count is at least `k`.

