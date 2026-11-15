# 1383. Maximum Performance of a Team

> Source: https://algo.monster/liteproblems/1383

## Problem Statement

You have `n` engineers, where each engineer has two attributes:
- `speed[i]`: the speed of the i-th engineer
- `efficiency[i]`: the efficiency of the i-th engineer

Your task is to select **at most** `k` engineers to form a team that maximizes the team's performance.

The **performance** of a team is calculated as:
- `(sum of all selected engineers' speeds) × (minimum efficiency among selected engineers)`

For example, if you select 3 engineers with speeds `[10, 5, 8]` and efficiencies `[4, 7, 2]`, the performance would be:
- Sum of speeds: `10 + 5 + 8 = 23`
- Minimum efficiency: `min(4, 7, 2) = 2`
- Performance: `23 × 2 = 46`

You need to find the maximum possible performance by choosing the optimal subset of engineers (up to `k` engineers). Since the result can be very large, return it modulo `10^9 + 7`.

The key insight is that for any team composition, the performance is bottlenecked by the engineer with the lowest efficiency. This means you need to carefully balance between:
1. Including engineers with high speeds (to maximize the sum)
2. Maintaining a reasonable minimum efficiency (since it multiplies the entire sum)

The challenge lies in finding the optimal trade-off between these two factors while respecting the constraint of selecting at most `k` engineers.

## Intuition

The key observation is that once we fix the minimum efficiency for our team, we want to maximize the sum of speeds. This suggests we should consider each engineer as a potential "minimum efficiency" candidate and build the best team around them.

Let's think about this step by step:

1. **If we fix an engineer with efficiency `e` as our minimum**, then we can only include engineers with efficiency `≥ e` in our team (otherwise, they would become the new minimum and lower our efficiency factor).

2. **Among all engineers with efficiency `≥ e`, we want the ones with the highest speeds** to maximize our sum. This is because the performance formula is `sum × min_efficiency`, and once `min_efficiency` is fixed, we just need to maximize the sum.

3. **We should try each engineer as a potential minimum efficiency candidate**. For each candidate, we build the best possible team that includes them.

This leads us to the following approach:
- Sort engineers by efficiency in descending order
- Process each engineer one by one as the potential minimum efficiency
- As we go through engineers in decreasing efficiency order, all previously seen engineers have higher or equal efficiency, so they're all valid team members
- Among all valid engineers seen so far, we keep track of the `k-1` fastest ones (we need `k-1` because the current engineer must be included)
- Use a min-heap to efficiently maintain the top `k` fastest engineers

The beauty of this approach is that by processing engineers in decreasing efficiency order, we naturally maintain the invariant that all engineers in our candidate pool have efficiency greater than or equal to the current minimum. We just need to greedily select the fastest ones among them.

When we have more than `k` engineers in our pool, we remove the slowest one (smallest speed) to make room, ensuring we always have at most `k` engineers with the highest possible speed sum for the current minimum efficiency.

**Learn more about [Greedy](/problems/greedy_intro), [Sorting](/problems/sorting_summary) and [Heap (Priority Queue)](/problems/heap_intro) patterns.**

## Solution Approach

Let's walk through the implementation step by step:

1. **Create and sort engineer pairs**: First, we combine speeds and efficiencies into tuples and sort them by efficiency in descending order:
   ```python
   t = sorted(zip(speed, efficiency), key=lambda x: -x[1])
   ```
   This gives us engineers ordered from highest to lowest efficiency.

2. **Initialize variables**:
   - `ans`: tracks the maximum performance found
   - `tot`: maintains the current sum of speeds in our team
   - `h`: min-heap to store speeds of current team members
   - `mod`: the modulo value `10^9 + 7`

3. **Process each engineer as minimum efficiency**:
   ```python
   for s, e in t:
   ```
   For each engineer with speed `s` and efficiency `e`:

4. **Add current engineer to the team**:
   ```python
   tot += s
   ans = max(ans, tot * e)
   heappush(h, s)
   ```
   - Add their speed to the total sum
   - Calculate performance with current engineer as minimum efficiency: `tot * e`
   - Update maximum performance if this is better
   - Add the speed to our min-heap

