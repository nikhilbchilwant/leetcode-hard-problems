# 60. Permutation Sequence

> Source: https://algo.monster/liteproblems/60

## Problem Statement

You are given the set `[1, 2, 3, ..., n]` which contains a total of `n!` unique permutations. When all permutations are listed in lexicographical (dictionary) order and labeled with indices starting from 1, you need to find and return the `k`-th permutation in this sequence.

For example, when `n = 3`, the set `[1, 2, 3]` generates 6 permutations in order:
1. `"123"` (1st permutation)
2. `"132"` (2nd permutation)
3. `"213"` (3rd permutation)
4. `"231"` (4th permutation)
5. `"312"` (5th permutation)
6. `"321"` (6th permutation)

Given two integers `n` and `k`, your task is to return the `k`-th permutation sequence as a string.

## Intuition

Think about how permutations are organized when listed in lexicographical order. If we have `n` numbers, and we fix the first position with a specific digit, how many permutations can we form with the remaining `n-1` digits? The answer is `(n-1)!`.

This observation leads to a pattern: permutations are naturally grouped by their first digit. For `n = 3`:
- Permutations 1-2 start with `1` (there are `2! = 2` of them)
- Permutations 3-4 start with `2` (there are `2! = 2` of them)  
- Permutations 5-6 start with `3` (there are `2! = 2` of them)

So if we want the 5th permutation, we first switch to a 0-based rank, `5 - 1 = 4`, and divide by `2!`: `4 // 2 = 2` with remainder `0`. This means we skip the first 2 groups (those starting with `1` and `2`), and we need the permutation at 0-based rank 0, the first one, in the group starting with `3`. (Dividing the 1-based `k` directly goes wrong at group boundaries: for `k = 4`, `4 // 2 = 2` would wrongly pick the group starting with `3`, while the 4th permutation `231` starts with `2`.)

We can extend this logic position by position. Once we determine the first digit, we remove it from our available digits and repeat the process for the second position with `(n-2)!` permutations per group, and so on.

The key insight is that we don't need to generate all permutations. Instead, we can directly calculate which digit belongs at each position by:
1. Computing how many complete groups of permutations we need to skip (based on factorial of remaining positions)
2. Determining which available digit corresponds to that group
3. Updating `k` to reflect our position within the selected group
4. Repeating for the next position with one less digit available

This transforms a potentially exponential problem (generating all permutations) into a linear one (selecting each digit one by one).

**Learn more about [Recursion](/problems/recursion_intro) and [Math](/problems/math-basics) patterns.**

## Solution Approach

The implementation follows a position-by-position selection strategy using factorial calculations:

**Data Structures Used:**
- `ans`: List to build the result permutation digit by digit
- `vis`: Boolean array of size `n+1` to track which numbers have been used (indexed from 1 to n)

**Algorithm Steps:**

1. **Iterate through each position** (from 0 to n-1):
   - For position `i`, we need to select from the remaining unused numbers
   - Calculate `fact = (n-i-1)!` - the number of permutations each remaining digit can form

2. **Find the correct digit for current position:**
   - Iterate through numbers 1 to n
   - Skip numbers already marked as visited in `vis`
   - For each unvisited number `j`:
     - If `k > fact`: This means the k-th permutation doesn't start with `j` at this position. Subtract `fact` from `k` to skip all permutations starting with `j`
     - If `k <= fact`: We found our digit! The k-th permutation has `j` at the current position
       - Append `j` to our answer
       - Mark `vis[j] = True`
       - Break to move to the next position

3. **Build the result:**
   - After filling all positions, join the collected digits into a string

**Example Walkthrough (n=3, k=5):**
- Position 0: `fact = 2! = 2`
  - Check 1: `k=5 > 2`, so `k = 5-2 = 3`
  - Check 2: `k=3 > 2`, so `k = 3-2 = 1`  
  - Check 3: `k=1 <= 2`, select 3, mark it as used
- Position 1: `fact = 1! = 1`
  - Check 1 (not used): `k=1 <= 1`, select 1, mark it as used
- Position 2: `fact = 0! = 1`
  - Only 2 remains unvisited, select it
- Result: "312"

The time complexity is `O(n²)` as we iterate through n positions and check up to n numbers for each position. The space complexity is `O(n)` for the visited array and answer list.

## Example Walkthrough

Let's find the 4th permutation when n = 3 (the answer should be "231").

**Initial Setup:**
- Numbers available: [1, 2, 3]
- k = 4
- vis = [False, False, False, False] (index 0 unused, tracking 1-3)
- ans = []

**Position 0 (First digit):**
- Calculate fact = (3-0-1)! = 2! = 2
- This means each first digit leads to 2 permutations
- Check digit 1: 
  - k=4 > fact=2, so skip all permutations starting with 1
  - Update k = 4 - 2 = 2
- Check digit 2:
  - k=2 <= fact=2, so digit 2 goes in position 0!
  - Add 2 to ans → ans = [2]
  - Mark vis[2] = True

**Position 1 (Second digit):**
- Calculate fact = (3-1-1)! = 1! = 1
- Available digits: [1, 3] (2 is already used)
- Check digit 1:
  - k=2 > fact=1, so skip permutations with 1 in position 1
  - Update k = 2 - 1 = 1
