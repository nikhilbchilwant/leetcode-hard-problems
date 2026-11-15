# 2398. Maximum Number of Robots Within Budget

> Source: https://algo.monster/liteproblems/2398

## Problem Statement

You have `n` robots, each with specific charging and running costs. Given two arrays:
- `chargeTimes[i]`: the cost to charge the i-th robot
- `runningCosts[i]`: the cost to run the i-th robot

You need to select `k` **consecutive** robots (a contiguous subarray) to run within a given `budget`.

The total cost formula for running `k` chosen consecutive robots is:
```
Total Cost = max(chargeTimes) + k * sum(runningCosts)
```

Where:
- `max(chargeTimes)` is the highest charge cost among the selected `k` robots
- `sum(runningCosts)` is the sum of all running costs for the selected `k` robots
- `k` is the number of consecutive robots chosen

Your task is to find the **maximum** number of consecutive robots you can run without exceeding the budget.

For example, if you select robots from index `i` to index `j`, you're selecting `j - i + 1` consecutive robots. The cost would be the maximum charge time among these robots plus the count of robots multiplied by the sum of their running costs.

The goal is to maximize the number of consecutive robots while keeping the total cost ≤ `budget`.

## Intuition

The key insight is recognizing this as a [sliding window](/problems/sliding_window_maximum) problem with a twist - we need to track the maximum value within each window.

Think about it this way: we want to find the longest consecutive subarray of robots that fits our budget. This naturally suggests trying different window sizes. We could check every possible consecutive subarray, but that would be inefficient.

Instead, we can use a [sliding window](/problems/sliding_window_maximum) approach where we expand the window by adding robots on the right and shrink it by removing robots from the left when the cost exceeds the budget. The challenge is efficiently tracking the maximum `chargeTime` in our current window as it changes.

For tracking the maximum in a [sliding window](/problems/sliding_window_maximum), a monotonic deque is perfect. Why? Because:
1. When a new element comes in that's larger than elements at the back of the deque, those smaller elements will never be the maximum as long as this new element is in the window
2. The front of the deque always contains the maximum element in the current window
3. When the window shrinks and removes the leftmost element, we can easily remove it from the deque if it's at the front

The two-pointer technique works naturally here:
- Right pointer (`r`) expands the window by including new robots
- Left pointer (`l`) shrinks the window when the total cost exceeds budget
- We continuously track the running sum of `runningCosts` for efficiency
- The monotonic deque maintains the maximum `chargeTime` in the current window

The formula `(r - l + 1) * s + chargeTimes[q[0]] > budget` checks if our current window exceeds the budget, where `(r - l + 1)` is the window size, `s` is the sum of running costs, and `chargeTimes[q[0]]` is the maximum charge time.

This approach efficiently finds the maximum window size in a single pass through the array, giving us an `O(n)` solution.

**Learn more about [Queue](/problems/queue_intro), [Binary Search](/problems/binary_search_intro), [Prefix Sum](/problems/subarray_sum), [Sliding Window](/problems/sliding_window_maximum), [Monotonic Queue](/problems/mono_stack_intro) and [Heap (Priority Queue)](/problems/heap_intro) patterns.**

## Solution Approach

The solution uses a [sliding window](/problems/sliding_window_maximum) technique combined with a monotonic deque to efficiently track the maximum charge time within the current window.

**Data Structures Used:**
- A deque `q` to maintain indices of robots in decreasing order of their charge times
- Two pointers `l` and `r` for the [sliding window](/problems/sliding_window_maximum) boundaries
- A variable `s` to track the sum of running costs in the current window

**Implementation Walkthrough:**

1. **Initialize variables:**
   - `q = deque()` - monotonic deque for tracking maximum charge time
   - `ans = 0` - stores the maximum number of consecutive robots
   - `s = 0` - running sum of `runningCosts` in current window
   - `l = 0` - left boundary of the [sliding window](/problems/sliding_window_maximum)

2. **Expand the window (right pointer):**
   ```python
   for r, (t, c) in enumerate(zip(chargeTimes, runningCosts)):
       s += c  # Add current robot's running cost to sum
   ```

3. **Maintain the monotonic deque:**
   ```python
   while q and chargeTimes[q[-1]] <= t:
       q.pop()
   q.append(r)
   ```
   - Remove indices from the back of deque if their charge times are less than or equal to the current robot's charge time
   - This ensures the deque front always has the index of the robot with maximum charge time in the current window

