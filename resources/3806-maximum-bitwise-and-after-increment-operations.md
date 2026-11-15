# 3806. Maximum Bitwise AND After Increment Operations

> Source: https://algo.monster/liteproblems/3806

## Problem Statement

You are given an integer array `nums` and two integers `k` and `m`.

You may perform **at most** `k` operations. In one operation, you may choose any index `i` and **increase** `nums[i]` by 1.

Your task is to return an integer denoting the **maximum** possible **bitwise AND** of any **subset** of size `m` after performing up to `k` operations optimally.

In other words, you want to select a subset of exactly `m` elements from `nums`, and you are allowed to spend up to `k` total increment operations distributed across the array's elements however you like. After applying these operations, you compute the bitwise AND of the chosen `m` elements. Among all possible choices of subsets and all possible ways of distributing the operations, you should find the largest bitwise AND value that can be achieved.

For example, the bitwise AND of a subset has a particular bit set to `1` only if **every** element in that subset has that bit set to `1`. By increasing certain elements, you may be able to "turn on" higher bits across enough elements so that the resulting bitwise AND becomes larger. The goal is to maximize this final AND value while staying within the budget of `k` operations.

## Intuition

The key observation is that the bitwise AND result is dominated by its **higher bits**. A single high bit contributes more to the final value than all lower bits combined. For example, having bit `3` set (value `8`) is worth more than having bits `0`, `1`, and `2` all set (value `7`). This naturally suggests a **[greedy](/problems/greedy_intro) strategy**: we should try to secure the highest bits first, and only then worry about lower bits.

So we build the answer **bit by bit, from the highest bit down to the lowest**. At each step, we maintain a current candidate answer `target` that represents the set of bits we have already committed to. We then ask: "Can we also turn on this next bit while keeping all the bits we already locked in?" To check this, we form a tentative value `target = ans | (1 << bit)` and test whether it is achievable within the budget `k`.

Why is the [greedy](/problems/greedy_intro) order safe? Because once we successfully lock in a high bit, no combination of lower bits can ever surpass it. So committing to a high bit whenever possible never hurts us — it always leads to the maximum possible result.

The next piece is figuring out the **cost** to make a single element's bitwise AND contribution match `target`. For the AND of a subset to have all the bits in `target` set, **every** chosen element must already have those bits set after the increments. For one element `x`, we need to increase it just enough so that it covers all the `1` bits of `target`.

Here comes a clever detail. We look for the **highest bit position** where `target` has a `1` but `x` has a `0`. Call this position `j - 1`. Above this position, `x` already matches `target` (or has the necessary bits), so we don't need to touch those. To satisfy the bit at position `j - 1`, we only need to raise the **lower `j` bits** of `x` up to the lower `j` bits of `target`. The cheapest way to do this is to simply set the lower portion of `x` equal to the lower portion of `target`, which costs `(target & mask) - (x & mask)` where `mask = (1 << j) - 1`. This works because incrementing carries propagate upward, but matching the lower `j` bits exactly is precisely enough to flip the deficient bit into place.

Once we know the cost for **every** element to reach `target`, we want to pick the `m` elements that are **cheapest** to upgrade — since the subset only needs `m` members. So we sort the costs and sum the smallest `m` of them. If that total is within `k`, the bit is affordable, and we permanently add it to `ans`. Otherwise, we drop this bit and move on to the next lower one.

By repeating this process across all bits, we greedily accumulate the maximum achievable bitwise AND, always favoring higher bits and always choosing the `m` elements that are cheapest to satisfy.

**Learn more about [Greedy](/problems/greedy_intro) and [Sorting](/problems/sorting_summary) patterns.**

## Solution Approach

We use **[Greedy](/problems/greedy_intro) Bit Construction** combined with **Bit Manipulation**. The implementation builds the answer from the highest bit down to the lowest, testing one bit at a time.

**Step 1: Determine the bit range.**

The largest value any element can possibly reach is `max(nums) + k`, since `k` is the total budget of increments. So the highest meaningful bit is bounded by:

```python
mx = (max(nums) + k).bit_length()
```

We will iterate `bit` from `mx - 1` down to `0`.

**Step 2: Maintain the running answer.**

We keep a variable `ans`, initialized to `0`, which holds all the bits we have already successfully committed to. We also pre-allocate a `cost` array of length `len(nums)` to reuse across iterations:

```python
ans = 0
cost = [0] * len(nums)
```

