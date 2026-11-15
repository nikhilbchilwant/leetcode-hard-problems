# 2355. Maximum Number of Books You Can Take

> Source: https://algo.monster/liteproblems/2355

## Problem Statement

You have a bookshelf with `n` shelves arranged in a row, numbered from `0` to `n-1`. Each shelf contains a certain number of books, given by the array `books` where `books[i]` represents the number of books on shelf `i`.

You want to take books from a **contiguous section** of the bookshelf. This means you'll choose a starting shelf `l` and an ending shelf `r` (where `0 <= l <= r < n`), and take books from all shelves in the range `[l, r]`.

However, there's a constraint on how many books you can take from each shelf:
- For any two consecutive shelves `i` and `i+1` within your chosen range (where `l <= i < r`), you must take **strictly fewer** books from shelf `i` than from shelf `i+1`
- This creates a strictly increasing sequence of books taken from left to right within your chosen range

The goal is to find the **maximum total number of books** you can take while following this constraint.

For example, if `books = [8, 5, 2, 7, 9]` and you choose the range `[1, 4]`, you might take:
- 1 book from shelf 1
- 2 books from shelf 2
- 7 books from shelf 3
- 9 books from shelf 4

This gives a total of 19 books, where each shelf has strictly more books taken than the previous one in the range.

Note that you don't have to take all books from a shelf - you can take any number from 0 up to `books[i]` from shelf `i`, as long as the strictly increasing constraint is satisfied.

## Intuition

The key insight is that for any ending position `i`, we want to find the optimal starting position where we can form a strictly increasing sequence of books taken, maximizing the total.

First, let's think about what makes a valid strictly increasing sequence. If we end at position `i` and take `books[i]` books from it, then at position `i-1` we can take at most `books[i]-1` books, at position `i-2` we can take at most `books[i]-2` books, and so on. This forms an arithmetic sequence counting backwards.

However, we're limited by the actual number of books available on each shelf. So at position `j`, we can take `min(books[j], books[i] - (i - j))` books. This means we take either all available books or the amount dictated by our arithmetic sequence, whichever is smaller.

The challenge is finding where to start our sequence. We can't always extend backwards indefinitely because:
1. We might run out of shelves (reach index 0)
2. The arithmetic sequence might demand negative books (when `books[i] - (i - j) <= 0`)
3. A shelf might have too few books to maintain the strictly increasing pattern

To handle this efficiently, we transform the problem. We create a new array `nums[i] = books[i] - i`. This transformation helps us identify "blocking points" - positions where we cannot extend our arithmetic sequence further backward. If `nums[j] >= nums[i]` for some `j < i`, then position `j` blocks us from extending past it when ending at position `i`.

We use a monotonic [stack](/problems/stack_intro) to find, for each position `i`, the nearest left position that would block our sequence. This gives us array `left[i]`, where `left[i]` is the rightmost position to the left of `i` that has `nums[left[i]] >= nums[i]`.

Finally, we use [dynamic programming](/problems/dynamic_programming_intro). For each position `i`:
- We find how many positions we can include in our sequence (from `left[i] + 1` to `i`)
- We calculate the sum of the arithmetic sequence for these positions
- We add the optimal sum from position `left[i]` (if it exists) since we can combine non-overlapping ranges

The maximum value among all `dp[i]` gives us our answer.

**Learn more about [Stack](/problems/stack_intro), [Dynamic Programming](/problems/dynamic_programming_intro) and [Monotonic Stack](/problems/mono_stack_intro) patterns.**

## Solution Approach

Let's walk through the implementation step by step:

**Step 1: Transform the array**
```python
nums = [v - i for i, v in enumerate(books)]
```
We create a transformed array where `nums[i] = books[i] - i`. This transformation helps identify blocking points. If `nums[j] >= nums[i]` for `j < i`, then position `j` prevents us from extending our arithmetic sequence past it when ending at position `i`.

**Step 2: Find blocking positions using monotonic [stack](/problems/stack_intro)**
```python
left = [-1] * n
stk = []
for i, v in enumerate(nums):
    while stk and nums[stk[-1]] >= v:
        stk.pop()
    if stk:
        left[i] = stk[-1]
    stk.append(i)
```
We maintain a monotonic increasing stack to find, for each position `i`, the nearest left position that blocks our sequence:
- Pop elements from stack while their `nums` values are `>= v`
- The remaining top of stack (if exists) is our blocking position
- Add current index to stack
- `left[i] = -1` means no blocking position exists (we can extend to the beginning)

**Step 3: [Dynamic programming](/problems/dynamic_programming_intro) to find maximum books**
```python
dp = [0] * n
dp[0] = books[0]
for i, v in enumerate(books):
    j = left[i]
    cnt = min(v, i - j)
    u = v - cnt + 1
    s = (u + v) * cnt // 2
    dp[i] = s + (0 if j == -1 else dp[j])
    ans = max(ans, dp[i])
```

