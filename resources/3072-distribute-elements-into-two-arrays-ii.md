# 3072. Distribute Elements Into Two Arrays II

> Source: https://algo.monster/liteproblems/3072

## Problem Statement

You have a 1-indexed array `nums` of length `n`. You need to distribute all elements from this array into two new arrays `arr1` and `arr2` following specific rules.

The distribution process works as follows:
- First operation: Put `nums[1]` into `arr1`
- Second operation: Put `nums[2]` into `arr2`
- For each subsequent element `nums[i]` (starting from index 3), you need to count how many elements in each array are strictly greater than `nums[i]`:
  - If `arr1` has more elements greater than `nums[i]` than `arr2`, add `nums[i]` to `arr1`
  - If `arr2` has more elements greater than `nums[i]` than `arr1`, add `nums[i]` to `arr2`
  - If both arrays have the same count of elements greater than `nums[i]`:
    - Add `nums[i]` to the array with fewer total elements
    - If both arrays have the same size, add `nums[i]` to `arr1`

After distributing all elements, concatenate `arr1` and `arr2` to form the result array. For example, if `arr1 = [1,2,3]` and `arr2 = [4,5,6]`, then `result = [1,2,3,4,5,6]`.

## Intuition

The key challenge in this problem is efficiently counting how many elements in each array are greater than the current element we're trying to place. A naive approach would be to iterate through both arrays each time and count, but this would result in `O(n²)` time complexity.

We need a data structure that can:
1. Handle insertions dynamically as we build the arrays
2. Quickly answer range queries (how many elements are greater than a given value)

This naturally leads us to think about data structures that support efficient range queries. A Binary Indexed Tree (Fenwick Tree) is perfect for this scenario because it can update and query in `O(log n)` time.

However, there's another challenge: the values in the array could be very large (up to `10^9`), and we can't create a Binary Indexed Tree of that size. This is where discretization comes in. Since we only care about the relative ordering of elements, not their actual values, we can map all unique values to a smaller range `[1, m]` where `m` is the number of unique values.

The insight for using Binary Indexed Trees specifically comes from recognizing that "count of elements greater than `x`" can be computed as: `total_elements - count_of_elements_less_than_or_equal_to_x`. Binary Indexed Trees excel at answering prefix sum queries (which gives us the count of elements less than or equal to a value when we treat each index as representing a discretized value).

By maintaining two separate Binary Indexed Trees (one for each array), we can track the distribution of elements in both arrays independently and make the placement decision in `O(log n)` time for each element. The discretization step ensures our trees are of manageable size while preserving the relative ordering needed for correct comparisons.

**Learn more about [Segment Tree](/problems/segment_tree_intro) patterns.**

## Solution Approach

The solution implements the strategy using discretization and Binary Indexed Trees. Let's walk through the implementation step by step:

**1. Discretization Phase:**
```python
st = sorted(set(nums))
m = len(st)
```
First, we create a sorted list of unique values from `nums`. This maps the potentially large range of values to indices `[0, m-1]` where `m` is the number of unique values.

**2. Initialize Data Structures:**
```python
tree1 = BinaryIndexedTree(m + 1)
tree2 = BinaryIndexedTree(m + 1)
arr1 = [nums[0]]
arr2 = [nums[1]]
```
We create two Binary Indexed Trees with size `m + 1` (1-indexed) and initialize `arr1` and `arr2` with the first two elements as per the problem requirements.

**3. Update Trees for Initial Elements:**
```python
tree1.update(bisect_left(st, nums[0]) + 1, 1)
tree2.update(bisect_left(st, nums[1]) + 1, 1)
```
We find the discretized positions of `nums[0]` and `nums[1]` using binary search (`bisect_left`) and update the respective trees. The `+1` is because Binary Indexed Trees are 1-indexed.

