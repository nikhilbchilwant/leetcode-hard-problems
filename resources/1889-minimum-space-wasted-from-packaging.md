# 1889. Minimum Space Wasted From Packaging

> Source: https://algo.monster/liteproblems/1889

## Problem Statement

You need to pack `n` packages into boxes, with exactly one package per box. There are `m` different suppliers, and each supplier produces boxes of various sizes with unlimited quantity available.

Given:
- An array `packages` where `packages[i]` represents the size of the `i-th` package
- A 2D array `boxes` where `boxes[j]` contains all the different box sizes that supplier `j` can provide

Rules:
- A package can only fit in a box if the package size ≤ box size
- You must choose exactly ONE supplier and use only their boxes
- Each package must go into a separate box

The goal is to minimize the total wasted space, where:
- Wasted space for one package = `box size - package size`
- Total wasted space = sum of wasted space across all packages

For example, with packages `[2,3,5]` and a supplier offering boxes `[4,8]`:
- Package size 2 → box size 4 (waste = 4-2 = 2)
- Package size 3 → box size 4 (waste = 4-3 = 1)
- Package size 5 → box size 8 (waste = 8-5 = 3)
- Total waste = 2 + 1 + 3 = 6

Return:
- The minimum total wasted space when choosing the optimal supplier
- `-1` if no supplier can accommodate all packages
- Since the answer can be large, return it modulo `10^9 + 7`

## Intuition

The key insight is that we need to try each supplier independently and find which one gives us the minimum waste. Since we can only choose one supplier, we need to check if that supplier can accommodate all packages first.

For each supplier, the optimal strategy is to match each package with the smallest possible box that can fit it. This minimizes waste per package. If we sort both packages and boxes, we can efficiently assign packages to boxes using a greedy approach.

Consider sorted packages `[2, 3, 5]` and sorted boxes `[4, 8]`:
- All packages with size ≤ 4 should go in box size 4
- All remaining packages with size ≤ 8 should go in box size 8

This suggests we can process boxes in ascending order. For each box size `b`, we find all packages that:
1. Haven't been assigned yet
2. Can fit in this box (size ≤ `b`)

These packages should all use this box size since it's the smallest available box that fits them.

To efficiently find which packages fit in each box size, we can use [binary search](/problems/binary_search_intro). After [sorting](/problems/sorting_summary) packages, `bisect_right(packages, b)` tells us the rightmost position where we could insert `b`, which gives us the count of packages ≤ `b`.

The total space used by a supplier equals the sum of all box sizes used. The waste is then `total_space_used - sum(packages)`. Since `sum(packages)` is constant regardless of supplier choice, minimizing total space used minimizes waste.

