# 3116. Kth Smallest Amount With Single Denomination Combination

> Source: https://algo.monster/liteproblems/3116

## Problem Statement

You have an integer array `coins` representing different coin denominations and an integer `k`. You have an infinite supply of coins of each denomination.

The key constraint is that you **cannot combine coins of different denominations** - you can only use multiple coins of the same denomination to form amounts.

For example, if `coins = [3, 5]`, you can form:
- Using only coins of value 3: 3, 6, 9, 12, 15, 18, ...
- Using only coins of value 5: 5, 10, 15, 20, 25, ...

The valid amounts you can make are: 3, 5, 6, 9, 10, 12, 15, 18, 20, ...

Your task is to find the `k`-th smallest amount that can be formed following this rule.

For instance, if `coins = [3, 5]` and `k = 4`, the first few smallest amounts are 3, 5, 6, 9, so the answer would be 9.

The problem asks you to return this `k`-th smallest amount as an integer.

## Intuition

The key insight is to recognize that this problem has a monotonic property: if we can form at least `k` amounts that are less than or equal to some value `x`, then we can definitely form at least `k` amounts for any value greater than `x`. This monotonicity suggests using [binary search](/problems/binary_search_intro).

Instead of directly finding the k-th smallest amount, we can reframe the problem: find the smallest value `x` such that there are exactly `k` amounts we can form that are less than or equal to `x`.

For a given value `x`, how do we count how many valid amounts are ≤ `x`?

For a single coin of value `a`, the amounts we can form are `a, 2a, 3a, ...`. The number of these amounts that are ≤ `x` is `⌊x/a⌋`.

With multiple coins, we might think to just sum up `⌊x/coins[i]⌋` for each coin, but this would overcount. For example, with coins `[3, 6]`, the amount `6` can be formed using either `2×3` or `1×6`, but we should only count it once.

This is where the inclusion-exclusion principle comes in. Some amounts can be formed by multiple coins - specifically, amounts that are multiples of the LCM (Least Common Multiple) of those coins.

For coins `[a, b]`:
- Add amounts formed by `a`: `⌊x/a⌋`
- Add amounts formed by `b`: `⌊x/b⌋`
- Subtract amounts formed by both (multiples of `lcm(a,b)`): `⌊x/lcm(a,b)⌋`

This pattern extends to any number of coins using inclusion-exclusion: add counts for single coins, subtract counts for pairs (to remove duplicates), add back counts for triples (since we over-subtracted), and so on.

Since we have at most 15 coins, we can use bit manipulation to enumerate all possible subsets (2^15 possibilities) and apply inclusion-exclusion to get the exact count of valid amounts ≤ `x`.

Once we can count amounts for any `x`, [binary search](/problems/binary_search_intro) finds the smallest `x` where the count reaches `k`.

**Learn more about [Math](/problems/math-basics), [Binary Search](/problems/binary_search_intro) and [Combinatorics](/problems/backtracking) patterns.**

## Solution Approach

The solution uses **[binary search](/problems/binary_search_intro)** combined with the **inclusion-exclusion principle** to find the k-th smallest amount.

**Binary Search Template:**
We use the standard binary search template to find the smallest value where at least `k` valid amounts exist:

```python
left, right = 1, 10**11
first_true_index = -1

while left <= right:
    mid = (left + right) // 2
    if feasible(mid):
        first_true_index = mid
        right = mid - 1
    else:
        left = mid + 1

return first_true_index
```

The `feasible(mid)` function returns `True` if there are at least `k` valid amounts ≤ `mid`.

**Counting Valid Amounts:**
For a given value `mid`, we count how many valid amounts exist using inclusion-exclusion:

1. **Enumerate all non-empty subsets** of coins using bitmask (from `1` to `2^n - 1`)

2. **Calculate LCM of each subset:**
   ```python
   lcm_value = 1
   for coin_index, coin_value in enumerate(coins):
       if subset_mask >> coin_index & 1:
           lcm_value = lcm(lcm_value, coin_value)
           if lcm_value > mid:
               break
   ```

3. **Apply inclusion-exclusion:**
   - For subsets with **odd** number of elements: **add** `mid // lcm_value`
   - For subsets with **even** number of elements: **subtract** `mid // lcm_value`

