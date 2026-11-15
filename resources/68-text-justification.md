# 68. Text Justification

> Source: https://algo.monster/liteproblems/68

## Problem Statement

This problem asks you to format text like a word processor would, creating fully justified text (aligned on both left and right margins) with a specific line width.

Given an array of words and a maximum width `maxWidth`, you need to format the text so that:

1. **Each line contains exactly `maxWidth` characters** - no more, no less.

2. **Pack words greedily** - fit as many words as possible on each line. Words must be separated by at least one space.

3. **Distribute extra spaces evenly** - When a line has fewer characters than `maxWidth`, add extra spaces between words to reach the exact width. If the extra spaces don't divide evenly among the gaps between words, the leftmost gaps get one more space than the rightmost gaps.

4. **Special case for the last line** - The last line should be left-aligned (all words at the beginning with single spaces between them, and all extra spaces at the end).

5. **Special case for single-word lines** - If a line contains only one word, it should be left-aligned with all extra spaces at the end.

**Example walkthrough:**

If `words = ["This", "is", "an", "example"]` and `maxWidth = 16`:
- First line: "This" + "is" + "an" can fit (4 + 2 + 2 = 8 characters plus 2 spaces = 10 total)
- "example" does not fit (10 + 1 + 7 = 18 > 16), so the line holds three words
- The words use 8 characters, leaving 16 - 8 = 8 spaces for 2 gaps
- Each gap gets 4 spaces: `"This    is    an"`
- Second line: "example" fits alone, so left-align: `"example         "`

## Intuition

The key insight is to break this problem into two main phases: **line formation** and **space distribution**.

First, we need to figure out which words belong on each line. Since we want to pack as many words as possible (greedy approach), we keep adding words to the current line until adding one more would exceed `maxWidth`. We need to account for the minimum required spaces between words - if we have `k` words on a line, we need at least `k-1` spaces between them.

Once we know which words go on a line, we face the challenge of distributing spaces. If a line has total character count `cnt` (including minimum spaces) and we need to reach `maxWidth`, then we have `maxWidth - cnt` extra spaces to distribute among the gaps between words.

The clever part is using division with remainder to distribute these spaces evenly. If we have `gaps` number of gaps between words and `extra_spaces` to distribute, then:
- Each gap gets at least `extra_spaces // gaps` spaces
- The first `extra_spaces % gaps` gaps get one additional space

For example, if we need to distribute 7 extra spaces among 3 gaps:
- `7 // 3 = 2` spaces per gap minimum
- `7 % 3 = 1` remaining space
- So the gaps get: 3, 2, 2 spaces respectively

The special cases (last line and single-word lines) are simpler - we just left-align by joining words with single spaces and padding the right side with the remaining spaces.

This approach naturally handles all requirements: we maximize words per line through greedy packing, we achieve exact width through calculated space distribution, and we handle edge cases with conditional logic. The simulation approach works because we're essentially mimicking what a text editor would do when justifying text.

## Solution Approach

The implementation follows a simulation approach where we process words sequentially and build justified lines one by one.

**Main Loop Structure:**
We use an index `i` to traverse through the `words` array. For each iteration, we build one complete line of text.

**Line Formation Phase:**
1. Start with an empty list `t` to hold words for the current line
2. Initialize `cnt` with the length of the first word
3. Keep adding words while checking: `cnt + 1 + len(words[i]) <= maxWidth`
   - The `+1` accounts for the minimum space needed before the next word
4. Update `cnt` by adding 1 (for space) plus the length of the new word

**Space Distribution Phase:**
Once we have all words for a line, we handle three cases:

**Case 1: Last line or single-word line** (`i == n or len(t) == 1`):
- Simply join words with single spaces: `' '.join(t)`
- Pad the right with spaces: `' ' * (maxWidth - len(left))`
- Append to result

**Case 2: Regular line with multiple words:**
- Calculate total extra spaces needed: `space_width = maxWidth - (cnt - len(t) + 1)`
  - `cnt` includes minimum spaces between words
  - We subtract `len(t) - 1` to get just the character count
  - The difference from `maxWidth` gives us extra spaces to distribute

- Divide extra spaces among gaps: `w, m = divmod(space_width, len(t) - 1)`
  - `w`: base number of spaces each gap gets
  - `m`: number of leftmost gaps that get one extra space

