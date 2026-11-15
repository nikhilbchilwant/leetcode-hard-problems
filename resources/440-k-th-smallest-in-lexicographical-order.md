# 440. K-th Smallest in Lexicographical Order

> Source: https://algo.monster/liteproblems/440

## Problem Statement

You are given two integers `n` and `k`. Your task is to find the `k`-th smallest integer when all integers from 1 to `n` are sorted in **lexicographical order** (dictionary order).

Lexicographical order means numbers are sorted as if they were strings. For example, if `n = 13`, the lexicographical ordering would be:
`1, 10, 11, 12, 13, 2, 3, 4, 5, 6, 7, 8, 9`

Notice how `10, 11, 12, 13` come immediately after `1` because when compared as strings, "10" comes before "2" (since "1" < "2" when comparing the first character).

The challenge is to directly find the `k`-th number in this ordering without generating and sorting all numbers from 1 to `n`, which would be inefficient for large values of `n` (up to `10^9`).

The solution uses a clever approach by treating the numbers as a 10-ary tree (trie) structure where each node represents a numeric prefix. Starting from `curr = 1`, it counts how many valid numbers exist in each subtree and decides whether to:
- Move to the next sibling (when the `k`-th number is not in the current subtree)
- Go deeper into the current subtree (when the `k`-th number is within it)

The `count` function calculates how many valid numbers (≤ `n`) exist with a given prefix by iterating through each level of the tree and summing up the valid numbers at each level.

## Intuition

The key insight is recognizing that lexicographical ordering follows a pattern similar to how we traverse a tree structure. When we write numbers in lexicographical order, we're essentially doing a depth-first traversal of an imaginary tree where each node represents a number prefix.

Think about how numbers are organized lexicographically: after `1`, we don't go to `2` immediately. Instead, we explore all numbers that start with `1`: `10, 11, 12, ..., 19`, then `100, 101, ...`, and so on. This is exactly like traversing a tree where `1` is the root of a subtree containing all numbers with prefix `1`.

Since we need the `k`-th number but can't afford to generate all numbers up to `n`, we need a way to "skip" entire groups of numbers. This leads to the counting strategy: if we can calculate how many valid numbers exist under any prefix (like how many numbers start with `1` and are ≤ `n`), we can decide whether our target `k`-th number is in that group or not.

The traversal strategy emerges naturally:
- Start at `1` (the lexicographically smallest positive integer)
- At each step, ask: "Is the `k`-th number in the current subtree?"
- If yes, dive deeper by appending a digit (multiply by 10)
- If no, skip to the next sibling (add 1)

The counting mechanism is clever: for a prefix `curr`, we count level by level. At each level, we're looking at ranges like `[curr, next)` where `next = curr + 1` initially. We then expand both by multiplying by 10 to go to the next level: `[curr*10, next*10)`. The count at each level is constrained by `n`, so we take `min(n - curr + 1, next - curr)`.

This approach essentially simulates the lexicographical traversal without actually generating the numbers, making it efficient even for large `n`.

## Solution Approach

The solution implements a **Trie-based counting with greedy traversal** strategy. We conceptually treat the range `[1, n]` as a 10-ary prefix tree where each node represents a numeric prefix.

### Core Algorithm Structure

We maintain a variable `curr` representing the current number/prefix we're examining, starting at `1`. We also adjust `k` to be 0-indexed by decrementing it initially (`k -= 1`).

### The Counting Function

The `count(curr)` function calculates how many valid numbers (≤ `n`) exist with prefix `curr`:

```python
def count(curr):
    next, cnt = curr + 1, 0
    while curr <= n:
        cnt += min(n - curr + 1, next - curr)
        next, curr = next * 10, curr * 10
    return cnt
```

This function works level by level:
- Initially, we're counting numbers in range `[curr, next)` where `next = curr + 1`
- At each iteration, we count valid numbers at the current level: `min(n - curr + 1, next - curr)`
  - `next - curr` gives the full range size at this level
  - `n - curr + 1` ensures we don't count beyond `n`
- We then move to the next level by multiplying both boundaries by 10

For example, if `curr = 1` and `n = 234`:
- Level 1: Count in `[1, 2)` → adds 1
- Level 2: Count in `[10, 20)` → adds 10 
- Level 3: Count in `[100, 200)` → adds 100
- Level 4: Count in `[1000, 2000)` → but `1000 > 234`, so we stop

### The Main Traversal Logic

```python
curr = 1
k -= 1  # Convert to 0-indexed
while k:
    cnt = count(curr)
    if k >= cnt:
        k -= cnt
        curr += 1  # Skip to next sibling
    else:
        k -= 1
        curr *= 10  # Go deeper into subtree
return curr
```

At each step, we make a greedy decision:

1. **Calculate subtree size**: `cnt = count(curr)` tells us how many numbers have prefix `curr`

