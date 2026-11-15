# 3640. Trionic Array II

> Source: https://algo.monster/liteproblems/3640

## Problem Statement

You are given an integer array `nums` of length `n`.

A **trionic subarray** is a contiguous subarray `nums[l...r]` (with `0 <= l < r < n`) for which there exist indices `l < p < q < r` such that:

- `nums[l...p]` is **strictly** increasing,
- `nums[p...q]` is **strictly** decreasing,
- `nums[q...r]` is **strictly** increasing.

Return the **maximum** sum of any trionic subarray in `nums`.

In other words, a trionic subarray follows an "up-down-up" pattern: it first rises strictly, then falls strictly, and finally rises strictly again. Each of the three segments must contain at least two elements (i.e., the boundaries satisfy `l < p < q < r`), so the whole subarray needs a minimum of four elements with three distinct "turning points" at indices `p` and `q`.

Your task is to examine every possible trionic subarray, compute the sum of its elements, and return the largest such sum found across all valid candidates.

## Intuition

The key observation is that a trionic subarray has a very specific shape: it goes **up**, then **down**, then **up** again. This shape is built entirely from "monotonic runs" — maximal stretches where the array is strictly increasing or strictly decreasing. So instead of checking every possible `(l, p, q, r)` combination, we can scan the array once and identify these runs as we go.

Think about how these runs connect. A strictly increasing run, followed by a strictly decreasing run, followed by another strictly increasing run, forms the core "up-down-up" skeleton. The natural strategy is to use a single pointer `i` that walks through the array and "consumes" each segment in turn:

1. First, consume a strictly increasing stretch (this gives us the rising part `nums[l...p]`).
2. Then, consume a strictly decreasing stretch (this gives us the falling part `nums[p...q]`).
3. Finally, consume another strictly increasing stretch (this gives us the second rising part `nums[q...r]`).

If any of these stretches fails to materialize (for example, a segment has only one element, or we hit equal adjacent values, or we run off the end of the array), the current candidate is invalid and we move on.

Once we lock onto a valid up-down-up core, we want to **maximize the sum**. Here's the clever part: the middle "down then up" structure (from `p` to just past `q`) is fixed for this candidate, but the two **ends** are flexible. We can extend the subarray:

- **Leftward** from the start of the first increasing run — by including extra elements before `p` that keep increasing into our segment. Since these extra elements could be negative, we don't blindly take all of them; we track the **maximum running sum** as we extend left and only keep the best portion (or nothing, if extending hurts).
- **Rightward** from the third increasing run — same idea. We extend through the trailing increasing elements and keep only the prefix that gives the maximum sum, using `0` if extending isn't beneficial.

This "take the best extension or none at all" logic is essentially a localized version of **maximum subarray** thinking applied to the two increasing tails.

The final reason this works in a single pass is the overlap trick: after processing one trionic subarray, the **third increasing run can serve as the first increasing run of the next candidate**. That's why the pointer is reset back to `q` rather than continuing forward — it lets adjacent up-down-up patterns chain together without re-scanning, keeping the whole algorithm linear.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

We use a **grouped loop** (also called a "segment scanning" technique): a single pointer `i` traverses the array, and within the loop we carve out the three required monotonic segments one after another. Let `n` be the length of `nums`, and initialize the answer `ans = -inf`.

**Step 1 — Find the first strictly increasing segment.**

We record the start `l = i`, then advance `i` while `nums[i - 1] < nums[i]`:

```python
l = i
i += 1
while i < n and nums[i - 1] < nums[i]:
    i += 1
if i == l + 1:
    continue
```

If after this `i == l + 1`, the segment has only one element (no real increase), so this start cannot anchor a trionic subarray and we move on.

**Step 2 — Find the strictly decreasing segment.**

Pointer `p = i - 1` marks the peak (end of the increasing part). We seed the running sum `s` with the last two values of the increasing run, `nums[p - 1] + nums[p]`, then keep adding elements while the array strictly decreases:

```python
p = i - 1
s = nums[p - 1] + nums[p]
while i < n and nums[i - 1] > nums[i]:
    s += nums[i]
    i += 1
if i == p + 1 or i == n or nums[i - 1] == nums[i]:
    continue
```

