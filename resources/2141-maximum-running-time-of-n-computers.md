# 2141. Maximum Running Time of N Computers

> Source: https://algo.monster/liteproblems/2141

## Problem Statement

You have `n` computers and an array of batteries. Each battery has a certain capacity - the `i-th` battery can power a computer for `batteries[i]` minutes.

Your goal is to run all `n` computers simultaneously for as long as possible. Here are the rules:

- Initially, you can insert at most one battery into each computer
- At any point in time, you can swap batteries between computers instantly (removing and inserting takes no time)
- Each battery can only power one computer at a time
- Batteries cannot be recharged - once their power is used, it's gone
- You want all `n` computers running at the same time

The problem asks you to find the maximum number of minutes that all `n` computers can run simultaneously.

For example, if you have 2 computers and batteries with capacities `[3, 3, 3]` minutes, you could:
- Start with 2 batteries in the 2 computers (each with 3 minutes)
- When those are depleted after 3 minutes, swap in the third battery to one computer
- But since you need both computers running simultaneously, the maximum runtime would be 4 minutes total (you can strategically swap batteries to distribute the 9 total minutes across 2 computers)

## Intuition

Let's think about what happens when we try to run all `n` computers for `t` minutes. The key observation is that if we can run all computers for `t` minutes, we can definitely run them for any time less than `t` minutes. This monotonic property suggests [binary search](/problems/binary_search_intro) might work.

Now, how do we check if it's possible to run all computers for `t` minutes? Consider what happens with each battery:

- If a battery has capacity greater than `t` minutes, we can only use `t` minutes from it (since we're only running for `t` minutes total)
- If a battery has capacity less than or equal to `t` minutes, we can use all of it

Since we can swap batteries freely and instantly, the problem becomes: can we distribute the available battery power to keep `n` computers running for `t` minutes?

For `n` computers running `t` minutes each, we need a total of `n * t` minutes of battery power. We can get at most `min(battery[i], t)` minutes from each battery. So the condition becomes:

`sum(min(battery[i], t) for all batteries) >= n * t`

This gives us a clear way to check feasibility for any given time `t`.

The [binary search](/problems/binary_search_intro) approach emerges naturally:
- Search space: from `0` to `sum(all batteries)` (maximum possible if we had just 1 computer)
- For each candidate time `mid`, check if we have enough total usable battery power
- If yes, we can potentially run longer, so search the right half
- If no, we need to reduce the time, so search the left half

The beauty of this solution is that it transforms a complex scheduling problem into a simple arithmetic check by recognizing that with free swapping, only the total usable power matters, not the specific assignment of batteries to computers.

**Learn more about [Greedy](/problems/greedy_intro), [Binary Search](/problems/binary_search_intro) and [Sorting](/problems/sorting_summary) patterns.**

## Solution Approach

We implement the solution using the [binary search](/problems/binary_search_intro) boundary-finding template on the answer space.

**The Feasible Function:**

For a given target runtime `t`, we check if it's achievable:

```python
def feasible(target_time):
    total_contribution = sum(min(battery, target_time) for battery in batteries)
    return total_contribution >= n * target_time
```

Each battery can contribute at most `target_time` minutes (capped by its capacity). If the total contribution is at least `n * target_time`, we have enough power to run all computers for `target_time` minutes.

**Binary Search Setup:**
- Initialize `left = 0` and `right = sum(batteries)` (maximum theoretical runtime)
- Initialize `first_true_index = -1` to track the last feasible time found

**Binary Search with Template Pattern:**

```python
left, right = 0, sum(batteries)
first_true_index = -1

while left <= right:
    mid = (left + right) // 2
    if feasible(mid):
        first_true_index = mid
        left = mid + 1  # Try for longer runtime
    else:
        right = mid - 1  # Reduce runtime

return first_true_index
```

The search pattern is:
- `false, false, ..., true, true, true` (lower times are feasible, higher times are not)
- Wait, actually it's inverted: lower times are always feasible, we want the **maximum** feasible time
- So we search for the **last** true (highest feasible runtime)

When `feasible(mid)` is true:
- Record `first_true_index = mid` (this could be our answer)
- Try for a longer time: `left = mid + 1`

When `feasible(mid)` is false:
- The time is too long, try shorter: `right = mid - 1`

**Time Complexity:** `O(m × log S)` where `m` is the number of batteries and `S` is the sum of all battery capacities.

**Space Complexity:** `O(1)` as we only use a few variables for the [binary search](/problems/binary_search_intro).

## Example Walkthrough

Let's walk through an example with `n = 2` computers and `batteries = [3, 3, 3]`.

**Step 1: Initialize Binary Search Boundaries**
- Left boundary: `left = 0` (minimum runtime)
- Right boundary: `right = 3 + 3 + 3 = 9` (total battery capacity)
- `first_true_index = -1`

**Step 2: First Iteration**
- Calculate middle: `mid = (0 + 9) / 2 = 4`
- Check if we can run for 4 minutes:
  - Battery 1: `min(3, 4) = 3` minutes usable
  - Battery 2: `min(3, 4) = 3` minutes usable
  - Battery 3: `min(3, 4) = 3` minutes usable
  - Total usable: `3 + 3 + 3 = 9` minutes
  - Need for 2 computers: `2 × 4 = 8` minutes
  - Since `9 >= 8`, feasible! → `first_true_index = 4`, `left = 5`

**Step 3: Second Iteration**
- Calculate middle: `mid = (5 + 9) / 2 = 7`
- Check if we can run for 7 minutes:
  - Each battery: `min(3, 7) = 3` minutes usable
  - Total usable: `3 + 3 + 3 = 9` minutes
  - Need for 2 computers: `2 × 7 = 14` minutes
  - Since `9 < 14`, NOT feasible → `right = 6`

**Step 4: Third Iteration**
- Calculate middle: `mid = (5 + 6) / 2 = 5`
- Check if we can run for 5 minutes:
  - Each battery: `min(3, 5) = 3` minutes usable
  - Total usable: `9` minutes
  - Need for 2 computers: `2 × 5 = 10` minutes
  - Since `9 < 10`, NOT feasible → `right = 4`

**Step 5: Termination**
- Now `left = 5 > right = 4`, so the loop ends
- The answer is `first_true_index = 4` minutes

This makes sense: With 9 total battery minutes and 2 computers, the maximum is 4 minutes (using 8 of the 9 available minutes).

## Python Solution

```python
class Solution:
    def maxRunTime(self, n: int, batteries: List[int]) -> int:
        """
        Find the maximum time all n computers can run simultaneously.

        Uses binary search with the boundary-finding template to find
        the maximum feasible runtime.

        Args:
            n: Number of computers to power
            batteries: List of battery capacities in minutes

        Returns:
            Maximum number of minutes all computers can run simultaneously
        """

        def feasible(target_time: int) -> bool:
            """
            Check if all n computers can run for target_time minutes.

            Each battery can contribute at most target_time minutes.
            Total contribution must be at least n * target_time.

            Args:
                target_time: The target runtime to check

            Returns:
                True if target_time is achievable, False otherwise
            """
            total_contribution = sum(min(battery, target_time) for battery in batteries)
            return total_contribution >= n * target_time

        # Binary search for the maximum feasible runtime
        left, right = 0, sum(batteries)
        first_true_index = -1

        while left <= right:
            mid = (left + right) // 2

            if feasible(mid):
                first_true_index = mid
                left = mid + 1  # Try for longer runtime
            else:
                right = mid - 1  # Reduce runtime

        return first_true_index
```

## Time and Space Complexity

The time complexity is `O(m × log S)`, where `m` is the number of batteries (length of the batteries list) and `S` is the sum of all battery powers.

The binary search runs for `O(log S)` iterations since we're searching in the range `[0, sum(batteries)]`. In each iteration of the binary search, we compute `sum(min(x, mid) for x in batteries)`, which iterates through all `m` batteries, taking `O(m)` time. Therefore, the overall time complexity is `O(m × log S)`.

The space complexity is `O(1)` as we only use a constant amount of extra space for variables `left`, `right`, `mid`, and `first_true_index`. The generator expression `sum(min(x, mid) for x in batteries)` doesn't create an intermediate list but computes the sum on-the-fly, using constant extra space.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Using Wrong Binary Search Template Variant**

**Pitfall:** Using `while left < right` with `left = mid` pattern, which can cause infinite loops and doesn't match the boundary-finding template.

**Example of incorrect approach:**
```python
while left < right:
    mid = (left + right + 1) >> 1
    if feasible(mid):
        left = mid
    else:
        right = mid - 1
return left
```

**Solution:** Use the standard boundary-finding template with `while left <= right` and `first_true_index` tracking:
```python
first_true_index = -1
while left <= right:
    mid = (left + right) // 2
    if feasible(mid):
        first_true_index = mid
        left = mid + 1
    else:
        right = mid - 1
return first_true_index
```

### 2. **Integer Overflow in Binary Search Boundary**

**Pitfall:** Setting the right boundary as `sum(batteries)` can cause integer overflow in languages with fixed integer sizes when battery capacities are large.

**Solution:** In Python, this isn't an issue due to arbitrary precision integers, but in languages like Java or C++, use `long` type for boundaries and calculations:
```java
long right = 0;
for (int battery : batteries) {
    right += battery;
}
```

### 3. **Misunderstanding the Feasibility Check Logic**

**Pitfall:** Thinking that batteries must be assigned to specific computers or that we need to simulate the actual battery swapping process.

**Incorrect approach:**
```python
# Wrong: Trying to assign batteries to computers
if len(batteries) >= n and all(b >= mid for b in batteries[:n]):
    # This doesn't account for battery swapping optimization
```

**Solution:** The correct approach uses the greedy insight that we can pool all available battery minutes (capped at `mid`) and check if the total is sufficient for `n * mid` minutes of runtime.

### 4. **Not Capping Battery Contribution**

**Pitfall:** Forgetting to use `min(battery_capacity, mid)` and instead using the full battery capacity when checking feasibility.

**Incorrect:**
```python
total_contribution = sum(batteries)  # Wrong!
if total_contribution >= n * mid:
    ...
```

**Solution:** Each battery can only contribute up to `mid` minutes toward the target runtime, regardless of its actual capacity:
```python
total_contribution = sum(min(battery, mid) for battery in batteries)
```

### 5. **Edge Case: first_true_index Never Updated**

**Pitfall:** If no runtime is feasible (which shouldn't happen since 0 is always feasible), returning -1 could cause issues.

**Solution:** Initialize `first_true_index = 0` since running for 0 minutes is always possible, or handle the edge case explicitly:
```python
if not batteries:
    return 0
```

