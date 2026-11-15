# 1751. Maximum Number of Events That Can Be Attended II

> Source: https://algo.monster/liteproblems/1751

## Problem Statement

You have an array of events where each event is represented as `events[i] = [startDay_i, endDay_i, value_i]`. This means:
- The i-th event starts on `startDay_i`
- The i-th event ends on `endDay_i`
- If you attend this event, you receive a value of `value_i`

You're also given an integer `k` which represents the maximum number of events you can attend.

Important constraints:
- You can only attend one event at a time
- If you choose to attend an event, you must attend the entire event from start to finish
- The end day is inclusive - you cannot attend two events where one starts on the same day another ends

Your goal is to find the maximum sum of values you can receive by attending at most `k` events.

For example, if you have events like `[[1,2,10], [2,3,20], [3,4,30]]` and `k=2`, you need to choose which 2 events to attend to maximize your total value. Since the first event ends on day 2 and the second event starts on day 2, you cannot attend both (as the end day is inclusive). You could attend events 1 and 3 for a total value of 40, or events 2 and 3 for a total value of 50, making 50 the maximum achievable value.

## Intuition

When faced with this problem, we need to make decisions about which events to attend to maximize our total value. This is a classic optimization problem where we have choices to make at each step.

The key insight is that for each event, we have two choices: either attend it or skip it. This binary decision-making pattern suggests a recursive approach where we explore both possibilities and choose the better outcome.

To structure our approach, we should first sort the events by start time. This ordering helps us process events chronologically and ensures that once we've made a decision about an event, we only need to consider future events.

For any event at position `i`, if we decide to skip it, we simply move to the next event at position `i+1` with the same number of remaining events we can attend. If we decide to attend it, we gain its value, but we need to find the next event we're allowed to attend (one that starts after the current event ends).

Since events are sorted by start time, we can use [binary search](/problems/binary_search_intro) to efficiently find the first event whose start time is greater than the current event's end time. This is crucial for performance - instead of checking each subsequent event linearly, we can jump directly to the next valid event in `O(log n)` time.

The recursive nature of the problem leads us to define a function `dfs(i, k)` that represents: "Starting from event `i`, with `k` events left to attend, what's the maximum value we can get?" At each step, we take the maximum of:
- Not attending event `i`: `dfs(i+1, k)`
- Attending event `i`: `value[i] + dfs(j, k-1)` where `j` is the next valid event

Since we'll encounter the same subproblems multiple times (same event index with same remaining capacity), we can use memoization to cache results and avoid redundant calculations. This transforms our solution from exponential to polynomial time complexity.

**Learn more about [Binary Search](/problems/binary_search_intro), [Dynamic Programming](/problems/dynamic_programming_intro) and [Sorting](/problems/sorting_summary) patterns.**

## Solution Approach

The implementation follows a [dynamic programming](/problems/dynamic_programming_intro) approach with memoization and [binary search](/problems/binary_search_intro) optimization.

**Step 1: Sort the events**
We begin by [sorting](/problems/sorting_summary) all events by their start time in ascending order. This allows us to process events chronologically and enables efficient [binary search](/problems/binary_search_intro) later.

```python
events.sort()
```

**Step 2: Define the recursive function with memoization**
We create a recursive function `dfs(i, k)` that computes the maximum value obtainable starting from event `i` with at most `k` events remaining to attend. The `@cache` decorator provides automatic memoization to store previously computed results.

```python
@cache
def dfs(i: int, k: int) -> int:
```

**Step 3: Handle base case**
If we've processed all events (i.e., `i >= len(events)`), there's no value to gain, so we return 0.

```python
if i >= len(events):
    return 0
```

**Step 4: Extract current event details**
We unpack the current event's information: start day (which we don't need here), end day, and value.

```python
_, ed, val = events[i]
```

**Step 5: Calculate value if we skip the current event**
First, we consider not attending event `i`, which means moving to the next event with the same capacity:

```python
ans = dfs(i + 1, k)
```

**Step 6: Calculate value if we attend the current event**
If we still have capacity (`k > 0`), we can consider attending the current event. We need to find the next event we can attend after this one ends.

Using `bisect_right` with a custom key function, we find the first event whose start time is strictly greater than the current event's end time `ed`. The search starts from `i + 1` to avoid reconsidering earlier events:

```python
if k:
    j = bisect_right(events, ed, lo=i + 1, key=lambda x: x[0])
    ans = max(ans, dfs(j, k - 1) + val)
```

