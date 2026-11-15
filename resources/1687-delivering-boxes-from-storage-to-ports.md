# 1687. Delivering Boxes from Storage to Ports

> Source: https://algo.monster/liteproblems/1687

## Problem Statement

You need to deliver boxes from a storage warehouse to various ports using a single ship. The ship has constraints on both the number of boxes it can carry and the total weight capacity.

You're given an array `boxes` where each element `boxes[i] = [ports_i, weight_i]` contains:
- `ports_i`: the destination port for the i-th box
- `weight_i`: the weight of the i-th box

You're also given:
- `portsCount`: the total number of ports
- `maxBoxes`: maximum number of boxes the ship can carry at once
- `maxWeight`: maximum total weight the ship can carry at once

**Important constraints:**
- Boxes must be delivered in the exact order they appear in the array
- The ship starts at the storage warehouse

**How the delivery process works:**
1. The ship loads some consecutive boxes from the front of the [queue](/problems/queue_intro) (respecting both `maxBoxes` and `maxWeight` limits)
2. The ship delivers each loaded box in order to its destination port:
   - If the ship is already at the correct port, no trip is needed
   - If the ship needs to go to a different port, it makes one trip
3. After delivering all loaded boxes, the ship makes a return trip to the storage warehouse
4. The process repeats until all boxes are delivered

**Counting trips:**
- Going from storage to the first port counts as 1 trip
- Moving between different ports counts as 1 trip per move
- Returning from the last port to storage counts as 1 trip
- Staying at the same port for consecutive deliveries counts as 0 trips

Your task is to find the **minimum number of trips** needed to deliver all boxes to their respective ports.

**Example scenario:**
If the ship takes boxes going to ports [4, 4, 5]:
- Trip 1: Storage → Port 4
- Trip 0: Stay at Port 4 (second box also goes here)
- Trip 1: Port 4 → Port 5
- Trip 1: Port 5 → Storage
- Total: 3 trips for this batch

The challenge is to optimally group consecutive boxes for each loading while respecting the ship's capacity constraints to minimize the total number of trips.

## Intuition

Let's think about this problem step by step. Since boxes must be delivered in order, we're essentially deciding where to "cut" the sequence - which groups of consecutive boxes should be loaded together in each batch.

For any batch of consecutive boxes from index `j+1` to `i`, we can calculate the number of trips needed:
- 1 trip from storage to the first port
- Additional trips between different consecutive ports (if port changes)
- 1 trip back to storage
- Total = 2 + (number of port changes within the batch)

This naturally leads to a [dynamic programming](/problems/dynamic_programming_intro) approach. Let `f[i]` represent the minimum trips needed to deliver the first `i` boxes. To compute `f[i]`, we need to consider all valid previous positions `j` where we could have ended the previous batch, then taken boxes `j+1` through `i` as the current batch.

The key insight is that for any batch `[j+1, ..., i]`:
- It must satisfy weight constraint: `sum(weights[j+1]...weights[i]) ≤ maxWeight`
- It must satisfy box count constraint: `i - j ≤ maxBoxes`
- The trip cost is: `2 + (number of port changes in range)`

To efficiently calculate port changes, we can use prefix sums. If we create an array `c` where `c[k] = 1` if `port[k] != port[k+1]`, then the number of port changes from position `j+1` to `i` is simply `cs[i-1] - cs[j]` where `cs` is the [prefix sum](/problems/subarray_sum) of `c`.

The state transition becomes:
`f[i] = min(f[j] + cs[i-1] - cs[j] + 2)` for all valid `j`

However, with data scale reaching `10^5`, the O(n²) approach times out. We need optimization.

Looking at the transition equation more carefully:
`f[i] = min(f[j] - cs[j]) + cs[i-1] + 2`

The term `cs[i-1] + 2` is constant for a fixed `i`. We're essentially finding the minimum value of `f[j] - cs[j]` among all valid `j` values in the sliding window `[i-maxBoxes, i-1]` that also satisfy the weight constraint.

This is a classic sliding window minimum problem! We can use a monotonic deque to maintain potential candidates:
- Keep indices in increasing order of `f[j] - cs[j]` values
- Remove indices that fall outside the box count window
- Remove indices that violate the weight constraint
- The front of the deque gives us the optimal `j` for current `i`

This optimization reduces the complexity from O(n²) to O(n), making the solution efficient enough for the given constraints.

**Learn more about [Segment Tree](/problems/segment_tree_intro), [Queue](/problems/queue_intro), [Dynamic Programming](/problems/dynamic_programming_intro), [Prefix Sum](/problems/subarray_sum), [Monotonic Queue](/problems/mono_stack_intro) and [Heap (Priority Queue)](/problems/heap_intro) patterns.**

