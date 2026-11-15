# 1723. Find Minimum Time to Finish All Jobs

> Source: https://algo.monster/liteproblems/1723

## Problem Statement

You have an array `jobs` where each element `jobs[i]` represents the time needed to complete the i-th job. You also have `k` workers available to do these jobs.

The task is to assign all jobs to workers following these rules:
- Each job must be assigned to exactly one worker
- A worker's total working time equals the sum of all jobs assigned to them
- The goal is to minimize the maximum working time among all workers

For example, if you have jobs = [3, 2, 3] and k = 2 workers:
- One possible assignment: Worker 1 gets jobs [3, 2] (total time = 5), Worker 2 gets job [3] (total time = 3). Maximum working time = 5.
- Another assignment: Worker 1 gets job [3] (total time = 3), Worker 2 gets jobs [2, 3] (total time = 5). Maximum working time = 5.
- Optimal assignment: Worker 1 gets jobs [3] (total time = 3), Worker 2 gets jobs [3] (total time = 3), and job [2] to either worker. Maximum working time = 3 or 5.

## Intuition

When we need to minimize the maximum working time, we're essentially trying to balance the workload as evenly as possible across all workers. Think of it like distributing books among shelves - we want to avoid having one shelf too full while others are nearly empty.

Since each job must go to exactly one worker and we want the optimal assignment, we need to explore different combinations. With a small number of jobs and workers, we can afford to try all possible assignments. This is where [backtracking](/problems/backtracking) comes in - it's like trying different arrangements and undoing them if they don't lead to better solutions.

The key insight is that we can optimize our search by sorting jobs in descending order. Why? When we assign larger jobs first, we can quickly identify and prune bad branches. If assigning a large job to a worker already makes their total time exceed our current best answer, we know this path won't lead to a better solution, so we can skip it immediately.

During the search, we maintain an array `cnt` tracking each worker's current workload. For each job, we try assigning it to each worker, recursively continue with the next job, then backtrack (remove the job from that worker) to try other assignments. This systematic exploration ensures we find the optimal distribution.

An important optimization: if a worker has 0 workload and adding the current job doesn't improve our answer, we can break early. This is because assigning this job to any other empty worker would yield the same result due to symmetry - empty workers are interchangeable.

The algorithm essentially answers: "What's the best way to distribute these tasks so no one is overwhelmed?" by trying all possibilities intelligently and keeping track of the best arrangement found.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro), [Backtracking](/problems/backtracking) and [Bitmask](/problems/bitmask_intro) patterns.**

## Solution Approach

The implementation uses a depth-first search (DFS) with [backtracking](/problems/backtracking) to explore all possible job assignments. Here's how the solution works:

**Data Structures:**
- `cnt`: An array of size `k` where `cnt[j]` tracks the total time assigned to worker `j`
- `ans`: Keeps track of the minimum maximum working time found so far (initialized to infinity)
- `jobs`: The input array sorted in descending order for optimization

**Algorithm Steps:**

1. **Preprocessing**: Sort the jobs array in descending order using `jobs.sort(reverse=True)`. This optimization allows us to assign larger jobs first and prune branches early.

2. **DFS Function**: The recursive function `dfs(i)` tries to assign job at index `i` to each worker:
   - **Base Case**: When `i == len(jobs)`, all jobs have been assigned. Calculate `max(cnt)` (the maximum working time among all workers) and update `ans` if this maximum is smaller.
 
   - **Recursive Case**: For the current job `jobs[i]`, try assigning it to each worker `j`:
     - **Pruning**: Skip if `cnt[j] + jobs[i] >= ans` since this assignment won't lead to a better solution
     - Assign the job: `cnt[j] += jobs[i]`
     - Recursively assign the next job: `dfs(i + 1)`
     - Backtrack: `cnt[j] -= jobs[i]` to try other assignments

3. **Symmetry Breaking**: If `cnt[j] == 0` after [backtracking](/problems/backtracking), we `break` the loop. This is because assigning the current job to any other empty worker would produce identical results (empty workers are interchangeable), so we avoid redundant explorations.

4. **Return Result**: After exploring all possibilities, return `ans` which contains the minimum possible maximum working time.

