# 1883. Minimum Skips to Arrive at Meeting On Time

> Source: https://algo.monster/liteproblems/1883

## Problem Statement

You need to travel through `n` roads to reach a meeting within `hoursBefore` hours. Each road has a specific length given in the array `dist`, where `dist[i]` represents the length of the i-th road in kilometers. You travel at a constant `speed` (in km/h).

**Key Rules:**
1. After traveling each road (except the last one), you must rest until the next integer hour before continuing.
   - Example: If traveling a road takes 1.4 hours, you must wait until hour 2 before starting the next road
   - If traveling takes exactly 2 hours, no waiting is needed

2. You can choose to **skip** some rest periods to save time and arrive on time.
   - When you skip a rest, you immediately continue to the next road without waiting
   - Example: If road 1 takes 1.4 hours and road 2 takes 0.6 hours, skipping the rest after road 1 means you finish road 2 at exactly 2 hours total

3. The last road doesn't require rest since you've already arrived at the meeting.

**Goal:** Find the minimum number of rest periods you need to skip to arrive within `hoursBefore` hours. Return `-1` if it's impossible to arrive on time even by skipping all rests.

## Intuition

The key insight is that we need to find the optimal balance between skipping and not skipping rest periods. Each skip saves us waiting time, but we have a limited number of skips that would be useful.

Think about it this way: if we travel a road in 1.4 hours, we waste 0.6 hours waiting for the next integer hour. By skipping this rest, we save those 0.6 hours. However, not all skips are equally valuable - some save more time than others.

This naturally leads to a [dynamic programming](/problems/dynamic_programming_intro) approach where we track the minimum time needed for different numbers of skips. We want to explore all possibilities: what's the minimum time if we skip 0 rests? 1 rest? 2 rests? And so on.

For each road we encounter, we face a decision: should we rest after this road or skip the rest? This decision depends on:
- How much time we've already spent
- How many skips we've already used
- Whether skipping here would help us arrive on time

The state `f[i][j]` captures this perfectly - it tells us the minimum time to complete `i` roads with exactly `j` skips. For each new road, we can:
1. **Not skip the rest**: We add the road's travel time and round up to the next integer hour (using ceiling function)
2. **Skip the rest**: We simply add the road's travel time without rounding

By building up from smaller subproblems (fewer roads, fewer skips) to larger ones (all roads, various skip counts), we can find the minimum number of skips needed. We just check which is the smallest `j` where `f[n][j] ≤ hoursBefore`.

The epsilon (`eps = 1e-8`) handling is a technical detail to avoid floating-point precision issues when using the ceiling function and comparing with `hoursBefore`.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

We implement a [dynamic programming](/problems/dynamic_programming_intro) solution where `f[i][j]` represents the minimum time needed to travel the first `i` roads while skipping exactly `j` rest periods.

**Initialization:**
- Create a 2D array `f` of size `(n+1) × (n+1)` initialized with infinity
- Set `f[0][0] = 0` (no roads traveled, no skips used, zero time)
- Define `eps = 1e-8` to handle floating-point precision issues

**State Transitions:**
For each road `i` (from 1 to n) and each possible number of skips `j` (from 0 to i):

1. **Option 1 - Don't skip the rest after road i:**
   - Only valid when `j < i` (we can't skip more rests than roads minus one)
   - Time needed: `ceil(f[i-1][j] + dist[i-1]/speed - eps)`
   - We subtract `eps` before ceiling to avoid floating-point errors
   - The ceiling function ensures we wait until the next integer hour

2. **Option 2 - Skip the rest after road i:**
   - Only valid when `j > 0` (we need at least one skip available)
   - Time needed: `f[i-1][j-1] + dist[i-1]/speed`
   - No ceiling needed since we're skipping the rest period

The state transition equation becomes:
```
f[i][j] = min(
    ceil(f[i-1][j] + dist[i-1]/speed - eps),  // don't skip
    f[i-1][j-1] + dist[i-1]/speed              // skip (if j > 0)
)
```

**Finding the Answer:**
After filling the DP table, we iterate through `j` from 0 to n:
- Check if `f[n][j] <= hoursBefore + eps`
- Return the first (minimum) `j` that satisfies this condition
- If no valid `j` exists, return `-1`

The `eps` addition when comparing with `hoursBefore` compensates for the earlier subtraction and ensures numerical stability in our floating-point comparisons.

**Time Complexity:** `O(n²)` for filling the DP table  
**Space Complexity:** `O(n²)` for the DP array

## Example Walkthrough

Let's walk through a concrete example to understand the solution approach.

**Input:** `dist = [2, 3, 2]`, `speed = 2`, `hoursBefore = 6`

This means we have 3 roads of lengths 2km, 3km, and 2km, traveling at 2 km/h, and need to arrive within 6 hours.

**Step 1: Calculate travel times for each road**
- Road 1: 2km ÷ 2km/h = 1.0 hours
- Road 2: 3km ÷ 2km/h = 1.5 hours  
- Road 3: 2km ÷ 2km/h = 1.0 hours

**Step 2: Build the DP table**

We'll create a table `f[i][j]` where rows represent roads completed and columns represent skips used.

Initial state: `f[0][0] = 0` (no roads, no time)

**For Road 1 (i=1):**
- `f[1][0]`: No skips → travel 1.0 hour, rest until hour 1 → ceil(0 + 1.0) = 1.0
- `f[1][1]`: Can't skip (no previous road to skip after)

**For Road 2 (i=2):**
- `f[2][0]`: No skips → from f[1][0]=1.0, add 1.5 hours, rest → ceil(1.0 + 1.5) = 3.0
- `f[2][1]`: Skip once
  - Option 1: Don't skip after road 2 → ceil(f[1][1] + 1.5) = ∞ (f[1][1] doesn't exist)
  - Option 2: Skip after road 1 → f[1][0] + 1.5 = 1.0 + 1.5 = 2.5
  - Choose: 2.5
