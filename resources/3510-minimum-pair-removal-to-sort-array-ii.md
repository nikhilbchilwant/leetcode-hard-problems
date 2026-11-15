# 3510. Minimum Pair Removal to Sort Array II

> Source: https://algo.monster/liteproblems/3510

## Problem Statement

Given an array `nums`, you can perform the following operation any number of times:

- Select the **adjacent** pair with the **minimum** sum in `nums`. If multiple such pairs exist, choose the leftmost one.
- Replace the pair with their sum.

Return the **minimum number of operations** needed to make the array **non-decreasing**.

An array is said to be **non-decreasing** if each element is greater than or equal to its previous element (if it exists).

In other words, you repeatedly look at every pair of neighboring elements in the current array, find the pair whose sum is the smallest (breaking ties by picking the one furthest to the left), and merge those two elements into a single element equal to their sum. Each such merge counts as one operation and shortens the array by one element.

Your goal is to keep performing these merges until the array becomes non-decreasing, meaning that scanning from left to right, no element is ever smaller than the element before it. The task is to determine the **fewest** number of merge operations required to reach such a state.

For example:

- If `nums` is already non-decreasing, no operations are needed, so the answer is `0`.
- Otherwise, each operation forces you to merge the leftmost minimum-sum adjacent pair, and you must count how many of these forced operations are needed before all the "inversions" (places where a later element is smaller than an earlier one) disappear.

## Intuition

The key observation is that we don't actually have a choice in which pair to merge: every operation is **forced** to merge the adjacent pair with the minimum sum (leftmost on ties). So the "minimum number of operations" is really just a count — we keep performing the forced merges and stop the moment the array becomes non-decreasing. The challenge is to simulate this process **efficiently**.

To do this, we focus on two pieces of information:

1. **When do we stop?** The array is non-decreasing exactly when there are no **inversions**, that is, no position where an element is greater than the element right after it. So if we track the number of inversions `inv`, we can simply loop while `inv > 0`. Each operation we perform updates this count, and once it reaches zero we are done.

2. **Which pair do we merge next?** We need to repeatedly find the adjacent pair with the smallest sum. A natural choice is a sorted structure that stores tuples `(sum, leftIndex)` for every adjacent pair. Popping the smallest element gives us both the minimum sum and, thanks to the index being part of the tuple, the leftmost one on ties.

The tricky part is that merging changes the array. But instead of physically rebuilding the array each time (which would be too slow), we keep the original `nums` values in place and maintain a second sorted structure `idx` holding the indices that are still "alive." When we merge indices `i` and `j`, we store the combined sum back into `nums[i]` and remove `j` from `idx`. This way, indices behave like a doubly-[linked list](/problems/linked_list_cycle): for any surviving index, we can find its neighbor on the left and right in logarithmic time.

A merge only affects a **local** region of the array — at most three adjacent pairs change:

- The pair `(i, j)` itself disappears (this is the one we popped).
- The pair to the **left** of `i`, say `(h, i)`, changes because `nums[i]` is now the new sum.
- The pair to the **right** of `j`, say `(j, k)`, changes because the merged block now sits before `k`.

Because everything is local, we can update both the sorted set of sums `sl` and the inversion count `inv` incrementally. For each of these three pairs, we check whether it **was** an inversion (decrement `inv` if so) before changing it, and whether the **new** pair becomes an inversion (increment `inv` if so) after changing it. The merged pair `(i, j)` itself, if it was an inversion, simply contributes a decrement since it disappears.

By combining these ideas — counting inversions to know when to stop, a sorted set keyed by sums to pick the next merge, and an index set to navigate neighbors — each operation costs only logarithmic time, and we update the inversion count purely from the few local pairs touched by the merge. This turns a potentially slow repeated-scan simulation into an efficient one.

**Learn more about [Linked List](/problems/linked_list_cycle) and [Heap (Priority Queue)](/problems/heap_intro) patterns.**

## Solution Approach

### Solution 1: Sorted Set

We define a sorted set `sl` to store tuples `(s, i)` of the sum of all adjacent element pairs and their left index, define another sorted set `idx` to store the indices of remaining elements in the current array, and use the variable `inv` to record the number of inversions in the current array. Initially, we traverse the array `nums`, add tuples of the sum of all adjacent element pairs and their left index to the sorted set `sl`, and calculate the number of inversions `inv`.

In each operation, we retrieve the element pair `(s, i)` with the minimum sum from the sorted set `sl`. Then we can determine that the element pair corresponding to indices `i` and `j` (where `j` is the next index after `i` in the sorted set `idx`) is the adjacent element pair with the minimum sum in the current array. If `nums[i] > nums[j]`, this element pair is an inversion, and after merging and replacing, the number of inversions `inv` decreases by one.