- Build the justified line:
  ```python
  for j, s in enumerate(t[:-1]):
      row.append(s)
      row.append(' ' * (w + (1 if j < m else 0)))
  row.append(t[-1])
  ```
  - Add each word (except the last)
  - Add `w` spaces, plus 1 extra if this is one of the first `m` gaps
  - Finally add the last word

**Data Structures:**
- `ans`: List to store the final justified lines
- `t`: Temporary list to collect words for the current line
- `row`: List to build the justified string with proper spacing

The algorithm runs in `O(n)` time where `n` is the total number of words, as each word is processed exactly once. The space complexity is `O(m)` where `m` is the total length of the output.

## Example Walkthrough

Let's walk through a small example with `words = ["The", "quick", "brown"]` and `maxWidth = 10`.

**Building Line 1:**
- Start with `i = 0`, empty line `t = []`
- Add "The" (3 chars): `t = ["The"]`, `cnt = 3`
- Check next word "quick": `3 + 1 + 5 = 9 ≤ 10` ✓
- Add "quick": `t = ["The", "quick"]`, `cnt = 9`
- Check next word "brown": `9 + 1 + 5 = 15 > 10` ✗
- Stop here, line 1 contains: ["The", "quick"]

**Justifying Line 1:**
- We have 2 words with total `cnt = 9` (including 1 minimum space)
- Character count only: `3 + 5 = 8`
- Extra spaces needed: `10 - 8 = 2`
- Number of gaps: `2 - 1 = 1`
- Distribute: `2 // 1 = 2` spaces per gap, `2 % 1 = 0` remainder
- Result: `"The" + "  " + "quick" = "The  quick"`

**Building Line 2:**
- Start with `i = 2`, empty line `t = []`
- Add "brown" (5 chars): `t = ["brown"]`, `cnt = 5`
- No more words, `i = 3 = n`

**Justifying Line 2 (Last Line):**
- This is the last line, so left-align
- Join with single spaces: `"brown"`
- Pad right: `10 - 5 = 5` spaces
- Result: `"brown     "`

**Final Output:**
```
["The  quick",
 "brown     "]
```

Each line has exactly 10 characters. The first line distributes extra spaces evenly between words, while the last line is left-aligned with trailing spaces.

## Python Solution

```python
class Solution:
    def fullJustify(self, words: List[str], maxWidth: int) -> List[str]:
        """
        Formats text with full justification where each line has exactly maxWidth characters.
      
        Args:
            words: List of words to be justified
            maxWidth: Maximum width of each line
          
        Returns:
            List of fully justified text lines
        """
        result = []
        word_index = 0
        total_words = len(words)
      
        # Process words until all are placed in lines
        while word_index < total_words:
            # Collect words that fit in the current line
            current_line_words = []
            current_line_length = len(words[word_index])
            current_line_words.append(words[word_index])
            word_index += 1
          
            # Keep adding words while they fit (accounting for minimum one space between words)
            while word_index < total_words and current_line_length + 1 + len(words[word_index]) <= maxWidth:
                current_line_length += 1 + len(words[word_index])
                current_line_words.append(words[word_index])
                word_index += 1
          
            # Handle last line or single-word line (left-justified)
            if word_index == total_words or len(current_line_words) == 1:
                # Join words with single spaces and pad right with spaces
                left_justified = ' '.join(current_line_words)
                right_padding = ' ' * (maxWidth - len(left_justified))
                result.append(left_justified + right_padding)
                continue
          
            # Calculate space distribution for full justification
            # Total spaces needed = maxWidth - sum of word lengths
            total_spaces = maxWidth - (current_line_length - len(current_line_words) + 1)
          
            # Distribute spaces evenly between words
            gaps_between_words = len(current_line_words) - 1
            base_spaces_per_gap, extra_spaces = divmod(total_spaces, gaps_between_words)
          
            # Build the justified line
            justified_line = []
            for word_position, word in enumerate(current_line_words[:-1]):
                justified_line.append(word)
                # Add base spaces plus one extra space for the first 'extra_spaces' gaps
                spaces_to_add = base_spaces_per_gap + (1 if word_position < extra_spaces else 0)
                justified_line.append(' ' * spaces_to_add)
          
            # Add the last word (no spaces after it)
            justified_line.append(current_line_words[-1])
            result.append(''.join(justified_line))
          
        return result
```

