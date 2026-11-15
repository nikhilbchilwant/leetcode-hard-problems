# 2449. Minimum Number of Operations to Make Arrays Similar

> Source: https://algo.monster/liteproblems/2449

## Problem Statement

You have two positive integer arrays `nums` and `target` of equal length. Your goal is to make `nums` "similar" to `target` through a series of operations.

In each operation, you can:
- Choose two different indices `i` and `j` (where `0 <= i, j < nums.length`)
- Add `2` to `nums[i]`
- Subtract `2` from `nums[j]`

Two arrays are considered **similar** when they contain the same elements with the same frequencies. For example, `[1, 2, 3]` and `[3, 1, 2]` are similar because they both have one occurrence each of 1, 2, and 3.

The task is to find the minimum number of operations needed to transform `nums` into an array that is similar to `target`.

The key insight is that each operation maintains the sum of the array (adding 2 to one element and subtracting 2 from another keeps the total unchanged). Also, operations preserve the parity of elements - odd numbers stay odd and even numbers stay even, since we're always adding or subtracting 2.

The solution uses these properties by:
1. Separating elements by parity (odd and even)
2. [Sorting](/problems/sorting_summary) both arrays with the same criteria
3. Pairing corresponding elements after sorting
4. Calculating the total adjustments needed

Since each operation changes two elements by a combined difference of 4 (one increases by 2, another decreases by 2), the total difference divided by 4 gives us the minimum number of operations required.

## Intuition

Let's think about what happens during each operation. When we add `2` to one element and subtract `2` from another, we're essentially transferring "2 units" between positions. This is like redistributing values while keeping the total sum constant.

The critical observation is that adding or subtracting `2` preserves parity - an odd number remains odd (odd ± 2 = odd), and an even number remains even (even ± 2 = even). This means we can never transform an odd number into an even number or vice versa through our operations.

Since we need the arrays to be similar (same elements with same frequencies), and we can't change parity, each odd number in `nums` must eventually become some odd number in `target`, and each even number in `nums` must become some even number in `target`.

This naturally divides our problem into two independent subproblems:
- Transform all odd numbers in `nums` to match odd numbers in `target`
- Transform all even numbers in `nums` to match even numbers in `target`

Now, how should we pair elements? If we have sorted odd numbers `[1, 5, 9]` in `nums` and `[3, 7, 11]` in `target`, the optimal pairing is to match them in order: `1→3`, `5→7`, `9→11`. Why? Because any other pairing would create unnecessary "crossovers" that increase the total difference we need to fix.

For calculating the number of operations: when we transform element `a` to element `b`, we need `|a - b| / 2` adjustments to that element. But remember, each operation affects two elements - one increases and one decreases. So when we sum up all the differences `Σ|a - b|`, we're counting each operation twice (once for the element that increases, once for the element that decreases). Additionally, since each operation changes values by `2`, we divide by `2` again. This gives us `Σ|a - b| / 4` as the total number of operations.

**Learn more about [Greedy](/problems/greedy_intro) and [Sorting](/problems/sorting_summary) patterns.**

## Solution Approach

The implementation follows a clever [sorting](/problems/sorting_summary) strategy to efficiently pair elements based on their parity:

**Step 1: Custom [Sorting](/problems/sorting_summary)**

Both `nums` and `target` arrays are sorted using the same key function: `lambda x: (x & 1, x)`

- `x & 1` is a bitwise operation that returns `1` for odd numbers and `0` for even numbers
- This creates a tuple `(parity, value)` for each element
- [Sorting](/problems/sorting_summary) by this tuple groups even numbers first (parity = 0), then odd numbers (parity = 1)
- Within each group, numbers are sorted in ascending order

For example, if `nums = [8, 3, 5, 2]`:
- After [sorting](/problems/sorting_summary): `[2, 8, 3, 5]` (evens first: 2, 8; then odds: 3, 5)

**Step 2: Element Pairing**

After [sorting](/problems/sorting_summary) both arrays with the same criteria, we use `zip(nums, target)` to pair corresponding elements:
- Even numbers in `nums` are paired with even numbers in `target`
- Odd numbers in `nums` are paired with odd numbers in `target`
- The sorted order ensures optimal pairing (smallest with smallest, etc.)

