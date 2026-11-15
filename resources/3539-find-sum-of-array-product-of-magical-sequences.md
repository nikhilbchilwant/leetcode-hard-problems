# 3539. Find Sum of Array Product of Magical Sequences

> Source: https://algo.monster/liteproblems/3539

## Problem Statement

You are given two integers, `m` and `k`, along with an integer array `nums`.

A sequence of integers `seq` is called **magical** if it satisfies all of the following conditions:

- The sequence `seq` has a size of `m` (it contains exactly `m` elements).
- Every element is a valid index into `nums`, meaning `0 <= seq[i] < nums.length`.
- When you compute the value `2^seq[0] + 2^seq[1] + ... + 2^seq[m - 1]`, its **binary representation** must contain exactly `k` **set bits** (bits equal to `1`).

For any sequence, its **array product** is defined as the product of the `nums` values located at each index in the sequence:

`prod(seq) = nums[seq[0]] * nums[seq[1]] * ... * nums[seq[m - 1]]`

Your task is to find the **sum** of the **array products** over **all** valid **magical** sequences.

Because the result can be very large, return the answer **modulo** `10^9 + 7`.

For clarity, a **set bit** is a bit in the binary representation of a number whose value is `1`.

**Key things to notice:**

- The sequence has a fixed length `m`, and each position independently chooses an index from `0` to `nums.length - 1`. Different positions may choose the same index, so repeated indices are allowed.
- Since the elements of `seq` are treated as positions in the sequence, two sequences that contain the same multiset of indices but in different orders are considered **distinct** sequences. This is why combinatorial counting (choosing which positions take a given index value) is involved.
- The condition involving `k` is about the number of set bits after summing the powers of two `2^seq[i]`. Because choosing the same index multiple times adds multiple copies of the same power of two, **carries** can occur when these powers add up, which affects the final count of set bits.

## Intuition

The first observation is that a magical sequence is built by assigning each of the `m` positions an index from `nums`. Instead of thinking about the order of positions directly, we can think about **how many times each index is chosen**. Suppose index `i` is chosen `t` times. Then those `t` positions contribute `nums[i]^t` to the array product, and the number of ways to pick exactly which `t` positions (out of the remaining positions) get value `i` is the binomial coefficient `C(j, t)`, where `j` is the number of positions still unassigned. This naturally turns the problem into processing the indices `0, 1, ..., n-1` one at a time and deciding how many positions go to each index.

The trickier part is the condition on set bits. Each time we pick index `i`, we add `2^i` to a running total. If index `i` is picked `t` times, that contributes `t * 2^i`. The key insight is that we don't need to track the entire huge sum — we only need to know how many set bits its binary representation has. We can process bit positions from **low to high**, just like adding numbers by hand and handling **carries**.

Think of it this way: when we are at index `i` (which corresponds to bit position `i`), the total contribution to that bit is `t` (the number of times we chose index `i`) plus whatever **carry** came in from the lower bits. Call this combined amount `nt = t + st`, where `st` is the incoming carry. The lowest bit of `nt`, that is `nt & 1`, determines whether bit `i` is set in the final number. If it is set, we have used up one of our `k` required set bits. The rest of `nt`, namely `nt >> 1`, becomes the carry that flows into the next higher bit position.

This leads directly to a recursive design `dfs(i, j, k, st)`:

- `i` is the current index / bit position we are processing.
- `j` is how many positions are still left to assign.
- `k` is how many set bits we still need to produce.
- `st` is the carry coming into bit position `i` from the lower bits.

At each step, we try every possible count `t` from `0` to `j` for the current index. For each choice we multiply by `C(j, t)` (ways to place those positions) and by `nums[i]^t` (their contribution to the product), then recurse with the updated carry `(t + st) >> 1` and updated set-bit budget `k - ((t + st) & 1)`.

When we finish processing all indices (`i == n`), the leftover carry `st` may still contain set bits at higher positions. We count those remaining set bits and subtract them from `k`. A solution is valid only when all positions are used (`j == 0`) and the set-bit count lands exactly at `k == 0`.

Finally, since binomial coefficients are needed repeatedly, we precompute factorials `f` and their modular inverses `g` so that `C(m, n) = f[m] * g[n] * g[m-n] % mod` can be evaluated in constant time. Combining the carry-aware bit counting with combinatorial position assignment gives us an efficient memoized search.

