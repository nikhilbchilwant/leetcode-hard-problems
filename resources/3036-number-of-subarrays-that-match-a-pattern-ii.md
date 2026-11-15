# 3036. Number of Subarrays That Match a Pattern II

> Source: https://algo.monster/liteproblems/3036

## Problem Statement

You are given two integer arrays: `nums` (0-indexed, size n) and `pattern` (0-indexed, size m). The pattern array contains only the values -1, 0, and 1.

Your task is to find how many subarrays of `nums` with length `m + 1` match the given `pattern`.

A subarray `nums[i..j]` of size `m + 1` matches the pattern if it satisfies these conditions for each element `pattern[k]`:

- If `pattern[k] == 1`: The next element must be greater than the current element (`nums[i + k + 1] > nums[i + k]`)
- If `pattern[k] == 0`: The next element must be equal to the current element (`nums[i + k + 1] == nums[i + k]`)
- If `pattern[k] == -1`: The next element must be less than the current element (`nums[i + k + 1] < nums[i + k]`)

In other words, the pattern describes the relationship between consecutive elements in the subarray. A value of 1 means "increasing", 0 means "equal", and -1 means "decreasing".

For example, if `nums = [1, 2, 3, 4, 5, 6]` and `pattern = [1, 1]`, you need to find subarrays of length 3 where both transitions are increasing. The subarray `[1, 2, 3]` matches because `2 > 1` and `3 > 2`. Similarly, `[2, 3, 4]`, `[3, 4, 5]`, and `[4, 5, 6]` all match the pattern.

## Intuition

The key insight is recognizing that this problem is essentially a pattern matching problem in disguise. Instead of directly comparing subarrays of `nums`, we can transform the problem into finding occurrences of one sequence within another.

Think about what the pattern array really represents - it's describing the relationships between consecutive elements. For any two adjacent numbers in `nums`, their relationship can only be one of three things: the second is greater (1), equal (0), or less (-1) than the first.

So if we convert our entire `nums` array into these relationships, we get a new array of the same symbols that the pattern uses. For example, if `nums = [3, 4, 1, 2]`, the relationships between consecutive elements would be `[1, -1, 1]` (since 4>3, 1<4, and 2>1).

Now the problem becomes: find all occurrences of the `pattern` array within this converted relationship array. This is exactly the classic string pattern matching problem!

Why does this work? Because a subarray of length `m+1` in `nums` produces exactly `m` relationships between its consecutive elements. If these `m` relationships match our `pattern` of length `m`, then we've found a matching subarray.

The KMP algorithm is perfect here because it efficiently finds all occurrences of a pattern within a text in `O(n + m)` time. We use KMP instead of naive searching (which would be `O(n*m)`) to optimize the solution. The algorithm builds a partial match table (also called failure function) that helps skip unnecessary comparisons when a mismatch occurs, making the search much more efficient.

## Solution Approach

The solution implements the KMP (Knuth-Morris-Pratt) pattern matching algorithm to efficiently find all occurrences of the pattern. Let's walk through the implementation step by step:

**Step 1: Convert nums array to relationship array**

First, we transform the `nums` array into a sequence of relationships between consecutive elements:

```python
s = []
for i in range(1, len(nums)):
    if nums[i] > nums[i - 1]:
        s.append(1)
    elif nums[i] == nums[i - 1]:
        s.append(0)
    else:
        s.append(-1)
```

This creates an array `s` of length `n-1` where each element represents the relationship between `nums[i]` and `nums[i-1]`.

**Step 2: Build the KMP partial match table (failure function)**

The `partial` function builds the prefix function (pi array) for the pattern:

```python
def partial(s):
    g, pi = 0, [0] * len(s)
    for i in range(1, len(s)):
        while g and (s[g] != s[i]):
            g = pi[g - 1]
        pi[i] = g = g + (s[g] == s[i])
    return pi
```

The `pi[i]` stores the length of the longest proper prefix of `pattern[0..i]` that is also a suffix. This helps us skip comparisons when a mismatch occurs.

**Step 3: Find all pattern occurrences using KMP search**

The `match` function performs the actual pattern matching:

```python
def match(s, pat):
    pi = partial(pat)
    g, idx = 0, []
    for i in range(len(s)):
        while g and pat[g] != s[i]:
            g = pi[g - 1]  # Skip to the next possible match position
        g += pat[g] == s[i]  # Increment if current characters match
        if g == len(pi):  # Complete pattern found
            idx.append(i + 1 - g)  # Store starting index
            g = pi[g - 1]  # Continue searching for more occurrences
    return idx
```

The algorithm maintains a pointer `g` that tracks how many characters of the pattern have been matched so far. When `g` equals the pattern length, we've found a complete match. The starting position of the match is `i + 1 - g`.

**Step 4: Count the matches**

Finally, the main function combines everything:

```python
def countMatchingSubarrays(self, nums: List[int], pattern: List[int]) -> int:
    s = []
    # Convert nums to relationship array
    for i in range(1, len(nums)):
        if nums[i] > nums[i - 1]:
            s.append(1)
        elif nums[i] == nums[i - 1]:
            s.append(0)
        else:
            s.append(-1)
    # Find all occurrences and return count
    return len(match(s, pattern))
```

**Time Complexity:** `O(n + m)` where n is the length of nums and m is the length of pattern. The KMP algorithm processes each element at most twice.

**Space Complexity:** `O(n)` for storing the relationship array and `O(m)` for the partial match table, resulting in `O(n + m)` total space.

## Example Walkthrough

Let's walk through a concrete example to understand how the solution works.

**Given:**
- `nums = [1, 4, 4, 1, 3, 5, 5, 3]`
- `pattern = [1, 0, -1]`

We need to find subarrays of length 4 (pattern length + 1) where:
- The 2nd element > 1st element (pattern[0] = 1)
- The 3rd element = 2nd element (pattern[1] = 0)  
- The 4th element < 3rd element (pattern[2] = -1)

**Step 1: Convert nums to relationship array**

Compare consecutive elements in `nums`:
- nums[1] vs nums[0]: 4 > 1 → 1
- nums[2] vs nums[1]: 4 = 4 → 0
- nums[3] vs nums[2]: 1 < 4 → -1
- nums[4] vs nums[3]: 3 > 1 → 1
- nums[5] vs nums[4]: 5 > 3 → 1
- nums[6] vs nums[5]: 5 = 5 → 0
- nums[7] vs nums[6]: 3 < 5 → -1

Relationship array `s = [1, 0, -1, 1, 1, 0, -1]`

**Step 2: Build KMP partial match table for pattern [1, 0, -1]**

The partial function calculates:
- pi[0] = 0 (by definition)
- pi[1] = 0 (no prefix of [1, 0] matches a suffix)
- pi[2] = 0 (no prefix of [1, 0, -1] matches a suffix)

So `pi = [0, 0, 0]`

**Step 3: Search for pattern in relationship array**

Using KMP to find `[1, 0, -1]` in `[1, 0, -1, 1, 1, 0, -1]`:

- i=0, s[0]=1: matches pattern[0]=1, g=1
- i=1, s[1]=0: matches pattern[1]=0, g=2  
- i=2, s[2]=-1: matches pattern[2]=-1, g=3 → **Match found at index 0!**
- Continue: g=pi[2]=0
- i=3, s[3]=1: matches pattern[0]=1, g=1
- i=4, s[4]=1: doesn't match pattern[1]=0, reset g=0
- i=4, s[4]=1: matches pattern[0]=1, g=1
- i=5, s[5]=0: matches pattern[1]=0, g=2
- i=6, s[6]=-1: matches pattern[2]=-1, g=3 → **Match found at index 4!**

**Step 4: Interpret results**

We found matches at indices 0 and 4 in the relationship array:

1. **Index 0**: This corresponds to the subarray starting at index 0 in nums:
   - Subarray: `[1, 4, 4, 1]`
   - Relationships: 4>1 (✓), 4=4 (✓), 1<4 (✓)

2. **Index 4**: This corresponds to the subarray starting at index 4 in nums:
   - Subarray: `[3, 5, 5, 3]`  
   - Relationships: 5>3 (✓), 5=5 (✓), 3<5 (✓)

