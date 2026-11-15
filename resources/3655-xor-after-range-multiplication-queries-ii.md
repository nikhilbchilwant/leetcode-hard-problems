# 3655. XOR After Range Multiplication Queries II

> Source: https://algo.monster/liteproblems/3655

## Problem Statement

You are given an integer array `nums` of length `n` and a 2D integer array `queries` of size `q`, where each query is represented as `queries[i] = [l​i, r​i, k​i, v​i]`.

For **each query**, you must perform a sequence of update operations on the array. The procedure works as follows:

- Start by setting an index pointer `idx = l​i`.
- Then, repeatedly do the following while `idx <= r​i`:
  - Multiply the value at position `idx` by `v​i`, taking the result modulo `10^9 + 7`. In other words, update `nums[idx] = (nums[idx] * v​i) % (10^9 + 7)`.
  - Advance the pointer by `k​i`, that is `idx += k​i`.

In simpler terms, for a given query you walk through the array starting at index `l​i`, stepping forward `k​i` positions at a time, and you keep going as long as you don't pass index `r​i`. At every position you visit, you scale the stored value by `v​i` under modular arithmetic.

After all `q` queries have been processed one after another, return the **bitwise XOR** of every element in the final `nums` array.

## Intuition

The most direct way to solve this problem is to simulate every query exactly as described: for each query, walk from `l​i` to `r​i` in steps of `k​i` and multiply each visited element by `v​i`. The trouble is that when the step `k​i` is small (for example `k​i = 1`), a single query can touch almost the entire array, and with many such queries the total work can become as large as `O(q * n)`, which is too slow.

The key observation is that the cost of a query depends on its step size `k`. A query with a **large step** only touches a few positions (at most `n / k`), so simulating it directly is cheap. A query with a **small step** can touch many positions, so simulating it directly is expensive. This natural split suggests a **square root decomposition** on the step size `k`:

- If `k > sqrt(n)`, the query visits at most `sqrt(n)` positions, so we just apply it directly. The total cost of all such queries is bounded.
- If `k <= sqrt(n)`, we cannot afford to walk each query individually. Instead, we need a smarter way to batch these updates.

For the small-step case, the trick is to notice that all positions touched by a query with step `k` and start `l` share the same remainder `res = l % k`. In other words, the array splits into `k` independent **arithmetic progressions** (one per remainder), and a query only multiplies a contiguous range of terms inside one of these progressions.

Multiplying a contiguous range by a value `v` is exactly the kind of operation we can handle with a **difference-array style technique**, but for products instead of sums. To "open" the range we multiply by `v` at the start term, and to "close" it we multiply by the modular inverse `v^(-1)` just past the end term. By sweeping through the progression while keeping a running product `cur`, every term automatically gets multiplied by the product of all queries that currently "cover" it.