**Step 3: Try to add the current bit.**

For each bit from high to low, we form a tentative target by turning on this bit on top of `ans`:

```python
target = ans | (1 << bit)
```

This `target` is the set of bits we would like every chosen element to cover.

**Step 4: Compute the cost for each element.**

For each element `x`, we find the **highest bit position** where `target` has a `1` but `x` has a `0`. This is computed with:

```python
j = (target & ~x).bit_length()
```

Here `target & ~x` isolates the bits that `target` requires but `x` is missing. Its `bit_length()` gives the position `j` just above the highest deficient bit. We then only need to fix the lower `j` bits of `x`:

```python
mask = (1 << j) - 1
cost[i] = (target & mask) - (x & mask)
```

The mask extracts the lower `j` bits. Raising the lower part of `x` to match the lower part of `target` costs exactly `(target & mask) - (x & mask)`. If `x` already covers all required bits, then `target & ~x` is `0`, `j` is `0`, the mask is `0`, and the cost is `0` — meaning no operations are needed.

**Step 5: Select the cheapest `m` elements.**

Since the subset only needs `m` members, we want the `m` elements that are cheapest to upgrade. We sort the `cost` array and sum the smallest `m` values:

```python
cost.sort()
if sum(cost[:m]) <= k:
    ans = target
```

If this total cost stays within the budget `k`, the bit is affordable, so we permanently commit to it by updating `ans = target`. Otherwise, we leave `ans` unchanged and move on to the next lower bit.

**Step 6: Return the result.**

After processing all bits, `ans` holds the maximum achievable bitwise AND:

```python
return ans
```

**Complexity Analysis.**

- Let `n` be the length of `nums` and `B` be the number of bits (`mx`).
- For each of the `B` bits, we compute the cost for all `n` elements in `O(n)` time and sort them in `O(n log n)` time.
- The overall time complexity is `O(B × n log n)`.
- The space complexity is `O(n)` for the `cost` array.

The [greedy](/problems/greedy_intro) correctness holds because securing a higher bit always outweighs any gain from lower bits, so committing to each affordable high bit in order guarantees the maximum final value.

## Example Walkthrough

Let's trace through the solution with a small, concrete example.

**Input:** `nums = [2, 5, 6]`, `k = 3`, `m = 2`

We want to select **2 elements** and maximize their bitwise AND, using at most **3** increment operations.

---

**Step 1: Determine the bit range.**

```
max(nums) = 6, so max(nums) + k = 9
9 in binary = 1001 → bit_length() = 4
```

So `mx = 4`, and we iterate `bit` from `3` down to `0`. Initialize `ans = 0`.

Binary forms for reference:
```
nums[0] = 2 = 0010
nums[1] = 5 = 0101
nums[2] = 6 = 0110
```

---

**Step 2: Try bit = 3 (value 8).**

```
target = ans | (1 << 3) = 0 | 1000 = 1000  (= 8)
```

Compute cost for each element to cover bit 3:

| x | x bin | target & ~x | j = bit_length | mask = (1<<j)-1 | cost = (target&mask)-(x&mask) |
|---|-------|-------------|----------------|------------------|-------------------------------|
| 2 | 0010  | 1000        | 4              | 1111             | (1000 & 1111) - (0010 & 1111) = 8 - 2 = 6 |
| 5 | 0101  | 1000        | 4              | 1111             | 8 - 5 = 3 |
| 6 | 0110  | 1000        | 4              | 1111             | 8 - 6 = 2 |

```
cost = [6, 3, 2] → sorted = [2, 3, 6]
sum of cheapest m=2 = 2 + 3 = 5
```

Is `5 <= k (3)`? **No.** Drop bit 3. `ans` stays `0`.

---

**Step 3: Try bit = 2 (value 4).**

```
target = 0 | (1 << 2) = 0100  (= 4)
```

| x | x bin | target & ~x | j | mask | cost |
|---|-------|-------------|---|------|------|
| 2 | 0010  | 0100        | 3 | 0111 | (0100 & 0111) - (0010 & 0111) = 4 - 2 = 2 |
| 5 | 0101  | 0000        | 0 | 0    | 0 (already has bit 2) |
| 6 | 0110  | 0000        | 0 | 0    | 0 (already has bit 2) |

```
cost = [2, 0, 0] → sorted = [0, 0, 2]
sum of cheapest m=2 = 0 + 0 = 0
```

Is `0 <= k (3)`? **Yes.** Commit bit 2. `ans = 0100 (= 4)`.

