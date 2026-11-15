# 3864. Minimum Cost to Partition a Binary String

> Source: https://algo.monster/liteproblems/3864

## Problem Statement

You are given a binary string `s` and two integers `encCost` and `flatCost`.

For each index `i`:

- `s[i] = '1'` means the `i`th element is **sensitive**.
- `s[i] = '0'` means the `i`th element is **not sensitive**.

The string must be partitioned into **segments**. At the start, the entire string is treated as a single segment.

For a segment of length `L` that contains `X` sensitive elements, the cost is computed as follows:

- If `X = 0` (no sensitive elements), the cost is `flatCost`.
- If `X > 0` (at least one sensitive element), the cost is `L * X * encCost`.

You are allowed to perform splits under the following rule:

- If a segment has **even length**, you may split it into **two contiguous segments of equal length**. The cost of doing this split is the **sum** of the costs of the two resulting segments.

Each resulting segment can again be split following the same rule, as long as it has even length.

Return an integer denoting the **minimum possible total cost** over all valid partitions of the string.

## Intuition

The key observation is that a segment can only be split in one specific way: an **even-length** segment can be divided into exactly **two equal halves**. This means the set of all possible segments is limited and follows a clear recursive structure — each segment either stays whole, or breaks into its left half and right half, which in turn may break again.

Because of this, for any segment we face a simple binary choice:

1. **Keep it as a single segment** and pay its direct cost. This cost depends on how many sensitive elements it contains: if there are sensitive elements (`X > 0`), the cost is `L * X * encCost`; otherwise it is `flatCost`.
2. **Split it into two equal halves** (only possible when the length is even) and pay the sum of the optimal costs of the two halves.

The best answer for a segment is simply the **minimum** of these two options. Since the cost of splitting depends on solving the same problem for smaller halves, this naturally leads to a **recursive** formulation: define a function over an interval `[l, r)` that returns the minimum cost for that portion of the string.

To compute the direct cost of any interval quickly, we need to know how many sensitive elements it contains. Instead of counting them every time, we precompute a **[prefix sum](/problems/subarray_sum)** array, so the number of sensitive elements in `[l, r)` is just `pre[r] - pre[l]` in constant time.

Putting it together: we recurse on the whole string `[0, n)`. At each interval we compute its direct cost, and if the length is even, we also try splitting at the midpoint `m = (l + r) // 2` and compare. Taking the smaller value at every step guarantees the overall **minimum possible total cost**.

**Learn more about [Divide and Conquer](/problems/divide_and_conquer_intro) and [Prefix Sum](/problems/subarray_sum) patterns.**

## Solution Approach

### Solution 1: Recursion

We define a function `dfs(l, r)` that represents the minimum cost for the interval `[l, r)` of string `s`. We use a [prefix sum](/problems/subarray_sum) array `pre` to quickly calculate the number of sensitive elements `x` in the interval `[l, r)`, which lets us compute the direct cost without splitting in constant time.

The calculation process of function `dfs(l, r)` is as follows:

1. Calculate the number of sensitive elements `x` in the interval `[l, r)` using `x = pre[r] - pre[l]`.
2. Calculate the cost without splitting: if `x > 0`, the cost is `(r - l) * x * encCost`; if `x = 0`, the cost is `flatCost`.
3. If the interval length `r - l` is even, we can try to split it into two equal segments. Let `m = (l + r) // 2`, then the cost after splitting is `dfs(l, m) + dfs(m, r)`. We take the smaller value between the non-split cost and the split cost.

The answer is `dfs(0, n)`, where `n` is the length of string `s`.

**Data structures and patterns used:**

- **[Prefix sum](/problems/subarray_sum) array `pre`**: We build `pre` so that `pre[i]` stores the number of sensitive elements (`'1'`s) in the first `i` characters. This is constructed by iterating over `s` and accumulating `pre[i] = pre[i - 1] + int(c)`. With this, the count of sensitive elements in any interval `[l, r)` is `pre[r] - pre[l]`.
- **Recursion ([divide and conquer](/problems/divide_and_conquer_intro))**: The function `dfs` explores the two choices — keep the segment whole or split it into halves — and returns the minimum. Because each even-length segment splits into exactly two equal halves, the recursion naturally mirrors the structure of valid partitions.

The time complexity is `O(n × log n)`, where `n` is the length of string `s`. This is because each level of the recursion processes `O(n)` total work across all segments, and the recursion depth is `O(log n)` since lengths are halved each time. The space complexity is `O(n)` for the [prefix sum](/problems/subarray_sum) array, plus `O(log n)` for the recursion stack.