We discard this candidate if the decreasing segment is empty (`i == p + 1`), if we reached the array's end (`i == n`, leaving no room for the third segment), or if we stopped because of equal neighbors (`nums[i - 1] == nums[i]`, which breaks strict monotonicity).

**Step 3 — Find the second strictly increasing segment.**

Pointer `q = i - 1` marks the valley (end of the decreasing part). We add `nums[i]` (the first rising element after the valley) into `s`, then track the **best prefix sum** of the trailing increasing run using a running total `t` and its max `mx`:

```python
q = i - 1
s += nums[i]
i += 1
mx = t = 0
while i < n and nums[i - 1] < nums[i]:
    t += nums[i]
    i += 1
    mx = max(mx, t)
s += mx
```

Here `mx` captures the maximum sum we can gain by extending **rightward** beyond the mandatory part of the third segment, and stays `0` if extending isn't beneficial. So far `s` covers the core range `[p - 1, q + 1]` plus the best right extension.

**Step 4 — Extend leftward for the best left tail.**

The increasing segment also lets us pull in elements before `p - 1`. We walk backward from `p - 2` down to `l`, again keeping the maximum running sum:

```python
mx = t = 0
for j in range(p - 2, l - 1, -1):
    t += nums[j]
    mx = max(mx, t)
s += mx
```

This adds the best **leftward extension**, again defaulting to `0` if no extension helps.

**Step 5 — Update the answer and reset the pointer.**

We update `ans` with the computed sum, then move `i` back to `q`:

```python
ans = max(ans, s)
i = q
```

Resetting `i = q` is the key chaining trick: the third increasing run of the current trionic subarray can act as the **first increasing run** of the next candidate, so we re-enter the loop starting from the valley and continue without re-scanning.

**Complexity.**

- **Time complexity:** `O(n)`. Although the pointer is reset to `q`, each segment is processed a constant number of times overall, so the total work is linear.
- **Space complexity:** `O(1)`, since we only use a handful of pointers and running-sum variables.

## Example Walkthrough

Let's trace through the solution approach with a small concrete example.

**Input:** `nums = [1, 4, 2, 1, 3, 5]`, so `n = 6`, and we initialize `ans = -inf`, pointer `i = 0`.

This array has the shape: rises `1 → 4`, falls `4 → 2 → 1`, then rises `1 → 3 → 5` — a clean "up-down-up" pattern.

---

**Step 1 — First strictly increasing segment.**

- Set `l = i = 0`, then advance `i` to `1`.
- Check `nums[0] < nums[1]` → `1 < 4` ✓, so `i` becomes `2`.
- Check `nums[1] < nums[2]` → `4 < 2` ✗, stop. Now `i = 2`.
- Is `i == l + 1`? → `2 == 1`? No. The increasing run `[1, 4]` is valid.

The peak is at index `1` (value `4`).

---

**Step 2 — Strictly decreasing segment.**

- Set `p = i - 1 = 1` (the peak).
- Seed `s = nums[0] + nums[1] = 1 + 4 = 5` (last two of the rising run).
- Check `nums[1] > nums[2]` → `4 > 2` ✓: `s += 2 → s = 7`, `i = 3`.
- Check `nums[2] > nums[3]` → `2 > 1` ✓: `s += 1 → s = 8`, `i = 4`.
- Check `nums[3] > nums[4]` → `1 > 3` ✗, stop. Now `i = 4`, `s = 8`.
- Validity checks: `i == p + 1`? `4 == 2`? No. `i == n`? `4 == 6`? No. `nums[3] == nums[4]`? `1 == 3`? No. **Valid.**

The valley is at index `3` (value `1`). Core so far covers indices `[0, 3]` with `s = 8`.

---

**Step 3 — Second strictly increasing segment.**

- Set `q = i - 1 = 3` (the valley).
- Add the first rising element after the valley: `s += nums[4] = 3 → s = 11`, then `i = 5`.
- Initialize `mx = t = 0` to track the best **right extension**.
- Check `nums[4] < nums[5]` → `3 < 5` ✓: `t += 5 → t = 5`, `i = 6`, `mx = max(0, 5) = 5`.
- Loop ends (`i = 6 = n`).
- Add best right extension: `s += mx = 5 → s = 16`.

Now `s = 16` covers the core `[0, 4]` plus the beneficial right tail (element `5`).

