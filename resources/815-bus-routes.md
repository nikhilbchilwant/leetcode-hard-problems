# 815. Bus Routes

> Source: https://algo.monster/liteproblems/815

## Problem Statement

You have multiple bus routes, where each bus travels in a circular loop through a sequence of stops forever. For example, if a bus route is `[1, 5, 7]`, the bus travels: `1 → 5 → 7 → 1 → 5 → 7 → 1 → ...` continuously.

Given:
- An array `routes` where `routes[i]` represents the stops that the `i-th` bus visits in its circular route
- A `source` bus stop where you start (you're initially not on any bus)
- A `target` bus stop where you want to go

You can only travel between bus stops by taking buses. Your goal is to find the minimum number of buses you need to take to travel from `source` to `target`.

Return the least number of buses required, or `-1` if it's impossible to reach the target from the source.

**Key Points:**
- Each bus follows its route in a circular pattern forever
- You start at a bus stop (not on a bus)
- You can transfer between buses at common stops
- The answer is the count of different buses used, not the number of stops

**Example:**
If you have routes `[[1, 2, 7], [3, 6, 7]]` and want to go from stop `1` to stop `6`:
- Take bus 0 from stop `1` to stop `7`
- Transfer to bus 1 at stop `7` and ride to stop `6`
- Total buses taken: 2

## Intuition

The key insight is to think of this problem not as traveling through individual stops, but as traveling through buses. Each time we board a new bus, that counts as one unit toward our answer.

Initially, we might think to model this as a graph where stops are nodes and direct connections between consecutive stops on the same route are edges. However, this would make finding the minimum number of buses complex, as we'd need to track which bus we're currently on.

Instead, we can restructure our thinking: from any stop, we can reach all other stops on the same bus route with just one bus ride. This means if we're at stop `A` and want to reach stop `B` on the same bus route, it takes exactly 1 bus, regardless of how many stops are between them.

This leads to a more efficient graph representation where:
- We first map each stop to all the buses that visit it
- When we're at a stop, we can explore all buses passing through that stop
- For each unvisited bus, we can reach all stops on that bus route with one additional bus ride

The BFS approach naturally fits because:
1. We start from the source stop with 0 buses taken
2. We explore all buses reachable from our current stop
3. For each bus, we add all its stops to our queue with `bus_count + 1`
4. The first time we reach the target stop, we've found the minimum number of buses

To avoid revisiting, we track:
- `vis_bus`: Buses we've already boarded (no need to board the same bus twice)
- `vis_stop`: Stops we've already visited (prevents infinite loops)

This way, BFS guarantees that when we first reach the target, we've taken the minimum number of buses possible.

**Learn more about [Breadth-First Search](/problems/bfs_intro) patterns.**

## Solution Approach

Following our BFS intuition, let's walk through the implementation step by step:

**1. Handle Base Cases**
```python
if source == target:
    return 0
```
If we're already at the target, no buses are needed.

**2. Build Stop-to-Bus Mapping**
```python
g = defaultdict(list)
for i, route in enumerate(routes):
    for stop in route:
        g[stop].append(i)
```
We create a hash table `g` (named `stop_to_buses` in the code; `q`, `vis_bus`, and `vis_stop` below are `queue`, `visited_buses`, and `visited_stops`) where `g[stop]` contains all bus indices that visit that stop. This allows us to quickly find which buses we can board from any given stop.

**3. Validate Source and Target**
```python
if source not in g or target not in g:
    return -1
```
If either the source or target stop isn't serviced by any bus, it's impossible to make the journey.

**4. Initialize BFS**
```python
q = [(source, 0)]
vis_bus = set()
vis_stop = {source}
```
- Queue `q` stores tuples of `(stop, bus_count)` representing our current position and buses taken
- `vis_bus` tracks which buses we've already boarded to avoid redundant exploration
- `vis_stop` tracks visited stops to prevent revisiting the same stop

**5. BFS Traversal**
```python
for stop, bus_count in q:
    if stop == target:
        return bus_count
```
We process the queue level by level. If we reach the target, we immediately return the bus count since BFS guarantees this is the minimum.

**6. Explore Available Buses**
```python
for bus in g[stop]:
    if bus not in vis_bus:
        vis_bus.add(bus)
        for next_stop in routes[bus]:
            if next_stop not in vis_stop:
                vis_stop.add(next_stop)
                q.append((next_stop, bus_count + 1))
```
For each unvisited bus at the current stop:
- Mark the bus as visited
- Add all unvisited stops on this bus route to the queue with `bus_count + 1`
- This represents taking one more bus to reach those stops

**7. No Path Found**
```python
return -1
```
If we exhaust the queue without reaching the target, no path exists.

**Time Complexity**: `O(N * M)` where `N` is the number of buses and `M` is the average number of stops per bus. We visit each bus at most once and process all its stops.

**Space Complexity**: `O(N * M)` for storing the graph mapping and visited sets.

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

**Example:** 
- Routes: `[[1, 2, 3], [3, 4, 5], [5, 6]]`
- Source: `1`
- Target: `6`

**Step 1: Build Stop-to-Bus Mapping**
We create a mapping of which buses visit each stop:
```
Stop 1: [Bus 0]
Stop 2: [Bus 0]
Stop 3: [Bus 0, Bus 1]
Stop 4: [Bus 1]
Stop 5: [Bus 1, Bus 2]
Stop 6: [Bus 2]
```

**Step 2: Initialize BFS**
- Queue: `[(1, 0)]` - starting at stop 1 with 0 buses taken
- Visited buses: `{}`
- Visited stops: `{1}`

**Step 3: First Iteration (Process stop 1)**
- Current: stop 1, bus_count = 0
- Not the target (6), so continue
- Buses available at stop 1: [Bus 0]
- Board Bus 0 (mark as visited)
- Add all stops from Bus 0 to queue:
  - Stop 2 (not visited) → add `(2, 1)` to queue
  - Stop 3 (not visited) → add `(3, 1)` to queue
  - Stop 1 (already visited) → skip

Queue now: `[(2, 1), (3, 1)]`
Visited buses: `{0}`
Visited stops: `{1, 2, 3}`

**Step 4: Second Iteration (Process stop 2)**
- Current: stop 2, bus_count = 1
- Not the target, continue
- Buses at stop 2: [Bus 0]
- Bus 0 already visited → skip

**Step 5: Third Iteration (Process stop 3)**
- Current: stop 3, bus_count = 1
- Not the target, continue
- Buses at stop 3: [Bus 0, Bus 1]
- Bus 0 already visited → skip
- Board Bus 1 (mark as visited)
- Add stops from Bus 1:
  - Stop 4 (not visited) → add `(4, 2)` to queue
  - Stop 5 (not visited) → add `(5, 2)` to queue
  - Stop 3 (already visited) → skip

Queue now: `[(4, 2), (5, 2)]`
Visited buses: `{0, 1}`
Visited stops: `{1, 2, 3, 4, 5}`

**Step 6: Fourth Iteration (Process stop 4)**
- Current: stop 4, bus_count = 2
- Not the target, continue
- Buses at stop 4: [Bus 1]
- Bus 1 already visited → skip

**Step 7: Fifth Iteration (Process stop 5)**
- Current: stop 5, bus_count = 2
- Not the target, continue
- Buses at stop 5: [Bus 1, Bus 2]
- Bus 1 already visited → skip
- Board Bus 2 (mark as visited)
- Add stops from Bus 2:
  - Stop 6 (not visited) → add `(6, 3)` to queue
  - Stop 5 (already visited) → skip

Queue now: `[(6, 3)]`
Visited buses: `{0, 1, 2}`
Visited stops: `{1, 2, 3, 4, 5, 6}`

**Step 8: Sixth Iteration (Process stop 6)**
- Current: stop 6, bus_count = 3
- **This is our target!**
- Return 3

**Answer: 3 buses needed**
- Bus 0: Travel from stop 1 to stop 3
- Bus 1: Travel from stop 3 to stop 5
- Bus 2: Travel from stop 5 to stop 6

The BFS approach ensures we find the minimum number of buses by exploring all possibilities level by level, where each level represents taking one additional bus.

## Python Solution

```python
from collections import defaultdict, deque
from typing import List

class Solution:
    def numBusesToDestination(
        self, routes: List[List[int]], source: int, target: int
    ) -> int:
        # If already at destination, no buses needed
        if source == target:
            return 0
      
        # Build adjacency list: stop -> list of bus indices that visit this stop
        stop_to_buses = defaultdict(list)
        for bus_index, route in enumerate(routes):
            for stop in route:
                stop_to_buses[stop].append(bus_index)
      
        # Check if source and target are reachable by any bus
        if source not in stop_to_buses or target not in stop_to_buses:
            return -1
      
        # BFS queue: stores (current_stop, number_of_buses_taken)
        queue = deque([(source, 0)])
      
        # Track visited buses to avoid revisiting the same bus route
        visited_buses = set()
      
        # Track visited stops to avoid revisiting the same stop
        visited_stops = {source}
      
        # BFS traversal
        while queue:
            current_stop, buses_taken = queue.popleft()
          
            # Check if we've reached the target
            if current_stop == target:
                return buses_taken
          
            # Explore all buses that visit the current stop
            for bus_index in stop_to_buses[current_stop]:
                # Skip if we've already taken this bus
                if bus_index in visited_buses:
                    continue
              
                # Mark this bus as visited
                visited_buses.add(bus_index)
              
                # Add all unvisited stops on this bus route to the queue
                for next_stop in routes[bus_index]:
                    if next_stop not in visited_stops:
                        visited_stops.add(next_stop)
                        queue.append((next_stop, buses_taken + 1))
      
        # Target is unreachable
        return -1
```

## Time and Space Complexity

**Time Complexity:** `O(L)`, where `L` is the total number of stops across all bus routes.

The algorithm builds a graph mapping each stop to its bus routes, which takes `O(L)` time since we iterate through every stop in every route once. In the BFS traversal, each bus route is visited at most once (tracked by `visited_buses`), and each stop is visited at most once (tracked by `visited_stops`). When a bus is visited, we iterate through all its stops. Since each stop belongs to at least one bus route and we process each stop-bus pair once, the total work done is proportional to the sum of all stops across all routes, which is `O(L)`.

**Space Complexity:** `O(L)`, where `L` is the total number of stops across all bus routes.

The mapping `stop_to_buses` stores mappings from stops to bus indices. In the worst case, if every stop appears in every route, the graph would store `O(L)` entries. The `visited_buses` set can contain at most `O(n)` buses (where `n` is the number of routes), and `visited_stops` can contain at most `O(L)` unique stops. The `queue` can also hold at most `O(L)` stops in the worst case. Since the number of routes `n` is bounded by `L` (as each route must have at least one stop), the overall space complexity is `O(L)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Treating Stops as Graph Nodes Instead of Buses**
A common mistake is building a graph where stops are nodes and edges connect neighboring stops along a route, then running BFS on it. The BFS distance is then the number of stops traveled, not the number of buses.

**Wrong Approach:**
```python
# Building a stop-to-stop graph from neighboring stops (INCORRECT)
graph = defaultdict(set)
for route in routes:
    for i in range(len(route)):
        a, b = route[i], route[(i + 1) % len(route)]
        graph[a].add(b)
        graph[b].add(a)
# BFS over stops then returns the number of edges traveled
```

**Why it's wrong:** This counts stop-to-stop moves, not buses. With `routes = [[1, 2, 3, 4]]`, `source = 1`, `target = 3`, the BFS distance is 2 (`1 → 2 → 3`, or `1 → 4 → 3` around the loop), but only 1 bus is needed. Connecting every pair of stops on the same route would give the right count (one edge per ride), but it builds up to `len(route)²` edges per route, which is far more work than the stop-to-bus mapping.

**Correct Approach:** Treat buses as the primary unit of traversal. When you board a bus, you can reach all stops on that bus with just that one bus.

### 2. **Not Tracking Visited Buses**
Without tracking visited buses, the algorithm repeatedly scans the same bus route from different stops. The `visited_stops` check keeps the answer correct and prevents infinite loops, but the repeated scans waste time.

**Wrong Implementation:**
```python
# Only tracking visited stops (INCOMPLETE)
visited_stops = {source}
for stop, bus_count in queue:
    for bus in stop_to_buses[stop]:
        # Missing: if bus in visited_buses: continue
        for next_stop in routes[bus]:
            if next_stop not in visited_stops:
                visited_stops.add(next_stop)
                queue.append((next_stop, bus_count + 1))
```

**Problem:** If bus 0 visits stops [1, 2, 3], you might explore bus 0 from stop 1, then again from stop 2, leading to redundant work.

**Solution:** Always maintain a `visited_buses` set and check it before exploring any bus route.

### 3. **Incrementing Bus Count at Wrong Time**
A subtle error is incrementing the bus count when you reach a stop rather than when you board a new bus.

**Wrong:**
```python
if current_stop == target:
    return buses_taken + 1  # WRONG: adds extra bus
```

**Correct:**
```python
# Increment only when adding stops from a NEW bus to the queue
queue.append((next_stop, buses_taken + 1))
```

### 4. **Using BFS Level-by-Level Instead of Simple Queue Processing**
Some implementations try to process BFS level-by-level to track bus counts, which adds unnecessary complexity.

**Overcomplicated:**
```python
level = 0
while queue:
    size = len(queue)
    for _ in range(size):
        stop = queue.popleft()
        if stop == target:
            return level
        # ... explore neighbors
    level += 1
```

**Better:** Store the bus count directly with each stop in the queue as a tuple `(stop, bus_count)`. This is cleaner and less error-prone.

### 5. **Forgetting Circular Nature Doesn't Matter**
Some solutions try to handle the circular nature of routes specially, but it's unnecessary for this problem.

**Unnecessary Complexity:**
```python
# Trying to handle circular routes specially
for route in routes:
    # Adding reverse connections or duplicating routes
    extended_route = route + route  # UNNECESSARY
```

**Why it's unnecessary:** Since we can board a bus at any stop and travel to any other stop on that route, the circular nature is implicitly handled. We just need to know which stops each bus visits.

