# 3495. Minimum Operations to Make Array Elements Zero

> Source: https://algo.monster/liteproblems/3495

## Problem Statement

You are given a 2D array `queries`, where `queries[i]` is of the form `[l, r]`. Each `queries[i]` defines an array of integers `nums` consisting of elements ranging from `l` to `r`, both **inclusive**.

In one operation, you can:

- Select two integers `a` and `b` from the array.
- Replace them with `floor(a / 4)` and `floor(b / 4)`.

Your task is to determine the **minimum** number of operations required to reduce all elements of the array to zero for each query. Return the sum of the results for all queries.

## Intuition

Let's start by understanding how many operations a single element needs. Each operation divides a number by `4` (taking the floor). So for an element `x`, the number of times we must divide it by `4` before it reaches `0` is the smallest integer `p` such that `4^p > x`. For example, numbers in `[1, 3]` need `1` operation, numbers in `[4, 15]` need `2` operations, numbers in `[16, 63]` need `3` operations, and so on. In general, numbers in the range `[4^(p-1), 4^p - 1]` each require `p` operations.

Now here's the key twist: each operation processes **two** elements at once (we pick `a` and `b` and divide both). This means the total "work" to be done, measured as the sum of individual operations needed across all elements, can be shared two-at-a-time.

Let `s` be the total number of single-element divisions needed across all elements in `[l, r]`. Since each operation handles two divisions simultaneously, in the best case we could finish in about `s / 2` operations, that is `ceil(s / 2)` operations. However, there's a constraint: a single element still needs its full count of operations performed on it one at a time. So if one element requires `mx` operations, we cannot finish in fewer than `mx` operations no matter how we pair things. Therefore the answer for a range is `max(ceil(s / 2), mx)`.

Within the range `[l, r]`, the largest element is `r`, and since the required operation count grows with the value, `r` needs the most operations. So `mx` equals the number of operations needed just for element `r`.

To compute `s` efficiently for many queries, we define `f(x)` as the sum of operation counts for all elements in `[1, x]`. We can build this by walking through each "bucket" `[4^(p-1), 4^p - 1]`, counting how many elements fall in that bucket and multiplying by the per-element cost `p`. Then for any query, `s = f(r) - f(l - 1)`, and the per-element count for `r` is `mx = f(r) - f(r - 1)`. Summing `max(ceil(s / 2), mx)` over all queries gives the final answer.

**Learn more about [Math](/problems/math-basics) patterns.**

## Solution Approach

We use a **prefix sum** idea built on top of a closed-form helper function.

**Step 1: Define the helper function `f(x)`.**

`f(x)` returns the sum of the minimum operations needed for every element in the range `[1, x]`. We compute it by iterating over the "buckets" of values that share the same operation count:

- We maintain `p`, the starting value of the current bucket, beginning at `p = 1` (which is `4^0`).
- We maintain `i`, the per-element operation count for the current bucket, starting at `i = 1`.
- The current bucket covers values `[p, 4*p - 1]`, but we clamp the upper end to `x` so we don't count beyond `x`. The number of elements in this bucket is `cnt = min(p * 4 - 1, x) - p + 1`.
- We add `cnt * i` to the running result, since each of those `cnt` elements costs `i` operations.
- We then advance to the next bucket by multiplying `p *= 4` and incrementing `i += 1`.
- The loop continues while `p <= x`.

```python
def f(x: int) -> int:
    res = 0
    p = i = 1
    while p <= x:
        cnt = min(p * 4 - 1, x) - p + 1
        res += cnt * i
        i += 1
        p *= 4
    return res
```

**Step 2: Process each query.**

For each query `[l, r]`:

- Compute `s = f(r) - f(l - 1)`. This gives the total number of single-element divisions required across all elements in `[l, r]`.
- Compute `mx = f(r) - f(r - 1)`. This isolates the operation count of just element `r`, which is the largest element and therefore needs the most operations.
- The answer for this query is `max((s + 1) // 2, mx)`, where `(s + 1) // 2` is the integer form of `ceil(s / 2)` (since two divisions can be handled per operation), and `mx` is the lower bound forced by the single hardest element.

We accumulate these values across all queries into `ans` and return it.

```python
ans = 0
for l, r in queries:
    s = f(r) - f(l - 1)
    mx = f(r) - f(r - 1)
    ans += max((s + 1) // 2, mx)
return ans
```

**Complexity Analysis:**

