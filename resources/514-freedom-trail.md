# 514. Freedom Trail

> Source: https://algo.monster/liteproblems/514

## Problem Statement

You're given a circular dial (like a combination lock) represented by a string `ring`, and you need to spell out a keyword represented by string `key`. The dial starts with its first character at the "12:00" position.

To spell each character in `key`, you need to:
1. **Rotate the ring** clockwise or counterclockwise to bring the desired character to the "12:00" position. Each position moved counts as one step.
2. **Press the center button** to confirm the character, which counts as one additional step.

Since the ring is circular, you can rotate in either direction to reach any character. For example, if the ring has length `n`, moving from position `i` to position `j` takes either `|i - j|` steps or `n - |i - j|` steps, whichever is smaller.

The goal is to find the **minimum total number of steps** needed to spell all characters in `key` in order.

**Example walkthrough:**
- If `ring = "godding"` and `key = "gd"`
- Initially, 'g' (at index 0) is at "12:00", so spelling 'g' takes 0 rotations + 1 button press = 1 step
- Next, to spell 'd', we can rotate to index 2 (clockwise 2 steps) or index 6 (counterclockwise 1 step)
- Choosing index 6: 1 rotation + 1 button press = 2 steps
- Total: 1 + 2 = 3 steps

## Intuition

The key insight is recognizing that at each step, we face a decision problem: which occurrence of the next character should we rotate to? Since the ring is circular and characters can repeat, we have multiple choices for each character we need to spell.

Think of it like planning a route through multiple cities where some city names repeat. At each step, you need to decide which instance of the next city to visit, considering both the distance from your current position and the future destinations you'll need to reach.

This naturally leads to a **state-based optimization problem**. Our state can be defined as: "I've spelled the first `i` characters of the key, and I'm currently at position `j` on the ring." From each state, we can transition to any position where the next required character appears.

The recursive structure becomes clear:
- To find the minimum steps to spell all characters ending at position `j`
- We need to know the minimum steps to spell all previous characters ending at some position `k`
- Plus the cost to rotate from `k` to `j`
- Plus 1 for pressing the button

Since the same subproblems (spelling the first `i` characters and ending at position `j`) will be encountered multiple times through different paths, we can use **[dynamic programming](/problems/dynamic_programming_intro)** to avoid recalculating them.

The DP approach emerges naturally: `f[i][j]` represents the minimum steps to spell the first `i+1` characters with position `j` aligned at 12:00. We build up the solution by considering all possible ways to reach each state and keeping only the minimum cost path. This is essentially DFS with memoization, where we explore all paths but remember the best solution for each state we've visited.

