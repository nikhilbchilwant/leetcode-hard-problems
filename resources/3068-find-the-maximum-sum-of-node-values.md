# 3068. Find the Maximum Sum of Node Values

> Source: https://algo.monster/liteproblems/3068

## Problem Statement

You have an undirected [tree](/problems/tree_intro) with `n` nodes numbered from `0` to `n - 1`. The tree structure is defined by an array `edges` of length `n - 1`, where each `edges[i] = [ui, vi]` represents an edge between nodes `ui` and `vi`. You're also given:
- A positive integer `k`
- An array `nums` of length `n`, where `nums[i]` is the value of node `i`

Alice can perform the following operation any number of times (including zero):
1. Choose any edge `[u, v]` in the [tree](/problems/tree_intro)
2. Update the values of both connected nodes:
   - `nums[u] = nums[u] XOR k`
   - `nums[v] = nums[v] XOR k`

The goal is to find the maximum possible sum of all node values after performing any number of operations.

**Key Insight**: When you perform an operation on an edge, both endpoints get XORed with `k`. This means that in any sequence of operations, the XOR with `k` will be applied to an even number of nodes total (since each operation affects exactly 2 nodes).

The problem essentially asks: given that you can XOR any even number of nodes with `k`, what's the maximum sum you can achieve?

For each node, you have two choices:
- Keep its original value `nums[i]`
- Change it to `nums[i] XOR k`

The constraint is that the total number of nodes that get XORed must be even.

The solution uses [dynamic programming](/problems/dynamic_programming_intro) where:
- `f0` tracks the maximum sum when an even number of nodes have been XORed
- `f1` tracks the maximum sum when an odd number of nodes have been XORed

Since we must end with an even count, the answer is `f0` after processing all nodes.

## Intuition

The first key observation is understanding what happens when we perform operations on the [tree](/problems/tree_intro). When we select an edge `[u, v]` and apply the XOR operation, both nodes `u` and `v` get XORed with `k`. If we think about any path in the tree and apply operations to all edges on that path, intermediate nodes will be XORed twice (once for each adjacent edge), while only the endpoints are XORed once.

Since `x XOR k XOR k = x` (XORing twice cancels out), the intermediate nodes return to their original values. This means we can effectively "transfer" the XOR operation between any two nodes in the [tree](/problems/tree_intro), as long as we apply it to both.

The crucial insight is that every operation affects exactly 2 nodes, so the total number of nodes that end up being XORed with `k` must always be even. This transforms our [tree](/problems/tree_intro) problem into a simpler array problem: select an even number of elements from `nums` to XOR with `k` to maximize the sum.

Now we need to decide for each node whether to XOR it or not, with the constraint that the total count must be even. This naturally leads to a [dynamic programming](/problems/dynamic_programming_intro) approach where we track two states:
- The maximum sum when we've XORed an even number of elements so far
- The maximum sum when we've XORed an odd number of elements so far

As we process each element, we have two choices:
1. Don't XOR the current element (keep it as is)
2. XOR the current element with `k`

Each choice affects whether our count remains even/odd or flips. If we've XORed an even number so far and XOR the current element, we'll have an odd count. If we've XORed an odd number and XOR the current element, we'll have an even count.

This parity tracking with maximization leads directly to the two-state DP solution where `f0` and `f1` represent our maximum sums for even and odd counts respectively.

**Learn more about [Greedy](/problems/greedy_intro), [Tree](/problems/tree_intro), [Dynamic Programming](/problems/dynamic_programming_intro) and [Sorting](/problems/sorting_summary) patterns.**

## Solution Approach

We implement a [dynamic programming](/problems/dynamic_programming_intro) solution with two states to track the maximum sum based on the parity of XORed elements.

**State Definition:**
- `f0`: Maximum sum when an even number of elements have been XORed with `k`
- `f1`: Maximum sum when an odd number of elements have been XORed with `k`

**Initialization:**
- `f0 = 0`: Initially, zero elements are XORed (even count), so the sum is 0
- `f1 = -inf`: Initially, it's impossible to have an odd count of XORed elements, so we use negative infinity