**Answer: 2 matching subarrays**

The algorithm efficiently found both matching subarrays by transforming the problem into pattern matching and using KMP to avoid redundant comparisons.

## Python Solution

```python
def partial(s):
    """
    Compute the partial match table (failure function) for KMP algorithm.
    pi[i] represents the length of the longest proper prefix of s[0:i+1] 
    that is also a suffix.
    """
    g, pi = 0, [0] * len(s)
  
    for i in range(1, len(s)):
        # While there's a mismatch and we haven't reached the beginning
        while g and (s[g] != s[i]):
            g = pi[g - 1]  # Fall back using the partial match table
      
        # Update pi[i] and g based on whether characters match
        pi[i] = g = g + (s[g] == s[i])
  
    return pi


def match(s, pat):
    """
    Find all occurrences of pattern 'pat' in string 's' using KMP algorithm.
    Returns a list of starting indices where the pattern is found.
    """
    pi = partial(pat)  # Build the partial match table for the pattern
    g, idx = 0, []
  
    for i in range(len(s)):
        # Handle mismatch by falling back using partial match table
        while g and pat[g] != s[i]:
            g = pi[g - 1]
      
        # Increment g if characters match
        g += pat[g] == s[i]
      
        # Pattern completely matched
        if g == len(pi):
            idx.append(i + 1 - g)  # Add starting index of the match
            g = pi[g - 1]  # Continue searching for more occurrences
  
    return idx


def string_find(s, pat):
    """
    Check if pattern 'pat' exists in string 's' using KMP algorithm.
    Returns True if pattern is found, False otherwise.
    """
    pi = partial(pat)  # Build the partial match table for the pattern
    g = 0
  
    for i in range(len(s)):
        # Handle mismatch by falling back using partial match table
        while g and pat[g] != s[i]:
            g = pi[g - 1]
      
        # Increment g if characters match
        g += pat[g] == s[i]
      
        # Pattern completely matched
        if g == len(pi):
            return True
  
    return False


from typing import List

class Solution:
    def countMatchingSubarrays(self, nums: List[int], pattern: List[int]) -> int:
        """
        Count the number of subarrays in 'nums' that match the given 'pattern'.
        Pattern matching is based on consecutive element comparisons:
        - 1: next element is greater than current
        - 0: next element equals current
        - -1: next element is less than current
        """
        # Convert nums array to pattern representation
        s = []
        for i in range(1, len(nums)):
            if nums[i] > nums[i - 1]:
                s.append(1)  # Increasing
            elif nums[i] == nums[i - 1]:
                s.append(0)  # Equal
            else:
                s.append(-1)  # Decreasing
      
        # Find all occurrences of pattern in the converted array
        return len(match(s, pattern))
```

## Time and Space Complexity

**Time Complexity:** `O(n + m)` where `n` is the length of the `nums` array and `m` is the length of the `pattern` array.

- The `partial` function computes the prefix function (failure function) for the pattern, which takes `O(m)` time. Although there's a while loop inside the for loop, each character in the pattern causes at most `O(1)` amortized work because `g` can only increase by 1 per iteration and decrease by at most the total amount it has increased.

- Converting `nums` to the difference array `s` takes `O(n-1) = O(n)` time through a single pass.

- The `match` function performs KMP pattern matching on the transformed array `s` (of length `n-1`) with the pattern (of length `m`). This takes `O(n + m)` time. Similar to the `partial` function, the while loop provides `O(1)` amortized time per character because `g` can only increase by 1 per iteration and the total decreases are bounded by total increases.

**Space Complexity:** `O(n + m)`

- The `partial` function creates a `pi` array of size `m`, requiring `O(m)` space.

- The transformed array `s` stores `n-1` elements, requiring `O(n)` space.

- The `match` function uses the `pi` array of size `O(m)` and an `idx` list that could potentially store up to `O(n)` indices in the worst case (when the pattern matches at every possible position).

- Additional variables like `g` and loop counters use `O(1)` space.

