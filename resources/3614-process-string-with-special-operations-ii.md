# 3614. Process String with Special Operations II

> Source: https://algo.monster/liteproblems/3614

## Problem Statement

You are given a string `s` made up of lowercase English letters along with three special characters: `'*'`, `'#'`, and `'%'`. You are also given an integer `k`.

Your task is to build a new string called `result` by reading through `s` one character at a time, from left to right, and applying the following rules:

- If the current character is a **lowercase English letter**, append it to the end of `result`.
- If the current character is a `'*'`, **remove the last character** from `result` (if `result` is currently empty, nothing happens).
- If the current character is a `'#'`, **duplicate** the current `result` and append that copy to itself. For example, if `result` is `"ab"`, it becomes `"abab"`.
- If the current character is a `'%'`, **reverse** the current `result`. For example, if `result` is `"abc"`, it becomes `"cba"`.

After processing every character in `s`, you obtain the final `result` string.

Return the character located at index `k` (0-indexed) of this final `result` string. If `k` is **out of bounds** (that is, `k` is greater than or equal to the length of `result`), return the character `'.'` instead.

## Intuition

The most direct idea is to actually build the `result` string by simulating every operation. However, the `'#'` operation **doubles** the length of `result` each time it appears. If the string `s` contains many `'#'` characters, the final `result` could grow exponentially large — far too big to store in memory or build in practice.

The key observation is that we **don't need the entire `result` string**. We only need to know a **single character**: the one at index `k`. This suggests that instead of building the string forward, we can work **backwards** and trace where that single position `k` came from at each step.

To do this, we first compute only the **length** `m` of the final `result`, without storing the actual characters. We scan `s` from left to right and track how the length changes:

- A letter increases the length by `1`.
- A `'*'` decreases the length by `1` (but never below `0`).
- A `'#'` doubles the length (`m <<= 1`).
- A `'%'` leaves the length unchanged, since reversing doesn't add or remove characters.

Once we know `m`, we can immediately check if `k >= m`. If so, `k` is out of bounds and the answer is `'.'`.

Otherwise, we walk through `s` in **reverse**, undoing each operation while keeping track of where index `k` would map to in the smaller, earlier version of `result`:

- Reversing a `'*'`: the string was longer before the removal, so we increase `m` by `1`. The index `k` is unaffected.
- Reversing a `'#'`: before duplication, the string was half as long. We set `m //= 2`. If `k` currently points into the **second half** (`k >= m`), it really originated from the first half, so we subtract `m` from `k` to map it back.
- Reversing a `'%'`: since the string was reversed, the character at index `k` was originally at index `m - 1 - k`, so we update `k` accordingly.
- Reversing a letter: this letter occupied the **last position** of `result` at that moment. We decrease `m` by `1`, and if `k == m`, then this very letter is the character we're looking for — so we return it.

By following the position `k` backward through every operation, we pinpoint the exact letter that ends up at index `k` without ever constructing the huge `result` string. This turns a potentially exponential problem into one that runs in linear time relative to the length of `s`.

## Solution Approach

### Solution 1: Reverse Tracking

We first calculate the length `m` of the processed result string `result`. If `k >= m`, it indicates that `k` exceeds the valid indices of the result string, so we return `'.'`.

To compute `m`, we scan `s` from left to right and simulate only the length changes:

- If the character is `'*'`, we set `m = max(0, m - 1)` to ensure the length never goes below `0`.
- If the character is `'#'`, we double the length with `m <<= 1`.
- If the character is a letter (i.e., not `'%'`), we increase `m` by `1`.
- If the character is `'%'`, the length stays the same, so we do nothing.

After this pass, `m` holds the exact length of the final `result`. We check `if k >= m: return "."`.

Otherwise, we traverse the string `s` in **reverse order** and handle each character based on the following cases, updating `m` and `k` as we undo each operation:

1. If `s[i]` is `'*'`, we increase `m` by `1`. This restores the character that was removed, but the position `k` we are tracking is not affected.
2. If `s[i]` is `'#'`, we divide `m` by `2` with `m //= 2`. At this point, if `k >= m`, it means `k` was pointing into the duplicated second half, so we map it back to the first half by subtracting: `k -= m`.
3. If `s[i]` is `'%'`, the string was reversed at this step, so the character now at index `k` originally sat at index `m - 1 - k`. We update `k = m - 1 - k`.
4. Otherwise, `s[i]` is a letter. We decrease `m` by `1`, since this letter occupied the last slot of `result` at this moment. If `k == m`, we have located the `k`-th character, so we return `s[i]`.

