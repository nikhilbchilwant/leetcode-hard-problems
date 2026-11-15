# 1998. GCD Sort of an Array

> Source: https://algo.monster/liteproblems/1998

## Problem Statement

You have an integer array `nums` that you want to sort in non-decreasing order (ascending order). However, you can only swap elements under a specific condition.

The swapping rule is: You can swap two elements `nums[i]` and `nums[j]` only if their greatest common divisor (GCD) is greater than 1. This means the two numbers must share at least one common factor other than 1.

For example:
- You can swap 6 and 9 because `gcd(6, 9) = 3 > 1`
- You can swap 4 and 10 because `gcd(4, 10) = 2 > 1`  
- You cannot swap 3 and 7 because `gcd(3, 7) = 1`

You can perform any number of these valid swaps. The question asks whether it's possible to sort the entire array into non-decreasing order using only these constrained swaps.

The key insight is that if two numbers share a common factor greater than 1, they can be swapped. Additionally, if number A can swap with B, and B can swap with C, then A and C are in the same "connected component" - they can eventually be rearranged among themselves through a series of swaps.

The solution uses Union-Find (Disjoint Set Union) to group numbers that can be swapped directly or indirectly. First, it finds all prime factors for each number in the array. Then it unions all numbers that share common prime factors. Finally, it checks if each number in the original array can reach its target position in the sorted array through the swap connections.

The function returns `true` if the array can be sorted using the allowed swaps, and `false` otherwise.

## Intuition

The first observation is that if two numbers share a common factor greater than 1, they can be swapped. But this creates a transitive relationship - if number A can swap with B (they share factor 2), and B can swap with C (they share factor 3), then A and C can eventually swap positions through B as an intermediary.

This leads to thinking about connected components. Numbers that can swap directly or indirectly form groups. Within each group, we can rearrange the numbers in any order we want through a series of swaps.

For example, consider `[6, 10, 15]`:
- 6 and 10 share factor 2, so they can swap
- 6 and 15 share factor 3, so they can swap  
- 10 and 15 share factor 5, so they can swap
- All three numbers are in the same group and can be arranged in any order

The key insight is: **for the array to be sortable, each number must be able to reach its target position in the sorted array**. This means that for each position, the number currently there and the number that should be there (in the sorted array) must be in the same connected component.

To efficiently track these connections, we can use Union-Find. But how do we connect numbers? We could check every pair of numbers for `gcd > 1`, but that's expensive. 

A better approach: two numbers share a factor greater than 1 if and only if they share at least one prime factor. So we can:
1. Find all prime factors for each number
2. Group numbers by their prime factors
3. Numbers sharing any prime factor belong to the same component

Finally, we check if the [sorting](/problems/sorting_summary) is possible by verifying that each number can reach its sorted position through the connection network we've built.

**Learn more about [Union Find](/problems/dsu_intro), [Math](/problems/math-basics) and [Sorting](/problems/sorting_summary) patterns.**

## Solution Approach

The solution uses Union-Find (Disjoint Set Union) data structure combined with prime factorization to solve this problem efficiently.

**Step 1: Initialize Union-Find Structure**
```python
n = 10**5 + 10
p = list(range(n))
```
We create a parent array `p` where initially each element is its own parent. The size `10^5 + 10` covers the maximum possible value in the constraints.

**Step 2: Build Prime Factor Mapping Using Sieve**
```python
f = defaultdict(list)
mx = max(nums)
for i in range(2, mx + 1):
    if f[i]:
        continue
    for j in range(i, mx + 1, i):
        f[j].append(i)
```
This implements a modified Sieve of Eratosthenes. For each number from 2 to the maximum value in `nums`:
- If `f[i]` is empty, then `i` is prime
- We mark all multiples of `i` (including `i` itself) as having `i` as a prime factor
- After this loop, `f[num]` contains all prime factors of `num`

**Step 3: Union Numbers with Common Prime Factors**
```python
def find(x):
    if p[x] != x:
        p[x] = find(p[x])
    return p[x]

for i in nums:
    for j in f[i]:
        p[find(i)] = find(j)
```
The `find` function implements path compression for efficient lookups. For each number in `nums`, we union it with all its prime factors. This cleverly connects all numbers sharing any prime factor:
- If numbers A and B both have prime factor P, both will be unioned with P
- This puts A and B in the same component

