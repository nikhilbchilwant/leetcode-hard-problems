# 1547. Minimum Cost to Cut a Stick

> Source: https://algo.monster/liteproblems/1547

## Problem Statement

You have a wooden stick of length `n` units that is labeled from `0` to `n`. You need to make cuts at specific positions given in an array `cuts`, where `cuts[i]` represents a position where you should make a cut.

The key points of the problem are:

1. **You can perform the cuts in any order** - you're not required to cut in the order given in the `cuts` array.

2. **Cost of each cut** - The cost of making a cut is equal to the length of the stick being cut at that moment. For example, if you're cutting a stick of length 10, the cost is 10.

3. **Stick splitting** - After each cut, the stick splits into two smaller pieces. The sum of their lengths equals the original stick's length.

4. **Total cost** - You need to find the minimum total cost to make all the required cuts.

For example, if you have a stick of length 7 and need to make cuts at positions [1, 3, 4, 5]:
- If you first cut at position 3, it costs 7 (the current stick length), creating pieces [0,3] and [3,7]
- Then cutting at position 1 costs 3 (length of [0,3]), creating [0,1] and [1,3]
- Cutting at position 5 costs 4 (length of [3,7]), creating [3,5] and [5,7]
- Finally cutting at position 4 costs 2 (length of [3,5]), creating [3,4] and [4,5]
- Total cost: 7 + 3 + 4 + 2 = 16

The challenge is to find the optimal order of cuts that minimizes the total cost.

## Intuition

When we look at this problem, we notice that the order of cuts matters significantly for the total cost. Each cut's cost depends on the current length of the stick being cut, which changes based on previous cuts. This suggests we need to explore different cutting orders.

The key insight is that once we decide to make a cut at position `k` in a stick segment from position `i` to position `j`, we're essentially dividing the problem into two independent subproblems: cutting the left segment `[i, k]` and cutting the right segment `[k, j]`. The total cost for this segment would be:
- The cost of making the cut at `k` (which is the length `j - i`)
- Plus the minimum cost to handle all cuts in the left segment
- Plus the minimum cost to handle all cuts in the right segment

This recursive structure immediately suggests a [dynamic programming](/problems/dynamic_programming_intro) approach. We can think of it as: for any segment of the stick, what's the minimum cost to make all required cuts within that segment?

To make this work cleanly, we add `0` and `n` to our cuts array as boundaries. This way, every segment has clear start and end points. After [sorting](/problems/sorting_summary) the cuts array, we can think of the problem as finding the minimum cost to cut between any two positions in our enhanced cuts array.

The problem becomes an interval DP problem where we build up solutions for smaller intervals and use them to solve larger intervals. For an interval `[i, j]`, we try every possible first cut `k` between `i` and `j`, and choose the one that gives us the minimum total cost. The formula becomes:

`f[i][j] = min(f[i][k] + f[k][j] + cuts[j] - cuts[i])` for all valid `k`

This approach ensures we consider all possible cutting orders and find the optimal one.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) and [Sorting](/problems/sorting_summary) patterns.**

## Solution Approach

The solution uses interval [dynamic programming](/problems/dynamic_programming_intro) to find the minimum cost. Here's how the implementation works:

**Step 1: Prepare the cuts array**
```python
cuts.extend([0, n])
cuts.sort()
```
We add the boundaries `0` and `n` to the cuts array, then sort it. This gives us all the positions where cuts can potentially divide the stick, including the endpoints. If we have `m` elements in the sorted array, we have `m-1` intervals between consecutive cut positions.

**Step 2: Initialize the DP table**
```python
f = [[0] * m for _ in range(m)]
```
We create a 2D DP table `f[i][j]` where `f[i][j]` represents the minimum cost to make all cuts between positions `cuts[i]` and `cuts[j]`. Initially, all values are 0, which correctly represents the base case where intervals with no cuts inside (intervals of length 2) have zero cost.

**Step 3: Fill the DP table by interval length**
```python
for l in range(2, m):
    for i in range(m - l):
        j = i + l
```
We iterate through intervals by their length `l` (number of cut positions in the interval). Starting from intervals of length 2 (which need no cuts) up to length `m-1` (the entire stick). For each length, we iterate through all possible starting positions `i`, and calculate the ending position as `j = i + l`.

**Step 4: Find minimum cost for each interval**
```python
f[i][j] = inf
for k in range(i + 1, j):
    f[i][j] = min(f[i][j], f[i][k] + f[k][j] + cuts[j] - cuts[i])
```
For each interval `[i, j]`, we try every possible position `k` to make the first cut (where `i < k < j`). The cost consists of:
- `cuts[j] - cuts[i]`: The cost of making the cut at position `k` (length of current stick segment)
- `f[i][k]`: The minimum cost to handle all cuts in the left sub-interval
- `f[k][j]`: The minimum cost to handle all cuts in the right sub-interval

