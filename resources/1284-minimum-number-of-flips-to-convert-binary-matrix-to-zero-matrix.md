# 1284. Minimum Number of Flips to Convert Binary Matrix to Zero Matrix

> Source: https://algo.monster/liteproblems/1284

## Problem Statement

You are given a `m x n` binary matrix `mat` containing only 0s and 1s. Your goal is to transform this matrix into a zero matrix (all cells equal to 0) using the minimum number of steps.

In each step, you can:
1. Choose any cell in the matrix
2. Flip that cell and all its four neighbors (up, down, left, right) if they exist
   - Flipping means changing `1` to `0` and `0` to `1`
   - Neighbors are cells that share an edge with the chosen cell

The problem asks you to return the minimum number of steps needed to convert the entire matrix to all zeros. If it's impossible to achieve this, return `-1`.

For example, if you choose cell `(i, j)`, the following cells get flipped:
- The cell itself: `(i, j)`
- Top neighbor: `(i-1, j)` if it exists
- Bottom neighbor: `(i+1, j)` if it exists  
- Left neighbor: `(i, j-1)` if it exists
- Right neighbor: `(i, j+1)` if it exists

## Intuition

The key insight is recognizing that this is a state-space exploration problem. Each possible configuration of the matrix represents a unique state, and performing a flip operation transitions us from one state to another. Since we want the minimum number of operations, we need to find the shortest path from our initial state to the target state (all zeros).

Why can we model this as a graph problem? Consider that:
- Each matrix configuration is a node in our graph
- An edge exists between two nodes if one flip operation can transform one configuration into the other
- The problem asks for the minimum number of steps, which translates to finding the shortest path

Since the matrix is small (constraints typically limit it to a small size), the total number of possible states is manageable. For an `m x n` matrix with binary values, there are at most `2^(m*n)` possible states.

To efficiently represent each state, we can encode the entire matrix as a single integer using bit manipulation. Each cell in the matrix corresponds to one bit position: if `mat[i][j] = 1`, we set the bit at position `i * n + j`. This gives us a unique integer for each matrix configuration.

The flip operation becomes a bit manipulation operation: when we choose to flip cell `(i, j)`, we toggle the bits at positions corresponding to `(i, j)` and its four neighbors. We can simulate all possible flip operations (choosing each cell as the flip center) and see which new states we can reach.

BFS is perfect here because:
1. It explores states level by level, guaranteeing that when we first reach the target state (0), we've found it via the shortest path
2. We can track visited states to avoid cycles and redundant work
3. Each "level" in BFS represents all states reachable with exactly `k` flips

The algorithm terminates either when we reach state 0 (all cells are zero) and return the current level number, or when we've explored all reachable states without finding the target, in which case we return -1.

**Learn more about [Breadth-First Search](/problems/bfs_intro) patterns.**

## Solution Approach

The implementation uses BFS with bit manipulation to efficiently explore all possible matrix states:

**1. State Encoding:**
First, we encode the initial matrix into a single integer. Each cell `mat[i][j]` that contains a `1` contributes to the state by setting the bit at position `i * n + j`:
```python
state = sum(1 << (i * n + j) for i in range(m) for j in range(n) if mat[i][j])
```
This creates a unique integer representation where each bit corresponds to a cell in the matrix.

**2. BFS Initialization:**
We initialize a queue with the starting state and a visited set to track explored states:
```python
q = deque([state])
vis = {state}
ans = 0
```

**3. Direction Array:**
The `dirs` array is cleverly designed to represent the current cell and its four neighbors:
```python
dirs = [0, -1, 0, 1, 0, 0]
```
When iterating with index `k` from 0 to 4:
- `k=0`: `(0, 0)` - current cell
- `k=1`: `(-1, 0)` - top neighbor
- `k=2`: `(0, 1)` - right neighbor
- `k=3`: `(1, 0)` - bottom neighbor
- `k=4`: `(0, -1)` - left neighbor (wraps around using `dirs[k+1]`)

**4. Level-by-Level Exploration:**
The outer while loop processes states level by level, where each level represents all states reachable with the same number of flips:
```python
while q:
    for _ in range(len(q)):  # Process all states at current level
        state = q.popleft()
        if state == 0:  # Found target state
            return ans
```

**5. Generating Next States:**
For each state, we try flipping every possible cell `(i, j)` in the matrix:
```python
for i in range(m):
    for j in range(n):
        nxt = state  # Start with current state
```

**6. Simulating Flip Operation:**
For each chosen cell, we flip it and its neighbors by toggling the corresponding bits:
```python
for k in range(5):
    x, y = i + dirs[k], j + dirs[k + 1]
    if not 0 <= x < m or not 0 <= y < n:
        continue  # Skip out-of-bounds neighbors
    if nxt & (1 << (x * n + y)):  # If bit is 1
        nxt -= 1 << (x * n + y)   # Set to 0
    else:                          # If bit is 0
        nxt |= 1 << (x * n + y)   # Set to 1
```

