# 1713. Minimum Operations to Make a Subsequence

> Source: https://algo.monster/liteproblems/1713

## Problem Statement

You have two integer arrays: `target` (containing distinct integers) and `arr` (which may contain duplicates).

You can perform operations to insert any integer at any position in `arr`. For instance, if `arr = [1,4,1,2]`, you could insert `3` in the middle to get `[1,4,3,1,2]`. Insertions can be made at the beginning, end, or anywhere in between.

Your goal is to find the **minimum number of operations** (insertions) needed to make `target` a subsequence of `arr`.

A subsequence is formed by deleting some (or no) elements from the original array without changing the order of the remaining elements. For example:
- `[2,7,4]` is a subsequence of `[4,2,3,7,2,1,4]` (taking the 2nd, 4th, and 7th elements)
- `[2,4,2]` is NOT a subsequence of the same array (the relative order isn't preserved)

The key insight is that you want to maximize the elements from `target` that already appear in `arr` in the correct relative order. The remaining elements from `target` that aren't part of this longest common subsequence will need to be inserted.

For example, if `target = [5,1,3]` and `arr = [9,4,2,3,4]`, the element `3` from target already exists in `arr`. So you only need to insert `5` and `1` (2 operations) to make `target` a subsequence of the modified `arr`.

## Intuition

The key observation is that we want to minimize insertions, which means we should maximize the use of elements that already exist in `arr`. If some elements from `target` already appear in `arr` in the correct relative order, we don't need to insert them - we only need to insert the missing ones.

This naturally leads us to think about the **longest common subsequence (LCS)** between `target` and `arr`. The elements in the LCS are already present in the right order, so we only need to insert the remaining `len(target) - len(LCS)` elements.

However, finding LCS directly has time complexity `O(m × n)`, which is too slow for large inputs. We need a clever transformation.

Since `target` contains **distinct** elements, we can map each element to its position in `target`. For example, if `target = [5,1,3]`, we create a mapping: `{5:0, 1:1, 3:2}`.

Now, when we scan through `arr`, we can convert each element that exists in `target` to its corresponding index. This gives us a new sequence `nums` containing indices. For instance, if `arr = [9,3,1,3]` and `target = [5,1,3]`, we get `nums = [2,1,2]` (ignoring 9 since it's not in target).

The crucial insight: Finding the LCS between `target` and `arr` is equivalent to finding the **longest increasing subsequence (LIS)** in `nums`. Why? Because:
- If elements appear in the correct order in `arr` (matching their order in `target`), their corresponding indices in `nums` will be increasing
- The longest such increasing sequence represents the maximum number of elements we can keep from `arr`

This transformation reduces our problem from LCS `O(m × n)` to LIS, which can be solved efficiently using Binary Indexed Tree or dynamic programming with [binary search](/problems/binary_search_intro) in `O(n log n)` time.

**Learn more about [Greedy](/problems/greedy_intro) and [Binary Search](/problems/binary_search_intro) patterns.**

## Solution Approach

The solution implements the LIS (Longest Increasing Subsequence) approach using a Binary Indexed Tree (Fenwick Tree) for efficient range maximum queries and updates.

**Step 1: Create Index Mapping**
```python
d = {x: i for i, x in enumerate(target, 1)}
```
We create a dictionary mapping each element in `target` to its 1-based index position. Using 1-based indexing simplifies the Binary Indexed Tree implementation.

**Step 2: Transform arr to Index Sequence**
```python
nums = [d[x] for x in arr if x in d]
```
We convert elements from `arr` that exist in `target` to their corresponding indices. Elements not in `target` are ignored since they can't contribute to the common subsequence.

**Step 3: Find LIS using Binary Indexed Tree**

The Binary Indexed Tree (BIT) is used to efficiently track the maximum LIS length ending at or before any position:

- `update(x, v)`: Updates position `x` with value `v` if `v` is greater than the current value
- `query(x)`: Returns the maximum value in the range `[1, x]`

The BIT operations use the bit manipulation trick `x & -x` to get the rightmost set bit, which helps navigate the tree structure efficiently.

**Step 4: Process Each Index**
```python
for x in nums:
    v = tree.query(x - 1) + 1
    ans = max(ans, v)
    tree.update(x, v)
```

For each index `x` in `nums`:
1. Query the maximum LIS length for all indices less than `x` using `tree.query(x - 1)`
2. The LIS length ending at `x` is this maximum plus 1
3. Update the global maximum LIS length
4. Update the BIT at position `x` with this new LIS length

**Step 5: Calculate Result**
```python
return len(target) - ans
```
The minimum number of insertions equals the total length of `target` minus the length of the longest common subsequence (which is the LIS length we found).

**Time Complexity**: `O(n log m)` where `n` is the length of `arr` and `m` is the length of `target`. Each BIT operation takes `O(log m)` time.

**Space Complexity**: `O(m)` for the BIT and the dictionary.

## Example Walkthrough

Let's walk through a concrete example with `target = [6,4,8,1,3,2]` and `arr = [4,7,6,2,3,8,6,1]`.

**Step 1: Create Index Mapping**
We map each element in `target` to its 1-based position:
```
d = {6:1, 4:2, 8:3, 1:4, 3:5, 2:6}
```

**Step 2: Transform arr to Index Sequence**
Convert elements from `arr` that exist in `target` to their indices:
- `arr[0] = 4` → index 2
- `arr[1] = 7` → not in target, skip
- `arr[2] = 6` → index 1
- `arr[3] = 2` → index 6
- `arr[4] = 3` → index 5
- `arr[5] = 8` → index 3
- `arr[6] = 6` → index 1
- `arr[7] = 1` → index 4

Result: `nums = [2, 1, 6, 5, 3, 1, 4]`

**Step 3 & 4: Find LIS using Binary Indexed Tree**
Process each index in `nums` to find the longest increasing subsequence:

| Index in nums | Value x | query(x-1) | LIS length at x | BIT state after update | ans |
|--------------|---------|------------|-----------------|------------------------|-----|
| 0 | 2 | query(1)=0 | 0+1=1 | BIT[2]=1 | 1 |
| 1 | 1 | query(0)=0 | 0+1=1 | BIT[1]=1 | 1 |
| 2 | 6 | query(5)=1 | 1+1=2 | BIT[6]=2 | 2 |
| 3 | 5 | query(4)=1 | 1+1=2 | BIT[5]=2 | 2 |
| 4 | 3 | query(2)=1 | 1+1=2 | BIT[3]=2 | 2 |
| 5 | 1 | query(0)=0 | 0+1=1 | BIT[1]=1 (no change) | 2 |
| 6 | 4 | query(3)=2 | 2+1=3 | BIT[4]=3 | 3 |

The longest increasing subsequence has length 3. Looking at our sequence `nums = [2, 1, 6, 5, 3, 1, 4]`:
- One valid LIS is `[1, 3, 4]` (positions 1, 4, 6 in nums)
- This corresponds to elements `[6, 8, 1]` from the original arrays

**Step 5: Calculate Result**
Minimum insertions = `len(target) - LIS_length = 6 - 3 = 3`

We need to insert 3 elements from `target` (specifically `4`, `3`, and `2`) to make `target` a subsequence of the modified `arr`.

**Verification**: The elements `[6, 8, 1]` already appear in `arr` in the correct relative order at positions 2, 5, and 7. By inserting the missing elements `[4, 3, 2]` at appropriate positions, we can make the entire `target` array a subsequence of the modified `arr`.

## Python Solution

```python
class BinaryIndexedTree:
    """
    Binary Indexed Tree (Fenwick Tree) for range maximum queries.
    This implementation supports point updates and prefix maximum queries.
    """
    __slots__ = "size", "tree"

    def __init__(self, n: int):
        """
        Initialize a Binary Indexed Tree with size n.

        Args:
            n: The size of the tree (1-indexed)
        """
        self.size = n
        self.tree = [0] * (n + 1)  # 1-indexed array for BIT

    def update(self, index: int, value: int):
        """
        Update the tree by setting the maximum value at position index.

        Args:
            index: The position to update (1-indexed)
            value: The value to compare with existing maximum
        """
        while index <= self.size:
            # Update current position with maximum value
            self.tree[index] = max(self.tree[index], value)
            # Move to next index affected by this update
            # index & -index gives the lowest set bit
            index += index & -index

    def query(self, index: int) -> int:
        """
        Query the maximum value in range [1, index].

        Args:
            index: The right boundary of the query range (1-indexed)

        Returns:
            Maximum value in the range [1, index]
        """
        max_value = 0
        while index > 0:
            # Take maximum of current position
            max_value = max(max_value, self.tree[index])
            # Move to parent node by removing lowest set bit
            index -= index & -index
        return max_value


class Solution:
    def minOperations(self, target: List[int], arr: List[int]) -> int:
        """
        Find minimum operations to make arr a subsequence of target.
        This is equivalent to finding (target_length - LCS_length).

        Args:
            target: The target sequence
            arr: The array to transform

        Returns:
            Minimum number of operations needed
        """
        # Create position mapping for target elements (1-indexed)
        position_map = {value: index for index, value in enumerate(target, 1)}

        # Convert arr elements to their positions in target
        # Filter out elements not in target
        mapped_positions = [position_map[value] for value in arr if value in position_map]

        # Initialize BIT with size of target
        target_length = len(target)
        fenwick_tree = BinaryIndexedTree(target_length)

        # Find longest increasing subsequence in mapped positions
        # This corresponds to finding the longest common subsequence
        longest_subsequence = 0

        for position in mapped_positions:
            # Query maximum length ending before current position
            prev_max_length = fenwick_tree.query(position - 1)
            # Current subsequence length including this element
            current_length = prev_max_length + 1
            # Update global maximum
            longest_subsequence = max(longest_subsequence, current_length)
            # Update tree with new length at this position
            fenwick_tree.update(position, current_length)

        # Return minimum operations: elements to insert
        return target_length - longest_subsequence
```

## Time and Space Complexity

**Time Complexity:** `O(n × log m)`

The algorithm iterates through the `nums` array, which has at most `n` elements (where `n` is the length of `arr`). For each element in `nums`:
- The `query` operation on the Binary Indexed Tree takes `O(log m)` time, where `m` is the length of `target`. This is because the query traverses up the tree by removing the lowest set bit (`x -= x & -x`), which takes at most `log m` iterations.
- The `update` operation also takes `O(log m)` time, as it traverses up the tree by adding the lowest set bit (`x += x & -x`), which similarly takes at most `log m` iterations.

Since we perform both operations for each element in `nums`, the total time complexity is `O(n × log m)`.

**Space Complexity:** `O(m)`

The space usage consists of:
- The dictionary `d` which stores `m` key-value pairs (one for each element in `target`): `O(m)`
- The Binary Indexed Tree's array `c` which has size `m + 1`: `O(m)`
- The `nums` array which stores at most `n` elements, but since `n` could be less than or equal to `m` in the worst case, this doesn't affect the overall complexity
- Other variables like `ans`, `v`, `x` use constant space: `O(1)`

The dominant space usage is `O(m)`, where `m` is the length of `target`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Handling of Duplicate Elements in arr**

**The Pitfall:**
A common mistake is not properly handling duplicates in `arr`. When the same element appears multiple times in `arr`, each occurrence should be considered independently for building the longest increasing subsequence. Some might incorrectly try to deduplicate `arr` or handle duplicates specially, which would lead to wrong answers.

**Example:**
- `target = [1, 2, 3]`
- `arr = [1, 1, 2, 2, 3, 3]`

Each duplicate can potentially contribute to different valid subsequences. The algorithm correctly processes each occurrence separately through the mapped positions.

**Solution:**
The current implementation handles this correctly by:
```python
mapped_positions = [position_map[value] for value in arr if value in position_map]
```
This preserves all occurrences of elements from `arr`, allowing the LIS algorithm to consider each one.

### 2. **Off-by-One Errors with BIT Indexing**

**The Pitfall:**
Binary Indexed Trees traditionally use 1-based indexing, but mixing 0-based and 1-based indexing can cause subtle bugs. A common error is forgetting to query `position - 1` instead of `position`:

```python
# WRONG: This would include the current position in the query
prev_max_length = fenwick_tree.query(position)

# CORRECT: Query strictly less than current position
prev_max_length = fenwick_tree.query(position - 1)
```

**Why it matters:**
We need the maximum LIS length ending at positions strictly less than the current position to maintain the increasing property.

**Solution:**
Always ensure:
- Map target elements to 1-based indices: `enumerate(target, 1)`
- Query `position - 1` to get maximum of all previous positions
- BIT size should be `len(target)` not `len(target) + 1` since we're already 1-indexed

### 3. **Using Standard LIS Instead of Position-Based LIS**

**The Pitfall:**
Some might try to find the LIS directly on the values in `arr` that appear in `target`, rather than converting to position indices first:

```python
# WRONG APPROACH:
common_elements = [x for x in arr if x in target]
lis_length = find_lis(common_elements)  # This won't work!
```

**Why this fails:**
The LIS of actual values doesn't correspond to the longest common subsequence. We need to respect the order of elements as they appear in `target`.

**Example:**
- `target = [5, 1, 3]`
- `arr = [1, 3, 5]`

Direct LIS on `[1, 3, 5]` gives length 3, but the actual LCS is length 0 because the order in `arr` doesn't match `target`'s order.

**Solution:**
Transform to position indices first, then find LIS on those positions. This ensures we're finding subsequences that respect `target`'s ordering.

### 4. **Memory Optimization Confusion**

**The Pitfall:**
Some might try to optimize memory by using a smaller BIT size based on the number of unique elements actually present:

```python
# WRONG: Using only the size of common elements
unique_common = len(set(arr) & set(target))
fenwick_tree = BinaryIndexedTree(unique_common)
```

**Why this fails:**
The position indices can range from 1 to `len(target)`, regardless of how many elements from `target` actually appear in `arr`. Using a smaller BIT would cause index out of bounds errors.

**Solution:**
Always initialize the BIT with the full size of `target`:
```python
fenwick_tree = BinaryIndexedTree(len(target))
```