The `key=lambda x: x[0]` tells `bisect_right` to compare against the start times of events. The function returns the index `j` of the first event that starts after day `ed`.

**Step 7: Return the maximum value**
We return the maximum between skipping and attending the current event:

```python
return ans
```

**Step 8: Initiate the recursion**
Finally, we start the recursion from the first event (index 0) with the full capacity of `k` events:

```python
return dfs(0, k)
```

The time complexity is `O(n * k * log n)` where `n` is the number of events. We have `n * k` possible states in our memoization, and for each state, we perform a [binary search](/problems/binary_search_intro) that takes `O(log n)` time. The space complexity is `O(n * k)` for the memoization cache.

## Example Walkthrough

Let's walk through a concrete example to illustrate the solution approach.

**Example Input:**
- `events = [[1,2,4], [3,4,3], [2,3,1]]`
- `k = 2` (we can attend at most 2 events)

**Step 1: Sort events by start time**
After sorting: `events = [[1,2,4], [2,3,1], [3,4,3]]`

**Step 2: Begin recursion with dfs(0, 2)**

Starting at event 0 `[1,2,4]` with k=2:
- Option 1: Skip event 0 → `dfs(1, 2)`
- Option 2: Attend event 0 → gain value 4, find next valid event
  - Event 0 ends on day 2
  - Using binary search, find first event starting after day 2
  - Event at index 1 starts on day 2 (not valid, same day)
  - Event at index 2 starts on day 3 (valid!)
  - So: `4 + dfs(2, 1)`

**Step 3: Evaluate dfs(1, 2)**

At event 1 `[2,3,1]` with k=2:
- Option 1: Skip event 1 → `dfs(2, 2)`
- Option 2: Attend event 1 → gain value 1
  - Event 1 ends on day 3
  - Binary search finds no events starting after day 3
  - So: `1 + dfs(3, 1)` = `1 + 0` = 1

**Step 4: Evaluate dfs(2, 2)**

At event 2 `[3,4,3]` with k=2:
- Option 1: Skip event 2 → `dfs(3, 2)` = 0 (no more events)
- Option 2: Attend event 2 → gain value 3
  - Event 2 ends on day 4
  - No events after this
  - So: `3 + dfs(3, 1)` = `3 + 0` = 3

Therefore, `dfs(2, 2) = max(0, 3) = 3`

**Step 5: Back to dfs(1, 2)**
`dfs(1, 2) = max(dfs(2, 2), 1 + dfs(3, 1)) = max(3, 1) = 3`

**Step 6: Evaluate dfs(2, 1)**

At event 2 `[3,4,3]` with k=1:
- Option 1: Skip → 0
- Option 2: Attend → `3 + dfs(3, 0)` = 3

Therefore, `dfs(2, 1) = 3`

**Step 7: Back to dfs(0, 2)**
`dfs(0, 2) = max(dfs(1, 2), 4 + dfs(2, 1)) = max(3, 4 + 3) = 7`

**Final Answer: 7**

We attend event 0 (days 1-2, value 4) and event 2 (days 3-4, value 3) for a total value of 7.

The memoization cache stores computed values for state combinations like (0,2), (1,2), (2,2), (2,1), preventing redundant calculations when the same subproblem is encountered again.

## Python Solution

```python
class Solution:
    def maxValue(self, events: List[List[int]], k: int) -> int:
        """
        Find maximum value by attending at most k non-overlapping events.

        Args:
            events: List of events where each event is [start_day, end_day, value]
            k: Maximum number of events that can be attended

        Returns:
            Maximum total value achievable
        """
        from functools import cache
        import bisect

        @cache
        def dp(index: int, remaining_events: int) -> int:
            """
            Dynamic programming function to find max value from index onwards.

            Args:
                index: Current index in sorted events list
                remaining_events: Number of events we can still attend

            Returns:
                Maximum value achievable from current state
            """
            # Base case: no more events to consider
            if index >= len(events):
                return 0

            # Extract current event details
            start_day, end_day, value = events[index]

            # Option 1: Skip current event
            max_value = dp(index + 1, remaining_events)

            # Option 2: Take current event (if we have remaining capacity)
            if remaining_events > 0:
                # Find next non-overlapping event using binary search
                # We need to find first event that starts after current event ends
                next_valid_index = bisect.bisect_right(
                    events,
                    end_day,
                    lo=index + 1,
                    key=lambda event: event[0]  # Compare by start day
                )

                # Take current event and continue from next valid index
                max_value = max(
                    max_value,
                    dp(next_valid_index, remaining_events - 1) + value
                )

            return max_value

        # Sort events by start day for binary search to work correctly
        events.sort()

        # Start recursion from first event with k available selections
        return dp(0, k)
```

