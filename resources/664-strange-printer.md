# 664. Strange Printer

> Source: https://algo.monster/liteproblems/664

## Problem Statement

You have a strange printer with two special properties:
1. The printer can only print a sequence of the same character each time (e.g., "aaa" or "bbb", but not "abc")
2. When printing, the printer can start and end at any position in the string, and the new characters will cover (overwrite) any existing characters at those positions

Given a string `s`, find the minimum number of printing operations needed to print the entire string.

For example, if you want to print "aba":
- One way: Print "aaa" first (1 operation), then print "b" at position 1 to get "aba" (2 operations total)
- This takes 2 operations minimum

The key insight is that when you print, you can strategically overwrite parts of previously printed characters to minimize the total number of operations. The printer can print any continuous segment of the same character and place it anywhere in the string, covering what was there before.

## Intuition

The key observation is that when we print a character at position `i`, we can extend that same print operation to cover any other position `j` where `s[i] == s[j]` without any additional cost. This means if the first and last characters of a substring are the same, we can handle them together in a single print operation.

Think about printing "aba". If we decide to print 'a' at position 0, we might as well extend that print to also cover position 2 (since both are 'a'), giving us "a_a" in one operation. Then we only need one more operation to fill in the 'b' at position 1.

This naturally leads to a range-based thinking: for any substring `s[i..j]`, we need to determine the minimum operations to print it. This screams [dynamic programming](/problems/dynamic_programming_intro) with intervals.

When looking at a substring `s[i..j]`:
- If `s[i] == s[j]`, we can "piggyback" the printing of `s[j]` onto the operation that prints `s[i]`. This means printing `s[i..j]` takes the same number of operations as printing `s[i..j-1]` (we get `s[j]` for free).
- If `s[i] != s[j]`, they must be printed in separate operations. We need to find the best way to split the substring into two parts. We try all possible split points `k` and choose the one that minimizes `f[i][k] + f[k+1][j]`.

The base case is straightforward: printing a single character always takes exactly 1 operation.

By building up from smaller substrings to larger ones (processing shorter ranges before longer ones), we ensure that when we need to look up `f[i][k]` or `f[k+1][j]`, these values have already been computed. This is why we iterate with `i` decreasing and `j` increasing - it guarantees we process all necessary subproblems first.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The solution uses a 2D [dynamic programming](/problems/dynamic_programming_intro) approach with a table `f[i][j]` representing the minimum operations needed to print substring `s[i..j]`.

**Initialization:**
- Create a 2D array `f` of size `n × n` where `n = len(s)`, initialized with infinity values
- The diagonal elements `f[i][i] = 1` since printing a single character always takes 1 operation

**Iteration Order:**
- Iterate `i` from `n-1` down to `0` (right to left)
- For each `i`, iterate `j` from `i+1` to `n-1` (left to right)
- This order ensures that when computing `f[i][j]`, all required subproblems are already solved

**Transition Logic:**
For each `f[i][j]`:
1. **Case 1: When `s[i] == s[j]`**
   - Set `f[i][j] = f[i][j-1]`
   - Since characters at both ends are the same, we can print them together in one operation
   - The cost is the same as printing `s[i..j-1]` since `s[j]` comes for free

2. **Case 2: When `s[i] != s[j]`**
   - Try all possible split points `k` where `i ≤ k < j`
   - For each split: `f[i][j] = min(f[i][j], f[i][k] + f[k+1][j])`
   - This represents printing `s[i..k]` first, then printing `s[k+1..j]` separately
   - Choose the split that gives the minimum total operations

**State Transition Formula:**
```
f[i][j] = {
    1,                                           if i = j
    f[i][j-1],                                   if s[i] = s[j]
    min(f[i][k] + f[k+1][j]) for k in [i,j),    if s[i] ≠ s[j]
}
```

**Final Answer:**
Return `f[0][n-1]`, which represents the minimum operations to print the entire string from index 0 to n-1.

The time complexity is `O(n³)` due to three nested loops (i, j, and k), and space complexity is `O(n²)` for the DP table.

## Example Walkthrough

Let's walk through the example string `s = "aba"` to understand how the dynamic programming solution works.

**Step 1: Initialize the DP table**
We create a 3×3 table `f[i][j]` and set diagonal values to 1:
```
    0   1   2
0 [ 1   ∞   ∞ ]
1 [ ∞   1   ∞ ]
2 [ ∞   ∞   1 ]
```

**Step 2: Process substring "ab" (i=0, j=1)**
- `s[0]='a'` and `s[1]='b'` are different
- We try split point k=0: `f[0][0] + f[1][1] = 1 + 1 = 2`
- So `f[0][1] = 2`

**Step 3: Process substring "ba" (i=1, j=2)**
- `s[1]='b'` and `s[2]='a'` are different
- We try split point k=1: `f[1][1] + f[2][2] = 1 + 1 = 2`
- So `f[1][2] = 2`

**Step 4: Process substring "aba" (i=0, j=2)**
- `s[0]='a'` and `s[2]='a'` are the same!
- According to our rule: `f[0][2] = f[0][1] = 2`
- This means we can print both 'a's in one operation, getting "a_a", then print 'b' separately

