# 1745. Palindrome Partitioning IV

> Source: https://algo.monster/liteproblems/1745

## Problem Statement

Given a string `s`, the task is to determine whether it's possible to split the string into exactly three **non-empty** palindromic substrings. Return `true` if such a split is possible, otherwise return `false`.

A palindrome is a string that reads the same forwards and backwards. For example, "aba", "racecar", and "a" are palindromes.

The key requirements are:
- The string must be split into exactly **three** parts
- Each part must be **non-empty** (contain at least one character)
- Each part must be a **palindrome**
- The three parts must be consecutive substrings that together form the original string

For example:
- If `s = "abcbdd"`, it can be split into `"a"`, `"bcb"`, `"dd"` - all three are palindromes, so return `true`
- If `s = "bcbddxy"`, it cannot be split into three palindromic substrings, so return `false`

## Intuition

The naive approach would be to try all possible ways to split the string into three parts and check if each part is a palindrome. However, checking if a substring is a palindrome takes O(n) time, and with O(n²) possible splits, this would be inefficient.

The key insight is that we can precompute which substrings are palindromes to avoid redundant palindrome checks. Once we know which substrings are palindromes, we can efficiently check all possible three-way splits.

To build our palindrome information, we observe that a substring `s[i..j]` is a palindrome if and only if:
1. The first and last characters match (`s[i] == s[j]`), AND
2. Either it's a substring of length 1 or 2 (base cases), OR the inner substring `s[i+1..j-1]` is also a palindrome

This recursive relationship suggests using [dynamic programming](/problems/dynamic_programming_intro). We create a 2D table `f[i][j]` where `f[i][j] = true` if substring `s[i..j]` is a palindrome.

Once we have this table, finding a valid three-way split becomes straightforward. We need to find two split points that divide the string into three parts. We can iterate through all possible positions for these two split points:
- First split after position `i` (creating substring `s[0..i]`)  
- Second split after position `j` (creating substring `s[i+1..j]`)
- The third substring is automatically `s[j+1..n-1]`

If all three parts are palindromes (which we can check in O(1) time using our precomputed table), we've found a valid partition. The beauty of this approach is that we reduce the problem from repeated substring checking to simple table lookups.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The solution uses [dynamic programming](/problems/dynamic_programming_intro) to efficiently determine all palindromic substrings, then checks all possible three-way partitions.

**Step 1: Build the Palindrome Table**

We create a 2D boolean table `f[i][j]` where `f[i][j] = True` if substring `s[i..j]` is a palindrome. The table is initialized with all values as `True`.

To fill this table, we use the state transition equation:
```
f[i][j] = (s[i] == s[j]) and (i + 1 == j or f[i + 1][j - 1])
```

This means substring `s[i..j]` is a palindrome if:
- Characters at positions `i` and `j` are equal, AND
- Either the substring has length 2 (when `i + 1 == j`), OR
- The inner substring `s[i+1..j-1]` is also a palindrome

Since `f[i][j]` depends on `f[i + 1][j - 1]`, we iterate `i` from `n-1` down to `0` (reverse order) and `j` from `i+1` to `n-1` (forward order). This ensures that when computing `f[i][j]`, the value `f[i + 1][j - 1]` has already been computed.

**Step 2: Check All Possible Three-Way Partitions**

After building the palindrome table, we enumerate all possible ways to split the string into three parts:
- First substring: `s[0..i]` where `i` ranges from `0` to `n-3`
- Second substring: `s[i+1..j]` where `j` ranges from `i+1` to `n-2`  
- Third substring: `s[j+1..n-1]` (automatically determined by first two splits)

For each partition, we check if all three parts are palindromes:
- `f[0][i]` checks if first part is palindrome
- `f[i + 1][j]` checks if second part is palindrome
- `f[j + 1][n-1]` checks if third part is palindrome

If we find any partition where all three conditions are true, we immediately return `True`.

**Time Complexity:** O(n²) for building the palindrome table and O(n²) for checking all partitions, giving overall O(n²).

**Space Complexity:** O(n²) for the palindrome table.

