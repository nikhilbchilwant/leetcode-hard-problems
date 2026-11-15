# 2307. Check for Contradictions in Equations

> Source: https://algo.monster/liteproblems/2307

## Problem Statement

You are given a 2D array of strings `equations` and an array of real numbers `values`. Each `equations[i] = [Ai, Bi]` paired with `values[i]` represents the mathematical relationship `Ai / Bi = values[i]`.

Your task is to determine if there exists a contradiction in these equations. Return `true` if a contradiction exists, otherwise return `false`.

For example, if you have equations stating `A / B = 2` and `B / C = 3`, you can derive that `A / C = 6`. If another equation states `A / C = 5`, this would be a contradiction.

When comparing two numbers for equality, consider them equal if their absolute difference is less than `10^-5`. This accounts for floating-point precision issues.

The problem essentially asks you to check if all the given division relationships are mathematically consistent with each other. If you can find any derived relationship that conflicts with a given equation, a contradiction exists.

## Intuition

The key insight is recognizing that division equations create a network of relationships between variables. If we know `A / B = 2` and `B / C = 3`, we can derive that `A / C = 6` through the transitive property of division. A contradiction occurs when we have two different paths between the same variables that yield different ratios.

Think of each variable as a node in a [graph](/problems/graph_intro). When we have an equation `A / B = value`, we're essentially saying these two nodes are connected with a specific ratio relationship. As we process more equations, we're either:
1. Connecting previously unconnected components (learning new relationships)
2. Adding redundant information that should be consistent with what we already know

The challenge is efficiently tracking these ratio relationships. If we pick any node as a reference point with value 1, we can express all other connected nodes as multiples of this reference. For example, if we set `B = 1` and `A / B = 2`, then `A = 2`. If we also know `B / C = 3`, then `C = 1/3`.

This is where weighted union-find shines. Each node stores its ratio relative to its root. When we union two components, we adjust the weights to maintain consistent ratios. When we encounter an equation between nodes already in the same component, we can quickly check if their current ratio matches the given equation.

The path compression in union-find naturally handles the chain of divisions. When finding the root of a node, we update its weight to be the direct ratio to the root, effectively "flattening" the calculation of `A / B × B / C × C / D = A / D`.

A contradiction is detected when two nodes that are already connected have a ratio that differs from the given equation by more than the tolerance `10^-5`.

**Learn more about [Depth-First Search](/problems/dfs_intro), [Union Find](/problems/dsu_intro) and [Graph](/problems/graph_intro) patterns.**

## Solution Approach

The solution uses a weighted union-find data structure to track the relationships between variables and detect contradictions.

**Step 1: Map strings to integers**
First, we convert all string variables to integer indices for easier manipulation. We iterate through all equations and assign a unique integer ID to each distinct variable using a dictionary `d`.

```python
d = defaultdict(int)
n = 0
for e in equations:
    for s in e:
        if s not in d:
            d[s] = n
            n += 1
```

**Step 2: Initialize union-find structures**
We create two arrays:
- `p`: parent array where `p[i]` stores the parent of node `i`
- `w`: weight array where `w[i]` stores the ratio of node `i` to its parent

Initially, each node is its own parent with weight 1.0.

```python
p = list(range(n))
w = [1.0] * n
```

**Step 3: Find operation with path compression**
The `find` function locates the root of a node while updating weights along the path. During path compression, we multiply weights to get the direct ratio to the root:

```python
def find(x: int) -> int:
    if p[x] != x:
        root = find(p[x])
        w[x] *= w[p[x]]  # Update weight to be ratio to root
        p[x] = root       # Path compression
    return p[x]
```

**Step 4: Process equations**
For each equation `A / B = v`:
1. Find the roots of A and B
2. If they're in different components:
   - Union them by making one root point to the other
   - Calculate the new weight: `w[pb] = v * w[a] / w[b]`
   - This ensures `A / B = (w[a] / w[root]) / (w[b] / w[root]) = w[a] / w[b] = v`
3. If they're already in the same component:
   - Check if the existing ratio matches the given value
   - If `|v * w[a] - w[b]| >= eps`, we have a contradiction

```python
for (a, b), v in zip(equations, values):
    a, b = d[a], d[b]
    pa, pb = find(a), find(b)
    if pa != pb:
        p[pb] = pa
        w[pb] = v * w[a] / w[b]
    elif abs(v * w[a] - w[b]) >= eps:
        return True
```