**4. Process Remaining Elements:**
For each element from `nums[2]` onwards:
```python
i = bisect_left(st, x) + 1
a = len(arr1) - tree1.query(i)
b = len(arr2) - tree2.query(i)
```
- Find the discretized position `i` of current element `x`
- `tree1.query(i)` returns count of elements ≤ `x` in `arr1`
- `len(arr1) - tree1.query(i)` gives count of elements > `x` in `arr1`
- Similarly calculate for `arr2`

**5. Decision Logic:**
```python
if a > b:
    arr1.append(x)
    tree1.update(i, 1)
elif a < b:
    arr2.append(x)
    tree2.update(i, 1)
elif len(arr1) <= len(arr2):
    arr1.append(x)
    tree1.update(i, 1)
else:
    arr2.append(x)
    tree2.update(i, 1)
```
Based on the greater count comparison:
- If `arr1` has more elements > `x`, add to `arr1`
- If `arr2` has more elements > `x`, add to `arr2`
- If tied, add to the smaller array (or `arr1` if equal sizes)

**Binary Indexed Tree Operations:**
- `update(x, delta)`: Adds `delta` to position `x`, propagating updates through the tree using `x += x & -x`
- `query(x)`: Returns sum of elements from positions 1 to `x`, traversing using `x -= x & -x`

Both operations work in `O(log m)` time due to the tree structure.

**Time Complexity:** `O(n log n)` for sorting + `O(n log m)` for processing = `O(n log n)`  
**Space Complexity:** `O(m)` for the discretization array and Binary Indexed Trees

## Example Walkthrough

Let's walk through a small example with `nums = [5, 14, 3, 1, 2]`.

**Step 1: Discretization**
- Extract unique values and sort: `st = [1, 2, 3, 5, 14]`
- This maps our values to indices: 1→0, 2→1, 3→2, 5→3, 14→4

**Step 2: Initialize Arrays and Trees**
- `arr1 = [5]` (first element)
- `arr2 = [14]` (second element)
- Update tree1 at position 4 (5 maps to index 3, +1 for 1-indexing)
- Update tree2 at position 5 (14 maps to index 4, +1 for 1-indexing)

**Step 3: Process nums[2] = 3**
- Find discretized position: 3 maps to index 2, so position = 3
- Count elements > 3 in arr1: `len(arr1) - tree1.query(3) = 1 - 0 = 1` (5 is greater)
- Count elements > 3 in arr2: `len(arr2) - tree2.query(3) = 1 - 0 = 1` (14 is greater)
- Both have 1 element > 3, so check sizes: both arrays have size 1
- Since sizes are equal, add to arr1: `arr1 = [5, 3]`
- Update tree1 at position 3

**Step 4: Process nums[3] = 1**
- Find discretized position: 1 maps to index 0, so position = 1
- Count elements > 1 in arr1: `len(arr1) - tree1.query(1) = 2 - 0 = 2` (both 5 and 3 are greater)
- Count elements > 1 in arr2: `len(arr2) - tree2.query(1) = 1 - 0 = 1` (14 is greater)
- arr1 has more elements > 1, so add to arr1: `arr1 = [5, 3, 1]`
- Update tree1 at position 1

**Step 5: Process nums[4] = 2**
- Find discretized position: 2 maps to index 1, so position = 2
- Count elements > 2 in arr1: `len(arr1) - tree1.query(2) = 3 - 1 = 2` (5 and 3 are greater)
- Count elements > 2 in arr2: `len(arr2) - tree2.query(2) = 1 - 0 = 1` (14 is greater)
- arr1 has more elements > 2, so add to arr1: `arr1 = [5, 3, 1, 2]`
- Update tree1 at position 2

**Final Result:**
- `arr1 = [5, 3, 1, 2]`
- `arr2 = [14]`
- Result = `[5, 3, 1, 2, 14]`

The Binary Indexed Trees efficiently tracked the distribution of values, allowing us to count elements greater than each new value in O(log m) time, where m is the number of unique values.

## Python Solution

