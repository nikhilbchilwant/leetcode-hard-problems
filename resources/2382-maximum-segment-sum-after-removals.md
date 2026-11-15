# 2382. Maximum Segment Sum After Removals

> Source: https://algo.monster/liteproblems/2382

## Problem Statement

You have an array `nums` and another array `removeQueries`, both containing `n` elements. The task is to process removal queries one by one and track the maximum segment sum after each removal.

Here's how the process works:

1. For each query at index `i`, you remove the element at position `removeQueries[i]` from `nums`
2. When an element is removed, it may split the array into different segments
3. A **segment** is defined as a contiguous sequence of positive (non-removed) integers
4. A **segment sum** is the total sum of all elements in that segment

The goal is to return an array `answer` where `answer[i]` represents the maximum segment sum that exists after applying the first `i+1` removals.

Key points to understand:
- Initially, all elements form one continuous segment
- As elements are removed, they create breaks in the array, forming multiple segments
- After each removal, you need to find which segment has the largest sum
- Each index will only be removed once (no duplicate removals)
- The segments only consist of elements that haven't been removed yet

For example, if `nums = [1, 2, 5, 6, 1]` and you remove index 2 (value 5), you get two segments: `[1, 2]` with sum 3 and `[6, 1]` with sum 7. The maximum segment sum would be 7.

## Intuition

The key insight is recognizing that removing elements and tracking maximum segment sums is difficult to do directly, but the reverse process - adding elements and merging segments - is much more manageable.

Think about it this way: if we process removals forward, each removal potentially splits a segment into two parts, and we'd need to:
1. Find which segment contains the element to be removed
2. Split that segment into two new segments
3. Recalculate sums for the new segments
4. Track all existing segments to find the maximum

This forward approach becomes complex because we're constantly breaking things apart and need to maintain information about all segments.

However, if we think backwards, the problem becomes much simpler. Starting from a state where all elements are removed (no segments exist), we can add elements back in reverse order of their removal. When we add an element back:
1. It initially forms its own segment with sum `nums[i]`
2. If adjacent positions already have segments, we merge them into one larger segment
3. We only need to check at most two neighbors (left and right)

This reverse approach naturally fits the Union-Find data structure pattern:
- Each position starts as its own component (or not yet added)
- When we add an element, we check if its neighbors are active
- If neighbors are active, we union the components and combine their sums
- We track the maximum segment sum as we build up the segments

The beauty of this approach is that Union-Find efficiently handles:
- Finding which segment a position belongs to (`find` operation)
- Merging adjacent segments (`merge` operation)
- Tracking segment sums (stored in the root of each component)

By processing in reverse, we transform a "breaking apart" problem into a "building up" problem, which is fundamentally easier to handle algorithmically. The answer for query `i` is the maximum segment sum after all removals from `i+1` onwards have been "undone" by our reverse addition process.

**Learn more about [Union Find](/problems/dsu_intro) and [Prefix Sum](/problems/subarray_sum) patterns.**

## Solution Approach

The solution implements a reverse Union-Find approach with the following components:

**Data Structures:**
- `p[]`: Parent array for Union-Find, where `p[i]` stores the parent of position `i`
- `s[]`: Sum array, where `s[i]` stores the segment sum if `i` is a root, otherwise 0
- `ans[]`: Result array to store maximum segment sums after each removal
- `mx`: Variable to track the current maximum segment sum

**Union-Find Operations:**

1. **Find with Path Compression:**
```python
def find(x):
    if p[x] != x:
        p[x] = find(p[x])  # Path compression
    return p[x]
```
This recursively finds the root of the component containing `x` and compresses the path for efficiency.

2. **Merge Operation:**
```python
def merge(a, b):
    pa, pb = find(a), find(b)
    p[pa] = pb  # Make pb the root
    s[pb] += s[pa]  # Combine segment sums
```
This merges two segments by making one root point to the other and combining their sums.

**Main Algorithm:**

1. **Initialization:**
   - Create parent array `p = [0, 1, 2, ..., n-1]` (each element is its own parent)
   - Initialize sum array `s = [0, 0, ..., 0]` (all segments start empty)
   - Set `mx = 0` (no segments initially)

2. **Reverse Processing:**
   - Loop from `j = n-1` down to `j = 1` (processing removals in reverse)
   - For each iteration, we're "adding back" the element at index `i = removeQueries[j]`

3. **Adding Elements Back:**
   - Set `s[i] = nums[i]` (create a new segment with just this element)
   - Check left neighbor: If `i > 0` and `s[find(i-1)] > 0`, merge with left segment
   - Check right neighbor: If `i < n-1` and `s[find(i+1)] > 0`, merge with right segment
   - The condition `s[find(neighbor)] > 0` checks if the neighbor is part of an active segment

4. **Update Maximum:**
   - After potential merges, update `mx = max(mx, s[find(i)])`
   - Store this maximum in `ans[j-1]` (the answer after `j` removals)

