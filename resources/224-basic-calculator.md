# 224. Basic Calculator

> Source: https://algo.monster/liteproblems/224

## Problem Statement

You need to implement a basic calculator that can evaluate a string expression containing:
- Non-negative integers
- Plus (`+`) and minus (`-`) operators  
- Opening (`(`) and closing (`)`) parentheses
- Spaces (which should be ignored)

The goal is to calculate and return the result of the mathematical expression represented by the string `s`.

For example:
- Input: `"1 + 1"` should return `2`
- Input: `"2-1 + 2"` should return `3`  
- Input: `"(1+(4+5+2)-3)+(6+8)"` should return `23`

The expression is guaranteed to be valid, meaning:
- Parentheses are properly matched
- The expression follows correct mathematical syntax
- Numbers are non-negative integers

You cannot use built-in functions like `eval()` that directly evaluate string expressions. You must implement the calculation logic yourself by parsing the string character by character and handling the operators and parentheses appropriately.

The main challenge is handling the parentheses correctly - when you encounter an opening parenthesis, you need to calculate the expression inside it first before continuing with the rest of the expression. Nested parentheses should also be handled properly.

## Intuition

When we evaluate a mathematical expression manually, we naturally process it from left to right, keeping track of the running sum and whether we're adding or subtracting the next number. The key insight is that we need to handle two main challenges: managing the current sign (`+` or `-`) and dealing with parentheses.

For the signs, we can maintain a `sign` variable that tells us whether to add (`+1`) or subtract (`-1`) the next number we encounter. When we see a number, we multiply it by the current sign and add it to our running answer.

The real complexity comes with parentheses. When we encounter an opening parenthesis `(`, we're essentially starting a new sub-expression that needs to be evaluated independently. But here's the crucial observation: we need to remember what we were doing before we entered the parentheses - what was our answer so far, and what sign should be applied to the result of the parentheses?

This naturally leads us to use a stack. When we see `(`, we save our current state (the answer so far and the sign that should be applied to the parentheses result) onto the stack. Then we reset our variables to start fresh for evaluating the expression inside the parentheses.

When we encounter `)`, we've finished evaluating the expression inside the parentheses. Now we need to combine this result with what we had before. We pop the sign and the previous answer from the stack. The sign tells us whether the parentheses result should be added or subtracted (remember, there could be a `-` before the `(`), and we combine it with the previous answer.

For example, in `5 - (2 + 3)`:
- We calculate `5` first
- When we see `- (`, we save `5` and `-1` (the sign) to the stack
- We calculate `2 + 3 = 5` inside the parentheses
- When we see `)`, we pop `-1` and `5`, then compute: `5 + (-1 * 5) = 0`

This stack-based approach elegantly handles nested parentheses too, as each level of nesting simply adds another pair of values to the stack.

## Solution Approach

We implement the calculator using a stack to handle parentheses and maintain the calculation state. Here's how the algorithm works:

**Data Structures:**
- `stk`: A stack to save the calculation state when entering parentheses
- `ans`: Accumulator for the current calculation result (initialized to 0)
- `sign`: Current sign for the next number (1 for positive, -1 for negative)
- `i`: Index pointer to traverse the string

**Algorithm Steps:**

1. **Initialize variables:** Set `ans = 0`, `sign = 1`, and create an empty stack.

2. **Traverse the string character by character:**

   - **If the character is a digit:** 
     - Use a while loop to read the complete multi-digit number
     - Build the number by: `x = x * 10 + int(s[j])`
     - Add the number to `ans` with the appropriate sign: `ans += sign * x`
     - Move the index to the last digit of the number

   - **If the character is `'+'`:** 
     - Set `sign = 1` for the next number

   - **If the character is `'-'`:** 
     - Set `sign = -1` for the next number

   - **If the character is `'('`:**
     - Push the current `ans` onto the stack (save the result before parentheses)
     - Push the current `sign` onto the stack (save the sign that should be applied to the parentheses result)
     - Reset `ans = 0` and `sign = 1` to start fresh for the expression inside parentheses

   - **If the character is `')'`:**
     - Pop twice from the stack:
       - First pop gives the sign before the parentheses
       - Second pop gives the answer calculated before the parentheses
     - Combine them: `ans = previous_sign * ans + previous_ans`
     - This applies the correct sign to the parentheses result and adds it to the previous calculation

   - **If the character is a space:** Simply skip it (increment index)