**State Transition:**
For each element `x` in `nums`, we have two choices:

1. **Keep `x` unchanged:**
   - If we had an even count (`f0`), adding unchanged `x` keeps it even: contributes to new `f0`
   - If we had an odd count (`f1`), adding unchanged `x` keeps it odd: contributes to new `f1`

2. **XOR `x` with `k`:**
   - If we had an even count (`f0`), XORing `x` makes it odd: contributes to new `f1`
   - If we had an odd count (`f1`), XORing `x` makes it even: contributes to new `f0`

The state transition equations become:
```
new_f0 = max(f0 + x, f1 + (x ^ k))
new_f1 = max(f1 + x, f0 + (x ^ k))
```

**Implementation:**
```python
f0, f1 = 0, -inf
for x in nums:
    f0, f1 = max(f0 + x, f1 + (x ^ k)), max(f1 + x, f0 + (x ^ k))
return f0
```

We iterate through each element once, updating both states simultaneously using parallel assignment. The parallel assignment ensures we use the old values of `f0` and `f1` for both calculations before updating them.

**Final Result:**
We return `f0` because we need an even number of elements to be XORed (as operations always affect pairs of nodes).

**Time Complexity:** `O(n)` - single pass through the array
**Space Complexity:** `O(1)` - only two variables for state tracking

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

**Example:** `nums = [2, 3, 1]`, `k = 6`

For each element, we can either keep it or XOR it with 6:
- Node 0: Keep value 2, or change to 2 XOR 6 = 4
- Node 1: Keep value 3, or change to 3 XOR 6 = 5  
- Node 2: Keep value 1, or change to 1 XOR 6 = 7

Since we must XOR an even number of nodes (0, 2, or more nodes), let's trace through our DP:

**Initial State:**
- `f0 = 0` (even count of XORed nodes, sum = 0)
- `f1 = -inf` (odd count impossible initially)

**Processing nums[0] = 2:**
- Keep 2 unchanged:
  - From f0 (even): `0 + 2 = 2` (stays even)
  - From f1 (odd): `-inf + 2 = -inf` (stays odd)
- XOR with k (2 XOR 6 = 4):
  - From f0 (even): `0 + 4 = 4` (becomes odd)
  - From f1 (odd): `-inf + 4 = -inf` (becomes even)