## Solution Approach

Let's implement the optimized [dynamic programming](/problems/dynamic_programming_intro) solution with monotonic [queue](/problems/queue_intro) step by step.

**Step 1: Preprocessing**

First, we need to prepare auxiliary arrays for efficient calculation:

```python
n = len(boxes)
ws = list(accumulate((box[1] for box in boxes), initial=0))
c = [int(a != b) for a, b in pairwise(box[0] for box in boxes)]
cs = list(accumulate(c, initial=0))
```

- `ws[i]`: [prefix sum](/problems/subarray_sum) of weights, where `ws[i]` = sum of weights from box 0 to box i-1
- `c[i]`: 1 if port changes between box i and i+1, 0 otherwise
- `cs[i]`: prefix sum of port changes

**Step 2: [Dynamic Programming](/problems/dynamic_programming_intro) with Monotonic [Queue](/problems/queue_intro)**

Initialize the DP array and deque:

```python
f = [0] * (n + 1)  # f[i] = minimum trips to deliver first i boxes
q = deque([0])     # monotonic [queue](/problems/queue_intro) storing indices
```

**Step 3: Main DP Loop**

For each position `i` from 1 to n:

```python
for i in range(1, n + 1):
    # Remove invalid indices from front of deque
    while q and (i - q[0] > maxBoxes or ws[i] - ws[q[0]] > maxWeight):
        q.popleft()
```

This step removes indices that:
- Violate box count constraint: `i - j > maxBoxes`
- Violate weight constraint: `ws[i] - ws[j] > maxWeight`

**Step 4: Calculate Minimum Trips**

```python
if q:
    f[i] = cs[i - 1] + f[q[0]] - cs[q[0]] + 2
```

The formula breaks down as:
- `f[q[0]]`: minimum trips to deliver first `q[0]` boxes
- `cs[i-1] - cs[q[0]]`: port changes between boxes `q[0]+1` to `i`
- `+2`: one trip from storage to first port, one trip back to storage

**Step 5: Maintain Monotonic Property**

```python
if i < n:
    while q and f[q[-1]] - cs[q[-1]] >= f[i] - cs[i]:
        q.pop()
    q.append(i)
```

We maintain the deque in increasing order of `f[j] - cs[j]` values:
- Remove indices from the back if their value is not better than current `i`
- Add current index `i` to the deque (only if not the last position)

**Why This Works:**

The monotonic [queue](/problems/queue_intro) ensures that:
1. The front element always gives the minimum value of `f[j] - cs[j]` in the valid window
2. Outdated or suboptimal indices are efficiently removed
3. Each index is added and removed at most once, giving O(n) complexity

**Time Complexity:** O(n) - each element is processed once through the deque
**Space Complexity:** O(n) - for the auxiliary arrays and DP array

The final answer is `f[n]`, representing the minimum trips needed to deliver all n boxes.

## Example Walkthrough

Let's walk through a concrete example to understand the solution approach.

**Input:**
- `boxes = [[1,2], [3,3], [3,1], [3,1], [4,4]]`
- `portsCount = 4`, `maxBoxes = 3`, `maxWeight = 6`

**Step 1: Preprocessing**

First, we build our auxiliary arrays:
- Weights: `[2, 3, 1, 1, 4]`
- Weight prefix sums `ws`: `[0, 2, 5, 6, 7, 11]`
- Port changes `c`: `[1, 0, 0, 1]` (comparing ports: 1→3, 3→3, 3→3, 3→4)
- Port change prefix sums `cs`: `[0, 1, 1, 1, 2]`

**Step 2: Initialize DP**
- `f = [0, 0, 0, 0, 0, 0]` (will fill positions 1-5)
- `q = deque([0])` (monotonic queue starts with index 0)

**Step 3: Process Each Position**

**i = 1** (Delivering first box [1,2]):
- Check deque front: Can we take boxes from position 0 to 1?
  - Box count: 1 - 0 = 1 ≤ 3 ✓
  - Weight: ws[1] - ws[0] = 2 - 0 = 2 ≤ 6 ✓
- Calculate: `f[1] = cs[0] + f[0] - cs[0] + 2 = 0 + 0 - 0 + 2 = 2`
  - This means: storage→port1→storage (2 trips)
- Update deque: Add index 1
- State: `f = [0, 2, 0, 0, 0, 0]`, `q = [0, 1]`

**i = 2** (Delivering first two boxes):
- Check deque front (index 0): 
  - Box count: 2 - 0 = 2 ≤ 3 ✓
  - Weight: ws[2] - ws[0] = 5 - 0 = 5 ≤ 6 ✓
