# 3651. Minimum Cost Path with Teleportations

> Source: https://algo.monster/liteproblems/3651

## Problem Statement

You are given a `m x n` 2D integer array `grid` and an integer `k`. You begin at the top-left cell `(0, 0)`, and your objective is to travel to the bottom-right cell `(m - 1, n - 1)`.

During your journey, two kinds of moves are available to you:

- **Normal move**: From your current cell `(i, j)`, you may move one step **right** to `(i, j + 1)` or one step **down** to `(i + 1, j)`. Each such move costs the value stored in the **destination** cell.

- **Teleportation**: From any cell `(i, j)`, you may teleport to any cell `(x, y)` as long as `grid[x][y] <= grid[i][j]` (the destination value must not exceed the current cell's value). This move costs `0`. However, you are limited to using teleportation **at most `k` times**.

Your task is to compute the **minimum** total cost required to reach cell `(m - 1, n - 1)` starting from cell `(0, 0)`.

## Intuition

The key observation is that the cost we pay depends on **two things**: where we are in the grid, and **how many teleportations we have used so far**. Since the number of teleportations is capped at `k`, the teleport count is a natural extra dimension to track in our state. This naturally points us toward dynamic programming, where we define `f[t][i][j]` as the minimum cost to reach cell `(i, j)` having used exactly `t` teleportations.

Let us think about the two move types separately.

For **normal moves**, the situation is the classic "minimum path sum" pattern: to arrive at cell `(i, j)`, we must have come from the cell above `(i - 1, j)` or the cell to the left `(i, j - 1)`, paying the value `grid[i][j]` to step in. So for a fixed teleport count `t`, we can fill the grid in order and take the cheaper of the two incoming directions.

The trickier part is **teleportation**. A teleport lets us jump from cell `(i, j)` to any cell `(x, y)` whose value is **less than or equal to** `grid[i][j]`, and it is free. The question becomes: if we are about to land at a cell with value `v` via teleport, where is the cheapest place we could have teleported **from**? It can be any cell whose value is at least `v`. So among all cells with value `>= v`, we want the one with the smallest cost reached using `t - 1` teleportations.

This is where the **sorting by value** trick comes in. If we group cells by their value and process the groups in **descending order** of value, we can maintain a running minimum `mn`. As we sweep from high values down to low values, `mn` keeps track of the cheapest `f[t - 1]` cost among all cells seen so far — and those are exactly the cells with value `>= v` that could legally teleport to the current group. So each cell in the current group can be reached for cost `mn` using one more teleport.

Putting it together, for each teleport count `t` we do two passes:

1. First, a **descending-value sweep** to apply teleportation, setting `f[t][i][j] = mn`, the cheapest cost to arrive here by teleporting from a higher-or-equal valued cell.
2. Then, a **normal grid sweep** to layer ordinary right/down moves on top of those teleport landing points.

Finally, since using fewer teleportations might sometimes be cheaper (extra teleports never hurt but the optimal answer could occur at any count), we take the minimum of `f[t][m - 1][n - 1]` across all `t` from `0` to `k`.

## Solution Approach

### Solution 1: Dynamic Programming

We define `f[t][i][j]` as the minimum cost to reach cell `(i, j)` using exactly `t` teleportations. Initially, `f[0][0][0] = 0`, and all other states are set to infinity.

**Step 1: Initialize the zero-teleport layer.**

First, we fill in `f[0][i][j]`, the costs reachable **without any teleportation**. In this case we can only arrive at a cell by moving right or down.

If `i > 0`, we can move from the cell above `(i - 1, j)`:

`f[0][i][j] = min(f[0][i][j], f[0][i - 1][j] + grid[i][j])`

If `j > 0`, we can move from the cell to the left `(i, j - 1)`:

`f[0][i][j] = min(f[0][i][j], f[0][i][j - 1] + grid[i][j])`

**Step 2: Group cells by value.**

To handle teleportation efficiently, we group the grid cells by their value using a hash map `g`, where the key is the cell value and the value is a list of coordinates of cells having that value. We then collect the keys and sort them in **descending order** into `keys`.

**Step 3: Process each teleport count.**

For each teleport count `t` from `1` to `k`, we perform two passes.

The first pass applies teleportation. We maintain a running minimum `mn`, and iterate over the value groups in descending order. For each group, we first update `mn` using every cell in the group based on the previous layer:

`mn = min(mn, f[t - 1][i][j])`

Then we assign that running minimum to every cell in the same group:

`f[t][i][j] = mn`

The crucial detail here is the **two-phase loop within each group**: we update `mn` over the whole group *before* writing `mn` back to those cells. This guarantees that a teleport must come from a cell with value strictly larger, or from another cell in the same group, all of which satisfy `grid[x][y] <= grid[i][j]` — exactly matching the teleport rule. Since groups are processed from high values down to low values, `mn` always represents the cheapest `f[t - 1]` cost over all cells whose value is `>= ` the current group's value.

The second pass layers ordinary moves on top of the teleport landing points by sweeping the grid in order:

If `i > 0`:

`f[t][i][j] = min(f[t][i][j], f[t][i - 1][j] + grid[i][j])`

If `j > 0`:

`f[t][i][j] = min(f[t][i][j], f[t][i][j - 1] + grid[i][j])`

**Step 4: Combine the answer.**

Since using fewer teleportations might yield a smaller total cost, the final answer is the minimum value of `f[t][m - 1][n - 1]` over all `t` from `0` to `k`:

`min(f[t][m - 1][n - 1] for t in range(k + 1))`

**Complexity Analysis.**

Let `m` and `n` be the dimensions of the grid. There are `k + 1` layers, and each layer requires processing all `m * n` cells for both the teleport sweep and the normal-move sweep. The time complexity is `O(k * m * n)` (the value grouping and sorting adds `O(m * n * log(m * n))`, which is dominated for typical inputs). The space complexity is `O(k * m * n)` for the `f` table.

## Example Walkthrough

Let's trace through the solution with a small concrete example.

**Input:**
```
grid = [[1, 3],
        [2, 1]]
k = 1
```

We start at `(0, 0)` (value `1`) and want to reach `(1, 1)` (value `1`). We may teleport at most `1` time.

---

**Step 1: Initialize the zero-teleport layer `f[0]`.**

We compute minimum path sums using only right/down moves. By convention, `f[0][0][0] = 0` (we don't pay for the starting cell).

- `f[0][0][0] = 0`
- `f[0][0][1] = f[0][0][0] + grid[0][1] = 0 + 3 = 3` (move right)
- `f[0][1][0] = f[0][0][0] + grid[1][0] = 0 + 2 = 2` (move down)
- `f[0][1][1]`: come from above `(0,1)` → `3 + 1 = 4`, or from left `(1,0)` → `2 + 1 = 3`. Take the min → `3`.

So without teleportation:
```
f[0] = [[0, 3],
        [2, 3]]
```
The best cost to reach `(1, 1)` with **0 teleports** is `3`.

---

**Step 2: Group cells by value and sort descending.**

```
g = { 1: [(0,0), (1,1)],
      3: [(0,1)],
      2: [(1,0)] }

keys (descending) = [3, 2, 1]
```

---

**Step 3: Process teleport count `t = 1`.**

**First pass — teleport sweep (descending value order).** Maintain running minimum `mn = ∞`.

Process value `3` → cells `[(0,1)]`:
- Update `mn`: `mn = min(∞, f[0][0][1]) = min(∞, 3) = 3`.
- Write back: `f[1][0][1] = mn = 3`. (Teleporting *into* `(0,1)` requires coming from a cell with value `≥ 3`; only `(0,1)` itself qualifies so far, giving cost `3`.)

Process value `2` → cells `[(1,0)]`:
- Update `mn`: `mn = min(3, f[0][1][0]) = min(3, 2) = 2`.
- Write back: `f[1][1][0] = mn = 2`. (We could teleport into `(1,0)` from any cell with value `≥ 2`, i.e. `(0,1)` or `(1,0)`; cheapest is cost `2`.)

Process value `1` → cells `[(0,0), (1,1)]`:
- Update `mn` over the **whole group first**:
  - `mn = min(2, f[0][0][0]) = min(2, 0) = 0`
  - `mn = min(0, f[0][1][1]) = min(0, 3) = 0`
- Write back to both cells: `f[1][0][0] = 0`, `f[1][1][1] = 0`.

Here's the key insight at `(1, 1)`: we teleported into it for cost `0`. The running `mn = 0` came from `(0, 0)`, which has value `1 ≥ 1`, so the teleport `(0,0) → (1,1)` is legal and free. We effectively jumped from start to the destination region for `0` cost.

After the teleport sweep:
```
f[1] = [[0, 3],
        [2, 0]]
```

**Second pass — normal-move sweep.** Layer right/down moves on top:

- `f[1][0][0] = 0` (no incoming moves)
- `f[1][0][1]`: from left `(0,0)` → `0 + 3 = 3`. Already `3`. Stays `3`.
- `f[1][1][0]`: from above `(0,0)` → `0 + 2 = 2`. Already `2`. Stays `2`.
- `f[1][1][1]`: from above `(0,1)` → `3 + 1 = 4`; from left `(1,0)` → `2 + 1 = 3`. Current value is `0`, which is smaller. Stays `0`.

Final layer:
```
f[1] = [[0, 3],
        [2, 0]]
```
The best cost to reach `(1, 1)` with **1 teleport** is `0`.

---

**Step 4: Combine the answer.**

Take the minimum over all teleport counts:
- `f[0][1][1] = 3`
- `f[1][1][1] = 0`

```
answer = min(3, 0) = 0
```

---

**Interpretation:** The optimal strategy is to teleport directly from `(0, 0)` (value `1`) to `(1, 1)` (value `1`). Since `1 ≤ 1`, the teleport is allowed, it costs `0`, and it uses exactly one of our `k = 1` allowed teleports. This beats the cheapest walking path (cost `3`), so the minimum total cost is **`0`**.

## Python Solution

```python
from collections import defaultdict
from math import inf
from typing import List


class Solution:
    def minCost(self, grid: List[List[int]], k: int) -> int:
        rows, cols = len(grid), len(grid[0])

        # cost[t][i][j]: minimum cost to reach cell (i, j) using exactly t teleports.
        # A teleport jumps to any other cell sharing the same grid value.
        cost = [[[inf] * cols for _ in range(rows)] for _ in range(k + 1)]
        cost[0][0][0] = 0

        # Base layer (t = 0): plain grid DP, only moving down or right.
        for i in range(rows):
            for j in range(cols):
                if i:
                    cost[0][i][j] = min(
                        cost[0][i][j], cost[0][i - 1][j] + grid[i][j]
                    )
                if j:
                    cost[0][i][j] = min(
                        cost[0][i][j], cost[0][i][j - 1] + grid[i][j]
                    )

        # Group cell coordinates by their value so teleports can connect them.
        value_to_cells = defaultdict(list)
        for i, row in enumerate(grid):
            for j, value in enumerate(row):
                value_to_cells[value].append((i, j))

        # Process distinct values in descending order.
        sorted_values = sorted(value_to_cells, reverse=True)

        for t in range(1, k + 1):
            best_prev = inf
            for value in sorted_values:
                cells = value_to_cells[value]

                # Find the cheapest cell (from the previous teleport layer)
                # among all cells sharing this value.
                for i, j in cells:
                    best_prev = min(best_prev, cost[t - 1][i][j])

                # Teleport into every same-valued cell at the best prior cost.
                for i, j in cells:
                    cost[t][i][j] = best_prev

            # After teleporting, allow normal down/right moves within layer t.
            for i in range(rows):
                for j in range(cols):
                    if i:
                        cost[t][i][j] = min(
                            cost[t][i][j], cost[t][i - 1][j] + grid[i][j]
                        )
                    if j:
                        cost[t][i][j] = min(
                            cost[t][i][j], cost[t][i][j - 1] + grid[i][j]
                        )

        # Answer is the minimum cost to reach the bottom-right corner
        # across any number of teleports (0 to k).
        return min(cost[t][rows - 1][cols - 1] for t in range(k + 1))
```

## Time and Space Complexity

- **Time complexity**: `O((k + log(mn)) × mn)`.

  - The initial dynamic programming pass (for `t = 0`) iterates over every cell of the grid once, costing `O(mn)`.
  - Building the dictionary `g` that maps each value to its list of positions also costs `O(mn)`.
  - Sorting the distinct keys with `keys = sorted(g, reverse=True)` costs at most `O(mn × log(mn))`, since there are at most `mn` distinct values.
  - The main loop runs `t` from `1` to `k`. For each `t`:
    - The teleportation update iterates over all positions grouped by key, which in total visits every cell once, costing `O(mn)`.
    - The subsequent walking DP over the whole grid also costs `O(mn)`.
  
    Hence each iteration of `t` costs `O(mn)`, and over `k` iterations this contributes `O(k × mn)`.
  - Combining all parts gives `O(mn) + O(mn × log(mn)) + O(k × mn) = O((k + log(mn)) × mn)`.

- **Space complexity**: `O(k × mn)`.

  - The 3-dimensional array `f` has dimensions `(k + 1) × m × n`, requiring `O(k × mn)` space.
  - The dictionary `g` stores every cell position exactly once, costing `O(mn)`.
  - The sorted `keys` list holds at most `mn` distinct values, costing `O(mn)`.
  - The dominant term is the array `f`, so the overall space complexity is `O(k × mn)`.

  Here, `m` and `n` are the number of rows and columns of the grid, respectively, and `k` is the maximum allowed number of teleportations.

## Common Pitfalls

### Pitfall 1: Misunderstanding the Teleport Rule — Using "Equal Value" Instead of "Less Than or Equal"

The most dangerous trap in this problem lies in the **two-phase loop inside each value group**. A careless reading of the code's comments (or a naive re-implementation) suggests teleportation only connects cells that share the *exact same value*. But the actual problem states teleportation is allowed whenever `grid[x][y] <= grid[i][j]`, i.e., you may jump to **any cell whose value is less than or equal to** your current cell.

The correctness of the solution hinges entirely on processing groups in **descending order of value** while carrying a *running* minimum `best_prev` across groups. If you reset `best_prev` for each group (treating teleports as "same value only"), you break the algorithm.

**Incorrect version:**

```python
for t in range(1, k + 1):
    for value in sorted_values:
        cells = value_to_cells[value]
        best_prev = inf          # WRONG: reset inside the loop
        for i, j in cells:
            best_prev = min(best_prev, cost[t - 1][i][j])
        for i, j in cells:
            cost[t][i][j] = best_prev
    ...
```

This only lets a cell teleport from another cell of the **identical** value, ignoring all higher-valued cells that are legal teleport sources. The resulting cost will be too high (or `inf` when no same-value path exists).

**Correct version:**

```python
for t in range(1, k + 1):
    best_prev = inf              # CORRECT: declared once, outside the value loop
    for value in sorted_values:  # descending order is essential
        cells = value_to_cells[value]
        for i, j in cells:
            best_prev = min(best_prev, cost[t - 1][i][j])  # phase 1: absorb
        for i, j in cells:
            cost[t][i][j] = best_prev                       # phase 2: write
    ...
```

Because `best_prev` persists across groups and the groups descend in value, by the time we reach value `v`, `best_prev` already incorporates the cheapest `cost[t-1]` over every cell with value `>= v` — exactly the set of legal teleport sources.

### Pitfall 2: Collapsing the Two Phases Into One Loop

Even with `best_prev` correctly placed outside the group loop, merging the "absorb" and "write" passes into a single loop is subtly wrong:

```python
for value in sorted_values:
    for i, j in value_to_cells[value]:
        best_prev = min(best_prev, cost[t - 1][i][j])
        cost[t][i][j] = best_prev      # WRONG: uses partial group minimum
```

Here, the first cell in a group writes a `best_prev` that hasn't yet seen the *other* cells of the same value. Since same-value cells are also valid teleport sources (`v <= v`), every cell in the group should land on the *full* group minimum. The two-phase split guarantees all same-valued cells receive an identical, complete minimum.

### Pitfall 3: Forgetting That Fewer Teleports May Be Cheaper

The DP layer `cost[t]` represents reaching a cell with **exactly** `t` teleports, not "at most" `t`. A teleport is free but forces you to a landing cell that may sit *behind* a cheaper non-teleport path. Returning only `cost[k][rows-1][cols-1]` is a mistake:

```python
return cost[k][rows - 1][cols - 1]   # WRONG: forces exactly k teleports
```

You must take the minimum across all teleport counts:

```python
return min(cost[t][rows - 1][cols - 1] for t in range(k + 1))
```

### Pitfall 4: Memory Blow-Up From the Full 3D Table

The `O(k * m * n)` space can be prohibitive for large `k`. Notice that layer `t` depends only on layer `t - 1`. You can roll the DP onto two 2D arrays (`prev` and `curr`), reducing space to `O(m * n)`, while tracking the running answer for the bottom-right cell as you finish each layer:

```python
prev = [[inf] * cols for _ in range(rows)]
prev[0][0] = 0
# fill prev with the t=0 base DP ...
answer = prev[rows - 1][cols - 1]

for t in range(1, k + 1):
    curr = [[inf] * cols for _ in range(rows)]
    best_prev = inf
    for value in sorted_values:
        cells = value_to_cells[value]
        for i, j in cells:
            best_prev = min(best_prev, prev[i][j])
        for i, j in cells:
            curr[i][j] = best_prev
    for i in range(rows):
        for j in range(cols):
            if i:
                curr[i][j] = min(curr[i][j], curr[i - 1][j] + grid[i][j])
            if j:
                curr[i][j] = min(curr[i][j], curr[i][j - 1] + grid[i][j])
    answer = min(answer, curr[rows - 1][cols - 1])
    prev = curr

return answer
```

This preserves the same time complexity while shrinking memory from `O(k * m * n)` to `O(m * n)`.

