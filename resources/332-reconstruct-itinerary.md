# 332. Reconstruct Itinerary

> Source: https://algo.monster/liteproblems/332

## Problem Statement

You need to reconstruct a travel itinerary from a collection of airline tickets. Each ticket is represented as `[from, to]` where `from` is the departure airport and `to` is the arrival airport.

The key requirements are:

1. **Starting Point**: The journey must begin at "JFK" airport
2. **Use All Tickets**: Every ticket must be used exactly once - no ticket can be skipped or used multiple times
3. **Valid Path**: The tickets must form a continuous path where each destination becomes the next departure point
4. **Lexicographical Order**: If multiple valid itineraries exist, return the one that comes first alphabetically when read as a sequence

For example, if you have tickets `[["JFK","SFO"],["JFK","ATL"],["SFO","ATL"],["ATL","JFK"],["ATL","SFO"]]`, you need to find a path starting from "JFK" that uses all 5 tickets exactly once. The result would be `["JFK","ATL","JFK","SFO","ATL","SFO"]`.

When comparing itineraries lexicographically, earlier airports in alphabetical order take precedence. For instance, `["JFK", "LGA"]` comes before `["JFK", "LGB"]` because "LGA" < "LGB" alphabetically.

The problem guarantees that at least one valid itinerary exists, so you don't need to handle cases where it's impossible to use all tickets.

This is fundamentally an Eulerian path problem - finding a path through a directed graph that visits every edge exactly once, starting from a specific vertex ("JFK"), with the additional constraint of choosing the lexicographically smallest path when multiple options exist.

## Intuition

The key insight is recognizing this as an Eulerian path problem - we need to visit every edge (ticket) exactly once. But how do we ensure we get the lexicographically smallest path?

Think about what happens when we're at an airport with multiple outgoing flights. If we always choose the lexicographically smallest destination first, we might get stuck! For example, if from "JFK" we can go to "ATL" or "SFO", choosing "ATL" first might lead us down a path where we can't use all tickets.

The clever trick is to **reverse our thinking**. Instead of building the path forward and potentially getting stuck, we can build it backward using post-order DFS traversal. Here's why this works:

1. **Sort destinations in reverse order**: For each airport, we arrange its destinations from largest to smallest lexicographically. This seems counterintuitive at first!

2. **Use DFS with a greedy approach**: We keep visiting airports, always taking the "last" destination from our sorted list (which due to reverse sorting is actually the lexicographically smallest available).

3. **Build the path in reverse**: The magic happens when we add airports to our result. We only add an airport to our answer AFTER we've visited all destinations reachable from it (post-order traversal). This ensures we don't get stuck.

4. **Why post-order works**: When we reach a dead-end (an airport with no more unused outgoing tickets), we add it to our result and backtrack. The airports get added in reverse order of visitation. By reversing the final result, we get the correct forward path.

The algorithm essentially finds the path by exhausting all edges, prioritizing lexicographically smaller choices (via the reverse sort trick), and using the call stack to naturally handle the backtracking. It's like solving a maze by always taking the leftmost path, but recording your steps as you backtrack from dead ends, then reversing that recording to get the actual path from start to finish.

## Solution Approach

The solution implements Hierholzer's algorithm for finding an Eulerian path. Let's break down the implementation step by step:

**1. Data Structure Setup:**
```python
g = defaultdict(list)
for f, t in sorted(tickets, reverse=True):
    g[f].append(t)
```
- We use a `defaultdict(list)` to represent the graph as an adjacency list
- Each airport maps to a list of destination airports
- **Critical trick**: We sort tickets in reverse order before building the graph. This ensures that when we pop from the end of each list later, we're actually getting the lexicographically smallest destination

**2. DFS Implementation:**
```python
def dfs(f: str):
    while g[f]:
        dfs(g[f].pop())
    ans.append(f)
```
- The DFS function takes a departure airport `f`
- While there are still unused tickets from this airport:
  - Pop the last destination (which is lexicographically smallest due to reverse sorting)
  - Recursively visit that destination
