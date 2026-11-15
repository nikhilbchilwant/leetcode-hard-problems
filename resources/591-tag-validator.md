# 591. Tag Validator

> Source: https://algo.monster/liteproblems/591

## Problem Statement

This problem asks you to implement a tag validator that checks if a given code snippet string follows specific XML-like formatting rules.

The code snippet is considered valid if:

1. **The entire code must be wrapped in a valid closed tag** - meaning it starts with `<TAG_NAME>` and ends with `</TAG_NAME>` where both tag names match.

2. **Tag names must follow strict rules**:
   - Only contain uppercase letters (A-Z)
   - Length must be between 1 and 9 characters inclusive

3. **Tag content rules**:
   - Can contain other valid nested tags
   - Can contain CDATA sections
   - Can contain any regular characters
   - Cannot contain unmatched `<` symbols
   - Cannot contain unmatched or improperly closed tags

4. **CDATA sections** have the format `<![CDATA[CDATA_CONTENT]]>`:
   - Everything between `<![CDATA[` and the first `]]>` is treated as raw text
   - Characters inside CDATA are not parsed as tags, even if they look like tags

5. **Matching rules**:
   - Every start tag `<TAG_NAME>` must have a corresponding end tag `</TAG_NAME>`
   - Tags must be properly nested (like balanced parentheses)
   - When a `<` or `</` is encountered, all characters until the next `>` are parsed as a tag name

The solution uses a [stack](/problems/stack_intro)-based approach to track opening tags and ensure they match with closing tags. It processes the string character by character, handling three main cases:
- CDATA sections (which are skipped over)
- Closing tags (which must match the most recent opening tag on the stack)
- Opening tags (which are pushed onto the stack)

The code is valid only if after processing the entire string, the [stack](/problems/stack_intro) is empty (all tags were properly closed) and there was always at least one tag on the stack during processing (except at the very beginning).

## Intuition

The key insight is recognizing that this is fundamentally a **bracket matching problem** with additional constraints. Just like checking if parentheses are balanced, we need to ensure every opening tag has a corresponding closing tag in the correct order.

Why use a [stack](/problems/stack_intro)? When we encounter nested tags like `<A><B></B></A>`, we need to remember which tags are still open. The most recently opened tag must be closed first (LIFO - Last In First Out), which naturally leads us to use a stack data structure.

The parsing strategy emerges from understanding that we need to handle three distinct patterns while scanning the string:
1. **CDATA sections** - These are special because everything inside them should be ignored. We need to identify them first and skip over their content entirely.
2. **Closing tags** (`</...>`) - These should match and pop from our [stack](/problems/stack_intro) of open tags.
3. **Opening tags** (`<...>`) - These should be validated and pushed onto the stack.

A critical observation is that **the entire code must be wrapped in at least one valid tag**. This means:
- We should never have an empty [stack](/problems/stack_intro) while processing (except at the very start)
- The stack should be empty at the end (all tags properly closed)

The linear scanning approach makes sense because we need to process characters in order to maintain the correct nesting structure. We can't jump around - we must respect the sequential nature of opening and closing tags.

The validation rules for tag names (uppercase only, length 1-9) are straightforward checks that can be done as we parse each tag. If any tag fails these checks, we can immediately return `false`.

By combining [stack](/problems/stack_intro)-based tracking with careful string parsing and validation checks, we arrive at a solution that handles all the edge cases: nested tags, CDATA sections, invalid tag names, and unmatched tags.

**Learn more about [Stack](/problems/stack_intro) patterns.**

## Solution Approach

The solution uses a **[stack](/problems/stack_intro)-based parsing approach** with linear scanning to validate the XML-like code structure.

**Key Data Structure:**
- `stk`: A [stack](/problems/stack_intro) (list) to keep track of currently open tag names

**Main Algorithm Flow:**

1. **Initialize variables:**
   - `i` as the current position pointer (starts at 0)
   - `n` as the length of the code string
   - Empty [stack](/problems/stack_intro) `stk` for tracking open tags

2. **Main parsing loop** (`while i < n`):
 
   First, we perform a crucial check: `if i and not stk`. If we've started processing (`i > 0`) but the [stack](/problems/stack_intro) is empty, it means we're outside any valid tag wrapper, making the code invalid.

3. **Handle three types of patterns in order:**

   **a) CDATA Section Detection** (`<![CDATA[`):
   - Check if current position starts with `<![CDATA[`
   - Find the closing `]]>` using `code.find(']]>', i + 9)`
   - If not found (returns -1), the CDATA is unclosed - return `False`
   - Skip to the position after `]]>` by setting `i += 2` after finding the end

   **b) Closing Tag** (`</`):
   - Detect closing tag by checking for `</` prefix
   - Extract tag name between `</` and `>`
   - Validate the tag name using `check()` function
   - Verify it matches the top of the [stack](/problems/stack_intro): `stk.pop() != t`
   - If validation fails or stack is empty or names don't match, return `False`

   **c) Opening Tag** (`<`):
   - Detect opening tag by single `<` character
   - Extract tag name between `<` and `>`
   - Validate tag name with `check()` function
   - If valid, push the tag name onto the [stack](/problems/stack_intro): `stk.append(t)`

