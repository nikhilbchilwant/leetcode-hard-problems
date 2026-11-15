# 174. Dungeon Game

> Source: https://algo.monster/liteproblems/174

## Problem Statement

You have a 2D grid representing a dungeon with `m x n` rooms. A knight starts at the top-left corner and needs to rescue a princess imprisoned in the bottom-right corner.

The knight begins with some amount of health points (a positive integer). If his health ever drops to 0 or below, he dies immediately.

Each room in the dungeon contains a value:
- **Negative values** represent demons that damage the knight (reduce health by that amount)
- **Zero** represents an empty room (no health change)
- **Positive values** represent magic orbs that heal the knight (increase health by that amount)

The knight can only move **right** or **down** in each step to reach the princess as quickly as possible.

Your task is to find the **minimum initial health** the knight must have to successfully reach the princess while keeping his health above 0 throughout the entire journey.

For example, if the dungeon is:
```
[[-3, 5],
 [1, -4]]
```
The knight would need at least 4 initial health points, following the path start → right → down → princess (4 − 3 = 1, then 1 + 5 = 6, then 6 − 4 = 2).

**Important notes:**
- The knight must have at least 1 health point at all times (including after reaching the princess)
- Even the starting room (top-left) and ending room (bottom-right) can contain threats or power-ups

## Intuition

The key insight is to work **backwards** from the destination to the starting point. Why? Because if we try to work forward, we'd need to keep track of all possible paths and their minimum health requirements, which becomes complex.

Think about it this way: when the knight reaches any cell, he needs enough health to:
1. Survive that cell's effect (demon damage or healing)
2. Have enough health to complete the rest of the journey to the princess

By working backwards, we can answer: "What's the minimum health needed when entering this cell to successfully reach the princess?"

Let's consider the princess's cell first. After dealing with whatever is in that room, the knight needs at least 1 health to survive. So if the princess's room has a demon dealing 5 damage, the knight needs at least 6 health when entering that room.

For any other cell `(i, j)`, the knight will move either right to `(i, j+1)` or down to `(i+1, j)`. Since the knight wants to minimize initial health, he'll choose the path that requires less health. Therefore:
- We need to know the minimum health required at `(i, j+1)` and `(i+1, j)`
- The knight will choose the path with the smaller requirement
- Then we account for the current cell's effect

The formula becomes: `dp[i][j] = max(1, min(dp[i+1][j], dp[i][j+1]) - dungeon[i][j])`

The `max(1, ...)` ensures the knight always has at least 1 health. The subtraction works because:
- If the current cell has a demon (-5), we need MORE initial health: `required - (-5) = required + 5`
- If the current cell has healing (+5), we need LESS initial health: `required - 5`

This backward dynamic programming approach elegantly solves the problem in `O(m*n)` time.

## Solution Approach

We implement the backward dynamic programming approach using a 2D array `dp` where `dp[i][j]` represents the minimum health required when entering cell `(i, j)` to successfully reach the princess.

**Step 1: Initialize the DP table**

Create a `(m+1) x (n+1)` table filled with infinity values. The extra row and column serve as boundaries to handle edge cases elegantly.

```python
dp = [[inf] * (n + 1) for _ in range(m + 1)]
```

**Step 2: Set base cases**

Since we work backwards, we need to set the values for cells adjacent to the princess's cell `(m-1, n-1)`:
- `dp[m][n-1] = 1`: Virtual cell below the princess
- `dp[m-1][n] = 1`: Virtual cell to the right of the princess

These ensure that after dealing with the princess's cell, the knight has at least 1 health.

```python
dp[m][n - 1] = dp[m - 1][n] = 1
```

**Step 3: Fill the DP table backwards**

Iterate from bottom-right to top-left. For each cell `(i, j)`, calculate the minimum health needed:

```python
for i in range(m - 1, -1, -1):
    for j in range(n - 1, -1, -1):
        dp[i][j] = max(1, min(dp[i + 1][j], dp[i][j + 1]) - dungeon[i][j])
```

The formula `max(1, min(dp[i+1][j], dp[i][j+1]) - dungeon[i][j])` works as follows:
- `min(dp[i+1][j], dp[i][j+1])`: Choose the path requiring less health (either go down or right)
- `- dungeon[i][j]`: Adjust for the current cell's effect
  - If `dungeon[i][j]` is negative (demon), this increases the required health
  - If `dungeon[i][j]` is positive (healing), this decreases the required health
- `max(1, ...)`: Ensure the knight always has at least 1 health point

**Step 4: Return the result**

`dp[0][0]` contains the minimum initial health required at the starting position.

```python
return dp[0][0]
```

The algorithm runs in `O(m*n)` time complexity and uses `O(m*n)` space for the DP table.

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

Consider this dungeon:
```
[[-3, 5],
 [1, -4]]
```

We need to find the minimum initial health for the knight to reach from top-left (0,0) to bottom-right (1,1).

**Step 1: Initialize the DP table**

We create a 3x3 DP table (one extra row and column for boundaries):
```
dp = [[inf, inf, inf],
      [inf, inf, inf],
      [inf, inf, inf]]
```

**Step 2: Set base cases**

We set the cells adjacent to the princess's position (virtual boundaries):
```
dp[2][1] = 1  (below princess)
dp[1][2] = 1  (right of princess)

dp = [[inf, inf, inf],
      [inf, inf,  1 ],
      [inf,  1,  inf]]
```

**Step 3: Fill the DP table backwards**

Starting from the princess's cell (1,1) where dungeon[1][1] = -4:
- `dp[1][1] = max(1, min(dp[2][1], dp[1][2]) - dungeon[1][1])`
- `dp[1][1] = max(1, min(1, 1) - (-4))`
- `dp[1][1] = max(1, 1 + 4) = 5`

