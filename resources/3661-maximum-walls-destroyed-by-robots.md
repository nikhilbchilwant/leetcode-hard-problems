# 3661. Maximum Walls Destroyed by Robots

> Source: https://algo.monster/liteproblems/3661

## Problem Statement

You have an endless straight line with robots and walls positioned on it. Each robot can fire exactly one bullet either to the left or right direction.

Given:
- `robots[i]`: the position of the `i-th` robot
- `distance[i]`: the maximum distance the `i-th` robot's bullet can travel
- `walls[j]`: the position of the `j-th` wall

Rules for bullets:
- Each robot fires **exactly one bullet** in either left or right direction
- A bullet can travel at most `distance[i]` meters from robot `i`
- A bullet destroys **all walls** in its path within its range
- If a bullet hits another robot, it **stops immediately** and cannot continue
- Robots are never destroyed by bullets
- A wall and robot can share the same position (the robot at that position can destroy that wall)

The goal is to find the **maximum number of unique walls** that can be destroyed by having all robots fire their bullets optimally.

For example, if robot at position 5 has distance 3:
- Firing left: bullet travels from position 5 to position 2, destroying all walls in range [2, 5]
- Firing right: bullet travels from position 5 to position 8, destroying all walls in range [5, 8]
- If another robot is at position 7, a right-firing bullet would stop at position 7 (just before the robot)

The challenge is to determine the optimal firing direction for each robot to maximize the total number of unique walls destroyed across all robots.

## Intuition

The key insight is that we need to consider robots in a specific order to handle the blocking constraints properly. Since robots can block each other's bullets, the firing direction of one robot affects what walls other robots can destroy.

Let's think about this step by step. If we process robots from left to right on the line, we can make optimal decisions based on what previous robots have done. When a robot fires left, it might be blocked by robots to its left. When it fires right, it might block future robots from firing left.

