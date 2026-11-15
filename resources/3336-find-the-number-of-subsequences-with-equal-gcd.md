# 3336. Find the Number of Subsequences With Equal GCD

> Source: https://algo.monster/liteproblems/3336

## Problem Statement

You are given an integer array `nums`. Count how many ways you can pick two subsequences `seq1` and `seq2` of `nums` such that all of the following hold:

- Both subsequences are **non-empty**.
- The two subsequences are **disjoint**: no index of `nums` is used by both. (The same value may appear in both if it occurs at different indices.)
- The greatest common divisor (GCD) of the elements of `seq1` equals the GCD of the elements of `seq2`.

The pair `(seq1, seq2)` is **ordered** — picking a set of indices as `seq1` and another as `seq2` is counted separately from the same two sets with the roles swapped.

Since the count can be enormous, return it **modulo** `10^9 + 7`.

**Example 1:** For `nums = [1, 2, 3, 4]` the answer is `10`. Every valid pair has GCD `1` on both sides; two of the ten are `seq1 = [1]` with `seq2 = [2, 3]`, and `seq1 = [2, 3]` with `seq2 = [1, 4]`.

**Example 2:** For `nums = [10, 20, 30]` the answer is `2`. The only valid selection uses `[10]` on one side and `[20, 30]` on the other (both have GCD `10`), and it is counted once for each ordering of the two roles.

**Example 3:** For `nums = [1, 1, 1, 1]` the answer is `50`. All elements equal `1`, so every ordered pair of disjoint non-empty subsequences qualifies, and there are `50` of them.

**Constraints:** `1 <= nums.length <= 200` and `1 <= nums[i] <= 200`.

## Intuition

A brute-force solution assigns each element of `nums` to one of three destinations — `seq1`, `seq2`, or neither — and then checks whether both subsequences are non-empty and their GCDs match. With `n` up to `200` there are `3^n` assignments, far too many to enumerate, so the assignments have to be counted in groups rather than one at a time. This is the setup for a counting [dynamic programming](/problems/dynamic_programming_intro) approach: find a compact summary of a partial assignment such that all assignments sharing the same summary can be carried forward together.

Suppose we build the two subsequences by scanning `nums` left to right and ask what the past choices actually contribute to the future. The final check compares two GCDs. When an element `x` joins a subsequence whose running GCD is `g`, that GCD becomes `gcd(g, x)`; which specific indices produced `g`, how many elements were involved, and in what order they arrived play no role in this update or in the final comparison. Two partial assignments that reach the same pair of running GCDs are therefore interchangeable from that point on, and a single counter per pair suffices. This is **why the state works**: the pair of running GCDs is everything the future can observe about the past.

The value range keeps the number of such pairs small. Every element lies between `1` and `200`, and the GCD of a non-empty subsequence divides each of its elements, so it also lies between `1` and `200`. Using `0` to mark a subsequence that is still empty, the state `(GCD of seq1, GCD of seq2)` takes at most `201 × 201` values. The sentinel meshes with the update rule because `gcd(0, x) = x`: the first element to join an empty subsequence sets the GCD to itself with no special case.

So let `f[j][k]` be the number of ways to place the elements processed so far such that `seq1` has GCD `j` and `seq2` has GCD `k`. Each element `x` sends every state `(j, k)` to three successor states: `(j, k)` if `x` is skipped, `(gcd(j, x), k)` if it joins `seq1`, and `(j, gcd(k, x))` if it joins `seq2`. Every one of the `3^n` assignments is covered exactly once. After the last element, `f[g][g]` with `g >= 1` counts the assignments in which both subsequences are non-empty and share GCD `g`, and the answer is the sum of these diagonal entries modulo `10^9 + 7`.

## Solution Approach

### Solution 1: DP over pairs of running GCDs

We process `nums` one element at a time and maintain a table `f`, where `f[j][k]` is the number of assignments of the processed elements in which `seq1` currently has GCD `j` and `seq2` has GCD `k`. The value `0` represents a subsequence that is still empty, so the table starts as `f[0][0] = 1`: before any element is placed, both subsequences are empty in exactly one way.

For each element `x` we build a fresh table `g` instead of editing `f` in place. Every state in `f` describes assignments that have not yet decided what to do with `x`; writing into the table being scanned would let a state created by `x` be processed again for the same `x`, placing it in both subsequences. Each state `(j, k)` with count `v` contributes `v` to three entries of the new table:

1. `g[j][k]` — `x` joins neither subsequence.
2. `g[gcd(j, x)][k]` — `x` joins `seq1`.
3. `g[j][gcd(k, x)]` — `x` joins `seq2`.