## Time and Space Complexity

The time complexity is `O(n × log n + n × k)`, where `n` is the number of events.

- The initial sorting of events takes `O(n × log n)` time.
- The recursive DFS function with memoization has at most `n × k` unique states (each combination of index `i` from 0 to n-1 and remaining count `k` from 0 to k).
- For each state, we perform:
  - A binary search using `bisect_right` which takes `O(log n)` time
  - Other operations that take `O(1)` time
- Since the binary search is performed at most once per state due to memoization, and we have `O(n × k)` states, the total time for all DFS calls is `O(n × k × log n)`.
- However, examining the recursion pattern more carefully: each state `(i, k)` either skips the current event or takes it and jumps to a future index. The binary search is only performed when we choose to take an event (when `k > 0`), which happens at most `k` times along any path. With memoization, each state is computed once, giving us `O(n × k)` for the DFS traversal plus the binary searches within.
- Combined with sorting: `O(n × log n + n × k)`

The space complexity is `O(n × k)`, where `n` is the number of events.

- The `@cache` decorator (memoization) stores up to `n × k` unique states in memory.
- The recursion stack depth is at most `O(n)` in the worst case when we skip all events.
- Since `n × k` dominates, the overall space complexity is `O(n × k)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Binary Search for Next Valid Event**

One of the most common mistakes is incorrectly finding the next non-overlapping event after attending the current one. Many developers make these errors:

**Pitfall Example:**
```python
# WRONG: Using bisect_left instead of bisect_right
next_valid_index = bisect.bisect_left(events, end_day, lo=index + 1, key=lambda x: x[0])

# WRONG: Searching for end_day + 1 (off-by-one error)
next_valid_index = bisect.bisect_left(events, end_day + 1, lo=index + 1, key=lambda x: x[0])
```

**Why it's wrong:**
- `bisect_left` finds the leftmost position where `end_day` could be inserted, which might give us an event that starts exactly on `end_day`. Since the problem states that the end day is inclusive (you cannot attend two events where one starts on the same day another ends), this would create an overlap.
- Using `end_day + 1` with `bisect_left` seems logical but is unnecessary and can cause confusion. The cleaner approach is using `bisect_right` with `end_day`.

**Correct Solution:**
```python
# Find the first event that starts AFTER end_day (strictly greater than)
next_valid_index = bisect.bisect_right(events, end_day, lo=index + 1, key=lambda x: x[0])
```

### 2. **Forgetting to Sort Events Before Processing**

**Pitfall Example:**
```python
def maxValue(self, events: List[List[int]], k: int) -> int:
    # WRONG: Forgot to sort events!
    # events.sort()  <- Missing this line

    @cache
    def dp(index: int, remaining: int) -> int:
        # ... rest of the code
```

**Why it's wrong:** The binary search relies on the events being sorted by start time. Without sorting, the binary search will produce incorrect results, leading to wrong answers or even infinite recursion in some cases.

**Correct Solution:**
```python
# Always sort events by start time before processing
events.sort()  # This sorts by start_day first (default behavior for lists)
```

### 3. **Incorrect Memoization State**

**Pitfall Example:**
```python
# WRONG: Including unnecessary information in memoization state
@cache
def dp(index: int, remaining: int, last_end_day: int) -> int:
    # Adding last_end_day is redundant since we use binary search
```

**Why it's wrong:** Including `last_end_day` in the state makes the solution less efficient. Since events are sorted and we use binary search to find the next valid event, we don't need to track the last end day explicitly. This would increase both time and space complexity unnecessarily.

**Correct Solution:**
```python
@cache
def dp(index: int, remaining_events: int) -> int:
    # Only track current index and remaining capacity
```

### 4. **Off-by-One Error in Base Cases**

**Pitfall Example:**
```python
# WRONG: Checking remaining_events before recursion
if remaining_events == 0:
    return 0

# WRONG: Using > instead of >= for index check
if index > len(events):
    return 0
```

**Why it's wrong:**
- Checking `remaining_events == 0` at the start prevents us from exploring the "skip" option, which might lead to a better solution later.
- Using `>` instead of `>=` would cause an index out of bounds error.

**Correct Solution:**
```python
# Check index bounds first
if index >= len(events):
    return 0

# Check remaining_events only when considering taking an event
if remaining_events > 0:
    # Try taking the current event
```