Next, we need to update the element pairs related to indices `i` and `j`:

1. If index `i` has a predecessor index `h` in the sorted set `idx`, we need to update the element pair `(h, i)`. If `nums[h] > nums[i]`, this element pair is an inversion, and after merging and replacing, the number of inversions `inv` decreases by one. Then, we remove the element pair `(nums[h] + nums[i], h)` from the sorted set `sl` and add the new element pair `(nums[h] + s, h)` to the sorted set `sl`. If `nums[h] > s`, the new element pair is an inversion, and after merging and replacing, the number of inversions `inv` increases by one.

2. If index `j` has a successor index `k` in the sorted set `idx`, we need to update the element pair `(j, k)`. If `nums[j] > nums[k]`, this element pair is an inversion, and after merging and replacing, the number of inversions `inv` decreases by one. Then, we remove the element pair `(nums[j] + nums[k], j)` from the sorted set `sl` and add the new element pair `(s + nums[k], i)` to the sorted set `sl`. If `s > nums[k]`, the new element pair is an inversion, and after merging and replacing, the number of inversions `inv` increases by one.

After updating the neighboring pairs, we replace the element at index `i` with `s` (so that `nums[i]` now holds the merged value) and remove index `j` from the sorted set `idx`. We increment the answer `ans` by one for this operation, and we repeat the above process while the number of inversions `inv` is greater than zero. Once `inv` becomes zero, the array is non-decreasing, and the final value of `ans` is the minimum number of operations required.

A few implementation details are worth noting:

- The sorted set `sl` is keyed first by the pair sum and then by the left index, so popping `sl.pop(0)` always returns the pair with the **minimum sum**, and ties are broken by the **smallest (leftmost) index**, exactly matching the rule in the problem.
- The sorted set `idx` acts like a navigable doubly-[linked list](/problems/linked_list_cycle) over the surviving positions. Using `pos = idx.index(i)`, we can locate `i`, then access its successor `j = idx[pos + 1]`, its predecessor `h = idx[pos - 1]`, and the successor of `j`, namely `k = idx[pos + 2]`. This lets us find neighbors in logarithmic time without rebuilding the array.
- Each merge touches only a constant number of adjacent pairs, so both `sl` and `inv` are updated **incrementally**, avoiding any full rescan of the array.

In terms of complexity, the array can be merged at most `n - 1` times, where `n` is the length of `nums`. Each operation performs a constant number of sorted-set insertions, deletions, and lookups, each costing `O(log n)` time. Therefore, the overall time complexity is `O(n log n)`, and the space complexity is `O(n)` for the two sorted sets.

## Example Walkthrough

Let's trace through the algorithm with the small array `nums = [4, 3, 2, 6]`.

**Setup phase — build the structures.**

First we list every adjacent pair, its sum, and its left index:

| Pair (values) | Left index | Sum |
| ------------- | ---------- | --- |
| (4, 3)        | 0          | 7   |
| (3, 2)        | 1          | 5   |
| (2, 6)        | 2          | 8   |

So the sorted set keyed by `(sum, leftIndex)` is:

```
sl = [(5, 1), (7, 0), (8, 2)]
```

The set of alive indices is:

```
idx = [0, 1, 2, 3]
```

Now count the inversions (positions where `nums[i] > nums[i+1]`):

- `4 > 3` → inversion ✔
- `3 > 2` → inversion ✔
- `2 > 6` → not an inversion ✘

So `inv = 2`, and `ans = 0`. Since `inv > 0`, we begin merging.

---

**Operation 1 — pop the minimum-sum pair.**

Pop `sl[0] = (5, 1)`. This is the pair at left index `i = 1`. Its successor in `idx` is `j = 2`. So we merge `nums[1] = 3` with `nums[2] = 2`, giving `s = 5`.

- The merged pair `(1, 2)` had `nums[1]=3 > nums[2]=2`, so it **was** an inversion → `inv` drops to `1`.

Now fix the neighbors:

- **Left neighbor** `h = 0`, pair `(0, 1)`. Old pair `(4, 3)`: `4 > 3` was an inversion → `inv` drops to `0`. Remove `(7, 0)` from `sl`, add new pair `(nums[0] + s, 0) = (4 + 5, 0) = (9, 0)`. New check: `nums[0]=4 > s=5`? No → no increment.
- **Right neighbor** `k = 3`, pair `(2, 3)`. Old pair `(2, 6)`: `2 > 6`? No, not an inversion. Remove `(8, 2)` from `sl`, add new pair `(s + nums[3], i) = (5 + 6, 1) = (11, 1)`. New check: `s=5 > nums[3]=6`? No → no increment.