```python
from typing import List
from bisect import bisect_left


class BinaryIndexedTree:
    """
    Binary Indexed Tree (Fenwick Tree) for efficient prefix sum queries and updates.
    Supports O(log n) update and query operations.
    """
    __slots__ = "n", "c"
  
    def __init__(self, n: int):
        """
        Initialize a Binary Indexed Tree with size n.
      
        Args:
            n: The size of the tree (1-indexed)
        """
        self.n = n
        self.c = [0] * (n + 1)  # Tree array (1-indexed)
  
    def update(self, x: int, delta: int) -> None:
        """
        Add delta to the value at position x.
      
        Args:
            x: The position to update (1-indexed)
            delta: The value to add
        """
        while x <= self.n:
            self.c[x] += delta
            x += x & -x  # Move to next node by adding lowest set bit
  
    def query(self, x: int) -> int:
        """
        Calculate the prefix sum from index 1 to x.
      
        Args:
            x: The ending position for prefix sum (1-indexed)
      
        Returns:
            The sum of elements from index 1 to x
        """
        prefix_sum = 0
        while x > 0:
            prefix_sum += self.c[x]
            x -= x & -x  # Move to parent node by removing lowest set bit
        return prefix_sum


class Solution:
    def resultArray(self, nums: List[int]) -> List[int]:
        """
        Divide the array into two subarrays based on counting elements greater than current.
        Uses Binary Indexed Trees to efficiently count elements greater than a given value.
      
        Args:
            nums: Input list of integers
      
        Returns:
            Concatenation of the two resulting arrays
        """
        # Create sorted unique values for coordinate compression
        sorted_unique = sorted(set(nums))
        unique_count = len(sorted_unique)
      
        # Initialize two Binary Indexed Trees for the two arrays
        tree1 = BinaryIndexedTree(unique_count + 1)
        tree2 = BinaryIndexedTree(unique_count + 1)
      
        # Initialize first two elements
        # Add 1 to indices since BIT is 1-indexed
        first_idx = bisect_left(sorted_unique, nums[0]) + 1
        second_idx = bisect_left(sorted_unique, nums[1]) + 1
      
        tree1.update(first_idx, 1)
        tree2.update(second_idx, 1)
      
        arr1 = [nums[0]]
        arr2 = [nums[1]]
      
        # Process remaining elements
        for value in nums[2:]:
            # Find compressed index for current value
            compressed_idx = bisect_left(sorted_unique, value) + 1
          
            # Count elements strictly greater than current value in each array
            # Elements greater = total elements - elements less than or equal
            count_greater_arr1 = len(arr1) - tree1.query(compressed_idx)
            count_greater_arr2 = len(arr2) - tree2.query(compressed_idx)
          
            # Decide which array to add the element to
            if count_greater_arr1 > count_greater_arr2:
                # More elements greater in arr1, add to arr1
                arr1.append(value)
                tree1.update(compressed_idx, 1)
            elif count_greater_arr1 < count_greater_arr2:
                # More elements greater in arr2, add to arr2
                arr2.append(value)
                tree2.update(compressed_idx, 1)
            elif len(arr1) <= len(arr2):
                # Equal count of greater elements, add to smaller array (arr1)
                arr1.append(value)
                tree1.update(compressed_idx, 1)
            else:
                # Equal count of greater elements, add to smaller array (arr2)
                arr2.append(value)
                tree2.update(compressed_idx, 1)
      
        # Return concatenation of both arrays
        return arr1 + arr2
```

## Time and Space Complexity

**Time Complexity:** `O(n × log n)`

The time complexity analysis breaks down as follows:
- Creating the sorted set `st = sorted(set(nums))` takes `O(n log n)` time
- Initializing the two Binary Indexed Trees takes `O(m)` time where `m ≤ n`
- Processing the first two elements involves:
  - `bisect_left` operations: `O(log m)` each
  - BIT update operations: `O(log m)` each
