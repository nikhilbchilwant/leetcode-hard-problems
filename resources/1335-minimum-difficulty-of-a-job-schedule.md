# 1335. Minimum Difficulty of a Job Schedule

> Source: https://algo.monster/liteproblems/1335

## Problem Statement

You have a list of jobs that must be completed over `d` days. The jobs have a dependency constraint - to work on the `i-th` job, you must first complete all jobs from index `0` to `i-1`. This means jobs must be done in order.

Each job has a difficulty value given in the array `jobDifficulty`, where `jobDifficulty[i]` represents the difficulty of the `i-th` job.

The scheduling rules are:
- You must complete at least one job every day
- Jobs must be done in order (can't skip ahead)
- The difficulty of a single day equals the maximum difficulty among all jobs completed that day
- The total difficulty of the schedule equals the sum of difficulties across all `d` days

Your goal is to partition the jobs across `d` days such that the total difficulty is minimized.

For example, if you have jobs with difficulties `[6, 5, 4, 3, 2, 1]` and `d = 2` days:
- One possible schedule: Day 1 do jobs `[6, 5, 4, 3]` (difficulty = 6), Day 2 do jobs `[2, 1]` (difficulty = 2). Total = 6 + 2 = 8.
- Another schedule: Day 1 do job `[6]` (difficulty = 6), Day 2 do jobs `[5, 4, 3, 2, 1]` (difficulty = 5). Total = 6 + 5 = 11.

Return the minimum possible total difficulty. If it's impossible to schedule all jobs in `d` days (when there are fewer jobs than days), return `-1`.

## Intuition

Since jobs must be completed in order and we need to assign them to exactly `d` days, this problem is about finding optimal partition points. We need to decide where to "cut" the job sequence to separate work for different days.

The key insight is that this has optimal substructure - if we know the minimum difficulty to complete the first `k-1` jobs in `j-1` days, we can extend this to find the minimum difficulty for completing `i` jobs in `j` days by trying all possible ways to schedule jobs `[k..i]` on day `j`.

Think of it this way: On the last day (day `j`), we must complete some consecutive sequence of jobs ending at job `i`. We could choose to do just job `i`, or jobs `[i-1, i]`, or jobs `[i-2, i-1, i]`, and so on. For each choice, the difficulty of day `j` is the maximum difficulty among those jobs.

This naturally leads to [dynamic programming](/problems/dynamic_programming_intro) where `f[i][j]` represents the minimum total difficulty to complete the first `i` jobs using exactly `j` days. 

For each state `f[i][j]`, we try all possible "starting points" `k` for the last day's work. If we do jobs `[k..i]` on day `j`, then:
- We need `f[k-1][j-1]` (minimum difficulty to complete first `k-1` jobs in `j-1` days)
- Plus the difficulty of day `j`, which is `max(jobDifficulty[k-1], ..., jobDifficulty[i-1])`

By trying all valid values of `k` and taking the minimum, we find the optimal way to schedule the first `i` jobs in `j` days.

The base case is `f[0][0] = 0` (no jobs, no days, zero difficulty), and we build up the solution incrementally until we reach `f[n][d]`, which gives us the answer for all `n` jobs in `d` days.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

We implement the [dynamic programming](/problems/dynamic_programming_intro) solution using a 2D array `f` where `f[i][j]` represents the minimum difficulty to complete the first `i` jobs in exactly `j` days.

**Initialization:**
- Create a 2D array `f` of size `(n+1) × (d+1)` initialized with infinity values
- Set `f[0][0] = 0` as the base case (0 jobs in 0 days has 0 difficulty)

**State Transition:**
For each state `f[i][j]`, we iterate through all possible ways to partition the jobs:
- `i` represents the number of jobs we're considering (from 1 to n)
- `j` represents the number of days we're using (from 1 to min(d, i))
- `k` represents the starting job index for the last day (we iterate k from i down to 1)

For each combination of `(i, j, k)`:
1. We calculate the maximum difficulty `mx` for jobs `[k..i]` which would be done on day `j`
2. We update `f[i][j]` with the minimum of its current value and `f[k-1][j-1] + mx`

The state transition equation is:
```
f[i][j] = min(f[i][j], f[k-1][j-1] + max(jobDifficulty[k-1], ..., jobDifficulty[i-1]))
```

**Implementation Details:**
- The outer loop iterates through jobs: `for i in range(1, n + 1)`
- The middle loop iterates through days: `for j in range(1, min(d + 1, i + 1))` 
  - We use `min(d + 1, i + 1)` because we can't have more days than jobs
- The inner loop tries all partition points: `for k in range(i, 0, -1)`
  - We iterate backwards from `i` to `1` to efficiently track the maximum difficulty
  - As we iterate, we maintain `mx = max(mx, jobDifficulty[k-1])` to get the maximum difficulty for jobs `[k..i]`

**Final Answer:**
- If `f[n][d] >= inf`, it means it's impossible to schedule all jobs in `d` days, so return `-1`
- Otherwise, return `f[n][d]` as the minimum difficulty

The time complexity is `O(n²d)` where `n` is the number of jobs, and the space complexity is `O(nd)` for the DP table.

## Example Walkthrough

Let's walk through a small example with `jobDifficulty = [6, 5, 4, 3]` and `d = 2` days.

We'll build a DP table `f[i][j]` where `i` is the number of jobs completed and `j` is the number of days used.

**Initialization:**
- Create a 5×3 table (since we have 4 jobs and 2 days)
- Initialize all values to infinity except `f[0][0] = 0`

Initial table:
```
    j=0   j=1   j=2
i=0  0    inf   inf
i=1  inf  inf   inf
i=2  inf  inf   inf
i=3  inf  inf   inf
i=4  inf  inf   inf
```

**Building the DP table:**

**For i=1 (first job):**
- j=1, k=1: Do job 1 on day 1
  - `mx = jobDifficulty[0] = 6`
  - `f[1][1] = f[0][0] + 6 = 0 + 6 = 6`

**For i=2 (first two jobs):**
- j=1, k=2: Do jobs 1-2 on day 1
  - `mx = max(6, 5) = 6`
  - `f[2][1] = f[0][0] + 6 = 6`
- j=1, k=1: Do job 1 on day 1 (invalid - need f[0][0] which requires 0 days for 0 jobs)
  - Skip as we'd need to complete job 2 somewhere
- j=2, k=2: Do job 2 on day 2, job 1 on day 1
  - `mx = jobDifficulty[1] = 5`
  - `f[2][2] = f[1][1] + 5 = 6 + 5 = 11`
- j=2, k=1: Do jobs 1-2 on day 2
  - `mx = max(6, 5) = 6`
  - `f[2][2] = min(11, f[0][1] + 6) = 11` (f[0][1] is inf)

**For i=3 (first three jobs):**
- j=1, k=3,2,1: Do all jobs on day 1
  - `mx = max(6, 5, 4) = 6`
  - `f[3][1] = 6`
- j=2, k=3: Do job 3 on day 2, jobs 1-2 on day 1
  - `mx = 4`
  - `f[3][2] = f[2][1] + 4 = 6 + 4 = 10`
- j=2, k=2: Do jobs 2-3 on day 2, job 1 on day 1
  - `mx = max(5, 4) = 5`
  - `f[3][2] = min(10, f[1][1] + 5) = min(10, 6 + 5) = 10`
- j=2, k=1: Do jobs 1-3 on day 2
  - `mx = max(6, 5, 4) = 6`
  - `f[3][2] = min(10, f[0][1] + 6) = 10` (f[0][1] is inf)

**For i=4 (all four jobs):**
- j=2, k=4: Do job 4 on day 2, jobs 1-3 on day 1
  - `mx = 3`
  - `f[4][2] = f[3][1] + 3 = 6 + 3 = 9`
- j=2, k=3: Do jobs 3-4 on day 2, jobs 1-2 on day 1
  - `mx = max(4, 3) = 4`
  - `f[4][2] = min(9, f[2][1] + 4) = min(9, 6 + 4) = 9`
- j=2, k=2: Do jobs 2-4 on day 2, job 1 on day 1
  - `mx = max(5, 4, 3) = 5`
  - `f[4][2] = min(9, f[1][1] + 5) = min(9, 6 + 5) = 9`
- j=2, k=1: Do all jobs on day 2 (invalid - need at least one job on day 1)
  - Skip

Final table:
```
    j=0   j=1   j=2
i=0  0    inf   inf
i=1  inf   6    inf
i=2  inf   6    11
i=3  inf   6    10
i=4  inf  inf    9
```

**Answer:** `f[4][2] = 9`

This corresponds to the optimal schedule:
- Day 1: Jobs [6, 5, 4] with difficulty = 6
- Day 2: Job [3] with difficulty = 3
- Total difficulty = 6 + 3 = 9

## Python Solution

```python
class Solution:
    def minDifficulty(self, jobDifficulty: List[int], d: int) -> int:
        """
        Find the minimum difficulty of a job schedule.
        Jobs must be done in order and distributed across d days.
        Each day's difficulty is the maximum difficulty of jobs done that day.
      
        Args:
            jobDifficulty: List of job difficulties
            d: Number of days to complete all jobs
      
        Returns:
            Minimum total difficulty, or -1 if impossible
        """
        n = len(jobDifficulty)
      
        # Edge case: cannot distribute n jobs across d days if n < d
        if n < d:
            return -1
      
        # dp[i][j] represents minimum difficulty to complete first i jobs in j days
        # Initialize with infinity (impossible state)
        dp = [[float('inf')] * (d + 1) for _ in range(n + 1)]
      
        # Base case: 0 jobs in 0 days has 0 difficulty
        dp[0][0] = 0
      
        # Fill the DP table
        for num_jobs in range(1, n + 1):
            # Can't use more days than jobs available
            for num_days in range(1, min(d + 1, num_jobs + 1)):
                max_difficulty = 0
              
                # Try different split points for the last day
                # k represents the starting job index for the last day (1-indexed)
                for k in range(num_jobs, 0, -1):
                    # Update maximum difficulty for jobs from k to num_jobs
                    max_difficulty = max(max_difficulty, jobDifficulty[k - 1])
                  
                    # Check if we can complete first (k-1) jobs in (num_days-1) days
                    if dp[k - 1][num_days - 1] != float('inf'):
                        # Update minimum difficulty for completing num_jobs in num_days
                        dp[num_jobs][num_days] = min(
                            dp[num_jobs][num_days],
                            dp[k - 1][num_days - 1] + max_difficulty
                        )
      
        # Return result: minimum difficulty to complete all n jobs in d days
        return -1 if dp[n][d] == float('inf') else dp[n][d]
```

## Time and Space Complexity

**Time Complexity:** `O(n² × d)`

The code uses three nested loops:
- The outer loop iterates through `i` from `1` to `n` (n iterations)
- The middle loop iterates through `j` from `1` to `min(d + 1, i + 1)` (at most d iterations)
- The inner loop iterates through `k` from `i` down to `1` (at most i iterations, which can be up to n)

For each combination of `(i, j)`, the inner loop performs `O(i)` operations. Since `i` can be at most `n`, and we have at most `n × d` combinations of `(i, j)`, the total time complexity is `O(n × d × n) = O(n² × d)`.

**Space Complexity:** `O(n × d)`

The code creates a 2D array `f` with dimensions `(n + 1) × (d + 1)`. This requires `O((n + 1) × (d + 1)) = O(n × d)` space. The only other variables used are loop counters and the variable `mx`, which require `O(1)` additional space. Therefore, the overall space complexity is `O(n × d)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Loop Bounds for Split Points**

A common mistake is using incorrect bounds when iterating through split points for the last day. Many implementations incorrectly start from `k = num_jobs - 1` or use wrong termination conditions.

**Incorrect Implementation:**
```python
# Wrong: Starting from num_jobs - 1
for k in range(num_jobs - 1, 0, -1):
    max_difficulty = max(max_difficulty, jobDifficulty[k])
    # This skips considering all num_jobs on the last day
```

**Correct Implementation:**
```python
# Correct: Starting from num_jobs to consider all possible splits
for k in range(num_jobs, 0, -1):
    max_difficulty = max(max_difficulty, jobDifficulty[k - 1])
```

### 2. **Off-by-One Errors in Index Mapping**

The DP table uses 1-based indexing for conceptual clarity (dp[i][j] means first i jobs), but the jobDifficulty array uses 0-based indexing. This frequently leads to confusion.

**Incorrect Implementation:**
```python
# Wrong: Using k directly as array index
for k in range(num_jobs, 0, -1):
    max_difficulty = max(max_difficulty, jobDifficulty[k])  # IndexError!
```

**Correct Implementation:**
```python
# Correct: Adjusting for 0-based array indexing
for k in range(num_jobs, 0, -1):
    max_difficulty = max(max_difficulty, jobDifficulty[k - 1])
```

### 3. **Inefficient Maximum Calculation**

Recalculating the maximum difficulty from scratch for each split point leads to unnecessary time complexity.

**Inefficient Implementation:**
```python
for k in range(num_jobs, 0, -1):
    # Recalculating max from scratch each time - O(n) operation
    max_difficulty = max(jobDifficulty[k-1:num_jobs])
    dp[num_jobs][num_days] = min(
        dp[num_jobs][num_days],
        dp[k - 1][num_days - 1] + max_difficulty
    )
```

**Efficient Implementation:**
```python
max_difficulty = 0
for k in range(num_jobs, 0, -1):
    # Incrementally update max - O(1) operation
    max_difficulty = max(max_difficulty, jobDifficulty[k - 1])
    dp[num_jobs][num_days] = min(
        dp[num_jobs][num_days],
        dp[k - 1][num_days - 1] + max_difficulty
    )
```

### 4. **Forgetting to Check Valid Previous States**

Not verifying whether the previous state is reachable before using it in the transition can lead to incorrect results.

**Incorrect Implementation:**
```python
# Wrong: Not checking if previous state is valid
dp[num_jobs][num_days] = min(
    dp[num_jobs][num_days],
    dp[k - 1][num_days - 1] + max_difficulty
)
```

**Correct Implementation:**
```python
# Correct: Only use valid previous states
if dp[k - 1][num_days - 1] != float('inf'):
    dp[num_jobs][num_days] = min(
        dp[num_jobs][num_days],
        dp[k - 1][num_days - 1] + max_difficulty
    )
```

### 5. **Incorrect Day Limit in Loop**

Using `range(1, d + 1)` for all job counts wastes computation and can cause logical errors when there are fewer jobs than days.

**Incorrect Implementation:**
```python
# Wrong: Always iterating up to d days regardless of job count
for num_days in range(1, d + 1):
    # This allows invalid states like 2 jobs in 5 days
```

**Correct Implementation:**
```python
# Correct: Limiting days to available jobs
for num_days in range(1, min(d + 1, num_jobs + 1)):
    # Ensures we never have more days than jobs
```