**Step 3: Calculate Total Difference**

We compute `sum(abs(a - b) for a, b in zip(nums, target))`:
- For each pair `(a, b)`, calculate the absolute difference `|a - b|`
- Sum all these differences

**Step 4: Determine Number of Operations**

The final answer is the total difference divided by `4`:
- Each element needs `|a - b| / 2` units of change
- Since each operation affects two elements (one +2, one -2), and we're summing differences for all elements, we count each operation twice
- Therefore, we divide by `4` to get the actual number of operations

**Example Walkthrough:**

If `nums = [8, 12, 6]` and `target = [2, 14, 10]`:
1. After [sorting](/problems/sorting_summary): `nums = [6, 8, 12]`, `target = [2, 10, 14]` (all even, so sorted by value)
2. Pairs: `(6, 2)`, `(8, 10)`, `(12, 14)`
3. Differences: `|6-2| + |8-10| + |12-14| = 4 + 2 + 2 = 8`
4. Operations needed: `8 / 4 = 2`

This approach ensures optimal pairing and correctly accounts for the bidirectional nature of each operation.

## Example Walkthrough

Let's work through a concrete example to illustrate the solution approach.

**Given:**
- `nums = [1, 4, 7, 10]`
- `target = [3, 8, 5, 12]`

**Step 1: Custom Sorting**

First, we sort both arrays using the key `lambda x: (x & 1, x)`:

For `nums`:
- `1`: `(1 & 1, 1) = (1, 1)` (odd)
- `4`: `(4 & 1, 4) = (0, 4)` (even)
- `7`: `(7 & 1, 7) = (1, 7)` (odd)
- `10`: `(10 & 1, 10) = (0, 10)` (even)

Sorted `nums`: `[4, 10, 1, 7]` (evens first: 4, 10; then odds: 1, 7)

For `target`:
- `3`: `(3 & 1, 3) = (1, 3)` (odd)
- `8`: `(8 & 1, 8) = (0, 8)` (even)
- `5`: `(5 & 1, 5) = (1, 5)` (odd)
- `12`: `(12 & 1, 12) = (0, 12)` (even)

Sorted `target`: `[8, 12, 3, 5]` (evens first: 8, 12; then odds: 3, 5)

**Step 2: Element Pairing**

After sorting, we pair corresponding elements:
- Even pairs: `(4, 8)` and `(10, 12)`
- Odd pairs: `(1, 3)` and `(7, 5)`

**Step 3: Calculate Differences**

For each pair, calculate the absolute difference:
- `|4 - 8| = 4`
- `|10 - 12| = 2`
- `|1 - 3| = 2`
- `|7 - 5| = 2`

Total difference: `4 + 2 + 2 + 2 = 10`

**Step 4: Calculate Operations**

Number of operations = `10 / 4 = 2.5 → 2` operations (integer division)

**Verification:**

Let's verify this makes sense:
- Operation 1: Add 2 to `nums[0]=1` (becomes 3), subtract 2 from `nums[2]=7` (becomes 5)
- Operation 2: Add 2 to `nums[1]=4` (becomes 6), subtract 2 from `nums[3]=10` (becomes 8)
- Operation 3: Add 2 to `nums[1]=6` (becomes 8), subtract 2 from `nums[3]=8` (becomes 6)
... continuing until we match the target frequencies

The actual operations would transform:
- `4 → 8` (needs +4, so 2 operations where it gains)
- `10 → 12` (needs +2, so 1 operation where it gains)
- `1 → 3` (needs +2, so 1 operation where it gains)
- `7 → 5` (needs -2, so 1 operation where it loses)

Since we need 2 operations total and each operation affects exactly 2 elements, this confirms our calculation.

## Python Solution

