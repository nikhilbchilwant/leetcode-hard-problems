# 2751. Robot Collisions

> Source: https://algo.monster/liteproblems/2751

## Problem Statement

You have `n` robots on a line, where each robot has three properties:
- **Position**: where the robot is located on the line
- **Health**: an integer value representing the robot's health
- **Direction**: either 'L' (moving left) or 'R' (moving right)

The robots are given to you through three arrays:
- `positions[i]` - the position of robot `i+1` (robots are 1-indexed but arrays are 0-indexed)
- `healths[i]` - the health of robot `i+1`
- `directions[i]` - the direction of robot `i+1` ('L' or 'R')

All positions are unique, and the robots all start moving simultaneously at the same speed.

**Collision Rules:**
- When two robots meet at the same position (one moving right, one moving left), they collide
- When robots collide:
  - If they have different health values: the robot with lower health is removed, and the surviving robot's health decreases by 1
  - If they have the same health: both robots are removed
- Surviving robots continue moving in their original direction

**Your Task:**
Return an array containing the final health values of all surviving robots, **in the same order as they were given in the input**. If a robot doesn't survive, it shouldn't appear in the result. If no robots survive, return an empty array.

**Important Note:** The positions array may not be sorted, meaning robots aren't necessarily given in left-to-right order by position.

**Example Walkthrough:**
If you have robots at positions `[5,4,3]` with healths `[2,17,9]` and directions `"RLR"`:
- Robot at position 3 moves right, robot at position 4 moves left - they collide
- Robot at position 4 (health 17) survives with health 16, robot at position 3 is removed
- Robot at position 5 moves right, robot at position 4 moves left - they collide  
- Robot at position 4 (health 16) survives with health 15, robot at position 5 is removed
- Final result: `[0,15,0]` → `[15]` (only the second robot survives)

## Intuition

The key insight is that we need to simulate the collisions as they would happen on a number line. Since all robots move at the same speed, we can think about this problem in terms of which robots will meet each other based on their positions and directions.

First, we need to process robots from left to right by position (not by their input order), because collisions happen based on spatial location, not input order. This is why we sort the indices by position.

The critical observation is that collisions only happen between robots moving in opposite directions - specifically, when a robot moving right ('R') meets a robot moving left ('L'). If we process robots from left to right:
- Robots moving right ('R') are potential collision candidates for future robots moving left
- Robots moving left ('L') will collide with any robots moving right that are to their left

This naturally suggests using a **[stack](/problems/stack_intro)** to track robots moving right. As we process each robot:
- If it's moving right ('R'), we add it to the stack (it might collide with future left-moving robots)
- If it's moving left ('L'), it needs to resolve collisions with all right-moving robots in its path (those in the stack)

When a left-moving robot encounters right-moving robots (in the [stack](/problems/stack_intro)), we simulate collisions one by one:
- Compare healths and update accordingly
- The robot with higher health survives (with health reduced by 1)
- If healths are equal, both are eliminated
- Continue until either the left-moving robot is eliminated or there are no more right-moving robots to collide with

After processing all robots, any robot with health > 0 has survived. We then need to return these survivors in their original input order (not the sorted order we used for processing), which is why we track indices and filter the original health array at the end.

The beauty of this approach is that the [stack](/problems/stack_intro) naturally handles the collision chain reactions - when a strong left-moving robot defeats multiple right-moving robots in sequence, we simply keep popping from the stack until the collision resolution is complete.

**Learn more about [Stack](/problems/stack_intro) and [Sorting](/problems/sorting_summary) patterns.**

## Solution Approach

Let's walk through the implementation step by step:

**1. Initialize Data Structures:**
```python
n = len(positions)
indices = list(range(n))  # [0, 1, 2, ..., n-1]
[stack](/problems/stack_intro) = []
```
We create an `indices` list to track the original order of robots while allowing us to sort by position. The `stack` will store indices of robots moving right.

**2. Sort by Position:**
```python
indices.sort(key=lambda i: positions[i])
```
We sort the indices based on their corresponding positions. This ensures we process robots from left to right on the number line, which is crucial for correctly simulating collisions.

**3. Process Each Robot:**
```python
for currentIndex in indices:
    if directions[currentIndex] == "R":
        [stack](/problems/stack_intro).append(currentIndex)
```
For each robot (in position order):
- If it's moving right ('R'), add its index to the stack. These robots are candidates for future collisions.