**Time Complexity**: O(k^n) in the worst case, where n is the number of jobs, as each job can be assigned to any of k workers. The pruning and optimizations significantly reduce the actual search space.

**Space Complexity**: O(n) for the recursion stack depth plus O(k) for the `cnt` array.

## Example Walkthrough

Let's walk through a small example with `jobs = [3, 2, 3]` and `k = 3` workers.

**Step 1: Preprocessing**
- Sort jobs in descending order: `jobs = [3, 3, 2]`
- Initialize `cnt = [0, 0, 0]` (tracking each worker's workload)
- Initialize `ans = infinity`

**Step 2: DFS Exploration**

Starting with `dfs(0)` - assigning job at index 0 (value = 3):

**Branch 1**: Assign `jobs[0]=3` to Worker 0
- `cnt = [3, 0, 0]`
- Call `dfs(1)` - assigning `jobs[1]=3`:
  - **Sub-branch 1.1**: Assign to Worker 0 → `cnt = [6, 0, 0]`
    - Call `dfs(2)` - assign `jobs[2]=2` to Worker 0 → `cnt = [8, 0, 0]`
    - All jobs assigned, `max(cnt) = 8`, update `ans = 8`
    - Backtrack: `cnt = [6, 0, 0]`
  - **Sub-branch 1.2**: Assign to Worker 1 → `cnt = [3, 3, 0]`
    - Call `dfs(2)` - try assigning `jobs[2]=2`:
      - Assign to Worker 0 → `cnt = [5, 3, 0]`, `max = 5`, update `ans = 5`
      - Assign to Worker 1 → `cnt = [3, 5, 0]`, `max = 5`, `ans` stays 5
      - Assign to Worker 2 → `cnt = [3, 3, 2]`, `max = 3`, update `ans = 3`
    - Backtrack: `cnt = [3, 0, 0]`
  - **Sub-branch 1.3**: Assign to Worker 2 → `cnt = [3, 0, 3]`
    - Similar exploration yields same results due to symmetry

**Pruning in Action**: 
After finding `ans = 3`, when we explore other branches:
- If we try `cnt[j] + jobs[i] >= 3`, we skip that assignment
- For example, assigning both jobs of value 3 to the same worker would give total ≥ 6, which exceeds our current best of 3, so we prune

**Symmetry Breaking**:
When backtracking from Worker 1 with `cnt[1] = 0`, we break instead of trying Worker 2, since both are empty workers and would yield identical results.

**Final Result**: The minimum maximum working time is **3**, achieved when each worker gets exactly one job.

## Python Solution

```python
from typing import List
from math import inf

class Solution:
    def minimumTimeRequired(self, jobs: List[int], k: int) -> int:
        """
        Find the minimum possible maximum working time when distributing jobs among k workers.
      
        Args:
            jobs: List of job durations
            k: Number of workers
          
        Returns:
            Minimum possible maximum working time among all workers
        """
      
        def backtrack(job_index: int) -> None:
            """
            Use backtracking to try all possible job assignments.
          
            Args:
                job_index: Current job index being assigned
            """
            nonlocal min_max_time
          
            # Base case: all jobs have been assigned
            if job_index == len(jobs):
                # Update the minimum maximum time if current assignment is better
                min_max_time = min(min_max_time, max(worker_times))
                return
          
            # Try assigning current job to each worker
            for worker_id in range(k):
                # Pruning: skip if this assignment would exceed current best answer
                if worker_times[worker_id] + jobs[job_index] >= min_max_time:
                    continue
              
                # Assign job to current worker
                worker_times[worker_id] += jobs[job_index]
              
                # Recursively assign remaining jobs
                backtrack(job_index + 1)
              
                # Backtrack: remove job from current worker
                worker_times[worker_id] -= jobs[job_index]
              
                # Optimization: if worker had no jobs before, no need to try other empty workers
                # (they would produce identical results due to symmetry)
                if worker_times[worker_id] == 0:
                    break
      
        # Initialize worker time tracking array
        worker_times = [0] * k
      
        # Sort jobs in descending order for better pruning efficiency
        # (larger jobs assigned first lead to earlier pruning)
        jobs.sort(reverse=True)
      
        # Initialize minimum maximum time to infinity
        min_max_time = inf
      
        # Start backtracking from the first job
        backtrack(0)
      
        return min_max_time
```

## Time and Space Complexity

**Time Complexity:** `O(k^n)` where `n` is the number of jobs and `k` is the number of workers.

The algorithm uses backtracking with pruning to explore all possible job assignments. In the worst case:
- For the first job, we have at most `k` choices (assigning to any of the `k` workers)
- For the second job, we again have at most `k` choices
- This continues for all `n` jobs

Therefore, the total number of states explored is bounded by `k^n`. Although the code includes several pruning strategies:
1. Early termination when `cnt[j] + jobs[i] >= ans` (branch pruning)
2. Breaking when `cnt[j] == 0` (symmetry breaking to avoid duplicate states)
3. Sorting jobs in descending order (to fail faster on infeasible branches)

These optimizations significantly reduce the practical runtime but don't change the worst-case complexity.

**Space Complexity:** `O(n + k)`

The space complexity consists of:
- `O(n)` for the recursion stack depth, as the DFS goes to a maximum depth of `n` (number of jobs)
- `O(k)` for the `cnt` array that tracks the workload of each worker
- `O(1)` for other variables like `ans` and loop counters

The total space complexity is `O(n + k)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Forgetting the Symmetry Breaking Optimization**

**Pitfall**: Without the `if worker_times[worker_id] == 0: break` optimization, the algorithm explores redundant branches when assigning a job to empty workers.

**Problem**: If multiple workers have 0 workload, assigning a job to worker 0 vs worker 1 vs worker 2 (all empty) produces identical subproblems. This leads to exponential redundant computation.

**Example**: With 3 empty workers and job value 5:
- Assigning to worker 0: `[5, 0, 0]`
- Assigning to worker 1: `[0, 5, 0]`  
- Assigning to worker 2: `[0, 0, 5]`

All three lead to the same set of future possibilities, just with different worker labels.

**Solution**: After backtracking from an empty worker, break the loop to avoid trying other empty workers:
```python
if worker_times[worker_id] == 0:
    break  # Don't try other empty workers
```

### 2. **Not Sorting Jobs in Descending Order**

**Pitfall**: Processing jobs in their original order or ascending order significantly reduces pruning effectiveness.

**Problem**: When smaller jobs are assigned first, the pruning condition `worker_times[worker_id] + jobs[job_index] >= min_max_time` is less likely to trigger early in the search, leading to more branches being explored.

**Example**: 
- Jobs = `[1, 2, 10]`, k = 2
- Without sorting: Assigning 1 and 2 first rarely triggers pruning
- With descending sort `[10, 2, 1]`: Assigning 10 first immediately limits future choices

**Solution**: Always sort jobs in descending order before starting the backtracking:
```python
jobs.sort(reverse=True)
```

### 3. **Incorrect Pruning Condition**

**Pitfall**: Using `>` instead of `>=` in the pruning condition, or forgetting the pruning entirely.

**Problem**: 
- Using `>` instead of `>=`: Allows exploration of branches that can only match (not improve) the current best, wasting computation
- No pruning: Explores all k^n possibilities without early termination

**Solution**: Use the correct pruning condition:
```python
if worker_times[worker_id] + jobs[job_index] >= min_max_time:
    continue  # Skip this assignment
```

### 4. **Not Using `nonlocal` for min_max_time**

**Pitfall**: Forgetting to declare `min_max_time` as `nonlocal` in the nested function.

**Problem**: Python will treat `min_max_time` as a local variable when you try to modify it, causing an `UnboundLocalError`.

**Solution**: Declare the variable as nonlocal at the beginning of the nested function:
```python
def backtrack(job_index: int) -> None:
    nonlocal min_max_time  # Essential for modifying outer scope variable
```

### 5. **Integer Overflow with Initial Value**

**Pitfall**: Initializing `min_max_time` with a finite but arbitrary large number like `10**9` or `sys.maxsize`.

**Problem**: If the actual answer exceeds your arbitrary maximum, the algorithm will fail to find the correct solution. While `sys.maxsize` is usually safe, using `inf` is cleaner and guaranteed to work.

**Solution**: Use `float('inf')` or `from math import inf`:
```python
min_max_time = inf  # Guaranteed to be larger than any possible answer
```

