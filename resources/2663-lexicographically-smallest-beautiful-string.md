# 2663. Lexicographically Smallest Beautiful String

> Source: https://algo.monster/liteproblems/2663

## Problem Statement

You are given a string `s` of length `n` that is "beautiful" and a positive integer `k`. A beautiful string has two properties:

1. It only uses the first `k` letters of the English lowercase alphabet (a, b, c, ...).
2. It contains no palindromic substrings of length 2 or more.

The second condition means:
- No two adjacent characters can be the same (this would form a palindrome of length 2)
- No character can be the same as the character two positions before it (this would form a palindrome of length 3)

Your task is to find the lexicographically smallest string that:
- Has the same length `n` as the input string `s`
- Is lexicographically larger than `s`
- Is also a beautiful string (satisfies both conditions above)

If no such string exists, return an empty string.

For example, if `s = "abcz"` and `k = 26`, you need to find the next beautiful string after "abcz" in lexicographic order. The string must still only use the first 26 letters (a-z) and avoid any palindromic substrings.

## Intuition

To find the next lexicographically larger beautiful string, we need to think about how we normally find the next lexicographic string - we increment from the rightmost position possible. However, the beauty constraint adds complexity: we can't use just any character, as it might create a palindrome.

The key insight is understanding what makes a string "not beautiful":
- If two adjacent characters are the same, we get a palindrome like "aa"
- If two characters with one character between them are the same, we get a palindrome like "aba"

This means each character must be different from its previous two neighbors. So at any position, we have at most `k-2` valid choices (excluding the two previous characters).

Why search from right to left? Consider how we increment numbers: to get the next number after 1299, we can't change the rightmost 9s to anything valid (they're already at maximum), so we move left until we find a digit we can increment (the 2 becomes 3, giving us 1300). Similarly, we need to find the rightmost position where we can place a larger character while maintaining the beauty property.

Once we find such a position `i` where we can place a larger valid character:
1. We update position `i` with the smallest valid character that's larger than the current one
2. For all positions after `i`, we greedily fill them with the smallest possible valid characters (starting from 'a', 'b', etc., skipping any that would create palindromes)

This [greedy](/problems/greedy_intro) approach after position `i` ensures we get the lexicographically smallest result among all valid beautiful strings larger than `s`.

If we traverse the entire string without finding any position where we can increment (meaning the string is already at its "maximum" beautiful form for the given length), we return an empty string.

**Learn more about [Greedy](/problems/greedy_intro) patterns.**

## Solution Approach

The implementation follows a [greedy](/problems/greedy_intro) backtracking strategy with two main phases: finding the increment position and filling the remaining positions.

**Phase 1: Finding the Position to Increment**

We iterate backwards from the last character (index `n-1`) to the first character (index `0`):
- For each position `i`, we get the current character's value: `p = ord(cs[i]) - ord('a') + 1`
- We try to replace it with a larger character from range `[p, k)` (characters from the next one up to the `k`-th letter)
- For each candidate character `c`, we check if it violates the beauty constraint:
  - If `i > 0` and `cs[i-1] == c`: skip (would create adjacent duplicates)
  - If `i > 1` and `cs[i-2] == c`: skip (would create a palindrome of length 3)
- If we find a valid character, we assign `cs[i] = c` and move to Phase 2

**Phase 2: Filling Remaining Positions**

Once we've successfully incremented position `i`, we need to fill positions `[i+1, n-1]` with the lexicographically smallest valid characters:
- For each position `l` from `i+1` to `n-1`:
  - Try characters from `'a'` to the `(k-1)`-th letter in order (range `[0, k)`)
  - For each candidate character `c`:
    - Check if `l > 0` and `cs[l-1] == c`: skip if true
    - Check if `l > 1` and `cs[l-2] == c`: skip if true
    - If valid, assign `cs[l] = c` and break to move to the next position
- After filling all positions, return the joined string `''.join(cs)`

**Edge Case:**

If we complete the backward iteration without finding any valid position to increment (the outer loop ends without returning), it means no larger beautiful string exists, so we return an empty string `''`.

