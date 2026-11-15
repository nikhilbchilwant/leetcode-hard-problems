# 1345. Jump Game IV

> Source: https://algo.monster/liteproblems/1345

## Problem Statement

You are given an array of integers `arr` and start at index 0 (the first position in the array). Your goal is to reach the last index of the array in the minimum number of steps.

From any index `i`, you can jump to:
- **Next position**: index `i + 1` (if it exists within the array bounds)
- **Previous position**: index `i - 1` (if it's not negative)
- **Any position with the same value**: any index `j` where `arr[i] == arr[j]` and `i != j`

Each jump counts as one step, regardless of the distance traveled. You cannot jump outside the array boundaries at any time.

The task is to find and return the minimum number of steps needed to reach from index 0 to the last index of the array.

For example, if `arr = [100, -23, -23, 404, 100, 23, 23, 23, 3, 404]`:
- From index 0 (value 100), you could jump to index 1 (next), or index 4 (same value 100)
- From index 2 (value -23), you could jump to index 1 (previous), index 3 (next), or index 1 (same value -23)

The algorithm uses BFS ([Breadth-First Search](/problems/bfs_intro)) to find the shortest path. It builds a graph where each value maps to all indices containing that value, then explores all possible jumps level by level to find the minimum steps. The key optimization is using `g.pop(arr[i], [])` to ensure each group of same-valued indices is only processed once, preventing redundant explorations.

## Intuition

The key insight is recognizing that this is fundamentally a shortest path problem in an unweighted graph. Each array index represents a node, and we need to find the minimum "distance" (number of jumps) from the start to the end.

Why model it as a graph? Consider what makes this problem unique - we can jump to indices with the same value, not just adjacent positions. This creates a complex network of connections. For instance, if value `5` appears at indices `[2, 7, 15]`, all three positions are interconnected, forming a "teleportation network" where we can instantly jump between any of them.

Since each jump has equal cost (1 step), we're dealing with an unweighted graph. In such scenarios, BFS naturally finds the shortest path because it explores all nodes at distance `d` before exploring nodes at distance `d+1`. This level-by-level exploration guarantees that the first time we reach our target, we've taken the minimum number of steps.

The clever optimization in the solution is how it handles same-value jumps. Instead of repeatedly checking all indices with the same value, we use `g.pop(arr[i], [])` to process each group of same-valued indices only once. Why does this work? Once we've explored jumping from value `x` to all its occurrences, we never need to do it again - any future encounter with value `x` won't provide a shorter path since BFS already found the optimal distances.

Think of it like exploring a city's subway system where some stations are connected by "color lines" - once you've explored all stations on the "red line" from one red station, you don't need to re-explore them from another red station you encounter later, as you've already found the shortest paths to all red stations.

**Learn more about [Breadth-First Search](/problems/bfs_intro) patterns.**

## Solution Approach

The implementation uses BFS with several key data structures to efficiently find the shortest path:

**1. Building the Graph Mapping (`g`):**
```python
g = defaultdict(list)
for i, x in enumerate(arr):
    g[x].append(i)
```
We create a dictionary where each value maps to a list of all indices containing that value. This preprocessing step allows O(1) lookup of all "teleportation" destinations for any value.

**2. BFS Initialization:**
```python
q = deque([0])
vis = {0}
ans = 0
```
- `q`: A deque (double-ended queue) for BFS traversal, starting with index 0
- `vis`: A set to track visited indices and prevent cycles
- `ans`: Counter for the number of steps (levels in BFS)

**3. Level-by-Level BFS Traversal:**
```python
while 1:
    for _ in range(len(q)):
        i = q.popleft()
        if i == len(arr) - 1:
            return ans
```
The outer `while` loop represents each level of BFS (each step). The inner `for` loop processes all nodes at the current level before incrementing the step counter.

**4. Exploring Neighbors:**
```python
for j in (i + 1, i - 1, *g.pop(arr[i], [])):
    if 0 <= j < len(arr) and j not in vis:
        q.append(j)
        vis.add(j)
```
For each index `i`, we explore three types of jumps:
- `i + 1`: Jump to the next index
- `i - 1`: Jump to the previous index  
- `*g.pop(arr[i], [])`: Jump to all indices with the same value

The critical optimization is `g.pop(arr[i], [])`. By using `pop` instead of `get`, we remove the value group from the dictionary after first use. This ensures each group of same-valued indices is processed only once, preventing redundant explorations. The unpacking operator `*` spreads the list of indices into the tuple.

**5. Boundary and Visit Checks:**
The condition `0 <= j < len(arr) and j not in vis` ensures we:
- Stay within array bounds
- Don't revisit already explored indices

**6. Step Increment:**
```python
ans += 1
```
After processing all nodes at the current level, we increment the step counter before moving to the next level.

The algorithm guarantees finding the shortest path because BFS explores nodes in order of their distance from the source. The first time we reach the last index, we've found the minimum number of steps required.

## Example Walkthrough

Let's trace through a small example to illustrate how the BFS solution finds the minimum steps.

**Example:** `arr = [7, 6, 9, 6, 9, 6, 9, 7]`

**Step 1: Build the value-to-indices mapping**
```
g = {
    7: [0, 7],
    6: [1, 3, 5],
    9: [2, 4, 6]
}
```

**Step 2: Initialize BFS**
- Start at index 0 (value 7)
- `q = [0]`, `vis = {0}`, `ans = 0`

**Step 3: First BFS level (ans = 0)**
- Process index 0:
  - Can jump to index 1 (next position)
  - Cannot jump to index -1 (out of bounds)
  - Can jump to index 7 using `g.pop(7)` which returns `[0, 7]`
    - Index 0 is already visited, so skip it
    - Index 7 is the target! Return `ans = 0`... wait, we haven't incremented yet
  - Actually, we check if current index is target first: 0 ≠ 7, continue
  - Add valid unvisited indices to queue: `q = [1, 7]`
  - Mark as visited: `vis = {0, 1, 7}`
  - Remove value 7 from g: `g = {6: [1, 3, 5], 9: [2, 4, 6]}`

**Step 4: Increment ans to 1 and start second BFS level**
- Process index 1:
  - Check if 1 == 7 (target)? No
  - Explore neighbors:
    - Index 0 (previous): already visited
    - Index 2 (next): add to queue
    - Indices with value 6 via `g.pop(6)`: `[1, 3, 5]`
      - Index 1: already visited
      - Index 3: add to queue
      - Index 5: add to queue
  - Remove value 6 from g: `g = {9: [2, 4, 6]}`

- Process index 7:
  - **Check if 7 == 7 (target)? Yes! Return ans = 1**

**Result:** Minimum steps = 1

The algorithm found that we can reach the last index in just 1 step by jumping from index 0 directly to index 7 (both have value 7). This demonstrates the power of the "same-value jump" feature and how BFS guarantees finding the shortest path.

## Python Solution

```python
class Solution:
    def minJumps(self, arr: List[int]) -> int:
        # Build a graph where each value maps to all its indices in the array
        value_to_indices = defaultdict(list)
        for index, value in enumerate(arr):
            value_to_indices[value].append(index)
      
        # Initialize BFS queue with starting position (index 0)
        queue = deque([0])
        # Track visited indices to avoid revisiting
        visited = {0}
        # Track the number of steps taken
        steps = 0
      
        # BFS to find minimum steps to reach the last index
        while True:
            # Process all nodes at current level
            for _ in range(len(queue)):
                current_index = queue.popleft()
              
                # Check if we've reached the target (last index)
                if current_index == len(arr) - 1:
                    return steps
              
                # Explore all possible next positions:
                # 1. Move one step forward (current_index + 1)
                # 2. Move one step backward (current_index - 1)
                # 3. Jump to any index with the same value
                # Note: pop() is used to avoid revisiting indices with same value
                next_indices = [current_index + 1, current_index - 1]
                same_value_indices = value_to_indices.pop(arr[current_index], [])
              
                for next_index in (*next_indices, *same_value_indices):
                    # Check if next_index is valid and unvisited
                    if 0 <= next_index < len(arr) and next_index not in visited:
                        queue.append(next_index)
                        visited.add(next_index)
          
            # Increment step count after processing current level
            steps += 1
```

## Time and Space Complexity

**Time Complexity:** `O(n)` where `n` is the length of the input array.

- Building the graph `g` takes `O(n)` time as we iterate through the array once.
- The BFS traversal visits each index at most once due to the `vis` set, contributing `O(n)`.
- For each index, we explore its neighbors: the previous index `i-1`, next index `i+1`, and all indices with the same value.
- The key optimization is `g.pop(arr[i], [])` which ensures that indices with the same value are only processed once collectively. After processing all indices with value `arr[i]`, they are removed from the graph, preventing redundant traversals.
- Each edge in the implicit graph is traversed at most once. The maximum number of edges is `O(n)` (considering next/previous connections and value-based connections).
- Overall: `O(n)` for preprocessing + `O(n)` for BFS = `O(n)`.

**Space Complexity:** `O(n)`

- The graph `g` stores at most `n` indices across all keys, requiring `O(n)` space.
- The visited set `vis` can contain at most `n` indices, using `O(n)` space.
- The queue `q` can hold at most `n` indices in the worst case, taking `O(n)` space.
- Overall space complexity: `O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Not Removing Same-Value Groups After First Use**

**The Pitfall:**
A critical performance issue occurs when developers use `g.get(arr[i], [])` instead of `g.pop(arr[i], [])` when exploring same-value jumps. This causes the algorithm to repeatedly process the same group of indices with identical values at every encounter.

**Why It's Problematic:**
Consider an array like `[7, 7, 7, 7, 7, 7, 7, 11]`. Without removing the value group after first use:
- When we first reach any index with value 7, we explore all other 7s
- Later, when BFS reaches another 7 from a different path, it will again try to explore all 7s
- This creates redundant work and can degrade time complexity from O(n) to O(n²) in worst cases

**The Solution:**
```python
# WRONG - causes redundant explorations
for j in (i + 1, i - 1, *g.get(arr[i], [])):
    # Process neighbors

# CORRECT - removes group after first use
for j in (i + 1, i - 1, *g.pop(arr[i], [])):
    # Process neighbors
```

The `pop()` operation ensures that once we've explored all indices with a particular value, we never process that group again. This is valid because BFS guarantees we find the shortest path on first exploration.

### 2. **Incorrect Handling of Single-Element Arrays**

**The Pitfall:**
Forgetting to handle the edge case where the array has only one element. The starting index (0) is also the last index, requiring 0 steps.

**The Solution:**
Add an early check before starting BFS:
```python
if len(arr) == 1:
    return 0
```

Or ensure the BFS check happens before exploring neighbors:
```python
# Check immediately after dequeuing
current_index = queue.popleft()
if current_index == len(arr) - 1:
    return steps
```

### 3. **Memory Issues with Large Arrays of Identical Values**

**The Pitfall:**
Arrays with many identical values (e.g., `[1] * 100000`) can cause memory issues because the initial graph construction creates large lists of indices.

**The Solution:**
Consider adding optimization for consecutive identical values:
```python
# Preprocess to remove unnecessary consecutive duplicates
# Keep only the first and last of each consecutive group
def preprocess(arr):
    if len(arr) <= 2:
        return arr
  
    processed = [arr[0]]
    for i in range(1, len(arr) - 1):
        if arr[i] != arr[i-1] or arr[i] != arr[i+1]:
            processed.append(arr[i])
    processed.append(arr[-1])
  
    # Build mapping for processed array indices to original
    # ... additional logic needed for index mapping
```

This reduces the graph size while maintaining correctness since jumping within consecutive identical values doesn't provide any advantage.

