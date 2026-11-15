# 1121. Divide Array Into Increasing Sequences

> Source: https://algo.monster/liteproblems/1121

## Problem Statement

You are given an integer array `nums` that is sorted in non-decreasing order and an integer `k`. Your task is to determine if you can divide this array into one or more disjoint increasing subsequences where each subsequence has a length of at least `k`.

A few key points to understand:
- The array is already sorted in non-decreasing order (elements can be equal or increasing)
- You need to partition all elements into groups
- Each group must form a strictly increasing subsequence (no equal elements allowed within a group)
- Each group must contain at least `k` elements
- The groups must be disjoint (each element belongs to exactly one group)

For example, if `nums = [1, 1, 2, 2, 3, 3]` and `k = 3`, you could potentially divide it into two subsequences like `[1, 2, 3]` and `[1, 2, 3]`, making it valid.

The solution uses a key insight: if a number appears `cnt` times in the array, those `cnt` occurrences must go into different subsequences (since subsequences must be strictly increasing). Therefore, you need at least `cnt` subsequences. Since each subsequence needs at least `k` elements, and you have `n` total elements, the condition `cnt × k ≤ n` must be satisfied. The algorithm finds the maximum frequency of any element and checks if this condition holds.

Return `true` if such a division is possible, `false` otherwise.

## Intuition

Let's think about what constraints we face when trying to divide the array into strictly increasing subsequences.

Since the array is sorted in non-decreasing order, we might have duplicate values. The critical observation is that duplicate values cannot be placed in the same subsequence because subsequences must be strictly increasing. If a value appears `cnt` times, we need at least `cnt` different subsequences to accommodate all occurrences of that value.

Consider the value that appears most frequently in the array - let's say it appears `max_count` times. This becomes our bottleneck. We absolutely need at least `max_count` subsequences because these duplicate values must be separated into different groups.

Now, given that we need at least `max_count` subsequences, and each subsequence must have at least `k` elements, we need a total of at least `max_count × k` elements in our array. If our array has `n` elements, then for a valid division to exist, we must have `max_count × k ≤ n`.

Why is this condition sufficient? Because the array is sorted, we can greedily distribute elements across our `max_count` subsequences. We can place the first occurrence of each distinct value in the first subsequence, the second occurrence in the second subsequence, and so on. Since the array is sorted, this naturally creates strictly increasing subsequences.

Therefore, the problem reduces to finding the maximum frequency of any element in the array and checking if `max_frequency × k ≤ array_length`. This elegant reduction transforms a seemingly complex partitioning problem into a simple counting problem.

## Solution Approach

The implementation follows directly from our intuition that we need to find the maximum frequency of any element and check if `max_frequency × k ≤ n`.

The solution uses Python's `groupby` function from the `itertools` module to efficiently count consecutive occurrences of each value in the sorted array. Here's how the implementation works:

1. **Group consecutive equal elements**: Since the array is already sorted, all occurrences of the same value are consecutive. The `groupby(nums)` function creates groups where each group contains all consecutive occurrences of the same value.

2. **Find maximum group size**: For each group created by `groupby`, we convert it to a list and find its length. The expression `max(len(list(x)) for _, x in groupby(nums))` iterates through all groups and finds the maximum size among them. This gives us the count of the most frequent element.

3. **Check the feasibility condition**: We multiply the maximum frequency `mx` by `k` and check if it's less than or equal to the total length of the array. If `mx * k <= len(nums)`, we can successfully divide the array into subsequences, so we return `true`. Otherwise, we return `false`.

The algorithm runs in `O(n)` time complexity where `n` is the length of the array, as we need to traverse the array once to group elements and find the maximum frequency. The space complexity is `O(1)` if we don't count the space used by the iterator, or `O(m)` where `m` is the maximum frequency of any element (for temporarily storing the group as a list).

This approach elegantly transforms the complex problem of partitioning into subsequences into a simple counting problem, using the sorted nature of the input array.

## Example Walkthrough

Let's walk through an example with `nums = [1, 2, 2, 3, 3, 3, 4, 4, 5]` and `k = 3`.

**Step 1: Identify consecutive groups**
Using `groupby`, we identify groups of consecutive equal elements:
- Group 1: `[1]` - one occurrence of 1
- Group 2: `[2, 2]` - two occurrences of 2
- Group 3: `[3, 3, 3]` - three occurrences of 3
- Group 4: `[4, 4]` - two occurrences of 4
- Group 5: `[5]` - one occurrence of 5

**Step 2: Find maximum frequency**
We find the maximum group size:
- Group sizes: 1, 2, 3, 2, 1
- Maximum frequency `mx = 3` (from the value 3 appearing three times)

