# 952. Largest Component Size by Common Factor

> Source: https://algo.monster/liteproblems/952

## Problem Statement

You have an array of unique positive integers called `nums`. Your task is to build a graph based on these numbers and find the largest connected component in that graph.

The graph is constructed with the following rules:
- Each number in the array becomes a node in the graph (so if you have `n` numbers, you have `n` nodes)
- The nodes are labeled with the actual values from the array (not indices)
- Two nodes `nums[i]` and `nums[j]` are connected by an edge if and only if they share a common factor greater than 1

A common factor greater than 1 means the two numbers have at least one common divisor that is greater than 1. For example:
- 6 and 9 share common factor 3 (both are divisible by 3)
- 4 and 6 share common factor 2 (both are divisible by 2)
- 5 and 7 don't share any common factor greater than 1

A connected component is a group of nodes where you can reach any node from any other node in the group by following edges. Your goal is to find the size (number of nodes) of the largest connected component in this graph.

For example, if `nums = [4, 6, 15, 35]`:
- 4 and 6 are connected (common factor: 2)
- 15 and 35 are connected (common factor: 5)
- But there's no connection between the group {4, 6} and {15, 35}
- So you have two connected components of size 2 each, and the answer would be 2

## Intuition

The key insight is that if two numbers share a common factor, they belong to the same connected component. But instead of checking every pair of numbers directly (which would be expensive), we can think about this problem differently.

Consider that if two numbers share a common factor `f`, then both numbers are connected to `f`. This means we can use the factors themselves as "bridges" to connect numbers. For example, if we have numbers 6 and 9, instead of directly connecting them, we can say:
- 6 is connected to its factor 3
- 9 is connected to its factor 3
- Therefore, 6 and 9 are in the same component through their shared factor 3

This naturally leads us to use Union-Find (Disjoint Set Union), where we can union each number with all of its factors. The beauty of this approach is that if multiple numbers share the same factor, they'll all get connected through that factor automatically.

For each number `v`, we only need to find its factors up to `√v` because factors come in pairs. If `i` divides `v`, then `v/i` also divides `v`. So when we find factor `i`, we union `v` with both `i` and `v/i`.

