# 564. Find the Closest Palindrome

> Source: https://algo.monster/liteproblems/564

## Problem Statement

Given a string `n` that represents an integer, you need to find the closest palindrome number to this integer. The returned palindrome should not be the number itself.

A palindrome is a number that reads the same forwards and backwards (like `121`, `1331`, or `9`).

The "closest" palindrome means the one with the minimum absolute difference from the original number. If there are two palindromes with the same distance from the original number (a tie), you should return the smaller one.

For example:
- If `n = "123"`, the closest palindrome is `"121"` (distance of 2)
- If `n = "1"`, there are two palindromes at the same distance: `"0"` (distance of 1) and `"2"` (distance of 1). Since there's a tie, return `"0"` as it's smaller

The key challenge is handling various edge cases:
- Numbers like `99`, `999` where the closest palindrome might be `101`, `1001` (crossing digit boundaries)
- Numbers that are already palindromes (you need to find the next closest one)
- Single digit numbers
- Numbers where multiple palindromes are equidistant from the input

The solution generates candidate palindromes by:
1. Creating palindromes one digit shorter (`10^(l-1) - 1`, like `99` for a 3-digit number)
2. Creating palindromes one digit longer (`10^l + 1`, like `1001` for a 3-digit number)  
3. Creating palindromes by mirroring the left half with small adjustments (-1, 0, +1 to handle cases where the middle digits need to change)

Then it selects the candidate with minimum distance from the original number, choosing the smaller value in case of ties.

## Intuition

When looking for the nearest palindrome, we need to think about where palindromes can exist relative to our number. The key insight is that we don't need to check every possible number - palindromes have a special structure that limits our search space.

First, consider what makes a palindrome: the left half mirrors to create the right half. So for a number like `12345`, potential nearby palindromes could be formed by taking prefixes like `123` and mirroring them to get `12321`.