```python
from typing import List

class Solution:
    def makeSimilar(self, nums: List[int], target: List[int]) -> int:
        # Sort both arrays by parity (odd/even) first, then by value
        # x & 1 returns 1 for odd numbers, 0 for even numbers
        # This groups even numbers first, then odd numbers, each group sorted in ascending order
        nums.sort(key=lambda x: (x & 1, x))
        target.sort(key=lambda x: (x & 1, x))
      
        # Calculate the total number of operations needed
        # Each operation involves adding/subtracting 2 to/from a pair of numbers
        # The sum of absolute differences gives us twice the total operations needed
        # (since each operation affects two numbers)
        # Dividing by 4 gives us the minimum number of operations
        total_difference = sum(abs(num - target_num) for num, target_num in zip(nums, target))
      
        return total_difference // 4
```

## Time and Space Complexity

**Time Complexity:** `O(n × log n)`, where `n` is the length of the array `nums`.

The time complexity is dominated by the sorting operations:
- `nums.sort(key=lambda x: (x & 1, x))` takes `O(n × log n)` time
- `target.sort(key=lambda x: (x & 1, x))` takes `O(n × log n)` time
- The `zip` operation and sum calculation take `O(n)` time

Since the sorting operations are performed sequentially, the overall time complexity is `O(n × log n) + O(n × log n) + O(n) = O(n × log n)`.

**Space Complexity:** `O(1)` or `O(n)` depending on the sorting algorithm implementation.

- If the sorting is done in-place (like Python's Timsort for lists), the space complexity is `O(1)` for the additional variables
- However, Python's sort may use up to `O(n)` auxiliary space in the worst case for its merge operations
- The lambda functions and the generator expression in sum use `O(1)` additional space

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### Pitfall 1: Incorrect Pairing Without Considering Parity

**The Problem:**
A common mistake is to simply sort both arrays by value and pair them directly, without considering the parity constraint. Since operations only add/subtract 2, odd numbers can never become even and vice versa.

**Incorrect Approach:**
```python
def makeSimilar(self, nums: List[int], target: List[int]) -> int:
    nums.sort()  # Wrong: sorts only by value
    target.sort()
    return sum(abs(a - b) for a, b in zip(nums, target)) // 4
```

**Why It Fails:**
If `nums = [1, 4, 5, 8]` and `target = [2, 3, 6, 7]`:
- Incorrect pairing: (1,2), (4,3), (5,6), (8,7)
- This tries to transform odd→even and even→odd, which is impossible

**Solution:**
Always separate by parity first using `key=lambda x: (x & 1, x)` or `key=lambda x: (x % 2, x)`.

### Pitfall 2: Wrong Division Factor

**The Problem:**
Dividing by 2 instead of 4 when calculating the final answer.

**Incorrect Approach:**
```python
def makeSimilar(self, nums: List[int], target: List[int]) -> int:
    nums.sort(key=lambda x: (x & 1, x))
    target.sort(key=lambda x: (x & 1, x))
    return sum(abs(a - b) for a, b in zip(nums, target)) // 2  # Wrong!
```

**Why It Fails:**
Each operation changes two elements: one by +2 and another by -2. When summing absolute differences, we count the effect of each operation twice (once for the element that increases, once for the element that decreases). The total difference represents the sum of all individual changes needed, where each unit of change is 2. Since each operation provides 4 units of total change (2 up + 2 down), we divide by 4.

**Solution:**
Remember that the division factor is 4 because:
- Each element needs `|a - b|` units of adjustment
- Each operation provides 4 units of total adjustment (2 added + 2 subtracted)
- The sum counts each operation's effect twice

### Pitfall 3: Using Modulo Instead of Bitwise AND

**The Problem:**
While both `x % 2` and `x & 1` work correctly for determining parity, using modulo can be slightly less efficient for large datasets.

**Less Optimal:**
```python
nums.sort(key=lambda x: (x % 2, x))  # Works but slightly slower
```

**Optimal:**
```python
nums.sort(key=lambda x: (x & 1, x))  # Bitwise operation is faster
```

**Why It Matters:**
Bitwise AND (`&`) is a single CPU instruction, while modulo (`%`) might require division. Though the performance difference is minimal in practice, using bitwise operations demonstrates better understanding of low-level optimizations.

