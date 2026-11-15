# 2499. Minimum Total Cost to Make Arrays Unequal

> Source: https://algo.monster/liteproblems/2499

## Problem Statement

You have two arrays `nums1` and `nums2` of equal length `n`, both 0-indexed. Your goal is to make sure that for every index `i`, the values at that position are different between the two arrays (i.e., `nums1[i] != nums2[i]`).

To achieve this, you can perform swap operations on `nums1`. In each operation, you can swap the values at any two indices in `nums1`. The cost of swapping values at indices `i` and `j` is `i + j`.

Your task is to find the minimum total cost needed to perform swaps so that `nums1[i] != nums2[i]` for all indices from `0` to `n-1`. If it's impossible to achieve this condition, return `-1`.

**Key Points:**
- You can only modify `nums1` through swaps
- `nums2` remains unchanged
- Each swap operation between indices `i` and `j` costs `i + j`
- You need all corresponding positions to have different values
- You can perform any number of swaps

**Example scenario:** If `nums1 = [1, 2, 3]` and `nums2 = [1, 4, 3]`, positions 0 and 2 have matching values. You'd need to swap elements in `nums1` to ensure these positions have different values from `nums2`.

## Intuition

When we look at positions where `nums1[i] == nums2[i]`, these are the problematic positions that need fixing. We must swap values at these positions with values from other positions to create differences.

The key insight is that we want to minimize the cost, which is the sum of indices involved in swaps. Since we're summing indices, we should prefer using smaller indices whenever possible.

First, let's identify all positions where `nums1[i] == nums2[i]`. These positions MUST participate in at least one swap. Since we want to minimize cost, the optimal strategy is to pair these problematic positions with each other when possible - this way, each position only needs to be involved in one swap.

However, there's a critical constraint: when we swap two positions that both have the same value as their corresponding `nums2` position, we need to ensure the swap actually fixes both positions. For example, if position `i` has `nums1[i] = nums2[i] = 3` and position `j` has `nums1[j] = nums2[j] = 5`, swapping them gives us `nums1[i] = 5 != nums2[i] = 3` and `nums1[j] = 3 != nums2[j] = 5`, which fixes both positions perfectly.

