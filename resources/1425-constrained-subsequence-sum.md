# 1425. Constrained Subsequence Sum

> Source: https://algo.monster/liteproblems/1425

## Problem Statement

You are given an integer array `nums` and an integer `k`. Your task is to find the maximum sum of a **non-empty** subsequence from this array with a special constraint.

The constraint is: for any two **consecutive** elements in your chosen subsequence, if they are at positions `i` and `j` in the original array (where `i < j`), then the distance between them must satisfy `j - i <= k`. In other words, you cannot pick elements that are more than `k` positions apart in the original array to be consecutive in your subsequence.

A subsequence means you can select any elements from the array while maintaining their original relative order. You can skip elements, but you cannot rearrange them.

For example:
- If `nums = [10, -2, -10, -5, 20]` and `k = 2`
- You could pick subsequence `[10, -2, 20]` since the consecutive pairs `(10, -2)` are 1 position apart and `(-2, 20)` are 2 positions apart, both satisfying the constraint `<= 2`
- You could NOT pick subsequence `[10, 20]` directly since they are 4 positions apart in the original array, which violates the constraint `j - i <= k = 2`

The goal is to find the subsequence with the maximum possible sum while respecting this distance constraint between consecutive elements in the subsequence.

## Intuition

Let's think about this problem step by step. We need to find a subsequence with maximum sum, where consecutive elements in the subsequence can't be more than `k` positions apart in the original array.

First insight: This is a [dynamic programming](/problems/dynamic_programming_intro) problem. For each position `i`, we need to decide whether to include `nums[i]` in our subsequence or not. If we include it, we need to connect it to some previous element within the allowed range.

Let's define `f[i]` as the maximum sum of a subsequence that **ends at position i**. To calculate `f[i]`, we have two choices:
1. Start a new subsequence from position `i` (just take `nums[i]` alone)
2. Extend a previous subsequence by adding `nums[i]` to it

If we choose option 2, which previous subsequence should we extend? We can only extend from positions `j` where `i - j <= k`. Among all valid positions, we want the one with the maximum `f[j]` value to maximize our sum.

So the recurrence relation becomes:
`f[i] = nums[i] + max(0, max(f[j]) for all j where i - k <= j < i)`

The `max(0, ...)` part handles the case where it's better to start fresh rather than extend any negative-sum subsequence.

Now here's the key optimization insight: Finding the maximum `f[j]` in the range `[i-k, i-1]` for each `i` is essentially a **[sliding window](/problems/sliding_window_maximum) maximum problem**. As we move from position `i` to `i+1`, the window shifts by one position.

This is where the monotonic [queue](/problems/queue_intro) comes in! A monotonic decreasing queue can efficiently maintain the maximum value in a [sliding window](/problems/sliding_window_maximum). The queue stores indices, and we maintain the property that `f[queue[0]] >= f[queue[1]] >= ... >= f[queue[back]]`.

When processing position `i`:
- Remove indices from the front that are outside the window (more than `k` positions away)
- The front of the [queue](/problems/queue_intro) gives us the index with maximum `f` value in the valid range
- After calculating `f[i]`, add `i` to the queue while maintaining the decreasing property

This optimization reduces the time complexity from `O(n*k)` to `O(n)`, making the solution efficient even for large inputs.

**Learn more about [Queue](/problems/queue_intro), [Dynamic Programming](/problems/dynamic_programming_intro), [Sliding Window](/problems/sliding_window_maximum), [Monotonic Queue](/problems/mono_stack_intro) and [Heap (Priority Queue)](/problems/heap_intro) patterns.**

## Solution Approach

Let's walk through the implementation step by step using [dynamic programming](/problems/dynamic_programming_intro) with a monotonic [queue](/problems/queue_intro) optimization.

**Step 1: Initialize Data Structures**

We create:
- A deque `q` to maintain our monotonic [queue](/problems/queue_intro), initialized with `[0]` as a sentinel value
- An array `f` of size `n` to store our DP values, where `f[i]` represents the maximum sum of subsequence ending at index `i`
- Variable `ans` to track the maximum sum found, initialized to negative infinity

**Step 2: Process Each Element**

For each position `i` and value `x` in `nums`:

**2.1 Maintain the [Sliding Window](/problems/sliding_window_maximum)**
```python
while i - q[0] > k:
    q.popleft()
```
We remove indices from the front of the [queue](/problems/queue_intro) that are outside our window of size `k`. Any index `j` where `i - j > k` is too far away to be considered.