5. **Maintain team size constraint**:
   ```python
   if len(h) == k:
       tot -= heappop(h)
   ```
   - If we now have `k` engineers in our heap, we need to remove one for the next iteration
   - Remove the engineer with minimum speed (top of min-heap) to make room
   - Subtract their speed from our total

   This ensures that at any point, we have at most `k` engineers, and they are the fastest ones among all engineers with efficiency `≥` current minimum.

6. **Return the result**:
   ```python
   return ans % mod
   ```

**Why this works**: 
- When we process engineer `i` with efficiency `e[i]`, all previously seen engineers have efficiency `≥ e[i]`
- The heap contains the speeds of the fastest `min(k, number_seen)` engineers so far
- By including the current engineer and calculating `tot * e[i]`, we're finding the best team with this engineer as the minimum efficiency
- The min-heap efficiently maintains the top-k speeds, removing the slowest when needed

**Time Complexity**: `O(n log n)` for [sorting](/problems/sorting_summary) + `O(n log k)` for heap operations = `O(n log n)`
**Space Complexity**: `O(n)` for the sorted array + `O(k)` for the heap

## Example Walkthrough

Let's walk through a concrete example with `n = 4` engineers, `k = 2` (select at most 2 engineers):
- `speed = [2, 10, 3, 1]`
- `efficiency = [5, 4, 3, 9]`

**Step 1: Create and sort engineer pairs by efficiency (descending)**
```
Engineers: [(1, 9), (2, 5), (10, 4), (3, 3)]
           (speed, efficiency)
```

**Step 2: Process each engineer as potential minimum efficiency**

**Iteration 1**: Engineer with speed=1, efficiency=9
- Add to team: `tot = 1`
- Performance: `1 × 9 = 9`
- Update `ans = 9`
- Heap contains: `[1]`
- Heap size < k, so no removal

**Iteration 2**: Engineer with speed=2, efficiency=5
- Available pool: engineers with efficiency ≥ 5 are `{1, 2}`
- Add to team: `tot = 1 + 2 = 3`
- Performance: `3 × 5 = 15`
- Update `ans = 15`
- Heap contains: `[1, 2]`
- Heap size = k = 2, remove minimum speed (1): `tot = 3 - 1 = 2`
- After removal, heap: `[2]`

**Iteration 3**: Engineer with speed=10, efficiency=4
- Available pool: engineers with efficiency ≥ 4 are `{2, 10}`
- Add to team: `tot = 2 + 10 = 12`
- Performance: `12 × 4 = 48`
- Update `ans = 48`
- Heap contains: `[2, 10]`
- Heap size = k = 2, remove minimum speed (2): `tot = 12 - 2 = 10`
- After removal, heap: `[10]`

**Iteration 4**: Engineer with speed=3, efficiency=3
- Available pool: engineers with efficiency ≥ 3 are `{10, 3}`
- Add to team: `tot = 10 + 3 = 13`
- Performance: `13 × 3 = 39`
- `ans` remains 48 (not better)
- Heap contains: `[3, 10]`

**Result**: Maximum performance = 48 (achieved by selecting engineers with speeds [2, 10] and efficiencies [5, 4], where min efficiency = 4)

The key insight illustrated here: When we fixed efficiency=4 as minimum (iteration 3), we could choose from engineers with efficiencies [9, 5, 4], and we greedily picked the two fastest ones: speeds [2, 10], giving us the optimal performance of 48.

## Python Solution

```python
from typing import List
import heapq

class Solution:
    def maxPerformance(
        self, n: int, speed: List[int], efficiency: List[int], k: int
    ) -> int:
        # Create pairs of (speed, efficiency) and sort by efficiency in descending order
        engineers = sorted(zip(speed, efficiency), key=lambda x: -x[1])
      
        max_performance = 0
        total_speed = 0
        MOD = 10**9 + 7
      
        # Min heap to store speeds of selected engineers
        # We use min heap to easily remove the engineer with minimum speed
        speed_heap = []
      
        # Iterate through engineers in decreasing order of efficiency
        for current_speed, current_efficiency in engineers:
            # Add current engineer's speed to total
            total_speed += current_speed
          
            # Calculate performance with current engineer as minimum efficiency
            # Performance = sum of speeds * minimum efficiency
            max_performance = max(max_performance, total_speed * current_efficiency)
          
            # Add current speed to heap
            heapq.heappush(speed_heap, current_speed)
          
            # If we have k engineers, remove the one with minimum speed
            # to make room for potentially better combinations
            if len(speed_heap) == k:
                total_speed -= heapq.heappop(speed_heap)
      
        return max_performance % MOD
```

