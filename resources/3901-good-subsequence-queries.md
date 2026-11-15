# 3901. Good Subsequence Queries

> Source: https://algo.monster/liteproblems/3901

## Problem Statement

You are given an integer array `nums` of length `n` and an integer `p`.

A **non-empty subsequence** of `nums` is called **good** if:

- Its length is **strictly less** than `n`.
- The **greatest common divisor (GCD)** of its elements is **exactly** `p`.

You are also given a 2D integer array `queries` of length `q`, where each `queries[i] = [ind_i, val_i]` indicates that you should update `nums[ind_i]` to `val_i`.

After each query, determine whether there exists **any good subsequence** in the current array.

Return the **number** of queries for which a **good subsequence** exists.

The term `gcd(a, b)` denotes the **greatest common divisor** of `a` and `b`.

## Intuition

The key observation is that we only care about numbers that are **multiples of** `p`. If a number is not divisible by `p`, including it in a subsequence would make the GCD a value that does not have `p` as a factor, so such a number can never belong to a subsequence whose GCD is exactly `p`. This lets us ignore all non-multiples of `p` and focus only on the relevant elements.

So a natural idea is to treat every position holding a value not divisible by `p` as `0`, and keep the actual value otherwise. Then the GCD of the whole array (treating these `0`s as "no contribution") is exactly the GCD of all current multiples of `p`. Call this combined value `g`.

From here, two things must be checked for a good subsequence to exist: the **GCD condition** and the **length condition**.

For the GCD condition, notice that the most generous choice we can make is to take **all** multiples of `p`. Their combined GCD is `g`. Adding more elements to a set can only make the GCD smaller or keep it the same, and removing elements can only make it larger or keep it the same. Since every multiple of `p` is a factor candidate, the smallest GCD we could possibly achieve from these elements is `g`. Therefore:

- If `g != p`, the GCD of every possible candidate selection cannot reach exactly `p`, so the answer is impossible.
- If `g == p`, then the set of all multiples of `p` already achieves GCD exactly `p`.

Now we add the **length condition**: the subsequence length must be strictly less than `n`. Let `cnt` be the number of multiples of `p`.

- If `cnt < n`, some element is not a multiple of `p`, meaning we are not forced to use all `n` positions. Taking all `cnt` multiples already gives a valid subsequence of length `cnt < n` with GCD `p`. Done.
- If `cnt == n`, every element is divisible by `p`, so using all of them gives length exactly `n`, which is not allowed. We must drop at least one element and check whether the remaining elements still have GCD `p`.

For the `cnt == n` case, there is a neat shortcut. When `n > 6` and the overall GCD is already `p`, we can always remove one element while keeping the GCD equal to `p`. The reasoning is that with many elements sharing the same overall GCD `p`, the property is "redundant" enough that no single element is solely responsible for it, so one can always be discarded safely. Hence brute force is only needed when `n <= 6`, where we try deleting each position one by one and merge the GCD of the left part and the right part using `gcd(left_g, right_g)`.

To support frequent updates and range GCD queries efficiently, a **[segment tree](/problems/segment_tree_intro)** is the perfect data structure. Each leaf stores either the value (if divisible by `p`) or `0`, internal nodes store the GCD of their children, and we use point updates for each query plus range queries to evaluate the small-`n` brute force. This makes every query fast while keeping the overall logic simple.

**Learn more about [Segment Tree](/problems/segment_tree_intro) and [Math](/problems/math-basics) patterns.**

## Solution Approach

We use a **[Segment Tree](/problems/segment_tree_intro)** that maintains the **GCD** over intervals, combined with the observations above.

#### Data Structure: [Segment Tree](/problems/segment_tree_intro) over GCD

Each node of the [segment tree](/problems/segment_tree_intro) stores three fields:

- `l`, `r`: the left and right boundaries of the interval the node covers.
- `g`: the GCD of all values in that interval.

The tree supports two operations:

- **Point update** `modify(u, x, v)`: set the leaf at position `x` to value `v`, then push the change upward by recomputing parents with `pushup`, where `tr[u].g = gcd(tr[u<<1].g, tr[u<<1|1].g)`.
- **Range query** `query(u, l, r)`: return the GCD over `[l, r]`. If the requested range is empty (`l > r`), it returns `0`, which is the identity element for GCD (since `gcd(0, x) = x`).

