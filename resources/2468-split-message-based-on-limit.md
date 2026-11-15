# 2468. Split Message Based on Limit

> Source: https://algo.monster/liteproblems/2468

## Problem Statement

You need to split a given string `message` into multiple parts, where each part has a special suffix format `<a/b>`.

The suffix format works as follows:
- `a` represents the current part number (starting from 1)
- `b` represents the total number of parts
- For example, if you split a message into 3 parts, the suffixes would be `<1/3>`, `<2/3>`, and `<3/3>`

The splitting must follow these rules:

1. **Length constraint**: Each resulting part (including its suffix) must have a length exactly equal to `limit`, except for the last part which can have a length at most `limit`.

2. **Message integrity**: When you remove all the suffixes from the parts and concatenate them back together in order, you must get the original `message` string.

3. **Minimum parts**: The split should use as few parts as possible.

4. **Return value**: Return an array of strings representing the split parts with their suffixes. If it's impossible to split the message according to these rules, return an empty array.

For example, if `message = "hello world"` and `limit = 10`:
- You might split it into parts like `"hello <1/2>"` and `"world<2/2>"`
- The first part has length 10 (exactly `limit`)
- The second part has length 10 (exactly `limit`)
- Removing the suffixes `<1/2>` and `<2/2>` and concatenating gives back `"hello world"`

The challenge is that the suffix itself takes up space in each part, and the suffix length can vary depending on the total number of parts (since `b` can be 1, 10, 100, etc., affecting the suffix length).

## Intuition

The key insight is that we need to find the minimum number of parts `k` that can accommodate the entire message. Since we want the minimum number of parts, we should try increasing values of `k` starting from 1.

For any given number of parts `k`, we need to determine if it's possible to fit the message. The challenge is that the suffix format `<a/b>` takes up space, and this space varies depending on the number of digits in both `a` and `b`.

Let's think about what happens when we have `k` parts:
- Each part needs a suffix in the format `<a/k>` where `a` goes from 1 to `k`
- The suffix length for part `j` is: 3 (for `<`, `/`, `>`) + length of digits in `j` + length of digits in `k`
- The total space taken by all suffixes needs to be calculated

For `k` parts, the total space available is `limit × k`. From this total space, we need to subtract:
1. **`sa`**: The sum of lengths of all part numbers (1, 2, ..., k). For example, if `k=12`, we need space for digits: 1,2,3,4,5,6,7,8,9,10,11,12. This gives us `9×1 + 3×2 = 15` characters total.
2. **`sb`**: The space for the total number `k` repeated in each suffix. Since `k` appears in all `k` parts, this is `len(str(k)) × k`.
3. **`sc`**: The space for symbols (`<`, `/`, `>`) in all parts, which is `3 × k`.

The remaining space after subtracting these suffix components is what's available for the actual message content. If this remaining space is at least `n` (the message length), then we can split the message into `k` parts.

Once we find the minimum valid `k`, we can construct the actual parts by:
- Taking chunks of the message
- Adding the appropriate suffix to each chunk
- Making sure each part (except possibly the last) has exactly `limit` characters

The reason we only need to check up to `k = n` is that having more parts than characters in the message would be wasteful and unnecessary - we're looking for the minimum number of parts.

**Learn more about [Binary Search](/problems/binary_search_intro) patterns.**

## Solution Approach

The solution implements the intuition by systematically checking each possible number of parts `k` from 1 to `n`.

For each value of `k`, we calculate three key components:

1. **`sa` - Sum of all part indices**: We accumulate the total length of all part numbers from 1 to `k`. This is done incrementally using `sa += len(str(k))`. For example, when `k=10`, `sa` includes the lengths of "1", "2", ..., "9" (9 characters) plus "10" (2 characters).

2. **`sb` - Total length of `k` repeated**: Since the total number `k` appears in every suffix, we calculate `sb = len(str(k)) × k`. For `k=10`, each suffix contains "10", so we need `2 × 10 = 20` characters for all the "10"s.

3. **`sc` - Fixed symbols length**: Every suffix has exactly 3 symbols (`<`, `/`, `>`), so for `k` parts: `sc = 3 × k`.

The condition for feasibility is:
```
limit × k - (sa + sb + sc) >= n
```