- `f[2][2]`: Can't use 2 skips for 2 roads (max 1 skip possible)

**For Road 3 (i=3):**
- `f[3][0]`: No skips → from f[2][0]=3.0, add 1.0 hour (no rest after last road) → 3.0 + 1.0 = 4.0
- `f[3][1]`: Skip once
  - Option 1: Don't skip after road 3 (last road, no rest anyway) → f[2][1] + 1.0 = 2.5 + 1.0 = 3.5
  - Option 2: Skip after road 2 → ceil(f[2][0] + 1.0) = ceil(3.0 + 1.0) = 4.0
  - Choose: min(3.5, 4.0) = 3.5
- `f[3][2]`: Skip twice
  - From f[2][1]=2.5, skip after road 2 → 2.5 + 1.0 = 3.5
  - (Other option gives ∞)
  - Choose: 3.5

**Step 3: Find the answer**

Final DP table:
```
     j=0   j=1   j=2   j=3
i=0  0     ∞     ∞     ∞
i=1  1.0   ∞     ∞     ∞
i=2  3.0   2.5   ∞     ∞
i=3  4.0   3.5   3.5   ∞
```

Check f[3][j] values against hoursBefore = 6:
- f[3][0] = 4.0 ≤ 6 ✓
- f[3][1] = 3.5 ≤ 6 ✓
- f[3][2] = 3.5 ≤ 6 ✓

The minimum j where f[3][j] ≤ 6 is j=0, so we need to skip **0** rest periods to arrive on time.

**Verification:**
- Travel road 1: 1.0 hour, rest until hour 1
- Travel road 2: 1.5 hours (from hour 1 to 2.5), rest until hour 3
- Travel road 3: 1.0 hour (from hour 3 to 4)
- Total: 4 hours ≤ 6 hours ✓

## Python Solution

```python
class Solution:
    def minSkips(self, dist: List[int], speed: int, hoursBefore: int) -> int:
        """
        Find the minimum number of skips needed to arrive before hoursBefore.
        A skip means not resting after completing a road segment.
      
        Args:
            dist: List of distances for each road segment
            speed: Constant travel speed
            hoursBefore: Maximum allowed total travel time
          
        Returns:
            Minimum number of skips needed, or -1 if impossible
        """
        num_segments = len(dist)
      
        # dp[i][j] represents minimum time to complete first i segments with j skips
        # Initialize with infinity to represent impossible states
        dp = [[float('inf')] * (num_segments + 1) for _ in range(num_segments + 1)]
      
        # Base case: 0 segments with 0 skips takes 0 time
        dp[0][0] = 0
      
        # Small epsilon to handle floating point precision issues
        epsilon = 1e-8
      
        # Process each road segment
        for segment_idx, distance in enumerate(dist, 1):
            # Try different numbers of skips (from 0 to segment_idx)
            for num_skips in range(segment_idx + 1):
              
                # Option 1: Don't skip at this segment (rest after this segment)
                # Can only do this if we haven't used all available segments for skips
                if num_skips < segment_idx:
                    # Add travel time and round up (rest until next hour)
                    # Subtract epsilon before ceiling to handle floating point errors
                    time_with_rest = dp[segment_idx - 1][num_skips] + distance / speed
                    dp[segment_idx][num_skips] = min(
                        dp[segment_idx][num_skips], 
                        ceil(time_with_rest - epsilon)
                    )
              
                # Option 2: Skip at this segment (don't rest after this segment)
                # Can only do this if we have at least one skip to use
                if num_skips > 0:
                    # Add travel time without rounding (no rest)
                    time_with_skip = dp[segment_idx - 1][num_skips - 1] + distance / speed
                    dp[segment_idx][num_skips] = min(
                        dp[segment_idx][num_skips], 
                        time_with_skip
                    )
      
        # Find minimum number of skips that allows arrival within time limit
        for skips_used in range(num_segments + 1):
            # Check if total time is within the allowed limit (with epsilon tolerance)
            if dp[num_segments][skips_used] <= hoursBefore + epsilon:
                return skips_used
      
        # If no valid solution exists, return -1
        return -1
```