- The main loop iterates through `n-2` elements, and for each element:
  - `bisect_left(st, x)`: `O(log m)` time
  - `tree1.query(i)` and `tree2.query(i)`: `O(log m)` time each
  - One BIT update operation: `O(log m)` time
- Since the loop runs `O(n)` times with `O(log m)` operations per iteration, and `m ≤ n`, the loop contributes `O(n log n)`
- The final concatenation `arr1 + arr2` takes `O(n)` time

Overall: `O(n log n) + O(n × log n) + O(n) = O(n × log n)`

**Space Complexity:** `O(n)`

The space complexity analysis:
- The sorted set `st` stores at most `n` unique elements: `O(n)`
- Two Binary Indexed Trees, each with size `m+1 ≤ n+1`: `O(n)`
- Arrays `arr1` and `arr2` together store all `n` elements: `O(n)`
- Additional variables use constant space: `O(1)`

Total space complexity: `O(n) + O(n) + O(n) = O(n)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Off-by-One Errors with 1-Indexed Binary Indexed Trees**

One of the most common mistakes when implementing this solution is forgetting that Binary Indexed Trees are traditionally 1-indexed, while Python lists are 0-indexed. This can lead to incorrect index calculations when updating or querying the tree.

**Pitfall Example:**
```python
# WRONG: Forgetting to add 1 when converting to BIT index
compressed_idx = bisect_left(sorted_unique, value)  # 0-indexed
tree1.update(compressed_idx, 1)  # BIT expects 1-indexed!
```

**Solution:**
Always add 1 when converting from the 0-indexed result of `bisect_left` to the 1-indexed BIT position:
```python
# CORRECT: Convert 0-indexed to 1-indexed
compressed_idx = bisect_left(sorted_unique, value) + 1
tree1.update(compressed_idx, 1)
```

### 2. **Incorrectly Calculating Count of Elements Greater Than Current**

Another frequent error is miscalculating the number of elements strictly greater than the current value. The BIT's `query(i)` returns the count of elements less than or equal to the value at index `i`, not strictly less than.

**Pitfall Example:**
```python
# WRONG: This counts elements >= value, not > value
count_greater = len(arr1) - tree1.query(compressed_idx - 1)
```

**Solution:**
Use the correct formula: total elements minus elements less than or equal to current:
```python
# CORRECT: Count elements strictly greater than value
count_greater = len(arr1) - tree1.query(compressed_idx)
```

### 3. **Forgetting to Handle Duplicate Values in Discretization**

When discretizing values, using `sorted(nums)` instead of `sorted(set(nums))` creates duplicate entries, leading to incorrect index mappings and wasted space.

**Pitfall Example:**
```python
# WRONG: Duplicates in sorted array cause incorrect mappings
sorted_values = sorted(nums)  # [1, 1, 2, 2, 3] - duplicates!
idx = bisect_left(sorted_values, 2)  # May return different indices for same value
```

**Solution:**
Always use `set()` to remove duplicates before sorting:
```python
# CORRECT: Remove duplicates for proper discretization
sorted_unique = sorted(set(nums))  # [1, 2, 3] - no duplicates
idx = bisect_left(sorted_unique, 2)  # Consistent index for value 2
```

### 4. **Incorrect Tie-Breaking Logic**

When the count of greater elements is equal in both arrays, the tie-breaking rule must check array sizes correctly. A common mistake is using the wrong comparison operator or forgetting this step entirely.

**Pitfall Example:**
```python
# WRONG: Using wrong comparison or always defaulting to arr1
if count_greater_arr1 == count_greater_arr2:
    arr1.append(value)  # Always adding to arr1 regardless of size!
    tree1.update(compressed_idx, 1)
```

**Solution:**
Implement proper tie-breaking by comparing array lengths:
```python
# CORRECT: Check array sizes when counts are equal
elif len(arr1) <= len(arr2):
    arr1.append(value)
    tree1.update(compressed_idx, 1)
else:
    arr2.append(value)
    tree2.update(compressed_idx, 1)
```

