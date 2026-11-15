# 2050. Parallel Courses III

> Source: https://algo.monster/liteproblems/2050

## Problem Statement

You have `n` courses numbered from `1` to `n`. Some courses have prerequisite relationships, meaning certain courses must be completed before others can be started. These relationships are given as a 2D array `relations`, where `relations[j] = [prevCoursej, nextCoursej]` indicates that `prevCoursej` must be completed before `nextCoursej`.

Each course takes a certain number of months to complete, specified in the array `time`, where `time[i]` represents the number of months needed to complete course `(i+1)`.

The key rules for taking courses are:
- You can start any course at any time as long as all its prerequisites are completed
- Multiple courses can be taken simultaneously (in parallel)

Your task is to find the minimum number of months needed to complete all `n` courses.

For example, if Course A takes 3 months and is a prerequisite for Course B which takes 2 months, and Course B is a prerequisite for Course C which takes 1 month, then:
- Course A must be completed first (3 months)
- Course B can start after A is done and takes 2 more months (total: 5 months)
- Course C can start after B is done and takes 1 more month (total: 6 months)

If there are independent courses with no prerequisites, they can all be started at month 0 and run in parallel. The total time would be determined by the longest path through the prerequisite chain.

The problem guarantees that the course dependencies form a directed acyclic [graph](/problems/graph_intro) (DAG), meaning there are no circular dependencies and it's always possible to complete all courses.

## Intuition

The key insight is that this problem is about finding the longest path in a directed acyclic [graph](/problems/graph_intro) (DAG) where each node has a weight (the time to complete that course).

Think about it this way: if we have a chain of prerequisites like A → B → C, we can't start B until A is finished, and we can't start C until B is finished. The total time for this chain would be `time[A] + time[B] + time[C]`. However, if we have parallel chains like A → B and C → D, both chains can execute simultaneously, so the total time would be `max(time[A] + time[B], time[C] + time[D])`.

This naturally leads us to think about topological sorting, which processes nodes in a DAG in dependency order. We can process courses level by level: first complete all courses with no prerequisites, then courses that only depend on those, and so on.

The [dynamic programming](/problems/dynamic_programming_intro) aspect comes from tracking the earliest completion time for each course. For any course `j`, its earliest completion time is determined by:
- The maximum completion time among all its prerequisites
- Plus its own duration

Mathematically: `f[j] = max(f[prerequisite] for all prerequisites) + time[j]`

We use BFS-based topological sorting because it naturally processes courses in levels. Starting with courses that have no prerequisites (in-degree = 0), we:
1. Calculate their completion times
2. Remove them from the [graph](/problems/graph_intro) (reduce in-degrees of dependent courses)
3. Add newly freed courses (those with in-degree now 0) to our queue
4. Repeat until all courses are processed

The answer is the maximum completion time among all courses, as this represents when the last course finishes and all courses are complete.