The time complexity is `O(n × α(n))` where `α` is the inverse Ackermann function (nearly constant), and space complexity is `O(n)` for storing the parent and weight arrays.

## Example Walkthrough

Let's walk through a small example to illustrate how the weighted union-find solution detects contradictions.

**Example Input:**
- equations = [["A", "B"], ["B", "C"], ["A", "C"]]
- values = [2.0, 3.0, 5.0]

This represents: A/B = 2, B/C = 3, A/C = 5

**Step 1: Map strings to integers**
- "A" → 0
- "B" → 1
- "C" → 2

**Step 2: Initialize union-find**
- Parent array p = [0, 1, 2] (each node is its own parent)
- Weight array w = [1.0, 1.0, 1.0] (initial weights are 1.0)

**Step 3: Process first equation A/B = 2**
- Find roots: root(0) = 0, root(1) = 1
- Different components, so union them
- Make node 1's parent = 0
- Calculate weight: w[1] = 2.0 × 1.0 / 1.0 = 2.0
- Result: p = [0, 0, 2], w = [1.0, 2.0, 1.0]
- This means A = 1.0 (relative to root 0), B = 2.0 (so A/B = 1.0/2.0 = 0.5... wait, let me recalculate)

Actually, let me reconsider the weight interpretation. The weight w[i] represents the value of node i divided by its parent's value. So if B's parent is A and w[B] = v, then B/A = v, which means A/B = 1/v.

Let me restart with the correct interpretation:

**Step 3: Process first equation A/B = 2**
- Find roots: root(0) = 0, root(1) = 1
- Different components, union them by making 1's parent = 0
- We need A/B = 2, which means B = A/2
- Calculate weight: w[1] = w[0] / (2 × w[1]) = 1.0 / (2 × 1.0) = 0.5
- Result: p = [0, 0, 2], w = [1.0, 0.5, 1.0]
- Now B/A = 0.5, so A/B = 2 ✓

**Step 4: Process second equation B/C = 3**
- Find roots: root(1) = 0 (with path compression, w[1] remains 0.5), root(2) = 2
- Different components, union them by making 2's parent = 0
- We need B/C = 3, where B has value 0.5 relative to root 0
- Calculate weight: w[2] = w[1] / (3 × w[2]) = 0.5 / (3 × 1.0) = 1/6
- Result: p = [0, 0, 0], w = [1.0, 0.5, 1/6]
- Now C = 1/6 relative to root, B = 0.5, so B/C = 0.5/(1/6) = 3 ✓

**Step 5: Process third equation A/C = 5**
- Find roots: root(0) = 0, root(2) = 0
- Same component! Check consistency
- Current ratio: A/C = w[0]/w[2] = 1.0/(1/6) = 6
- Given value: 5
- Check: |6 - 5| = 1 ≥ 10^-5
- **Contradiction detected!** Return true

The algorithm correctly identifies that A/C cannot equal 5 when A/B = 2 and B/C = 3 (which implies A/C = 6).

## Python Solution

```python
class Solution:
    def checkContradictions(
        self, equations: List[List[str]], values: List[float]
    ) -> bool:
        # Find the root of a node with path compression and weight update
        def find_root(node: int) -> int:
            if parent[node] != node:
                # Recursively find the root
                root = find_root(parent[node])
                # Path compression: multiply weights along the path
                weight[node] *= weight[parent[node]]
                # Update parent to root for path compression
                parent[node] = root
            return parent[node]

        # Map variable names to unique integer IDs
        variable_to_id = defaultdict(int)
        next_id = 0

        # Assign unique ID to each variable in equations
        for equation in equations:
            for variable in equation:
                if variable not in variable_to_id:
                    variable_to_id[variable] = next_id
                    next_id += 1

        # Initialize Union-Find structure
        # parent[i] stores the parent of node i
        parent = list(range(next_id))
        # weight[i] stores the ratio from node i to its parent
        weight = [1.0] * next_id

        # Tolerance for floating-point comparison
        epsilon = 1e-5

        # Process each equation
        for (var_a, var_b), value in zip(equations, values):
            # Get IDs for both variables
            id_a, id_b = variable_to_id[var_a], variable_to_id[var_b]

            # Find roots of both variables
            root_a, root_b = find_root(id_a), find_root(id_b)

            if root_a != root_b:
                # Variables are in different components, merge them
                # Set root_a as parent of root_b
                parent[root_b] = root_a
                # Calculate weight for root_b to maintain the equation
                # var_a / var_b = value, so root_b's weight = value * weight_a / weight_b
                weight[root_b] = value * weight[id_a] / weight[id_b]
            else:
                # Variables are already in the same component
                # Check if the new equation contradicts existing relationships
                # Expected ratio: var_a / var_b should equal value
                # Actual ratio: weight[id_a] / weight[id_b]
                if abs(value * weight[id_a] - weight[id_b]) >= epsilon:
                    return True  # Contradiction found

        return False  # No contradictions found
```

