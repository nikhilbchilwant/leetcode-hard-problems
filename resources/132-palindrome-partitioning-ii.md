# 132. Palindrome Partitioning II

> Source: https://algo.monster/liteproblems/132

## Problem Statement

Given a string `s`, you need to partition it into substrings where every substring is a palindrome. The goal is to find the **minimum number of cuts** needed to achieve such a partitioning.

A palindrome is a string that reads the same forwards and backwards (like "aba" or "racecar"). A partition means dividing the string into consecutive non-overlapping substrings.

For example:
- If `s = "aab"`, you can partition it as `["aa", "b"]` with 1 cut (between the second 'a' and 'b')
- If `s = "aba"`, the entire string is already a palindrome, so 0 cuts are needed

The problem asks you to return the minimum number of cuts required to ensure every resulting substring is a palindrome.

## Intuition

To find the minimum cuts, we need to think about this problem systematically. Every cut we make creates a new partition point, and we want to minimize these cuts while ensuring all resulting substrings are palindromes.

The key insight is that this is an optimization problem that can be solved using dynamic programming. We need to break it down into smaller subproblems.

First, we realize that to efficiently determine where to make cuts, we need to know which substrings of `s` are palindromes. If we keep checking whether each substring is a palindrome repeatedly, it would be inefficient. So, we can precompute this information for all possible substrings using a 2D table `g[i][j]`, where `g[i][j]` tells us if the substring from index `i` to `j` is a palindrome.

Next, we think about how to find the minimum cuts. For any position `i` in the string, we ask: "What's the minimum number of cuts needed to partition `s[0...i]` into palindromes?" 

To answer this, we consider all possible positions `j` where we could make the last cut before position `i`. If the substring `s[j...i]` is a palindrome, then we can make a cut at position `j-1` and the minimum cuts for `s[0...i]` would be the minimum cuts for `s[0...j-1]` plus one additional cut.

However, there's a special case: if the entire substring `s[0...i]` is itself a palindrome, then we don't need any cuts at all for this portion.

By building up the solution from smaller substrings to larger ones, we can efficiently compute the minimum cuts needed for the entire string. The answer will be the minimum cuts needed for the complete string `s[0...n-1]`.

## Solution Approach

The implementation consists of two main phases: preprocessing to identify palindromes and dynamic programming to find minimum cuts.

**Phase 1: Palindrome Preprocessing**

We create a 2D boolean array `g[i][j]` where `g[i][j] = True` if substring `s[i...j]` is a palindrome. We fill this table from bottom-right to top-left:

```python
g = [[True] * n for _ in range(n)]
for i in range(n - 1, -1, -1):
    for j in range(i + 1, n):
        g[i][j] = s[i] == s[j] and g[i + 1][j - 1]
```

The logic here is:
- All single characters are palindromes (diagonal elements are `True` by default)
- For substring `s[i...j]`, it's a palindrome if `s[i] == s[j]` AND the inner substring `s[i+1...j-1]` is also a palindrome
- We iterate backwards on `i` to ensure `g[i+1][j-1]` is computed before `g[i][j]`

**Phase 2: Dynamic Programming for Minimum Cuts**

We define `f[i]` as the minimum cuts needed for substring `s[0...i]`. Initially, we set `f[i] = i` (worst case: cut between every character).

```python
f = list(range(n))
for i in range(1, n):
    for j in range(i + 1):
        if g[j][i]:
            f[i] = min(f[i], 1 + f[j - 1] if j else 0)
```

For each position `i`, we check all possible last palindrome segments `s[j...i]`:
- If `g[j][i]` is `True` (substring `s[j...i]` is a palindrome), we have two cases:
  - If `j = 0`: The entire `s[0...i]` is a palindrome, so `f[i] = 0` (no cuts needed)
  - If `j > 0`: We need `f[j-1] + 1` cuts (cuts for `s[0...j-1]` plus one cut at position `j-1`)
- We take the minimum among all valid options

