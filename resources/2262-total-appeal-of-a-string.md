# 2262. Total Appeal of A String

> Source: https://algo.monster/liteproblems/2262

## Problem Statement

The **appeal** of a string is defined as the number of **distinct** characters found in that string.

For example:
- The appeal of `"abbca"` is `3` because it contains 3 distinct characters: `'a'`, `'b'`, and `'c'`.

Given a string `s`, you need to find the **total appeal of all of its substrings**.

A **substring** is a contiguous sequence of characters within a string. This means you need to:
1. Find all possible substrings of the given string
2. Calculate the appeal (number of distinct characters) for each substring
3. Return the sum of all these appeal values

For instance, if `s = "abc"`, the substrings would be:
- `"a"` with appeal = 1
- `"b"` with appeal = 1  
- `"c"` with appeal = 1
- `"ab"` with appeal = 2
- `"bc"` with appeal = 2
- `"abc"` with appeal = 3

The total appeal sum would be `1 + 1 + 1 + 2 + 2 + 3 = 10`.

The challenge is to efficiently calculate this sum for potentially long strings, as the naive approach of generating all substrings and counting distinct characters for each would be computationally expensive.

## Intuition

Instead of calculating the appeal for each substring individually (which would be inefficient), let's think about how the total appeal changes as we build substrings incrementally.

Consider what happens when we process each character position `i` in the string. At position `i`, we're looking at all substrings that **end** at this position. These substrings are:
- `s[0...i]`
- `s[1...i]`
- `s[2...i]`
- ... 
- `s[i]` (just the character itself)

The key insight is that when we move from position `i-1` to position `i`, we're essentially adding character `s[i]` to the end of all substrings that ended at position `i-1`, plus creating a new substring with just `s[i]` itself.

Now, how does adding `s[i]` affect the appeal of these substrings?

