# 3859. Count Subarrays With K Distinct Integers

> Source: https://algo.monster/liteproblems/3859

## Problem Statement

You are given an integer array `nums` along with two integers `k` and `m`.

Your task is to count how many **subarrays** of `nums` satisfy **both** of the following conditions at the same time:

1. **Exactly `k` distinct integers**: The subarray must contain precisely `k` different integer values — no more and no fewer.
2. **Each distinct integer appears at least `m` times**: Every one of those `k` distinct integers must occur **at least** `m` times within the subarray.

A **subarray** is a contiguous (and non-empty) sequence of elements taken from the array. The order of elements is preserved, and the elements must be adjacent in the original array.

Return an integer representing the total number of subarrays that meet both requirements.

For example, suppose `k = 2` and `m = 2`. A valid subarray would need to contain exactly two distinct values, and each of those two values would need to appear two or more times inside that subarray. Any subarray that contains a third distinct value, or where one of the distinct values appears fewer than `m` times, would not be counted.

## Intuition

The condition "**exactly** `k` distinct integers" is tricky to handle directly, because "exactly" forces us to track an upper bound that is hard to maintain with a single [sliding window](/problems/sliding_window_maximum). A classic technique for dealing with "exactly `k`" counting problems is to convert it into a difference of two "at least" style counts:

`exactly k = (at least k) - (at least k + 1)`

So if we define a helper `f(lim)` that counts subarrays which have **at least `lim` distinct integers** (with each of those integers appearing at least `m` times), then our final answer is simply:

`f(k) - f(k + 1)`

This works because every subarray with **at least `k`** distinct valid integers is counted in `f(k)`, and subtracting `f(k + 1)` removes those subarrays with **at least `k + 1`** distinct valid integers, leaving exactly those with precisely `k`.

Now the question becomes: how do we count subarrays with **at least `lim`** distinct integers, where each appears at least `m` times? Here we use a **[sliding window](/problems/sliding_window_maximum) with two pointers**. The key observation is **monotonicity**: if a window `[l, r]` already satisfies "has at least `lim` distinct integers each appearing at least `m` times", then **extending it to the left** (making it bigger) can only keep that property — adding more elements never reduces the distinct count nor the frequency of any value. This monotonic behavior is what makes the sliding window valid.

To make tracking efficient, we maintain:
- A counter `cnt` that records how many times each value currently appears in the window.
- A variable `t` that counts how many **distinct values have reached frequency `m`**. We increment `t` exactly when a value's count hits `m`, and decrement it when a value's count drops back to `m - 1`.

For each right endpoint, we keep shrinking the left side (advancing `l`) **as long as** the window still meets the requirement — meaning it has at least `lim` distinct values (`len(cnt) >= lim`) and at least `k` of them have reached frequency `m` (`t >= k`). Once we can no longer shrink, the value `l` tells us how many valid subarrays end at the current right endpoint: every starting position from `0` to `l - 1` forms a valid subarray. So we add `l` to the answer.

Putting it together, summing `l` over all right endpoints in `f(lim)` gives the total count of qualifying subarrays for that bound, and the difference `f(k) - f(k + 1)` cleanly isolates the "exactly `k`" case.

**Learn more about [Sliding Window](/problems/sliding_window_maximum) patterns.**

## Solution Approach

We implement the idea of `exactly k = (at least k) - (at least k + 1)` by writing a helper function `f(lim)` and returning `f(k) - f(k + 1)`.

**Data structures used:**

- A hash map / `Counter` named `cnt` to store the frequency of each value inside the current [sliding window](/problems/sliding_window_maximum).
- An integer `t` that tracks how many **distinct values have a frequency of at least `m`** in the current window.
- Two pointers, `l` (left boundary) and the loop variable `x` (driven by the right boundary), forming a **sliding window**.

**Walking through `f(lim)`:**

1. Initialize `cnt` as an empty counter, `t = 0`, `ans = 0`, and `l = 0`.