5. **Final State:**
   - `ans[0]` remains 0 (all elements removed)
   - Return the complete `ans` array

**Time Complexity:** `O(n × α(n))` where `α` is the inverse Ackermann function (practically constant)

**Space Complexity:** `O(n)` for the Union-Find structure and answer array

The elegance of this solution lies in reversing the problem: instead of tracking how segments break apart during removals, we track how they merge together during additions, making the implementation clean and efficient.

## Example Walkthrough

Let's walk through a small example to illustrate the reverse Union-Find approach.

**Input:**
- `nums = [3, 2, 11, 1]`
- `removeQueries = [3, 2, 1, 0]`

We need to process removals and track the maximum segment sum after each removal.

**Initial Setup:**
- `p = [0, 1, 2, 3]` (each index is its own parent)
- `s = [0, 0, 0, 0]` (no segments exist yet)
- `ans = [0, 0, 0, 0]` (to be filled)
- `mx = 0` (current maximum segment sum)

**Processing in Reverse (adding elements back):**

**Step 1: j=3 (adding back index 0, value 3)**
- Last removal was index 0, so we add it back
- `s[0] = 3` (create segment [3])
- Check neighbors:
  - Left: none (index -1 doesn't exist)
  - Right: `s[find(1)] = 0` (index 1 not active)
- No merges needed
- `mx = max(0, 3) = 3`
- `ans[2] = 3`
- State: `[3]` `_` `_` `_` (segments shown, _ means removed)

**Step 2: j=2 (adding back index 1, value 2)**
- Add back index 1
- `s[1] = 2` (create segment [2])
- Check neighbors:
  - Left: `s[find(0)] = 3 > 0` (index 0 is active) → merge!
  - After merge: `p[0] = 1`, `s[1] = 3 + 2 = 5`
  - Right: `s[find(2)] = 0` (index 2 not active)
- `mx = max(3, 5) = 5`
- `ans[1] = 5`
- State: `[3, 2]` `_` `_` (one segment with sum 5)

**Step 3: j=1 (adding back index 2, value 11)**
- Add back index 2
- `s[2] = 11` (create segment [11])
- Check neighbors:
  - Left: `s[find(1)] = 5 > 0` (index 1 is active) → merge!
  - After merge: `p[1] = 2`, `s[2] = 5 + 11 = 16`
  - Right: `s[find(3)] = 0` (index 3 not active)
- `mx = max(5, 16) = 16`
- `ans[0] = 16`
- State: `[3, 2, 11]` `_` (one segment with sum 16)

**Step 4: j=0 (all elements removed)**
- `ans[0]` was already set to 16 in the previous step
- We don't process j=0 as it represents the state before any removals

**Final Answer:** `[16, 5, 3, 0]`

**Verification (forward perspective):**
- After 1 removal (index 3): `[3, 2, 11, _]` → max segment sum = 16 ✓
- After 2 removals (indices 3, 2): `[3, 2, _, _]` → max segment sum = 5 ✓
- After 3 removals (indices 3, 2, 1): `[3, _, _, _]` → max segment sum = 3 ✓
- After 4 removals (all): `[_, _, _, _]` → max segment sum = 0 ✓

The reverse approach elegantly builds up segments by adding elements back, using Union-Find to efficiently merge adjacent segments and track the maximum sum at each step.

## Python Solution

```python
class Solution:
    def maximumSegmentSum(self, nums: List[int], removeQueries: List[int]) -> List[int]:
        def find(x: int) -> int:
            """Find the root parent of element x with path compression."""
            if parent[x] != x:
                parent[x] = find(parent[x])  # Path compression
            return parent[x]

        def union(a: int, b: int) -> None:
            """Unite two segments by merging element a's segment into element b's segment."""
            root_a, root_b = find(a), find(b)
            parent[root_a] = root_b
            segment_sum[root_b] += segment_sum[root_a]

        n = len(nums)
      
        # Initialize Union-Find structures
        parent = list(range(n))  # Each element is initially its own parent
        segment_sum = [0] * n     # Sum of each segment (initially all 0)
      
        # Result array to store maximum segment sum after each removal
        result = [0] * n
        max_segment_sum = 0
      
        # Process removals in reverse order (building segments by adding elements back)
        for query_idx in range(n - 1, 0, -1):
            # Get the index of element to add back
            element_idx = removeQueries[query_idx]
          
            # Initialize this element's segment with its own value
            segment_sum[element_idx] = nums[element_idx]
          
            # Check and merge with left neighbor if it exists and is active
            if element_idx > 0 and segment_sum[find(element_idx - 1)] > 0:
                union(element_idx, element_idx - 1)
          
            # Check and merge with right neighbor if it exists and is active
            if element_idx < n - 1 and segment_sum[find(element_idx + 1)] > 0:
                union(element_idx, element_idx + 1)
          
            # Update maximum segment sum
            max_segment_sum = max(max_segment_sum, segment_sum[find(element_idx)])
          
            # Store the maximum segment sum for this state
            result[query_idx - 1] = max_segment_sum
      
        return result
```

## Time and Space Complexity

**Time Complexity:** `O(n × α(n))` where `n` is the length of the array and `α(n)` is the inverse Ackermann function.

- The main loop runs `n-1` times (from `n-1` down to `1`)
- Inside each iteration:
  - Setting `s[i] = nums[i]` takes `O(1)`
  - The find operations and merge operations use Union-Find with path compression
  - Each `find()` operation takes `O(α(n))` amortized time due to path compression
  - Each `merge()` operation calls `find()` twice and performs constant work, so it's `O(α(n))`
  - We perform at most 2 merge operations per iteration (merging with left and right neighbors)
  - The `max()` operation takes `O(1)`
- Total: `O(n × α(n))`, which is nearly linear since `α(n)` is effectively constant for all practical values of `n`

**Space Complexity:** `O(n)`

- The parent array `p` uses `O(n)` space
- The sum array `s` uses `O(n)` space  
- The answer array `ans` uses `O(n)` space
- The recursion stack for `find()` can go up to `O(log n)` in the worst case with path compression, but this doesn't change the overall complexity
- Total: `O(n)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Union Direction Leading to Lost Segment Sums**

**The Pitfall:**
One of the most common mistakes is implementing the union operation incorrectly, particularly regarding which root becomes the parent and where the sum is accumulated:

```python
# INCORRECT implementation
def union(a: int, b: int) -> None:
    root_a, root_b = find(a), find(b)
    parent[root_a] = root_b
    segment_sum[root_a] += segment_sum[root_b]  # Wrong! Sum gets lost
```

In this incorrect version, we make `root_b` the parent but update the sum at `root_a`, which is no longer the root. Future `find()` operations will return `root_b`, which won't have the correct sum.

**The Solution:**
Always accumulate the sum at the node that will be the root after the union:

```python
# CORRECT implementation
def union(a: int, b: int) -> None:
    root_a, root_b = find(a), find(b)
    parent[root_a] = root_b
    segment_sum[root_b] += segment_sum[root_a]  # Correct! Sum at the new root
```

### 2. **Checking Neighbor Activity Before Finding Root**

**The Pitfall:**
Checking if a neighbor is active by directly examining `segment_sum[i-1]` or `segment_sum[i+1]` instead of checking the root's sum:

```python
# INCORRECT: Checking non-root nodes
if element_idx > 0 and segment_sum[element_idx - 1] > 0:  # Wrong!
    union(element_idx, element_idx - 1)
```

This fails because only root nodes maintain the actual segment sum. Non-root nodes have `segment_sum = 0` after being merged.

**The Solution:**
Always check the sum at the root of the neighbor:

```python
# CORRECT: Check the root's sum
if element_idx > 0 and segment_sum[find(element_idx - 1)] > 0:
    union(element_idx, element_idx - 1)
```

### 3. **Union Order Affects Final Root**

**The Pitfall:**
The order of arguments in the union function matters. Consider this scenario:

```python
# If we have segments [1,2] and [3,4,5] where 2 and 3 are being connected
union(2, 3)  # Makes find(3) the root
# vs
union(3, 2)  # Makes find(2) the root
```

While both are functionally correct, consistently using the wrong order might cause confusion when debugging or if the implementation assumes a specific pattern.

**The Solution:**
Be consistent with union order. The provided solution always unions in the pattern `union(current_element, neighbor)`, making the neighbor's root the parent. This is arbitrary but should be consistent throughout.

### 4. **Forgetting to Update Maximum After Each Addition**

**The Pitfall:**
Only checking the final segment sum without comparing it to the previous maximum:

```python
# INCORRECT: Overwriting instead of comparing
max_segment_sum = segment_sum[find(element_idx)]  # Wrong!
```

**The Solution:**
Always compare with the existing maximum:

```python
# CORRECT: Compare and keep the maximum
max_segment_sum = max(max_segment_sum, segment_sum[find(element_idx)])
```

### 5. **Off-by-One Error in Result Array Indexing**

**The Pitfall:**
Confusion about where to store results when processing in reverse:

```python
# Processing query at index query_idx (going from n-1 to 1)
# INCORRECT placements:
result[query_idx] = max_segment_sum      # Wrong! Off by one
result[n - query_idx] = max_segment_sum  # Wrong! Reversed incorrectly
```

**The Solution:**
Remember that when processing query `j` in reverse, we're computing the state after the first `j` removals, which should be stored at index `j-1`:

```python
# CORRECT: Store at query_idx - 1
result[query_idx - 1] = max_segment_sum
```

This is because `result[i]` represents the maximum segment sum after `i+1` removals.

