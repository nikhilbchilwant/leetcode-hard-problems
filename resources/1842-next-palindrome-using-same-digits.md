# 1842. Next Palindrome Using Same Digits

> Source: https://algo.monster/liteproblems/1842

## Problem Statement

You are given a numeric string `num` that represents a very large palindrome (a number that reads the same forwards and backwards).

Your task is to find the smallest palindrome that is larger than `num` and can be created by rearranging the digits of `num`. If no such palindrome exists, return an empty string `""`.

The key insight is that since the input is already a palindrome, we need to find the next lexicographically larger arrangement of its digits that still forms a palindrome.

The solution works by focusing on just the first half of the palindrome. Since a palindrome is symmetric, once we determine what the first half should be, the second half is automatically determined (it mirrors the first half).

The algorithm:
1. Extract the first half of the palindrome string
2. Find the next permutation of this first half (the next lexicographically larger arrangement)
3. If a next permutation exists, mirror it to create the second half, forming a complete palindrome
4. If no next permutation exists (meaning the first half is already in descending order), return `""`

For example, if `num = "1221"`, the first half is `"12"`. The next permutation of `"12"` is `"21"`, so the resulting palindrome would be `"2112"`.

The `next_permutation` function implements the classic algorithm to find the next lexicographically larger permutation by:
- Finding the rightmost position where a digit can be increased
- Swapping it with the smallest digit to its right that is larger than it
- Reversing the remaining digits to get the smallest possible arrangement

## Intuition

Since we're dealing with a palindrome, we know that the second half is completely determined by the first half - it's just a mirror image. This means we don't need to consider all the digits independently; we only need to focus on rearranging the first half.

The key realization is that finding the "smallest palindrome larger than `num`" is equivalent to finding the next lexicographically larger arrangement of the first half. Why? Because once we have the next arrangement of the first half, mirroring it automatically gives us the next palindrome.

Think about it this way: if we have a palindrome like `"1221"`, the first half is `"12"` and it determines the entire palindrome. To get the next larger palindrome, we need the first half to become `"21"`, which then creates `"2112"` when mirrored.

But how do we ensure it's the *smallest* palindrome larger than the original? This is where the "next permutation" algorithm comes in. The next permutation gives us the very next arrangement in lexicographical order - there's no smaller arrangement between the current one and the next permutation. This guarantees we get the smallest possible larger palindrome.

The reason we can't just randomly rearrange digits is that we need to maintain the property that our result is larger than the original while being as small as possible. The next permutation algorithm achieves this by:
1. Finding the rightmost position where we can make an increase
2. Making the smallest possible increase at that position
3. Arranging the remaining digits in ascending order to minimize the overall value

If no next permutation exists (when the first half is already in descending order like `"321"`), then we've exhausted all possible arrangements, and no larger palindrome can be formed by rearranging the digits.

**Learn more about [Two Pointers](/problems/two_pointers_intro) patterns.**

## Solution Approach

The solution implements the strategy of finding the next permutation of the first half of the palindrome string. Let's walk through the implementation step by step:

**Main Function Structure:**

The solution consists of two parts:
1. A helper function `next_permutation` that finds the next lexicographically larger arrangement
2. The main logic that applies this to create the palindrome

**Next Permutation Algorithm:**

The `next_permutation` function works on the first half of the digits:

1. **Find the pivot point** (`i`): Starting from the second-to-last position of the first half (`n - 2`), scan leftward to find the first position where `nums[i] < nums[i + 1]`. This is the rightmost position where we can make an increase.
   ```python
   i = n - 2
   while i >= 0 and nums[i] >= nums[i + 1]:
       i -= 1
   ```

2. **Check if permutation exists**: If `i < 0`, all digits are in descending order, meaning we already have the largest possible permutation. Return `False`.

3. **Find the swap candidate** (`j`): From the end of the first half, find the smallest digit that is still larger than `nums[i]`.
   ```python
   j = n - 1
   while j >= 0 and nums[j] <= nums[i]:
       j -= 1
   ```

4. **Swap and reverse**: Swap `nums[i]` with `nums[j]`, then reverse all digits after position `i` to get the smallest arrangement of the remaining digits.
   ```python
   nums[i], nums[j] = nums[j], nums[i]
   nums[i + 1 : n] = nums[i + 1 : n][::-1]
   ```

