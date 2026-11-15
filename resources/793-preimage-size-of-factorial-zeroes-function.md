# 793. Preimage Size of Factorial Zeroes Function

> Source: https://algo.monster/liteproblems/793

## Problem Statement

This problem asks us to find how many non-negative integers have exactly `k` trailing zeros in their factorial.

First, let's understand what trailing zeros in a factorial mean. The function `f(x)` counts the number of zeros at the end of `x!` (x factorial). For example:
- `3! = 6` has 0 trailing zeros, so `f(3) = 0`
- `11! = 39916800` has 2 trailing zeros, so `f(11) = 2`

Trailing zeros in factorials come from multiplying by 10, which is the product of 2 and 5. Since there are always more factors of 2 than 5 in factorials, the number of trailing zeros equals the number of times 5 appears as a factor in the factorial.

The key insight is that the number of trailing zeros in `n!` can be calculated by counting how many times 5 appears as a factor in all numbers from 1 to n. This is computed as: `n/5 + n/25 + n/125 + ...` (using integer division).

Given an integer `k`, we need to find how many non-negative integers `x` satisfy `f(x) = k`. For example:
- If `k = 0`, the answer is 5 because `f(0) = f(1) = f(2) = f(3) = f(4) = 0`
- If `k = 1`, the answer is 5 because `f(5) = f(6) = f(7) = f(8) = f(9) = 1`

The solution uses [binary search](/problems/binary_search_intro) to find the range of values where `f(x) = k`. The function `g(k)` finds the smallest `x` such that `f(x) >= k`. The answer is then `g(k+1) - g(k)`, which gives the count of integers with exactly `k` trailing zeros.

Note that for some values of `k`, there might be no integers `x` where `f(x) = k`, in which case the answer would be 0.

## Intuition

The first observation is that trailing zeros in factorials follow a specific pattern. Since trailing zeros come from factors of 5 (paired with factors of 2), we can calculate `f(x)` using the formula: `x/5 + x/25 + x/125 + ...`. This gives us a way to compute how many trailing zeros any factorial has.

Next, we notice that `f(x)` is a non-decreasing function - as `x` increases, `f(x)` either stays the same or increases. Moreover, `f(x)` increases by exactly 1 when `x` is divisible by 5, by 2 when divisible by 25, and so on. This means consecutive values of `x` often have the same `f(x)` value.

For example:
- `f(0) = f(1) = f(2) = f(3) = f(4) = 0`
- `f(5) = f(6) = f(7) = f(8) = f(9) = 1`
- `f(10) = f(11) = f(12) = f(13) = f(14) = 2`

The pattern shows that for most values of `k`, exactly 5 integers have `f(x) = k`. However, some values of `k` are impossible to achieve (like when `k` would require a number to be divisible by a higher power of 5 than possible).

Since `f(x)` is monotonic, we can use [binary search](/problems/binary_search_intro) to find boundaries. We can think of the search space as having a boolean property: for each position `x`, we ask "does `f(x) >= k`?" This creates a pattern like `[false, false, ..., true, true, ...]`. Our goal is to find the first `true` position - the smallest `x` where `f(x) >= k`.