**2.2 Calculate DP Value**
```python
f[i] = max(0, f[q[0]]) + x
```
The front of the [queue](/problems/queue_intro) `q[0]` gives us the index with the maximum `f` value within the valid range `[i-k, i-1]`. We calculate:
- `f[i] = f[q[0]] + nums[i]` if we extend the best previous subsequence
- `f[i] = nums[i]` if it's better to start fresh (when `f[q[0]] < 0`)

This is why we use `max(0, f[q[0]])` - it handles both cases elegantly.

**2.3 Update the Answer**
```python
ans = max(ans, f[i])
```
We keep track of the maximum subsequence sum seen so far.

**2.4 Maintain Monotonic Property**
```python
while q and f[q[-1]] <= f[i]:
    q.pop()
q.append(i)
```
Before adding index `i` to the [queue](/problems/queue_intro), we remove all indices from the back whose `f` values are less than or equal to `f[i]`. This maintains the decreasing monotonic property of the queue.

Why can we safely remove these elements? If `f[j] <= f[i]` and `j < i`, then for any future position, `i` will always be a better choice than `j` because:
- `i` is closer (less likely to fall outside the window)
- `f[i]` is greater or equal in value

**Step 3: Return Result**

After processing all elements, `ans` contains the maximum sum of any valid subsequence.

**Time Complexity:** `O(n)` - Each element is added and removed from the [queue](/problems/queue_intro) at most once.

**Space Complexity:** `O(n)` - We use an array `f` of size `n` and a [queue](/problems/queue_intro) that can contain at most `min(n, k+1)` elements.

## Example Walkthrough

Let's trace through the algorithm with `nums = [10, -2, -10, -5, 20]` and `k = 2`.

**Initialization:**
- `q = deque([0])` (sentinel value)
- `f = [0, 0, 0, 0, 0]` (size 5)
- `ans = -inf`

**Processing i = 0, x = 10:**
- Window check: `0 - q[0] = 0 - 0 = 0 ≤ 2` ✓ (no removal needed)
- Calculate: `f[0] = max(0, f[0]) + 10 = max(0, 0) + 10 = 10`
- Update answer: `ans = max(-inf, 10) = 10`
- Maintain queue: `f[0] = 0 ≤ f[0] = 10`, so pop 0
- Queue becomes: `q = [0]` (index 0 for nums[0])

**Processing i = 1, x = -2:**
- Window check: `1 - 0 = 1 ≤ 2` ✓ (no removal)
- Calculate: `f[1] = max(0, f[0]) + (-2) = max(0, 10) + (-2) = 8`
- Update answer: `ans = max(10, 8) = 10`
- Maintain queue: `f[0] = 10 > f[1] = 8`, so keep 0
- Queue becomes: `q = [0, 1]`

**Processing i = 2, x = -10:**
- Window check: `2 - 0 = 2 ≤ 2` ✓ (no removal)
- Front of queue is 0, so `f[2] = max(0, f[0]) + (-10) = 10 + (-10) = 0`
- Update answer: `ans = max(10, 0) = 10`
- Maintain queue: Remove 1 since `f[1] = 8 ≤ f[2] = 0` is false, keep both
- Actually, `f[1] = 8 > f[2] = 0`, so keep 1
- Queue becomes: `q = [0, 1, 2]`

**Processing i = 3, x = -5:**
- Window check: `3 - 0 = 3 > 2` ✗ (remove 0 from front)
- Queue becomes: `q = [1, 2]`
- Front is now 1, so `f[3] = max(0, f[1]) + (-5) = 8 + (-5) = 3`
- Update answer: `ans = max(10, 3) = 10`
- Maintain queue: `f[2] = 0 ≤ f[3] = 3`, so pop 2
- `f[1] = 8 > f[3] = 3`, so keep 1
- Queue becomes: `q = [1, 3]`

**Processing i = 4, x = 20:**
- Window check: `4 - 1 = 3 > 2` ✗ (remove 1 from front)
- Queue becomes: `q = [3]`
- Front is 3, so `f[4] = max(0, f[3]) + 20 = 3 + 20 = 23`
- Update answer: `ans = max(10, 23) = 23`
- Maintain queue: `f[3] = 3 ≤ f[4] = 23`, so pop 3
- Queue becomes: `q = [4]`

**Final Result: 23**

This corresponds to the subsequence `[10, -2, -5, 20]`:
- 10 at index 0
- -2 at index 1 (distance from 0: 1 ≤ 2) ✓
- -5 at index 3 (distance from 1: 2 ≤ 2) ✓  
- 20 at index 4 (distance from 3: 1 ≤ 2) ✓
- Sum: 10 + (-2) + (-5) + 20 = 23

The algorithm correctly identifies that connecting through the negative values is worth it to reach the large positive value at the end.

## Python Solution

