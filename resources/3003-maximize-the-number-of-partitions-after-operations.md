# 3003. Maximize the Number of Partitions After Operations

> Source: https://algo.monster/liteproblems/3003

## Problem Statement

You are given a string `s` consisting of lowercase English letters and an integer `k`.

The problem allows you to perform two operations:

1. **Optional Change Operation**: You can change at most one character in the string `s` to any other lowercase English letter. This step is optional - you can choose to change one character or leave the string unchanged.

2. **Partitioning Operation**: After the optional change, you must partition the string using the following process:
   - Start from the beginning of `s`
   - Find the longest prefix that contains at most `k` distinct characters
   - Remove this prefix from `s` and count it as one partition
   - Repeat this process with the remaining string until `s` becomes empty
   - The characters that remain after removing each prefix maintain their original order

Your goal is to find the maximum number of partitions you can create by optimally choosing whether to change a character (and which one to change if you do).

**Example Walkthrough:**

If `s = "aabaab"` and `k = 2`:
- Without changing any character: 
  - First partition: "aabaa" (contains 2 distinct characters: 'a' and 'b')
  - Second partition: "b" (contains 1 distinct character: 'b')
  - Total: 2 partitions

- If we change the character at index 2 from 'b' to 'a':
  - New string: "aaaaab"
  - First partition: "aaaaa" (contains 1 distinct character: 'a')
  - Second partition: "b" (contains 1 distinct character: 'b')
  - Total: 2 partitions

The task is to determine the optimal change (if any) that maximizes the number of resulting partitions.

## Intuition

The key insight is that we need to explore two scenarios: partitioning with the original string and partitioning after changing one character. Since we want to maximize partitions, we need to consider what happens when we change any character at any position.

When we're processing the string character by character, at each position we face a decision point. If adding the current character to our current partition would exceed `k` distinct characters, we must start a new partition. This greedy approach of taking the longest possible prefix ensures we're not creating unnecessary partitions.

The challenge is handling the "change at most one character" constraint efficiently. We can't simply try changing every character to every possible letter and recompute partitions from scratch - that would be too expensive. Instead, we need a way to track our state as we go through the string.

We can think of this as a [dynamic programming](/problems/dynamic_programming_intro) problem where our state includes:
1. Our current position in the string (`i`)
2. The set of distinct characters in the current partition (`cur`)
3. Whether we still have the option to change a character (`t`)

For tracking distinct characters efficiently, we can use bit manipulation. Each bit in an integer represents whether a specific character (a-z) is present in the current partition. This allows us to quickly check how many distinct characters we have using `bit_count()` and combine character sets using bitwise OR.

At each position, we have several choices:
- If we haven't changed a character yet (`t = 1`), we can either:
  - Keep the current character and continue
  - Change it to any of the 26 letters and continue
- If we've already used our change (`t = 0`), we just process the current character

When the current partition would exceed `k` distinct characters after adding a character, we start a new partition. The recursion naturally handles trying all possibilities and returns the maximum number of partitions achievable.

The memoization (`@cache`) is crucial here because the same state (position, current character set, change availability) can be reached through different paths, and we don't want to recompute the result each time.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) and [Bitmask](/problems/bitmask_intro) patterns.**

## Solution Approach

The solution uses [dynamic programming](/problems/dynamic_programming_intro) with memoization to explore all possible partitioning strategies. Here's how the implementation works:

**State Representation:**
- `i`: Current position in the string
- `cur`: [Bitmask](/problems/bitmask_intro) representing the set of distinct characters in the current partition
- `t`: Boolean flag (1 or 0) indicating whether we can still change a character

**Main Function Setup:**
```python
n = len(s)
return dfs(0, 0, 1)
```
We start at position 0, with an empty character set ([bitmask](/problems/bitmask_intro) = 0), and the ability to change one character (`t = 1`).

**Recursive DFS Function:**

1. **Base Case:**
   ```python
   if i >= n:
       return 1
   ```
   When we've processed all characters, we have one final partition.

2. **Process Current Character:**
   ```python
   v = 1 << (ord(s[i]) - ord("a"))
   nxt = cur | v
   ```
   - Convert the current character to its corresponding bit position (0-25 for 'a'-'z')
   - Create a [bitmask](/problems/bitmask_intro) `v` with only that bit set
   - Combine with current partition's character set using bitwise OR

3. **Check if New Partition is Needed:**
   ```python
   if nxt.bit_count() > k:
       ans = dfs(i + 1, v, t) + 1
   else:
       ans = dfs(i + 1, nxt, t)
   ```
   - If adding the current character exceeds `k` distinct characters, start a new partition
   - The new partition begins with just the current character (`v`)
   - Add 1 to the partition count when starting a new partition
   - Otherwise, continue with the expanded character set