The key insight is to find:
1. The smallest `x` where `f(x) >= k` (let's call this `start`)
2. The smallest `x` where `f(x) >= k+1` (let's call this `end`)

The difference `end - start` gives us the count of integers with exactly `k` trailing zeros.

The search space for [binary search](/problems/binary_search_intro) is bounded by `5*k` because if an integer has `k` trailing zeros, it must be at least around `5*k` (since we need at least `k` factors of 5). This gives us an efficient way to find the boundaries without checking every possible integer.

**Learn more about [Math](/problems/math-basics) and [Binary Search](/problems/binary_search_intro) patterns.**

## Solution Approach

The solution uses the standard binary search template to find boundaries. We need to find the first position where `f(x) >= k`, which maps perfectly to the "find first true" pattern.

**1. Function `countTrailingZeros(x)`: Calculate trailing zeros in x!**

```python
def count_trailing_zeros(x):
    if x == 0:
        return 0
    return x // 5 + count_trailing_zeros(x // 5)
```

This recursive function counts the number of trailing zeros in `x!` by counting factors of 5. It computes `x//5 + x//25 + x//125 + ...` through recursion.

**2. Function `findFirstWithAtLeastKZeros(k)`: Binary search for first x where f(x) >= k**

We use the standard binary search template with `first_true_index`:

```python
def find_first_with_k_zeros(k):
    left, right = 0, 5 * k
    first_true_index = -1

    while left <= right:
        mid = (left + right) // 2
        if count_trailing_zeros(mid) >= k:  # feasible condition
            first_true_index = mid
            right = mid - 1
        else:
            left = mid + 1

    return first_true_index if first_true_index != -1 else 5 * k + 1
```

The feasible condition is `count_trailing_zeros(mid) >= k`. When this is true, `mid` could be our answer, so we save it and search left for an even smaller value.

**3. Main calculation: Count integers with exactly k trailing zeros**

```python
return find_first_with_k_zeros(k + 1) - find_first_with_k_zeros(k)
```

The difference gives us the count of integers `x` where `f(x) = k`.

For example, if `k = 1`:
- `find_first_with_k_zeros(1) = 5` (first number with at least 1 trailing zero)
- `find_first_with_k_zeros(2) = 10` (first number with at least 2 trailing zeros)
- Answer: `10 - 5 = 5` (the numbers 5, 6, 7, 8, 9 all have exactly 1 trailing zero)

The time complexity is `O(log^2(k))` due to the [binary search](/problems/binary_search_intro) and the recursive calculation of `f(x)` within it.

## Example Walkthrough

Let's walk through finding how many non-negative integers have exactly `k = 2` trailing zeros in their factorial.

**Step 1: Understanding what we're looking for**
We need to find all integers `x` where `x!` has exactly 2 trailing zeros. Let's check some values:
- `9! = 362,880` → 1 trailing zero
- `10! = 3,628,800` → 2 trailing zeros
- `11! = 39,916,800` → 2 trailing zeros
- `14! = 87,178,291,200` → 2 trailing zeros
- `15! = 1,307,674,368,000` → 3 trailing zeros

So we expect `x = 10, 11, 12, 13, 14` to have exactly 2 trailing zeros.

**Step 2: Binary search to find first x where f(x) >= 2**

Using the template with `left = 0`, `right = 10`, `first_true_index = -1`:

| Iteration | left | right | mid | f(mid) | f(mid) >= 2? | Action |
|-----------|------|-------|-----|--------|--------------|--------|
| 1 | 0 | 10 | 5 | 1 | false | left = 6 |
| 2 | 6 | 10 | 8 | 1 | false | left = 9 |
| 3 | 9 | 10 | 9 | 1 | false | left = 10 |
| 4 | 10 | 10 | 10 | 2 | true | first_true_index = 10, right = 9 |

Loop ends (left > right). Result: `first_true_index = 10`

**Step 3: Binary search to find first x where f(x) >= 3**

Using the template with `left = 0`, `right = 15`, `first_true_index = -1`:

| Iteration | left | right | mid | f(mid) | f(mid) >= 3? | Action |
|-----------|------|-------|-----|--------|--------------|--------|
| 1 | 0 | 15 | 7 | 1 | false | left = 8 |
| 2 | 8 | 15 | 11 | 2 | false | left = 12 |
| 3 | 12 | 15 | 13 | 2 | false | left = 14 |
| 4 | 14 | 15 | 14 | 2 | false | left = 15 |
| 5 | 15 | 15 | 15 | 3 | true | first_true_index = 15, right = 14 |

Loop ends. Result: `first_true_index = 15`

**Step 4: Calculate the answer**
Count = `15 - 10 = 5`

This means exactly 5 integers (10, 11, 12, 13, 14) have factorials with exactly 2 trailing zeros.

## Python Solution

```python
class Solution:
    def preimageSizeFZF(self, k: int) -> int:
        def count_trailing_zeros(n: int) -> int:
            """Count trailing zeros in n! by counting factors of 5."""
            if n == 0:
                return 0
            return n // 5 + count_trailing_zeros(n // 5)

        def find_first_with_at_least_k_zeros(target_zeros: int) -> int:
            """
            Binary search to find smallest n where n! has >= target_zeros trailing zeros.
            Uses the standard template: find first true in [false, false, ..., true, true, ...]
            """
            if target_zeros == 0:
                return 0

            left, right = 0, 5 * target_zeros
            first_true_index = -1

            while left <= right:
                mid = (left + right) // 2
                if count_trailing_zeros(mid) >= target_zeros:
                    first_true_index = mid
                    right = mid - 1
                else:
                    left = mid + 1

            return first_true_index if first_true_index != -1 else 5 * target_zeros + 1

        # Count = (first n with >= k+1 zeros) - (first n with >= k zeros)
        return find_first_with_at_least_k_zeros(k + 1) - find_first_with_at_least_k_zeros(k)
```

## Time and Space Complexity

**Time Complexity:** `O(log^2 k)`

The time complexity is determined by `find_first_with_at_least_k_zeros`, which binary searches over the range `[0, 5k]`. The search takes `O(log(5k)) = O(log k)` iterations. In each iteration, `count_trailing_zeros(mid)` is called.

The `count_trailing_zeros` function recursively calculates the number of trailing zeros in `n!` by counting factors of 5. For a number `n`, the recursion depth is `O(log_5 n)`. Since `n` is at most `5k` here, the recursion depth is `O(log_5(5k)) = O(log k)`.

Therefore, each call to `find_first_with_at_least_k_zeros` costs `O(log k) * O(log k) = O(log^2 k)`.

Since the search runs twice (with `k+1` and `k`), the overall time complexity remains `O(log^2 k)`.

**Space Complexity:** `O(log k)`

The space complexity is dominated by the recursion stack in `count_trailing_zeros`. When it is called with a value up to `5k`, the maximum recursion depth is `O(log_5(5k)) = O(log k)`. The binary search itself uses `O(1)` additional space. Therefore, the overall space complexity is `O(log k)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Mixing Up Binary Search Templates**

**Pitfall:** Combining parts of two binary search variants, for example using `while left < right` together with `right = mid - 1`, or using `while left <= right` together with `right = mid`.

**Why it happens:** Different binary search variants exist, and each one is only correct with its own loop condition and update rules.

**Incorrect (mixed variants):**
```python
while left <= right:
    mid = (left + right) // 2
    if count_trailing_zeros(mid) >= k:
        right = mid          # never shrinks when left == right == mid
    else:
        left = mid + 1
return left
```

With `k = 1`, `left` and `right` both reach 5, `mid` stays 5, and the loop never ends.

**Correct (the template used in the solution):**
```python
first_true_index = -1
while left <= right:
    mid = (left + right) // 2
    if count_trailing_zeros(mid) >= k:
        first_true_index = mid
        right = mid - 1
    else:
        left = mid + 1
return first_true_index
```

The `while left < right` / `right = mid` / `return left` variant is also correct on its own; the bug comes only from mixing the two.

### 2. **Incorrect Search Space Upper Bound**

**Pitfall:** Using an insufficient upper bound for the binary search range, such as `k` or `2*k`.

**Why it happens:** It's tempting to think that if we want `k` trailing zeros, we only need to search up to around `k` values. However, since trailing zeros come from factors of 5, and each multiple of 5 contributes at least one factor, we need approximately `5k` numbers to get `k` trailing zeros.

**Solution:** Always use `5 * k` as the upper bound for `right`.

### 3. **Forgetting Edge Case: k = 0**

**Pitfall:** Assuming the answer for `k = 0` needs no thought, or writing a boundary search whose range is empty when the target is 0.

**Why it happens:** It's easy to overlook that `0!, 1!, 2!, 3!, 4!` all have zero trailing zeros, so the answer for `k = 0` is 5, and the first `x` with at least 0 zeros is `x = 0`.

**Solution:** The search with `target_zeros = 0` uses the range `[0, 0]`, checks `count_trailing_zeros(0) >= 0`, and returns 0, so it already handles this case. The explicit `if target_zeros == 0: return 0` in the solution is a shortcut that makes the base case obvious, not a required fix.

### 4. **Misunderstanding the Problem: Looking for Exact Match vs Range**

**Pitfall:** Trying to find a single value `x` where `f(x) = k` instead of counting all values in the range.

**Why it happens:** The problem asks for the count of integers, not just whether such an integer exists.

**Solution:** Remember that the answer is a count. The pattern is that consecutive integers often have the same number of trailing zeros in their factorials (e.g., 5!, 6!, 7!, 8!, 9! all have exactly 1 trailing zero). The solution correctly uses the difference to count all such integers.

