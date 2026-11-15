# 1203. Sort Items by Groups Respecting Dependencies

> Source: https://algo.monster/liteproblems/1203

## Problem Statement

You are given `n` items numbered from 0 to n-1. Each item can belong to at most one group out of `m` groups (numbered 0 to m-1). The group membership is specified by the array `group`, where:
- `group[i]` represents the group that item `i` belongs to
- `group[i] = -1` means item `i` doesn't belong to any group

Additionally, there are dependency constraints between items specified by `beforeItems`, where `beforeItems[i]` is a list of items that must appear before item `i` in the final sorted order.

Your task is to return a sorted list of all items that satisfies two conditions:
1. **Group Constraint**: Items belonging to the same group must be placed consecutively (next to each other) in the sorted list
2. **Dependency Constraint**: For each item `i`, all items in `beforeItems[i]` must appear before item `i` in the sorted list

If multiple valid orderings exist, return any one of them. If no valid ordering exists (due to conflicting constraints), return an empty list.

For example, if items 0, 2, and 4 belong to group 1, they must appear consecutively in the result (like ...0,2,4... or ...4,0,2...), but their internal order depends on the dependency constraints. Meanwhile, if item 3 has `beforeItems[3] = [0, 2]`, then both items 0 and 2 must appear before item 3 in the final ordering.

## Intuition

The key insight is recognizing that we have two levels of dependencies to handle: dependencies between individual items and implicit dependencies between groups.

When items belong to the same group, they must appear consecutively in the final ordering. This means if item `A` from group 1 must come before item `B` from group 2, then **all items** in group 1 must come before **all items** in group 2. This creates a group-level dependency.

Think of it like organizing tasks in a project where tasks are grouped by teams. If Task A (from Team 1) must be done before Task B (from Team 2), then Team 1 must complete all their tasks before Team 2 starts any of theirs.

This leads us to a two-layer [graph](/problems/graph_intro) structure:
1. **Group Graph**: Where nodes are groups and edges represent dependencies between groups
2. **Item Graph**: Where nodes are items and edges represent dependencies between items within the same group

For example, if item `i` (in group `G1`) must come after item `j` (in group `G2`), we need:
- An edge from `G2` to `G1` in the group [graph](/problems/graph_intro) (group `G2` must be processed before group `G1`)
- If both items are in the same group, we add an edge from `j` to `i` in the item graph

The solution approach becomes clear:
1. First, determine the order of groups using [topological sort](/problems/topo_intro) on the group [graph](/problems/graph_intro)
2. Then, for each group in that order, determine the internal ordering of its items using topological sort

Why assign new groups to items with `group[i] = -1`? This simplifies our logic - instead of handling ungrouped items as special cases, we treat each as its own single-item group. This uniform treatment makes the algorithm cleaner.

The beauty of this approach is that it decomposes a complex constraint problem into two simpler topological sorting problems, each handling one type of dependency.

**Learn more about [Depth-First Search](/problems/dfs_intro), [Breadth-First Search](/problems/bfs_intro), [Graph](/problems/graph_intro) and [Topological Sort](/problems/topo_intro) patterns.**

## Solution Approach

The implementation follows our two-layer topological sorting strategy. Let's walk through the key steps:

**Step 1: Assign Groups to Ungrouped Items**

First, we handle items with `group[i] = -1` by assigning each a unique group ID starting from `m`:
```python
idx = m
for i, g in enumerate(group):
    if g == -1:
        group[i] = idx
        idx += 1
```
This ensures every item belongs to exactly one group, simplifying our logic.

**Step 2: Build Group Membership Mapping**

We create `group_items` to track which items belong to each group:
```python
group_items = [[] for _ in range(n + m)]
for i, g in enumerate(group):
    group_items[group[i]].append(i)
```
Note we allocate `n + m` slots since we might create up to `n` new groups for ungrouped items.

**Step 3: Build Two Graphs with In-Degree Tracking**

For each dependency in `beforeItems[i]`, we determine whether it creates an item-level or group-level edge:
```python
for i, gi in enumerate(group):
    for j in beforeItems[i]:
        gj = group[j]
        if gi == gj:  # Same group - item-level dependency
            item_degree[i] += 1
            item_graph[j].append(i)
        else:  # Different groups - group-level dependency
            group_degree[gi] += 1
            group_graph[gj].append(gi)
```