**Learn more about [Depth-First Search](/problems/dfs_intro), [Breadth-First Search](/problems/bfs_intro) and [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The implementation uses [dynamic programming](/problems/dynamic_programming_intro) with the following components:

**1. Preprocessing Character Positions**

First, we create a dictionary `pos` that maps each character to its list of positions in the ring:
```python
pos = defaultdict(list)
for i, c in enumerate(ring):
    pos[c].append(i)
```
This allows us to quickly find all occurrences of any character we need to spell.

**2. DP Table Initialization**

We create a 2D DP table `f[i][j]` where:
- `i` represents the index in the `key` (which character we're spelling)
- `j` represents the position in the `ring` (where we're currently aligned)
- `f[i][j]` stores the minimum steps to spell the first `i+1` characters with position `j` at 12:00

```python
f = [[inf] * n for _ in range(m)]
```

**3. Base Case**

For the first character of the key, we calculate the cost to rotate to each occurrence of that character from position 0:
```python
for j in pos[key[0]]:
    f[0][j] = min(j, n - j) + 1
```
The rotation cost is `min(j, n - j)` because we can rotate either clockwise or counterclockwise. We add 1 for the button press.

**4. State Transitions**

For each subsequent character `key[i]`, we consider:
- All positions `j` where `key[i]` appears in the ring
- All positions `k` where the previous character `key[i-1]` appeared

The transition formula is:
```python
f[i][j] = min(f[i][j], f[i-1][k] + min(abs(j - k), n - abs(j - k)) + 1)
```

This calculates:
- The cost to spell the first `i` characters ending at position `k`: `f[i-1][k]`
- The rotation cost from position `k` to position `j`: `min(abs(j - k), n - abs(j - k))`
- Plus 1 for pressing the button

**5. Finding the Answer**

The final answer is the minimum value among all possible ending positions for the last character:
```python
return min(f[-1][j] for j in pos[key[-1]])
```

**Time Complexity:** `O(m × n²)` where `m` is the length of the key and `n` is the length of the ring. In the worst case, each character appears at every position in the ring.

**Space Complexity:** `O(m × n)` for the DP table plus `O(n)` for the position dictionary.

## Example Walkthrough

Let's trace through a small example with `ring = "godding"` and `key = "gd"`:

**Initial Setup:**
- Ring: "godding" (length n = 7)
- Key: "gd" (length m = 2)
- Starting position: index 0 ('g' is at 12:00)

**Step 1: Preprocess character positions**
```
pos = {
    'g': [0, 6],  # 'g' appears at indices 0 and 6
    'o': [1],     # 'o' appears at index 1
    'd': [2, 3],  # 'd' appears at indices 2 and 3
    'i': [4],     # 'i' appears at index 4
    'n': [5]      # 'n' appears at index 5
}
```

**Step 2: Initialize DP table**
```
f = [[inf, inf, inf, inf, inf, inf, inf],
     [inf, inf, inf, inf, inf, inf, inf]]
```

**Step 3: Base case - spelling first character 'g'**

Since 'g' appears at positions [0, 6]:
- Position 0: Rotation cost = min(0, 7-0) = 0, Total = 0 + 1 = 1
- Position 6: Rotation cost = min(6, 7-6) = 1, Total = 1 + 1 = 2

```
f[0] = [1, inf, inf, inf, inf, inf, 2]
```

**Step 4: Spell second character 'd'**

'd' appears at positions [2, 3]. We check transitions from previous 'g' positions:

For position j=2 (spelling 'd' at index 2):
- From k=0: f[0][0] + rotation(0→2) + 1 = 1 + min(2, 7-2) + 1 = 1 + 2 + 1 = 4
- From k=6: f[0][6] + rotation(6→2) + 1 = 2 + min(4, 7-4) + 1 = 2 + 3 + 1 = 6
- Choose minimum: f[1][2] = 4

For position j=3 (spelling 'd' at index 3):
- From k=0: f[0][0] + rotation(0→3) + 1 = 1 + min(3, 7-3) + 1 = 1 + 3 + 1 = 5
- From k=6: f[0][6] + rotation(6→3) + 1 = 2 + min(3, 7-3) + 1 = 2 + 3 + 1 = 6
- Choose minimum: f[1][3] = 5

```
f[1] = [inf, inf, 4, 5, inf, inf, inf]
```

**Step 5: Find the answer**

The minimum among all valid ending positions for 'd':
- min(f[1][2], f[1][3]) = min(4, 5) = 4

**Final answer: 4 steps**

The optimal path is:
1. Start at position 0 with 'g' → Press button (1 step)
2. Rotate clockwise 2 positions to reach 'd' at index 2 → 2 rotations + 1 button press (3 steps)
3. Total: 1 + 3 = 4 steps

## Python Solution

```python
from collections import defaultdict
from typing import List

class Solution:
    def findRotateSteps(self, ring: str, key: str) -> int:
        key_length, ring_length = len(key), len(ring)
      
        # Store all positions of each character in the ring
        char_positions = defaultdict(list)
        for index, char in enumerate(ring):
            char_positions[char].append(index)
      
        # dp[i][j] = minimum steps to spell key[0..i] with ring[j] at 12:00
        # Initialize with infinity for all states
        dp = [[float('inf')] * ring_length for _ in range(key_length)]
      
        # Base case: spell the first character of key
        # For each position where key[0] appears in ring
        for position in char_positions[key[0]]:
            # Calculate minimum rotation steps (clockwise or counter-clockwise)
            # Plus 1 for pressing the button
            dp[0][position] = min(position, ring_length - position) + 1
      
        # Fill dp table for remaining characters in key
        for key_index in range(1, key_length):
            # For each position where current key character appears in ring
            for current_pos in char_positions[key[key_index]]:
                # Try all possible previous positions
                for prev_pos in char_positions[key[key_index - 1]]:
                    # Calculate rotation distance between positions
                    distance = abs(current_pos - prev_pos)
                    # Choose minimum between clockwise and counter-clockwise rotation
                    min_rotation = min(distance, ring_length - distance)
                    # Update dp with minimum cost
                    dp[key_index][current_pos] = min(
                        dp[key_index][current_pos], 
                        dp[key_index - 1][prev_pos] + min_rotation + 1
                    )
      
        # Return minimum steps among all possible final positions
        return min(dp[-1][position] for position in char_positions[key[-1]])
```

## Time and Space Complexity

**Time Complexity:** `O(m × n²)`

The algorithm uses dynamic programming with the following time analysis:
- Building the position dictionary `pos` takes `O(n)` time where `n = len(ring)`
- The main DP computation has three nested loops:
  - Outer loop iterates through each character in `key`: `O(m)` where `m = len(key)`
  - Middle loop iterates through all positions of `key[i]` in the ring: worst case `O(n)` positions
  - Inner loop iterates through all positions of `key[i-1]` in the ring: worst case `O(n)` positions
- The final minimum calculation takes `O(n)` in the worst case

The dominant operation is the triple nested loop structure, giving us `O(m × n × n) = O(m × n²)`

**Space Complexity:** `O(m × n)`

The space usage consists of:
- The position dictionary `pos`: `O(n)` space to store all character positions
- The DP table `f`: a 2D array of size `m × n`, requiring `O(m × n)` space
- Other variables use constant space

The DP table dominates the space complexity, resulting in `O(m × n)` total space.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Forgetting the Circular Nature of the Ring**

A common mistake is calculating the rotation distance as simply `abs(j - k)` without considering that the ring is circular. This leads to suboptimal or incorrect results.

**Incorrect approach:**
```python
# Wrong: Only considers linear distance
rotation_cost = abs(current_pos - prev_pos)
```

**Correct approach:**
```python
# Consider both clockwise and counterclockwise paths
distance = abs(current_pos - prev_pos)
rotation_cost = min(distance, ring_length - distance)
```

### 2. **Off-by-One Error with Button Presses**

Many implementations forget to add the button press step for each character, or accidentally add it multiple times in the calculation.

**Incorrect approach:**
```python
# Forgetting the button press
dp[i][j] = dp[i-1][k] + min_rotation  # Missing +1

# Or double-counting in base case
dp[0][position] = min(position, ring_length - position) + 1
# Then later also adding +1 in transitions
```

**Correct approach:**
```python
# Consistently add exactly one button press per character
dp[i][j] = dp[i-1][k] + min_rotation + 1  # Always include the +1
```

### 3. **Improper Base Case Initialization**

Starting from an incorrect initial position or not properly handling the first character's rotation from position 0.

**Incorrect approach:**
```python
# Wrong: Assumes we can start from any position
for j in char_positions[key[0]]:
    dp[0][j] = 1  # Ignores rotation cost from initial position
```

**Correct approach:**
```python
# Account for rotation from starting position (index 0)
for j in char_positions[key[0]]:
    dp[0][j] = min(j, ring_length - j) + 1
```

### 4. **Not Handling Duplicate Characters Properly**

When a character appears multiple times in the ring, failing to consider all occurrences can lead to missing the optimal solution.

**Incorrect approach:**
```python
# Only considering the first occurrence
position = ring.index(key[i])  # Gets only the first occurrence
```

**Correct approach:**
```python
# Consider all positions where the character appears
for position in char_positions[key[i]]:
    # Process each possible position
```

### 5. **Memory Optimization Oversight**

While the current solution works, it uses O(m × n) space. For very large inputs, this could be optimized to O(n) since we only need the previous row of the DP table.

**Space-optimized approach:**
```python
def findRotateSteps(self, ring: str, key: str) -> int:
    n = len(ring)
    char_positions = defaultdict(list)
    for i, c in enumerate(ring):
        char_positions[c].append(i)
  
    # Use only two rows instead of full table
    prev = [float('inf')] * n
    curr = [float('inf')] * n
  
    # Base case
    for j in char_positions[key[0]]:
        prev[j] = min(j, n - j) + 1
  
    # Process remaining characters
    for i in range(1, len(key)):
        curr = [float('inf')] * n
        for j in char_positions[key[i]]:
            for k in char_positions[key[i-1]]:
                dist = abs(j - k)
                curr[j] = min(curr[j], prev[k] + min(dist, n - dist) + 1)
        prev = curr
  
    return min(prev[j] for j in char_positions[key[-1]])
```