By tracing the position `k` backward through every operation, we identify the original letter without building the enormous `result` string.

**Complexity Analysis:**

- **Time complexity:** `O(n)`, where `n` is the length of `s`. We make one forward pass to compute the length and one reverse pass to locate the character.
- **Space complexity:** `O(1)`. We only use a few integer variables (`m` and `k`) and never construct the actual `result` string.

## Example Walkthrough

Let's trace through a small example to see how reverse tracking works.

**Input:** `s = "ab#%*"`, `k = 2`

First, let's understand what the **forward simulation** would produce (for verification only):

| Step | Char | Operation | `result` |
|------|------|-----------|----------|
| 1 | `'a'` | append `'a'` | `"a"` |
| 2 | `'b'` | append `'b'` | `"ab"` |
| 3 | `'#'` | duplicate | `"abab"` |
| 4 | `'%'` | reverse | `"baba"` |
| 5 | `'*'` | remove last | `"bab"` |

So the final `result` is `"bab"`, and the character at index `k = 2` is `'b'`. Now let's confirm the algorithm reaches the same answer **without building this string**.

---

#### Phase 1: Compute the length `m` (forward pass)

We scan `s` left to right, tracking only the length:

| Char | Rule | `m` |
|------|------|-----|
| start | — | `0` |
| `'a'` | letter, `m + 1` | `1` |
| `'b'` | letter, `m + 1` | `2` |
| `'#'` | double, `m << 1` | `4` |
| `'%'` | reverse, unchanged | `4` |
| `'*'` | remove, `max(0, m - 1)` | `3` |

Final length `m = 3`. Since `k = 2` and `2 < 3`, `k` is **in bounds**, so we proceed.

---

#### Phase 2: Trace `k` backward (reverse pass)

We now walk `s` from right to left, undoing each operation. We start with `m = 3` and `k = 2`.

**Step 1 — `s[4] = '*'`** (undo a removal)
The string was *longer* before the removal, so restore: `m = m + 1 = 4`.
The tracked index `k` is unaffected → `k = 2`.

**Step 2 — `s[3] = '%'`** (undo a reverse)
Before reversal, index `k` sat at position `m - 1 - k`:
`k = 4 - 1 - 2 = 1`.
`m` stays `4` → `k = 1`.

**Step 3 — `s[2] = '#'`** (undo a duplication)
Before duplication the string was half as long: `m = m // 2 = 2`.
Is `k >= m`? `1 >= 2` is **false**, so `k` was already in the first half — no change.
→ `k = 1`.

**Step 4 — `s[1] = 'b'`** (undo a letter append)
This letter occupied the last slot, so `m = m - 1 = 1`.
Is `k == m`? `1 == 1` is **true** → this letter is our answer.

**Return `'b'`.** ✅

---

#### Why this matches

The forward simulation gave us `"bab"` with `'b'` at index `2`, and the backward trace pinpointed the exact same `'b'` (the letter `'b'` from `s[1]`) — all while never constructing a string larger than a few integer variables. This is the core benefit: even if `'#'` had appeared many times and blown `result` up to an enormous size, we still follow just **one** position back to its origin in `O(n)` time.

## Python Solution

```python
class Solution:
    def processStr(self, s: str, k: int) -> str:
        # Phase 1: compute the final length of the transformed string.
        length = 0
        for ch in s:
            if ch == "*":
                # Remove the last character; length cannot go below 0.
                length = max(0, length - 1)
            elif ch == "#":
                # Duplicate the entire string; length doubles.
                length <<= 1
            elif ch != "%":
                # A normal character is appended; length grows by 1.
                # '%' (reverse) leaves the length unchanged, so it's skipped here.
                length += 1

        # If k is beyond the final length, the index is invalid.
        if k >= length:
            return "."

        # Phase 2: walk the operations in reverse, mapping index k back
        # through each transformation to find the originating character.
        for ch in reversed(s):
            if ch == "*":
                # Undo a removal: the string was one character longer before.
                length += 1
            elif ch == "#":
                # Undo a duplication: the original half had length // 2.
                length //= 2
                # If k fell in the duplicated (second) half, shift it back
                # into the corresponding index of the first half.
                if k >= length:
                    k -= length
            elif ch == "%":
                # Undo a reversal: index k maps to its mirrored position.
                k = length - 1 - k
            else:
                # Undo an append: this character occupied the last slot.
                length -= 1
                # If k points exactly to that last slot, it's the answer.
                if k == length:
                    return ch
```