3. **Return the final result:** After processing all characters, `ans` contains the final result.

**Example Walkthrough for `"2-(3+4)"`:**
- Process `2`: `ans = 2`, `sign = 1`
- Process `-`: `sign = -1`
- Process `(`: Push `2` and `-1` to stack, reset `ans = 0`, `sign = 1`
- Process `3`: `ans = 3`
- Process `+`: `sign = 1`
- Process `4`: `ans = 3 + 4 = 7`
- Process `)`: Pop `-1` and `2`, calculate `ans = -1 * 7 + 2 = -5`
- Return `-5`

The time complexity is `O(n)` where `n` is the length of the string, as we process each character once. The space complexity is `O(n)` in the worst case for the stack when dealing with nested parentheses.

## Example Walkthrough

Let's walk through the expression `"8-(3+2)"` step by step to see how our stack-based solution works:

**Initial State:**
- `ans = 0`, `sign = 1`, `stack = []`, `i = 0`

**Step 1: Character '8' (digit)**
- Build number: `x = 8`
- Calculate: `ans = 0 + 1 * 8 = 8`
- Move to next character

**Step 2: Character '-' (minus operator)**
- Set `sign = -1` (next number/expression will be subtracted)
- State: `ans = 8`, `sign = -1`, `stack = []`

**Step 3: Character '(' (opening parenthesis)**
- Push current `ans` (8) onto stack
- Push current `sign` (-1) onto stack
- Reset for new sub-expression: `ans = 0`, `sign = 1`
- State: `ans = 0`, `sign = 1`, `stack = [8, -1]`

**Step 4: Character '3' (digit)**
- Build number: `x = 3`
- Calculate: `ans = 0 + 1 * 3 = 3`
- State: `ans = 3`, `sign = 1`, `stack = [8, -1]`

**Step 5: Character '+' (plus operator)**
- Set `sign = 1` (next number will be added)
- State: `ans = 3`, `sign = 1`, `stack = [8, -1]`

**Step 6: Character '2' (digit)**
- Build number: `x = 2`
- Calculate: `ans = 3 + 1 * 2 = 5`
- State: `ans = 5`, `sign = 1`, `stack = [8, -1]`

**Step 7: Character ')' (closing parenthesis)**
- Pop sign from stack: `previous_sign = -1`
- Pop answer from stack: `previous_ans = 8`
- Combine: `ans = previous_sign * ans + previous_ans = -1 * 5 + 8 = 3`
- State: `ans = 3`, `stack = []`

**Final Result: 3**

The key insight here is how the parentheses handling works: when we encounter `-(`, we save both the current result (8) and the sign (-1) that should be applied to whatever we calculate inside the parentheses. After evaluating `3+2=5` inside the parentheses, we apply the saved sign to get `-5` and add it to the saved result to get `8 + (-5) = 3`.

## Python Solution

```python
class Solution:
    def calculate(self, s: str) -> int:
        """
        Evaluates a mathematical expression string containing +, -, parentheses, and spaces.
        Uses a stack-based approach to handle nested parentheses.
      
        Args:
            s: Input string expression
          
        Returns:
            Integer result of the evaluated expression
        """
        stack = []
        current_result = 0
        current_sign = 1  # 1 for positive, -1 for negative
        index = 0
        length = len(s)
      
        while index < length:
            char = s[index]
          
            # Parse multi-digit numbers
            if char.isdigit():
                number = 0
                digit_start = index
              
                # Continue reading digits to form the complete number
                while digit_start < length and s[digit_start].isdigit():
                    number = number * 10 + int(s[digit_start])
                    digit_start += 1
              
                # Apply the current sign to the number and add to result
                current_result += current_sign * number
              
                # Adjust index to account for multi-digit parsing
                index = digit_start - 1
              
            # Handle addition operator
            elif char == "+":
                current_sign = 1
              
            # Handle subtraction operator
            elif char == "-":
                current_sign = -1
              
            # Handle opening parenthesis
            elif char == "(":
                # Save current state before entering new scope
                stack.append(current_result)
                stack.append(current_sign)
              
                # Reset for new expression inside parentheses
                current_result = 0
                current_sign = 1
              
            # Handle closing parenthesis
            elif char == ")":
                # Pop the sign before the parenthesis
                prev_sign = stack.pop()
                # Pop the result before the parenthesis
                prev_result = stack.pop()
              
                # Apply the sign to current result and combine with previous
                current_result = prev_sign * current_result + prev_result
          
            # Skip spaces implicitly (no action needed)
          
            index += 1
          
        return current_result
```