For each position `i`:
- `j = left[i]`: Get the blocking position
- `cnt = min(v, i - j)`: Calculate how many positions we can include in our sequence
  - Limited by either the number of books at position `i` or the distance to blocking position
- `u = v - cnt + 1`: Calculate the first term of our arithmetic sequence
  - If we take `v` books from position `i`, we take `v-1` from `i-1`, `v-2` from `i-2`, etc.
  - The first term is `v - (cnt - 1) = v - cnt + 1`
- `s = (u + v) * cnt // 2`: Sum of arithmetic sequence from `u` to `v`
  - Using formula: sum = (first + last) × count / 2
- `dp[i] = s + dp[j]`: Add the optimal sum from the blocking position (if it exists)
- Track the maximum value seen

**Time Complexity:** `O(n)` - single pass for [stack](/problems/stack_intro) and single pass for DP
**Space Complexity:** `O(n)` - for the auxiliary arrays `nums`, `left`, `dp`, and stack

## Example Walkthrough

Let's walk through a small example with `books = [8, 5, 2, 7, 9]`.

**Step 1: Transform the array**
- `nums[0] = 8 - 0 = 8`
- `nums[1] = 5 - 1 = 4`
- `nums[2] = 2 - 2 = 0`
- `nums[3] = 7 - 3 = 4`
- `nums[4] = 9 - 4 = 5`

So `nums = [8, 4, 0, 4, 5]`

**Step 2: Find blocking positions using monotonic stack**

Processing each index:
- `i=0`: Stack empty, `left[0] = -1`, push 0. Stack: `[0]`
- `i=1`: `nums[0]=8 >= 4`, pop 0. Stack empty, `left[1] = -1`, push 1. Stack: `[1]`
- `i=2`: `nums[1]=4 >= 0`, pop 1. Stack empty, `left[2] = -1`, push 2. Stack: `[2]`
- `i=3`: `nums[2]=0 < 4`, `left[3] = 2`, push 3. Stack: `[2, 3]`
- `i=4`: `nums[3]=4 < 5`, `left[4] = 3`, push 4. Stack: `[2, 3, 4]`

Result: `left = [-1, -1, -1, 2, 3]`

**Step 3: Dynamic programming**

For each position `i`:

**i=0**:
- `j = -1` (no blocking position)
- `cnt = min(8, 0-(-1)) = min(8, 1) = 1`
- Taking 8 books from shelf 0
- `u = 8 - 1 + 1 = 8`
- `s = (8 + 8) × 1 / 2 = 8`
- `dp[0] = 8 + 0 = 8`

**i=1**:
- `j = -1` (no blocking position)
- `cnt = min(5, 1-(-1)) = min(5, 2) = 2`
- Can take: 4 books from shelf 0, 5 books from shelf 1
- `u = 5 - 2 + 1 = 4`
- `s = (4 + 5) × 2 / 2 = 9`
- `dp[1] = 9 + 0 = 9`

**i=2**:
- `j = -1` (no blocking position)
- `cnt = min(2, 2-(-1)) = min(2, 3) = 2`
- Can take: 1 book from shelf 1, 2 books from shelf 2
- `u = 2 - 2 + 1 = 1`
- `s = (1 + 2) × 2 / 2 = 3`
- `dp[2] = 3 + 0 = 3`

**i=3**:
- `j = 2` (blocked by position 2)
- `cnt = min(7, 3-2) = min(7, 1) = 1`
- Taking 7 books from shelf 3 only
- `u = 7 - 1 + 1 = 7`
- `s = (7 + 7) × 1 / 2 = 7`
- `dp[3] = 7 + dp[2] = 7 + 3 = 10`

**i=4**:
- `j = 3` (blocked by position 3)
- `cnt = min(9, 4-3) = min(9, 1) = 1`
- Taking 9 books from shelf 4 only
- `u = 9 - 1 + 1 = 9`
- `s = (9 + 9) × 1 / 2 = 9`
- `dp[4] = 9 + dp[3] = 9 + 10 = 19`

**Answer: 19**

This corresponds to taking books from two separate ranges:
- Range [1,2]: Take 1 book from shelf 1 and 2 books from shelf 2 (total: 3)
- Range [3,4]: Take 7 books from shelf 3 and 9 books from shelf 4 (total: 16)
- Combined total: 19 books

## Python Solution

