# 1235. Maximum Profit in Job Scheduling

> Source: https://algo.monster/liteproblems/1235

## Problem Statement

You are given `n` jobs where each job has:
- A start time (`startTime[i]`)
- An end time (`endTime[i]`)
- A profit value (`profit[i]`)

Your task is to select a subset of jobs that maximizes the total profit, with the constraint that no two selected jobs can have overlapping time ranges.

**Important note about overlapping:** Two jobs are considered non-overlapping if one job ends exactly when another job starts. In other words, if a job ends at time `X`, you can select another job that starts at time `X` - they won't be considered overlapping.

**Example:** If you have:
- Job A: starts at 1, ends at 3, profit 50
- Job B: starts at 3, ends at 5, profit 40
- Job C: starts at 2, ends at 4, profit 10

You could select jobs A and B (total profit = 90) since A ends at 3 and B starts at 3 (non-overlapping). However, you couldn't select both A and C since they overlap (A runs from 1-3, C runs from 2-4).

The goal is to find the maximum total profit possible by selecting a valid subset of non-overlapping jobs.

## Intuition

The key insight is that this is a classic [dynamic programming](/problems/dynamic_programming_intro) problem where we need to make optimal decisions at each step. For each job, we face a choice: either take it or skip it.

Let's think about this systematically. If we sort the jobs by their start time, we can process them in chronological order. For any job at position `i`, we have two options:

1. **Skip the job**: If we don't take job `i`, our maximum profit is whatever we can get from the remaining jobs starting at position `i+1`.

2. **Take the job**: If we take job `i`, we get its profit, but then we can only consider jobs that start after this job ends. We need to find the next valid job that doesn't overlap.

This naturally leads to a recursive structure: `maximum_profit(i) = max(skip_job, take_job)`, where:
- `skip_job = maximum_profit(i+1)`
- `take_job = profit[i] + maximum_profit(next_valid_job)`

The challenge is efficiently finding the "next valid job" after taking job `i`. Since we've sorted jobs by start time, we can use [binary search](/problems/binary_search_intro) to quickly find the first job whose start time is greater than or equal to the current job's end time. This gives us `O(log n)` time for finding the next valid position instead of `O(n)` with linear search.

We notice that the same subproblems (starting from position `i`) will be computed multiple times in different branches of our recursion tree. This overlap suggests using memoization to cache results and avoid redundant calculations.

The base case is straightforward: when we've processed all jobs (i >= n), the profit is 0.

By combining these ideas - [sorting](/problems/sorting_summary), [binary search](/problems/binary_search_intro) for efficiency, and memoization for optimization - we arrive at an elegant recursive solution that explores all valid combinations while avoiding redundant work.

**Learn more about [Binary Search](/problems/binary_search_intro), [Dynamic Programming](/problems/dynamic_programming_intro) and [Sorting](/problems/sorting_summary) patterns.**

## Solution Approach

The implementation follows the memoization search with [binary search](/problems/binary_search_intro) pattern we identified in our intuition.

**Step 1: Data Preparation**
First, we combine the three arrays into a single list of tuples and sort by start time:
```python
jobs = sorted(zip(startTime, endTime, profit))
```
This creates tuples of `(start, end, profit)` sorted by start time, allowing us to process jobs chronologically.

**Step 2: Recursive Function with Memoization**
We define a recursive function `dfs(i)` that returns the maximum profit starting from job index `i`:
```python
@cache
def dfs(i):
    if i >= n:
        return 0
```
The `@cache` decorator automatically memoizes results, preventing redundant calculations for the same `i`.

**Step 3: Processing Each Job**
For job `i`, we extract its details:
```python
_, e, p = jobs[i]  # start (unused), end, profit
```

**Step 4: Finding Next Valid Job**
We use [binary search](/problems/binary_search_intro) to find the first job that starts at or after the current job's end time:
```python
j = bisect_left(jobs, e, lo=i + 1, key=lambda x: x[0])
```
- `bisect_left` finds the leftmost position where we could insert `e`
- `lo=i + 1` ensures we only look at jobs after the current one
- `key=lambda x: x[0]` compares based on start times (first element of tuple)

