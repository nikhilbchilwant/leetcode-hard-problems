# 2334. Subarray With Elements Greater Than Varying Threshold

> Source: https://algo.monster/liteproblems/2334

## Problem Statement

You are given an integer array `nums` and an integer `threshold`.

Your task is to find a contiguous subarray of `nums` with length `k` where every element in that subarray is strictly greater than `threshold / k`.

The key constraint is that **all elements** in the chosen subarray must satisfy the condition: each element > `threshold / k`, where `k` is the length of the subarray.

Return the size (length) of any such valid subarray. If no such subarray exists, return `-1`.

For example:
- If you have a subarray of length 3, each element in it must be greater than `threshold / 3`
- If you have a subarray of length 5, each element in it must be greater than `threshold / 5`

The problem asks you to find **any** valid subarray (not necessarily the smallest or largest), and return its length. The subarray must be contiguous, meaning the elements must be adjacent in the original array.

## Intuition

The key insight is to think about this problem from a different angle. Instead of checking every possible subarray (which would be inefficient), we can observe that for a subarray of length `k` to be valid, its minimum element must be greater than `threshold / k`.

This means: `min_element_in_subarray > threshold / k`

Rearranging: `min_element_in_subarray * k > threshold`

Now, here's the clever part: if we process elements from largest to smallest, we can build connected components gradually. When we add an element to our structure, it can either:
1. Stand alone as a subarray of length 1
2. Connect with already processed neighbors (which are all larger than the current element)

Since we process from largest to smallest, when we add element `v` at position `i`, it becomes the minimum element in any connected component it joins. The size of this component represents a potential subarray length `k`.

By using Union-Find (Disjoint Set Union), we can efficiently:
- Track connected components of processed elements
- Merge adjacent components when we process a new element
- Keep track of component sizes

At each step, when processing element `v` with its component of size `k`, we check if `v > threshold / k`. If true, we've found a valid subarray where `v` is the minimum element and all other elements in the component are larger than `v` (since we processed them earlier).

This approach is optimal because:
1. We only need to check each element once as a potential minimum
2. Union-Find operations are nearly constant time
3. We can stop as soon as we find any valid subarray

**Learn more about [Stack](/problems/stack_intro), [Union Find](/problems/dsu_intro) and [Monotonic Stack](/problems/mono_stack_intro) patterns.**

## Solution Approach

The implementation uses a Union-Find data structure with a greedy approach, processing elements from largest to smallest:

**1. Data Structure Setup:**
- `p`: Parent array for Union-Find, initially `p[i] = i` (each element is its own parent)
- `size`: Array tracking the size of each component, initially all 1
- `vis`: Boolean array marking which positions have been processed

**2. Sorting Strategy:**
Create `arr` by sorting `(value, index)` pairs in descending order by value. This ensures we process elements from largest to smallest.

**3. Union-Find Operations:**
- `find(x)`: Finds the root of component containing `x` with path compression
- `merge(a, b)`: Merges two components, updating the size of the root

**4. Main Algorithm:**
For each element `(v, i)` in sorted order:
1. **Check left neighbor**: If `i > 0` and `vis[i-1]` is true (left neighbor already processed), merge current position with left neighbor
2. **Check right neighbor**: If `i < n-1` and `vis[i+1]` is true (right neighbor already processed), merge current position with right neighbor
3. **Validate subarray**: After merging, check if `v > threshold // size[find(i)]`
   - If true, return `size[find(i)]` as we found a valid subarray
   - The current element `v` is the minimum in this component (all others are larger)
4. **Mark as visited**: Set `vis[i] = True` to indicate this position is processed

**5. Key Observations:**
- When we process element `v`, all previously processed elements are `≥ v`
- The connected component containing `v` forms a contiguous subarray
- `v` is the minimum element in its component
- The condition `v > threshold / k` becomes `v > threshold // k` using integer division

**6. Time Complexity:**
- Sorting: `O(n log n)`
- Union-Find operations: `O(n × α(n))` where `α` is the inverse Ackermann function (nearly constant)
- Overall: `O(n log n)`

The algorithm returns `-1` if no valid subarray is found after processing all elements.

## Example Walkthrough

Let's walk through a small example with `nums = [3, 5, 2, 4]` and `threshold = 10`.

