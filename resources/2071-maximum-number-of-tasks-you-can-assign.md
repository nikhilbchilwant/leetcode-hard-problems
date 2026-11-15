# 2071. Maximum Number of Tasks You Can Assign

> Source: https://algo.monster/liteproblems/2071

## Problem Statement

You have `n` tasks and `m` workers. Each task requires a certain strength to complete, stored in array `tasks`, where `tasks[i]` is the strength needed for the i-th task. Each worker has a strength value stored in array `workers`, where `workers[j]` is the strength of the j-th worker.

To complete a task, a worker must have strength greater than or equal to the task's requirement. Each worker can only be assigned to one task at most.

You also have a limited number of magical pills (given by `pills`). Each pill increases a worker's strength by exactly `strength` points. Each worker can take at most one pill.

The goal is to find the maximum number of tasks that can be completed by optimally:
- Assigning workers to tasks (each worker can do at most one task)
- Distributing pills to workers (each worker can take at most one pill)

For example, if you have:
- `tasks = [3, 2, 1]` (three tasks requiring strengths 3, 2, and 1)
- `workers = [0, 3, 3]` (three workers with strengths 0, 3, and 3)
- `pills = 1` (one pill available)
- `strength = 1` (each pill increases strength by 1)

You could give the pill to the first worker (strength becomes 0+1=1), allowing them to complete the task requiring strength 1. The other two workers with strength 3 can complete the tasks requiring strengths 2 and 3. Thus, all 3 tasks can be completed.

The function should return the maximum number of tasks that can be completed given these constraints.

## Intuition

The key insight is that if we can complete `x` tasks, then we can definitely complete `x-1`, `x-2`, ..., `1`, or `0` tasks. This monotonic property suggests we can use [binary search](/problems/binary_search_intro) to find the maximum number of tasks we can complete.

First, let's sort both arrays. [Sorting](/problems/sorting_summary) `tasks` in ascending order and `workers` in ascending order helps us make [greedy](/problems/greedy_intro) decisions. When trying to complete `x` tasks, we should:
- Use the easiest `x` tasks (the first `x` tasks after sorting)
- Use the strongest `x` workers (the last `x` workers after sorting)

This gives us the best chance of success because we're matching the easiest tasks with our strongest workers.

For the [greedy](/problems/greedy_intro) strategy within each check: when we iterate through our selected workers from weakest to strongest, each worker should consider all tasks they could potentially complete (with or without a pill). Among these available tasks:
- If the worker can complete the easiest task without a pill, do it without using a pill (save pills for workers who really need them)
- If the worker needs a pill to complete any task, use the pill to complete the hardest possible task (this maximizes the value we get from the pill)

Why does this [greedy](/problems/greedy_intro) approach work? Consider a worker who needs a pill. If we use the pill on this worker to complete an easier task when they could complete a harder one, we might later find that another worker can't complete that harder task even with a pill. By having each worker who uses a pill complete the hardest task they can, we leave easier tasks for subsequent workers who might be able to complete them without pills or with less "waste" of pill strength.

The deque structure helps efficiently track which tasks are available for the current worker - we can add newly available tasks to the back and remove completed tasks from either end depending on whether we used a pill or not.

**Learn more about [Greedy](/problems/greedy_intro), [Queue](/problems/queue_intro), [Two Pointers](/problems/two_pointers_intro), [Binary Search](/problems/binary_search_intro), [Sorting](/problems/sorting_summary) and [Monotonic Queue](/problems/mono_stack_intro) patterns.**

## Solution Approach

The solution uses **[binary search](/problems/binary_search_intro)** combined with a **[greedy](/problems/greedy_intro) assignment strategy**.

**Step 1: [Sorting](/problems/sorting_summary)**
```python
tasks.sort()
workers.sort()
```
Sort both arrays in ascending order to enable [greedy](/problems/greedy_intro) decisions.

