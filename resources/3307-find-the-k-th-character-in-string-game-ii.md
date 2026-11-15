# 3307. Find the K-th Character in String Game II

> Source: https://algo.monster/liteproblems/3307

## Problem Statement

Alice and Bob are playing a game where Alice starts with a string `word = "a"`.

Given a positive integer `k` and an integer array `operations`, Bob asks Alice to perform all operations in sequence. Each `operations[i]` determines the type of operation:

- If `operations[i] == 0`: Append a copy of the current `word` to itself. For example, `"a"` becomes `"aa"`, and `"ab"` becomes `"abab"`.

- If `operations[i] == 1`: Generate a new string by changing each character in `word` to its next character in the English alphabet, then append this new string to the original `word`. For example, `"c"` becomes `"cd"` (c + next(c)), and `"zb"` becomes `"zbac"` (zb + next(zb) where next(z)=a, next(b)=c).

The character `'z'` wraps around to `'a'` when applying the second type of operation.

After performing all operations in order, return the character at position `k` (1-indexed) in the final string.

**Example walkthrough:**
- Start with `word = "a"`
- If `operations = [0, 1]` and `k = 3`:
  - After operation 0 (copy): `"a"` → `"aa"`
  - After operation 1 (shift and append): `"aa"` → `"aabc"` (aa + shift(aa) where shift(a)=b, shift(a)=b)
  - The 3rd character is `'b'`

## Intuition

The key observation is that we don't need to build the entire string - we only need to find the character at position `k`. Since each operation doubles the string length, after `i` operations, the string has length `2^i`.

Think of the string growth as a binary tree structure. Each operation creates two halves: the left half (original) and the right half (newly appended). For operation type 0, both halves are identical. For operation type 1, the right half is the left half with each character shifted by 1.

Instead of building forward (which could create an exponentially large string), we can work backward from position `k`. We first determine how many operations are needed to create a string of length at least `k`. This gives us the final string length `n`.

Now we trace back through the operations in reverse. At each step, we ask: "Is position `k` in the left half or right half of the current string?"

- If `k > n/2` (right half): Position `k` came from position `k - n/2` in the left half. If this was a type 1 operation, the character was shifted, so we track this shift with a counter `d`.
- If `k ≤ n/2` (left half): Position `k` stays the same, unaffected by this operation.

We continue halving `n` and adjusting `k` until we reach the original string `"a"`. The accumulated shifts in `d` tell us how many times the character 'a' was shifted forward in the alphabet.

The final answer is `chr((d % 26) + ord('a'))`, where we use modulo 26 to handle the wraparound from 'z' to 'a'.

This approach is efficient because we only need `O(log k)` operations instead of actually building a string of length `2^i`.

**Learn more about [Recursion](/problems/recursion_intro) and [Math](/problems/math-basics) patterns.**

## Solution Approach

The implementation follows the backtracking strategy described in the intuition:

**Step 1: Find the required string length**
```python
n, i = 1, 0
while n < k:
    n *= 2
    i += 1
```
We start with `n = 1` (initial string "a") and keep doubling it until `n >= k`. The variable `i` tracks how many operations we need to perform to reach this length.

**Step 2: Initialize shift counter**
```python
d = 0
```
This counter tracks the total number of character shifts accumulated as we trace back through the operations.

**Step 3: Backtrack through operations**
```python
while n > 1:
    if k > n // 2:
        k -= n // 2
        d += operations[i - 1]
    n //= 2
    i -= 1
```

For each operation (working backward):
- Check if `k` is in the right half (`k > n // 2`)
- If yes:
  - Update `k` to its corresponding position in the left half: `k -= n // 2`
  - If `operations[i - 1] == 1`, add 1 to the shift counter `d` (characters in the right half were shifted)
- Halve the string length `n` and decrement the operation index `i`
- Continue until we reach the original string (`n = 1`)

**Step 4: Calculate the final character**
```python
return chr(d % 26 + ord("a"))
```
- Take `d % 26` to handle wraparound (since there are 26 letters)
- Add to the ASCII value of 'a' to get the final character
- Convert back to a character using `chr()`

**Example walkthrough:**
If `k = 5` and `operations = [0, 1, 0]`:
1. After 3 operations, `n = 8` (string length is 8)
2. Backtrack:
   - `k = 5 > 4`: In right half, `operations[2] = 0`, so no shift. Update `k = 1`
   - `k = 1 ≤ 2`: In left half, no change
   - `k = 1 ≤ 1`: In left half, no change
3. Final: `d = 0`, so return 'a'

The time complexity is `O(log k)` for finding the position, and space complexity is `O(1)`.

## Example Walkthrough

Let's trace through a concrete example with `operations = [0, 1, 0]` and `k = 6`.

**Building the string conceptually (to understand the problem):**
- Start: `word = "a"` (length 1)
- After operation 0 (copy): `"aa"` (length 2)
- After operation 1 (shift and append): `"aabc"` (length 4)
  - Original: `"aa"`
  - Shifted: `"bc"` (a→b, a→b)
  - Combined: `"aabc"`
