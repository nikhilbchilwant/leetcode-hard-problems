# 864. Shortest Path to Get All Keys

> Source: https://algo.monster/liteproblems/864

## Problem Statement

You are given an `m x n` grid where each cell contains one of the following characters:
- `'.'` represents an empty cell that you can walk through
- `'#'` represents a wall that blocks your path
- `'@'` represents your starting position
- Lowercase letters (`'a'` to `'f'`) represent keys you can collect
- Uppercase letters (`'A'` to `'F'`) represent locks that require corresponding keys

You start at the `'@'` position and can move one step at a time in any of the four cardinal directions (up, down, left, right). You cannot walk outside the grid boundaries or walk through walls.

The movement rules are:
- You can walk over empty cells freely
- When you walk over a key, you automatically pick it up
- You cannot walk through a lock unless you have already collected its corresponding key (e.g., you need key `'a'` to pass through lock `'A'`)

The grid contains exactly `k` keys and `k` locks, where `1 <= k <= 6`. Each key has exactly one corresponding lock, and they use the first `k` letters of the alphabet. For example, if there are 3 keys, the grid will contain keys `'a'`, `'b'`, `'c'` and locks `'A'`, `'B'`, `'C'`.

Your goal is to find the minimum number of moves required to collect all keys in the grid. If it's impossible to collect all keys, return `-1`.

For example, if you have a grid with keys `'a'` and `'b'`, you need to find the shortest path that collects both keys, which might require backtracking or taking a specific route to unlock certain areas of the grid.

## Intuition

The key insight is recognizing that this isn't just a simple pathfinding problem - we need to track not only our position but also which keys we've collected. This is because the same position can be visited multiple times with different sets of keys, opening up new paths that were previously blocked by locks.

Think about it this way: imagine you're at position `(x, y)` and you encounter lock `'B'`. Without key `'b'`, you're stuck. But if you come back to the same position `(x, y)` later after collecting key `'b'`, you can now pass through. This means the state of our search isn't just "where we are" but "where we are AND what keys we have."

Since we want the shortest path, BFS is natural - it explores all positions reachable in 1 move, then 2 moves, then 3 moves, and so on. The first time we reach a state where we have all keys is guaranteed to be the shortest path.

The clever part is how to represent "what keys we have." Since there are at most 6 keys (`'a'` through `'f'`), we can use a 6-bit number where each bit represents whether we have that key. For example:
- `000000` means we have no keys
- `000001` means we have key `'a'`
- `000011` means we have keys `'a'` and `'b'`
- `111111` means we have all 6 keys

This bit representation allows us to efficiently check if we have a key (using bitwise AND) and add a new key to our collection (using bitwise OR).

The visited set needs to track triplets `(row, column, key_state)` because visiting position `(2, 3)` with keys `'a'` and `'b'` is different from visiting `(2, 3)` with just key `'a'`. Each represents a unique state in our search space, and we might need to explore both to find the optimal solution.

When we pick up a new key, we update our state by setting the corresponding bit to 1. When we encounter a lock, we check if the corresponding bit is set. If we ever reach a state where all `k` bits are set (meaning we have all keys), we've found our answer.

**Learn more about [Breadth-First Search](/problems/bfs_intro) patterns.**

## Solution Approach

The implementation follows a state-compressed BFS approach. Let's walk through the key components:

**1. Initial Setup**
First, we scan the grid to find:
- Starting position `(start_row, start_col)` where `grid[i][j] == '@'`
- Total number of keys `total_keys` by counting all lowercase letters in the grid

**2. State Representation**
Each state in our BFS is a tuple `(current_row, current_col, key_state)` where:
- `(current_row, current_col)` is the current position
- `key_state` is a bitmask representing collected keys
  - If we have key `'a'`, bit 0 is set: `key_state |= (1 << 0)`
  - If we have key `'c'`, bit 2 is set: `key_state |= (1 << 2)`
  - All keys collected when `key_state == (1 << total_keys) - 1`

**3. BFS Queue and Visited Set**
- Initialize queue `queue` with starting state: `[(start_row, start_col, 0)]` (no keys initially)
- Initialize visited set `visited` with `{(start_row, start_col, 0)}` to avoid revisiting same state

**4. BFS Exploration**
For each state `(current_row, current_col, key_state)` popped from queue:

**Check completion**: If `key_state == (1 << total_keys) - 1`, we have all keys - return current step count.

**Explore four directions**: For each adjacent cell `(next_row, next_col)`:
- **Boundary check**: Ensure `0 <= next_row < rows` and `0 <= next_col < cols`
- **Wall check**: Skip if `grid[next_row][next_col] == '#'`
- **Lock check**: If uppercase letter (lock), check if we have the key:
  ```python
  if cell.isupper() and (key_state & (1 << (ord(cell) - ord('A')))) == 0:
      continue  # Don't have the key, can't pass
  ```
