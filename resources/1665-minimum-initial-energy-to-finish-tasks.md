# 1665. Minimum Initial Energy to Finish Tasks

> Source: https://algo.monster/liteproblems/1665

## Problem Statement

You have an array of tasks where each task is represented as `[actual, minimum]`:
- `actual` is the energy consumed when completing the task
- `minimum` is the minimum energy level required to start the task

To complete a task, your current energy must be at least equal to the `minimum` value. After completing the task, your energy decreases by the `actual` amount.

For example, with task `[10, 12]`:
- If your current energy is 11, you cannot start the task (11 < 12)
- If your current energy is 13, you can start the task, and after completion you'll have 13 - 10 = 3 energy remaining

You can complete the tasks in any order you choose. The goal is to find the minimum initial energy needed to complete all tasks.

The solution uses a [greedy](/problems/greedy_intro) approach with custom [sorting](/problems/sorting_summary). The key insight is that to minimize the initial energy, we should complete tasks in a specific order based on the difference `actual - minimum`. By sorting tasks in ascending order of `actual[i] - minimum[i]`, we ensure that tasks requiring more "buffer" energy (where `minimum` is much larger than `actual`) are done later when we have more flexibility.

## Intuition

Let's think about what happens when we complete tasks in different orders. The key observation is that the total energy consumed (sum of all `actual` values) is fixed regardless of order, but the initial energy requirement changes based on the sequence.

Consider two adjacent tasks in our sequence. If we have tasks A and B, should we do A first or B first? Let's analyze what initial energy we need for each ordering:

**If we do A then B:**
- We need at least `minimum_A` energy to start
- After A, we have energy reduced by `actual_A`
- To do B next, we need the remaining energy to be at least `minimum_B`
- So initial energy must be at least `max(minimum_A, actual_A + minimum_B)`

**If we do B then A:**
- Initial energy must be at least `max(minimum_B, actual_B + minimum_A)`

To minimize initial energy, we want to choose the ordering that gives us the smaller requirement. Through mathematical analysis, we find that we should do A before B when `actual_A - minimum_A < actual_B - minimum_B`.

This leads to the insight: tasks with smaller `actual - minimum` difference should be done first. Why? Because these tasks give us less "energy buffer" after completion. Tasks where `minimum` is much larger than `actual` are like "energy gates" - they require a lot to start but don't consume much. It's better to save these for last when we've already accumulated enough energy.

By [sorting](/problems/sorting_summary) tasks by `actual - minimum` in ascending order, we ensure we're always making the optimal local choice, which leads to the global minimum initial energy. As we traverse the sorted tasks, we only add energy when absolutely necessary (when current energy falls below a task's minimum), ensuring we use the least possible initial energy.

**Learn more about [Greedy](/problems/greedy_intro) and [Sorting](/problems/sorting_summary) patterns.**

## Solution Approach

The implementation follows the [greedy](/problems/greedy_intro) strategy with custom [sorting](/problems/sorting_summary) that we identified from our intuition.

**Step 1: Sort the tasks**

We sort the tasks based on `actual - minimum` in ascending order using a lambda function:
```python
sorted(tasks, key=lambda x: x[0] - x[1])
```
This ensures tasks with smaller difference (less energy buffer) are processed first.

**Step 2: Initialize variables**

