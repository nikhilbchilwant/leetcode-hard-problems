# 3534. Path Existence Queries in a Graph II

> Source: https://algo.monster/liteproblems/3534

## Problem Statement

You are given `n` nodes, labeled from `0` to `n - 1`, together with an integer array `nums` of length `n` and an integer `maxDiff`.

These inputs define an undirected, unweighted [graph](/problems/graph_intro) implicitly: nodes `i` and `j` are joined by an edge exactly when their values differ by at most `maxDiff`, that is, when `|nums[i] - nums[j]| <= maxDiff`.

You are also given a 2D integer array `queries`, where each query `queries[i] = [ui, vi]` names two nodes. For each query, determine the **minimum number of edges** on a path between `ui` and `vi`. If the two nodes cannot reach each other at all, the answer for that query is `-1`.

Return an integer array `answer` where `answer[i]` is the result of the i-th query.

**Example 1:**

Input: `n = 5`, `nums = [1, 8, 3, 4, 2]`, `maxDiff = 3`, `queries = [[0, 3], [2, 4]]`
Output: `[1, 1]`

Nodes `0` and `3` carry the values `1` and `4`. Their difference is exactly `3`, so a direct edge joins them and the distance is `1`. Nodes `2` and `4` carry the values `3` and `2`, which differ by `1`, so they are also adjacent and the distance is again `1`.

**Example 2:**

Input: `n = 5`, `nums = [5, 3, 1, 9, 10]`, `maxDiff = 2`, `queries = [[0, 1], [0, 2], [2, 3], [4, 3]]`
Output: `[1, 2, -1, 1]`

Nodes `0` and `1` (values `5` and `3`) differ by `2`, so one edge suffices. Nodes `0` and `2` (values `5` and `1`) differ by `4`, which is too far for a direct edge, but the path `0 → 1 → 2` works because each step changes the value by exactly `2`; the distance is `2`. Node `2` (value `1`) can never reach node `3` (value `9`): no chain of edges crosses the gap between the values `5` and `9`, so the answer is `-1`. Nodes `4` and `3` (values `10` and `9`) are adjacent, giving `1`.

**Example 3:**

Input: `n = 3`, `nums = [3, 6, 1]`, `maxDiff = 1`, `queries = [[0, 0], [0, 1], [1, 2]]`
Output: `[0, -1, -1]`

Every pair of distinct values differs by more than `1`, so the graph contains no edges at all. A node reaches itself with `0` edges, and both remaining queries return `-1`.

**Constraints:**

- `1 <= n == nums.length <= 10^5`
- `0 <= nums[i] <= 10^5`
- `0 <= maxDiff <= 10^5`
- `1 <= queries.length <= 10^5`
- `queries[i] == [ui, vi]`
- `0 <= ui, vi < n`

## Intuition

The direct approach — build the adjacency lists and run a [BFS](/problems/graph_intro) for every query — collapses under the constraints. If many values sit close together (for instance, all values equal), every pair of nodes is adjacent and the graph holds about `n²/2` edges. With `n` and the number of queries both up to `10^5`, we can neither store that graph nor afford a full traversal per query.

The structure worth noticing is that node labels are irrelevant; only values matter. Sort the values. In sorted order, the nodes a given node can reach in one hop form one **contiguous block**: from the node at sorted position `r`, every position whose value lies within `maxDiff` of `vals[r]` is a neighbor, and those positions sit consecutively around `r`. So the graph is really a line of sorted values where each position can jump anywhere inside its block.

Now consider a query. Map both endpoints to their sorted positions and call the left one `a` and the right one `b`. Walking toward smaller values never helps, because reach grows with value: if position `p` sits left of position `p'`, then the farthest position reachable from `p` is no farther right than the one reachable from `p'`. Stepping backward can only shrink (never extend) how far right the next hop lands, so some shortest path moves monotonically to the right.

Once the walk is one-directional, minimizing hops becomes the same problem as [Jump Game II](/liteproblems/45): from the current position, the best move is to jump to the **farthest position reachable in one hop**. An exchange argument shows why this greedy walk is optimal. Take any shortest path and compare it hop by hop with the greedy walk: after the first hop, the greedy walker stands at least as far right as the path does, and because farther-right positions reach at least as far, this lead is preserved after every subsequent hop. The greedy walk therefore never needs more hops than the optimal path — it *is* optimal.

