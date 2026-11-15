# 3600. Maximize Spanning Tree Stability with Upgrades

> Source: https://algo.monster/liteproblems/3600

## Problem Statement

You are given an integer `n`, representing `n` nodes numbered from `0` to `n - 1`, and a list of `edges`, where each `edges[i] = [uᵢ, vᵢ, sᵢ, mustᵢ]` describes the following:

- `uᵢ` and `vᵢ` indicate an **undirected edge** between nodes `uᵢ` and `vᵢ`.
- `sᵢ` is the **strength** of the edge.
- `mustᵢ` is an integer (`0` or `1`). If `mustᵢ == 1`, the edge **must** be included in the spanning tree, and such edges **cannot** be upgraded.

You are also given an integer `k`, which is the **maximum** number of upgrades you can perform. Each upgrade **doubles** the strength of an edge, and each eligible edge (one with `mustᵢ == 0`) can be upgraded **at most once**.

The **stability** of a spanning tree is defined as the **minimum** strength score among all edges included in it.

Your task is to return the **maximum** possible stability of any valid spanning tree. If it is impossible to connect all the nodes, return `-1`.

A **spanning tree** of a [graph](/problems/graph_intro) with `n` nodes is a subset of the edges that connects all nodes together (i.e., the graph is **connected**) without forming any cycles, and uses **exactly** `n - 1` edges.

**In summary**, you need to:

1. Build a spanning tree that **includes every required edge** (those with `mustᵢ == 1`).
2. Optionally upgrade up to `k` non-required edges, each upgrade **doubling** the strength of one edge (each such edge upgraded at most once).
3. **Maximize** the smallest edge strength in the resulting spanning tree.
4. Return `-1` if no valid spanning tree exists (for example, the [graph](/problems/graph_intro) cannot be connected, or the required edges already form a cycle).

## Intuition

The key observation lies in the definition of **stability**: it is the **minimum** strength among all edges in the spanning tree. We want to make this minimum value as **large** as possible.

This kind of "maximize the minimum" objective is a strong hint toward **[binary search](/problems/binary_search_intro) on the answer**. Why? Because the feasibility of a target stability value is **monotonic**. If we can build a valid spanning tree where every edge has strength at least `x`, then we can certainly build one where every edge has strength at least `y` for any `y < x` (the constraint `y` is looser, so any tree satisfying `x` also satisfies `y`). This monotonic property is exactly what makes binary search applicable.

So the problem reduces to: **for a given target stability `lim`, can we build a valid spanning tree where every edge has strength at least `lim`?**

To answer this `check(lim)` question, we think about which edges are usable when the target is `lim`:

- Any edge whose original strength `s >= lim` can be used **directly**, no upgrade needed.
- Any edge whose strength is below `lim` but where `s * 2 >= lim` can still be used **if we spend one upgrade on it** (doubling brings it up to the requirement). Since each eligible edge can be upgraded at most once, doubling is the best we can do for a single edge.
- Edges with `s * 2 < lim` are useless for this target, even with an upgrade.

Now connecting the nodes naturally becomes a **Union-Find** task. We first merge all nodes using the "free" edges (those with `s >= lim`), since using them costs nothing. Only after that do we spend our limited `k` upgrades on the weaker-but-upgradable edges (`s * 2 >= lim`), and only when such an edge actually joins two **separate** components (otherwise the upgrade would be wasted on a cycle). If after this process all nodes belong to a single component, then `lim` is achievable.

There are two important details handled **before** the [binary search](/problems/binary_search_intro):

1. The **required edges** (`must == 1`) must always be in the tree. They directly limit the answer, because the final stability can never exceed the smallest strength among these forced edges. This gives us the natural **upper bound** `mn` for [binary search](/problems/binary_search_intro). We also union these edges first; if any of them forms a cycle, the required set is invalid and we return `-1`.
2. If, after trying to union with **all** edges, the [graph](/problems/graph_intro) is still not fully connected (`cnt > 1`), then no spanning tree exists at all, so we return `-1`.

With the search range fixed at `[1, mn]` and a clear `check` function, [binary search](/problems/binary_search_intro) efficiently narrows down to the **largest** feasible stability, which is our answer.