## Time and Space Complexity

**Time Complexity:** `O(L)` where `L` is the sum of the lengths of all words.

The algorithm iterates through each word exactly once using the outer while loop with index `i`. For each word, we:
- Add it to the current line (constant time operations)
- Calculate the length (accessing already known lengths)
- When a line is complete, we format it by iterating through the words in that line once

Even though there are nested loops (the inner while loop and the formatting loop), each word is processed a constant number of times:
- Once when added to a line
- Once when the line is formatted

Therefore, the total time is proportional to the number of words and their lengths, giving us `O(L)`.

**Space Complexity:** `O(L)` where `L` is the sum of the lengths of all words.

The space is used for:
- The output list `ans` which stores all formatted lines. In the worst case, this contains all the input words plus the necessary spaces, which is bounded by `O(L)`
- The temporary list `t` which holds words for the current line being processed. This is at most `O(maxWidth)` which is typically less than `O(L)`
- The `row` list used for formatting, which is also bounded by the current line's content

Since the output must contain all the input words, the space complexity is `O(L)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Space Calculation Formula**
One of the most common mistakes is incorrectly calculating the number of extra spaces to distribute. Many developers confuse the total character count with the space calculation.

**Wrong approach:**
```python
# Incorrect: Using current_line_length directly
total_spaces = maxWidth - current_line_length
```

**Why it's wrong:** The `current_line_length` already includes minimum spaces between words. When we want to calculate extra spaces to distribute, we need to first remove those minimum spaces from consideration.

**Correct approach:**
```python
# First, get the total character count of words only
word_chars = sum(len(word) for word in current_line_words)
# Or equivalently: current_line_length - (len(current_line_words) - 1)

# Then calculate total spaces needed
total_spaces = maxWidth - word_chars

# Finally, distribute among gaps
gaps = len(current_line_words) - 1
spaces_per_gap, extra_spaces = divmod(total_spaces, gaps)
```

### 2. **Off-by-One Error in Extra Space Distribution**
Another frequent mistake is incorrectly determining which gaps should receive extra spaces when they don't divide evenly.

**Wrong approach:**
```python
# Incorrect: Using <= instead of <
spaces_to_add = base_spaces + (1 if word_position <= extra_spaces else 0)
```

**Why it's wrong:** If we have 3 extra spaces and 4 gaps (positions 0, 1, 2, 3), using `<=` would distribute extra spaces to positions 0, 1, 2, AND 3 (4 spaces total), which is one too many.

**Correct approach:**
```python
# Correct: First 'extra_spaces' gaps get one additional space
spaces_to_add = base_spaces + (1 if word_position < extra_spaces else 0)
```

### 3. **Forgetting to Handle Edge Cases Properly**
The problem has two critical edge cases that are easy to miss or handle incorrectly:

**Missing edge case handling:**
```python
# Wrong: Only checking for last line
if word_index == total_words:
    # Left justify
```

**Why it's wrong:** Single-word lines that aren't the last line also need to be left-justified, not center or right-justified.

**Correct approach:**
```python
# Check for BOTH last line AND single-word lines
if word_index == total_words or len(current_line_words) == 1:
    left_justified = ' '.join(current_line_words)
    result.append(left_justified + ' ' * (maxWidth - len(left_justified)))
```

### 4. **Division by Zero When Line Has Only One Word**
If not handled properly, attempting to distribute spaces when there's only one word can cause a division by zero error.

**Wrong approach:**
```python
# Dangerous: No check for single word case
gaps = len(current_line_words) - 1  # Could be 0!
spaces_per_gap = total_spaces // gaps  # Division by zero!
```

**Correct approach:**
```python
# Handle single word case before space distribution
if len(current_line_words) == 1:
    # Left justify with padding
    result.append(current_line_words[0] + ' ' * (maxWidth - len(current_line_words[0])))
else:
    # Safe to divide by gaps now
    gaps = len(current_line_words) - 1
    spaces_per_gap, extra_spaces = divmod(total_spaces, gaps)
```

