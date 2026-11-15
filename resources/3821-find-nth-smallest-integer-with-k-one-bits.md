# 3821. Find Nth Smallest Integer With K One Bits

> Source: https://algo.monster/liteproblems/3821

## Problem Statement

You are given two positive integers `n` and `k`.

Your task is to find the `n`th smallest positive integer whose binary representation contains **exactly** `k` ones (set bits).

In other words, among all positive integers that have precisely `k` ones in their binary form, list them in increasing order, and return the one that appears in the `n`th position.

It is guaranteed that the answer is **strictly less** than `2^50`.

**Example walkthrough:**

Consider `k = 2`. The positive integers with exactly two ones in their binary representation, listed in increasing order, are:

- `3` → binary `11`
- `5` → binary `101`
- `6` → binary `110`
- `9` → binary `1001`
- `10` → binary `1010`
- `12` → binary `1100`
- ...

If `n = 4` and `k = 2`, then the `4`th number in this sequence is `9`.

**Key points to understand:**

- A number's binary representation must have **exactly** `k` set bits — not more, not fewer.
- The numbers are considered in **ascending order**, and you must return the value at the `n`th position.
- Since the answer is guaranteed to be less than `2^50`, you only need to consider integers that fit within `50` bits.

## Intuition

A natural first idea is to simply count upward, checking each integer's number of set bits until we reach the `n`th one. But since the answer can be as large as `2^50`, this brute-force approach is far too slow.

The key insight is to **build the answer bit by bit**, from the most significant bit down to the least significant bit, instead of searching number by number.

The crucial observation is this: when deciding the value of a particular bit, we can use **[combinatorics](/problems/backtracking)** to count how many valid numbers fall on each side of that decision — without actually listing them.

Let's think about it carefully. Suppose we are looking at bit position `i` (where positions range from `49` down to `0`), and we still need to place `k` ones among bits `0` through `i`.

- If we decide to set bit `i` to `0`, then all `k` ones must be distributed among the lower `i` bits (positions `0` to `i-1`). The number of distinct ways to choose where those `k` ones go is exactly `C(i, k)` — that is, "choose `k` positions out of `i`". Every such arrangement produces a number **smaller** than any number where bit `i` is `1`.

- If we decide to set bit `i` to `1`, then we have used one of our ones, and we still need to place `k-1` ones in the remaining lower bits.

So the numbers with bit `i` equal to `0` are precisely the `C(i, k)` **smallest** options at this step, and the numbers with bit `i` equal to `1` come **after** them.

This gives us a clean decision rule:

- If `n > C(i, k)`, the `n`th number cannot be among the first `C(i, k)` smaller options, so bit `i` **must** be `1`. We then "skip past" those `C(i, k)` smaller numbers by subtracting them: `n -= C(i, k)`, and decrease `k` by `1` since we just used one of our ones.

- Otherwise (`n <= C(i, k)`), the `n`th number lies within the group where bit `i` is `0`, so we leave bit `i` as `0` and move on to the next lower bit without changing `n` or `k`.

This is essentially a **greedy ranking process**, similar to how we find the `n`th permutation by deciding each digit one at a time. Each step locks in one bit and narrows down which "slice" of the ordering our target falls into.

We repeat this from bit `49` down to bit `0`. The moment `k` reaches `0`, all required ones have been placed, and the remaining lower bits stay `0`, so we can stop early.

To make the combination counts `C(i, k)` instant to look up, we **precompute Pascal's triangle** once before answering. This lets the entire bit-by-bit construction run in just `50` steps, making the solution extremely fast regardless of how large `n` is.

**Learn more about [Math](/problems/math-basics) and [Combinatorics](/problems/backtracking) patterns.**

## Solution Approach

### [Combinatorics](/problems/backtracking) + Greedy

We construct the answer one bit at a time, from the most significant bit (position `49`) down to the least significant bit (position `0`), using precomputed binomial coefficients to make each decision instantly.

**Step 1: Precompute Pascal's Triangle**

Before solving any query, we build a table `c` where `c[i][j]` stores the value of `C(i, j)` — the number of ways to choose `j` items out of `i`. This is done using the classic recurrence:

```
c[i][j] = c[i-1][j-1] + c[i-1][j]
```

