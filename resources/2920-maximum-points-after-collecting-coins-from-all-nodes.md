# 2920. Maximum Points After Collecting Coins From All Nodes

> Source: https://algo.monster/liteproblems/2920

## Problem Statement

You are given an undirected [tree](/problems/tree_intro) rooted at node `0` with `n` nodes labeled from `0` to `n - 1`. The tree structure is defined by a 2D array `edges` of length `n - 1`, where `edges[i] = [ai, bi]` indicates an edge between nodes `ai` and `bi`.

Each node `i` contains `coins[i]` coins, and you need to collect all coins from the [tree](/problems/tree_intro) starting from the root. There's an important constraint: you can only collect coins from a node after collecting coins from all its ancestors.

When collecting coins from node `i`, you have two options:

1. **Option 1**: Collect all coins and receive `coins[i] - k` points. If this value is negative, you lose `abs(coins[i] - k)` points.

2. **Option 2**: Collect all coins and receive `floor(coins[i] / 2)` points. However, if you choose this option, the coins at all nodes in the subtree rooted at node `i` (including node `i` itself) will be halved - specifically, for each node `j` in the subtree, `coins[j]` becomes `floor(coins[j] / 2)`.

Your goal is to determine the maximum total points you can obtain by collecting coins from all nodes in the [tree](/problems/tree_intro), choosing the optimal collection method for each node.

The key challenge is deciding which collection method to use at each node, considering that Option 2 affects all descendants of that node by halving their coin values, which impacts future collection decisions.

## Intuition

When we look at this problem, we need to make a choice at each node: take `coins[i] - k` points (Option 1) or take `floor(coins[i] / 2)` points and halve all coins in the subtree (Option 2).

The key insight is that Option 2 doesn't just affect the current node - it affects all descendants. If we choose Option 2 at a node, every coin value in its subtree gets halved. This means if we use Option 2 multiple times along a path from root to leaf, the coins get halved multiple times.

Think about it this way: if a node's ancestor used Option 2 `j` times, then the effective coin value at this node becomes `coins[i] >> j` (right shift by `j` bits, which is equivalent to dividing by `2^j`).

This observation leads us to track how many times Option 2 has been used by ancestors. We can model our state as `(current_node, times_halved)`. At each state, we have two choices:
- Use Option 1: Get `(coins[i] >> j) - k` points, where `j` is the number of times ancestors used Option 2
- Use Option 2: Get `coins[i] >> (j + 1)` points, and pass `j + 1` to all children

Since `coins[i] ≤ 10^4`, after about 14 right shifts, the value becomes 0. This bounds our state space - we don't need to consider `j > 14`.

The recursive nature of the problem (making decisions that affect subtrees) combined with the bounded state space makes this perfect for DFS with [memoization](/problems/memoization_intro). We explore all possible combinations of using Option 1 or Option 2 at each node, keeping track of how many halvings have been applied, and return the maximum points achievable.

The beauty of this approach is that instead of actually modifying the coin values in the [tree](/problems/tree_intro), we just track how many times we've applied the halving operation and compute the effective value on the fly using bit shifts.