**Learn more about [Math](/problems/math-basics), [Dynamic Programming](/problems/dynamic_programming_intro), [Bitmask](/problems/bitmask_intro) and [Combinatorics](/problems/backtracking) patterns.**

## Solution Approach

We use **[combinatorics](/problems/backtracking) combined with memoized search**.

We design a function `dfs(i, j, k, st)`, which represents the number of weighted ways (weighted by array product) when we are currently processing the `i`-th element of array `nums`, still need to select numbers for the remaining `j` positions to fill into the magical sequence, still need to satisfy having `k` set bits in binary form, and the current carry coming from the lower bits is `st`. Then the answer is `dfs(0, m, k, 0)`.

The execution process of function `dfs(i, j, k, st)` is as follows:

- If `k < 0`, or if `i == len(nums)` and `j > 0`, it means the current solution is not feasible (we ran out of set-bit budget, or we reached the end without placing all positions), so we return `0`.

- If `i == len(nums)`, it means we have finished processing array `nums`. We still need to account for any remaining set bits hidden in the carry `st`. We repeatedly examine the lowest bit of `st`, subtracting each set bit from `k`:
  ```python
  while st:
      k -= st & 1
      st >>= 1
  ```
  If `k == 0` after this, the solution is feasible and we return `1`, otherwise we return `0`.

- Otherwise, we enumerate the count `t` of positions assigned to index `i`, where `0 <= t <= j`:
  - The number of ways to choose which `t` of the `j` remaining positions take value `i` is the binomial coefficient `C(j, t)`.
  - The product contribution from these `t` positions is `nums[i]^t`, computed as `pow(nums[i], t, mod)`.
  - The combined amount at bit position `i` is `nt = t + st`. Its lowest bit `nt & 1` tells whether bit `i` is set, so the updated set-bit budget becomes `k - (nt & 1)`.
  - The remainder `nt >> 1` becomes the carry flowing into the next bit position.
  - We recurse into `dfs(i + 1, j - t, k - (nt & 1), nt >> 1)` and accumulate:
    ```python
    res += comb(j, t) * p * dfs(i + 1, j - t, nk, nt >> 1)
    res %= mod
    ```

To compute binomial coefficients efficiently, we **preprocess** the factorial array `f` and the inverse factorial array `g`, where `f[i] = i! mod (10^9 + 7)` and `g[i] = (i!)^{-1} mod (10^9 + 7)`. The modular inverse of each factorial is obtained via **Fermat's little theorem** using `pow(f[i], mod - 2, mod)`. Then each combination is evaluated in constant time:

`comb(m, n) = f[m] * g[n] * g[m - n] % mod`

The search is decorated with `@cache` so that repeated states `(i, j, k, st)` are computed only once. After obtaining the answer with `dfs(0, m, k, 0)`, we call `dfs.cache_clear()` to release the memoization table.

**Complexity analysis:** Let `n = len(nums)`. The state is determined by `i` (up to `n`), `j` (up to `m`), `k` (up to `k`), and the carry `st` (bounded by roughly `m` since at most `m` powers of two are summed). Each state iterates over up to `m + 1` choices of `t`. Combining these factors, the overall time complexity is approximately `O(n * m^2 * k * m)` in the worst case, and the space complexity is dominated by the number of distinct memoized states.

## Example Walkthrough

Let's use a small concrete example to trace through the solution approach.

**Input:** `m = 2`, `k = 1`, `nums = [2, 3]`

So `n = len(nums) = 2`. We want all sequences `seq` of length `2`, where each element is an index into `nums` (so each is `0` or `1`), such that `2^seq[0] + 2^seq[1]` has exactly **1 set bit**.

#### Step 1: Enumerate the magical sequences by hand (to verify)

All sequences of length 2 over indices `{0, 1}`:

| `seq`     | `2^seq[0] + 2^seq[1]` | binary | set bits | magical? | product `nums[seq[0]] * nums[seq[1]]` |
|-----------|------------------------|--------|----------|----------|---------------------------------------|
| `[0, 0]`  | `1 + 1 = 2`            | `10`   | 1        | ✅       | `2 * 2 = 4`                            |
| `[0, 1]`  | `1 + 2 = 3`            | `11`   | 2        | ❌       | —                                      |
| `[1, 0]`  | `2 + 1 = 3`            | `11`   | 2        | ❌       | —                                      |
| `[1, 1]`  | `2 + 2 = 4`            | `100`  | 1        | ✅       | `3 * 3 = 9`                            |