The knight needs 5 health entering this room to survive the 4 damage and have 1 health left.

Next, cell (1,0) where dungeon[1][0] = 1:
- `dp[1][0] = max(1, min(dp[2][0], dp[1][1]) - dungeon[1][0])`
- `dp[1][0] = max(1, min(inf, 5) - 1)`
- `dp[1][0] = max(1, 4) = 4`

The healing orb reduces the requirement from 5 to 4.

Cell (0,1) where dungeon[0][1] = 5:
- `dp[0][1] = max(1, min(dp[1][1], dp[0][2]) - dungeon[0][1])`
- `dp[0][1] = max(1, min(5, inf) - 5)`
- `dp[0][1] = max(1, 0) = 1`

The strong healing orb means we only need 1 health to enter this cell.

Finally, cell (0,0) where dungeon[0][0] = -3:
- `dp[0][0] = max(1, min(dp[1][0], dp[0][1]) - dungeon[0][0])`
- `dp[0][0] = max(1, min(4, 1) - (-3))`
- `dp[0][0] = max(1, 1 + 3) = 4`

The demon in the starting room requires 4 initial health.

**Final DP table:**
```
dp = [[ 4,  1, inf],
      [ 4,  5,  1 ],
      [inf, 1, inf]]
```

**Step 4: Return the result**

`dp[0][0] = 4` is our answer. The knight needs at least 4 initial health points.

Let's verify with the optimal path (right → down):
- Start at (0,0) with 4 health, take 3 damage → 1 health
- Move right to (0,1), gain 5 health → 6 health  
- Move down to (1,1), take 4 damage → 2 health
- Princess rescued with 2 health remaining!

## Python Solution

```python
class Solution:
    def calculateMinimumHP(self, dungeon: List[List[int]]) -> int:
        # Get dimensions of the dungeon
        rows, cols = len(dungeon), len(dungeon[0])
      
        # Initialize DP table with infinity values
        # Extra row and column for boundary conditions
        dp = [[float('inf')] * (cols + 1) for _ in range(rows + 1)]
      
        # Set base cases: minimum health needed just outside the princess room is 1
        dp[rows][cols - 1] = dp[rows - 1][cols] = 1
      
        # Fill DP table from bottom-right to top-left
        for row in range(rows - 1, -1, -1):
            for col in range(cols - 1, -1, -1):
                # Minimum health needed at current cell is:
                # - At least 1 (knight must survive)
                # - Minimum health from next cell minus current cell's value
                min_health_on_exit = min(dp[row + 1][col], dp[row][col + 1])
                dp[row][col] = max(1, min_health_on_exit - dungeon[row][col])
      
        # Return minimum initial health needed at starting position
        return dp[0][0]
```

## Time and Space Complexity

The time complexity is `O(m × n)`, where `m` is the number of rows and `n` is the number of columns of the dungeon. This is because the algorithm uses nested loops that iterate through each cell of the dungeon exactly once. The outer loop runs `m` times (from `m-1` to `0`), and the inner loop runs `n` times (from `n-1` to `0`) for each iteration of the outer loop.

The space complexity is `O(m × n)`. The algorithm creates a 2D DP table `dp` with dimensions `(m+1) × (n+1)` to store the minimum health required at each position. Since this auxiliary space grows proportionally with the size of the input dungeon, the space complexity is `O((m+1) × (n+1))`, which simplifies to `O(m × n)`.

## Common Pitfalls

### 1. **Attempting Forward Dynamic Programming**

Many people's first instinct is to use forward DP, tracking the minimum health needed as we traverse from top-left to bottom-right. This approach fails because we need to consider future threats when determining the minimum health at each position.

**Why it fails:** When moving forward, we don't know what dangers lie ahead. A cell with a large healing value might seem to allow very low health, but if the next cell has massive damage, we'd die.

**Example:**
```
dungeon = [[0, -5],
           [-6, 0]]
```
Forward DP might suggest we need only 1 health at start since the first cell is empty. But both routes pass a demon: going right needs 6 health to survive the -5, and going down needs 7 to survive the -6. The answer is 6, which depends on cells the knight has not reached yet.

**Solution:** Always use backward DP for this problem, starting from the destination and working backwards to determine minimum requirements.

### 2. **Incorrect Boundary Initialization**

A common mistake is initializing the boundary cells incorrectly or forgetting to set them properly.

**Wrong approach:**
```python
dp[rows-1][cols] = 0  # Wrong! This would mean no health needed
dp[rows][cols-1] = 0  # Wrong!
```

**Correct approach:**
```python
dp[rows-1][cols] = 1  # Knight needs at least 1 health after the princess
dp[rows][cols-1] = 1  # Knight needs at least 1 health after the princess
```

### 3. **Forgetting the Minimum Health Constraint**

Forgetting to enforce that the knight must always have at least 1 health point can lead to incorrect results.

**Wrong calculation:**
```python
dp[row][col] = min(dp[row+1][col], dp[row][col+1]) - dungeon[row][col]
# This could result in 0 or negative values
```

**Correct calculation:**
```python
dp[row][col] = max(1, min(dp[row+1][col], dp[row][col+1]) - dungeon[row][col])
# Ensures knight always has at least 1 health
```

### 4. **Misunderstanding Health Calculation with Positive Values**

When a room contains a healing orb (positive value), some might think it adds to the minimum required health. Actually, it reduces the health requirement for that path.

**Example:** If `dungeon[i][j] = 5` (healing) and we need 10 health for the next cell, then:
- We need `max(1, 10 - 5) = 5` health when entering this cell
- NOT `10 + 5 = 15` health

The healing orb helps us, so we need less initial health to survive the journey.

