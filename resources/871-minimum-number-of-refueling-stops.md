# 871. Minimum Number of Refueling Stops

> Source: https://algo.monster/liteproblems/871

## Problem Statement

You have a car that needs to travel from a starting position to a destination that is `target` miles east of the starting position.

Along the route, there are gas stations represented by an array `stations`, where each `stations[i] = [position_i, fuel_i]` means:
- The `i-th` gas station is located `position_i` miles east of the starting position
- This station has `fuel_i` liters of gas available

The car operates under these rules:
- It starts with `startFuel` liters of fuel in its tank (which has infinite capacity)
- It consumes exactly 1 liter of gas per mile driven
- When reaching a gas station, the car can choose to stop and refuel, taking all the gas from that station
- The car can refuel at a station even if it arrives with 0 fuel remaining
- Reaching the destination with 0 fuel counts as successfully arriving

Your task is to find the minimum number of refueling stops needed to reach the destination. If it's impossible to reach the destination, return `-1`.

For example, if `target = 100`, `startFuel = 10`, and `stations = [[10, 60], [20, 30], [30, 30], [60, 40]]`:
- The car can drive 10 miles to the first station
- Refuel with 60 liters (1 stop)
- Drive to mile 60 and refuel with 40 liters (2 stops)
- Now with enough fuel to reach mile 100
- Total minimum stops: 2

## Intuition

The key insight is that we want to minimize the number of stops, so we should be strategic about which gas stations to use. We don't need to decide immediately whether to refuel at each station we pass - instead, we can keep track of all the stations we've passed and make refueling decisions retroactively when we run out of fuel.

Think of it this way: as we drive along, we "collect" information about available gas stations. When we run out of fuel, we ask ourselves: "Of all the stations I've already passed, which one would have been the best to stop at?" The answer is always the station with the most fuel, as this maximizes how far we can continue driving.

This suggests a [greedy](/problems/greedy_intro) strategy: 
1. Drive as far as possible with current fuel
2. When we can't reach the next station (or destination), look back at all passed stations
3. Retroactively "refuel" at the station with the most fuel
4. Repeat until we can reach the next checkpoint

Why does this work? Because if we must make `k` stops, choosing the `k` stations with the largest fuel amounts will always give us the maximum possible range. Any other selection would give us less total fuel and potentially require more stops.

The beauty of this approach is that we don't need to make decisions in advance. We can simulate driving forward, and whenever we get stuck, we look back and pick the best option we skipped. This is implemented using a max-heap to efficiently track and retrieve the largest fuel amount from stations we've passed.

By adding `[target, 0]` to the stations list, we treat the destination as a final "station" with no fuel, simplifying our loop logic to handle reaching the destination uniformly with reaching any other station.

**Learn more about [Greedy](/problems/greedy_intro), [Dynamic Programming](/problems/dynamic_programming_intro) and [Heap (Priority Queue)](/problems/heap_intro) patterns.**

## Solution Approach

The solution implements the [greedy](/problems/greedy_intro) strategy using a max-heap (priority queue) to efficiently track and retrieve the gas stations with the most fuel.

**Data Structure**: We use a max-heap `pq` to store fuel amounts from passed stations. In Python, `heapq` provides a min-heap, so we store negative values to simulate a max-heap.

**Algorithm Steps**:

1. **Initialize variables**:
   - `pq = []` - empty heap to store fuel from passed stations
   - `ans = 0` - count of refueling stops
   - `pre = 0` - previous position (starts at 0)
   - Append `[target, 0]` to stations to treat destination as final checkpoint

2. **Process each station/checkpoint**:
   ```python
   for pos, fuel in stations:
       dist = pos - pre          # distance to travel
       startFuel -= dist         # consume fuel for this distance
   ```

3. **Handle fuel shortage** ([greedy](/problems/greedy_intro) refueling):
   ```python
   while startFuel < 0 and pq:
       startFuel -= heappop(pq)  # take largest fuel (negative value)
       ans += 1                   # increment stop count
   ```
   - If `startFuel < 0`, we don't have enough fuel to reach current position
   - Pop the largest fuel amount from heap (most negative value)
   - Add it to our fuel tank (subtracting negative = adding positive)
   - Count this as a refueling stop

