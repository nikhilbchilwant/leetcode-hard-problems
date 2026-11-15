# 3363. Find the Maximum Number of Fruits Collected

> Source: https://algo.monster/liteproblems/3363

## Problem Statement

You have an `n x n` grid representing a game dungeon where each room `(i, j)` contains `fruits[i][j]` fruits. Three children start at three corner positions:
- Child 1: starts at `(0, 0)` (top-left corner)
- Child 2: starts at `(0, n-1)` (top-right corner)  
- Child 3: starts at `(n-1, 0)` (bottom-left corner)

All three children need to reach the room `(n-1, n-1)` (bottom-right corner) in exactly `n-1` moves.

**Movement Rules:**
- Child from `(0, 0)` can move from room `(i, j)` to: `(i+1, j+1)`, `(i+1, j)`, or `(i, j+1)`
- Child from `(0, n-1)` can move from room `(i, j)` to: `(i+1, j-1)`, `(i+1, j)`, or `(i+1, j+1)`
- Child from `(n-1, 0)` can move from room `(i, j)` to: `(i-1, j+1)`, `(i, j+1)`, or `(i+1, j+1)`

**Fruit Collection Rules:**
- When a child enters a room, they collect all fruits in that room
- If multiple children visit the same room, only one child collects the fruits (the room becomes empty after the first child leaves)

The goal is to find the maximum total number of fruits that all three children can collect combined.

**Key Insight:** Due to the movement constraints and the requirement of exactly `n-1` moves:
- Child 1 (starting from `(0, 0)`) can only reach `(n-1, n-1)` by moving along the main diagonal
- Child 2 (starting from `(0, n-1)`) can only move through rooms above the main diagonal
- Child 3 (starting from `(n-1, 0)`) can only move through rooms below the main diagonal

This ensures that except for the destination `(n-1, n-1)`, no room is visited by multiple children, simplifying the problem to finding optimal paths for each child in their respective regions.

## Intuition

The key observation is understanding the movement constraints and what they mean for each child's possible paths.

First, let's think about Child 1 starting from `(0, 0)`. To reach `(n-1, n-1)` in exactly `n-1` moves, we need to increase both row and column indices from 0 to `n-1`. Since we have exactly `n-1` moves and need to increase each coordinate by `n-1`, the only way is to move diagonally at each step. This means Child 1 must follow the main diagonal path: `(0,0) → (1,1) → (2,2) → ... → (n-1, n-1)`.

For Child 2 starting from `(0, n-1)`, they need to move down (increase row from 0 to `n-1`) while moving left (decrease column from `n-1` to `n-1`, so net change is 0). Looking at their movement options, they can move diagonally down-left, straight down, or diagonally down-right. Since they start at the top-right corner and must reach the bottom-right corner in exactly `n-1` moves, they can only traverse rooms that are above or on the main diagonal.

Similarly, Child 3 starting from `(n-1, 0)` needs to move right (increase column from 0 to `n-1`) while their row stays at `n-1`. Their movement options allow them to move up-right, right, or down-right. Given their starting position and destination, they can only traverse rooms that are below or on the main diagonal.

This spatial separation is crucial - it means the three children operate in mostly non-overlapping regions:
- Child 1: main diagonal only
- Child 2: upper triangle (above main diagonal)
- Child 3: lower triangle (below main diagonal)

The only potential overlap is at the destination `(n-1, n-1)`, but since all children must reach there, we can handle this separately.

Since the regions don't overlap (except at the destination), we can solve each child's path independently using [dynamic programming](/problems/dynamic_programming_intro). For each child in their respective region, we want to find the path that collects maximum fruits.

For [dynamic programming](/problems/dynamic_programming_intro), we define `f[i][j]` as the maximum fruits a child can collect when reaching position `(i, j)`. The state transitions follow from the movement rules - at each position, we take the maximum from all valid previous positions plus the fruits at the current position.

The final answer combines:
1. All fruits on the main diagonal (collected by Child 1)
2. Maximum fruits Child 2 can collect reaching `(n-2, n-1)` (one step before destination)
3. Maximum fruits Child 3 can collect reaching `(n-1, n-2)` (one step before destination)