## Time and Space Complexity

**Time Complexity:** `O(n log n + n log k)`

- Sorting the array of `(speed, efficiency)` pairs by efficiency in descending order takes `O(n log n)` time
- Iterating through all `n` engineers takes `O(n)` time
- For each engineer, we perform:
  - `heappush()` operation which takes `O(log k)` time in the worst case when heap size is `k`
  - `heappop()` operation (when heap size reaches `k`) which takes `O(log k)` time
- Overall: `O(n log n)` for sorting + `O(n log k)` for heap operations = `O(n log n + n log k)`
- Since `k ≤ n`, this simplifies to `O(n log n)`

**Space Complexity:** `O(n)`

- The sorted array `t` stores all `n` pairs of `(speed, efficiency)`, requiring `O(n)` space
- The min-heap `h` stores at most `k` elements, requiring `O(k)` space
- Since `k ≤ n`, the total space complexity is `O(n)` dominated by the sorted array

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Off-by-One Error in Team Size Management**

**The Pitfall:**
A common mistake is removing an engineer from the heap **before** calculating the performance with the current engineer. This leads to never actually considering teams of size `k`.

**Incorrect Implementation:**
```python
for current_speed, current_efficiency in engineers:
    # WRONG: Removing before calculation
    if len(speed_heap) == k:
        total_speed -= heapq.heappop(speed_heap)
  
    total_speed += current_speed
    max_performance = max(max_performance, total_speed * current_efficiency)
    heapq.heappush(speed_heap, current_speed)
```

**Why It's Wrong:**
- When we have `k-1` engineers and add the k-th one, we immediately remove one before calculating performance
- We never actually evaluate a team with exactly `k` engineers
- This results in suboptimal answers

**Correct Approach:**
```python
for current_speed, current_efficiency in engineers:
    # Add current engineer first
    total_speed += current_speed
    heapq.heappush(speed_heap, current_speed)
  
    # Calculate performance with current team (including new engineer)
    max_performance = max(max_performance, total_speed * current_efficiency)
  
    # THEN remove if we exceed k engineers
    if len(speed_heap) > k:  # Note: > k, not == k
        total_speed -= heapq.heappop(speed_heap)
```

### 2. **Forgetting to Apply Modulo at the Right Time**

**The Pitfall:**
Applying modulo during intermediate calculations instead of only at the final return.

**Incorrect Implementation:**
```python
# WRONG: Applying modulo too early
max_performance = max(max_performance, (total_speed * current_efficiency) % MOD)
```

**Why It's Wrong:**
- Modulo operation changes the actual value
- When comparing `max(a % MOD, b % MOD)`, you might get incorrect results
- Example: `max(1000000009 % MOD, 1000000008 % MOD)` = `max(2, 1)` = `2`, but the actual maximum should be based on `1000000009`

**Correct Approach:**
```python
# Keep track of actual maximum
max_performance = max(max_performance, total_speed * current_efficiency)

# Apply modulo only when returning
return max_performance % MOD
```

### 3. **Not Handling Edge Cases Properly**

**The Pitfall:**
Not considering cases where `k >= n` or when we should select fewer than `k` engineers.

**Why It Matters:**
- When `k >= n`, we might want to select all engineers, or just a subset
- The algorithm should naturally handle this by considering all possible team sizes up to `k`
- The current implementation handles this correctly by processing all engineers and only removing when heap size exceeds `k`

**Verification:**
The solution correctly handles these cases because:
- It considers every possible minimum efficiency
- For each minimum efficiency, it maintains the best possible team of size ≤ k
- It doesn't force exactly `k` engineers; it finds the optimal size ≤ k