The state transition equation is:
$$f[i] = \min_{0 \leq j \leq i} \begin{cases} f[j-1] + 1, & \text{if } g[j][i] = \text{True} \\ 0, & \text{if } g[0][i] = \text{True} \end{cases}$$

The final answer is `f[n-1]`, representing the minimum cuts for the entire string.

## Example Walkthrough

Let's walk through the solution with `s = "aab"`.

**Step 1: Palindrome Preprocessing**

We create a 3×3 table `g[i][j]` to identify all palindromic substrings:

Initial state (all single characters are palindromes):
```
    0   1   2
0 [True, ?, ?]
1 [  -,True, ?]
2 [  -,  -,True]
```

Now we fill from bottom-right to top-left:
- `g[1][2]`: Is "ab" a palindrome? `s[1] == s[2]`? ('a' == 'b'?) No → `g[1][2] = False`
- `g[0][1]`: Is "aa" a palindrome? `s[0] == s[1]`? ('a' == 'a'?) Yes, and `g[1][0]` (empty substring) is True → `g[0][1] = True`
- `g[0][2]`: Is "aab" a palindrome? `s[0] == s[2]`? ('a' == 'b'?) No → `g[0][2] = False`

Final palindrome table:
```
    0    1     2
0 [True, True, False]  // "a", "aa", "aab"
1 [  -,  True, False]  // "a", "ab"
2 [  -,   -,   True]   // "b"
```

**Step 2: Dynamic Programming for Minimum Cuts**

Initialize `f = [0, 1, 2]` (worst case: cut after each character).

For `i = 1` (substring "aa"):
- Check `j = 0`: Is `g[0][1]` (substring "aa") a palindrome? Yes!
  - Since `j = 0`, the entire "aa" is a palindrome → `f[1] = 0`
- Check `j = 1`: Is `g[1][1]` (substring "a") a palindrome? Yes!
  - `f[1] = min(0, f[0] + 1) = min(0, 1) = 0`
- Result: `f[1] = 0`

For `i = 2` (substring "aab"):
- Check `j = 0`: Is `g[0][2]` (substring "aab") a palindrome? No, skip.
- Check `j = 1`: Is `g[1][2]` (substring "ab") a palindrome? No, skip.
- Check `j = 2`: Is `g[2][2]` (substring "b") a palindrome? Yes!
  - `f[2] = min(2, f[1] + 1) = min(2, 0 + 1) = 1`
- Result: `f[2] = 1`

Final `f = [0, 0, 1]`

**Answer: 1 cut is needed** (partition "aab" into ["aa", "b"] with a cut between position 1 and 2).

## Python Solution

```python
class Solution:
    def minCut(self, s: str) -> int:
        n = len(s)
      
        # Build a 2D table to store palindrome information
        # is_palindrome[i][j] = True if s[i:j+1] is a palindrome
        is_palindrome = [[True] * n for _ in range(n)]
      
        # Fill the palindrome table using dynamic programming
        # Start from the end of string and work backwards
        for start in range(n - 1, -1, -1):
            for end in range(start + 1, n):
                # A substring is palindrome if:
                # 1. First and last characters match
                # 2. The inner substring is also a palindrome (or length <= 2)
                is_palindrome[start][end] = (s[start] == s[end] and 
                                             is_palindrome[start + 1][end - 1])
      
        # min_cuts[i] represents minimum cuts needed for s[0:i+1]
        # Initialize with worst case: i cuts for string of length i+1
        min_cuts = list(range(n))
      
        # Calculate minimum cuts for each position
        for i in range(1, n):
            # Check all possible last palindrome ending at position i
            for j in range(i + 1):
                # If s[j:i+1] is a palindrome
                if is_palindrome[j][i]:
                    if j == 0:
                        # If entire s[0:i+1] is palindrome, no cuts needed
                        min_cuts[i] = 0
                    else:
                        # Otherwise, it's cuts for s[0:j] plus one more cut
                        min_cuts[i] = min(min_cuts[i], 1 + min_cuts[j - 1])
      
        # Return minimum cuts for entire string
        return min_cuts[-1]
```