4. **Consider Changing Current Character (if allowed):**
   ```python
   if t:
       for j in range(26):
           nxt = cur | (1 << j)
           if nxt.bit_count() > k:
               ans = max(ans, dfs(i + 1, 1 << j, 0) + 1)
           else:
               ans = max(ans, dfs(i + 1, nxt, 0))
   ```
   - If we haven't used our change yet (`t = 1`), try changing the current character to each of the 26 letters
   - For each possible letter `j`:
     - Calculate what the character set would be if we change to that letter
     - If it exceeds `k` distinct characters, start a new partition
     - Otherwise, continue with the current partition
     - Set `t = 0` in recursive calls since we've used our change
     - Keep track of the maximum partitions achievable

**Memoization:**
The `@cache` decorator automatically memoizes the function results. This prevents redundant calculations when the same state `(i, cur, t)` is reached through different paths.

**Time Complexity:**
- Without memoization: O(26^n) in the worst case
- With memoization: O(n × 2^26 × 2) states, but in practice much fewer states are visited since we rarely have all 26 characters in a partition

**Space Complexity:**
O(n × 2^26 × 2) for the memoization cache in the worst case, though actual usage is typically much smaller.

## Example Walkthrough

Let's walk through a small example with `s = "abb"` and `k = 1` to illustrate how the solution explores different possibilities.

**Initial State:** We start with `dfs(0, 0, 1)` - position 0, empty character set (bitmask = 0), can still change a character.

**Step 1: Process index 0 (character 'a')**
- Current character 'a' has bitmask: `1 << 0 = 1` (bit 0 set)
- Adding 'a' to empty set: `0 | 1 = 1` (one distinct character)
- Since `bit_count(1) = 1 ≤ k`, we can add 'a' to current partition
- We explore two paths:

  **Path A: Keep 'a' unchanged**
  - Continue with `dfs(1, 1, 1)` - move to index 1, partition has {'a'}, still can change

  **Path B: Change 'a' to another letter**
  - Try all 26 letters. For example, changing 'a' to 'b':
    - New bitmask: `1 << 1 = 2` (bit 1 set for 'b')
    - Continue with `dfs(1, 2, 0)` - move to index 1, partition has {'b'}, used our change

**Step 2: Following Path A - at index 1 (character 'b')**
- State: `dfs(1, 1, 1)` - partition currently has {'a'}
- Current character 'b' has bitmask: `1 << 1 = 2`
- Adding 'b' to {'a'}: `1 | 2 = 3` (bits 0 and 1 set)
- Since `bit_count(3) = 2 > k`, we must start a new partition
- Result: `dfs(2, 2, 1) + 1` - new partition starts with {'b'}, add 1 for completing previous partition

**Step 3: Continuing Path A - at index 2 (character 'b')**
- State: `dfs(2, 2, 1)` - partition currently has {'b'}
- Current character 'b' has bitmask: `1 << 1 = 2`
- Adding 'b' to {'b'}: `2 | 2 = 2` (still just 'b')
- Since `bit_count(2) = 1 ≤ k`, we can add 'b' to current partition
- Continue with `dfs(3, 2, 1)`

**Step 4: Base case reached**
- `dfs(3, 2, 1)` returns 1 (one final partition)
- Backtracking: Step 3 returns 1, Step 2 returns 1+1=2, Step 1 Path A returns 2

**Alternative: Following Path B where we changed 'a' to 'b'**
- At index 1 with state `dfs(1, 2, 0)`, partition has {'b'}, no changes left
- Current character 'b' has bitmask: `2`
- Adding 'b' to {'b'}: `2 | 2 = 2` (still one distinct character)
- Continue to index 2, then base case
- This path gives us only 1 partition total: "bbb"

**Final Result:**
The algorithm explores all possibilities and returns the maximum. Path A (keeping original string) gives 2 partitions: "a" and "bb". This is better than changing 'a' to 'b' which gives only 1 partition: "bbb".

The memoization ensures that if we reach the same state `(i, cur, t)` through different paths, we only compute it once.

## Python Solution

```python
class Solution:
    def maxPartitionsAfterOperations(self, s: str, k: int) -> int:
        from functools import cache
      
        @cache
        def dfs(index: int, current_chars: int, can_change: int) -> int:
            """
            Dynamic programming function to find maximum partitions.
          
            Args:
                index: Current position in string
                current_chars: Bitmask representing characters in current partition
                can_change: Flag indicating if we can still change one character (1 = can change, 0 = already used)
          
            Returns:
                Maximum number of partitions from current position
            """
            # Base case: reached end of string
            if index >= n:
                return 1
          
            # Get bitmask for current character
            char_bit = 1 << (ord(s[index]) - ord('a'))
          
            # Add current character to the partition
            next_chars = current_chars | char_bit
          
            # Check if adding current character exceeds k distinct characters
            if next_chars.bit_count() > k:
                # Start new partition with current character
                result = dfs(index + 1, char_bit, can_change) + 1
            else:
                # Continue with current partition
                result = dfs(index + 1, next_chars, can_change)
          
            # If we can still change one character
            if can_change:
                # Try changing current character to each possible letter
                for letter_index in range(26):
                    # Create bitmask for the new character
                    next_chars = current_chars | (1 << letter_index)
                  
                    if next_chars.bit_count() > k:
                        # Start new partition with the changed character
                        result = max(result, dfs(index + 1, 1 << letter_index, 0) + 1)
                    else:
                        # Continue with current partition including changed character
                        result = max(result, dfs(index + 1, next_chars, 0))
          
            return result
      
        # Store string length
        n = len(s)
      
        # Start DFS from index 0, empty partition, with ability to change one character
        return dfs(0, 0, 1)
```

