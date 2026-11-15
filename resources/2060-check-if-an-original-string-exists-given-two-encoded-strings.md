# 2060. Check if an Original String Exists Given Two Encoded Strings

> Source: https://algo.monster/liteproblems/2060

## Problem Statement

This problem asks you to determine if two encoded strings could have originated from the same original string.

The encoding process works as follows:
1. Start with an original string containing only lowercase English letters
2. Split this string into a sequence of non-empty substrings (in any way you choose)
3. Optionally replace some of these substrings with their lengths (as numeric strings)
4. Concatenate everything together to form the encoded string

For example, the original string `"abcdefghijklmnop"` could be encoded as `"ab121p"` by:
- Splitting into: `["ab", "cdefghijklmn", "o", "p"]`
- Replacing the 2nd and 3rd elements with their lengths: `["ab", "12", "1", "p"]`
- Concatenating to get: `"ab121p"`

Given two encoded strings `s1` and `s2` (containing lowercase letters and digits 1-9), you need to return `true` if there exists some original string that could be encoded to produce both `s1` and `s2`. Otherwise, return `false`.

The key insight is that digits in the encoded strings represent the lengths of substrings from the original. So when comparing two encoded strings, you need to account for the fact that:
- Letters must match exactly with corresponding letters
- Digits represent "wildcards" that can match any substring of the specified length
- A digit sequence like `"12"` means 12 consecutive characters from the original string

The challenge is handling all possible interpretations of digit sequences (since `"12"` could mean 12 characters, or it could be `"1"` and `"2"` representing 1 and 2 characters respectively) and determining if there's a valid matching between the two encoded strings.

Note: The problem guarantees that no more than 3 consecutive digits appear in the encoded strings.

## Intuition

The key realization is that we need to match two encoded strings character by character, but digits create flexibility since they represent "wildcards" of variable length. Think of it like having two pointers moving through `s1` and `s2`, but sometimes one pointer can "skip ahead" when it encounters digits.

The main challenge is handling the mismatch between positions. When `s1` has a digit like `"5"`, it means 5 characters from the original string, while `s2` might have those same 5 characters represented as `"2"` + `"3"` or as actual letters like `"ab3"`. We need to track this "difference" or "delta" between how many characters each string has consumed from the original.

Consider this example: if `s1 = "a3"` and `s2 = "2bc"`, both could represent `"abbc"`:
- `s1`: `"a"` + 3 chars (`"bbc"`)
- `s2`: 2 chars (`"ab"`) + `"b"` + `"c"`

The insight is to use [dynamic programming](/problems/dynamic_programming_intro) with state `dp[i][j][delta]` where:
- `i` is the current position in `s1`
- `j` is the current position in `s2`
- `delta` represents the "balance" - how many more characters `s1` has consumed versus `s2`

When `delta > 0`, `s1` is "ahead" and needs `s2` to catch up. When `delta < 0`, `s2` is ahead. When `delta = 0`, both strings have consumed the same number of characters from the original string and can potentially match letters directly.

At each state, we have several transitions:
1. If we see digits, we can interpret them as numbers and adjust the delta accordingly
2. If `delta ≠ 0`, we can match letters against the "debt" created by previous digits
3. If `delta = 0` and both positions have the same letter, they can match directly

The answer is `true` if we can reach `dp[n][m][0]`, meaning we've processed both strings entirely and they've consumed the same total number of characters from the original string.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The solution uses [dynamic programming](/problems/dynamic_programming_intro) with a 3D state space represented as a 2D array where each cell contains a `Set` of possible delta values.

**State Definition:**
- `dp[i][j]` is a `Set` containing all possible delta values when we've processed `i` characters from `s1` and `j` characters from `s2`
- `delta` represents the difference in consumed characters: positive means `s1` consumed more, negative means `s2` consumed more

**Initialization:**
- `dp[0][0].add(0)` - at the start, both strings have consumed 0 characters

**State Transitions:**

For each state `(i, j)` with delta value, we have five possible transitions:

