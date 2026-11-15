# 76. Minimum Window Substring

> Source: https://algo.monster/liteproblems/76

## Problem Statement

You are given two strings `s` and `t` with lengths `m` and `n` respectively. Your task is to find the **minimum window substring** in `s` that contains all characters from `t`, including duplicates.

A window substring means a contiguous sequence of characters within `s`. The window must contain every character that appears in `t`, with at least the same frequency. For example, if `t = "AAB"`, then the window must contain at least two 'A's and one 'B'.

The goal is to find the shortest such substring. The problem guarantees that the minimum window is unique. If no valid window exists that contains all characters from `t`, return an empty string `""`.

**Key requirements:**
- The window must be a substring (contiguous characters) of `s`
- Every character in `t` must appear in the window with at least the same frequency as in `t`
- You need to find the minimum length window that satisfies these conditions
- If no valid window exists, return an empty string

**Example scenarios:**
- If `s = "ADOBECODEBANC"` and `t = "ABC"`, the minimum window would be `"BANC"` which contains all characters A, B, and C
- If `s = "a"` and `t = "aa"`, no valid window exists since `s` doesn't have enough 'a's, so return `""`
- If `s = "ab"` and `t = "b"`, the minimum window would be `"b"`

## Intuition

The key insight is that we need to find a substring that contains all characters from `t`, and we want this substring to be as short as possible. A brute force approach would check every possible substring of `s`, but that would be inefficient with time complexity `O(n^3)`.

Instead, we can use a **[sliding window](/problems/sliding_window_maximum)** technique. Think of it like a flexible window that can expand and contract as we move through the string `s`. The idea is:

1. **Expand the window** by moving the right pointer to include more characters until we have all characters from `t`
2. Once we have a valid window, **contract it** by moving the left pointer to make it as small as possible while still containing all required characters
3. Record the minimum valid window found so far

To efficiently track whether our current window contains all characters from `t`, we use two key components:
- A frequency map `need` that tells us how many of each character we need from `t`
- A frequency map `window` that tracks what we currently have in our [sliding window](/problems/sliding_window_maximum)
- A counter `cnt` that tracks how many characters we've satisfied (matching the required frequency)

The clever part is knowing when we have a valid window. Instead of checking the entire frequency map each time, we maintain a count `cnt` of how many characters from `t` are satisfied in our current window. When `cnt` equals the length of `t`, we know we have all required characters.

The algorithm naturally finds the minimum window because:
- We only expand when we don't have enough characters
- We always try to contract when we have a valid window
- We keep track of the smallest valid window seen throughout the process

This approach ensures we examine each character at most twice (once when expanding, once when contracting), giving us an efficient `O(m + n)` solution where `m` and `n` are the lengths of strings `s` and `t`.

**Learn more about [Sliding Window](/problems/sliding_window_maximum) patterns.**


### Watch it run

Step through it below. The window expands until it covers every needed letter, then contracts from the left to find the smallest valid one.

<iframe src="/widgets/min-window-substring.html" width="100%" height="580" frameborder="0" style="border:1px solid #e5e7eb;border-radius:12px;width:100%;min-height:160px;touch-action:pan-y;" title="Minimum Window Substring walkthrough"></iframe>

## Solution Approach

We implement the [sliding window](/problems/sliding_window_maximum) technique using two pointers and frequency counting:

**1. Initialize Data Structures:**
- `need = Counter(t)`: Count the frequency of each character in string `t`. This tells us what we need to find.
- `window = Counter()`: Track the frequency of characters in our current [sliding window](/problems/sliding_window_maximum).
- `cnt = 0`: Count how many characters from `t` are satisfied in the current window.
- `l = 0`: Left pointer of the sliding window.
- `k = -1, mi = inf`: Track the starting position and length of the minimum window found so far.

**2. Expand the Window (Right Pointer Movement):**
Iterate through string `s` using index `r` and character `c`:
- Add the current character to the window: `window[c] += 1`
- Check if this character contributes to our requirement: If `need[c] >= window[c]`, it means this character instance is "necessary" (we still need more of this character or just got enough), so increment `cnt += 1`

**3. Contract the Window (Left Pointer Movement):**
When `cnt == len(t)`, we have a valid window containing all characters from `t`:
- First, try to update the minimum window: If `r - l + 1 < mi`, we found a shorter window, so update `mi = r - l + 1` and `k = l`
- Then, try to shrink the window from the left:
  - Check if removing `s[l]` would break the validity: If `need[s[l]] >= window[s[l]]`, removing this character would make the window invalid, so decrement `cnt -= 1`
  - Remove the leftmost character: `window[s[l]] -= 1`
  - Move the left pointer: `l += 1`
