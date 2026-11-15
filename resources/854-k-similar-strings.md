# 854. K-Similar Strings

> Source: https://algo.monster/liteproblems/854

## Problem Statement

This problem asks you to find the minimum number of swaps needed to transform one string into another string.

You are given two strings `s1` and `s2` that are anagrams of each other (meaning they contain the same characters with the same frequencies, just in different positions). 

The concept of **k-similarity** is defined as follows: two strings are k-similar if you can transform `s1` into `s2` by performing exactly `k` swaps of character positions. Each swap involves selecting two positions in the string and exchanging the characters at those positions.

Your task is to find the smallest value of `k` such that `s1` and `s2` are k-similar. In other words, find the minimum number of swaps required to convert `s1` into `s2`.

For example:
- If `s1 = "ab"` and `s2 = "ba"`, then they are 1-similar because one swap (swapping positions 0 and 1) transforms `s1` into `s2`
- If `s1 = "abc"` and `s2 = "bca"`, then they are 2-similar because you need at least 2 swaps to transform `s1` into `s2`

## Intuition

The key insight is recognizing that this problem is fundamentally about finding the shortest transformation sequence between two states. Each state is a string configuration, and we can move from one state to another through a swap operation.

Think of it like a puzzle where you need to rearrange letters to match a target pattern. Every possible arrangement of the string forms a node in an imaginary graph, and swapping two characters creates an edge between nodes. Since we want the minimum number of swaps, we're looking for the shortest path in this graph.

Why BFS? When all moves have equal cost (each swap counts as 1), BFS naturally finds the shortest path by exploring all possibilities at distance 1, then distance 2, and so on. The first time we reach our target string `s2`, we've found it using the minimum number of steps.

The clever optimization in the solution comes from how we generate the next states. Instead of trying all possible swaps randomly, we use a strategic approach:
1. Find the first position where `s1` and `s2` differ
2. Only swap this mismatched character with positions that would place the correct character (`s2[i]`) in this position
3. Additionally, we only swap with positions `j` where `s[j] != s2[j]` (the character at position `j` is also mismatched)

This strategy reduces the search space significantly. We're not wasting time on swaps that don't make progress toward the target. Each swap fixes at least one position and potentially sets up another character to be in a better position for future swaps.

The visited set (`visited`) ensures we don't revisit the same string configuration, preventing infinite loops and redundant work. Since BFS explores level by level, when we first encounter `s2`, we can immediately return the current level number as our answer - this represents the minimum number of swaps needed.

**Learn more about [Breadth-First Search](/problems/bfs_intro) patterns.**

## Solution Approach

The solution implements BFS using a queue to explore string states level by level. Let's walk through the implementation details:

**Data Structures Used:**
- `deque`: A double-ended queue for BFS traversal, allowing efficient append and popleft operations
- `visited` (set): Tracks visited string configurations to avoid revisiting states
- `swap_count`: Counter for the number of swap levels (distance from start)

**The Main BFS Loop:**
```python
queue = deque([s1])
visited = {s1}
swap_count, string_length = 0, len(s1)
```
We initialize the queue with the starting string `s1`, mark it as visited, and set the answer counter to 0.

**Level-by-level Processing:**
```python
while 1:
    for _ in range(len(queue)):
        s = queue.popleft()
        if s == s2:
            return swap_count
```
The outer `while` loop represents each level of BFS (each swap distance). The inner `for` loop processes all strings at the current distance before moving to the next level.

**The `get_next_states` Function - Generating Valid Next States:**
```python
def get_next_states(s):
    i = 0
    while s[i] == s2[i]:
        i += 1
```
This function finds the first mismatched position between current string `s` and target `s2`. This is the position we'll fix with our swap.

```python
res = []
for j in range(i + 1, string_length):
    if s[j] == s2[i] and s[j] != s2[j]:
        res.append(s2[: i + 1] + s[i + 1 : j] + s[i] + s[j + 1 :])
```
For the mismatched position `i`, we look for positions `j` where:
- `s[j] == s2[i]`: Character at position `j` is what we need at position `i`
- `s[j] != s2[j]`: Character at position `j` is also mismatched (not in its final position)

When both conditions are met, we perform the swap and create a new string state. The string construction `s2[: i + 1] + s[i + 1 : j] + s[i] + s[j + 1 :]` cleverly uses `s2[: i + 1]` to ensure all positions up to `i` are already correct.