**Step 5: Making the Optimal Choice**
We apply our recurrence relation:
```python
return max(dfs(i + 1), p + dfs(j))
```
This compares:
- Skipping job `i`: `dfs(i + 1)`
- Taking job `i`: `p + dfs(j)` (current profit plus maximum from next valid job)

The formula can be expressed as:
$$dfs(i) = \max(dfs(i+1), profit[i] + dfs(j))$$

where `j` is the smallest index satisfying `startTime[j] ≥ endTime[i]`.

**Time Complexity**: `O(n log n)` where `n` is the number of jobs
- [Sorting](/problems/sorting_summary): `O(n log n)`
- Each of `n` states is computed once (memoization)
- Each state does `O(log n)` [binary search](/problems/binary_search_intro)
- Total: `O(n log n)`

**Space Complexity**: `O(n)` for the memoization cache and recursion stack

The solution elegantly combines [sorting](/problems/sorting_summary), [binary search](/problems/binary_search_intro), and [dynamic programming](/problems/dynamic_programming_intro) through memoization to efficiently solve what could otherwise be an exponential-time problem.

## Example Walkthrough

Let's walk through a small example with 3 jobs to illustrate the solution approach:

**Input:**
- Job 0: start=1, end=3, profit=50
- Job 1: start=2, end=5, profit=20
- Job 2: start=3, end=6, profit=70

**Step 1: Sort jobs by start time**
```
jobs = [(1,3,50), (2,5,20), (3,6,70)]
```
Already sorted in this case.

**Step 2: Apply recursive function starting from index 0**

`dfs(0)`: Processing job (1,3,50)
- Option 1 - Skip: `dfs(1)`
- Option 2 - Take: profit=50 + `dfs(j)` where j is next valid job
  - Binary search for jobs starting at time ≥ 3 (when job 0 ends)
  - Found j=2 (job starting at time 3)
  - Take option: 50 + `dfs(2)`

`dfs(1)`: Processing job (2,5,20)
- Option 1 - Skip: `dfs(2)`
- Option 2 - Take: profit=20 + `dfs(j)` where j is next valid job
  - Binary search for jobs starting at time ≥ 5
  - No jobs found (j=3, which is ≥ n)
  - Take option: 20 + `dfs(3)` = 20 + 0 = 20

`dfs(2)`: Processing job (3,6,70)
- Option 1 - Skip: `dfs(3)` = 0 (base case)
- Option 2 - Take: profit=70 + `dfs(j)`
  - Binary search for jobs starting at time ≥ 6
  - No jobs found
  - Take option: 70 + 0 = 70
- Return max(0, 70) = 70

**Backtracking:**
- `dfs(1)` = max(`dfs(2)`, 20 + `dfs(3)`) = max(70, 20) = 70
- `dfs(0)` = max(`dfs(1)`, 50 + `dfs(2)`) = max(70, 50 + 70) = 120

**Result:** Maximum profit = 120 (by taking jobs 0 and 2)

The key insight is that job 0 ends at time 3, and job 2 starts at time 3, so they don't overlap and can both be selected. The binary search efficiently finds that job 2 is the next valid job after job 0.

## Python Solution

```python
class Solution:
    def jobScheduling(
        self, startTime: List[int], endTime: List[int], profit: List[int]
    ) -> int:
        """
        Find the maximum profit from non-overlapping jobs.

        Args:
            startTime: List of job start times
            endTime: List of job end times
            profit: List of job profits

        Returns:
            Maximum profit achievable by scheduling non-overlapping jobs
        """
        from functools import cache
        from bisect import bisect_left
        from typing import List

        # Sort jobs by start time for efficient searching
        jobs = sorted(zip(startTime, endTime, profit))
        n = len(jobs)

        @cache
        def dfs(index: int) -> int:
            """
            Dynamic programming function to find maximum profit from index onwards.

            Args:
                index: Current job index to consider

            Returns:
                Maximum profit achievable from current index to end
            """
            # Base case: no more jobs to process
            if index >= n:
                return 0

            # Extract current job details
            current_start, current_end, current_profit = jobs[index]

            # Find the next job that starts after current job ends
            # Using binary search for efficiency
            next_valid_job_index = bisect_left(
                jobs,
                current_end,
                lo=index + 1,
                key=lambda job: job[0]  # Compare by start time
            )

            # Two choices: skip current job or take current job
            skip_current = dfs(index + 1)
            take_current = current_profit + dfs(next_valid_job_index)

            # Return the maximum profit between the two choices
            return max(skip_current, take_current)

        # Start the recursion from the first job
        return dfs(0)
```