One greedy walk can still take up to `n` hops, which is too slow for `10^5` queries. The fix is **binary lifting**: precompute `up[k][r]`, the position reached from `r` after `2^k` greedy hops, by doubling (`up[k][r] = up[k-1][up[k-1][r]]`). To answer a query, take the largest jumps that keep the walker strictly left of `b`, accumulate their hop counts, and finish with one final hop. If even that final hop cannot reach `b`'s position, the two nodes lie in different connected components and the answer is `-1`. Each query then costs `O(log n)`.

**Learn more about [Greedy](/problems/greedy_intro), [Binary Search](/problems/binary_search_intro) and [Graph](/problems/graph_intro) patterns.**

## Solution Approach

### Step 1: Sort nodes by value

Sort the node indices by their values. Keep three arrays: `order[r]` (which original node sits at sorted position `r`), `rank[i]` (the sorted position of original node `i`), and `vals[r] = nums[order[r]]` (the values in non-decreasing order). All later work happens on sorted positions; `rank` translates each query's node labels into that coordinate system.

### Step 2: Compute the farthest one-hop reach

For every sorted position `r`, find `far[r]`, the largest position `j` with `vals[j] - vals[r] <= maxDiff`. This is the farthest position reachable from `r` in a single hop when moving right. Because `vals` is sorted, the threshold `vals[r] + maxDiff` only grows as `r` grows, so a two-pointer sweep computes the whole array in `O(n)`: advance `j` while `vals[j + 1] - vals[r] <= maxDiff`, and never move it back. (A binary search per position works equally well at `O(n log n)`.) Note that `far[r] >= r` always holds, since a value is within `maxDiff` of itself.

### Step 3: Build the binary lifting table

Let `LOG` be the smallest number of doublings that covers `n` positions. Define `up[0] = far` and fill

```
up[k][r] = up[k - 1][up[k - 1][r]]
```

so that `up[k][r]` is the position reached from `r` after `2^k` greedy hops. The table has `O(n log n)` entries and each entry is filled in constant time. If a position cannot move (`far[r] == r`), every level maps it to itself, which is exactly the behavior the query phase relies on to detect dead ends.

### Step 4: Answer each query

For a query `(u, v)`:

1. If `u == v`, the distance is `0`.
2. Otherwise set `a = min(rank[u], rank[v])` and `b = max(rank[u], rank[v])`.
3. If `far[a] >= b`, one hop suffices; the answer is `1`.
4. Otherwise walk `cur` from `a` using the table, from the highest level down: whenever `up[k][cur] < b`, take that jump (`cur = up[k][cur]`) and add `2^k` to the hop count. This leaves `cur` at the farthest position reachable with some number of hops that is still strictly left of `b`.
5. Take one final hop: if `far[cur] >= b`, the answer is the accumulated count plus one. If not, the walk is stuck before `b` — the two nodes are in different components — and the answer is `-1`.

The descent works because after processing level `k`, the remaining distance to `b` needs fewer than `2^k` additional hops of that size; choosing greedily from the top level down reproduces exactly the hop count of the plain greedy walk, just in `O(log n)` table lookups instead of one lookup per hop.

**Complexity Analysis:**

- **Time complexity:** `O((n + q) log n)`, where `q` is the number of queries — sorting costs `O(n log n)`, the `far` array `O(n)`, the lifting table `O(n log n)`, and each query `O(log n)`.
- **Space complexity:** `O(n log n)` for the lifting table, plus `O(n)` for the sorted arrays.

## Example Walkthrough

Take Example 2: `n = 5`, `nums = [5, 3, 1, 9, 10]`, `maxDiff = 2`, `queries = [[0, 1], [0, 2], [2, 3], [4, 3]]`.

#### Setup: sort by value

Sorting the nodes by value gives:

| Sorted position `r` | 0 | 1 | 2 | 3 | 4 |
|---------------------|---|---|---|---|---|
| Original node       | 2 | 1 | 0 | 3 | 4 |
| `vals[r]`           | 1 | 3 | 5 | 9 | 10 |