- Calculate: `f[2] = cs[1] + f[0] - cs[0] + 2 = 1 + 0 - 0 + 2 = 3`
  - Taking both boxes: storage→port1→port3→storage (3 trips)
- Alternative from index 1: `f[2] = cs[1] + f[1] - cs[1] + 2 = 1 + 2 - 1 + 2 = 4`
  - Taking second box alone after first: more trips
- Deque maintenance: index 0 gives better value (0 - 0 = 0) than index 1 (2 - 1 = 1)
- State: `f = [0, 2, 3, 0, 0, 0]`, `q = [0, 2]`

**i = 3** (Delivering first three boxes):
- Check deque front (index 0):
  - Box count: 3 - 0 = 3 ≤ 3 ✓
  - Weight: ws[3] - ws[0] = 6 - 0 = 6 ≤ 6 ✓
- Calculate: `f[3] = cs[2] + f[0] - cs[0] + 2 = 1 + 0 - 0 + 2 = 3`
  - All three boxes: storage→port1→port3(stay)→port3(stay)→storage (3 trips)
- State: `f = [0, 2, 3, 3, 0, 0]`, `q = [0, 3]`

**i = 4** (Delivering first four boxes):
- Check deque front (index 0):
  - Box count: 4 - 0 = 4 > 3 ✗ (exceeds maxBoxes)
  - Remove index 0 from deque
- Next in deque (index 3):
  - Box count: 4 - 3 = 1 ≤ 3 ✓
  - Weight: ws[4] - ws[3] = 7 - 6 = 1 ≤ 6 ✓
- Calculate: `f[4] = cs[3] + f[3] - cs[3] + 2 = 1 + 3 - 1 + 2 = 5`
  - Take 4th box alone after first three
- State: `f = [0, 2, 3, 3, 5, 0]`, `q = [3, 4]`

**i = 5** (Delivering all five boxes):
- Check deque front (index 3):
  - Box count: 5 - 3 = 2 ≤ 3 ✓
  - Weight: ws[5] - ws[3] = 11 - 6 = 5 ≤ 6 ✓
- Calculate: `f[5] = cs[4] + f[3] - cs[3] + 2 = 2 + 3 - 1 + 2 = 6`
  - Take last two boxes together after first three
- Final answer: `f[5] = 6`

**Verification:**
- Batch 1: Boxes [1,2], [3,3], [3,1] → storage→port1→port3→storage (3 trips)
- Batch 2: Boxes [3,1], [4,4] → storage→port3→port4→storage (3 trips)
- Total: 6 trips ✓

## Python Solution

```python
class Solution:
    def boxDelivering(
        self, boxes: List[List[int]], portsCount: int, maxBoxes: int, maxWeight: int
    ) -> int:
        n = len(boxes)
      
        # Prefix sum of weights for quick range sum calculation
        weight_prefix_sum = list(accumulate((box[1] for box in boxes), initial=0))
      
        # Count transitions between different ports (1 if port changes, 0 otherwise)
        port_changes = [int(a != b) for a, b in pairwise(box[0] for box in boxes)]
      
        # Prefix sum of port changes for quick range calculation
        changes_prefix_sum = list(accumulate(port_changes, initial=0))
      
        # dp[i] represents minimum trips needed to deliver first i boxes
        dp = [0] * (n + 1)
      
        # Monotonic deque to maintain optimal starting points for current batch
        deque_indices = deque([0])
      
        for i in range(1, n + 1):
            # Remove indices that violate constraints (too many boxes or too heavy)
            while deque_indices and (
                i - deque_indices[0] > maxBoxes or 
                weight_prefix_sum[i] - weight_prefix_sum[deque_indices[0]] > maxWeight
            ):
                deque_indices.popleft()
          
            # Calculate minimum trips if we have valid starting points
            if deque_indices:
                # Trips = previous trips + port changes in current batch + 2 (pickup and delivery)
                start_idx = deque_indices[0]
                dp[i] = (changes_prefix_sum[i - 1] + dp[start_idx] - 
                        changes_prefix_sum[start_idx] + 2)
          
            # Prepare for next iteration (only if not at last box)
            if i < n:
                # Maintain monotonic property: remove suboptimal starting points
                while deque_indices and (
                    dp[deque_indices[-1]] - changes_prefix_sum[deque_indices[-1]] >= 
                    dp[i] - changes_prefix_sum[i]
                ):
                    deque_indices.pop()
                deque_indices.append(i)
      
        return dp[n]
```

## Time and Space Complexity

**Time Complexity:** `O(n)`, where `n` is the number of boxes.