- Continue shrinking while the window remains valid (`cnt == len(t)`)

**4. Return the Result:**
- If `k < 0`, no valid window was found, return empty string `""`
- Otherwise, return the substring `s[k : k + mi]`

**Key Algorithm Details:**

The condition `need[c] >= window[c]` is crucial for correctness:
- When expanding: It ensures we only increment `cnt` for characters that are actually needed
- When contracting: It tells us when removing a character would make the window invalid

The algorithm maintains the invariant that `cnt` represents the total count of characters from `t` that are satisfied in the current window. This allows us to check window validity in `O(1)` time instead of comparing entire frequency maps.

**Time and Space Complexity:**
- Time: `O(m + n)` where `m = len(s)` and `n = len(t)`. Each character in `s` is visited at most twice.
- Space: `O(|Σ|)` where `|Σ|` is the size of the character set (constant for ASCII/Unicode).

## Example Walkthrough

<details>
<summary>Prefer text? Read the step-by-step walkthrough.</summary>

Let's walk through the algorithm with `s = "ADOBECODEBANC"` and `t = "ABC"`.

**Initial Setup:**
- `need = {'A': 1, 'B': 1, 'C': 1}` (we need 1 of each character)
- `window = {}` (empty window)
- `cnt = 0` (no characters satisfied yet)
- `l = 0` (left pointer at start)
- `k = -1, mi = inf` (no minimum window found yet)

**Expanding Phase (r = 0 to 5):**

| r | char | window | cnt | Action |
|---|------|--------|-----|--------|
| 0 | A | {'A': 1} | 1 | Add A, need[A]=1 >= window[A]=1, cnt++ |
| 1 | D | {'A': 1, 'D': 1} | 1 | Add D, D not needed |
| 2 | O | {'A': 1, 'D': 1, 'O': 1} | 1 | Add O, O not needed |
| 3 | B | {'A': 1, 'D': 1, 'O': 1, 'B': 1} | 2 | Add B, need[B]=1 >= window[B]=1, cnt++ |
| 4 | E | {'A': 1, 'D': 1, 'O': 1, 'B': 1, 'E': 1} | 2 | Add E, E not needed |
| 5 | C | {'A': 1, 'D': 1, 'O': 1, 'B': 1, 'E': 1, 'C': 1} | 3 | Add C, need[C]=1 >= window[C]=1, cnt++ |

**First Valid Window Found (cnt = 3):**
- Window is "ADOBEC" (indices 0-5), length = 6
- Update: `mi = 6, k = 0`

**Contracting Phase:**
Now we try to shrink from left while maintaining validity:

| l | Remove | window after removal | cnt | Can continue? |
|---|--------|---------------------|-----|---------------|
| 0 | A | {'D': 1, 'O': 1, 'B': 1, 'E': 1, 'C': 1} | 2 | No, A was needed |

Since removing 'A' breaks validity (cnt < 3), we stop contracting and continue expanding.

**Continue Expanding (r = 6 to 9):**

| r | char | window | cnt | Action |
|---|------|--------|-----|--------|
| 6 | O | {..., 'O': 2} | 2 | Add O, O not needed |
| 7 | D | {..., 'D': 2} | 2 | Add D, D not needed |
| 8 | E | {..., 'E': 2} | 2 | Add E, E not needed |
| 9 | B | {..., 'B': 2} | 2 | Add B, B already satisfied |

Still not valid (cnt = 2), keep expanding.

**r = 10 (char = 'A'):**
- Add A: `window['A'] = 1`, now `cnt = 3` again
- Current window is "DOBECODEBA" (indices 1-10), but we need to contract

**Second Contraction Phase:**
Window is valid again. Current substring length = 10, not better than mi = 6.

Try shrinking from left (l = 1):

| l | Remove | Effect |
|---|--------|--------|
| 1 | D | D not needed, can remove |
| 2 | O | O not needed, can remove |
| 3 | B | B is in `t`, but window[B]=2 > need[B]=1, so cnt stays 3; after removal window[B]=1 and the window is still valid |
| 4 | E | E not needed, can remove |
| 5 | C | C is needed: need[C]=1 >= window[C]=1, so cnt drops to 2 and the window becomes invalid |

After removing D, O, B, E, the window is "CODEBA" (indices 5-10), length = 6, which is not shorter than the current minimum, so `mi` stays 6. Removing C then ends this contraction with `l = 6`.

**Continue to r = 12 (char = 'C'):**
After more expansion and contraction, we eventually find:
- Window "BANC" (indices 9-12)
- Length = 4
- Update: `mi = 4, k = 9`