with the base case `c[i][0] = 1` for every `i`. We fill this table for all `i` up to `mx = 50`, which covers every combination count we might need. Computing it once upfront means each lookup of `C(i, k)` during the main loop costs `O(1)`.

**Step 2: Greedily decide each bit**

We initialize `ans = 0` and loop over bit positions `i` from `49` down to `0`. At each position, we ask: *if bit `i` were `0`, how many valid numbers would be smaller?* That count is exactly `c[i][k]` — the number of ways to place all `k` remaining ones among the lower `i` bits.

- **If `n > c[i][k]`:** the `n`th number cannot fall into the group where bit `i` is `0`, so bit `i` must be `1`. We perform three updates:
  - `n -= c[i][k]` — skip past the `c[i][k]` smaller numbers we have ruled out.
  - `ans |= 1 << i` — set bit `i` of the answer to `1`.
  - `k -= 1` — we have used one of our `k` ones.

  After decrementing `k`, if `k == 0`, all ones have been placed, so we `break` out of the loop early. The remaining lower bits stay `0`, which is already their default value in `ans`.

- **Otherwise (`n <= c[i][k]`):** the `n`th number lies within the group where bit `i` is `0`. We leave bit `i` as `0`, keep `n` and `k` unchanged, and continue to the next lower bit.

**Step 3: Return the result**

Once the loop finishes (either by reaching bit `0` or by `k` dropping to `0`), `ans` holds the fully constructed number, which we return.

### Complexity Analysis

- **Time Complexity:** The precomputation of Pascal's triangle takes `O(mx^2)` once. Each call to `nthSmallest` runs a single loop over at most `50` bit positions, each doing `O(1)` work, so each query is `O(50)`, effectively constant.

- **Space Complexity:** The table `c` uses `O(mx^2)` space, which is also a one-time cost. The query itself uses only `O(1)` extra space.

### Data Structures and Patterns Used

- **Pascal's Triangle (2D DP table):** a precomputed lookup table for binomial coefficients `C(i, j)`, enabling instant combination counts.
- **Greedy bit construction:** deciding each bit from high to low, similar to finding the `n`th permutation digit by digit.
- **Combinatorial ranking:** using `C(i, k)` to count how many candidates lie "below" a given choice, allowing us to jump directly to the correct number rather than enumerating.

## Example Walkthrough

Let's trace through the solution with a small example: **`n = 4`, `k = 2`**. We expect the answer to be `9` (binary `1001`), as shown in the problem description.

To keep the walkthrough readable, we'll loop over bit positions from a small upper bound rather than `49` — the first set bit of our answer is at position `3`, so positions above that contribute nothing. We start at bit position `i = 4` and work downward.

**Precomputed combination values we'll need:**

| `C(i, k)` | Value |
| --------- | ----- |
| `C(4, 2)` | `6`   |
| `C(3, 2)` | `3`   |
| `C(2, 2)` | `1`   |
| `C(2, 1)` | `2`   |
| `C(1, 1)` | `1`   |

We initialize `ans = 0`, `n = 4`, `k = 2`.

**Bit `i = 4`:** How many valid numbers exist if bit `4` is `0`? That's `C(4, 2) = 6`.
- Compare: `n = 4 > 6`? **No.** The 4th number lies among the group where bit `4` is `0`.
- Leave bit `4` as `0`. State unchanged: `n = 4`, `k = 2`, `ans = 0`.

**Bit `i = 3`:** If bit `3` is `0`, count is `C(3, 2) = 3`.
- Compare: `n = 4 > 3`? **Yes.** The 4th number cannot be in the first 3 smaller numbers, so bit `3` must be `1`.
- `n -= 3` → `n = 1`
- `ans |= 1 << 3` → `ans = 8` (binary `1000`)
- `k -= 1` → `k = 1`
- `k == 0`? No, continue.

**Bit `i = 2`:** If bit `2` is `0`, count is `C(2, 1) = 2`.
- Compare: `n = 1 > 2`? **No.** The target lies in the group where bit `2` is `0`.
- Leave bit `2` as `0`. State unchanged: `n = 1`, `k = 1`, `ans = 8`.