- **Key collection**: If lowercase letter (key), update state:
  ```python
  if cell.islower():
      next_key_state |= 1 << (ord(cell) - ord('a'))  # Add key to our collection
  ```

**5. State Tracking**
Only add new state `(next_row, next_col, next_key_state)` to queue if not visited before:
```python
if (next_row, next_col, next_key_state) not in visited:
    visited.add((next_row, next_col, next_key_state))
    queue.append((next_row, next_col, next_key_state))
```

**6. Level-by-Level Processing**
The BFS processes all states at distance `d` before moving to distance `d+1`:
```python
for _ in range(len(queue)):  # Process all states at current distance
    # ... process each state
steps += 1  # Increment distance after processing current level
```

**7. Impossible Case**
If the queue becomes empty without finding all keys, return `-1`.

The time complexity is `O(m * n * 2^k)` where we might visit each cell with each possible key combination. The space complexity is also `O(m * n * 2^k)` for the visited set. Since `k <= 6`, the `2^k` factor is at most 64, making this approach efficient for the given constraints.

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach:

```
Grid:
['@', '.', 'a'],
['A', '#', 'b'],
['.', '.', '.']
```

**Initial Setup:**
- Starting position: (0, 0) where '@' is located
- Total keys k = 2 (keys 'a' and 'b')
- Target state: 11 in binary (both bits set) = 3 in decimal

**BFS Execution:**

**Step 0:** Queue = [(0, 0, state=00)]
- Start at (0,0) with no keys (state = 00 in binary)

**Step 1:** Process (0, 0, 00)
- Try moving to (0,1): Valid, empty cell '.'
- Try moving to (1,0): It's lock 'A', need key 'a' (bit 0). State=00, bit 0 not set → blocked
- Add (0,1,00) to queue

**Step 2:** Process (0, 1, 00)
- Try moving to (0,0): Already visited with state 00
- Try moving to (0,2): It's key 'a'! Pick it up → state becomes 01
- Try moving to (1,1): Wall '#' → blocked
- Add (0,2,01) to queue

**Step 3:** Process (0, 2, 01) (distance 2)
- We now have key 'a' (state=01)
- Try moving down to (1,2): It's key 'b'! Pick it up → state becomes 11. Add (1,2,11)
- Try moving left to (0,1): Add (0,1,01) - different from the earlier visit (0,1,00)
- Queue for distance 3: [(1,2,11), (0,1,01)]

**Step 4:** Process (1, 2, 11) (distance 3)
- State = 11 (binary) = 3 (decimal) = (1 << 2) - 1
- **All keys collected! Return 3 steps**

The path taken: 
1. (0,0) → (0,1) [move right]
2. (0,1) → (0,2) [move right, collect 'a']
3. (0,2) → (1,2) [move down, collect 'b']

Total moves: 3

Note how the state tracking matters here: lock 'A' at (1,0) blocked the first move down. With key 'a' collected, the state (0,1,01) is new, so the search is allowed to walk back through cells it already visited without keys. In this grid the shortest route never needs the lock, and BFS finds it because it explores all states level by level.

## Python Solution

```python
class Solution:
    def shortestPathAllKeys(self, grid: List[str]) -> int:
        rows, cols = len(grid), len(grid[0])
      
        # Find the starting position marked with '@'
        start_row, start_col = next(
            (i, j) for i in range(rows) for j in range(cols) 
            if grid[i][j] == '@'
        )
      
        # Count total number of keys (lowercase letters)
        total_keys = sum(cell.islower() for row in grid for cell in row)
      
        # Direction vectors for moving up, right, down, left
        directions = [(-1, 0), (0, 1), (1, 0), (0, -1)]
      
        # BFS queue: (row, col, key_state)
        # key_state uses bits to represent which keys we have
        queue = deque([(start_row, start_col, 0)])
      
        # Set to track visited states: (row, col, key_state)
        visited = {(start_row, start_col, 0)}
      
        steps = 0
      
        # BFS to find shortest path
        while queue:
            # Process all nodes at current level
            for _ in range(len(queue)):
                current_row, current_col, key_state = queue.popleft()
              
                # Check if we have collected all keys
                if key_state == (1 << total_keys) - 1:
                    return steps
              
                # Explore all four directions
                for delta_row, delta_col in directions:
                    next_row = current_row + delta_row
                    next_col = current_col + delta_col
                    next_key_state = key_state
                  
                    # Check if next position is within grid bounds
                    if 0 <= next_row < rows and 0 <= next_col < cols:
                        cell = grid[next_row][next_col]
                      
                        # Check if cell is a wall or a locked door without key
                        if cell == '#':
                            continue
                        if cell.isupper():
                            # Check if we have the key for this lock
                            key_bit = ord(cell) - ord('A')
                            if (key_state & (1 << key_bit)) == 0:
                                continue
                      
                        # If cell contains a key, add it to our key state
                        if cell.islower():
                            key_bit = ord(cell) - ord('a')
                            next_key_state |= (1 << key_bit)
                      
                        # Add unvisited state to queue
                        if (next_row, next_col, next_key_state) not in visited:
                            visited.add((next_row, next_col, next_key_state))
                            queue.append((next_row, next_col, next_key_state))
          
            # Increment step count after processing current level
            steps += 1
      
        # No path found that collects all keys
        return -1
```

