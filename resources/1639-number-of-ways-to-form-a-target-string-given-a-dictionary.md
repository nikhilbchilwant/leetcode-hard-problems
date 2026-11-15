# 1639. Number of Ways to Form a Target String Given a Dictionary

> Source: https://algo.monster/liteproblems/1639

## Problem Statement

You have a list of strings `words` where all strings have the **same length**, and you need to form a target string `target` by selecting characters from these strings following specific rules.

**Rules for forming the target:**

1. **Build from left to right**: You must construct `target` character by character from index 0 to the end.

2. **Character selection**: To form the `i`-th character of `target` (0-indexed), you can pick the `k`-th character from any string `j` in `words`, as long as `words[j][k]` equals `target[i]`.

3. **Position restriction**: Once you use a character at position `k` from any string in `words`, all characters at positions `0` through `k` become **permanently unavailable** across **all strings** in `words`. This means you can only move forward in the character positions.

4. **Multiple uses allowed**: You can use multiple characters from the same string in `words`, as long as you follow the position restriction rule.

**Example visualization:**
- If `words = ["acca", "bbbb", "caca"]` and you want to form `target = "aba"`
- You could pick 'a' from position 0 of `words[0]` for `target[0]`
- Then pick 'b' from position 2 of `words[1]` for `target[1]` (positions 0-2 are now unusable)
- Finally pick 'a' from position 3 of `words[2]` for `target[2]`

The task is to count **how many different ways** you can form the complete `target` string following these rules. Since the answer can be very large, return it modulo `10^9 + 7`.

## Intuition

When we look at this problem, we need to make a series of choices: for each character in `target`, we need to decide which position across all strings in `words` to pick it from. The key constraint is that once we use position `k`, we can't use any position `≤ k` again.

This naturally leads us to think about [dynamic programming](/problems/dynamic_programming_intro) or recursion with memoization. At each step, we have two indices to track:
- `i`: which character of `target` we're trying to match
- `j`: the current position in the strings of `words` we're considering

For any state `(i, j)`, we have two choices:
1. **Skip position `j`**: We don't use any character from position `j` of any string, and move to position `j+1` while still trying to match `target[i]`
2. **Use position `j`**: If there are characters at position `j` that match `target[i]`, we can use them and move to match `target[i+1]` from position `j+1` onwards

The second choice brings up an important observation: since all strings in `words` have the same length, at any position `j`, multiple strings might have the character we need. If 3 strings have 'a' at position `j` and we need 'a' for `target[i]`, we have 3 different ways to pick it.

This suggests we should **preprocess** the data: for each position `j` and each character `c`, count how many strings have character `c` at position `j`. This gives us `cnt[j][c]` - the frequency of character `c` at position `j` across all strings.

With this preprocessing, our recursive formula becomes clearer:
- `dfs(i, j)` = number of ways to form `target[i:]` starting from position `j` in `words`
- `dfs(i, j) = dfs(i, j+1) + cnt[j][target[i]] * dfs(i+1, j+1)`

The first term represents skipping position `j`, and the second term represents using position `j` (with `cnt[j][target[i]]` different ways to do so, then continuing from position `j+1` for the next character).

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The solution implements a **top-down [dynamic programming](/problems/dynamic_programming_intro)** approach with memoization using the insights from our intuition.

**Step 1: Preprocessing**

First, we build a frequency table `cnt` where `cnt[j][c]` stores how many strings in `words` have character `c` at position `j`:

```python
cnt = [[0] * 26 for _ in range(n)]  # n is the length of each string
for w in words:
    for j, c in enumerate(w):
        cnt[j][ord(c) - ord('a')] += 1
```

This preprocessing takes `O(len(words) × n)` time and allows us to quickly look up how many ways we can pick a specific character from a specific position.

**Step 2: Recursive Function with Memoization**

We define `dfs(i, j)` which returns the number of ways to form `target[i:]` when we can only use characters from positions `j` onwards in `words`:

```python
@cache
def dfs(i: int, j: int) -> int:
    if i >= m:  # All characters of target are matched
        return 1
    if j >= n:  # No more positions available in words
        return 0
  
    # Option 1: Use position j to match target[i]
    ans = dfs(i + 1, j + 1) * cnt[j][ord(target[i]) - ord('a')]
  
    # Option 2: Skip position j
    ans = (ans + dfs(i, j + 1)) % mod
  
    return ans
```

**Base Cases:**
- `i >= m`: We've successfully matched all characters in `target`, so return `1`
- `j >= n`: We've exhausted all positions in `words` but haven't matched all of `target`, so return `0`

**Recursive Cases:**
- **Use position `j`**: If we use position `j` to match `target[i]`, we have `cnt[j][target[i]]` ways to do so. After using position `j`, we move to `target[i+1]` and can only use positions from `j+1` onwards. This gives us `cnt[j][target[i]] × dfs(i+1, j+1)` ways.
- **Skip position `j`**: We don't use position `j` at all and try to match `target[i]` from position `j+1` onwards. This gives us `dfs(i, j+1)` ways.