After processing all numbers and their factors, numbers that share any common factors will end up in the same connected component (they'll have the same root in the Union-Find structure). We then count how many numbers belong to each root and return the maximum count.

The trick of using factors as intermediary nodes transforms the problem from "check if two numbers share factors" to "connect numbers to their factors", which is much more efficient and elegant.

**Learn more about [Union Find](/problems/dsu_intro) and [Math](/problems/math-basics) patterns.**

## Solution Approach

The solution uses the Union-Find (Disjoint Set Union) data structure to efficiently group numbers that share common factors.

**Union-Find Implementation:**

The `UnionFind` class maintains a parent array `parent` where `parent[x]` represents the parent of element `x`. Initially, each element is its own parent.

- `find(x)`: Returns the root of the set containing `x`. It uses path compression (`self.parent[x] = self.find(self.parent[x])`) to optimize future lookups by making every node point directly to the root.
- `union(a, b)`: Merges the sets containing `a` and `b` by making one root point to the other.

**Main Algorithm:**

1. **Initialize Union-Find**: Create a Union-Find structure with size `max(nums) + 1` to accommodate all possible values and their factors.

2. **Factor Finding and Union Operations**: For each number `num` in the array:
   - Start checking potential factors from `factor = 2`
   - Continue while `factor <= √num` (written as `factor * factor <= num` to avoid floating-point operations)
   - If `factor` divides `num` evenly (`num % factor == 0`):
     - Union `num` with `factor`
     - Union `num` with its complementary factor `num // factor`
   - Increment `factor` and continue

3. **Count Component Sizes**: After all unions are complete:
   - For each number in the original array, find its root using `union_find.find(num)`
   - Count how many numbers belong to each root using `Counter`
   - Return the maximum count, which represents the largest connected component

**Example Walkthrough:**

For `nums = [6, 8, 12]`:
- Process 6: factors are 2 and 3, union(6,2), union(6,3)
- Process 8: factors are 2 and 4, union(8,2), union(8,4)
- Process 12: factors are 2,6 and 3,4, union(12,2), union(12,6), union(12,3), union(12,4)

After all unions, 6, 8, and 12 are all connected through their shared factor 2, forming one component of size 3.

The time complexity is `O(n * √max(nums))` for factorization, where `n` is the array length. The space complexity is `O(max(nums))` for the Union-Find structure.

## Example Walkthrough

Let's walk through the solution with `nums = [20, 50, 9, 63]`.

**Step 1: Initialize Union-Find**
Create a Union-Find structure with size 64 (max(nums) + 1). Each element starts as its own parent.

**Step 2: Process each number and find factors**

**Processing 20:**
- Check i = 2: 20 % 2 = 0 ✓
  - union(20, 2)
  - union(20, 10)
- Check i = 3: 20 % 3 ≠ 0 ✗
- Check i = 4: 20 % 4 = 0 ✓
  - union(20, 4)
  - union(20, 5)
- Stop (i = 5 > √20)

Current components: {20, 2, 10, 4, 5}

**Processing 50:**
- Check i = 2: 50 % 2 = 0 ✓
  - union(50, 2)
  - union(50, 25)
- Check i = 3: 50 % 3 ≠ 0 ✗
- Check i = 4: 50 % 4 ≠ 0 ✗
- Check i = 5: 50 % 5 = 0 ✓
  - union(50, 5)
  - union(50, 10)
- Check i = 6: 50 % 6 ≠ 0 ✗
- Check i = 7: 50 % 7 ≠ 0 ✗
- Stop (i = 8 > √50)

Now 50 connects to the existing component through factors 2, 5, and 10.
Current components: {20, 50, 2, 10, 4, 5, 25}

**Processing 9:**
- Check i = 2: 9 % 2 ≠ 0 ✗
- Check i = 3: 9 % 3 = 0 ✓
  - union(9, 3)
  - union(9, 3) (since 9/3 = 3)
- Stop (i = 4 > √9)

9 forms a new component: {9, 3}

**Processing 63:**
- Check i = 2: 63 % 2 ≠ 0 ✗
- Check i = 3: 63 % 3 = 0 ✓
  - union(63, 3)
  - union(63, 21)
- Check i = 4: 63 % 4 ≠ 0 ✗
- Check i = 5: 63 % 5 ≠ 0 ✗
- Check i = 6: 63 % 6 ≠ 0 ✗
- Check i = 7: 63 % 7 = 0 ✓
  - union(63, 7)
  - union(63, 9)
- Stop (i = 8 > √63)

63 connects to 9's component through factor 3 and also through 9 itself.
Current components: {9, 63, 3, 21, 7}

**Step 3: Count component sizes**
Find the root for each original number:
- find(20) → root1
- find(50) → root1 (same component as 20)
- find(9) → root2
- find(63) → root2 (same component as 9)

Component sizes:
- Component with root1: contains {20, 50} → size 2
- Component with root2: contains {9, 63} → size 2

**Answer: 2** (both components have the same size)

The key insight demonstrated here is how shared factors act as bridges. Numbers 20 and 50 are connected through their common factors (2, 5, 10), while 9 and 63 are connected through their common factor 3. The Union-Find structure efficiently manages these connections without explicitly checking every pair of numbers.

## Python Solution

```python
from typing import List
from collections import Counter


class UnionFind:
    """Union-Find (Disjoint Set Union) data structure with path compression."""
  
    def __init__(self, n: int) -> None:
        """Initialize n disjoint sets, each element is its own parent initially."""
        self.parent = list(range(n))
  
    def union(self, a: int, b: int) -> None:
        """Unite the sets containing elements a and b."""
        root_a = self.find(a)
        root_b = self.find(b)
      
        # Only unite if they belong to different sets
        if root_a != root_b:
            self.parent[root_a] = root_b
  
    def find(self, x: int) -> int:
        """Find the root of the set containing x with path compression."""
        if self.parent[x] != x:
            # Path compression: make every node point directly to the root
            self.parent[x] = self.find(self.parent[x])
        return self.parent[x]


class Solution:
    def largestComponentSize(self, nums: List[int]) -> int:
        """
        Find the size of the largest connected component where two numbers 
        are connected if they share a common factor greater than 1.
      
        Args:
            nums: List of positive integers
          
        Returns:
            Size of the largest connected component
        """
        # Create UnionFind structure with size equal to max value + 1
        max_value = max(nums)
        union_find = UnionFind(max_value + 1)
      
        # For each number, connect it with all its factors
        for num in nums:
            # Find all factors by iterating up to sqrt(num)
            factor = 2
            while factor * factor <= num:
                if num % factor == 0:
                    # Connect num with its factor
                    union_find.union(num, factor)
                    # Connect num with its complementary factor (num // factor)
                    union_find.union(num, num // factor)
                factor += 1
      
        # Count the size of each connected component
        # by finding the root of each number and counting occurrences
        component_sizes = Counter(union_find.find(num) for num in nums)
      
        # Return the size of the largest component
        return max(component_sizes.values())
```

## Time and Space Complexity

**Time Complexity:** `O(m + n * √m * log m)` where `n` is the length of the input array and `m` is the maximum value in the array. The `log m` factor is an amortized bound for `find`; in practice the factorization loop of `O(n * √m)` iterations dominates.

- The UnionFind initialization takes `O(m)` time to create the parent array.
- For each number `num` in `nums` (n iterations):
  - The factorization loop runs from `factor = 2` to `√num`, which is `O(√num)` iterations in the worst case.
  - Inside the loop, we check if `factor` divides `num` and perform at most 2 union operations.
  - Each union operation involves two find operations. This `UnionFind` uses path compression but not union by rank or size, so each find costs `O(log m)` amortized (the near-constant `O(α(m))` bound requires both optimizations).
- The final step uses Counter on n elements, where each element requires a find operation: `O(n * log m)` amortized.
- Overall: `O(m) + O(n * √m * log m) + O(n * log m)` = `O(m + n * √m * log m)`.

**Space Complexity:** `O(m)`

- The UnionFind structure uses an array of size `m + 1` to store parent pointers: `O(m)`.
- The Counter in the final step stores at most `n` entries: `O(n)`.
- The recursion stack for `find` is as deep as the current tree. Without union by rank there is no `O(log m)` guarantee on height, but it is bounded by `O(m)`, so it does not change the total.
- Since typically `m > n` in this problem context, the overall space complexity is `O(m)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### Pitfall 1: Trial Division Work for Prime Numbers

**The Problem:**
The current implementation tries every potential factor from 2 to √num for every number. For a large prime such as 99991, the loop runs about 315 iterations and finds no factor, so the prime ends up as a singleton component. With up to `2 * 10^4` numbers, this adds up to millions of modulo operations, which is acceptable for the constraints but can be reduced.

**Example:** 
If `nums = [99991, 99989, 99971]` (all prime and at most `10^5`), the loop runs about 315 iterations for each number, roughly 950 modulo operations in total, and makes no unions.

**Solution:**
Pre-compute the smallest prime factor (SPF) of every value with a sieve, then factor each number by repeated division. Each number then needs only `O(log num)` steps, and it is unioned only with its prime factors, which is enough to connect numbers that share any factor:

```python
def largestComponentSize(self, nums: List[int]) -> int:
    max_value = max(nums)
    union_find = UnionFind(max_value + 1)
  
    # Pre-compute smallest prime factor for each number using sieve
    spf = list(range(max_value + 1))  # smallest prime factor
    for i in range(2, int(max_value**0.5) + 1):
        if spf[i] == i:  # i is prime
            for j in range(i * i, max_value + 1, i):
                if spf[j] == j:
                    spf[j] = i
  
    # Use SPF for efficient factorization
    for num in nums:
        factors_seen = set()
        temp = num
        while temp > 1:
            prime_factor = spf[temp]
            if prime_factor not in factors_seen:
                union_find.union(num, prime_factor)
                factors_seen.add(prime_factor)
            temp //= prime_factor
  
    component_sizes = Counter(union_find.find(num) for num in nums)
    return max(component_sizes.values())
```

### Pitfall 2: Memory Use for Large Values

**The Problem:**
The Union-Find structure allocates space for `max(nums) + 1` elements. Within this problem's constraints (`nums[i] <= 10^5`), that is at most 100,001 entries, which is fine. The issue only appears in a variant with much larger values: for `nums = [2, 999983]`, an array of 1,000,000 entries would be allocated to connect two numbers.

**Solution:**
For such a variant, use a dictionary-based Union-Find that only stores values that are actually used:

```python
class UnionFindDict:
    def __init__(self):
        self.parent = {}
  
    def find(self, x):
        if x not in self.parent:
            self.parent[x] = x
        if self.parent[x] != x:
            self.parent[x] = self.find(self.parent[x])
        return self.parent[x]
  
    def union(self, a, b):
        root_a, root_b = self.find(a), self.find(b)
        if root_a != root_b:
            self.parent[root_a] = root_b
```

### Pitfall 3: Adding Edge-Case Filtering That Breaks Correct Code

**The Problem:**
The shown solution already handles the edge cases. A single-element array gives a `Counter` with one entry of count 1. Pairwise coprime numbers each stay in their own set, so the answer is 1. The value 1 has no factor loop iterations, so it stays a singleton. Extra filtering added "for safety" can introduce a bug instead:

```python
# WRONG: drops the 1 and then reports 0 for nums = [1]
nums = [n for n in nums if n > 1]
if not nums:
    return 0
```

For `nums = [1]`, the correct answer is 1, because the single node forms a component of size 1.

**Solution:**
Keep the general code path. If you filter out 1s, return at least 1 whenever the original array was non-empty.