## Time and Space Complexity

**Time Complexity:** `O(m × n × 2^k)`

The algorithm uses BFS to explore all possible states. Each state consists of:
- Current position `(x, y)` - there are `m × n` possible positions
- Key collection state - represented as a bitmask with `2^k` possible combinations (where `k` is the number of keys)

In the worst case, we may need to visit each cell with every possible key combination, giving us `m × n × 2^k` total states to explore. Each state is processed once due to the visited set `visited`, and for each state, we check 4 directions (constant time). Therefore, the overall time complexity is `O(m × n × 2^k)`.

**Space Complexity:** `O(m × n × 2^k)`

The space is dominated by:
- The `visited` set which stores all visited states `(next_row, next_col, key_state)`. In the worst case, this can contain up to `m × n × 2^k` entries
- The queue `queue` which in the worst case can also hold `O(m × n × 2^k)` states

Both data structures can grow to the same order of magnitude, so the total space complexity is `O(m × n × 2^k)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrectly Treating Position-Only States as Unique**

A critical mistake is visiting the same cell multiple times with different key combinations but treating them as duplicate states. For example:

**Incorrect approach:**
```python
visited = set()
queue = deque([(start_row, start_col, 0)])
visited.add((start_row, start_col))  # Wrong: only tracking position

while queue:
    row, col, keys = queue.popleft()
    # ...
    if (next_row, next_col) not in visited:  # Wrong check
        visited.add((next_row, next_col))
```

**Why this fails:** Consider a grid where you need to:
1. Visit cell (2, 3) to get key 'a'
2. Return to cell (1, 1) with key 'a' to unlock door 'A'
3. Pass through (1, 1) again to reach key 'b'

If we only track positions, we'd mark (1, 1) as visited on first pass and never revisit it with new keys, making the solution impossible.

**Correct approach:**
```python
visited = set()
visited.add((start_row, start_col, 0))  # Include key state

# Later in the loop:
if (next_row, next_col, next_key_state) not in visited:
    visited.add((next_row, next_col, next_key_state))
```

### 2. **Mishandling the Target State Check**

**Check after popping (used by the main solution):**
```python
# Correct: Checking after popping from queue
current_row, current_col, key_state = queue.popleft()
if key_state == (1 << total_keys) - 1:
    return steps
```

This is correct but finds the answer one level later than necessary. An alternative checks when a key is collected and the next state is created:

```python
# Alternative: Check when collecting a key
if cell.islower():
    key_bit = ord(cell) - ord('a')
    next_key_state |= (1 << key_bit)
    if next_key_state == (1 << total_keys) - 1:
        return steps + 1  # Found all keys
```

### 3. **Incorrect Bit Manipulation for Keys and Locks**

**Common mistakes:**
```python
# Wrong: Using wrong bit position
if cell.isupper():
    if key_state & (1 << ord(cell)):  # Wrong: should subtract 'A'
        continue

# Wrong: Not preserving existing keys
if cell.islower():
    next_key_state = 1 << (ord(cell) - ord('a'))  # Wrong: overwrites
    # Should be: next_key_state |= 1 << (ord(cell) - ord('a'))
```

### 4. **Level-Order BFS Implementation Error**

**Incorrect distance tracking:**
```python
# Wrong: Incrementing steps for each node
while queue:
    current = queue.popleft()
    steps += 1  # Wrong: counts each node, not each level
```

**Correct level-order processing:**
```python
steps = 0
while queue:
    # Process all nodes at current distance
    for _ in range(len(queue)):
        current = queue.popleft()
        # Process current node
    steps += 1  # Increment once per level
```

### 5. **Edge Case: Starting Position Contains a Key**

While rare, if the starting position '@' is replaced with a key in some variant:
```python
# Handle initial key collection
initial_keys = 0
if grid[start_row][start_col].islower():
    initial_keys = 1 << (ord(grid[start_row][start_col]) - ord('a'))
queue = deque([(start_row, start_col, initial_keys)])
```

**Solution Summary:**
The key insight is that the same physical location with different key combinations represents different states in the search space. Always track the triple (row, col, key_state) as the complete state, not just the position.