**Bit `i = 1`:** If bit `1` is `0`, count is `C(1, 1) = 1`.
- Compare: `n = 1 > 1`? **No.** The target lies in the group where bit `1` is `0`.
- Leave bit `1` as `0`. State unchanged: `n = 1`, `k = 1`, `ans = 8`.

**Bit `i = 0`:** If bit `0` is `0`, count is `C(0, 1) = 0`.
- Compare: `n = 1 > 0`? **Yes.** Bit `0` must be `1`.
- `n -= 0` → `n = 1`
- `ans |= 1 << 0` → `ans = 9` (binary `1001`)
- `k -= 1` → `k = 0`
- `k == 0`? **Yes — break.**

**Result:** `ans = 9` (binary `1001`), which has exactly `k = 2` set bits. ✅

This matches the expected 4th number in the sequence `3, 5, 6, 9, ...`. Notice how the algorithm never enumerated any candidate numbers — at each bit it used a single combination lookup to decide whether to "jump past" a whole slice of smaller numbers (set the bit to `1`) or descend into that slice (leave the bit `0`).

## Python Solution

```python
# Precompute binomial coefficients (Pascal's triangle).
# MAX_BITS is the upper bound on the number of bits we may need.
MAX_BITS = 50

# binomial[i][j] = C(i, j) = number of ways to choose j items from i items
binomial = [[0] * (MAX_BITS + 1) for _ in range(MAX_BITS)]
for i in range(MAX_BITS):
    binomial[i][0] = 1  # C(i, 0) is always 1
    for j in range(1, i + 1):
        # Pascal's rule: C(i, j) = C(i-1, j-1) + C(i-1, j)
        binomial[i][j] = binomial[i - 1][j - 1] + binomial[i - 1][j]


class Solution:
    def nthSmallest(self, n: int, k: int) -> int:
        """
        Return the n-th smallest non-negative integer whose binary
        representation contains exactly k set bits (1s).

        Uses the combinatorial number system: scan bit positions from
        high to low and greedily decide whether each bit should be set.
        """
        ans = 0

        # Iterate bit positions from the most significant down to the least.
        for bit in range(MAX_BITS - 1, -1, -1):
            # binomial[bit][k] = how many numbers can be formed using the
            # lower `bit` positions while still needing k set bits, i.e.
            # the count of candidates that keep this bit unset.
            if n > binomial[bit][k]:
                # Not enough candidates with this bit unset, so it must
                # be set. Skip past those candidates.
                n -= binomial[bit][k]
                ans |= 1 << bit  # set this bit
                k -= 1           # one fewer set bit remains to place

                # All required set bits placed; remaining bits stay 0.
                if k == 0:
                    break

        return ans
```

## Time and Space Complexity

The time complexity is `O(log² M)`, and the space complexity is `O(log² M)`, where `M` is the upper bound of the answer, `2^50`.

### Time Complexity Analysis

The precomputation of the binomial coefficient table involves a nested loop. The outer loop runs `mx` times, and the inner loop runs up to `i + 1` times, resulting in `O(mx²)` operations. Since `mx` corresponds to `log M` (the number of bits, here `50`), this precomputation costs `O(log² M)`.

The `nthSmallest` method iterates from bit position `49` down to `0`, performing constant-time work in each iteration. This loop runs at most `O(log M)` times, contributing `O(log M)` to the overall time.

Combining both parts, the dominant term is the table precomputation, so the total time complexity is `O(log² M)`.

### Space Complexity Analysis

The two-dimensional array `c` has dimensions `mx × (mx + 1)`, storing `O(mx²)` entries. Since `mx` corresponds to `log M`, the space required is `O(log² M)`. The remaining variables in `nthSmallest` use only constant extra space, so the overall space complexity is `O(log² M)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### Pitfall 1: Off-by-One Confusion Between 0-Indexed and 1-Indexed Counting

The most frequent mistake is mishandling the comparison between `n` and `binomial[bit][k]`. The natural instinct is to write the condition as `n >= binomial[bit][k]`, but this is **wrong** and produces an off-by-one error.

**Why it's wrong:** `binomial[bit][k]` counts how many valid numbers exist when the current bit is left *unset* (all `k` ones must fit in the lower `bit` positions). These candidates occupy ranks `1` through `binomial[bit][k]`. The `n`th number belongs to this "bit unset" group precisely when `n <= binomial[bit][k]`. Therefore, we set the bit only when `n > binomial[bit][k]`.

```python
# WRONG: incorrectly classifies the boundary case where n exactly
# equals binomial[bit][k], pushing it into the "set bit" branch.
if n >= binomial[bit][k]:
    n -= binomial[bit][k]
    ...

