# 2426. Number of Pairs Satisfying Inequality

> Source: https://algo.monster/liteproblems/2426

## Problem Statement

You are given two integer arrays `nums1` and `nums2`, both of size `n`, and an integer `diff`. Your task is to find the number of pairs `(i, j)` that satisfy both of these conditions:

1. `0 <= i < j <= n - 1` (meaning `i` comes before `j` in the arrays)
2. `nums1[i] - nums1[j] <= nums2[i] - nums2[j] + diff`

The second condition can be understood as: the difference between elements at positions `i` and `j` in `nums1` should not exceed the difference between elements at the same positions in `nums2` plus the given `diff` value.

For example, if you have:
- `nums1 = [3, 5]`
- `nums2 = [1, 2]`
- `diff = 1`

You would check the pair `(0, 1)`:
- Left side: `nums1[0] - nums1[1] = 3 - 5 = -2`
- Right side: `nums2[0] - nums2[1] + diff = 1 - 2 + 1 = 0`
- Since `-2 <= 0`, this pair satisfies the condition

The problem asks you to count all such valid pairs and return the total count.

## Intuition

The key insight is to rearrange the given inequality to make it easier to work with. Starting with `nums1[i] - nums1[j] <= nums2[i] - nums2[j] + diff`, we can rearrange the terms by grouping elements from the same index together:

`nums1[i] - nums2[i] <= nums1[j] - nums2[j] + diff`

This transformation is crucial because now we can define a new array where each element is `nums[k] = nums1[k] - nums2[k]`. The problem then simplifies to finding pairs `(i, j)` where `i < j` and `nums[i] <= nums[j] + diff`.

Now we need to count how many valid pairs exist. For each position `j`, we want to count how many positions `i` before it (where `i < j`) satisfy `nums[i] <= nums[j] + diff`. This is essentially asking: "For each element, how many elements before it are at most `nums[j] + diff`?"

This type of query - counting elements less than or equal to a certain value in a dynamic set - is a classic use case for data structures like Binary Indexed Tree (Fenwick Tree) or [Segment Tree](/problems/segment_tree_intro). As we iterate through the array from left to right:
1. For each position `j`, we query how many elements we've seen so far are `<= nums[j] + diff`
2. Then we add `nums[j]` to our data structure for future queries

The Binary Indexed Tree is particularly efficient here because it supports both point updates and range queries in `O(log n)` time. Since the values might be negative or very large, we apply an offset (like 40000) to ensure all values are positive and within a reasonable range for the tree implementation.

**Learn more about [Segment Tree](/problems/segment_tree_intro), [Binary Search](/problems/binary_search_intro), [Divide and Conquer](/problems/divide_and_conquer_intro) and [Merge Sort](/problems/merge_k_sorted_lists) patterns.**

## Solution Approach

The solution implements the intuition using a Binary Indexed Tree (Fenwick Tree) to efficiently count valid pairs.

**Step 1: Transform the Problem**

First, we create a transformed view of the problem. For each index, we calculate `v = nums1[i] - nums2[i]`. This transforms our inequality into finding pairs where `v[i] <= v[j] + diff` with `i < j`.

**Step 2: Binary Indexed Tree Setup**

The `BinaryIndexedTree` class is initialized with a size of `10^5` to handle the range of possible values. The tree supports:
- `update(x, delta)`: Adds `delta` to the count at position `x`
- `query(x)`: Returns the sum of all elements from index 1 to `x`
- `lowbit(x)`: Helper function that returns `x & -x`, which isolates the rightmost set bit

**Step 3: Process Elements and Count Pairs**

The main algorithm iterates through both arrays simultaneously:

```python
for a, b in zip(nums1, nums2):
    v = a - b  # Calculate the difference
    ans += tree.query(v + diff + 40000)  # Count valid pairs
    tree.update(v + 40000, 1)  # Add current element to tree
```

For each position `j`:
1. Calculate `v = nums1[j] - nums2[j]`
2. Query the tree for count of all elements `<= v + diff`. The offset of 40000 is added to handle negative values
3. Add this count to our answer (these are all valid pairs with current position as `j`)
4. Update the tree by adding the current `v` value (with offset) for future queries

**Why the Offset of 40000?**

Since the Binary Indexed Tree uses 1-based indexing and needs positive indices, we add 40000 to shift all possible values into a positive range. This ensures that even if `v` is negative (as low as around -40000), the index `v + 40000` remains positive.

**Time Complexity**: `O(n log n)` where `n` is the length of the arrays. Each query and update operation takes `O(log n)` time, and we perform `n` such operations.

