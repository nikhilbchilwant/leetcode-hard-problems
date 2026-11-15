# 3699. Number of ZigZag Arrays I

> Source: https://algo.monster/liteproblems/3699

## Problem Statement

You are given three integers `n`, `l`, and `r`.

Count how many arrays of length `n` can be built subject to all of the following rules:

- Every entry is an integer drawn from the range `[l, r]` (inclusive).
- Neighboring entries are never equal.
- No three consecutive entries climb strictly upward, and no three consecutive entries fall strictly downward.

An array that satisfies every rule is called a **ZigZag** array. Because the count can be very large, return it **modulo** `10^9 + 7`.

**Example 1:** For `n = 3, l = 4, r = 5` the answer is `2`. Only the values `4` and `5` are available, and neighbors must differ, so the array is forced to alternate between the two. Starting with `4` produces `[4, 5, 4]`, and starting with `5` produces `[5, 4, 5]`.

**Example 2:** For `n = 3, l = 1, r = 3` the answer is `10`. The valid arrays are `[1,2,1]`, `[1,3,1]`, `[1,3,2]`, `[2,1,2]`, `[2,1,3]`, `[2,3,1]`, `[2,3,2]`, `[3,1,2]`, `[3,1,3]`, and `[3,2,3]`. An array such as `[1,2,3]` is rejected because its three entries are strictly increasing, and `[1,2,2]` is rejected because two equal entries sit side by side.

The constraints are `3 <= n <= 2000` and `1 <= l < r <= 2000`.

## Intuition

A direct approach would enumerate every candidate array: try each of the `m = r - l + 1` values at each of the `n` positions and test the rules. That is `m^n` arrays, and with `n` and `m` as large as `2000`, enumeration is out of the question. We need to **count arrays without listing them**.

The rules interact in a useful way. Because neighboring entries can never be equal, every step from one position to the next moves either strictly **up** or strictly **down**. Because three consecutive entries can never be strictly increasing, an up-step is never followed by another up-step, and the same argument forbids two down-steps in a row. Combining the two conditions, the directions must alternate perfectly: `up, down, up, down, ...` or `down, up, down, up, ...`. The *shape* of every valid array is completely determined — a zigzag — and the only remaining freedom is **which values** occupy the positions.

This suggests building arrays one position at a time and asking what information a partial array must carry for us to extend it. Two facts suffice: the value at its last position (the next value must be strictly smaller or strictly larger than it) and the direction of its last step (which fixes whether the next step goes down or up). Nothing earlier in the array matters. When the future depends only on such a small summary of the past, we can count with dynamic programming over those summaries: for every pair (last value, last direction), track how many partial arrays end in that state.

Absolute values are irrelevant beyond their order, so relabel the allowed values `l..r` as `1..m`. Let `up[v]` be the number of arrays of the current length that end at value `v` with a final up-step, and `down[v]` the same for a final down-step. To append value `v` with an up-step, the previous entry must be some `u < v`, and that shorter array must have ended with a down-step — so the new `up[v]` is the sum of `down[u]` over all `u < v`. Computing each of these range sums with its own loop costs `O(m^2)` per position, which is too slow at the given limits. The sums we need, however, are exactly **prefix sums** of `down` (and suffix sums of `up`), so a single running total produces an entire new table in `O(m)`.

## Solution Approach

### Solution 1: Counting DP with Prefix Sums

Only the relative order of values matters, so map the allowed range `l..r` onto `1..m`, where `m = r - l + 1`.

For arrays of the current length, define two tables:

- `up[v]` — the number of valid arrays ending at value `v` whose final step went up (the previous entry is smaller than `v`).
- `down[v]` — the number of valid arrays ending at value `v` whose final step went down (the previous entry is larger than `v`).

**Base case (length 2).** Any two distinct values form a valid array of length 2. An array ending at `v` with an up-step chooses its first entry among the `v - 1` values below `v`, so `up[v] = v - 1`. Symmetrically, `down[v] = m - v`. (The code stores values 0-indexed as `0..m-1`, which turns these formulas into `up[v] = v` and `down[v] = m - 1 - v`.)

**Transition (length k to length k + 1).** Directions alternate, so an up-step onto `v` can only extend an array that ended with a down-step at some smaller value, and a down-step onto `v` can only extend an array that ended with an up-step at some larger value:

```
new_up[v]   = down[1] + down[2] + ... + down[v-1]
new_down[v] = up[v+1] + up[v+2] + ... + up[m]
```