The whole tree is built once over `1..n` with `build`.

#### Step 1: Initialize the tree

We iterate over `nums` (using `1`-based indexing). For each value `x`:

- If `x % p == 0`, it is a multiple of `p`, so we store its actual value with `tree.modify(1, i, x)` and increase the counter `cnt`.
- Otherwise we leave it as `0` (the default), meaning it does not contribute to the GCD.

After this, `tree.tr[1].g` holds the GCD of all current multiples of `p`, and `cnt` is how many such multiples exist.

#### Step 2: Process each query

For each `[idx, val]` (with `idx` being `0`-based in the input, so we use `idx + 1` in the tree):

1. **Remove the old value's effect.** If `nums[idx] % p == 0`, the old value was a multiple of `p`. We set that position to `0` with `tree.modify(1, idx + 1, 0)` and do `cnt -= 1`.
2. **Add the new value's effect.** If `val % p == 0`, we store it with `tree.modify(1, idx + 1, val)` and do `cnt += 1`.
3. Update the array itself: `nums[idx] = val`.

#### Step 3: Decide whether a good subsequence exists

After updating, we apply the rules:

- **GCD check.** If `tree.tr[1].g != p`, the GCD of all multiples of `p` is not exactly `p`, so no selection can reach GCD `p`. We `continue` without counting this query.
- **Easy length check.** If `cnt < n` or `n > 6`, a good subsequence always exists:
  - `cnt < n` means not every element is a multiple of `p`, so taking all `cnt` multiples gives a valid subsequence of length less than `n` with GCD `p`.
  - `n > 6` (with all elements divisible by `p` and overall GCD `p`) guarantees we can always drop one element and keep GCD `p`.

  In both cases we do `ans += 1` and `continue`.
- **Brute force for small `n`.** If `cnt == n` and `n <= 6`, every element is a multiple of `p` and we must delete at least one. We try deleting each position `i` from `1` to `n`:
  - `left_g = tree.query(1, 1, i - 1)` is the GCD of everything to the left.
  - `right_g = tree.query(1, i + 1, n)` is the GCD of everything to the right.
  - If `gcd(left_g, right_g) == p`, then removing position `i` still yields GCD exactly `p`, so a good subsequence exists. We do `ans += 1` and break.

#### Step 4: Return the result

After processing all queries, `ans` holds the number of queries for which a good subsequence exists, and we return it.

#### Complexity

Let `n` be the array length and `q` the number of queries.

- **Time:** Building the tree is `O(n)`. Each query performs a constant number of point updates, each costing `O(log n * log V)` (where `V` is the maximum value, due to the cost of `gcd`). The brute-force branch runs only when `n <= 6`, so it is bounded by a constant number of range queries. Overall the per-query cost is `O(log n * log V)`, giving `O((n + q) * log n * log V)`.
- **Space:** `O(n)` for the [segment tree](/problems/segment_tree_intro).

## Example Walkthrough

Let's trace through a small example to see the solution approach in action.

**Setup:**
- `nums = [2, 4, 6]`, so `n = 3`
- `p = 2`
- `queries = [[2, 9], [1, 6], [0, 8]]`

**Step 1: Initialize the segment tree (1-based indexing)**

We check each element against `p = 2`:

| Position | Value | Divisible by 2? | Stored in tree | 
|----------|-------|-----------------|----------------|
| 1        | 2     | Yes             | 2              |
| 2        | 4     | Yes             | 4              |
| 3        | 6     | Yes             | 6              |

All three are multiples of `2`, so `cnt = 3`.

The segment tree now holds GCDs over intervals. The root: `tr[1].g = gcd(2, 4, 6) = 2`.

---

**Query 1: `[2, 9]` → update `nums[2]` to `9`** (tree position `3`)

1. **Remove old value:** `nums[2] = 6` is divisible by `2`. Set position `3` to `0`, `cnt = 2`.
2. **Add new value:** `9 % 2 != 0`, so we do nothing. `cnt` stays at `2`.
3. Update array: `nums = [2, 4, 9]`.

Now the tree leaves are `[2, 4, 0]`, and root `g = gcd(2, 4, 0) = 2`.

