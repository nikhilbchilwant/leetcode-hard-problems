# 761. Special Binary String

> Source: https://algo.monster/liteproblems/761

## Problem Statement

You're given a special type of binary string that has two key properties:
1. It contains an equal number of `0`s and `1`s
2. At any point while reading the string from left to right, you'll always have seen at least as many `1`s as `0`s (every prefix has more or equal `1`s than `0`s)

For example, `"1100"` is special because:
- It has 2 ones and 2 zeros (equal count)
- Reading left to right: `"1"` has 1 one, 0 zeros ✓; `"11"` has 2 ones, 0 zeros ✓; `"110"` has 2 ones, 1 zero ✓; `"1100"` has 2 ones, 2 zeros ✓

Your task is to rearrange the given special binary string `s` to make it lexicographically largest possible. You can do this through a series of moves where each move involves:
- Finding two special substrings that are next to each other (consecutive)
- Swapping their positions

Two substrings are consecutive if they're adjacent in the string with no gap between them. Both substrings being swapped must themselves be special (following the same two properties).

The goal is to return the largest possible string in lexicographical order after performing any number of these swapping operations. In lexicographical order for binary strings, strings starting with `1` come after strings starting with `0`, and longer sequences of `1`s at the beginning are considered larger.

For instance, if you have a string like `"11011000"`, you might be able to rearrange its special substrings to get something like `"11100100"` which would be lexicographically larger.

## Intuition

The key insight is recognizing that special binary strings have a recursive structure similar to balanced parentheses. We can think of `1` as an opening parenthesis and `0` as a closing parenthesis. 

Every special binary string can be viewed as either:
1. A concatenation of multiple special binary strings, or
2. A string of the form `"1" + inner + "0"` where `inner` is itself a (possibly empty) special binary string

This recursive structure suggests we can decompose the problem. When we encounter a complete special substring (where the count of `1`s equals the count of `0`s), we've found a "unit" that can be treated independently.

To maximize the lexicographical value, we need to arrange these independent special substrings in descending order. Why? Because in binary, having `1`s appear earlier makes the string larger. If we have special substrings like `"1100"` and `"111000"`, placing `"111000"` first gives us `"1110001100"` which is larger than `"1100111000"`.

The algorithm tracks a running count: add 1 for each `'1'` and subtract 1 for each `'0'`. When this count returns to 0, we've identified a complete special substring. For each such substring, we recursively process its interior (the part between the first `1` and last `0`) to maximize it, then wrap it back with `"1"` and `"0"`.

After finding all special substrings at the current level, we sort them in descending order to get the lexicographically largest arrangement. This greedy approach works because special substrings are independent units that can be freely rearranged without breaking the special property of the overall string.

The beauty of this solution is that it naturally handles nested structures - inner special substrings are optimized recursively before the outer level arranges them optimally.

**Learn more about [Recursion](/problems/recursion_intro) patterns.**

## Solution Approach

The implementation uses a recursive approach with string manipulation and sorting to solve the problem.

**Algorithm Steps:**

1. **Base Case**: If the string is empty, return an empty string.

2. **Initialize Variables**:
   - `ans` - list to store all special substrings found at the current level
   - `cnt` - counter to track the balance between `1`s and `0`s
   - `i` - current position pointer
   - `j` - start position of the current special substring

3. **Iterate Through the String**:
   - For each character at position `i`:
     - If it's `'1'`, increment `cnt` by 1
     - If it's `'0'`, decrement `cnt` by 1
   - When `cnt` becomes 0, we've found a complete special substring from position `j` to `i`

4. **Process Each Special Substring**:
   - Extract the inner part: `s[j+1 : i]` (excluding the outer `1` and `0`)
   - Recursively call `makeLargestSpecial` on the inner part to maximize it
   - Reconstruct the special substring: `'1' + maximized_inner + '0'`
   - Add this to the `ans` list
   - Update `j = i + 1` to start looking for the next special substring