## Time and Space Complexity

**Time Complexity:** `O(n * 2^k * 26)`

The time complexity is determined by the memoized recursive function `dfs`:
- The function has three parameters: `i` (ranging from 0 to n), `cur` (a bitmask with at most k bits set), and `t` (binary: 0 or 1)
- The number of possible states for `cur` is at most `2^k` (all possible subsets of k distinct characters)
- The total number of unique states is `O(n * 2^k * 2)`
- For each state where `t = 1`, we try all 26 possible character replacements, adding a factor of 26
- Therefore, the overall time complexity is `O(n * 2^k * 26)`

**Space Complexity:** `O(n * 2^k)`

The space complexity consists of:
- The memoization cache stores at most `O(n * 2^k * 2) = O(n * 2^k)` states
- The recursion stack depth is at most `O(n)` in the worst case
- Since `2^k` is typically larger than `n` for reasonable values of k, the dominant factor is the cache
- Therefore, the overall space complexity is `O(n * 2^k)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Partition Boundary Handling**

**The Pitfall:**
A common mistake is misunderstanding when to create a new partition. Developers often incorrectly assume that when the character count exceeds `k`, they should:
- Keep the current partition as-is and start fresh with an empty partition
- Or include the problematic character in the current partition before splitting

**Incorrect Implementation:**
```python
if next_chars.bit_count() > k:
    # WRONG: Starting with empty partition
    result = dfs(index + 1, 0, can_change) + 1
    # Or WRONG: Including the character that causes overflow
    result = dfs(index, 0, can_change) + 1
```

**Why It's Wrong:**
When we exceed `k` distinct characters, we must:
1. End the current partition BEFORE adding the problematic character
2. Start a new partition WITH the current character (not empty)
3. Continue processing from the NEXT position

**Correct Implementation:**
```python
if next_chars.bit_count() > k:
    # Start new partition with ONLY the current character
    result = dfs(index + 1, char_bit, can_change) + 1
```

### 2. **Inefficient State Representation Leading to TLE**

**The Pitfall:**
Using a set or string to track distinct characters instead of a bitmask causes Time Limit Exceeded (TLE) due to:
- Expensive hashing operations for memoization
- Slower set operations compared to bitwise operations
- Larger memory footprint

**Incorrect Implementation:**
```python
@cache
def dfs(index: int, current_chars: frozenset, can_change: int) -> int:
    # Using frozenset for distinct characters
    next_chars = current_chars | {s[index]}
    if len(next_chars) > k:
        # ...
```

**Why It's Wrong:**
- `frozenset` operations are O(n) for union and require hashing for cache
- The cache size becomes unnecessarily large
- Bit operations are O(1) and integers hash efficiently

**Correct Implementation:**
```python
# Use bitmask for O(1) operations
char_bit = 1 << (ord(s[index]) - ord('a'))
next_chars = current_chars | char_bit
if next_chars.bit_count() > k:
    # ...
```

### 3. **Off-by-One Error in Partition Counting**

**The Pitfall:**
Incorrectly counting partitions, especially at the base case or when starting new partitions.

**Incorrect Implementation:**
```python
if index >= n:
    return 0  # WRONG: Missing the last partition
  
# Or when creating new partition:
if next_chars.bit_count() > k:
    result = dfs(index + 1, char_bit, can_change)  # WRONG: Forgot to add 1
```

**Why It's Wrong:**
- When we reach the end of the string, we still have one final partition to count
- When starting a new partition, we're completing the previous one, so we must add 1

**Correct Implementation:**
```python
if index >= n:
    return 1  # Count the final partition

if next_chars.bit_count() > k:
    result = dfs(index + 1, char_bit, can_change) + 1  # Add 1 for completed partition
```

### 4. **Exploring Redundant Character Changes**

**The Pitfall:**
When trying character changes, not optimizing which characters to try, leading to unnecessary computation.

**Suboptimal Approach:**
```python
if can_change:
    for letter_index in range(26):  # Always trying all 26 letters
        # ...
```

**Optimization:**
```python
if can_change:
    # Skip trying to change to the same character
    original_char = ord(s[index]) - ord('a')
    for letter_index in range(26):
        if letter_index == original_char:
            continue  # Skip redundant change
        # ...
```

**Better Optimization:**
Only try characters that would actually affect the partition boundary:
- Characters already in the current partition (won't increase distinct count)
- Characters that would trigger a new partition
- Skip characters that would result in the same distinct count without benefit

