# 3620. Network Recovery Pathways

> Source: https://algo.monster/liteproblems/3620

## Problem Statement

You are given a [directed acyclic graph](/problems/graph_intro) with `n` nodes labeled `0` through `n - 1`. The 2D array `edges` has `m` entries, where `edges[i] = [uᵢ, vᵢ, costᵢ]` describes a one-way link from node `uᵢ` to node `vᵢ` that takes `costᵢ` units to recover.

Some nodes are down. The boolean array `online` records which ones are working: `online[i] = true` means node `i` is up. Nodes `0` and `n - 1` are guaranteed to be up.

A route from node `0` to node `n - 1` counts as **valid** when both of the following hold:

- Every node strictly between the two endpoints is online.
- The recovery costs of the edges along the route add up to at most `k`.

Each valid route earns a **score** equal to the cheapest edge that appears on it. Return the largest score over all valid routes. If no valid route exists, return `-1`.

**Example 1:** Given `edges = [[0,1,5],[1,3,10],[0,2,3],[2,3,4]]`, `online = [true,true,true,true]`, and `k = 10`, the output is `3`. Two routes lead from node `0` to node `3`. The route `0 → 1 → 3` sums to `5 + 10 = 15`, which is over the budget of `10`, so it is invalid. The route `0 → 2 → 3` sums to `3 + 4 = 7`, which fits the budget, and its cheapest edge is `min(3, 4) = 3`. That is the only valid route, so the answer is `3`.

**Example 2:** Given `edges = [[0,1,7],[1,4,5],[0,2,6],[2,3,6],[3,4,2],[2,4,6]]`, `online = [true,true,true,false,true]`, and `k = 12`, the output is `6`. Node `3` is down, so the route `0 → 2 → 3 → 4` is ruled out no matter what it costs. The route `0 → 1 → 4` totals `7 + 5 = 12` with cheapest edge `5`; the route `0 → 2 → 4` totals `6 + 6 = 12` with cheapest edge `6`. The better of the two scores is `6`.

**Constraints:**

- `n == online.length` and `2 <= n <= 5 * 10⁴`
- `0 <= m == edges.length <= min(10⁵, n * (n - 1) / 2)`
- `edges[i] = [uᵢ, vᵢ, costᵢ]` with `0 <= uᵢ, vᵢ < n` and `uᵢ != vᵢ`
- `0 <= costᵢ <= 10⁹`
- `0 <= k <= 5 * 10¹³`
- `online[0]` and `online[n - 1]` are both `true`
- The graph contains no directed cycles.

## Intuition

Every route is judged by two different numbers at once: its **total** cost, which must stay within the budget `k`, and its **cheapest edge**, which we want to make as large as possible. A single traversal struggles to optimize both together. A route whose edges are all expensive scores well but may spend too much in total, while the cheapest route overall may cross one very cheap edge and score poorly. The two objectives pull in opposite directions.

The way out is to turn the optimization into a yes/no question. Instead of asking "what is the best score?", fix a guess `x` and ask: **does some valid route use only edges costing at least `x`?** This question has a useful monotone structure. If a route qualifies for `x`, the very same route qualifies for every smaller guess, because each of its edges is still expensive enough. So as `x` grows, the answer flips from yes to no exactly once, and the largest `x` that still gets a yes is the score we want. That is the setting for a [binary search](/problems/binary_search_intro) on the answer. Since the score of any route equals one of its edge costs, only the distinct edge costs need to be tried as guesses.

For a fixed guess `x`, the check becomes an ordinary shortest-path problem. Discard every edge cheaper than `x` and every edge that touches an offline node, then compute the cheapest total from node `0` to node `n - 1` in what remains. The guess is feasible exactly when that cheapest total is at most `k` — if even the cheapest qualifying route blows the budget, every other qualifying route does too. All costs are non-negative, so Dijkstra with a min-[heap](/problems/heap_intro) answers this directly. The input happens to be acyclic, so a relaxation in topological order would work as well; Dijkstra does not need the ordering.

Offline nodes need no special machinery during the search: a valid route may only pass through online intermediate nodes, and the two endpoints are always online, so keeping an edge only when **both** of its endpoints are online removes every forbidden route and nothing else.

## Solution Approach

### Solution: Binary Search on the Answer + Dijkstra

The solution has three parts: filter the graph once, define a feasibility test, and binary search over the candidate answers.

### Filtering the graph

Build an adjacency list that keeps an edge `[u, v, cost]` only when `online[u]` and `online[v]` are both true. Because nodes `0` and `n - 1` are always online, a route in the filtered graph is exactly a route whose intermediate nodes are all online. While filtering, collect the costs of the kept edges; after sorting and removing duplicates they form the candidate answers. If no edge survives, no route can exist and the answer is `-1`.

### The feasibility test: Dijkstra with a cost floor