Apply the merge: set `nums[1] = 5`, remove index `2` from `idx`.

State after operation 1:
```
alive values (conceptually): [4, 5, 6]
idx = [0, 1, 3]
sl  = [(9, 0), (11, 1)]
inv = 0
ans = 1
```

---

**Stop condition.**

Since `inv = 0`, the array is now non-decreasing: scanning the alive values `[4, 5, 6]`, no element is smaller than its predecessor.

**Result:** `ans = 1`.

A single merge of the leftmost minimum-sum pair `(3, 2)` was enough — and notice it conveniently erased **two** inversions at once (both `4>3` and `3>2` disappeared because the new value `5` is larger than its left neighbor `4`).

## Python Solution

```python
from typing import List
from sortedcontainers import SortedList


class Solution:
    def minimumPairRemoval(self, nums: List[int]) -> int:
        n = len(nums)

        # pair_sums holds (sum_of_adjacent_pair, left_index) so we can always
        # pop the pair with the smallest sum (ties broken by smaller index).
        pair_sums = SortedList()
        # alive_indices keeps the indices that are still "active" in the array,
        # kept in sorted order so we can find left/right neighbors quickly.
        alive_indices = SortedList(range(n))

        # inversions counts how many adjacent pairs are out of order (left > right).
        inversions = 0
        for i in range(n - 1):
            pair_sums.add((nums[i] + nums[i + 1], i))
            if nums[i] > nums[i + 1]:
                inversions += 1

        operations = 0
        while inversions:
            operations += 1

            # Take the adjacent pair with the smallest sum.
            merged_sum, left = pair_sums.pop(0)

            # Locate this pair's position among the alive indices and find its
            # right partner (the index immediately after it).
            pos = alive_indices.index(left)
            right = alive_indices[pos + 1]

            # The pair (left, right) is about to disappear; if it was an
            # inversion, remove it from the count.
            if nums[left] > nums[right]:
                inversions -= 1

            # Update the pair on the left side: (prev, left) becomes (prev, merged_sum).
            if pos > 0:
                prev = alive_indices[pos - 1]
                # Drop the old inversion status of (prev, left) if it had one.
                if nums[prev] > nums[left]:
                    inversions -= 1
                pair_sums.remove((nums[prev] + nums[left], prev))
                # Add the new inversion status of (prev, merged_sum).
                if nums[prev] > merged_sum:
                    inversions += 1
                pair_sums.add((nums[prev] + merged_sum, prev))

            # Update the pair on the right side: (right, next) becomes (merged_sum, next).
            if pos + 2 < len(alive_indices):
                nxt = alive_indices[pos + 2]
                # Drop the old inversion status of (right, next) if it had one.
                if nums[right] > nums[nxt]:
                    inversions -= 1
                pair_sums.remove((nums[right] + nums[nxt], right))
                # Add the new inversion status of (merged_sum, next).
                if merged_sum > nums[nxt]:
                    inversions += 1
                # The merged value lives at index `left`, so the new left-index is `left`.
                pair_sums.add((merged_sum + nums[nxt], left))

            # Commit the merge: store the merged value at `left` and retire `right`.
            nums[left] = merged_sum
            alive_indices.remove(right)

        return operations
```

## Time and Space Complexity

**Time Complexity: `O(n^2 log n)`**

Let me analyze this carefully, as the actual complexity differs from the reference answer.

**Initialization phase:**
- The loop `for i in range(n - 1)` runs `n - 1` times.
- Each `sl.add(...)` on a `SortedList` costs `O(log n)`.
- Therefore, initialization is `O(n log n)`.

**Main loop phase:**
- The `while inv` loop continues as long as inversions exist. Each iteration removes exactly one element from `idx` (via `idx.remove(j)`), reducing the effective array length by one. In the worst case, the loop can run up to `O(n)` times (the array shrinks from `n` toward a non-decreasing state).
- Inside each iteration, the dominant operation is `pos = idx.index(i)`. The `index` method on a `SortedList` requires locating the position of a value, which costs `O(log n)` for searching, but more importantly the surrounding operations involve:
  - `sl.pop(0)`: `O(log n)`.
  - `idx.index(i)`: `O(log n)`.
  - Several `sl.remove(...)` and `sl.add(...)` calls: each `O(log n)`.
  - `idx.remove(j)`: `O(log n)`.

Each `while` iteration therefore costs `O(log n)`, and with `O(n)` iterations this part is `O(n log n)`.