- After all destinations are exhausted, add the current airport to the answer

**3. Building the Result:**
```python
ans = []
dfs("JFK")
return ans[::-1]
```
- Start DFS from "JFK" as required
- The `ans` list gets populated in post-order (airports are added after visiting all their destinations)
- Reverse the final answer to get the correct forward path

**Why This Works:**

The algorithm exploits the property of Eulerian paths: if we keep following edges and removing them as we go, we'll eventually use all edges exactly once. The post-order traversal ensures that:

1. Dead-end airports (with no outgoing tickets) are added to the result first
2. Airports that lead to dead-ends are added next
3. The starting airport "JFK" is added last

When we reverse this, we get a valid path from "JFK" that uses all tickets exactly once.

**Time Complexity:** `O(E log E)` where E is the number of tickets (for sorting)
**Space Complexity:** `O(E)` for storing the graph and recursion stack

## Example Walkthrough

Let's trace through a small example with tickets: `[["JFK","B"],["B","JFK"],["JFK","A"],["A","JFK"]]`

**Step 1: Sort tickets in reverse order**
After sorting in reverse: `[["JFK","B"],["JFK","A"],["B","JFK"],["A","JFK"]]`

**Step 2: Build the graph**
```
g = {
  "JFK": ["B", "A"]  // Note: B comes before A (we'll pop from the end)
  "B": ["JFK"]
  "A": ["JFK"]
}
```

**Step 3: DFS traversal starting from JFK**

Let's trace the DFS calls:

1. `dfs("JFK")` - JFK has destinations ["B", "A"]
   - Pop "A" (last element), so we visit A first
   - Call `dfs("A")`

2. `dfs("A")` - A has destination ["JFK"]
   - Pop "JFK", call `dfs("JFK")`

3. `dfs("JFK")` again - JFK now has ["B"] left
   - Pop "B", call `dfs("B")`

4. `dfs("B")` - B has destination ["JFK"]
   - Pop "JFK", call `dfs("JFK")`

5. `dfs("JFK")` again - JFK has no more destinations
   - No more tickets, append "JFK" to ans
   - ans = ["JFK"]

6. Back to `dfs("B")` - finished with while loop
   - Append "B" to ans
   - ans = ["JFK", "B"]

7. Back to `dfs("JFK")` from step 3 - finished with while loop
   - Append "JFK" to ans
   - ans = ["JFK", "B", "JFK"]

8. Back to `dfs("A")` - finished with while loop
   - Append "A" to ans
   - ans = ["JFK", "B", "JFK", "A"]

9. Back to initial `dfs("JFK")` - finished with while loop
   - Append "JFK" to ans
   - ans = ["JFK", "B", "JFK", "A", "JFK"]

**Step 4: Reverse the result**
Final answer: `["JFK", "A", "JFK", "B", "JFK"]`

This gives us the lexicographically smallest path that uses all tickets exactly once. Notice how by popping from the end (which gives us the smallest destination due to reverse sorting) and using post-order traversal, we naturally find the correct path without getting stuck!

## Python Solution

```python
from collections import defaultdict
from typing import List

class Solution:
    def findItinerary(self, tickets: List[List[str]]) -> List[str]:
        # Build adjacency list (graph) from tickets
        # Sort tickets in reverse order so that when we pop from the list,
        # we get the lexicographically smallest destination first
        graph = defaultdict(list)
        for from_airport, to_airport in sorted(tickets, reverse=True):
            graph[from_airport].append(to_airport)
      
        # Result list to store the itinerary in reverse order
        result = []
      
        def dfs(current_airport: str) -> None:
            """
            Perform depth-first search using Hierholzer's algorithm
            to find Eulerian path.
          
            Args:
                current_airport: Current airport code
            """
            # Visit all destinations from current airport
            while graph[current_airport]:
                # Pop the lexicographically smallest destination
                # (since list is in reverse order, pop gives smallest)
                next_airport = graph[current_airport].pop()
                dfs(next_airport)
          
            # Add current airport to result after visiting all its destinations
            # This ensures we build the path from end to start
            result.append(current_airport)
      
        # Start DFS from JFK airport
        dfs("JFK")
      
        # Reverse the result to get the correct order of itinerary
        return result[::-1]
```