- `ans`: tracks the minimum initial energy required (this is what we'll return)
- `cur`: tracks the current energy level as we process tasks

**Step 3: Process tasks in sorted order**

For each task `[a, m]` in the sorted list:

1. **Check if we have enough energy to start**: If `cur < m`, we don't have enough energy to begin this task. We need to add energy to our initial amount:
   ```python
   if cur < m:
       ans += m - cur  # Add just enough to meet the minimum
       cur = m         # Update current energy to the minimum required
   ```
 
2. **Complete the task**: After ensuring we have enough energy to start, we complete the task and reduce our current energy by the actual amount consumed:
   ```python
   cur -= a
   ```

**Why this works:**

As proven in the reference approach, for any sequence of tasks, the minimum initial energy needed is:
$$E \geq \sum_{i=1}^{n} a_i + (m_n - a_n)$$

Where the last task determines the additional buffer needed beyond the total energy consumed. By [sorting](/problems/sorting_summary) tasks to minimize `m_n - a_n` (equivalently, maximize `a_n - m_n`), we minimize the required initial energy.

The algorithm cleverly tracks this by only adding energy when absolutely necessary, ensuring the final `ans` represents the minimum initial energy needed to complete all tasks in the optimal order.

## Example Walkthrough

Let's walk through a concrete example with tasks: `[[1, 3], [2, 4], [10, 11], [10, 12], [8, 9]]`

**Step 1: Sort by (actual - minimum)**

Calculate the differences:
- Task `[1, 3]`: difference = 1 - 3 = -2
- Task `[2, 4]`: difference = 2 - 4 = -2  
- Task `[10, 11]`: difference = 10 - 11 = -1
- Task `[10, 12]`: difference = 10 - 12 = -2
- Task `[8, 9]`: difference = 8 - 9 = -1

After sorting in ascending order: `[[1, 3], [2, 4], [10, 12], [10, 11], [8, 9]]`

**Step 2: Initialize tracking variables**
- `ans = 0` (minimum initial energy needed)
- `cur = 0` (current energy level)

**Step 3: Process each task in sorted order**

**Task 1: [1, 3]**
- Current energy: 0, Required minimum: 3
- Since 0 < 3, we need more energy
- Add to initial: `ans = 0 + (3 - 0) = 3`
- Update current: `cur = 3`
- Complete task: `cur = 3 - 1 = 2`

**Task 2: [2, 4]**  
- Current energy: 2, Required minimum: 4
- Since 2 < 4, we need more energy
- Add to initial: `ans = 3 + (4 - 2) = 5`
- Update current: `cur = 4`
- Complete task: `cur = 4 - 2 = 2`

**Task 3: [10, 12]**
- Current energy: 2, Required minimum: 12
- Since 2 < 12, we need more energy
- Add to initial: `ans = 5 + (12 - 2) = 15`
- Update current: `cur = 12`
- Complete task: `cur = 12 - 10 = 2`

**Task 4: [10, 11]**
- Current energy: 2, Required minimum: 11
- Since 2 < 11, we need more energy
- Add to initial: `ans = 15 + (11 - 2) = 24`
- Update current: `cur = 11`
- Complete task: `cur = 11 - 10 = 1`

**Task 5: [8, 9]**
- Current energy: 1, Required minimum: 9
- Since 1 < 9, we need more energy
- Add to initial: `ans = 24 + (9 - 1) = 32`
- Update current: `cur = 9`
- Complete task: `cur = 9 - 8 = 1`

**Result:** The minimum initial energy needed is **32**.

This example demonstrates how the algorithm greedily adds just enough energy whenever needed, ensuring we find the absolute minimum initial energy required to complete all tasks in the optimal order.

## Python Solution

```python
class Solution:
    def minimumEffort(self, tasks: List[List[int]]) -> int:
        # Sort tasks by the difference (actual - minimum) in ascending order
        # This ensures we handle tasks with larger "extra" requirements first
        sorted_tasks = sorted(tasks, key=lambda task: task[0] - task[1])
      
        total_initial_effort = 0  # Total initial effort needed
        current_energy = 0  # Current available energy
      
        # Process each task in the sorted order
        for actual_effort, minimum_effort in sorted_tasks:
            # If current energy is less than minimum required to start this task
            if current_energy < minimum_effort:
                # Add the deficit to our total initial effort
                effort_deficit = minimum_effort - current_energy
                total_initial_effort += effort_deficit
                # Update current energy to the minimum required
                current_energy = minimum_effort
          
            # Consume the actual effort for this task
            current_energy -= actual_effort
      
        return total_initial_effort
```

## Time and Space Complexity

The time complexity is `O(n × log n)`, where `n` is the number of tasks. This is dominated by the sorting operation `sorted(tasks, key=lambda x: x[0] - x[1])`, which uses a comparison-based sorting algorithm (typically Timsort in Python) that has `O(n × log n)` time complexity. The subsequent for loop iterates through all tasks once, contributing `O(n)` time, but this is absorbed by the larger sorting complexity.

The space complexity is `O(n)`. While the reference answer mentions `O(1)` when ignoring the space overhead of sorting, the `sorted()` function in Python creates a new sorted list rather than sorting in-place, which requires `O(n)` additional space to store the sorted copy of the tasks list. If we were to ignore this sorting overhead as suggested in the reference, then the remaining operations use only a constant amount of extra space (`ans`, `cur`, and loop variables), resulting in `O(1)` space complexity.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. Incorrect Sorting Logic - Sorting by Wrong Criteria

**The Pitfall:**
A common mistake is sorting by the wrong criteria, such as:
- Sorting by `minimum` alone (ascending or descending)
- Sorting by `actual` alone
- Sorting by `minimum - actual` instead of `actual - minimum`

**Why it fails:**
Consider tasks `[[10, 10], [1, 20]]`:
- If sorted by minimum ascending: `[[10, 10], [1, 20]]` → requires 30 initial energy
- If sorted by `actual - minimum`: `[[10, 10], [1, 20]]` → requires 21 initial energy (optimal)

Sorting by the wrong criteria leads to suboptimal task ordering and higher initial energy requirements.

**Solution:**
Always sort by `actual - minimum` in ascending order (or equivalently, `minimum - actual` in descending order):
```python
sorted_tasks = sorted(tasks, key=lambda task: task[0] - task[1])
```

### 2. Not Understanding the Energy Update Logic

**The Pitfall:**
Incorrectly updating the current energy when insufficient, such as:
```python
# Wrong approach 1: Adding the minimum required instead of the deficit
if cur < m:
    ans += m  # Wrong! This adds the full minimum, not the deficit
    cur = m
  
# Wrong approach 2: Setting ans to maximum seen so far
if cur < m:
    ans = max(ans, m)  # Wrong! This doesn't accumulate properly
```

**Why it fails:**
The algorithm needs to track the cumulative energy additions, not individual minimums. Each time we're short on energy, we need to add exactly the deficit to our initial energy pool.

**Solution:**
Add only the deficit (shortage) to the initial energy:
```python
if cur < m:
    ans += m - cur  # Add only what's missing
    cur = m         # Update current to minimum required
```

### 3. Confusing Initial Energy vs Current Energy

**The Pitfall:**
Mixing up what `ans` (initial energy) and `cur` (current energy) represent:
```python
# Wrong: Thinking ans represents current energy at any point
for a, m in sorted_tasks:
    if ans < m:  # Wrong! Should check 'cur' not 'ans'
        ans += m - ans
```

**Why it fails:**
- `ans` accumulates the total initial energy we need to start with
- `cur` tracks our energy level as we progress through tasks
- Checking `ans < m` doesn't tell us if we can currently start a task

**Solution:**
Keep the roles clear:
- Use `cur` to track current energy state
- Use `ans` only to accumulate the initial energy requirement
- Always check `cur < minimum` to determine if we can start a task