---

**Step 4 — Extend leftward.**

- Reset `mx = t = 0`. We walk from `j = p - 2 = -1` down to `l = 0`.
- Since `p - 2 = -1 < l = 0`, the range `range(-1, -1, -1)` is **empty** — there are no elements to the left of the mandatory start.
- Add `s += mx = 0 → s = 16` (no left extension available).

---

**Step 5 — Update answer and reset pointer.**

- `ans = max(-inf, 16) = 16`.
- Reset `i = q = 3`.

Re-entering the loop from `i = 3`, Step 1 starts a new increasing run at `l = 3` (`1 → 3 → 5`), but no decreasing segment follows (we hit the end), so that candidate is discarded.

---

**Result:** The maximum trionic subarray sum is **`16`**, corresponding to the full subarray `[1, 4, 2, 1, 3, 5]` (indeed `1+4+2+1+3+5 = 16`).

This example shows how the single pointer carves out each segment, how the running-sum extensions default to `0` when they'd hurt (here the left side simply had nothing to add), and how `s` accumulates the core plus the best flexible tails.

## Python Solution

```python
from typing import List
from math import inf


class Solution:
    def maxSumTrionic(self, nums: List[int]) -> int:
        n = len(nums)
        i = 0
        ans = -inf

        while i < n:
            # ---- Segment 1: strictly increasing run starting at `left` ----
            left = i
            i += 1
            while i < n and nums[i - 1] < nums[i]:
                i += 1
            # No actual ascent: skip
            if i == left + 1:
                continue

            # `peak` is the index of the peak (top of the first rise)
            peak = i - 1
            # Start the running sum with the last two elements of the rise:
            # the element just before the peak and the peak itself.
            total = nums[peak - 1] + nums[peak]

            # ---- Segment 2: strictly decreasing run from the peak ----
            while i < n and nums[i - 1] > nums[i]:
                total += nums[i]
                i += 1
            # Need a real descent followed by a real ascent;
            # also reject if we hit the end or a plateau (equal values).
            if i == peak + 1 or i == n or nums[i - 1] == nums[i]:
                continue

            # `valley` is the index of the valley (bottom of the descent)
            valley = i - 1
            total += nums[i]
            i += 1

            # ---- Segment 3: strictly increasing run from the valley ----
            # Extend the third (rising) part optimally: track the best
            # running prefix sum so we only add a beneficial extension.
            best, running = 0, 0
            while i < n and nums[i - 1] < nums[i]:
                running += nums[i]
                i += 1
                best = max(best, running)
            total += best

            # ---- Optionally extend the leading rise backwards ----
            # Walk leftward from just before the (peak-1) element down to
            # `left`, keeping the best running suffix sum to add.
            best, running = 0, 0
            for j in range(peak - 2, left - 1, -1):
                running += nums[j]
                best = max(best, running)
            total += best

            ans = max(ans, total)

            # Reset to the valley so the next iteration can treat this
            # valley as the start of a new (overlapping) trionic pattern.
            i = valley

        return ans
```

## Time and Space Complexity

**Time Complexity:** `O(n)`, where `n` is the length of the array `nums`.

Although the code contains nested `while` loops and a `for` loop, each element of the array is processed a constant number of times overall. Let's break down the reasoning:

- The outer `while i < n` loop drives the main traversal. Inside it, several inner loops advance the pointer `i`:
  - The first inner loop (`nums[i - 1] < nums[i]`) scans the strictly increasing segment.
  - The second inner loop (`nums[i - 1] > nums[i]`) scans the strictly decreasing segment.
  - The third inner loop scans the next increasing segment.
- After processing a valid "trionic" pattern, the line `i = q` resets the pointer back to the position `q` (the valley/turning point). This means part of the array may be revisited, but `q` is the start of the second increasing segment of the next candidate, so the backtracking is bounded.
- The inner `for j in range(p - 2, l - 1, -1)` loop iterates over the first increasing segment in reverse. Its length is bounded by the size of the current increasing run, and across all iterations these runs sum to at most `O(n)`.

Because the `i = q` reset only moves the pointer back to the boundary of the just-processed increasing run (not to the very beginning), the total number of pointer advances and the cumulative work of the reverse `for` loop remain linear. Thus the overall time complexity is `O(n)`.