**7. State Management:**
Only unvisited states are added to the queue for future exploration:
```python
if nxt not in vis:
    vis.add(nxt)
    q.append(nxt)
```

**8. Level Counter:**
After processing each level, we increment the answer counter:
```python
ans += 1
```

The algorithm returns `-1` if the queue becomes empty without finding the zero state, meaning it's impossible to convert the matrix to all zeros.

**Time Complexity:** `O(2^(m*n) * m * n)` - We potentially visit all `2^(m*n)` states, and for each state, we try `m * n` flip operations.

**Space Complexity:** `O(2^(m*n))` - For storing visited states in the worst case.

## Example Walkthrough

Let's walk through a small example to illustrate how the BFS solution works.

**Initial Matrix:**
```
mat = [[1, 0],
       [0, 1]]
```

**Step 1: Encode the Initial State**
- Cell (0,0) = 1 → bit position 0*2 + 0 = 0
- Cell (0,1) = 0 → bit position 0*2 + 1 = 1 
- Cell (1,0) = 0 → bit position 1*2 + 0 = 2
- Cell (1,1) = 1 → bit position 1*2 + 1 = 3

Initial state = `1001` in binary = 9 in decimal (bits 0 and 3 are set)

**Step 2: BFS Level 0**
- Queue: [9]
- Visited: {9}
- Check if state = 0? No, continue

**Step 3: BFS Level 1 - Try All Possible Flips**

*Flip centered at (0,0):*
- Flips: (0,0), (0,1), (1,0)
- Current bits: 1001
- Toggle bits at positions 0, 1, 2
- Result: 0110 = 6
- Add to queue since unvisited

*Flip centered at (0,1):*
- Flips: (0,0), (0,1), (1,1)
- Current bits: 1001
- Toggle bits at positions 0, 1, 3
- Result: 0011 = 3
- Add to queue since unvisited

*Flip centered at (1,0):*
- Flips: (0,0), (1,0), (1,1)
- Current bits: 1001
- Toggle bits at positions 0, 2, 3
- Result: 0110 = 6 (already in queue)

*Flip centered at (1,1):*
- Flips: (0,1), (1,0), (1,1)
- Current bits: 1001
- Toggle bits at positions 1, 2, 3
- Result: 1110 = 14
- Add to queue since unvisited

After Level 1:
- Queue: [6, 3, 14]
- Visited: {9, 6, 3, 14}
- ans = 1

**Step 4: BFS Level 2 - Process State 6 (0110)**

*Flip centered at (0,0):*
- Result: 1001 = 9 (already visited)

*Flip centered at (0,1):*
- Flips toggle bits 0, 1, 3
- Current: 0110
- Result: 1001 = 9 (already visited)

*Flip centered at (1,0):*
- Flips toggle bits 0, 2, 3
- Current: 0110
- Result: 1001 = 9 (already visited)

*Flip centered at (1,1):*
- Flips toggle bits 1, 2, 3
- Current: 0110
- Result: 0000 = 0 ✓

**Found the target state 0!**

The algorithm returns 2, meaning it takes a minimum of 2 steps to convert the matrix to all zeros.

**Verification:**
1. Start: [[1,0],[0,1]]
2. Flip at (1,1): [[1,1],[1,0]] (flipped cells: (0,1), (1,0), (1,1))
3. Flip at (0,0): [[0,0],[0,0]] (flipped cells: (0,0), (0,1), (1,0))

The solution correctly finds that 2 flips are needed to transform the matrix to all zeros.

## Python Solution

```python
class Solution:
    def minFlips(self, mat: List[List[int]]) -> int:
        # Get matrix dimensions
        rows, cols = len(mat), len(mat[0])
      
        # Convert initial matrix to a bit representation
        # Each cell (i, j) is represented by bit at position (i * cols + j)
        initial_state = sum(
            1 << (i * cols + j) 
            for i in range(rows) 
            for j in range(cols) 
            if mat[i][j] == 1
        )
      
        # BFS queue starting with initial state
        queue = deque([initial_state])
        # Set to track visited states to avoid cycles
        visited = {initial_state}
        # Number of flip operations performed
        flip_count = 0
      
        # Direction vectors: current position, up, down, left, right
        # Using a flat array to represent (0,0), (-1,0), (1,0), (0,-1), (0,1)
        directions = [0, -1, 0, 1, 0, 0]
      
        # BFS to find minimum flips to reach all-zeros state
        while queue:
            # Process all states at current level
            level_size = len(queue)
            for _ in range(level_size):
                current_state = queue.popleft()
              
                # Check if we've reached the target (all zeros)
                if current_state == 0:
                    return flip_count
              
                # Try flipping each cell in the matrix
                for i in range(rows):
                    for j in range(cols):
                        next_state = current_state
                      
                        # Flip current cell and its 4 adjacent cells
                        for k in range(5):
                            new_row = i + directions[k]
                            new_col = j + directions[k + 1]
                          
                            # Check if the cell is within bounds
                            if not (0 <= new_row < rows and 0 <= new_col < cols):
                                continue
                          
                            # Toggle the bit at position (new_row * cols + new_col)
                            bit_position = new_row * cols + new_col
                            if next_state & (1 << bit_position):
                                # If bit is 1, turn it to 0
                                next_state -= 1 << bit_position
                            else:
                                # If bit is 0, turn it to 1
                                next_state |= 1 << bit_position
                      
                        # Add new state to queue if not visited
                        if next_state not in visited:
                            visited.add(next_state)
                            queue.append(next_state)
          
            # Increment flip count after processing each level
            flip_count += 1
      
        # If no solution found, return -1
        return -1
```

