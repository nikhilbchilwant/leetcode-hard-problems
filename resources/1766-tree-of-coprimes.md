# 1766. Tree of Coprimes

> Source: https://algo.monster/liteproblems/1766

## Problem Statement

You are given a [tree](/problems/tree_intro) with `n` nodes numbered from `0` to `n - 1`, where node `0` is the root. The tree is represented by:
- An array `nums` where `nums[i]` is the value of node `i`
- A 2D array `edges` where each `edges[j] = [u, v]` represents an undirected edge between nodes `u` and `v`

Two numbers `x` and `y` are **coprime** if their greatest common divisor (GCD) equals 1, meaning `gcd(x, y) == 1`.

An **ancestor** of a node `i` is any node that lies on the shortest path from node `i` to the root (excluding node `i` itself).

Your task is to find, for each node `i`, the **closest ancestor** whose value is coprime with `nums[i]`. The closest ancestor means the one with the minimum distance from node `i` when traveling up toward the root.

Return an array `ans` of size `n` where:
- `ans[i]` is the node number of the closest ancestor to node `i` such that `nums[i]` and `nums[ans[i]]` are coprime
- `ans[i] = -1` if no such coprime ancestor exists

For example, if we have a path from node `i` to the root as: `i → parent → grandparent → ... → root`, we check each ancestor (parent, grandparent, etc.) and find the first one whose value is coprime with `nums[i]`.

## Intuition

When we need to find the closest coprime ancestor for each node, we need to think about what information we have access to at any given point during [tree](/problems/tree_intro) traversal.

The key insight is that when we're visiting a node during DFS traversal, we have a complete path from the root to that node - this path contains all the ancestors of the current node. As we traverse deeper into the [tree](/problems/tree_intro), we build up this ancestor path, and when we backtrack, we remove nodes from this path.

Since `nums[i]` values are limited to the range `[1, 50]`, we can use this constraint. Instead of checking every single ancestor for each node (which could be expensive in a deep [tree](/problems/tree_intro)), we can maintain separate stacks for each possible value (1 through 50). Each stack tracks all ancestors that have that particular value, along with their depth in the tree.

Here's the clever part: when we're at node `i` with value `nums[i]`, we don't need to check all ancestors. We only need to check ancestors whose values are coprime with `nums[i]`. Since there are only 50 possible values, we can precompute which values are coprime with each other. For example, if `nums[i] = 6`, we know we only need to check ancestors with values like 1, 5, 7, 11, 13, etc. (values coprime to 6).

For each coprime value, we look at the top of its corresponding stack (which represents the most recent ancestor with that value). Among all these coprime ancestors, we pick the one with the maximum depth - this gives us the closest coprime ancestor.

The backtracking nature of DFS ensures that when we finish processing a subtree and return to process another branch, we remove the current node from its value stack, maintaining the correct ancestor information for each branch of the [tree](/problems/tree_intro).

This approach efficiently combines:
- DFS traversal to explore the [tree](/problems/tree_intro) structure
- Stack-based tracking of ancestors by their values
- Precomputed coprime relationships to avoid redundant GCD calculations
- Backtracking to maintain correct ancestor paths for different branches

**Learn more about [Tree](/problems/tree_intro), [Depth-First Search](/problems/dfs_intro) and [Math](/problems/math-basics) patterns.**

## Solution Approach

The implementation follows the intuition we developed, using DFS with backtracking and using the constraint that values are in range `[1, 50]`.

**Step 1: Preprocess Coprime Relationships**

Since values are limited to `[1, 50]`, we precompute all coprime pairs:
```python
f = defaultdict(list)
for i in range(1, 51):
    for j in range(1, 51):
        if gcd(i, j) == 1:
            f[i].append(j)
```
This creates a dictionary `f` where `f[i]` contains all numbers from 1 to 50 that are coprime with `i`. This preprocessing avoids repeated GCD calculations during traversal.

**Step 2: Build the [Tree](/problems/tree_intro) Structure**