But simply mirroring the left half might give us the original number itself (if it's already a palindrome) or might not give us the closest one. For instance, if we have `12399`, mirroring `123` gives us `12321`, but `12421` (formed by incrementing the middle and mirroring) might be closer.

This leads us to consider three variations of mirroring:
- Mirror the left half as-is
- Decrement the left half by 1, then mirror it
- Increment the left half by 1, then mirror it

These three candidates handle most cases, but there are special edge cases at digit boundaries. Consider `999` - the nearest palindromes are `1001` (going up) and `989` (going down). The number `1001` represents crossing into 4-digit palindromes, while `989` stays within 3 digits.

This observation reveals that we also need to consider:
- The largest palindrome with one fewer digit: `10^(l-1) - 1` (like `99` for a 3-digit input)
- The smallest palindrome with one more digit: `10^l + 1` (like `1001` for a 3-digit input)

By generating these 5 candidates (3 from mirroring variations + 2 from boundary cases), we guarantee that one of them will be the nearest palindrome. We then simply compare distances and pick the closest one, using the smaller value to break ties.

The elegance of this approach is that it reduces an potentially infinite search space to just 5 candidates, using the mathematical structure of palindromes to ensure we don't miss the optimal answer.

**Learn more about [Math](/problems/math-basics) patterns.**

## Solution Approach

The implementation follows our intuition by systematically generating the candidate palindromes and selecting the best one.

**Step 1: Initialize candidate set with boundary palindromes**
```python
res = {10 ** (l - 1) - 1, 10**l + 1}
```
We start by adding the two boundary cases:
- `10^(l-1) - 1`: The largest palindrome with one fewer digit (e.g., `99` for a 3-digit number)
- `10^l + 1`: The smallest palindrome with one more digit (e.g., `1001` for a 3-digit number)

**Step 2: Extract the left half of the number**
```python
left = int(n[: (l + 1) >> 1])
```
We take the first `(l+1)//2` digits. The bit shift `(l + 1) >> 1` is equivalent to `(l + 1) // 2`. For odd-length numbers, this includes the middle digit; for even-length numbers, it's exactly half.

**Step 3: Generate palindromes by mirroring variations**
```python
for i in range(left - 1, left + 2):
    j = i if l % 2 == 0 else i // 10
    while j:
        i = i * 10 + j % 10
        j //= 10
    res.add(i)
```
For each variation (`left - 1`, `left`, `left + 1`):
- If the original length is even, we mirror the entire left part
- If the original length is odd, we skip the middle digit by dividing by 10 first (`i // 10`)
- The while loop performs the mirroring by extracting digits from `j` and appending them to `i`

For example, with `12345` (odd length, left = `123`):
- When `i = 123`, `j = 12` (removing middle), we build `123` → `1232` → `12321`
- When `i = 122`, `j = 12`, we build `122` → `1222` → `12221`
- When `i = 124`, `j = 12`, we build `124` → `1242` → `12421`

**Step 4: Remove the original number and find the closest**
```python
res.discard(x)  # Remove the original number itself

ans = -1
for t in res:
    if (
        ans == -1
        or abs(t - x) < abs(ans - x)
        or (abs(t - x) == abs(ans - x) and t < ans)
    ):
        ans = t
```
We iterate through all candidates and select the one that:
1. Has the minimum absolute difference from the original number
2. In case of a tie (same distance), choose the smaller number

The use of a set for `res` automatically handles duplicate candidates, and the final loop ensures we pick the optimal palindrome according to the problem's requirements.

Only 5 candidates are checked, and building each one takes time proportional to the number of digits, so the running time is `O(l)` for an input of `l` digits, with `O(l)` space for the candidate numbers.

## Example Walkthrough

Let's walk through the solution with `n = "123"`:

**Step 1: Initialize boundary candidates**
- Length `l = 3`
- Add `10^(3-1) - 1 = 99` (largest 2-digit palindrome)
- Add `10^3 + 1 = 1001` (smallest 4-digit palindrome)
- Candidates: `{99, 1001}`

**Step 2: Extract left half**
- Left half length: `(3+1)//2 = 2`
- Left = `12` (first 2 digits of "123")

**Step 3: Generate mirrored palindromes**
For each variation of left half:

*Variation 1: left - 1 = 11*
- Since length is odd, `j = 11 // 10 = 1`
- Build palindrome: `11` → `11 * 10 + (1 % 10) = 111`
- Add `111` to candidates

*Variation 2: left = 12*
- Since length is odd, `j = 12 // 10 = 1`  
- Build palindrome: `12` → `12 * 10 + (1 % 10) = 121`
- Add `121` to candidates

*Variation 3: left + 1 = 13*
- Since length is odd, `j = 13 // 10 = 1`
- Build palindrome: `13` → `13 * 10 + (1 % 10) = 131`
- Add `131` to candidates

Candidates now: `{99, 111, 121, 131, 1001}`

**Step 4: Remove original and find closest**
- Original number `x = 123`
- Remove 123 from candidates (not present, so no change)
- Calculate distances:
  - `|99 - 123| = 24`
  - `|111 - 123| = 12`
  - `|121 - 123| = 2` ← minimum distance
  - `|131 - 123| = 8`
  - `|1001 - 123| = 878`

**Result: "121"** (closest palindrome with distance of 2)

## Python Solution

```python
class Solution:
    def nearestPalindromic(self, n: str) -> str:
        """
        Find the nearest palindrome to the given number n.
        If there are two palindromes with the same distance, return the smaller one.
        """
        original_num = int(n)
        length = len(n)
      
        # Initialize candidate set with edge cases:
        # 1. Largest palindrome with (length-1) digits: 999...999
        # 2. Smallest palindrome with (length+1) digits: 100...001
        candidates = {10 ** (length - 1) - 1, 10 ** length + 1}
      
        # Extract the left half of the number (including middle digit if odd length)
        prefix_length = (length + 1) // 2
        prefix = int(n[:prefix_length])
      
        # Generate palindromes by mirroring prefix-1, prefix, and prefix+1
        for prefix_variant in range(prefix - 1, prefix + 2):
            # Create palindrome by mirroring the prefix
            palindrome = prefix_variant
          
            # Determine what to mirror (exclude middle digit if odd length)
            suffix_to_mirror = prefix_variant if length % 2 == 0 else prefix_variant // 10
          
            # Mirror the digits to create the full palindrome
            while suffix_to_mirror > 0:
                palindrome = palindrome * 10 + suffix_to_mirror % 10
                suffix_to_mirror //= 10
          
            candidates.add(palindrome)
      
        # Remove the original number from candidates (we need a different palindrome)
        candidates.discard(original_num)
      
        # Find the nearest palindrome from candidates
        nearest_palindrome = -1
        for candidate in candidates:
            distance_to_candidate = abs(candidate - original_num)
            distance_to_current_nearest = abs(nearest_palindrome - original_num)
          
            # Update if this is the first candidate, or if it's closer,
            # or if it's equally close but smaller
            if (nearest_palindrome == -1 or 
                distance_to_candidate < distance_to_current_nearest or
                (distance_to_candidate == distance_to_current_nearest and 
                 candidate < nearest_palindrome)):
                nearest_palindrome = candidate
      
        return str(nearest_palindrome)
```

## Time and Space Complexity

**Time Complexity:** `O(n)` where `n` is the length of the input string.

- Converting string to integer: `O(n)`
- Calculating boundary candidates (`10^(l-1) - 1` and `10^l + 1`): `O(n)` for power operations
- Extracting left half of the string: `O(n)`
- The for loop runs 3 iterations (from `left - 1` to `left + 1`):
  - Inside each iteration, the while loop mirrors the left half to create a palindrome
  - The while loop runs `O(n/2)` times (processing half the digits)
  - Each operation inside the while loop is `O(1)`
  - Total for palindrome generation: `O(n)` per iteration
- Adding elements to set: `O(1)` average case per addition
- Discarding element from set: `O(1)` average case
- Finding the nearest palindrome by iterating through the set (at most 5 elements): `O(1)`
- Converting result to string: `O(n)`

Overall: `O(n) + O(n) + O(n) + 3 * O(n) + O(1) + O(n) = O(n)`

**Space Complexity:** `O(n)`

- The set `res` stores at most 5 candidates: `O(1)` in terms of count, but each number can be up to `n` digits long, requiring `O(n)` space to store
- Variable `left` stores half of the input: `O(n)` space
- Other variables (`x`, `l`, `i`, `j`, `ans`, `t`) are single integers: `O(1)` each, but their values can be up to `n` digits, so `O(n)` space
- String slicing `n[:(l+1)>>1]` creates a temporary string: `O(n)`

Overall: `O(n)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Integer Overflow with Very Large Numbers**
One critical pitfall is that Python's `int` type can handle arbitrarily large numbers, but in other languages like Java or C++, converting very large string numbers to integers can cause overflow issues.

**Problem Example:**
```python
# This works in Python but would overflow in languages with fixed integer sizes
n = "9" * 100  # A 100-digit number
original_num = int(n)  # This could overflow in other languages
```

**Solution:**
For extremely large numbers or when porting to other languages, consider using string comparison instead of integer conversion for the final selection:

```python
def compare_distance_using_strings(n: str, candidate: str) -> int:
    # Implement string-based subtraction to avoid overflow
    # This is more complex but necessary for very large numbers
    pass
```

### 2. **Assuming Single-Digit Inputs Need Special Handling**
For `length = 1`, the boundary candidates are `10 ** 0 - 1 = 0` and `10 ** 1 + 1 = 11`. It is easy to assume these are the only candidates and add a separate branch for single digits, but the mirroring loop already covers them.

**Example:**
```python
n = "5"
length = 1
candidates = {10 ** (1 - 1) - 1, 10 ** 1 + 1}  # {0, 11}
# prefix = 5; variants 4, 5, 6 have length 1 (odd), so suffix_to_mirror = variant // 10 = 0
# The palindromes 4, 5, 6 are added unchanged
```

After discarding 5, the candidates are `{0, 4, 6, 11}`. Both 4 and 6 are at distance 1, and the tie-break returns 4. For `n = "9"`, the variant 10 mirrors to 101, and the answer is 8. No special case is needed, and a hand-written branch (such as returning `"1"` for `"0"`) risks disagreeing with the general logic.

### 3. **Missing Edge Case: Numbers Like "99...99" and "10...01"**
Mirroring the prefix alone cannot cross a change in the number of digits. For `n = "99"`, the prefix is 9, and the variants 8, 9, 10 mirror to 88, 99, and 1001. The closest palindrome, 101, has more digits and only appears through the boundary candidate `10^length + 1`. Similarly, for `n = "100"` the closest palindrome is 99, which comes from `10^(length - 1) - 1`.

**Problem Example:**
```python
n = "99"
# Candidates: {9, 101, 88, 99, 1001}; 99 is discarded
# Without the boundary candidates, the answer would be 88 instead of 101
```

**Solution:**
Always add both boundary candidates, `10^(length - 1) - 1` and `10^length + 1`, before choosing the closest palindrome.

### 4. **Incorrect Mirroring for Even vs Odd Length Numbers**
A common mistake is incorrectly determining what to mirror, especially confusing when to include or exclude the middle digit.

**Problem Example:**
```python
# Incorrect implementation might do:
suffix_to_mirror = prefix_variant // 10  # Always divide by 10
# This would be wrong for even-length numbers
```

**Solution:**
Always use the correct conditional:
```python
suffix_to_mirror = prefix_variant if length % 2 == 0 else prefix_variant // 10
```

### 5. **Not Handling the Original Number Being a Palindrome**
If the input is already a palindrome, we must find a different palindrome, not return the same number.

**Problem Example:**
```python
n = "121"  # Already a palindrome
# Must not return "121" again
```

**Solution:**
The `candidates.discard(original_num)` line handles this, but it's essential not to forget this step. Some developers might try to optimize by checking if the number is a palindrome first and handling it separately, which can introduce bugs.

### 6. **Tie-Breaking Logic Error**
When two palindromes are equidistant, the smaller one should be returned, but the comparison logic can be tricky.

**Problem Example:**
```python
# Incorrect tie-breaking:
if distance_to_candidate <= distance_to_current_nearest:  # Wrong!
    nearest_palindrome = candidate
# This doesn't ensure we pick the smaller value in case of a tie
```

**Solution:**
Use explicit tie-breaking logic as shown in the original code:
```python
if (nearest_palindrome == -1 or 
    distance_to_candidate < distance_to_current_nearest or
    (distance_to_candidate == distance_to_current_nearest and 
     candidate < nearest_palindrome)):
```

