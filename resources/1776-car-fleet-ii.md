# 1776. Car Fleet II

> Source: https://algo.monster/liteproblems/1776

## Problem Statement

You have `n` cars driving in the same direction on a single-lane road. Each car has a starting position and speed given in the array `cars`, where `cars[i] = [position_i, speed_i]`:
- `position_i` is the distance (in meters) of the `i`-th car from the start of the road
- `speed_i` is the speed (in meters per second) of the `i`-th car
- Cars are ordered by position, so `position_i < position_{i+1}`

When a faster car catches up to a slower car ahead of it, they collide and form a fleet. After collision:
- The cars move together as a single unit
- The fleet moves at the speed of the slower car

Your task is to determine when each car will collide with the car directly in front of it. Return an array `answer` where `answer[i]` represents:
- The time (in seconds) when car `i` collides with car `i+1`
- `-1` if car `i` never collides with the next car

The solution uses a monotonic [stack](/problems/stack_intro) approach, processing cars from right to left. For each car, it checks if and when it will catch up to cars ahead of it, considering that some cars ahead might have already collided with others. The stack maintains cars that could potentially be collision targets, removing those that would be "bypassed" due to earlier collisions.

The collision time between car `i` and car `j` (where `j` is ahead) is calculated as: `t = (position_j - position_i) / (speed_i - speed_j)`, which only applies when car `i` is faster than car `j`.

## Intuition

The key insight is to process the cars from right to left (back to front). Why? Because a car can only collide with cars ahead of it, never behind. When we process from the back, we already know the collision information for all cars ahead, which helps us make decisions for the current car.

Consider what happens when car `i` is faster than car `j` ahead of it. Car `i` will eventually catch up to car `j` at time `t = (position_j - position_i) / (speed_i - speed_j)`. But there's a catch - car `j` might collide with another car before car `i` reaches it. If car `j` collides at time `ans[j]` and this happens before car `i` would reach `j`, then car `i` won't actually collide with `j` directly.

This leads us to think about which cars are "reachable" for collision. If car `i` would reach car `j` after `j` has already collided with someone else, we need to look further ahead. The car that `j` collided with might be our actual collision target, or perhaps an even further car.

A monotonic [stack](/problems/stack_intro) naturally handles this scenario. As we process each car from right to left:
1. We maintain a stack of potential collision targets
2. For the current car, we check the top of the stack (the nearest car ahead)
3. If the current car is slower or same speed, it will never catch up - we add it to the stack for future cars to consider
4. If the current car is faster, we calculate when it would collide
5. If this collision happens after the target car has already collided elsewhere, we pop that target from the stack and check the next one
6. We continue popping until we find a valid collision or run out of targets

The [stack](/problems/stack_intro) effectively maintains a "chain" of cars where each subsequent car either doesn't collide or collides before being reached by cars behind it. This ensures we always find the correct first collision for each car.

**Learn more about [Stack](/problems/stack_intro), [Math](/problems/math-basics), [Monotonic Stack](/problems/mono_stack_intro) and [Heap (Priority Queue)](/problems/heap_intro) patterns.**

## Solution Approach

The solution uses a **monotonic [stack](/problems/stack_intro)** to efficiently track potential collision targets while processing cars from right to left.

**Data Structure:**
- `stk`: A [stack](/problems/stack_intro) storing indices of cars that could be collision targets
- `ans`: Result array initialized with `-1` for all cars

**Algorithm Steps:**

1. **Iterate from right to left** (`i` from `n-1` to `0`):
   - Start from the rightmost car since it has no car ahead to collide with
   - Each car only needs to consider cars ahead of it

2. **For each car `i`, check potential collisions:**
   ```python
   while stk:
       j = stk[-1]  # Get the nearest car ahead
   ```
 
3. **Determine if collision is possible:**
   - Check if car `i` is faster: `cars[i][1] > cars[j][1]`
   - If not faster, car `i` will never catch up, so pop car `j` from [stack](/problems/stack_intro) and check next