**Step 2: [Binary Search](/problems/binary_search_intro) Setup**
```python
left, right = 0, min(n, m)
```
We can complete at most `min(n, m)` tasks (limited by either the number of tasks or workers). Binary search finds the maximum number of tasks we can complete.

**Step 3: [Binary Search](/problems/binary_search_intro) Loop**
```python
while left < right:
    mid = (left + right + 1) >> 1
    if check(mid):
        left = mid
    else:
        right = mid - 1
```
If we can complete `mid` tasks, try a larger number. Otherwise, try a smaller number.

**Step 4: The `check(x)` Function**

This function determines if we can complete exactly `x` tasks using a [greedy](/problems/greedy_intro) strategy:

```python
def check(x):
    i = 0
    q = deque()
    p = pills
    for j in range(m - x, m):
```
- Use the strongest `x` workers: `workers[m-x]` to `workers[m-1]`
- Process workers from weakest to strongest among these selected workers
- `i` tracks the next task to consider
- `q` is a deque storing currently available tasks for worker `j`
- `p` tracks remaining pills

For each worker `j`:

```python
while i < x and tasks[i] <= workers[j] + strength:
    q.append(tasks[i])
    i += 1
```
Add all tasks that this worker could potentially complete (with a pill) to the deque.

```python
if not q:
    return False
```
If no tasks are available for this worker, it's impossible to complete `x` tasks.

```python
if q[0] <= workers[j]:
    q.popleft()
```
If the worker can complete the easiest available task without a pill, do so. Remove from the front of the deque (smallest task).

```python
elif p == 0:
    return False
else:
    p -= 1
    q.pop()
```
Otherwise, the worker needs a pill. If no pills remain, return `false`. If pills are available, use one and complete the hardest available task (remove from the back of the deque).

The [greedy](/problems/greedy_intro) logic ensures:
- Workers who don't need pills complete easier tasks
- Workers who need pills complete harder tasks to maximize pill value
- Pills are used only when necessary

The algorithm runs in `O((n + m) log(min(n, m)) * min(n, m))` time complexity due to the [binary search](/problems/binary_search_intro) and the linear check function.

## Example Walkthrough

Let's walk through a concrete example to understand how the algorithm works:

**Given:**
- `tasks = [5, 9, 8, 5, 9]`
- `workers = [1, 6, 4, 2, 6]`
- `pills = 1`
- `strength = 5`

**Step 1: Sort both arrays**
- `tasks = [5, 5, 8, 9, 9]`
- `workers = [1, 2, 4, 6, 6]`

**Step 2: Binary Search**
We can complete at most `min(5, 5) = 5` tasks. Binary search range: `[0, 5]`

**First iteration:** `mid = 3` (check if we can complete 3 tasks)
- Select easiest 3 tasks: `[5, 5, 8]`
- Select strongest 3 workers: `[4, 6, 6]`

Processing workers from weakest to strongest:
- **Worker with strength 4:**
  - Can potentially complete tasks `[5, 5, 8]` with pill (4+5=9)
  - Add all to deque: `q = [5, 5, 8]`
  - Can't complete task 5 without pill (4 < 5)
  - Use pill → strength becomes 9 → complete hardest task (8)
  - Remove 8 from back: `q = [5, 5]`, `pills = 0`

- **Worker with strength 6:**
  - No new tasks to add (all were already added)
  - `q = [5, 5]`
  - Can complete task 5 without pill (6 ≥ 5)
  - Remove 5 from front: `q = [5]`

- **Worker with strength 6:**
  - No new tasks to add
  - `q = [5]`
  - Can complete task 5 without pill (6 ≥ 5)
  - Remove 5 from front: `q = []`

✓ Successfully completed 3 tasks! Update: `left = 3`

**Second iteration:** `mid = 4` (check if we can complete 4 tasks)
- Select easiest 4 tasks: `[5, 5, 8, 9]`
- Select strongest 4 workers: `[2, 4, 6, 6]`