This checks if the total available space minus the space needed for all suffixes can accommodate the entire message of length `n`.

Once we find the smallest valid `k`, we construct the result:

1. Initialize an empty result array and a pointer `i = 0` to track our position in the message
2. For each part `j` from 1 to `k`:
   - Create the suffix: `tail = f'<{j}/{k}>'`
   - Calculate how much message content can fit: `limit - len(tail)`
   - Extract that portion from the message: `message[i : i + limit - len(tail)]`
   - Combine content with suffix to create the complete part
   - Update the pointer `i` to move forward in the message

The algorithm returns the constructed array if a valid split is found, or an empty array if no valid split exists within the range `[1, n]`.

The time complexity is `O(n²)` in the worst case, where we might need to check all values of `k` up to `n`, and for each valid `k`, we construct `k` parts. The space complexity is `O(n)` for storing the result array.

## Example Walkthrough

Let's walk through the solution with `message = "hi there"` (length 8) and `limit = 6`.

**Step 1: Try k = 1 part**
- We need suffix `<1/1>` which has length 5 (characters: '<', '1', '/', '1', '>')
- Calculate space components:
  - `sa = 1` (length of "1")
  - `sb = 1 × 1 = 1` (length of "1" × 1 occurrence)
  - `sc = 3 × 1 = 3` (3 symbols × 1 part)
- Total suffix space: `sa + sb + sc = 1 + 1 + 3 = 5`
- Available space for message: `6 × 1 - 5 = 1`
- Need space: 8 (message length)
- Since 1 < 8, k = 1 doesn't work ❌

**Step 2: Try k = 2 parts**
- We need suffixes `<1/2>` and `<2/2>`, each with length 5
- Calculate space components:
  - `sa = 1 + 1 = 2` (length of "1" + length of "2")
  - `sb = 1 × 2 = 2` (length of "2" × 2 occurrences)
  - `sc = 3 × 2 = 6` (3 symbols × 2 parts)
- Total suffix space: `sa + sb + sc = 2 + 2 + 6 = 10`
- Available space for message: `6 × 2 - 10 = 2`
- Need space: 8
- Since 2 < 8, k = 2 doesn't work ❌

**Step 3: Try k = 3 parts**
- We need suffixes `<1/3>`, `<2/3>`, `<3/3>`, each with length 5
- Calculate space components:
  - `sa = 1 + 1 + 1 = 3` (lengths of "1", "2", "3")
  - `sb = 1 × 3 = 3` (length of "3" × 3 occurrences)
  - `sc = 3 × 3 = 9` (3 symbols × 3 parts)
- Total suffix space: `sa + sb + sc = 3 + 3 + 9 = 15`
- Available space for message: `6 × 3 - 15 = 3`
- Need space: 8
- Since 3 < 8, k = 3 doesn't work ❌

**Step 4: Try k = 4 parts**
- We need suffixes `<1/4>`, `<2/4>`, `<3/4>`, `<4/4>`, each with length 5
- Calculate space components:
  - `sa = 1 + 1 + 1 + 1 = 4` (lengths of "1", "2", "3", "4")
  - `sb = 1 × 4 = 4` (length of "4" × 4 occurrences)
  - `sc = 3 × 4 = 12` (3 symbols × 4 parts)
- Total suffix space: `sa + sb + sc = 4 + 4 + 12 = 20`
- Available space for message: `6 × 4 - 20 = 4`
- Need space: 8
- Since 4 < 8, k = 4 doesn't work ❌

**Step 5: Try k = 5 parts**
- We need suffixes `<1/5>` through `<5/5>`, each with length 5
- Calculate space components:
  - `sa = 1 + 1 + 1 + 1 + 1 = 5` (lengths of "1" through "5")
  - `sb = 1 × 5 = 5` (length of "5" × 5 occurrences)
  - `sc = 3 × 5 = 15` (3 symbols × 5 parts)
- Total suffix space: `sa + sb + sc = 5 + 5 + 15 = 25`
- Available space for message: `6 × 5 - 25 = 5`
- Need space: 8
- Since 5 < 8, k = 5 doesn't work ❌