So `rank = [2, 1, 0, 3, 4]` (node `0` sits at sorted position `2`, node `1` at position `1`, and so on).

#### Farthest one-hop reach

With `maxDiff = 2`, the two-pointer sweep produces:

| `r` | `vals[r]` | Largest value allowed (`vals[r] + 2`) | `far[r]` |
|-----|-----------|----------------------------------------|----------|
| 0   | 1         | 3                                      | 1        |
| 1   | 3         | 5                                      | 2        |
| 2   | 5         | 7                                      | 2        |
| 3   | 9         | 11                                     | 4        |
| 4   | 10        | 12                                     | 4        |

Position `2` (value `5`) cannot move at all — the next value, `9`, is out of range — so `far[2] = 2`. That frozen entry is how the algorithm will detect the broken component.

#### Binary lifting table

With `n = 5` we need `LOG = 3` levels:

- `up[0] = far = [1, 2, 2, 4, 4]`
- `up[1][r] = up[0][up[0][r]] = [2, 2, 2, 4, 4]`
- `up[2][r] = up[1][up[1][r]] = [2, 2, 2, 4, 4]`

#### Query `[0, 1]` — answer `1`

Ranks are `2` and `1`, so `a = 1`, `b = 2`. Since `far[1] = 2 >= 2`, one hop reaches the target. Answer: `1`.

#### Query `[0, 2]` — answer `2`

Ranks are `2` and `0`, so `a = 0`, `b = 2`. `far[0] = 1 < 2`, so we descend the table with `cur = 0`, `steps = 0`:

- `k = 2`: `up[2][0] = 2`, not strictly left of `b = 2` — skip.
- `k = 1`: `up[1][0] = 2` — skip.
- `k = 0`: `up[0][0] = 1 < 2` — jump. Now `cur = 1`, `steps = 1`.

Final hop: `far[1] = 2 >= 2`, so the answer is `steps + 1 = 2`. This matches the path `0 → 1 → 2` in original labels (values `5 → 3 → 1`).

#### Query `[2, 3]` — answer `-1`

Ranks are `0` and `3`, so `a = 0`, `b = 3`. `far[0] = 1 < 3`. During the descent the walker reaches position `2` (value `5`) and then stops moving: `up[k][2] = 2` at every level, because `far[2] = 2`. After the descent `cur = 2`, and the final check `far[2] = 2 >= 3` fails. The walk is stuck strictly left of `b`, so nodes `2` and `3` lie in different components. Answer: `-1`. (Whatever hop count accumulated during the descent is discarded on this branch.)

#### Query `[4, 3]` — answer `1`

Ranks are `4` and `3`, so `a = 3`, `b = 4`. `far[3] = 4 >= 4`, so the nodes are adjacent. Answer: `1`.

Final output: `[1, 2, -1, 1]`, matching the expected result.

## Python Solution

```python
from typing import List


class Solution:
    def pathExistenceQueries(
        self, n: int, nums: List[int], maxDiff: int, queries: List[List[int]]
    ) -> List[int]:
        # Sort node indices by value. rank[i] = sorted position of node i,
        # vals[r] = value at sorted position r.
        order = sorted(range(n), key=lambda i: nums[i])
        rank = [0] * n
        for r, i in enumerate(order):
            rank[i] = r
        vals = [nums[i] for i in order]

        # far[r] = farthest sorted position reachable from r in one hop.
        # Both pointers only move right, so this pass is O(n).
        far = [0] * n
        j = 0
        for r in range(n):
            if j < r:
                j = r
            while j + 1 < n and vals[j + 1] - vals[r] <= maxDiff:
                j += 1
            far[r] = j

        # Binary lifting table: up[k][r] = position after 2^k greedy hops.
        LOG = max(1, (n - 1).bit_length())
        up = [far]
        for k in range(1, LOG):
            prev = up[k - 1]
            up.append([prev[prev[r]] for r in range(n)])

        ans = []
        for u, v in queries:
            if u == v:
                # A node reaches itself with zero edges.
                ans.append(0)
                continue
            # Work on sorted positions; always walk left to right.
            a, b = rank[u], rank[v]
            if a > b:
                a, b = b, a
            if far[a] >= b:
                # The target block is reachable in a single hop.
                ans.append(1)
                continue
            # Take the largest jumps that still land strictly left of b.
            steps = 0
            cur = a
            for k in range(LOG - 1, -1, -1):
                nxt = up[k][cur]
                if nxt < b:
                    steps += 1 << k
                    cur = nxt
            # One final hop either reaches b or the walk is stuck.
            if far[cur] >= b:
                ans.append(steps + 1)
            else:
                ans.append(-1)
        return ans
```

