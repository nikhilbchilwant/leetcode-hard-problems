# 3850. Count Sequences to K

> Source: https://algo.monster/liteproblems/3850

## Problem Statement

You are given an integer array `nums` and an integer `k`.

Start with an initial value `val = 1` and process the elements of `nums` from left to right. At each index `i`, you must choose **exactly one** of the following three actions:

- Multiply `val` by `nums[i]`.
- Divide `val` by `nums[i]`.
- Leave `val` unchanged.

The key detail here is that division is treated as **exact rational division**, not integer division. This means `val` is kept as a precise fraction at all times. For example, `2 / 4` equals `1 / 2`, not `0`.

After processing **all** elements of `nums`, the value `val` is considered **equal** to `k` only if its final rational value **exactly** equals `k`.

Your task is to return the count of **distinct** sequences of choices that result in `val == k`.

A sequence of choices is defined by the action taken at each index. Since there are three possible actions at every index, there are `3^n` total possible sequences (where `n` is the length of `nums`). You need to count how many of these sequences end with `val` exactly equal to `k`.

**Note:** Division is rational (exact), not integer division. For example, `2 / 4 = 1 / 2`.

## Intuition

The first thing to notice is that at each index we make an independent choice among three options, and the choices accumulate to form a final value. This naturally suggests exploring all possibilities through **recursion**, where we process one element at a time and branch into three directions.

A challenge is how to represent `val`. Since division is **exact rational division**, `val` can be a fraction like `1/2` rather than an integer. To track this precisely, we represent `val` as a pair `(p, q)`, meaning the fraction `p / q`. This avoids floating-point errors and lets us compare against `k` exactly.

Now we can define the recursive process. At index `i` with the current value `p / q`, we try each of the three actions:

- **Leave unchanged:** the value stays `p / q`.
- **Multiply by `nums[i]`:** the value becomes `(p * nums[i]) / q`.
- **Divide by `nums[i]`:** the value becomes `p / (q * nums[i])`.

When we reach the end (`i == n`), we check whether the fraction exactly equals `k`. Since `k` is an integer, this means `p == k` and `q == 1` (after the fraction is fully reduced). If so, this sequence of choices is valid and contributes `1` to the count.

There are two important refinements that make this work efficiently:

1. **Reducing the fraction.** If we keep multiplying numerators and denominators without simplifying, the numbers grow huge. After every multiply or divide, we compute the `gcd` of the numerator and denominator and divide both by it. This keeps `(p, q)` in lowest terms, which also gives a **canonical form** so that equal fractions always look identical.

2. **[Memoization](/problems/memoization_intro).** Different sequences of choices can lead to the same state `(i, p, q)`. Once the fraction is reduced to its canonical form, identical states recur often. By caching results based on `(i, p, q)`, we avoid recomputing the same subproblems, turning an exponential search into a much faster one.

Putting these ideas together, we get a memoized depth-first search that explores all valid choice sequences while keeping the rational values exact and the state space compact.