## Time and Space Complexity

**Time Complexity:** `O(n)`, where `n` is the length of the string `s`.

The algorithm performs a single pass through the string using a while loop that iterates from index `0` to `n-1`. For each character in the string:
- Digit parsing: When encountering a digit, the inner while loop extracts the complete number. Although there's a nested loop, each character is visited at most once across all iterations, maintaining `O(n)` complexity.
- Operators (`+`, `-`): Constant time operations to update the sign variable.
- Opening parenthesis `(`: Constant time operations to push two values onto the stack.
- Closing parenthesis `)`: Constant time operations to pop two values from the stack and perform arithmetic.
- Space characters are skipped in constant time.

Since each character is processed exactly once and all operations per character are `O(1)`, the total time complexity is `O(n)`.

**Space Complexity:** `O(n)`, where `n` is the length of the string `s`.

The space complexity is determined by the stack `stk` which stores intermediate results and signs when encountering parentheses. In the worst case, the string could contain nested parentheses like `((((1))))`, where the nesting depth is proportional to `n`. For each opening parenthesis, two values are pushed onto the stack (the current `ans` and `sign`), potentially requiring `O(n)` space in total. Additional variables (`ans`, `sign`, `i`, `j`, `x`) use constant space `O(1)`.

## Common Pitfalls

### 1. **Incorrect Multi-Digit Number Parsing**

**The Pitfall:** 
A common mistake is forgetting to adjust the main loop index after parsing a multi-digit number. When you use an inner loop to read all digits of a number, the inner loop advances past multiple characters. If you don't update the outer loop's index accordingly, the remaining digits are processed again. For `"12"`, the code adds 12 and then 2, returning 14.

**Problematic Code Example:**
```python
# WRONG - doesn't adjust outer loop index
while index < length:
    if char.isdigit():
        number = 0
        j = index
        while j < length and s[j].isdigit():
            number = number * 10 + int(s[j])
            j += 1
        current_result += current_sign * number
    index += 1  # Only moves one step: in "12", the "2" is read again as its own number
```

**The Solution:**
After parsing a multi-digit number, set the main loop index to point to the last digit that was processed:
```python
if char.isdigit():
    number = 0
    digit_start = index
    while digit_start < length and s[digit_start].isdigit():
        number = number * 10 + int(s[digit_start])
        digit_start += 1
    current_result += current_sign * number
    index = digit_start - 1  # Critical: adjust index to last processed digit
```

### 2. **Stack Order Confusion When Handling Closing Parenthesis**

**The Pitfall:**
When encountering a closing parenthesis, the order in which you pop from the stack matters. Since you push `current_result` first and then `current_sign`, you must pop in reverse order (LIFO). Getting this order wrong leads to incorrect calculations.

**Problematic Code Example:**
```python
# WRONG - incorrect pop order
elif char == ")":
    prev_result = stack.pop()  # This actually gets the sign!
    prev_sign = stack.pop()    # This actually gets the result!
    current_result = prev_sign * current_result + prev_result
```

**The Solution:**
Always pop in the reverse order of how you pushed:
```python
elif char == "(":
    stack.append(current_result)  # Push result first
    stack.append(current_sign)    # Push sign second
  
elif char == ")":
    prev_sign = stack.pop()        # Pop sign first (was pushed last)
    prev_result = stack.pop()      # Pop result second (was pushed first)
    current_result = prev_sign * current_result + prev_result
```

### 3. **Not Resetting State After Opening Parenthesis**

**The Pitfall:**
When entering a new parenthesis scope, forgetting to reset `current_result` and `current_sign` can cause the calculation inside parentheses to inherit values from outside, leading to wrong results.

**Problematic Code Example:**
```python
# WRONG - doesn't reset state
elif char == "(":
    stack.append(current_result)
    stack.append(current_sign)
    # Forgot to reset! The next number will be added to the old current_result
```

**The Solution:**
Always reset both the result accumulator and sign when starting a new parenthesis scope:
```python
elif char == "(":
    stack.append(current_result)
    stack.append(current_sign)
    current_result = 0  # Reset for fresh calculation
    current_sign = 1    # Reset to positive sign
```

These pitfalls are particularly tricky because they may work correctly for simple test cases but fail on more complex expressions with nested parentheses or multi-digit numbers.