**Adding New States:**
```python
for nxt in get_next_states(s):
    if nxt not in visited:
        visited.add(nxt)
        queue.append(nxt)
swap_count += 1
```
For each valid next state generated, if it hasn't been visited, we add it to both the visited set and the queue for future exploration. After processing all states at the current level, we increment `swap_count` to move to the next swap distance.

The algorithm guarantees finding the minimum number of swaps because BFS explores states in order of increasing distance from the start. The first time we encounter `s2`, we've found the shortest path.

## Example Walkthrough

Let's trace through the algorithm with `s1 = "abc"` and `s2 = "bca"`.

**Initial State:**
- Queue: `["abc"]`
- Visited: `{"abc"}`
- Answer: 0

**Level 0 (swap_count = 0):**
- Process `"abc"`:
  - Check if `"abc" == "bca"`: No
  - Generate next states using the `get_next_states` function:
    - Find first mismatch: position 0 (`'a'` vs `'b'`)
    - Need `'b'` at position 0, look for `'b'` in positions 1-2:
      - Position 1: `s[1] = 'b'` (matches what we need) and `s2[1] = 'c'` (so `'b' != 'c'`, also mismatched) ✓
      - Swap positions 0 and 1: `"abc"` → `"bac"`
    - Return `["bac"]`
  - Add `"bac"` to queue and visited set
- Queue after level: `["bac"]`
- Increment swap_count to 1

**Level 1 (swap_count = 1):**
- Process `"bac"`:
  - Check if `"bac" == "bca"`: No
  - Generate next states:
    - Find first mismatch: position 1 (`'a'` vs `'c'`)
    - Need `'c'` at position 1, look for `'c'` in position 2:
      - Position 2: `s[2] = 'c'` (matches what we need) and `s2[2] = 'a'` (so `'c' != 'a'`, also mismatched) ✓
      - Swap positions 1 and 2: `"bac"` → `"bca"`
    - Return `["bca"]`
  - Add `"bca"` to queue and visited set
- Queue after level: `["bca"]`
- Increment swap_count to 2

**Level 2 (swap_count = 2):**
- Process `"bca"`:
  - Check if `"bca" == "bca"`: Yes! ✓
  - Return swap_count = 2

The algorithm found that the minimum number of swaps needed is 2:
1. Swap positions 0 and 1: `"abc"` → `"bac"`
2. Swap positions 1 and 2: `"bac"` → `"bca"`

The key optimization is that at each step, we only consider swaps that:
1. Fix the first mismatched position
2. Use a character that's also currently misplaced

This strategic approach significantly reduces the search space compared to trying all possible swaps.

## Python Solution

```python
from collections import deque
from typing import List

class Solution:
    def kSimilarity(self, s1: str, s2: str) -> int:
        def get_next_states(current_string: str) -> List[str]:
            """
            Generate all possible next states by performing one swap operation.
            The swap should move us closer to the target string s2.
            """
            # Find the first position where current string differs from target
            first_diff_index = 0
            while current_string[first_diff_index] == s2[first_diff_index]:
                first_diff_index += 1
          
            next_states = []
            # Look for positions to swap with
            for swap_index in range(first_diff_index + 1, string_length):
                # Only swap if:
                # 1. The character at swap_index matches what we need at first_diff_index
                # 2. The character at swap_index is not already in its correct position
                if (current_string[swap_index] == s2[first_diff_index] and 
                    current_string[swap_index] != s2[swap_index]):
                    # Create new string with characters swapped
                    swapped_string = (s2[:first_diff_index + 1] + 
                                    current_string[first_diff_index + 1:swap_index] + 
                                    current_string[first_diff_index] + 
                                    current_string[swap_index + 1:])
                    next_states.append(swapped_string)
          
            return next_states
      
        # BFS initialization
        queue = deque([s1])
        visited = {s1}
        swap_count = 0
        string_length = len(s1)
      
        # BFS to find minimum swaps
        while True:
            # Process all strings at current level
            for _ in range(len(queue)):
                current = queue.popleft()
              
                # Check if we've reached the target
                if current == s2:
                    return swap_count
              
                # Generate and enqueue all valid next states
                for next_state in get_next_states(current):
                    if next_state not in visited:
                        visited.add(next_state)
                        queue.append(next_state)
          
            # Increment swap count after processing each level
            swap_count += 1
```

## Time and Space Complexity

**Time Complexity:** `O(n! * n^2)` in the worst case, where `n` is the length of the strings.

- The algorithm uses BFS to explore all possible states (string permutations) that can be reached through swaps
- In the worst case, we might need to explore up to `O(n!)` different permutations of the string
- For each state processed, the `get_next_states()` function takes `O(n^2)` time:
  - Finding the first mismatched position: `O(n)`
  - Iterating through remaining positions to find swap candidates: `O(n)`
  - Creating new strings after swapping: `O(n)` for each swap