## Example Walkthrough

Let's walk through the solution with `s = "abcbdd"`.

**Step 1: Build the Palindrome Table**

We create a 6×6 table (string length is 6) to track which substrings are palindromes.

Starting with the table initialized to `True`, we fill it by checking each substring:

For substring length 1 (single characters):
- All single characters are palindromes by default

For substring length 2:
- `s[0..1]` = "ab": `s[0] ≠ s[1]` → `f[0][1] = False`
- `s[1..2]` = "bc": `s[1] ≠ s[2]` → `f[1][2] = False`
- `s[2..3]` = "cb": `s[2] ≠ s[3]` → `f[2][3] = False`
- `s[3..4]` = "bd": `s[3] ≠ s[4]` → `f[3][4] = False`
- `s[4..5]` = "dd": `s[4] = s[5]` → `f[4][5] = True` ✓

For substring length 3:
- `s[1..3]` = "bcb": `s[1] = s[3] = 'b'` and `f[2][2] = True` → `f[1][3] = True` ✓
- Other length-3 substrings are not palindromes

Continuing this process, our final palindrome table shows:
- "a" (index 0-0): palindrome ✓
- "bcb" (index 1-3): palindrome ✓
- "dd" (index 4-5): palindrome ✓
- "abcbd" (index 0-4): not a palindrome
- And so on...

**Step 2: Check All Three-Way Partitions**

Now we try all possible ways to split into three parts:

Split at positions (0, 1):
- Part 1: `s[0..0]` = "a" → `f[0][0] = True` ✓
- Part 2: `s[1..1]` = "b" → `f[1][1] = True` ✓
- Part 3: `s[2..5]` = "cbdd" → `f[2][5] = False` ✗
- Not valid

Split at positions (0, 2):
- Part 1: `s[0..0]` = "a" → `f[0][0] = True` ✓
- Part 2: `s[1..2]` = "bc" → `f[1][2] = False` ✗
- Not valid

Split at positions (0, 3):
- Part 1: `s[0..0]` = "a" → `f[0][0] = True` ✓
- Part 2: `s[1..3]` = "bcb" → `f[1][3] = True` ✓
- Part 3: `s[4..5]` = "dd" → `f[4][5] = True` ✓
- **Valid partition found!** Return `True`

The string "abcbdd" can be split into "a" + "bcb" + "dd", where all three parts are palindromes.

## Python Solution

```python
class Solution:
    def checkPartitioning(self, s: str) -> bool:
        """
        Check if a string can be partitioned into exactly 3 palindromic substrings.
      
        Args:
            s: Input string to be partitioned
          
        Returns:
            True if the string can be partitioned into 3 palindromes, False otherwise
        """
        n = len(s)
      
        # Build a 2D DP table where is_palindrome[i][j] indicates 
        # whether substring s[i:j+1] is a palindrome
        is_palindrome = [[True] * n for _ in range(n)]
      
        # Fill the palindrome table using bottom-up approach
        # Start from the end of string and work backwards
        for start in range(n - 1, -1, -1):
            for end in range(start + 1, n):
                # A substring is palindrome if:
                # 1. First and last characters match, AND
                # 2. Either it's a 2-character string OR the inner substring is also palindrome
                is_palindrome[start][end] = (
                    s[start] == s[end] and 
                    (end - start == 1 or is_palindrome[start + 1][end - 1])
                )
      
        # Try all possible ways to split the string into 3 parts
        # First cut after index i, second cut after index j
        for first_cut in range(n - 2):  # Leave room for at least 2 more characters
            for second_cut in range(first_cut + 1, n - 1):  # Leave room for last substring
                # Check if all three parts are palindromes:
                # Part 1: s[0:first_cut+1]
                # Part 2: s[first_cut+1:second_cut+1]  
                # Part 3: s[second_cut+1:n]
                if (is_palindrome[0][first_cut] and 
                    is_palindrome[first_cut + 1][second_cut] and 
                    is_palindrome[second_cut + 1][n - 1]):
                    return True
      
        return False
```

## Time and Space Complexity