**4. Handle Left-Moving Robots:**
```python
else:  # directions[currentIndex] == "L"
    while [stack](/problems/stack_intro) and healths[currentIndex] > 0:
        topIndex = stack.pop()
```
When we encounter a left-moving robot, it needs to collide with right-moving robots (stored in the stack). We continue collisions while:
- There are right-moving robots in the stack
- The current left-moving robot is still alive (`healths[currentIndex] > 0`)

**5. Resolve Individual Collisions:**
```python
if healths[topIndex] > healths[currentIndex]:
    healths[topIndex] -= 1
    healths[currentIndex] = 0
    [stack](/problems/stack_intro).append(topIndex)  # Right robot survives, goes back to stack
elif healths[topIndex] < healths[currentIndex]:
    healths[currentIndex] -= 1
    healths[topIndex] = 0
    # Left robot survives, continues to next collision
else:  # Equal health
    healths[currentIndex] = 0
    healths[topIndex] = 0
    # Both robots are eliminated
```
For each collision:
- Compare healths of the colliding robots
- Update healths based on collision rules (survivor loses 1 health, loser gets 0)
- If the right-moving robot survives, push it back to the stack
- If the left-moving robot survives, continue the while loop for more collisions

**6. Collect Survivors:**
```python
result = [health for health in healths if health > 0]
return result
```
After all collisions are resolved, we filter the original `healths` array to get only surviving robots (health > 0). Since we modified the `healths` array in place and iterate through it in original order, the survivors are automatically in the correct order as required.

**Time Complexity:** `O(n log n)` for [sorting](/problems/sorting_summary), plus `O(n)` for processing, giving `O(n log n)` overall.
**Space Complexity:** `O(n)` for the indices list and [stack](/problems/stack_intro).

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

**Input:**
- positions = [3, 5, 2, 6]
- healths = [10, 10, 15, 10]
- directions = "RLRL"

**Step 1: Setup and Sort**
We create indices [0, 1, 2, 3] and sort them by position:
- Sorted indices: [2, 0, 1, 3] (corresponding to positions [2, 3, 5, 6])

**Step 2: Process robots from left to right by position**

**Processing robot at index 2 (position 2, health 15, direction 'R'):**
- It's moving right, so add index 2 to stack
- Stack: [2]
- Current healths: [10, 10, 15, 10]

**Processing robot at index 0 (position 3, health 10, direction 'R'):**
- It's moving right, so add index 0 to stack
- Stack: [2, 0]
- Current healths: [10, 10, 15, 10]

**Processing robot at index 1 (position 5, health 10, direction 'L'):**
- It's moving left, so it collides with right-moving robots in the stack
- Pop index 0 from stack (robot at position 3, health 10)
- They have equal health (10 vs 10), so both are eliminated
- healths[0] = 0, healths[1] = 0
- Stack: [2]
- Current healths: [0, 0, 15, 10]

**Processing robot at index 3 (position 6, health 10, direction 'L'):**
- It's moving left, so it collides with right-moving robots in the stack
- Pop index 2 from stack (robot at position 2, health 15)
- Robot 2 has higher health (15 vs 10):
  - healths[2] = 14 (survives with reduced health)
  - healths[3] = 0 (eliminated)
  - Push index 2 back to stack
- Stack: [2]
- Current healths: [0, 0, 14, 0]

**Step 3: Collect survivors**
Filter the healths array for values > 0:
- healths[0] = 0 (eliminated)
- healths[1] = 0 (eliminated)
- healths[2] = 14 (survived)
- healths[3] = 0 (eliminated)

**Final Result: [14]**

The only survivor is the robot originally at index 2 (position 2), which survived with health 14 after defeating the robot at position 6.

## Python Solution

```python
class Solution:
    def survivedRobotsHealths(
        self, positions: List[int], healths: List[int], directions: str
    ) -> List[int]:
        """
        Simulates robot collisions on a line and returns surviving robots' healths.
      
        Args:
            positions: List of initial positions of robots
            healths: List of initial health values of robots
            directions: String where each character is 'L' or 'R' indicating direction
          
        Returns:
            List of health values for surviving robots in their original order
        """
        n = len(positions)
        # Create indices to track original robot positions
        indices = list(range(n))
        # Stack to store indices of robots moving right
        stack = []
      
        # Sort indices by robot positions to process collisions from left to right
        indices.sort(key=lambda i: positions[i])
      
        # Process each robot in position order
        for current_index in indices:
            if directions[current_index] == "R":
                # Robot moving right: add to stack for potential future collisions
                stack.append(current_index)
            else:
                # Robot moving left: check for collisions with robots moving right
                while stack and healths[current_index] > 0:
                    # Get the rightmost robot moving right
                    top_index = stack.pop()
                  
                    if healths[top_index] > healths[current_index]:
                        # Right-moving robot wins: reduce its health by 1
                        healths[top_index] -= 1
                        healths[current_index] = 0
                        # Put winner back on stack
                        stack.append(top_index)
                    elif healths[top_index] < healths[current_index]:
                        # Left-moving robot wins: reduce its health by 1
                        healths[current_index] -= 1
                        healths[top_index] = 0
                        # Continue checking for more collisions
                    else:
                        # Both robots have equal health: both are destroyed
                        healths[current_index] = 0
                        healths[top_index] = 0
      
        # Return health values of surviving robots (health > 0) in original order
        result = [health for health in healths if health > 0]
        return result
```