**Step 3: Check feasibility condition**
- We need at least `mx = 3` subsequences (since the three 3's must go into different subsequences)
- Each subsequence needs at least `k = 3` elements
- Total elements needed: `mx × k = 3 × 3 = 9`
- Total elements available: `len(nums) = 9`
- Since `9 ≤ 9`, the condition is satisfied ✓

**Step 4: Visualize the division (for understanding)**
We can divide the array into 3 subsequences:
- Subsequence 1: `[1, 2, 3]`
- Subsequence 2: `[2, 3, 4]`
- Subsequence 3: `[3, 4, 5]`

Each subsequence is strictly increasing and has exactly 3 elements, meeting our requirements.

**Counter-example**: If `k = 4` instead:
- We'd need `mx × k = 3 × 4 = 12` elements
- But we only have 9 elements
- Since `12 > 9`, we'd return `false`

The algorithm correctly identifies that when an element appears `mx` times, we need exactly `mx` subsequences, and with each needing `k` elements, we can quickly determine feasibility by checking if `mx × k ≤ n`.

## Python Solution

```python
from typing import List
from itertools import groupby

class Solution:
    def canDivideIntoSubsequences(self, nums: List[int], k: int) -> bool:
        # Find the maximum frequency of any element in the sorted array
        # groupby groups consecutive equal elements together
        max_frequency = max(len(list(group)) for _, group in groupby(nums))

        # Check if we can form valid subsequences
        # We need at least max_frequency subsequences (one for each occurrence of the most frequent element)
        # Each subsequence must have at least k elements
        # So we need max_frequency * k <= total number of elements
        return max_frequency * k <= len(nums)
```

## Time and Space Complexity

The time complexity is `O(n)`, where `n` is the length of the array `nums`. This is because:
- The `groupby` function iterates through the array once to create groups of consecutive equal elements, which takes `O(n)` time
- For each group created by `groupby`, we convert it to a list using `list(x)` and calculate its length
- Finding the maximum among all group lengths requires iterating through all groups once
- The total number of elements across all groups is `n`, so the overall time complexity remains `O(n)`

The space complexity is `O(n)` in the worst case, not `O(1)` as stated in the reference answer. This is because:
- The `list(x)` operation inside the generator expression materializes each group into a list
- In the worst case (when all elements are the same), a single group would contain all `n` elements, requiring `O(n)` space
- Even though we process groups one at a time in the generator expression, each group still needs to be converted to a list to calculate its length

Note: If the code were modified to count group sizes without materializing lists (e.g., using `sum(1 for _ in x)`), the space complexity could be reduced to `O(1)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### Pitfall 1: Empty Array Handling
The current implementation will raise a `ValueError` when the input array is empty because `max()` is called on an empty sequence.

**Why it happens:** When `nums` is empty, `groupby(nums)` produces no groups, making the generator expression empty. Calling `max()` on an empty sequence raises an exception.

**Solution:**
```python
def canDivideIntoSubsequences(self, nums: List[int], k: int) -> bool:
    if not nums:
        return k == 0  # or return True, depending on problem requirements

    max_frequency = max(len(list(group)) for _, group in groupby(nums))
    return max_frequency * k <= len(nums)
```

### Pitfall 2: Misunderstanding the Strictly Increasing Requirement
Developers might incorrectly assume that the sorted array itself forms a valid subsequence, forgetting that strictly increasing means no duplicate values are allowed within the same subsequence.

**Wrong assumption:** If `nums = [1, 2, 2, 3]` and `k = 4`, one might think this can form a single subsequence, but `[1, 2, 2, 3]` is not strictly increasing due to the duplicate 2s.

**Correct understanding:** The two 2s must go into different subsequences, requiring at least 2 subsequences. With `k = 4`, we'd need at least 8 elements total, but we only have 4, so it's impossible.

### Pitfall 3: Manual Frequency Counting Instead of Using groupby
Some might attempt to manually count frequencies using a dictionary or Counter, which works but is less efficient for sorted arrays:

```python
# Less efficient approach for sorted arrays:
from collections import Counter
max_frequency = max(Counter(nums).values())  # Works but unnecessary
```

**Why groupby is better:** Since the array is already sorted, consecutive equal elements are grouped together. Using `groupby` avoids the overhead of hashing and provides a more elegant solution that takes advantage of the sorted property.

### Pitfall 4: Consuming the groupby Iterator Multiple Times
The `group` iterator from `groupby` can only be consumed once. If you try to use it multiple times, you'll get unexpected results:

```python
# Wrong approach:
for value, group in groupby(nums):
    print(len(list(group)))  # Consumes the iterator
    print(len(list(group)))  # Will always print 0!
```

**Solution:** Always convert to a list immediately if you need to use the group multiple times or access its length.

