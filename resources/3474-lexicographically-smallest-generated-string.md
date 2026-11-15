# 3474. Lexicographically Smallest Generated String

> Source: https://algo.monster/liteproblems/3474

## Problem Statement

You are given two strings, `str1` and `str2`, of lengths `n` and `m`, respectively.

A string `word` of length `n + m - 1` is said to be **generated** by `str1` and `str2` if it satisfies the following conditions for **every** index `0 <= i <= n - 1`:

- If `str1[i] == 'T'`, then the **substring** of `word` of length `m` starting at index `i` must be **equal** to `str2`. In other words, `word[i..(i + m - 1)] == str2`.
- If `str1[i] == 'F'`, then the **substring** of `word` of length `m` starting at index `i` must **not be equal** to `str2`. In other words, `word[i..(i + m - 1)] != str2`.

Your task is to return the **lexicographically smallest** possible string that can be **generated** by `str1` and `str2`. If no valid string can be generated, return an empty string `""`.

In short, the string `str1` acts as a sequence of constraints: each `'T'` forces a specific window of `word` to match `str2` exactly, while each `'F'` forbids the corresponding window from matching `str2`. Among all strings `word` that satisfy all of these constraints simultaneously, you must find the one that is smallest in lexicographic order.

## Intuition

The key observation is that the two types of constraints have very different natures, and we should handle them in a way that keeps the result as small as possible.

First, let's think about what makes a string lexicographically small. Since `'a'` is the smallest character, our ideal goal is to fill `word` with as many `'a'`s as possible. So we start by assuming every position is `'a'`, and only change a position when a constraint forces us to.

Now consider the `'T'` constraints. These are **hard requirements**: whenever `str1[i] == 'T'`, the window `word[i..(i + m - 1)]` is completely determined—it **must** equal `t`. There is no freedom here. So we go through all the `'T'` positions and stamp `t` onto the corresponding windows of `word`. We also mark these positions as **fixed**, because they can never be altered later.

What if two different `'T'` windows overlap and demand different characters at the same position? Then the requirements contradict each other, and no valid string exists, so we return `""` right away. This is why we keep a `fixed` array: it lets us detect a conflict the moment two `'T'` constraints disagree on the same index.

After all `'T'` constraints are placed, every remaining unfixed position is still `'a'`, which is exactly what we want for the smallest string.

Next come the `'F'` constraints. These are the opposite: whenever `str1[i] == 'F'`, the window `word[i..(i + m - 1)]` must **not** equal `t`. We only have a problem if this window currently happens to equal `t`. If it already differs from `t`, the constraint is satisfied and we do nothing.

When a window does match `t` and we must break it, we want to make the **minimal and smallest** change. To keep the string lexicographically small, we should change a position as far to the **right** as possible (changing later positions affects the order less than changing earlier ones). And to keep that change minimal, we set the character to `'b'`, the smallest character that differs from `'a'`. But we can only modify a position that is **not fixed**, since fixed positions belong to a `'T'` window and changing them would break a hard requirement.

So we scan the window from right to left looking for the first unfixed position and turn it into `'b'`. If every position in the window is fixed, then the window is entirely locked to equal `t` by the `'T'` constraints while an `'F'` constraint demands it differ—an impossible situation, so we return `""`.

By always preferring `'a'`, only escalating to `'b'` when absolutely necessary, and always breaking ties at the rightmost free position, we naturally build the lexicographically smallest valid string in a single [greedy](/problems/greedy_intro) pass.

**Learn more about [Greedy](/problems/greedy_intro) patterns.**

## Solution Approach

We use a **[greedy](/problems/greedy_intro)** strategy. Let `str1` be `s` and `str2` be `t`, with `n = len(s)` and `m = len(t)`.

**Data structures.**

- An array `ans` of length `n + m - 1`, initialized so that every character is `'a'`. This holds the string we are building, and starting with all `'a'`s keeps it as small as possible by default.
- A boolean array `fixed` of length `n + m - 1`, initialized to all `False`. Position `k` is `True` once it has been locked by a `'T'` constraint and must never be changed again.

