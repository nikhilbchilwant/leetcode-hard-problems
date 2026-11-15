# 2872. Maximum Number of K-Divisible Components

> Source: https://algo.monster/liteproblems/2872

## Problem Statement

You are given an undirected [tree](/problems/tree_intro) with `n` nodes labeled from `0` to `n - 1`. The tree structure is defined by a 2D array `edges` of length `n - 1`, where each `edges[i] = [ai, bi]` represents an edge connecting nodes `ai` and `bi`.

Each node has an associated value given in the array `values`, where `values[i]` is the value of node `i`. You are also given an integer `k`.

Your task is to find a way to split the [tree](/problems/tree_intro) into multiple connected components by removing some edges (possibly none). For the split to be **valid**, each resulting connected component must have a sum of node values that is divisible by `k`.

The **value of a connected component** is defined as the sum of all node values within that component.

You need to return the **maximum number of components** you can create through any valid split.

For example, if you have a [tree](/problems/tree_intro) with nodes having values that sum to 12 and `k = 3`, you might be able to split it into components where each component's sum is divisible by 3 (like components with sums 3, 3, and 6, giving you 3 components total).

## Intuition

The key observation is that if we cut an edge in the [tree](/problems/tree_intro), we create two separate components. For both components to have sums divisible by `k`, the subtree "below" the cut must have a sum divisible by `k`. Since the total sum is divisible by `k` (given by the problem), if one part is divisible by `k`, the remaining part will automatically be divisible by `k` too.

Think of it like cutting a cake that weighs 12 pounds into pieces where each piece must weigh a multiple of 3 pounds. If you cut off a 3-pound piece, the remaining 9 pounds is also divisible by 3.

This leads us to a greedy insight: whenever we find a subtree whose sum is divisible by `k`, we can immediately "cut it off" as a separate component. We don't need to keep it attached to its parent because:
1. The subtree itself forms a valid component (sum divisible by `k`)
2. The remaining [tree](/problems/tree_intro) will still have a sum divisible by `k`

Using DFS, we can traverse the [tree](/problems/tree_intro) from bottom to top (post-order traversal). At each node, we:
1. Calculate the sum of the current subtree (node value + sum of all child subtrees)
2. If this sum is divisible by `k`, we've found a valid component - increment our answer
3. Return the sum to the parent (this represents the "weight" of the edge to the parent)

The beauty of this approach is that when a subtree sum is divisible by `k`, we conceptually "cut" it off by counting it as a component. From the parent's perspective, this subtree contributes 0 to the parent's sum (since `sum % k = 0`), effectively simulating the removal of that edge.

This greedy strategy works because we're maximizing components - any valid cut we can make should be made, as it increases our component count without affecting the validity of the remaining [tree](/problems/tree_intro).

**Learn more about [Tree](/problems/tree_intro) and [Depth-First Search](/problems/dfs_intro) patterns.**

## Solution Approach

The implementation uses DFS to traverse the [tree](/problems/tree_intro) and calculate subtree sums. Here's how the solution works:

**Data Structure Setup:**
- Build an adjacency list `g` to represent the [tree](/problems/tree_intro), where `g[i]` contains all neighbors of node `i`
- Since the tree is undirected, we add edges in both directions

**DFS Function:**
The core logic is in the `dfs(i, fa)` function where:
- `i` is the current node being visited
- `fa` is the parent node (to avoid revisiting in an undirected [tree](/problems/tree_intro))

**Algorithm Steps:**

1. **Initialize the sum** for the current subtree with the current node's value: `s = values[i]`

2. **Recursively visit all children:** For each neighbor `j` of node `i`:
   - Skip if `j` is the parent (to avoid going back up the [tree](/problems/tree_intro))
   - Recursively call `dfs(j, i)` to get the sum of the subtree rooted at `j`
   - Add this subtree sum to our current sum: `s += dfs(j, i)`

3. **Check divisibility:** After collecting all child subtree sums:
   - If `s % k == 0`, we've found a valid component
   - Increment the answer counter: `ans += s % k == 0`
   - This effectively "cuts" this subtree from its parent

4. **Return the sum:** Return `s` to the parent call
   - If `s % k == 0`, this subtree conceptually contributes 0 to its parent's sum calculation
   - This simulates cutting the edge between this node and its parent

**Starting the traversal:**
- Call `dfs(0, -1)` starting from node 0 with -1 as the parent (indicating no parent)
- The `ans` variable accumulates the total number of valid components found