4. **Check feasibility**:
   ```python
   if startFuel < 0:
       return -1
   ```
   - If still negative after using all available stations, it's impossible

5. **Record current station**:
   ```python
   heappush(pq, -fuel)   # store as negative for max-heap behavior
   pre = pos             # update position
   ```
   - Add current station's fuel to heap (we might use it later)
   - Update our position marker

**Why this works**: 
- We only refuel when absolutely necessary (fuel goes negative)
- When we must refuel, we choose the best option from our past (largest fuel)
- The heap ensures we can always get the maximum fuel in `O(log n)` time
- Total time complexity: `O(n log n)` where `n` is the number of stations

## Example Walkthrough

Let's trace through a small example to illustrate the solution approach.

**Example**: `target = 25`, `startFuel = 10`, `stations = [[5, 5], [15, 10], [20, 5]]`

**Step-by-step execution**:

**Initial Setup:**
- Add destination as a checkpoint: `stations = [[5, 5], [15, 10], [20, 5], [25, 0]]`
- `pq = []` (max-heap for fuel amounts)
- `ans = 0` (refuel stops)
- `pre = 0` (current position)
- `startFuel = 10`

**Processing Station [5, 5]:**
- Distance to travel: `5 - 0 = 5`
- Fuel after travel: `10 - 5 = 5`
- Fuel is positive, so we can reach this station
- Add station's fuel to heap: `pq = [-5]` (negative for max-heap)
- Update position: `pre = 5`

**Processing Station [15, 10]:**
- Distance to travel: `15 - 5 = 10`
- Fuel after travel: `5 - 10 = -5`
- Fuel is negative! We need to refuel retroactively
- Pop largest fuel from heap: `-(-5) = 5`, so `startFuel = -5 + 5 = 0`
- Increment stops: `ans = 1`
- Now fuel is 0 (just enough to reach)
- Add station's fuel to heap: `pq = [-10]`
- Update position: `pre = 15`

**Processing Station [20, 5]:**
- Distance to travel: `20 - 15 = 5`
- Fuel after travel: `0 - 5 = -5`
- Fuel is negative! Need to refuel again
- Pop largest fuel from heap: `-(-10) = 10`, so `startFuel = -5 + 10 = 5`
- Increment stops: `ans = 2`
- Now fuel is 5 (enough to reach)
- Add station's fuel to heap: `pq = [-5]`
- Update position: `pre = 20`

**Processing Destination [25, 0]:**
- Distance to travel: `25 - 20 = 5`
- Fuel after travel: `5 - 5 = 0`
- Fuel is exactly 0 (we reach destination)
- No need to refuel

**Result**: Minimum stops = 2

The key insight: We drove past the first station without stopping initially, but when we couldn't reach the second station, we retroactively "decided" to refuel there. Similarly, we took fuel from the second station when we couldn't reach the third. This greedy approach of always choosing the station with the most fuel when needed guarantees the minimum number of stops.

## Python Solution

```python
class Solution:
    def minRefuelStops(
        self, target: int, startFuel: int, stations: List[List[int]]
    ) -> int:
        # Max heap to store available fuel amounts (negative values for max heap)
        max_heap = []
      
        # Initialize counters
        refuel_count = 0
        previous_position = 0
      
        # Add the target as a final station with 0 fuel to simplify logic
        stations.append([target, 0])
      
        # Process each station (including the target)
        for position, fuel_available in stations:
            # Calculate distance from previous position
            distance = position - previous_position
          
            # Consume fuel for the distance traveled
            startFuel -= distance
          
            # If we don't have enough fuel, refuel from the largest available stations
            while startFuel < 0 and max_heap:
                # Pop the maximum fuel amount (negative value, so negate it back)
                startFuel -= heappop(max_heap)
                refuel_count += 1
          
            # If still not enough fuel after using all available stations, impossible
            if startFuel < 0:
                return -1
          
            # Add current station's fuel to available options (negative for max heap)
            heappush(max_heap, -fuel_available)
          
            # Update previous position for next iteration
            previous_position = position
      
        return refuel_count
```