New states:
- `f0 = max(2, -inf) = 2` (best even: don't XOR node 0)
- `f1 = max(-inf, 4) = 4` (best odd: XOR node 0)

**Processing nums[1] = 3:**
- Keep 3 unchanged:
  - From f0: `2 + 3 = 5` (stays even)
  - From f1: `4 + 3 = 7` (stays odd)
- XOR with k (3 XOR 6 = 5):
  - From f0: `2 + 5 = 7` (becomes odd)
  - From f1: `4 + 5 = 9` (becomes even)

New states:
- `f0 = max(5, 9) = 9` (best even: XOR nodes 0 and 1)
- `f1 = max(7, 7) = 7` (best odd: either XOR only node 0 or only node 1)

**Processing nums[2] = 1:**
- Keep 1 unchanged:
  - From f0: `9 + 1 = 10` (stays even)
  - From f1: `7 + 1 = 8` (stays odd)
- XOR with k (1 XOR 6 = 7):
  - From f0: `9 + 7 = 16` (becomes odd)
  - From f1: `7 + 7 = 14` (becomes even)

Final states:
- `f0 = max(10, 14) = 14` (best even: XOR all three nodes, then one more)
- `f1 = max(8, 16) = 16` (best odd: XOR nodes 0, 1, and 2)

**Result:** Return `f0 = 14`

The optimal solution XORs nodes 0 and 2 (even count):
- Node 0: 2 → 4 (XORed)
- Node 1: 3 → 3 (unchanged)  
- Node 2: 1 → 7 (XORed)
- Sum: 4 + 3 + 7 = 14

## Python Solution

```python
class Solution:
    def maximumValueSum(self, nums: List[int], k: int, edges: List[List[int]]) -> int:
        # Dynamic programming approach
        # even_count: max sum when we've XORed an even number of nodes
        # odd_count: max sum when we've XORed an odd number of nodes
        even_count, odd_count = 0, float('-inf')
      
        # Process each node value
        for value in nums:
            # Calculate new states after considering current node
            # Option 1: Keep current node unchanged
            # Option 2: XOR current node with k
            new_even = max(
                even_count + value,           # Don't XOR: even stays even
                odd_count + (value ^ k)        # XOR: odd becomes even
            )
            new_odd = max(
                odd_count + value,             # Don't XOR: odd stays odd
                even_count + (value ^ k)       # XOR: even becomes odd
            )
          
            # Update states for next iteration
            even_count, odd_count = new_even, new_odd
      
        # Return maximum sum with even number of XOR operations
        # (valid configuration requires even number of operations)
        return even_count
```

## Time and Space Complexity

The time complexity is `O(n)`, where `n` is the length of the array `nums`. The algorithm iterates through the array exactly once, performing constant-time operations (addition, XOR operation, and max comparisons) for each element.

The space complexity is `O(1)`. The algorithm only uses two variables `f0` and `f1` to track the state throughout the iteration, regardless of the input size. No additional data structures that scale with input size are created.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Misunderstanding the Tree Structure Constraint**

**Pitfall:** Many developers initially think they need to traverse the tree structure using DFS/BFS since edges are provided. They might try to process nodes based on parent-child relationships or attempt to find connected components.

**Why it happens:** The problem provides edges and describes a tree structure, which naturally leads to thinking about graph traversal algorithms.

**The Reality:** The tree structure is actually irrelevant to the solution! The key insight is that each operation XORs exactly 2 nodes, so we just need to ensure an even total count of XORed nodes. The actual connectivity doesn't matter.

**Solution:** Focus on the constraint that operations always affect pairs of nodes, making the total count of XORed nodes even. Treat it as a simple array problem with a parity constraint.

### 2. **Incorrect State Initialization**

**Pitfall:** Initializing both states to 0:
```python
even_count, odd_count = 0, 0  # WRONG!
```

**Why it's wrong:** Starting with `odd_count = 0` implies we can have a valid sum of 0 with an odd number of XORed elements, which is impossible at the start (we haven't XORed anything yet).

**Solution:** Initialize `odd_count` to negative infinity:
```python
even_count, odd_count = 0, float('-inf')
```
This ensures that any path requiring an odd count at intermediate steps is properly handled.

### 3. **Sequential Assignment Instead of Parallel**

**Pitfall:** Updating states sequentially:
```python
# WRONG approach
even_count = max(even_count + value, odd_count + (value ^ k))
odd_count = max(odd_count + value, even_count + (value ^ k))  # Uses updated even_count!
```

**Why it's wrong:** The second assignment uses the already-updated `even_count` value instead of the original value from the previous iteration.

**Solution:** Use parallel assignment or temporary variables:
```python
# Correct: Parallel assignment
even_count, odd_count = max(...), max(...)

# Alternative: Using temporary variables
new_even = max(even_count + value, odd_count + (value ^ k))
new_odd = max(odd_count + value, even_count + (value ^ k))
even_count, odd_count = new_even, new_odd
```

### 4. **Returning the Wrong State**

**Pitfall:** Returning `max(even_count, odd_count)` thinking we want the maximum possible sum regardless of parity.

**Why it's wrong:** The problem requires that the total number of XORed nodes must be even (since each operation affects exactly 2 nodes).

**Solution:** Always return `even_count`:
```python
return even_count  # Must have even number of XORed nodes
```

### 5. **Overthinking the XOR Operation**

**Pitfall:** Trying to analyze when `value ^ k > value` for each node individually and making greedy decisions based on this comparison.

**Why it's wrong:** The constraint that we must XOR an even number of nodes means we can't just greedily pick all beneficial XORs. Sometimes we need to XOR a node that decreases its value to maintain the even count constraint while maximizing the total.

**Solution:** Let the DP handle all cases. The algorithm automatically considers both keeping and XORing each value while maintaining the parity constraint.