## Time and Space Complexity

**Time Complexity: `O(n)`**

The algorithm consists of two sequential passes over the string `s`, where `n` is the length of `s`:

1. **First pass (forward):** The loop iterates through each character `c` in `s` to compute the final length `m`. For each character, only constant-time operations are performed (comparisons, addition, subtraction, bit shift `m <<= 1`). This pass takes `O(n)` time.

2. **Second pass (reverse):** The loop iterates through each character in `reversed(s)`, again performing only constant-time operations (comparisons, addition, subtraction, integer division `m //= 2`, and the assignment `k = m - 1 - k`). This pass also takes `O(n)` time.

Since the two passes are sequential, the total time complexity is `O(n) + O(n) = O(n)`.

**Space Complexity: `O(1)`**

The algorithm only uses a constant amount of extra space, regardless of the input size:

- The variable `m` stores an integer count.
- The variable `k` stores the index being tracked.
- The loop variable `c` holds a single character at a time.

Note that `reversed(s)` returns an iterator rather than creating a new copy of the string, so it does not consume additional `O(n)` space. Therefore, the auxiliary space complexity is `O(1)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### Pitfall 1: Forgetting to clamp the length at `0` when processing `'*'`

A very common mistake is to write `length -= 1` instead of `length = max(0, length - 1)` when handling the `'*'` operation in **Phase 1**.

The problem explicitly states that if `result` is empty, a `'*'` does nothing. If you blindly decrement, the length can become **negative**, which then corrupts every later computation. For example, with `s = "*#a"`:

- Correct: `*` → `length = max(0, 0 - 1) = 0`; `#` → `0 << 1 = 0`; `a` → `1`. Final length `1`.
- Wrong: `*` → `length = -1`; `#` → `-1 << 1 = -2`; `a` → `-1`. The length is garbage, and the `k >= length` check breaks entirely.

**Solution:** Always clamp the length when undoing a deletion in the forward pass:

```python
if ch == "*":
    length = max(0, length - 1)   # never let length drop below 0
```

---

### Pitfall 2: Mishandling the `'#'` (duplicate) mapping in reverse

When undoing a `'#'`, the order of operations matters. You must **first** halve the length, and **then** compare `k` against the new (halved) length to decide whether `k` landed in the duplicated second half.

```python
elif ch == "#":
    length //= 2          # halve FIRST
    if k >= length:       # THEN test against the new length
        k -= length
```

A frequent error is testing `k` against the *old* (full) length, or subtracting before halving. Both produce an incorrect mapping. Remember: after duplication the string is `first + first`. An index `k` in `[length, 2*length)` corresponds to position `k - length` in the original half, so the subtraction must use the **halved** length.

---

### Pitfall 3: Off-by-one in the reverse (`'%'`) mapping

When undoing a reversal, the mirrored index is `length - 1 - k`, **not** `length - k`:

```python
elif ch == "%":
    k = length - 1 - k
```

Because indices are 0-based, the first element (index `0`) and the last element (index `length - 1`) swap places. Using `length - k` shifts everything by one and silently returns the wrong character (or even an out-of-range index). Note also that `length` here is the *current* length at this step — since `'%'` does not change the length, no length update is needed, but the mapping must use the length that is correct **at that moment** in the reverse walk.

---

### Pitfall 4: Building the actual `result` string

The most tempting but fatal approach is to literally simulate the operations and construct `result`, then index into it:

```python
# DON'T DO THIS
result = []
for ch in s:
    if ch == "#":
        result = result + result   # length doubles every '#'
    ...
return result[k]
```

Each `'#'` **doubles** the string, so with just 60 `'#'` characters the string would exceed `2^60` characters — far beyond available memory, causing a `MemoryError` or timeout.

**Solution:** Never materialize `result`. Track only the integer `length` and map the index `k` backward through the operations, as the reverse-tracking approach does. This keeps the solution at `O(n)` time and `O(1)` extra space regardless of how large the conceptual `result` grows.