**Learn more about [Tree](/problems/tree_intro), [Depth-First Search](/problems/dfs_intro), [Memoization](/problems/memoization_intro) and [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The solution uses DFS with [memoization](/problems/memoization_intro) to explore all possible collection strategies. Let's walk through the implementation step by step:

**1. Graph Construction:**
First, we build an adjacency list representation of the [tree](/problems/tree_intro) from the edges:
```python
g = [[] for _ in range(n)]
for a, b in edges:
    g[a].append(b)
    g[b].append(a)
```

**2. DFS Function Design:**
The core function `dfs(i, fa, j)` has three parameters:
- `i`: Current node being processed
- `fa`: Parent node (to avoid revisiting in undirected [tree](/problems/tree_intro))
- `j`: Number of times Option 2 has been used by ancestors (number of right shifts needed)

**3. Computing Scores for Both Options:**
At each node, we calculate scores for both collection methods:
- **Option 1**: `a = (coins[i] >> j) - k`
  - The actual coin value is `coins[i] >> j` (shifted right `j` times due to ancestor halvings)
  - We subtract `k` to get the points

- **Option 2**: `b = coins[i] >> (j + 1)`
  - This is equivalent to taking half of the current effective value
  - We shift right by `j + 1` total times

**4. Recursive Exploration:**
For each child node `c` (excluding the parent):
- For Option 1: Add `dfs(c, i, j)` to score `a`
  - Children inherit the same number of halvings `j`

- For Option 2: Add `dfs(c, i, j + 1)` to score `b`
  - Children see one additional halving, so we pass `j + 1`
  - Important optimization: Only recurse if `j < 14` since after 14 shifts, values become 0

**5. [Memoization](/problems/memoization_intro):**
The `@cache` decorator automatically memoizes the function results. This prevents recomputation when we encounter the same `(node, parent, halvings)` state multiple times.

**6. Final Result:**
The function returns `max(a, b)` - the better of the two options. Starting from `dfs(0, -1, 0)`:
- Node 0 (root)
- Parent -1 (no parent for root)
- 0 halvings initially

The time complexity is `O(n × 15)` since we have at most `n` nodes and at most 15 different values of `j` (0 to 14). The space complexity is also `O(n × 15)` for the [memoization](/problems/memoization_intro) cache.

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

**Example Tree:**
```
    0 (coins=8)
   / \
  1   2
(coins=4) (coins=6)
```
- Tree edges: [[0,1], [0,2]]
- coins = [8, 4, 6]
- k = 3

**Step-by-step DFS traversal:**

**Starting at root (node 0, j=0):**
- Current coins: 8 >> 0 = 8
- Option 1: 8 - 3 = 5 points
- Option 2: 8 >> 1 = 4 points

Now we need to explore both options:

**If we choose Option 1 at node 0:**
- We get 5 points from node 0
- Visit child 1 with j=0:
  - coins[1] >> 0 = 4
  - Option 1: 4 - 3 = 1 point
  - Option 2: 4 >> 1 = 2 points
  - Choose Option 2: 2 points (better)
- Visit child 2 with j=0:
  - coins[2] >> 0 = 6
  - Option 1: 6 - 3 = 3 points
  - Option 2: 6 >> 1 = 3 points
  - Choose either: 3 points
- Total for Option 1 path: 5 + 2 + 3 = 10 points

**If we choose Option 2 at node 0:**
- We get 4 points from node 0
- Visit child 1 with j=1 (one halving applied):
  - coins[1] >> 1 = 2 (effective coins after parent's halving)
  - Option 1: 2 - 3 = -1 point
  - Option 2: 4 >> 2 = 1 point
  - Choose Option 2: 1 point (better)
- Visit child 2 with j=1:
  - coins[2] >> 1 = 3 (effective coins after parent's halving)
  - Option 1: 3 - 3 = 0 points
  - Option 2: 6 >> 2 = 1 point
  - Choose Option 2: 1 point (better)
- Total for Option 2 path: 4 + 1 + 1 = 6 points

**Final Decision:**
- Option 1 at root gives total: 10 points
- Option 2 at root gives total: 6 points
- Maximum points achievable: **10**

**Key Observations from the Example:**
1. When we choose Option 2 at node 0, both children see halved coin values (j=1)
2. The actual coins array never changes - we use bit shifts to calculate effective values
3. At each node, we independently evaluate both options and choose the maximum
4. The memoization would cache results like `dfs(1, 0, 0) = 2` and `dfs(1, 0, 1) = 1` for reuse if needed

This demonstrates how the algorithm explores all possible combinations of using Option 1 or Option 2 at each node while tracking the cumulative effect of halvings through the parameter j.

## Python Solution

```python
class Solution:
    def maximumPoints(self, edges: List[List[int]], coins: List[int], k: int) -> int:
        from functools import cache
        from typing import List
      
        # Build adjacency list representation of the tree
        num_nodes = len(coins)
        adjacency_list = [[] for _ in range(num_nodes)]
        for node_a, node_b in edges:
            adjacency_list[node_a].append(node_b)
            adjacency_list[node_b].append(node_a)
      
        @cache
        def dfs(current_node: int, parent_node: int, shift_count: int) -> int:
            """
            Calculate maximum points from subtree rooted at current_node.
          
            Args:
                current_node: Current node being processed
                parent_node: Parent of current node (to avoid revisiting)
                shift_count: Number of right shifts already applied to coin values
          
            Returns:
                Maximum points achievable from this subtree
            """
            # Option 1: Collect coin at current node with penalty k
            # Coin value after shift_count right shifts, minus penalty
            collect_with_penalty = (coins[current_node] >> shift_count) - k
          
            # Option 2: Apply one more right shift (halve the coin value)
            # Coin value after (shift_count + 1) right shifts
            collect_with_extra_shift = coins[current_node] >> (shift_count + 1)
          
            # Process all children of current node
            for child_node in adjacency_list[current_node]:
                if child_node != parent_node:  # Avoid going back to parent
                    # Add points from child subtree with same shift count
                    collect_with_penalty += dfs(child_node, current_node, shift_count)
                  
                    # Only consider extra shift if we haven't shifted too many times
                    # (14 shifts would make most values 0 due to integer division)
                    if shift_count < 14:
                        collect_with_extra_shift += dfs(child_node, current_node, shift_count + 1)
          
            # Return the maximum of the two options
            return max(collect_with_penalty, collect_with_extra_shift)
      
        # Start DFS from node 0 (root), with no parent (-1) and 0 shifts
        maximum_points = dfs(0, -1, 0)
      
        # Clear the cache to free memory
        dfs.cache_clear()
      
        return maximum_points
```

## Time and Space Complexity

**Time Complexity:** `O(n × log M)`

The algorithm uses DFS with memoization. The key insight is that parameter `j` represents how many times we've right-shifted the coin values. Since we're dividing by 2 with each shift, and the condition `j < 14` limits the recursion depth, we effectively process at most `log M` levels where `M` is the maximum coin value. The memoization cache stores states `(i, fa, j)`, and for each node `i`, we can have at most `n` different parent values and approximately `log M` different `j` values. Each state is computed once, and for each state, we iterate through all adjacent nodes. Since the tree has `n-1` edges and each edge is traversed twice (once from each direction), the total time complexity is `O(n × log M)`.

**Space Complexity:** `O(n × log M)`

The space is dominated by the memoization cache and the recursion stack. The cache stores up to `n × n × log M` states in the worst case (though in practice it's much less since not all parent-child combinations are valid in a tree). However, considering the tree structure, for each node we only need to store states with valid parent nodes, reducing the effective space to `O(n × log M)`. The recursion stack depth is at most `O(n)` for a skewed tree, but since we're also considering the `j` parameter which goes up to `log M`, the total space complexity remains `O(n × log M)`. The adjacency list `g` takes `O(n)` space, which doesn't affect the overall complexity.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Handling of the Shift Count Limit**

**The Pitfall:** One common mistake is not properly limiting the recursion depth for Option 2. Without a proper bound, the recursion could continue indefinitely even when coin values have already become 0 after multiple halvings.

**Why it happens:** After repeatedly halving (right-shifting), any integer will eventually become 0. For most practical values, 14-15 right shifts are sufficient to reduce any reasonable coin value to 0. However, developers might:
- Forget to add this optimization entirely, causing unnecessary recursion
- Use an incorrect bound (too small or too large)
- Place the condition check in the wrong location

**Incorrect Implementation:**
```python
# WRONG: No limit on shift count
for child_node in adjacency_list[current_node]:
    if child_node != parent_node:
        collect_with_penalty += dfs(child_node, current_node, shift_count)
        collect_with_extra_shift += dfs(child_node, current_node, shift_count + 1)  # Could recurse forever!
```

**Correct Implementation:**
```python
for child_node in adjacency_list[current_node]:
    if child_node != parent_node:
        collect_with_penalty += dfs(child_node, current_node, shift_count)
        # Only recurse for Option 2 if we haven't shifted too many times
        if shift_count < 14:
            collect_with_extra_shift += dfs(child_node, current_node, shift_count + 1)
```

### 2. **Misunderstanding the Coin Value Calculation**

**The Pitfall:** Incorrectly calculating the effective coin value after ancestors have used Option 2. A common mistake is to apply the halvings incorrectly or forget that the halvings accumulate.

**Why it happens:** The problem states that when Option 2 is chosen, all descendants' coins are halved. This effect is cumulative - if multiple ancestors choose Option 2, the coins are halved multiple times.

**Incorrect Implementation:**
```python
# WRONG: Dividing by 2^j instead of right-shifting
collect_with_penalty = coins[current_node] // (2 ** shift_count) - k  # Inefficient
collect_with_extra_shift = coins[current_node] // (2 ** (shift_count + 1))  # Inefficient

# WRONG: Not accumulating the halvings correctly
collect_with_penalty = (coins[current_node] // 2) - k  # Ignores previous halvings
```

**Correct Implementation:**
```python
# Right-shift is equivalent to integer division by 2^shift_count but more efficient
collect_with_penalty = (coins[current_node] >> shift_count) - k
collect_with_extra_shift = coins[current_node] >> (shift_count + 1)
```

### 3. **Not Handling Negative Scores Properly**

**The Pitfall:** The problem explicitly states that if `coins[i] - k` is negative, you lose points. This is already handled correctly by allowing negative values in the score calculation, but some might mistakenly try to "fix" this.

**Incorrect Implementation:**
```python
# WRONG: Trying to avoid negative scores
collect_with_penalty = max(0, (coins[current_node] >> shift_count) - k)  # This changes the problem!
```

**Correct Implementation:**
```python
# Allow negative values as per problem requirements
collect_with_penalty = (coins[current_node] >> shift_count) - k  # Can be negative
```

The key insight is that sometimes taking a negative score at one node might still be optimal if it prevents halving the coins of a profitable subtree.

