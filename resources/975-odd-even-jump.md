# 975. Odd Even Jump

> Source: https://algo.monster/liteproblems/975

## Problem Statement

You have an integer array `arr` and need to find how many starting positions allow you to reach the last index through a series of alternating jump rules.

The jumping mechanics work as follows:

**Jump Types:**
- **Odd-numbered jumps** (1st, 3rd, 5th, etc.): From index `i`, jump forward to index `j` where:
  - `arr[i] <= arr[j]` 
  - `arr[j]` is the smallest value among all valid options
  - If multiple indices have the same smallest value, jump to the leftmost one

- **Even-numbered jumps** (2nd, 4th, 6th, etc.): From index `i`, jump forward to index `j` where:
  - `arr[i] >= arr[j]`
  - `arr[j]` is the largest value among all valid options  
  - If multiple indices have the same largest value, jump to the leftmost one

**Key Rules:**
- You can only jump forward (from index `i` to `j` where `i < j`)
- Your first jump from any starting position is always an odd-numbered jump
- Jumps alternate between odd and even
- Some positions may have no valid jumps available
- A starting index is "good" if you can eventually reach the last index (`arr.length - 1`) from there

**Goal:** Count how many indices in the array are "good" starting positions.

**Example walkthrough:**
If `arr = [10, 13, 12, 14, 15]`:
- Starting from index 0 (value 10):
  - 1st jump (odd): Find smallest value >= 10 to the right → jump to index 2 (value 12)
  - 2nd jump (even): Find largest value <= 12 to the right → no valid jump available
  - Cannot reach the end, so index 0 is not good

- Starting from index 4 (value 15):
  - Already at the last index, so index 4 is good

## Intuition

The key insight is that from any position, the next jump is deterministic - there's only one valid destination (or no valid jump at all). This means we can precompute where each position jumps to for both odd and even jumps.

**Breaking down the problem:**

1. **The jumping pattern is fixed**: From any index `i`, if we're making an odd jump, we always go to the same specific index. Similarly for even jumps. This suggests we can build a "jump graph" ahead of time.

2. **Working backwards is easier**: If we process the array from right to left, we can maintain a sorted structure of indices we've seen so far. For any current position, we need to find:
   - For odd jumps: The smallest value >= current value (lower bound search)
   - For even jumps: The largest value <= current value (upper bound search minus 1)

3. **Why SortedDict?** We need to:
   - Find values in sorted order (for the jump rules)
   - Map those values back to their indices
   - Handle duplicates by keeping the leftmost index
 
   A `SortedDict` with values as keys and indices as values perfectly handles this. When we encounter duplicates, we naturally overwrite with the leftmost (most recent in right-to-left traversal) index.

4. **The two-state nature**: Each position has two states - whether we can reach the end starting with an odd jump or an even jump. This naturally leads to:
   - `g[i][0]`: where to jump from index `i` on an even jump
   - `g[i][1]`: where to jump from index `i` on an odd jump

5. **[Dynamic programming](/problems/dynamic_programming_intro) emerges naturally**: Once we know where each position jumps to, checking if a position is "good" becomes a graph traversal problem. We alternate between odd and even jumps (`k ^ 1` flips between 0 and 1) and use memoization to avoid recalculating paths.

The approach rests on recognizing that the complex jumping rules can be preprocessed into a simple directed graph, where each node has at most two outgoing edges (one for odd, one for even), and then the problem reduces to path-finding with memoization.

**Learn more about [Stack](/problems/stack_intro), [Dynamic Programming](/problems/dynamic_programming_intro), [Sorting](/problems/sorting_summary) and [Monotonic Stack](/problems/mono_stack_intro) patterns.**

## Solution Approach

The solution builds a jump graph preprocessing all possible jumps, then uses [dynamic programming](/problems/dynamic_programming_intro) with memoization to count good starting positions.

**Step 1: Initialize the jump graph**
```python
g = [[0] * 2 for _ in range(n)]
```
Create a 2D array where `g[i][0]` stores the next position for an even jump from index `i`, and `g[i][1]` stores the next position for an odd jump. Initialize with 0, but will use -1 to indicate no valid jump exists.

**Step 2: Build the jump graph (right to left)**
```python
sd = SortedDict()
for i in range(n - 1, -1, -1):
```
Process the array from right to left, maintaining a `SortedDict` where keys are array values and values are their indices. This ensures we always have the leftmost index for duplicates.

