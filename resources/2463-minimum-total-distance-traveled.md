# 2463. Minimum Total Distance Traveled

> Source: https://algo.monster/liteproblems/2463

## Problem Statement

You have robots and factories positioned along the X-axis. Each robot starts at a specific position given in the array `robot[i]`, and each factory is located at `factory[j][0]` with a repair capacity limit of `factory[j][1]` robots.

All robots are initially broken and need to be repaired. You can choose the initial movement direction (left or right) for each robot. When a robot reaches a factory that hasn't reached its repair limit, the factory repairs the robot and it stops moving.

The goal is to assign directions to all robots such that the total distance traveled by all robots is minimized.

Key points to understand:
- Each robot and factory has a unique position on the X-axis
- A robot can initially be at the same position as a factory
- All robots move at the same speed
- Robots don't collide - they pass through each other
- If a factory has reached its repair limit, robots pass through it without stopping
- The distance traveled by a robot moving from position `x` to position `y` is `|y - x|`

The solution uses [dynamic programming](/problems/dynamic_programming_intro) with memoization. After [sorting](/problems/sorting_summary) both robots and factories by position, it defines `dfs(i, j)` to find the minimum distance for assigning robots starting from index `i` to factories starting from index `j`.

For each factory `j`, there are two choices:
1. Skip this factory: `dfs(i, j) = dfs(i, j+1)`
2. Use this factory to repair `k` robots (where `k` ranges from 0 to the factory's limit): Calculate the total distance for these `k` robots to reach factory `j`, then add `dfs(i+k+1, j+1)` for the remaining robots and factories

## Intuition

The key insight is that once we sort both robots and factories by position, the problem becomes an assignment problem where we want to match robots to factories optimally.

Why [sorting](/problems/sorting_summary) helps: After sorting, we can observe that it's never optimal for robot assignments to "cross over". If robot A is to the left of robot B, and factory X is to the left of factory Y, it wouldn't make sense for robot A to go to factory Y while robot B goes to factory X - they would travel unnecessary extra distance.

This observation leads us to think about the problem sequentially: for each group of robots, we decide which factory they should go to. This naturally suggests a [dynamic programming](/problems/dynamic_programming_intro) approach where we process robots and factories in order.

The state we need to track is: "Starting from robot `i` and factory `j`, what's the minimum distance to repair all remaining robots?" This is our `dfs(i, j)` function.

For each factory, we face a decision: how many robots should this factory repair? Since factories have limits, we can't send all robots to one factory. We need to try different possibilities:
- Don't use this factory at all (skip to the next factory)
- Send 1 robot to this factory
- Send 2 robots to this factory
- ... up to the factory's limit

The greedy insight within this DP approach: if we decide factory `j` will repair `k` robots, it should repair the next `k` consecutive robots starting from position `i`. Why? Because these are the closest unassigned robots to this factory (remember, we sorted everything).

This combination of [sorting](/problems/sorting_summary) + [dynamic programming](/problems/dynamic_programming_intro) with memoization allows us to efficiently explore all valid assignments while avoiding redundant calculations, ultimately finding the optimal solution where the total distance is minimized.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) and [Sorting](/problems/sorting_summary) patterns.**

## Solution Approach

The solution implements a top-down [dynamic programming](/problems/dynamic_programming_intro) approach with memoization. Here's how the implementation works:

**Step 1: [Sorting](/problems/sorting_summary)**
```python
robot.sort()
factory.sort()
```
First, we sort both the robots and factories arrays in ascending order by position. This ensures we can process them sequentially and avoid crossing assignments.

**Step 2: Define the DP State**
The function `dfs(i, j)` represents the minimum total distance needed to repair all robots starting from index `i` using factories starting from index `j`.

**Step 3: Base Cases**
```python
if i == len(robot):
    return 0
if j == len(factory):
    return inf
```
- If `i == len(robot)`, all robots have been assigned, so the distance is 0
- If `j == len(factory)`, we've run out of factories but still have robots to repair, which is impossible, so we return infinity

**Step 4: Recursive Transitions**
For each state `(i, j)`, we have two main choices:

1. **Skip the current factory**: 
   ```python
   ans = dfs(i, j + 1)
   ```
   Move to the next factory without using the current one.

2. **Use the current factory for k robots**:
   ```python
   t = 0
   for k in range(factory[j][1]):
       if i + k == len(robot):
           break
       t += abs(robot[i + k] - factory[j][0])
       ans = min(ans, t + dfs(i + k + 1, j + 1))
   ```
   - Try assigning 1, 2, ..., up to `limit` robots to factory `j`
   - For each choice of `k` robots, calculate the cumulative distance `t`
   - The distance for robot at index `i+k` to reach factory `j` is `|robot[i+k] - factory[j][0]|`
   - After assigning `k+1` robots to this factory, recursively solve for the remaining robots starting from `i+k+1` and the next factory `j+1`