5. **Sort and Combine**:
   - Sort all special substrings in `ans` in descending order (reverse=True)
   - Join them together to form the final result

**Example Walkthrough:**

For input `s = "11011000"`:
- The balance goes 1, 2, 1, 2, 3, 2, 1, 0, so it first returns to 0 at index 7: the whole string is one special substring
  - Inner part is `"101100"`, recursively processed
  - Inside `"101100"`, the balance returns to 0 at index 1 (`"10"`) and at index 5 (`"1100"`, whose inner `"10"` stays `"10"`)
  - Sort `["10", "1100"]` in descending order → `["1100", "10"]`, giving `"110010"`
- Reconstructed as `"1" + "110010" + "0" = "11100100"`
- Final result: `"11100100"`

The time complexity is `O(n²)` in the worst case due to recursive calls and sorting, where `n` is the length of the string. The space complexity is `O(n)` for the [recursion](/problems/recursion_intro) stack and temporary storage.

## Example Walkthrough

Let's walk through the solution with the input string `s = "11011000"`.

**Step 1: Initial Setup**
- Start with `balance = 0`, `start = 0`, `substrings = []`
- We'll track the balance of 1s and 0s as we traverse

**Step 2: First Pass - Finding Special Substrings**

Position 0: `'1'` → `balance = 1`
Position 1: `'1'` → `balance = 2`
Position 2: `'0'` → `balance = 1`
Position 3: `'1'` → `balance = 2`
Position 4: `'1'` → `balance = 3`
Position 5: `'0'` → `balance = 2`
Position 6: `'0'` → `balance = 1`
Position 7: `'0'` → `balance = 0` ✓ Found special substring!

- The balance never reached 0 earlier, so the whole string `"11011000"` (from `start=0` to `i=7`) is one special substring
- Inner part (excluding outer 1 and 0): `"101100"`

**Step 3: Recursively Process `"101100"`**

Position 0: `'1'` → `balance = 1`
Position 1: `'0'` → `balance = 0` ✓ Found `"10"` (inner part is empty, returns `""`), reconstructed as `"10"`
Position 2: `'1'` → `balance = 1`
Position 3: `'1'` → `balance = 2`
Position 4: `'0'` → `balance = 1`
Position 5: `'0'` → `balance = 0` ✓ Found `"1100"`

- Inner part of `"1100"` is `"10"`, which returns `"10"`, so it is reconstructed as `"1100"`
- Substrings at this level: `["10", "1100"]`
- Sort in descending order: `["1100", "10"]`
- Join: `"110010"`

**Step 4: Reconstruct the Outer Level**
- Wrap the optimized inner part: `"1" + "110010" + "0" = "11100100"`
- The outer level has only one substring, so sorting leaves it unchanged

**Final Result**: `"11100100"`

This matches the result of swapping the consecutive special substrings `"10"` and `"1100"` inside the original string.

Note: This example shows how the algorithm identifies special substrings when the count returns to 0, recursively optimizes their inner parts, and then arranges them in descending order to achieve the lexicographically largest result.

## Python Solution

```python
class Solution:
    def makeLargestSpecial(self, s: str) -> str:
        """
        Rearranges a special binary string to get the lexicographically largest result.
        A special string starts with '1', ends with '0', and has equal number of '1's and '0's
        with every prefix having at least as many '1's as '0's.
        """
        # Base case: empty string returns empty
        if not s:
            return ''
      
        # List to store special substrings found at the current level
        substrings = []
      
        # Counter to track balance of '1's and '0's
        balance = 0
      
        # Start index of current special substring
        start = 0
      
        # Iterate through the string to find special substrings
        for i in range(len(s)):
            # Increment balance for '1', decrement for '0'
            balance += 1 if s[i] == '1' else -1
          
            # When balance returns to 0, we've found a complete special substring
            if balance == 0:
                # Recursively process the inner content (excluding first '1' and last '0')
                # Then wrap it with '1' at start and '0' at end
                inner_content = self.makeLargestSpecial(s[start + 1:i])
                special_substring = '1' + inner_content + '0'
                substrings.append(special_substring)
              
                # Move start pointer to begin searching for next special substring
                start = i + 1
      
        # Sort all special substrings in descending order (lexicographically largest first)
        substrings.sort(reverse=True)
      
        # Concatenate all sorted substrings to form the final result
        return ''.join(substrings)
```