**Final Result:**
Return `s[9:13] = "BANC"`

The algorithm efficiently found the minimum window by expanding only when needed and contracting whenever possible, examining each character at most twice.


</details>

## Python Solution

```python
from collections import Counter
from math import inf

class Solution:
    def minWindow(self, s: str, t: str) -> str:
        # Count frequency of each character in target string t
        target_freq = Counter(t)
      
        # Track frequency of characters in current window
        window_freq = Counter()
      
        # Count of characters in window that match required frequency
        matched_chars = 0
      
        # Left pointer of sliding window
        left = 0
      
        # Track the start index and length of minimum window found
        min_start = -1
        min_length = inf
      
        # Expand window by moving right pointer
        for right, char in enumerate(s):
            # Add current character to window
            window_freq[char] += 1
          
            # Check if this character contributes to matching t
            if char in target_freq and window_freq[char] <= target_freq[char]:
                matched_chars += 1
          
            # Contract window when all characters from t are matched
            while matched_chars == len(t):
                # Update minimum window if current window is smaller
                current_window_size = right - left + 1
                if current_window_size < min_length:
                    min_length = current_window_size
                    min_start = left
              
                # Remove leftmost character from window
                left_char = s[left]
                if left_char in target_freq and window_freq[left_char] <= target_freq[left_char]:
                    matched_chars -= 1
                window_freq[left_char] -= 1
              
                # Move left pointer to contract window
                left += 1
      
        # Return empty string if no valid window found, otherwise return minimum window
        return "" if min_start == -1 else s[min_start : min_start + min_length]
```

## Time and Space Complexity

The time complexity is `O(m + n)`, where `m` is the length of string `s` and `n` is the length of string `t`. The algorithm initializes a `Counter` for string `t` which takes `O(n)` time. Then it uses a sliding window approach with two pointers traversing string `s` once, where each character is visited at most twice (once by the right pointer and once by the left pointer), resulting in `O(m)` operations. Therefore, the total time complexity is `O(m + n)`.

The space complexity is `O(|Σ|)`, where `|Σ|` represents the size of the character set. The algorithm uses two `Counter` objects (`need` and `window`) to store character frequencies. In the worst case, these counters will store all unique characters that appear in strings `s` and `t`. Since the problem deals with ASCII characters, the maximum size of the character set is `128`, making the space complexity `O(128)` which simplifies to `O(1)` constant space in terms of the character set size, or more precisely `O(|Σ|)` where `|Σ| = 128`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### Pitfall 1: Mismatched Comparison When Removing a Character

**The Problem:**
`matched_chars` must go up and down for exactly the same character copies. The solution uses `<=` in both places, but the two checks happen at different moments:
- When **adding**, the check runs *after* `window_freq[char] += 1`: the new copy counts if `window_freq[char] <= target_freq[char]`
- When **removing**, the check runs *before* `window_freq[left_char] -= 1`: the copy being removed counted if `window_freq[left_char] <= target_freq[left_char]`

In both cases the comparison looks at the count that includes the copy in question, so `<=` is correct for both. A tempting "fix" is to switch the removal check to a strict `<`:

```python
# WRONG: strict inequality when removing
left_char = s[left]
if left_char in target_freq and window_freq[left_char] < target_freq[left_char]:
    matched_chars -= 1
window_freq[left_char] -= 1
left += 1
```

**Why it fails:** For `s = "ADOBECODEBANC"` and `t = "ABC"`, the first valid window `"ADOBEC"` has `window_freq['A'] = 1` and `target_freq['A'] = 1`. Removing that `A` should make the window invalid, but `1 < 1` is false, so `matched_chars` stays at 3. The `while` loop keeps shrinking a window it wrongly believes is valid, `left` runs past `right`, and `s[left]` eventually raises an `IndexError`.

**The Solution:**
Compare against the count that still includes the character being added or removed:
- When adding (after incrementing): `window_freq[char] <= target_freq[char]`
- When removing (before decrementing): `window_freq[left_char] <= target_freq[left_char]`

If you decrement first, the removal check becomes `window_freq[left_char] < target_freq[left_char]`. What matters is that the comparison matches the order of the update.

### Pitfall 2: Using Dictionary Size Instead of Character Count

Another common mistake is using `len(need)` (number of unique characters) instead of `len(t)` (total character count) for the validity check. This would fail for cases where `t` has duplicate characters like `"AAB"` because it would consider the window valid as soon as it has at least one 'A' and one 'B', ignoring the frequency requirement.