**Learn more about [Memoization](/problems/memoization_intro), [Math](/problems/math-basics) and [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

### Solution 1: [Memoization](/problems/memoization_intro) Search

We define a function `dfs(i, p, q)` that represents the number of different choice sequences when processing at index `i`, with the current rational value being `p / q`. The entry point is `dfs(0, 1, 1)`, which represents starting at index `0` with the initial value of `1` (i.e., the fraction `1 / 1`).

For each index `i`, we have three choices, and we add up the results of all three:

1. **Keep it unchanged**, i.e., `dfs(i + 1, p, q)`.
2. **Multiply by `nums[i]`**, i.e., the new value is `(p * nums[i]) / q`.
3. **Divide by `nums[i]`**, i.e., the new value is `p / (q * nums[i])`.

To avoid excessively large numbers, we simplify the numerator and denominator after each multiplication or division. Concretely, for the multiply branch, we let `x = nums[i]`, compute `g = gcd(p * x, q)`, and recurse with `dfs(i + 1, p * x // g, q // g)`. For the divide branch, we compute `g = gcd(p, q * x)` and recurse with `dfs(i + 1, p // g, q * x // g)`. Reducing the fraction to lowest terms after each step ensures the state `(p, q)` stays in a **canonical form**, so equal fractions are represented identically and the cache hits more often.

When `i` equals `n`, all elements have been processed. We check whether `p / q` exactly equals `k`: since `k` is an integer and the fraction is fully reduced, this holds only when `p == k` and `q == 1`. In that case we return `1`; otherwise we return `0`.

The base case and three recursive branches are combined as follows:

- If `i == n`: return `1` if `p == k and q == 1`, else `0`.
- Otherwise: `res = dfs(i + 1, p, q) + dfs(i + 1, p*x//g1, q//g1) + dfs(i + 1, p//g2, q*x//g2)`.

**Data structures and patterns used:**

- **Recursion (DFS):** We branch into three sub-states at every index, exploring all `3^n` possible sequences implicitly.
- **[Memoization](/problems/memoization_intro) (`@cache`):** Many distinct choice paths converge to the same state `(i, p, q)`. By caching the result of each state, repeated subproblems are computed only once, dramatically reducing the work. After computing the answer, we call `dfs.cache_clear()` to release the cached states.
- **GCD reduction:** Using `gcd` to normalize the fraction after each operation keeps numbers small and guarantees a unique representation for each rational value, which is essential for both correctness of the final comparison and effectiveness of the cache.

In summary, the algorithm performs a top-down memoized search over states `(i, p, q)`, where the fraction `p / q` is always kept in lowest terms, and accumulates the count of choice sequences that end exactly at the value `k`.

## Example Walkthrough

Let's trace through a small example to see how the memoized DFS explores choices and counts valid sequences.

**Input:** `nums = [2, 2]`, `k = 1`

We want to count how many of the `3^2 = 9` choice sequences leave `val` exactly equal to `1`. We start with `dfs(0, 1, 1)`, meaning index `0`, current value `1/1`.

**Step 1 — Index 0, state `(0, 1, 1)`, with `x = nums[0] = 2`:**

We branch into three actions:

- **Leave unchanged** → `dfs(1, 1, 1)` (value stays `1/1`).
- **Multiply by 2** → new value `2/1`. We reduce: `g = gcd(2, 1) = 1`, so `dfs(1, 2, 1)`.
- **Divide by 2** → new value `1/2`. We reduce: `g = gcd(1, 2) = 1`, so `dfs(1, 1, 2)`.

**Step 2 — Process each child at index 1, with `x = nums[1] = 2`:**

*Child A: `dfs(1, 1, 1)` (value `1/1`)*
- Leave → `dfs(2, 1, 1)` → at end, `p==1, q==1` ✅ returns **1**
- Multiply → `2/1` → `dfs(2, 2, 1)` → `p==2 ≠ 1` ❌ returns 0
- Divide → `1/2` → `dfs(2, 1, 2)` → `q==2 ≠ 1` ❌ returns 0
- Subtotal: **1**

*Child B: `dfs(1, 2, 1)` (value `2/1`)*
- Leave → `dfs(2, 2, 1)` → `p==2` ❌ returns 0
- Multiply → `4/1` → `dfs(2, 4, 1)` → ❌ returns 0
- Divide → `2/2` reduced to `1/1` → `dfs(2, 1, 1)` → ✅ returns **1**
- Subtotal: **1**

*Child C: `dfs(1, 1, 2)` (value `1/2`)*
- Leave → `dfs(2, 1, 2)` → `q==2` ❌ returns 0
- Multiply → `2/2` reduced to `1/1` → `dfs(2, 1, 1)` → ✅ returns **1**
- Divide → `1/4` → `dfs(2, 1, 4)` → ❌ returns 0
- Subtotal: **1**

**Step 3 — Combine results:**

```
dfs(0, 1, 1) = dfs(1,1,1) + dfs(1,2,1) + dfs(1,1,2)
             =     1      +     1      +     1
             = 3
```

**Answer:** `3`

**Verification of the 3 valid sequences** (action at index 0, action at index 1):

| Seq | Index 0 | Index 1 | Computation | Final |
|-----|---------|---------|-------------|-------|
| 1 | leave | leave | `1` | `1` ✅ |
| 2 | × 2 | ÷ 2 | `1 × 2 ÷ 2` | `1` ✅ |
| 3 | ÷ 2 | × 2 | `1 ÷ 2 × 2` | `1` ✅ |

**Key observations from the trace:**

- **GCD reduction in action:** In Child B, dividing `2/1` by `2` produced `2/2`, which we reduced to the canonical `1/1`. Likewise in Child C, multiplying `1/2` by `2` gave `2/2 → 1/1`. Without reduction these would look different from `1/1` and the final check `p==k and q==1` would fail.
- **Memoization payoff:** Notice that the state `dfs(2, 1, 1)` is reached from three different paths (Child A's leave branch, Child B's divide branch, and Child C's multiply branch). With `@cache`, it is computed once and reused, illustrating how distinct choice paths converge to identical states `(i, p, q)`.

## Python Solution

```python
class Solution:
    def countSequences(self, nums: List[int], k: int) -> int:
        n = len(nums)

        @cache
        def dfs(index: int, numerator: int, denominator: int) -> int:
            # Base case: all numbers have been processed.
            # A valid sequence requires the resulting fraction to equal k / 1.
            if index == n:
                return 1 if numerator == k and denominator == 1 else 0

            current = nums[index]

            # Choice 1: skip the current number entirely.
            count = dfs(index + 1, numerator, denominator)

            # Choice 2: multiply the current number into the numerator,
            # then reduce the fraction to its lowest terms.
            divisor = gcd(numerator * current, denominator)
            count += dfs(
                index + 1,
                numerator * current // divisor,
                denominator // divisor,
            )

            # Choice 3: multiply the current number into the denominator,
            # then reduce the fraction to its lowest terms.
            divisor = gcd(numerator, denominator * current)
            count += dfs(
                index + 1,
                numerator // divisor,
                denominator * current // divisor,
            )

            return count

        answer = dfs(0, 1, 1)
        dfs.cache_clear()  # Free the memoization cache to avoid cross-test contamination.
        return answer
```

## Time and Space Complexity

- **Time complexity**: `O(n^4 + log k)`, where `n` is the length of the array `nums`.

  The core of the algorithm is the recursive function `dfs(i, p, q)`, which is memoized via `@cache`. The number of distinct states determines the overall complexity:

  - The first parameter `i` ranges over `O(n)` values.
  - The parameters `p` and `q` represent the numerator and denominator of a fraction in lowest terms, formed by multiplying or dividing by elements of `nums`. The number of distinct values each of `p` and `q` can take is bounded by `O(n^2)`, because each is built from products of subsets of the `n` numbers (the distinct attainable values for the reduced numerator/denominator scale on the order of `n^2`).

  Combining these, the total number of states is `O(n) * O(n^2) * O(n^2)`, but since `p` and `q` are linked (forming a single reduced fraction), the effective number of distinct `(p, q)` pairs at each level is `O(n^2)`, giving `O(n) * O(n^2) = O(n^3)` reachable states. Each state performs constant transitions, but each transition involves a `gcd` computation costing `O(log k)`. Accounting for the full enumeration of fraction states and the comparison against `k`, the total work resolves to `O(n^4 + log k)`, where the additional `log k` term comes from the gcd operations dependent on the magnitude of `k`.

- **Space complexity**: `O(n^4)`, where `n` is the length of the array `nums`.

  The space is dominated by the memoization cache, which stores results for all distinct `(i, p, q)` states reached during the recursion. The number of such states is `O(n^4)`, matching the dominant term of the time complexity. The recursion depth contributes only `O(n)`, which is negligible compared to the cache size.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### Pitfall 1: Comparing fractions without reducing to lowest terms

The most subtle and dangerous mistake is checking whether `val == k` by comparing the **raw** numerator and denominator, or by storing the fraction in a non-canonical form. If you skip the GCD reduction after each multiply/divide step, two states that represent the *same* rational value can be stored as **different** `(numerator, denominator)` pairs — for example `2/4` and `1/2`.

This breaks the code in two ways:

1. **Incorrect final comparison.** The base case relies on the fact that a fully reduced fraction equals an integer `k` *only* when `numerator == k and denominator == 1`. If the fraction is not reduced, a value like `4/2` (which equals `2`) would fail the check `numerator == k and denominator == 1`, producing a wrong count.
2. **Cache ineffectiveness.** Memoization only collapses repeated work when equal values map to identical keys. Without reduction, `2/4` and `1/2` become distinct cache keys, so the cache rarely hits and the search degenerates toward `3^n` behavior.

**Solution:** Always normalize after every operation using `gcd`, and only ever compare in canonical form.

```python
# Multiply branch — reduce immediately
g = gcd(numerator * current, denominator)
nxt = dfs(index + 1, numerator * current // g, denominator // g)

# Divide branch — reduce immediately
g = gcd(numerator, denominator * current)
nxt = dfs(index + 1, numerator // g, denominator * current // g)

# Final check is then safe and correct
if index == n:
    return 1 if numerator == k and denominator == 1 else 0
```

A quick way to verify you reduced correctly: the denominator should always be a positive integer, and any state whose value is integral must have `denominator == 1`.

---

### Pitfall 2: Using integer (floor) division instead of rational division

It is tempting to track `val` as a single integer and write `val // nums[i]` for the divide branch. This is **wrong** for this problem, because division is defined as *exact rational division*. With floor division, `2 // 4 == 0`, permanently destroying information — a later multiply can never recover the lost fraction. This silently undercounts valid sequences (and may even overcount if a value collapses to `0`).

**Solution:** Represent the value as a pair `(numerator, denominator)` (or use Python's `fractions.Fraction`) so that intermediate values like `1/2` are preserved exactly. Floor division must never appear except as the `// g` step that *divides out a known common factor* — which is exact by construction.

---

### Pitfall 3: Forgetting `nums[i]` may be `0` or negative

If the problem constraints allow `nums[i] == 0`, the divide branch `numerator / (denominator * 0)` is undefined, and the multiply branch forces the value to `0`. Blindly computing `gcd(..., denominator * 0)` will give `gcd(x, 0) == x`, leading to a division by zero or a corrupted state.

**Solution:** Confirm the constraints. If zeros are possible, special-case them:

```python
if current == 0:
    # Multiply makes val == 0 (never equals a nonzero k);
    # divide is undefined and must be skipped.
    count = dfs(index + 1, numerator, denominator)          # skip
    count += dfs(index + 1, 0, 1)                            # multiply -> 0
    return count
```

Similarly, if negatives are allowed, keep the sign **only in the numerator** so that equal values share one canonical key (e.g., store `-1/2`, never `1/-2`). Normalize the sign right after each reduction.

---

### Pitfall 4: Not clearing the memoization cache between test cases

Decorating `dfs` with `@cache` inside the method still binds the cache to the *function object*, which captures `nums` and `k` via closure. Across multiple `Solution` instances or repeated calls, a stale cache can return results computed for a **different** `nums`/`k`, causing intermittent wrong answers that are hard to reproduce.

**Solution:** Call `dfs.cache_clear()` after producing the answer (as the code does), or define the cached function fresh on each call so each invocation starts with an empty cache. Always release the cache to avoid both incorrect cross-test reuse and unbounded memory growth.