## Time and Space Complexity

**Time Complexity:** `O(n^2)` where `n` is the length of the input string.

The analysis breaks down as follows:
- The main function traverses the string once with the `for` loop: `O(n)`
- For each balanced substring found (when `balance == 0`), we make a recursive call on the substring between the first '1' and last '0'
- In the worst case, the recursion depth can be `O(n)` (for deeply nested special strings like "111...000...")
- At each recursion level, we perform string slicing `s[start + 1:i]` which takes `O(n)` time
- The sorting operation `substrings.sort(reverse=True)` takes `O(k log k)` where `k` is the number of substrings at that level, but `k ≤ n`
- Overall, considering the recursive calls and string operations at each level: `O(n) * O(n) = O(n^2)`

**Space Complexity:** `O(n^2)`

The space analysis includes:
- Recursion call stack depth: `O(n)` in the worst case
- At each recursion level, we create new substrings through slicing and store them in the `substrings` list
- String slicing creates new strings, and the total space for all substrings across all recursive calls can be `O(n^2)` in the worst case
- The accumulated space for storing intermediate results and substrings: `O(n^2)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Misunderstanding What Constitutes a "Special Substring"**

A common mistake is thinking that any substring with equal `1`s and `0`s is special. However, a special substring must satisfy BOTH conditions:
- Equal number of `1`s and `0`s
- Every prefix has at least as many `1`s as `0`s

**Incorrect assumption:** Treating `"0110"` as special (it has equal counts but fails the prefix condition - the first character is `0`).

**Solution:** The algorithm correctly identifies special substrings by tracking the balance and only considering a substring complete when balance returns to 0 after starting from a positive value (since special strings must start with `1`).

### 2. **Incorrectly Processing the Inner Content**

When finding a special substring like `"110100"`, developers might forget to:
- Remove the outer `1` and `0` before recursive processing
- Add them back after processing

**Common mistake:**
```python
# Wrong: Processing the entire substring recursively
special_substring = self.makeLargestSpecial(s[start:i+1])
```

**Correct approach:**
```python
# Right: Process only the inner content, then wrap with '1' and '0'
inner_content = self.makeLargestSpecial(s[start + 1:i])
special_substring = '1' + inner_content + '0'
```

### 3. **Forgetting to Handle Multiple Special Substrings at the Same Level**

The string might contain multiple consecutive special substrings at the root level. For example, `"11001100"` contains two special substrings: `"1100"` and `"1100"`.

**Pitfall:** Assuming the entire string is always one special substring and returning after finding the first one.

**Solution:** The algorithm correctly continues searching for more special substrings after finding one by updating `start = i + 1` and continuing the loop.

### 4. **Incorrect Sorting Direction**

Since we want the lexicographically largest result, substrings should be sorted in descending order.

**Common mistake:**
```python
# Wrong: This would give the smallest result
substrings.sort()  # Default is ascending order
```

**Correct approach:**
```python
# Right: Sort in descending order for largest result
substrings.sort(reverse=True)
```

### 5. **Off-by-One Errors in Index Handling**

When extracting the inner content, it's easy to make indexing errors:

**Common mistakes:**
- Using `s[start:i+1]` instead of `s[start+1:i]` for inner content
- Forgetting to update `start = i + 1` (not just `start = i`)

These errors would either include the outer `1` and `0` in the recursive call or miss characters when searching for the next special substring.