**Why Inclusion-Exclusion Works:**
- Single coins (size 1): Add counts like `⌊mid/a⌋`, `⌊mid/b⌋`
- Pairs (size 2): Subtract overlaps like `⌊mid/lcm(a,b)⌋`
- Triples (size 3): Add back over-corrections like `⌊mid/lcm(a,b,c)⌋`

**Feasible Function Pattern:**
The feasible function creates a boolean pattern over the search space:
```
false, false, false, ..., true, true, true, ...
```
We find the first `true` position, which is exactly the k-th smallest amount.

The time complexity is `O(2^n × log(max_value))` where `n` is the number of coins (at most 15).

## Example Walkthrough

Let's walk through finding the 4th smallest amount with `coins = [3, 6]` and `k = 4`.

**Step 1: Understanding what amounts we can form**
- Using coin 3: 3, 6, 9, 12, 15, ...
- Using coin 6: 6, 12, 18, 24, ...
- Combined (sorted): 3, 6, 9, 12, ... (note that 6 appears in both but we count it once)

**Step 2: Binary Search Process**

We'll search for the smallest value `x` such that there are at least 4 amounts ≤ `x`.

Let's trace through some binary search iterations:

**Check x = 5:**
- Subsets to consider: {3}, {6}, {3,6}
- Subset {3} (size=1, odd): Add ⌊5/3⌋ = 1
- Subset {6} (size=1, odd): Add ⌊5/6⌋ = 0
- Subset {3,6} (size=2, even): Subtract ⌊5/lcm(3,6)⌋ = ⌊5/6⌋ = 0
- Total count = 1 + 0 - 0 = 1
- Since 1 < 4, we need a larger x

**Check x = 10:**
- Subset {3} (size=1, odd): Add ⌊10/3⌋ = 3 (amounts: 3, 6, 9)
- Subset {6} (size=1, odd): Add ⌊10/6⌋ = 1 (amount: 6)
- Subset {3,6} (size=2, even): Subtract ⌊10/lcm(3,6)⌋ = ⌊10/6⌋ = 1 (overlap: 6)
- Total count = 3 + 1 - 1 = 3
- Since 3 < 4, we need a larger x

**Check x = 12:**
- Subset {3} (size=1, odd): Add ⌊12/3⌋ = 4 (amounts: 3, 6, 9, 12)
- Subset {6} (size=1, odd): Add ⌊12/6⌋ = 2 (amounts: 6, 12)
- Subset {3,6} (size=2, even): Subtract ⌊12/lcm(3,6)⌋ = ⌊12/6⌋ = 2 (overlaps: 6, 12)
- Total count = 4 + 2 - 2 = 4
- Since 4 ≥ 4, this satisfies our condition!

**Check x = 11:**
- Subset {3} (size=1, odd): Add ⌊11/3⌋ = 3
- Subset {6} (size=1, odd): Add ⌊11/6⌋ = 1
- Subset {3,6} (size=2, even): Subtract ⌊11/6⌋ = 1
- Total count = 3 + 1 - 1 = 3
- Since 3 < 4, x = 11 is too small

**Step 3: Binary Search Conclusion**

The binary search finds that x = 12 is the smallest value where we have at least 4 amounts ≤ x.

Therefore, the 4th smallest amount is **12**.

We can verify: The amounts in order are 3, 6, 9, 12, and indeed 12 is the 4th one.

**Key Insight from the Example:**
Notice how inclusion-exclusion correctly handled the overlap at 6 and 12:
- Both values were counted once by coin 3 and once by coin 6
- The subtraction of lcm(3,6) = 6 removed these duplicates
- This gave us the exact count of unique amounts

## Python Solution

```python
class Solution:
    def findKthSmallest(self, coins: List[int], k: int) -> int:
        def count_multiples_up_to(max_value: int) -> int:
            """
            Count how many valid amounts can be formed up to max_value.
            Uses inclusion-exclusion principle to avoid double counting.

            Args:
                max_value: The upper bound to check multiples up to

            Returns:
                Count of valid amounts <= max_value
            """
            total_count = 0

            # Iterate through all non-empty subsets of coins using bitmask
            for subset_mask in range(1, 1 << len(coins)):
                lcm_value = 1

                # Calculate LCM of all coins in current subset
                for coin_index, coin_value in enumerate(coins):
                    if subset_mask >> coin_index & 1:
                        lcm_value = lcm(lcm_value, coin_value)
                        if lcm_value > max_value:
                            break

                # Apply inclusion-exclusion principle
                subset_size = subset_mask.bit_count()
                if subset_size & 1:
                    total_count += max_value // lcm_value
                else:
                    total_count -= max_value // lcm_value

            return total_count

        def feasible(mid: int) -> bool:
            """Check if there are at least k valid amounts <= mid."""
            return count_multiples_up_to(mid) >= k

        # Binary search using the standard template
        left, right = 1, 10**11
        first_true_index = -1

        while left <= right:
            mid = (left + right) // 2
            if feasible(mid):
                first_true_index = mid
                right = mid - 1
            else:
                left = mid + 1

        return first_true_index
```