- **Time complexity:** Each call to `f(x)` runs in `O(log x)` time because the bucket value `p` quadruples each iteration. With `q` queries and a maximum value `r`, the overall time is `O(q * log r)`.
- **Space complexity:** `O(1)`, as only a constant number of variables are used.

## Example Walkthrough

Let's trace through a small example with `queries = [[1, 5], [2, 4]]`.

**Setup: Understanding `f(x)`**

Recall `f(x)` sums the per-element operation counts for all values in `[1, x]`. The buckets are:

- Bucket `[1, 3]` → each element costs `1` operation
- Bucket `[4, 15]` → each element costs `2` operations
- Bucket `[16, 63]` → each element costs `3` operations

Let's precompute the values we'll need by walking the buckets.

**Computing `f(5)`:**

| Iteration | `p` | `i` | Bucket `[p, 4p-1]` clamped to `5` | `cnt = min(4p-1, 5) - p + 1` | Contribution `cnt * i` |
|-----------|-----|-----|-----------------------------------|------------------------------|------------------------|
| 1 | 1 | 1 | `[1, 3]` | `3 - 1 + 1 = 3` | `3 * 1 = 3` |
| 2 | 4 | 2 | `[4, 5]` (clamped from `[4,15]`) | `5 - 4 + 1 = 2` | `2 * 2 = 4` |

Now `p = 16 > 5`, so we stop. **`f(5) = 3 + 4 = 7`.**

This makes sense: values `1, 2, 3` cost `1` each (`3` total), and `4, 5` cost `2` each (`4` total).

**Computing the other needed values** (using the same bucket logic):

- `f(0) = 0` (empty range)
- `f(1) = 1` (just value `1`, costs `1`)
- `f(3) = 3` (values `1,2,3`, each costs `1`)
- `f(4) = 3 + 2 = 5` (values `1,2,3` cost `1` each, value `4` costs `2`)

---

**Query 1: `[1, 5]`**

The array is `nums = [1, 2, 3, 4, 5]`. Per-element costs are `[1, 1, 1, 2, 2]`.

- `s = f(5) - f(0) = 7 - 0 = 7`  → total single-element divisions needed
- `mx = f(5) - f(4) = 7 - 5 = 2`  → cost of the largest element `5`
- `answer = max(ceil(7 / 2), 2) = max(4, 2) = 4`

**Why `4`?** We have `7` divisions to perform, and each operation handles `2` of them. That requires at least `ceil(7/2) = 4` operations. The hardest single element (`5`) only needs `2`, which isn't the binding constraint here. So **`4` operations**.

---

**Query 2: `[2, 4]`**

The array is `nums = [2, 3, 4]`. Per-element costs are `[1, 1, 2]`.

- `s = f(4) - f(1) = 5 - 1 = 4`  → total single-element divisions needed
- `mx = f(4) - f(3) = 5 - 3 = 2`  → cost of the largest element `4`
- `answer = max(ceil(4 / 2), 2) = max(2, 2) = 2`

**Why `2`?** We have `4` divisions, handled `2` at a time → `2` operations. We can verify by hand: pair `(4, 2)` → `(1, 0)`, then pair `(3, 1)` → `(0, 0)`, and the `0`s stay `0`. That's exactly **`2` operations**, and all elements reach zero.

---

**Final Result**

Sum the per-query answers:

```
ans = 4 + 2 = 6
```

The walkthrough shows the two competing bounds in action: in Query 1 the **shared-work bound** `ceil(s/2)` dominates, while in Query 2 both bounds happen to tie. Taking `max` of the two guarantees we never undercount.

## Python Solution

```python
from typing import List


class Solution:
    def minOperations(self, queries: List[List[int]]) -> int:
        def total_operations(x: int) -> int:
            """
            Compute the sum of operations needed to reduce every integer
            in the range [1, x] down to 0, where each integer n requires
            floor(log4(n)) + 1 single-divisions.

            Numbers in the range [4^(level-1), 4^level - 1] each need
            'level' divisions to reach 0.
            """
            result = 0
            power = level = 1  # power = 4^(level-1), level = number of ops for this band
            while power <= x:
                # Count how many integers in [power, 4*power - 1] are <= x
                count = min(power * 4 - 1, x) - power + 1
                result += count * level
                level += 1
                power *= 4
            return result

        answer = 0
        for left, right in queries:
            # Total single-division cost for all numbers in [left, right]
            single_cost = total_operations(right) - total_operations(left - 1)
            # Maximum operations any single number needs (the largest value, 'right')
            max_cost = total_operations(right) - total_operations(right - 1)
            # Each operation processes two numbers, so we need ceil(single_cost / 2),
            # but never fewer than the worst single number's requirement.
            answer += max((single_cost + 1) // 2, max_cost)
        return answer
```