4. **Shrink the window when budget is exceeded:**
   ```python
   while q and (r - l + 1) * s + chargeTimes[q[0]] > budget:
       if q[0] == l:
           q.popleft()  # Remove from deque if it's going out of window
       s -= runningCosts[l]  # Subtract the running cost of removed robot
       l += 1  # Move left boundary right
   ```
   - Check if total cost exceeds budget: `window_size * sum_of_running_costs + max_charge_time`
   - If the robot at index `l` is the maximum (at deque front), remove it from deque
   - Update the running sum and move the left pointer

5. **Update the answer:**
   ```python
   ans = max(ans, r - l + 1)
   ```
   - After adjusting the window to fit the budget, record the window size

The algorithm processes each robot exactly once with the right pointer and at most once with the left pointer, resulting in `O(n)` time complexity. The space complexity is `O(n)` for the deque in the worst case.

## Example Walkthrough

Let's walk through a concrete example to illustrate the solution approach.

**Given:**
- `chargeTimes = [3, 6, 1, 3, 4]`
- `runningCosts = [2, 1, 3, 4, 5]`
- `budget = 25`

We want to find the maximum number of consecutive robots we can run.

**Step-by-step execution:**

**Initial state:** `l = 0`, `r = 0`, `s = 0`, `q = []`, `ans = 0`

**Iteration 1 (r = 0):**
- Add robot 0: `s = 0 + 2 = 2`
- Deque is empty, add index 0: `q = [0]`
- Check cost: `(0-0+1) * 2 + 3 = 1 * 2 + 3 = 5 ≤ 25` ✓
- Update answer: `ans = max(0, 1) = 1`

**Iteration 2 (r = 1):**
- Add robot 1: `s = 2 + 1 = 3`
- `chargeTimes[1] = 6 > chargeTimes[0] = 3`, add to deque: `q = [0, 1]`
- But wait, we maintain decreasing order, so we first remove 0: `q = [1]`
- Check cost: `(1-0+1) * 3 + 6 = 2 * 3 + 6 = 12 ≤ 25` ✓
- Update answer: `ans = max(1, 2) = 2`

**Iteration 3 (r = 2):**
- Add robot 2: `s = 3 + 3 = 6`
- `chargeTimes[2] = 1 < chargeTimes[1] = 6`, add to deque: `q = [1, 2]`
- Check cost: `(2-0+1) * 6 + 6 = 3 * 6 + 6 = 24 ≤ 25` ✓
- Update answer: `ans = max(2, 3) = 3`

**Iteration 4 (r = 3):**
- Add robot 3: `s = 6 + 4 = 10`
- `chargeTimes[3] = 3 > chargeTimes[2] = 1`, remove 2: `q = [1, 3]`
- Check cost: `(3-0+1) * 10 + 6 = 4 * 10 + 6 = 46 > 25` ✗
- **Need to shrink window:**
  - `q[0] = 1 ≠ l = 0`, so don't remove from deque
  - Remove robot 0: `s = 10 - 2 = 8`, `l = 1`
  - Check cost: `(3-1+1) * 8 + 6 = 3 * 8 + 6 = 30 > 25` ✗
  - Continue shrinking:
  - `q[0] = 1 = l = 1`, remove from deque: `q = [3]`
  - Remove robot 1: `s = 8 - 1 = 7`, `l = 2`
  - Check cost: `(3-2+1) * 7 + 3 = 2 * 7 + 3 = 17 ≤ 25` ✓
- Update answer: `ans = max(3, 2) = 3`

**Iteration 5 (r = 4):**
- Add robot 4: `s = 7 + 5 = 12`
- `chargeTimes[4] = 4 > chargeTimes[3] = 3`, remove 3: `q = [4]`
- Check cost: `(4-2+1) * 12 + 4 = 3 * 12 + 4 = 40 > 25` ✗
- **Need to shrink window:**
  - `q[0] = 4 ≠ l = 2`, so don't remove from deque
  - Remove robot 2: `s = 12 - 3 = 9`, `l = 3`
  - Check cost: `(4-3+1) * 9 + 4 = 2 * 9 + 4 = 22 ≤ 25` ✓
- Update answer: `ans = max(3, 2) = 3`

**Final answer: 3** (we can run at most 3 consecutive robots within the budget)

The optimal selection is robots [0, 1, 2] with total cost = max(3, 6, 1) + 3 * (2 + 1 + 3) = 6 + 18 = 24 ≤ 25.

## Python Solution