Because `j` and `k` both range over `0..m` where `m = max(nums)`, we precompute `gcd(j, x)` for every `j` once per element and store it in an array `gcd_with_x`. The inner double loop then performs only table lookups and modular additions. Skipping zero-count states (`if v == 0: continue`) does not change the asymptotic bound but avoids useless work early on, when few states are reachable.

After the final element, the diagonal entries with `j = k = d` and `d >= 1` are exactly the assignments in which both subsequences are non-empty (a positive GCD implies at least one element) and their GCDs are equal. The answer is:

`answer = (f[1][1] + f[2][2] + ... + f[m][m]) mod (10^9 + 7)`

### Why every pair is counted exactly once

An ordered pair of disjoint subsequences corresponds to exactly one assignment of each index to `seq1`, `seq2`, or neither, and the DP applies exactly one of the three transitions per index along every path. The mapping between assignments and DP paths is one-to-one, so no pair is missed and none is double-counted. Ordered pairs come out naturally: the state keeps `seq1`'s GCD in the first coordinate and `seq2`'s in the second, so `([10], [20, 30])` and `([20, 30], [10])` follow different paths and are tallied separately, matching the expected output of Example 2.

**Complexity Analysis:**

- **Time complexity:** `O(n * m^2)`, where `n` is the length of `nums` and `m = max(nums)`. Each of the `n` elements updates `(m + 1)^2` states with constant work per state, plus `O(m log m)` per element to precompute the GCD row. At the limits `n = m = 200`, this is about 8.1 million state updates.
- **Space complexity:** `O(m^2)` for the current table and the next table.

## Example Walkthrough

Let's trace `nums = [10, 20, 30]`, whose expected answer is `2`.

Throughout, `f[j][k]` counts the ways to assign the elements seen so far such that `seq1` has GCD `j` and `seq2` has GCD `k`, with `0` meaning "still empty". We start with `f[0][0] = 1`.

---

#### Element 10

The single starting state `(0, 0)` branches three ways: skip 10, put it in `seq1` (GCD becomes `gcd(0, 10) = 10`), or put it in `seq2`.

| State `(j, k)` | Count | Meaning |
|---|---|---|
| `(0, 0)` | 1 | 10 unused |
| `(10, 0)` | 1 | `seq1 = [10]` |
| `(0, 10)` | 1 | `seq2 = [10]` |

Total ways: `3 = 3^1`. ✓

---

#### Element 20

Each of the three states branches again. Note that two different choices can land in the same state: from `(10, 0)`, skipping 20 and adding 20 to `seq1` both give GCD `gcd(10, 20) = 10`, so `(10, 0)` accumulates a count of `2`.

| State `(j, k)` | Count |
|---|---|
| `(0, 0)` | 1 |
| `(10, 0)` | 2 |
| `(0, 10)` | 2 |
| `(20, 0)` | 1 |
| `(0, 20)` | 1 |
| `(10, 20)` | 1 |
| `(20, 10)` | 1 |

Total ways: `9 = 3^2`. ✓ The state `(10, 20)` is `seq1 = [10]`, `seq2 = [20]`; its GCDs differ, but it is kept because a future element may still change either side.

---

#### Element 30

After processing 30 the table holds fourteen states:

| State `(j, k)` | Count | | State `(j, k)` | Count |
|---|---|---|---|---|
| `(0, 0)` | 1 | | `(10, 30)` | 2 |
| `(10, 0)` | 5 | | `(20, 10)` | 2 |
| `(0, 10)` | 5 | | `(30, 10)` | 2 |
| `(20, 0)` | 1 | | `(20, 30)` | 1 |
| `(0, 20)` | 1 | | `(30, 20)` | 1 |
| `(30, 0)` | 1 | | `(0, 30)` | 1 |
| `(10, 20)` | 2 | | **`(10, 10)`** | **2** |

Total ways: `27 = 3^3`. ✓

The two paths into `(10, 10)` are worth spelling out. From `(10, 20)` — that is, `seq1 = [10]`, `seq2 = [20]` — element 30 joins `seq2`, turning its GCD into `gcd(20, 30) = 10`. From `(20, 10)` — `seq1 = [20]`, `seq2 = [10]` — element 30 joins `seq1` with the same effect. These are the assignments `([10], [20, 30])` and `([20, 30], [10])`.

---

#### Reading the answer

We sum the diagonal entries `f[d][d]` for `d >= 1`. Only `f[10][10] = 2` is non-zero, so the answer is `2`, matching the expected output. The entry `f[0][0] = 1` (nothing selected at all) sits on the diagonal too, which is exactly why the sum must start at `d = 1`.

## Python Solution