**Space Complexity**: `O(10^5)` for the Binary Indexed Tree array, which is constant space relative to the input size.

## Example Walkthrough

Let's walk through a concrete example to understand how the solution works:

**Given:**
- `nums1 = [3, 2, 5]`
- `nums2 = [2, 2, 1]`
- `diff = 1`

**Step 1: Transform the arrays**

First, we calculate `v[i] = nums1[i] - nums2[i]` for each index:
- `v[0] = 3 - 2 = 1`
- `v[1] = 2 - 2 = 0`
- `v[2] = 5 - 1 = 4`

So our transformed array is `v = [1, 0, 4]`.

**Step 2: Process each element with Binary Indexed Tree**

We'll use a Binary Indexed Tree (BIT) to count valid pairs. For simplicity in this example, let's use a smaller offset of 10 instead of 40000.

**Iteration 1 (i=0, v=1):**
- Query: How many elements in BIT are ≤ `v + diff + offset = 1 + 1 + 10 = 12`?
  - BIT is empty, so count = 0
- Update: Add `v + offset = 1 + 10 = 11` to BIT
- Running answer: 0

**Iteration 2 (i=1, v=0):**
- Query: How many elements in BIT are ≤ `v + diff + offset = 0 + 1 + 10 = 11`?
  - BIT contains one element at index 11, so count = 1
  - This represents the valid pair (0, 1) where `v[0]=1 ≤ v[1]+diff=0+1=1` ✓
- Update: Add `v + offset = 0 + 10 = 10` to BIT
- Running answer: 0 + 1 = 1

**Iteration 3 (i=2, v=4):**
- Query: How many elements in BIT are ≤ `v + diff + offset = 4 + 1 + 10 = 15`?
  - BIT contains elements at indices 10 and 11, so count = 2
  - This represents valid pairs:
    - (0, 2): `v[0]=1 ≤ v[2]+diff=4+1=5` ✓
    - (1, 2): `v[1]=0 ≤ v[2]+diff=4+1=5` ✓
- Update: Add `v + offset = 4 + 10 = 14` to BIT
- Running answer: 1 + 2 = 3

**Result:** The total number of valid pairs is 3.

**Verification:**
Let's verify our three pairs using the original condition:
1. Pair (0,1): `nums1[0]-nums1[1] = 3-2 = 1` ≤ `nums2[0]-nums2[1]+diff = 2-2+1 = 1` ✓
2. Pair (0,2): `nums1[0]-nums1[2] = 3-5 = -2` ≤ `nums2[0]-nums2[2]+diff = 2-1+1 = 2` ✓
3. Pair (1,2): `nums1[1]-nums1[2] = 2-5 = -3` ≤ `nums2[1]-nums2[2]+diff = 2-1+1 = 2` ✓

All three pairs satisfy the condition, confirming our answer of 3.

## Python Solution

```python
from typing import List


class BinaryIndexedTree:
    """Binary Indexed Tree (Fenwick Tree) for efficient prefix sum queries and updates."""

    def __init__(self, n: int) -> None:
        """
        Initialize the Binary Indexed Tree.

        Args:
            n: The size of the tree (1-indexed, so actual array size is n+1)
        """
        self.n = n
        self.c = [0] * (n + 1)  # Tree array, 1-indexed

    @staticmethod
    def lowbit(x: int) -> int:
        """
        Get the lowest set bit of x.

        Args:
            x: Input integer

        Returns:
            The value of the lowest set bit
        """
        return x & -x

    def update(self, x: int, delta: int) -> None:
        """
        Update the value at index x by adding delta.

        Args:
            x: The index to update (1-indexed)
            delta: The value to add
        """
        while x <= self.n:
            self.c[x] += delta
            x += BinaryIndexedTree.lowbit(x)

    def query(self, x: int) -> int:
        """
        Query the prefix sum from index 1 to x.

        Args:
            x: The end index of the range (1-indexed)

        Returns:
            The sum of elements from index 1 to x
        """
        sum_value = 0
        while x > 0:
            sum_value += self.c[x]
            x -= BinaryIndexedTree.lowbit(x)
        return sum_value


class Solution:
    def numberOfPairs(self, nums1: List[int], nums2: List[int], diff: int) -> int:
        """
        Count the number of pairs (i, j) where i < j and
        nums1[i] - nums1[j] <= nums2[i] - nums2[j] + diff.

        This can be rewritten as: (nums1[i] - nums2[i]) <= (nums1[j] - nums2[j]) + diff

        Args:
            nums1: First array of integers
            nums2: Second array of integers
            diff: The difference threshold

        Returns:
            The count of valid pairs
        """
        # Create a BIT with size 10^5 to handle the range of possible values
        tree = BinaryIndexedTree(100000)

        # Initialize answer counter
        answer = 0

        # Process each pair of elements from nums1 and nums2
        for a, b in zip(nums1, nums2):
            # Calculate the difference value for current position
            value = a - b

            # Query how many previous values are <= value + diff
            # Add 40000 as offset to handle negative values (map to positive range)
            answer += tree.query(value + diff + 40000)

            # Update the tree with current value (also with offset)
            tree.update(value + 40000, 1)

        return answer
```