2. Iterate over the array, treating each element `x` as the right end of the window:
   - Increase its count: `cnt[x] += 1`.
   - If this increment causes `cnt[x]` to exactly reach `m`, it means a new value has just satisfied the "appears at least `m` times" requirement, so increment `t`: 
     ```python
     if cnt[x] == m:
         t += 1
     ```

3. **Shrink the window from the left** while it still satisfies the requirement, i.e. while `len(cnt) >= lim and t >= k`:
   - Take the leftmost element `y = nums[l]` and remove it: `cnt[y] -= 1`.
   - If removing it dropped `cnt[y]` from `m` down to `m - 1`, then this value no longer qualifies, so decrement `t`:
     ```python
     if cnt[y] == m - 1:
         t -= 1
     ```
   - If the count reached `0`, remove the key entirely so that `len(cnt)` correctly reflects the number of distinct values:
     ```python
     if cnt[y] == 0:
         cnt.pop(y)
     ```
   - Advance the left pointer: `l += 1`.

4. After the shrinking loop, the pointer `l` marks the boundary: for the current right end, every starting index from `0` to `l - 1` produces a subarray that satisfies "at least `lim` distinct values, with at least `k` of them appearing `m` or more times". So we add `l` to the answer:
   ```python
   ans += l
   ```

5. Return `ans` once the loop finishes.

**Why the shrinking condition uses two checks:**

- `len(cnt) >= lim` enforces the "at least `lim` distinct integers" requirement.
- `t >= k` enforces that **at least `k`** of those distinct integers have already appeared at least `m` times.

Together, while both hold, the window (and any prefix extension to its left) is valid, so accumulating `l` counts all valid subarrays ending at the current position.

**Final combination:**

```python
return f(k) - f(k + 1)
```

`f(k)` counts subarrays with **at least `k`** qualifying distinct integers, and `f(k + 1)` counts those with **at least `k + 1`**. Subtracting removes the over-counted cases and leaves exactly those subarrays with **precisely `k`** distinct integers, each appearing at least `m` times.

**Complexity:**

- **Time:** `O(n)` per call to `f`, since each element is added once and removed at most once by the two pointers; with two calls the total is `O(n)`.
- **Space:** `O(n)` in the worst case for the `Counter` holding distinct values in a window.

## Example Walkthrough

```markdown

Let's trace through a small example to see how the solution approach works.

**Input:** `nums = [1, 1, 2, 2]`, `k = 2`, `m = 2`

We want subarrays with **exactly 2 distinct integers**, where **each appears at least 2 times**.

By inspection, the only subarray that qualifies is `[1, 1, 2, 2]` (indices 0–3): it has exactly two distinct values (`1` and `2`), and each appears twice. So the expected answer is **1**.

Our formula is `answer = f(2) - f(3)`, where `f(lim)` counts subarrays with **at least `lim`** distinct values, each appearing at least `m = 2` times (with at least `k = 2` of them reaching frequency `m`).

---

#### Computing `f(2)` (lim = 2, k = 2, m = 2)

We slide the right end `x` across the array, maintaining `cnt`, `t` (values that reached frequency `m`), and left pointer `l`. We shrink while `len(cnt) >= 2 and t >= 2`, then add `l` to `ans`.

| step | x (value) | cnt after add | t | shrink? (`len(cnt)>=2 and t>=2`) | l after shrink | ans += l | ans |
|------|-----------|---------------|---|----------------------------------|----------------|----------|-----|
| 1    | 1 (idx 0) | {1:1}         | 0 | no (t=0)                          | 0              | +0       | 0   |
| 2    | 1 (idx 1) | {1:2}         | 1 | no (len=1)                        | 0              | +0       | 0   |
| 3    | 2 (idx 2) | {1:2, 2:1}    | 1 | no (t=1)                          | 0              | +0       | 0   |
| 4    | 2 (idx 3) | {1:2, 2:2}    | 2 | yes → see below                   | 1              | +1       | 1   |

**Detail of the shrink at step 4:** Window `[0,3]` has `len(cnt)=2`, `t=2`, so we shrink:
- Remove `nums[0]=1` → `cnt={1:1, 2:2}`. Since `cnt[1]` dropped to `m-1=1`, decrement `t` → `t=1`. Advance `l=1`.
- Now `t=1`, condition fails, stop shrinking.

So `f(2) = 1`.

---

#### Computing `f(3)` (lim = 3, k = 2, m = 2)

Here we need **at least 3 distinct values** in the window (`len(cnt) >= 3`). But `nums` only ever contains the values `1` and `2`, so `len(cnt)` never reaches 3. The shrink condition `len(cnt) >= 3` is never satisfied, so `l` stays at `0` throughout, and `ans` stays `0`.

So `f(3) = 0`.

---

#### Final Answer

```
answer = f(2) - f(3) = 1 - 0 = 1
```

This matches our manual count: only `[1, 1, 2, 2]` qualifies. ✅

**Key takeaways from the trace:**
- `t` only became `2` once both values reached frequency `m`, which is precisely when a valid window first formed.
- Adding `l` after shrinking counts all valid starting positions (`0 .. l-1`) for each right endpoint.
- The subtraction `f(k) - f(k+1)` isolates the "exactly `k`" case — here `f(3)=0` because no third distinct value exists.
```