We use positions `(n-2, n-1)` and `(n-1, n-2)` because these are the last positions before the destination for Children 2 and 3 respectively, avoiding double-counting the fruits at `(n-1, n-1)`.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

Based on our intuition, we implement the solution using [dynamic programming](/problems/dynamic_programming_intro) with a single 2D array that we reuse for calculating paths for both Child 2 and Child 3.

**Step 1: Initialize the DP array**
We create a 2D array `f` of size `n x n`, initialized with negative infinity (`-inf`) to represent unvisited states. This ensures we only consider valid paths.

**Step 2: Calculate maximum fruits for Child 2 (starting from `(0, n-1)`)**

We set the starting position: `f[0][n-1] = fruits[0][n-1]`

Then we iterate through the upper triangle region. For each position `(i, j)` where `j > i` (above the main diagonal), we calculate:

```python
f[i][j] = max(f[i-1][j], f[i-1][j-1]) + fruits[i][j]
if j + 1 < n:
    f[i][j] = max(f[i][j], f[i-1][j+1] + fruits[i][j])
```

This implements the state transition: Child 2 can reach `(i, j)` from three possible previous positions:
- `(i-1, j-1)` - diagonal down-left move
- `(i-1, j)` - straight down move  
- `(i-1, j+1)` - diagonal down-right move (only valid if `j+1 < n`)

We take the maximum fruits from these previous positions and add the fruits at the current position.

**Step 3: Calculate maximum fruits for Child 3 (starting from `(n-1, 0)`)**

We reuse the same `f` array. First, set the starting position: `f[n-1][0] = fruits[n-1][0]`

Then iterate through the lower triangle region. For each position `(i, j)` where `i > j` (below the main diagonal), we calculate:

```python
f[i][j] = max(f[i][j-1], f[i-1][j-1]) + fruits[i][j]
if i + 1 < n:
    f[i][j] = max(f[i][j], f[i+1][j-1] + fruits[i][j])
```

This implements Child 3's possible moves to reach `(i, j)` from:
- `(i-1, j-1)` - diagonal up-right move
- `(i, j-1)` - straight right move
- `(i+1, j-1)` - diagonal down-right move (only valid if `i+1 < n`)

**Step 4: Calculate the final answer**

The total maximum fruits collected is:
```python
sum(fruits[i][i] for i in range(n)) + f[n-2][n-1] + f[n-1][n-2]
```

This combines:
- `sum(fruits[i][i] for i in range(n))`: All fruits on the main diagonal collected by Child 1
- `f[n-2][n-1]`: Maximum fruits Child 2 can collect when reaching `(n-2, n-1)`
- `f[n-1][n-2]`: Maximum fruits Child 3 can collect when reaching `(n-1, n-2)`

The positions `(n-2, n-1)` and `(n-1, n-2)` are the last positions before the destination for Children 2 and 3 respectively. Since all three children reach `(n-1, n-1)`, the fruits there are already counted in the diagonal sum for Child 1.

**Time Complexity:** `O(n²)` as we iterate through the grid twice
**Space Complexity:** `O(n²)` for the DP array

## Example Walkthrough

Let's walk through a small example with a 3×3 grid:

```
fruits = [[1, 2, 3],
          [4, 5, 6],
          [7, 8, 9]]
```

**Step 1: Identify each child's path constraints**

- Child 1 (starts at (0,0)): Must follow the main diagonal to reach (2,2) in exactly 2 moves
  - Path: (0,0) → (1,1) → (2,2)
  - Collects: 1 + 5 + 9 = 15

- Child 2 (starts at (0,2)): Can only move through the upper triangle
  - Must reach (2,2) from (0,2) in 2 moves
  - Can visit positions above the diagonal

- Child 3 (starts at (2,0)): Can only move through the lower triangle
  - Must reach (2,2) from (2,0) in 2 moves
  - Can visit positions below the diagonal

**Step 2: Calculate optimal path for Child 2**

Initialize DP array `f` with -∞, then set starting position:
- `f[0][2] = 3`

For position (1,2) (above diagonal since 2 > 1):
- Can come from: (0,1) or (0,2)
- `f[0][1]` is -∞ (not reachable)
- `f[0][2] = 3`
- So `f[1][2] = 3 + 6 = 9`