The reason division works here is that we are operating under a prime modulus `10^9 + 7`, so every non-zero `v` has a modular inverse `pow(v, MOD - 2, MOD)` (by Fermat's little theorem). This lets us undo a multiplication just like subtraction undoes addition in an ordinary difference array.

So the plan becomes:
- Group small-step queries by their `(k, remainder)` bucket, recording a multiply event at the start term and an inverse event one step past the end term.
- For each bucket, sort the events by their term index, sweep through the progression maintaining a running product, and apply that product to each element.
- Handle large-step queries by brute force.
- Finally, XOR all elements together.

This balances the two extremes: large steps are cheap to brute force, and small steps are handled efficiently by the product difference-array, giving an overall complexity around `O((n + q) * sqrt(n))`.

**Learn more about [Divide and Conquer](/problems/divide_and_conquer_intro) patterns.**

## Solution Approach

The implementation follows the square root decomposition idea. Let `n` be the length of `nums`, `MOD = 10^9 + 7`, and choose a block size `B = isqrt(n) + 1`. Queries with step `k > B` are handled directly, while queries with `k <= B` are batched.

**1. The events container**

We build a nested list `events`, where `events[k][res]` holds the multiply/divide events for the arithmetic progression with step `k` and remainder `res`:

```python
events = [[[] for _ in range(k)] for k in range(B + 1)]
```

For a fixed `k`, there are exactly `k` possible remainders, so `events[k]` has `k` slots. Each slot stores a list of `(t, v)` pairs, where `t` is the **term index** within the progression and `v` is the value to multiply by.

**2. Distributing the queries**

For each query `[l, r, k, v]`:

- **Large step (`k > B`)**: simply walk the indices and apply the multiplication directly. Because `k > sqrt(n)`, this loop runs at most `sqrt(n)` times.

```python
if k > B:
    for idx in range(l, r + 1, k):
        nums[idx] = nums[idx] * v % MOD
```

- **Small step (`k <= B`)**: convert the range update into two events on the progression. All visited indices share the remainder `res = l % k`. The term indices range from `t1 = (l - res) // k` to `t2 = (r - res) // k`. We record a multiply event at `t1`:

```python
res = l % k
t1 = (l - res) // k
t2 = (r - res) // k
events[k][res].append((t1, v))
```

To "close" the range, we multiply by the modular inverse `v^(-1)` one term past `t2`, but only if that term actually exists in the array (the guard `t2 + 1 <= (n - 1 - res) // k`):

```python
if t2 + 1 <= (n - 1 - res) // k:
    invv = pow(v, MOD - 2, MOD)
    events[k][res].append((t2 + 1, invv))
```

The inverse is computed with Fermat's little theorem via `pow(v, MOD - 2, MOD)`, valid because `MOD` is prime.

**3. Sweeping each progression**

After all queries are distributed, we process every bucket `events[k][res]`:

First, sort the events by term index, then **compress** events that fall on the same term by combining their values into a single product. This avoids reapplying at the same point multiple times:

```python
ev.sort()
comp = []
for t, val in ev:
    if comp and comp[-1][0] == t:
        comp[-1] = (t, comp[-1][1] * val % MOD)
    else:
        comp.append([t, val])
```

Then we walk through the progression term by term (`idx = res, res + k, res + 2k, ...`), keeping a running product `cur`. Whenever the current term `t` matches the next event's term, we fold that event into `cur`. Every element is then multiplied by `cur`, which represents the combined effect of all queries currently covering this term:

```python
cur = 1
ptr = 0
t = 0
idx = res
while idx < n:
    while ptr < len(comp) and comp[ptr][0] == t:
        cur = cur * comp[ptr][1] % MOD
        ptr += 1
    nums[idx] = nums[idx] * cur % MOD
    idx += k
    t += 1
```

This is the **product difference-array** in action: the multiply event at `t1` switches a factor "on", and the inverse event at `t2 + 1` switches it back "off", so only terms in `[t1, t2]` keep that factor.

**4. Final answer**

Once all buckets are processed and large-step queries applied, XOR every element together:

```python
xr = 0
for x in nums:
    xr ^= x
return xr
```

**Complexity**

- Each large-step query costs `O(sqrt(n))`, so all of them together cost `O(q * sqrt(n))`.
- Small-step queries add at most `2` events each, and sweeping all progressions for a given `k` visits every array index once, costing `O(n)` per value of `k` up to `B`. Combined with sorting the events, the total is roughly `O((n + q) * sqrt(n) + q * log q)`.
- Space is `O(n + q)` for the events and the array.

## Example Walkthrough

Let's trace through a small example to see how the **square root decomposition** and **product difference-array** work together.

**Setup**

```
nums    = [2, 3, 1, 4, 5, 6]   (n = 6)
queries = [[0, 4, 1, 2],        // l=0, r=4, k=1, v=2
           [1, 5, 4, 3]]        // l=1, r=5, k=4, v=3
MOD     = 10^9 + 7
B       = isqrt(6) + 1 = 3      // block size threshold
```

Because `B = 3`, any query with step `k > 3` is brute-forced, and any query with `k <= 3` is batched into events.

---

**Step 1 — Classify the queries**

- Query 1: `k = 1`, and `1 <= 3`, so it is a **small-step** query → batch via events.
- Query 2: `k = 4`, and `4 > 3`, so it is a **large-step** query → apply directly.

---

**Step 2 — Handle the large-step query directly (Query 2)**

We walk from `l = 1` to `r = 5` in steps of `k = 4`, multiplying by `v = 3`:

- `idx = 1`: `nums[1] = 3 * 3 = 9`
- `idx = 5`: `nums[5] = 6 * 3 = 18`
- `idx = 9` → exceeds `r`, stop.

This loop runs only `2` times (at most `n / k`), confirming large steps are cheap.

```
nums = [2, 9, 1, 4, 5, 18]
```

---

**Step 3 — Build events for the small-step query (Query 1)**

Query 1 has `l = 0, r = 4, k = 1, v = 2`. With step `1`, the remainder is:

```
res = l % k = 0 % 1 = 0
t1  = (l - res) / k = (0 - 0) / 1 = 0      // first term index
t2  = (r - res) / k = (4 - 0) / 1 = 4      // last term index
```

We record a **multiply event** at term `t1 = 0`:

```
events[1][0] = [(0, 2)]
```

To "close" the range, we add an **inverse event** one term past `t2`, but only if that term exists. The largest valid term is `(n - 1 - res) / k = 5`, and `t2 + 1 = 5 <= 5`, so it exists. The inverse of `2` is `pow(2, MOD-2, MOD) = invv`:

```
events[1][0] = [(0, 2), (5, invv)]
```

Intuitively: term `0` switches the factor `×2` **on**, and term `5` switches it back **off**, so only terms `[0, 4]` keep the factor — exactly the queried range.

---

**Step 4 — Sweep the progression `(k=1, res=0)`**

Sort and compress events (no duplicate term indices here):

```
comp = [[0, 2], [5, invv]]
```

Now walk through every term `idx = 0, 1, 2, 3, 4, 5` keeping a running product `cur`:

| term `t` | `idx` | event fires?         | `cur`  | `nums[idx]` before → after |
|----------|-------|----------------------|--------|-----------------------------|
| 0        | 0     | yes, `cur = 1*2 = 2` | 2      | 2 → `2*2 = 4`               |
| 1        | 1     | no                   | 2      | 9 → `9*2 = 18`              |
| 2        | 2     | no                   | 2      | 1 → `1*2 = 2`               |
| 3        | 3     | no                   | 2      | 4 → `4*2 = 8`               |
| 4        | 4     | no                   | 2      | 5 → `5*2 = 10`              |
| 5        | 5     | yes, `cur=2*invv=1`  | 1      | 18 → `18*1 = 18`            |

Notice how the inverse event at term `5` restores `cur` to `1`, leaving `nums[5]` untouched — precisely because index `5` was outside `[0, 4]`.

```
nums = [4, 18, 2, 8, 10, 18]
```

---

**Step 5 — Verify against brute force**

Directly simulating both queries on the original `[2, 3, 1, 4, 5, 6]`:

- Query 1 (`×2` on indices 0–4): `[4, 6, 2, 8, 10, 6]`
- Query 2 (`×3` on indices 1, 5): `[4, 18, 2, 8, 10, 18]` ✓

The results match.

---

**Step 6 — Final XOR**

```
4 ^ 18 ^ 2 ^ 8 ^ 10 ^ 18
= (4 ^ 18) = 22
22 ^ 2     = 20
20 ^ 8     = 28
28 ^ 10    = 22
22 ^ 18    = 4
```

**Answer: `4`**

This small trace shows both halves of the strategy: the large-step query was cheaply simulated, while the small-step query was converted into a pair of on/off events and resolved in a single sweep using the running product.

## Python Solution

```python
import math
from typing import List


class Solution:
    def xorAfterQueries(self, nums: List[int], queries: List[List[int]]) -> int:
        MOD = 1_000_000_007
        n = len(nums)

        # Block size threshold for the sqrt decomposition strategy.
        # Queries with step > block_size are handled directly (few iterations),
        # while queries with step <= block_size are batched via difference arrays.
        block_size = int(math.isqrt(n)) + 1

        # events[step][residue] holds a list of (time_index, value) pairs.
        # For each (step, residue), the indices l, l+step, l+2*step, ... form a
        # subsequence; "time_index" is the position within that subsequence.
        # We use a difference-array style technique on multiplication:
        # multiply at the start time, and divide (multiply by inverse) right
        # after the end time, so prefix-products give the effective multiplier.
        events = [[[] for _ in range(step)] for step in range(block_size + 1)]

        for left, right, step, value in queries:
            if step > block_size:
                # Large step: directly apply multiplication to each affected index.
                for idx in range(left, right + 1, step):
                    nums[idx] = nums[idx] * value % MOD
            else:
                # Small step: record the range as difference-array events.
                residue = left % step
                start_time = (left - residue) // step
                end_time = (right - residue) // step

                # Open the multiplication interval at start_time.
                events[step][residue].append((start_time, value))

                # Close the interval right after end_time by applying the inverse,
                # but only if there is still an index beyond the range to affect.
                last_time = (n - 1 - residue) // step
                if end_time + 1 <= last_time:
                    inverse_value = pow(value, MOD - 2, MOD)
                    events[step][residue].append((end_time + 1, inverse_value))

        # Process all batched (small-step) events.
        for step in range(1, block_size + 1):
            for residue in range(step):
                event_list = events[step][residue]
                if not event_list:
                    continue

                # Sort events by time so we can sweep them in order.
                event_list.sort()

                # Compress events that share the same time index by combining
                # their multipliers, producing a clean list of [time, product].
                compressed = []
                for time_index, val in event_list:
                    if compressed and compressed[-1][0] == time_index:
                        compressed[-1][1] = compressed[-1][1] * val % MOD
                    else:
                        compressed.append([time_index, val])

                # Sweep through the subsequence indices residue, residue+step, ...
                # maintaining a running product of all active multipliers.
                current_multiplier = 1
                ptr = 0
                time_index = 0
                idx = residue
                while idx < n:
                    # Apply all events scheduled at the current time index.
                    while ptr < len(compressed) and compressed[ptr][0] == time_index:
                        current_multiplier = current_multiplier * compressed[ptr][1] % MOD
                        ptr += 1
                    nums[idx] = nums[idx] * current_multiplier % MOD
                    idx += step
                    time_index += 1

        # Compute the final XOR of all transformed values.
        result = 0
        for value in nums:
            result ^= value
        return result
```

## Time and Space Complexity

### Time Complexity

Let `n = len(nums)`, `q = len(queries)`, and `B = ⌊√n⌋ + 1`.

**1. Processing queries:**

For each query `(l, r, k, v)`:

- **Heavy case (`k > B`):** We directly iterate over indices `l, l+k, l+2k, ...` up to `r`. Since `k > B ≈ √n`, the number of iterations is at most `r/k < n/√n = √n`. So each heavy query costs `O(√n)`, totaling `O(q·√n)` in the worst case.

- **Light case (`k ≤ B`):** We do `O(1)` work, appending at most two events (the modular inverse `pow(v, MOD-2, MOD)` is `O(log MOD)`). So light queries cost `O(q·log MOD)` total.

Combined: `O(q·√n + q·log MOD)`.

**2. Processing events (light queries):**

The events are bucketed in `events[k][res]`. Each light query contributes at most 2 events, so the **total number of events across all buckets is `O(q)`**.

For each bucket `(k, res)`:

- **Sorting:** Each bucket's events are sorted; summed over all buckets this is `O(q·log q)` in the worst case (all events could fall in one bucket).
- **Compression:** Linear in the number of events, `O(q)` total.
- **Sweeping the arithmetic sequence:** For a fixed `k`, iterating over all residues `res = 0..k-1` covers each array index exactly once. So the sweep over one full `k` value costs `O(n)`. Summing over all `k` from `1` to `B` gives `O(B·n) = O(n·√n) = O(n^{1.5})`.

So event processing is `O(n^{1.5} + q·log q)`.

**3. Final XOR:** `O(n)`.

**Overall time complexity:**

`O(n^{1.5} + q·√n + q·log q + q·log MOD)`

which simplifies to roughly **`O((n + q)·√n + q·log q)`**.

### Space Complexity

- The `events` structure has `B + 1` rows, where row `k` has `k` residue lists, giving `O(1 + 2 + ... + B) = O(B²) = O(n)` empty list overhead for the buckets.
- The total number of stored event tuples is `O(q)`.
- The compressed list `comp` and other auxiliary variables use `O(q)` in the worst case.

**Overall space complexity:** `O(n + q)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### Pitfall: Closing the difference-array interval at the wrong time index (or skipping the bounds guard)

The trickiest part of this solution is the **product difference-array** logic. A query `[l, r, k, v]` should multiply `v` into terms `t1 = (l - res) // k` through `t2 = (r - res) // k` of the progression (where `res = l % k`). The technique is:

1. Multiply by `v` at term `t1` (turn the factor **on**).
2. Multiply by `v⁻¹` at term `t2 + 1` (turn the factor **off**).

Two mistakes commonly arise here:

**Mistake A — Forgetting the existence guard on the "close" event.**

If `t2` is the **last** term of the progression that fits in the array, then `t2 + 1` corresponds to an index `res + (t2 + 1) * k` that is `>= n`. The sweep loop (`while idx < n`) will never reach that term, so the inverse event never gets applied. By itself that's harmless — the factor simply stays "on" for terms that don't exist. The danger is the **opposite**: if you blindly insert the inverse event without the guard, and then for *some other query* a real term happens to land exactly on `t2 + 1`, the spurious inverse gets folded into `current_multiplier` and silently corrupts the result. The guard

```python
last_time = (n - 1 - residue) // step
if end_time + 1 <= last_time:
    ...append the inverse...
```

ensures the close event is only recorded when that term **actually exists** in the array.

**Mistake B — Using modular subtraction/division naively instead of an inverse.**

A frequent instinct is to "undo" a multiplication with integer division (`nums[idx] // v`) or to track sums with a plain difference array and subtraction. Both are wrong under modular arithmetic:

- `(a * v) % MOD // v` is **not** `a`, because the modulo already collapsed the value.
- Multiplication does not distribute over a subtractive difference array the way addition does.

The correct "inverse" is the modular multiplicative inverse via Fermat's little theorem, `pow(v, MOD - 2, MOD)`, valid because `MOD = 10⁹ + 7` is prime. This works only when `v` is not a multiple of `MOD` (always true for the given constraints, since `v < MOD`).

### Solution

Keep both safeguards explicit and verify them together. Here is a small self-check you can run to confirm the difference-array closing logic matches a brute-force simulation:

```python
def brute(nums, queries, MOD=1_000_000_007):
    nums = nums[:]
    for l, r, k, v in queries:
        idx = l
        while idx <= r:
            nums[idx] = nums[idx] * v % MOD
            idx += k
    xr = 0
    for x in nums:
        xr ^= x
    return xr

# Example that specifically exercises the boundary:
# a query whose t2 is the LAST term, and another query landing on t2+1.
nums = [1, 1, 1, 1, 1, 1]
queries = [
    [0, 4, 2, 3],   # touches indices 0,2,4 ; t2 = 2 is the last term (idx 4)
    [0, 4, 2, 5],   # same progression, overlapping
]
assert Solution().xorAfterQueries(nums[:], [q[:] for q in queries]) == brute(nums, queries)
```

**Key takeaways for avoiding the pitfall:**

1. **Always gate the close event** with `end_time + 1 <= (n - 1 - residue) // step`. Omitting it can corrupt unrelated terms of the same progression; adding an *unconditional* close that lands out of range is wasted work at best and a bug at worst.
2. **Never use integer division or subtraction to reverse a modular multiply.** Precompute `pow(v, MOD - 2, MOD)` and multiply by it instead.
3. **Compress events on the same term before sweeping.** If two queries open at the same `t1`, their factors must be folded into a single combined multiplier; otherwise a later inverse that assumes a single fold could mismatch. The compression step (combining `(t, val)` pairs with equal `t`) keeps the on/off bookkeeping consistent.
4. **Validate against brute force** on small random inputs, paying special attention to cases where a range ends exactly at the final reachable term of its progression — that's precisely where the boundary guard matters.