**Why this works:**
When we find a subtree with sum divisible by `k`, we count it as a separate component. The mathematical property that makes this work is that if the total sum is divisible by `k` and we remove a portion divisible by `k`, the remainder is also divisible by `k`. This allows us to greedily cut off valid subtrees without worrying about the rest of the [tree](/problems/tree_intro) becoming invalid.

The time complexity is `O(n)` as we visit each node exactly once, and the space complexity is `O(n)` for the adjacency list and recursion stack.

## Example Walkthrough

Let's walk through a concrete example to illustrate how the solution works.

**Example Tree:**
```
    Node values: [6, 2, 3, 3, 2]
    Edges: [[0,1], [0,2], [2,3], [2,4]]
    k = 3
  
    Tree structure:
           0(6)
          / \
        1(2) 2(3)
            / \
          3(3) 4(2)
```

**Step-by-step DFS traversal:**

1. **Start at node 0** with parent -1
   - Initialize sum: `s = 6`
   - Visit children: nodes 1 and 2

2. **Visit node 1** (child of 0)
   - Initialize sum: `s = 2`
   - No children to visit
   - Check: `2 % 3 ≠ 0`, so don't increment answer
   - Return sum = 2 to parent (node 0)

3. **Visit node 2** (child of 0)
   - Initialize sum: `s = 3`
   - Visit children: nodes 3 and 4

4. **Visit node 3** (child of 2)
   - Initialize sum: `s = 3`
   - No children to visit
   - Check: `3 % 3 = 0` ✓ Found a valid component!
   - Increment answer: `ans = 1`
   - Return sum = 3 to parent (node 2)

5. **Visit node 4** (child of 2)
   - Initialize sum: `s = 2`
   - No children to visit
   - Check: `2 % 3 ≠ 0`, so don't increment answer
   - Return sum = 2 to parent (node 2)

6. **Back at node 2**
   - Current sum: `s = 3 + 3 + 2 = 8`
   - Check: `8 % 3 ≠ 0`, so don't increment answer
   - Return sum = 8 to parent (node 0)

7. **Back at node 0**
   - Current sum: `s = 6 + 2 + 8 = 16`
   - Check: `16 % 3 ≠ 0`, so don't increment answer
   - Return sum = 16

**Result:** Maximum components = 1

In this example, we found only one valid split: we can cut off the subtree rooted at node 3 (which has sum 3, divisible by k=3). The remaining tree would have sum 13, which is not divisible by 3, so this entire example actually shows an invalid case where the total sum (16) is not divisible by k.

Let's try a valid example where total sum is divisible by k:

**Better Example:**
```
    Node values: [3, 3, 3]
    Edges: [[0,1], [0,2]]
    k = 3
  
    Tree:     0(3)
             / \
           1(3) 2(3)
```

**DFS traversal:**
1. Visit node 1: sum = 3, `3 % 3 = 0` ✓, ans = 1
2. Visit node 2: sum = 3, `3 % 3 = 0` ✓, ans = 2  
3. Back at node 0: sum = 3 + 0 + 0 = 3 (children contribute 0 since they were cut), `3 % 3 = 0` ✓, ans = 3

**Result:** Maximum components = 3 (each node becomes its own component)

This demonstrates how when we find a subtree divisible by k, it effectively contributes 0 to its parent's sum, simulating the edge removal.

## Python Solution

```python
class Solution:
    def maxKDivisibleComponents(
        self, n: int, edges: List[List[int]], values: List[int], k: int
    ) -> int:
        def dfs(node: int, parent: int) -> int:
            """
            Perform DFS to calculate subtree sums and count valid components.
          
            Args:
                node: Current node being visited
                parent: Parent node to avoid revisiting
          
            Returns:
                Sum of values in the subtree rooted at current node
            """
            # Start with the value of current node
            subtree_sum = values[node]
          
            # Visit all adjacent nodes except the parent
            for neighbor in adjacency_list[node]:
                if neighbor != parent:
                    # Add the sum of child subtree to current subtree sum
                    subtree_sum += dfs(neighbor, node)
          
            # If subtree sum is divisible by k, we can cut this as a separate component
            if subtree_sum % k == 0:
                nonlocal component_count
                component_count += 1
          
            return subtree_sum
      
        # Build adjacency list representation of the tree
        adjacency_list = [[] for _ in range(n)]
        for node_a, node_b in edges:
            adjacency_list[node_a].append(node_b)
            adjacency_list[node_b].append(node_a)
      
        # Initialize counter for valid components
        component_count = 0
      
        # Start DFS from node 0 with -1 as parent (indicating no parent)
        dfs(0, -1)
      
        return component_count
```