**Step 5: Memoization**
The `@cache` decorator automatically memoizes the results of `dfs(i, j)`, preventing redundant calculations for the same state.

**Step 6: Return the Result**
```python
ans = dfs(0, 0)
dfs.cache_clear()
return ans
```
Start the recursion from the first robot and first factory, clear the cache to free memory, and return the minimum total distance.

The time complexity is `O(m * n * limit)` where `m` is the number of robots, `n` is the number of factories, and `limit` is the maximum capacity of any factory. The space complexity is `O(m * n)` for the memoization cache.

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

**Example:**
- `robot = [1, 6]` (2 robots at positions 1 and 6)
- `factory = [[2, 1], [7, 1]]` (Factory at position 2 with capacity 1, Factory at position 7 with capacity 1)

**Step 1: Sort arrays**
- `robot = [1, 6]` (already sorted)
- `factory = [[2, 1], [7, 1]]` (already sorted)

**Step 2: Start DFS from dfs(0, 0)**

We need to assign robots starting from index 0 to factories starting from index 0.

**At dfs(0, 0):** (Robot index 0, Factory index 0)
- Current robot at position 1, current factory at position 2 with capacity 1
- **Option 1:** Skip factory 0 → `dfs(0, 1)`
- **Option 2:** Send 1 robot to factory 0
  - Distance = |1 - 2| = 1
  - Continue with `dfs(1, 1)` 
  - Total for this option: 1 + `dfs(1, 1)`

Let's explore Option 2 first:

**At dfs(1, 1):** (Robot index 1, Factory index 1)
- Current robot at position 6, current factory at position 7 with capacity 1
- **Option 1:** Skip factory 1 → `dfs(1, 2)` → returns ∞ (no more factories)
- **Option 2:** Send 1 robot to factory 1
  - Distance = |6 - 7| = 1
  - Continue with `dfs(2, 2)` → returns 0 (all robots assigned)
  - Total for this option: 1 + 0 = 1

So `dfs(1, 1) = 1`

Back to **dfs(0, 0):**
- Option 2 gives us: 1 + 1 = 2

Now let's explore Option 1:

**At dfs(0, 1):** (Robot index 0, Factory index 1)
- Current robot at position 1, current factory at position 7 with capacity 1
- **Option 1:** Skip factory 1 → `dfs(0, 2)` → returns ∞ (no more factories)
- **Option 2:** Send 1 robot to factory 1
  - Distance = |1 - 7| = 6
  - Continue with `dfs(1, 2)` → returns ∞ (robot at index 1 can't be assigned)
  - Total: ∞

So `dfs(0, 1) = ∞`

**Final Result:**
- `dfs(0, 0) = min(∞, 2) = 2`

The optimal assignment is:
- Robot at position 1 → Factory at position 2 (distance = 1)
- Robot at position 6 → Factory at position 7 (distance = 1)
- Total distance = 2

This example shows how the algorithm explores different assignments, using memoization to avoid recalculating states, and ultimately finds the minimum total distance by trying all valid combinations of robot-to-factory assignments.

## Python Solution

```python
class Solution:
    def minimumTotalDistance(self, robot: List[int], factory: List[List[int]]) -> int:
        """
        Calculate the minimum total distance to repair all robots using available factories.
      
        Args:
            robot: List of robot positions
            factory: List of [position, capacity] for each factory
          
        Returns:
            Minimum total distance for all robots to reach factories
        """
      
        @cache
        def dp(robot_idx: int, factory_idx: int) -> int:
            """
            Dynamic programming function to find minimum distance.
          
            Args:
                robot_idx: Current index of robot being considered
                factory_idx: Current index of factory being considered
              
            Returns:
                Minimum distance to repair robots from robot_idx onwards
                using factories from factory_idx onwards
            """
            # Base case: All robots have been assigned
            if robot_idx == len(robot):
                return 0
          
            # Base case: No more factories available but robots remain
            if factory_idx == len(factory):
                return float('inf')
          
            # Option 1: Skip current factory
            min_distance = dp(robot_idx, factory_idx + 1)
          
            # Option 2: Use current factory for some robots
            cumulative_distance = 0
            factory_position = factory[factory_idx][0]
            factory_capacity = factory[factory_idx][1]
          
            # Try assigning 1 to capacity robots to current factory
            for num_robots in range(factory_capacity):
                # Check if we have enough robots remaining
                if robot_idx + num_robots >= len(robot):
                    break
              
                # Add distance for current robot to current factory
                cumulative_distance += abs(robot[robot_idx + num_robots] - factory_position)
              
                # Calculate total distance if we assign (num_robots + 1) robots to this factory
                min_distance = min(
                    min_distance,
                    cumulative_distance + dp(robot_idx + num_robots + 1, factory_idx + 1)
                )
          
            return min_distance
      
        # Sort both lists to enable optimal assignment
        robot.sort()
        factory.sort()
      
        # Calculate the minimum total distance
        result = dp(0, 0)
      
        # Clear the cache to free memory
        dp.cache_clear()
      
        return result
```

## Time and Space Complexity

**Time Complexity:** `O(m^2 × n)`

The time complexity analysis breaks down as follows:
- The recursive function `dfs(i, j)` has states defined by parameters `i` (robot index) and `j` (factory index), creating `m × n` unique states where `m = len(robot)` and `n = len(factory)`.
- Due to memoization with `@cache`, each state is computed only once.
- For each state `(i, j)`, the function iterates through up to `factory[j][1]` positions to assign robots to the current factory. In the worst case, this can be up to `m` robots (when a factory has capacity equal to or greater than the number of robots).
- Within this loop, each iteration performs `O(1)` operations (calculating distance and making recursive calls).
- Therefore, the work done per state is `O(m)`, and with `m × n` states, the total time complexity is `O(m × n × m) = O(m^2 × n)`.

**Space Complexity:** `O(m × n)`

The space complexity consists of:
- The memoization cache stores up to `m × n` states (all possible combinations of robot and factory indices).
- The recursion depth is at most `O(m + n)` in the worst case (processing all robots and factories).
- Since `m × n` dominates `m + n`, the overall space complexity is `O(m × n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Forgetting to Sort the Arrays**
One of the most common mistakes is forgetting to sort both the `robot` and `factory` arrays before applying dynamic programming. Without sorting, the greedy assignment strategy breaks down because we might assign distant robots to nearby factories, leading to suboptimal solutions.

**Why it matters:** The DP solution relies on the fact that once sorted, we can assign robots to factories in order without worrying about crossing assignments (where a robot further left goes to a factory further right while a robot further right goes to a factory further left).

**Solution:**
```python
# Always sort both arrays at the beginning
robot.sort()
factory.sort()
```

### 2. **Off-by-One Error in the Loop**
When iterating through the number of robots to assign to a factory, it's easy to make an off-by-one error:

**Incorrect:**
```python
for num_robots in range(factory_capacity + 1):  # Wrong! This tries to assign capacity+1 robots
    if robot_idx + num_robots >= len(robot):
        break
```

**Correct:**
```python
for num_robots in range(factory_capacity):  # Assigns 0 to capacity-1 robots (total of capacity iterations)
    if robot_idx + num_robots >= len(robot):
        break
    # When using num_robots as index offset, we're actually assigning num_robots+1 robots
    cumulative_distance += abs(robot[robot_idx + num_robots] - factory_position)
    min_distance = min(min_distance, 
                      cumulative_distance + dp(robot_idx + num_robots + 1, factory_idx + 1))
```

### 3. **Not Handling the "Assign Zero Robots" Case**
Some implementations forget that it's valid to not assign any robots to a factory (essentially skipping it). This is already handled by having two separate options in the code, but merging them incorrectly can cause issues:

**Problematic approach:**
```python
# Starting from 1 instead of considering the skip option separately
for num_robots in range(1, factory_capacity + 1):  # Misses the case of assigning 0 robots
    ...
```

**Correct approach:**
```python
# Option 1: Skip factory (assign 0 robots)
min_distance = dp(robot_idx, factory_idx + 1)

# Option 2: Assign 1 or more robots
for num_robots in range(factory_capacity):
    ...
```

### 4. **Integer Overflow with Large Values**
When no valid assignment exists, returning a very large number is necessary. Using `sys.maxsize` or a large integer might cause overflow issues in some languages or when adding distances.

**Solution:**
```python
# Use float('inf') for impossible cases
if factory_idx == len(factory):
    return float('inf')
```

### 5. **Forgetting to Clear the Cache**
While not affecting correctness, forgetting to clear the cache can cause memory issues in systems where the same Solution instance is reused for multiple test cases:

**Solution:**
```python
result = dp(0, 0)
dp.cache_clear()  # Important for memory management
return result
```