**Creating the Palindrome:**

Once we have the next permutation of the first half:

1. Convert the input string to a list for easier manipulation
2. Apply `next_permutation` to get the next arrangement
3. If successful, mirror the first half to create the second half:
   ```python
   for i in range(n // 2):
       nums[n - i - 1] = nums[i]
   ```
4. Join the list back into a string and return

The algorithm handles both even and odd-length palindromes correctly because:
- For even-length palindromes, we work with exactly half the digits
- For odd-length palindromes, the middle digit remains unchanged, and we still only need to rearrange the first half

**Time Complexity:** `O(n)` where `n` is the length of the string
**Space Complexity:** `O(n)` for storing the list representation of the string

## Example Walkthrough

Let's walk through the solution with `num = "12321"`, a 5-digit palindrome.

**Step 1: Extract the first half**
- Length = 5, so first half length = 5 // 2 = 2
- First half: positions [0, 1] = "12"
- Convert entire string to list: ['1', '2', '3', '2', '1']

**Step 2: Find next permutation of first half**

We need to find the next permutation of indices [0, 1]:

a) **Find pivot point (i):**
   - Start at i = 0 (n-2 where n=2)
   - Check: nums[0] < nums[1]? → '1' < '2'? Yes!
   - Found pivot at i = 0

b) **Find swap candidate (j):**
   - Start at j = 1 (n-1 where n=2)
   - Check: nums[1] > nums[0]? → '2' > '1'? Yes!
   - Found swap candidate at j = 1

c) **Perform swap:**
   - Swap nums[0] and nums[1]
   - List becomes: ['2', '1', '3', '2', '1']

d) **Reverse after pivot:**
   - Reverse from position i+1 to n-1 (positions 1 to 1)
   - Only one element, so no change
   - List remains: ['2', '1', '3', '2', '1']

**Step 3: Mirror the first half to create palindrome**
- Copy position 0 to position 4: nums[4] = '2'
- Copy position 1 to position 3: nums[3] = '1'
- Middle element (position 2) stays unchanged
- Final list: ['2', '1', '3', '1', '2']

**Step 4: Convert back to string**
- Result: "21312"

**Verification:**
- Original: "12321"
- Result: "21312"
- Is it a palindrome? Yes (reads same forwards and backwards)
- Is it larger? Yes (21312 > 12321)
- Is it the smallest larger palindrome? Yes (next permutation guarantees this)

**Edge Case Example:**

If `num = "32123"`:
- First half: "32"
- Already in descending order (3 ≥ 2)
- No next permutation exists
- Return: ""

This demonstrates why the algorithm correctly identifies when no larger palindrome can be formed by rearrangement.

## Python Solution

```python
class Solution:
    def nextPalindrome(self, num: str) -> str:
        """
        Find the next palindrome by rearranging the digits of the given number.

        Args:
            num: A string representing a palindromic number

        Returns:
            The next palindrome that can be formed by rearranging digits,
            or empty string if no such palindrome exists
        """

        def next_permutation(nums: list[str]) -> bool:
            """
            Find the next lexicographically greater permutation of the first half.
            This modifies the list in-place.

            Args:
                nums: List of digit characters (only first half is considered)

            Returns:
                True if next permutation exists, False otherwise
            """
            # Only work with the first half of the palindrome
            n = len(nums) // 2

            # Step 1: Find the rightmost position i where nums[i] < nums[i+1]
            # This is the position that needs to be increased
            i = n - 2
            while i >= 0 and nums[i] >= nums[i + 1]:
                i -= 1

            # If no such position exists, we've reached the largest permutation
            if i < 0:
                return False

            # Step 2: Find the rightmost position j where nums[j] > nums[i]
            # This will be swapped with position i
            j = n - 1
            while j >= 0 and nums[j] <= nums[i]:
                j -= 1

            # Step 3: Swap the elements at positions i and j
            nums[i], nums[j] = nums[j], nums[i]

            # Step 4: Reverse the suffix starting at position i+1
            # This gives us the smallest permutation greater than the original
            nums[i + 1 : n] = nums[i + 1 : n][::-1]

            return True

        # Convert string to list for in-place modifications
        nums = list(num)

        # Try to find the next permutation of the first half
        if not next_permutation(nums):
            return ""

        # Mirror the first half to create a palindrome
        n = len(nums)
        for i in range(n // 2):
            nums[n - i - 1] = nums[i]

        # Convert back to string and return
        return "".join(nums)
```