```python
from collections import deque
from typing import List


class Solution:
    def maximumRobots(
        self, chargeTimes: List[int], runningCosts: List[int], budget: int
    ) -> int:
        # Monotonic deque to maintain indices of charge times in descending order
        max_charge_deque = deque()

        # Variables to track the maximum number of robots and running sum
        max_robots = 0
        running_sum = 0
        left = 0

        # Iterate through each robot using sliding window approach
        for right, (charge_time, running_cost) in enumerate(zip(chargeTimes, runningCosts)):
            # Add current robot's running cost to the sum
            running_sum += running_cost

            # Maintain monotonic deque: remove elements smaller than current charge time
            while max_charge_deque and chargeTimes[max_charge_deque[-1]] <= charge_time:
                max_charge_deque.pop()

            # Add current index to deque
            max_charge_deque.append(right)

            # Shrink window from left while total cost exceeds budget
            # Total cost = max(chargeTimes in window) + k * sum(runningCosts in window)
            # where k is the number of robots (window size)
            while max_charge_deque and (right - left + 1) * running_sum + chargeTimes[max_charge_deque[0]] > budget:
                # Remove leftmost element from deque if it's going out of window
                if max_charge_deque[0] == left:
                    max_charge_deque.popleft()

                # Remove leftmost robot's running cost from sum
                running_sum -= runningCosts[left]

                # Move left pointer forward
                left += 1

            # Update maximum number of robots that can be used
            max_robots = max(max_robots, right - left + 1)

        return max_robots
```

## Time and Space Complexity

**Time Complexity:** `O(n)`

The algorithm uses a sliding window approach with a deque to track maximum charge times. The outer loop iterates through all `n` robots once. While there are nested while loops inside, each element can only be added to and removed from the deque at most once, and each element can only be involved in the window expansion (right pointer movement) and contraction (left pointer movement) at most once. Therefore, despite the nested structure, each operation on each element happens a constant number of times, resulting in `O(n)` total time complexity.

**Space Complexity:** `O(n)`

The deque `q` stores indices of robots within the current window. In the worst case, when all charge times are in increasing order and the budget allows all robots to be selected, the deque could contain all `n` indices, requiring `O(n)` space. Other variables (`ans`, `s`, `l`, `r`, `t`, `c`) use constant space `O(1)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Total Cost Calculation**
**Pitfall:** A common mistake is misunderstanding the cost formula. Some might incorrectly calculate the total cost as:
- `max(chargeTimes) + sum(runningCosts)` (missing the multiplication by k)
- `sum(chargeTimes) + k * sum(runningCosts)` (using sum instead of max for charge times)

**Solution:** Remember the correct formula:
```python
total_cost = max(chargeTimes[l:r+1]) + (r - l + 1) * sum(runningCosts[l:r+1])
```
Where `(r - l + 1)` is the window size (k), not just 1.

### 2. **Deque Maintenance Error When Shrinking Window**
**Pitfall:** Forgetting to remove the leftmost element from the deque when it goes out of the window bounds. This causes the deque to contain indices that are no longer in the current window, leading to incorrect maximum charge time calculations.

```python
# Wrong: Only updating sum and left pointer
while total_cost > budget:
    running_sum -= runningCosts[left]
    left += 1
    # Missing: deque cleanup!
```

**Solution:** Always check if the front of the deque equals the left boundary before incrementing:
```python
while max_charge_deque and total_cost > budget:
    if max_charge_deque[0] == left:  # Critical check
        max_charge_deque.popleft()
    running_sum -= runningCosts[left]
    left += 1
```

### 3. **Empty Deque Access**
**Pitfall:** Accessing `max_charge_deque[0]` when the deque might be empty, causing an IndexError. This can happen after removing elements or when no valid window exists.

**Solution:** Always check if the deque is non-empty before accessing:
```python
# Check deque is not empty in the while condition
while max_charge_deque and (right - left + 1) * running_sum + chargeTimes[max_charge_deque[0]] > budget:
    # Safe to access max_charge_deque[0] here
```

### 4. **Integer Overflow in Cost Calculation**
**Pitfall:** For large values, the expression `(r - l + 1) * running_sum` might overflow in languages with fixed integer sizes. Even in Python, this could cause performance issues with very large numbers.

**Solution:** Consider the order of operations and potential early termination:
```python
# If window size alone exceeds budget/min_cost, skip calculation
window_size = right - left + 1
if window_size > budget:  # Early check for impossible cases
    # Shrink window
```

### 5. **Off-by-One Error in Window Size**
**Pitfall:** Incorrectly calculating the window size as `r - l` instead of `r - l + 1`, leading to wrong cost calculations and potentially selecting more robots than the budget allows.

**Solution:** Always use `right - left + 1` for the window size:
```python
window_size = right - left + 1  # Correct
# NOT: window_size = right - left  # Wrong!
```