**Learn more about [Greedy](/problems/greedy_intro), [Union Find](/problems/dsu_intro), [Graph](/problems/graph_intro), [Binary Search](/problems/binary_search_intro) and [Minimum Spanning Tree](/problems/mst_forest) patterns.**

## Solution Approach

We solve this problem using **[Binary Search](/problems/binary_search_intro) + Union-Find**.

### Step 1: Set up the Union-Find data structure

We use a `UnionFind` class that supports the standard operations with two common optimizations:

- **Path compression** in the `find` method, so that lookups become nearly constant time.
- **Union by size** in the `union` method, attaching the smaller tree under the larger one.

It also tracks `cnt`, the current number of connected components. Whenever a successful union merges two components, `cnt` is decreased by `1`. When `cnt == 1`, all nodes are connected. The `union` method returns `False` when the two nodes are already in the same component (i.e., adding this edge would form a cycle).

### Step 2: Handle required edges and determine the upper bound

We first iterate over all edges and process the **required** ones (`must == 1`):

- Record the **minimum strength** `mn` among them, since the final stability can never exceed this value.
- Union the two endpoints of each required edge. If a `union` call returns `False`, the required edges contain a cycle, so a valid spanning tree is impossible and we return `-1`.

This gives us the search upper bound `mn`.

### Step 3: Check overall connectivity

Next, we add **all** edges into the Union-Find (continuing from the required-edge state). If the final number of components `uf.cnt > 1`, the [graph](/problems/graph_intro) simply cannot be fully connected, so we return `-1`.

### Step 4: The `check(lim)` feasibility function

For a candidate stability `lim`, we determine whether a valid spanning tree with every edge strength at least `lim` exists:

1. Create a **fresh** Union-Find.
2. First pass: union all edges whose original strength satisfies `s >= lim`. These are "free" edges that meet the requirement without any upgrade.
3. Second pass: with remaining upgrade budget `rem = k`, scan edges where `s * 2 >= lim`. For each such edge, if it connects two **separate** components (`uf.union(u, v)` returns `True`), we spend one upgrade and decrement `rem`. We stop spending once `rem` reaches `0`.
4. Return whether `uf.cnt == 1` (all nodes connected).

The order matters here: we prioritize free edges first, so that the precious `k` upgrades are only consumed on edges that genuinely need them to bridge disconnected parts.

### Step 5: [Binary search](/problems/binary_search_intro) for the maximum stability

We [binary search](/problems/binary_search_intro) over the range `[1, mn]`:

```python
l, r = 1, mn
while l < r:
    mid = (l + r + 1) >> 1
    if check(mid):
        l = mid
    else:
        r = mid - 1
return l
```

Because we are searching for the **largest** feasible value, we use the upper-biased midpoint `mid = (l + r + 1) >> 1`:

- If `check(mid)` is `True`, the answer can be at least `mid`, so we move `l = mid`.
- Otherwise, `mid` is too large, so we shrink the range with `r = mid - 1`.

When the loop ends, `l` holds the **maximum possible stability**.

### Complexity Analysis

- **Time complexity**: `O(m × log(mn) × α(n))`, where `m` is the number of edges, `mn` is the upper bound of stability, and `α` is the inverse Ackermann function from Union-Find. Each `check` call scans all edges twice, and we perform `O(log(mn))` such calls during [binary search](/problems/binary_search_intro).
- **Space complexity**: `O(n)`, used by the Union-Find arrays `p` and `size`.

## Example Walkthrough

Let's trace through a concrete example to see how **Binary Search + Union-Find** solves this problem.

**Input:**
- `n = 4` (nodes `0, 1, 2, 3`)
- `k = 1` (at most one upgrade allowed)
- `edges`:

| Index | u | v | s (strength) | must |
|-------|---|---|--------------|------|
| 0 | 0 | 1 | 6 | 1 |
| 1 | 1 | 2 | 3 | 0 |
| 2 | 2 | 3 | 5 | 0 |
| 3 | 0 | 3 | 2 | 0 |
| 4 | 1 | 3 | 4 | 0 |

---

#### Step 1 & 2: Process required edges and find the upper bound