## Time and Space Complexity

**Time Complexity: `O(q × log M)`**

The code processes each query in the `queries` list, where `q` is the number of queries. For each query `[l, r]`, it performs the following operations:

- It calls the helper function `f` three times: `f(r)`, `f(l - 1)`, `f(r - 1)`.
- Inside `f(x)`, the `while` loop variable `p` starts at `1` and is multiplied by `4` each iteration (`p *= 4`), continuing while `p <= x`. This means the loop runs approximately `log₄(x)` times, which is `O(log x)`.

Since `x` is bounded by the maximum value `M` in the query range, each call to `f` takes `O(log M)` time. With three calls per query and `q` queries total, the overall time complexity is `O(q × log M)`.

**Space Complexity: `O(1)`**

The function `f` uses only a constant number of variables (`res`, `p`, `i`, `cnt`). The main loop uses the constant-space accumulator `ans` along with `s` and `mx`. No additional data structures that scale with input size are allocated, so the space complexity is `O(1)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### Pitfall 1: Using `ceil(single_cost / 2)` as the only answer and ignoring the single-element lower bound

The most tempting mistake is to reason that since **each operation processes two numbers**, the answer is simply `ceil(single_cost / 2)`. This overlooks a critical constraint.

**Why it's wrong:**

An operation must select **two distinct elements** and divides both. Consider a query where one element (the largest, `r`) needs many divisions but every other element needs very few. You cannot "pair up" the hardest element with itself — each operation on it must also waste a division on some other element that may already be at zero. Once all the smaller elements hit zero, the hardest element still must be paired with a zero each step.

**Example:**

Take the range `[1, 100]`. The element `100` needs `4` divisions (`100 → 25 → 6 → 1 → 0`), while many small elements need only `1`. Suppose `single_cost` is small enough that `ceil(single_cost / 2) < 4`. You would return a number smaller than `4`, but it's **physically impossible** to finish in fewer steps than the `4` operations forced by element `100`.

**Solution:** Always take the maximum of the two bounds:

```python
answer += max((single_cost + 1) // 2, max_cost)
```

This `max_cost = total_operations(right) - total_operations(right - 1)` isolates the operation count of the single hardest element and enforces it as a floor.

---

### Pitfall 2: Off-by-one errors in the prefix-sum subtraction

When computing the cost over `[left, right]`, it's easy to write `total_operations(right) - total_operations(left)` instead of `total_operations(left - 1)`.

**Why it's wrong:**

`total_operations(left)` already includes the cost of the element `left`, so subtracting it **excludes** `left` from the range, giving the cost of `[left + 1, right]`. Since the range is **inclusive** of `left`, you must subtract `total_operations(left - 1)`.

**Solution:**

```python
single_cost = total_operations(right) - total_operations(left - 1)
```

Note that `total_operations(0)` correctly returns `0` because the loop condition `power <= x` (i.e., `1 <= 0`) is immediately false, so the `left = 1` edge case is handled safely.

---

### Pitfall 3: Incorrect integer ceiling computation

Writing `single_cost // 2` (floor division) instead of `(single_cost + 1) // 2` undercounts when `single_cost` is odd.

**Why it's wrong:**

If `single_cost = 7`, you genuinely need `ceil(7 / 2) = 4` operations, but `7 // 2 = 3`. The leftover single division still requires a full operation (paired with a zero element).

**Solution:** Use the standard integer-ceiling idiom:

```python
(single_cost + 1) // 2   # equals ceil(single_cost / 2)
```

Avoid `math.ceil(single_cost / 2)`, which converts to floating point and can produce rounding errors for very large `single_cost` values.

---

### Pitfall 4: Miscomputing the bucket boundaries in `total_operations`

A subtle error is forgetting to **clamp** the bucket's upper bound to `x`:

```python
count = power * 4 - 1 - power + 1   # WRONG: counts beyond x
```

**Why it's wrong:**

The final (highest) bucket usually extends past `x`. Without clamping, you count values larger than `x` that don't exist in the range, inflating the result.

**Solution:** Always clamp with `min`:

```python
count = min(power * 4 - 1, x) - power + 1
```

This ensures the last partial bucket contributes only the elements that are actually `≤ x`.