**Decision:**
- GCD check: `g = 2 == p`. ✅ Passes.
- Length check: `cnt = 2 < n = 3`. ✅ Not every element is a multiple of `p`.

Taking the multiples `{2, 4}` gives a subsequence of length `2 < 3` with GCD `2`. **Good subsequence exists!** → `ans = 1`

---

**Query 2: `[1, 6]` → update `nums[1]` to `6`** (tree position `2`)

1. **Remove old value:** `nums[1] = 4` is divisible by `2`. Set position `2` to `0`, `cnt = 1`.
2. **Add new value:** `6 % 2 == 0`. Set position `2` to `6`, `cnt = 2`.
3. Update array: `nums = [2, 6, 9]`.

Now tree leaves are `[2, 6, 0]`, root `g = gcd(2, 6, 0) = 2`.

**Decision:**
- GCD check: `g = 2 == p`. ✅ Passes.
- Length check: `cnt = 2 < n = 3`. ✅

Taking `{2, 6}` gives length `2 < 3` with GCD `2`. **Good subsequence exists!** → `ans = 2`

---

**Query 3: `[0, 8]` → update `nums[0]` to `8`** (tree position `1`)

1. **Remove old value:** `nums[0] = 2` is divisible by `2`. Set position `1` to `0`, `cnt = 1`.
2. **Add new value:** `8 % 2 == 0`. Set position `1` to `8`, `cnt = 2`.
3. Update array: `nums = [8, 6, 9]`.

Now tree leaves are `[8, 6, 0]`, root `g = gcd(8, 6, 0) = 2`.

**Decision:**
- GCD check: `g = 2 == p`. ✅ Passes.
- Length check: `cnt = 2 < n = 3`. ✅

Taking `{8, 6}` gives length `2 < 3` with GCD `2`. **Good subsequence exists!** → `ans = 3`

---

**Final Result:** `ans = 3`

---

**Bonus: Illustrating the `cnt == n` brute-force branch**

Suppose at some point `nums = [4, 6, 8]` with `p = 2`, `n = 3`. All elements are multiples of `2`, so `cnt = 3 == n`. Since `n = 3 <= 6`, we must brute-force by deleting one element:

| Delete position | Left GCD | Right GCD | `gcd(left, right)` | Equals `p = 2`? |
|-----------------|----------|-----------|--------------------|-----------------|
| 1 (remove 4)    | 0        | gcd(6,8)=2| 2                  | ✅ Yes          |

Removing position `1` leaves `{6, 8}` with GCD `2 == p` and length `2 < 3`. A good subsequence exists, so we count this query and break.

This shows how the empty-range query returning `0` (the GCD identity) lets `gcd(0, 2) = 2` work cleanly when an element is dropped from the edge.

## Python Solution