```python
class Solution:
    def maximumBooks(self, books: List[int]) -> int:
        # Transform array: subtract index from each value
        # This helps identify positions where we need to restart counting
        adjusted_values = [value - index for index, value in enumerate(books)]
        n = len(adjusted_values)

        # left[i] stores the index of the previous element with adjusted_value < adjusted_values[i]
        # -1 means no such element exists
        left_boundary = [-1] * n

        # Monotonic stack to find previous smaller element
        stack = []
        for i, current_value in enumerate(adjusted_values):
            # Pop elements from stack that are >= current value
            while stack and adjusted_values[stack[-1]] >= current_value:
                stack.pop()

            # If stack has elements, top element is the previous smaller element
            if stack:
                left_boundary[i] = stack[-1]

            stack.append(i)

        # Dynamic programming to calculate maximum books
        max_books = 0
        dp = [0] * n  # dp[i] = maximum books we can take ending at position i
        dp[0] = books[0]

        for i, shelf_books in enumerate(books):
            # j is the left boundary (previous position with smaller adjusted value)
            j = left_boundary[i]

            # Calculate how many consecutive positions we can take books from
            # We can take at most 'shelf_books' books and at most (i - j) positions
            consecutive_positions = min(shelf_books, i - j)

            # Calculate the starting value of the arithmetic sequence
            # We take books in decreasing order: shelf_books, shelf_books-1, ..., start_value
            start_value = shelf_books - consecutive_positions + 1

            # Sum of arithmetic sequence from start_value to shelf_books
            arithmetic_sum = (start_value + shelf_books) * consecutive_positions // 2

            # Add books from current sequence plus optimal solution up to position j
            dp[i] = arithmetic_sum + (0 if j == -1 else dp[j])

            # Update maximum books found so far
            max_books = max(max_books, dp[i])

        return max_books
```

## Time and Space Complexity

The time complexity is `O(n)`, where `n` is the length of the books array. The algorithm consists of:
- A first pass to create the `nums` array: `O(n)`
- A monotonic stack operation to fill the `left` array: `O(n)` - each element is pushed and popped at most once
- A final pass to compute the DP values: `O(n)`

The space complexity is `O(n)` due to:
- The `nums` array storing `n` elements
- The `left` array storing `n` elements
- The `dp` array storing `n` elements
- The stack `stk` which can contain at most `n` elements in the worst case

Note: The reference answer appears to be for a different problem involving a matrix grid with `O(n^3)` time complexity, not for this books shelf problem which uses dynamic programming with a monotonic stack.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Integer Overflow in Arithmetic Sum Calculation**

**The Pitfall:**
When calculating the sum of an arithmetic sequence using `(start_value + shelf_books) * consecutive_positions // 2`, the intermediate multiplication can cause integer overflow for large values. In Python, this isn't typically an issue due to arbitrary precision integers, but in languages like Java or C++, this can lead to incorrect results.

**Example scenario:**
- If `shelf_books = 10^9` and `consecutive_positions = 10^5`
- The multiplication `(start_value + shelf_books) * consecutive_positions` could exceed the integer limit before division

**Solution:**
Use a safer formula or ensure proper type casting:
```python
# Instead of:
arithmetic_sum = (start_value + shelf_books) * consecutive_positions // 2

# Use:
arithmetic_sum = consecutive_positions * start_value + consecutive_positions * (consecutive_positions - 1) // 2
# This breaks down the calculation to avoid large intermediate values
```

### 2. **Misunderstanding the Adjusted Values Transformation**

**The Pitfall:**
The transformation `adjusted_values[i] = books[i] - i` might seem arbitrary and developers often make mistakes by:
- Using the wrong comparison operator when checking blocking positions
- Forgetting why we need `>=` instead of `>` in the monotonic stack comparison

**Why this matters:**
If position `j` has `adjusted_values[j] >= adjusted_values[i]` where `j < i`, it means we cannot extend a strictly increasing sequence from position `j` through position `i`. This is because:
- At position `j`: we can take at most `books[j]` books
- At position `i`: we need to take at least `books[j] + (i-j)` books for strict increase
- But if `books[j] - j >= books[i] - i`, then `books[j] + (i-j) > books[i]`, which is impossible

**Solution:**
Always validate the logic with a simple example:
```python
# Example: books = [4, 5, 3, 8]
# adjusted = [4, 4, 1, 5]
# Position 1 blocks position 0 because adjusted[0] >= adjusted[1]
# We cannot have a strictly increasing sequence from index 0 through index 1
```

### 3. **Off-by-One Error in Consecutive Positions Calculation**

**The Pitfall:**
Calculating `consecutive_positions = min(shelf_books, i - j)` seems straightforward, but it's easy to make an off-by-one error by using `i - j - 1` or `i - j + 1`.

**Why `i - j` is correct:**
- If `j = -1` (no blocking position), then `i - j = i + 1`, which means we can use positions `[0, 1, ..., i]` (total of `i+1` positions)
- If `j = 2` and `i = 5`, then `i - j = 3`, meaning we can use positions `[3, 4, 5]` (total of 3 positions, starting right after position 2)

**Solution:**
Test with concrete examples:
```python
# If j = -1, i = 2: we can use positions [0, 1, 2] → count = 3 = i - j
# If j = 1, i = 4: we can use positions [2, 3, 4] → count = 3 = i - j
```

### 4. **Incorrect DP Initialization**

**The Pitfall:**
Forgetting to initialize `dp[0] = books[0]` or incorrectly initializing all dp values to 0 can lead to wrong answers for cases where the optimal solution includes the first shelf.

**Solution:**
Always handle the base case explicitly:
```python
dp = [0] * n
dp[0] = books[0]  # Critical: first position can always take all its books
max_books = books[0]  # Don't forget to consider dp[0] in the maximum
```