Child 2's optimal path: (0,2) → (1,2) → (2,2)
Maximum fruits before destination: `f[1][2] = 9`

**Step 3: Calculate optimal path for Child 3**

Reset relevant parts of `f` and set starting position:
- `f[2][0] = 7`

For position (2,1) (below diagonal since 2 > 1):
- Can come from: (1,0) or (2,0)
- `f[1][0]` is -∞ (not reachable)
- `f[2][0] = 7`
- So `f[2][1] = 7 + 8 = 15`

Child 3's optimal path: (2,0) → (2,1) → (2,2)
Maximum fruits before destination: `f[2][1] = 15`

**Step 4: Calculate total fruits**

Total = Diagonal sum + Child 2's fruits at (1,2) + Child 3's fruits at (2,1)
Total = (1 + 5 + 9) + 9 + 15 = 15 + 9 + 15 = 39

**Verification of non-overlapping paths:**
- Child 1 visits: (0,0), (1,1), (2,2)
- Child 2 visits: (0,2), (1,2), (2,2)
- Child 3 visits: (2,0), (2,1), (2,2)

Only (2,2) is visited by multiple children, and we've counted its fruits only once (in Child 1's diagonal sum).

## Python Solution

```python
class Solution:
    def maxCollectedFruits(self, fruits: List[List[int]]) -> int:
        n = len(fruits)
      
        # Initialize DP table with negative infinity
        # dp[i][j] represents maximum fruits collected at position (i, j)
        dp = [[float('-inf')] * n for _ in range(n)]
      
        # First path: from top-right corner moving downward-left
        # Start from position (0, n-1)
        dp[0][n - 1] = fruits[0][n - 1]
      
        # Fill DP table for the first path
        # Can move from (i-1, j), (i-1, j-1), or (i-1, j+1) to (i, j)
        for i in range(1, n):
            for j in range(i + 1, n):  # Only consider valid positions where j > i
                # Try moving from directly above
                dp[i][j] = max(dp[i][j], dp[i - 1][j] + fruits[i][j])
              
                # Try moving from upper-left diagonal
                if j - 1 >= 0:
                    dp[i][j] = max(dp[i][j], dp[i - 1][j - 1] + fruits[i][j])
              
                # Try moving from upper-right diagonal
                if j + 1 < n:
                    dp[i][j] = max(dp[i][j], dp[i - 1][j + 1] + fruits[i][j])
      
        # Second path: from bottom-left corner moving rightward-up
        # Reset starting position at (n-1, 0)
        dp[n - 1][0] = fruits[n - 1][0]
      
        # Fill DP table for the second path
        # Can move from (i, j-1), (i-1, j-1), or (i+1, j-1) to (i, j)
        for j in range(1, n):
            for i in range(j + 1, n):  # Only consider valid positions where i > j
                # Try moving from left
                dp[i][j] = max(dp[i][j], dp[i][j - 1] + fruits[i][j])
              
                # Try moving from upper-left diagonal
                if i - 1 >= 0:
                    dp[i][j] = max(dp[i][j], dp[i - 1][j - 1] + fruits[i][j])
              
                # Try moving from lower-left diagonal
                if i + 1 < n:
                    dp[i][j] = max(dp[i][j], dp[i + 1][j - 1] + fruits[i][j])
      
        # Calculate total fruits collected:
        # Main diagonal path + first path ending at (n-2, n-1) + second path ending at (n-1, n-2)
        diagonal_sum = sum(fruits[i][i] for i in range(n))
        first_path_max = dp[n - 2][n - 1]
        second_path_max = dp[n - 1][n - 2]
      
        return diagonal_sum + first_path_max + second_path_max
```

## Time and Space Complexity

The time complexity is `O(n²)` and the space complexity is `O(n²)`, where `n` is the side length of the grid.