- After operation 0 (copy): `"aabcaabc"` (length 8)

We want the 6th character, which is 'a'.

**Using our backtracking solution:**

**Step 1:** Find required length
- Start with `n = 1, i = 0`
- `n = 1 < 6`, so `n = 2, i = 1`
- `n = 2 < 6`, so `n = 4, i = 2`
- `n = 4 < 6`, so `n = 8, i = 3`
- Now `n = 8 >= 6`, stop

**Step 2:** Initialize shift counter `d = 0`

**Step 3:** Backtrack through operations
- **Iteration 1:** `n = 8, k = 6, i = 3`
  - Is `k = 6 > n/2 = 4`? Yes, in right half
  - Update `k = 6 - 4 = 2`
  - `operations[i-1] = operations[2] = 0`, so `d = 0 + 0 = 0`
  - Update `n = 4, i = 2`

- **Iteration 2:** `n = 4, k = 2, i = 2`
  - Is `k = 2 > n/2 = 2`? No, in left half
  - No changes to `k` or `d`
  - Update `n = 2, i = 1`

- **Iteration 3:** `n = 2, k = 2, i = 1`
  - Is `k = 2 > n/2 = 1`? Yes, in right half
  - Update `k = 2 - 1 = 1`
  - `operations[i-1] = operations[0] = 0`, so `d = 0 + 0 = 0`
  - Update `n = 1, i = 0`

**Step 4:** Calculate final character
- `d = 0`, so `d % 26 = 0`
- `chr(0 + ord('a')) = chr(97) = 'a'`

The answer is 'a', which matches our expectation from the conceptual build!

## Python Solution

```python
class Solution:
    def kthCharacter(self, k: int, operations: List[int]) -> str:
        # Find the smallest power of 2 that is >= k
        # This determines how many operations we need to consider
        length = 1
        operation_index = 0
        while length < k:
            length *= 2
            operation_index += 1
      
        # Track the total shift amount for the character
        total_shift = 0
      
        # Work backwards through the operations to find which character
        # at position k comes from and accumulate shifts
        while length > 1:
            # If k is in the second half of current segment
            if k > length // 2:
                # Move k to corresponding position in first half
                k -= length // 2
                # Add the operation's shift value
                total_shift += operations[operation_index - 1]
          
            # Move to previous operation level
            length //= 2
            operation_index -= 1
      
        # Convert the accumulated shift to the final character
        # Starting from 'a' and wrapping around after 'z'
        return chr(total_shift % 26 + ord('a'))
```

## Time and Space Complexity

The time complexity is `O(log k)`. The first while loop runs until `n >= k`, where `n` starts at 1 and doubles each iteration, so it takes `O(log k)` iterations to reach a value greater than or equal to `k`. The second while loop runs for the same number of iterations as the first loop (from `i` down to 0), performing constant-time operations in each iteration. Therefore, the overall time complexity is `O(log k) + O(log k) = O(log k)`.

The space complexity is `O(1)`. The algorithm only uses a constant amount of extra space for variables `n`, `i`, and `d`, regardless of the input size `k`. No additional data structures that grow with the input are created, making the space complexity constant.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Off-by-One Error in Operation Index**
A common mistake is misaligning the operation index when backtracking. When we determine we need `i` operations to reach length `n`, the operations are indexed from `0` to `i-1` in the array.

**Incorrect:**
```python
# Wrong: Using operations[i] instead of operations[i-1]
total_shift += operations[i]
```

**Correct:**
```python
# Right: The i-th operation is at index i-1
total_shift += operations[operation_index - 1]
```

### 2. **Incorrect Position Mapping in Backtracking**
When determining if position `k` is in the right half and mapping it back to the left half, using the wrong comparison or calculation can lead to incorrect results.

**Incorrect:**
```python
# Wrong: Using >= instead of >
if k >= length // 2:
    k -= length // 2
```
This would incorrectly treat position `length//2` as being in the right half when it's actually the last position of the left half.

**Correct:**
```python
# Right: Positions 1 to length//2 are left half, length//2+1 to length are right half
if k > length // 2:
    k -= length // 2
```

### 3. **Not Handling Character Wraparound Properly**
Forgetting to use modulo 26 when calculating the final character can cause issues when the total shift exceeds 25.

**Incorrect:**
```python
# Wrong: This will crash or give wrong character if total_shift >= 26
return chr(total_shift + ord('a'))
```

**Correct:**
```python
# Right: Use modulo to handle wraparound from 'z' to 'a'
return chr(total_shift % 26 + ord('a'))
```

### 4. **Building the Actual String**
A naive approach would be to actually build the entire string by performing all operations, which is inefficient and may cause memory issues for large k values.

**Inefficient:**
```python
word = "a"
for op in operations:
    if op == 0:
        word = word + word
    else:
        word = word + ''.join(chr((ord(c) - ord('a') + 1) % 26 + ord('a')) for c in word)
return word[k-1]
```

**Efficient:**
Use the backtracking approach without building the string, as shown in the solution.