## Time and Space Complexity

**Time Complexity:** `O(n × 2^n × log(k × M))`

The time complexity consists of three main components:

1. **Binary search range:** The binary search is performed on the range `[0, 10^11)`, which takes `O(log(10^11))` iterations. Since we're looking for the k-th smallest value that can be formed by multiples of coins, and the maximum coin value is M, the actual search range is approximately `O(log(k × M))`.

2. **Check function per binary search iteration:** For each binary search iteration, the `check` function is called, which:
   - Iterates through all possible subsets of coins: `2^n` subsets (from 1 to `2^n - 1`)
   - For each subset, computes the LCM of selected coins, which takes `O(n)` time in the worst case (iterating through all coins when all bits are set)
   - The LCM calculation itself has a cost, but it's dominated by the iteration through coins

3. **Overall:** Combining these factors: `O(log(k × M))` binary search iterations × `O(2^n)` subsets × `O(n)` operations per subset = `O(n × 2^n × log(k × M))`

**Space Complexity:** `O(1)`

The space complexity is constant as the algorithm only uses:
- A few integer variables (`cnt`, `v`, `m`, `i`, `j`)
- No additional data structures that scale with input size
- The recursive calls in `bisect_left` use `O(log(k × M))` stack space, but this is typically considered negligible compared to the dominant factors

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Using Wrong Binary Search Template Variant**
A critical mistake is using the `while left < right` with `right = mid` pattern instead of the standard template:

```python
# WRONG - Non-standard template variant
while left < right:
    mid = (left + right) // 2
    if feasible(mid):
        right = mid
    else:
        left = mid + 1
return left
```

**Correct approach using the standard template:**
```python
left, right = 1, 10**11
first_true_index = -1

while left <= right:
    mid = (left + right) // 2
    if feasible(mid):
        first_true_index = mid
        right = mid - 1
    else:
        left = mid + 1

return first_true_index
```

The standard template explicitly tracks the answer with `first_true_index`, uses `while left <= right`, and always moves bounds by exactly 1 (`right = mid - 1`).

### 2. **Misunderstanding the Problem Constraint**
Many people initially think this is a standard coin change problem where you can mix different denominations. However, the constraint explicitly states you can **only use multiple coins of the same denomination**.

**Wrong interpretation:** With `coins = [3, 5]`, thinking you can make 8 (3 + 5)
**Correct interpretation:** You cannot make 8 because it requires mixing denominations

### 3. **Incorrect Inclusion-Exclusion Logic**
A common mistake is getting the inclusion-exclusion signs backwards:

```python
# WRONG - Signs are reversed!
if subset_size & 1:  # Odd size
    total_count -= max_value // lcm_value  # Should be ADD
else:  # Even size
    total_count += max_value // lcm_value  # Should be SUBTRACT
```

**Solution:** Remember that inclusion-exclusion starts with **adding** single elements (odd-sized subsets) and **subtracting** their intersections (even-sized subsets).

### 4. **LCM Overflow Not Handled Properly**
The LCM can grow very large, potentially exceeding the search bounds:

```python
# WRONG - No overflow check
for coin_index, coin_value in enumerate(coins):
    if subset_mask >> coin_index & 1:
        lcm_value = lcm(lcm_value, coin_value)
        # Missing: if lcm_value > max_value: break
```

**Solution:** Always check `if lcm_value > max_value: break` after computing each LCM step.

### 5. **Incorrect Binary Search Bounds**
Using too small an upper bound for binary search. With k potentially being up to 10^9 and coins as small as 1, the k-th smallest value could be quite large.

**Solution:** Use a sufficiently large upper bound like `10**11` to ensure the answer falls within the search range.