`feasible(x)` asks whether some route from `0` to `n - 1` uses only edges costing at least `x` and has total cost at most `k`. Run Dijkstra from node `0`, but while relaxing, skip any edge with `cost < x`, and only accept a new distance `d + cost` when it does not exceed `k`. Because every pushed distance already respects the budget, the test can stop with a yes as soon as node `n - 1` is popped from the heap. If the heap drains first, the target is unreachable within budget and the test answers no.

Distances must be stored in 64-bit integers: a route can traverse up to `10⁵` edges of cost up to `10⁹`, and `k` itself reaches `5 * 10¹³`, both far beyond a 32-bit range.

### Binary search over distinct edge costs

Let `costs` be the sorted distinct usable edge costs. Standard binary search finds the largest element for which `feasible` answers yes:

```
lo, hi = 0, len(costs) - 1
ans = -1
while lo <= hi:
    mid = (lo + hi) // 2
    if feasible(costs[mid]):
        ans = costs[mid]      # record and try a larger guess
        lo = mid + 1
    else:
        hi = mid - 1          # too strict; try a smaller guess
return ans
```

If no guess is feasible — including the smallest edge cost, which permits every usable edge — then no valid route exists at all and `ans` stays `-1`.

**Complexity Analysis:**

- **Time complexity:** `O((n + m) · log n · log m)`. Sorting the distinct costs takes `O(m log m)`. The binary search performs `O(log m)` feasibility tests, and each test is one Dijkstra run costing `O((n + m) log n)`.
- **Space complexity:** `O(n + m)` for the adjacency list, the distance array, and the heap.

## Example Walkthrough

Let's trace Example 2: `edges = [[0,1,7],[1,4,5],[0,2,6],[2,3,6],[3,4,2],[2,4,6]]`, `online = [true,true,true,false,true]`, `k = 12`.

#### Phase 1: Filter the graph

Node `3` is offline, so every edge touching it is dropped.

| Edge | Online check | Kept? |
|------|--------------|-------|
| `0 → 1` (cost 7) | both online | yes |
| `1 → 4` (cost 5) | both online | yes |
| `0 → 2` (cost 6) | both online | yes |
| `2 → 3` (cost 6) | node 3 offline | no |
| `3 → 4` (cost 2) | node 3 offline | no |
| `2 → 4` (cost 6) | both online | yes |

The distinct usable costs, sorted, are `costs = [5, 6, 7]`.

#### Phase 2: Binary search

Start with `lo = 0`, `hi = 2`, `ans = -1`.

**Iteration 1 — `mid = 1`, guess `x = 6`.**
Edges costing at least `6`: `0 → 1` (7), `0 → 2` (6), `2 → 4` (6). The edge `1 → 4` (5) is off limits. Dijkstra from node `0`:

| Pop | Relaxations |
|-----|-------------|
| `(0, node 0)` | node 1 gets `7`, node 2 gets `6` (both `<= 12`) |
| `(6, node 2)` | node 4 gets `6 + 6 = 12 <= 12` |
| `(7, node 1)` | its only edge `1 → 4` costs `5 < 6`, skipped |
| `(12, node 4)` | target reached within budget — **feasible** |

Record `ans = 6` and move `lo = 2`.

**Iteration 2 — `mid = 2`, guess `x = 7`.**
Only `0 → 1` (7) survives the floor. Dijkstra reaches node `1` at distance `7`, but the edge `1 → 4` costs `5 < 7` and is skipped. The heap drains without touching node `4` — **infeasible**. Move `hi = 1`.

Now `lo = 2 > hi = 1`, so the search stops.

#### Result

The answer is `ans = 6`, matching the expected output: the route `0 → 2 → 4` spends exactly `12` and never uses an edge cheaper than `6`.

## Python Solution

```python
import heapq
from typing import List


class Solution:
    def findMaxPathScore(self, edges: List[List[int]], online: List[bool], k: int) -> int:
        n = len(online)

        # Build the adjacency list, keeping only edges whose endpoints are
        # both online. Nodes 0 and n-1 are guaranteed online, so a route in
        # the filtered graph is exactly a route with online intermediates.
        graph = [[] for _ in range(n)]
        cost_set = set()
        for u, v, cost in edges:
            if online[u] and online[v]:
                graph[u].append((v, cost))
                cost_set.add(cost)

        # No usable edge means node n-1 cannot be reached at all.
        if not cost_set:
            return -1

        # The score of any route equals one of its edge costs, so the
        # sorted distinct usable costs are the candidate answers.
        costs = sorted(cost_set)

        def feasible(threshold: int) -> bool:
            # Dijkstra restricted to edges with cost >= threshold.
            # Returns True if some 0 -> n-1 route has total cost <= k.
            dist = [float("inf")] * n
            dist[0] = 0
            heap = [(0, 0)]  # (distance so far, node)
            while heap:
                d, u = heapq.heappop(heap)
                # Skip stale entries whose recorded distance is outdated.
                if d > dist[u]:
                    continue
                # Every pushed distance already respects the budget, so
                # reaching the target means a valid route exists.
                if u == n - 1:
                    return True
                for v, cost in graph[u]:
                    # Edges cheaper than the current guess are forbidden.
                    if cost < threshold:
                        continue
                    nd = d + cost
                    # Discard routes that already exceed the budget k.
                    if nd <= k and nd < dist[v]:
                        dist[v] = nd
                        heapq.heappush(heap, (nd, v))
            return dist[n - 1] <= k

        # Binary search the largest edge cost that still admits a valid route.
        ans = -1
        lo, hi = 0, len(costs) - 1
        while lo <= hi:
            mid = (lo + hi) // 2
            if feasible(costs[mid]):
                ans = costs[mid]  # record and try a larger guess
                lo = mid + 1
            else:
                hi = mid - 1  # too strict; try a smaller guess
        return ans
```