Convert the edge list into an adjacency list for efficient traversal:
```python
g = defaultdict(list)
for u, v in edges:
    g[u].append(v)
    g[v].append(u)
```

**Step 3: Initialize Stack Structure**

Create stacks for each possible value to track ancestors:
```python
stks = defaultdict(list)
```
Each `stks[v]` will maintain a stack of `(node_id, depth)` pairs for all ancestors with value `v`.

**Step 4: DFS Traversal with Backtracking**

The core DFS function processes each node:

```python
def dfs(i, fa, depth):
    # Find closest coprime ancestor
    t = k = -1
    for v in f[nums[i]]:  # Check all coprime values
        stk = stks[v]
        if stk and stk[-1][1] > k:  # Get the one with max depth
            t, k = stk[-1]
    ans[i] = t
  
    # Process children
    for j in g[i]:
        if j != fa:  # Avoid going back to parent
            stks[nums[i]].append((i, depth))  # Add current node to stack
            dfs(j, i, depth + 1)
            stks[nums[i]].pop()  # Backtrack: remove from stack
```

**Key Algorithm Details:**

1. **Finding Closest Coprime Ancestor**: For node `i`, we iterate through all values coprime to `nums[i]` using our precomputed list `f[nums[i]]`. For each coprime value `v`, we check the top of `stks[v]` which gives us the most recent (closest) ancestor with that value.

2. **Depth Tracking**: We compare depths (`stk[-1][1] > k`) to find the ancestor with maximum depth among all coprime ancestors, which corresponds to the closest one.

3. **Stack Management**: Before recursing into a child, we push the current node onto `stks[nums[i]]`. After processing the entire subtree rooted at that child, we pop it off (backtracking), ensuring the stack correctly represents ancestors for other branches.

4. **Parent Tracking**: We pass the parent node `fa` to avoid revisiting it, since the [tree](/problems/tree_intro) is undirected.

The algorithm has time complexity of `O(n × C)` where `n` is the number of nodes and `C` is the constant factor from checking coprime values (at most 50). Space complexity is `O(n + C²)` for storing the [tree](/problems/tree_intro), stacks, and precomputed coprime relationships.

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

Consider a tree with 5 nodes where:
- `nums = [2, 3, 3, 2, 5]` (values at each node)
- `edges = [[0,1], [1,2], [1,3], [0,4]]`

This creates the following tree structure:
```
      0(2)
     /    \
   1(3)   4(5)
   /  \
 2(3) 3(2)
```

**Step 1: Precompute Coprime Relationships**

We build our coprime dictionary `f`. For the values in our tree:
- `f[2] = [1, 3, 5, 7, 9, 11, ...]` (all odd numbers)
- `f[3] = [1, 2, 4, 5, 7, 8, 10, 11, ...]` (all numbers not divisible by 3)
- `f[5] = [1, 2, 3, 4, 6, 7, 8, 9, 11, ...]` (all numbers not divisible by 5)

**Step 2: DFS Traversal**

Starting from root node 0, let's trace through the DFS:

**Visit Node 0 (value=2, depth=0):**
- No ancestors exist, so `ans[0] = -1`
- Push `(0, 0)` to `stks[2]`
- Current stacks: `stks[2] = [(0, 0)]`

**Visit Node 1 (value=3, depth=1):**
- Check coprime ancestors: Look for values in `f[3]` = includes 2
- Check `stks[2]`: has `(0, 0)` at top
- Found coprime ancestor: node 0 at depth 0
- `ans[1] = 0`
- Push `(1, 1)` to `stks[3]`
- Current stacks: `stks[2] = [(0, 0)]`, `stks[3] = [(1, 1)]`

**Visit Node 2 (value=3, depth=2):**
- Check coprime ancestors: Look for values in `f[3]` = includes 2
- Check `stks[2]`: has `(0, 0)` at top
- Check `stks[3]`: skip (3 is not coprime with 3)
- Found coprime ancestor: node 0 at depth 0
- `ans[2] = 0`
- No children, backtrack