**Step 4: Verify Sortability**
```python
s = sorted(nums)
for i, num in enumerate(nums):
    if s[i] != num and find(num) != find(s[i]):
        return False
return True
```
We create the sorted version of the array. Then for each position:
- If the current number `nums[i]` differs from what should be there `s[i]`
- We check if they're in the same connected component using `find`
- If they're not connected (can't be swapped), return `False`
- If all positions can be satisfied, return `True`

The time complexity is `O(n log n + m log m)` where `n` is the array length and `m` is the maximum value, due to [sorting](/problems/sorting_summary) and the sieve. The space complexity is `O(m)` for the Union-Find structure and prime factor storage.

## Example Walkthrough

Let's walk through the solution with the array `nums = [6, 10, 15]`.

**Step 1: Initialize Union-Find**
We create a parent array where each element points to itself initially:
- `p[6] = 6`, `p[10] = 10`, `p[15] = 15`, etc.

**Step 2: Find Prime Factors**
Using the sieve approach, we find prime factors for each number up to 15:
- `f[6] = [2, 3]` (6 = 2 × 3)
- `f[10] = [2, 5]` (10 = 2 × 5)  
- `f[15] = [3, 5]` (15 = 3 × 5)

**Step 3: Union Numbers with Their Prime Factors**
For each number, we union it with its prime factors:

For `6` with prime factors `[2, 3]`:
- Union 6 with 2: `p[6] = 2`
- Union 6 with 3: `p[2] = 3` (since find(6) = 2)

For `10` with prime factors `[2, 5]`:
- Union 10 with 2: Since find(2) = 3, we set `p[10] = 3`
- Union 10 with 5: `p[3] = 5` (connecting the whole component)

For `15` with prime factors `[3, 5]`:
- Union 15 with 3: Since find(3) = 5, we set `p[15] = 5`
- Union 15 with 5: Already connected

Now all numbers 6, 10, and 15 are in the same component (all eventually point to 5).

**Step 4: Check if Sorting is Possible**
- Original: `[6, 10, 15]`
- Sorted: `[6, 10, 15]` (already sorted!)
- Position 0: `nums[0] = 6`, `sorted[0] = 6` - same number, OK
- Position 1: `nums[1] = 10`, `sorted[1] = 10` - same number, OK
- Position 2: `nums[2] = 15`, `sorted[2] = 15` - same number, OK