**Step 6: Try k = 6 parts**
- Calculate space components:
  - `sa = 1 + 1 + 1 + 1 + 1 + 1 = 6`
  - `sb = 1 × 6 = 6`
  - `sc = 3 × 6 = 18`
- Total suffix space: `30`
- Available space: `6 × 6 - 30 = 6`
- Since 6 < 8, doesn't work ❌

**Step 7: Try k = 7 parts**
- Calculate space components:
  - `sa = 1 + 1 + 1 + 1 + 1 + 1 + 1 = 7`
  - `sb = 1 × 7 = 7`
  - `sc = 3 × 7 = 21`
- Total suffix space: `35`
- Available space: `6 × 7 - 35 = 7`
- Since 7 < 8, doesn't work ❌

**Step 8: Try k = 8 parts**
- Calculate space components:
  - `sa = 1 + 1 + 1 + 1 + 1 + 1 + 1 + 1 = 8`
  - `sb = 1 × 8 = 8`
  - `sc = 3 × 8 = 24`
- Total suffix space: `40`
- Available space: `6 × 8 - 40 = 8`
- Since 8 = 8, k = 8 works! ✓

**Constructing the result:**
Now we split "hi there" into 8 parts:
1. Part 1: suffix `<1/8>` (5 chars) → can fit 1 char → "h<1/8>"
2. Part 2: suffix `<2/8>` (5 chars) → can fit 1 char → "i<2/8>"
3. Part 3: suffix `<3/8>` (5 chars) → can fit 1 char → " <3/8>"
4. Part 4: suffix `<4/8>` (5 chars) → can fit 1 char → "t<4/8>"
5. Part 5: suffix `<5/8>` (5 chars) → can fit 1 char → "h<5/8>"
6. Part 6: suffix `<6/8>` (5 chars) → can fit 1 char → "e<6/8>"
7. Part 7: suffix `<7/8>` (5 chars) → can fit 1 char → "r<7/8>"
8. Part 8: suffix `<8/8>` (5 chars) → can fit 1 char → "e<8/8>"

**Final result:** `["h<1/8>", "i<2/8>", " <3/8>", "t<4/8>", "h<5/8>", "e<6/8>", "r<7/8>", "e<8/8>"]`

Each part has exactly 6 characters (meeting the limit), and removing all suffixes gives us back "hi there".

## Python Solution

```python
class Solution:
    def splitMessage(self, message: str, limit: int) -> List[str]:
        """
        Split a message into parts with suffix tags <current/total>.
        Each part including the tag must not exceed the limit.

        Args:
            message: The original message string to split
            limit: Maximum length for each part including the tag

        Returns:
            List of message parts with tags, or empty list if impossible
        """
        message_length = len(message)

        # cumulative_digits tracks total digits used for part numbers 1 to k
        cumulative_digits = 0

        # Try different values of k (total number of parts)
        for total_parts in range(1, message_length + 1):
            # Add digits needed for current part number
            cumulative_digits += len(str(total_parts))

            # Calculate total overhead for all parts:
            # digits_for_current: digits used for current part numbers (1, 2, ..., k)
            digits_for_current = cumulative_digits
            # digits_for_total: digits used for total (k appears k times)
            digits_for_total = len(str(total_parts)) * total_parts
            # brackets_and_slashes: 3 chars per part for '<', '/', '>'
            brackets_and_slashes = 3 * total_parts

            # Check if we have enough space for the actual message content
            total_available_space = limit * total_parts
            total_overhead = digits_for_current + digits_for_total + brackets_and_slashes

            if total_available_space - total_overhead >= message_length:
                # We can split the message with k parts
                result = []
                current_index = 0

                # Create each part with its tag
                for part_number in range(1, total_parts + 1):
                    # Create the suffix tag <current/total>
                    tag = f'<{part_number}/{total_parts}>'

                    # Calculate how much message content fits in this part
                    content_length = limit - len(tag)

                    # Extract message content and append tag
                    part_with_tag = message[current_index : current_index + content_length] + tag
                    result.append(part_with_tag)

                    # Move to next position in message
                    current_index += content_length

                return result

        # No valid split found
        return []
```

## Time and Space Complexity

The time complexity is `O(n × log n)`, where `n` is the length of the string `message`.