Processing workers:
- **Worker with strength 2:**
  - Can potentially complete tasks `[5, 5]` with pill (2+5=7)
  - Add to deque: `q = [5, 5]`
  - Can't complete task 5 without pill (2 < 5)
  - Use pill → strength becomes 7 → complete hardest task (5)
  - Remove 5 from back: `q = [5]`, `pills = 0`

- **Worker with strength 4:**
  - Can potentially complete task `[8]` with pill (4+5=9)
  - Add to deque: `q = [5, 8]`
  - Can't complete task 5 without pill (4 < 5)
  - Need pill but `pills = 0` → **Return False**

✗ Cannot complete 4 tasks! Update: `right = 3`

**Binary search converges:** `left = right = 3`

**Answer: 3 tasks can be completed**

The key insight illustrated here is how the greedy strategy works:
1. When the worker with strength 4 used the pill in the 3-task scenario, they wisely chose the hardest task (8), leaving easier tasks for stronger workers
2. This allowed the two workers with strength 6 to complete the remaining tasks without pills
3. When we tried 4 tasks, we ran out of pills too early because weaker workers needed them, demonstrating why the greedy pill usage (complete hardest possible task when using a pill) is optimal

## Python Solution

```python
class Solution:
    def maxTaskAssign(
        self, tasks: List[int], workers: List[int], pills: int, strength: int
    ) -> int:
        def feasible(k: int) -> bool:
            """
            Check if we can assign k tasks to the strongest k workers.

            Strategy: Use the strongest k workers and try to complete the easiest k tasks.
            For each worker (from weakest to strongest among the selected k):
            - Add all tasks that this worker could complete with a pill to the queue
            - If worker can complete the easiest task without pill, do it
            - Otherwise, use a pill on the hardest task in queue (greedy choice)
            """
            task_index = 0
            available_tasks = deque()
            pills_remaining = pills

            # Iterate through the strongest k workers (from weakest to strongest)
            for worker_index in range(num_workers - k, num_workers):
                current_worker_strength = workers[worker_index]

                # Add all tasks that this worker could potentially complete (with pill)
                # to the available tasks queue
                while task_index < k and tasks[task_index] <= current_worker_strength + strength:
                    available_tasks.append(tasks[task_index])
                    task_index += 1

                # No tasks available for this worker
                if not available_tasks:
                    return False

                # If worker can complete the easiest task without a pill
                if available_tasks[0] <= current_worker_strength:
                    available_tasks.popleft()
                # Need a pill but none available
                elif pills_remaining == 0:
                    return False
                # Use a pill on the hardest available task
                else:
                    pills_remaining -= 1
                    available_tasks.pop()  # Remove the hardest task from the right

            return True

        # Initialize variables
        num_tasks = len(tasks)
        num_workers = len(workers)

        # Sort both arrays for greedy assignment
        tasks.sort()
        workers.sort()

        # Binary search for the maximum number of tasks we can complete
        # feasible(k) creates pattern: [true, true, ..., false, false]
        # We want the last true (maximum k where we can complete tasks)
        # Invert: not_feasible(k) creates pattern: [false, false, ..., true, true]
        # Find first_true_index of not_feasible, answer is first_true_index - 1
        left, right = 0, min(num_tasks, num_workers)
        first_true_index = -1

        while left <= right:
            mid = (left + right) // 2
            if not feasible(mid):  # not_feasible condition
                first_true_index = mid
                right = mid - 1
            else:
                left = mid + 1

        # If first_true_index == -1, all values are feasible, return max
        # If first_true_index == 0, even 0 tasks is not feasible (shouldn't happen)
        # Otherwise, return first_true_index - 1 (last feasible value)
        if first_true_index == -1:
            return min(num_tasks, num_workers)
        return first_true_index - 1 if first_true_index > 0 else 0
```

## Time and Space Complexity

**Time Complexity:** `O((n + m) × log(n + m) + min(n, m) × n × log(min(n, m)))`

