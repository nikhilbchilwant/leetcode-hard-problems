# 936. Stamping The Sequence

> Source: https://algo.monster/liteproblems/936

## Problem Statement

You are given two strings: `stamp` and `target`. Initially, you have a string `s` of the same length as `target`, where all characters are `'?'`.

In each turn, you can place the `stamp` over `s` at any valid position and replace the characters in `s` with the corresponding characters from `stamp`. The stamp must be fully contained within the boundaries of `s`.

For example, if `stamp = "abc"` and `target = "abcba"`, then `s` starts as `"?????"`. You can:
- Place `stamp` at index 0 to get `"abc??"`
- Place `stamp` at index 1 to get `"?abc?"`  
- Place `stamp` at index 2 to get `"??abc"`
- But you cannot place `stamp` at index 3 since it would go out of bounds

Your goal is to transform `s` from all `'?'` characters into the `target` string by repeatedly stamping. You must do this in at most `10 * target.length` turns.

Return an array containing the starting indices where you placed the stamp at each turn. The indices should be listed in the order you performed the stamping operations. If it's impossible to achieve the target string within the allowed number of turns, return an empty array.

## Intuition

The key insight is that working forward from all `'?'` characters to the `target` string is extremely complex because later stamp operations can overwrite earlier ones, making it difficult to determine the correct sequence of operations.

Instead, we can think backwards: start from the completed `target` string and work our way back to all `'?'` characters. This reverse approach is much cleaner because we're essentially "removing" stamps rather than placing them, and we know that once a position becomes `'?'`, it stays that way.

Consider how we would "unstamp" or remove a stamp from the target. We can only remove a stamp at position `i` if all characters in that window match either the stamp characters or are already `'?'` (previously unstamped). This creates a dependency relationship - some stamps can only be removed after others have been removed first.

This dependency structure naturally leads us to think about topological sorting. Each possible stamp position (window) can be viewed as a node in a graph. The "in-degree" of each window represents how many of its characters still need to match the stamp before we can remove it. When a window's in-degree reaches 0, it means all its characters either match the stamp or are already `'?'`, so we can safely remove this stamp.

As we remove each stamp (converting its characters to `'?'`), we potentially enable other overlapping stamps to be removed. This is similar to how in topological sorting, processing one node can reduce the in-degrees of its neighbors, potentially making them ready for processing.

The adjacency list `g[i]` tracks which windows are affected when position `i` in the target becomes `'?'`. When we "unstamp" a window and turn some positions to `'?'`, we update the in-degrees of all windows that were waiting for those positions to become available.

Since we're working backwards, the final answer needs to be reversed to give us the forward sequence of stamp operations that would build from `'?'` to `target`.

**Learn more about [Stack](/problems/stack_intro), [Greedy](/problems/greedy_intro) and [Queue](/problems/queue_intro) patterns.**

## Solution Approach

The implementation uses a reverse topological sorting approach to find the sequence of stamp operations.

**Initialization Phase:**

First, we set up the necessary data structures:
- `indeg[i]` array: Initially set to `m` (stamp length) for each of the `n-m+1` possible stamp positions. This represents how many characters in window `i` differ from the stamp.
- `g[i]` adjacency list: For each position `i` in the target string, we store which windows have a mismatched character at that position.
- `q` [queue](/problems/queue_intro): Stores windows with `indeg = 0` (ready to be unstamped).
- `vis[i]` array: Tracks which positions in the target have been covered (turned to `'?'`).

**Building the Graph:**

For each possible stamp position `i` from `0` to `n-m`:
- We check each character `j` in the stamp (from `0` to `m-1`)
- If `target[i+j] == stamp[j]`, the characters match, so we decrement `indeg[i]`
- If they don't match, we add window `i` to `g[i+j]`, meaning position `i+j` blocks window `i` from being unstamped
- If `indeg[i]` becomes `0` after checking all characters, window `i` is ready to be unstamped, so we add it to [queue](/problems/queue_intro) `q`