**Learn more about [Graph](/problems/graph_intro), [Topological Sort](/problems/topo_intro) and [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The solution implements topological sorting with [dynamic programming](/problems/dynamic_programming_intro) using BFS. Here's the step-by-step implementation:

**1. Build the [Graph](/problems/graph_intro) and Track In-degrees**
```python
g = defaultdict(list)  # Adjacency list for the graph
indeg = [0] * n       # In-degree count for each course
```
- Convert 1-indexed courses to 0-indexed for easier array manipulation
- For each relation `[a, b]`, add edge `g[a-1].append(b-1)` and increment `indeg[b-1]`

**2. Initialize Data Structures**
```python
q = deque()     # Queue for BFS
f = [0] * n     # f[i] = earliest completion time for course i
ans = 0         # Track the maximum completion time
```

**3. Find Starting Courses (No Prerequisites)**
```python
for i, (v, t) in enumerate(zip(indeg, time)):
    if v == 0:  # No prerequisites
        q.append(i)
        f[i] = t  # Completion time = course duration
        ans = max(ans, t)
```
Courses with in-degree 0 can start immediately at time 0, so their completion time equals their duration.

**4. Process Courses Level by Level (BFS)**
```python
while q:
    i = q.popleft()  # Current completed course
    for j in g[i]:   # For each dependent course j
        f[j] = max(f[j], f[i] + time[j])
        ans = max(ans, f[j])
        indeg[j] -= 1
        if indeg[j] == 0:
            q.append(j)
```

For each course `i` that completes:
- Update all dependent courses `j` with their new earliest completion time: `f[j] = max(f[j], f[i] + time[j])`
- The `max` operation ensures we wait for the slowest prerequisite
- Decrement the in-degree of `j`
- If `j` has no more prerequisites (`indeg[j] == 0`), add it to the queue

**5. Return the Result**
The variable `ans` tracks the maximum completion time across all courses, which represents when the last course finishes.

**Time Complexity:** `O(V + E)` where V is the number of courses and E is the number of prerequisite relationships.

**Space Complexity:** `O(V + E)` for storing the [graph](/problems/graph_intro), in-degrees, and completion times.

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

**Example Input:**
- `n = 4` courses
- `time = [3, 2, 1, 4]` (Course 1 takes 3 months, Course 2 takes 2 months, etc.)
- `relations = [[1, 3], [2, 3], [3, 4]]` 

This creates the dependency graph:
```
Course 1 (3 months) ──┐
                      ├──> Course 3 (1 month) ──> Course 4 (4 months)
Course 2 (2 months) ──┘
```

**Step 1: Build Graph and Calculate In-degrees**
- Convert to 0-indexed: Course 1→0, Course 2→1, Course 3→2, Course 4→3
- Build adjacency list: `g = {0: [2], 1: [2], 2: [3]}`
- Calculate in-degrees: `indeg = [0, 0, 2, 1]`
  - Courses 0 and 1 have no prerequisites (in-degree = 0)
  - Course 2 has 2 prerequisites (in-degree = 2)
  - Course 3 has 1 prerequisite (in-degree = 1)

**Step 2: Initialize and Process Starting Courses**
- Queue: `[0, 1]` (courses with in-degree = 0)
- Set completion times: `f = [3, 2, 0, 0]`
- Current max: `ans = 3`

**Step 3: BFS Processing**

*Round 1 - Process Course 0 (completes at month 3):*
- Check dependent Course 2
- Update: `f[2] = max(0, 3 + 1) = 4`
- Update: `ans = max(3, 4) = 4`
- Decrement: `indeg[2] = 2 - 1 = 1`
- Course 2 still has prerequisites, don't add to queue

*Round 2 - Process Course 1 (completes at month 2):*
- Check dependent Course 2
- Update: `f[2] = max(4, 2 + 1) = 4` (no change, Course 0's path is longer)
- Decrement: `indeg[2] = 1 - 1 = 0`
- Course 2 now has no prerequisites, add to queue: `[2]`

*Round 3 - Process Course 2 (completes at month 4):*
- Check dependent Course 3
- Update: `f[3] = max(0, 4 + 4) = 8`
- Update: `ans = max(4, 8) = 8`
- Decrement: `indeg[3] = 1 - 1 = 0`
- Course 3 now has no prerequisites, add to queue: `[3]`

*Round 4 - Process Course 3 (completes at month 8):*
- No dependents, queue becomes empty

**Final Result:** `ans = 8` months

The critical path is: Course 1 (3 months) → Course 3 (1 month) → Course 4 (4 months) = 8 months total. Course 2 runs in parallel and finishes earlier, so it doesn't affect the overall completion time.

## Python Solution

```python
from collections import defaultdict, deque
from typing import List

class Solution:
    def minimumTime(self, n: int, relations: List[List[int]], time: List[int]) -> int:
        # Build adjacency list for the directed graph (prerequisite relationships)
        adjacency_list = defaultdict(list)
        in_degree = [0] * n
      
        # Process each relation: course a must be completed before course b
        for prerequisite, course in relations:
            # Convert to 0-indexed (input is 1-indexed)
            adjacency_list[prerequisite - 1].append(course - 1)
            in_degree[course - 1] += 1
      
        # Queue for topological sort (BFS approach)
        queue = deque()
        # Track the earliest completion time for each course
        completion_time = [0] * n
        max_completion_time = 0
      
        # Initialize queue with courses that have no prerequisites
        for course_id in range(n):
            if in_degree[course_id] == 0:
                queue.append(course_id)
                # Courses with no prerequisites start immediately
                completion_time[course_id] = time[course_id]
                max_completion_time = max(max_completion_time, completion_time[course_id])
      
        # Process courses in topological order
        while queue:
            current_course = queue.popleft()
          
            # Update all courses that depend on the current course
            for next_course in adjacency_list[current_course]:
                # The next course can only start after current course completes
                completion_time[next_course] = max(
                    completion_time[next_course], 
                    completion_time[current_course] + time[next_course]
                )
                max_completion_time = max(max_completion_time, completion_time[next_course])
              
                # Decrease in-degree and add to queue if all prerequisites are met
                in_degree[next_course] -= 1
                if in_degree[next_course] == 0:
                    queue.append(next_course)
      
        return max_completion_time
```

## Time and Space Complexity

**Time Complexity:** `O(m + n)`

The algorithm uses topological sorting with BFS. Breaking down the operations:
- Building the adjacency list `g` and calculating in-degrees takes `O(m)` time, where `m` is the length of the `relations` array
- The initial loop to find nodes with zero in-degree takes `O(n)` time
- The BFS traversal visits each node exactly once (`O(n)`) and processes each edge exactly once (`O(m)`)
- Overall time complexity: `O(m + n)`

**Space Complexity:** `O(m + n)`

The space usage includes:
- Adjacency list `g`: stores all edges, taking `O(m)` space
- In-degree array `indeg`: stores degree for each node, taking `O(n)` space
- Queue `q`: in worst case can contain all nodes, taking `O(n)` space
- Array `f`: stores the completion time for each node, taking `O(n)` space
- Overall space complexity: `O(m + n)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Forgetting to Handle Multiple Prerequisites with Different Completion Times**

**The Pitfall:** A common mistake is assuming that a course can start as soon as ANY prerequisite is completed, rather than waiting for ALL prerequisites to complete. This leads to incorrect calculation of the earliest start time.

**Example Scenario:**
- Course A takes 5 months
- Course B takes 3 months  
- Both A and B are prerequisites for Course C (which takes 2 months)
- Courses A and B can run in parallel starting at month 0

**Incorrect Approach:**
```python
# WRONG: Taking the first available prerequisite
for next_course in adjacency_list[current_course]:
    completion_time[next_course] = completion_time[current_course] + time[next_course]
```
This would incorrectly allow Course C to start after 3 months (when B completes) instead of waiting for 5 months (when both A and B complete).

**Correct Solution:**
```python
# CORRECT: Using max() to ensure we wait for the slowest prerequisite
completion_time[next_course] = max(
    completion_time[next_course], 
    completion_time[current_course] + time[next_course]
)
```
The `max()` operation ensures that if multiple prerequisites exist, the course waits for the last one to complete before starting.

### 2. **Not Tracking the Global Maximum Completion Time**

**The Pitfall:** Only returning the completion time of the last processed course or assuming the last course in topological order takes the longest.

**Example Scenario:**
- Course A (5 months) → Course B (1 month)
- Course C (10 months, no prerequisites)
- Processing order might be: C, A, B

**Incorrect Approach:**
```python
# WRONG: Returning the last processed course's completion time
while queue:
    current_course = queue.popleft()
    # ... process course ...
return completion_time[current_course]  # Returns 6 instead of 10
```

**Correct Solution:**
```python
# CORRECT: Track the maximum across all courses
max_completion_time = 0
# Update after processing each course
max_completion_time = max(max_completion_time, completion_time[next_course])
# Return the global maximum
return max_completion_time
```

### 3. **Incorrectly Handling 1-Indexed vs 0-Indexed Courses**

**The Pitfall:** The problem states courses are numbered from 1 to n, but arrays in most programming languages are 0-indexed. Mixing these up causes array out-of-bounds errors or incorrect course mappings.

**Incorrect Approach:**
```python
# WRONG: Using 1-indexed values directly
for prerequisite, course in relations:
    adjacency_list[prerequisite].append(course)  # Should be prerequisite-1
    in_degree[course] += 1  # Should be course-1
```

**Correct Solution:**
```python
# CORRECT: Convert to 0-indexed immediately
for prerequisite, course in relations:
    adjacency_list[prerequisite - 1].append(course - 1)
    in_degree[course - 1] += 1
```
Convert to 0-indexed values as soon as you read the input to avoid confusion throughout the rest of the algorithm.