Only edge `0` has `must == 1`:
- Record `mn = 6` (the minimum strength among required edges → our **upper bound**).
- Union nodes `0` and `1`. This succeeds (no cycle).

Components so far: `{0, 1}`, `{2}`, `{3}` → `cnt = 3`.

Since the only required edge formed no cycle, we continue.

---

#### Step 3: Check overall connectivity

Add **all** remaining edges into Union-Find to test if the graph can connect at all:
- Edge `1` (1–2): merges `{0,1}` and `{2}` → `{0,1,2}`, `{3}`
- Edge `2` (2–3): merges `{0,1,2}` and `{3}` → `{0,1,2,3}` → `cnt = 1`

`cnt == 1`, so the graph **can** be connected. We proceed to binary search. (If `cnt > 1` here, we'd return `-1`.)

---

#### Step 5: Binary search over `[1, mn] = [1, 6]`

We search for the **largest** stability that passes `check(lim)`.

**Iteration 1:** `l=1, r=6`, `mid = (1+6+1)>>1 = 4`. Run `check(4)`:

- *Free pass* (`s >= 4`): edge0(6), edge2(5), edge4(4).
  - 0–1 ✓ → `{0,1}`
  - 2–3 ✓ → `{2,3}`
  - 1–3 ✓ → merges → `{0,1,2,3}`, `cnt = 1`
- Already connected with free edges, no upgrades needed. **Returns `True`.**
- `check(4)` is `True` → set `l = 4`.

**Iteration 2:** `l=4, r=6`, `mid = (4+6+1)>>1 = 5`. Run `check(5)`:

- *Free pass* (`s >= 5`): edge0(6), edge2(5).
  - 0–1 ✓ → `{0,1}`
  - 2–3 ✓ → `{2,3}`
  - Components: `{0,1}`, `{2,3}`, `cnt = 2`
- *Upgrade pass* (`s*2 >= 5`, i.e. `s >= 2.5`), `rem = 1`:
  - edge1(1–2, s=3, 3*2=6≥5): connects `{0,1}` and `{2,3}` ✓ → spend 1 upgrade → `{0,1,2,3}`, `cnt = 1`, `rem = 0`.
- **Returns `True`.**
- `check(5)` is `True` → set `l = 5`.

**Iteration 3:** `l=5, r=6`, `mid = (5+6+1)>>1 = 6`. Run `check(6)`:

- *Free pass* (`s >= 6`): edge0(6).
  - 0–1 ✓ → `{0,1}`, `{2}`, `{3}`, `cnt = 3`
- *Upgrade pass* (`s*2 >= 6`, i.e. `s >= 3`), `rem = 1`:
  - edge1(1–2, s=3, 6≥6): connects `{0,1}` and `{2}` ✓ → spend upgrade → `{0,1,2}`, `{3}`, `cnt = 2`, `rem = 0`.
  - Budget exhausted, but node `3` is still isolated.
- **Returns `False`** (`cnt == 2`).
- `check(6)` is `False` → set `r = 6 - 1 = 5`.

**Loop ends:** `l == r == 5`.

---

#### Result

The maximum possible stability is **`5`**.

**Verifying the winning tree:** edges `{0–1 (6), 2–3 (5), 1–2 (3→upgraded to 6)}` form a valid spanning tree connecting all 4 nodes, includes the required edge `0–1`, uses exactly one upgrade (≤ k), and its minimum strength is `min(6, 5, 6) = 5`. This matches our answer, and we confirmed `6` is unreachable because only one upgrade is available — never enough to lift two weak edges simultaneously to bridge all components.

## Python Solution

```python
from typing import List


class UnionFind:
    def __init__(self, n: int) -> None:
        # parent[i] is the parent of node i; initially each node is its own root
        self.parent = list(range(n))
        # size[i] is the number of nodes in the tree rooted at i
        self.size = [1] * n
        # count tracks the number of disjoint connected components
        self.count = n

    def find(self, x: int) -> int:
        # Find the root of x with path compression
        if self.parent[x] != x:
            self.parent[x] = self.find(self.parent[x])
        return self.parent[x]

    def union(self, a: int, b: int) -> bool:
        # Merge the sets containing a and b; return False if already connected
        root_a, root_b = self.find(a), self.find(b)
        if root_a == root_b:
            return False
        # Union by size: attach the smaller tree under the larger one
        if self.size[root_a] > self.size[root_b]:
            self.parent[root_b] = root_a
            self.size[root_a] += self.size[root_b]
        else:
            self.parent[root_a] = root_b
            self.size[root_b] += self.size[root_a]
        self.count -= 1
        return True


class Solution:
    def maxStability(self, n: int, edges: List[List[int]], k: int) -> int:
        def check(limit: int) -> bool:
            # Determine whether all nodes can be connected such that every
            # edge used has an effective stability >= limit.
            uf = UnionFind(n)
            # First, use all edges whose original stability already meets the limit (free).
            for u, v, strength, _ in edges:
                if strength >= limit:
                    uf.union(u, v)
            # Then, use upgradeable edges (doubled stability) up to k times,
            # only if they actually connect two separate components.
            remaining = k
            for u, v, strength, _ in edges:
                if strength * 2 >= limit and remaining > 0:
                    if uf.union(u, v):
                        remaining -= 1
            # The graph is fully connected when only one component remains.
            return uf.count == 1

        # Validate mandatory edges and compute the upper bound for the binary search.
        uf = UnionFind(n)
        min_must_strength = 10**6
        for u, v, strength, must in edges:
            if must:
                # The answer can never exceed the smallest mandatory edge's stability.
                min_must_strength = min(min_must_strength, strength)
                # Mandatory edges forming a cycle make the configuration invalid.
                if not uf.union(u, v):
                    return -1

        # Add all remaining edges to verify overall connectivity is achievable.
        for u, v, _, _ in edges:
            uf.union(u, v)
        if uf.count > 1:
            return -1

        # Binary search for the maximum achievable minimum stability.
        left, right = 1, min_must_strength
        while left < right:
            mid = (left + right + 1) >> 1
            if check(mid):
                left = mid
            else:
                right = mid - 1
        return left
```

## Time and Space Complexity

**Time Complexity:** `O((m × α(n) + n) × log M)`

The analysis is as follows:

- **Binary search range:** The binary search is performed over the strength values in the range `[1, mn]`, where `mn` is bounded by the maximum edge strength `M`. This results in `O(log M)` iterations.

- **Cost per `check(lim)` call:** Inside each invocation:
  - A new `UnionFind(n)` instance is created, costing `O(n)` for initializing the `p`, `size` arrays.
  - The edges are iterated twice. Each iteration processes all `m` edges, and each `union` / `find` operation costs `O(α(n))` amortized time (where `α` is the inverse Ackermann function) due to path compression and union by size. This contributes `O(m × α(n))`.
  - Thus, each `check` call costs `O(m × α(n) + n)`.

- **Combining:** The binary search runs `check` for `O(log M)` iterations, giving `O((m × α(n) + n) × log M)`.

- **Preprocessing before binary search:** Building the initial `UnionFind` and processing the must-have edges and connectivity check costs `O(n + m × α(n))`, which is dominated by the binary search portion.

Therefore, the overall time complexity is `O((m × α(n) + n) × log M)`.

**Space Complexity:** `O(n)`

The `UnionFind` data structure uses the `p` and `size` arrays, each of length `n`, requiring `O(n)` space. The recursive `find` with path compression may add `O(n)` stack depth in the worst case. No additional structures scale beyond `O(n)`, so the total space complexity is `O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### Pitfall 1: Forgetting to force-include mandatory edges inside `check(limit)`

The most dangerous bug in this solution is that the feasibility function `check(limit)` **completely ignores the `must` flag**. It only unions edges based on whether their (possibly doubled) strength meets `limit`:

```python
for u, v, strength, _ in edges:   # the must flag is discarded with `_`
    if strength >= limit:
        uf.union(u, v)
```

This means a mandatory edge whose strength is **below** `limit` will silently be left out of the candidate spanning tree, yet the algorithm may still happily connect all the nodes using *other* edges and report success. The returned tree then violates the core requirement that **every required edge must be included**.

Concretely, suppose a `must == 1` edge has strength `3`, but the rest of the graph can be connected with edges of strength `5`. The current code would happily return `5` (because the mandatory edge is simply never selected), even though any valid spanning tree is *forced* to contain that strength-`3` edge, capping the true answer at `3`.

The reason the bug is partially masked is that `min_must_strength` clamps the binary-search **upper bound** to the smallest mandatory strength. So for `limit <= min_must_strength`, the mandatory edges *happen* to satisfy `strength >= limit` and get unioned anyway. This makes the code produce the correct answer **only because** the upper bound equals the minimum mandatory strength — a fragile coincidence rather than correct logic. If the upper bound logic were ever changed, or if you reasoned about a mandatory edge that could be upgraded, the bug would surface immediately.

**Solution:** Always union mandatory edges first inside `check`, unconditionally, and detect cycles among them:

```python
def check(limit: int) -> bool:
    uf = UnionFind(n)
    # 1. Mandatory edges MUST be included regardless of strength.
    for u, v, strength, must in edges:
        if must:
            uf.union(u, v)          # always include
    # 2. Free edges that meet the limit.
    for u, v, strength, must in edges:
        if not must and strength >= limit:
            uf.union(u, v)
    # 3. Upgradeable edges, spending at most k upgrades on edges
    #    that actually bridge two components.
    remaining = k
    for u, v, strength, must in edges:
        if not must and strength < limit and strength * 2 >= limit and remaining > 0:
            if uf.union(u, v):
                remaining -= 1
    return uf.count == 1
```

Note the refinement in step 3: only edges with `strength < limit` need an upgrade. An edge that already satisfies `strength >= limit` was handled for free in step 2, so spending an upgrade on it would waste budget.

---

### Pitfall 2: Wasting upgrade budget on edges that already meet the limit

In the original second pass, the condition is only `strength * 2 >= limit`:

```python
if strength * 2 >= limit and remaining > 0:
    if uf.union(u, v):
        remaining -= 1
```

An edge with `strength >= limit` also satisfies `strength * 2 >= limit`. If such an edge was *not* used in the first pass (because it would have formed a cycle at that moment) but later becomes a bridging edge, the code "spends" an upgrade on it even though no upgrade is needed. This prematurely drains `remaining`, potentially causing `check` to return `False` for a `limit` that is actually feasible — yielding an answer that is too small.

**Solution:** Restrict the upgrade pass to edges that genuinely require an upgrade by adding `strength < limit`:

```python
if strength < limit and strength * 2 >= limit and remaining > 0:
    if uf.union(u, v):
        remaining -= 1
```

---

### Pitfall 3: Off-by-one in the binary search midpoint causing an infinite loop

Because we search for the **maximum** feasible value, the midpoint must be **upper-biased**:

```python
mid = (left + right + 1) >> 1
if check(mid):
    left = mid          # keep mid as a candidate
else:
    right = mid - 1
```

If you mistakenly use the standard lower-biased midpoint `mid = (left + right) >> 1` together with the `left = mid` branch, then when `right == left + 1`, `mid` evaluates to `left`, and `left = mid` leaves the interval unchanged — an **infinite loop**. The `+ 1` in the midpoint is essential to guarantee progress when assigning `left = mid`.

**Solution:** Pair the assignment style with the matching midpoint bias:
- When the "keep" branch does `left = mid`, use `mid = (left + right + 1) >> 1`.
- When the "keep" branch does `right = mid`, use `mid = (left + right) >> 1`.

---

### Pitfall 4: Integer overflow / wrong initial upper bound when no mandatory edges exist

`min_must_strength` is initialized to `10**6`. If there are **no** mandatory edges, this sentinel survives and becomes the binary-search upper bound `right`. This is only correct if `10**6` is a genuine upper bound on the achievable stability. If an edge's doubled strength can exceed `10**6` (i.e., original strength approaching or above `10**6`), the true answer could be larger than the sentinel, and the binary search will never explore it — silently capping the result.

**Solution:** When there are no mandatory edges, derive the upper bound from the data itself, accounting for the doubling:

```python
if min_must_strength == 10**6 and not any(must for *_, must in edges):
    # No mandatory edge: the cap is the largest possible upgraded strength.
    min_must_strength = max((s * 2 for u, v, s, _ in edges), default=0)
```

This guarantees the search range covers every reachable stability value, regardless of the input magnitude.