2. **Decision branching**:
   - If `k >= cnt`: The target is not in this subtree. We skip all `cnt` numbers and move to the next sibling (`curr + 1`)
   - If `k < cnt`: The target is within this subtree. We descend one level deeper (`curr * 10`) and decrement `k` by 1 (accounting for the current node itself)

3. **Termination**: When `k` becomes 0, we've found our target number

### Example Walkthrough

For `n = 13, k = 2`:
- Start: `curr = 1, k = 1` (after converting to 0-indexed)
- `count(1) = 5` (numbers: 1, 10, 11, 12, 13)
- Since `k = 1 < 5`, target is in subtree of 1
- Update: `curr = 10, k = 0`
- `k = 0`, so return `10`

This approach efficiently navigates through the lexicographical ordering without generating all numbers, achieving `O(log n)` time complexity per level with at most `O(log n)` levels.

## Example Walkthrough

Let's trace through finding the 6th lexicographically smallest number when n = 15.

The lexicographical ordering for n = 15 is: `1, 10, 11, 12, 13, 14, 15, 2, 3, 4, 5, 6, 7, 8, 9`

**Initial Setup:**
- `curr = 1`, `k = 6 - 1 = 5` (converting to 0-indexed)

**Step 1:**
- Current position: `curr = 1`
- Count how many numbers have prefix 1: `count(1)`
  - Level 1: `[1, 2)` → 1 number
  - Level 2: `[10, 20)` but limited by n=15 → min(15-10+1, 20-10) = 6 numbers (10,11,12,13,14,15)
  - Total: 7 numbers with prefix 1
- Decision: Since `k = 5 < 7`, our target is in the subtree of 1
- Action: Go deeper → `curr = 1 * 10 = 10`, `k = 5 - 1 = 4`

**Step 2:**
- Current position: `curr = 10`
- Count how many numbers have prefix 10: `count(10)`
  - Level 1: `[10, 11)` → 1 number (just 10)
  - Level 2: `[100, 110)` but 100 > 15, so stop
  - Total: 1 number with prefix 10
- Decision: Since `k = 4 >= 1`, our target is NOT in the subtree of 10
- Action: Skip to sibling → `curr = 10 + 1 = 11`, `k = 4 - 1 = 3`

**Step 3:**
- Current position: `curr = 11`
- Count how many numbers have prefix 11: `count(11)`
  - Level 1: `[11, 12)` → 1 number (just 11)
  - Level 2: `[110, 120)` but 110 > 15, so stop
  - Total: 1 number with prefix 11
- Decision: Since `k = 3 >= 1`, our target is NOT in the subtree of 11
- Action: Skip to sibling → `curr = 11 + 1 = 12`, `k = 3 - 1 = 2`

**Step 4:**
- Current position: `curr = 12`
- Count how many numbers have prefix 12: `count(12)`
  - Level 1: `[12, 13)` → 1 number (just 12)
  - Total: 1 number with prefix 12
- Decision: Since `k = 2 >= 1`, our target is NOT in the subtree of 12
- Action: Skip to sibling → `curr = 12 + 1 = 13`, `k = 2 - 1 = 1`

**Step 5:**
- Current position: `curr = 13`
- Count how many numbers have prefix 13: `count(13)`
  - Level 1: `[13, 14)` → 1 number (just 13)
  - Total: 1 number with prefix 13
- Decision: Since `k = 1 >= 1`, our target is NOT in the subtree of 13
- Action: Skip to sibling → `curr = 13 + 1 = 14`, `k = 1 - 1 = 0`

**Step 6:**
- `k = 0`, so we've found our answer!
- Return `curr = 14`

The 6th number in lexicographical order is **14**, which matches our expected ordering.

## Python Solution

```python
class Solution:
    def findKthNumber(self, n: int, k: int) -> int:
        """
        Find the k-th smallest number in lexicographical order from 1 to n.
      
        Args:
            n: The upper bound of the range [1, n]
            k: The position of the number to find (1-indexed)
          
        Returns:
            The k-th smallest number in lexicographical order
        """
      
        def count_steps_between_prefixes(current_prefix: int) -> int:
            """
            Count how many numbers exist in the lexicographical range 
            starting with current_prefix up to the next prefix.
          
            For example, if current_prefix = 1, this counts all numbers
            starting with 1 (like 1, 10, 11, ..., 19, 100, 101, ...) 
            up to but not including those starting with 2.
          
            Args:
                current_prefix: The current prefix to count from
              
            Returns:
                Number of valid numbers with this prefix within range [1, n]
            """
            next_prefix = current_prefix + 1
            steps_count = 0
          
            # Count numbers at each level of the tree
            # Level 1: single digits (1-9)
            # Level 2: double digits (10-99)
            # Level 3: triple digits (100-999), etc.
            while current_prefix <= n:
                # Count valid numbers at this level
                # Either count all numbers between current_prefix and next_prefix-1
                # Or stop at n if it's smaller
                steps_count += min(n - current_prefix + 1, next_prefix - current_prefix)
              
                # Move to next level (add a digit)
                next_prefix *= 10
                current_prefix *= 10
              
            return steps_count
      
        # Start with number 1
        current_number = 1
      
        # We've already counted the first number (1), so decrement k
        k -= 1
      
        # Find the k-th number by navigating the lexicographical tree
        while k > 0:
            # Count how many numbers are under the current prefix
            steps_to_next_prefix = count_steps_between_prefixes(current_number)
          
            if k >= steps_to_next_prefix:
                # Skip entire subtree and move to next sibling
                k -= steps_to_next_prefix
                current_number += 1  # Move to next prefix at same level
            else:
                # The target is within current subtree, go deeper
                k -= 1  # Count current number
                current_number *= 10  # Go to first child (append 0)
              
        return current_number
```