**Step 3: Find odd jump destination**
```python
j = sd.bisect_left(arr[i])
g[i][1] = sd.values()[j] if j < len(sd) else -1
```
For odd jumps, we need the smallest value >= `arr[i]`. The `bisect_left` finds the insertion point for `arr[i]`, which gives us the smallest value >= `arr[i]`. If `j` is within bounds, we get the index from `sd.values()[j]`, otherwise set to -1 (no valid jump).

**Step 4: Find even jump destination**
```python
j = sd.bisect_right(arr[i]) - 1
g[i][0] = sd.values()[j] if j >= 0 else -1
```
For even jumps, we need the largest value <= `arr[i]`. The `bisect_right` finds the insertion point after all occurrences of `arr[i]`, so `j - 1` gives us the largest value <= `arr[i]`. If `j >= 0`, we have a valid jump.

**Step 5: Update the SortedDict**
```python
sd[arr[i]] = i
```
Add current element to the SortedDict. If the value already exists, it updates to the current (leftmost in original array) index.

**Step 6: [Dynamic programming](/problems/dynamic_programming_intro) with memoization**
```python
@cache
def dfs(i: int, k: int) -> bool:
    if i == n - 1:
        return True
    if g[i][k] == -1:
        return False
    return dfs(g[i][k], k ^ 1)
```
The `dfs` function checks if we can reach the end from position `i` with jump type `k` (1 for odd, 0 for even):
- Base case: If at the last index, return `True`
- If no valid jump exists (`g[i][k] == -1`), return `False`
- Otherwise, jump to the next position and flip the jump type using XOR (`k ^ 1`)

**Step 7: Count good starting positions**
```python
return sum(dfs(i, 1) for i in range(n - 1, -1, -1))
```
Check each index as a starting position. Since the first jump is always odd, we call `dfs(i, 1)` for each index and sum up the results. The starting indices are visited from right to left. Every jump goes to a larger index, so by the time `dfs(i, 1)` runs, the states it jumps into have mostly been cached already and the recursion stays only a few calls deep. Visiting them left to right gives the same answer, but the first call can then follow a chain of up to `n` jumps, which exceeds Python's default recursion limit of 1000 when `n` is 20000.

**Time Complexity:** `O(n log n)` due to the sorted dictionary operations and `O(n)` for the DFS with memoization.

**Space Complexity:** `O(n)` for the jump graph and memoization cache.

## Example Walkthrough

Let's trace through a small example: `arr = [5, 1, 3, 4, 2]`

**Step 1: Build the jump graph (process right to left)**

Starting with an empty `SortedDict`, we process each index:

**Index 4 (value 2):**
- SortedDict is empty
- Odd jump: bisect_left(2) = 0, but SortedDict is empty → `g[4][1] = -1`
- Even jump: bisect_right(2) - 1 = -1 → `g[4][0] = -1`
- Add to SortedDict: `{2: 4}`

**Index 3 (value 4):**
- SortedDict: `{2: 4}`
- Odd jump: bisect_left(4) = 1, out of bounds → `g[3][1] = -1`
- Even jump: bisect_right(4) - 1 = 0, get index at position 0 → `g[3][0] = 4`
- Add to SortedDict: `{2: 4, 4: 3}`

**Index 2 (value 3):**
- SortedDict: `{2: 4, 4: 3}`
- Odd jump: bisect_left(3) = 1, get index at position 1 → `g[2][1] = 3`
- Even jump: bisect_right(3) - 1 = 0, get index at position 0 → `g[2][0] = 4`
- Add to SortedDict: `{2: 4, 3: 2, 4: 3}`

**Index 1 (value 1):**
- SortedDict: `{2: 4, 3: 2, 4: 3}`
- Odd jump: bisect_left(1) = 0, get index at position 0 → `g[1][1] = 4`
- Even jump: bisect_right(1) - 1 = -1 → `g[1][0] = -1`
- Add to SortedDict: `{1: 1, 2: 4, 3: 2, 4: 3}`

**Index 0 (value 5):**
- SortedDict: `{1: 1, 2: 4, 3: 2, 4: 3}`
- Odd jump: bisect_left(5) = 4, out of bounds → `g[0][1] = -1`
- Even jump: bisect_right(5) - 1 = 3, get index at position 3 → `g[0][0] = 3`
- Final SortedDict: `{1: 1, 2: 4, 3: 2, 4: 3, 5: 0}`

**Final jump graph:**
```
g = [[3, -1],  # Index 0: even→3, odd→none
     [-1, 4],  # Index 1: even→none, odd→4
     [4, 3],   # Index 2: even→4, odd→3
     [4, -1],  # Index 3: even→4, odd→none
     [-1, -1]] # Index 4: even→none, odd→none
```

**Step 2: Check each starting position using DFS**

