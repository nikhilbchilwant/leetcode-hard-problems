# 2127. Maximum Employees to Be Invited to a Meeting

> Source: https://algo.monster/liteproblems/2127

## Problem Statement

You have a company organizing a meeting with `n` employees (numbered from `0` to `n - 1`) who need to be seated at a large circular table. Each employee has exactly one favorite person (different from themselves), and they will only attend the meeting if they can sit next to their favorite person.

Given an array `favorite` where `favorite[i]` represents the favorite person of employee `i`, you need to find the maximum number of employees that can be invited to the meeting.

The key constraints are:
- Employees must sit in a circle at the table
- Each employee will only attend if they can sit next to their favorite person
- An employee cannot be their own favorite person

For example, if employee A's favorite is B, then A will only attend if they can sit next to B at the circular table. The challenge is to find the largest possible group of employees where everyone's seating preference can be satisfied.

The problem essentially asks you to find valid seating arrangements where:
1. Employees form a cycle where each person sits next to their favorite, OR
2. Multiple smaller valid groups can be combined (specifically when two employees are each other's favorites, allowing chains to extend from both sides)

## Intuition

Let's think about what makes a valid seating arrangement. Since everyone sits in a circle and must sit next to their favorite person, we need to trace the "favorite" relationships to see what patterns emerge.

If we start with any employee and follow their favorite person chain: Employee A → likes B → likes C → likes D → ... Eventually, since there are finite employees and everyone has exactly one favorite, we must encounter someone we've already seen, forming a cycle.

This creates two possible valid configurations:

**Case 1: Large Cycles**
When employees form a cycle where A likes B, B likes C, ..., and the last person likes A, everyone in this cycle can sit together with their favorite next to them. The key insight is that only one such cycle can be selected because cycles are isolated from each other.

**Case 2: Mutual Pairs with Chains**
There's a special case when exactly two people like each other (A likes B and B likes A). This creates a "stable core" where both are satisfied sitting next to each other. Now here's the clever part: other employees who like A can form a chain leading to A, and similarly for B. These chains can all be seated together because:
- The mutual pair (A, B) sits in the middle
- Chains of people can extend from both sides
- Everyone in the chains gets to sit next to their favorite person

For example: `X → Y → A ↔ B ← C ← D` forms a valid arrangement where the mutual pair A-B anchors the configuration.

The beautiful insight is that multiple such mutual pairs with their chains can coexist at the table since they're independent structures. We can seat all of them together.

Therefore, our answer is the maximum of:
1. The largest single cycle we can find
2. The sum of all mutual pairs plus their extending chains

This explains why the solution uses two different approaches: finding the maximum cycle length for Case 1, and using [topological sort](/problems/topo_intro) to find the longest chains extending to mutual pairs for Case 2.

**Learn more about [Depth-First Search](/problems/dfs_intro), [Graph](/problems/graph_intro) and [Topological Sort](/problems/topo_intro) patterns.**

## Solution Approach

The solution implements two key algorithms to handle the two cases we identified:

**1. Finding Maximum Cycle (`max_cycle` function)**

This function uses DFS to detect and measure cycles in the [graph](/problems/graph_intro):
- We maintain a `vis` array to track visited nodes
- For each unvisited employee `i`, we start building a path by following the favorite relationships
- We keep adding employees to a `cycle` list and marking them as visited
- When we encounter an already visited employee `j`, we've found a cycle
- We then backtrack through our `cycle` list to find where `j` appears - the distance from that position to the end gives us the cycle length
- We track the maximum cycle length found

The key insight is that when we hit a visited node, only the portion from that node onwards forms the actual cycle.

**2. Finding Mutual Pairs with Chains (`topological_sort` function)**

This function uses a modified [topological sort](/problems/topo_intro) to find all 2-cycles and their extending chains:
- We calculate the in-degree for each node (how many people have them as favorite)
- We initialize `dist[i] = 1` for each employee (representing the chain length ending at them)
- Starting with nodes of in-degree 0 (leaves of the chains), we process using BFS:
  - For each processed employee `i`, we update their favorite's distance: `dist[fa[i]] = max(dist[fa[i]], dist[i] + 1)`
  - We decrement the in-degree of their favorite and add them to queue if it becomes 0
- After processing, employees in 2-cycles will still have non-zero in-degree
- For each 2-cycle (where `i == fa[fa[i]]`), we sum up their `dist` values

The distance array `dist[i]` represents the longest chain of employees that can eventually lead to employee `i`. For mutual pairs, both employees' distances include their respective longest chains.

**3. Combining Results**

The final answer is `max(max_cycle(favorite), topological_sort(favorite))`:
- Either we take the largest single cycle
- Or we take all mutual pairs with their extending chains (which can all sit together)

The data structures used are:
- Arrays for tracking visited states and distances
- Queue (deque) for BFS in [topological sort](/problems/topo_intro)
- Lists to build cycles during DFS

This approach efficiently handles both types of valid seating arrangements in `O(n)` time complexity.

## Example Walkthrough

Let's walk through a concrete example with 6 employees to illustrate both cases:

**Example Input:** `favorite = [2, 3, 1, 4, 5, 3]`

This means:
- Employee 0 likes employee 2
- Employee 1 likes employee 3
- Employee 2 likes employee 1
- Employee 3 likes employee 4
- Employee 4 likes employee 5
- Employee 5 likes employee 3

**Step 1: Finding Maximum Cycle**

Starting DFS from employee 0:
- Visit 0 → favorite is 2 → Visit 2 → favorite is 1 → Visit 1 → favorite is 3 → Visit 3 → favorite is 4 → Visit 4 → favorite is 5 → Visit 5 → favorite is 3 (already visited!)
- We've built the path: [0, 2, 1, 3, 4, 5] and hit 3 again
- Find where 3 appears in our path: position 3
- Cycle length = 6 - 3 = 3 (the cycle is 3→4→5→3)

We also have a mutual pair: employees 1 and 2 like each other (1→3 and 2→1 would be processed, but 3 is part of the larger cycle).

**Step 2: Finding Mutual Pairs with Chains**

Let's modify the example to better show this case: `favorite = [1, 0, 3, 2, 5, 4]`

Now we have three mutual pairs:
- Employees 0 and 1 like each other
- Employees 2 and 3 like each other  
- Employees 4 and 5 like each other

Using topological sort:
1. Calculate in-degrees: all employees have in-degree 1 (each has exactly one person who likes them)
2. Since no one has in-degree 0, all employees are in 2-cycles
3. Process each mutual pair:
   - Pair (0,1): dist[0]=1, dist[1]=1, total=2
   - Pair (2,3): dist[2]=1, dist[3]=1, total=2
   - Pair (4,5): dist[4]=1, dist[5]=1, total=2
4. Sum of all pairs = 2 + 2 + 2 = 6

**More Complex Chain Example:** `favorite = [1, 2, 3, 4, 3, 1]`

Here:
- Employee 0 → 1 → 2 → 3 ↔ 4 (mutual pair)
- Employee 5 → 1

Processing:
1. In-degrees: [0, 2, 1, 2, 1, 0]
2. Start with employees 0 and 5 (in-degree 0):
   - Process 0: dist[0]=1, update dist[1]=2, reduce in-degree[1] to 1
   - Process 5: dist[5]=1, update dist[1]=max(2,2)=2, reduce in-degree[1] to 0
   - Process 1: dist[1]=2, update dist[2]=3, reduce in-degree[2] to 0
   - Process 2: dist[2]=3, update dist[3]=4, reduce in-degree[3] to 1
3. Employees 3 and 4 form a mutual pair (in-degree remains 1)
4. Total for this configuration: dist[3] + dist[4] = 4 + 1 = 5

The seating would be: 0 → 1 → 2 → 3 ↔ 4 (5 people can attend)

**Final Answer:** We take the maximum between the largest cycle and the sum of all mutual pair configurations.

## Python Solution

```python
from typing import List
from collections import deque


class Solution:
    def maximumInvitations(self, favorite: List[int]) -> int:
        """
        Find the maximum number of employees that can be invited to a circular table
        where each employee wants to sit next to their favorite person.
      
        Two valid configurations:
        1. A single cycle where everyone's favorite is their neighbor
        2. Multiple 2-cycles with chains leading into them
        """
      
        def find_max_cycle_length(favorite_mapping: List[int]) -> int:
            """
            Find the length of the longest cycle in the graph.
            This handles case 1: a single large cycle.
            """
            n = len(favorite_mapping)
            visited = [False] * n
            max_cycle_length = 0
          
            for start_node in range(n):
                if visited[start_node]:
                    continue
              
                # Traverse and record the path until we hit a visited node
                path = []
                current = start_node
              
                while not visited[current]:
                    path.append(current)
                    visited[current] = True
                    current = favorite_mapping[current]
              
                # Find where the cycle starts in our path
                for index, node in enumerate(path):
                    if node == current:
                        cycle_length = len(path) - index
                        max_cycle_length = max(max_cycle_length, cycle_length)
                        break
          
            return max_cycle_length
      
        def find_two_cycles_with_chains(favorite_mapping: List[int]) -> int:
            """
            Find the sum of all 2-cycles with their incoming chains.
            This handles case 2: multiple pairs with chains leading to them.
            """
            n = len(favorite_mapping)
            in_degree = [0] * n
            max_chain_length = [1] * n  # Length of longest chain ending at each node
          
            # Calculate in-degrees for each node
            for target in favorite_mapping:
                in_degree[target] += 1
          
            # Start topological sort from nodes with no incoming edges
            queue = deque(node for node, degree in enumerate(in_degree) if degree == 0)
          
            # Process nodes level by level, updating chain lengths
            while queue:
                current = queue.popleft()
                next_node = favorite_mapping[current]
              
                # Update the maximum chain length reaching the next node
                max_chain_length[next_node] = max(
                    max_chain_length[next_node], 
                    max_chain_length[current] + 1
                )
              
                # Decrease in-degree and add to queue if it becomes 0
                in_degree[next_node] -= 1
                if in_degree[next_node] == 0:
                    queue.append(next_node)
          
            # Sum up all 2-cycles with their incoming chains
            # A 2-cycle exists when i -> j and j -> i
            total_length = 0
            for node in range(n):
                next_node = favorite_mapping[node]
                if node == favorite_mapping[next_node]:  # Found a 2-cycle
                    total_length += max_chain_length[node]
          
            return total_length
      
        # Return the maximum of both configurations
        max_single_cycle = find_max_cycle_length(favorite)
        max_two_cycles = find_two_cycles_with_chains(favorite)
      
        return max(max_single_cycle, max_two_cycles)
```

## Time and Space Complexity

**Time Complexity: `O(n)`**

The algorithm consists of two main functions: `max_cycle` and `topological_sort`.

For `max_cycle`:
- The outer loop iterates through all `n` nodes once
- For each unvisited node, we traverse until we find a visited node, marking each node as visited
- Each node is visited at most once throughout the entire execution
- The inner enumeration to find the cycle start is bounded by the nodes in the current path
- Total operations: `O(n)`

For `topological_sort`:
- Building the indegree array: `O(n)` 
- Initializing the queue with nodes having indegree 0: `O(n)`
- BFS traversal processes each node exactly once: `O(n)`
- Final sum calculation: `O(n)`
- Total operations: `O(n)`

Since both functions run sequentially and each has `O(n)` complexity, the overall time complexity is `O(n) + O(n) = O(n)`.

**Space Complexity: `O(n)`**

For `max_cycle`:
- `vis` array: `O(n)`
- `cycle` list: worst case `O(n)` when all nodes form one cycle
- Other variables: `O(1)`

For `topological_sort`:
- `indeg` array: `O(n)`
- `dist` array: `O(n)`
- Queue `q`: worst case `O(n)` when all nodes initially have indegree 0
- Other variables: `O(1)`

The maximum space used at any point is `O(n)`, giving an overall space complexity of `O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrectly Handling Cycle Detection in DFS**

**Pitfall:** A common mistake is assuming that when we encounter a visited node during DFS, the entire path we've traversed forms a cycle. In reality, only the portion from the visited node onwards is the actual cycle.

**Example of incorrect logic:**
```python
# WRONG: Assuming entire path is a cycle
while not visited[current]:
    path.append(current)
    visited[current] = True
    current = favorite_mapping[current]
cycle_length = len(path)  # This is wrong!
```

**Correct approach:** When we hit a visited node, we need to find where that node appears in our current path. Only the nodes from that position onwards form the cycle:
```python
# Find where the cycle actually starts
for index, node in enumerate(path):
    if node == current:  # current is the revisited node
        cycle_length = len(path) - index
        break
```

### 2. **Double-Counting 2-Cycles**

**Pitfall:** When summing up 2-cycles with their chains, it's easy to count each 2-cycle twice since both nodes in the pair satisfy the mutual favorite condition.

**Example of incorrect logic:**
```python
# WRONG: This counts each 2-cycle twice
total_length = 0
for node in range(n):
    next_node = favorite_mapping[node]
    if node == favorite_mapping[next_node]:
        total_length += max_chain_length[node] + max_chain_length[next_node]
```

**Correct approach:** Only add the chain length for one node in each iteration, as the total naturally includes both chains:
```python
# Correct: Each node's chain length is added once
for node in range(n):
    next_node = favorite_mapping[node]
    if node == favorite_mapping[next_node]:
        total_length += max_chain_length[node]
```

### 3. **Misunderstanding Chain Length Initialization**

**Pitfall:** Initializing `max_chain_length` to 0 instead of 1, forgetting that each employee counts as a chain of length 1 by themselves.

**Example of incorrect logic:**
```python
# WRONG: Starting with 0
max_chain_length = [0] * n  # Each employee should count as 1
```

**Correct approach:**
```python
# Correct: Each employee is a chain of length 1
max_chain_length = [1] * n
```

### 4. **Not Considering All Starting Points for Cycle Detection**

**Pitfall:** Breaking early after finding one cycle or not checking all unvisited nodes, potentially missing the maximum cycle.

**Example of incorrect logic:**
```python
# WRONG: Returning after first cycle found
for start_node in range(n):
    if not visited[start_node]:
        # Find a cycle
        return cycle_length  # Missing other potentially larger cycles
```

**Correct approach:**
```python
# Correct: Check all components and track maximum
max_cycle_length = 0
for start_node in range(n):
    if not visited[start_node]:
        # Find cycle in this component
        max_cycle_length = max(max_cycle_length, cycle_length)
return max_cycle_length
```

### 5. **Confusing Graph Direction**

**Pitfall:** Treating the favorite relationship bidirectionally or getting confused about edge direction. Remember that `favorite[i] = j` means there's a directed edge from `i` to `j`, not the other way around.

**Solution:** Always be clear that:
- `favorite[i]` represents an outgoing edge from `i`
- In-degree counts how many people have node `i` as their favorite
- Chains grow in the reverse direction of edges (from leaves toward cycles)