The analysis breaks down as follows:
- The outer loop iterates from 1 to `n` in the worst case, contributing `O(n)` iterations
- Inside each iteration, we compute `len(str(k))` multiple times, which takes `O(log k)` time since the number of digits in `k` is proportional to `log k`
- When we find a valid `k`, we construct the answer by iterating through `k` parts, where each part involves string slicing and concatenation operations
- The string slicing `message[i : i + limit - len(tail)]` takes `O(limit)` time, and since `limit` is bounded by `n`, this is `O(n)` in the worst case
- However, the dominant factor is the outer loop with the logarithmic digit calculations, resulting in `O(n × log n)` overall complexity

The space complexity is `O(1)` when ignoring the space consumption of the answer array. The only auxiliary space used consists of:
- Variables `n`, `sa`, `sb`, `sc`, `i`, `j`, `k` which are all integers taking constant space
- The `tail` string which has a bounded size based on the number of digits in `k`, which is `O(log n)`
- Since `O(log n)` is considered negligible compared to the input size, the space complexity excluding the output is effectively `O(1)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrectly Handling Variable Suffix Lengths**

One of the most common mistakes is not accounting for the fact that suffix lengths change as the number of parts increases. For example:
- With 9 parts: suffixes are `<1/9>`, `<2/9>`, ..., `<9/9>` (all 5 characters)
- With 10 parts: suffixes are `<1/10>`, `<2/10>`, ..., `<9/10>`, `<10/10>` (5-7 characters)

**Pitfall Example:**
```python
# WRONG: Assuming fixed suffix length
def splitMessage_wrong(message, limit):
    n = len(message)
    for k in range(1, n + 1):
        suffix_len = len(f'<1/{k}>')  # Wrong! This assumes all suffixes have same length
        if (limit - suffix_len) * k >= n:
            # This will fail for cases where part numbers have different digit counts
            ...
```

**Solution:**
The correct approach tracks cumulative digits separately for part numbers and total count:
```python
cumulative_digits = 0  # Sum of digits for 1, 2, ..., k
for total_parts in range(1, n + 1):
    cumulative_digits += len(str(total_parts))  # Accumulate digits for each part number
    digits_for_total = len(str(total_parts)) * total_parts  # Total appears k times
```

### 2. **Off-by-One Errors in Space Calculation**

Another pitfall is making mistakes when calculating available space versus required space, especially with inequality conditions.

**Pitfall Example:**
```python
# WRONG: Using wrong inequality or forgetting edge cases
if total_space - overhead > message_length:  # Should be >= not >
    # This misses cases where space exactly equals message length
```

**Solution:**
Use the correct inequality that includes the boundary case:
```python
if total_available_space - total_overhead >= message_length:
    # Correctly handles when available space exactly matches message length
```

### 3. **Not Validating Individual Part Constraints**

A subtle pitfall is assuming that if the total space works out, each individual part will automatically satisfy the limit constraint. While the algorithm's math ensures this, forgetting to verify can lead to issues.

**Pitfall Example:**
```python
# WRONG: Not checking if any single suffix might exceed the limit
for k in range(1, n + 1):
    # What if len(f'<{k}/{k}>') > limit?
    # For example, if k=1000 and limit=5, the suffix alone is 11 characters!
```

**Solution:**
Add an early check to ensure the suffix itself doesn't exceed the limit:
```python
for total_parts in range(1, message_length + 1):
    # Early termination if suffix alone would exceed limit
    max_suffix_len = len(f'<{total_parts}/{total_parts}>')
    if max_suffix_len > limit:
        break  # No point checking further as suffixes will only get longer

    cumulative_digits += len(str(total_parts))
    # ... rest of the logic
```

### 4. **Integer Overflow in Large Calculations**

When dealing with very large messages or limits, multiplication operations might cause issues in some languages (though Python handles big integers well).

**Pitfall Example:**
```python
# In languages with fixed integer sizes, this might overflow:
total_space = limit * total_parts  # Could overflow for large values
```

**Solution:**
Be mindful of the scale of operations and add bounds checking if necessary:
```python
# Add reasonable bounds checking
if total_parts > message_length:  # Can't need more parts than characters
    break
if len(str(total_parts)) * 2 + 3 > limit:  # Minimum suffix size check
    break
```