## Time and Space Complexity

The time complexity is `O(n)`, where `n` is the length of the input string.

- The `next_permutation` function performs operations on the first half of the string (length `n/2`):
  - The first while loop iterates at most `n/2` times to find the pivot point: `O(n/2)`
  - The second while loop iterates at most `n/2` times to find the swap position: `O(n/2)`
  - The swap operation is `O(1)`
  - The reversal operation `nums[i + 1 : n][::-1]` affects at most `n/2` elements: `O(n/2)`
- After `next_permutation`, the for loop runs `n/2` times to mirror the first half to the second half: `O(n/2)`
- The `join` operation to create the final string is `O(n)`

All these operations sum to `O(n/2) + O(n/2) + O(n/2) + O(n/2) + O(n) = O(n)`.

The space complexity is `O(n)`, where `n` is the length of the input string.

- Converting the string to a list creates a new list of size `n`: `O(n)`
- The slicing operation `nums[i + 1 : n][::-1]` creates a temporary list of at most `n/2` elements: `O(n/2)`
- The final string creation with `join` requires `O(n)` space

The dominant space usage is `O(n)` for storing the list representation of the string.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrectly Handling Odd-Length Palindromes**

A common mistake is assuming that the middle character in odd-length palindromes needs special treatment during the next permutation calculation. This can lead to incorrect boundary calculations or attempting to include the middle character in the permutation logic.

**Pitfall Example:**
```python
# Incorrect: Trying to include middle character in permutation
n = len(nums) // 2 + (len(nums) % 2)  # Wrong boundary for odd-length
```

**Solution:**
The algorithm correctly uses `n = len(nums) // 2` for both even and odd-length palindromes. The middle character (if it exists) remains unchanged and doesn't participate in the permutation:
```python
n = len(nums) // 2  # Correct for both even and odd lengths
```

### 2. **Off-by-One Errors in Index Calculations**

When mirroring the first half to create the palindrome, it's easy to make indexing errors, especially with the relationship between `i` and its mirror position `n - i - 1`.

**Pitfall Example:**
```python
# Incorrect mirroring
for i in range(n // 2):
    nums[n - i] = nums[i]  # Wrong: should be n - i - 1
```

**Solution:**
Use the correct mirror index formula:
```python
for i in range(n // 2):
    nums[n - i - 1] = nums[i]  # Correct mirroring
```

### 3. **Not Preserving Original Input When No Solution Exists**

Some implementations might partially modify the input array before determining that no next permutation exists, leading to corrupted results.

**Pitfall Example:**
```python
def next_permutation(nums):
    # Modifying nums directly without checking if solution exists first
    nums[0], nums[1] = nums[1], nums[0]  # Premature modification
    # Then checking if valid...
```

**Solution:**
The algorithm correctly checks for the existence of a valid pivot point (`i`) before making any modifications. If `i < 0`, it returns `False` immediately without changing the array.

### 4. **Forgetting to Reverse the Suffix After Swapping**

A critical step in the next permutation algorithm is reversing the suffix after the swap to ensure we get the smallest possible next permutation. Omitting this step results in a larger-than-necessary palindrome.

**Pitfall Example:**
```python
# After swapping nums[i] and nums[j]
nums[i], nums[j] = nums[j], nums[i]
# Forgot to reverse the suffix - results in wrong answer
```

**Solution:**
Always reverse the suffix after the swap:
```python
nums[i], nums[j] = nums[j], nums[i]
nums[i + 1 : n] = nums[i + 1 : n][::-1]  # Essential step
```

### 5. **String vs List Confusion**

Attempting to modify a string directly (strings are immutable in Python) or forgetting to convert back to string at the end.

**Pitfall Example:**
```python
# Trying to modify string directly
num[i] = num[j]  # Error: strings are immutable
```

**Solution:**
Convert to list for manipulation, then back to string:
```python
nums = list(num)  # Convert to mutable list
# ... perform operations ...
return "".join(nums)  # Convert back to string
```