Evaluating each sum with an inner loop costs `O(m)` per entry, `O(m^2)` per length, and `O(n * m^2)` overall — about `8 * 10^9` additions when `n` and `m` both reach `2000`. That is too slow.

**Prefix-sum optimization.** The values `new_up[1..m]` are the prefix sums of `down`, and `new_down[1..m]` are the suffix sums of `up`. A running variable computes each table in one sweep:

```python
s = 0
for v in range(m):          # left to right
    new_up[v] = s           # sum of down[0..v-1]
    s = (s + down[v]) % MOD
```

The mirrored right-to-left sweep fills `new_down` from `up`. Each length extension now costs `O(m)`, and the array grows from length 2 to length `n`, so there are `n - 2` extensions.

**Final answer.** Every valid array of length `n` ends with either an up-step or a down-step, so the answer is the sum of both tables, taken modulo `10^9 + 7`.

**Complexity Analysis:**

- **Time complexity:** `O(n * m)`, where `m = r - l + 1`. Each of the `n - 2` extensions performs two `O(m)` sweeps.
- **Space complexity:** `O(m)`. Only the tables for the current length are kept; earlier lengths are discarded.

## Example Walkthrough

Trace the computation for `n = 4`, `l = 1`, `r = 3`. Here `m = 3` and the relabeled values `1..3` coincide with the actual values, which keeps the tables easy to read.

---

#### Length 2 (base case)

Every ordered pair of distinct values is a valid array of length 2. `up[v] = v - 1` and `down[v] = m - v`:

| `v` | `up[v]` | arrays counted | `down[v]` | arrays counted |
|-----|---------|----------------|-----------|----------------|
| 1 | 0 | — | 2 | `[2,1]`, `[3,1]` |
| 2 | 1 | `[1,2]` | 1 | `[3,2]` |
| 3 | 2 | `[1,3]`, `[2,3]` | 0 | — |

---

#### Extend to length 3

An up-step onto `v` extends a down-ending array at a smaller value, so `new_up` accumulates `down` from the left:

- `new_up[1] = 0` (no smaller value exists)
- `new_up[2] = down[1] = 2` — the arrays `[2,1,2]` and `[3,1,2]`
- `new_up[3] = down[1] + down[2] = 3` — the arrays `[2,1,3]`, `[3,1,3]`, `[3,2,3]`

A down-step onto `v` extends an up-ending array at a larger value, so `new_down` accumulates `up` from the right:

- `new_down[1] = up[2] + up[3] = 3` — the arrays `[1,2,1]`, `[1,3,1]`, `[2,3,1]`
- `new_down[2] = up[3] = 2` — the arrays `[1,3,2]`, `[2,3,2]`
- `new_down[3] = 0` (no larger value exists)

The tables are now `up = [0, 2, 3]` and `down = [3, 2, 0]`. Their combined total is `0 + 2 + 3 + 3 + 2 + 0 = 10`, which matches the answer for `n = 3, l = 1, r = 3` given in the problem — a useful checkpoint that the transition is correct.

---

#### Extend to length 4

Repeat the same two sweeps on the new tables:

- `new_up` = prefix sums of `down = [3, 2, 0]`, giving `[0, 3, 5]`
- `new_down` = suffix sums of `up = [0, 2, 3]`, giving `[5, 3, 0]`

For instance, `new_down[1] = up[2] + up[3] = 2 + 3 = 5`: each of the five up-ending length-3 arrays — `[2,1,2]`, `[3,1,2]`, `[2,1,3]`, `[3,1,3]`, `[3,2,3]` — gains a final `1`. Note that `[1,2,3,1]` is not counted, because its prefix `[1,2,3]` climbs three times in a row and therefore never entered the tables at length 3.

The answer for `n = 4` is `0 + 3 + 5 + 5 + 3 + 0 = 16`.

## Python Solution