The time complexity is `O(n * k^2)` in the worst case, where we check all positions, try all possible characters at each position, and for each successful increment, fill the remaining positions. The space complexity is `O(n)` for storing the character list.

## Example Walkthrough

Let's walk through finding the next beautiful string after `s = "abcb"` with `k = 4` (using letters a, b, c, d).

**Initial Setup:**
- Input string: `"abcb"`
- We convert it to a list: `['a', 'b', 'c', 'b']`
- We need to find the next lexicographically larger beautiful string

**Phase 1: Finding the Position to Increment**

Starting from the rightmost position (index 3):

*Position 3 (character 'b'):*
- Current value: `p = ord('b') - ord('a') + 1 = 2`
- Try characters from position 2 to k-1: ['c', 'd']
- Check 'c': 
  - Is `cs[2] == 'c'`? Yes! This would create adjacent duplicates "cc"
  - Skip 'c'
- Check 'd':
  - Is `cs[2] == 'd'`? No ✓
  - Is `cs[1] == 'd'`? No ✓
  - Valid! Set `cs[3] = 'd'`
  - String becomes: `['a', 'b', 'c', 'd']`

Since we successfully incremented position 3 and it's the last position, we're done!

**Result:** `"abcd"`

---

Let's try a more complex example: `s = "abcd"` with `k = 4`.

**Phase 1: Finding the Position to Increment**

*Position 3 (character 'd'):*
- Current value: `p = 4`
- Try characters from position 4 to k-1: No characters available (d is already the 4th letter)
- Can't increment here, move left

*Position 2 (character 'c'):*
- Current value: `p = 3`
- Try character 'd':
  - Is `cs[1] == 'd'`? No ✓
  - Is `cs[0] == 'd'`? No ✓
  - Valid! Set `cs[2] = 'd'`
  - String becomes: `['a', 'b', 'd', 'd']`

**Phase 2: Filling Remaining Positions**