## Time and Space Complexity

**Time Complexity: `O((n + m) · log n · log m)`**

Let `n` be the number of nodes and `m` the number of edges.

1. **Filtering and sorting:** Building the adjacency list scans the edges once, `O(m)`. Sorting the distinct usable costs takes `O(m log m)`.

2. **Binary search:** The candidate answers are the distinct usable edge costs, at most `m` values, so the search performs `O(log m)` feasibility tests.

3. **Each feasibility test:** One Dijkstra run over the filtered graph. Every edge is relaxed at most once per run and each heap operation costs `O(log)` of the heap size, giving `O((n + m) log n)` per test.

Multiplying the per-test cost by the number of tests dominates the total: `O((n + m) · log n · log m)`. With `n <= 5 * 10⁴` and `m <= 10⁵`, this is roughly `10⁷` elementary operations, comfortable for the limits.

**Space Complexity: `O(n + m)`**

The adjacency list stores every usable edge once (`O(n + m)`), the distance array holds one 64-bit value per node (`O(n)`), the sorted cost array holds at most `m` values (`O(m)`), and the heap never exceeds `O(m)` entries.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### Pitfall 1: Maximizing the bottleneck first, then checking the budget

A tempting shortcut is to solve the classic maximum-bottleneck problem — find the route whose cheapest edge is largest, ignoring totals — and then verify that this one route fits the budget `k`. That answers a different question. The bottleneck-optimal route and the budget-respecting routes can be entirely different paths.

Example 1 shows the failure directly: `edges = [[0,1,5],[1,3,10],[0,2,3],[2,3,4]]`, `k = 10`. The bottleneck-optimal route is `0 → 1 → 3` with score `min(5, 10) = 5`, but its total is `15 > 10`. Checking only that route and returning `-1` (or worse, returning `5`) is wrong; the correct answer is `3` via `0 → 2 → 3`.

**Solution:** Fix the score threshold first, then minimize the total under that threshold. For each binary-search guess `x`, run a shortest-path computation over edges costing at least `x` and compare the cheapest total against `k`. The two objectives must be handled in this order because feasibility is monotone in `x`, while "best bottleneck" alone says nothing about totals.

---

### Pitfall 2: Treating the check as plain reachability, or overflowing 32-bit totals

Two closely related mistakes hide in the feasibility test. The first is forgetting the budget entirely: reaching node `n - 1` is not enough, the route must also cost at most `k`. With `edges = [[0,1,7]]` and `k = 6`, node `1` is reachable, but the only route costs `7 > 6`, so the answer is `-1`, not `7`. If your Dijkstra returns "yes" the moment it pops the target, make sure over-budget distances were never pushed in the first place:

```python
nd = d + cost
if nd <= k and nd < dist[v]:   # budget check happens here
    dist[v] = nd
    heapq.heappush(heap, (nd, v))
```

The second is arithmetic range. A route can traverse up to `10⁵` edges of cost up to `10⁹`, so totals reach `10¹⁴`, and `k` itself goes up to `5 * 10¹³`. Both exceed a 32-bit integer. In Java and C++, store distances in `long` / `long long`; note that the function signature already passes `k` as a 64-bit value.

---

### Pitfall 3: Routing through offline nodes

Running Dijkstra on the raw edge list, without removing offline nodes, silently accepts forbidden routes. With `edges = [[0,1,5],[1,2,5]]`, `online = [true, false, true]`, and `k = 10`, the unfiltered graph offers the route `0 → 1 → 2` with total `10 <= k` and score `5` — but node `1` is offline, so no valid route exists and the correct answer is `-1`.

**Solution:** Drop an edge unless **both** endpoints are online, before any traversal:

```python
for u, v, cost in edges:
    if online[u] and online[v]:
        graph[u].append((v, cost))
```

Because nodes `0` and `n - 1` are guaranteed online, this filter enforces exactly the "all intermediate nodes online" rule — no special-casing of the endpoints is needed during the search.