## Time and Space Complexity

**Time Complexity:** `O(n × log n)`

The solution iterates through the input arrays once with `n` elements. For each element:
- The `query` operation on the Binary Indexed Tree takes `O(log n)` time, as it traverses at most `log n` nodes by repeatedly subtracting `lowbit(x)` until reaching 0
- The `update` operation also takes `O(log n)` time, as it traverses at most `log n` nodes by repeatedly adding `lowbit(x)` until exceeding the tree size

Since we perform both operations for each of the `n` elements, the total time complexity is `O(n × log n)`.

**Space Complexity:** `O(10^5)`

The space complexity is determined by:
- The Binary Indexed Tree array `c` which has a fixed size of `10^5 + 1` elements, giving `O(10^5)` space
- Other variables like `ans`, `v`, `a`, `b` use `O(1)` space

Therefore, the overall space complexity is `O(10^5)`, which is constant with respect to the input size `n`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Handling of Negative Values**

**The Pitfall:** The most common mistake is forgetting to handle negative values properly when using a Binary Indexed Tree. Since BIT uses 1-based indexing and requires positive indices, directly using `v = nums1[i] - nums2[i]` as an index will cause runtime errors when `v` is negative.

**Example of Wrong Implementation:**
```python
# WRONG - This will fail when v is negative
for a, b in zip(nums1, nums2):
    v = a - b
    ans += tree.query(v + diff)  # Can be negative!
    tree.update(v, 1)  # Can be negative!
```

**The Solution:** Add a sufficient offset to ensure all indices are positive. The code uses 40000 as an offset because:
- The constraint typically has `|nums1[i]|, |nums2[i]| <= 10^4`
- The minimum possible value of `v = nums1[i] - nums2[i]` is `-10^4 - 10^4 = -2*10^4`
- Adding 40000 ensures all indices are positive: `-20000 + 40000 = 20000 > 0`

### 2. **Off-by-One Error in BIT Size**

**The Pitfall:** Declaring the BIT with insufficient size or not accounting for the offset properly.

**Example of Wrong Implementation:**
```python
# WRONG - Size might be too small
tree = BinaryIndexedTree(50000)  # Not enough for all possible values with offset
```

**The Solution:** Calculate the maximum possible range:
- With values ranging from -20000 to 20000 and an offset of 40000
- The actual range becomes 20000 to 60000
- Add buffer for the diff value: could go up to 60000 + 10000 = 70000
- Use 100000 to be safe: `tree = BinaryIndexedTree(100000)`

### 3. **Forgetting to Apply Offset Consistently**

**The Pitfall:** Applying the offset during query but forgetting it during update, or vice versa.

**Example of Wrong Implementation:**
```python
# WRONG - Inconsistent offset usage
for a, b in zip(nums1, nums2):
    v = a - b
    ans += tree.query(v + diff + 40000)  # Offset applied
    tree.update(v, 1)  # FORGOT offset here!
```

**The Solution:** Always apply the same offset to both query and update operations:
```python
# CORRECT
ans += tree.query(v + diff + 40000)
tree.update(v + 40000, 1)
```

### 4. **Misunderstanding the Inequality Direction**

**The Pitfall:** Confusing which elements should be counted in the query. The transformed inequality `v[i] <= v[j] + diff` means for each `j`, we need to count all previous `i` where `v[i] <= v[j] + diff`.

**Example of Wrong Implementation:**
```python
# WRONG - Querying the wrong range
for a, b in zip(nums1, nums2):
    v = a - b
    ans += tree.query(v + 40000)  # Should be v + diff + 40000!
    tree.update(v + 40000, 1)
```

**The Solution:** Remember that we're looking for all previous values that are at most `v + diff`, not just `v`:
```python
ans += tree.query(v + diff + 40000)  # Correct: v + diff
```