The code evaluates starting positions from right to left, which keeps the recursion shallow. The traces below are listed from index 0 upward for readability; each start gives the same result in either order.

**Starting from index 0:**
- `dfs(0, 1)` - start with odd jump
- `g[0][1] = -1` - no valid odd jump
- Return `False` ❌

**Starting from index 1:**
- `dfs(1, 1)` - start with odd jump
- `g[1][1] = 4` - jump to index 4
- `dfs(4, 0)` - now even jump from index 4
- Index 4 is the last index
- Return `True` ✓

**Starting from index 2:**
- `dfs(2, 1)` - start with odd jump
- `g[2][1] = 3` - jump to index 3
- `dfs(3, 0)` - now even jump from index 3
- `g[3][0] = 4` - jump to index 4
- `dfs(4, 1)` - now odd jump from index 4
- Index 4 is the last index
- Return `True` ✓

**Starting from index 3:**
- `dfs(3, 1)` - start with odd jump
- `g[3][1] = -1` - no valid odd jump
- Return `False` ❌

**Starting from index 4:**
- `dfs(4, 1)` - start with odd jump
- Index 4 is the last index
- Return `True` ✓

**Result:** 3 good starting positions (indices 1, 2, and 4)

## Python Solution

```python
class Solution:
    def oddEvenJumps(self, arr: List[int]) -> int:
        from sortedcontainers import SortedDict
        from functools import cache
      
        @cache
        def can_reach_end(index: int, jump_type: int) -> bool:
            """
            Check if we can reach the end from given index with given jump type.
          
            Args:
                index: Current position in array
                jump_type: 1 for odd jump, 0 for even jump
          
            Returns:
                True if we can reach the last index, False otherwise
            """
            # Base case: already at the last index
            if index == array_length - 1:
                return True
          
            # No valid next jump exists
            if next_jump_indices[index][jump_type] == -1:
                return False
          
            # Recursively check from next position with alternating jump type
            return can_reach_end(next_jump_indices[index][jump_type], jump_type ^ 1)
      
        array_length = len(arr)
      
        # Build next jump indices table
        # next_jump_indices[i][0] = next index for even jump from i
        # next_jump_indices[i][1] = next index for odd jump from i
        next_jump_indices = [[0] * 2 for _ in range(array_length)]
      
        # SortedDict to maintain sorted order of values and their indices
        sorted_dict = SortedDict()
      
        # Process array from right to left to build jump table
        for current_index in range(array_length - 1, -1, -1):
            current_value = arr[current_index]
          
            # Find next index for odd jump (smallest value >= current_value)
            position = sorted_dict.bisect_left(current_value)
            if position < len(sorted_dict):
                next_jump_indices[current_index][1] = sorted_dict.values()[position]
            else:
                next_jump_indices[current_index][1] = -1
          
            # Find next index for even jump (largest value <= current_value)
            position = sorted_dict.bisect_right(current_value) - 1
            if position >= 0:
                next_jump_indices[current_index][0] = sorted_dict.values()[position]
            else:
                next_jump_indices[current_index][0] = -1
          
            # Add current value and index to sorted dict
            sorted_dict[current_value] = current_index
      
        # Count how many starting indices can reach the end (first jump is odd)
        # Going right to left means every jump target is already cached, so recursion stays shallow
        return sum(can_reach_end(start_index, 1) for start_index in range(array_length - 1, -1, -1))
```

## Time and Space Complexity

**Time Complexity:** `O(n log n)`

The time complexity breaks down as follows:
- Building the graph `g` requires iterating through the array once in reverse order (`O(n)`), and for each element:
  - Performing `bisect_left` operation on SortedDict: `O(log n)`
  - Performing `bisect_right` operation on SortedDict: `O(log n)`
  - Inserting into SortedDict: `O(log n)`
  - Total for graph building: `O(n log n)`
- The DFS with memoization visits each state `(i, k)` at most once, where `i` ranges from `0` to `n-1` and `k` is either `0` or `1`, giving us `O(2n) = O(n)` possible states
- Each DFS call does `O(1)` work (excluding recursive calls)
- The final sum calls DFS for each starting position: `O(n)`
- Overall: `O(n log n) + O(n) = O(n log n)`

**Space Complexity:** `O(n)`