The magical sequences are `[0, 0]` and `[1, 1]`. Notice that choosing the same index twice causes a **carry**: `1 + 1 = 2 = 10b`, collapsing two powers into one higher set bit.

**Expected answer:** `4 + 9 = 13`.

#### Step 2: Trace `dfs(0, m, k, 0)` = `dfs(0, 2, 1, 0)`

We process index `0` (bit position 0). Enumerate `t` = number of positions assigned to index `0`, where `0 <= t <= j = 2`. Carry in `st = 0`.

**Branch `t = 0`** (index 0 chosen 0 times):
- `C(2, 0) = 1`, contribution `nums[0]^0 = 1`.
- `nt = t + st = 0`. Bit set? `nt & 1 = 0`. New `k = 1 - 0 = 1`. New carry `nt >> 1 = 0`.
- Recurse `dfs(1, 2, 1, 0)`.

**Branch `t = 1`** (index 0 chosen once):
- `C(2, 1) = 2`, contribution `nums[0]^1 = 2`.
- `nt = 1`. Bit set? `nt & 1 = 1`. New `k = 1 - 1 = 0`. New carry `0`.
- Recurse `dfs(1, 1, 0, 0)`, multiplied by `2 * 2 = 4`.

**Branch `t = 2`** (index 0 chosen twice):
- `C(2, 2) = 1`, contribution `nums[0]^2 = 4`.
- `nt = 2`. Bit set? `nt & 1 = 0`. New `k = 1 - 0 = 1`. New carry `nt >> 1 = 1` (the carry from `1+1`).
- Recurse `dfs(1, 0, 1, 1)`, multiplied by `1 * 4 = 4`.

#### Step 3: Resolve the sub-calls at index `1` (bit position 1)

**`dfs(1, 2, 1, 0)`** (from `t=0`): still need to place all 2 positions into index 1.
- `t = 2`: `C(2,2)=1`, `nums[1]^2 = 9`, `nt = 2 + 0 = 2`, bit `= 0`, `k = 1`, carry `= 1`. Recurse `dfs(2, 0, 1, 1)`.
  - At `i == n = 2`, `j = 0` ✅. Process carry `st = 1`: lowest bit is `1`, so `k = 1 - 1 = 0`. Now `k == 0` ✅ → returns `1`. Weighted: `9 * 1 = 9`.
- Other `t` values (0 or 1) leave positions unfilled or yield wrong bit counts and ultimately return 0.
- So `dfs(1, 2, 1, 0)` contributes `9`.

**`dfs(1, 1, 0, 0)`** (from `t=1`): one position left, budget `k = 0`.
- Any `t = 1` here sets bit 1, dropping `k` below 0 → infeasible. Only `t = 0` keeps `k = 0`, but then `j = 1 != 0` at the end → returns `0`.
- So this branch contributes `0`. (This correctly kills sequences like `[0, 1]` / `[1, 0]`.)

**`dfs(1, 0, 1, 1)`** (from `t=2`): no positions left (`j = 0`), carry `st = 1` coming in.
- Only `t = 0` is possible. `nt = 0 + 1 = 1`, bit `= 1`, `k = 1 - 1 = 0`, carry `= 0`. Recurse `dfs(2, 0, 0, 0)`.
  - At `i == n`, `j = 0` ✅, no carry, `k == 0` ✅ → returns `1`.
- So `dfs(1, 0, 1, 1)` returns `1`.

#### Step 4: Combine everything

```
dfs(0, 2, 1, 0)
  = (t=0)  1 * 1 * dfs(1, 2, 1, 0) = 1 * 1 * 9 = 9
  + (t=1)  2 * 2 * dfs(1, 1, 0, 0) = 4 * 0     = 0
  + (t=2)  1 * 4 * dfs(1, 0, 1, 1) = 4 * 1     = 4
  = 9 + 0 + 4
  = 13
```

The result is **13**, matching the hand-enumerated answer `4 + 9 = 13`. ✅

#### Key takeaways from the trace