We process each supplier, calculate their total space usage, track the minimum, and return the corresponding waste. If no supplier can fit all packages (detected when the largest package exceeds the supplier's largest box), we skip that supplier.

**Learn more about [Binary Search](/problems/binary_search_intro), [Prefix Sum](/problems/subarray_sum) and [Sorting](/problems/sorting_summary) patterns.**

## Solution Approach

The implementation follows these steps:

1. **Initialize variables**: Set `mod = 10^9 + 7` for the modulo operation, and `ans = inf` to track the minimum total space used across all suppliers.

2. **Sort packages**: `packages.sort()` enables us to use [binary search](/problems/binary_search_intro) and ensures we can check if a supplier can accommodate all packages by comparing the largest package with the supplier's largest box.

3. **Process each supplier**: For each supplier's box list:
   - **Sort boxes**: `box.sort()` allows us to process boxes from smallest to largest
   - **Early termination check**: If `packages[-1] > box[-1]` (largest package > largest box), skip this supplier as it cannot accommodate all packages

4. **Calculate total space for current supplier**:
   - Initialize `s = 0` (total space used) and `i = 0` (starting index for packages)
   - For each box size `b` in ascending order:
     - Use `j = bisect_right(packages, b, lo=i)` to find how many packages from index `i` onwards can fit in box size `b`
     - These are packages in range `[i, j)` that haven't been assigned yet and have size ≤ `b`
     - Add space used: `s += (j - i) * b` (number of packages × box size)
     - Update starting position: `i = j` for the next iteration

5. **Track minimum**: Update `ans = min(ans, s)` to keep the minimum total space across all suppliers

6. **Calculate final result**:
   - If `ans == inf`, no supplier could accommodate all packages, return `-1`
   - Otherwise, the waste is `total_space_used - total_package_size`
   - Return `(ans - sum(packages)) % mod`

The [binary search](/problems/binary_search_intro) optimization is crucial: `bisect_right(packages, b, lo=i)` finds packages in range `[i, n)` with size ≤ `b` in `O(log n)` time. The `lo=i` parameter ensures we only search among unassigned packages, making each package get assigned exactly once across all boxes of a supplier.

Time complexity: `O(n log n + m * k * log n)` where `n` is number of packages, `m` is number of suppliers, and `k` is average number of box sizes per supplier.

## Example Walkthrough

Let's walk through a concrete example with packages `[3, 5, 8, 10, 11, 12]` and two suppliers:
- Supplier 0: boxes `[15, 10, 20]`
- Supplier 1: boxes `[12, 11, 8, 9]`

**Step 1: Sort packages**
Packages after sorting: `[3, 5, 8, 10, 11, 12]`

**Step 2: Try Supplier 0**
Sort boxes: `[10, 15, 20]`
Check if feasible: largest package (12) ≤ largest box (20) ✓

Process each box size:
- Box size 10 (i=0):
  - Binary search: packages ≤ 10 are at indices [0,4) → packages `[3, 5, 8, 10]`
  - Space used: 4 packages × 10 = 40
  - Update i=4

- Box size 15 (i=4):
  - Binary search: packages ≤ 15 from index 4 are at indices [4,6) → packages `[11, 12]`
  - Space used: 2 packages × 15 = 30
  - Update i=6

- Box size 20 (i=6):
  - No remaining packages
  - Space used: 0

Total space for Supplier 0: 40 + 30 = 70

**Step 3: Try Supplier 1**
Sort boxes: `[8, 9, 11, 12]`
Check if feasible: largest package (12) ≤ largest box (12) ✓

Process each box size:
- Box size 8 (i=0):
  - Binary search: packages ≤ 8 are at indices [0,3) → packages `[3, 5, 8]`
  - Space used: 3 packages × 8 = 24
  - Update i=3

- Box size 9 (i=3):
  - Binary search: packages ≤ 9 from index 3 are at indices [3,3) → no packages
  - Space used: 0
  - Update i=3

- Box size 11 (i=3):
  - Binary search: packages ≤ 11 from index 3 are at indices [3,5) → packages `[10, 11]`
  - Space used: 2 packages × 11 = 22
  - Update i=5

- Box size 12 (i=5):
  - Binary search: packages ≤ 12 from index 5 are at indices [5,6) → package `[12]`
  - Space used: 1 package × 12 = 12
  - Update i=6

Total space for Supplier 1: 24 + 22 + 12 = 58

**Step 4: Calculate minimum waste**
- Supplier 0 total space: 70
- Supplier 1 total space: 58 (minimum)
- Sum of packages: 3 + 5 + 8 + 10 + 11 + 12 = 49
- Minimum waste: 58 - 49 = 9

Return: 9

## Python Solution

```python
from typing import List
from math import inf
from bisect import bisect_right

class Solution:
    def minWastedSpace(self, packages: List[int], boxes: List[List[int]]) -> int:
        MOD = 10**9 + 7
        min_wasted_space = inf

        # Sort packages to enable binary search
        packages.sort()

        # Try each supplier's box sizes
        for box_sizes in boxes:
            # Sort current supplier's box sizes
            box_sizes.sort()

            # Skip this supplier if their largest box can't fit the largest package
            if packages[-1] > box_sizes[-1]:
                continue

            # Calculate total space used by this supplier's boxes
            total_space_used = 0
            package_index = 0

            # For each box size, find how many packages it will contain
            for box_size in box_sizes:
                # Find the rightmost package that fits in current box size
                next_package_index = bisect_right(packages, box_size, lo=package_index)

                # Add space used: number of packages * box size
                packages_count = next_package_index - package_index
                total_space_used += packages_count * box_size

                # Update starting position for next box size
                package_index = next_package_index

            # Update minimum total space used
            min_wasted_space = min(min_wasted_space, total_space_used)

        # If no valid supplier found, return -1
        if min_wasted_space == inf:
            return -1

        # Calculate wasted space: total space used - actual package sizes
        total_package_size = sum(packages)
        wasted_space = (min_wasted_space - total_package_size) % MOD

        return wasted_space
```

## Time and Space Complexity

**Time Complexity:** `O(n log n + m * k log k + m * n log n)` where:
- `n` is the length of the packages array
- `m` is the number of suppliers (length of boxes array)
- `k` is the average number of boxes per supplier

Breaking down the operations:
- Sorting packages: `O(n log n)`
- For each supplier box array (m iterations):
  - Sorting each box array: `O(k log k)`
  - For each box size in the sorted array (k iterations):
    - Binary search using `bisect_right`: `O(log n)`
  - Total per supplier: `O(k log k + k * log n)`
- Overall: `O(n log n + m * (k log k + k * log n))`

In the worst case where `k ≈ n`, this becomes `O(n log n + m * n log n)` = `O((m + 1) * n log n)`

**Space Complexity:** `O(1)` or `O(log n)` depending on the sorting algorithm implementation
- The algorithm sorts arrays in-place
- Variables `ans`, `s`, `i`, `j`, `mod` use constant space
- The sorting operation may use `O(log n)` space for the recursion stack in languages like Python (Timsort)
- No additional data structures are created

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Integer Overflow When Calculating Total Space**

The most critical pitfall in this solution is that `total_space_used` can exceed Python's integer limits before applying the modulo operation. When you have many packages and large box sizes, the multiplication `packages_count * box_size` and the accumulation in `total_space_used` can produce extremely large numbers.

**Why this happens:**
- Constraints typically allow up to 10^5 packages and box sizes up to 10^9
- The total space calculation accumulates these products: `total_space_used += packages_count * box_size`
- This can result in values around 10^14 or larger before taking modulo

**The problem:**
```python
# Current problematic approach
total_space_used += packages_count * box_size  # Can overflow
# ...later...
wasted_space = (min_wasted_space - total_package_size) % MOD  # Modulo applied too late
```

**Solution:**
Apply modulo during accumulation to keep numbers manageable:

```python
# Corrected approach
total_space_used = (total_space_used + packages_count * box_size) % MOD
```

However, this creates another issue: you can't directly compare modulo values for finding the minimum. The complete solution requires tracking both the actual minimum and applying modulo only at the final return.

### 2. **Incorrect Minimum Comparison After Modulo**

If you apply modulo during calculation, comparing suppliers becomes incorrect:

```python
# Wrong: Can't find true minimum after modulo
for box_sizes in boxes:
    total_space_used = 0
    # ... calculate with modulo ...
    total_space_used = (total_space_used + packages_count * box_size) % MOD
    min_wasted_space = min(min_wasted_space, total_space_used)  # Incorrect comparison!
```

**Correct Solution:**

```python
class Solution:
    def minWastedSpace(self, packages: List[int], boxes: List[List[int]]) -> int:
        MOD = 10**9 + 7
        min_wasted_space = inf

        packages.sort()
        total_package_size = sum(packages)

        for box_sizes in boxes:
            box_sizes.sort()

            if packages[-1] > box_sizes[-1]:
                continue

            # Keep actual value for comparison
            total_space_used = 0
            package_index = 0

            for box_size in box_sizes:
                next_package_index = bisect_right(packages, box_size, lo=package_index)
                packages_count = next_package_index - package_index
                total_space_used += packages_count * box_size
                package_index = next_package_index

            # Compare actual values
            min_wasted_space = min(min_wasted_space, total_space_used)

        if min_wasted_space == inf:
            return -1

        # Apply modulo only at the end
        wasted_space = (min_wasted_space - total_package_size) % MOD
        return wasted_space
```

### 3. **Not Handling the Case Where All Packages Are Already Processed**

A subtle issue occurs when iterating through box sizes if all packages have already been assigned:

```python
# Potential inefficiency
for box_size in box_sizes:
    next_package_index = bisect_right(packages, box_size, lo=package_index)
    # If package_index == len(packages), this still runs but does nothing
```

**Optimization:**
```python
for box_size in box_sizes:
    if package_index >= len(packages):
        break  # All packages already assigned
    next_package_index = bisect_right(packages, box_size, lo=package_index)
    # ... rest of the logic
```

This early termination can improve performance when a supplier has many more box sizes than necessary.