(We pick elements 5 and 6, which already cover bit 2 — zero cost.)

---

**Step 4: Try bit = 1 (value 2).**

```
target = ans | (1 << 1) = 0100 | 0010 = 0110  (= 6)
```

Now every chosen element must cover bits 2 **and** 1.

| x | x bin | target & ~x | j | mask | cost |
|---|-------|-------------|---|------|------|
| 2 | 0010  | 0100        | 3 | 0111 | (0110 & 0111) - (0010 & 0111) = 6 - 2 = 4 |
| 5 | 0101  | 0010        | 2 | 0011 | (0110 & 0011) - (0101 & 0011) = 2 - 1 = 1 |
| 6 | 0110  | 0000        | 0 | 0    | 0 |

```
cost = [4, 1, 0] → sorted = [0, 1, 4]
sum of cheapest m=2 = 0 + 1 = 1
```

Is `1 <= k (3)`? **Yes.** Commit bit 1. `ans = 0110 (= 6)`.

(We pick element 6 for free and upgrade element 5 from `5` to `6` at cost 1.)

---

**Step 5: Try bit = 0 (value 1).**

```
target = ans | (1 << 0) = 0110 | 0001 = 0111  (= 7)
```

Every chosen element must cover bits 2, 1, and 0.

| x | x bin | target & ~x | j | mask | cost |
|---|-------|-------------|---|------|------|
| 2 | 0010  | 0101        | 3 | 0111 | (0111 & 0111) - (0010 & 0111) = 7 - 2 = 5 |
| 5 | 0101  | 0010        | 2 | 0011 | (0111 & 0011) - (0101 & 0011) = 3 - 1 = 2 |
| 6 | 0110  | 0001        | 1 | 0001 | (0111 & 0001) - (0110 & 0001) = 1 - 0 = 1 |

```
cost = [5, 2, 1] → sorted = [1, 2, 5]
sum of cheapest m=2 = 1 + 2 = 3
```

Is `3 <= k (3)`? **Yes.** Commit bit 0. `ans = 0111 (= 7)`.

(Upgrade element 6 → 7 at cost 1, and element 5 → 7 at cost 2, total 3 operations — exactly the budget.)

---

**Step 6: Return the result.**

All bits processed. `ans = 7`.

**Verification:** Pick elements `5` and `6`. Spend 2 ops on `5 → 7` and 1 op on `6 → 7` (total 3 ops, within budget). Then:
```
7 AND 7 = 7 ✓
```

The maximum achievable bitwise AND is **`7`**, which matches our greedy result. Notice how the algorithm wisely *skipped* the tempting high bit 3 (too expensive at cost 5) and instead accumulated bits 2, 1, and 0 to reach a value of 7 within budget.

## Python Solution

```python
from typing import List


class Solution:
    def maximumAND(self, nums: List[int], k: int, m: int) -> int:
        # Determine the highest bit we may need to consider.
        # The largest achievable value is at most max(nums) + k,
        # so its bit length tells us the topmost relevant bit.
        max_bits = (max(nums) + k).bit_length()

        # 'answer' is built greedily from the most significant bit downward.
        answer = 0

        # Reusable buffer holding the cost to raise each number so that
        # it matches all bits currently set in the candidate target.
        costs = [0] * len(nums)

        # Try to turn on each bit from high to low.
        for bit in range(max_bits - 1, -1, -1):
            # Candidate value: current answer with this extra bit set.
            target = answer | (1 << bit)

            for index, value in enumerate(nums):
                # Find the lowest bit position where 'target' requires a 1
                # but 'value' currently has a 0. Bits above this position
                # are already aligned, so we only need to "fill up" the
                # suffix below 'highest_missing_bit'.
                highest_missing_bit = (target & ~value).bit_length()

                # Mask covering all bit positions below 'highest_missing_bit'.
                suffix_mask = (1 << highest_missing_bit) - 1

                # Cost to increment 'value' so that its lower suffix equals
                # the target's lower suffix. Because increments can only add,
                # we compute the numeric difference on the suffix part.
                costs[index] = (target & suffix_mask) - (value & suffix_mask)

            # Choose the 'm' cheapest numbers to satisfy this candidate target.
            costs.sort()

            # If the total cost for the cheapest m numbers fits the budget,
            # commit to this bit being set.
            if sum(costs[:m]) <= k:
                answer = target

        return answer
```

## Time and Space Complexity

