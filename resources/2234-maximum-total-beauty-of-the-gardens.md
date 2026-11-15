# 2234. Maximum Total Beauty of the Gardens

> Source: https://algo.monster/liteproblems/2234

## Problem Statement

Alice manages `n` gardens and wants to maximize their total beauty by planting additional flowers.

You're given:
- An array `flowers` where `flowers[i]` represents the number of flowers already planted in garden `i` (these cannot be removed)
- `newFlowers`: the maximum number of additional flowers Alice can plant
- `target`: the minimum number of flowers needed for a garden to be considered "complete"
- `full`: the beauty value contributed by each complete garden
- `partial`: the beauty value multiplier for the minimum flower count among incomplete gardens

A garden is **complete** if it has at least `target` flowers. The total beauty is calculated as:
- Number of complete gardens × `full`
- Plus: Minimum flower count among all incomplete gardens × `partial` (or 0 if all gardens are complete)

The goal is to distribute up to `newFlowers` additional flowers among the gardens to maximize the total beauty.

For example, if you have 3 gardens with `[1, 2, 4]` flowers, `target = 4`, `full = 5`, `partial = 2`, and `newFlowers = 3`:
- Garden 2 already has 4 flowers (complete)
- You could add 2 flowers to garden 1 to make it complete (having 4 flowers)
- This would give: 2 complete gardens × 5 + minimum of remaining incomplete gardens (2) × 2 = 10 + 4 = 14 total beauty
- Alternatively, you could distribute flowers to increase the minimum among incomplete gardens for a different beauty value

The challenge is finding the optimal balance between creating more complete gardens versus increasing the minimum flower count in incomplete gardens to maximize the total beauty score.

## Intuition

The key insight is recognizing that we face a trade-off: we can either use our additional flowers to create more complete gardens (earning `full` points each) or increase the minimum flower count among incomplete gardens (earning `partial` points per flower in the minimum garden).

First, we should sort the gardens by their current flower count. This helps us identify which gardens are easiest to complete (those closest to `target`) and which incomplete gardens we can efficiently boost together.

Why enumerate the number of complete gardens? Since we want to maximize beauty, we need to try different configurations. If we decide to have `x` complete gardens, we should complete the `x` gardens that already have the most flowers - this minimizes the cost and leaves more flowers for boosting incomplete gardens.

For each configuration with `x` complete gardens:
1. We complete the rightmost `x` gardens in our sorted array (they need the fewest additional flowers)
2. With the remaining flowers, we want to maximize the minimum among the incomplete gardens

The second part is clever: to maximize the minimum efficiently, we should bring all gardens up to the same level together. If we have gardens with `[1, 2, 3]` flowers and want to increase the minimum, we should first bring them all to 3, then all to 4, and so on. This is why we use [binary search](/problems/binary_search_intro) - to find the highest level we can afford to bring a group of gardens to.

The [binary search](/problems/binary_search_intro) finds the largest prefix of incomplete gardens we can afford to level up to the same height. The cost to bring gardens `0` to `mid` up to height `flowers[mid]` is `flowers[mid] * (mid + 1) - sum(flowers[0:mid+1])`. If we have extra flowers after this leveling, we can increase all these gardens further by dividing the remaining flowers equally among them.

By trying all possible values of `x` (number of complete gardens) and finding the optimal minimum for incomplete gardens in each case, we guarantee finding the maximum total beauty.

**Learn more about [Greedy](/problems/greedy_intro), [Two Pointers](/problems/two_pointers_intro), [Binary Search](/problems/binary_search_intro), [Prefix Sum](/problems/subarray_sum) and [Sorting](/problems/sorting_summary) patterns.**

## Solution Approach

First, we sort the `flowers` array and create a [prefix sum](/problems/subarray_sum) array for efficient range sums.

```python
flowers.sort()
n = len(flowers)
prefix_sum = list(accumulate(flowers, initial=0))
```

For each configuration with `x` complete gardens, we use the [binary search](/problems/binary_search_intro) template to find the maximum garden index we can afford to level up.

**Inner Binary Search Using first_true_index Template:**

We want to find the **maximum index** where we can afford to raise gardens `[0...idx]` to `flowers[idx]` level. This is a "last true" pattern, so we invert:

```
too_expensive(idx) = cost_to_level(idx) > newFlowers
```