## Time and Space Complexity

**Time Complexity:** `O(log^2 n)`

The algorithm traverses a conceptual trie structure of numbers from 1 to n in lexicographical order. The main loop runs at most `O(log n)` times because in the worst case, we traverse down the trie by multiplying `curr` by 10, which can happen at most `log₁₀ n` times. 

Within each iteration of the main loop, we call the `count` function which also runs `O(log n)` times. The `count` function calculates how many numbers exist with a given prefix by iterating through each level of the trie (each digit position), which takes `O(log n)` operations since we're essentially counting numbers digit by digit.

Therefore, the overall time complexity is `O(log n) × O(log n) = O(log^2 n)`.

**Space Complexity:** `O(1)`

The algorithm uses only a constant amount of extra space. The variables used are:
- `curr`: tracks the current number/prefix
- `k`: decremented counter
- In the `count` function: `next`, `cnt` for temporary calculations

No additional data structures like arrays or recursive call stacks are used, so the space complexity remains constant regardless of the input size.

## Common Pitfalls

### 1. Off-by-One Error in Index Conversion
**Pitfall**: Forgetting to convert from 1-indexed to 0-indexed positioning, or incorrectly handling the conversion.

Many developers mistakenly keep `k` as 1-indexed throughout the algorithm, leading to returning the (k+1)-th number instead of the k-th number.

**Incorrect approach:**
```python
# Wrong: Not adjusting k at all
curr = 1
while k > 0:  # This will iterate one extra time
    cnt = count(curr)
    if k > cnt:  # Wrong comparison
        k -= cnt
        curr += 1
    else:
        curr *= 10
```

**Correct approach:**
```python
# Convert to 0-indexed immediately
k -= 1
curr = 1
while k > 0:
    # ... rest of logic
```

### 2. Integer Overflow in Count Function
**Pitfall**: In languages with 32-bit integers, `next` overflows. Python integers do not overflow, but in Java or C++ with `n` close to `10^9`, the loop multiplies `next` by 10 until `curr` exceeds `n`, so `next` can reach about `10^10`, which does not fit in a 32-bit `int`. The wrapped value makes `next - curr` meaningless.

**Problematic code (Java):**
```java
int count(int curr) {
    int next = curr + 1, cnt = 0;
    while (curr <= n) {
        cnt += Math.min(n - curr + 1, next - curr);
        next *= 10;  // overflows for large n
        curr *= 10;  // can also overflow
    }
    return cnt;
}
```

**Correct approach:** Use 64-bit values for `curr`, `next`, and the count:
```java
long count(long curr) {
    long next = curr + 1, cnt = 0;
    while (curr <= n) {
        cnt += Math.min(n - curr + 1, next - curr);
        next *= 10;
        curr *= 10;
    }
    return cnt;
}
```

The two ways of writing the level count, `min(n - curr + 1, next - curr)` and `min(n + 1, next) - curr`, are equal, so either is fine once the types are wide enough.

### 3. Incorrect Subtree Traversal Logic
**Pitfall**: Confusing when to move to sibling vs. when to go deeper into the subtree, especially with the k decrement.

**Common mistake:**
```python
while k > 0:
    cnt = count(curr)
    if k > cnt:  # Wrong: should be k >= cnt
        k -= cnt
        curr += 1
    else:
        # Forgot to decrement k here
        curr *= 10
```

**Correct logic:**
```python
while k > 0:
    cnt = count(curr)
    if k >= cnt:
        # Skip entire subtree
        k -= cnt
        curr += 1
    else:
        # Go deeper, accounting for current node
        k -= 1
        curr *= 10
```

### 4. Boundary Condition in Count Function
**Pitfall**: Not properly handling the upper bound `n` when counting numbers in a range.

**Incorrect:**
```python
def count(curr):
    next = curr + 1
    cnt = 0
    while curr <= n:
        # Wrong: doesn't account for n being within the range
        cnt += next - curr
        next *= 10
        curr *= 10
    return cnt
```

This would count numbers beyond `n`. For example, if `curr = 10`, `next = 20`, and `n = 15`, it would incorrectly count 10 numbers instead of 6.

**Solution:**
Always use `min(n - curr + 1, next - curr)` to ensure you don't count beyond `n`. The `n - curr + 1` gives the exact count of numbers from `curr` to `n` inclusive.