- **Time Complexity**: `O(n × log n × log M)`, where `n` is the length of the array `nums` and `M` is the maximum value in the array `nums`.

  - The outer loop iterates over each bit position from `mx - 1` down to `0`. Since `mx` is the bit length of `max(nums) + k`, this loop runs `O(log M)` times.
  - Inside the outer loop, the first inner loop computes the `cost` for each of the `n` elements, taking `O(n)` time.
  - After computing the costs, `cost.sort()` is performed, which takes `O(n × log n)` time and dominates the `O(n)` cost computation.
  - The `sum(cost[:m])` operation takes `O(m)` time, which is bounded by `O(n)`.
  - Therefore, each iteration of the outer loop costs `O(n × log n)`, and the total time complexity is `O(log M × n × log n) = O(n × log n × log M)`.

- **Space Complexity**: `O(n)`, where `n` is the length of the array `nums`.

  - The auxiliary array `cost` stores one value per element, requiring `O(n)` space.
  - The sorting operation may require `O(n)` additional space depending on the implementation.
  - Hence, the overall space complexity is `O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### Pitfall: Miscomputing the increment cost by ignoring the "carry" effect of lower bits

The most common and subtle mistake in this problem is assuming that turning on a specific bit `b` in a number `x` costs simply `(1 << b) - (x & (1 << b))` — i.e., treating each bit independently. This is **wrong** because incrementing only adds to a number, and adding to lower bits can cascade (carry) into higher bits in ways that interact with the target.

The **correct** insight, captured in the reference solution, is:

> To make `x` cover all bits required by `target`, you only need to raise the **suffix** of `x` below the highest missing bit so that it matches the suffix of `target`.

Consider why the naive per-bit cost fails. Suppose `target` requires bits at positions 3 and 1, and `x` is missing bit 3. A naive approach might compute the cost to set bit 3 alone. But to *guarantee* the AND keeps bit 3 set, the entire lower portion of `x` must be increased to at least `target`'s lower portion — otherwise the carry produced while raising bit 3 might disturb (zero out) a bit you previously committed to.

Here is a buggy implementation that falls into this trap:

```python
# WRONG: treats each required bit independently
def cost_buggy(value, target):
    total = 0
    for b in range(target.bit_length()):
        if (target >> b) & 1 and not ((value >> b) & 1):
            total += (1 << b)   # naive "cost" to set bit b
    return total
```

This overcounts (or undercounts) because increments do not let you set arbitrary bits in isolation — you cannot "turn on" bit 3 of `4` (`0b100`) without passing through `0b101`, `0b110`, `0b111`, etc.

### The Correct Cost Computation

The reference solution handles this elegantly:

```python
highest_missing_bit = (target & ~value).bit_length()
suffix_mask = (1 << highest_missing_bit) - 1
costs[index] = (target & suffix_mask) - (value & suffix_mask)
```

- `target & ~value` isolates the bits required but missing.
- Its `bit_length()` gives the position just above the **highest** missing bit. Everything above this is already satisfied and must remain untouched.
- We then only need the lower suffix of `value` to reach the lower suffix of `target`. The difference `(target & suffix_mask) - (value & suffix_mask)` is exactly how many increments achieve this **without** carrying into the higher already-satisfied bits.

Because the suffix difference is always non-negative (any borrow would have been resolved at a higher missing bit), this formula never produces a negative cost.

### A Second Pitfall: Forgetting that committing a high bit constrains lower bits

A related conceptual error is computing each bit's affordability **independently** rather than relative to the already-committed `answer`. The cost to add bit `b` depends on what bits are already locked in:

```python
target = answer | (1 << bit)   # cost is measured against ALL committed bits
```

If you only test `1 << bit` instead of `answer | (1 << bit)`, you may report a bit as affordable in isolation, yet the combination with previously secured bits exceeds the budget. Always measure cost against the cumulative `target`, and only commit when the **combined** requirement fits within `k`.

### Minor Pitfall: Edge cases with `m` and the bit range

- **`m` larger than available cheap elements:** `costs[:m]` will simply take all `m` smallest after sorting; ensure `m <= len(nums)` per the problem constraints, otherwise `sum(costs[:m])` silently sums fewer elements.
- **Underestimating `max_bits`:** Using `max(nums).bit_length()` instead of `(max(nums) + k).bit_length()` would miss the highest reachable bits, since spending `k` increments can push a value into a new bit range. Always include `+ k`.