- **Same group** (`gi == gj`): Add edge `j → i` in item [graph](/problems/graph_intro)
- **Different groups**: Add edge `gj → gi` in group graph (group of `j` must come before group of `i`)

**Step 4: [Topological Sort](/problems/topo_intro) Using Kahn's Algorithm**

The helper function `topo_sort` implements Kahn's algorithm:
```python
def topo_sort(degree, [graph](/problems/graph_intro), items):
    q = deque(i for i in items if degree[i] == 0)
    res = []
    while q:
        i = q.popleft()
        res.append(i)
        for j in graph[i]:
            degree[j] -= 1
            if degree[j] == 0:
                q.append(j)
    return res if len(res) == len(items) else []
```

This BFS-based approach:
1. Starts with all nodes having in-degree 0 (no prerequisites)
2. Processes each node and reduces in-degrees of its neighbors
3. Returns empty list if cycle detected (when `len(res) != len(items)`)

**Step 5: Two-Level Sorting**

First, sort groups:
```python
group_order = topo_sort(group_degree, group_graph, range(n + m))
if not group_order:
    return []  # Cycle in group dependencies
```

Then, for each group in order, sort its items:
```python
for gi in group_order:
    items = group_items[gi]
    item_order = topo_sort(item_degree, item_graph, items)
    if len(items) != len(item_order):
        return []  # Cycle in item dependencies within group
    ans.extend(item_order)
```

The algorithm ensures:
- Groups are processed in valid dependency order
- Within each group, items are arranged respecting their dependencies
- Cycles at any level result in returning an empty list

Time complexity: `O(n + m + E)` where `E` is the total number of dependencies.
Space complexity: `O(n + m + E)` for storing graphs and in-degree arrays.

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

**Given:**
- `n = 5` items (0, 1, 2, 3, 4)
- `m = 2` groups (0, 1)
- `group = [1, -1, 0, 1, -1]`
  - Items 0, 3 belong to group 1
  - Item 2 belongs to group 0
  - Items 1, 4 have no group (-1)
- `beforeItems = [[], [0], [], [2], [1, 3]]`
  - Item 1 must come after item 0
  - Item 3 must come after item 2
  - Item 4 must come after items 1 and 3

**Step 1: Assign groups to ungrouped items**
- Item 1 gets group 2 (first available after m=2)
- Item 4 gets group 3
- Updated `group = [1, 2, 0, 1, 3]`

**Step 2: Build group membership**
```
group_items:
- Group 0: [2]
- Group 1: [0, 3]
- Group 2: [1]
- Group 3: [4]
```

**Step 3: Build graphs**

For each item's dependencies:
- Item 1 depends on item 0:
  - Item 0 (group 1) → Item 1 (group 2): Different groups
  - Add edge: group 1 → group 2 in group graph

- Item 3 depends on item 2:
  - Item 2 (group 0) → Item 3 (group 1): Different groups
  - Add edge: group 0 → group 1 in group graph

- Item 4 depends on items 1 and 3:
  - Item 1 (group 2) → Item 4 (group 3): Different groups
  - Add edge: group 2 → group 3 in group graph
  - Item 3 (group 1) → Item 4 (group 3): Different groups
  - Add edge: group 1 → group 3 in group graph

**Resulting graphs:**
```
Group graph:
- Group 0 → Group 1
- Group 1 → Group 2
- Group 1 → Group 3
- Group 2 → Group 3

Group in-degrees: [0, 1, 1, 2]

Item graph (only same-group edges): Empty in this example
Item in-degrees: [0, 0, 0, 0, 0]
```

**Step 4: Topological sort of groups**
- Start with group 0 (in-degree 0)
- Process group 0: reduces group 1's in-degree to 0
- Process group 1: reduces group 2's in-degree to 0, group 3's to 1
- Process group 2: reduces group 3's in-degree to 0
- Process group 3

Group order: `[0, 1, 2, 3]`

**Step 5: Process each group and sort its items**
- Group 0: items [2], no internal dependencies → output: `[2]`
- Group 1: items [0, 3], no internal dependencies → output: `[0, 3]`
- Group 2: items [1], no internal dependencies → output: `[1]`
- Group 3: items [4], no internal dependencies → output: `[4]`