```python
from typing import List
from math import gcd

class Solution:
    def subsequencePairCount(self, nums: List[int]) -> int:
        MOD = 10**9 + 7
        mx = max(nums)

        # f[j][k] = number of ways to distribute the processed elements so that
        # seq1 currently has GCD j and seq2 has GCD k (0 means "still empty").
        f = [[0] * (mx + 1) for _ in range(mx + 1)]
        f[0][0] = 1

        for x in nums:
            # Precompute gcd(j, x) for every possible running GCD j.
            # gcd(0, x) = x, which covers the "first element" case naturally.
            gcd_with_x = [gcd(j, x) for j in range(mx + 1)]

            g = [[0] * (mx + 1) for _ in range(mx + 1)]
            for j in range(mx + 1):
                gj = gcd_with_x[j]
                row = f[j]
                for k in range(mx + 1):
                    v = row[k]
                    if v == 0:
                        continue
                    # Choice 1: x joins neither subsequence.
                    g[j][k] = (g[j][k] + v) % MOD
                    # Choice 2: x joins seq1; its GCD becomes gcd(j, x).
                    g[gj][k] = (g[gj][k] + v) % MOD
                    # Choice 3: x joins seq2; its GCD becomes gcd(k, x).
                    gk = gcd_with_x[k]
                    g[j][gk] = (g[j][gk] + v) % MOD
            f = g

        # Both subsequences must be non-empty (GCD > 0) and share the same GCD.
        return sum(f[d][d] for d in range(1, mx + 1)) % MOD
```

## Time and Space Complexity

**Time Complexity: `O(n * m^2)`**

Let `n` be the length of `nums` and `m = max(nums)`; the constraints cap both at `200`.

For each of the `n` elements, the algorithm performs two pieces of work:

1. **GCD row precomputation:** `gcd(j, x)` is evaluated for every `j` in `0..m`, costing `O(m log m)` per element (each Euclidean GCD on values up to `m` takes `O(log m)` steps).
2. **State transitions:** the double loop visits all `(m + 1)^2` states `(j, k)` and performs a constant number of array reads and modular additions per state, costing `O(m^2)` per element.

The transition loop dominates, giving `O(n * m^2)` overall. At the maximum input size `n = m = 200`, that is about `200 * 201 * 201 ≈ 8.1` million state updates, well within limits in every language. The `if v == 0: continue` guard skips unreachable states and speeds up the early iterations, though it does not change the worst-case bound.

**Space Complexity: `O(m^2)`**

The algorithm keeps two `(m + 1) x (m + 1)` tables at a time — the current table `f` and the next table `g` — plus an `O(m)` array for the precomputed GCD row. The `3^n` assignments are never materialized; they exist only as counts inside the tables.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### Pitfall 1: Including the empty state when summing the diagonal

The both-empty state `(0, 0)` sits on the diagonal with `f[0][0] = 1` at the end (the assignment that skips every element), and states like `(d, 0)` pair a real subsequence with an empty one. If the final sum starts at `d = 0`, the skip-everything assignment is counted as a "pair with equal GCD":

```python
# WRONG: counts the assignment where both subsequences are empty
return sum(f[d][d] for d in range(0, mx + 1)) % MOD
```

For `nums = [10, 20, 30]` this returns `3` instead of `2`. The problem requires both subsequences to be non-empty, and a non-empty subsequence always has GCD at least `1`, so the sum must run over `d = 1..m` only.

---

### Pitfall 2: Updating the DP table in place

Writing transition results back into the table being scanned lets a single element be placed twice. Take `nums = [6]`: starting from `f[0][0] = 1`, an in-place update adds `1` to `f[6][0]` (6 joins `seq1`). When the scan later reaches cell `(6, 0)` — still processing the same element — it treats that count as undecided and adds `1` to `f[6][gcd(0, 6)] = f[6][6]`. The algorithm then reports `1` for a single-element array, counting the pair `([6], [6])` that uses index 0 on both sides. The correct answer is `0`, since two disjoint non-empty subsequences need at least two indices.

```python
g = [[0] * (mx + 1) for _ in range(mx + 1)]  # fresh table per element
...
f = g  # replace only after all states of f are processed
```

Reading from `f` and writing to `g` guarantees each element makes exactly one choice per assignment.

---

### Pitfall 3: Treating the pair as unordered

Example 2 lists `([10], [20, 30])` and `([20, 30], [10])` as two distinct pairs — the roles of `seq1` and `seq2` matter, and the expected output is `2`, not `1`. A solution that counts each unordered selection once, or "fixes" a doubled count by dividing the final sum by two, returns `1` for `[10, 20, 30]` and `5` instead of `10` for `[1, 2, 3, 4]`. The division is also numerically wrong under a modulus: halving a value mod `10^9 + 7` requires multiplying by the modular inverse of 2, not integer division. The DP needs no correction factor at all — its state already distinguishes `seq1`'s GCD (first coordinate) from `seq2`'s (second coordinate), so ordered pairs fall out naturally.