**Initial Setup:**
- Array: `[3, 5, 2, 4]` with indices `[0, 1, 2, 3]`
- Create sorted array by value (descending): `[(5, 1), (4, 3), (3, 0), (2, 2)]`
- Initialize Union-Find: `p = [0, 1, 2, 3]`, `size = [1, 1, 1, 1]`, `vis = [False, False, False, False]`

**Step 1: Process (5, 1)**
- Value = 5, Index = 1
- Check neighbors:
  - Left (index 0): Not visited yet, skip
  - Right (index 2): Not visited yet, skip
- Component size = 1
- Check condition: 5 > 10/1? → 5 > 10? → No
- Mark `vis[1] = True`
- State: `vis = [False, True, False, False]`

**Step 2: Process (4, 3)**
- Value = 4, Index = 3
- Check neighbors:
  - Left (index 2): Not visited yet, skip
  - Right: Out of bounds
- Component size = 1
- Check condition: 4 > 10/1? → 4 > 10? → No
- Mark `vis[3] = True`
- State: `vis = [False, True, False, True]`

**Step 3: Process (3, 0)**
- Value = 3, Index = 0
- Check neighbors:
  - Left: Out of bounds
  - Right (index 1): Visited! Merge components 0 and 1
- After merge: Component rooted at 1 has size 2
- Check condition: 3 > 10/2? → 3 > 5? → No
- Mark `vis[0] = True`
- State: `vis = [True, True, False, True]`, subarray [3, 5] forms a component

**Step 4: Process (2, 2)**
- Value = 2, Index = 2
- Check neighbors:
  - Left (index 1): Visited! Merge with component containing indices 0 and 1
  - Right (index 3): Visited! Also merge with component at index 3
- After merges: All indices form one component of size 4
- Check condition: 2 > 10/4? → 2 > 2.5? → No (using integer division: 2 > 2? → No)
- Mark `vis[2] = True`

**Result:** Return -1 (no valid subarray found)

---

**Another example where we find a valid subarray:**
`nums = [6, 5, 6, 5, 8]` and `threshold = 7`

Processing in order: `[(8, 4), (6, 0), (6, 2), (5, 1), (5, 3)]`

1. Process (8, 4): 8 > 7/1? → Yes! Return 1 immediately
   - The single element subarray [8] at index 4 satisfies the condition

The algorithm efficiently finds valid subarrays by ensuring that when we check element `v` with its component of size `k`, `v` is guaranteed to be the minimum in that component, making it sufficient to only check if `v > threshold/k`.

## Python Solution

```python
class Solution:
    def validSubarraySize(self, nums: List[int], threshold: int) -> int:
        """
        Find the size of a subarray where all elements are greater than threshold/k,
        where k is the size of the subarray.
      
        Strategy: Process elements from largest to smallest, building connected components
        of visited elements. Check if current element satisfies the threshold condition
        for its component size.
        """
      
        def find_root(node: int) -> int:
            """Find the root of the set containing node with path compression."""
            if parent[node] != node:
                parent[node] = find_root(parent[node])  # Path compression
            return parent[node]
      
        def union_sets(node_a: int, node_b: int) -> None:
            """Merge the sets containing node_a and node_b."""
            root_a = find_root(node_a)
            root_b = find_root(node_b)
          
            if root_a == root_b:
                return  # Already in the same set
          
            # Merge set a into set b
            parent[root_a] = root_b
            component_size[root_b] += component_size[root_a]
      
        # Initialize variables
        array_length = len(nums)
        parent = list(range(array_length))  # Each element is initially its own parent
        component_size = [1] * array_length  # Each component initially has size 1
      
        # Sort elements by value in descending order, keeping track of original indices
        sorted_elements = sorted(zip(nums, range(array_length)), reverse=True)
      
        # Track which indices have been visited/processed
        visited = [False] * array_length
      
        # Process elements from largest to smallest
        for value, index in sorted_elements:
            # Merge with left neighbor if it has been visited
            if index > 0 and visited[index - 1]:
                union_sets(index, index - 1)
          
            # Merge with right neighbor if it has been visited
            if index < array_length - 1 and visited[index + 1]:
                union_sets(index, index + 1)
          
            # Check if current value satisfies the threshold condition
            # For a subarray of size k, all elements must be > threshold/k
            current_component_size = component_size[find_root(index)]
            if value > threshold // current_component_size:
                return current_component_size
          
            # Mark current index as visited
            visited[index] = True
      
        # No valid subarray found
        return -1
```

