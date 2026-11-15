# 3562. Maximum Profit from Trading Stocks with Discounts

> Source: https://algo.monster/liteproblems/3562

## Problem Statement

You are given an integer `n`, representing the number of employees in a company. Each employee has a unique ID from `1` to `n`, where employee `1` is the CEO and is the direct or indirect boss of every other employee.

You are given two **1-based** integer arrays, `present` and `future`, each of length `n`, where:

- `present[i]` is the **current** price at which the `i`th employee can buy a stock today.
- `future[i]` is the **expected** price at which the `i`th employee can sell the stock tomorrow.

The company's hierarchy is described by a 2D integer array `hierarchy`, where `hierarchy[i] = [u_i, v_i]` means employee `u_i` is the **direct boss** of employee `v_i`. This structure forms a [tree](/problems/tree_intro) rooted at employee `1`.

You also have an integer `budget`, representing the total funds available for investment.

The company has a **discount policy**: if an employee's direct boss buys their own stock, then that employee can buy their stock at **half** the original price, i.e. `floor(present[v] / 2)`.

Your task is to return the **maximum** profit that can be achieved **without exceeding** the given `budget`.

**Notes:**

- You may buy each stock at most **once**.
- You **cannot** use any profit earned from future stock prices to fund additional investments; all purchases must come only from `budget`.

The profit gained from buying a stock at cost `cost` and selling it later is `future[v] - cost`, where `cost` is either `present[v]` (no discount) or `floor(present[v] / 2)` (if the direct boss bought their own stock). The goal is to choose which employees buy stocks so that the total spent stays within `budget` and the total profit is maximized.

## Intuition

The first thing to notice is the **structure** of the problem. Every employee has exactly one direct boss, and employee `1` is the boss of everyone. This means the hierarchy forms a **[tree](/problems/tree_intro)** rooted at employee `1`. Whenever we deal with a tree where each node's answer depends on its children, a natural idea is to solve it from the bottom up using **tree DFS**.

The second key observation is the **discount rule**: an employee's buying cost depends on whether their *direct boss* bought their own stock. The cost is `present[v]` normally, but drops to `floor(present[v] / 2)` if the boss bought. This tells us that when we process a node, its decision is influenced by what its parent decided. So for each node we should track **two scenarios**: one where the boss bought (so this node gets the discount) and one where the boss did not (full price). We capture this with an extra dimension `pre`, where `pre = 1` means the boss bought and `pre = 0` means the boss did not.

The third observation is the **budget constraint**. We have a limited amount of money, each stock can be bought at most once, and we want to maximize total profit. This is exactly the flavor of a **0/1 knapsack** problem: each employee is an "item" that either gets bought (paying its cost, gaining `future[v] - cost` profit) or skipped. The twist is that the items are arranged in a [tree](/problems/tree_intro), and the cost of children depends on whether the parent was bought.

Putting these ideas together, for each node `u` we define `f[j][pre]` as the maximum profit obtainable in the **subtree rooted at `u`** when:
- the available budget is at most `j`, and
- the parent's purchase state is `pre`.

To compute this, we first need to combine all of `u`'s children. Since the budget is shared among the children, we merge them one by one using a knapsack: for each child, we decide how much budget `j_v` to give it. The merge formula is

`nxt[j][pre] = max(nxt[j][pre], nxt[j - j_v][pre] + fv[j_v][pre])`

This builds up `nxt`, the best profit obtainable from `u`'s children for every budget level and every parent state.

Finally, we decide whether `u` itself buys. Its cost is `cost = present[u] // (pre + 1)`, which automatically applies the discount when `pre = 1`. If `u` buys, it consumes `cost` from the budget and earns `future[u] - cost`, and crucially its own children now see `pre = 1`. If `u` does not buy, the children see `pre = 0`. So:

`f[j][pre] = max(nxt[j][0], nxt[j - cost][1] + (future[u] - cost))`

The left side means `u` skips buying (children inherit state `0`), and the right side means `u` buys (children inherit state `1`), as long as the budget allows.