**Topological Sorting Process:**

While the [queue](/problems/queue_intro) is not empty:
1. Pop a window `i` from the queue and add it to the answer array
2. For each position `j` in the stamp (from `0` to `m-1`):
   - If position `i+j` hasn't been visited yet:
     - Mark it as visited (`vis[i+j] = True`)
     - For each window `k` in `g[i+j]` (windows blocked by this position):
       - Decrement `indeg[k]` by 1 (one less blocking character)
       - If `indeg[k]` becomes `0`, add window `k` to the queue

**Final Check:**

After processing all windows:
- If all positions in the target are visited (`all(vis)` is `True`), we've successfully covered the entire string
- Return the reversed answer array (`ans[::-1]`) since we worked backwards
- If any position remains unvisited, it's impossible to form the target, so return an empty array

The time complexity is `O((n-m+1) * m)` where `n` is the length of the target and `m` is the length of the stamp, as each window is checked against `m` characters, each position is visited at most once, and the graph has at most `(n-m+1) * m` edges.

## Example Walkthrough

Let's walk through a small example with `stamp = "abc"` and `target = "ababc"`.

**Initial Setup:**
- Target length n = 5, stamp length m = 3
- Possible stamp positions: 0, 1, 2 (windows of size 3)
- Initialize `indeg = [3, 3, 3]` for all windows
- Initialize `g` as empty adjacency lists for each position
- Initialize `vis = [False, False, False, False, False]`

**Building the Graph:**

Window 0 (positions 0-2): "aba" vs "abc"
- Position 0: 'a' == 'a' ✓ → decrement indeg[0] to 2
- Position 1: 'b' == 'b' ✓ → decrement indeg[0] to 1  
- Position 2: 'a' != 'c' ✗ → add window 0 to g[2]
- Final: indeg[0] = 1

Window 1 (positions 1-3): "bab" vs "abc"
- Position 1: 'b' != 'a' ✗ → add window 1 to g[1]
- Position 2: 'a' != 'b' ✗ → add window 1 to g[2]
- Position 3: 'b' != 'c' ✗ → add window 1 to g[3]
- Final: indeg[1] = 3

Window 2 (positions 2-4): "abc" vs "abc"
- Position 2: 'a' == 'a' ✓ → decrement indeg[2] to 2
- Position 3: 'b' == 'b' ✓ → decrement indeg[2] to 1
- Position 4: 'c' == 'c' ✓ → decrement indeg[2] to 0
- Final: indeg[2] = 0 → add window 2 to queue

State after building:
- `indeg = [1, 3, 0]`
- `g = {1: [1], 2: [0, 1], 3: [1]}`
- `q = [2]`

**Topological Sorting:**

**Round 1:** Process window 2
- Add 2 to answer: `ans = [2]`
- Mark positions 2, 3, 4 as visited
- Position 2 was blocking windows 0 and 1:
  - indeg[0] becomes 0 → add to queue
  - indeg[1] becomes 2
- Position 3 was blocking window 1:
  - indeg[1] becomes 1
- Updated state: `vis = [F, F, T, T, T]`, `q = [0]`

**Round 2:** Process window 0
- Add 0 to answer: `ans = [2, 0]`
- Mark positions 0, 1 as visited (position 2 already visited)
- Position 1 was blocking window 1:
  - indeg[1] becomes 0 → add to queue
- Updated state: `vis = [T, T, T, T, T]`, `q = [1]`

**Round 3:** Process window 1
- Add 1 to answer: `ans = [2, 0, 1]`
- All positions already visited, no updates needed

**Final Check:**
- All positions visited: `vis = [T, T, T, T, T]` ✓
- Return reversed answer: `[1, 0, 2]`

**Verification (forward direction):**
1. Start with "?????"
2. Stamp at position 1: "?abc?"
3. Stamp at position 0: "?abc?" → "abcc?"
4. Stamp at position 2: "abcc?" → "ababc" ✓