**Final Result:** `[2, 0, 3, 1, 4]`

Let's verify:
- ✓ Items in same group are consecutive: Group 1 items (0, 3) are together
- ✓ Dependencies satisfied:
  - Item 0 before item 1: 0 appears at index 1, 1 at index 3 ✓
  - Item 2 before item 3: 2 appears at index 0, 3 at index 2 ✓
  - Items 1, 3 before item 4: 1 at index 3, 3 at index 2, 4 at index 4 ✓

The algorithm successfully found a valid ordering by first determining the group order (0→1→2→3) and then placing each group's items consecutively in that order.

## Python Solution

```python
class Solution:
    def sortItems(
        self, n: int, m: int, group: List[int], beforeItems: List[List[int]]
    ) -> List[int]:
        """
        Sort items respecting both group constraints and dependency constraints.
      
        Args:
            n: Number of items
            m: Number of groups (excluding items with no group)
            group: List where group[i] is the group of item i (-1 means no group)
            beforeItems: List where beforeItems[i] contains items that must come before item i
      
        Returns:
            A valid ordering of items, or empty list if impossible
        """
      
        def topological_sort(in_degree: List[int], adjacency_list: List[List[int]], nodes: List[int]) -> List[int]:
            """
            Perform topological sort on given nodes using Kahn's algorithm.
          
            Args:
                in_degree: In-degree count for each node
                adjacency_list: Adjacency list representation of the graph
                nodes: List of nodes to sort
          
            Returns:
                Sorted order of nodes, or empty list if cycle detected
            """
            # Initialize queue with nodes having zero in-degree
            queue = deque(node for node in nodes if in_degree[node] == 0)
            sorted_result = []
          
            while queue:
                current_node = queue.popleft()
                sorted_result.append(current_node)
              
                # Process all neighbors of current node
                for neighbor in adjacency_list[current_node]:
                    in_degree[neighbor] -= 1
                    if in_degree[neighbor] == 0:
                        queue.append(neighbor)
          
            # Return result only if all nodes were processed (no cycle)
            return sorted_result if len(sorted_result) == len(nodes) else []
      
        # Assign unique group IDs to items without a group (-1)
        next_group_id = m
        groups_to_items = [[] for _ in range(n + m)]  # Map group ID to its items
      
        for item_id, group_id in enumerate(group):
            if group_id == -1:
                # Assign new group ID for ungrouped item
                group[item_id] = next_group_id
                next_group_id += 1
            groups_to_items[group[item_id]].append(item_id)
      
        # Initialize degree and graph structures for both items and groups
        item_in_degree = [0] * n
        group_in_degree = [0] * (n + m)
        item_adjacency = [[] for _ in range(n)]
        group_adjacency = [[] for _ in range(n + m)]
      
        # Build dependency graphs
        for current_item, current_group in enumerate(group):
            for prerequisite_item in beforeItems[current_item]:
                prerequisite_group = group[prerequisite_item]
              
                if current_group == prerequisite_group:
                    # Same group: add item-level dependency
                    item_in_degree[current_item] += 1
                    item_adjacency[prerequisite_item].append(current_item)
                else:
                    # Different groups: add group-level dependency
                    group_in_degree[current_group] += 1
                    group_adjacency[prerequisite_group].append(current_group)
      
        # Sort groups topologically
        sorted_groups = topological_sort(group_in_degree, group_adjacency, list(range(n + m)))
        if not sorted_groups:
            return []  # Cycle detected in group dependencies
      
        # Process each group and sort items within it
        final_result = []
        for group_id in sorted_groups:
            items_in_group = groups_to_items[group_id]
            sorted_items = topological_sort(item_in_degree, item_adjacency, items_in_group)
          
            # Check if all items in the group were successfully sorted
            if len(items_in_group) != len(sorted_items):
                return []  # Cycle detected in item dependencies
          
            final_result.extend(sorted_items)
      
        return final_result
```

## Time and Space Complexity

The time complexity is `O(n + m + E)`, where `n` is the number of items, `m` is the initial number of groups, and `E` is the total number of edges (dependencies) in the `beforeItems` list.

