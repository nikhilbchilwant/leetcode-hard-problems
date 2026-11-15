# 927. Three Equal Parts

> Source: https://algo.monster/liteproblems/927

## Problem Statement

You have an array `arr` containing only 0s and 1s. Your task is to divide this array into **three non-empty parts** where each part represents the same binary value.

Given an array, you need to find two indices `[i, j]` where `i + 1 < j` such that:
- The first part: `arr[0], arr[1], ..., arr[i]`
- The second part: `arr[i + 1], arr[i + 2], ..., arr[j - 1]`
- The third part: `arr[j], arr[j + 1], ..., arr[arr.length - 1]`

All three parts must have equal binary values when interpreted as binary numbers.

**Important notes:**
- Each part is interpreted as a complete binary number. For example, `[1,1,0]` represents 6 in decimal (110 in binary = 6), not 3.
- Leading zeros are allowed, meaning `[0,1,1]` and `[1,1]` both represent the value 3 in decimal.
- If it's impossible to divide the array into three equal parts, return `[-1, -1]`.
- If a valid division exists, return any valid `[i, j]` pair.

**Example considerations:**
- If the array is `[1,0,1,0,1]`, one valid division could make all three parts equal to the binary value 5 (binary 101).
- If the array is all zeros like `[0,0,0,0,0,0]`, any division would work since all parts would equal 0.
- The division must create exactly three non-empty parts - you cannot have empty parts.

## Intuition

The key insight is that for three parts to represent the same binary value, they must have the same number of 1s. This is because the positions and count of 1s fundamentally determine a binary number's value.

Let's think about what makes binary numbers equal:
- They must have the same number of 1s (since each 1 contributes to the value)
- The pattern of 1s and 0s must match when we ignore leading zeros

First, we count the total number of 1s in the array. If this count isn't divisible by 3, it's impossible to split the array into three equal parts - we return `[-1, -1]` immediately.

If the count is 0 (all zeros), any split works since `0 == 0 == 0`, so we can return `[0, n-1]`.