## Time and Space Complexity

**Time Complexity:** `O(n × log n)`, where `n` is the number of jobs.

The time complexity breaks down as follows:
- Sorting the jobs takes `O(n × log n)` time
- The recursive function `dfs` is called at most `n` times due to memoization with `@cache` (each index from 0 to n-1 is visited at most once)
- Inside each `dfs` call, `bisect_left` performs a binary search which takes `O(log n)` time
- Therefore, the total time for all `dfs` calls is `O(n × log n)`
- Overall: `O(n × log n) + O(n × log n) = O(n × log n)`

**Space Complexity:** `O(n)`

The space complexity consists of:
- `O(n)` for storing the sorted `jobs` list
- `O(n)` for the memoization cache that stores results for up to `n` different indices
- `O(n)` for the recursion call stack in the worst case (though typically much less due to the binary search jumps)
- Overall: `O(n)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Overlap Definition**
A frequent mistake is misunderstanding when jobs are considered non-overlapping. Many assume jobs cannot touch at all, but the problem explicitly allows a job to start exactly when another ends.

**Incorrect assumption:**
```python
# Wrong: Treating jobs as overlapping when one ends where another starts
next_job = bisect_right(jobs, current_end, lo=index + 1, key=lambda x: x[0])
```

**Correct approach:**
```python
# Right: Jobs can start exactly when another ends (non-overlapping)
next_job = bisect_left(jobs, current_end, lo=index + 1, key=lambda x: x[0])
```

### 2. **Forgetting to Sort the Jobs**
The binary search optimization only works if jobs are sorted by start time. Processing jobs in their original order breaks the algorithm.

**Pitfall:**
```python
# Wrong: Using unsorted jobs
jobs = list(zip(startTime, endTime, profit))
```

**Solution:**
```python
# Correct: Sort by start time first
jobs = sorted(zip(startTime, endTime, profit))
```

### 3. **Incorrect Binary Search Bounds**
Using `lo=i` instead of `lo=i+1` in binary search can cause infinite recursion since the function might repeatedly call itself with the same index.

**Pitfall:**
```python
# Wrong: Can select the same job again
next_job = bisect_left(jobs, current_end, lo=i, key=lambda x: x[0])
```

**Solution:**
```python
# Correct: Search only among jobs after current one
next_job = bisect_left(jobs, current_end, lo=i + 1, key=lambda x: x[0])
```

### 4. **Memory Limit Issues with Large Inputs**
While `@cache` is convenient, it can cause memory issues with very large inputs. If encountering memory problems, consider using a bottom-up approach or manual memoization with cleanup.

**Alternative for memory-constrained scenarios:**
```python
def jobScheduling(self, startTime, endTime, profit):
    jobs = sorted(zip(startTime, endTime, profit))
    n = len(jobs)
    dp = [0] * (n + 1)

    for i in range(n - 1, -1, -1):
        _, end, prof = jobs[i]
        next_job = bisect_left(jobs, end, lo=i + 1, key=lambda x: x[0])
        dp[i] = max(dp[i + 1], prof + dp[next_job])

    return dp[0]
```

### 5. **Using Wrong Comparison Key in Binary Search**
The key function must extract the start time for comparison, not the entire tuple or wrong field.

**Pitfall:**
```python
# Wrong: Comparing entire tuples or wrong field
next_job = bisect_left(jobs, current_end, lo=i + 1)  # No key specified
# Or
next_job = bisect_left(jobs, current_end, lo=i + 1, key=lambda x: x[1])  # Using end time
```

**Solution:**
```python
# Correct: Extract start time for comparison
next_job = bisect_left(jobs, current_end, lo=i + 1, key=lambda x: x[0])
```