The space complexity consists of:
- Graph `g`: `O(n × 2) = O(n)` for storing two values per position
- SortedDict `sd`: `O(n)` in the worst case when all elements are unique
- Cache for memoization: `O(n × 2) = O(n)` for storing results of all possible states
- Recursion stack: shallow in practice, because starting indices are evaluated from right to left and jump targets are already cached; it is bounded by `O(n)` in any case
- Overall: `O(n)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrectly Handling Duplicate Values**
**Pitfall:** When multiple indices have the same value, forgetting that we must jump to the leftmost (earliest) index among them.

**Wrong approach:**
```python
# This might store the rightmost index for duplicates
sorted_dict[current_value] = current_index
```

**Why it fails:** If we process left-to-right and continuously update the SortedDict, we'd end up with the rightmost index for duplicate values, violating the "jump to leftmost" rule.

**Solution:** Process the array from right-to-left. When we encounter a duplicate value, the current index (being further left) naturally overwrites the previous one:
```python
# Process right to left
for current_index in range(array_length - 1, -1, -1):
    # Later in the loop:
    sorted_dict[current_value] = current_index  # Automatically keeps leftmost
```

### 2. **Confusion Between bisect_left and bisect_right**
**Pitfall:** Using the wrong bisect function or incorrectly adjusting the index for even/odd jumps.

**Wrong approach for even jumps:**
```python
# Incorrect: This finds smallest value >= current_value
position = sorted_dict.bisect_left(current_value)
next_jump_indices[current_index][0] = sorted_dict.values()[position]
```

**Why it fails:** Even jumps need the largest value ≤ current_value, not the smallest value ≥ current_value.

**Solution:** Use `bisect_right` and subtract 1 for even jumps:
```python
# Correct for even jumps (largest value <= current_value)
position = sorted_dict.bisect_right(current_value) - 1
if position >= 0:
    next_jump_indices[current_index][0] = sorted_dict.values()[position]
```

### 3. **Forgetting the Jump Type Alternation**
**Pitfall:** Not properly alternating between odd and even jumps, or starting with the wrong jump type.

**Wrong approach:**
```python
# Incorrect: Always using the same jump type
return can_reach_end(next_jump_indices[index][1], 1)  # Always odd

# Or forgetting first jump is always odd
return sum(can_reach_end(start_index, 0) for start_index in range(array_length))
```

**Solution:** Use XOR to alternate jump types and always start with odd (1):
```python
# Alternate jump types using XOR
return can_reach_end(next_jump_indices[index][jump_type], jump_type ^ 1)

# Always start with odd jump
return sum(can_reach_end(start_index, 1) for start_index in range(array_length))
```

### 4. **Not Handling Edge Cases Properly**
**Pitfall:** Forgetting that the last index is always a valid ending position, or not handling arrays with only one element.

**Wrong approach:**
```python
# Forgetting to check if already at last index
def can_reach_end(index: int, jump_type: int) -> bool:
    if next_jump_indices[index][jump_type] == -1:
        return False
    # Missing base case check
```

**Solution:** Always check the base case first:
```python
def can_reach_end(index: int, jump_type: int) -> bool:
    # Check if already at destination
    if index == array_length - 1:
        return True
    # Then check for valid jumps
    if next_jump_indices[index][jump_type] == -1:
        return False
```

### 5. **Inefficient Data Structure Choice**
**Pitfall:** Using a regular dictionary or list to maintain sorted order, leading to O(n²) complexity.

**Wrong approach:**
```python
# Inefficient: manually sorting after each insertion
values_dict = {}
for i in range(n-1, -1, -1):
    sorted_values = sorted(values_dict.keys())  # O(n log n) each time!
    # Find next jump position...
```

**Solution:** Use `SortedDict` which maintains sorted order with O(log n) operations:
```python
from sortedcontainers import SortedDict
sorted_dict = SortedDict()
# O(log n) for bisect operations and insertions
```

### 6. **Deep Recursion When Starting From the Left**
**Pitfall:** Calling `can_reach_end(start_index, 1)` for `start_index` from `0` upward. The first call may follow a chain of jumps across the whole array before anything is cached. For an array such as `[0, 10, -1, 11, -2, 12, ...]` with 20000 elements, index 0 jumps through every index in order, and Python raises `RecursionError` (the default limit is 1000).

**Solution:** Evaluate starting positions from right to left. Each jump moves to a larger index, so the jump targets are already cached and each call recurses only a few levels:
```python
return sum(can_reach_end(start_index, 1) for start_index in range(array_length - 1, -1, -1))
```

### 7. **Using 0 as the "Not Computed" Memo Marker**
**Pitfall:** In languages where the memo table stores `0`/`1`, initializing it with `0` and treating `0` as "not computed" means unreachable states are never cached. Every start that fails recomputes its whole chain, which can take `O(n²)` time.

**Solution:** Use a separate marker such as `-1` (or `null` in Java's `Integer[][]`) for states that have not been computed yet.