**Visit Node 3 (value=2, depth=2):**
- Check coprime ancestors: Look for values in `f[2]` = includes 3
- Check `stks[3]`: has `(1, 1)` at top
- Check `stks[2]`: skip (2 is not coprime with 2)
- Found coprime ancestor: node 1 at depth 1
- `ans[3] = 1`
- No children, backtrack

**Backtrack to Node 1, then to Node 0**
- Pop `(1, 1)` from `stks[3]`
- Current stacks: `stks[2] = [(0, 0)]`

**Visit Node 4 (value=5, depth=1):**
- Check coprime ancestors: Look for values in `f[5]` = includes 2
- Check `stks[2]`: has `(0, 0)` at top
- Found coprime ancestor: node 0 at depth 0
- `ans[4] = 0`
- No children, backtrack

**Final Result:** `ans = [-1, 0, 0, 1, 0]`

This walkthrough demonstrates:
1. How we maintain stacks for each value to track ancestors
2. How we only check stacks for coprime values (avoiding unnecessary GCD calculations)
3. How backtracking removes nodes from stacks to maintain correct ancestor information
4. How we select the closest ancestor by comparing depths among all coprime candidates

## Python Solution

```python
class Solution:
    def getCoprimes(self, nums: List[int], edges: List[List[int]]) -> List[int]:
        from collections import defaultdict
        from math import gcd
      
        # Build adjacency list for the tree
        adjacency_list = defaultdict(list)
        for u, v in edges:
            adjacency_list[u].append(v)
            adjacency_list[v].append(u)
      
        # Precompute coprime relationships for all values from 1 to 50
        # coprimes[i] contains all numbers from 1 to 50 that are coprime with i
        coprimes = defaultdict(list)
        for i in range(1, 51):
            for j in range(1, 51):
                if gcd(i, j) == 1:
                    coprimes[i].append(j)
      
        # Stack for each value to track ancestors with that value
        # ancestor_stacks[value] = [(node_index, depth), ...]
        ancestor_stacks = defaultdict(list)
      
        # Result array to store the answer for each node
        result = [-1] * len(nums)
      
        def dfs(current_node: int, parent: int, depth: int) -> None:
            """
            Depth-first search to find the nearest coprime ancestor for each node.
          
            Args:
                current_node: Current node being processed
                parent: Parent of the current node (-1 for root)
                depth: Current depth in the tree
            """
            # Find the deepest ancestor with a value coprime to current node's value
            nearest_coprime_ancestor = -1
            max_depth = -1
          
            # Check all values that are coprime with current node's value
            for coprime_value in coprimes[nums[current_node]]:
                stack = ancestor_stacks[coprime_value]
                # If there are ancestors with this coprime value
                if stack and stack[-1][1] > max_depth:
                    # Update to the deepest (nearest) coprime ancestor
                    nearest_coprime_ancestor = stack[-1][0]
                    max_depth = stack[-1][1]
          
            # Store the result for current node
            result[current_node] = nearest_coprime_ancestor
          
            # Process all children of current node
            for child in adjacency_list[current_node]:
                if child != parent:  # Avoid going back to parent
                    # Add current node to the ancestor stack for its value
                    ancestor_stacks[nums[current_node]].append((current_node, depth))
                  
                    # Recursively process the child
                    dfs(child, current_node, depth + 1)
                  
                    # Remove current node from ancestor stack (backtrack)
                    ancestor_stacks[nums[current_node]].pop()
      
        # Start DFS from node 0 (root) with no parent (-1) at depth 0
        dfs(0, -1, 0)
      
        return result
```

## Time and Space Complexity

**Time Complexity:** `O(n × M)`

The algorithm performs a DFS traversal of the tree. For each node visited:
- We iterate through all coprime values of `nums[i]` to find the ancestor with maximum depth. Since each number has at most `M` coprime values (where `M = 50`), this takes `O(M)` time.
- The DFS visits each node exactly once, so we have `n` nodes total.
- Therefore, the overall time complexity is `O(n × M)`.