## Time and Space Complexity

**Time Complexity:** `O(n * α(n))` where `n` is the total number of unique variables in all equations, and `α(n)` is the inverse Ackermann function.

- Building the dictionary `d` requires iterating through all equations and variables: `O(n)` where `n` is the total number of unique variables
- Initializing parent array `p` and weight array `w`: `O(n)`
- Processing each equation involves:
  - Two `find()` operations with path compression
  - Either a union operation or a contradiction check
  - Each `find()` operation has amortized time complexity of `O(α(n))` due to path compression
  - There are `m` equations to process where `m = len(equations)`
- Overall: `O(n + m * α(n))` which simplifies to `O(n * α(n))` since `m ≤ n²` in the worst case

**Space Complexity:** `O(n)`

- Dictionary `d` stores mappings for `n` unique variables: `O(n)`
- Parent array `p` of size `n`: `O(n)`
- Weight array `w` of size `n`: `O(n)`
- Recursion stack for `find()` function in worst case (before path compression): `O(n)`
- Total space complexity: `O(n)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Weight Calculation During Union Operation**

One of the most common mistakes is incorrectly calculating the weight when merging two components. When setting `root_b` as a child of `root_a`, many implementations mistakenly use:

**Incorrect:**
```python
weight[root_b] = value * weight[id_b] / weight[id_a]  # Wrong formula!
```

**Why it's wrong:** The relationship we need to maintain is `var_a / var_b = value`. After union, both variables should have the correct ratio to their common root. The formula should ensure that `(weight[id_a] / weight[root]) / (weight[id_b] / weight[root]) = value`.

**Correct:**
```python
weight[root_b] = value * weight[id_a] / weight[id_b]
```

### 2. **Forgetting Path Compression Weight Update**

Another critical pitfall is implementing path compression without updating weights along the compressed path:

**Incorrect:**
```python
def find_root(node: int) -> int:
    if parent[node] != node:
        parent[node] = find_root(parent[node])  # Only compress path
        # Forgot to update weight[node]!
    return parent[node]
```

**Why it's wrong:** When we compress the path, we're changing a node's parent from its immediate parent to the root. The weight must be updated to reflect the ratio to the new parent (root).

**Correct:**
```python
def find_root(node: int) -> int:
    if parent[node] != node:
        root = find_root(parent[node])
        weight[node] *= weight[parent[node]]  # Update weight BEFORE updating parent
        parent[node] = root
    return parent[node]
```

### 3. **Incorrect Contradiction Check**

When checking for contradictions in already-connected components, a common error is comparing the wrong values:

**Incorrect:**
```python
if abs(value - weight[id_a] / weight[id_b]) >= epsilon:  # Direct division
    return True
```

**Why it's wrong:** After path compression, `weight[id_a]` and `weight[id_b]` represent ratios to the root, not to each other. Direct division can cause division by zero or incorrect results.

**Correct:**
```python
if abs(value * weight[id_b] - weight[id_a]) >= epsilon:
    return True
```

This rearranged formula avoids division and correctly checks if `weight[id_a] / weight[id_b] ≈ value`.

### 4. **Not Handling Self-Division Cases**

If the input contains equations like `A / A = value` where `value ≠ 1.0`, this represents a contradiction that should be detected immediately:

**Enhanced Solution:**
```python
for (var_a, var_b), value in zip(equations, values):
    if var_a == var_b:
        if abs(value - 1.0) >= epsilon:
            return True  # Self-division must equal 1
        continue  # Skip processing if valid

    # ... rest of the logic
```

### 5. **Floating-Point Precision Issues**

Using exact equality (`==`) for floating-point comparison instead of epsilon-based comparison:

**Incorrect:**
```python
if value * weight[id_b] != weight[id_a]:  # Exact comparison
    return True
```

**Correct:**
```python
if abs(value * weight[id_b] - weight[id_a]) >= epsilon:
    return True
```

Always use tolerance-based comparison for floating-point numbers to avoid false positives due to rounding errors.