1. **Process digits in `s1` (when `delta <= 0`):**
   ```typescript
   for (let p = i; p < Math.min(i + 3, n); p++) {
       if (isDigit(s1[p])) {
           num = num * 10 + Number(s1[p]);
           dp[p + 1][j].add(delta + num);
       }
   }
   ```
   When `s1` is not ahead, we can interpret consecutive digits (up to 3) as a number and add it to delta.

2. **Process digits in `s2` (when `delta >= 0`):**
   ```typescript
   for (let q = j; q < Math.min(j + 3, m); q++) {
       if (isDigit(s2[q])) {
           num = num * 10 + Number(s2[q]);
           dp[i][q + 1].add(delta - num);
       }
   }
   ```
   When `s2` is not ahead, we can interpret consecutive digits and subtract from delta.

3. **Match letter in `s1` against deficit (when `delta < 0`):**
   ```typescript
   if (i < n && delta < 0 && !isDigit(s1[i])) {
       dp[i + 1][j].add(delta + 1);
   }
   ```
   If `s2` previously consumed more characters (negative delta), a letter in `s1` can match one of those characters.

4. **Match letter in `s2` against surplus (when `delta > 0`):**
   ```typescript
   if (j < m && delta > 0 && !isDigit(s2[j])) {
       dp[i][j + 1].add(delta - 1);
   }
   ```
   If `s1` previously consumed more characters (positive delta), a letter in `s2` can match one of those characters.

5. **Direct letter matching (when `delta == 0`):**
   ```typescript
   if (i < n && j < m && delta == 0 && s1[i] == s2[j]) {
       dp[i + 1][j + 1].add(0);
   }
   ```
   When both strings have consumed the same amount, matching letters can directly correspond.

**Final Check:**
Return `dp[n][m].has(0)` - we need both strings fully processed with delta = 0, meaning they represent the same total length from the original string.

The algorithm efficiently explores all possible interpretations of digit sequences and letter matchings, using the delta mechanism to ensure consistency between the two encoded strings.

## Example Walkthrough

Let's trace through a small example: `s1 = "l23"` and `s2 = "4e"`. These could both represent the original string "love":
- `s1 = "l23"` → `"l"` + 2 chars (`"ov"`) + 3 chars (`"e"`) → but wait, that's only 4 chars total
- Let me reconsider: `s1 = "l23"` → `"l"` + 23 chars? No, that doesn't match with `s2 = "4e"`

Let me use a clearer example: `s1 = "a3"` and `s2 = "2bc"`. Both could represent `"abbc"`:
- `s1`: `"a"` + 3 chars (`"bbc"`)  
- `s2`: 2 chars (`"ab"`) + `"b"` + `"c"`

**Step-by-step DP traversal:**

Initialize: `dp[0][0] = {0}` (both strings start at position 0 with delta 0)

**From state (0, 0, delta=0):**
1. Process digit in `s2` at position 0: `"2"` → `dp[0][1] = {-2}` (s2 consumes 2 chars, s1 consumes 0)
2. Direct match letters: `s1[0]='a'` and `s2[0]='2'` → no match (one is digit)

**From state (0, 1, delta=-2):**
Since delta < 0, s2 is ahead by 2 chars. We can:
1. Match letter `s1[0]='a'` against deficit → `dp[1][1] = {-1}` (s1 catches up by 1)

**From state (1, 1, delta=-1):**
Since delta < 0, s2 is still ahead by 1. We can:
1. Process digit `s1[1]='3'` (since delta ≤ 0) → `dp[2][1] = {2}` (delta = -1 + 3 = 2)

**From state (2, 1, delta=2):**
Since delta > 0, s1 is ahead by 2 chars. We can:
1. Match letter `s2[1]='b'` against surplus → `dp[2][2] = {1}` (s2 catches up by 1)

**From state (2, 2, delta=1):**
Since delta > 0, s1 is still ahead by 1. We can:
1. Match letter `s2[2]='c'` against surplus → `dp[2][3] = {0}` (s2 catches up completely)

**Final state (2, 3, delta=0):**
We've processed all of `s1` (length 2) and all of `s2` (length 3) with delta=0.

