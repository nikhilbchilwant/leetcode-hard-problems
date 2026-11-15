# 1659. Maximize Grid Happiness

> Source: https://algo.monster/liteproblems/1659

## Problem Statement

You have an `m x n` grid and two types of people to potentially place in it: introverts and extroverts. You're given `introvertsCount` introverts and `extrovertsCount` extroverts available to place.

The key rules are:
- You can choose to place any number of people (from 0 up to the total available) in the grid
- Each person occupies exactly one cell
- Not all cells need to be filled, and not all people need to be placed

Each person type has different happiness calculations:
- **Introverts**: Start with `120` happiness points, but lose `30` points for each neighbor they have
- **Extroverts**: Start with `40` happiness points and gain `20` points for each neighbor they have

Neighbors are defined as people in directly adjacent cells (up, down, left, right) - diagonal cells don't count as neighbors.

The grid happiness is the sum of all placed people's happiness values. Your goal is to find the maximum possible grid happiness by optimally choosing:
1. How many people to place
2. Which type of people to place
3. Where to place them in the grid

For example, if you place an introvert with 2 neighbors, their happiness would be `120 - 30*2 = 60`. If you place an extrovert with 2 neighbors, their happiness would be `40 + 20*2 = 80`.

The problem asks you to return the maximum achievable grid happiness value.

## Intuition

The first observation is that the grid is small (`m, n ≤ 5`), which suggests we can use state compression techniques. Each cell has three possible states: empty (0), has an introvert (1), or has an extrovert (2). This naturally leads to thinking about representing each row's state as a ternary number.

Since we need to maximize happiness and the happiness of each person depends on their neighbors, we realize that:
1. The happiness within a row depends on horizontal neighbors
2. The happiness between rows depends on vertical neighbors

This suggests processing the grid row by row. When we place people in row `i`, we need to know what was placed in row `i-1` to calculate the vertical neighbor interactions.

The key insight is that we can precompute several things:
- `f[state]`: The base happiness for a given row state (considering only horizontal neighbors within that row)
- `g[state1][state2]`: The happiness change when two row states are adjacent vertically

For example, if we have state `[1, 0, 2]` (introvert, empty, extrovert) in a row:
- The introvert starts with 120 happiness
- The extrovert starts with 40 happiness  
- They're not horizontal neighbors, so no interaction
- Total base happiness for this row state is 160

When two rows are adjacent, we need to calculate the vertical interactions. If position `j` in the upper row has person type `a` and position `j` in the lower row has person type `b`, they affect each other's happiness according to a fixed pattern:
- Introvert-Introvert: both lose 30, total change = -60
- Introvert-Extrovert: introvert loses 30, extrovert gains 20, total change = -10  
- Extrovert-Extrovert: both gain 20, total change = 40

This leads us to [dynamic programming](/problems/dynamic_programming_intro) with [memoization](/problems/memoization_intro). We process row by row, keeping track of:
- Current row index `i`
- Previous row's state `pre` 
- Remaining introverts `ic`
- Remaining extroverts `ec`

