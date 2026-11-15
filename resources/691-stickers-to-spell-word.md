# 691. Stickers to Spell Word

> Source: https://algo.monster/liteproblems/691

## Problem Statement

You have `n` different types of stickers, where each sticker contains a lowercase English word. You need to spell out a given `target` string by cutting individual letters from your stickers and rearranging them.

Key points:
- You can use each sticker multiple times (infinite supply of each sticker type)
- You can cut individual letters from any sticker and use them in any order
- The goal is to collect enough letters to spell the entire `target` string

Your task is to find the **minimum number of stickers** needed to spell out the `target`. If it's impossible to spell the target using the available stickers, return `-1`.

For example, if you have stickers with words `["with", "example", "science"]` and need to spell `"thehat"`:
- You could use the sticker `"with"` to get letters 't' and 'h'
- Use another `"with"` to get another 'h'
- Use `"example"` to get 'e' and 'a'
- Use `"with"` again to get the final 't'
- This would require 4 stickers total (though this might not be the optimal solution)

## Intuition

When we first look at this problem, we need to find the minimum number of stickers to spell out a target string. This immediately suggests we need to explore different combinations of stickers and find the optimal one.

The key insight is recognizing that the target string has a maximum length of 15 characters. This small constraint is crucial - it means we can track which characters we've collected using a [bitmask](/problems/bitmask_intro). Each bit position represents whether we've collected the character at that position in the target string.

