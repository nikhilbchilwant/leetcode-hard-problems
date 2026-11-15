# 857. Minimum Cost to Hire K Workers

> Source: https://algo.monster/liteproblems/857

## Problem Statement

You have `n` workers, each with a quality value and a minimum wage expectation. The goal is to hire exactly `k` workers while minimizing the total cost.

The hiring rules are:
1. Each hired worker must receive at least their minimum wage expectation
2. Within the hired group, pay must be proportional to quality (if worker A has twice the quality of worker B, then worker A must be paid exactly twice as much)

The key insight is that once you form a group, the pay ratio is determined by the worker with the highest wage-to-quality ratio in that group. This "captain" sets the wage rate for everyone else.

The solution sorts workers by their wage/quality ratio. For each potential captain (worker with highest ratio), it considers forming a group with that captain and the `k-1` workers with lowest quality among those with equal or lower wage/quality ratios. This ensures all workers meet their minimum wage requirement.

The algorithm uses a max heap to track the `k` workers with smallest quality. For each captain position, it calculates the total cost as `(captain's wage/quality ratio) × (sum of k workers' qualities)`. The heap helps efficiently maintain the k smallest qualities as we iterate through potential captains.

The time complexity is `O(n log n)` for [sorting](/problems/sorting_summary) and heap operations, and space complexity is `O(n)` for storing the sorted array and heap.

## Intuition

Let's think about what happens when we hire a group of workers. Since everyone's pay must be proportional to their quality, we can express each worker's pay as `pay[i] = quality[i] × rate`, where `rate` is the same for all workers in the group.

For each worker to meet their minimum wage expectation, we need `quality[i] × rate ≥ wage[i]`, which means `rate ≥ wage[i] / quality[i]`. 

Here's the crucial observation: the rate for the entire group must be at least the maximum `wage[i] / quality[i]` ratio among all workers in the group. This worker with the highest ratio becomes the "bottleneck" - they determine the minimum rate everyone must be paid at.

So if we fix a particular rate (determined by some worker as the bottleneck), the total cost becomes `rate × (sum of qualities of k workers)`. To minimize this cost for a fixed rate, we want to choose the `k` workers with the smallest qualities.

But which workers can we actually choose? We can only include workers whose `wage/quality` ratio is less than or equal to our chosen rate. Otherwise, they wouldn't meet their minimum wage expectation.

This leads us to the strategy: 
1. Sort workers by their `wage/quality` ratio
2. Consider each worker as a potential "captain" (the one setting the rate)
3. For each captain, form a group with them and the `k-1` workers with smallest qualities among those who appear before them in the sorted order (these have lower or equal ratios)
4. Calculate the cost for each such group and keep track of the minimum

The [sorting](/problems/sorting_summary) ensures that when we consider a worker as captain, all previous workers in the sorted list are eligible to be in their group. We use a heap to efficiently maintain the `k` smallest qualities as we iterate through potential captains.

**Learn more about [Greedy](/problems/greedy_intro), [Sorting](/problems/sorting_summary) and [Heap (Priority Queue)](/problems/heap_intro) patterns.**

## Solution Approach

The implementation follows our intuition step by step:

1. **Sort workers by wage/quality ratio**: We create tuples of `(quality, wage)` and sort them by `wage/quality` ratio using a lambda function `lambda worker: worker[1] / worker[0]`. This ensures workers are ordered from lowest to highest ratio.