Since `dp[2][3]` contains 0, return `true` - both strings can represent the same original string "abbc" where:
- `s1 = "a3"` encodes it as letter 'a' followed by 3 characters
- `s2 = "2bc"` encodes it as 2 characters followed by letters 'b' and 'c'

## Python Solution

```python
def possiblyEquals(s1: str, s2: str) -> bool:
    """
    Determines if two strings with wildcard numbers can possibly be equal.
    Numbers in strings represent that many lowercase letters.
  
    Args:
        s1: First string containing letters and numbers
        s2: Second string containing letters and numbers
  
    Returns:
        True if the strings can possibly be equal, False otherwise
    """
    s1_length = len(s1)
    s2_length = len(s2)
  
    # dp[i][j] stores set of possible delta values at position (i, j)
    # delta represents the difference in wildcard counts between s1 and s2
    # positive delta means s1 has more wildcards to consume
    # negative delta means s2 has more wildcards to consume
    dp = [[set() for _ in range(s2_length + 1)] for _ in range(s1_length + 1)]
  
    # Initialize: at position (0, 0) with no wildcard difference
    dp[0][0].add(0)
  
    # Process all positions in both strings
    for i in range(s1_length + 1):
        for j in range(s2_length + 1):
            # Process each possible delta value at current position
            for delta in dp[i][j]:
              
                # Case 1: Parse consecutive digits in s1 as a wildcard number
                if delta <= 0:
                    wildcard_value = 0
                    # Parse up to 3 consecutive digits (max wildcard is 999)
                    for next_pos in range(i, min(i + 3, s1_length)):
                        if is_digit(s1[next_pos]):
                            wildcard_value = wildcard_value * 10 + int(s1[next_pos])
                            # Add this wildcard value to delta
                            dp[next_pos + 1][j].add(delta + wildcard_value)
                        else:
                            break  # Stop if we encounter a non-digit
              
                # Case 2: Parse consecutive digits in s2 as a wildcard number
                if delta >= 0:
                    wildcard_value = 0
                    # Parse up to 3 consecutive digits (max wildcard is 999)
                    for next_pos in range(j, min(j + 3, s2_length)):
                        if is_digit(s2[next_pos]):
                            wildcard_value = wildcard_value * 10 + int(s2[next_pos])
                            # Subtract this wildcard value from delta
                            dp[i][next_pos + 1].add(delta - wildcard_value)
                        else:
                            break  # Stop if we encounter a non-digit
              
                # Case 3: Match a letter in s1 with pending wildcards from s2
                if i < s1_length and delta < 0 and not is_digit(s1[i]):
                    # Consume one wildcard from s2 to match letter in s1
                    dp[i + 1][j].add(delta + 1)
              
                # Case 4: Match a letter in s2 with pending wildcards from s1
                if j < s2_length and delta > 0 and not is_digit(s2[j]):
                    # Consume one wildcard from s1 to match letter in s2
                    dp[i][j + 1].add(delta - 1)
              
                # Case 5: Match two identical letters when no wildcards are pending
                if i < s1_length and j < s2_length and delta == 0 and s1[i] == s2[j]:
                    # Both letters match, move to next position with delta = 0
                    dp[i + 1][j + 1].add(0)
  
    # Check if we can reach the end of both strings with balanced wildcards
    return 0 in dp[s1_length][s2_length]


def is_digit(char: str) -> bool:
    """
    Checks if a character is a single digit (0-9).
  
    Args:
        char: Character to check
  
    Returns:
        True if the character is a digit, False otherwise
    """
    return char.isdigit()
```

## Time and Space Complexity

**Time Complexity:** `O(n * m * (n + m))`

The algorithm uses dynamic programming with three nested loops:
- The outer two loops iterate through positions `i` from `0` to `n` and `j` from `0` to `m`, giving us `O(n * m)` iterations
- For each `(i, j)` position, we iterate through all values in `dp[i][j]` set. In the worst case, the set can contain `O(n + m)` different delta values because:
  - Delta represents the difference in "virtual positions" between the two strings
  - The maximum positive delta occurs when s1 has consumed all its digits (up to `3n` in value)
  - The maximum negative delta occurs when s2 has consumed all its digits (up to `3m` in value)
  - The range of possible deltas is bounded by `O(n + m)`