The time complexity is `O(n^2)`, where `n` is the length of the string `s`. This is because:
- The first nested loop (building the palindrome table `f`) iterates through all pairs `(i, j)` where `i < j`, which takes `O(n^2)` time
- The second nested loop also iterates through all valid pairs `(i, j)` where `i` ranges from `0` to `n-3` and `j` ranges from `i+1` to `n-2`, which is also `O(n^2)` in the worst case
- Each operation inside both loops (array access and comparison) takes `O(1)` time
- Therefore, the overall time complexity is `O(n^2) + O(n^2) = O(n^2)`

The space complexity is `O(n^2)`, where `n` is the length of the string `s`. This is because:
- The 2D array `f` is created with dimensions `n × n`, requiring `O(n^2)` space to store boolean values indicating whether substrings `s[i:j+1]` are palindromes
- No other significant auxiliary space is used besides a few integer variables for loop indices
- Therefore, the overall space complexity is `O(n^2)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Palindrome Table Initialization**

A common mistake is initializing the entire palindrome table as `True`, which can lead to incorrect results for substrings that haven't been properly validated.

**Pitfall Example:**
```python
# Incorrect - assumes all substrings are palindromes initially
is_palindrome = [[True] * n for _ in range(n)]
```

**Solution:**
Initialize only the diagonal (single characters) as `True`, and set other values during computation:
```python
is_palindrome = [[False] * n for _ in range(n)]
# Single characters are always palindromes
for i in range(n):
    is_palindrome[i][i] = True
```

### 2. **Wrong Iteration Order When Building Palindrome Table**

The most critical pitfall is iterating in the wrong order when filling the palindrome table. Since `is_palindrome[i][j]` depends on `is_palindrome[i+1][j-1]`, we must ensure the inner substring is computed before the outer one.

**Pitfall Example:**
```python
# Incorrect - forward iteration doesn't guarantee dependencies are met
for start in range(n):
    for end in range(start + 1, n):
        is_palindrome[start][end] = ...  # May access uncomputed values
```

**Solution:**
Use reverse iteration for the start index and forward iteration for the end index:
```python
for start in range(n - 1, -1, -1):  # Reverse order
    for end in range(start + 1, n):   # Forward order
        is_palindrome[start][end] = (
            s[start] == s[end] and 
            (end - start == 1 or is_palindrome[start + 1][end - 1])
        )
```

### 3. **Off-by-One Errors in Partition Boundaries**

Confusing inclusive vs exclusive indices when checking partitions can lead to missing valid solutions or checking invalid partitions.

**Pitfall Example:**
```python
# Incorrect bounds - might create empty partitions
for first_cut in range(n - 1):  # Could go too far
    for second_cut in range(first_cut, n):  # Could create empty second part
```

**Solution:**
Ensure each partition has at least one character:
```python
for first_cut in range(n - 2):  # Stop at n-2 to leave room for 2 more chars
    for second_cut in range(first_cut + 1, n - 1):  # Start after first_cut, stop before last
        # This guarantees:
        # - First part: [0, first_cut] has at least 1 char
        # - Second part: [first_cut+1, second_cut] has at least 1 char  
        # - Third part: [second_cut+1, n-1] has at least 1 char
```

### 4. **Handling Edge Cases for Length-2 Palindromes**

Forgetting to handle the special case where a substring has exactly 2 characters can cause index out-of-bounds errors.

**Pitfall Example:**
```python
# Incorrect - doesn't handle 2-character substrings properly
is_palindrome[start][end] = (
    s[start] == s[end] and is_palindrome[start + 1][end - 1]
)  # When end - start == 1, this accesses is_palindrome[start+1][start]
```

**Solution:**
Add explicit check for 2-character substrings:
```python
is_palindrome[start][end] = (
    s[start] == s[end] and 
    (end - start == 1 or is_palindrome[start + 1][end - 1])
)
```

### 5. **Memory Optimization Consideration**

While not necessarily a pitfall, the O(n²) space complexity can be problematic for very long strings. Consider checking palindromes on-the-fly if memory is constrained, though this increases time complexity.