2. **Initialize variables**:
   - `min_cost = inf`: Tracks the minimum cost found so far
   - `total_quality = 0`: Maintains the sum of qualities in our current group
   - `max_heap = []`: A max heap (using negative values in Python's min heap) to store qualities

3. **Iterate through sorted workers**: For each worker `(worker_quality, worker_wage)` in sorted order:
   - Add their quality to the running total: `total_quality += worker_quality`
   - Push their negative quality to the heap: `heappush(max_heap, -worker_quality)` (negative because Python only has min heap)
 
4. **When we have k workers** (`len(max_heap) == k`):
   - Calculate cost with current worker as captain: `worker_wage / worker_quality * total_quality`
     - `worker_wage / worker_quality` is the captain's wage/quality ratio (the rate)
     - `total_quality` is the sum of k smallest qualities
     - Their product gives the total cost
   - Update minimum cost: `min_cost = min(min_cost, worker_wage / worker_quality * total_quality)`
   - Remove the worker with highest quality to make room for next iteration: `removed_quality = -heappop(max_heap)`, then `total_quality -= removed_quality`
     - We pop the largest quality (most negative value in heap) and negate it back to a positive quality
     - Subtracting it from `total_quality` removes it from the sum

5. **Return the minimum cost** found across all valid groups.

The key insight is that as we iterate through workers in increasing wage/quality ratio order, each worker becomes the new captain (highest ratio in group). The heap maintains the `k` workers with smallest qualities that can be paired with this captain, ensuring we always compute the minimum possible cost for each captain choice.

## Example Walkthrough

Let's walk through a small example with `n = 4` workers and we want to hire `k = 2`:
- Worker 0: quality = 10, wage = 70 (ratio = 7.0)
- Worker 1: quality = 20, wage = 50 (ratio = 2.5)
- Worker 2: quality = 5, wage = 30 (ratio = 6.0)
- Worker 3: quality = 4, wage = 28 (ratio = 7.0)

**Step 1: Sort by wage/quality ratio**
After sorting:
- Worker 1: quality = 20, wage = 50 (ratio = 2.5)
- Worker 2: quality = 5, wage = 30 (ratio = 6.0)
- Worker 0: quality = 10, wage = 70 (ratio = 7.0)
- Worker 3: quality = 4, wage = 28 (ratio = 7.0)

**Step 2: Iterate through sorted workers**

*Iteration 1 (Worker 1 as potential captain):*
- Add quality 20 to heap: max_heap = [-20], total_quality = 20
- Heap size < k, continue

*Iteration 2 (Worker 2 as captain):*
- Add quality 5 to heap: max_heap = [-20, -5], total_quality = 25
- Heap size = k = 2, calculate cost:
  - Captain's ratio: 30/5 = 6.0
  - Total qualities: 25
  - Cost = 6.0 × 25 = 150
- Update min_cost = 150
- Remove largest quality (20): total_quality = 25 - 20 = 5

*Iteration 3 (Worker 0 as captain):*
- Add quality 10 to heap: max_heap = [-10, -5], total_quality = 15
- Heap size = k = 2, calculate cost:
  - Captain's ratio: 70/10 = 7.0
  - Total qualities: 15
  - Cost = 7.0 × 15 = 105
- Update min_cost = min(150, 105) = 105
- Remove largest quality (10): total_quality = 15 - 10 = 5

*Iteration 4 (Worker 3 as captain):*
- Add quality 4 to heap: max_heap = [-5, -4], total_quality = 9
- Heap size = k = 2, calculate cost:
  - Captain's ratio: 28/4 = 7.0
  - Total qualities: 9
  - Cost = 7.0 × 9 = 63
- Update min_cost = min(105, 63) = 63

**Final answer: 63**

The optimal group consists of Workers 2 and 3, with Worker 3 as the captain setting the rate at 7.0. Worker 2 gets paid 5 × 7.0 = 35 (above their minimum of 30), and Worker 3 gets paid 4 × 7.0 = 28 (exactly their minimum).

## Python Solution

```python
import heapq
from typing import List
from math import inf

class Solution:
    def mincostToHireWorkers(
        self, quality: List[int], wage: List[int], k: int
    ) -> float:
        # Create pairs of (quality, wage) and sort by wage/quality ratio
        # Workers with lower ratio are more cost-efficient
        workers = sorted(zip(quality, wage), key=lambda worker: worker[1] / worker[0])
      
        min_cost = inf
        total_quality = 0
        max_heap = []  # Max heap to track k workers with smallest quality
      
        for worker_quality, worker_wage in workers:
            # Add current worker's quality to the total
            total_quality += worker_quality
            # Use negative value to simulate max heap (Python has min heap by default)
            heapq.heappush(max_heap, -worker_quality)
          
            # When we have exactly k workers
            if len(max_heap) == k:
                # Calculate cost using current worker's wage/quality ratio
                # All selected workers must be paid at this ratio or better
                current_ratio = worker_wage / worker_quality
                current_cost = current_ratio * total_quality
                min_cost = min(min_cost, current_cost)
              
                # Remove worker with highest quality to make room for next iteration
                # This keeps the k-1 workers with lowest quality
                removed_quality = -heapq.heappop(max_heap)
                total_quality -= removed_quality
      
        return min_cost
```

## Time and Space Complexity

**Time Complexity:** `O(n log n)` where `n` is the length of the quality/wage arrays.

- Sorting the workers by their wage-to-quality ratio takes `O(n log n)`
- The main loop iterates through all `n` workers
- Inside the loop, each `heappush` operation takes `O(log k)` where `k` is the size of the heap
- When the heap size reaches `k`, `heappop` also takes `O(log k)`
- Since `k ≤ n`, the heap operations are bounded by `O(log n)`
- Total: `O(n log n)` for sorting + `O(n log n)` for heap operations = `O(n log n)`

**Space Complexity:** `O(n)`

- The sorted list `workers` stores all `n` worker pairs: `O(n)`
- The max heap `max_heap` stores at most `k` elements: `O(k)` where `k ≤ n`
- Other variables (`min_cost`, `total_quality`, `worker_quality`, `worker_wage`) use constant space: `O(1)`
- Total: `O(n) + O(k) = O(n)` since `k ≤ n`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Integer Division Instead of Float Division**
A critical pitfall is using integer division when calculating the wage/quality ratio, which can lead to incorrect sorting and wrong results.

**Incorrect:**
```python
workers = sorted(zip(quality, wage), key=lambda worker: worker[1] // worker[0])  # Wrong!
```

**Correct:**
```python
workers = sorted(zip(quality, wage), key=lambda worker: worker[1] / worker[0])
```

### 2. **Forgetting to Use Negative Values for Max Heap**
Python's heapq module only provides a min heap. To simulate a max heap, you must push negative values. Forgetting this will keep the workers with highest quality instead of lowest.

**Incorrect:**
```python
heapq.heappush(max_heap, worker_quality)  # Wrong - min heap of qualities
removed_quality = heapq.heappop(max_heap)  # Wrong - removes the smallest quality
```

**Correct:**
```python
heapq.heappush(max_heap, -worker_quality)  # Push negative
removed_quality = -heapq.heappop(max_heap)  # Pop and negate back
```

### 3. **Updating Total Quality Incorrectly After Popping**
The heap stores negated qualities, so `heapq.heappop(max_heap)` returns `-quality`. Mixing up the sign adds the removed quality instead of subtracting it.

**Incorrect:**
```python
total_quality -= heapq.heappop(max_heap)  # Wrong - subtracting a negative value adds the quality
```

**Correct (either form):**
```python
removed_quality = -heapq.heappop(max_heap)
total_quality -= removed_quality

# Equivalent: adding the negative value subtracts the quality
total_quality += heapq.heappop(max_heap)
```

### 4. **Calculating Cost Before Having k Workers**
Computing the cost when you have fewer than k workers will give invalid results. Always check that the heap size equals k.

**Incorrect:**
```python
for worker_quality, worker_wage in workers:
    total_quality += worker_quality
    heapq.heappush(max_heap, -worker_quality)
    # Wrong - calculating cost regardless of group size
    current_cost = (worker_wage / worker_quality) * total_quality
    min_cost = min(min_cost, current_cost)
```

**Correct:**
```python
if len(max_heap) == k:
    current_cost = (worker_wage / worker_quality) * total_quality
    min_cost = min(min_cost, current_cost)
```

### 5. **Not Handling Edge Cases**
Ensure your solution handles edge cases like when k equals n (all workers must be hired) or when k equals 1.

**Solution:** The current implementation handles these naturally, but always verify with test cases:
```python
# Edge case: k = 1 (hire cheapest worker)
# Edge case: k = n (must hire all workers)
```