**Step 1: Apply the `'T'` constraints.**

We iterate over `s` with index `i`. We skip any position where `s[i] != 'T'`. When `s[i] == 'T'`, the window starting at `i` must equal `t`, so we copy `t` into `ans`:

- For each character `c = t[j]` (with `j` from `0` to `m - 1`), let `k = i + j`.
- If `fixed[k]` is already `True` but `ans[k] != c`, two `'T'` windows disagree on the same index, so generation is impossible and we return `""`.
- Otherwise we set `ans[k] = c` and `fixed[k] = True`.

After this pass, all hard requirements are stamped in, and every still-unfixed position remains `'a'`.

**Step 2: Apply the `'F'` constraints.**

We iterate over `s` again with index `i`, skipping any position where `s[i] != 'F'`. When `s[i] == 'F'`, we check the current window:

- If `"".join(ans[i : i + m]) != t`, the constraint is already satisfied, so we move on.
- If the window currently equals `t`, we must break it. To keep `ans` lexicographically smallest, we scan the window from **right to left**, i.e. `j` from `i + m - 1` down to `i`, and find the first position with `fixed[j] == False`. We set `ans[j] = "b"` and stop. Choosing the rightmost free slot and the smallest differing character `'b'` makes the minimal possible impact on the order.
- If the loop finds no unfixed position (the `for ... else` branch runs), the entire window is locked to `t` by `'T'` constraints while an `'F'` constraint forbids it. This is a contradiction, so we return `""`.

**Step 3: Build the answer.**

Finally, we join `ans` into a string and return it.

**Complexity.** Each `'T'` and `'F'` constraint touches at most `m` positions, and there are at most `n` constraints, so the time complexity is `O(n * m)`. The space complexity is `O(n + m)` for the `ans` and `fixed` arrays.

## Example Walkthrough

Let's trace through a small example to see the greedy approach in action.

**Input:** `str1 = "TFT"`, `str2 = "ab"`

So `n = 3`, `m = 2`, and the answer length is `n + m - 1 = 4`.

**Initial setup:**

```
index:   0    1    2    3
ans  :  'a'  'a'  'a'  'a'
fixed:   F    F    F    F
```

---

**Step 1: Apply the `'T'` constraints.**

We scan `str1 = "TFT"`.

- **`i = 0`, `s[0] == 'T'`** → window `ans[0..1]` must equal `"ab"`.
  - `j = 0`: `k = 0`, `c = 'a'`. `fixed[0]` is `False`, so set `ans[0] = 'a'`, `fixed[0] = True`.
  - `j = 1`: `k = 1`, `c = 'b'`. `fixed[1]` is `False`, so set `ans[1] = 'b'`, `fixed[1] = True`.

  ```
  index:   0    1    2    3
  ans  :  'a'  'b'  'a'  'a'
  fixed:   T    T    F    F
  ```

- **`i = 1`, `s[1] == 'F'`** → skip for now (handled in Step 2).

- **`i = 2`, `s[2] == 'T'`** → window `ans[2..3]` must equal `"ab"`.
  - `j = 0`: `k = 2`, `c = 'a'`. `fixed[2]` is `False`, so set `ans[2] = 'a'`, `fixed[2] = True`.
  - `j = 1`: `k = 3`, `c = 'b'`. `fixed[3]` is `False`, so set `ans[3] = 'b'`, `fixed[3] = True`.

  ```
  index:   0    1    2    3
  ans  :  'a'  'b'  'a'  'b'
  fixed:   T    T    T    T
  ```

No conflicts arose, so all hard requirements are satisfied.

---

**Step 2: Apply the `'F'` constraints.**

We scan `str1 = "TFT"` again, only acting on `'F'`.

- **`i = 1`, `s[1] == 'F'`** → window `ans[1..2]` must **not** equal `"ab"`.
  - Current window: `ans[1] + ans[2] = 'b' + 'a' = "ba"`.
  - Since `"ba" != "ab"`, the constraint is **already satisfied** — do nothing.