**Case 1: Character `s[i]` hasn't appeared before**
- Every substring ending at `i-1` gets its appeal increased by 1 (since we're adding a new distinct character)
- There are exactly `i` such substrings (starting from positions 0, 1, 2, ..., i-1)
- Plus the single character substring `s[i]` itself has appeal 1
- So the total contribution increases by `i + 1`

**Case 2: Character `s[i]` has appeared before at position `j`**
- Substrings that already contain this character (those starting at or before position `j`) don't get their appeal increased
- Only substrings starting after position `j` will see their appeal increase by 1
- There are `i - j - 1` such substrings (starting from positions j+1, j+2, ..., i-1)
- Plus the single character substring `s[i]` itself has appeal 1
- So the total contribution increases by `i - j`

This leads us to maintain:
1. A running sum `t` that tracks the total appeal of all substrings ending at the current position
2. An array `pos` that stores the last seen position of each character (initialized to -1)

At each position `i`, we update `t` by adding `i - pos[c]` where `c` is the current character. This formula works for both cases above (when `pos[c] = -1` for unseen characters, we get `i - (-1) = i + 1`).

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

Based on our intuition, we'll implement a solution that processes each character position and maintains the cumulative appeal sum efficiently.

**Data Structures:**
- `ans`: Accumulates the total appeal sum of all substrings
- `t`: Tracks the sum of appeals for all substrings ending at the current position
- `pos`: An array of size 26 (for lowercase English letters) that stores the last occurrence position of each character, initialized to -1

**Algorithm Steps:**

1. **Initialize variables:**
   ```python
   ans = t = 0
   pos = [-1] * 26  # Last position of each character ('a' to 'z')
   ```

2. **Process each character in the string:**
   For each position `i` with character `s[i]`:
 
   a. **Convert character to index:**
   ```python
   c = ord(s[i]) - ord('a')  # Maps 'a'->0, 'b'->1, ..., 'z'->25
   ```
 
   b. **Update the appeal sum for substrings ending at position i:**
   ```python
   t += i - pos[c]
   ```
   - If character hasn't appeared before (`pos[c] = -1`), this adds `i + 1`
   - If character appeared at position `j` (`pos[c] = j`), this adds `i - j`
   - This represents the contribution of character `s[i]` to all substrings ending at position `i`
 
   c. **Add current position's total to answer:**
   ```python
   ans += t
   ```
   This accumulates the appeal of all substrings ending at position `i`
 
   d. **Update last occurrence position:**
   ```python
   pos[c] = i
   ```

3. **Return the total appeal sum:**
   ```python
   return ans
   ```

**Example Walkthrough:**
For `s = "abc"`:
- Position 0 ('a'): `t = 0 + (0 - (-1)) = 1`, `ans = 1`, update `pos[0] = 0`
- Position 1 ('b'): `t = 1 + (1 - (-1)) = 3`, `ans = 1 + 3 = 4`, update `pos[1] = 1`
- Position 2 ('c'): `t = 3 + (2 - (-1)) = 6`, `ans = 4 + 6 = 10`, update `pos[2] = 2`

The final answer is 10, which matches our manual calculation from the problem description.

**Time Complexity:** `O(n)` where n is the length of the string, as we process each character once.

**Space Complexity:** `O(1)` as we use a fixed-size array of 26 elements regardless of input size.

## Example Walkthrough

Let's walk through the solution with `s = "abbca"` to see how the algorithm efficiently calculates the total appeal.

**Initial Setup:**
- `ans = 0` (total appeal sum)
- `t = 0` (appeal sum for substrings ending at current position)
- `pos = [-1, -1, -1, ..., -1]` (26 elements, all initialized to -1)

**Position 0: character 'a'**
- Character index: `c = ord('a') - ord('a') = 0`
- Update t: `t = 0 + (0 - (-1)) = 1`
  - This means substring "a" has appeal 1
- Update ans: `ans = 0 + 1 = 1`
- Update pos: `pos[0] = 0` (mark 'a' seen at position 0)

**Position 1: character 'b'**
- Character index: `c = ord('b') - ord('a') = 1`
- Update t: `t = 1 + (1 - (-1)) = 3`
  - Substrings ending here: "b" (appeal 1), "ab" (appeal 2)
  - Total appeal = 1 + 2 = 3
- Update ans: `ans = 1 + 3 = 4`
- Update pos: `pos[1] = 1` (mark 'b' seen at position 1)

**Position 2: character 'b'**
- Character index: `c = 1`
- Update t: `t = 3 + (2 - 1) = 4`
  - 'b' was last seen at position 1
  - Only substring "b" (starting after position 1) gets +1 appeal
  - Substrings: "b" (appeal 1), "bb" (appeal 1), "abb" (appeal 2)
  - Total appeal = 1 + 1 + 2 = 4
- Update ans: `ans = 4 + 4 = 8`
- Update pos: `pos[1] = 2` (update 'b' position to 2)

**Position 3: character 'c'**
- Character index: `c = ord('c') - ord('a') = 2`
- Update t: `t = 4 + (3 - (-1)) = 8`
  - 'c' hasn't appeared before, so all 4 substrings get +1 appeal
  - Substrings: "c" (appeal 1), "bc" (appeal 2), "bbc" (appeal 2), "abbc" (appeal 3)
  - Total appeal = 1 + 2 + 2 + 3 = 8
- Update ans: `ans = 8 + 8 = 16`
- Update pos: `pos[2] = 3` (mark 'c' seen at position 3)

**Position 4: character 'a'**
- Character index: `c = 0`
- Update t: `t = 8 + (4 - 0) = 12`
  - 'a' was last seen at position 0
  - Substrings starting after position 0 get +1 appeal
  - Substrings: "a" (appeal 1), "ca" (appeal 2), "bca" (appeal 3), "bbca" (appeal 3), "abbca" (appeal 3)
  - Total appeal = 1 + 2 + 3 + 3 + 3 = 12
- Update ans: `ans = 16 + 12 = 28`
- Update pos: `pos[0] = 4` (update 'a' position to 4)

**Final Result: 28**

The algorithm correctly computes that the total appeal of all substrings of "abbca" is 28, processing each character only once in O(n) time.

## Python Solution

```python
class Solution:
    def appealSum(self, s: str) -> int:
        # Total sum of appeals across all substrings
        total_appeal_sum = 0
      
        # Current contribution to appeal from substrings ending at current position
        current_contribution = 0
      
        # Track the last seen position of each character (a-z)
        # Initialize with -1 to indicate character hasn't been seen yet
        last_position = [-1] * 26
      
        # Iterate through each character in the string
        for current_index, char in enumerate(s):
            # Convert character to index (0-25 for a-z)
            char_index = ord(char) - ord('a')
          
            # Calculate new substrings that include current character
            # These are all substrings from (last occurrence of char + 1) to current_index
            # Each adds 1 to the appeal count if char wasn't in that substring before
            current_contribution += current_index - last_position[char_index]
          
            # Add current contribution to total
            # This represents appeal sum of all substrings ending at current_index
            total_appeal_sum += current_contribution
          
            # Update last seen position of current character
            last_position[char_index] = current_index
      
        return total_appeal_sum
```

## Time and Space Complexity

The time complexity is `O(n)`, where `n` is the length of the string `s`. The algorithm iterates through the string once with a single for loop, and within each iteration, it performs constant-time operations: calculating the character index, updating the running sum `t`, adding to the answer `ans`, and updating the position array `pos[c]`.

The space complexity is `O(|Σ|)` or equivalently `O(26)` which simplifies to `O(1)`, where `|Σ|` represents the size of the character set. In this problem, the character set consists of lowercase English letters, so `|Σ| = 26`. The algorithm uses a fixed-size array `pos` of length 26 to track the last occurrence position of each character, regardless of the input string length.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Misunderstanding the Contribution Update Logic**

**Pitfall:** Many developers struggle to understand why we use `t += i - pos[c]` instead of simply counting distinct characters for each substring. The confusion often leads to incorrect implementations that try to maintain a set of characters or recalculate appeals from scratch.

**Why it happens:** The formula `i - pos[c]` represents how many new substrings ending at position `i` will have character `c` contributing to their appeal. This is counterintuitive because we're not directly counting distinct characters.

**Solution:** Understand that when character `c` appears at position `i`:
- All substrings starting from positions `pos[c] + 1` to `i` and ending at `i` will have `c` as a new distinct character
- The count of such substrings is exactly `i - pos[c]`
- These substrings inherit the appeal contributions from previous characters plus this new contribution

### 2. **Incorrect Initialization of Last Position Array**

**Pitfall:** Initializing the `last_position` array with `0` instead of `-1`:
```python
# Wrong
last_position = [0] * 26  

# Correct
last_position = [-1] * 26
```

**Why it happens:** Developers might think position indices start at 0, so they initialize with 0. However, this causes incorrect calculations for the first occurrence of each character.

**Solution:** Always initialize with `-1` to indicate "not yet seen". This ensures that when a character appears for the first time at position `i`, the contribution calculation `i - (-1) = i + 1` correctly accounts for all `i + 1` substrings that start from position 0 to `i` and end at `i`.

### 3. **Forgetting to Accumulate the Running Sum**

**Pitfall:** Directly adding the contribution to the final answer without maintaining the running sum:
```python
# Wrong approach
for i, char in enumerate(s):
    char_index = ord(char) - ord('a')
    total_appeal_sum += i - last_position[char_index]  # Missing accumulation
    last_position[char_index] = i
```

**Why it happens:** The algorithm requires tracking two separate values: the cumulative contribution (`current_contribution`) and the total appeal sum. The contribution at each position builds upon the previous position's contribution.

**Solution:** Maintain both variables:
- `current_contribution`: Accumulates the appeal sum for all substrings ending at the current position
- `total_appeal_sum`: Accumulates the total appeal across all substrings
The relationship is: `current_contribution` grows incrementally, and we add it to `total_appeal_sum` at each step.

### 4. **Character Index Calculation Errors**

**Pitfall:** Incorrect character-to-index conversion, especially when dealing with uppercase letters or mixed cases:
```python
# Wrong for lowercase only
char_index = ord(char) - ord('A')  # Uses 'A' instead of 'a'

# Wrong for mixed case without handling
char_index = ord(char) - ord('a')  # Fails for uppercase letters
```

**Why it happens:** The problem typically specifies lowercase English letters, but developers might accidentally use the wrong base character or forget to handle case sensitivity if the problem allows mixed cases.

**Solution:** 
- For lowercase only: Use `ord(char) - ord('a')`
- For uppercase only: Use `ord(char) - ord('A')`
- For mixed case: Convert to lowercase first with `char.lower()` or handle both cases separately with a larger position array (size 52)

### 5. **Off-by-One Errors in Position Tracking**

**Pitfall:** Confusion about whether to update the position before or after calculating the contribution:
```python
# Wrong order
last_position[char_index] = current_index  # Updated too early
current_contribution += current_index - last_position[char_index]  # Now always 0!
```

**Why it happens:** The order of operations matters. We need the previous position to calculate the contribution before updating it.

**Solution:** Always follow this order:
1. Calculate the contribution using the old position: `current_contribution += i - last_position[char_index]`
2. Add to total: `total_appeal_sum += current_contribution`
3. Update the position for next iteration: `last_position[char_index] = i`