The total number of ways is the sum of both options, taken modulo `10^9 + 7`.

**Time Complexity:** `O(m × n)` where `m = len(target)` and `n = len(words[0])`, since we have at most `m × n` unique states and each state is computed once due to memoization.

**Space Complexity:** `O(m × n)` for the memoization cache plus `O(26 × n)` for the preprocessing array.

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

**Given:**
- `words = ["acca", "bbbb", "caca"]`
- `target = "aba"`

**Step 1: Preprocessing**

First, we build the frequency table `cnt[j][c]` that counts how many strings have character `c` at position `j`:

```
Position 0: a appears 1 time (in "acca")
           b appears 1 time (in "bbbb")
           c appears 1 time (in "caca")

Position 1: a appears 1 time (in "caca")
           b appears 1 time (in "bbbb")
           c appears 1 time (in "acca")

Position 2: a appears 0 times
           b appears 1 time (in "bbbb")
           c appears 2 times (in "acca", "caca")

Position 3: a appears 2 times (in "acca", "caca")
           b appears 1 time (in "bbbb")
           c appears 0 times
```

**Step 2: Recursive Calculation**

Now we calculate `dfs(0, 0)` - ways to form `target[0:]` = "aba" starting from position 0.

`dfs(0, 0)`: Form "aba" from position 0
- Option 1: Use position 0 for 'a' → `cnt[0]['a'] × dfs(1, 1)` = `1 × dfs(1, 1)`
- Option 2: Skip position 0 → `dfs(0, 1)`
- Result: `dfs(1, 1) + dfs(0, 1)`

`dfs(1, 1)`: Form "ba" from position 1
- Option 1: Use position 1 for 'b' → `cnt[1]['b'] × dfs(2, 2)` = `1 × dfs(2, 2)`
- Option 2: Skip position 1 → `dfs(1, 2)`
- Result: `dfs(2, 2) + dfs(1, 2)`

`dfs(2, 2)`: Form "a" from position 2
- Option 1: Use position 2 for 'a' → `cnt[2]['a'] × dfs(3, 3)` = `0 × 1` = `0`
- Option 2: Skip position 2 → `dfs(2, 3)`
- Result: `0 + dfs(2, 3)`

`dfs(2, 3)`: Form "a" from position 3
- Option 1: Use position 3 for 'a' → `cnt[3]['a'] × dfs(3, 4)` = `2 × 1` = `2`
- Option 2: Skip position 3 → `dfs(2, 4)` = `0` (no more positions)
- Result: `2 + 0 = 2`