**Final DP table:**
```
    0   1   2
0 [ 1   2   2 ]
1 [ ∞   1   2 ]
2 [ ∞   ∞   1 ]
```

**Answer:** `f[0][2] = 2` operations minimum

**How the actual printing works:**
1. First operation: Print "aaa" (covers positions 0, 1, 2)
2. Second operation: Print "b" at position 1 (overwrites the middle 'a')
3. Result: "aba"

The key insight demonstrated here is that when `s[0]='a'` equals `s[2]='a'`, we can handle both in a single print operation, which is why `f[0][2] = f[0][1]` rather than trying to split the string.

## Python Solution

```python
class Solution:
    def strangePrinter(self, s: str) -> int:
        n = len(s)
      
        # dp[i][j] represents minimum number of turns to print substring s[i:j+1]
        dp = [[float('inf')] * n for _ in range(n)]
      
        # Iterate through all possible substrings from bottom-right to top-left
        for i in range(n - 1, -1, -1):
            # Single character needs only 1 turn to print
            dp[i][i] = 1
          
            # Check all substrings starting at position i
            for j in range(i + 1, n):
                if s[i] == s[j]:
                    # If first and last characters are same,
                    # we can print them together in the same turn
                    # So the cost is same as printing s[i:j]
                    dp[i][j] = dp[i][j - 1]
                else:
                    # If first and last characters are different,
                    # try splitting at each position k and find minimum
                    for k in range(i, j):
                        # Split into s[i:k+1] and s[k+1:j+1]
                        dp[i][j] = min(dp[i][j], dp[i][k] + dp[k + 1][j])
      
        # Return minimum turns needed to print entire string
        return dp[0][n - 1]
```

## Time and Space Complexity

The time complexity is `O(n^3)` where `n` is the length of string `s`. This is because:
- The outer loop iterates through all starting positions `i` from `n-1` to `0`, which is `O(n)`
- For each `i`, the middle loop iterates through all ending positions `j` from `i+1` to `n-1`, which is `O(n)` 
- For each pair `(i, j)` where `s[i] != s[j]`, the inner loop iterates through all possible split points `k` from `i` to `j-1`, which is `O(n)`
- Overall, we have three nested loops resulting in `O(n^3)` time complexity

The space complexity is `O(n^2)` where `n` is the length of string `s`. This is due to:
- The 2D DP table `f` of size `n × n` that stores the minimum number of turns needed to print each substring `s[i:j+1]`
- No additional significant space is used beyond this table

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect DP Table Initialization**
A common mistake is initializing the DP table with 0 or leaving it uninitialized, which leads to incorrect minimum calculations.

**Wrong approach:**
```python
dp = [[0] * n for _ in range(n)]  # Wrong: 0 initialization
```

**Correct approach:**
```python
dp = [[float('inf')] * n for _ in range(n)]  # Correct: infinity initialization
```

### 2. **Wrong Iteration Order**
Many people iterate in the wrong direction, causing access to unsolved subproblems.

**Wrong approach:**
```python
for i in range(n):
    for j in range(i + 1, n):  # Wrong: subproblems not yet solved
```

**Correct approach:**
```python
for i in range(n - 1, -1, -1):  # Right to left
    for j in range(i + 1, n):       # Left to right
```

### 3. **Mishandling the s[i] == s[j] Case**
A critical error is setting `dp[i][j] = dp[i+1][j-1]` when characters match, which is intuitive but incorrect.

**Wrong approach:**
```python
if s[i] == s[j]:
    dp[i][j] = dp[i+1][j-1]  # Wrong: doesn't account for the printing strategy
```

**Correct approach:**
```python
if s[i] == s[j]:
    dp[i][j] = dp[i][j-1]  # Correct: we can print s[j] together with s[i]
```

### 4. **Off-by-One Errors in Split Point Iteration**
Incorrectly setting the range for the split point k can miss optimal solutions or cause index errors.

**Wrong approach:**
```python
for k in range(i, j + 1):  # Wrong: k = j reads dp[j+1][j]
    dp[i][j] = min(dp[i][j], dp[i][k] + dp[k+1][j])
```

With `k = j`, the right part `s[j+1..j]` is empty. `dp[j+1][j]` is still infinity (or out of range when `j = n - 1`), so the extra split either adds nothing or raises an `IndexError`.

**Correct approach:**
```python
for k in range(i, j):  # Correct: k ranges from i to j-1
    dp[i][j] = min(dp[i][j], dp[i][k] + dp[k+1][j])
```

### 5. **Not Handling Edge Cases**
Forgetting to handle single character substrings or empty strings.

**Solution:** Always set `dp[i][i] = 1` inside the loop to ensure single characters are handled correctly:
```python
for i in range(n - 1, -1, -1):
    dp[i][i] = 1  # Essential: single character takes 1 operation
    for j in range(i + 1, n):
        # rest of the logic
```

### 6. **Attempting to Optimize Prematurely**
Some try to remove consecutive duplicate characters first, but this can complicate the logic unnecessarily and introduce bugs.

**Better approach:** Keep the solution simple and let the DP handle all cases uniformly. The algorithm naturally handles consecutive duplicates efficiently through the `s[i] == s[j]` case.