4. **Calculate collision time if car `i` is faster:**
   ```python
   t = (cars[j][0] - cars[i][0]) / (cars[i][1] - cars[j][1])
   ```
   This formula comes from solving: `position_i + speed_i * t = position_j + speed_j * t`

5. **Validate the collision:**
   - Check if `ans[j] == -1` (car `j` doesn't collide with anyone)
   - OR check if `t <= ans[j]` (car `i` reaches `j` before `j` collides with another car)
   - If valid, set `ans[i] = t` and break
   - If invalid, pop car `j` from [stack](/problems/stack_intro) (it's unreachable) and continue checking

6. **Add current car to [stack](/problems/stack_intro):**
   ```python
   stk.append(i)
   ```
   After processing, add car `i` to the stack as a potential target for cars behind it

**Why this works:**
- The [stack](/problems/stack_intro) maintains cars in increasing order of "reachability"
- When we pop a car from the stack, it means it's "blocked" by its own collision and won't be the direct collision target
- The first valid collision we find is guaranteed to be the actual collision for car `i`
- Cars that never collide remain `-1` in the answer array

**Time Complexity:** `O(n)` - each car is pushed and popped from the [stack](/problems/stack_intro) at most once
**Space Complexity:** `O(n)` - for the stack and answer array

## Example Walkthrough

Let's walk through a small example with 4 cars:
```
cars = [[0, 2], [4, 1], [8, 3], [10, 1]]
```

**Visual representation:**
```
Position: 0    4    8    10
Speed:    2    1    3    1
Car:      0    1    2    3
```

**Processing from right to left:**

**Step 1: Process car 3 (position=10, speed=1)**
- Stack is empty, no car ahead
- `ans[3] = -1`
- Push 3 to stack: `stk = [3]`

**Step 2: Process car 2 (position=8, speed=3)**
- Stack has car 3 (position=10, speed=1)
- Car 2 is faster (3 > 1), can catch up
- Collision time: `t = (10-8)/(3-1) = 1 second`
- Car 3 doesn't collide with anyone (`ans[3] = -1`), so this collision is valid
- `ans[2] = 1.0`
- Push 2 to stack: `stk = [3, 2]`

**Step 3: Process car 1 (position=4, speed=1)**
- Stack top is car 2 (position=8, speed=3)
- Car 1 is slower (1 < 3), will never catch up
- Pop car 2: `stk = [3]`
- Check car 3 (position=10, speed=1)
- Same speed (1 = 1), will never catch up
- Pop car 3: `stk = []`
- No more cars to check
- `ans[1] = -1`
- Push 1 to stack: `stk = [1]`

**Step 4: Process car 0 (position=0, speed=2)**
- Stack has car 1 (position=4, speed=1)
- Car 0 is faster (2 > 1), can catch up
- Collision time: `t = (4-0)/(2-1) = 4 seconds`
- Car 1 doesn't collide with anyone (`ans[1] = -1`), so this collision is valid
- `ans[0] = 4.0`
- Push 0 to stack: `stk = [1, 0]`

**Final answer:** `[4.0, -1, 1.0, -1]`

**Interpretation:**
- Car 0 collides with car 1 after 4 seconds
- Car 1 never collides (moves at same speed as car 3, slower than car 2)
- Car 2 collides with car 3 after 1 second
- Car 3 has no car ahead to collide with

The key insight demonstrated here is how car 1, despite being between cars 0 and 2, never collides with anyone because it's too slow to catch car 3 and car 2 (which is faster) collides with car 3 first.

## Python Solution

```python
class Solution:
    def getCollisionTimes(self, cars: List[List[int]]) -> List[float]:
        # Stack to maintain cars that could potentially collide with current car
        stack = []
        n = len(cars)
        # Initialize result array with -1 (no collision)
        result = [-1] * n
      
        # Process cars from right to left (last car to first car)
        for i in range(n - 1, -1, -1):
            # Check potential collisions with cars ahead (in the stack)
            while stack:
                j = stack[-1]
              
                # Current car is faster than car j ahead
                if cars[i][1] > cars[j][1]:
                    # Calculate collision time between car i and car j
                    collision_time = (cars[j][0] - cars[i][0]) / (cars[i][1] - cars[j][1])
                  
                    # Valid collision if car j doesn't collide before or collides after this time
                    if result[j] == -1 or collision_time <= result[j]:
                        result[i] = collision_time
                        break
              
                # Remove car j from stack as it won't collide with cars before i
                stack.pop()
          
            # Add current car to stack for processing previous cars
            stack.append(i)
      
        return result
```

## Time and Space Complexity

**Time Complexity: `O(n)`**

The algorithm iterates through all `n` cars once from right to left. Although there's a nested while loop that may pop elements from the stack, each car index is pushed onto the stack exactly once and popped at most once throughout the entire execution. This means the total number of operations across all iterations is bounded by `2n` (each element can be pushed once and popped once), resulting in an amortized `O(1)` time per car and `O(n)` overall time complexity.

**Space Complexity: `O(n)`**

The algorithm uses:
- A stack `stk` that can contain at most `n` elements (in the worst case, all car indices could be on the stack simultaneously)
- An answer array `ans` of size `n` to store the collision times

Both data structures require `O(n)` space, so the total space complexity is `O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Collision Logic - Not Removing Unreachable Cars**

**The Pitfall:**
A common mistake is to check only if the current car is faster than the car ahead, without considering whether the car ahead might collide with another car before being reached. This leads to incorrect collision times.

**Incorrect Implementation:**
```python
# WRONG: Only checks speed difference
for i in range(n - 1, -1, -1):
    while stack:
        j = stack[-1]
        if cars[i][1] > cars[j][1]:
            collision_time = (cars[j][0] - cars[i][0]) / (cars[i][1] - cars[j][1])
            result[i] = collision_time
            break  # Wrong! Doesn't validate if this collision actually happens
        stack.pop()
    stack.append(i)
```

**The Problem:**
Car `j` might collide with another car at time `t1`, but car `i` might calculate it reaches car `j` at time `t2 > t1`. In reality, car `i` can't collide with car `j` after `j` has already collided with another car.

**Correct Solution:**
```python
# Check if the collision happens before car j collides with another car
if result[j] == -1 or collision_time <= result[j]:
    result[i] = collision_time
    break
else:
    # Car j will collide before car i reaches it, so remove it
    stack.pop()
```

### 2. **Division by Zero - Cars with Same Speed**

**The Pitfall:**
When two cars have the same speed, the collision time formula divides by zero:
```python
collision_time = (cars[j][0] - cars[i][0]) / (cars[i][1] - cars[j][1])
# If cars[i][1] == cars[j][1], this causes division by zero!
```

**Correct Solution:**
Always check that the current car is strictly faster before calculating collision time:
```python
if cars[i][1] > cars[j][1]:  # Ensures denominator is positive
    collision_time = (cars[j][0] - cars[i][0]) / (cars[i][1] - cars[j][1])
```

### 3. **Stack Management - Forgetting to Pop Invalid Targets**

**The Pitfall:**
Not removing cars from the stack when they're unreachable can cause the algorithm to get stuck or produce wrong results.

**Incorrect Pattern:**
```python
# WRONG: Breaks without popping unreachable cars
while stack:
    j = stack[-1]
    if cars[i][1] <= cars[j][1]:
        break  # Wrong! Should pop and continue
```

**Correct Solution:**
```python
while stack:
    j = stack[-1]
    if cars[i][1] <= cars[j][1]:
        stack.pop()  # Remove car j as it's not reachable
        continue     # Check next car in stack
```

### 4. **Processing Order - Iterating Left to Right**

**The Pitfall:**
Processing cars from left to right makes it impossible to know future collisions, leading to complex dependency chains.

**Why Right to Left Works:**
- The rightmost car has no car ahead, so its collision time is definitively `-1`
- Each car only needs information about cars ahead (already processed)
- This creates a clean dependency order without circular references

**Example to Illustrate:**
```python
# Cars: [[1,2], [3,1], [5,3]]
# Processing right to left:
# - Car at position 5: no collision (-1)
# - Car at position 3: slower than car ahead, no collision (-1)  
# - Car at position 1: catches car at position 3 at time 1.0
```