## Time and Space Complexity

**Time Complexity:** `O(n log n)`

The time complexity is dominated by the sorting operation `sorted(zip(nums, range(n)), reverse=True)` which takes `O(n log n)` time. After sorting, the code iterates through the sorted array once, performing union-find operations. Each union-find operation (both `find` and `merge`) has an amortized time complexity of `O(α(n))` where `α` is the inverse Ackermann function, which is effectively constant for practical purposes. Since we perform at most `2n` union operations (each element can be merged with its left and right neighbors at most once) and `n` find operations, the total time for union-find operations is `O(n × α(n)) ≈ O(n)`. Therefore, the overall time complexity is `O(n log n) + O(n) = O(n log n)`.

**Space Complexity:** `O(n)`

The space complexity consists of:
- `p` array: `O(n)` for storing parent pointers in the union-find structure
- `size` array: `O(n)` for storing the size of each component
- `arr` sorted array: `O(n)` for storing the sorted (value, index) pairs
- `vis` array: `O(n)` for tracking visited elements
- Recursion stack for `find` function: `O(log n)` in average case with path compression, `O(n)` in worst case without optimization

The total space complexity is `O(n) + O(n) + O(n) + O(n) + O(n) = O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Integer Division vs Float Division**
The most critical pitfall is using integer division (`//`) instead of proper comparison with floating-point division.

**Problem:** The code uses `value > threshold // current_component_size`, which performs integer division and can incorrectly reject valid subarrays.

**Example:** 
- `threshold = 7`, `current_component_size = 3`, `value = 3`
- Integer division: `7 // 3 = 2`, so `3 > 2` returns `True` ✓
- Actual requirement: `3 > 7/3 = 2.333...` returns `True` ✓
- But consider `threshold = 8`, `current_component_size = 3`, `value = 3`
- Integer division: `8 // 3 = 2`, so `3 > 2` returns `True` ✓ (WRONG!)
- Actual requirement: `3 > 8/3 = 2.666...` returns `True` ✓

**Solution:** Use multiplication to avoid division precision issues:
```python
# Instead of: value > threshold // current_component_size
# Use: value * current_component_size > threshold
if value * current_component_size > threshold:
    return current_component_size
```

### 2. **Handling Edge Cases with Zero or Negative Values**
If the array contains zeros or negative numbers, the algorithm might behave unexpectedly.

**Problem:** The comparison logic assumes positive values. With negative numbers, larger absolute values processed first might not form valid subarrays.

**Solution:** Add validation or handle negative cases explicitly:
```python
# If problem guarantees positive integers, add assertion
assert all(num > 0 for num in nums), "All numbers must be positive"

# Or handle negative threshold cases
if threshold < 0:
    # All positive numbers satisfy the condition for negative threshold
    return 1
```

### 3. **Union-Find Path Compression Inconsistency**
The current implementation doesn't consistently use path compression in all find operations.

**Problem:** The `component_size` array is accessed using `find_root(index)` but not all paths are compressed optimally.

**Solution:** Ensure consistent path compression:
```python
def find_root(node: int) -> int:
    if parent[node] != node:
        parent[node] = find_root(parent[node])
    return parent[node]

# Always use find_root when accessing component_size
root = find_root(index)
current_component_size = component_size[root]
```

### 4. **Overflow Risk with Large Values**
When using multiplication to avoid division issues, there's a risk of integer overflow.

**Problem:** `value * current_component_size` might overflow for large values.

**Solution:** Use careful comparison or Python's arbitrary precision integers:
```python
# Python handles large integers automatically, but in other languages:
# Check for potential overflow
if current_component_size > 0 and value > threshold // current_component_size:
    # Additional check for exact division
    if value * current_component_size > threshold:
        return current_component_size
```

### 5. **Empty Array or Invalid Input**
The code doesn't handle edge cases like empty arrays or invalid threshold values.

**Problem:** Empty array will cause the function to return -1, but might need explicit handling.

**Solution:** Add input validation:
```python
if not nums:
    return -1
if threshold < 0:
    return len(nums)  # All subarrays valid for negative threshold
```