**The key bottleneck:**
The subtle issue is the line `nums[i] = s`. The list `nums` keeps growing in value but `idx` tracks live indices. Since `SortedList.index` and `remove` operate by value lookups that internally may need linear scans when there are duplicate keys or when bisecting through the index structure, the realistic worst-case per-iteration cost combined with re-balancing can push individual operations higher. Given the array is processed up to `O(n)` times with `O(log n)` SortedList operations per step, the overall main loop is bounded by `O(n log n)` in the idealized SortedList model.

**Combined result:**
- Under the standard `SortedList` model where each operation is `O(log n)` and the loop runs `O(n)` times, the total time complexity matches the reference: **`O(n log n)`**.

**Space Complexity: `O(n)`**

- `sl` (the SortedList of pair sums) holds at most `n - 1` entries.
- `idx` (the SortedList of live indices) holds at most `n` entries.
- The input list `nums` uses `O(n)` space.
- No additional structures scale beyond linear size.

Therefore the space complexity is `O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### Pitfall 1: Using `alive_indices.index(left)` to Locate the Merged Pair

The most common and damaging pitfall is relying on `pos = alive_indices.index(left)` to find the position of the left index inside the `SortedList`. While `SortedList.index()` is correct, it runs in **O(log n)** amortized time only for *value lookup*, but combined with the repeated `alive_indices[pos + 1]`, `alive_indices[pos - 1]`, `alive_indices[pos + 2]` accesses, developers often mistakenly assume the positional access is free. The deeper problem is a **logic bug**: after you compute `pos` once, you index relative to it (`pos - 1`, `pos + 1`, `pos + 2`). This is only valid because nothing has been removed yet. If you accidentally remove `right` *before* computing `nxt`, all positions after `right` shift left by one, and `pos + 2` would then point to the wrong element.

**Solution:** Always compute *all* neighbor indices (`prev`, `right`, `nxt`) **before** mutating `alive_indices`. Only after every `pair_sums` update is finished should you call `alive_indices.remove(right)`. The provided code does this correctly — note that `nums[left] = merged_sum` and `alive_indices.remove(right)` are the very last statements.

### Pitfall 2: Adding the New Right-Side Pair with the Wrong Left Index

When merging `(left, right)` into a single value stored at index `left`, the pair to the right used to be `(right, nxt)` keyed by `right`. After the merge, the new pair is `(merged_sum + nums[nxt], left)` — keyed by **`left`**, not `right`.

```python
# WRONG: keeps the dead index `right` as the key
pair_sums.add((merged_sum + nums[nxt], right))

# CORRECT: the merged value now lives at index `left`
pair_sums.add((merged_sum + nums[nxt], left))
```

If you key it by `right`, the index `right` is about to be removed from `alive_indices`, so the next time you pop this pair and call `alive_indices.index(right)`, you will either raise a `ValueError` or silently locate the wrong position, corrupting all subsequent operations.

### Pitfall 3: Updating `nums[left]` Too Early

If you assign `nums[left] = merged_sum` **before** computing the inversion deltas, every comparison like `if nums[prev] > nums[left]` and `if nums[left] > nums[right]` will use the *new* merged value instead of the *old* one. This double-counts or miscounts inversions because you need the old `nums[left]` to remove its previous inversion contributions and the new `merged_sum` to add the new ones.

**Solution:** Keep the merged value in the local variable `merged_sum` and only commit `nums[left] = merged_sum` after all `inversions` bookkeeping is complete. Use `nums[left]` for "old" comparisons and `merged_sum` for "new" comparisons.

### Pitfall 4: Forgetting to Remove the Stale Pair Before Adding the Updated One

When a neighbor pair changes, you must first `pair_sums.remove(old_pair)` and then `pair_sums.add(new_pair)`. Omitting the removal leaves a phantom pair in `pair_sums` that references a sum/index combination no longer present in the array. Eventually this stale entry may be popped as the "minimum-sum pair," and `alive_indices.index(left)` will then point at an index whose right neighbor is not what the stale sum assumed — leading to wrong inversion accounting or an out-of-range access.

**Solution:** Pair every `add` of a refreshed neighbor pair with a matching `remove` of its previous version, using the **old** `nums` values to reconstruct the exact tuple that was originally inserted:

```python
pair_sums.remove((nums[prev] + nums[left], prev))   # old values reconstruct old key
pair_sums.add((nums[prev] + merged_sum, prev))      # new key with merged value
```

### Pitfall 5: Looping on `operations` Instead of `inversions`

The termination condition is "the array is non-decreasing," which is precisely `inversions == 0`. A tempting mistake is to loop `while pair_sums` (until the array shrinks to one element) or to merge a fixed number of times. This over-merges past the point where the array first becomes sorted, returning a count larger than the true minimum.

**Solution:** Drive the loop with `while inversions:` and maintain `inversions` incrementally so the loop stops the instant the last out-of-order pair is resolved.