The `'F'` constraint happened to be broken for free by the surrounding `'T'` windows.

---

**Step 3: Build the answer.**

```
index:   0    1    2    3
ans  :  'a'  'b'  'a'  'b'
```

Joining gives **`"abab"`**.

---

**Verification:**

- `str1[0] = 'T'` → `word[0..1] = "ab" == "ab"` ✓
- `str1[1] = 'F'` → `word[1..2] = "ba" != "ab"` ✓
- `str1[2] = 'T'` → `word[2..3] = "ab" == "ab"` ✓

All constraints hold, and because every free position was kept at the smallest possible character, `"abab"` is the lexicographically smallest valid string.

---

**A note on the rightmost-`'b'` rule:** In this example the `'F'` window was already broken, so we never needed to escalate. To see that mechanism, imagine an `'F'` window like `ans[1..2] = "ab"` where only index `2` is unfixed. We would scan right-to-left, find index `2` free first, and set `ans[2] = 'b'`, turning the window into `"ab" → "ab"`... wait, into the **rightmost** free slot to minimize lexicographic impact — choosing a later position to change keeps the earlier (more significant) characters as small as possible.

## Python Solution

```python
class Solution:
    def generateString(self, s: str, t: str) -> str:
        s_len, t_len = len(s), len(t)
        total_len = s_len + t_len - 1

        # Result buffer, default-filled with 'a'.
        result = ["a"] * total_len
        # Marks positions whose character is forced by a 'T' constraint.
        is_fixed = [False] * total_len

        # Pass 1: enforce every 'T' window to exactly match t.
        for start, flag in enumerate(s):
            if flag != "T":
                continue
            for offset, ch in enumerate(t):
                pos = start + offset
                # Conflict between two overlapping 'T' windows.
                if is_fixed[pos] and result[pos] != ch:
                    return ""
                result[pos] = ch
                is_fixed[pos] = True

        # Pass 2: enforce every 'F' window to differ from t.
        for start, flag in enumerate(s):
            if flag != "F":
                continue
            # Already different — nothing to do.
            if "".join(result[start : start + t_len]) != t:
                continue
            # Try to flip a free (non-fixed) position to break the match.
            for pos in range(start + t_len - 1, start - 1, -1):
                if not is_fixed[pos]:
                    result[pos] = "b"
                    break
            else:
                # Every position is fixed; cannot break the match.
                return ""

        return "".join(result)
```

## Time and Space Complexity

### Time Complexity

The time complexity is `O(n × m)`, where `n = len(s)` and `m = len(t)`.

The analysis proceeds as follows:

- **Initialization**: Creating the `ans` and `fixed` arrays takes `O(n + m)` time, since their length is `n + m - 1`.

- **First loop (handling `"T"` positions)**: The outer loop iterates over all `n` characters of `s`. For each character equal to `"T"`, the inner loop iterates over all `m` characters of `t`, performing constant-time comparisons and assignments. In the worst case (all characters of `s` are `"T"`), this contributes `O(n × m)`.

- **Second loop (handling `"F"` positions)**: The outer loop again iterates over all `n` characters of `s`. For each character equal to `"F"`, the operation `"".join(ans[i : i + m]) != t` constructs and compares a substring of length `m`, costing `O(m)`. The subsequent inner loop scans at most `m` positions, also `O(m)`. In the worst case (all characters of `s` are `"F"`), this contributes `O(n × m)`.

- **Final join**: Joining `ans` into the result string takes `O(n + m)`.

Combining all parts, the dominant term is `O(n × m)`, so the overall time complexity is `O(n × m)`.

### Space Complexity

The space complexity is `O(n + m)`.

The analysis proceeds as follows:

- The `ans` array and the `fixed` array each have length `n + m - 1`, requiring `O(n + m)` space.

- In the second loop, the temporary substring created by `"".join(ans[i : i + m])` uses `O(m)` extra space, which does not exceed `O(n + m)`.