4. **Tag Name Validation** (`check()` function):
   - Verifies length is between 1 and 9: `1 <= len(tag) <= 9`
   - Ensures all characters are uppercase: `all(c.isupper() for c in tag)`

5. **Increment position** (`i += 1`) after processing each pattern

6. **Final validation:**
   - Return `not stk` - the [stack](/problems/stack_intro) must be empty (all tags closed)

**Why this approach works:**

- The order of pattern checking matters: CDATA first (longest match), then closing tags, then opening tags
- The [stack](/problems/stack_intro) ensures proper nesting - tags must be closed in reverse order of opening
- The position tracking with `find()` efficiently locates closing markers
- Early termination on any validation failure optimizes performance
- The empty stack check during processing ensures the entire code is wrapped in tags

This implementation handles all edge cases: unmatched tags, invalid tag names, improperly nested tags, and CDATA sections containing tag-like content.

## Example Walkthrough

Let's trace through the algorithm with the input: `"<A>text<![CDATA[<B>]]></A>"`

**Initial State:**
- `code = "<A>text<![CDATA[<B>]]></A>"`
- `i = 0`, `n = 26`, `stk = []`

**Step 1: Position 0 (`<A>`)**
- Check: `i and not stk` → `0 and []` = False (continue)
- Not CDATA (doesn't start with `<![CDATA[`)
- Not closing tag (doesn't start with `</`)
- Is opening tag (starts with `<`)
- Find `>` at position 2
- Extract tag name: `"A"`
- Validate: length=1 ✓, uppercase ✓
- Push to stack: `stk = ["A"]`
- Move to position 3

**Step 2: Positions 3-6 (`text`)**
- For each character in "text":
  - Check: `i and not stk` → stack not empty ✓
  - Not `<`, so increment `i`
- Move through positions 3→4→5→6→7

**Step 3: Position 7 (`<![CDATA[<B>]]>`)**
- Check: `i and not stk` → stack not empty ✓
- Is CDATA! (starts with `<![CDATA[`)
- Find `]]>` starting from position 16 (i+9)
- Found at position 19
- Set `i = 19 + 2 = 21` (the last `>` of `]]>`), then the loop's `i += 1` moves to 22

**Step 4: Position 22 (`</A>`)**
- Check: `i and not stk` → stack not empty ✓
- Not CDATA
- Is closing tag (starts with `</`)
- Find `>` at position 25
- Extract tag name: `"A"`
- Validate: length=1 ✓, uppercase ✓
- Pop from stack: `"A"` (matches!) 
- Stack now empty: `stk = []`
- Set `i = 25` (the `>`), then `i += 1` moves to 26

**Step 5: Loop ends**
- `i = 26`, which equals `n`, so the loop exits before the `i and not stk` check runs again
- Final check: `not stk` → `not []` = True ✓

**Result: Valid!** The code is properly wrapped in `<A>` tags, and the CDATA section was correctly skipped over, preventing `<B>` from being parsed as a tag.

**Key observations from this walkthrough:**
1. The CDATA section protects `<B>` from being parsed as a tag
2. The stack tracks that we're inside the `<A>` tag throughout processing
3. The algorithm correctly jumps over the entire CDATA content
4. The final empty stack confirms all tags were properly matched

## Python Solution

```python
class Solution:
    def isValid(self, code: str) -> bool:
        """
        Validates if the given code string follows proper tag format rules.
        Rules:
        - Tags must be 1-9 characters long and contain only uppercase letters
        - Tags must be properly nested and closed
        - CDATA sections are handled specially
        - The entire code must be wrapped in a tag
        """
      
        def is_valid_tag(tag: str) -> bool:
            """Check if a tag name is valid (1-9 uppercase letters)"""
            return 1 <= len(tag) <= 9 and all(char.isupper() for char in tag)
      
        tag_stack = []  # Stack to track open tags
        index = 0
        code_length = len(code)
      
        while index < code_length:
            # If we're not at the start and stack is empty, code is not wrapped in tags
            if index > 0 and not tag_stack:
                return False
          
            # Check for CDATA section: <![CDATA[...]]>
            if code[index:index + 9] == '<![CDATA[':
                # Find the closing ]]> for CDATA
                closing_index = code.find(']]>', index + 9)
                if closing_index < 0:
                    return False  # CDATA section not properly closed
                index = closing_index + 2  # Move past ]]>
              
            # Check for closing tag: </TAG>
            elif code[index:index + 2] == '</':
                start_tag_name = index + 2
                closing_bracket = code.find('>', start_tag_name)
                if closing_bracket < 0:
                    return False  # No closing bracket found
              
                tag_name = code[start_tag_name:closing_bracket]
                # Validate closing tag matches the most recent opening tag
                if not is_valid_tag(tag_name) or not tag_stack or tag_stack.pop() != tag_name:
                    return False
                index = closing_bracket
              
            # Check for opening tag: <TAG>
            elif code[index] == '<':
                start_tag_name = index + 1
                closing_bracket = code.find('>', start_tag_name)
                if closing_bracket < 0:
                    return False  # No closing bracket found
              
                tag_name = code[start_tag_name:closing_bracket]
                if not is_valid_tag(tag_name):
                    return False
                tag_stack.append(tag_name)  # Push opening tag to stack
                index = closing_bracket
          
            index += 1
      
        # All tags should be closed (stack should be empty)
        return not tag_stack
```

## Time and Space Complexity

**Time Complexity:** `O(n)`

The algorithm performs a single pass through the string with a while loop that iterates through each character position. Within each iteration:
- String slicing operations like `code[i:i+9]` and `code[i:i+2]` take `O(1)` time for fixed-length slices
- The `find()` method searches for patterns, but each character is visited at most a constant number of times throughout the entire execution
- The `check()` function validates tags with length up to 9, taking `O(1)` time since tag length is bounded
- Stack operations (`append()` and `pop()`) are `O(1)`

Although there are nested operations with `find()`, each character in the string is processed a bounded number of times, resulting in overall linear time complexity.

**Space Complexity:** `O(n)`

The space complexity is determined by:
- The stack `stk` which stores tag names. In the worst case, there could be `O(n/3)` nested tags (minimum tag is `<A>` with 3 characters), giving `O(n)` space
- String slicing creates temporary strings, but the maximum slice size is bounded by the tag length constraint (≤9 characters), contributing `O(1)` additional space
- The recursive depth is `O(1)` since `check()` is not recursive

Therefore, the dominant factor is the stack storage, resulting in `O(n)` space complexity.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Order of Pattern Matching**
One of the most critical pitfalls is checking for patterns in the wrong order. The code must check for CDATA sections BEFORE checking for closing tags `</`, and closing tags BEFORE opening tags `<`.

**Why this matters:**
- `<![CDATA[` starts with `<`, so if we check for opening tags first, we'd incorrectly try to parse `![CDATA[` as a tag name
- `</` starts with `<`, so if we check for opening tags before closing tags, we'd miss the closing tag pattern

**Incorrect approach:**
```python
if code[index] == '<':  # Checking opening tag first - WRONG!
    # This would catch CDATA and closing tags incorrectly
elif code[index:index + 2] == '</':
    # Never reached for closing tags
elif code[index:index + 9] == '<![CDATA[':
    # Never reached for CDATA
```

**Correct approach:**
```python
# Check patterns from most specific to least specific
if code[index:index + 9] == '<![CDATA[':  # CDATA first
    # Handle CDATA
elif code[index:index + 2] == '</':  # Then closing tags
    # Handle closing tag
elif code[index] == '<':  # Finally opening tags
    # Handle opening tag
```

### 2. **Not Validating That Entire Code is Wrapped in Tags**
A subtle but crucial requirement is that the ENTIRE code must be wrapped in at least one valid tag. Simply checking if the stack is empty at the end isn't sufficient.

**The problem:**
```python
# Input: "text<A></A>"
# This would pass if we only check stack at the end, but it's invalid
# because "text" appears outside any tag
```

**Solution:**
The code correctly handles this with:
```python
if index > 0 and not tag_stack:
    return False
```
This ensures that once we've started processing (index > 0), we're always inside at least one tag (stack is never empty during processing).

### 3. **Incorrectly Handling CDATA Content**
A common mistake is trying to parse content inside CDATA sections. Everything between `<![CDATA[` and `]]>` should be treated as raw text, even if it looks like tags.

**Example that could break incorrect implementations:**
```python
code = "<A><![CDATA[<B>]]></A>"
# The <B> inside CDATA should NOT be parsed as a tag
```

**Solution:**
Skip directly to the end of CDATA section:
```python
closing_index = code.find(']]>', index + 9)
index = closing_index + 2  # Jump past the entire CDATA content
```

### 4. **Off-by-One Errors in Index Management**
Managing the index correctly when jumping past processed sections is error-prone.

**Common mistakes:**
- After finding a closing `>`, forgetting to set index to the position of `>` (not past it)
- After CDATA, setting index to `closing_index + 3` instead of `closing_index + 2`

**Correct approach:**
```python
# For tags: set index to the position of '>'
index = closing_bracket
# The main loop will increment it by 1

# For CDATA: set index to position after ']]'
index = closing_index + 2  # Points to the last '>'
# The main loop will increment it by 1 to move past
```

### 5. **Not Handling Edge Cases in Tag Name Extraction**
When extracting tag names, ensure proper bounds checking:

**Potential issue:**
```python
# If code ends with "<" or "</" without closing ">"
closing_bracket = code.find('>', start_tag_name)
if closing_bracket < 0:  # Must check this!
    return False
```

Without this check, `code[start_tag_name:closing_bracket]` would use -1 as the end index, causing incorrect slicing behavior.