**Space Complexity:** `O(1)`. The algorithm uses only a constant number of scalar variables (`i`, `l`, `p`, `q`, `s`, `mx`, `t`, `ans`, `n`, `j`) and does not allocate any auxiliary data structures that scale with the input size.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### Pitfall: Greedily extending the third increasing segment with a max-prefix-sum, but forgetting that the mandatory part of each segment must always be included

A very natural mistake when writing this solution is to treat **all** of the third increasing run as optional and let the "best prefix sum" logic decide how much to include — including the *first* rising step after the valley.

Look closely at the order of operations in Step 3:

```python
valley = i - 1
total += nums[i]      # <-- the first rise after the valley is MANDATORY
i += 1

best, running = 0, 0
while i < n and nums[i - 1] < nums[i]:   # <-- only the REST is optional
    running += nums[i]
    i += 1
    best = max(best, running)
total += best
```

The element `nums[valley + 1]` (the first element of the third rise) is added **unconditionally**, *before* the max-prefix-sum loop begins. Only the elements *beyond* that mandatory rise are treated as an optional extension via `best`.

#### Why the naive version is wrong

A tempting but **incorrect** rewrite folds the first rise into the optional loop:

```python
# WRONG: makes the first rising step optional too
best, running = 0, 0
while i < n and nums[i - 1] < nums[i]:
    running += nums[i]
    i += 1
    best = max(best, running)
total += best   # `best` could be 0, dropping the mandatory third segment!
```

Because `best` is seeded at `0`, if every element of the third rise is negative (e.g. the run is `-5, -3` after a valley of `-7`), `best` stays `0`. The code would then **discard the entire third segment**, producing a subarray that is *not even trionic* — it would only have an up-down shape. The structural requirement `q < r` (third segment has at least two elements) would be silently violated.

Concretely, for `nums = [3, 1, -5, -3]`:
- Increasing: `3` is not a real rise on its own — but consider `[1, 3, ... ]` style inputs. With negative valleys/rises, the naive code can report a sum that corresponds to a *non-trionic* shape, or even leave `ans = -inf` incorrectly when a valid (but negative-sum) trionic subarray exists.

The mandatory `total += nums[i]` guarantees the third segment always contributes its required minimum two elements (the valley `nums[valley]` already accounted for plus `nums[valley+1]`), keeping the candidate structurally valid even when its sum is negative.

#### Same trap on the left side

The symmetric pitfall exists in Step 4. The left extension walks from `peak - 2` **downward**:

```python
for j in range(peak - 2, left - 1, -1):
    ...
```

It starts at `peak - 2`, **not** `peak - 1`, because `nums[peak - 1]` and `nums[peak]` were *already* included unconditionally in Step 2 (`total = nums[peak - 1] + nums[peak]`). A common off-by-one bug is to start the loop at `peak - 1` and double-count `nums[peak - 1]`, or to start at `peak` and double-count the peak itself.

### Solution

Keep the **mandatory minimum** of each segment separate from its **optional extension**:

1. **First rise:** always include `nums[peak - 1] + nums[peak]` (the two-element minimum).
2. **Descent:** always include every strictly-decreasing element (it is not optional — it is forced by the structure).
3. **Second rise:** always include the first step `nums[valley + 1]`; treat only the elements *after* it as an optional max-prefix extension.
4. **Left extension:** start the backward scan at `peak - 2` to avoid re-adding `nums[peak - 1]`.

A useful defensive check is to verify with an all-negative trionic input, e.g. `nums = [-1, -3, -2, -4, -3]` (or similar up-down-up shapes of negative numbers), where the correct answer must still be a finite negative sum — not `0` and not `-inf`. If your code returns `0`, you have made some part of a mandatory segment optional; if it returns `-inf`, you have over-pruned a valid candidate.

```python
# Correct: mandatory parts added directly, only true extensions are max-pruned
total = nums[peak - 1] + nums[peak]   # mandatory rise minimum
# ... add all descent elements (mandatory) ...
total += nums[valley + 1]             # mandatory first step of third rise
total += best_right_extension         # optional, defaults to 0
total += best_left_extension          # optional, defaults to 0
```

This separation is what guarantees every reported sum corresponds to a genuinely trionic (up-down-up) subarray while still maximizing the total via the optional tails.