For each row, we try all possible valid states (those that don't exceed our remaining people counts) and recursively solve for the rest of the grid, taking the maximum happiness achievable.

**Learn more about [Memoization](/problems/memoization_intro), [Dynamic Programming](/problems/dynamic_programming_intro) and [Bitmask](/problems/bitmask_intro) patterns.**

## Solution Approach

The solution uses ternary state compression combined with [dynamic programming](/problems/dynamic_programming_intro) and [memoization](/problems/memoization_intro). Let's walk through the implementation step by step:

### State Representation
Each row's state is represented as a ternary number where:
- `0` = empty cell
- `1` = introvert placed
- `2` = extrovert placed

For a row of length `n`, there are `3^n` possible states, stored in variable `mx`.

### Preprocessing Phase

1. **Row State Information**: For each possible row state `i` from `0` to `3^n - 1`:
   - Convert the state to its ternary representation and store in `bits[i]`
   - Count introverts (`ix[i]`) and extroverts (`ex[i]`) in this state
   - Calculate base happiness `f[i]` considering only horizontal neighbors within the row

2. **Happiness Interaction Matrix**: Define `h[a][b]` representing the happiness change when person type `a` is adjacent to person type `b`:
   ```
   h = [[0, 0, 0],      # empty adjacent to anything = 0
        [0, -60, -10],  # introvert adjacent to [empty, introvert, extrovert]
        [0, -10, 40]]   # extrovert adjacent to [empty, introvert, extrovert]
   ```

3. **Vertical Interaction Matrix**: Calculate `g[i][j]` representing the total happiness change when row state `i` is placed above row state `j`:
   ```python
   for each position k in the row:
       g[i][j] += h[bits[i][k]][bits[j][k]]
   ```

### [Dynamic Programming](/problems/dynamic_programming_intro) with [Memoization](/problems/memoization_intro)

The main solving function `dfs(i, pre, ic, ec)` represents:
- `i`: current row index
- `pre`: previous row's state
- `ic`: remaining introverts
- `ec`: remaining extroverts

The recurrence relation is:
```
dfs(i, pre, ic, ec) = max over all valid cur states {
    f[cur] + g[pre][cur] + dfs(i + 1, cur, ic - ix[cur], ec - ex[cur])
}
```

Where:
- `f[cur]` is the base happiness of row state `cur`
- `g[pre][cur]` is the vertical interaction happiness between consecutive rows
- We recurse to the next row with updated counts

### Base Cases
1. If `i == m` (all rows processed), return `0`
2. If `ic == 0` and `ec == 0` (no people left to place), return `0`

### Optimization
The `@cache` decorator memoizes results to avoid recalculating the same states multiple times, which is crucial since there can be many overlapping subproblems.

### Final Answer
The solution starts with `dfs(0, 0, introvertsCount, extrovertsCount)`, meaning we start from row 0 with no previous row state and all people available to place.

## Example Walkthrough

Let's walk through a small example with a 2×2 grid, 2 introverts, and 1 extrovert.

**Step 1: State Representation**
For a row of length 2, we have 3² = 9 possible states:
- State 0: `[0,0]` (empty, empty)
- State 1: `[0,1]` (empty, introvert)
- State 2: `[0,2]` (empty, extrovert)
- State 3: `[1,0]` (introvert, empty)
- State 4: `[1,1]` (introvert, introvert)
- State 5: `[1,2]` (introvert, extrovert)
- State 6: `[2,0]` (extrovert, empty)
- State 7: `[2,1]` (extrovert, introvert)
- State 8: `[2,2]` (extrovert, extrovert)

**Step 2: Calculate Base Happiness f[state]**
For each row state, calculate happiness considering only horizontal neighbors:
- f[0] = 0 (no people)
- f[1] = 120 (one introvert, no neighbors)
- f[2] = 40 (one extrovert, no neighbors)
- f[3] = 120 (one introvert, no neighbors)
- f[4] = 120 + 120 - 30 - 30 = 180 (two introverts, each loses 30)
- f[5] = 120 + 40 - 30 + 20 = 150 (introvert loses 30, extrovert gains 20)
- f[6] = 40 (one extrovert, no neighbors)
- f[7] = 40 + 120 + 20 - 30 = 150 (extrovert gains 20, introvert loses 30)
- f[8] = 40 + 40 + 20 + 20 = 120 (two extroverts, each gains 20)

**Step 3: Calculate Vertical Interactions g[i][j]**
For example, g[4][5] (state [1,1] above state [1,2]):
- Position 0: introvert above introvert → -60 happiness
- Position 1: introvert above extrovert → -10 happiness
- Total: g[4][5] = -70

**Step 4: Dynamic Programming**
Starting with dfs(0, 0, 2, 1):

*Row 0 choices:*
- State 0 (empty): happiness = 0 + dfs(1, 0, 2, 1)
- State 1 (0,I): happiness = 120 + dfs(1, 1, 1, 1)
- State 3 (I,0): happiness = 120 + dfs(1, 3, 1, 1)
- State 4 (I,I): happiness = 180 + dfs(1, 4, 0, 1)
- State 2 (0,E): happiness = 40 + dfs(1, 2, 2, 0)
- State 6 (E,0): happiness = 40 + dfs(1, 6, 2, 0)

Let's trace state 4 (I,I) for row 0:
- Base happiness: 180
- Remaining: 0 introverts, 1 extrovert

*Row 1 choices after row 0 = [I,I]:*
- State 0: 0 + 0 = 0
- State 2 (0,E): 40 + g[4][2] = 40 + (-30) = 10
- State 6 (E,0): 40 + g[4][6] = 40 + (-30) = 10

Best for row 1 is 10, so total = 180 + 10 = 190

After exploring all possibilities, the optimal placement might be:
```
Row 0: [I, I]  (state 4, happiness = 180)
Row 1: [0, E]  (state 2, happiness = 40 - 30 = 10)
```
Total grid happiness = 190

This demonstrates how the algorithm:
1. Tries different row configurations
2. Calculates horizontal interactions within rows
3. Calculates vertical interactions between adjacent rows
4. Tracks remaining people counts
5. Memoizes results to avoid recalculation
6. Returns the maximum achievable happiness

## Python Solution

```python
class Solution:
    def getMaxGridHappiness(
        self, m: int, n: int, introvertsCount: int, extrovertsCount: int
    ) -> int:
        """
        Calculate maximum grid happiness by placing introverts and extroverts.
        Uses dynamic programming with state compression.
      
        Args:
            m: Number of rows in the grid
            n: Number of columns in the grid
            introvertsCount: Number of introverts to place
            extrovertsCount: Number of extroverts to place
          
        Returns:
            Maximum happiness score achievable
        """
      
        @cache
        def dfs(row: int, prev_state: int, introverts_left: int, extroverts_left: int) -> int:
            """
            Dynamic programming function to calculate max happiness.
          
            Args:
                row: Current row being processed
                prev_state: State of the previous row (ternary representation)
                introverts_left: Remaining introverts to place
                extroverts_left: Remaining extroverts to place
              
            Returns:
                Maximum happiness from current position onwards
            """
            # Base cases: reached end of grid or no people left to place
            if row == m or (introverts_left == 0 and extroverts_left == 0):
                return 0
          
            max_happiness = 0
            # Try all possible states for current row
            for curr_state in range(max_states):
                # Check if we have enough people for this state
                if introverts_in_state[curr_state] <= introverts_left and \
                   extroverts_in_state[curr_state] <= extroverts_left:
                    # Calculate happiness for current row + interaction with previous row
                    current_happiness = base_happiness[curr_state] + interaction_happiness[prev_state][curr_state]
                    # Recursively calculate happiness for remaining rows
                    future_happiness = dfs(
                        row + 1, 
                        curr_state, 
                        introverts_left - introverts_in_state[curr_state], 
                        extroverts_left - extroverts_in_state[curr_state]
                    )
                    max_happiness = max(max_happiness, current_happiness + future_happiness)
          
            return max_happiness
      
        # Maximum possible states for a row (3^n: empty, introvert, extrovert)
        max_states = 3 ** n
      
        # Precomputed values for efficiency
        base_happiness = [0] * max_states  # Base happiness for each row state
        interaction_happiness = [[0] * max_states for _ in range(max_states)]  # Interaction between adjacent rows
      
        # Happiness matrix for adjacent cells
        # happiness_matrix[i][j] = happiness change when cell type i is adjacent to cell type j
        # 0: empty, 1: introvert, 2: extrovert
        happiness_matrix = [
            [0, 0, 0],      # Empty cell adjacent to anything
            [0, -60, -10],  # Introvert adjacent to [empty, introvert, extrovert]
            [0, -10, 40]    # Extrovert adjacent to [empty, introvert, extrovert]
        ]
      
        # Decode each state into cell configurations
        state_cells = [[0] * n for _ in range(max_states)]
        introverts_in_state = [0] * max_states
        extroverts_in_state = [0] * max_states
      
        # Precompute state information
        for state in range(max_states):
            temp_state = state
            for col in range(n):
                # Extract cell type using ternary representation
                temp_state, cell_type = divmod(temp_state, 3)
                state_cells[state][col] = cell_type
              
                # Count people and calculate base happiness
                if cell_type == 1:  # Introvert
                    introverts_in_state[state] += 1
                    base_happiness[state] += 120
                elif cell_type == 2:  # Extrovert
                    extroverts_in_state[state] += 1
                    base_happiness[state] += 40
              
                # Add horizontal interaction happiness (within same row)
                if col > 0:
                    base_happiness[state] += happiness_matrix[cell_type][state_cells[state][col - 1]]
      
        # Precompute vertical interaction happiness between adjacent rows
        for prev_state in range(max_states):
            for curr_state in range(max_states):
                for col in range(n):
                    interaction_happiness[prev_state][curr_state] += \
                        happiness_matrix[state_cells[prev_state][col]][state_cells[curr_state][col]]
      
        # Start DFS from first row with no previous state
        return dfs(0, 0, introvertsCount, extrovertsCount)
```

## Time and Space Complexity

**Time Complexity:** `O(3^(2n) × m × ic × ec + 3^(2n) × n)`

The time complexity consists of two main parts:

1. **Preprocessing phase:** `O(3^(2n) × n)`
   - Computing arrays `f`, `bits`, `ix`, `ex` requires iterating through all `3^n` states, and for each state, we process `n` positions: `O(3^n × n)`
   - Computing the 2D array `g` requires iterating through all pairs of states (`3^n × 3^n`) and for each pair, comparing `n` positions: `O(3^(2n) × n)`

2. **Dynamic programming phase:** `O(3^n × m × ic × ec)`
   - The `dfs` function has state space of `(i, pre, ic, ec)` where:
     - `i` ranges from `0` to `m-1` (m states)
     - `pre` can be any of the `3^n` masks
     - `ic` ranges from `0` to `introvertsCount`
     - `ec` ranges from `0` to `extrovertsCount`
   - For each state, we iterate through all `3^n` possible current masks
   - Total: `O(3^n × m × ic × ec × 3^n) = O(3^(2n) × m × ic × ec)`

The overall time complexity is dominated by the DP phase when `m × ic × ec` is large, giving us `O(3^(2n) × m × ic × ec)`. When including preprocessing, it becomes `O(3^(2n) × (m × ic × ec + n))`.

**Space Complexity:** `O(3^(2n) + 3^n × m × ic × ec)`

The space complexity includes:
- Arrays `f`, `ix`, `ex`: each of size `3^n`, total `O(3^n)`
- 2D array `bits`: `3^n × n`, which is `O(3^n × n)`
- 2D array `g`: `3^n × 3^n`, which is `O(3^(2n))`
- Memoization cache for `dfs`: stores results for all possible states `(i, pre, ic, ec)`, which is `O(m × 3^n × ic × ec)`

The dominant terms are `O(3^(2n))` from array `g` and `O(3^n × m × ic × ec)` from the memoization cache, resulting in total space complexity of `O(3^(2n) + 3^n × m × ic × ec)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect State Encoding/Decoding**
One of the most common mistakes is incorrectly converting between ternary representation and the state index. Many implementations get the order wrong or mishandle the base-3 conversion.

**Pitfall Example:**
```python
# Wrong: This reverses the order of cells
for col in range(n):
    cell_type = state % 3
    state //= 3
    state_cells[state][col] = cell_type  # This fills from left to right
```

**Solution:**
The correct approach stores the rightmost column in the least significant digit:
```python
# Correct: Consistent encoding/decoding
temp_state = state
for col in range(n):
    temp_state, cell_type = divmod(temp_state, 3)
    state_cells[state][col] = cell_type
```

### 2. **Double-Counting Neighbor Interactions**
When calculating happiness, it's easy to accidentally count the interaction between two neighbors twice - once from each person's perspective.

**Pitfall Example:**
```python
# Wrong: Double-counting horizontal neighbors
for col in range(n):
    if cell_type == 1:
        base_happiness[state] += 120
        if col > 0:  # Left neighbor
            base_happiness[state] -= 30
        if col < n-1:  # Right neighbor  
            base_happiness[state] -= 30  # This will be counted again!
```

**Solution:**
Only count the interaction once, either when processing the left cell looking right, or the right cell looking left:
```python
# Correct: Count interaction only once
if col > 0:
    base_happiness[state] += happiness_matrix[cell_type][state_cells[state][col - 1]]
```

### 3. **Forgetting to Handle Empty Grid States**
A subtle bug occurs when not properly handling the case where placing no one might be optimal.

**Pitfall Example:**
```python
# Wrong: Forces at least one person to be placed
def dfs(row, prev_state, introverts_left, extroverts_left):
    if row == m:
        return 0
  
    max_happiness = -float('inf')  # Wrong! This prevents empty grid from being valid
    for curr_state in range(max_states):
        # ...
```

**Solution:**
Initialize with 0 to allow empty configurations:
```python
# Correct: Allow empty grid as valid option
max_happiness = 0  # Start with 0, not negative infinity
```

### 4. **Memory Limit Issues with Large Grids**
For larger values of n, the state space (3^n) grows exponentially, potentially causing memory issues.

**Pitfall Example:**
```python
# Wrong: Creating all states even when m*n is small
max_states = 3 ** n  # Could be huge even if m is 1
state_cells = [[0] * n for _ in range(max_states)]  # Memory explosion
```

**Solution:**
Add early validation and consider the actual grid size:
```python
# Better: Check feasibility early
if n > 5:  # 3^5 = 243 states is manageable
    # Consider alternative approach or optimize memory usage
    pass

# Or limit based on actual capacity
max_people = min(m * n, introvertsCount + extrovertsCount)
if max_people == 0:
    return 0
```

### 5. **Incorrect Happiness Matrix Values**
The happiness changes for neighbors must account for bidirectional effects.

**Pitfall Example:**
```python
# Wrong: Only considering one-directional effect
happiness_matrix = [
    [0, 0, 0],
    [0, -30, -30],  # Wrong! Should be -60 for two introverts
    [0, 20, 20]     # Wrong! Should be 40 for two extroverts
]
```

**Solution:**
Each value should represent the total happiness change for both neighbors:
```python
# Correct: Bidirectional effects
happiness_matrix = [
    [0, 0, 0],      
    [0, -60, -10],  # -30 (introvert loses) + -30 (other introvert loses) = -60
    [0, -10, 40]    # -30 (introvert loses) + 20 (extrovert gains) = -10
]
```