The algorithm successfully found the stamping sequence by working backwards from the target to all '?' characters.

## Python Solution

```python
class Solution:
    def movesToStamp(self, stamp: str, target: str) -> List[int]:
        """
        Find sequence of stamp positions to transform target string to all '?'s.
        Uses reverse simulation: find which positions can be "unstamped" from target.
      
        Args:
            stamp: The stamp string to use
            target: The target string to achieve
          
        Returns:
            List of stamp positions in order, or empty list if impossible
        """
        stamp_length, target_length = len(stamp), len(target)
      
        # Track how many characters at each position still need to match
        # to successfully place stamp at that position
        unmatched_count = [stamp_length] * (target_length - stamp_length + 1)
      
        # Queue for positions that can be stamped (all characters match)
        ready_positions = deque()
      
        # Build dependency graph: for each target position, track which
        # stamp positions depend on it for matching
        dependencies = [[] for _ in range(target_length)]
      
        # Initialize unmatched counts and find initially ready positions
        for stamp_pos in range(target_length - stamp_length + 1):
            for offset, stamp_char in enumerate(stamp):
                target_pos = stamp_pos + offset
              
                if target[target_pos] == stamp_char:
                    # This character already matches
                    unmatched_count[stamp_pos] -= 1
                    if unmatched_count[stamp_pos] == 0:
                        # All characters match, can stamp here
                        ready_positions.append(stamp_pos)
                else:
                    # This stamp position depends on target_pos being wildcard
                    dependencies[target_pos].append(stamp_pos)
      
        # Process positions in reverse order (unstamping)
        stamp_sequence = []
        is_covered = [False] * target_length
      
        while ready_positions:
            # Process next ready position
            current_stamp_pos = ready_positions.popleft()
            stamp_sequence.append(current_stamp_pos)
          
            # Mark all positions covered by this stamp as wildcards
            for offset in range(stamp_length):
                target_pos = current_stamp_pos + offset
              
                if not is_covered[target_pos]:
                    is_covered[target_pos] = True
                  
                    # Update all stamp positions that were waiting for this wildcard
                    for dependent_stamp_pos in dependencies[target_pos]:
                        unmatched_count[dependent_stamp_pos] -= 1
                        if unmatched_count[dependent_stamp_pos] == 0:
                            ready_positions.append(dependent_stamp_pos)
      
        # Check if all positions are covered and return result
        # Reverse the sequence since we found it backwards (unstamping order)
        return stamp_sequence[::-1] if all(is_covered) else []
```

## Time and Space Complexity

**Time Complexity:** `O((n - m + 1) × m)`, which is at most `O(n × m)`

Here `n` is the length of `target` and `m` is the length of `stamp`.
- Initial setup: For each of the `n - m + 1` windows, we compare `m` characters, giving `O((n - m + 1) × m)`. Each mismatch adds one entry to the graph, so the graph has at most `(n - m + 1) × m` entries in total.
- BFS processing: Each window enters the queue at most once and scans its `m` positions, which is `O((n - m + 1) × m)`. Each target position is marked as covered at most once, and at that moment we walk its dependency list once. The dependency lists together hold at most `(n - m + 1) × m` entries, so all those walks cost `O((n - m + 1) × m)` as well.

The dominant term is `O((n - m + 1) × m)`.

**Space Complexity:** `O((n - m + 1) × m)`

The space is used for:
- `indeg` array: `O(n - m + 1)`
- Graph `g`: One entry per mismatched (window, offset) pair, at most `(n - m + 1) × m` entries. A single position is covered by at most `m` windows, so no list is longer than `m`
- Queue `q`: At most `O(n - m + 1)` elements
- `vis` array: `O(n)`
- `ans` list: `O(n - m + 1)`