- The `t = 2` branch at index 0 corresponds to sequence `[0, 0]`: two identical low powers carry up into bit 1, producing exactly one set bit, with product `nums[0]^2 = 4`.
- The `t = 2` branch at index 1 corresponds to sequence `[1, 1]`: product `nums[1]^2 = 9`.
- The binomial coefficient `C(j, t)` correctly accounts for **distinct orderings** — though here the magical solutions happen to use a single repeated index (`C(2,2)=1`), the `C(2,1)=2` factor in the dead branch shows how `[0,1]` and `[1,0]` would both be counted if they were valid.
- The **carry** (`st`) is what links bit positions together and is the mechanism that turns "two copies of `2^i`" into "one copy of `2^(i+1)`", which is exactly why repeated indices change the set-bit count.

## Python Solution

```python
mx = 30
MOD = 10**9 + 7

# Precompute factorials and their modular inverses
fact = [1] + [0] * mx          # fact[i] = i! % MOD
inv_fact = [1] + [0] * mx      # inv_fact[i] = (i!)^{-1} % MOD

for i in range(1, mx + 1):
    fact[i] = fact[i - 1] * i % MOD
    inv_fact[i] = pow(fact[i], MOD - 2, MOD)  # Fermat's little theorem


def comb(n: int, r: int) -> int:
    """Binomial coefficient C(n, r) modulo MOD."""
    return fact[n] * inv_fact[r] % MOD * inv_fact[n - r] % MOD


class Solution:
    def magicalSum(self, m: int, k: int, nums: List[int]) -> int:
        n = len(nums)

        @cache
        def dfs(idx: int, remaining: int, bits_left: int, carry: int) -> int:
            """
            Count valid distributions.

            idx        : current index into nums being processed
            remaining  : number of selections still to assign among nums[idx:]
            bits_left  : popcount budget still allowed in the final magical sum
            carry      : accumulated carry value propagated from lower bit positions
            """
            # Pruning: budget exhausted, or all elements used but selections remain
            if bits_left < 0 or (idx == n and remaining > 0):
                return 0

            # Base case: all elements processed, flush the remaining carry bits
            if idx == n:
                while carry:
                    bits_left -= carry & 1
                    carry >>= 1
                return int(bits_left == 0)

            total = 0
            # Choose to assign `cnt` selections to the current element nums[idx]
            for cnt in range(remaining + 1):
                combined = cnt + carry                  # add current count to carry
                power = pow(nums[idx], cnt, MOD)         # nums[idx]^cnt
                next_bits = bits_left - (combined & 1)   # consume one bit by parity

                # Multinomial factor C(remaining, cnt) times the contribution,
                # then recurse with the higher carry bits (combined >> 1)
                total += comb(remaining, cnt) * power % MOD * \
                    dfs(idx + 1, remaining - cnt, next_bits, combined >> 1)
                total %= MOD

            return total

        ans = dfs(0, m, k, 0)
        dfs.cache_clear()  # clear memoization cache between test cases
        return ans
```

## Time and Space Complexity

**Time Complexity:** `O(n × m³ × k)`

The core of the algorithm is the recursive function `dfs(i, j, k, st)`, whose complexity is determined by the number of distinct states multiplied by the work done per state.

- **State analysis:**
  - `i` ranges over the array indices, giving `O(n)` values, where `n` is the length of `nums`.
  - `j` represents the remaining count to distribute, ranging from `0` to `m`, giving `O(m)` values.
  - `k` represents the remaining required popcount, giving `O(k)` values.
  - `st` is the carry accumulated from lower bits. Since at each step at most `m` items contribute and carries are halved (`nt >> 1`), the value of `st` is bounded by `O(m)`, giving `O(m)` values.

  Thus the total number of states is `O(n × m × k × m) = O(n × m² × k)`.

- **Work per state:**
  - Inside each call, the loop `for t in range(j + 1)` iterates up to `m + 1` times, contributing a factor of `O(m)`.
  - The `pow(nums[i], t, mod)` and `comb(j, t)` are effectively `O(1)` (with precomputed factorials, and `t` bounded by a small constant `mx = 30`).

  So each state does `O(m)` work.

- **Total:** combining states and per-state work yields `O(n × m² × k) × O(m) = O(n × m³ × k)`.

**Space Complexity:** `O(n × m² × k)`

The space is dominated by the memoization cache (`@cache`), which stores one entry for each distinct state `(i, j, k, st)`. As analyzed above, the number of distinct states is `O(n × m² × k)`. The recursion depth is only `O(n)`, and the precomputed factorial arrays `f` and `g` use `O(mx)` space, both of which are dominated by the cache. Hence the overall space complexity is `O(n × m² × k)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### Pitfall 1: Forgetting to Flush the Final Carry's Set Bits

The most frequent and damaging mistake is treating the popcount condition as if it only concerns the bits produced *while* iterating over `nums[0..n-1]`. When you finish processing the last index (`idx == n`), there can still be a leftover `carry` representing bits at positions `>= n` that have not yet been counted toward `bits_left`.

**Buggy version:**

```python
if idx == n:
    # WRONG: ignores any remaining carry bits above position n
    return int(bits_left == 0)