We choose the `k` that gives the minimum total cost.

**Step 5: Return the result**
```python
return f[0][-1]
```
The answer is `f[0][m-1]`, which represents the minimum cost to make all cuts in the interval from position `cuts[0] = 0` to position `cuts[m-1] = n`, i.e., the entire stick.

The time complexity is `O(m³)` where `m` is the number of cuts plus 2, and the space complexity is `O(m²)` for the DP table.

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

**Example:** We have a stick of length `n = 7` and need to make cuts at positions `cuts = [1, 3, 4, 5]`.

**Step 1: Prepare the cuts array**
- Add boundaries: `cuts = [1, 3, 4, 5, 0, 7]`
- Sort: `cuts = [0, 1, 3, 4, 5, 7]`
- We now have `m = 6` positions

**Step 2: Initialize DP table**
Create a 6×6 table `f[i][j]` initialized with zeros. Each `f[i][j]` will store the minimum cost to make all cuts between positions `cuts[i]` and `cuts[j]`.

**Step 3: Fill DP table by interval length**

*Length 2 intervals (no cuts needed):*
- `f[0][2]` (positions 0→3): Try cut at position 1. Cost = `f[0][1] + f[1][2] + (3-0) = 0 + 0 + 3 = 3`
- `f[1][3]` (positions 1→4): Try cut at position 3. Cost = `0 + 0 + (4-1) = 3`
- `f[2][4]` (positions 3→5): Try cut at position 4. Cost = `0 + 0 + (5-3) = 2`
- `f[3][5]` (positions 4→7): Try cut at position 5. Cost = `0 + 0 + (7-4) = 3`

*Length 3 intervals:*
- `f[0][3]` (positions 0→4): 
  - Try cut at 1: `f[0][1] + f[1][3] + 4 = 0 + 3 + 4 = 7`
  - Try cut at 3: `f[0][2] + f[2][3] + 4 = 3 + 0 + 4 = 7`
  - Minimum: 7
- `f[1][4]` (positions 1→5):
  - Try cut at 3: `f[1][2] + f[2][4] + 4 = 0 + 2 + 4 = 6`
  - Try cut at 4: `f[1][3] + f[3][4] + 4 = 3 + 0 + 4 = 7`
  - Minimum: 6
- `f[2][5]` (positions 3→7):
  - Try cut at 4: `f[2][3] + f[3][5] + 4 = 0 + 3 + 4 = 7`
  - Try cut at 5: `f[2][4] + f[4][5] + 4 = 2 + 0 + 4 = 6`
  - Minimum: 6

*Length 4 intervals:*
- `f[0][4]` (positions 0→5):
  - Try cut at 1: `f[0][1] + f[1][4] + 5 = 0 + 6 + 5 = 11`
  - Try cut at 3: `f[0][2] + f[2][4] + 5 = 3 + 2 + 5 = 10`
  - Try cut at 4: `f[0][3] + f[3][4] + 5 = 7 + 0 + 5 = 12`
  - Minimum: 10
- `f[1][5]` (positions 1→7):
  - Try cut at 3: `f[1][2] + f[2][5] + 6 = 0 + 6 + 6 = 12`
  - Try cut at 4: `f[1][3] + f[3][5] + 6 = 3 + 3 + 6 = 12`
  - Try cut at 5: `f[1][4] + f[4][5] + 6 = 6 + 0 + 6 = 12`
  - Minimum: 12

*Length 5 interval (entire stick):*
- `f[0][5]` (positions 0→7):
  - Try cut at 1: `f[0][1] + f[1][5] + 7 = 0 + 12 + 7 = 19`
  - Try cut at 3: `f[0][2] + f[2][5] + 7 = 3 + 6 + 7 = 16`
  - Try cut at 4: `f[0][3] + f[3][5] + 7 = 7 + 3 + 7 = 17`
  - Try cut at 5: `f[0][4] + f[4][5] + 7 = 10 + 0 + 7 = 17`
  - Minimum: 16

**Step 4: Return result**
The answer is `f[0][5] = 16`, which represents the minimum cost to make all cuts in the stick.

This corresponds to the optimal cutting order: first cut at position 3 (cost 7), then cut at 1 (cost 3), then at 5 (cost 4), and finally at 4 (cost 2), for a total of 16.

## Python Solution