## Python Solution

```python
from collections import Counter


class Solution:
    def countSubarrays(self, nums: list[int], k: int, m: int) -> int:
        def count_at_least(distinct_lower_bound: int) -> int:
            # freq: frequency of each value inside the current window [left, right]
            freq = Counter()
            # qualified: number of distinct values whose frequency reaches exactly m
            qualified = 0
            total = 0      # accumulated count of valid subarrays
            left = 0       # left boundary of the sliding window

            for value in nums:
                # Expand the window by including the current value
                freq[value] += 1
                if freq[value] == m:
                    qualified += 1

                # Shrink the window from the left while it still satisfies:
                #   - distinct element count >= distinct_lower_bound
                #   - number of values reaching frequency m >= k
                while len(freq) >= distinct_lower_bound and qualified >= k:
                    left_value = nums[left]
                    freq[left_value] -= 1
                    # Crossing the threshold m downward removes a qualified value
                    if freq[left_value] == m - 1:
                        qualified -= 1
                    # Remove the value entirely if its frequency hits zero
                    if freq[left_value] == 0:
                        freq.pop(left_value)
                    left += 1

                # For every subarray ending at the current index, all left
                # boundaries in [0, left) form valid windows, so add `left`.
                total += left

            return total

        # Subarrays with distinct count exactly k (and the frequency-m condition)
        # equal those with distinct count >= k minus those with distinct count >= k + 1.
        return count_at_least(k) - count_at_least(k + 1)
```

## Time and Space Complexity

**Time Complexity:** `O(n)`

The core logic is the helper function `f(lim)`, which uses a sliding window (two-pointer) technique over the array `nums` of length `n`.

- The outer `for` loop iterates over each element `x` in `nums`, contributing `O(n)` iterations.
- The inner `while` loop advances the left pointer `l`. Although it is a nested loop, the pointer `l` only ever moves forward and never resets, so across the entire execution of `f`, the inner loop runs at most `n` times in total (amortized analysis).
- Each `Counter` operation (`cnt[x] += 1`, `cnt[y] -= 1`, `cnt.pop(y)`, `len(cnt)`) is `O(1)` on average.

Therefore, a single call to `f` costs `O(n)`. The final result calls `f(k)` and `f(k + 1)`, which is just two `O(n)` invocations, yielding an overall time complexity of `2 * O(n) = O(n)`.

**Space Complexity:** `O(n)`

The dominant space usage comes from the `Counter` (`cnt`), which stores the frequency of distinct elements currently inside the window. In the worst case (e.g., all elements distinct), the window may contain up to `n` distinct keys, requiring `O(n)` space. The remaining variables (`t`, `ans`, `l`, `lim`) use only `O(1)` additional space.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### Pitfall 1: Misunderstanding what the `inclusion-exclusion` actually subtracts