## Time and Space Complexity

The time complexity of this algorithm is `O(n²)`, where `n` is the length of the `dist` array (number of roads).

This is because:
- The outer loop iterates through each road position from `1` to `n`, giving us `n` iterations
- For each road position `i`, the inner loop iterates from `0` to `i`, which in the worst case is also up to `n` iterations
- Inside the nested loops, all operations (comparisons, arithmetic operations, `min()`, `ceil()`) are `O(1)`
- Therefore, the overall time complexity is `O(n) × O(n) × O(1) = O(n²)`

The space complexity is `O(n²)`, where `n` is the length of the `dist` array.

This is because:
- We create a 2D array `f` with dimensions `(n+1) × (n+1)` to store the dynamic programming states
- Each cell stores a single floating-point value
- The total space required is `(n+1) × (n+1) = O(n²)`
- Other variables like `eps`, `i`, `j`, and `x` use constant space `O(1)`
- Therefore, the overall space complexity is dominated by the 2D array, giving us `O(n²)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Floating-Point Precision Errors**
The most critical pitfall in this problem is handling floating-point arithmetic incorrectly. When dividing distances by speed, you get floating-point numbers that may have tiny precision errors.

**Problem Example:**
```python
# Without epsilon adjustment
time = 1.9999999999999998  # Due to floating-point arithmetic
ceil(time) = 2  # Correct

time = 2.0000000000000004  # Due to floating-point arithmetic  
ceil(time) = 3  # WRONG! Should be 2
```

**Solution:**
Always subtract a small epsilon (like `1e-8`) before applying the ceiling function:
```python
ceil(time - epsilon)  # Handles floating-point errors correctly
```

### 2. **Incorrect DP Boundaries**
A common mistake is setting incorrect loop boundaries or array dimensions.

**Wrong Implementation:**
```python
# WRONG: Can't have more skips than roads minus 1
for num_skips in range(num_segments + 2):  # Too many possible skips
    ...
```

**Correct Implementation:**
```python
# Maximum skips = n-1 (can't skip rest after the last road)
for num_skips in range(segment_idx + 1):  # Bounded by current segment
    ...
```

### 3. **Forgetting the Last Road Special Case**
The problem states that the last road doesn't require rest. Some implementations might incorrectly apply ceiling to the last road.

**Wrong Approach:**
```python
# WRONG: Applying ceiling even for the last road
for i in range(1, n + 1):
    time = prev_time + dist[i-1] / speed
    dp[i][j] = ceil(time)  # Always ceiling
```

**Correct Approach:**
The DP naturally handles this because:
- We only apply ceiling when NOT skipping (adding rest)
- The final answer checks `dp[n][j]` directly without additional ceiling
- The last segment's time is included but no rest is added after it

### 4. **Off-by-One Errors in Indexing**
Mixing 0-based and 1-based indexing can lead to errors.

**Common Mistake:**
```python
# Confusing dist[i] vs dist[i-1]
dp[i][j] = dp[i-1][j] + dist[i] / speed  # WRONG if i is 1-based
```

**Correct Pattern:**
```python
for segment_idx, distance in enumerate(dist, 1):  # segment_idx is 1-based
    # Use 'distance' directly instead of dist[segment_idx-1]
    time = dp[segment_idx - 1][num_skips] + distance / speed
```

### 5. **Not Handling the Impossible Case**
Forgetting to return `-1` when it's impossible to arrive on time.

**Incomplete Solution:**
```python
# WRONG: Assumes a solution always exists
return min(j for j in range(n+1) if dp[n][j] <= hoursBefore)
```

**Complete Solution:**
```python
for skips_used in range(num_segments + 1):
    if dp[num_segments][skips_used] <= hoursBefore + epsilon:
        return skips_used
return -1  # Important: Handle impossible case
```