- The total number of states visited is bounded by the number of permutations that differ from the target by at most `k` swaps, where `k` is the minimum number of swaps needed
- Therefore, the overall time complexity is `O(n! * n^2)`

**Space Complexity:** `O(n! * n)` in the worst case.

- The `visited` set can store up to `O(n!)` different string permutations in the worst case
- Each string has length `n`, so storing each string takes `O(n)` space
- The BFS queue can also contain up to `O(n!)` strings at any level
- Therefore, the total space complexity is `O(n! * n)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Inefficient Swap Selection - Swapping Without Strategic Planning**

A common mistake is to generate all possible swaps at each step, rather than focusing on productive swaps that fix at least one mismatched position. This leads to an exponential explosion of states and makes the algorithm impractically slow.

**Problematic Approach:**
```python
# BAD: Trying all possible swaps
def get_all_swaps(s):
    result = []
    for i in range(len(s)):
        for j in range(i + 1, len(s)):
            # Swap any two positions
            swapped = list(s)
            swapped[i], swapped[j] = swapped[j], swapped[i]
            result.append(''.join(swapped))
    return result
```

**Why it fails:** This generates O(n²) states at each level, leading to exponential growth in the search space. Many of these swaps don't make progress toward the target.

**Solution:** Always fix the first mismatched position and only swap with positions that:
1. Have the correct character needed at the first mismatch
2. Are themselves in wrong positions (to avoid wasting a swap)

### 2. **String Reconstruction Error - Incorrect Slicing**

When constructing the swapped string, a subtle but critical error is using the wrong base string for the prefix portion.

**Problematic Code:**
```python
# WRONG: Using current string for prefix
swapped_string = (current_string[:first_diff_index + 1] +  # Bug here!
                 current_string[first_diff_index + 1:swap_index] + 
                 current_string[first_diff_index] + 
                 current_string[swap_index + 1:])
```

**Why it fails:** The positions before `first_diff_index` already match `s2`, but position `first_diff_index` does not. Using `current_string[:first_diff_index + 1]` keeps the old, mismatched character at that position instead of placing `current_string[swap_index]` there. The result is not a swap at all: the old character is duplicated and the needed character is lost. For `current_string = "abc"`, `s2 = "bca"`, the swap of positions 0 and 1 should give `"bac"`, but this code produces `"aac"`.

**Correct Approach:**
```python
# CORRECT: Using target string for already-matched prefix
swapped_string = (s2[:first_diff_index + 1] +  # Use s2 for matched portion
                 current_string[first_diff_index + 1:swap_index] + 
                 current_string[first_diff_index] + 
                 current_string[swap_index + 1:])
```

This ensures that:
- All positions up to and including `first_diff_index` are correct (matching s2)
- The swap is properly reflected in the remaining portion

### 3. **Redundant State Exploration - Missing Early Termination**

**Problematic Pattern:**
```python
# Processing unnecessary swaps even when strings match
def get_next_states(current_string):
    # Missing check for already matching strings
    next_states = []
    for i in range(len(current_string)):
        if current_string[i] != s2[i]:
            # Generate swaps...
```

**Solution:** The main solution avoids this because it checks `current == s2` and returns before calling `get_next_states`, so `get_next_states` is only called on a string with at least one mismatch. If the helper is reused elsewhere, add a bounds check so it does not run past the end of the string:
```python
def get_next_states(current_string):
    # Find first mismatch - will skip all matching positions
    first_diff_index = 0
    while (first_diff_index < len(current_string) and
           current_string[first_diff_index] == s2[first_diff_index]):
        first_diff_index += 1
  
    if first_diff_index == len(current_string):
        return []  # All positions match
```

### 4. **Memory Inefficiency - Not Pruning Invalid Swaps**

**Issue:** Including swaps where the character at position `j` is already in its correct position wastes memory and processing time.

**Example:**
If `current = "baa"` and `target = "aab"`, the first mismatch is position 0, which needs an `'a'`. Both positions 1 and 2 hold `'a'`, but position 1 already matches the target. Swapping positions 0 and 1 gives `"aba"`, which moves a correct `'a'` out of place and leaves two mismatches. Swapping positions 0 and 2 gives `"aab"`, which finishes in one swap.

**Solution:** Always check `current_string[swap_index] != s2[swap_index]` before including a swap in the next states. This ensures we only move characters that need to be moved eventually.