The most subtle and dangerous pitfall in this solution is assuming that `count_at_least(k) - count_at_least(k + 1)` cleanly isolates "exactly `k` distinct integers, each appearing at least `m` times." It does **not** directly do that, because the helper function `count_at_least` mixes **two different conditions** inside one shrinking loop:

```python
while len(freq) >= distinct_lower_bound and qualified >= k:
```

Notice that `distinct_lower_bound` (the parameter `lim`) controls the **distinct-count** dimension, while `qualified >= k` is hardcoded to `k` and controls the **frequency-m** dimension. When you call `count_at_least(k + 1)`, only the distinct-count bound increases — the `qualified >= k` check stays fixed at `k`.

**Why this matters:** The subtraction is only valid if both helper calls hold the frequency condition (`qualified >= k`) constant while varying *only* the distinct-count threshold. If a reader "improves" the code by parameterizing `qualified >= distinct_lower_bound` instead of `qualified >= k`, the inclusion-exclusion silently breaks, and the function returns wrong counts that are hard to debug because the code still runs without errors.

**Solution:** Keep the two dimensions decoupled. The `qualified >= k` comparison must **always** reference the original `k`, never the loop parameter. Add a clarifying assertion or comment:

```python
def count_at_least(distinct_lower_bound: int) -> int:
    ...
    while len(freq) >= distinct_lower_bound and qualified >= k:
        #                                       ^^^^^^^^^^^^^
        # This MUST stay `k`, not `distinct_lower_bound`.
        # Inclusion-exclusion only varies the distinct-count dimension.
        ...
```

---

### Pitfall 2: Updating `qualified` with the wrong threshold check on shrink

When shrinking the window, it is tempting to write the symmetric decrement as:

```python
if freq[left_value] < m:   # WRONG
    qualified -= 1
```

This over-decrements. Once a value's frequency falls below `m`, every subsequent removal of that same value would keep triggering `freq[left_value] < m` and decrement `qualified` repeatedly, corrupting the counter.

**Solution:** Decrement **only at the exact moment** the count crosses the threshold downward — i.e., when it becomes exactly `m - 1`:

```python
freq[left_value] -= 1
if freq[left_value] == m - 1:   # crosses threshold exactly once
    qualified -= 1
```

This mirrors the expansion side, where you increment only when `freq[value] == m` (exactly reaching the threshold), guaranteeing each value contributes at most `+1`/`-1` to `qualified`.

---

### Pitfall 3: Forgetting to pop zero-count keys, breaking `len(freq)`

The distinct-count condition relies on `len(freq)` accurately reflecting the number of distinct values currently in the window. If you decrement a count to `0` but leave the key in the `Counter`, `len(freq)` over-counts distinct values, and the shrink loop terminates too late, inflating the result.

```python
freq[left_value] -= 1
# ... must remove dead keys:
if freq[left_value] == 0:
    freq.pop(left_value)
```

**Solution:** Always `pop` keys whose count reaches `0`. Note that a plain `Counter` does **not** auto-remove zero entries, so this step is mandatory. If you were tracking distinct count with a separate integer instead of relying on `len(freq)`, you would decrement that integer here instead.

---

### Pitfall 4: Edge cases with `m` or `k` larger than feasible

If `m` is large enough that no value can appear `m` times, or `k` exceeds the number of distinct values in `nums`, the answer should be `0`. The sliding window handles this naturally (`qualified` never reaches `k`, so `left` never advances and `total` stays `0`), but reviewers sometimes add premature guards that introduce bugs.

**Solution:** Trust the window mechanics — no special-casing is needed. Just verify with a quick test such as `nums=[1,2,3], k=2, m=5`, which should return `0`.

---

### Pitfall 5: Integer-type result on very large inputs

The count of valid subarrays can be on the order of `n²/2`, which for large `n` exceeds 32-bit integer range in languages like Java or C++. Python integers are unbounded so this is a non-issue here, but porting this logic to another language without using a 64-bit type (`long`) will silently overflow.

**Solution:** When translating to a fixed-width-integer language, declare `total` and the return value as 64-bit integers.