## Time and Space Complexity

**Time Complexity:** `O(m × log m)`

The time complexity is dominated by two main operations:
1. **Sorting the tickets:** The `sorted(tickets, reverse=True)` operation takes `O(m × log m)` time, where `m` is the number of tickets (edges).
2. **DFS traversal:** The DFS visits each edge exactly once. Since we pop each destination from the adjacency list, each edge is processed once, taking `O(m)` time in total.

Therefore, the overall time complexity is `O(m × log m) + O(m) = O(m × log m)`.

**Space Complexity:** `O(m)`

The space complexity consists of:
1. **Graph storage (`g`):** The defaultdict stores all edges, requiring `O(m)` space to store all destinations.
2. **Answer list (`ans`):** The answer list will eventually contain `m + 1` airports (vertices), which is `O(m)` space.
3. **Recursion stack:** In the worst case, the DFS recursion can go as deep as `m` levels (a linear path through all edges), requiring `O(m)` stack space.

Therefore, the overall space complexity is `O(m) + O(m) + O(m) = O(m)`.

## Common Pitfalls

### 1. **Incorrect Sorting Direction**
One of the most common mistakes is sorting the tickets in ascending order instead of reverse order:

**Wrong Approach:**
```python
# This seems intuitive but is incorrect!
for f, t in sorted(tickets):  # Ascending order
    graph[f].append(t)
```

**Why it fails:** When you pop from the end of a list sorted in ascending order, you get the lexicographically *largest* destination, not the smallest. This produces an incorrect itinerary.

**Correct Approach:**
```python
# Sort in reverse so pop() gives smallest
for f, t in sorted(tickets, reverse=True):
    graph[f].append(t)
```

### 2. **Using a Priority Queue Instead of Sorted List**
Some might try using a min-heap for lexicographical ordering:

**Alternative Approach:**
```python
import heapq
graph = defaultdict(list)
for f, t in tickets:
    heapq.heappush(graph[f], t)

# In DFS:
while graph[current]:
    next_airport = heapq.heappop(graph[current])  # This works but...
```

**Trade-off:** This gives correct results with the same overall bound, `O(E log E)`: each push and pop costs `O(log E)`. The sorted-list version pays the `log` factor once, in the initial sort, and then pops in `O(1)`, so it is usually a little faster in practice and simpler to read.

### 3. **Building Path in Forward Direction**
Attempting to build the itinerary directly in forward order:

**Wrong Approach:**
```python
def dfs(current):
    result.append(current)  # Add before visiting destinations
    while graph[current]:
        next_airport = graph[current].pop()
        dfs(next_airport)
```

**Why it fails:** This produces an incomplete path. The algorithm relies on post-order traversal to ensure all edges are used. Adding airports before exploring their destinations breaks the Eulerian path construction.

### 4. **Not Handling Multiple Tickets Between Same Airports**
Forgetting that there can be multiple tickets from airport A to airport B:

**Wrong Approach:**
```python
graph = {}  # Using set or single value
for f, t in tickets:
    graph[f] = t  # Overwrites previous ticket!
```

**Correct Approach:**
```python
graph = defaultdict(list)  # Must use list to store multiple tickets
for f, t in sorted(tickets, reverse=True):
    graph[f].append(t)  # Preserves all tickets
```

### 5. **Misunderstanding the Lexicographical Requirement**
Thinking you need to sort the entire path afterward:

**Wrong Approach:**
```python
# Build any valid path first
result = build_any_path(tickets)
# Then try to sort it somehow
return sorted(result)  # This doesn't work!
```

**Why it fails:** The lexicographical ordering must be considered *during* path construction, not after. The requirement is for the earliest lexicographical *valid itinerary*, not a sorted list of airports.

**Solution:** The algorithm handles this by always choosing the lexicographically smallest available destination at each step (via the reverse sort + pop combination).