Why BFS instead of traditional [backtracking](/problems/backtracking)? Think about the problem structure:
- We want the **minimum** number of stickers (shortest path)
- Each "step" involves using one sticker
- We can represent our progress as states (which characters we've collected so far)

This naturally fits a BFS pattern where we explore all possible states level by level. Each level represents using one additional sticker. The first time we reach the complete state (all characters collected), we've found the minimum number of stickers needed.

The state compression technique comes from observing that:
- We only care about **which** characters we've collected, not the order
- With 15 characters max, we have at most `2^15 = 32,768` possible states
- We can represent each state as an integer where bit `i` indicates if we've collected the `i`-th character

For each state, we try adding each available sticker. When we use a sticker, we greedily take as many useful characters as possible from it (characters that we still need). This generates a new state with more bits set to 1.

The visited array prevents us from exploring the same state multiple times - once we've reached a particular combination of collected characters with `k` stickers, there's no point reaching the same combination with more stickers.

**Learn more about [Memoization](/problems/memoization_intro), [Dynamic Programming](/problems/dynamic_programming_intro), [Backtracking](/problems/backtracking) and [Bitmask](/problems/bitmask_intro) patterns.**

## Solution Approach

The solution implements BFS with state compression to find the minimum number of stickers needed.

**State Representation:**
- We use a [bitmask](/problems/bitmask_intro) to represent which characters of the target have been collected
- If bit `i` is set to 1, it means the character at position `i` in the target has been collected
- Initial state is `0` (no characters collected)
- Goal state is `(1 << n) - 1` (all bits set, all characters collected)

**BFS Implementation:**
1. **Initialize the queue** with state `0` and mark it as visited
2. **Process level by level** (each level represents using one more sticker):
   ```python
   for _ in range(len(q)):  # Process all states at current level
   ```

3. **For each state, try all stickers:**
   - Create a counter for the current sticker's characters: `cnt = Counter(s)`
   - Start with the current state: `nxt = cur`
 
4. **Greedily collect characters from the sticker:**
   ```python
   for i, c in enumerate(target):
       if (cur >> i & 1) == 0 and cnt[c] > 0:  # Character not collected yet and available
           cnt[c] -= 1      # Use this character from sticker
           nxt |= 1 << i    # Set bit i to 1 (mark as collected)
   ```
   - `(cur >> i & 1) == 0` checks if bit `i` is 0 (character not collected)
   - `cnt[c] > 0` checks if the sticker has this character available
   - `nxt |= 1 << i` sets bit `i` to 1 in the new state

5. **Add new states to the queue:**
   - Only add unvisited states to avoid redundant exploration
   - Mark the new state as visited

6. **Check for completion:**
   - If we reach state `(1 << n) - 1`, all characters are collected
   - Return the current level (number of stickers used)

7. **Return -1 if impossible** (queue empties without reaching goal state)

**Why This Works:**
- BFS guarantees we find the minimum number of stickers (shortest path)
- State compression allows efficient tracking of progress with just an integer
- The visited array ensures we don't explore the same state multiple times
- Greedy character collection within each sticker is optimal since using more characters from one sticker doesn't increase the sticker count

**Time Complexity:** `O(m * 2^n * n)` where `m` is the number of stickers and `n` is the target length
**Space Complexity:** `O(2^n)` for the visited array and queue

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

**Input:**
- Stickers: `["ab", "bc", "ac"]`
- Target: `"abc"`

**Goal:** Find the minimum number of stickers to spell "abc"

**State Representation:**
- Target "abc" has 3 characters at positions 0, 1, 2
- State bitmask: bit 0 for 'a', bit 1 for 'b', bit 2 for 'c'
- Initial state: `000` (binary) = 0
- Goal state: `111` (binary) = 7

**BFS Execution:**

**Level 0 (0 stickers used):**
- Queue: [0]
- State 0 = `000` (no characters collected)

**Level 1 (1 sticker used):**
Process state 0:
- Try sticker "ab":
  - Need 'a' at position 0? Yes (bit 0 = 0)
  - Have 'a'? Yes → collect it, set bit 0
  - Need 'b' at position 1? Yes (bit 1 = 0)
  - Have 'b'? Yes → collect it, set bit 1
  - Need 'c' at position 2? Yes (bit 2 = 0)
  - Have 'c'? No
  - New state: `011` = 3

- Try sticker "bc":
  - Need 'a'? Yes, but don't have it
  - Need 'b'? Yes, have it → collect it, set bit 1
  - Need 'c'? Yes, have it → collect it, set bit 2
  - New state: `110` = 6

- Try sticker "ac":
  - Need 'a'? Yes, have it → collect it, set bit 0
  - Need 'b'? Yes, but don't have it
  - Need 'c'? Yes, have it → collect it, set bit 2
  - New state: `101` = 5

Queue after Level 1: [3, 6, 5]
Visited: {0, 3, 6, 5}

**Level 2 (2 stickers used):**
Process state 3 (`011` - have 'a' and 'b', need 'c'):
- Try all stickers, only "bc" and "ac" can provide 'c'
- Using either gives us state `111` = 7 (goal reached!)

**Result:** Minimum 2 stickers needed

One optimal solution: Use "ab" first (getting 'a' and 'b'), then "ac" or "bc" (getting 'c').

## Python Solution

```python
class Solution:
    def minStickers(self, stickers: List[str], target: str) -> int:
        """
        Find minimum number of stickers needed to spell out the target string.
        Uses BFS with bitmask to track which characters of target have been covered.
      
        Args:
            stickers: List of available sticker strings
            target: Target string to spell out
          
        Returns:
            Minimum number of stickers needed, or -1 if impossible
        """
        target_length = len(target)
      
        # Initialize BFS queue with empty state (no characters covered yet)
        # State is represented as bitmask where bit i=1 means target[i] is covered
        queue = deque([0])
      
        # Track visited states to avoid revisiting
        visited = [False] * (1 << target_length)
        visited[0] = True
      
        # Track number of stickers used (BFS level)
        stickers_used = 0
      
        # BFS to find minimum stickers needed
        while queue:
            # Process all states at current level
            level_size = len(queue)
            for _ in range(level_size):
                current_state = queue.popleft()
              
                # Check if all target characters are covered
                if current_state == (1 << target_length) - 1:
                    return stickers_used
              
                # Try applying each sticker to current state
                for sticker in stickers:
                    # Count available characters in current sticker
                    sticker_chars = Counter(sticker)
                    next_state = current_state
                  
                    # Try to use sticker characters to cover uncovered target positions
                    for position, char in enumerate(target):
                        # If position is uncovered and sticker has this character
                        if (current_state >> position & 1) == 0 and sticker_chars[char] > 0:
                            sticker_chars[char] -= 1
                            next_state |= 1 << position  # Mark position as covered
                  
                    # Add new state to queue if not visited
                    if not visited[next_state]:
                        visited[next_state] = True
                        queue.append(next_state)
          
            # Increment sticker count after processing each level
            stickers_used += 1
      
        # Target cannot be formed with given stickers
        return -1
```

## Time and Space Complexity

**Time Complexity:** `O(2^n × m × (l + n))`

The algorithm uses BFS to explore different states represented by bitmasks. There are `2^n` possible states (each bit represents whether a character at position i in target has been covered). For each state, we iterate through all `m` stickers. For each sticker, we:
- Create a counter from the sticker string, which takes `O(l)` time where `l` is the average length of stickers
- Iterate through all `n` positions in the target string to check if we can use characters from the current sticker

Therefore, the total time complexity is `O(2^n × m × (l + n))`.

**Space Complexity:** `O(2^n)`

The space is dominated by:
- The `vis` array which tracks visited states: `O(2^n)`
- The queue which in the worst case can contain all states: `O(2^n)`
- The counter for each sticker is created temporarily and doesn't affect the overall complexity

Thus, the space complexity is `O(2^n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Inefficient Sticker Filtering**
A major performance pitfall is trying to use stickers that don't contain any characters needed by the target. This wastes computation time exploring states that can never lead to a solution.

**Problem Example:**
```python
# If target = "abc" and we have stickers = ["xyz", "def", "abc"]
# The sticker "xyz" will never contribute to the solution
# But without filtering, we still process it at every BFS level
```

**Solution:**
Pre-filter stickers to only keep those that contain at least one character from the target:
```python
def minStickers(self, stickers: List[str], target: str) -> int:
    target_chars = set(target)
  
    # Filter out useless stickers
    useful_stickers = []
    for sticker in stickers:
        if any(c in target_chars for c in sticker):
            useful_stickers.append(sticker)
  
    # Check if target is achievable
    all_sticker_chars = set(''.join(useful_stickers))
    if not all(c in all_sticker_chars for c in target):
        return -1
  
    # Continue with BFS using only useful_stickers...
```

### 2. **Not Checking Early for Impossible Cases**
The code might run through the entire BFS exploration even when it's impossible to form the target from the given stickers.

**Problem Example:**
```python
# If target = "abc" but stickers = ["aaa", "bbb"]
# We're missing 'c' entirely, so it's impossible
# Without early checking, BFS will exhaust all states
```

**Solution:**
Check if all target characters exist in at least one sticker before starting BFS:
```python
def minStickers(self, stickers: List[str], target: str) -> int:
    # Create set of all available characters from all stickers
    available_chars = set()
    for sticker in stickers:
        available_chars.update(sticker)
  
    # Early return if any target character is not available
    for char in target:
        if char not in available_chars:
            return -1
  
    # Proceed with BFS...
```

### 3. **Redundant State Transitions**
Using a sticker that doesn't add any new characters to the current state creates redundant transitions and slows down the algorithm.

**Problem Example:**
```python
# Current state has already collected 'a' and 'b'
# Using sticker "ab" won't change the state
# But we still add it to the queue (though visited check prevents reprocessing)
```

**Solution:**
Skip stickers that don't contribute any new characters:
```python
for sticker in stickers:
    sticker_chars = Counter(sticker)
    next_state = current_state
    state_changed = False  # Track if this sticker adds anything new
  
    for position, char in enumerate(target):
        if (current_state >> position & 1) == 0 and sticker_chars[char] > 0:
            sticker_chars[char] -= 1
            next_state |= 1 << position
            state_changed = True  # Mark that we made progress
  
    # Only add to queue if the sticker actually contributed
    if state_changed and not visited[next_state]:
        visited[next_state] = True
        queue.append(next_state)
```

### 4. **Memory Optimization for Large Targets**
For targets with length > 20, the space requirement of `2^n` becomes prohibitive (over 1 million states).

**Solution:**
Use a set instead of a boolean array for visited states, as most states won't be visited:
```python
def minStickers(self, stickers: List[str], target: str) -> int:
    if len(target) > 20:
        # Use set for sparse state tracking
        visited = {0}  # Use set instead of array
      
        # In BFS loop:
        if next_state not in visited:
            visited.add(next_state)
            queue.append(next_state)
    else:
        # Use array for dense state tracking (original approach)
        visited = [False] * (1 << len(target))
```

These optimizations can significantly improve both time and space efficiency, especially for edge cases and larger inputs.