- The final returned string requires `O(n + m)` space.

Therefore, the overall space complexity is `O(n + m)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### Pitfall: Flipping the wrong slot in a `'F'` window breaks lexicographic minimality (or correctness)

The trickiest part of this problem is **Step 2**, where an `'F'` window currently equals `t` and must be broken by changing one free character. Two related mistakes commonly appear here:

**1. Scanning the window left-to-right instead of right-to-left.**

A natural-but-wrong instinct is to walk the window from `start` toward `start + t_len - 1` and flip the *first* free slot found:

```python
# WRONG: flips the leftmost free position
for pos in range(start, start + t_len):
    if not is_fixed[pos]:
        result[pos] = "b"   # changes a more significant character
        break
```

Changing an **earlier** index has a much larger impact on lexicographic order than changing a later one. For example, flipping index 2 from `'a'` to `'b'` produces `aaba...`, whereas flipping index 5 produces `aaaaab...`, and `aaaaab` < `aaba`. To keep the answer smallest, you **must** scan from the right end of the window backward and flip the **rightmost** free position. The provided code does this correctly:

```python
for pos in range(start + t_len - 1, start - 1, -1):
    if not is_fixed[pos]:
        result[pos] = "b"
        break
```

**2. Forgetting that a flipped position is *not* fixed and can be re-flipped or overwritten.**

When we set `result[pos] = "b"`, we deliberately do **not** mark it as `is_fixed`. This is correct: only `'T'` constraints create hard locks. However, a subtle danger is assuming the single `'b'` flip permanently satisfies *all* later `'F'` windows that overlap this position. It does not necessarily — but the algorithm re-checks each `'F'` window independently with the up-to-date `result`, so this stays correct. The pitfall is *adding* premature `is_fixed[pos] = True` after a flip "to be safe," which would wrongly block later constraints and can produce spurious `""` returns.

### Solution / Defensive Reasoning

- **Always scan right-to-left** when breaking an `'F'` match, and use `'b'` (the smallest character that differs from `'a'`) only on a free slot.
- **Never mark a `'b'`-flipped slot as fixed.** Reserve `is_fixed` exclusively for `'T'`-forced characters.
- **Trust the `for ... else` contradiction check.** If no free slot exists in an `'F'` window, every position is `'T'`-locked to `t`, which is a genuine contradiction → return `""`.

### A deeper correctness subtlety worth verifying

There is one further pitfall hiding in the greedy claim itself. When we flip the **rightmost** free position to `'b'`, we assume this single change is enough to make `word[start..]` differ from `t`. This holds **only because** that position is free, meaning the corresponding character of `t` at that offset is whatever — but `result[pos]` was `'a'` (or possibly already `'b'` from an earlier flip). Two cases:

1. If `result[pos] == 'a'` and `t[pos - start] == 'a'`, flipping to `'b'` correctly creates a mismatch.
2. If `t[pos - start] == 'b'` already, then setting `result[pos] = 'b'` does **not** break the match!

This is the most easily-missed bug. Consider `t = "ab"` and a free position aligned with `t`'s `'b'`: the window is `"ab"` (because the `'a'` slot held `'a'` and the `'b'` slot held... wait, a free slot defaults to `'a'`, so the window could only equal `"ab"` if some `'T'` forced the `'b'`). In practice, for the window to *currently equal* `t`, any free (non-`'T'`) slot must hold `'a'`, which means `t` at that offset must also be `'a'` — so flipping to `'b'` is guaranteed to differ. The greedy is therefore safe, but a robust implementation should make this explicit:

```python
for pos in range(start + t_len - 1, start - 1, -1):
    if not is_fixed[pos]:
        # Pick the smallest char that actually differs from t at this offset.
        result[pos] = "b" if t[pos - start] != "b" else "a"
        break
else:
    return ""
```

Adding `result[pos] = "b" if t[pos - start] != "b" else "a"` documents the invariant and guards against any future change to the default-fill character, eliminating the silent failure where a flip leaves the window unchanged.