*Position 3:*
- Try 'a':
  - Is `cs[2] == 'a'`? No (it's 'd') ✓
  - Is `cs[1] == 'a'`? No (it's 'b') ✓
  - Valid! Set `cs[3] = 'a'`
  - String becomes: `['a', 'b', 'd', 'a']`

**Result:** `"abda"`

This demonstrates how the algorithm:
1. Searches right-to-left for a position to increment
2. Checks palindrome constraints when trying new characters
3. Fills remaining positions with the smallest valid characters

## Python Solution

```python
class Solution:
    def smallestBeautifulString(self, s: str, k: int) -> str:
        """
        Find the lexicographically smallest beautiful string greater than s.
        A beautiful string has no palindromic substring of length 2 or 3.
      
        Args:
            s: Input string containing only lowercase letters
            k: Size of the alphabet (first k letters from 'a')
      
        Returns:
            The smallest beautiful string greater than s, or empty string if none exists
        """
        n = len(s)
        char_list = list(s)
      
        # Try to increment the string from right to left
        for i in range(n - 1, -1, -1):
            # Get the next character value after current character
            current_char_value = ord(char_list[i]) - ord('a') + 1
          
            # Try each possible character from current+1 to k-1
            for next_value in range(current_char_value, k):
                next_char = chr(ord('a') + next_value)
              
                # Check if this character would create a palindrome of length 2 or 3
                # Skip if it matches the previous character (palindrome of length 2)
                if i > 0 and char_list[i - 1] == next_char:
                    continue
                # Skip if it matches the character two positions back (palindrome of length 3)
                if i > 1 and char_list[i - 2] == next_char:
                    continue
              
                # Valid character found, update it
                char_list[i] = next_char
              
                # Fill the remaining positions with the smallest valid characters
                for pos in range(i + 1, n):
                    # Try each character starting from 'a'
                    for char_value in range(k):
                        candidate_char = chr(ord('a') + char_value)
                      
                        # Check for palindrome constraints
                        # Skip if it matches the previous character
                        if pos > 0 and char_list[pos - 1] == candidate_char:
                            continue
                        # Skip if it matches the character two positions back
                        if pos > 1 and char_list[pos - 2] == candidate_char:
                            continue
                      
                        # Valid character found for this position
                        char_list[pos] = candidate_char
                        break
              
                # Successfully constructed a valid beautiful string
                return ''.join(char_list)
      
        # No valid beautiful string found
        return ''
```

## Time and Space Complexity

The time complexity is `O(n × k)`, where `n` is the length of the string and `k` is the size of the alphabet (parameter k). 

The algorithm iterates through the string from right to left (outer loop: `O(n)`). For each position `i`, it tries up to `k` different characters (loop with range `(p, k)`). When a valid character is found, it fills the remaining positions from `i+1` to `n-1` (inner loop: `O(n)`), and for each position, it tries up to `k` characters to find the first valid one.

In the worst case, the algorithm needs to:
- Try multiple positions from the end: `O(n)`
- For each position, try multiple characters: `O(k)`  
- Fill the suffix after finding a valid character: `O(n × k)`

However, the reference answer states `O(n)`. This suggests that despite the nested loops, the amortized complexity is `O(n)` because:
1. The algorithm returns immediately after finding the first valid solution
2. Each position is modified at most once in a successful path
3. The constant `k` is bounded (at most 26 for lowercase letters)

Therefore, treating `k` as a constant, the time complexity simplifies to `O(n)`.

The space complexity is `O(n)` for creating the list `cs` from the input string. If we consider only auxiliary space (excluding the space needed for the output), it would be `O(1)` as stated in the reference answer, since we only use a few variables and modify the list in-place.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Off-by-One Error in Character Range**

**Pitfall:** A common mistake is incorrectly handling the character range when trying to increment. The code `current_char_value = ord(char_list[i]) - ord('a') + 1` correctly gets the next value after the current character, but developers often forget the `+1` or mistakenly use `ord(char_list[i]) - ord('a')`, which would include the current character itself in the range.

**Why it's problematic:** If you include the current character in the range, you might end up with the same string instead of finding the next lexicographically larger one.

**Solution:** Always ensure you're starting from `current_char_value + 1` when looking for the next valid character:
```python
# Correct: starts from the character after current
current_char_value = ord(char_list[i]) - ord('a') + 1

# Incorrect: would include current character
# current_char_value = ord(char_list[i]) - ord('a')
```

### 2. **Not Checking Bounds Before Accessing Previous Characters**

**Pitfall:** Forgetting to check if `i > 0` before accessing `char_list[i-1]` or if `i > 1` before accessing `char_list[i-2]` can lead to negative indexing, which in Python doesn't cause an error but gives unexpected behavior by wrapping around to the end of the list.

**Why it's problematic:** Without proper bounds checking, `char_list[-1]` would access the last character of the string when you meant to skip the check entirely for the first character.

**Solution:** Always include boundary checks before accessing previous positions:
```python
# Correct approach with bounds checking
if i > 0 and char_list[i - 1] == next_char:
    continue
if i > 1 and char_list[i - 2] == next_char:
    continue

# Incorrect: missing bounds check
# if char_list[i - 1] == next_char:  # Dangerous when i == 0
```

### 3. **Forgetting to Fill Remaining Positions After Increment**

**Pitfall:** After successfully incrementing a character at position `i`, some implementations forget to properly fill positions `[i+1, n-1]` with valid characters, or they might leave them unchanged from the original string.

**Why it's problematic:** The remaining characters from the original string might not be the lexicographically smallest valid option, or worse, they might create palindromes with the newly changed character.

**Solution:** Always reset and fill all positions after the increment point with the smallest valid characters:
```python
# After setting char_list[i] = next_char
for pos in range(i + 1, n):
    # Must find the smallest valid character for each position
    for char_value in range(k):
        candidate_char = chr(ord('a') + char_value)
        # Check palindrome constraints and assign
```

### 4. **Incorrect Palindrome Detection Logic**

**Pitfall:** Only checking for adjacent duplicates (length-2 palindromes) but forgetting about length-3 palindromes like "aba", or vice versa.

**Why it's problematic:** A beautiful string must avoid ALL palindromic substrings of length 2 or more. Missing either check would produce invalid results.

**Solution:** Always check both conditions:
- No character should equal its immediate predecessor (avoids "aa")
- No character should equal the character two positions before it (avoids "aba")

Both checks are essential and neither can be omitted.