For the general case, each part must contain exactly `cnt/3` ones. Here's the clever part: we can find where each part should approximately start by locating:
- The 1st one (start of first part's significant digits)
- The `(cnt/3 + 1)`-th one (start of second part's significant digits)  
- The `(2*cnt/3 + 1)`-th one (start of third part's significant digits)

Once we have these starting positions `i`, `j`, and `k`, we need to verify that the three parts are actually equal. We do this by moving through all three parts simultaneously, comparing corresponding positions. If at any point `arr[i] != arr[j]` or `arr[j] != arr[k]`, the parts aren't equal.

The traversal continues until `k` reaches the end of the array. If we successfully reach the end with all corresponding elements matching, we've found our answer: the first part ends at `i-1`, and the third part starts at `j`.

This approach works because:
1. We ensure each part has the correct number of 1s
2. We verify the pattern matches by parallel traversal
3. Leading zeros are naturally handled since we start comparing from the first 1 in each part

**Learn more about [Math](/problems/math-basics) patterns.**

## Solution Approach

The implementation follows a systematic approach using counting and three pointers:

**Step 1: Count and Validate**

First, we count the total number of 1s in the array and check if it's divisible by 3:
```python
cnt, mod = divmod(sum(arr), 3)
```
- If `mod != 0`, the array cannot be split into three equal parts, return `[-1, -1]`
- If `cnt == 0` (all zeros), return `[0, n-1]` since any split works

**Step 2: Find Starting Positions**

We use a helper function `find(x)` to locate the index of the x-th 1 in the array:
```python
def find(x):
    s = 0
    for i, v in enumerate(arr):
        s += v
        if s == x:
            return i
```

Using this function, we find three critical positions:
- `i = find(1)`: Position of the 1st one (start of first part's significant bits)
- `j = find(cnt + 1)`: Position of the `(cnt + 1)`-th one (start of second part's significant bits)
- `k = find(cnt * 2 + 1)`: Position of the `(2*cnt + 1)`-th one (start of third part's significant bits)

**Step 3: Parallel Traversal and Verification**

Starting from positions `i`, `j`, and `k`, we traverse all three parts simultaneously:
```python
while k < n and arr[i] == arr[j] == arr[k]:
    i, j, k = i + 1, j + 1, k + 1
```

This loop continues as long as:
- `k` hasn't reached the end of the array
- The corresponding elements in all three parts are equal

**Step 4: Return Result**

After the traversal:
- If `k == n`, we've successfully verified all three parts are equal
  - Return `[i - 1, j]` where:
    - First part: `arr[0...i-1]`
    - Second part: `arr[i...j-1]`
    - Third part: `arr[j...n-1]`
- Otherwise, return `[-1, -1]` indicating no valid split exists

**Example Walkthrough:**

For array `[1,0,1,0,1,0,1,0,1]`:
1. Total 1s = 5, not divisible by 3 → return `[-1, -1]`

For array `[1,1,0,0,1,1,0,0,1,1]`:
1. Total 1s = 6, each part needs 2 ones
2. Find positions: `i=0`, `j=4`, `k=8` (positions of 1st, 3rd, and 5th ones)
3. Parallel traversal verifies the pattern matches
4. Return appropriate indices

**Time Complexity:** `O(n)` - single pass to count, find positions, and verify
**Space Complexity:** `O(1)` - only using a few variables

## Example Walkthrough

Let's walk through the solution with the array `[1,0,1,0,1]`:

**Step 1: Count and Validate**
- Count total 1s: We have three 1s in the array
- Check divisibility: 3 ÷ 3 = 1 with remainder 0 ✓
- Each part needs exactly 1 one

**Step 2: Find Starting Positions**
- Find the 1st one: Position 0 (value = 1)
- Find the 2nd one (1 + 1): Position 2 (value = 1) 
- Find the 3rd one (2 + 1): Position 4 (value = 1)
- So `i = 0`, `j = 2`, `k = 4`

**Step 3: Parallel Traversal**
Starting from positions i=0, j=2, k=4:

| Step | i | j | k | arr[i] | arr[j] | arr[k] | Match? |
|------|---|---|---|--------|--------|--------|--------|
| 1 | 0 | 2 | 4 | 1 | 1 | 1 | ✓ |
| 2 | 1 | 3 | 5 | 0 | 0 | - | k >= n |

When k reaches 5 (array length), we stop.

**Step 4: Return Result**
Since k successfully reached the end of the array with all matches:
- After the loop, `i = 1` and `j = 3`
- Return `[i - 1, j] = [0, 3]`

**Verification:**
- Part 1: arr[0:1] = `[1]` → binary value = 1
- Part 2: arr[1:3] = `[0,1]` → binary value = 1 (leading zero ignored)
- Part 3: arr[3:5] = `[0,1]` → binary value = 1 (leading zero ignored)

All three parts equal 1 ✓

---

Let's try another example with `[0,1,0,0,1,0,0,1]`:

**Step 1:** Count 1s = 3, divisible by 3, each part needs 1 one

**Step 2:** Find positions:
- 1st one at index 1: `i = 1`
- 2nd one at index 4: `j = 4` 
- 3rd one at index 7: `k = 7`

**Step 3:** Parallel traversal from i=1, j=4, k=7:

| Step | i | j | k | arr[i] | arr[j] | arr[k] | Match? |
|------|---|---|---|--------|--------|--------|--------|
| 1 | 1 | 4 | 7 | 1 | 1 | 1 | ✓ |
| 2 | 2 | 5 | 8 | 0 | 0 | - | k >= n |

**Step 4:** Return `[1, 5]`

**Verification:**
- Part 1: arr[0:2] = `[0,1]` → value = 1
- Part 2: arr[2:5] = `[0,0,1]` → value = 1  
- Part 3: arr[5:8] = `[0,0,1]` → value = 1

All parts equal 1 ✓

## Python Solution

```python
class Solution:
    def threeEqualParts(self, arr: List[int]) -> List[int]:
        """
        Split binary array into three parts with equal binary values.
      
        Args:
            arr: List of 0s and 1s representing a binary number
          
        Returns:
            List of two indices [i, j] where:
            - Part 1: arr[0] to arr[i]
            - Part 2: arr[i+1] to arr[j-1]  
            - Part 3: arr[j] to arr[n-1]
            Returns [-1, -1] if no valid split exists
        """
      
        def find_nth_one(target_ones: int) -> int:
            """
            Find the index of the nth '1' in the array.
          
            Args:
                target_ones: The target count of 1s to find
              
            Returns:
                Index where cumulative count of 1s equals target_ones
            """
            ones_count = 0
            for index, value in enumerate(arr):
                ones_count += value
                if ones_count == target_ones:
                    return index
      
        # Get array length and total count of 1s
        array_length = len(arr)
        total_ones = sum(arr)
      
        # Check if total 1s can be evenly divided into 3 parts
        ones_per_part, remainder = divmod(total_ones, 3)
      
        # If not divisible by 3, no valid split exists
        if remainder != 0:
            return [-1, -1]
      
        # Special case: if all zeros, any split works
        if ones_per_part == 0:
            return [0, array_length - 1]
      
        # Find starting positions of first '1' in each part
        # first_start: index of 1st '1' in part 1
        # second_start: index of 1st '1' in part 2 (after ones_per_part 1s)
        # third_start: index of 1st '1' in part 3 (after 2*ones_per_part 1s)
        first_start = find_nth_one(1)
        second_start = find_nth_one(ones_per_part + 1)
        third_start = find_nth_one(ones_per_part * 2 + 1)
      
        # Compare the three parts digit by digit from their first '1'
        # All three parts must match exactly to be equal binary numbers
        while third_start < array_length and \
              arr[first_start] == arr[second_start] == arr[third_start]:
            first_start += 1
            second_start += 1
            third_start += 1
      
        # If we compared all digits successfully (third_start reached end),
        # return the split indices, otherwise no valid split exists
        if third_start == array_length:
            return [first_start - 1, second_start]
        else:
            return [-1, -1]
```

## Time and Space Complexity

**Time Complexity:** `O(n)`

The algorithm performs the following operations:
- `sum(arr)`: Iterates through the array once - `O(n)`
- `find(x)` function: Called 3 times, each iterating through the array in worst case - `3 * O(n) = O(n)`
- The while loop at the end: Iterates through remaining elements after position `k`, which is at most `n - k` elements - `O(n)` in worst case

Since all operations are sequential and not nested, the overall time complexity is `O(n) + O(n) + O(n) = O(n)`.

**Space Complexity:** `O(1)`

The algorithm uses only a constant amount of extra space:
- Variables `n`, `cnt`, `mod`: `O(1)`
- Variables `i`, `j`, `k`: `O(1)` 
- Variable `s` inside the `find` function: `O(1)`

No additional data structures that scale with input size are created, so the space complexity is `O(1)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Misunderstanding Binary Value Equality vs String Equality**

**Pitfall:** A common mistake is thinking that the three parts need to be identical strings of 0s and 1s, rather than having equal binary values. For example, thinking `[0,1,1]` and `[1,1]` are different, when they both represent the decimal value 3.

**Why it happens:** The problem requires comparing binary VALUES, not binary STRINGS. Leading zeros don't affect the value.

**Solution:** The algorithm correctly handles this by:
- Finding the first '1' in each part (skipping leading zeros)
- Comparing from these starting positions onward
- This ensures `[0,0,1,0,1]` and `[1,0,1]` are recognized as equal (both are 5)

### 2. **Incorrect Handling of All-Zeros Case**

**Pitfall:** When the array contains only zeros, the `find_nth_one()` function would return `None` since it never finds a '1', causing the code to crash when trying to use these positions.

**Why it happens:** The special case of all zeros needs different handling since there are no '1's to find and any split is valid.

**Solution:** The code handles this correctly by:
```python
if ones_per_part == 0:  # All zeros case
    return [0, array_length - 1]
```
This returns immediately before attempting to find any '1's.

### 3. **Off-by-One Errors in Index Calculation**

**Pitfall:** Confusion about whether indices are inclusive or exclusive, especially when returning `[i-1, j]` instead of `[i, j]`.

**Why it happens:** The problem requires:
- Part 1: `arr[0...i]` (inclusive)
- Part 2: `arr[i+1...j-1]` (inclusive)
- Part 3: `arr[j...n-1]` (inclusive)

After the while loop, `first_start` has moved one position past the last matching element, so we need to return `[first_start - 1, second_start]`.

**Solution:** Carefully track what each pointer represents:
- During comparison: pointers track current comparison position
- After comparison: `first_start - 1` gives the correct end of Part 1
- `second_start` gives the correct start of Part 3

### 4. **Not Validating Trailing Zeros in the Third Part**

**Pitfall:** The algorithm must ensure the third part ends exactly at the array's end. If there are different numbers of trailing zeros in each part, they won't be equal.

**Example:** Array `[1,0,1,1,0]` cannot be split into three equal parts. The third part must contain the last '1' and the trailing '0', so its value is at least `10` in binary (2). The first '1' is followed by only one '0' before the next '1', and the second '1' is immediately followed by the third, so the first two parts cannot both end in a '0'. The comparison from positions 0, 2, 3 fails at the second digit (`arr[1] = 0`, `arr[3] = 1`, `arr[4] = 0`), and the function returns `[-1, -1]`.

**Solution:** The code handles this by:
```python
while third_start < array_length and \
      arr[first_start] == arr[second_start] == arr[third_start]:
```
The condition `third_start == array_length` after the loop ensures we've compared ALL elements, including any trailing zeros in the third part.

### 5. **Integer Overflow in Very Large Arrays**

**Pitfall:** While not an issue in Python, in other languages, interpreting very long binary strings as integers could cause overflow.

**Solution:** The algorithm avoids this by never actually converting the binary parts to integers. Instead, it compares them digit-by-digit, which works regardless of the array size.