Pattern: `false, false, ..., false, true, true`
- For small indices: cost is affordable → `too_expensive = false`
- For large indices: cost exceeds budget → `too_expensive = true`

The maximum affordable index = `first_true_index - 1`.

**Binary Search Template:**

```python
left, right = 0, n - num_complete  # Extended right by 1 for edge case
first_true_index = 0  # Default: even index 0 is too expensive

while left <= right:
    mid = (left + right) // 2
    cost = flowers[mid] * (mid + 1) - prefix_sum[mid + 1]
    if cost > newFlowers:  # Too expensive
        first_true_index = mid
        right = mid - 1
    else:
        left = mid + 1

max_affordable_idx = first_true_index - 1
```

If `max_affordable_idx >= 0`, we can level gardens `[0...max_affordable_idx]` and distribute remaining flowers evenly.

**Time Complexity:** `O(n log n)` - outer loop `O(n)` × inner binary search `O(log n)`.

**Space Complexity:** `O(n)` for the [prefix sum](/problems/subarray_sum) array.

## Example Walkthrough

Let's walk through a concrete example to illustrate the solution approach.

**Input:**
- `flowers = [2, 4, 5, 3]`
- `newFlowers = 10`
- `target = 6`
- `full = 20`
- `partial = 5`

**Step 1: Sort and Setup**
After sorting: `flowers = [2, 3, 4, 5]`
Prefix sum array: `s = [0, 2, 5, 9, 14]`

**Step 2: Find Initial Complete Gardens**
Using binary search, we find that garden at index 3 (with 5 flowers) is the only one close to being complete. So initially, `i = 1` (we can try having 1 to 4 complete gardens).

**Step 3: Enumerate Complete Garden Configurations**

**Configuration 1: x = 1 complete garden (completing the rightmost garden)**
- Garden at index 3 needs `6 - 5 = 1` flower to complete
- Remaining flowers: `10 - 1 = 9`
- Incomplete gardens: `[2, 3, 4]` at indices 0, 1, 2
- Binary search to maximize minimum:
  - Can we bring all 3 gardens to level 4? Cost = `4 × 3 - (2+3+4) = 12 - 9 = 3` ✓
  - After bringing all to level 4, we have `9 - 3 = 6` flowers left
  - We can add `6 // 3 = 2` flowers to each, making them all have 6 flowers
  - But wait! 6 would make them complete, so cap at `target - 1 = 5`
- Minimum among incomplete: 5
- Beauty: `1 × 20 + 5 × 5 = 45`

**Configuration 2: x = 2 complete gardens**
- Gardens at indices 2 and 3 need `(6-4) + (6-5) = 3` flowers total
- Remaining flowers: `10 - 3 = 7`
- Incomplete gardens: `[2, 3]` at indices 0, 1
- Binary search to maximize minimum:
  - Can we bring both to level 3? Cost = `3 × 2 - (2+3) = 6 - 5 = 1` ✓
  - After bringing both to level 3, we have `7 - 1 = 6` flowers left
  - We can add `6 // 2 = 3` flowers to each, making them both have 6 flowers
  - Cap at `target - 1 = 5`
- Minimum among incomplete: 5
- Beauty: `2 × 20 + 5 × 5 = 65`

**Configuration 3: x = 3 complete gardens**
- Gardens at indices 1, 2, 3 need `(6-3) + (6-4) + (6-5) = 6` flowers total
- Remaining flowers: `10 - 6 = 4`
- Incomplete gardens: `[2]` at index 0
- We can add all 4 flowers to this garden: `2 + 4 = 6`
- But 6 would make it complete, so cap at 5
- Minimum among incomplete: 5
- Beauty: `3 × 20 + 5 × 5 = 85`

**Configuration 4: x = 4 complete gardens**
- All gardens need to be complete
- Cost: `(6-2) + (6-3) + (6-4) + (6-5) = 4 + 3 + 2 + 1 = 10` flowers
- Remaining flowers: `10 - 10 = 0` ✓
- No incomplete gardens, so partial beauty = 0
- Beauty: `4 × 20 + 0 × 5 = 80`

**Result:** The maximum beauty is **85**, achieved by completing 3 gardens and keeping 1 incomplete garden at 5 flowers.