## Time and Space Complexity

**Time Complexity:** `O(2^(m*n) * m * n)`

The algorithm uses BFS to explore all possible states of the matrix. Each cell in the `m × n` matrix can be either 0 or 1, giving us `2^(m*n)` possible states. For each state, we iterate through all `m * n` positions to generate next states by flipping cells. The flip operation for each position takes `O(5)` = `O(1)` time (flipping the cell itself and up to 4 adjacent cells). Since we visit each state at most once due to the visited set, the overall time complexity is `O(2^(m*n) * m * n)`.

**Space Complexity:** `O(2^(m*n))`

The space is dominated by:
- The visited set `vis` which can store up to `2^(m*n)` states in the worst case
- The BFS queue `q` which can also contain up to `O(2^(m*n))` states in the worst case
- The state representation uses bit manipulation with an integer, requiring `O(1)` space per state

Therefore, the total space complexity is `O(2^(m*n))`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Direction Array Indexing**
One of the most common mistakes is misunderstanding how the direction array works. The array `[0, -1, 0, 1, 0, 0]` might seem confusing at first glance.

**Pitfall:**
```python
# Wrong interpretation - treating it as pairs
dirs = [0, -1, 0, 1, 0, 0]
for k in range(5):
    x = i + dirs[2*k]      # This would cause index out of bounds
    y = j + dirs[2*k + 1]
```

**Correct Approach:**
```python
dirs = [0, -1, 0, 1, 0, 0]
for k in range(5):
    x = i + dirs[k]      # Uses k and k+1 as sliding window
    y = j + dirs[k + 1]
```
The trick is that `dirs[k]` and `dirs[k+1]` form sliding pairs that represent:
- (0,0), (-1,0), (0,1), (1,0), (0,-1)

### 2. **Bit Manipulation Errors When Toggling**
Another frequent mistake occurs when toggling bits, especially confusing XOR with conditional logic.

**Pitfall:**
```python
# Using XOR without checking might seem simpler but can be error-prone
next_state ^= (1 << bit_position)  # This works but less readable
```

**Better Approach:**
```python
# Explicitly check and toggle for clarity
if next_state & (1 << bit_position):  # If bit is 1
    next_state -= 1 << bit_position   # Turn to 0
else:                                  # If bit is 0
    next_state |= 1 << bit_position   # Turn to 1
```

### 3. **State Encoding Formula Confusion**
Getting the bit position calculation wrong is easy when converting between 2D and 1D representations.

**Pitfall:**
```python
# Wrong formula - using cols for row offset
bit_position = i * rows + j  # Incorrect!
```

**Correct Formula:**
```python
bit_position = i * cols + j  # Row index * number of columns + column index
```

### 4. **Not Processing BFS Level by Level**
Forgetting to process all nodes at the current level before incrementing the counter leads to incorrect step counts.

**Pitfall:**
```python
while queue:
    state = queue.popleft()
    if state == 0:
        return flip_count
    flip_count += 1  # Wrong! Increments for each state, not each level
    # ... generate next states
```

**Correct Implementation:**
```python
while queue:
    level_size = len(queue)
    for _ in range(level_size):  # Process entire level
        state = queue.popleft()
        if state == 0:
            return flip_count
        # ... generate next states
    flip_count += 1  # Increment after processing whole level
```

### 5. **Matrix Size Limitations**
Not considering the practical limitations of bit manipulation for larger matrices.

**Issue:**
- Python integers can handle arbitrary precision, but the algorithm's time/space complexity makes it impractical for matrices larger than ~20 cells total
- For a 5×4 matrix (20 cells), we could have up to 2^20 = 1,048,576 states

**Solution:**
Add a size check at the beginning:
```python
def minFlips(self, mat: List[List[int]]) -> int:
    rows, cols = len(mat), len(mat[0])
  
    # Practical limitation check
    if rows * cols > 20:
        # Consider alternative approaches for larger matrices
        raise ValueError("Matrix too large for exhaustive BFS approach")
  
    # ... rest of the implementation
```

### 6. **Off-by-One Errors in Boundary Checking**
Using wrong comparison operators for boundary checks.

**Pitfall:**
```python
# Wrong boundary check
if 0 < new_row < rows and 0 < new_col < cols:  # Excludes row/col 0!
    continue
```

**Correct:**
```python
if not (0 <= new_row < rows and 0 <= new_col < cols):
    continue
```