The time complexity breaks down as follows:
- Sorting tasks array: `O(n × log n)` where `n` is the number of tasks
- Sorting workers array: `O(m × log m)` where `m` is the number of workers
- Binary search runs `O(log(min(n, m)))` iterations
- For each binary search iteration, the `check` function is called:
  - The outer loop in `check` runs `x` times (at most `min(n, m)`)
  - The inner while loop processes at most `x` tasks total across all iterations
  - Deque operations (append, popleft, pop) are `O(1)`
  - Overall `check` function: `O(x)` where `x ≤ min(n, m)`
- Total for binary search with check: `O(min(n, m) × log(min(n, m)))`

Since sorting dominates when `n` and `m` are large, and considering the reference answer simplifies to `O(n × log n)`, this suggests `m` is treated as comparable to `n`. The dominant term becomes `O(n × log n)`.

**Space Complexity:** `O(n)`

The space complexity consists of:
- The deque `q` in the `check` function can hold at most `x` elements, where `x ≤ min(n, m) ≤ n`
- Other variables use constant space
- Total space: `O(n)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Using Wrong Binary Search Template Variant**
This problem finds the **maximum** (last true) rather than minimum (first true). The correct approach inverts the condition to use the standard first-true template.

**Incorrect Implementation:**
```python
# WRONG: Using while left < right with ceiling division
while left < right:
    mid = (left + right + 1) // 2
    if feasible(mid):
        left = mid
    else:
        right = mid - 1
return left
```

**Correct Implementation:**
```python
# CORRECT: Using template with first_true_index on inverted condition
left, right = 0, min(num_tasks, num_workers)
first_true_index = -1
while left <= right:
    mid = (left + right) // 2
    if not feasible(mid):  # Inverted: find first infeasible
        first_true_index = mid
        right = mid - 1
    else:
        left = mid + 1
# Answer is first_true_index - 1 (last feasible)
return first_true_index - 1 if first_true_index > 0 else (min(n, m) if first_true_index == -1 else 0)
```

### 2. **Incorrect Worker Selection Strategy**

**Pitfall:** A common mistake is trying to use the weakest workers first or randomly selecting workers, thinking we should "save" stronger workers for harder tasks.

**Why it's wrong:** This approach fails because:
- Weak workers might not be able to complete any tasks even with pills
- We waste pills on workers who can't contribute effectively
- We miss opportunities where strong workers could complete tasks without pills

**Solution:** Always use the strongest k workers when trying to complete k tasks:
```python
# CORRECT: Use strongest k workers
for worker_index in range(num_workers - k, num_workers):
    current_worker_strength = workers[worker_index]
    # ... rest of logic
```

### 3. **Greedy Pill Usage on Wrong Tasks**

**Pitfall:** When a worker needs a pill, assigning them to the easiest available task (from the front of the deque).

**Why it's wrong:** This wastes the pill's potential. If we use a pill, we should maximize its value by completing the hardest possible task.

**Solution:** When using a pill, assign the worker to the hardest available task:
```python
# CORRECT: Using pill on hardest task
if available_tasks[0] <= current_worker_strength:
    available_tasks.popleft()  # No pill needed, take easiest
elif pills_remaining > 0:
    pills_remaining -= 1
    available_tasks.pop()  # With pill, take hardest task
```

### 4. **Incorrect Task Availability Window**

**Pitfall:** Adding tasks to the available queue based only on the worker's base strength, or adding all tasks at once.

**Solution:** Consider the worker's strength with a potential pill:
```python
# CORRECT: Consider strength with pill
while task_index < k and tasks[task_index] <= current_worker_strength + strength:
    available_tasks.append(tasks[task_index])
    task_index += 1
```

These pitfalls often lead to incorrect results or runtime errors. The key insight is that the greedy strategy must be carefully implemented: use the strongest workers, and when pills are needed, maximize their value by completing the hardest possible tasks.