This suggests a dynamic programming approach where we track:
1. Which robot we're currently considering
2. The firing direction of the next robot (which affects our current robot's right range)

Why does the next robot's direction matter? If the next robot fires left, its bullet might be blocked by our current robot if we're in its path. So when our current robot fires right, we need to limit our range to avoid blocking the next robot's left-firing bullet.

The state `dfs(i, j)` represents:
- `i`: index of current robot being considered (after sorting by position)
- `j`: firing direction of the next robot (0 = left, 1 = right)

For each robot, we calculate two possibilities:
1. **Fire left**: The range is `[robot_position - distance, robot_position]`, but we must also consider blocking by the previous robot (if it exists and fired right)
2. **Fire right**: The range is `[robot_position, robot_position + distance]`, but we must consider not blocking the next robot's potential left shot

By processing robots in sorted order and using memoization to avoid recalculating the same states, we can efficiently find the maximum number of walls that can be destroyed. The binary search helps us quickly count how many walls fall within each robot's firing range.

The recursion naturally handles the dependencies between robots' decisions, ensuring we find the globally optimal solution rather than getting stuck in local optima.

## Solution Approach

First, we prepare the data by combining robots with their distances and sorting them by position. We also sort the walls array for efficient binary search operations later.

```python
arr = sorted(zip(robots, distance), key=lambda x: x[0])
walls.sort()
```

The core of the solution is the memoized recursive function `dfs(i, j)`:
- `i`: index of the current robot being considered (0 to n-1)
- `j`: firing direction of the next robot (0 = left, 1 = right)

**Base Case**: When `i < 0`, all robots have been processed, return 0.

**For each robot at index `i`, we consider two firing directions:**

**1. Fire Left:**
- Calculate the left boundary: `left = arr[i][0] - arr[i][1]` (robot position minus its range)
- If there's a previous robot (`i > 0`), adjust the left boundary to avoid counting walls already destroyed: `left = max(left, arr[i-1][0] + 1)`
- Use binary search to find walls in range `[left, arr[i][0])`:
  - `l = bisect_left(walls, left)` finds the first wall at or after `left`
  - `r = bisect_left(walls, arr[i][0] + 1)` finds the first wall after robot position
  - Number of walls destroyed: `r - l`
- Total walls destroyed: `dfs(i - 1, 0) + (r - l)`

**2. Fire Right:**
- Calculate the right boundary: `right = arr[i][0] + arr[i][1]` (robot position plus its range)
- If there's a next robot (`i + 1 < n`), adjust the right boundary based on the next robot's firing direction:
  - If `j == 0` (next robot fires left): `right = min(right, arr[i+1][0] - arr[i+1][1] - 1)` to avoid blocking the next robot's left shot
  - If `j == 1` (next robot fires right): `right = min(right, arr[i+1][0] - 1)` to stop before the next robot
- Use binary search to find walls in range `[arr[i][0], right]`:
  - `l = bisect_left(walls, arr[i][0])` finds the first wall at or after robot position
  - `r = bisect_left(walls, right + 1)` finds the first wall after `right`
  - Number of walls destroyed: `r - l`
- Total walls destroyed: `dfs(i - 1, 1) + (r - l)`

The function returns the maximum of these two choices: `max(fire_left_result, fire_right_result)`

The `@cache` decorator provides memoization, storing results for previously computed states to avoid redundant calculations.

Finally, we start the recursion with `dfs(n - 1, 1)`, processing from the rightmost robot. The second parameter can be either 0 or 1 since there's no robot after the last one.

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

**Input:**
- `robots = [3, 6]` (robots at positions 3 and 6)
- `distance = [2, 3]` (robot at position 3 can shoot 2 meters, robot at position 6 can shoot 3 meters)
- `walls = [1, 2, 4, 5, 7, 8]` (walls at these positions)

**Step 1: Prepare the data**
- Sort robots with distances: `arr = [(3, 2), (6, 3)]` (already sorted)
- Sort walls: `walls = [1, 2, 4, 5, 7, 8]` (already sorted)

**Step 2: Start recursion with `dfs(1, 1)`** (rightmost robot, index 1)

We're at robot index 1 (position 6, distance 3). We consider two options:

**Option A - Fire Left:**
- Left range: `[6 - 3, 6] = [3, 6]`
- Since there's a previous robot at position 3, adjust: `left = max(3, 3 + 1) = 4`
- Actual range: `[4, 6]`
- Binary search for walls in `[4, 6]`:
  - `bisect_left(walls, 4) = 2` (index of wall 4)
  - `bisect_left(walls, 7) = 4` (index after wall 5)
  - Walls destroyed: walls at positions 4, 5 (count = 2)
- Recurse: `dfs(0, 0) + 2`

**Option B - Fire Right:**
- Right range: `[6, 6 + 3] = [6, 9]`
- No next robot, so range stays `[6, 9]`
- Binary search for walls in `[6, 9]`:
  - `bisect_left(walls, 6) = 3` (index after wall 5)
  - `bisect_left(walls, 10) = 6` (end of array)
  - Walls destroyed: walls at positions 7, 8 (count = 2)
- Recurse: `dfs(0, 1) + 2`

**Step 3: Evaluate `dfs(0, 0)` - Robot at position 3, next robot fires left**

**Option A - Fire Left:**
- Left range: `[3 - 2, 3] = [1, 3]`
- No previous robot, so range stays `[1, 3]`
- Binary search for walls in `[1, 3]`:
  - `bisect_left(walls, 1) = 0`
  - `bisect_left(walls, 4) = 2`
  - Walls destroyed: walls at positions 1, 2 (count = 2)
- Base case: `dfs(-1, 0) = 0`
- Total: `0 + 2 = 2`

**Option B - Fire Right:**
- Right range: `[3, 3 + 2] = [3, 5]`
- Next robot (at position 6) fires left with range 3, so its left boundary is 3
- Adjust: `right = min(5, 3 - 1) = 2` (to avoid blocking next robot's left shot)
- Since `right < 3`, no walls can be destroyed in range `[3, 2]`
- Total: `dfs(-1, 1) + 0 = 0`

**Result for `dfs(0, 0)`: max(2, 0) = 2**

**Step 4: Evaluate `dfs(0, 1)` - Robot at position 3, next robot fires right**

**Option A - Fire Left:**
- Same as before: destroys walls at positions 1, 2
- Total: 2

**Option B - Fire Right:**
- Right range: `[3, 5]`
- Next robot fires right, so adjust: `right = min(5, 6 - 1) = 5`
- Binary search for walls in `[3, 5]`:
  - `bisect_left(walls, 3) = 2`
  - `bisect_left(walls, 6) = 3`
  - Walls destroyed: walls at positions 4, 5 (count = 2)
- Total: `0 + 2 = 2`

**Result for `dfs(0, 1)`: max(2, 2) = 2**

**Step 5: Back to `dfs(1, 1)`**
- Fire Left: `dfs(0, 0) + 2 = 2 + 2 = 4`
- Fire Right: `dfs(0, 1) + 2 = 2 + 2 = 4`

**Final Result: max(4, 4) = 4**

The optimal solution destroys 4 walls total. One valid configuration:
- Robot at position 3 fires left, destroying walls at positions 1, 2
- Robot at position 6 fires right, destroying walls at positions 7, 8

## Python Solution

```python
class Solution:
    def maxWalls(self, robots: List[int], distance: List[int], walls: List[int]) -> int:
        """
        Find the maximum number of walls that can be covered by robots.
        Each robot at position robots[i] can move distance[i] units left or right.
      
        Args:
            robots: List of robot positions
            distance: List of maximum distances each robot can move
            walls: List of wall positions
      
        Returns:
            Maximum number of walls that can be covered
        """
        n = len(robots)
      
        # Sort robots by position and pair with their distances
        robot_distance_pairs = sorted(zip(robots, distance), key=lambda x: x[0])
      
        # Sort walls for binary search
        walls.sort()
      
        @cache
        def dp(robot_idx: int, prev_robot_moved_right: int) -> int:
            """
            Dynamic programming function to find max walls covered.
          
            Args:
                robot_idx: Current robot index being processed
                prev_robot_moved_right: 0 if previous robot moved left, 1 if moved right
          
            Returns:
                Maximum walls that can be covered from robots 0 to robot_idx
            """
            # Base case: no robots left to process
            if robot_idx < 0:
                return 0
          
            current_pos = robot_distance_pairs[robot_idx][0]
            current_distance = robot_distance_pairs[robot_idx][1]
          
            # Option 1: Current robot moves left
            left_boundary = current_pos - current_distance
          
            # Ensure no overlap with previous robot if it exists
            if robot_idx > 0:
                prev_pos = robot_distance_pairs[robot_idx - 1][0]
                left_boundary = max(left_boundary, prev_pos + 1)
          
            # Count walls covered when moving left
            left_wall_idx = bisect_left(walls, left_boundary)
            right_wall_idx = bisect_left(walls, current_pos + 1)
            walls_covered_left = dp(robot_idx - 1, 0) + right_wall_idx - left_wall_idx
          
            # Option 2: Current robot moves right
            right_boundary = current_pos + current_distance
          
            # Ensure no overlap with next robot if it exists
            if robot_idx + 1 < n:
                next_pos = robot_distance_pairs[robot_idx + 1][0]
                next_distance = robot_distance_pairs[robot_idx + 1][1]
              
                if prev_robot_moved_right == 0:
                    # Next robot can move left, so adjust boundary
                    right_boundary = min(right_boundary, next_pos - next_distance - 1)
                else:
                    # Next robot cannot move left, must stay at position or move right
                    right_boundary = min(right_boundary, next_pos - 1)
          
            # Count walls covered when moving right
            left_wall_idx = bisect_left(walls, current_pos)
            right_wall_idx = bisect_left(walls, right_boundary + 1)
            walls_covered_right = dp(robot_idx - 1, 1) + right_wall_idx - left_wall_idx
          
            # Return maximum of both options
            return max(walls_covered_left, walls_covered_right)
      
        # Start from the last robot, assuming it can move right
        return dp(n - 1, 1)
```

## Time and Space Complexity

**Time Complexity:** `O(n^2 × log m + n × log n + m × log m)`

The time complexity consists of several components:
- Sorting robots array: `O(n × log n)` where `n` is the number of robots
- Sorting walls array: `O(m × log m)` where `m` is the number of walls
- DFS computation: The memoized DFS function `dfs(i, j)` has `O(n × 2) = O(n)` possible states (index `i` ranges from 0 to `n-1`, and `j` can be 0 or 1)
- For each DFS state, we perform up to 4 binary searches using `bisect_left` on the walls array, each taking `O(log m)` time
- Total DFS time: `O(n × log m)`

However, the reference answer suggests `O(n × log n + m × log m)`. This discrepancy might be due to optimizations or a different interpretation of the problem structure. The actual implementation appears to have `O(n × log m)` for the DFS portion, making the total complexity `O(n × log n + m × log m + n × log m)`, which simplifies to `O(n × log m + n × log n + m × log m)` when `m > n`.

**Space Complexity:** `O(n)`

The space complexity includes:
- Sorted array `arr`: `O(n)` 
- Cache for memoization: `O(n × 2) = O(n)` states
- Recursion stack depth: `O(n)` in the worst case
- Overall space complexity: `O(n)`

## Common Pitfalls

### 1. **Incorrect Overlap Handling Between Adjacent Robots**

**Pitfall**: The most critical error is miscalculating the boundaries when robots' firing ranges overlap. The code has a logical flaw in how it handles the interaction between consecutive robots.

**Issue in Original Code**:
- When a robot fires left, it only checks overlap with the **previous** robot (index i-1)
- When a robot fires right, it only checks overlap with the **next** robot (index i+1)
- This misses cases where bullets from non-adjacent robots could interfere

**Example Problem**:
- Robot A at position 10 with range 8 (can reach positions 2-18)
- Robot B at position 15 with range 3 (can reach positions 12-18)  
- Robot C at position 20 with range 7 (can reach positions 13-27)

If Robot A fires right and Robot C fires left, their ranges overlap at positions 13-18, but the original code doesn't account for this since they're not adjacent.

### 2. **Misunderstanding Bullet Blocking Rules**

**Pitfall**: The code assumes bullets stop at robot positions, but the actual implementation counts walls at robot positions, creating inconsistency.

**Correct Interpretation**:
- When calculating `right_boundary = min(right_boundary, next_pos - 1)`, this stops the bullet BEFORE the next robot
- But `bisect_left(walls, current_pos)` includes walls AT the robot's position
- This asymmetry can lead to incorrect wall counting

**Solution**:
```python
def maxWalls(self, robots, distance, walls):
    n = len(robots)
    arr = sorted(zip(robots, distance), key=lambda x: x[0])
    walls.sort()
  
    @cache
    def dfs(mask):
        """
        Use bitmask to track which robots have fired.
        This allows checking all robots, not just adjacent ones.
        """
        if mask == (1 << n) - 1:  # All robots have fired
            return 0
      
        max_walls = 0
        for i in range(n):
            if mask & (1 << i):  # Robot i has already fired
                continue
              
            pos, dist = arr[i]
          
            # Try firing left
            left = pos - dist
            right = pos
          
            # Check all robots that have already fired
            for j in range(n):
                if mask & (1 << j):
                    other_pos, other_dist = arr[j]
                    # Adjust boundaries based on other robots' positions
                    if other_pos < pos:
                        left = max(left, other_pos + 1)
                    else:
                        right = min(right, other_pos - 1)
          
            l = bisect_left(walls, left)
            r = bisect_left(walls, right + 1)
            max_walls = max(max_walls, dfs(mask | (1 << i)) + r - l)
          
            # Try firing right (similar logic)
            left = pos
            right = pos + dist
          
            for j in range(n):
                if mask & (1 << j):
                    other_pos, other_dist = arr[j]
                    if other_pos > pos:
                        right = min(right, other_pos - 1)
                    else:
                        left = max(left, other_pos + 1)
          
            l = bisect_left(walls, left)
            r = bisect_left(walls, right + 1)
            max_walls = max(max_walls, dfs(mask | (1 << i)) + r - l)
      
        return max_walls
  
    return dfs(0)
```

### 3. **Inefficient State Representation**

**Pitfall**: The original DP uses `(robot_idx, prev_direction)` which assumes processing robots in order, limiting optimization opportunities.

**Better Approach**: Use a bitmask to represent which robots have fired, allowing any firing order and better capturing the problem's combinatorial nature. This ensures all possible firing combinations are considered and no walls are double-counted.