The dominant term is the graph storage, which is `O((n - m + 1) × m)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Handling of Already Covered Positions**

**The Pitfall:**
When processing a stamp position from the queue, developers often forget to check whether a position has already been covered (turned into a wildcard) before updating dependent stamp positions. This can lead to:
- Double-counting: Decrementing `unmatched_count` multiple times for the same position
- Adding stamp positions to the queue prematurely when they shouldn't be ready yet
- Infinite loops or incorrect results

**Example of Incorrect Code:**
```python
# WRONG: Not checking if position is already covered
for offset in range(stamp_length):
    target_pos = current_stamp_pos + offset
    is_covered[target_pos] = True  # Mark as covered regardless
  
    # This will update dependencies even if already processed!
    for dependent_stamp_pos in dependencies[target_pos]:
        unmatched_count[dependent_stamp_pos] -= 1
        if unmatched_count[dependent_stamp_pos] == 0:
            ready_positions.append(dependent_stamp_pos)
```

**The Solution:**
Always check if a position has been covered before processing its dependencies:
```python
# CORRECT: Check if position is already covered
for offset in range(stamp_length):
    target_pos = current_stamp_pos + offset
  
    if not is_covered[target_pos]:  # Only process if not yet covered
        is_covered[target_pos] = True
      
        # Now safely update dependencies
        for dependent_stamp_pos in dependencies[target_pos]:
            unmatched_count[dependent_stamp_pos] -= 1
            if unmatched_count[dependent_stamp_pos] == 0:
                ready_positions.append(dependent_stamp_pos)
```

### 2. **Misunderstanding the Initial Match Counting**

**The Pitfall:**
During initialization, developers might incorrectly count matches or set up the dependency graph wrong. Common mistakes include:
- Forgetting to immediately add positions with zero unmatched count to the queue
- Adding positions to dependencies even when they match

**Example of Incorrect Code:**
```python
# WRONG: windows that already match fully are never queued
for stamp_pos in range(target_length - stamp_length + 1):
    for offset, stamp_char in enumerate(stamp):
        target_pos = stamp_pos + offset
      
        if target[target_pos] != stamp_char:
            unmatched_count[stamp_pos] += 1  # Counting up from 0 is fine on its own
            dependencies[target_pos].append(stamp_pos)
  
    # Forgot to check if ready to stamp immediately!
```

**The Solution:**
Counting mismatches up from 0 gives the same numbers as counting down from `m`, but every window whose count is 0 after the scan must go into the queue. Without that, `stamp = "abc"`, `target = "abc"` returns `[]` instead of `[0]`. The main solution counts down from the full stamp length and queues a window as soon as its count reaches 0:
```python
# CORRECT: Start with stamp_length and decrement for matches
unmatched_count = [stamp_length] * (target_length - stamp_length + 1)

for stamp_pos in range(target_length - stamp_length + 1):
    for offset, stamp_char in enumerate(stamp):
        target_pos = stamp_pos + offset
      
        if target[target_pos] == stamp_char:
            unmatched_count[stamp_pos] -= 1
            if unmatched_count[stamp_pos] == 0:
                ready_positions.append(stamp_pos)  # Add immediately if ready
        else:
            dependencies[target_pos].append(stamp_pos)
```

### 3. **Forgetting to Reverse the Final Answer**

**The Pitfall:**
Since the algorithm works backwards (from target to all wildcards), the stamp positions are collected in reverse order. Forgetting to reverse the final answer will give the wrong sequence.

**Example of Incorrect Code:**
```python
# WRONG: Returning the answer without reversing
return stamp_sequence if all(is_covered) else []
```

**The Solution:**
Always reverse the answer before returning:
```python
# CORRECT: Reverse the sequence since we found it backwards
return stamp_sequence[::-1] if all(is_covered) else []
```

These pitfalls are particularly dangerous because they might work correctly for simple test cases but fail on more complex inputs where stamps overlap or the same position needs to be considered multiple times.

