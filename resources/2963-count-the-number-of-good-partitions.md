# 2963. Count the Number of Good Partitions

> Source: https://algo.monster/liteproblems/2963

## Problem Statement

You have an array `nums` containing positive integers (0-indexed). Your task is to find how many ways you can partition this array into contiguous subarrays such that no two subarrays contain the same number.

A **partition** means dividing the array into one or more contiguous subarrays where:
- Each element belongs to exactly one subarray
- The subarrays cover the entire array
- The subarrays maintain the original order

A partition is considered **good** if no number appears in more than one subarray. In other words, if a number appears multiple times in the array, all its occurrences must be grouped together in the same subarray.

For example, if you have `[1, 2, 1, 3]`, you cannot split it as `[1, 2] | [1, 3]` because the number `1` would appear in two different subarrays. Instead, `[1, 2, 1] | [3]` would be valid since all occurrences of `1` are in the same subarray.

Your goal is to count the total number of different ways to create good partitions of the array. Since this count can be very large, return the result modulo `10^9 + 7`.

## Intuition

Let's think about what makes a partition "good". If a number appears multiple times in the array, all its occurrences must stay together in the same subarray. This is the key constraint.

For each number, we need to know where it last appears in the array. Why? Because if a number appears at index `i` and its last occurrence is at index `j`, then the segment from `i` to `j` must be kept together - we cannot split anywhere between them.

Consider the array `[1, 2, 3, 2, 1, 4]`. The number `1` appears at indices `0` and `4`, so indices `0` through `4` must be in the same subarray. The number `2` appears at indices `1` and `3`, which are already within the range `[0, 4]`. So the minimum segment that must stay together is `[0, 4]`.

As we traverse the array from left to right, we can track the rightmost boundary that we must reach before we can make a cut. When we reach this boundary (when current index equals the rightmost boundary), we've found a position where we *can* potentially make a partition.

Each such position represents a choice: we can either make a cut here or continue without cutting. If we identify `k` such positions where cuts are possible, we have `k` segments that could be independent subarrays.