```python
from math import gcd
from typing import List, Optional


class Node:
    """A single node in the segment tree storing a range and its GCD value."""

    __slots__ = "left", "right", "g"

    def __init__(self, left: int, right: int):
        self.left = left  # left boundary of the segment
        self.right = right  # right boundary of the segment
        self.g = 0  # GCD of all values within [left, right]


class SegmentTree:
    """Segment tree supporting point updates and range GCD queries."""

    __slots__ = "tree"

    def __init__(self, n: int):
        # Allocate 4n nodes (a safe upper bound for a segment tree).
        self.tree: List[Optional[Node]] = [None] * (n << 2)
        self.build(1, 1, n)

    def build(self, u: int, left: int, right: int):
        """Recursively construct the tree structure for range [left, right]."""
        self.tree[u] = Node(left, right)
        if left == right:
            return
        mid = (left + right) >> 1
        self.build(u << 1, left, mid)  # build left child
        self.build(u << 1 | 1, mid + 1, right)  # build right child

    def pushup(self, u: int):
        """Update the current node's GCD from its two children."""
        self.tree[u].g = gcd(self.tree[u << 1].g, self.tree[u << 1 | 1].g)

    def modify(self, u: int, x: int, v: int):
        """Set the value at position x to v and propagate the change upward."""
        if self.tree[u].left == self.tree[u].right:
            self.tree[u].g = v
            return
        mid = (self.tree[u].left + self.tree[u].right) >> 1
        if x <= mid:
            self.modify(u << 1, x, v)  # target lies in the left subtree
        else:
            self.modify(u << 1 | 1, x, v)  # target lies in the right subtree
        self.pushup(u)

    def query(self, u: int, left: int, right: int) -> int:
        """Return the GCD of all values within range [left, right]."""
        if left > right:
            # Empty range; GCD identity is 0.
            return 0
        if self.tree[u].left >= left and self.tree[u].right <= right:
            # Current segment is fully covered by the query range.
            return self.tree[u].g
        mid = (self.tree[u].left + self.tree[u].right) >> 1
        if right <= mid:
            return self.query(u << 1, left, right)  # entirely in the left child
        if left > mid:
            return self.query(u << 1 | 1, left, right)  # entirely in the right child
        # The query range spans both children; combine their results.
        return gcd(
            self.query(u << 1, left, mid),
            self.query(u << 1 | 1, mid + 1, right),
        )


class Solution:
    def countGoodSubseq(
        self, nums: List[int], p: int, queries: List[List[int]]
    ) -> int:
        n = len(nums)
        tree = SegmentTree(n)
        cnt = 0  # number of elements currently divisible by p

        # Initialize the tree: only keep elements divisible by p.
        for i, x in enumerate(nums, 1):
            if x % p == 0:
                tree.modify(1, i, x)
                cnt += 1

        ans = 0
        for idx, val in queries:
            # Remove the old value at idx if it was divisible by p.
            if nums[idx] % p == 0:
                tree.modify(1, idx + 1, 0)
                cnt -= 1
            # Insert the new value if it is divisible by p.
            if val % p == 0:
                tree.modify(1, idx + 1, val)
                cnt += 1
            nums[idx] = val

            # The overall GCD must equal p for any valid configuration.
            if tree.tree[1].g != p:
                continue

            # Fast path: if not all elements contribute or the array is large,
            # the condition is guaranteed to hold.
            if cnt < n or n > 6:
                ans += 1
                continue

            # Otherwise, verify that removing one element still yields GCD == p.
            for i in range(1, n + 1):
                left_g = tree.query(1, 1, i - 1)  # GCD of the prefix
                right_g = tree.query(1, i + 1, n)  # GCD of the suffix
                if gcd(left_g, right_g) == p:
                    ans += 1
                    break

        return ans
```

## Time and Space Complexity

**Time Complexity: `O((n + q) × log n)`**

The analysis proceeds as follows:

- **Building the segment tree**: The `build` method recursively constructs `4n` nodes, taking `O(n)` time.

- **Initial setup**: Iterating over `nums` and calling `modify` for each element divisible by `p` costs `O(n × log n)`, since each `modify` operation traverses the tree height `O(log n)`.

- **Processing queries**: For each of the `q` queries:
  - The `modify` operations (removing the old value and inserting the new value) each cost `O(log n)`.
  - Checking `tree.tr[1].g != p` is `O(1)`.
  - When `cnt < n or n > 6`, the query is handled in `O(1)` additional time.
  - When `cnt == n and n <= 6`, the code enumerates each of the `n` positions, performing two `query` calls per position, each costing `O(log n)`. Since `n <= 6`, this is at most `6 × 2 × log n = O(log n)`, i.e., only a constant factor of work.

  Therefore, each query is processed in `O(log n)` time, yielding `O(q × log n)` total.

Combining the build, setup, and query phases gives a total time complexity of `O((n + q) × log n)`, where `n` is the length of `nums` and `q` is the number of queries.

**Space Complexity: `O(n)`**

The segment tree allocates an array of size `n << 2` (i.e., `4n`), and each entry stores a `Node` object using `__slots__` to limit memory. The recursion depth during `build`, `modify`, and `query` is `O(log n)`. Thus the dominant space usage is the segment tree itself, giving an overall space complexity of `O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### Pitfall 1: Mishandling the "all elements divisible by `p`" boundary case (`cnt == n`)

The most subtle bug lies in the logic that decides whether a good subsequence exists when **every** element is a multiple of `p`. Recall the length constraint: a good subsequence must be **strictly shorter** than `n`. When `cnt == n`, the GCD of the *entire* array might equal `p`, but you are **not allowed** to use the whole array — you must drop at least one element.

A common mistake is to write the check as:

```python
# WRONG: forgets that the full array (length n) is not allowed
if tree.tree[1].g == p:
    ans += 1
    continue