**Time Complexity Analysis:**
- The code initializes a 2D array `f` with dimensions `n × n`, which takes `O(n²)` time.
- The first nested loop iterates through positions where `i` ranges from `1` to `n-1` and for each `i`, `j` ranges from `i+1` to `n-1`. This results in approximately `n²/2` iterations, which is `O(n²)`.
- The second nested loop similarly iterates through positions where `j` ranges from `1` to `n-1` and for each `j`, `i` ranges from `j+1` to `n-1`. This also results in approximately `n²/2` iterations, which is `O(n²)`.
- The final return statement computes the sum of diagonal elements, which takes `O(n)` time.
- Overall: `O(n²) + O(n²) + O(n²) + O(n) = O(n²)`

**Space Complexity Analysis:**
- The code creates a 2D array `f` of size `n × n`, which requires `O(n²)` space.
- No other significant auxiliary space is used besides a few variables.
- Therefore, the space complexity is `O(n²)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### Pitfall 1: Incorrect DP Table Reuse Between Paths

**The Problem:** When reusing the same DP table for both Child 2 and Child 3, failing to properly reset or isolate the calculations can lead to incorrect results. The code initializes `dp[0][n-1]` for Child 2, then later sets `dp[n-1][0]` for Child 3, but the previous calculations for Child 2 remain in the table and could interfere with Child 3's path calculation.

**Why It Happens:** The DP values from Child 2's calculation (in the upper triangle) might accidentally be accessed when computing Child 3's path if boundary conditions aren't carefully checked.

**Solution:** Ensure strict boundary checking so that each child's calculation only accesses valid positions:
- For Child 2: Only compute positions where `j > i` (upper triangle)
- For Child 3: Only compute positions where `i > j` (lower triangle)
- Add explicit checks before accessing any DP values

### Pitfall 2: Off-by-One Errors in Movement Validation

**The Problem:** The movement rules have different constraints for each child. For example, when Child 2 at position `(i, j)` tries to move from `(i-1, j+1)`, we need to check if `j+1 < n`. However, it's easy to forget these boundary checks or apply them incorrectly.

**Why It Happens:** Each child has three possible previous positions, and some of these moves require boundary validation while others don't, making it easy to miss or misplace checks.

**Solution:** Always validate array bounds before accessing:
```python
# For Child 2 moving from (i-1, j+1)
if j + 1 < n and i - 1 >= 0:  # Check both dimensions
    dp[i][j] = max(dp[i][j], dp[i - 1][j + 1] + fruits[i][j])

# For Child 3 moving from (i+1, j-1)  
if i + 1 < n and j - 1 >= 0:  # Check both dimensions
    dp[i][j] = max(dp[i][j], dp[i + 1][j - 1] + fruits[i][j])
```

### Pitfall 3: Incorrect Loop Order for Child 3

**The Problem:** The loop order matters significantly for dynamic programming. For Child 3 starting at `(n-1, 0)` and moving towards `(n-1, n-1)`, using the wrong loop order can cause attempts to access DP values that haven't been computed yet.

**Why It Happens:** Child 3 moves rightward and potentially up/down. The outer loop should iterate through columns (j) and the inner loop through rows (i) to ensure we compute positions in the correct dependency order.

**Solution:** Use the correct loop structure:
```python
# Correct order for Child 3
for j in range(1, n):  # Column first (moving right)
    for i in range(j + 1, n):  # Then rows in valid range
        # Compute dp[i][j] based on positions to the left
```

### Pitfall 4: Missing Initialization After Reusing DP Table

**The Problem:** When reusing the DP table between children, forgetting to handle the initialization properly can cause incorrect propagation of values. The code sets `dp[n-1][0] = fruits[n-1][0]` for Child 3, but if this overwrites a value needed from Child 2's calculation, it could affect the final result.

**Why It Happens:** The regions for Child 2 (upper triangle) and Child 3 (lower triangle) are separate except at the boundaries, but the initialization points and final collection points need careful handling.

**Solution:** Ensure that:
1. Child 2's final position `(n-2, n-1)` is computed and stored before starting Child 3's calculation
2. Child 3's starting position `(n-1, 0)` doesn't interfere with Child 2's results
3. Consider storing intermediate results if needed:
```python
# Store Child 2's result before processing Child 3
child2_result = dp[n - 2][n - 1]
# Now safe to reinitialize for Child 3
dp[n - 1][0] = fruits[n - 1][0]
```