Between `k` segments, there are `k-1` positions where we can choose to cut or not cut. Each position gives us 2 choices (cut or don't cut), so the total number of ways to partition is `2^(k-1)`.

The algorithm tracks:
- `last[x]`: the last occurrence index of each number `x`
- `j`: the furthest index we must reach before considering a cut
- `k`: the count of positions where we can potentially make a cut

When we encounter a number at index `i`, we update `j` to be the maximum of current `j` and the last occurrence of this number. If `i == j`, we've reached a boundary where all constraints are satisfied, and we can potentially partition here.

**Learn more about [Math](/problems/math-basics) and [Combinatorics](/problems/backtracking) patterns.**

## Solution Approach

The solution uses a hash table combined with a greedy approach to identify partition points, followed by fast power calculation to count the total number of valid partitions.

**Step 1: Build the Last Occurrence Map**

First, we create a hash table `last` that maps each number to its last occurrence index in the array:
```python
last = {x: i for i, x in enumerate(nums)}
```
This preprocessing step allows us to quickly look up where each number last appears, which is crucial for determining partition boundaries.

**Step 2: Identify Partition Points**

We traverse the array from left to right, maintaining two key variables:
- `j`: The maximum index we must reach before we can make a partition (initialized to `-1`)
- `k`: The count of positions where we can potentially partition

For each element at index `i`:
1. Update `j = max(j, last[nums[i]])` - this extends our current segment to include all occurrences of the current number
2. Check if `i == j` - if true, we've reached a point where all numbers seen so far have their last occurrences within this segment
3. When `i == j`, increment `k` by 1 as this is a valid partition point

```python
j, k = -1, 0
for i, x in enumerate(nums):
    j = max(j, last[x])
    k += i == j
```

**Step 3: Calculate Total Partitions**

After identifying `k` partition points, we have `k` segments. Between these `k` segments, there are `k-1` positions where we can choose to either make a cut or not. Each position gives us 2 choices, resulting in `2^(k-1)` total ways to partition the array.

We use Python's built-in `pow` function with three arguments for modular exponentiation:
```python
return pow(2, k - 1, mod)
```
This efficiently computes `2^(k-1) mod (10^9 + 7)` using fast power algorithm, avoiding overflow for large values of `k`.

**Time Complexity**: `O(n)` where `n` is the length of the array - we traverse the array twice (once to build the hash table, once to find partition points).

**Space Complexity**: `O(n)` for storing the hash table of last occurrences.

## Example Walkthrough

Let's walk through the solution with the array `nums = [1, 2, 3, 2, 1, 4]`.

**Step 1: Build Last Occurrence Map**

First, we create a hash table mapping each number to its last occurrence index:
- `1` appears at indices 0 and 4, so `last[1] = 4`
- `2` appears at indices 1 and 3, so `last[2] = 3`
- `3` appears at index 2, so `last[3] = 2`
- `4` appears at index 5, so `last[4] = 5`

Result: `last = {1: 4, 2: 3, 3: 2, 4: 5}`

**Step 2: Identify Partition Points**

Now we traverse the array to find valid partition points. We track:
- `j`: the furthest index we must reach (initially -1)
- `k`: count of partition points (initially 0)

| Index i | nums[i] | last[nums[i]] | j = max(j, last[nums[i]]) | i == j? | k |
|---------|---------|---------------|---------------------------|---------|---|
| 0       | 1       | 4             | max(-1, 4) = 4           | No      | 0 |
| 1       | 2       | 3             | max(4, 3) = 4            | No      | 0 |
| 2       | 3       | 2             | max(4, 2) = 4            | No      | 0 |
| 3       | 2       | 3             | max(4, 3) = 4            | No      | 0 |
| 4       | 1       | 4             | max(4, 4) = 4            | Yes     | 1 |
| 5       | 4       | 5             | max(4, 5) = 5            | Yes     | 2 |

At index 4, we have `i == j`, meaning we've included all occurrences of numbers 1, 2, and 3. This is our first partition point.

At index 5, we again have `i == j`, giving us a second partition point.

**Step 3: Calculate Total Partitions**

We found `k = 2` partition points, which means we have 2 segments:
- Segment 1: indices [0, 4] containing `[1, 2, 3, 2, 1]`
- Segment 2: index [5] containing `[4]`

Between 2 segments, there is 1 position where we can choose to cut or not cut.

Number of ways = `2^(k-1) = 2^(2-1) = 2^1 = 2`

The two valid partitions are:
1. Keep everything together: `[1, 2, 3, 2, 1, 4]`
2. Split after index 4: `[1, 2, 3, 2, 1] | [4]`

Both partitions are valid because in each case, all occurrences of each number remain in the same subarray. The answer is **2**.

## Python Solution

```python
class Solution:
    def numberOfGoodPartitions(self, nums: List[int]) -> int:
        # Create a dictionary mapping each element to its last occurrence index
        last_occurrence = {element: index for index, element in enumerate(nums)}
      
        # Define the modulo value for the result
        MOD = 10**9 + 7
      
        # Initialize variables:
        # max_reach: tracks the furthest index we need to include in current partition
        # partition_count: counts the number of partition boundaries
        max_reach = -1
        partition_count = 0
      
        # Iterate through the array
        for current_index, element in enumerate(nums):
            # Update max_reach to ensure all occurrences of current element are in same partition
            max_reach = max(max_reach, last_occurrence[element])
          
            # If current index equals max_reach, we've completed a mandatory partition
            if current_index == max_reach:
                partition_count += 1
      
        # Between k mandatory partitions, there are k-1 gaps where we can choose to split or not
        # Each gap has 2 choices (split or not split), giving us 2^(k-1) total combinations
        return pow(2, partition_count - 1, MOD)
```

## Time and Space Complexity

**Time Complexity:** `O(n)`

The algorithm consists of two main passes through the array:
1. Building the `last` dictionary using dictionary comprehension iterates through all `n` elements once, taking `O(n)` time
2. The main loop iterates through the array once more with `enumerate(nums)`, performing constant-time operations (`max()`, dictionary lookup, comparison, and increment) at each step, taking `O(n)` time
3. The final `pow(2, k-1, mod)` operation uses modular exponentiation which takes `O(log k)` time, and since `k ≤ n`, this is `O(log n)`

Overall time complexity: `O(n) + O(n) + O(log n) = O(n)`

**Space Complexity:** `O(n)`

The space usage comes from:
1. The `last` dictionary stores at most `n` key-value pairs (one for each unique element in `nums`), requiring `O(n)` space
2. Variables `j`, `k`, and `mod` use constant `O(1)` space

Overall space complexity: `O(n) + O(1) = O(n)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Handling of Edge Cases with Single Element Arrays**

**Pitfall**: Not properly handling arrays with only one element or arrays where all elements are the same. The formula `2^(k-1)` can produce incorrect results when `k = 0`.

**Example**: For `nums = [1]`, the code correctly identifies `k = 1` partition point, resulting in `2^(1-1) = 2^0 = 1`, which is correct. However, if the logic for counting partition points is slightly modified incorrectly, you might get `k = 0`, leading to `2^(-1)` which would cause an error or incorrect result.

**Solution**: Always ensure that at least one partition point is counted (the end of the array). The current implementation handles this correctly by checking `i == j` at each position.

### 2. **Off-by-One Error in Counting Partition Points**

**Pitfall**: Confusing the number of partition points with the number of partitions. If you have `k` partition points, you actually have `k` segments, but only `k-1` positions where you can choose to make or not make a cut.

**Incorrect approach**:
```python
# Wrong: Using k instead of k-1
return pow(2, partition_count, MOD)  # This would give 2^k instead of 2^(k-1)
```

**Solution**: Remember that between `k` mandatory segments, there are exactly `k-1` decision points. The formula should always be `2^(k-1)`.

### 3. **Integer Overflow When Computing Powers**

**Pitfall**: Using regular exponentiation without modulo can cause integer overflow for large values of `k`:
```python
# Wrong: Can cause overflow for large k
result = (2 ** (partition_count - 1)) % MOD
```

**Solution**: Always use modular exponentiation with three-argument `pow()`:
```python
# Correct: Efficiently computes (2^(k-1)) mod MOD
return pow(2, partition_count - 1, MOD)
```

### 4. **Misunderstanding the Partition Boundary Logic**

**Pitfall**: Incorrectly updating the `max_reach` variable or checking partition boundaries. A common mistake is resetting `max_reach` after finding a partition point:

```python
# Wrong: Resetting max_reach after each partition
for current_index, element in enumerate(nums):
    max_reach = max(max_reach, last_occurrence[element])
    if current_index == max_reach:
        partition_count += 1
        max_reach = -1  # WRONG: This would break the logic
```

**Solution**: Never reset `max_reach`. It should continuously track the furthest point we need to reach based on all elements seen so far. The variable naturally "resets" conceptually when we pass a partition boundary because the next element's last occurrence becomes the new constraint.

### 5. **Using Sets Instead of Last Occurrence Tracking**

**Pitfall**: Trying to track which elements have been "completed" using a set, which adds unnecessary complexity and can lead to incorrect results:

```python
# Wrong approach: Overly complex and error-prone
seen = set()
completed = set()
for i, x in enumerate(nums):
    seen.add(x)
    # Complex logic to track when all occurrences are seen...
```

**Solution**: Stick to the simple approach of tracking the last occurrence index. This single piece of information per element is sufficient to determine all partition boundaries.

