# 1163. Last Substring in Lexicographical Order

> Source: https://algo.monster/liteproblems/1163

## Problem Statement

Given a string `s`, you need to find and return the last substring of `s` when all possible substrings are arranged in lexicographical (dictionary) order.

For example, if `s = "abab"`, all possible substrings are: `"a"`, `"ab"`, `"aba"`, `"abab"`, `"b"`, `"ba"`, `"bab"`, `"a"`, `"ab"`, `"b"`. When sorted lexicographically, they become: `"a"`, `"a"`, `"ab"`, `"ab"`, `"aba"`, `"abab"`, `"b"`, `"b"`, `"ba"`, `"bab"`. The last substring in this sorted order is `"bab"`.

The key insight is that among all substrings starting from a position `i`, the suffix `s[i:]` (substring from position `i` to the end) will always be the lexicographically largest one starting from that position. This is because any shorter substring starting from `i` would be a prefix of `s[i:]`, and prefixes are always lexicographically smaller than the full string when the string has more characters.

Therefore, the problem reduces to finding which suffix of the string is lexicographically largest. The solution uses a two-pointer technique to efficiently compare different suffixes and identify the one with the maximum lexicographical value.

## Intuition

The first observation is that we don't need to consider all possible substrings. Why? Because for any starting position `i`, the longest substring starting from `i` (which is the suffix `s[i:]`) will always be lexicographically larger than any shorter substring starting from the same position. This is because when comparing strings lexicographically, if one string is a prefix of another, the longer string is always greater.

So our problem simplifies to: find the lexicographically largest suffix of the string.

A naive approach would be to compare all suffixes pairwise, but that would be inefficient. Instead, we can use a two-pointer technique to eliminate multiple candidates at once.

The key insight is when we're comparing two suffixes starting at positions `i` and `j`. If we find that at some offset `k`, `s[i+k] < s[j+k]` (the first differing character), then not only is suffix `s[i:]` smaller than `s[j:]`, but also all suffixes `s[i+1:]`, `s[i+2:]`, ..., `s[i+k:]` are smaller than their corresponding counterparts starting from `j`. Why? Because they all share the same prefix relationship that made them smaller.

For example, if `s = "abcabd"` and we're comparing suffix starting at `i=0` ("abcabd") with suffix starting at `j=3` ("abd"), we find they differ at position `k=2` where `s[0+2]='c'` > `s[3+2]='d'`. This tells us that not only "abcabd" > "abd", but also "bcabd" > "bd" and "cabd" > "d".

This observation allows us to skip multiple positions at once: when `s[i+k] < s[j+k]`, we can directly jump `i` to `i+k+1`, skipping all intermediate positions. Similarly, when `s[i+k] > s[j+k]`, we can jump `j` to `j+k+1`.

This clever jumping strategy ensures we examine each character at most a constant number of times, leading to an efficient linear-time algorithm.

**Learn more about [Two Pointers](/problems/two_pointers_intro) patterns.**

## Solution Approach

We implement a two-pointer algorithm to find the lexicographically largest suffix. Here's how the algorithm works:

**Initialization:**
- Pointer `i = 0`: tracks the starting position of the current best (lexicographically largest) suffix candidate
- Pointer `j = 1`: tracks the starting position of the suffix we're currently comparing against
- Variable `k = 0`: tracks the offset for character-by-character comparison

**Main Loop:**
The algorithm continues while `j + k < len(s)`, ensuring we don't go out of bounds.

At each iteration, we compare characters `s[i + k]` and `s[j + k]`:

1. **Case 1: `s[i + k] == s[j + k]`**
   - The characters match, so we increment `k` to compare the next pair of characters
   - We continue extending the comparison to find where the two suffixes differ

2. **Case 2: `s[i + k] < s[j + k]`**
   - The suffix starting at `j` is lexicographically larger
   - We update `i = i + k + 1`, effectively skipping all suffixes `s[i:]`, `s[i+1:]`, ..., `s[i+k:]`
   - Why can we skip these? Because if `s[i:i+k]` equals `s[j:j+k]` but `s[i+k] < s[j+k]`, then any suffix starting in the range `[i, i+k]` will be smaller than the corresponding suffix starting from `j`
   - Reset `k = 0` for the next comparison
   - If `i >= j`, we set `j = i + 1` to ensure `j` is always ahead of `i`