Working backwards:
- `dfs(2, 2) = 0 + 2 = 2`
- `dfs(1, 2)`: Form "ba" from position 2
  - Use position 2 for 'b': `1 × dfs(2, 3)` = `1 × 2` = `2`
  - Skip position 2: `dfs(1, 3)` = `0` (can't form 'b' from position 3)
  - Result: `2 + 0 = 2`
- `dfs(1, 1) = 2 + 2 = 4`

For `dfs(0, 1)`: Form "aba" from position 1
- Eventually evaluates to `0` (we can't form all of "aba" starting from position 1)

**Final Answer:** `dfs(0, 0) = 4 + 0 = 4`

The 4 ways are:
1. `words[0][0]='a', words[1][1]='b', words[0][3]='a'`
2. `words[0][0]='a', words[1][1]='b', words[2][3]='a'`
3. `words[0][0]='a', words[1][2]='b', words[0][3]='a'`
4. `words[0][0]='a', words[1][2]='b', words[2][3]='a'`

## Python Solution

```python
class Solution:
    def numWays(self, words: List[str], target: str) -> int:
        """
        Count the number of ways to form the target string by selecting characters
        from columns of the words array.
      
        Args:
            words: List of strings with equal length
            target: Target string to form
          
        Returns:
            Number of ways to form target modulo 10^9 + 7
        """
        from functools import cache
      
        # Get dimensions
        target_length = len(target)
        word_length = len(words[0])
      
        # Count frequency of each character at each column position
        # char_count[column][character] = frequency of character at column
        char_count = [[0] * 26 for _ in range(word_length)]
      
        for word in words:
            for column_idx, char in enumerate(word):
                char_index = ord(char) - ord('a')
                char_count[column_idx][char_index] += 1
      
        # Modulo value for large numbers
        MOD = 10**9 + 7
      
        @cache
        def dp(target_idx: int, column_idx: int) -> int:
            """
            Dynamic programming function to count ways to form target[target_idx:]
            using columns starting from column_idx.
          
            Args:
                target_idx: Current index in target string
                column_idx: Current column index in words
              
            Returns:
                Number of ways to form the remaining target string
            """
            # Base case: Successfully formed the entire target string
            if target_idx >= target_length:
                return 1
          
            # Base case: No more columns available but target not complete
            if column_idx >= word_length:
                return 0
          
            # Option 1: Use current column to match target[target_idx]
            # Get the character we need from target
            needed_char_index = ord(target[target_idx]) - ord('a')
          
            # Count ways if we use this column (multiply by frequency of needed character)
            use_column = dp(target_idx + 1, column_idx + 1) * char_count[column_idx][needed_char_index]
          
            # Option 2: Skip current column
            skip_column = dp(target_idx, column_idx + 1)
          
            # Total ways is sum of both options
            total_ways = (use_column + skip_column) % MOD
          
            return total_ways
      
        # Start the dynamic programming from beginning of both target and columns
        return dp(0, 0)
```

## Time and Space Complexity

**Time Complexity:** `O(m × n + k × n)`

The algorithm consists of two main parts:
1. **Preprocessing phase**: Building the `cnt` array takes `O(k × n)` time, where `k` is the number of words in the `words` array and `n` is the length of each word. We iterate through all characters in all words.

2. **Dynamic Programming phase**: The `dfs` function with memoization explores at most `m × n` unique states, where `m` is the length of the target string and `n` is the length of each word. Each state `(i, j)` is computed only once due to the `@cache` decorator. For each state, the computation takes `O(1)` time.

Therefore, the overall time complexity is `O(k × n + m × n)`. If we assume `k` is relatively small or consider it as part of the input preprocessing, the dominant term becomes `O(m × n)`.

**Space Complexity:** `O(m × n + n)`

The space usage includes:
1. **Memoization cache**: The `@cache` decorator stores up to `m × n` states, each taking `O(1)` space, resulting in `O(m × n)` space.

2. **Count array**: The `cnt` array has dimensions `n × 26`, which is `O(n)` space since 26 is a constant.

3. **Recursion stack**: In the worst case, the recursion depth can be `O(m + n)`.

The dominant term is `O(m × n)` from the memoization cache, making the overall space complexity `O(m × n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Integer Overflow When Computing Products**

**The Pitfall:**
When calculating `dp(target_idx + 1, column_idx + 1) * char_count[column_idx][needed_char_index]`, the multiplication can produce very large numbers that exceed Python's integer limits in other languages or cause issues even before applying the modulo operation. While Python handles arbitrary precision integers, in languages like Java or C++, this would cause overflow. Even in Python, performing modulo operations too late can impact performance.

**Example of the Issue:**
```python
# Problematic approach - modulo applied after multiplication
use_column = dp(target_idx + 1, column_idx + 1) * char_count[column_idx][needed_char_index]
total_ways = (use_column + skip_column) % MOD  # Modulo applied too late
```

**The Solution:**
Apply modulo operations immediately after each arithmetic operation to keep numbers manageable:

```python
# Better approach - apply modulo after each operation
use_column = (dp(target_idx + 1, column_idx + 1) * char_count[column_idx][needed_char_index]) % MOD
skip_column = dp(target_idx, column_idx + 1) % MOD
total_ways = (use_column + skip_column) % MOD
```

### 2. **Incorrect Understanding of the Position Restriction**

**The Pitfall:**
A common misunderstanding is thinking that once you use a character from a specific string at position `k`, you can't use that particular string anymore. The actual rule is that positions 0 through `k` become unavailable across ALL strings, not just the one you picked from.

**Incorrect Mental Model:**
```python
# Wrong: Tracking which strings have been used
def dp(target_idx, column_idx, used_strings):
    # This would incorrectly track individual string usage
    pass
```

**Correct Understanding:**
The restriction is column-based (position-based) across all strings:
```python
# Correct: Only tracking column position
def dp(target_idx, column_idx):
    # When we use column_idx, all columns 0 to column_idx become unavailable
    # We can only proceed with column_idx + 1 onwards
    use_column = dp(target_idx + 1, column_idx + 1) * char_count[column_idx][needed_char_index]
```

### 3. **Off-by-One Errors in Index Handling**

**The Pitfall:**
Mixing up whether indices are inclusive or exclusive, especially in base cases:

**Common Mistake:**
```python
# Wrong base case checks
if target_idx > target_length:  # Should be >=
    return 1
if column_idx > word_length:    # Should be >=
    return 0
```

**Correct Implementation:**
```python
# Correct base case checks
if target_idx >= target_length:  # We've processed all characters (0 to target_length-1)
    return 1
if column_idx >= word_length:    # No more columns available (0 to word_length-1)
    return 0
```

### 4. **Forgetting to Handle the Case Where Character Doesn't Exist**

**The Pitfall:**
Not considering that `char_count[column_idx][needed_char_index]` might be 0, meaning the needed character doesn't exist at that column position.

**Why It's Not Obviously Wrong:**
The code still works because multiplying by 0 gives 0 ways, which is correct. However, this can lead to unnecessary recursive calls and reduced performance.

**Optimization:**
```python
# More efficient - skip recursion if character doesn't exist
needed_char_index = ord(target[target_idx]) - ord('a')
if char_count[column_idx][needed_char_index] > 0:
    use_column = (dp(target_idx + 1, column_idx + 1) * 
                  char_count[column_idx][needed_char_index]) % MOD
else:
    use_column = 0  # Skip unnecessary recursion

skip_column = dp(target_idx, column_idx + 1)
return (use_column + skip_column) % MOD
```