## Example Walkthrough

Let's trace through a small example to see how the recursive solution works.

**Input:**
- `s = "1001"`
- `encCost = 2`
- `flatCost = 5`

**Step 1: Build the prefix sum array**

We accumulate the count of sensitive elements (`'1'`s):

| Index `i` | 0 | 1 | 2 | 3 | 4 |
|-----------|---|---|---|---|---|
| `pre[i]`  | 0 | 1 | 1 | 1 | 2 |

So the number of sensitive elements in any interval `[l, r)` is `pre[r] - pre[l]`. For example, the whole string `[0, 4)` has `pre[4] - pre[0] = 2` sensitive elements.

**Step 2: Start the recursion with `dfs(0, 4)`**

The whole string `"1001"` has length `4` and `x = 2` sensitive elements.

- **Cost without splitting:** since `x > 0`, cost = `(4 - 0) * 2 * 2 = 16`.
- **Length is even**, so we try splitting at `m = (0 + 4) // 2 = 2`, giving us `dfs(0, 2) + dfs(2, 4)`.

**Step 3: Evaluate the left half `dfs(0, 2)`** → segment `"10"`

- `x = pre[2] - pre[0] = 1` sensitive element.
- **Cost without splitting:** `(2 - 0) * 1 * 2 = 4`.
- **Length is even**, split at `m = 1` → `dfs(0, 1) + dfs(1, 2)`.
  - `dfs(0, 1)` → `"1"`: `x = 1`, cost = `1 * 1 * 2 = 2` (length odd, no split).
  - `dfs(1, 2)` → `"0"`: `x = 0`, cost = `flatCost = 5` (length odd, no split).
  - Split cost = `2 + 5 = 7`.
- Compare: `min(4, 7) = 4`. So `dfs(0, 2) = 4`.

**Step 4: Evaluate the right half `dfs(2, 4)`** → segment `"01"`

- `x = pre[4] - pre[2] = 1` sensitive element.
- **Cost without splitting:** `(4 - 2) * 1 * 2 = 4`.
- **Length is even**, split at `m = 3` → `dfs(2, 3) + dfs(3, 4)`.
  - `dfs(2, 3)` → `"0"`: `x = 0`, cost = `flatCost = 5`.
  - `dfs(3, 4)` → `"1"`: `x = 1`, cost = `1 * 1 * 2 = 2`.
  - Split cost = `5 + 2 = 7`.
- Compare: `min(4, 7) = 4`. So `dfs(2, 4) = 4`.

**Step 5: Combine back at `dfs(0, 4)`**

- Split cost = `dfs(0, 2) + dfs(2, 4) = 4 + 4 = 8`.
- Compare with the non-split cost: `min(16, 8) = 8`.

**Result:** The minimum possible total cost is **`8`**, achieved by splitting the string into the two halves `"10"` and `"01"`, keeping each as a whole segment.

This walkthrough shows the core idea in action: at every interval we weigh "keep whole" against "split into halves," and the prefix sum lets us compute each segment's direct cost instantly.

## Python Solution

```python
class Solution:
    def minCost(self, s: str, encCost: int, flatCost: int) -> int:
        # prefix[i] holds the sum of digits for s[0:i],
        # so the count of '1's in s[left:right] is prefix[right] - prefix[left].
        n = len(s)
        prefix = [0] * (n + 1)
        for i, char in enumerate(s, 1):
            prefix[i] = prefix[i - 1] + int(char)

        def dfs(left: int, right: int) -> int:
            # Number of '1's within the current segment s[left:right].
            ones_count = prefix[right] - prefix[left]

            # Cost of encoding the whole segment as a single block:
            # - if it contains any '1', cost is length * ones_count * encCost
            # - if it is all '0's, the segment is "flat", costing flatCost
            result = (right - left) * ones_count * encCost if ones_count else flatCost

            # Only segments of even length can be split into two equal halves.
            if (right - left) % 2 == 0:
                mid = (left + right) // 2
                # Compare keeping the segment whole vs. splitting it in two.
                result = min(result, dfs(left, mid) + dfs(mid, right))

            return result

        # Solve for the entire string.
        return dfs(0, n)
```

## Time and Space Complexity

