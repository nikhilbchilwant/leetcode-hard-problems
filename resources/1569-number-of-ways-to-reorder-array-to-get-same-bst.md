# 1569. Number of Ways to Reorder Array to Get Same BST

> Source: https://algo.monster/liteproblems/1569

## Problem Statement

You are given an array `nums` that represents a permutation of integers from `1` to `n`. When you insert the elements of `nums` in order into an initially empty Binary Search [Tree](/problems/tree_intro) (BST), you get a specific tree structure.

The task is to find how many different ways you can reorder the elements in `nums` such that when you insert them into an empty BST in the new order, you get the exact same BST structure as the original.

For example, with `nums = [2,1,3]`:
- Inserting in order `[2,1,3]` creates a BST with `2` as root, `1` as left child, and `3` as right child
- The reordering `[2,3,1]` produces the same BST structure (insert 2 first as root, then 3 goes right, then 1 goes left)
- But `[3,2,1]` produces a different BST (3 becomes root instead of 2)

The key insight is that to maintain the same BST structure:
- The first element must remain the same (it's always the root)
- All elements less than the root must go to the left subtree
- All elements greater than the root must go to the right subtree
- The relative insertion order within left and right subtrees must preserve their respective BST structures

Since the answer can be very large, return it modulo `10^9 + 7`.

The solution uses a recursive approach with [combinatorics](/problems/backtracking). For any given array, it:
1. Separates elements into left subtree (values < root) and right subtree (values > root)
2. Calculates the number of valid orderings for each subtree recursively
3. Uses combinations `C(m+n, m)` to determine how many ways to interleave `m` left elements with `n` right elements while maintaining their relative orders
4. Multiplies these values together: `C(m+n, m) × ways(left) × ways(right)`

The final answer subtracts 1 because the original ordering is not considered a "reordering."

## Intuition

To understand why certain reorderings produce the same BST, let's think about what determines a BST's structure. When we insert elements sequentially, each element finds its position by comparing with existing nodes - going left if smaller, right if larger. The crucial observation is that **the first element always becomes the root**, and this root permanently divides all subsequent elements into two groups: those that will go to the left subtree (smaller values) and those that will go to the right subtree (larger values).

Once we insert the root, it doesn't matter in what order we insert elements `1` and `3` after `2` in the example `[2,1,3]` - element `1` will always end up as the left child and `3` as the right child. This is because their final positions are determined solely by their values relative to `2`.

This leads to a key insight: **we can recursively break down the problem**. For any array with root `r`, we can identify:
- Left elements: all values < `r` 
- Right elements: all values > `r`

Now, how many valid orderings exist? We need the root first, but after that, we can interleave the left and right elements in any way, as long as:
1. The relative order among left elements produces the correct left subtree
2. The relative order among right elements produces the correct right subtree

If we have `m` elements for the left subtree and `n` for the right, we're essentially choosing `m` positions out of `m+n` total positions for the left elements. This is the combination `C(m+n, m)`. The remaining positions automatically go to the right elements.

For example, with left = `[1]` and right = `[3,4]`, we can arrange them as:
- `[1,3,4]` - choosing position 0 for left
- `[3,1,4]` - choosing position 1 for left  
- `[3,4,1]` - choosing position 2 for left

That's `C(3,1) = 3` ways.

The total number of valid arrangements becomes:
`C(m+n, m) × (valid arrangements of left subtree) × (valid arrangements of right subtree)`

We apply this formula recursively until we reach base cases (arrays with 0 or 1 element have exactly 1 arrangement). The final answer subtracts 1 because we want reorderings different from the original.

**Learn more about [Tree](/problems/tree_intro), [Union Find](/problems/dsu_intro), [Binary Search Tree](/problems/bst_intro), [Memoization](/problems/memoization_intro), [Math](/problems/math-basics), [Divide and Conquer](/problems/divide_and_conquer_intro), [Dynamic Programming](/problems/dynamic_programming_intro), [Binary Tree](/problems/binary_tree_min_depth) and [Combinatorics](/problems/backtracking) patterns.**

## Solution Approach

The implementation consists of two main components: preprocessing combinations and a recursive DFS function.

**Step 1: Preprocessing Combinations Table**

First, we build a 2D array `c` where `c[i][j]` represents `C(i, j)` - the number of ways to choose `j` items from `i` items. We use Pascal's triangle formula:
- Base case: `c[i][0] = 1` for all `i` (choosing 0 items has 1 way)
- Recurrence: `c[i][j] = c[i-1][j] + c[i-1][j-1]`

This preprocessing allows us to look up combination values in O(1) time during recursion.

```python
c = [[0] * n for _ in range(n)]
c[0][0] = 1
for i in range(1, n):
    c[i][0] = 1
    for j in range(1, i + 1):
        c[i][j] = (c[i - 1][j] + c[i - 1][j - 1]) % mod
```

**Step 2: Recursive DFS Function**

The `dfs(nums)` function calculates the number of valid arrangements for array `nums`:

1. **Base Case**: If the array has less than 2 elements, return 1 (only one way to arrange)

2. **Partition**: Split the array based on the first element (root):
   - `left = [x for x in nums if x < nums[0]]` - elements going to left subtree
   - `right = [x for x in nums if x > nums[0]]` - elements going to right subtree

3. **Recursive Calls**: Calculate valid arrangements for each subtree:
   - `a = dfs(left)` - number of valid arrangements for left subtree
   - `b = dfs(right)` - number of valid arrangements for right subtree

4. **Combine Results**: Use the formula:
   ```
   result = C(m+n, m) × a × b
   ```
   where `m = len(left)` and `n = len(right)`

The multiplication `C(m+n, m) × a × b` represents:
- `C(m+n, m)`: ways to interleave left and right elements
- `a`: valid internal arrangements of left subtree
- `b`: valid internal arrangements of right subtree

**Step 3: Final Answer**

The function `dfs(nums)` returns the total number of ways to arrange elements (including the original). Since we want only the reorderings (excluding the original), we subtract 1:

```python
return (dfs(nums) - 1 + mod) % mod
```

The `+ mod` before the final modulo ensures the result remains positive even after subtraction.

**Time Complexity**: O(n²) where n is the length of the array - we visit each element once in recursion, and for each recursive call, we partition the array in O(n) time.

**Space Complexity**: O(n²) for the combination table plus O(n) for recursion stack depth.

## Example Walkthrough

Let's walk through the solution with `nums = [3, 1, 2, 4]`.

**Step 1: Build Combinations Table**
We create a table for C(n,k) values. For n=4, we need:
- C(0,0) = 1
- C(1,0) = 1, C(1,1) = 1  
- C(2,0) = 1, C(2,1) = 2, C(2,2) = 1
- C(3,0) = 1, C(3,1) = 3, C(3,2) = 3, C(3,3) = 1

**Step 2: Process dfs([3, 1, 2, 4])**
- Root = 3 (first element)
- Partition around 3:
  - left = [1, 2] (elements < 3)
  - right = [4] (elements > 3)
- Need to calculate: C(2+1, 2) × dfs([1,2]) × dfs([4])

**Step 3: Process dfs([1, 2])** 
- Root = 1
- Partition around 1:
  - left = [] (no elements < 1)
  - right = [2] (elements > 1)
- Calculate: C(0+1, 0) × dfs([]) × dfs([2])
- dfs([]) = 1 (base case)
- dfs([2]) = 1 (single element)
- Result: C(1,0) × 1 × 1 = 1 × 1 × 1 = 1

**Step 4: Process dfs([4])**
- Single element, return 1

**Step 5: Combine Results for Original Array**
- dfs([3,1,2,4]) = C(3,2) × dfs([1,2]) × dfs([4])
- = 3 × 1 × 1 = 3

This means there are 3 total ways to arrange [3,1,2,4] that produce the same BST:
1. [3, 1, 2, 4] - original order
2. [3, 1, 4, 2] - insert 4 before 2 (both go right of 1 anyway)
3. [3, 4, 1, 2] - insert 4 before both 1 and 2

Let's verify [3, 4, 1, 2] produces the same BST:
- Insert 3: becomes root
- Insert 4: 4 > 3, goes right of 3
- Insert 1: 1 < 3, goes left of 3
- Insert 2: 2 < 3 (go left), 2 > 1 (go right of 1)

This creates the same structure as the original!

**Step 6: Final Answer**
Total arrangements - 1 (excluding original) = 3 - 1 = 2

The answer is 2 different ways to reorder the array.

## Python Solution

```python
class Solution:
    def numOfWays(self, nums: List[int]) -> int:
        def count_bst_permutations(arr):
            """
            Count the number of permutations that produce the same BST structure.
          
            Args:
                arr: Current array/subtree to process
          
            Returns:
                Number of valid permutations for this subtree
            """
            # Base case: arrays with less than 2 elements have only 1 permutation
            if len(arr) < 2:
                return 1
          
            # Split array into left and right subtrees based on first element (root)
            root = arr[0]
            left_subtree = [x for x in arr if x < root]
            right_subtree = [x for x in arr if x > root]
          
            # Get sizes of left and right subtrees
            left_size = len(left_subtree)
            right_size = len(right_subtree)
          
            # Recursively count permutations for left and right subtrees
            left_permutations = count_bst_permutations(left_subtree)
            right_permutations = count_bst_permutations(right_subtree)
          
            # Calculate total permutations using combinations
            # C(left_size + right_size, left_size) represents ways to interleave
            # left and right subtree elements while maintaining their relative order
            total_size = left_size + right_size
            combinations = binomial_coefficients[total_size][left_size]
          
            # Total permutations = combinations * left_permutations * right_permutations
            result = (combinations * left_permutations % MOD) * right_permutations % MOD
            return result
      
        # Initialize constants
        n = len(nums)
        MOD = 10**9 + 7
      
        # Precompute binomial coefficients using Pascal's triangle
        # binomial_coefficients[i][j] represents C(i, j) = i! / (j! * (i-j)!)
        binomial_coefficients = [[0] * n for _ in range(n)]
        binomial_coefficients[0][0] = 1
      
        for i in range(1, n):
            binomial_coefficients[i][0] = 1  # C(i, 0) = 1
            for j in range(1, i + 1):
                # Pascal's triangle formula: C(i, j) = C(i-1, j) + C(i-1, j-1)
                binomial_coefficients[i][j] = (
                    binomial_coefficients[i - 1][j] + 
                    binomial_coefficients[i - 1][j - 1]
                ) % MOD
      
        # Calculate total permutations and subtract 1 (excluding original arrangement)
        total_permutations = count_bst_permutations(nums)
        return (total_permutations - 1 + MOD) % MOD
```

## Time and Space Complexity

**Time Complexity: `O(n²)`**

The time complexity analysis consists of two main parts:

1. **Preprocessing the combination table**: The nested loops to build the combination table `c` take `O(n²)` time, where we compute `C(i,j)` for all valid `i` and `j` values up to `n`.

2. **DFS traversal**: 
   - Each recursive call processes each element in the input array exactly once to partition it into left and right subarrays (using list comprehensions to filter elements less than and greater than `nums[0]`).
   - Across all recursive calls, every element is visited and processed `O(n)` times in total (once at each level it appears).
   - The partitioning at each level takes linear time relative to the size of that subarray.
   - Overall, this contributes `O(n²)` time in the worst case.

The dominant operation is `O(n²)` from both the preprocessing and the recursive processing.

**Space Complexity: `O(n²)`**

The space complexity breaks down as follows:

1. **Combination table**: The 2D array `c` of size `n × n` requires `O(n²)` space to store all the binomial coefficients.

2. **Recursion stack**: In the worst case (skewed tree), the recursion depth can be `O(n)`.

3. **Temporary arrays**: At each recursive level, new arrays `left` and `right` are created. In total across all recursive calls, this uses `O(n)` space since each element appears in exactly one array at each recursion level.

The dominant space usage is the `O(n²)` combination table, making the overall space complexity `O(n²)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Modulo Operation on Subtraction**

**Pitfall**: When subtracting 1 from the result to exclude the original arrangement, directly doing `(dfs(nums) - 1) % MOD` can produce negative results if `dfs(nums)` returns 0 or 1.

**Why it happens**: In modular arithmetic, if `dfs(nums) % MOD` equals 0, then `(0 - 1) % MOD` gives -1 in many programming languages, not the expected `MOD - 1`.

**Solution**: Always add `MOD` before taking the final modulo:
```python
return (total_permutations - 1 + MOD) % MOD
```

### 2. **Integer Overflow in Multiplication**

**Pitfall**: Computing `combinations * left_permutations * right_permutations` without intermediate modulo operations can cause integer overflow even in Python (though Python handles big integers, it's inefficient).

**Why it happens**: The multiplication of three potentially large numbers can exceed typical integer limits before the modulo operation is applied.

**Solution**: Apply modulo after each multiplication:
```python
# Wrong - can overflow or be inefficient
result = (combinations * left_permutations * right_permutations) % MOD

# Correct - prevents overflow
result = (combinations * left_permutations % MOD) * right_permutations % MOD
```

### 3. **Off-by-One Error in Combinations Table**

**Pitfall**: Incorrectly sizing the combinations table or using wrong indices when accessing it.

**Why it happens**: The combinations table needs to handle cases where we might need `C(n-1, k)` where `n` is the array length. If the table is sized as `[n-1][n-1]`, it will cause index out of bounds errors.

**Solution**: Size the table as `[n][n]` and ensure proper bounds in the loop:
```python
# Correct sizing
binomial_coefficients = [[0] * n for _ in range(n)]

# Correct loop bounds
for i in range(1, n):
    for j in range(1, i + 1):  # j should not exceed i
```

### 4. **Forgetting Base Cases**

**Pitfall**: Not handling arrays with 0 or 1 elements properly in the recursive function.

**Why it happens**: The recursive logic assumes there's a root and possible left/right subtrees, which doesn't apply to empty or single-element arrays.

**Solution**: Always check for base cases at the beginning:
```python
if len(arr) < 2:  # Handles both empty and single-element arrays
    return 1
```

### 5. **Incorrect Array Partitioning**

**Pitfall**: Including the root element in left or right subtree partitions, or using wrong comparison operators.

**Why it happens**: Confusion about whether to use `<`, `<=`, `>`, or `>=` when partitioning around the root.

**Solution**: Use strict inequalities and skip the root element:
```python
root = arr[0]
left_subtree = [x for x in arr[1:] if x < root]  # Note arr[1:] to skip root
right_subtree = [x for x in arr[1:] if x > root]

# Or iterate through all elements but skip when x equals root
left_subtree = [x for x in arr if x < root]  # This naturally excludes root
right_subtree = [x for x in arr if x > root]
```

