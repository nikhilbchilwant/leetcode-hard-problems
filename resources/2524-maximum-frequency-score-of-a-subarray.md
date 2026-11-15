# 2524. Maximum Frequency Score of a Subarray

> Source: https://algo.monster/liteproblems/2524

## Problem Statement

You have an integer array `nums` and a positive integer `k`. Your task is to find a contiguous subarray of exactly size `k` that has the maximum possible frequency score.

The **frequency score** of an array is calculated as follows:
1. Count the frequency of each distinct value in the array
2. For each distinct value, raise it to the power of its frequency
3. Sum all these powers together
4. Take the result modulo `10^9 + 7`

For example, given the array `[5,4,5,7,4,4]`:
- The value `4` appears 3 times, contributing `4^3 = 64`
- The value `5` appears 2 times, contributing `5^2 = 25`
- The value `7` appears 1 time, contributing `7^1 = 7`
- The frequency score is `(64 + 25 + 7) mod (10^9 + 7) = 96`

You need to examine all possible subarrays of length `k` in `nums` and return the maximum frequency score among them. Note that you should maximize the value after applying the modulo operation, not the actual mathematical value before modulo.

A subarray is a contiguous sequence of elements from the original array, meaning the elements must appear consecutively in their original order.

## Intuition

The naive approach would be to calculate the frequency score for every possible subarray of size `k`, but this would be inefficient since we'd be recounting frequencies from scratch for each window.

The key insight is that consecutive windows of size `k` share most of their elements. When we slide the window one position to the right, we only remove one element from the left and add one element from the right. This means we can maintain the frequency score incrementally rather than recalculating it entirely.

Consider what happens to the frequency score when we modify a single element's frequency:
- When an element `x` with frequency `f` gets its frequency increased to `f+1`, the contribution changes from `x^f` to `x^(f+1)`. The difference is `x^(f+1) - x^f = x^f * (x - 1)`.
- Similarly, when frequency decreases from `f` to `f-1`, the contribution changes by `x^(f-1) * (x - 1)` in the opposite direction.

This observation allows us to update the score efficiently. When sliding the window:
1. If we remove element `a` and add element `b`, and they're different:
   - The frequency of `a` decreases by 1
   - The frequency of `b` increases by 1
   - We can update the current score by adjusting these two contributions

We can maintain a hash table to track frequencies and use fast power computation with modulo to handle the large numbers. Starting with the first window of size `k`, we calculate its initial score, then slide the window through the array, updating the score incrementally and keeping track of the maximum score encountered.

The [sliding window](/problems/sliding_window_maximum) technique combined with incremental updates reduces the time complexity significantly compared to recalculating the score for each window from scratch.

**Learn more about [Stack](/problems/stack_intro), [Math](/problems/math-basics) and [Sliding Window](/problems/sliding_window_maximum) patterns.**

## Solution Approach

The solution uses a **hash table with [sliding window](/problems/sliding_window_maximum) technique** combined with **fast power computation**.

**Step 1: Initialize the first window**
- Create a Counter (hash table) `cnt` to store the frequency of each element in the first `k` elements
- Calculate the initial frequency score using the formula: `sum(pow(k, v, mod) for k, v in cnt.items())`
- Store this as both the current score `cur` and the maximum score `ans`

**Step 2: Slide the window through the array**
Starting from index `k`, process each element by:
- Identifying the element to remove `a = nums[i - k]` (leftmost element of previous window)
- Identifying the element to add `b = nums[i]` (new rightmost element)

