# 2386. Find the K-Sum of an Array

> Source: https://algo.monster/liteproblems/2386

## Problem Statement

You have an integer array `nums` and a positive integer `k`. Your task is to find the k-th largest sum among all possible subsequence sums of the array.

A subsequence is formed by selecting any number of elements from the array (including zero elements or all elements) while maintaining their original order. For example, if `nums = [1, 2, 3]`, then `[1, 3]`, `[2]`, `[]`, and `[1, 2, 3]` are all valid subsequences.

The **K-Sum** is defined as the k-th largest value when you list all possible subsequence sums in descending order. Note that:
- The empty subsequence has a sum of `0`
- The same sum value can appear multiple times if different subsequences produce it (they are counted separately in the ranking)

For instance, if `nums = [2, 4, -2]` and `k = 5`:
- All possible subsequence sums are: `6` (from `[2, 4]`), `4` (from `[4]`), `2` (from `[2]`), `2` (from `[4, -2]`), `0` (from `[]` or `[2, -2]`), `-2` (from `[-2]`)
- When sorted in descending order: `6, 4, 2, 2, 0, 0, -2`
- The 5th largest sum is `0`

Your goal is to return this k-th largest subsequence sum.

## Intuition

The key insight is to transform the problem from finding the k-th largest sum to finding the k-th smallest "reduction" from the maximum possible sum.