3. **Case 3: `s[i + k] > s[j + k]`**
   - The suffix starting at `i` is lexicographically larger
   - We update `j = j + k + 1`, skipping all suffixes from `j` to `j+k`
   - Reset `k = 0` for the next comparison

**Return Value:**
After the loop completes, `i` points to the starting position of the lexicographically largest suffix, so we return `s[i:]`.

**Time Complexity:** O(n) where n is the length of the string. Although it might seem like we're doing nested comparisons, each character is examined at most twice due to the jumping strategy.

**Space Complexity:** O(1) as we only use a constant amount of extra space for the pointers and counter.

## Example Walkthrough

Let's trace through the algorithm with `s = "abcab"`.

We need to find the lexicographically largest suffix among: "abcab", "bcab", "cab", "ab", "b".

**Initial State:**
- `i = 0` (candidate: "abcab")
- `j = 1` (comparing with: "bcab")  
- `k = 0`

**Iteration 1:**
- Compare `s[0+0] = 'a'` with `s[1+0] = 'b'`
- Since `'a' < 'b'`, suffix "bcab" is larger than "abcab"
- Update: `i = 0 + 0 + 1 = 1`
- Since `i >= j`, set `j = i + 1 = 2`
- Reset `k = 0`
- New state: `i = 1` (candidate: "bcab"), `j = 2` (comparing with: "cab")

**Iteration 2:**
- Compare `s[1+0] = 'b'` with `s[2+0] = 'c'`
- Since `'b' < 'c'`, suffix "cab" is larger than "bcab"
- Update: `i = 1 + 0 + 1 = 2`
- Since `i >= j`, set `j = i + 1 = 3`
- Reset `k = 0`
- New state: `i = 2` (candidate: "cab"), `j = 3` (comparing with: "ab")

**Iteration 3:**
- Compare `s[2+0] = 'c'` with `s[3+0] = 'a'`
- Since `'c' > 'a'`, suffix "cab" is larger than "ab"
- Update: `j = 3 + 0 + 1 = 4`
- Reset `k = 0`
- New state: `i = 2` (candidate: "cab"), `j = 4` (comparing with: "b")

**Iteration 4:**
- Compare `s[2+0] = 'c'` with `s[4+0] = 'b'`
- Since `'c' > 'b'`, suffix "cab" is larger than "b"
- Update: `j = 4 + 0 + 1 = 5`
- Reset `k = 0`
- New state: `i = 2`, `j = 5`

**Termination:**
- `j + k = 5 + 0 = 5` which equals `len(s)`, so the loop ends
- Return `s[2:] = "cab"`

The algorithm correctly identifies "cab" as the lexicographically largest suffix. When we consider all substrings of "abcab" and sort them, "cab" would indeed be the last substring in the sorted order.

## Python Solution

```python
class Solution:
    def lastSubstring(self, s: str) -> str:
        """
        Find the lexicographically largest substring of the given string.
        The largest substring will always be a suffix of the original string.
      
        Uses a two-pointer approach with comparison offset to efficiently find
        the starting position of the lexicographically largest suffix.
      
        Args:
            s: Input string
          
        Returns:
            The lexicographically largest substring (suffix)
        """
        # Initialize two pointers for comparing potential starting positions
        left_start = 0   # First candidate starting position
        right_start = 1  # Second candidate starting position
        offset = 0       # Current comparison offset from both starting positions
      
        # Continue until we've examined all necessary characters
        while right_start + offset < len(s):
            # Case 1: Characters at current offset are equal
            if s[left_start + offset] == s[right_start + offset]:
                # Move to next character for comparison
                offset += 1
              
            # Case 2: Left substring is smaller at current offset
            elif s[left_start + offset] < s[right_start + offset]:
                # Skip the entire left substring and its compared portion
                # The new left_start jumps past all compared characters
                left_start = left_start + offset + 1
                offset = 0  # Reset comparison offset
              
                # Ensure right_start is always ahead of left_start
                if left_start >= right_start:
                    right_start = left_start + 1
                  
            # Case 3: Right substring is smaller at current offset
            else:
                # Skip the entire right substring and its compared portion
                right_start = right_start + offset + 1
                offset = 0  # Reset comparison offset
      
        # Return the suffix starting from the optimal position
        return s[left_start:]
```