## Time and Space Complexity

The time complexity is `O(n^2)`, where `n` is the length of the string `s`. This is because:
- The first nested loop (building the palindrome table `g`) iterates through all pairs `(i, j)` where `i` goes from `n-1` to `0` and `j` goes from `i+1` to `n`, resulting in `O(n^2)` operations.
- The second nested loop (computing minimum cuts in array `f`) has an outer loop from `1` to `n` and an inner loop from `0` to `i+1`, which also gives us `O(n^2)` operations in total.
- Since both parts are `O(n^2)` and run sequentially, the overall time complexity remains `O(n^2)`.

The space complexity is `O(n^2)`, where `n` is the length of the string `s`. This is because:
- The 2D boolean array `g` of size `n × n` is used to store whether each substring `s[i:j+1]` is a palindrome, requiring `O(n^2)` space.
- The 1D array `f` of size `n` is used to store the minimum number of cuts needed for each prefix, requiring `O(n)` space.
- Since `O(n^2) + O(n) = O(n^2)`, the overall space complexity is `O(n^2)`.

## Common Pitfalls

### 1. Incorrect Palindrome Table Initialization for Length-2 Substrings

**The Pitfall:**
When checking if a substring is a palindrome, the condition `is_palindrome[start][end] = (s[start] == s[end] and is_palindrome[start + 1][end - 1])` can fail for length-2 substrings. When `end = start + 1`, accessing `is_palindrome[start + 1][end - 1]` means accessing `is_palindrome[start + 1][start]`, which represents an invalid range where the end index is before the start index.

While the code initializes all values to `True`, relying on this for invalid ranges is implicit and can lead to confusion or bugs if the initialization changes.

**Example of the issue:**
For string "ab" where start=0, end=1:
- We check `is_palindrome[1][0]` which doesn't represent a valid substring
- It works only because we initialized everything to `True`

**Solution:**
Explicitly handle base cases for substrings of length 1 and 2:

```python
for start in range(n - 1, -1, -1):
    for end in range(start + 1, n):
        if end - start == 1:
            # Length-2 substring: only check if characters match
            is_palindrome[start][end] = (s[start] == s[end])
        else:
            # Length > 2: check both ends and inner substring
            is_palindrome[start][end] = (s[start] == s[end] and 
                                         is_palindrome[start + 1][end - 1])
```

### 2. Optimization Opportunity: Early Termination

**The Pitfall:**
The current implementation always checks all possible partitions even when we've already found that no cuts are needed (when `min_cuts[i] = 0`). This wastes computational cycles.

**Solution:**
Add a break statement when we find the entire substring from start is a palindrome:

```python
for i in range(1, n):
    for j in range(i + 1):
        if is_palindrome[j][i]:
            if j == 0:
                min_cuts[i] = 0
                break  # No need to check other partitions
            else:
                min_cuts[i] = min(min_cuts[i], 1 + min_cuts[j - 1])
```

### 3. Memory Optimization Consideration

**The Pitfall:**
The palindrome table uses O(n²) space. Storing only the upper triangle, or switching to a dictionary keyed by `(start, end)`, still keeps O(n²) entries (and a dictionary uses more memory per entry than a list of booleans), so neither changes the bound.

**Solution:**
To get O(n) extra space, drop the table and expand around every center instead. Each palindrome `s[left..right]` found during the expansion directly updates the cut count for the prefix that ends at `right`:

```python
def minCut(self, s: str) -> int:
    n = len(s)
    cuts = [i - 1 for i in range(n + 1)]  # cuts[k] = min cuts for s[:k]; cuts[0] = -1
    for center in range(n):
        for left, right in ((center, center), (center, center + 1)):
            while left >= 0 and right < n and s[left] == s[right]:
                cuts[right + 1] = min(cuts[right + 1], cuts[left] + 1)
                left -= 1
                right += 1
    return cuts[n]
```

This keeps the O(n²) running time and uses only the O(n) `cuts` array.