- Inside each delta iteration, we perform operations that take at most `O(1)` time (the inner loops for parsing digits are bounded by constant 3)

Therefore, the total time complexity is `O(n * m * (n + m))`.

**Space Complexity:** `O(n * m * (n + m))`

The space is dominated by the `dp` array:
- We create a 2D array of size `(n + 1) × (m + 1)`
- Each cell contains a `Set` that can hold up to `O(n + m)` different delta values in the worst case
- Therefore, the total space complexity is `O(n * m * (n + m))`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Handling of Multi-Digit Numbers**

**Pitfall:** A common mistake is treating each digit independently rather than considering all possible ways to parse consecutive digits. For example, "12" could represent:
- A single number 12 (matching 12 characters)
- Two separate numbers 1 and 2 (matching 1 and 2 characters respectively)

**Why it happens:** It's tempting to greedily parse digits or use a simpler approach that doesn't explore all interpretations.

**Solution:** The code correctly handles this by using nested loops that try all possible digit sequence lengths (up to 3):
```python
for next_pos in range(i, min(i + 3, s1_length)):
    if is_digit(s1[next_pos]):
        wildcard_value = wildcard_value * 10 + int(s1[next_pos])
        dp[next_pos + 1][j].add(delta + wildcard_value)
```

### 2. **Misunderstanding the Delta Mechanism**

**Pitfall:** Incorrectly interpreting what the delta value represents or when to allow certain transitions. Common errors include:
- Allowing digit parsing from s1 when delta > 0 (s1 is already ahead)
- Allowing digit parsing from s2 when delta < 0 (s2 is already ahead)

**Why it happens:** The delta concept is counterintuitive - it tracks the "debt" of characters that need to be matched.

**Solution:** The code enforces correct constraints:
- Only parse digits from s1 when `delta <= 0` (s1 is not ahead)
- Only parse digits from s2 when `delta >= 0` (s2 is not ahead)
- Only match letters against wildcards when there's a corresponding debt

### 3. **Forgetting to Break on Non-Digit Characters**

**Pitfall:** When parsing consecutive digits, forgetting to stop when encountering a letter:
```python
# WRONG - continues even after hitting a letter
for next_pos in range(i, min(i + 3, s1_length)):
    wildcard_value = wildcard_value * 10 + int(s1[next_pos])
    dp[next_pos + 1][j].add(delta + wildcard_value)
```

**Why it happens:** Overlooking that digit sequences must be contiguous.

**Solution:** Always check if the character is a digit and break if not:
```python
if is_digit(s1[next_pos]):
    wildcard_value = wildcard_value * 10 + int(s1[next_pos])
    dp[next_pos + 1][j].add(delta + wildcard_value)
else:
    break  # Stop parsing when we hit a non-digit
```

### 4. **Using Insufficient Data Structure for DP State**

**Pitfall:** Using a 2D boolean array `dp[i][j]` instead of storing sets of possible delta values:
```python
# WRONG - loses information about possible states
dp = [[False] * (m + 1) for _ in range(n + 1)]
```

**Why it happens:** Trying to simplify the state space without realizing that multiple delta values can lead to the same (i, j) position.

**Solution:** Use sets to store all possible delta values at each position:
```python
dp = [[set() for _ in range(s2_length + 1)] for _ in range(s1_length + 1)]
```

### 5. **Off-by-One Errors in Index Management**

**Pitfall:** Incorrectly managing indices when transitioning between states, especially when parsing multi-digit numbers:
```python
# WRONG - might access out of bounds or skip positions
for next_pos in range(i + 1, i + 4):  # Should start at i, not i+1
    if next_pos < s1_length and is_digit(s1[next_pos]):
        ...
```

**Why it happens:** Confusion about whether indices represent positions before or after consuming characters.

**Solution:** Carefully track that `dp[i][j]` means we've consumed exactly i characters from s1 and j from s2, and update indices accordingly.