The algorithm processes each box exactly once through the following operations:
- Building the weight prefix sum array `ws` takes `O(n)` time
- Creating the consecutive port difference array `c` takes `O(n)` time  
- Building the prefix sum array `cs` takes `O(n)` time
- The main dynamic programming loop runs `n` iterations
  - Each iteration performs deque operations (append, pop, popleft)
  - While there are nested while loops, each element is added to and removed from the deque at most once throughout the entire algorithm
  - This amortizes to `O(1)` per iteration
- Total: `O(n) + O(n) + O(n) + O(n) = O(n)`

**Space Complexity:** `O(n)`, where `n` is the number of boxes.

The algorithm uses the following data structures:
- `ws`: weight prefix sum array of size `n + 1`
- `c`: consecutive port difference array of size `n - 1`
- `cs`: prefix sum of differences array of size `n`
- `f`: dynamic programming array of size `n + 1`
- `q`: deque that can contain at most `min(n, maxBoxes)` elements, which is `O(n)` in the worst case
- Total: `O(n) + O(n) + O(n) + O(n) + O(n) = O(n)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Off-by-One Errors in Index Management**

**Pitfall:** The most common mistake is confusion between box indices and DP array indices. The DP array `dp[i]` represents the state after delivering the first `i` boxes (0-indexed), while `boxes[i-1]` is the i-th box. This leads to errors when:
- Calculating weight sums: Using `weight_prefix_sum[i] - weight_prefix_sum[j]` for boxes j to i-1
- Accessing port changes: Using `changes_prefix_sum[i-1]` for changes up to box i-1

**Solution:** Always be explicit about what each index represents:
```python
# When dp[i] represents first i boxes delivered:
# - The last box included is boxes[i-1]
# - Weight from box j to i-1: weight_prefix_sum[i] - weight_prefix_sum[j]
# - Port changes up to box i-1: changes_prefix_sum[i-1]
```

### 2. **Incorrect Monotonic Queue Maintenance**

**Pitfall:** Forgetting to skip adding the last index `n` to the deque. Since `dp[n]` is the final answer and won't be used as a starting point for future batches, adding it causes unnecessary comparisons and potential errors.

**Solution:** Add the guard condition before deque maintenance:
```python
if i < n:  # Critical: Don't add the last position to deque
    while deque_indices and (...):
        deque_indices.pop()
    deque_indices.append(i)
```

### 3. **Misunderstanding Port Change Calculation**

**Pitfall:** Incorrectly calculating port changes when the ship delivers multiple boxes. Remember that consecutive boxes to the same port require 0 additional trips, not 1 trip each.

**Example Error:**
```python
# WRONG: Counting each box delivery as a separate trip
trips = number_of_boxes + 1  # Incorrect!

# CORRECT: Only count trips when port changes
trips = port_changes_in_batch + 2  # +2 for initial and return trips
```

**Solution:** Use the preprocessed `port_changes` array which correctly marks transitions:
```python
port_changes = [int(boxes[i][0] != boxes[i+1][0]) for i in range(n-1)]
```

### 4. **Missing Edge Cases in Constraint Validation**

**Pitfall:** Not handling the case where even a single box exceeds constraints, or when the deque becomes empty after removing invalid indices.

**Solution:** Always check if the deque is non-empty before accessing:
```python
if deque_indices:  # Essential check
    dp[i] = changes_prefix_sum[i - 1] + dp[deque_indices[0]] - 
            changes_prefix_sum[deque_indices[0]] + 2
else:
    # This shouldn't happen with valid inputs, but good to handle
    dp[i] = float('inf')  # Or handle the error appropriately
```

### 5. **Inefficient Pairwise Implementation**

**Pitfall:** For Python versions before 3.10, `pairwise` isn't available in itertools, causing runtime errors.

**Solution:** Provide a fallback implementation or use manual iteration:
```python
# Option 1: Manual iteration
port_changes = [int(boxes[i][0] != boxes[i+1][0]) for i in range(len(boxes)-1)]

# Option 2: Custom pairwise for older Python versions
def pairwise(iterable):
    a, b = tee(iterable)
    next(b, None)
    return zip(a, b)
```

### 6. **Integer Overflow in Weight Calculations**

**Pitfall:** When dealing with large weight values and many boxes, prefix sums might overflow in languages with fixed integer sizes (though Python handles this automatically).

**Solution:** In Python this isn't an issue, but in other languages, use appropriate data types:
```python
# For other languages, ensure using long/int64 for weight calculations
weight_prefix_sum = [0]
for box in boxes:
    weight_prefix_sum.append(weight_prefix_sum[-1] + box[1])
```