Breaking down the time complexity:
- Creating `group_items` and assigning isolated items to new groups: `O(n)`
- Building the dependency graphs (`item_graph` and `group_graph`): `O(E)` where we iterate through all items and their dependencies
- Topological sort for groups: `O(n + m + E_g)` where `E_g` is the number of group-level edges
- Topological sort for items within each group: `O(n + E_i)` where `E_i` is the number of item-level edges
- Since `E_g + E_i ≤ E`, the total is `O(n + m + E)`

However, if we consider that `E` can be at most `O(n²)` in the worst case (each item can depend on all other items), and typically `m ≤ n`, the time complexity simplifies to `O(n²)` in the worst case.

The space complexity is `O(n + m + E)`:
- `group_items`: `O(n + m)` lists
- `item_degree` and `item_graph`: `O(n)` and `O(n + E_i)`
- `group_degree` and `group_graph`: `O(n + m)` and `O(n + m + E_g)`
- Queue in topological sort: `O(n + m)` at most
- Total space: `O(n + m + E)`

Note: The reference answer states `O(n + m)` for both complexities, which appears to be considering only the structural overhead without accounting for the edges in the dependency graph. This would be accurate only if the number of dependencies is bounded by `O(n + m)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Duplicate Edges in Group Dependencies**

**The Pitfall:** When building the group dependency graph, if items A and B are in different groups and there are multiple dependencies between items in their respective groups, we might add the same group-level edge multiple times. This leads to incorrect in-degree counts.

**Example Scenario:**
- Item 0 and 1 are in group G1
- Item 2 and 3 are in group G2  
- Dependencies: item 2 depends on both items 0 and 1

Without proper handling, we'd add the edge G1→G2 twice, making `group_in_degree[G2] = 2` instead of 1. This breaks the topological sort since G2 will never reach in-degree 0 even after processing G1.

**Solution:** Track visited group pairs to avoid duplicate edges:

```python
# Build dependency graphs with duplicate edge prevention
visited_group_edges = set()

for current_item, current_group in enumerate(group):
    for prerequisite_item in beforeItems[current_item]:
        prerequisite_group = group[prerequisite_item]
      
        if current_group == prerequisite_group:
            # Same group: add item-level dependency
            item_in_degree[current_item] += 1
            item_adjacency[prerequisite_item].append(current_item)
        else:
            # Different groups: add group-level dependency only if not seen
            edge = (prerequisite_group, current_group)
            if edge not in visited_group_edges:
                visited_group_edges.add(edge)
                group_in_degree[current_group] += 1
                group_adjacency[prerequisite_group].append(current_group)
```

### 2. **Self-Loop Detection in Group Dependencies**

**The Pitfall:** The code doesn't explicitly check for self-loops at the group level. If item dependencies create a circular dependency between groups (e.g., item in G1 depends on item in G2, and item in G2 depends on item in G1), this creates an impossible constraint.

**Solution:** Add explicit self-loop detection or rely on the topological sort to catch cycles:

```python
# When building group dependencies, check for self-loops
if current_group == prerequisite_group:
    # Handle same-group dependency
    ...
elif (prerequisite_group, current_group) in visited_group_edges:
    # Skip duplicate edge
    continue
elif (current_group, prerequisite_group) in visited_group_edges:
    # Detected bidirectional dependency - will be caught by topo sort
    # But could return early here for efficiency
    pass
```

### 3. **Incorrect Group Count Assumption**

**The Pitfall:** Allocating exactly `n + m` slots for groups assumes worst case where all `n` items have `group[i] = -1`. However, if some items already have groups assigned, we don't need all `n` extra groups.

**Solution:** While the current approach works (unused groups are harmless), a more memory-efficient approach would be:

```python
# Count actual number of ungrouped items first
ungrouped_count = sum(1 for g in group if g == -1)
total_groups = m + ungrouped_count
groups_to_items = [[] for _ in range(total_groups)]
```

### 4. **Missing Validation of Input Constraints**

**The Pitfall:** The code doesn't validate that items in `beforeItems[i]` are valid item indices (0 to n-1), which could cause index errors.

**Solution:** Add input validation:

```python
# Validate dependencies reference valid items
for i, deps in enumerate(beforeItems):
    for dep in deps:
        if not (0 <= dep < n):
            return []  # Invalid dependency
        if dep == i:
            return []  # Self-dependency
```