```python
class Solution:
    def minCost(self, n: int, cuts: List[int]) -> int:
        # Add boundaries (0 and n) to the cuts array
        cuts.extend([0, n])
        # Sort all cut positions for ordered processing
        cuts.sort()
      
        # Total number of positions including boundaries
        m = len(cuts)
      
        # dp[i][j] represents minimum cost to make all cuts between position i and j
        dp = [[0] * m for _ in range(m)]
      
        # Iterate over all possible lengths of segments
        # length starts from 2 (at least 3 positions: start, one cut, end)
        for length in range(2, m):
            # Try all possible starting positions for current length
            for i in range(m - length):
                # Calculate ending position based on starting position and length
                j = i + length
              
                # Initialize with infinity to find minimum
                dp[i][j] = float('inf')
              
                # Try all possible positions to make the first cut
                # k represents the position of the cut between i and j
                for k in range(i + 1, j):
                    # Cost = left segment + right segment + current cut cost
                    # Current cut cost is the length of stick being cut (cuts[j] - cuts[i])
                    dp[i][j] = min(dp[i][j], dp[i][k] + dp[k][j] + cuts[j] - cuts[i])
      
        # Return minimum cost to cut the entire stick (from position 0 to last position)
        return dp[0][-1]
```

## Time and Space Complexity

The time complexity of this code is `O(m^3)`, where `m` is the length of the modified `cuts` array after adding the boundaries `0` and `n`.

The analysis breaks down as follows:
- After extending `cuts` with `[0, n]`, the array has `m = len(cuts)` elements
- The outer loop iterates through lengths from `2` to `m-1`, giving `O(m)` iterations
- The middle loop iterates through starting positions, with up to `m - l` iterations for each length, averaging `O(m)` iterations
- The inner loop iterates through all possible split points `k` between `i+1` and `j-1`, which is `O(m)` in the worst case
- Overall: `O(m) × O(m) × O(m) = O(m^3)`

The space complexity is `O(m^2)` due to:
- The 2D DP table `f` with dimensions `m × m`, requiring `O(m^2)` space
- The sorted `cuts` array uses `O(m)` additional space, but this is dominated by the DP table

Therefore, the time complexity is `O(m^3)` and the space complexity is `O(m^2)`, where `m` is the length of the modified `cuts` array.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Forgetting to Add Boundaries to the Cuts Array**

One of the most common mistakes is forgetting to add the stick boundaries (0 and n) to the cuts array. Without these boundaries, the DP solution cannot properly calculate the cost of cuts at the edges of the stick.

**Incorrect approach:**
```python
def minCost(self, n: int, cuts: List[int]) -> int:
    cuts.sort()  # Only sorting original cuts
    m = len(cuts)
    dp = [[0] * m for _ in range(m)]
    # ... rest of the logic
```

**Why it fails:** Without boundaries, you can't calculate the actual length of stick segments. For example, if cuts = [3, 5] and n = 7, you need to know that the first segment is from 0 to 3 (length 3), not just starting at position 3.

**Solution:** Always add the boundaries before processing:
```python
cuts.extend([0, n])  # Add boundaries
cuts.sort()
```

### 2. **Incorrect Loop Boundaries When Iterating Through Intervals**

Another pitfall is using wrong loop boundaries, especially for the innermost loop that tries different cut positions.

**Incorrect approach:**
```python
for k in range(i, j):  # Wrong: includes i
    dp[i][j] = min(dp[i][j], dp[i][k] + dp[k][j] + cuts[j] - cuts[i])
```

**Why it fails:** Including `k = i` would mean making a cut at the starting position of the interval, which doesn't make sense. Similarly, including `k = j` would mean cutting at the end position.

**Solution:** The cut position k must be strictly between i and j:
```python
for k in range(i + 1, j):  # Correct: k is strictly between i and j
```

### 3. **Not Handling the Base Case Properly**

Some implementations try to explicitly set base cases for intervals with no cuts, which can lead to errors if not done correctly.

**Incorrect approach:**
```python
# Trying to explicitly set base cases
for i in range(m - 1):
    dp[i][i + 1] = 0  # This is redundant and can cause confusion
```

**Why it's problematic:** The initialization `dp = [[0] * m for _ in range(m)]` already handles the base case. Intervals of length 1 (like dp[i][i+1]) naturally have no cuts between them, so their cost is 0.

**Solution:** Trust the zero initialization for base cases. The DP table is initialized with zeros, which correctly represents that intervals with no interior cuts have zero cost.

### 4. **Confusing Interval Length with Number of Cuts**

A subtle pitfall is misunderstanding what the `length` variable represents in the outer loop.

**Incorrect understanding:**
```python
for num_cuts in range(1, m):  # Thinking this is the number of cuts
    for i in range(m - num_cuts):
        j = i + num_cuts
```

**Why it's confusing:** The variable actually represents the distance between indices in the cuts array, not the number of cuts. An interval from cuts[i] to cuts[i+2] has length 2 in the array but contains only 1 cut position (at cuts[i+1]).

**Solution:** Think of it as interval length in the sorted cuts array:
```python
for length in range(2, m):  # length is the span in the cuts array
    # An interval of length 2 means cuts[i] to cuts[i+2], with one cut at cuts[i+1]
```

