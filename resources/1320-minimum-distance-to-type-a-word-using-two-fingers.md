# 1320. Minimum Distance to Type a Word Using Two Fingers

> Source: https://algo.monster/liteproblems/1320

## Problem Statement

You have a keyboard layout where English uppercase letters are arranged in a grid pattern. The keyboard has 26 letters arranged with 6 letters per row (except the last row which has 2 letters). The letters are positioned as follows:
- Row 0: A B C D E F (positions 0-5)
- Row 1: G H I J K L (positions 6-11)
- Row 2: M N O P Q R (positions 12-17)
- Row 3: S T U V W X (positions 18-23)
- Row 4: Y Z (positions 24-25)

Each letter has coordinates `(row, column)`. For example:
- Letter 'A' is at coordinate `(0, 0)`
- Letter 'B' is at coordinate `(0, 1)`
- Letter 'P' is at coordinate `(2, 3)`
- Letter 'Z' is at coordinate `(4, 1)`

You need to type a given string `word` using two fingers. The distance between any two coordinates `(x₁, y₁)` and `(x₂, y₂)` is calculated using Manhattan distance: `|x₁ - x₂| + |y₁ - y₂|`.

Key rules:
- You can use two fingers to type the word
- The initial positions of your fingers are free (don't count toward the total distance)
- Your fingers don't have to start at the first letter or first two letters
- Each letter in the word must be typed in order
- Only one finger moves to type each letter

The goal is to find the minimum total distance traveled by your fingers to type the entire word.

For example, if you need to type "CAKE":
- You could position one finger initially at 'C' and another at 'A' (no cost for initial positioning)
- Then move fingers optimally to type 'K' and 'E'
- The total distance would be the sum of all movements made after the initial positioning

Return the minimum total distance required to type the given word using two fingers.

## Intuition

The key insight is that at any point while typing, we need to know where both fingers are positioned to make optimal decisions about which finger to use for the next letter. This naturally leads to a [dynamic programming](/problems/dynamic_programming_intro) approach where we track the positions of both fingers after typing each character.

Think about it this way: when we need to type the next letter, we have two choices - use finger 1 or use finger 2. The optimal choice depends on:
1. Where each finger currently is
2. How far each finger would need to move
3. Where the remaining letters are (which affects future moves)

Since we can't predict all future moves easily, we can use [dynamic programming](/problems/dynamic_programming_intro) to explore all possibilities and keep track of the minimum cost for each possible configuration.

The state representation becomes clear: `f[i][j][k]` represents "after typing the i-th character, what's the minimum distance if finger 1 is at position j and finger 2 is at position k?"

For the base case, when typing the first letter, one finger moves to that letter (with 0 cost since initial positions are free) while the other finger can be anywhere. This gives us all possible starting configurations.

For subsequent letters, we need to consider:
- If we use finger 1 to type the current letter, finger 1 moves from its previous position to the new letter's position, while finger 2 stays put
- If we use finger 2 to type the current letter, finger 2 moves while finger 1 stays put

The tricky part is that we need to track which finger was at the previous letter's position. If finger 1 was at position `a` (previous letter) and we move it to position `b` (current letter), then we add `dist(a, b)` to our cost. But if finger 2 was at position `a`, then we need to consider all possible previous positions of finger 1 to find the minimum cost path.

By building up the solution letter by letter and considering all possible finger configurations, we ensure we find the globally optimal solution rather than getting stuck in a locally optimal but globally suboptimal choice.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The solution uses [dynamic programming](/problems/dynamic_programming_intro) with a 3D state array to track all possible finger positions after typing each character.

**State Definition:**
- `f[i][j][k]` represents the minimum distance after typing `word[i]`, with finger 1 at position `j` and finger 2 at position `k`
- Positions `j` and `k` range from 0 to 25 (representing letters A to Z)
- Initialize all states to infinity

**Distance Calculation:**
The `dist(a, b)` function calculates Manhattan distance between two letter positions:
- Convert position to row/column: `row = position // 6`, `column = position % 6`
- Distance = `|row1 - row2| + |col1 - col2|`

**Base Case (First Letter):**
When typing `word[0]`, we have two scenarios:
- Finger 1 at `word[0]`, finger 2 anywhere: `f[0][word[0]][k] = 0` for all `k ∈ [0, 25]`
- Finger 2 at `word[0]`, finger 1 anywhere: `f[0][k][word[0]] = 0` for all `k ∈ [0, 25]`

The cost is 0 because initial finger positions are free.

**Transition (Subsequent Letters):**
For each letter `word[i]` (where `i ≥ 1`):
- Let `a = word[i-1]` (previous letter position)
- Let `b = word[i]` (current letter position)

We consider two main cases:

**Case 1: Use finger 1 to type current letter (finger 1 moves to position `b`):**
- For each possible position `j` of finger 2:
  - If finger 1 was at position `a` previously: `f[i][b][j] = min(f[i][b][j], f[i-1][a][j] + dist(a, b))`
  - If finger 2 is at position `a` (i.e., `j = a`), then finger 2 typed the previous letter. We enumerate all possible previous positions `k` of finger 1: `f[i][b][j] = min(f[i][b][j], f[i-1][k][a] + dist(k, b))`

**Case 2: Use finger 2 to type current letter (finger 2 moves to position `b`):**
- For each possible position `j` of finger 1:
  - If finger 2 was at position `a` previously: `f[i][j][b] = min(f[i][j][b], f[i-1][j][a] + dist(a, b))`
  - If finger 1 is at position `a` (i.e., `j = a`), then finger 1 typed the previous letter. We enumerate all possible previous positions `k` of finger 2: `f[i][j][b] = min(f[i][j][b], f[i-1][a][k] + dist(k, b))`

**Final Answer:**
After processing all letters, the last letter must be at position `word[n-1]`. We find the minimum among:
- All states where finger 1 is at the last letter: `min(f[n-1][word[n-1]][j])` for all `j`
- All states where finger 2 is at the last letter: `min(f[n-1][j][word[n-1]])` for all `j`

The algorithm systematically explores all possible finger configurations and movement sequences, ensuring we find the globally optimal solution with time complexity `O(n × 26²)` and space complexity `O(n × 26²)`.

## Example Walkthrough

Let's walk through typing the word "HI" to illustrate the solution approach.

First, let's identify the letter positions:
- 'H' is at position 7 (row 1, column 1)
- 'I' is at position 8 (row 1, column 2)

**Step 1: Initialize the DP array**
- Create `f[2][26][26]` initialized to infinity
- `f[i][j][k]` = minimum distance after typing character i, with finger 1 at position j and finger 2 at position k

**Step 2: Base case - typing 'H' (word[0])**
Since initial finger positions are free, we set:
- Finger 1 at 'H', finger 2 anywhere: `f[0][7][k] = 0` for all k ∈ [0, 25]
- Finger 2 at 'H', finger 1 anywhere: `f[0][j][7] = 0` for all j ∈ [0, 25]

This gives us 52 possible starting configurations, all with cost 0.

**Step 3: Transition - typing 'I' (word[1])**
Previous letter: a = 7 ('H')
Current letter: b = 8 ('I')
Distance from 'H' to 'I': dist(7, 8) = |1-1| + |1-2| = 1

Now we consider all valid previous states and calculate new states:

**Case 1: Use finger 1 to type 'I' (finger 1 moves to position 8)**
- From state `f[0][7][k] = 0` (finger 1 was at 'H'):
  - `f[1][8][k] = min(∞, 0 + 1) = 1` for all k ≠ 7
- When k = 7 (both fingers were at 'H'), we need to check all possible previous positions of finger 1:
  - From state `f[0][j][7] = 0`: `f[1][8][7] = min(∞, 0 + dist(j, 8))`
  - The minimum occurs when j = 7: `f[1][8][7] = 0 + dist(7, 8) = 1`

**Case 2: Use finger 2 to type 'I' (finger 2 moves to position 8)**
- From state `f[0][j][7] = 0` (finger 2 was at 'H'):
  - `f[1][j][8] = min(∞, 0 + 1) = 1` for all j ≠ 7
- When j = 7 (both fingers were at 'H'), we need to check all possible previous positions of finger 2:
  - From state `f[0][7][k] = 0`: `f[1][7][8] = min(∞, 0 + dist(k, 8))`
  - The minimum occurs when k = 7: `f[1][7][8] = 0 + dist(7, 8) = 1`

**Step 4: Find the minimum distance**
The last letter 'I' is at position 8. We check:
- States where finger 1 is at 'I': `min(f[1][8][k])` for all k = min(1) = 1
- States where finger 2 is at 'I': `min(f[1][j][8])` for all j = min(1) = 1

**Result:** The minimum distance to type "HI" is 1.

This makes intuitive sense: we position both fingers initially at 'H' (free), then move one finger from 'H' to 'I' (distance = 1).

## Python Solution

```python
class Solution:
    def minimumDistance(self, word: str) -> int:
        """
        Calculate minimum distance to type a word using two fingers on a keyboard.
        The keyboard layout is A-Z arranged in a 6x5 grid (last row has 2 letters).
        """
      
        def calculate_distance(char1: int, char2: int) -> int:
            """
            Calculate Manhattan distance between two characters on the keyboard.
          
            Args:
                char1: First character index (0-25 for A-Z)
                char2: Second character index (0-25 for A-Z)
          
            Returns:
                Manhattan distance between the two positions
            """
            row1, col1 = divmod(char1, 6)
            row2, col2 = divmod(char2, 6)
            return abs(row1 - row2) + abs(col1 - col2)
      
        word_length = len(word)
      
        # dp[i][left_finger][right_finger] represents minimum distance after typing
        # word[0:i+1] with left finger at position left_finger and right finger at right_finger
        dp = [[[float('inf')] * 26 for _ in range(26)] for _ in range(word_length)]
      
        # Initialize first character - can be typed by either finger with 0 cost
        first_char_index = ord(word[0]) - ord('A')
        for finger_pos in range(26):
            dp[0][first_char_index][finger_pos] = 0  # Left finger typed first char
            dp[0][finger_pos][first_char_index] = 0  # Right finger typed first char
      
        # Process each subsequent character
        for i in range(1, word_length):
            prev_char_index = ord(word[i - 1]) - ord('A')
            curr_char_index = ord(word[i]) - ord('A')
            distance_between_chars = calculate_distance(prev_char_index, curr_char_index)
          
            for other_finger in range(26):
                # Case 1: Same finger that typed previous character types current character
                # Left finger was at prev_char, moves to curr_char
                dp[i][curr_char_index][other_finger] = min(
                    dp[i][curr_char_index][other_finger],
                    dp[i - 1][prev_char_index][other_finger] + distance_between_chars
                )
                # Right finger was at prev_char, moves to curr_char
                dp[i][other_finger][curr_char_index] = min(
                    dp[i][other_finger][curr_char_index],
                    dp[i - 1][other_finger][prev_char_index] + distance_between_chars
                )
              
                # Case 2: The other finger (not at prev_char) types current character
                if other_finger == prev_char_index:
                    for previous_position in range(26):
                        distance_to_current = calculate_distance(previous_position, curr_char_index)
                        # Left finger moves from previous_position to curr_char
                        dp[i][curr_char_index][other_finger] = min(
                            dp[i][curr_char_index][other_finger],
                            dp[i - 1][previous_position][prev_char_index] + distance_to_current
                        )
                        # Right finger moves from previous_position to curr_char
                        dp[i][other_finger][curr_char_index] = min(
                            dp[i][other_finger][curr_char_index],
                            dp[i - 1][prev_char_index][previous_position] + distance_to_current
                        )
      
        # Find minimum distance among all valid final states
        last_char_index = ord(word[-1]) - ord('A')
      
        # Minimum when left finger is at last character
        min_left_finger = min(dp[word_length - 1][last_char_index])
      
        # Minimum when right finger is at last character
        min_right_finger = min(dp[word_length - 1][j][last_char_index] for j in range(26))
      
        return int(min(min_left_finger, min_right_finger))
```

## Time and Space Complexity

The time complexity is `O(n × |Σ|²)`, where `n` is the length of the string `word` and `|Σ| = 26` represents the size of the alphabet.

The analysis breaks down as follows:
- The outer loop runs `n` times (for each character in the word)
- For each iteration `i`, we iterate through all 26 possible positions for finger `j`, contributing `O(|Σ|)` operations
- Within the special case when `j == a`, we have another loop through all 26 positions `k`, adding another `O(|Σ|)` factor
- Overall, this gives us `O(n × |Σ| × |Σ|) = O(n × |Σ|²)`

The space complexity is `O(n × |Σ|²)` due to the 3-dimensional DP array `f` with dimensions `[n][26][26]`, which stores the minimum distance for each position `i` in the word with the two fingers at positions `[left_finger][right_finger]`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Assumption About Initial Finger Positions**

**Pitfall:** Many solutions incorrectly assume that fingers must start at the first one or two letters of the word, leading to suboptimal solutions.

**Example:** For the word "HAPPY", you might assume:
- Finger 1 starts at 'H', Finger 2 starts at 'A'
- Or both fingers start at 'H'

**Why it's wrong:** The problem states that initial finger positions are free (cost = 0). Fingers can start anywhere on the keyboard, not necessarily at the beginning letters.

**Correct Approach:** Initialize the DP state for the first letter with both possibilities:
- One finger at the first letter, other finger anywhere (cost = 0)
- This allows the algorithm to find the optimal starting position for the idle finger

### 2. **Mishandling the Transition Between Letters**

**Pitfall:** Failing to correctly track which finger typed the previous letter when calculating transitions.

**Problematic Code Example:**
```python
# WRONG: Only considering the case where the same finger continues typing
dp[i][curr][other] = dp[i-1][prev][other] + dist(prev, curr)
```

**Why it's wrong:** This doesn't account for the case where the other finger (currently at position `other`) might have typed the previous letter.

**Correct Implementation:**
```python
# Must check if other_finger is at prev_char position
if other_finger == prev_char_index:
    # The other finger typed previous letter, so we need to consider
    # all possible previous positions of the current moving finger
    for previous_position in range(26):
        distance = calculate_distance(previous_position, curr_char_index)
        dp[i][curr_char_index][other_finger] = min(
            dp[i][curr_char_index][other_finger],
            dp[i-1][previous_position][prev_char_index] + distance
        )
```

### 3. **Space Optimization Trap**

**Pitfall:** Attempting to optimize space by using only 2D DP array instead of 3D.

**Problematic Approach:**
```python
# WRONG: Trying to use dp[finger1_pos][finger2_pos] without tracking progress
dp = [[float('inf')] * 26 for _ in range(26)]
```

**Why it fails:** Without tracking which character we're currently typing (the `i` dimension), we lose critical information about the sequence and can't correctly compute transitions.

**Solution:** Maintain the full 3D state or use rolling array optimization:
```python
# Option 1: Full 3D array (shown in solution)
dp = [[[float('inf')] * 26 for _ in range(26)] for _ in range(word_length)]

# Option 2: Rolling array optimization (if space is critical)
prev_dp = [[float('inf')] * 26 for _ in range(26)]
curr_dp = [[float('inf')] * 26 for _ in range(26)]
# Swap prev_dp and curr_dp after each iteration
```

### 4. **Incorrect Distance Calculation**

**Pitfall:** Using Euclidean distance or incorrect grid mapping.

**Wrong Example:**
```python
# WRONG: Using Euclidean distance
import math
distance = math.sqrt((row1 - row2)**2 + (col1 - col2)**2)

# WRONG: Incorrect grid layout (assuming 5 columns)
row, col = divmod(char_index, 5)
```

**Correct Approach:**
```python
# Manhattan distance with 6 columns per row
row, col = divmod(char_index, 6)
distance = abs(row1 - row2) + abs(col1 - col2)
```

### 5. **Edge Case: Single Character Word**

**Pitfall:** Not handling words with length 1 correctly.

**Issue:** The algorithm might fail or return incorrect results for single-character words.

**Solution:** The current implementation handles this correctly by:
- Initializing both finger positions for the first (and only) character
- Returning 0 as the minimum distance (no movement needed after initial positioning)

**Test Case:**
```python
assert minimumDistance("A") == 0
assert minimumDistance("Z") == 0
```

