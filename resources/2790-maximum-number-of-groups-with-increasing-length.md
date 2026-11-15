# 2790. Maximum Number of Groups With Increasing Length

> Source: https://algo.monster/liteproblems/2790

## Problem Statement

You have `n` numbers labeled from `0` to `n - 1`, and each number `i` can be used at most `usageLimits[i]` times total. Your goal is to form the maximum number of groups following these rules:

1. **Each group contains distinct numbers** - no number can appear twice in the same group
2. **Groups must increase in size** - each group (after the first) must contain strictly more numbers than the previous group
3. **Usage limits must be respected** - across all groups, number `i` cannot be used more than `usageLimits[i]` times

For example, if `usageLimits = [1, 2, 3]`:
- Number `0` can be used at most 1 time total
- Number `1` can be used at most 2 times total
- Number `2` can be used at most 3 times total

Valid group formations could be:
- Group 1: `[0]` (size 1)
- Group 2: `[1, 2]` (size 2)
- Group 3: `[1, 2]` (size 3 - but we can't form this since `0` is already used up)

So the maximum number of groups would be 2.

The task is to find the maximum number of groups that can be created while satisfying all constraints.

## Intuition

To maximize the number of groups, we need to think about the optimal way to distribute numbers across groups of increasing sizes (1, 2, 3, 4, ...).

The key insight is that **we should prioritize using numbers with higher usage limits** for filling larger groups, since larger groups require more numbers. If we have `k` groups, they will have sizes 1, 2, 3, ..., k, requiring a total of `1 + 2 + 3 + ... + k = k*(k+1)/2` number slots to fill.

Consider [sorting](/problems/sorting_summary) the `usageLimits` array. Numbers with smaller usage limits should be used sparingly and strategically. The [greedy](/problems/greedy_intro) approach is to try forming groups sequentially:
- Can we form a group of size 1?
- Can we form a group of size 2?
- Can we form a group of size 3?
- And so on...

For each potential group size, we need enough numbers available to fill it. Since each number can be used multiple times (across different groups), we can think of the total "capacity" we have available.

The clever trick in the solution is to **accumulate unused capacity**. As we iterate through the sorted `usageLimits`:
- If a number has more usage capacity than needed for the current group size, we use what we need and carry forward the excess
- This excess capacity can help fill future, larger groups
- By accumulating unused limits to the next number (`usageLimits[i + 1] += usageLimits[i]`), we're essentially pooling our resources

This [greedy](/problems/greedy_intro) approach works because:
1. [Sorting](/problems/sorting_summary) ensures we consider numbers with lower limits first
2. We only create a new group when we have enough capacity
3. Excess capacity is never wasted - it's carried forward to help with larger groups
4. Each group requires exactly one more number than the previous group, so we increment `k` by 1 each time we successfully form a group

**Learn more about [Greedy](/problems/greedy_intro), [Math](/problems/math-basics), [Binary Search](/problems/binary_search_intro) and [Sorting](/problems/sorting_summary) patterns.**

## Solution Approach

The implementation follows a [greedy](/problems/greedy_intro) strategy with accumulation:

**Step 1: Sort the array**
```python
usageLimits.sort()
```
[Sorting](/problems/sorting_summary) ensures we process numbers with smaller usage limits first, allowing us to make optimal decisions about when to form new groups.

**Step 2: Initialize variables**
```python
k, n = 0, len(usageLimits)
```
- `k` tracks the number of groups formed (also represents the size of the next group to form)
- `n` is the total count of available numbers

**Step 3: Iterate through sorted limits**
```python
for i in range(n):
```
Process each number's usage limit in ascending order.

**Step 4: Check if we can form a new group**
```python
if usageLimits[i] > k:
    k += 1
    usageLimits[i] -= k
```
- If the current accumulated usage limit `usageLimits[i]` is greater than `k`, we have enough capacity to form a new group of size `k + 1`
- Increment `k` to represent that we've formed a new group
- Deduct `k` from `usageLimits[i]` since we've "used up" `k` slots from our available capacity

**Step 5: Accumulate unused capacity**
```python
if i + 1 < n:
    usageLimits[i + 1] += usageLimits[i]
```
- Transfer any remaining unused capacity from the current position to the next position
- This accumulation is crucial - it pools together all available usage limits as we progress through the array

**How the accumulation works:**
- After [sorting](/problems/sorting_summary): `[1, 2, 3, 4, 5]`
- Process index 0: Can form group of size 1? Yes (1 > 0). Now `k=1`, `usageLimits[0]=0`
- Accumulate to next: `usageLimits[1] = 2 + 0 = 2`
- Process index 1: Can form group of size 2? No (2 ≤ 2)
- Accumulate to next: `usageLimits[2] = 3 + 2 = 5`
- Process index 2: Can form group of size 2? Yes (5 > 1). Now `k=2`, `usageLimits[2]=3`
- Continue this process...

The algorithm efficiently determines the maximum number of increasing groups by greedily forming groups whenever possible and carrying forward unused capacity to help form larger groups later.

## Example Walkthrough

Let's walk through a concrete example with `usageLimits = [2, 1, 3, 1]`.

**Step 1: Sort the array**
After sorting: `usageLimits = [1, 1, 2, 3]`

**Step 2: Initialize variables**
- `k = 0` (no groups formed yet, next group would be size 1)
- `n = 4` (we have 4 numbers: 0, 1, 2, 3)

**Step 3: Process each usage limit**

**Iteration 1 (i=0):** `usageLimits[0] = 1`
- Check: Is `1 > 0`? Yes! We can form a group of size 1
- Form group 1: `[number_0]` (size 1)
- Update: `k = 1`, `usageLimits[0] = 1 - 1 = 0`
- Accumulate: `usageLimits[1] = 1 + 0 = 1`
- Current state: `usageLimits = [0, 1, 2, 3]`, `k = 1`

**Iteration 2 (i=1):** `usageLimits[1] = 1`
- Check: Is `1 > 1`? No! Cannot form a group of size 2
- No new group formed
- Accumulate: `usageLimits[2] = 2 + 1 = 3`
- Current state: `usageLimits = [0, 0, 3, 3]`, `k = 1`

**Iteration 3 (i=2):** `usageLimits[2] = 3`
- Check: Is `3 > 1`? Yes! We can form a group of size 2
- Form group 2: `[number_1, number_2]` (size 2)
- Update: `k = 2`, `usageLimits[2] = 3 - 2 = 1`
- Accumulate: `usageLimits[3] = 3 + 1 = 4`
- Current state: `usageLimits = [0, 0, 1, 4]`, `k = 2`

**Iteration 4 (i=3):** `usageLimits[3] = 4`
- Check: Is `4 > 2`? Yes! We can form a group of size 3
- Form group 3: `[number_0, number_2, number_3]` (size 3)
- Update: `k = 3`, `usageLimits[3] = 4 - 3 = 1`
- No more elements to accumulate to
- Final state: `k = 3`

**Result:** Maximum of 3 groups can be formed

The groups could be:
- Group 1 (size 1): Uses 1 number slot
- Group 2 (size 2): Uses 2 number slots
- Group 3 (size 3): Uses 3 number slots

Total slots used: 1 + 2 + 3 = 6, which fits within our total capacity of 7 (sum of original `usageLimits = [2, 1, 3, 1]`).

## Python Solution

```python
class Solution:
    def maxIncreasingGroups(self, usageLimits: List[int]) -> int:
        # Sort the usage limits in ascending order
        usageLimits.sort()

        # Initialize group count and get array length
        group_count = 0
        n = len(usageLimits)

        # Iterate through each usage limit
        for i in range(n):
            # Check if current limit can form a new group of size (group_count + 1)
            if usageLimits[i] > group_count:
                # Increment group count (next group needs group_count + 1 elements)
                group_count += 1
                # Deduct the used elements from current limit
                usageLimits[i] -= group_count

            # Transfer remaining unused elements to the next limit
            if i + 1 < n:
                usageLimits[i + 1] += usageLimits[i]

        return group_count
```

## Time and Space Complexity

**Time Complexity:** `O(n log n)` where `n` is the length of the `usageLimits` array.

- The sorting operation takes `O(n log n)` time
- The for loop iterates through all `n` elements once, performing constant time operations in each iteration, which takes `O(n)` time
- Therefore, the overall time complexity is dominated by the sorting step: `O(n log n)`

**Space Complexity:** `O(1)` or `O(n)` depending on the sorting algorithm implementation.

- The algorithm modifies the input array in-place and uses only a constant amount of extra variables (`k`, `n`, `i`)
- However, the sorting algorithm may require additional space:
  - If the sort is implemented using an in-place algorithm like heapsort, the space complexity is `O(1)`
  - If the sort uses algorithms like mergesort or the Python's Timsort (which is typically used), the space complexity is `O(n)` in the worst case
- Most commonly, Python's `sort()` has a space complexity of `O(n)`, so the overall space complexity is `O(n)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### Pitfall 1: Misunderstanding the Group Formation Requirements

**The Mistake:**
Many people initially think that each number must be used exactly once per group, leading to incorrect approaches like:
```python
# WRONG: Trying to form groups by directly matching sizes
def maxIncreasingGroups(usageLimits):
    usageLimits.sort()
    groups = 0
    group_size = 1

    for limit in usageLimits:
        if limit >= group_size:  # Wrong logic!
            groups += 1
            group_size += 1

    return groups
```

**Why it's wrong:** This approach doesn't account for the fact that forming a group of size `k` requires `k` different numbers, each contributing one element to that group. The algorithm needs to track cumulative capacity across all available numbers.

**The Fix:** Use accumulation to pool together usage limits:
```python
# Accumulate unused capacity to help form larger groups
if i + 1 < n:
    usageLimits[i + 1] += usageLimits[i]
```

### Pitfall 2: Not Sorting the Array First

**The Mistake:**
```python
# WRONG: Processing without sorting
def maxIncreasingGroups(usageLimits):
    k = 0
    n = len(usageLimits)

    for i in range(n):
        if usageLimits[i] > k:
            k += 1
            usageLimits[i] -= k
        if i + 1 < n:
            usageLimits[i + 1] += usageLimits[i]

    return k
```

**Why it's wrong:** Without sorting, you might waste high-capacity numbers early on small groups, leaving insufficient capacity for larger groups later. For example, with `[5, 1, 1]`, processing without sorting would use the 5 for early small groups instead of accumulating the smaller values first.

**The Fix:** Always sort first to process from smallest to largest capacity:
```python
usageLimits.sort()  # Critical first step
```

### Pitfall 3: Incorrect Group Size Tracking

**The Mistake:**
```python
# WRONG: Using k to track current group size instead of next group size
def maxIncreasingGroups(usageLimits):
    usageLimits.sort()
    k = 1  # Starting with 1 instead of 0
    n = len(usageLimits)

    for i in range(n):
        if usageLimits[i] >= k:  # Using >= instead of >
            usageLimits[i] -= k
            k += 1
        if i + 1 < n:
            usageLimits[i + 1] += usageLimits[i]

    return k - 1  # Adjusting at the end
```

**Why it's wrong:** The variable `k` should represent both the number of groups formed AND the size of the next group to form (since group sizes are 1, 2, 3, ...). Starting with `k=0` means the first group needs size 1, which is correct.

**The Fix:** Initialize `k = 0` and use strict inequality:
```python
k = 0  # Number of groups formed = size of next group - 1
if usageLimits[i] > k:  # Strict inequality for next group size
    k += 1
```