The CEO (employee `1`) has no boss, so we read the answer from `dfs(1)[budget][0]`, using `pre = 0` since there is no one above to grant a discount.

**Learn more about [Tree](/problems/tree_intro), [Depth-First Search](/problems/dfs_intro) and [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

We solve this with **[Tree](/problems/tree_intro) [Dynamic Programming](/problems/dynamic_programming_intro)** combined with a **0/1 knapsack** merge over the budget.

**Building the [tree](/problems/tree_intro).** We first construct an adjacency list `g`, where `g[u]` holds all the direct subordinates of employee `u`:

```python
g = [[] for _ in range(n + 1)]
for u, v in hierarchy:
    g[u].append(v)
```

**The DFS state.** For each node `u`, the function `dfs(u)` returns a `(budget + 1) x 2` array `f`, where `f[j][pre]` is the maximum profit achievable in the subtree rooted at `u` when:
- the budget is at most `j`, and
- `pre` is the purchase state of `u`'s direct boss (`pre = 1` means the boss bought, `pre = 0` means not).

**Merging children with a knapsack.** Before deciding whether `u` buys, we combine the results of all children into an array `nxt`, initialized to all zeros. The budget must be split among the children, so we treat each child as an item in a 0/1 knapsack. For each child `v`, we get `fv = dfs(v)` and merge it:

```python
for v in g[u]:
    fv = dfs(v)
    for j in range(budget, -1, -1):
        for jv in range(j + 1):
            for pre in (0, 1):
                val = nxt[j - jv][pre] + fv[jv][pre]
                if val > nxt[j][pre]:
                    nxt[j][pre] = val
```

Here `j` is the total budget given to all children processed so far, and `jv` is the portion allocated to the current child `v`. We iterate `j` from high to low so each child is counted only once. The merge follows the formula:

`nxt[j][pre] = max(nxt[j][pre], nxt[j - jv][pre] + fv[jv][pre])`

After all children are merged, `nxt[j][pre]` is the best profit from `u`'s subtree **excluding `u`'s own decision**, given budget `j` and boss state `pre`.

**Deciding whether `u` buys.** Now we compute `f`. The cost for `u` is `cost = present[u - 1] // (pre + 1)`, which is the full price when `pre = 0` and the half (discounted) price when `pre = 1`:

```python
for j in range(budget + 1):
    for pre in (0, 1):
        cost = present[u - 1] // (pre + 1)
        if j >= cost:
            f[j][pre] = max(nxt[j][0], nxt[j - cost][1] + (price - cost))
        else:
            f[j][pre] = nxt[j][0]
```

Two cases arise:
- If `j < cost`, `u` cannot afford its stock, so it must skip. Its children then see boss state `0`, giving `f[j][pre] = nxt[j][0]`.
- If `j >= cost`, `u` may either skip or buy:
  - **Skip:** profit is `nxt[j][0]` (children see boss state `0`).
  - **Buy:** profit is `nxt[j - cost][1] + (price - cost)`, where we spend `cost`, gain `future[u] - cost`, and children now see boss state `1`.

  We take the maximum of the two:

  `f[j][pre] = max(nxt[j][0], nxt[j - cost][1] + (price - cost))`

Note that the `pre` index in `f[j][pre]` only changes the **cost** of `u` itself (through the discount), while the children always branch on `u`'s own buy/skip decision.

**The answer.** The CEO (employee `1`) has no boss, so no discount applies and we read the result with `pre = 0`:

```python
return dfs(1)[budget][0]
```

**Complexity.** Let `B` be the budget. The knapsack merge at each node costs `O(B^2)` per child, and summed across the [tree](/problems/tree_intro) this gives an overall time complexity of roughly `O(n * B^2)`. The space used by the DP arrays is `O(n * B)` across the recursion stack.

## Example Walkthrough

Let's trace through a small example to see how the tree DP + knapsack works.

**Input:**
- `n = 3`
- `present = [4, 8, 6]`
- `future = [7, 10, 12]`
- `hierarchy = [[1, 2], [1, 3]]`
- `budget = 10`

**Step 1 — Build the tree.**

Employee `1` is the CEO and the direct boss of both `2` and `3`:

```
        1
       / \
      2   3
```

So the adjacency list is:
- `g[1] = [2, 3]`
- `g[2] = []`
- `g[3] = []`

**Step 2 — DFS the leaves first.**

We call `dfs(1)`, which recurses into its children before deciding for itself.

---

**`dfs(2)` (leaf):** Employee `2` has no children, so `nxt` stays all zeros. We compute `f` for every budget `j` and every boss-state `pre`.

- For `pre = 0` (boss did **not** buy): `cost = present[1] // 1 = 8 // 1 = 8`, profit if bought = `future[1] - 8 = 10 - 8 = 2`.
- For `pre = 1` (boss **bought**): `cost = present[1] // 2 = 8 // 2 = 4`, profit if bought = `10 - 4 = 6`.

Filling `f[j][pre] = max(nxt[j][0], nxt[j-cost][1] + (price - cost))` (both `nxt` terms are 0):

| `j`  | `f[j][0]` (cost 8) | `f[j][1]` (cost 4) |
|------|--------------------|--------------------|
| 0–3  | 0                  | 0                  |
| 4–7  | 0                  | 6                  |
| 8–10 | 2                  | 6                  |

So with the discount (`pre=1`), employee `2` is far more profitable: a profit of **6** for only **4** spent.

---

**`dfs(3)` (leaf):** Same idea for employee `3`.

- For `pre = 0`: `cost = present[2] // 1 = 6`, profit = `future[2] - 6 = 12 - 6 = 6`.
- For `pre = 1`: `cost = present[2] // 2 = 3`, profit = `12 - 3 = 9`.

| `j`  | `f[j][0]` (cost 6) | `f[j][1]` (cost 3) |
|------|--------------------|--------------------|
| 0–2  | 0                  | 0                  |
| 3–5  | 0                  | 9                  |
| 6–10 | 6                  | 9                  |

Again the discounted branch wins: profit **9** for just **3** spent.

---

**Step 3 — Merge children into `nxt` at node `1`.**

We start with `nxt` all zeros, then knapsack-merge `dfs(2)`, then `dfs(3)`.

After merging **both** children, `nxt[j][pre]` holds the best combined profit when total budget `j` is split between employees `2` and `3`, all under boss-state `pre`.

Look at the two states that matter for the CEO's decision:

- **`pre = 0` (CEO does not buy):** children pay full price. To buy both we need `8 + 6 = 14 > 10`, too much. Best single buy is employee `3` (profit 6, cost 6) or employee `2` (profit 2, cost 8). For `j = 10`: `nxt[10][0] = 6` (buy only `3`).

- **`pre = 1` (CEO bought):** children get the discount. Employee `2` costs 4 (profit 6), employee `3` costs 3 (profit 9). Buying both costs `4 + 3 = 7 ≤ 10`, total profit `6 + 9 = 15`. So `nxt[j][1] = 15` for `j ≥ 7`.

**Step 4 — Decide whether employee `1` buys.**

The CEO has no boss, so we evaluate with `pre = 0`, where `cost = present[0] // 1 = 4`, and buying earns `future[0] - 4 = 7 - 4 = 3`.

At `j = budget = 10`:

`f[10][0] = max(nxt[10][0], nxt[10 - 4][1] + 3) = max(6, nxt[6][1] + 3)`

Since `nxt[6][1] = 15` (we can afford both discounted children with budget 6: `4 + 3 = 7`... actually 7 > 6, so check `nxt[6][1]`).

Let's be precise. With budget 6 and both children discounted: buying both needs 7, not affordable. Best is buy `2` (cost 4, profit 6) **and** leftover 2 — can't afford `3` (needs 3). Or buy `3` (cost 3, profit 9) plus leftover 3 — can't afford `2` (needs 4). So `nxt[6][1] = 9`.

`f[10][0] = max(6, 9 + 3) = max(6, 12) = 12`

**Verifying the winning plan:**
- CEO buys: spend 4, earn 3. Children now get the discount.
- Remaining budget = 6, children see `pre = 1`.
- With 6 left, buy employee `3` (cost 3, earn 9). Remaining 3 — can't afford employee `2` (cost 4).
- **Total spent:** `4 + 3 = 7 ≤ 10`. **Total profit:** `3 + 9 = 12`.

**Step 5 — Read the answer.**

```python
return dfs(1)[budget][0]   # = 12
```

The maximum profit within the budget is **12**. The key insight the DP captures: the CEO buying its own stock (a modest profit of 3) *unlocks the discount* for the subtree, turning employee `3`'s purchase from cost-6 into cost-3 and making the whole plan more profitable than skipping the CEO.

## Python Solution

```python
from typing import List


class Solution:
    def maxProfit(
        self,
        n: int,
        present: List[int],
        future: List[int],
        hierarchy: List[List[int]],
        budget: int,
    ) -> int:
        # Build the adjacency list for the tree (1-indexed nodes).
        # graph[u] holds the direct subordinates (children) of u.
        graph: List[List[int]] = [[] for _ in range(n + 1)]
        for parent, child in hierarchy:
            graph[parent].append(child)

        def dfs(node: int) -> List[List[int]]:
            # combined[j][parent_bought] = best total profit from the children's
            # subtrees using budget j, where parent_bought indicates whether
            # the current `node` ends up buying its own stock (which gives its
            # children the 50% discount).
            combined = [[0, 0] for _ in range(budget + 1)]

            for child in graph[node]:
                child_dp = dfs(child)
                # Group knapsack: merge each child's dp into combined.
                # Iterate budget descending to keep a 0/1-style combine per child.
                for j in range(budget, -1, -1):
                    for spent in range(j + 1):
                        for parent_bought in (0, 1):
                            value = (
                                combined[j - spent][parent_bought]
                                + child_dp[spent][parent_bought]
                            )
                            if value > combined[j][parent_bought]:
                                combined[j][parent_bought] = value

            # dp[j][parent_bought] = best profit in this whole subtree with
            # budget j, where parent_bought tells whether THIS node's parent
            # bought its stock (so this node may get the discount).
            dp = [[0, 0] for _ in range(budget + 1)]
            sell_price = future[node - 1]

            for j in range(budget + 1):
                for parent_bought in (0, 1):
                    # Cost of buying this node: half price if the parent bought.
                    cost = present[node - 1] // (parent_bought + 1)
                    if j >= cost:
                        # Option 1: do not buy this node -> children see parent_bought = 0.
                        # Option 2: buy this node -> children see parent_bought = 1,
                        #           and we add the profit (sell_price - cost).
                        dp[j][parent_bought] = max(
                            combined[j][0],
                            combined[j - cost][1] + (sell_price - cost),
                        )
                    else:
                        # Not enough budget to buy this node; must skip it.
                        dp[j][parent_bought] = combined[j][0]

            return dp

        # Root (node 1) has no parent, so parent_bought = 0; use full budget.
        return dfs(1)[budget][0]
```

## Time and Space Complexity

**Time Complexity: `O(n × budget²)`**

The code performs a tree DP via the `dfs` function, which is called once for each of the `n` nodes (since the hierarchy forms a tree rooted at node `1`).

- For each node `u`, the merging of children's DP results runs a nested loop:
  ```python
  for j in range(budget, -1, -1):
      for jv in range(j + 1):
          for pre in (0, 1):
              ...
  ```
  The outer loop runs `budget + 1` times and the inner loop runs up to `j + 1` times, giving `O(budget²)` work per child. A subtle but crucial point: when summing this knapsack-merge cost over all edges across the entire tree, the total work is bounded by `O(n × budget²)`. This is because each node is merged into its parent exactly once, so the merging cost is distributed across the `n - 1` edges, contributing `O(n × budget²)` overall.

- The second loop in each `dfs` call computes `f` and runs `O(budget)` times per node, contributing `O(n × budget)` in total, which is dominated by the merging cost.

Therefore, the total time complexity is `O(n × budget²)`.

**Space Complexity: `O(n × budget)`**

- Each call to `dfs` allocates the arrays `nxt` and `f`, each of size `O(budget)`.
- Due to the recursion, multiple `dfs` frames can be active simultaneously along a path from the root to a leaf. In the worst case (a degenerate chain-like tree), the recursion depth reaches `O(n)`, and each frame holds `O(budget)` space.
- This yields a stack-related space usage of `O(n × budget)`.

Additionally, the adjacency list `g` requires `O(n)` space. The dominant term is the recursion stack, so the overall space complexity is `O(n × budget)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### Pitfall 1: Iterating the knapsack budget in ascending order (item reuse)

The most frequent mistake in the **group-knapsack merge** is iterating the total budget `j` in **ascending** order instead of **descending** order. Although the merge here reads from a *separate* `child_dp` array (rather than the same array being written, as in a classic 1D knapsack), the subtle danger appears when people try to "optimize" by collapsing `combined` into the merge in place, or when they reverse the loop directions out of habit.

The correct loop must process `j` from high to low **so that the `combined[j - spent]` value used as a base never already includes the current child's contribution**:

```python
for j in range(budget, -1, -1):       # CORRECT: descending
    for spent in range(j + 1):
        ...
```

If you write `for j in range(budget + 1):` (ascending) **and** you ever change `combined[j - spent]` to refer to an updated value of the *same* child, you would let one child be allocated budget multiple times, inflating profit. Keep the descending order as a defensive habit, and never write back into a slot you will read again for the same child.

### Pitfall 2: Forgetting that `pre` only affects `node`'s own cost, not its children

A very common conceptual error is to propagate the `parent_bought` flag straight through to the children. People mistakenly write:

```python
# WRONG
dp[j][parent_bought] = max(
    combined[j][parent_bought],                       # wrong index
    combined[j - cost][parent_bought] + (sell_price - cost),
)
```

The flag `parent_bought` describes whether **this node's** boss bought, which only changes **this node's** discounted `cost`. What the *children* care about is whether **this node itself** bought. Therefore:

- The **skip** branch must read `combined[j][0]` (this node did not buy → children get no discount).
- The **buy** branch must read `combined[j - cost][1]` (this node bought → children get the discount).

Using `parent_bought` as the index into `combined` mixes up two different "who bought" questions and produces wrong answers on any tree deeper than two levels.

### Pitfall 3: Computing the discounted cost incorrectly

The discount is `floor(present[v] / 2)`, elegantly expressed as `present[node - 1] // (parent_bought + 1)`:

- `parent_bought = 0` → divide by `1` → full price.
- `parent_bought = 1` → divide by `2` → floored half price.

Pitfalls here include:
- Applying the discount with `present[node - 1] / 2` (true division) instead of `//` (floor division), which yields a float and breaks integer budget comparisons.
- Applying the discount to the **selling** price `future` as well — only the **buying** cost is discounted; `sell_price` is always `future[node - 1]`.
- Off-by-one indexing: arrays are described as 1-based in the statement but stored 0-based, so the node's own data is at `present[node - 1]` / `future[node - 1]`.

### Pitfall 4: Recursion depth and performance on large/deep trees

`dfs` is recursive, and a degenerate "chain" tree (each boss has exactly one subordinate) reaches depth `n`. For large `n` this can hit Python's default recursion limit. Mitigations:

```python
import sys
sys.setrecursionlimit(n + 10)
```

Additionally, the `O(n · B²)` knapsack can be slow for large budgets. A practical optimization is to cap the inner `spent` loop by the budget actually consumable in the child's subtree, or to track each subtree's maximum useful spend and avoid iterating past it — this prunes the quadratic merge significantly when subtrees are small.

### Pitfall 5: Reading the final answer with the wrong flag

The CEO (node `1`) has no boss, so no discount can ever apply to it. The answer must be read with `parent_bought = 0`:

```python
return dfs(1)[budget][0]   # CORRECT
```

Reading `dfs(1)[budget][1]` would incorrectly grant the CEO a phantom half-price discount that has no parent to justify it, overstating the profit.