## Time and Space Complexity

**Time Complexity: `O(n log n)`**

The time complexity is dominated by the sorting operation:
- Creating the indices list takes `O(n)` time
- Sorting the indices based on positions takes `O(n log n)` time
- The main loop iterates through all `n` indices once: `O(n)`
  - For each robot moving left, the while loop processes collisions with robots moving right
  - Each robot can be pushed to and popped from the stack at most once
  - Therefore, the total work done by all while loop iterations across all indices is `O(n)`
- Creating the final result list by filtering healths takes `O(n)` time

Overall: `O(n) + O(n log n) + O(n) + O(n) = O(n log n)`

**Space Complexity: `O(n)`**

The space complexity consists of:
- `indices` list storing `n` indices: `O(n)`
- `stack` can contain at most `n` elements (all robots moving right): `O(n)`
- `result` list can contain at most `n` elements: `O(n)`

Overall: `O(n) + O(n) + O(n) = O(n)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Not Preserving Original Order in the Result**

**The Pitfall:**
A common mistake is returning the surviving robots in the order they appear on the line (sorted by position) rather than in their original input order. For example:

```python
# WRONG: Returns survivors in position-sorted order
def survivedRobotsHealths(positions, healths, directions):
    robots = sorted(zip(positions, healths, directions))
    # ... process collisions ...
    return [h for p, h, d in robots if h > 0]  # Wrong order!
```

This would fail for input like `positions=[5,4,3]`, `healths=[2,17,9]`, `directions="RLR"` where the survivor at position 4 should be returned as `[17]` (second in original order), not based on its position.

**The Solution:**
Track the original indices throughout the process and modify the health array in-place:

```python
# CORRECT: Maintains original order
indices = list(range(n))
indices.sort(key=lambda i: positions[i])  # Sort indices, not robots
# Process using indices...
healths[index] = new_health  # Modify original array
return [h for h in healths if h > 0]  # Preserves original order
```

### 2. **Incorrectly Handling the Collision Stack**

**The Pitfall:**
Forgetting to put the right-moving robot back on the stack when it survives a collision:

```python
# WRONG: Doesn't return survivor to stack
if healths[top_index] > healths[current_index]:
    healths[top_index] -= 1
    healths[current_index] = 0
    # Missing: stack.append(top_index)
```

This causes the surviving right-moving robot to disappear from future collision checks, leading to incorrect results.

**The Solution:**
Always return the surviving right-moving robot to the stack:

```python
# CORRECT: Returns survivor to stack
if healths[top_index] > healths[current_index]:
    healths[top_index] -= 1
    healths[current_index] = 0
    stack.append(top_index)  # Critical line!
```

### 3. **Processing Robots in Input Order Instead of Position Order**

**The Pitfall:**
Processing robots in the order they appear in the input arrays rather than by their positions on the line:

```python
# WRONG: Processes in input order
for i in range(n):
    if directions[i] == "R":
        stack.append(i)
    else:
        # Process collisions...
```

This fails because collisions happen based on physical position, not input order. A robot at position 10 moving left should collide with a robot at position 8 moving right, regardless of their order in the input.

**The Solution:**
Sort the processing order by position while maintaining index references:

```python
# CORRECT: Process by position order
indices = list(range(n))
indices.sort(key=lambda i: positions[i])
for current_index in indices:
    # Now processing in correct spatial order
```

### 4. **Not Handling Edge Cases Properly**

**The Pitfall:**
Not considering cases where:
- All robots move in the same direction (no collisions)
- No robots survive (should return empty array, not array of zeros)
- Single robot (no possible collisions)

**The Solution:**
The algorithm naturally handles these cases if implemented correctly:
- Same direction: Stack either stays empty (all 'L') or accumulates all robots (all 'R')
- No survivors: The list comprehension `[h for h in healths if h > 0]` returns empty array
- Single robot: Loop executes once, no collisions possible

The key is using `if health > 0` when collecting results rather than checking for specific values.