```python
class Solution:
    def constrainedSubsetSum(self, nums: List[int], k: int) -> int:
        # Monotonic deque to maintain indices of maximum dp values within window
        indices_deque = deque([0])
        n = len(nums)
      
        # dp[i] represents the maximum sum of subsequence ending at index i
        dp = [0] * n
      
        # Track the maximum sum found so far
        max_sum = float('-inf')
      
        # Process each element in the array
        for current_index, current_value in enumerate(nums):
            # Remove indices that are outside the k-distance window
            while indices_deque and current_index - indices_deque[0] > k:
                indices_deque.popleft()
          
            # Calculate dp value for current position
            # Either start fresh from current element or extend from best previous subsequence
            dp[current_index] = max(0, dp[indices_deque[0]]) + current_value
          
            # Update the maximum sum encountered
            max_sum = max(max_sum, dp[current_index])
          
            # Maintain monotonic decreasing property of deque
            # Remove elements with smaller or equal dp values from the back
            while indices_deque and dp[indices_deque[-1]] <= dp[current_index]:
                indices_deque.pop()
          
            # Add current index to the deque
            indices_deque.append(current_index)
      
        return max_sum
```

## Time and Space Complexity

**Time Complexity:** `O(n)`

The algorithm iterates through the array once with a for loop that runs `n` times. Within each iteration:
- The first while loop removes outdated indices from the deque. Each element can be removed at most once throughout the entire execution, contributing `O(n)` total operations across all iterations.
- The second while loop maintains the monotonic property of the deque by removing elements. Similarly, each element can be added and removed from the deque at most once, contributing `O(n)` total operations.
- All other operations (max comparisons, deque append, array access) are `O(1)`.

Since each element is processed a constant number of times despite the nested loops, the overall time complexity is `O(n)`.

**Space Complexity:** `O(n)`

The algorithm uses:
- An array `f` of size `n` to store the maximum sum ending at each position: `O(n)`
- A deque `q` that can contain at most `min(k+1, n)` elements, which is bounded by `O(n)`
- A few constant space variables (`ans`, `i`, `x`): `O(1)`

The dominant space usage comes from the array `f`, resulting in a total space complexity of `O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. Incorrect Initialization of the Deque

**Pitfall:** Starting with an empty deque or initializing it with `[0]` when processing starts from index 0 can cause issues.

When we reach the first element (index 0), if the deque is initialized with `[0]`, the code tries to access `dp[0]` before it's been set, potentially causing incorrect calculations or errors.

**Solution:** Initialize the deque as empty and handle the first element specially, or ensure proper bounds checking:

```python
def constrainedSubsetSum(self, nums: List[int], k: int) -> int:
    indices_deque = deque()  # Start with empty deque
    n = len(nums)
    dp = [0] * n
    max_sum = float('-inf')
  
    for current_index, current_value in enumerate(nums):
        # Remove indices outside window
        while indices_deque and current_index - indices_deque[0] > k:
            indices_deque.popleft()
      
        # Handle first element or when deque is empty
        if not indices_deque:
            dp[current_index] = current_value
        else:
            dp[current_index] = max(0, dp[indices_deque[0]]) + current_value
      
        max_sum = max(max_sum, dp[current_index])
      
        # Maintain monotonic property
        while indices_deque and dp[indices_deque[-1]] <= dp[current_index]:
            indices_deque.pop()
      
        indices_deque.append(current_index)
  
    return max_sum
```

### 2. Forgetting to Handle Negative Values Properly

**Pitfall:** Not using `max(0, dp[indices_deque[0]])` and instead directly using `dp[indices_deque[0]]` can lead to suboptimal solutions when dealing with negative values.

If the best previous subsequence has a negative sum, it's better to start fresh from the current element rather than extending that negative subsequence.

**Wrong approach:**
```python
dp[current_index] = dp[indices_deque[0]] + current_value  # Wrong!
```

**Correct approach:**
```python
dp[current_index] = max(0, dp[indices_deque[0]]) + current_value  # Correct!
```

This ensures that we either extend a positive subsequence or start fresh if all previous options are negative.

### 3. Using Strict Inequality in Monotonic Queue Maintenance

**Pitfall:** Using strict inequality (`<`) instead of `<=` when maintaining the monotonic property:

```python
# Wrong - can lead to keeping redundant elements
while indices_deque and dp[indices_deque[-1]] < dp[current_index]:
    indices_deque.pop()
```

**Why it matters:** If we keep elements with equal dp values, we're storing redundant information. Since the current index is more recent (and thus will stay in the window longer), it's always preferable to indices with equal dp values.

**Correct approach:**
```python
while indices_deque and dp[indices_deque[-1]] <= dp[current_index]:
    indices_deque.pop()
```

