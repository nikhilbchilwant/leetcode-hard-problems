# 2818. Apply Operations to Maximize Score

> Source: https://algo.monster/liteproblems/2818

## Problem Statement

You are given an array `nums` containing `n` positive integers and an integer `k`. Your goal is to maximize a score that starts at `1` by performing at most `k` operations.

Each operation works as follows:
1. Select any subarray `nums[l, ..., r]` that hasn't been chosen before
2. From this subarray, pick the element `x` with the highest **prime score** (if there's a tie, choose the one with the smallest index)
3. Multiply your current score by `x`

The **prime score** of a number is defined as the count of its distinct prime factors. For example:
- The prime score of `300` is `3` because `300 = 2 × 2 × 3 × 5 × 5`, which has three distinct prime factors: `2`, `3`, and `5`

The challenge is to strategically select subarrays and elements to maximize your final score after performing at most `k` operations. Since the result can be very large, return it modulo `10^9 + 7`.

## Intuition

The first observation is that we want to multiply our score by the largest numbers as many times as possible. However, we're constrained by the rule that each chosen element must have the highest prime score in its selected subarray.

Think about it this way: for any element `nums[i]`, in how many different subarrays can it be the element with the highest prime score? If we can answer this question, we know the maximum number of times we can potentially use `nums[i]` in our operations.

For an element at position `i` to have the highest prime score in a subarray `[l, r]`:
- No element to its left in the range `[l, i-1]` should have a higher prime score
- No element to its right in the range `[i+1, r]` should have a higher prime score

This means we need to find:
- The leftmost position `left[i]` where all elements between `left[i]+1` and `i-1` have a lower prime score than `nums[i]`
- The rightmost position `right[i]` where all elements between `i+1` and `right[i]-1` have a lower or equal prime score than `nums[i]`

Once we have these boundaries, the number of valid subarrays where `nums[i]` is the maximum prime score element is `(i - left[i]) × (right[i] - i)`. This is because we can choose any starting position from `left[i]+1` to `i` and any ending position from `i` to `right[i]-1`.

Now comes the [greedy](/problems/greedy_intro) part: since we want to maximize our score, we should prioritize using larger numbers. We sort elements by their values in descending order and greedily use each element as many times as possible (up to its maximum count of valid subarrays) until we've performed `k` operations.

The monotonic [stack](/problems/stack_intro) technique efficiently computes the boundaries for each element by maintaining elements in order of their prime scores, allowing us to find the nearest greater element on both sides in linear time.

**Learn more about [Stack](/problems/stack_intro), [Greedy](/problems/greedy_intro), [Math](/problems/math-basics), [Sorting](/problems/sorting_summary) and [Monotonic Stack](/problems/mono_stack_intro) patterns.**

## Solution Approach

The implementation consists of three main phases:

**Phase 1: Calculate Prime Scores**
First, we compute the prime score for each element using a helper function `primeFactors(n)`. This function finds all distinct prime factors by:
- Iterating from `2` to `√n`
- Dividing `n` by each factor until it's no longer divisible
- Adding each distinct prime to a set
- If `n > 1` after the loop, it's itself a prime factor

**Phase 2: Find Boundaries Using Monotonic [Stack](/problems/stack_intro)**
For each element at position `i`, we need to find:
- `left[i]`: The rightmost index to the left where an element has a prime score ≥ `nums[i]`'s prime score
- `right[i]`: The leftmost index to the right where an element has a prime score > `nums[i]`'s prime score

We use monotonic stacks to efficiently compute these boundaries:

For `left` boundaries:
- Traverse the array from left to right
- Maintain a [stack](/problems/stack_intro) of `(prime_score, index)` pairs
- For each element, pop all stack elements with smaller prime scores
- The top of the stack (if exists) gives us `left[i]`
- Push current element onto stack

For `right` boundaries:
- Traverse the array from right to left (reverse iteration)
- Similar process but pop elements with prime scores ≤ current element's score
- This ensures elements with equal prime scores but smaller indices are prioritized

**Phase 3: [Greedy](/problems/greedy_intro) Selection**
After computing boundaries, we sort all elements by their values in descending order. For each element at position `i`:
- Calculate the number of valid subarrays: `cnt = (i - left[i]) × (right[i] - i)`
- If `cnt ≤ k`, use this element `cnt` times: `ans = ans × nums[i]^cnt mod (10^9 + 7)`
- If `cnt > k`, use this element only `k` times: `ans = ans × nums[i]^k mod (10^9 + 7)` and stop
- Update `k = k - cnt` after each iteration

The use of modular exponentiation `pow(x, cnt, mod)` ensures efficient computation of large powers while keeping results within bounds.

This [greedy](/problems/greedy_intro) approach works because once we know how many times each element can be used, it's optimal to use larger elements as many times as possible first.

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

**Input:** `nums = [8, 3, 9, 6]`, `k = 2`

**Step 1: Calculate Prime Scores**
- `8 = 2³` → prime score = 1 (only prime factor: 2)
- `3 = 3` → prime score = 1 (only prime factor: 3)  
- `9 = 3²` → prime score = 1 (only prime factor: 3)
- `6 = 2 × 3` → prime score = 2 (prime factors: 2, 3)

So prime scores are: `[1, 1, 1, 2]`

**Step 2: Find Boundaries Using Monotonic Stack**

Finding `left` boundaries (traverse left to right):
- Index 0: Stack empty, `left[0] = -1`
- Index 1: Prime score 1 = stack top's score 1, `left[1] = 0`
- Index 2: Prime score 1 = stack top's score 1, `left[2] = 1`
- Index 3: Prime score 2 > all previous, pop all, `left[3] = -1`

Result: `left = [-1, 0, 1, -1]`

Finding `right` boundaries (traverse right to left):
- Index 3: Stack empty, `right[3] = 4`
- Index 2: Prime score 1 < stack top's score 2, `right[2] = 3`
- Index 1: Prime score 1 < stack top's score 2, `right[1] = 3`
- Index 0: Prime score 1 < stack top's score 2, `right[0] = 3`

Result: `right = [3, 3, 3, 4]`

**Step 3: Calculate Valid Subarrays for Each Element**
- Element at index 0 (value 8): `(0 - (-1)) × (3 - 0) = 1 × 3 = 3` subarrays
- Element at index 1 (value 3): `(1 - 0) × (3 - 1) = 1 × 2 = 2` subarrays
- Element at index 2 (value 9): `(2 - 1) × (3 - 2) = 1 × 1 = 1` subarray
- Element at index 3 (value 6): `(3 - (-1)) × (4 - 3) = 4 × 1 = 4` subarrays

**Step 4: Greedy Selection**
Sort elements by value (descending): `[(9, 1 subarray), (8, 3 subarrays), (6, 4 subarrays), (3, 2 subarrays)]`

With `k = 2` operations:
- First pick: 9 can be used 1 time, score = 1 × 9 = 9, remaining k = 1
- Second pick: 8 can be used up to 3 times, but we only have k = 1 left, score = 9 × 8 = 72

**Final Answer:** 72

This example shows how the algorithm identifies which element has the highest prime score in each possible subarray, then greedily selects the largest values to maximize the final score.

## Python Solution

```python
def count_distinct_prime_factors(n: int) -> int:
    """
    Count the number of distinct prime factors of n.
  
    Args:
        n: The number to factorize
  
    Returns:
        The count of distinct prime factors
    """
    i = 2
    distinct_factors = set()
  
    # Check divisors up to sqrt(n)
    while i * i <= n:
        # Extract all occurrences of prime factor i
        while n % i == 0:
            distinct_factors.add(i)
            n //= i
        i += 1
  
    # If n > 1, then it's a prime factor itself
    if n > 1:
        distinct_factors.add(n)
  
    return len(distinct_factors)


class Solution:
    def maximumScore(self, nums: List[int], k: int) -> int:
        """
        Calculate the maximum score by selecting subsequences and computing products.
      
        The score is based on selecting elements and raising them to powers determined
        by the number of valid subsequences containing each element.
      
        Args:
            nums: List of positive integers
            k: Maximum number of operations allowed
      
        Returns:
            The maximum score modulo 10^9 + 7
        """
        MOD = 10**9 + 7
        n = len(nums)
      
        # Create array with (index, prime_factor_count, value) for each element
        elements_with_info = [(i, count_distinct_prime_factors(nums[i]), nums[i]) 
                               for i in range(n)]
      
        # Arrays to store boundaries for each element
        # left[i]: rightmost index j < i where prime_factor_count[j] >= prime_factor_count[i]
        # right[i]: leftmost index j > i where prime_factor_count[j] > prime_factor_count[i]
        left_boundary = [-1] * n
        right_boundary = [n] * n
      
        # Calculate left boundaries using monotonic stack
        # Stack maintains elements with decreasing prime factor counts
        stack = []
        for index, prime_count, value in elements_with_info:
            # Pop elements with smaller prime factor count
            while stack and stack[-1][0] < prime_count:
                stack.pop()
          
            # If stack not empty, top element is the left boundary
            if stack:
                left_boundary[index] = stack[-1][1]
          
            stack.append((prime_count, index))
      
        # Calculate right boundaries using monotonic stack (process from right to left)
        stack = []
        for index, prime_count, value in reversed(elements_with_info):
            # Pop elements with smaller or equal prime factor count
            while stack and stack[-1][0] <= prime_count:
                stack.pop()
          
            # If stack not empty, top element is the right boundary
            if stack:
                right_boundary[index] = stack[-1][1]
          
            stack.append((prime_count, index))
      
        # Sort elements by value in descending order to maximize score
        elements_with_info.sort(key=lambda x: -x[2])
      
        # Calculate the maximum score
        result = 1
        for index, prime_count, value in elements_with_info:
            left_bound = left_boundary[index]
            right_bound = right_boundary[index]
          
            # Calculate number of subsequences where this element is the minimum
            # based on prime factor count
            # Number of ways to choose left part * number of ways to choose right part
            subsequence_count = (index - left_bound) * (right_bound - index)
          
            if subsequence_count <= k:
                # Use all possible subsequences for this element
                result = result * pow(value, subsequence_count, MOD) % MOD
                k -= subsequence_count
            else:
                # Use only k remaining subsequences
                result = result * pow(value, k, MOD) % MOD
                break
      
        return result
```

## Time and Space Complexity

**Time Complexity: `O(n × √M + n × log n)`** where `n` is the length of the array and `M` is the maximum value in the array.

- The `primeFactors` function has time complexity `O(√M)` for each number, where `M` is the value being factorized. The while loop runs up to `√M` iterations.
- Computing prime factors for all `n` elements: `O(n × √M)`
- Building the monotonic stack for `left` array: `O(n)` - each element is pushed and popped at most once
- Building the monotonic stack for `right` array: `O(n)` - each element is pushed and popped at most once  
- Sorting the array by value in descending order: `O(n × log n)`
- The final loop to calculate the answer: `O(n)` in the worst case, iterating through all elements
- Total: `O(n × √M + n × log n)`

Since `√M` is typically much smaller than `log n` for reasonable input sizes, and considering the reference answer simplifies to `O(n × log n)`, the dominant term is the sorting operation.

**Space Complexity: `O(n)`**

- `arr` list storing tuples for each element: `O(n)`
- `left` array: `O(n)`
- `right` array: `O(n)`
- Stack `stk` used in monotonic stack operations: `O(n)` in worst case
- The `ans` set in `primeFactors` function: `O(log M)` at most, which is negligible
- Total: `O(n)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Boundary Conditions for Elements with Equal Prime Scores**

One of the most common pitfalls in this problem is mishandling elements with equal prime scores when determining boundaries. The problem states that when multiple elements have the same prime score, we should choose the one with the smallest index.

**The Pitfall:**
When calculating boundaries, developers often treat elements with equal prime scores inconsistently:
- For the left boundary, we need `>=` comparison (elements with equal or greater prime scores)
- For the right boundary, we need `>` comparison (strictly greater prime scores)

This asymmetry ensures that when multiple elements have the same prime score, only the leftmost one can be selected from any subarray containing them.

**Incorrect Implementation Example:**
```python
# WRONG: Using the same comparison for both boundaries
for index, prime_count, value in elements_with_info:
    while stack and stack[-1][0] <= prime_count:  # Should be < for left boundary
        stack.pop()
    # ...

for index, prime_count, value in reversed(elements_with_info):
    while stack and stack[-1][0] < prime_count:  # Should be <= for right boundary
        stack.pop()
    # ...
```

**Correct Implementation:**
```python
# Left boundary: pop elements with STRICTLY SMALLER prime scores
for index, prime_count, value in elements_with_info:
    while stack and stack[-1][0] < prime_count:  # < comparison
        stack.pop()
    # ...

# Right boundary: pop elements with SMALLER OR EQUAL prime scores
for index, prime_count, value in reversed(elements_with_info):
    while stack and stack[-1][0] <= prime_count:  # <= comparison
        stack.pop()
    # ...
```

### 2. **Integer Overflow in Subsequence Count Calculation**

**The Pitfall:**
When calculating the number of valid subarrays for each element, the multiplication `(index - left_bound) * (right_bound - index)` can overflow for large arrays, even though Python handles big integers automatically. More importantly, developers might forget to handle the modulo operation correctly when raising to large powers.

**Incorrect Approach:**
```python
# WRONG: Computing large power then taking modulo
result = (result * (value ** subsequence_count)) % MOD  # Can cause memory/time issues
```

**Correct Approach:**
```python
# RIGHT: Using built-in modular exponentiation
result = result * pow(value, subsequence_count, MOD) % MOD
```

### 3. **Edge Cases in Prime Factorization**

**The Pitfall:**
The prime factorization function might miss edge cases:
- Not handling when `n` itself is prime after the loop
- Not handling the case where `n = 1` (though this shouldn't occur given "positive integers")
- Using incorrect loop bounds (should be `i * i <= n`, not `i <= n`)

**Common Mistake:**
```python
def count_distinct_prime_factors(n: int) -> int:
    i = 2
    distinct_factors = set()
  
    while i <= n:  # WRONG: Should be i * i <= n for efficiency
        while n % i == 0:
            distinct_factors.add(i)
            n //= i
        i += 1
  
    # MISSING: Not checking if n > 1 after the loop
    return len(distinct_factors)
```

**Correct Implementation:**
```python
def count_distinct_prime_factors(n: int) -> int:
    i = 2
    distinct_factors = set()
  
    while i * i <= n:  # Correct bound
        while n % i == 0:
            distinct_factors.add(i)
            n //= i
        i += 1
  
    if n > 1:  # Important: n itself might be a prime factor
        distinct_factors.add(n)
  
    return len(distinct_factors)
```

### 4. **Misunderstanding the Subarray Selection Rule**

**The Pitfall:**
Developers might misinterpret "each subarray can only be selected once" and incorrectly calculate the contribution of each element. The key insight is that an element can contribute to multiple different subarrays, and we need to count all valid subarrays where it has the maximum prime score.

**Solution:**
Ensure you understand that:
- Each element's contribution is based on ALL subarrays where it's the maximum (by prime score)
- The count is `(i - left[i]) × (right[i] - i)` representing all possible combinations
- We're not selecting k different elements, but rather using elements up to k times total