```python
class Solution:
    def zigZagArrays(self, n: int, l: int, r: int) -> int:
        MOD = 10**9 + 7
        # Relabel the allowed values l..r as 0..m-1; only their relative
        # order matters, not the absolute numbers.
        m = r - l + 1

        # up[v]:   arrays of the current length ending at value v whose
        #          final step went UP (previous entry < v).
        # down[v]: arrays of the current length ending at value v whose
        #          final step went DOWN (previous entry > v).
        # Base case: arrays of length 2. An up-ending array picks its first
        # entry among the v values below v, so up[v] = v; symmetrically
        # down[v] = m - 1 - v.
        up = list(range(m))
        down = [m - 1 - v for v in range(m)]

        # Grow the arrays one position at a time: length 2 -> length n.
        for _ in range(n - 2):
            new_up = [0] * m
            new_down = [0] * m

            # An up-step onto v extends a down-ending array at some u < v:
            # new_up[v] = down[0] + ... + down[v-1], a running prefix sum.
            s = 0
            for v in range(m):
                new_up[v] = s
                s = (s + down[v]) % MOD

            # A down-step onto v extends an up-ending array at some u > v:
            # new_down[v] = up[v+1] + ... + up[m-1], a running suffix sum.
            s = 0
            for v in range(m - 1, -1, -1):
                new_down[v] = s
                s = (s + up[v]) % MOD

            up, down = new_up, new_down

        # Every valid array ends with either an up-step or a down-step.
        return (sum(up) + sum(down)) % MOD
```

## Time and Space Complexity

**Time Complexity: `O(n * m)`**, where `m = r - l + 1` is the number of distinct allowed values.

The tables start at length 2 and are extended `n - 2` times. Each extension performs two sweeps over the value range:

1. **Left-to-right sweep:** fills `new_up` while maintaining a running prefix sum of `down`. Each of the `m` iterations does constant work (one assignment, one addition, one modulo).
2. **Right-to-left sweep:** fills `new_down` while maintaining a running suffix sum of `up`, again `O(1)` work per value.

Each extension therefore costs `O(m)`, and the whole computation costs `O(n * m)` — at the maximum constraints (`n = m = 2000`), roughly `4 * 10^6` iterations per direction table, comfortably fast. The final summation of both tables adds one more `O(m)` pass.

**Space Complexity: `O(m)`**

Only four arrays of length `m` exist at any moment: the current `up` and `down` tables and the `new_up` and `new_down` tables being built. Tables for earlier lengths are discarded as soon as the next length is computed (a rolling-array scheme), so memory does not grow with `n`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### Pitfall 1: Evaluating each transition sum with its own loop

The transition `new_up[v] = down[1] + ... + down[v-1]` invites a direct nested loop:

```python
# DON'T DO THIS: O(m) work for every value at every length
for v in range(m):
    new_up[v] = sum(down[u] for u in range(v)) % MOD
    new_down[v] = sum(up[u] for u in range(v + 1, m)) % MOD
```

This makes each extension `O(m^2)` and the whole algorithm `O(n * m^2)`. At `n = m = 2000` that is on the order of `8 * 10^9` additions — a guaranteed time-limit exceeded.

**Solution:** The `m` sums needed for `new_up` are the prefix sums of `down`, so one running variable produces all of them in a single `O(m)` sweep (and a mirrored sweep produces `new_down`). This is the difference between an accepted `O(n * m)` solution and a TLE.

---

### Pitfall 2: Adding the current cell to the running sum too early

The order of the two statements inside the sweep matters:

```python
# WRONG: down[v] is folded in before new_up[v] is stored
s = 0
for v in range(m):
    s = (s + down[v]) % MOD
    new_up[v] = s            # now includes down[v] itself
```

Including `down[v]` in `new_up[v]` counts a "step" from a value to itself — an array with two equal neighbors, which the problem forbids. The error is easy to miss because the answer is still plausible-looking. On Example 1 (`n = 3, l = 4, r = 5`) this version returns `4` instead of `2`: it additionally counts `[5,4,4]` and `[4,5,5]`.

**Solution:** Store first, then accumulate. `new_up[v]` must be the running total *before* `down[v]` is added, so the sum covers exactly the values strictly below `v` (and symmetrically, strictly above `v` for `new_down`).

---

### Pitfall 3: 32-bit overflow and misplaced modulo in Java and C++

Table entries are reduced modulo `10^9 + 7`, so each one can be as large as about `10^9`. In the final accumulation

```java
total = total + up[v] + down[v];   // three terms, each up to ~1e9
```

the intermediate value can reach about `3 * 10^9`, which exceeds the 32-bit `int` maximum of `2147483647` and silently wraps to a negative number. The same wraparound threatens any accumulation that skips the `% MOD` reduction for a few steps.

**Solution:** Use `long` (Java) or `long long` (C++) for the tables and every accumulator, and apply `% MOD` at each addition — including the final total. In TypeScript the analogous danger is exceeding `Number.MAX_SAFE_INTEGER` (`2^53 - 1`) if reductions are skipped; taking `% MOD` after every addition keeps all intermediates below `2 * 10^9`, well within safe integer range. Python integers do not overflow, but the final `% MOD` is still required by the problem statement.