**Space Complexity:** `O(M² + n)`

The space complexity consists of several components:
- The graph `g` stores all edges, using `O(n)` space (since it's a tree with `n-1` edges).
- The coprime mapping `f` stores coprime relationships for all pairs of numbers from 1 to 50. Each number can have at most `M` coprimes, and we have `M` numbers, resulting in `O(M²)` space.
- The stacks `stks` store ancestors along the current path. In the worst case (a linear tree), the maximum depth is `n`, but since we only store ancestors with specific values (at most `M` different values), and each value appears at most `n` times in the path, the space is bounded by `O(n)`.
- The answer array `ans` uses `O(n)` space.
- The recursion stack depth is at most `O(n)` in the worst case.

Combining all components, the total space complexity is `O(M² + n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. Incorrectly Managing the Ancestor Stack Timing

**The Pitfall:**
A critical mistake is adding the current node to the ancestor stack BEFORE checking for coprime ancestors, rather than adding it only when recursing to children. This would cause a node to potentially find itself as its own ancestor if its value is coprime with itself (which happens when the value is 1).

**Incorrect Implementation:**
```python
def dfs(current_node, parent, depth):
    # WRONG: Adding to stack before finding coprime ancestors
    ancestor_stacks[nums[current_node]].append((current_node, depth))
  
    # Finding coprime ancestors
    nearest_coprime_ancestor = -1
    max_depth = -1
    for coprime_value in coprimes[nums[current_node]]:
        stack = ancestor_stacks[coprime_value]
        if stack and stack[-1][1] > max_depth:
            nearest_coprime_ancestor = stack[-1][0]
            max_depth = stack[-1][1]
  
    result[current_node] = nearest_coprime_ancestor
  
    # Process children
    for child in adjacency_list[current_node]:
        if child != parent:
            dfs(child, current_node, depth + 1)
  
    # Backtrack
    ancestor_stacks[nums[current_node]].pop()
```

**Why This Fails:**
- When `nums[current_node] = 1`, since `gcd(1, 1) = 1`, the node would find itself as a coprime "ancestor"
- This violates the problem constraint that ancestors exclude the node itself
- Even for other values, the timing is conceptually wrong - a node only becomes an ancestor when we traverse to its descendants

**The Solution:**
Add the current node to the ancestor stack only when recursing into each child, and remove it immediately after that child's subtree is processed:

```python
def dfs(current_node, parent, depth):
    # First, find coprime ancestors (current node is NOT in any stack yet)
    nearest_coprime_ancestor = -1
    max_depth = -1
    for coprime_value in coprimes[nums[current_node]]:
        stack = ancestor_stacks[coprime_value]
        if stack and stack[-1][1] > max_depth:
            nearest_coprime_ancestor = stack[-1][0]
            max_depth = stack[-1][1]
  
    result[current_node] = nearest_coprime_ancestor
  
    # Process each child
    for child in adjacency_list[current_node]:
        if child != parent:
            # Add current node ONLY when going to process child
            ancestor_stacks[nums[current_node]].append((current_node, depth))
            dfs(child, current_node, depth + 1)
            # Remove immediately after child is processed
            ancestor_stacks[nums[current_node]].pop()
```

### 2. Not Handling the Root Node Specially

**The Pitfall:**
Another subtle issue occurs if you add the root node to the ancestor stack outside the DFS function, thinking it should always be available as an ancestor:

```python
# WRONG: Pre-adding root to stack
ancestor_stacks[nums[0]].append((0, 0))
dfs(0, -1, 0)
# Root remains in stack after DFS completes
```

**Why This Fails:**
- The root node itself should never find any ancestors (it should always get -1)
- If root's value is 1 (coprime with itself), it would incorrectly find itself
- The stack state becomes inconsistent after DFS completes

**The Solution:**
Let the DFS handle all stack operations uniformly. The root will naturally get -1 as its answer since no ancestors exist when it's processed, and it will only be added to the stack when processing its children.