```

Consider `m = 2`, `nums.length = 1`. Choosing index `0` twice gives `2^0 + 2^0 = 2 = (10)_2`, which has **one** set bit at position `1`. While processing `idx = 0`, the parity `combined & 1 = 0` consumes **no** bit, and the carry `1` flows out. If you stop at `idx == n` without flushing, that genuine set bit is never counted, and you wrongly reject (or accept) sequences.

**Fix:** Drain every remaining bit of `carry` before deciding feasibility.

```python
if idx == n:
    while carry:
        bits_left -= carry & 1
        carry >>= 1
    return int(bits_left == 0)
```

This is precisely why `bits_left < 0` must also be checked *after* the flush — the carry can push the budget negative.

---

### Pitfall 2: Confusing "Set Bits" with "Total Count of Selections"

A natural but incorrect intuition is to think the answer relates to selecting exactly `k` distinct indices, ignoring that **carries merge powers of two**. Because `2^i + 2^i = 2^{i+1}`, picking the same index multiple times does **not** add multiple set bits — it propagates a carry.

```python
# WRONG mental model: "k distinct indices chosen"
next_bits = bits_left - (1 if cnt > 0 else 0)
```

The correct rule consumes a bit based on the **parity of `cnt + carry`**, not on whether the index was used at all:

```python
combined = cnt + carry
next_bits = bits_left - (combined & 1)   # parity determines the resulting bit
carry_out = combined >> 1                # everything above parity carries forward
```

Missing this leads to silently wrong counts on any input where some index is reused (which is allowed and very common).

---

### Pitfall 3: Forgetting the Multinomial (Ordering) Factor

Since two sequences with the same multiset of indices in different orders are **distinct**, you must multiply by `C(remaining, cnt)` — the number of ways to choose which of the remaining positions take value `idx`.

```python
# WRONG: counts each multiset once, undercounting ordered sequences
total += power * dfs(idx + 1, remaining - cnt, next_bits, combined >> 1)

# CORRECT: account for which positions receive index idx
total += comb(remaining, cnt) * power % MOD * \
         dfs(idx + 1, remaining - cnt, next_bits, combined >> 1)
```

Telescoping `C(m, t0) * C(m - t0, t1) * ...` correctly reproduces the full multinomial coefficient `m! / (t0! t1! ... )`, which is the number of orderings.

---

### Pitfall 4: Carry Range and Cache Explosion

The carry can grow up to roughly `m` (at most `m` powers summed at one position before shifting), but a common slip is bounding it too tightly (e.g., assuming it never exceeds `1`) or, conversely, failing to clear the cache between test cases.

```python
ans = dfs(0, m, k, 0)
dfs.cache_clear()   # essential: stale states across calls corrupt later answers
return ans
```

When `@cache` decorates a closure that captures `nums`, leftover entries from a previous instance can leak if the function object is reused. Clearing the cache (or rebuilding `dfs` per call, as done here) prevents subtle cross-test contamination.

---

### Pitfall 5: Precomputed Table Too Small

The factorial tables are sized by `mx = 30`. Binomial coefficients `C(remaining, cnt)` require indices up to `m`. If `m` ever exceeds `mx`, `comb` reads beyond the initialized range (where `fact[i] == 0`), producing zeros and silently wrong answers.

**Fix:** Size the tables to at least `m + 1` (or `max(m, k) + 1`):

```python
mx = max(31, m + 1)   # ensure fact/inv_fact cover all needed indices
fact = [1] * mx
inv_fact = [1] * mx
for i in range(1, mx):
    fact[i] = fact[i - 1] * i % MOD
inv_fact[mx - 1] = pow(fact[mx - 1], MOD - 2, MOD)
for i in range(mx - 1, 0, -1):
    inv_fact[i - 1] = inv_fact[i] * i % MOD   # O(n) inverse, avoids per-element pow
```

The backward recurrence for inverse factorials also removes the per-element `pow` call, turning the precompute from `O(mx log MOD)` into `O(mx)`.