Return `true` - the array can be sorted (it's already sorted).

Let's try a different example where swapping is needed: `nums = [10, 6, 15]`

Using the same Union-Find structure from above:
- Original: `[10, 6, 15]`
- Sorted: `[6, 10, 15]`
- Position 0: `nums[0] = 10`, `sorted[0] = 6`
  - find(10) = 5, find(6) = 5 - same component, can swap ✓
- Position 1: `nums[1] = 6`, `sorted[1] = 10`  
  - find(6) = 5, find(10) = 5 - same component, can swap ✓
- Position 2: `nums[2] = 15`, `sorted[2] = 15` - same number, OK

Return `true` - the array can be sorted through valid swaps.

For a negative example, consider `nums = [3, 7]`:
- `f[3] = [3]` (3 is prime)
- `f[7] = [7]` (7 is prime)
- After unions: 3 and 7 remain in separate components
- Original: `[3, 7]`, Sorted: `[3, 7]` - already sorted, returns `true`

But if we had `nums = [7, 3]`:
- Original: `[7, 3]`, Sorted: `[3, 7]`
- Position 0: `nums[0] = 7`, `sorted[0] = 3`
  - find(7) = 7, find(3) = 3 - different components, cannot swap ✗
- Return `false` - cannot sort this array

## Python Solution

```python
class Solution:
    def gcdSort(self, nums: List[int]) -> bool:
        # Initialize Union-Find parent array for all possible values
        max_value = 10**5 + 10
        parent = list(range(max_value))
      
        # Dictionary to store prime factors for each number
        prime_factors = defaultdict(list)
      
        # Find the maximum value in nums to limit our factorization
        max_num = max(nums)
      
        # Sieve-like approach to find prime factors for all numbers up to max_num
        for prime_candidate in range(2, max_num + 1):
            # Skip if we've already found factors for this number
            if prime_factors[prime_candidate]:
                continue
          
            # Mark all multiples of this prime
            for multiple in range(prime_candidate, max_num + 1, prime_candidate):
                prime_factors[multiple].append(prime_candidate)
      
        # Union-Find find function with path compression
        def find(x):
            if parent[x] != x:
                parent[x] = find(parent[x])  # Path compression
            return parent[x]
      
        # Union all numbers that share prime factors
        for number in nums:
            # Connect this number with all its prime factors
            for prime_factor in prime_factors[number]:
                parent[find(number)] = find(prime_factor)
      
        # Check if the array can be sorted
        sorted_nums = sorted(nums)
      
        # Verify each position: if elements differ, they must be in the same component
        for index, original_num in enumerate(nums):
            target_num = sorted_nums[index]
            # If numbers are different and not in the same connected component, cannot sort
            if target_num != original_num and find(original_num) != find(target_num):
                return False
      
        return True
```

## Time and Space Complexity

**Time Complexity:** `O(n * log(log(n)) + m * α(n) * f + m * log(m))`

Where:
- `n` is the maximum value in nums (used as the upper bound for sieve)
- `m` is the length of the nums array
- `α(n)` is the inverse Ackermann function (practically constant)
- `f` is the average number of prime factors per number

Breaking down the operations:
1. **Prime factorization using sieve:** `O(n * log(log(n)))` - The nested loops iterate through all numbers up to `mx` and mark their factors, which follows a harmonic series pattern similar to the Sieve of Eratosthenes
2. **Union operations:** `O(m * f * α(n))` - For each number in nums, we perform union operations for all its prime factors
3. **Sorting:** `O(m * log(m))` - Standard sorting of the nums array
4. **Final verification:** `O(m * α(n))` - For each element, we perform find operations

The dominant term is typically `O(n * log(log(n)))` when `n` is large relative to `m`.

**Space Complexity:** `O(n + m)`

Breaking down the space usage:
1. **Parent array `p`:** `O(n)` - stores parent pointers for values up to 10^5 + 10
2. **Factor dictionary `f`:** `O(n * log(n))` in worst case - each number can have at most `O(log(n))` prime factors
3. **Sorted array `s`:** `O(m)` - copy of the input array

The dominant term is `O(n)` due to the parent array initialization.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Inefficient Prime Factorization**
A common mistake is using naive factorization for each number individually, which leads to timeout for large inputs.

**Incorrect approach:**
```python
def get_prime_factors(n):
    factors = []
    for i in range(2, int(n**0.5) + 1):
        while n % i == 0:
            factors.append(i)
            n //= i
    if n > 1:
        factors.append(n)
    return factors

# Then calling this for each number - O(n * sqrt(m))
for num in nums:
    factors = get_prime_factors(num)
    # ... union operations
```

**Why it fails:** This approach has O(n * sqrt(m)) complexity for factorization alone, where n is array length and m is the maximum value. For arrays with many large numbers, this becomes prohibitively slow.

**Solution:** Use the sieve-based approach shown in the main solution, which pre-computes all prime factors in O(m log m) time once, rather than factorizing each number separately.

### 2. **Incorrect Union-Find Implementation**
Forgetting path compression or implementing union incorrectly can cause wrong results or poor performance.

**Common mistakes:**
```python
# Mistake 1: No path compression
def find(x):
    while parent[x] != x:
        x = parent[x]
    return x

# Mistake 2: Direct assignment without find
for num in nums:
    for prime in prime_factors[num]:
        parent[num] = prime  # Wrong! Should use find()
```

**Solution:** Always use path compression and ensure both elements are reduced to their roots before union:
```python
def find(x):
    if parent[x] != x:
        parent[x] = find(parent[x])  # Path compression
    return parent[x]

# Correct union
parent[find(num)] = find(prime)
```

### 3. **Memory Limit Issues with Large Range**
Creating arrays for the full range up to 10^5 when the actual maximum is much smaller wastes memory.

**Better approach:**
```python
# Dynamically size based on actual maximum
max_num = max(nums)
parent = list(range(max_num + 1))
```

### 4. **Missing Edge Cases**
Not handling special cases like arrays with duplicate numbers or single elements.

**Solution:** The algorithm naturally handles these cases, but be aware:
- Duplicate numbers will always be in the same component (they share all prime factors)
- Single element arrays always return `True`
- Arrays already sorted return `True` (each element is compared with itself)