The algorithm implements the KMP (Knuth-Morris-Pratt) string matching algorithm to find all occurrences of the pattern in the transformed difference array efficiently.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Off-by-One Error in Index Calculation**

A common mistake is misunderstanding the relationship between the pattern array indices and the nums array indices. The pattern has length `m`, but it describes relationships between `m+1` elements in nums.

**Pitfall Example:**
```python
# WRONG: Trying to directly use pattern length as subarray length
for i in range(len(nums) - len(pattern)):  # This is incorrect!
    # Check if nums[i:i+len(pattern)] matches pattern
```

**Correct Approach:**
```python
# The subarray length should be len(pattern) + 1
for i in range(len(nums) - len(pattern)):
    # Check if nums[i:i+len(pattern)+1] matches pattern
    # This subarray has len(pattern)+1 elements
```

### 2. **Incorrect Pattern Conversion**

When converting the nums array to a pattern representation, developers might accidentally create the wrong comparison or use incorrect indices.

**Pitfall Example:**
```python
# WRONG: Comparing in the wrong direction
s = []
for i in range(1, len(nums)):
    if nums[i-1] > nums[i]:  # Backwards comparison!
        s.append(1)
    elif nums[i-1] == nums[i]:
        s.append(0)
    else:
        s.append(-1)
```

**Correct Approach:**
```python
# Compare current element with previous element
s = []
for i in range(1, len(nums)):
    if nums[i] > nums[i-1]:  # Current > Previous means increasing
        s.append(1)
    elif nums[i] == nums[i-1]:
        s.append(0)
    else:
        s.append(-1)
```

### 3. **Edge Case: Empty Pattern or Single Element**

The code might fail when the pattern is empty or when nums has fewer elements than needed.

**Pitfall Example:**
```python
def countMatchingSubarrays(self, nums: List[int], pattern: List[int]) -> int:
    # Doesn't handle edge cases
    s = []
    for i in range(1, len(nums)):
        # ... conversion logic
    return len(match(s, pattern))
```

**Correct Approach with Edge Case Handling:**
```python
def countMatchingSubarrays(self, nums: List[int], pattern: List[int]) -> int:
    # Handle edge cases
    if not pattern:
        return len(nums) - 1  # Every single element is a valid subarray
    if len(nums) <= len(pattern):
        return 0  # Not enough elements to form a matching subarray
  
    s = []
    for i in range(1, len(nums)):
        if nums[i] > nums[i-1]:
            s.append(1)
        elif nums[i] == nums[i-1]:
            s.append(0)
        else:
            s.append(-1)
  
    return len(match(s, pattern))
```

### 4. **KMP Implementation Bug: Not Resetting After Match**

In the KMP matching function, forgetting to reset `g` after finding a match will cause the algorithm to miss overlapping patterns.

**Pitfall Example:**
```python
def match(s, pat):
    pi = partial(pat)
    g, idx = 0, []
    for i in range(len(s)):
        while g and pat[g] != s[i]:
            g = pi[g - 1]
        g += pat[g] == s[i]
        if g == len(pi):
            idx.append(i + 1 - g)
            g = 0  # WRONG: Resetting to 0 misses overlapping patterns!
    return idx
```

**Correct Approach:**
```python
def match(s, pat):
    pi = partial(pat)
    g, idx = 0, []
    for i in range(len(s)):
        while g and pat[g] != s[i]:
            g = pi[g - 1]
        g += pat[g] == s[i]
        if g == len(pi):
            idx.append(i + 1 - g)
            g = pi[g - 1]  # CORRECT: Use failure function to find overlapping matches
    return idx
```

### 5. **Type Mismatch in Pattern Values**

The pattern uses integers (-1, 0, 1), but developers might accidentally use different data types or values.

**Pitfall Example:**
```python
# Using strings instead of integers
s = []
for i in range(1, len(nums)):
    if nums[i] > nums[i-1]:
        s.append("1")  # String instead of integer!
    elif nums[i] == nums[i-1]:
        s.append("0")
    else:
        s.append("-1")
```

This would cause the KMP matching to fail since the pattern array contains integers, not strings. Always ensure consistent data types between the converted array and the pattern.