- Check digit 3:
  - k=1 <= fact=1, so digit 3 goes in position 1!
  - Add 3 to ans → ans = [2, 3]
  - Mark vis[3] = True

**Position 2 (Third digit):**
- Calculate fact = (3-2-1)! = 0! = 1
- Available digits: [1] (only 1 remains)
- Check digit 1:
  - k=1 <= fact=1, so digit 1 goes in position 2!
  - Add 1 to ans → ans = [2, 3, 1]
  - Mark vis[1] = True

**Final Result:** Join [2, 3, 1] → "231"

This matches our expectation! The 4th permutation of [1,2,3] in lexicographical order is indeed "231".

## Python Solution

```python
class Solution:
    def getPermutation(self, n: int, k: int) -> str:
        """
        Find the k-th permutation sequence of numbers from 1 to n.
      
        Args:
            n: The range of numbers from 1 to n
            k: The k-th permutation to find (1-indexed)
      
        Returns:
            The k-th permutation as a string
        """
        result = []
        # Track which numbers have been used
        used = [False] * (n + 1)  # Index 0 unused, indices 1 to n represent numbers 1 to n
      
        # Build the permutation digit by digit
        for position in range(n):
            # Calculate factorial of remaining positions
            # This represents how many permutations exist for the remaining positions
            factorial = 1
            for i in range(1, n - position):
                factorial *= i
          
            # Find the next digit to add to the result
            for digit in range(1, n + 1):
                if not used[digit]:
                    # Check if k is in the current block of permutations
                    if k > factorial:
                        # Skip this block of permutations
                        k -= factorial
                    else:
                        # Found the correct digit for this position
                        result.append(str(digit))
                        used[digit] = True
                        break
      
        return ''.join(result)
```

## Time and Space Complexity

The time complexity is `O(n²)` and the space complexity is `O(n)`.

**Time Complexity Analysis:**
- The outer loop runs `n` times (iterating through each position in the permutation)
- For each iteration of the outer loop:
  - Computing the factorial takes `O(n - i)` time, which is at most `O(n)` operations
  - The inner loop iterates through numbers 1 to `n` to find the next valid digit, taking `O(n)` time in the worst case
- Therefore, the total time complexity is `O(n) × O(n) = O(n²)`

**Space Complexity Analysis:**
- The `ans` list stores at most `n` characters: `O(n)`
- The `vis` boolean array has size `n + 1`: `O(n)`
- The variable `fact` and loop counters use constant space: `O(1)`
- Overall space complexity is `O(n) + O(n) + O(1) = O(n)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Off-by-One Error with k-indexing**
One of the most frequent mistakes is forgetting that the problem uses 1-based indexing for k (the k-th permutation starts from k=1, not k=0). Many developers instinctively work with 0-based indexing, which leads to incorrect results.

**Pitfall Example:**
```python
# Incorrect: treating k as 0-indexed throughout
def getPermutation(self, n: int, k: int) -> str:
    # This will give wrong results because the logic assumes 0-indexed k
    ...
```

**Solution:**
Convert k to 0-indexed at the beginning of the function:
```python
def getPermutation(self, n: int, k: int) -> str:
    k -= 1  # Convert to 0-indexed
    # Now use k as 0-indexed throughout the algorithm
    result = []
    used = [False] * (n + 1)
  
    for position in range(n):
        factorial = 1
        for i in range(1, n - position):
            factorial *= i
      
        for digit in range(1, n + 1):
            if not used[digit]:
                if k >= factorial:  # Note: >= instead of >
                    k -= factorial
                else:
                    result.append(str(digit))
                    used[digit] = True
                    break
  
    return ''.join(result)
```

### 2. **Recomputing the Factorial Every Iteration**
With `n <= 9`, the largest factorial needed is `8! = 40320`, so overflow is not a concern in any language. Recomputing the factorial from scratch at every position is still wasted work.

**Pitfall Example:**
```python
# Inefficient: recalculating factorial each time
for position in range(n):
    factorial = 1
    for i in range(1, n - position):
        factorial *= i  # Recalculating from scratch each time
```

**Solution:**
Pre-calculate all factorials or use division to update the factorial:
```python
def getPermutation(self, n: int, k: int) -> str:
    # Pre-calculate factorials
    factorials = [1] * n
    for i in range(1, n):
        factorials[i] = factorials[i-1] * i
  
    result = []
    used = [False] * (n + 1)
  
    for position in range(n):
        factorial = factorials[n - position - 1] if position < n - 1 else 1
        # Rest of the logic...
```

### 3. **Incorrect Handling of the Last Position**
When reaching the last position (when only one number remains), the factorial becomes 0! = 1, but some implementations might incorrectly handle this edge case.

**Pitfall Example:**
```python
# Might cause issues if not careful with edge cases
factorial = 1
for i in range(1, n - position):  # When position = n-1, this loop doesn't execute
    factorial *= i
# factorial remains 1, which is correct, but logic might break if not handled properly
```

**Solution:**
Explicitly handle or verify that the algorithm works correctly for the last position, or add a special case:
```python
for position in range(n):
    if position == n - 1:
        # Only one number left, add it directly
        for digit in range(1, n + 1):
            if not used[digit]:
                result.append(str(digit))
                break
    else:
        # Normal factorial-based selection
        factorial = 1
        for i in range(1, n - position):
            factorial *= i
        # ... rest of logic
```