But what if we have too many positions with the same problematic value? If more than half of the problematic positions have the same value (let's call it the "dominant" value), we can't fix all of them by swapping among themselves. For instance, if we have three positions all with `nums1[i] = nums2[i] = 3`, we can't fix all three by swapping among themselves.

In this case, we need to involve "clean" positions (where `nums1[i] != nums2[i]`) in our swaps. We must carefully choose clean positions that won't create new problems - specifically, positions where neither `nums1[i]` nor `nums2[i]` equals the dominant problematic value.

The algorithm counts how many extra swaps with clean positions are needed (this is `m` in the code), then greedily selects the clean positions with smallest indices to minimize cost. If we can't find enough suitable clean positions, the problem is unsolvable and we return `-1`.

**Learn more about [Greedy](/problems/greedy_intro) patterns.**

## Solution Approach

The implementation follows a two-phase approach to solve this problem:

**Phase 1: Identify and count problematic positions**

We iterate through both arrays simultaneously and identify positions where `nums1[i] == nums2[i]`. For each such position:
- Increment the `same` counter to track total problematic positions
- Add index `i` to `ans` (since this position must participate in a swap)
- Use a `Counter` to track the frequency of each problematic value

```python
for i, (a, b) in enumerate(zip(nums1, nums2)):
    if a == b:
        same += 1
        ans += i
        cnt[a] += 1
```

**Phase 2: Check for dominant value problem**

We check if any value appears in more than half of the problematic positions. If a value `v` has frequency greater than `same/2`, it's impossible to fix all occurrences by swapping among problematic positions alone:

```python
m = lead = 0
for k, v in cnt.items():
    if v * 2 > same:
        m = v * 2 - same  # number of extra swaps needed
        lead = k          # the dominant value
        break
```

The formula `m = v * 2 - same` calculates how many additional clean positions we need. For example, if we have 5 problematic positions and 3 of them have the same value, we need `3 * 2 - 5 = 1` extra clean position.

**Phase 3: Find suitable clean positions**

If `m > 0`, we need to involve clean positions in our swaps. We iterate through the arrays again, looking for positions where:
- `nums1[i] != nums2[i]` (it's a clean position)
- `nums1[i] != lead` (swapping won't create a new problem at the problematic position)
- `nums2[i] != lead` (swapping won't create a new problem at this clean position)

```python
for i, (a, b) in enumerate(zip(nums1, nums2)):
    if m and a != b and a != lead and b != lead:
        ans += i
        m -= 1
```

We greedily select positions from smallest to largest indices to minimize cost, decrementing `m` each time we find a suitable position.

**Final Check**

If `m > 0` after checking all positions, we couldn't find enough suitable clean positions, making the problem unsolvable. Otherwise, return the total cost:

```python
return -1 if m else ans
```

The algorithm runs in `O(n)` time with two passes through the arrays and uses `O(n)` space for the counter in the worst case.

## Example Walkthrough

Let's walk through a concrete example to illustrate the solution approach.

**Example:** `nums1 = [1, 2, 3, 3, 2]` and `nums2 = [3, 2, 3, 1, 4]`

**Phase 1: Identify problematic positions**

Let's check each index:
- Index 0: `nums1[0] = 1`, `nums2[0] = 3` → Different ✓
- Index 1: `nums1[1] = 2`, `nums2[1] = 2` → Same ✗ (problematic)
- Index 2: `nums1[2] = 3`, `nums2[2] = 3` → Same ✗ (problematic)
- Index 3: `nums1[3] = 3`, `nums2[3] = 1` → Different ✓
- Index 4: `nums1[4] = 2`, `nums2[4] = 4` → Different ✓

Problematic positions: indices 1 and 2
- `same = 2` (total problematic positions)
- `ans = 1 + 2 = 3` (sum of problematic indices)
- Counter: `{2: 1, 3: 1}` (value 2 appears once, value 3 appears once)

**Phase 2: Check for dominant value**

We check if any value appears more than `same/2 = 1` times:
- Value 2: appears 1 time → `1 * 2 = 2` is not greater than `same = 2`
- Value 3: appears 1 time → `1 * 2 = 2` is not greater than `same = 2`

No dominant value exists, so `m = 0`. This means we can fix all problematic positions by swapping them with each other.

**Phase 3: Find clean positions**

Since `m = 0`, we don't need any additional clean positions.

**Solution:**

We can swap indices 1 and 2:
- Before swap: `nums1 = [1, 2, 3, 3, 2]`
- After swap: `nums1 = [1, 3, 2, 3, 2]`
- Cost: `1 + 2 = 3`

Verification after swap:
- Index 0: `1 ≠ 3` ✓
- Index 1: `3 ≠ 2` ✓ (fixed!)
- Index 2: `2 ≠ 3` ✓ (fixed!)
- Index 3: `3 ≠ 1` ✓
- Index 4: `2 ≠ 4` ✓

Total cost: `3`

---

**Example with dominant value:** `nums1 = [5, 5, 5, 1]` and `nums2 = [5, 5, 5, 2]`

**Phase 1:** Problematic positions are 0, 1, 2 (all have value 5)
- `same = 3`, `ans = 0 + 1 + 2 = 3`
- Counter: `{5: 3}`

**Phase 2:** Value 5 appears 3 times
- `3 * 2 = 6 > 3` → We have a dominant value!
- `m = 6 - 3 = 3` extra swaps needed
- `lead = 5`

**Phase 3:** Check clean positions:
- Index 3: `nums1[3] = 1 ≠ nums2[3] = 2` (clean)
  - `1 ≠ 5` and `2 ≠ 5` → Suitable! Add to answer
  - `ans = 3 + 3 = 6`, `m = 2`

We still need 2 more suitable positions but only have 1 position total left. Since `m > 0` after checking all positions, return `-1` (impossible).

## Python Solution

```python
class Solution:
    def minimumTotalCost(self, nums1: List[int], nums2: List[int]) -> int:
        # Initialize total cost and count of same elements at same index
        total_cost = 0
        same_count = 0
      
        # Counter to track frequency of values that are same at each index
        frequency_counter = Counter()
      
        # First pass: identify positions where nums1[i] == nums2[i]
        for index, (value1, value2) in enumerate(zip(nums1, nums2)):
            if value1 == value2:
                same_count += 1
                total_cost += index  # Add index to cost (these positions must be swapped)
                frequency_counter[value1] += 1
      
        # Find if any value appears more than half of the same positions
        # This would make it impossible to eliminate all duplicates
        excess_count = 0
        dominant_value = 0
      
        for value, count in frequency_counter.items():
            if count * 2 > same_count:
                # This value appears too frequently among the same positions
                excess_count = count * 2 - same_count
                dominant_value = value
                break
      
        # Second pass: use additional positions to handle excess dominant values
        for index, (value1, value2) in enumerate(zip(nums1, nums2)):
            if excess_count > 0 and value1 != value2 and value1 != dominant_value and value2 != dominant_value:
                # This position can be used to help distribute the dominant value
                total_cost += index
                excess_count -= 1
      
        # If we still have excess, it's impossible to solve
        return -1 if excess_count > 0 else total_cost
```

## Time and Space Complexity

**Time Complexity:** `O(n)`

The algorithm consists of several sequential operations:
- First loop iterates through both arrays once using `enumerate(zip(nums1, nums2))`: `O(n)`
- Finding the maximum count value by iterating through the Counter dictionary: `O(k)` where `k` is the number of unique values in positions where `nums1[i] == nums2[i]`, and `k ≤ n`
- Second loop iterates through both arrays again: `O(n)`

Since all operations are sequential and not nested, the overall time complexity is `O(n) + O(k) + O(n) = O(n)`.

**Space Complexity:** `O(k)`

The algorithm uses:
- A Counter dictionary `cnt` that stores at most `k` unique values where `nums1[i] == nums2[i]`
- Several scalar variables (`ans`, `same`, `m`, `lead`): `O(1)`

In the worst case, `k` could be `O(n)` if all positions where `nums1[i] == nums2[i]` have different values. However, typically `k` is much smaller than `n`. The space complexity is `O(k)` where `k` is the number of distinct values at positions where the two arrays have equal elements.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. Misunderstanding the Swap Mechanics

**Pitfall:** A common misconception is thinking that when we swap indices `i` and `j` in `nums1`, we're directly fixing both positions. In reality, a swap might fix one position while potentially breaking another.

**Example:** 
- `nums1 = [1, 2, 1]`, `nums2 = [1, 3, 4]`
- Position 0 needs fixing (both have value 1)
- If we swap positions 0 and 2 in `nums1`, we get `[1, 2, 1]` → `[1, 2, 1]`
- This doesn't help because we just moved one 1 to another position

**Solution:** Understand that the algorithm counts positions that MUST participate in swaps and calculates the minimum cost by selecting the cheapest indices. The actual swap partners are implicitly determined by the greedy selection.

### 2. Incorrectly Handling the Dominant Value Check

**Pitfall:** Forgetting why we check if `count * 2 > same_count` rather than `count > same_count / 2`. Integer division could cause precision issues.

**Example:**
- If `same_count = 5` and a value appears 3 times
- Using `count > same_count // 2` would give `3 > 2` (True)
- But we can actually handle this case: pair the 3 occurrences with the 2 other problematic positions

**Solution:** Always use `count * 2 > same_count` to avoid integer division issues and ensure correct logic.

### 3. Not Validating Both Conditions for Clean Positions

**Pitfall:** When selecting clean positions to help with excess dominant values, forgetting to check BOTH conditions: `value1 != dominant_value` AND `value2 != dominant_value`.

**Wrong approach:**
```python
# Only checking one condition
if value1 != value2 and value1 != dominant_value:
    # This could create a new problem if value2 == dominant_value
```

**Correct approach:**
```python
if value1 != value2 and value1 != dominant_value and value2 != dominant_value:
    # Safe to use this position
```

### 4. Misunderstanding the Cost Calculation

**Pitfall:** Thinking we need to track actual swap pairs and calculate `i + j` for each swap. The algorithm actually uses a mathematical property: if we need to involve `k` positions in swaps, the minimum cost is the sum of the `k` smallest indices.

**Example:** If positions [0, 2, 4] need swapping:
- We add `0 + 2 + 4 = 6` to the cost
- The actual pairings will be determined optimally (implicitly)
- We don't need to track that position 0 swaps with position 2, etc.

### 5. Edge Case: Empty Arrays or No Problems

**Pitfall:** Not handling the case where no positions have matching values.

**Solution:** The algorithm naturally handles this - if no positions match, `same_count = 0`, `total_cost = 0`, and the function returns 0, which is correct.