## Time and Space Complexity

**Time Complexity: `O((n + q) log n)`**

Let `n` be the number of nodes and `q` the number of queries. The work splits into four phases:

1. **Sorting:** Ordering the node indices by value costs `O(n log n)`.

2. **Farthest reach:** The two-pointer sweep that fills `far` moves each pointer at most `n` times in total, so it costs `O(n)`.

3. **Lifting table:** The table has `LOG = O(log n)` levels of `n` entries each, and every entry is a single array lookup of the previous level, giving `O(n log n)`.

4. **Queries:** Each query performs at most one table lookup per level plus a constant amount of extra work, so it costs `O(log n)`, and all queries together cost `O(q log n)`.

Adding the phases gives `O(n log n + q log n) = O((n + q) log n)`.

**Space Complexity: `O(n log n)`**

The dominant structure is the binary lifting table with `O(log n)` rows of `n` integers each. The auxiliary arrays `order`, `rank`, `vals`, and `far` each hold `n` integers, adding `O(n)`. The output array is `O(q)` and is usually not counted as auxiliary space.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### Pitfall 1: Building the graph and running BFS per query

The edge rule looks like an invitation to materialize an adjacency list and answer each query with a breadth-first search. That approach fails on size alone. If all values are equal (say `nums = [7, 7, ..., 7]` with any `maxDiff >= 0`), every pair of nodes is adjacent and the graph holds `n(n-1)/2` edges — about `5 × 10^9` for `n = 10^5`, which does not fit in memory. Even with a sparser graph, one BFS costs `O(n + edges)` and `10^5` queries multiply that into a timeout.

**Solution:** Never build the edges. Sort the values and represent one hop as "move to any position whose value is within `maxDiff`," which reduces each query to jumping along a line.

---

### Pitfall 2: Skipping the `u == v` check or mixing up labels and ranks

Two bookkeeping mistakes produce wrong answers on valid inputs:

- **Same-node queries.** After mapping to sorted positions, the query logic sets `a = b` when `u == v`, and the one-hop test `far[a] >= b` is then always true, returning `1` instead of `0`. Example 3 contains exactly this trap: query `[0, 0]` must return `0`. Handle `u == v` before anything else.
- **Original labels vs. sorted positions.** The arrays `far` and `up` are indexed by sorted position, but queries arrive as original node labels. Using `u` and `v` directly (without `rank`) happens to work on inputs where `nums` is already sorted — which is how Path Existence Queries in a Graph **I** presents its data — and silently breaks on this problem's unsorted `nums`. Always translate through `rank` first.

```python
if u == v:
    ans.append(0)          # zero edges, not one
    continue
a, b = rank[u], rank[v]    # sorted positions, not labels
```

---

### Pitfall 3: Wrong boundary in the binary lifting descent

The descent must jump only while the walker stays **strictly left** of the target position `b`, and then add one final hop:

```python
if nxt < b:          # correct: stay strictly left of b
    steps += 1 << k
    cur = nxt
```

Using `nxt <= b` overcounts. Suppose the greedy walk needs exactly two hops and `up[1][a] == b`: the `<=` version jumps straight onto `b` (recording `2` hops), then the final-hop check `far[b] >= b` still succeeds and returns `3`.

A related mistake is replacing the descent with a plain loop such as `while cur < b: cur = far[cur]` without checking for progress. When the two nodes are disconnected, the walker reaches a position with `far[cur] == cur` and the loop never terminates — Example 2's query `[2, 3]` hangs exactly this way, since `far` freezes at the position holding value `5`. The table-descent structure avoids both problems: jumps that do not move the walker change nothing, and the final `far[cur] >= b` test cleanly separates "one more hop reaches the target" from "stuck, return -1."