## Python Solution

```python
class Solution:
    def maximumBeauty(
        self, flowers: List[int], newFlowers: int, target: int, full: int, partial: int
    ) -> int:
        flowers.sort()
        n = len(flowers)
        prefix_sum = list(accumulate(flowers, initial=0))

        max_beauty = 0
        gardens_already_full = n - bisect_left(flowers, target)

        for num_complete in range(gardens_already_full, n + 1):
            if num_complete > 0:
                flowers_needed = max(target - flowers[n - num_complete], 0)
                newFlowers -= flowers_needed

            if newFlowers < 0:
                break

            # Binary search using first_true_index template (inverted condition)
            # Find first index where cost > newFlowers (too expensive)
            incomplete_count = n - num_complete
            left, right = 0, incomplete_count  # Extended right for edge case
            first_true_index = incomplete_count  # Default: no index is too expensive

            while left <= right:
                mid = (left + right) // 2
                if mid >= incomplete_count:
                    # Out of bounds, this is too expensive
                    first_true_index = mid
                    right = mid - 1
                else:
                    cost = flowers[mid] * (mid + 1) - prefix_sum[mid + 1]
                    if cost > newFlowers:  # Too expensive
                        first_true_index = mid
                        right = mid - 1
                    else:
                        left = mid + 1

            # Maximum affordable index = first_true_index - 1
            max_affordable_idx = first_true_index - 1

            partial_level = 0
            if max_affordable_idx >= 0:
                base_cost = flowers[max_affordable_idx] * (max_affordable_idx + 1) - prefix_sum[max_affordable_idx + 1]
                remaining = newFlowers - base_cost
                partial_level = min(flowers[max_affordable_idx] + remaining // (max_affordable_idx + 1), target - 1)

            beauty_score = num_complete * full + partial_level * partial
            max_beauty = max(max_beauty, beauty_score)

        return max_beauty
```

## Time and Space Complexity

**Time Complexity:** `O(n × log n)`

The time complexity is dominated by:
1. Initial sorting of the flowers array: `O(n log n)`
2. Creating the prefix sum array using accumulate: `O(n)`
3. Binary search to find initial value of `i`: `O(log n)`
4. The main loop runs at most `n + 1` iterations (from `i` to `n`)
5. Inside each iteration:
   - Calculating `max(target - flowers[n - x], 0)`: `O(1)`
   - Binary search loop to find the optimal position: `O(log n)`
   - Other operations are `O(1)`

Therefore, the overall time complexity is `O(n log n) + O(n) × O(log n) = O(n log n)`.

**Space Complexity:** `O(n)`

The space complexity comes from:
1. The prefix sum array `s` created by `accumulate`: `O(n)`
2. Other variables (`ans`, `i`, `l`, `r`, `mid`, `x`, `y`, `cost`) use constant space: `O(1)`

Therefore, the overall space complexity is `O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Using Wrong Binary Search Template for Inner Search**

The inner binary search finds the **maximum affordable index** (a "last true" pattern). Using the standard template directly fails.

**Wrong approach:**
```python
while left < right:
    mid = (left + right + 1) >> 1
    if cost <= newFlowers:
        left = mid
    else:
        right = mid - 1
```

**Correct approach (inverted to first_true_index):**
```python
first_true_index = incompleteCount  # Default
while left <= right:
    mid = (left + right) // 2
    if cost > newFlowers:  # Too expensive
        first_true_index = mid
        right = mid - 1
    else:
        left = mid + 1
max_affordable_idx = first_true_index - 1
```

### 2. **Not Handling Empty Incomplete Gardens**

When all gardens are complete (`num_complete == n`), the binary search has no valid range.

**Solution:** The template handles this naturally - when `incompleteCount = 0`, `first_true_index = 0`, so `max_affordable_idx = -1`, giving `partial_level = 0`.

### 3. **Integer Overflow in Cost Calculation**

The cost `flowers[mid] * (mid + 1)` can overflow in languages with fixed integer sizes.

**Solution:** Use appropriate data types:
```java
long cost = (long) flowers[mid] * (mid + 1) - prefixSum[mid + 1];
```

### 4. **Forgetting to Restore newFlowers**

The Java solution modifies `newFlowers` in-place and must restore it at the end of each iteration for the next configuration to work correctly.