# CORRECT: the equality case belongs to the "bit unset" group.
if n > binomial[bit][k]:
    n -= binomial[bit][k]
    ...
```

**Concrete example:** With `k = 2`, `n = 1`, the answer should be `3` (binary `11`). If you used `>=`, at `bit = 1`, `binomial[1][2] = 0`, so `n >= 0` is true and you would wrongly set bit `1` while still having `n = 1`, corrupting the construction. The strict `>` keeps the logic aligned with 1-indexed ranking.

### Pitfall 2: Insufficient Pascal's Triangle Dimensions

The table is declared as `binomial[i][j]` for `i` in `range(MAX_BITS)` and `j` up to `MAX_BITS`. A subtle trap is accessing `binomial[bit][k]` when `k` could exceed the inner dimension, or when `bit < k`.

**The safe-by-design detail:** When `bit < k`, you cannot fit `k` ones into `bit` positions, so `C(bit, k)` must be `0`. The initialization `[[0] * (MAX_BITS + 1)]` ensures these entries are `0` by default, and the inner loop `for j in range(1, i + 1)` never overwrites them. If you mistakenly looped `j` only up to some smaller bound, or shrank the column dimension below the maximum possible `k`, you would read garbage or trigger an `IndexError`.

```python
# WRONG: inner dimension too small — if k can be up to 50,
# accessing binomial[bit][50] raises IndexError.
binomial = [[0] * MAX_BITS for _ in range(MAX_BITS)]  # only 0..49 columns

# CORRECT: allocate MAX_BITS + 1 columns to safely index k.
binomial = [[0] * (MAX_BITS + 1) for _ in range(MAX_BITS)]
```

### Pitfall 3: Forgetting the Early `break` and Leaving `k` Unhandled

If you omit the `if k == 0: break`, the loop continues to lower bits. Although `binomial[bit][0] = 1` for every bit, leaving `k = 0` means every subsequent comparison becomes `n > 1`. If `n` was reduced to exactly `1` (its expected terminal value once all ones are placed), the condition `n > 1` is false, so no extra bits get set and the answer stays correct — but relying on this is fragile.

**Why the `break` matters:** It makes the termination explicit and guards against any scenario where `n` hasn't settled to `1`. Without it, a logic error elsewhere could silently set spurious low bits. The defensive `break` documents intent and short-circuits the remaining work for an `O(remaining bits)` saving.

```python
# Risky: relies on n already being 1 and binomial[bit][0] == 1
# to avoid setting extra bits. Harder to reason about.
if n > binomial[bit][k]:
    n -= binomial[bit][k]
    ans |= 1 << bit
    k -= 1
    # no break

# Robust: stop immediately once all k ones are placed.
if n > binomial[bit][k]:
    n -= binomial[bit][k]
    ans |= 1 << bit
    k -= 1
    if k == 0:
        break
```

### Pitfall 4: Misinterpreting the Bit Range (Including or Excluding Bit 49)

The loop runs `for bit in range(MAX_BITS - 1, -1, -1)`, i.e. from bit `49` down to bit `0`. A common error is writing `range(MAX_BITS, -1, -1)` (starting at bit `50`) or `range(MAX_BITS - 1, 0, -1)` (stopping at bit `1` and skipping bit `0`).

- Starting at bit `50` would attempt `1 << 50`, exceeding the guaranteed `2^50` bound and reading `binomial[50][k]`, which is out of range.
- Stopping at bit `1` skips the least significant bit entirely, so any answer needing bit `0` set (such as odd numbers) would be wrong.

```python
# WRONG: skips bit 0, breaks all numbers requiring the lowest bit.
for bit in range(MAX_BITS - 1, 0, -1):
    ...

# CORRECT: includes bit 0 as the final position considered.
for bit in range(MAX_BITS - 1, -1, -1):
    ...
```

Always verify the loop covers exactly bits `0` through `49`, matching the problem's `2^50` upper bound.