## Time and Space Complexity

**Time Complexity:** `O(n)`

The algorithm performs a depth-first search (DFS) traversal of the tree. Each node is visited exactly once during the DFS traversal. For each node, we:
- Process its value (constant time)
- Iterate through its adjacent nodes
- Perform modulo operation and increment counter (constant time)

Since this is a tree with `n` nodes and `n-1` edges, the total number of edges traversed is `2(n-1)` (each edge is traversed twice, once from each direction). Therefore, the overall time complexity is `O(n + 2(n-1)) = O(n)`.

**Space Complexity:** `O(n)`

The space complexity consists of:
- The adjacency list `g` which stores all edges: `O(n)` space (storing `2(n-1)` edge relationships)
- The recursive call stack for DFS: `O(h)` where `h` is the height of the tree. In the worst case (skewed tree), `h = n`, giving us `O(n)`
- Other variables (`ans`, `s`, loop variables): `O(1)`

The dominant factor is the adjacency list and the potential recursive stack depth, resulting in an overall space complexity of `O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Handling of Subtree Sum After Finding a Valid Component**

**The Pitfall:**
A common mistake is to reset the subtree sum to 0 when finding a component divisible by k, thinking this "cuts" the edge:

```python
def dfs(node: int, parent: int) -> int:
    subtree_sum = values[node]
  
    for neighbor in adjacency_list[node]:
        if neighbor != parent:
            subtree_sum += dfs(neighbor, node)
  
    if subtree_sum % k == 0:
        nonlocal component_count
        component_count += 1
        return 0  # WRONG: This breaks the logic for parent calculations
  
    return subtree_sum
```

**Why It's Wrong:**
Returning 0 when `subtree_sum % k == 0` seems logical (to simulate cutting the edge), but it causes problems:
- When a parent node receives 0 from a child, it loses information about that entire subtree
- This can lead to incorrect calculations for ancestor nodes
- The parent might incorrectly think it can form another valid component

**The Correct Approach:**
The original solution correctly returns the actual `subtree_sum` regardless of divisibility:

```python
def dfs(node: int, parent: int) -> int:
    subtree_sum = values[node]
  
    for neighbor in adjacency_list[node]:
        if neighbor != parent:
            subtree_sum += dfs(neighbor, node)
  
    if subtree_sum % k == 0:
        nonlocal component_count
        component_count += 1
  
    return subtree_sum  # Always return the actual sum
```

This works because:
- We count valid components when found but continue passing the actual sum up the tree
- The mathematical property ensures that if we remove components divisible by k from a total divisible by k, the remainder is also divisible by k
- Each valid subtree is counted exactly once

### 2. **Forgetting to Check if Total Sum is Divisible by k**

**The Pitfall:**
Not verifying that the total sum of all node values is divisible by k before proceeding:

```python
# Missing validation
def maxKDivisibleComponents(self, n, edges, values, k):
    # Directly start DFS without checking total sum
    # ...
```

**Why It Matters:**
If the total sum isn't divisible by k, it's impossible to split the tree into valid components where each has a sum divisible by k.

**The Solution:**
Add a preliminary check (though the given solution implicitly handles this):

```python
def maxKDivisibleComponents(self, n, edges, values, k):
    total_sum = sum(values)
    if total_sum % k != 0:
        return 0  # No valid split possible
  
    # Continue with DFS...
```

### 3. **Double-Counting Components in Undirected Trees**

**The Pitfall:**
Forgetting to track the parent node during DFS traversal, causing revisits:

```python
def dfs(node: int) -> int:  # Missing parent parameter
    subtree_sum = values[node]
  
    for neighbor in adjacency_list[node]:
        # No parent check - will revisit nodes!
        subtree_sum += dfs(neighbor)
```

**Why It's Wrong:**
In an undirected tree, edges go both ways. Without tracking the parent, the DFS will:
- Visit node A from B
- Then try to visit B from A again
- Leading to infinite recursion or incorrect calculations

**The Correct Approach:**
Always pass and check the parent:

```python
def dfs(node: int, parent: int) -> int:
    subtree_sum = values[node]
  
    for neighbor in adjacency_list[node]:
        if neighbor != parent:  # Skip the parent
            subtree_sum += dfs(neighbor, node)
```