## Time and Space Complexity

**Time Complexity:** `O(n × log n)`

The algorithm iterates through all `n` gas stations once. For each station:
- Computing the distance and updating fuel takes `O(1)` time
- In the worst case, we might need to pop from the heap multiple times. Across all iterations, each station's fuel can be pushed to the heap once and popped at most once
- Each heap push operation (`heappush`) takes `O(log n)` time
- Each heap pop operation (`heappop`) takes `O(log n)` time
- Since we have at most `n` push operations and at most `n` pop operations total across all iterations, the overall time complexity is `O(n × log n)`

**Space Complexity:** `O(n)`

The space is primarily used by:
- The priority queue `pq` which can store at most `n` fuel values (one from each station) at any point, requiring `O(n)` space
- The modified `stations` list with the appended target station adds one extra element, but this doesn't change the asymptotic space complexity
- Other variables (`ans`, `pre`, `startFuel`, etc.) use `O(1)` space

Therefore, the total space complexity is `O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Forgetting to Handle the Target Position**
**Pitfall**: Many implementations forget that we need to check if we can reach the target itself, not just the last gas station. If the last station is at position 70 but the target is at 100, we still need 30 more units of fuel.

**Solution**: The code handles this elegantly by appending `[target, 0]` to the stations list. This treats the destination as a final checkpoint with no fuel, ensuring we verify we can reach it.

### 2. **Using Min-Heap Instead of Max-Heap**
**Pitfall**: Python's `heapq` is a min-heap by default. If you directly push positive fuel values and pop them thinking you'll get the maximum, you'll actually get the minimum fuel amount, leading to suboptimal refueling choices.

```python
# Wrong approach - gets minimum fuel
heappush(max_heap, fuel_available)  
fuel_to_add = heappop(max_heap)  # This gives MINIMUM fuel!
```

**Solution**: Store negative values to simulate max-heap behavior:
```python
# Correct approach - simulates max-heap
heappush(max_heap, -fuel_available)  # Store as negative
fuel_to_add = -heappop(max_heap)     # Negate back to get maximum
```

### 3. **Refueling Before It's Necessary**
**Pitfall**: Some might think to refuel at every station with large fuel amounts preemptively. This doesn't minimize the number of stops.

```python
# Wrong greedy approach
for position, fuel_available in stations:
    if fuel_available > threshold:  # Some arbitrary threshold
        startFuel += fuel_available
        refuel_count += 1
```

**Solution**: Only refuel when fuel becomes negative (when we absolutely must). The current implementation correctly uses:
```python
while startFuel < 0 and max_heap:  # Only refuel when necessary
    startFuel -= heappop(max_heap)
    refuel_count += 1
```

### 4. **Not Tracking Previous Position Correctly**
**Pitfall**: Calculating distance from the starting position (0) each time instead of from the previous position leads to incorrect fuel consumption.

```python
# Wrong - always calculating from start
for position, fuel_available in stations:
    startFuel -= position  # This assumes we're always starting from 0!
```

**Solution**: Track and update the previous position:
```python
distance = position - previous_position  # Correct incremental distance
startFuel -= distance
previous_position = position  # Update for next iteration
```

### 5. **Adding Station Fuel Before Checking Reachability**
**Pitfall**: Adding the current station's fuel to the heap before verifying we can actually reach that station.

```python
# Wrong order
for position, fuel_available in stations:
    heappush(max_heap, -fuel_available)  # Added before checking!
    distance = position - previous_position
    startFuel -= distance
    # What if startFuel < 0 and max_heap is empty?
```

**Solution**: First verify we can reach the station (refuel if needed), then add its fuel to available options:
```python
# Correct order
startFuel -= distance
while startFuel < 0 and max_heap:  # Check and refuel if needed
    startFuel -= heappop(max_heap)
    refuel_count += 1
if startFuel < 0:
    return -1  # Can't reach this station
heappush(max_heap, -fuel_available)  # Now safe to add
```