## Time and Space Complexity

**Time Complexity:** `O(n)`, where `n` is the length of string `s`.

The algorithm uses a two-pointer approach with variables `i` and `j` to find the lexicographically largest substring. While it may appear that the nested nature of the while loop could lead to `O(n²)` complexity, careful analysis reveals that each character is visited at most twice:

- The pointer `j` moves forward through the string, and when `s[i + k] < s[j + k]`, the pointer `i` jumps ahead by `k + 1` positions
- When `s[i + k] > s[j + k]`, the pointer `j` jumps ahead by `k + 1` positions
- The key insight is that when either pointer jumps, it skips over positions that have already been compared (the `k` positions), ensuring that no position is redundantly processed

Since each position in the string is examined at most a constant number of times, the total time complexity is `O(n)`.

**Space Complexity:** `O(1)`

The algorithm only uses a fixed number of integer variables (`i`, `j`, and `k`) to track positions and comparison lengths. No additional data structures are created that scale with the input size. The returned substring `s[i:]` is not counted toward space complexity as it's the output. Therefore, the space complexity is constant, `O(1)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### Pitfall 1: Incorrect Pointer Update Logic
**The Problem:** A common mistake is updating the pointers incorrectly when characters don't match. Developers often write:
```python
# WRONG: Simply incrementing by 1
if s[i + k] < s[j + k]:
    i = i + 1  # This misses the optimization!
```

**Why It's Wrong:** This approach doesn't use the information we've already gathered. If we've already compared `k` characters and found them equal, we know that any suffix starting between positions `i` and `i+k` will be lexicographically smaller than the corresponding suffix starting from `j`.

**The Solution:** Always jump by `k + 1` to skip all the positions we've already implicitly compared:
```python
if s[i + k] < s[j + k]:
    i = i + k + 1  # Correct: Skip all positions from i to i+k
```

### Pitfall 2: Forgetting to Maintain the Invariant `j > i`
**The Problem:** After updating `i`, it's possible that `i` becomes greater than or equal to `j`:
```python
# WRONG: Not adjusting j after updating i
if s[i + k] < s[j + k]:
    i = i + k + 1
    k = 0
    # Missing: j might now be <= i!
```

**Why It's Wrong:** The algorithm relies on comparing two different positions. If `j <= i`, we'd either be comparing a position with itself or comparing in the wrong order, leading to incorrect results or infinite loops.

**The Solution:** Always ensure `j` stays ahead of `i`:
```python
if s[i + k] < s[j + k]:
    i = i + k + 1
    k = 0
    if i >= j:
        j = i + 1  # Maintain j > i invariant
```

### Pitfall 3: Not Resetting the Offset `k`
**The Problem:** Forgetting to reset `k` to 0 after updating either `i` or `j`:
```python
# WRONG: k not reset
if s[i + k] < s[j + k]:
    i = i + k + 1
    # Missing: k = 0
```

**Why It's Wrong:** The offset `k` represents how many characters we've compared from the current starting positions. When we change either starting position, we need to begin a fresh comparison from offset 0.

**The Solution:** Always reset `k` when changing starting positions:
```python
if s[i + k] < s[j + k]:
    i = i + k + 1
    k = 0  # Reset for new comparison
```

### Pitfall 4: Misunderstanding Why This is O(n)
**The Problem:** Developers might think this is O(n²) because of the nested nature of the comparisons (the while loop with internal offset increments).

**Why It Seems Wrong:** At first glance, it appears we might compare each character multiple times in different suffix comparisons.

**The Clarification:** Each character is examined at most twice throughout the entire algorithm:
- Once as part of the suffix starting at `i`
- Once as part of the suffix starting at `j`

When we update `i` or `j` by jumping `k + 1` positions, we're effectively marking those skipped positions as "processed" and never return to them. This jumping mechanism ensures linear time complexity.