- **Time Complexity**: `O(n)`, where `n` is the length of string `s`.

  First, building the prefix sum array `pre` takes `O(n)` time.

  For the `dfs` function, the recursion only splits the interval into two equal halves when the length `r - l` is even, and the splitting point is exactly the midpoint `m = (l + r) // 2`. This forms a recursion tree that resembles a binary tree built over the string.

  At each level of recursion, the total length covered by all intervals at that level sums to `n`. The work done at each node (computing `x`, `res`, etc.) is `O(1)`. The number of levels is at most `O(log n)`, but the key observation is that the total number of distinct intervals processed is bounded by `O(n)`.

  Specifically, since each interval can only be split into two halves at the exact midpoint, the recursion tree only branches on power-of-two aligned segments. The total number of nodes across the entire recursion is `O(n)`, because the intervals at each level partition (a subset of) the original range without overlap, and the recursion depth is `O(log n)` with each level contributing at most `O(n)` total intervals counted overall as `O(n)`. Thus the overall time complexity is `O(n)`.

- **Space Complexity**: `O(n)`, where `n` is the length of string `s`.

  The prefix sum array `pre` requires `O(n)` space. The recursion stack depth is at most `O(log n)` since the interval is halved at each recursive step. Therefore, the dominant term is the prefix array, giving an overall space complexity of `O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### Pitfall 1: Misunderstanding the Split Rule and Computing the Cost at the Wrong Granularity

The most common mistake is misreading **when** and **how** a segment can be split. The rule states that a segment can only be split if it has **even length**, and the split must produce **two contiguous halves of equal length**. Developers often make one of these errors:

- **Splitting at an arbitrary index** (e.g., trying every possible `mid` from `left+1` to `right-1`) instead of only at the exact midpoint `mid = (left + right) // 2`. This violates the "equal length" requirement and explores invalid partitions, producing a wrong (often too-low) answer.
- **Attempting to split odd-length segments.** If you forget the `(right - left) % 2 == 0` check, you'll generate unequal halves and potentially recurse forever or compute illegal states.

**Solution:** Always guard the split with an even-length check and only ever cut at the exact midpoint.

```python
if (right - left) % 2 == 0:          # only even-length segments are splittable
    mid = (left + right) // 2        # split must be at the exact midpoint
    result = min(result, dfs(left, mid) + dfs(mid, right))
```

### Pitfall 2: Confusing the Two Cost Formulas (Flat vs. Encoded)

It's easy to apply the wrong cost branch. The flat cost (`flatCost`) applies **only** when a segment has **zero** sensitive elements. As soon as `X > 0`, the cost becomes `L * X * encCost`. A frequent bug is writing something like:

```python
# WRONG: uses length * encCost regardless of ones_count, or adds flatCost too
result = (right - left) * encCost
```

or accidentally taking `min(flatCost, L * X * encCost)` even when `X > 0`, which silently allows the flat cost where it isn't permitted.

**Solution:** Branch strictly on whether `ones_count` is zero:

```python
ones_count = prefix[right] - prefix[left]
result = (right - left) * ones_count * encCost if ones_count else flatCost
```

The ternary ensures `flatCost` is used **exclusively** for all-zero segments and the encoded formula is used otherwise.

### Pitfall 3: Off-by-One Errors with the Prefix Sum and Half-Open Intervals

The recursion uses **half-open intervals** `[left, right)`, and the prefix array is 1-indexed so that `prefix[r] - prefix[l]` counts `'1'`s in `s[l:r]`. Mixing this up — for example using inclusive intervals `[left, right]`, sizing `prefix` as `n` instead of `n + 1`, or computing length as `right - left + 1` — leads to counting elements twice, index-out-of-range errors, or an incorrect segment length.

**Solution:** Keep the conventions consistent everywhere:

```python
prefix = [0] * (n + 1)               # size n + 1 for 1-indexed prefix sums
for i, char in enumerate(s, 1):      # enumerate starting at 1
    prefix[i] = prefix[i - 1] + int(char)

# length is (right - left), NOT (right - left + 1) for half-open intervals
length = right - left
ones_count = prefix[right] - prefix[left]
```

### Pitfall 4: Redundant Recomputation Without Memoization

Although the natural recursion here visits each distinct segment only once (because splits always go to the exact midpoint, the set of reachable intervals forms a balanced binary tree of size `O(n)`), developers sometimes "optimize" by trying multiple split points or rewrite the recursion in a way that revisits the same `(left, right)` repeatedly. Once you deviate from the midpoint-only split, the state space can explode and the same interval may be solved many times.

**Solution:** Stick to the midpoint-only recursion, which keeps the total number of unique calls at `O(n)`. If you ever generalize the splitting logic, add memoization keyed on `(left, right)`:

```python
from functools import lru_cache

@lru_cache(maxsize=None)
def dfs(left: int, right: int) -> int:
    ...
```

This guarantees each interval is computed at most once, preserving the `O(n × log n)` overall time complexity.