First, consider what the maximum possible subsequence sum would be - it's simply the sum of all positive numbers in the array. Let's call this `mx`. Any other subsequence sum can be viewed as `mx` minus some non-negative value (the sum of positive numbers we didn't take plus the absolute value of negative numbers we did take).

For example, if `nums = [3, -1, 2]`, the maximum sum is `3 + 2 = 5`. If we want the subsequence `[3, -1]`, its sum is `2`, which equals `5 - 3` where `3` is the cost of not taking `2` and taking `-1`.

This transformation is powerful because:
1. The 1st largest sum corresponds to the smallest reduction (0 reduction gives us `mx`)
2. The 2nd largest sum corresponds to the 2nd smallest reduction
3. The k-th largest sum corresponds to the k-th smallest reduction

Now we need to find the k-th smallest reduction efficiently. We can reformulate the array by replacing all negative numbers with their absolute values and [sorting](/problems/sorting_summary). This way, selecting elements from this transformed array represents the "cost" or "reduction" from the maximum sum.

To avoid generating all possible subsequences (which would be `2^n`), we use a min-heap with a clever expansion strategy. Starting with an empty subsequence (0 reduction), we expand by considering:
- Including the next element in our subsequence
- Swapping the last included element with the next element (excluding the previous and including the current)

This approach ensures we explore subsequence sums in increasing order of their reduction from `mx`, allowing us to find the k-th smallest reduction in `O(k log k)` time rather than exploring all possibilities.

**Learn more about [Sorting](/problems/sorting_summary) and [Heap (Priority Queue)](/problems/heap_intro) patterns.**

## Solution Approach

The implementation follows the intuition by first calculating the maximum possible sum and then using a min-heap to find the k-th smallest reduction.

**Step 1: Transform the array and find maximum sum**

```python
mx = 0
for i, x in enumerate(nums):
    if x > 0:
        mx += x
    else:
        nums[i] = -x
```

We iterate through `nums` and:
- Add all positive numbers to `mx` (the maximum possible sum)
- Replace negative numbers with their absolute values

This transformation allows us to treat all elements uniformly as "costs" that reduce from the maximum sum.

**Step 2: Sort the transformed array**

```python
nums.sort()
```

[Sorting](/problems/sorting_summary) in ascending order ensures that we can systematically explore subsequences with increasing reductions.

**Step 3: Use a min-heap to find k-th smallest reduction**

```python
h = [(0, 0)]
for _ in range(k - 1):
    s, i = heappop(h)
    if i < len(nums):
        heappush(h, (s + nums[i], i + 1))
        if i:
            heappush(h, (s + nums[i] - nums[i - 1], i + 1))
```

The heap stores tuples `(s, i)` where:
- `s` is the current reduction sum
- `i` is the index of the next element to consider

Starting with `(0, 0)` (empty subsequence, no reduction), we perform `k-1` iterations:

1. Pop the smallest reduction `(s, i)` from the heap
2. If `i < len(nums)`, we can still add more elements:
   - Push `(s + nums[i], i + 1)`: Include `nums[i]` in the subsequence
   - If `i > 0`, push `(s + nums[i] - nums[i-1], i + 1)`: Replace `nums[i-1]` with `nums[i]` (swap the last included element)

This expansion strategy ensures that:
- We never miss any possible subsequence
- We explore them in order of increasing reduction
- We avoid redundant states

**Step 4: Return the result**

```python
return mx - h[0][0]
```

After `k-1` iterations, the top of the heap contains the k-th smallest reduction. The k-th largest sum is `mx` minus this reduction.

The time complexity is `O(n log n + k log k)` where `n` is the array length (for [sorting](/problems/sorting_summary)) and `k` iterations with heap operations. The space complexity is `O(k)` for the heap.

## Example Walkthrough

Let's walk through the solution with `nums = [2, -3, 5]` and `k = 4`.

**Step 1: Transform array and find maximum sum**
- Original: `[2, -3, 5]`
- `mx = 2 + 5 = 7` (sum of positive numbers)
- Transform negative numbers: `[2, 3, 5]` (replaced `-3` with `3`)

**Step 2: Sort the transformed array**
- Sorted: `[2, 3, 5]`

**Step 3: Use min-heap to find 4th smallest reduction**

Initial heap: `[(0, 0)]` (no reduction, start at index 0)

**Iteration 1** (finding 1st smallest reduction):
- Pop `(0, 0)` from heap
- Since `i = 0 < 3`:
  - Push `(0 + 2, 1) = (2, 1)` (include element at index 0)
  - Skip second push since `i = 0`
- Heap: `[(2, 1)]`

**Iteration 2** (finding 2nd smallest reduction):
- Pop `(2, 1)` from heap  
- Since `i = 1 < 3`:
  - Push `(2 + 3, 2) = (5, 2)` (include element at index 1)
  - Push `(2 + 3 - 2, 2) = (3, 2)` (swap elements at indices 0 and 1)
- Heap: `[(3, 2), (5, 2)]`

**Iteration 3** (finding 3rd smallest reduction):
- Pop `(3, 2)` from heap
- Since `i = 2 < 3`:
  - Push `(3 + 5, 3) = (8, 3)` (include element at index 2)
  - Push `(3 + 5 - 3, 3) = (5, 3)` (swap elements at indices 1 and 2)
- Heap: `[(5, 2), (5, 3), (8, 3)]`

After 3 iterations, the heap's minimum is `(5, 2)`, which is the 4th smallest reduction.

**Step 4: Return result**
- 4th largest sum = `mx - reduction = 7 - 5 = 2`

**Verification:**
All possible subsequence sums for `[2, -3, 5]`:
- `[2, 5]` → sum = 7
- `[5]` → sum = 5  
- `[2]` → sum = 2
- `[2, -3, 5]` → sum = 4
- `[]` → sum = 0
- `[-3, 5]` → sum = 2
- `[-3]` → sum = -3
- `[2, -3]` → sum = -1

Sorted descending: `7, 5, 4, 2, 2, 0, -1, -3`

The 4th largest is indeed `2` ✓

## Python Solution

```python
class Solution:
    def kSum(self, nums: List[int], k: int) -> int:
        # Calculate the maximum possible sum (sum of all positive numbers)
        max_possible_sum = 0
        for i, num in enumerate(nums):
            if num > 0:
                max_possible_sum += num
            else:
                # Convert negative numbers to positive for unified handling
                nums[i] = -num
      
        # Sort the absolute values in ascending order
        nums.sort()
      
        # Min-heap to track the k smallest "reduction amounts" from max_possible_sum
        # Each tuple contains: (reduction_sum, next_index_to_consider)
        min_heap = [(0, 0)]
      
        # Find the (k-1)th smallest reduction to get the k-th largest sum
        for _ in range(k - 1):
            current_reduction, current_index = heappop(min_heap)
          
            if current_index < len(nums):
                # Option 1: Include nums[current_index] in the reduction
                heappush(min_heap, (current_reduction + nums[current_index], current_index + 1))
              
                # Option 2: Replace nums[current_index - 1] with nums[current_index]
                # This only makes sense if we previously included nums[current_index - 1]
                if current_index > 0:
                    heappush(min_heap, (current_reduction + nums[current_index] - nums[current_index - 1], current_index + 1))
      
        # The k-th largest sum = max_possible_sum - (k-1)th smallest reduction
        return max_possible_sum - min_heap[0][0]
```

## Time and Space Complexity

**Time Complexity:** `O(n × log n + k × log k)`

The time complexity consists of two main parts:
- `O(n × log n)` for sorting the `nums` array, where `n` is the length of the array
- `O(k × log k)` for the heap operations:
  - The loop runs `k - 1` times
  - Each iteration performs one `heappop` operation (`O(log h)` where `h` is heap size) and at most two `heappush` operations (also `O(log h)` each)
  - The heap size is bounded by `O(k)` since we only push at most 2 elements per iteration and pop 1 element
  - Therefore, the total heap operations cost is `O(k × log k)`

**Space Complexity:** `O(n)`

The space complexity is determined by:
- `O(n)` for storing the modified `nums` array (in-place modification)
- `O(k)` for the heap `h`, which can contain at most `O(k)` elements
- Since `k ≤ n` (we can't find more than `n` sums), and we need `O(n)` space for the array, the overall space complexity is `O(n)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Misunderstanding the Heap Expansion Logic**

**Pitfall:** The most common mistake is not understanding why we need both expansion options when popping from the heap. Developers often think adding just `(s + nums[i], i + 1)` is sufficient.

**Why it's wrong:** Using only one expansion option will miss many valid subsequences. For example, if `nums = [1, 2, 3]` and we only use the "include" option, we'd generate subsequences like `[]`, `[1]`, `[1,2]`, `[1,2,3]` but miss `[2]`, `[3]`, `[2,3]`, etc.

**Solution:** Always include both expansion options:
```python
# Option 1: Include the current element
heappush(min_heap, (current_reduction + nums[current_index], current_index + 1))

# Option 2: Replace the previous element with current (if previous exists)
if current_index > 0:
    heappush(min_heap, (current_reduction + nums[current_index] - nums[current_index - 1], current_index + 1))
```

### 2. **Incorrectly Handling Negative Numbers**

**Pitfall:** Forgetting to transform negative numbers or transforming them incorrectly. Some might try to simply ignore negative numbers or handle them separately.

**Why it's wrong:** Negative numbers can be part of subsequences and affect the final sum. Simply ignoring them would give incorrect results.

**Solution:** Transform all numbers to their absolute values after calculating the maximum sum:
```python
for i, num in enumerate(nums):
    if num > 0:
        max_possible_sum += num
    else:
        nums[i] = -num  # Convert to positive for unified handling
```

### 3. **Off-by-One Error in Loop Count**

**Pitfall:** Running the heap expansion loop `k` times instead of `k-1` times.

**Why it's wrong:** The initial state `(0, 0)` already represents the first (largest) subsequence sum. We only need `k-1` more iterations to find the k-th largest.

**Solution:** Use `range(k - 1)` for the loop:
```python
# Start with the maximum sum (reduction = 0)
min_heap = [(0, 0)]

# Find k-1 more sums to get to the k-th largest
for _ in range(k - 1):
    # ... heap operations
```

### 4. **Heap State Duplication**

**Pitfall:** Not realizing that the same reduction sum might be reached through different paths, potentially leading to duplicate counting or inefficiency.

**Why it's acceptable here:** While duplicates can occur, they represent different subsequences that happen to have the same sum. Since we want the k-th largest sum (not k distinct sums), counting duplicates is correct. The algorithm naturally handles this by treating each path independently.

**Note:** If you needed k distinct sums, you'd need to track visited states using a set:
```python
visited = set()
while heap and len(result) < k:
    reduction, index = heappop(heap)
    if (reduction, index) in visited:
        continue
    visited.add((reduction, index))
    # ... rest of logic
```