```

This counts a query as valid simply because the GCD of all `n` elements is `p`. But if removing *any* single element raises the GCD above `p`, then **no** subsequence of length `< n` actually reaches GCD `p`, and the query should **not** be counted.

**Example.** Let `p = 1` and `nums = [6, 10, 15]`.
- `gcd(6, 10, 15) = 1`, so the full-array GCD equals `p`.
- But every proper subsequence:
  - `gcd(6, 10) = 2`
  - `gcd(6, 15) = 3`
  - `gcd(10, 15) = 5`
  - singletons are `6, 10, 15`

None of these equal `1`. So **no good subsequence exists**, yet the naive check would wrongly count it.

**Solution.** When `cnt == n`, do not trust the full-array GCD alone. Verify that there is *at least one* element whose removal keeps the GCD exactly `p`, using prefix/suffix GCDs (exactly what the brute-force branch does):

```python
if cnt == n:
    found = False
    for i in range(1, n + 1):
        left_g = tree.query(1, 1, i - 1)
        right_g = tree.query(1, i + 1, n)
        if gcd(left_g, right_g) == p:
            found = True
            break
    if found:
        ans += 1
    continue
```

The reference code handles this correctly by combining the `cnt < n or n > 6` fast path with the explicit deletion loop for `cnt == n and n <= 6`.

---

### Pitfall 2: Trusting the `n > 6` shortcut without understanding *why* it works

The fast path counts the query whenever `n > 6` (given the overall GCD is `p` and all elements are multiples of `p`). It is tempting to use a different threshold (e.g., `n > 1`) or to drop this branch entirely and brute-force everything.

The reasoning behind `n > 6`: among `n` numbers all divisible by `p` whose overall GCD is exactly `p`, if removing *every* candidate element raised the GCD, each element would have to be the unique carrier of some prime factor that prevents the GCD from dropping. The number of distinct "blocking" prime powers is bounded, so once `n` is large enough, by a pigeonhole-style argument some element is always safe to delete. The constant `6` is a safe bound that guarantees a deletable element always exists.

**Pitfall consequence:** If you lower the threshold (say to `n > 2`) you may incorrectly count cases like the `[6, 10, 15]` example above (`n = 3`). If you remove the brute force and rely only on the GCD check, you also count invalid cases.

**Solution.** Keep the threshold conservative (`n <= 6` triggers brute force) and only use the `n > 6` shortcut after confirming `tree.tree[1].g == p` and `cnt == n`. The brute force is cheap because it only runs for tiny arrays.

---

### Pitfall 3: Using `0` incorrectly as the GCD identity in updates

The tree initializes leaves to `0` and uses `gcd(0, x) = x` so that non-multiples of `p` "disappear." A frequent bug is forgetting to **reset a leaf to `0`** when an old value (that *was* divisible by `p`) is overwritten by a new value that is **not** divisible by `p`.

```python
# WRONG: only updates when new value is divisible by p,
# leaving the stale old value in the tree
if val % p == 0:
    tree.modify(1, idx + 1, val)
```

If `nums[idx]` was a multiple of `p` but `val` is not, the old value lingers in the tree, corrupting both `tree.tree[1].g` and `cnt`.

**Solution.** Always perform the **remove-then-add** sequence independently:

```python
if nums[idx] % p == 0:          # remove old contribution
    tree.modify(1, idx + 1, 0)
    cnt -= 1
if val % p == 0:                # add new contribution
    tree.modify(1, idx + 1, val)
    cnt += 1
nums[idx] = val
```

Updating `nums[idx]` **after** both checks is essential — if you update `nums[idx]` first, the removal check reads the new value and skips the necessary cleanup.

---

### Pitfall 4: Off-by-one between 0-based queries and 1-based tree indexing

Queries use 0-based indices, while the segment tree is built over `1..n`. Forgetting the `+ 1` shift (or applying it inconsistently between `modify` and `query`) silently updates or reads the wrong position.

**Solution.** Standardize on one convention: store data 1-based in the tree, and convert exactly once at the boundary (`idx + 1`). The prefix query `query(1, 1, i - 1)` and suffix query `query(1, i + 1, n)` already correctly rely on the empty-range guard (`left > right` returns `0`) for the endpoints `i = 1` and `i = n`.