**Step 3: Update the score incrementally**
When `a != b` (only need to update when they're different):

For adding element `b`:
- If `cnt[b] = 0` (new element): contribution is simply `b^1 = b`
- If `cnt[b] > 0` (element already exists): the contribution changes from `b^cnt[b]` to `b^(cnt[b]+1)`
- The difference is: `b^(cnt[b]+1) - b^cnt[b] = b^cnt[b] * (b - 1)`
- Update: `cur += (b - 1) * pow(b, cnt[b], mod) if cnt[b] else b`

For removing element `a`:
- If `cnt[a] = 1` (last occurrence): we remove contribution of `a^1 = a`
- If `cnt[a] > 1` (multiple occurrences): the contribution changes from `a^cnt[a]` to `a^(cnt[a]-1)`
- The difference is: `a^cnt[a] - a^(cnt[a]-1) = a^(cnt[a]-1) * (a - 1)`
- Update: `cur -= (a - 1) * pow(a, cnt[a] - 1, mod) if cnt[a] > 1 else a`

**Step 4: Update frequencies and track maximum**
- Increment `cnt[b]` by 1
- Decrement `cnt[a]` by 1
- Apply modulo to keep `cur` within bounds: `cur %= mod`
- Update the maximum score: `ans = max(ans, cur)`

**Step 5: Return the result**
After sliding through all possible windows, return `ans` as the maximum frequency score.

The use of fast power with modulo (`pow(base, exp, mod)`) ensures efficient computation while preventing integer overflow. The [sliding window](/problems/sliding_window_maximum) approach reduces time complexity from O(n*k) to O(n) for the window updates.

## Example Walkthrough

Let's walk through the solution with `nums = [1, 1, 2, 2, 3]` and `k = 3`.

**Step 1: Initialize the first window [1, 1, 2]**
- Build frequency counter: `cnt = {1: 2, 2: 1}`
- Calculate initial score:
  - Element 1 appears 2 times: contribution = `1^2 = 1`
  - Element 2 appears 1 time: contribution = `2^1 = 2`
  - Initial score `cur = 1 + 2 = 3`
- Set `ans = 3`

**Step 2: Slide to window [1, 2, 2] (remove nums[0]=1, add nums[3]=2)**
- Element to remove: `a = 1` (currently has frequency 2)
- Element to add: `b = 2` (currently has frequency 1)
- Since `a ≠ b`, we update incrementally:

For adding `b = 2`:
- Current frequency of 2 is 1, will become 2
- Old contribution: `2^1 = 2`
- New contribution: `2^2 = 4`
- Change: `(2 - 1) * 2^1 = 1 * 2 = 2`
- Update: `cur += 2`

For removing `a = 1`:
- Current frequency of 1 is 2, will become 1
- Old contribution: `1^2 = 1`
- New contribution: `1^1 = 1`
- Change: `(1 - 1) * 1^1 = 0`
- Update: `cur -= 0`

- New score: `cur = 3 + 2 - 0 = 5`
- Update frequencies: `cnt = {1: 1, 2: 2}`
- Update maximum: `ans = max(3, 5) = 5`

**Step 3: Slide to window [2, 2, 3] (remove nums[1]=1, add nums[4]=3)**
- Element to remove: `a = 1` (currently has frequency 1)
- Element to add: `b = 3` (currently has frequency 0, new element)

For adding `b = 3`:
- New element, frequency 0 → 1
- Contribution: `3^1 = 3`
- Update: `cur += 3`

For removing `a = 1`:
- Last occurrence, frequency 1 → 0
- Removes contribution: `1^1 = 1`
- Update: `cur -= 1`

- New score: `cur = 5 + 3 - 1 = 7`
- Update frequencies: `cnt = {1: 0, 2: 2, 3: 1}`
- Update maximum: `ans = max(5, 7) = 7`

**Result:** The maximum frequency score is 7, achieved by the subarray [2, 2, 3] where:
- Element 2 appears 2 times: `2^2 = 4`
- Element 3 appears 1 time: `3^1 = 3`
- Total: `4 + 3 = 7`

## Python Solution

```python
class Solution:
    def maxFrequencyScore(self, nums: List[int], k: int) -> int:
        # Define modulo constant for large number handling
        MOD = 10**9 + 7

        # Initialize frequency counter for the first window of size k
        frequency_map = Counter(nums[:k])

        # Calculate initial score: sum of (value^frequency) for each unique value
        current_score = sum(pow(value, freq, MOD) for value, freq in frequency_map.items()) % MOD
        max_score = current_score

        # Slide the window through the remaining elements
        for i in range(k, len(nums)):
            # Get the element leaving the window and the element entering the window
            leaving_element = nums[i - k]
            entering_element = nums[i]

            # Only update if the elements are different
            if leaving_element != entering_element:
                # Update score for the entering element
                # If element already exists: add (value - 1) * value^(old_frequency)
                # If new element: add value
                if frequency_map[entering_element]:
                    current_score += (entering_element - 1) * pow(entering_element, frequency_map[entering_element], MOD)
                else:
                    current_score += entering_element

                # Update score for the leaving element
                # If frequency > 1: subtract (value - 1) * value^(new_frequency)
                # If frequency becomes 0: subtract value
                if frequency_map[leaving_element] > 1:
                    current_score -= (leaving_element - 1) * pow(leaving_element, frequency_map[leaving_element] - 1, MOD)
                else:
                    current_score -= leaving_element

                # Apply modulo to keep number manageable
                current_score %= MOD

                # Update frequency map
                frequency_map[entering_element] += 1
                frequency_map[leaving_element] -= 1

                # Track maximum score
                max_score = max(max_score, current_score)

        return max_score
```

## Time and Space Complexity

The time complexity is `O(n × k)`, where `n` is the length of the array `nums` and `k` is the window size. This is because:
- The initial window setup takes `O(k)` time to process the first `k` elements
- The main while loop runs `n - k` times
- In each iteration, the `pow(b, cnt[b], mod)` and `pow(a, cnt[a] - 1, mod)` operations take `O(log(cnt[b]))` and `O(log(cnt[a]))` time respectively
- In the worst case, `cnt[b]` or `cnt[a]` can be at most `k`, making each power operation `O(log k)`
- Therefore, the overall time complexity is `O(k) + O((n - k) × log k) = O(n × log k)`

However, upon closer examination, the initial computation `sum(pow(k, v, mod) for k, v in cnt.items())` uses variable `k` incorrectly (it shadows the parameter `k`). This should be `sum(pow(key, v, mod) for key, v in cnt.items())`. Each `pow(key, v, mod)` operation takes `O(log v)` time, and in the worst case `v` can be `k`, so this initial computation takes `O(k × log k)` time.

Since `k ≤ n`, the overall time complexity simplifies to `O(n × log n)` when considering `k` can be as large as `n`.

The space complexity is `O(n)` because:
- The `Counter` object `cnt` can store at most `min(k, n)` unique elements, which is `O(n)` in the worst case
- Other variables use constant space

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Modulo Application During Incremental Updates**

One of the most common mistakes is forgetting that subtraction with modulo arithmetic can produce negative results. When we subtract a value from `current_score`, the result might become negative, and applying `%` operator in Python on a negative number doesn't give the expected result in modular arithmetic.

**Problem Example:**
```python
# If current_score = 5 and we subtract 10
current_score = 5
current_score -= 10  # current_score = -5
current_score %= MOD  # This gives 10^9 + 2, not the mathematically correct value
```

**Solution:**
Always ensure the result is positive before applying modulo, or add MOD before taking modulo:
```python
current_score = (current_score - value + MOD) % MOD
```

### 2. **Integer Overflow in Power Calculation**

When computing `(entering_element - 1) * pow(entering_element, frequency_map[entering_element], MOD)`, the multiplication happens after the power calculation. Even though `pow` returns a value less than MOD, multiplying it by `(entering_element - 1)` can cause overflow if the values are large.

**Solution:**
Apply modulo after each multiplication:
```python
contribution = ((entering_element - 1) * pow(entering_element, frequency_map[entering_element], MOD)) % MOD
current_score = (current_score + contribution) % MOD
```

### 3. **Not Handling Zero Frequencies Properly**

The code checks `if frequency_map[entering_element]` to determine if an element is new. However, after decrementing, elements with zero frequency remain in the Counter, which could lead to incorrect calculations if not handled properly.

**Solution:**
Either remove elements with zero frequency or consistently check for zero values:
```python
if frequency_map[leaving_element] == 1:
    del frequency_map[leaving_element]  # Remove from map entirely
else:
    frequency_map[leaving_element] -= 1
```

### 4. **Forgetting Edge Cases in Window Updates**

A subtle bug can occur if we don't handle the case where the same element is both entering and leaving the window (when `leaving_element == entering_element`). The current code correctly skips updates in this case, but developers might accidentally process both additions and removals, leading to incorrect scores.

**Solution:**
Always check if the entering and leaving elements are the same before making any updates:
```python
if leaving_element == entering_element:
    continue  # No change needed, skip to next iteration
```

### 5. **Inefficient Power Recalculation**

While the code uses incremental updates, some developers might accidentally recalculate the entire score for each window, which defeats the purpose of the sliding window optimization.

**Incorrect approach:**
```python
# Don't do this - O(k) per window
for i in range(k, len(nums)):
    frequency_map = Counter(nums[i-k+1:i+1])  # Recounting entire window
    current_score = sum(pow(v, f, MOD) for v, f in frequency_map.items())
```

**Correct approach:**
Use incremental updates as shown in the original solution to maintain O(1) updates per window movement.

