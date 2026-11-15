# 772. Basic Calculator III

> Source: https://algo.monster/liteproblems/772

## Problem Statement

You need to implement a calculator that can evaluate mathematical expressions given as strings. The expressions can contain:

- Non-negative integers (0, 1, 2, ...)
- Four basic arithmetic operators: `+`, `-`, `*`, `/`
- Parentheses: `(` and `)`

The calculator must follow standard mathematical rules:
- Operations inside parentheses are evaluated first
- Multiplication and division have higher precedence than addition and subtraction
- Operations of the same precedence are evaluated left to right
- Integer division should truncate toward zero (e.g., `5/2 = 2`, `-5/2 = -2`)

For example:
- `"2+3*4"` should return `14` (not 20, because multiplication happens first)
- `"(2+3)*4"` should return `20` (parentheses override normal precedence)
- `"10/3"` should return `3` (integer division truncates)

The solution uses a recursive depth-first search approach with a [stack](/problems/stack_intro). It processes the expression character by character, handling:
1. Building multi-digit numbers by accumulating digits
2. Recursively evaluating sub-expressions within parentheses
3. Applying operators based on their precedence - multiplication and division are applied immediately by modifying the last value on the stack, while addition and subtraction push new values onto the stack
4. Returning the sum of all values in the stack when a closing parenthesis is encountered or the expression ends

The key insight is that by applying `*` and `/` immediately but deferring `+` and `-` operations to the [stack](/problems/stack_intro), the solution naturally handles operator precedence without needing explicit precedence rules.

## Intuition

When evaluating mathematical expressions, we face two main challenges: handling operator precedence and dealing with parentheses. Let's think about how we naturally solve `"2+3*4"` on paper - we recognize that multiplication must happen before addition, giving us `2+12=14`.

The key insight is that we can handle precedence by treating operators differently. For addition and subtraction, we can defer the calculation by storing values on a [stack](/problems/stack_intro). For multiplication and division, we need to apply them immediately to respect their higher precedence.

Consider the expression `"2+3*4-5"`. As we scan through:
- See `2` and `+`: push `2` to [stack](/problems/stack_intro) → stack: `[2]`
- See `3` and `*`: push `3` to stack → stack: `[2, 3]`
- See `4` and `-`: since previous operator was `*`, we pop `3`, multiply by `4`, push back `12` → stack: `[2, 12]`
- See `5` and end: push `-5` to stack → stack: `[2, 12, -5]`
- Sum the stack: `2+12-5 = 9`

Notice how multiplication modifies the last value on the [stack](/problems/stack_intro) immediately, while addition/subtraction just add new values. This naturally preserves the correct order of operations.

For parentheses, we recognize that `(expression)` is essentially a sub-problem. When we encounter `(`, we can recursively evaluate everything inside until we find the matching `)`, treating that entire sub-expression as a single number. This recursive approach elegantly handles nested parentheses too.

The beauty of this solution is that by combining these two ideas - using a [stack](/problems/stack_intro) with immediate application of high-precedence operators and recursive evaluation of parentheses - we get a clean algorithm that handles all cases without explicitly tracking operator precedence levels or building expression trees.

**Learn more about [Stack](/problems/stack_intro), [Recursion](/problems/recursion_intro) and [Math](/problems/math-basics) patterns.**

## Solution Approach

The solution uses a recursive depth-first search (DFS) with a [stack](/problems/stack_intro)-based approach to handle operator precedence. Let's walk through the implementation:

**Data Structures:**
- A `deque` to efficiently process characters from left to right
- A [stack](/problems/stack_intro) (`stk`) to store intermediate results
- Variables to track the current number being built (`num`) and the previous operator (`sign`)

**Main Algorithm Flow:**

1. **Initialize State**: Start with `num = 0`, `sign = "+"`, and an empty [stack](/problems/stack_intro) `stk`.

2. **Process Characters One by One**:
   - If the character is a digit, build the multi-digit number: `num = num * 10 + int(c)`
   - If the character is `(`, recursively call `dfs()` to evaluate the sub-expression and treat the result as `num`
   - If the character is an operator (`+-*/`), closing parenthesis `)`, or we've reached the end of the expression, apply the previous operator

3. **Apply Operators Based on Precedence**:
   ```python
   match sign:
       case "+":
           stk.append(num)        # Defer addition
       case "-":
           stk.append(-num)       # Convert subtraction to adding negative
       case "*":
           stk.append(stk.pop() * num)  # Apply immediately
       case "/":
           stk.append(int(stk.pop() / num))  # Apply immediately with truncation
   ```

   The key insight: `+` and `-` push values to the [stack](/problems/stack_intro) for later summation, while `*` and `/` immediately modify the last stack value.

4. **Handle Parentheses**:
   - When encountering `(`, recursively evaluate the sub-expression
   - When encountering `)`, break the current loop and return the sum of the [stack](/problems/stack_intro)
   - This naturally handles nested parentheses through [recursion](/problems/recursion_intro)

5. **Final Result**: After processing all characters, return `sum(stk)` which gives the final result.

**Example Walkthrough** for `"2*(3+4)"`:
- Start with main expression
- Process `2`, then see `*`
- Push `2` to [stack](/problems/stack_intro), set `sign = "*"`
- Encounter `(`, recursively evaluate `"3+4)"`
  - In [recursion](/problems/recursion_intro): push `3`, then push `4`, hit `)`, return `7`
- Back in main: `num = 7`, apply `*`: pop `2`, push `2*7=14`
- Return sum of stack: `14`

The elegance lies in how the [stack](/problems/stack_intro) naturally maintains evaluation order while the recursive calls handle nested expressions seamlessly.

## Example Walkthrough

Let's trace through the expression `"6+2*3-4"` to see how the algorithm works:

**Initial State:**
- `num = 0`, `sign = "+"`, `stk = []`

**Step 1: Process '6'**
- Character is digit: `num = 6`
- Move to next character '+'

**Step 2: Process '+'**
- Character is operator, so apply previous sign ('+')
- Execute: `stk.append(6)` → `stk = [6]`
- Reset: `num = 0`, `sign = "+"`

**Step 3: Process '2'**
- Character is digit: `num = 2`
- Move to next character '*'

**Step 4: Process '*'**
- Character is operator, so apply previous sign ('+')
- Execute: `stk.append(2)` → `stk = [6, 2]`
- Reset: `num = 0`, `sign = "*"`

**Step 5: Process '3'**
- Character is digit: `num = 3`
- Move to next character '-'

**Step 6: Process '-'**
- Character is operator, so apply previous sign ('*')
- Execute: `stk.append(stk.pop() * 3)` → `stk.pop()` gives 2, then `2 * 3 = 6`
- Result: `stk = [6, 6]`
- Reset: `num = 0`, `sign = "-"`

**Step 7: Process '4'**
- Character is digit: `num = 4`
- Reached end of expression

**Step 8: End of Expression**
- Apply previous sign ('-')
- Execute: `stk.append(-4)` → `stk = [6, 6, -4]`

**Step 9: Calculate Final Result**
- Return `sum(stk) = 6 + 6 + (-4) = 8`

The key observation: multiplication was applied immediately when we encountered the '-' after '3', modifying the last stack value from 2 to 6. Addition and subtraction operations were deferred by pushing values to the stack, which we sum at the end. This ensures correct operator precedence without explicit priority rules.

## Python Solution

```python
from collections import deque
from typing import Deque

class Solution:
    def calculate(self, s: str) -> int:
        def evaluate_expression(char_queue: Deque[str]) -> int:
            """
            Recursively evaluate mathematical expression from a queue of characters.
            Handles +, -, *, / operations and parentheses.

            Args:
                char_queue: A deque containing characters of the expression

            Returns:
                The evaluated result as an integer
            """
            current_number = 0
            operator = "+"
            operand_stack = []

            while char_queue:
                char = char_queue.popleft()

                # Build multi-digit numbers
                if char.isdigit():
                    current_number = current_number * 10 + int(char)

                # Handle nested parentheses with recursion
                if char == "(":
                    current_number = evaluate_expression(char_queue)

                # Process operator or end of expression
                if char in "+-*/)" or not char_queue:
                    # Apply the previous operator with the current number
                    if operator == "+":
                        operand_stack.append(current_number)
                    elif operator == "-":
                        operand_stack.append(-current_number)
                    elif operator == "*":
                        operand_stack.append(operand_stack.pop() * current_number)
                    elif operator == "/":
                        # Integer division with truncation toward zero
                        operand_stack.append(int(operand_stack.pop() / current_number))

                    # Reset for next operation
                    current_number = 0
                    operator = char

                # Exit recursion when closing parenthesis is found
                if char == ")":
                    break

            # Sum all values in the stack to get final result
            return sum(operand_stack)

        # Convert string to deque and start evaluation
        return evaluate_expression(deque(s))
```

## Time and Space Complexity

**Time Complexity: `O(n)`**

The algorithm processes each character in the input string exactly once. The main operations performed are:
- Traversing through the deque with `q.popleft()` - each character is visited once
- Digit parsing with `num = num * 10 + int(c)` - `O(1)` per digit
- Stack operations (`append`, `pop`) - `O(1)` per operation
- The recursive call for parentheses doesn't add extra iterations, it just continues processing the same string
- The final `sum(stk)` operation is `O(k)` where `k` is the number of elements in the stack, but `k ≤ n`

Since each character is processed exactly once with constant-time operations, the overall time complexity is `O(n)` where `n` is the length of the input string.

**Space Complexity: `O(n)`**

The space usage comes from:
- The deque created from the input string - `O(n)`
- The stack `stk` used in each recursive call - in the worst case (all additions/subtractions), it can hold `O(n)` elements
- The recursive call stack depth - in the worst case with deeply nested parentheses like `((((1))))`, the depth can be `O(n)`

The dominant factor is the deque and the potential recursive depth, both of which can be `O(n)` in the worst case. Therefore, the overall space complexity is `O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Integer Division Truncation Issues**

**Pitfall:** Using `//` (floor division) instead of `int(a/b)` for integer division can give incorrect results for negative numbers.

```python
# Incorrect - floor division rounds down (toward negative infinity)
-5 // 2  # Returns -3 (wrong!)

# Correct - truncation toward zero
int(-5 / 2)  # Returns -2 (correct!)
```

**Why it matters:** The problem specifically requires truncation toward zero, not floor division. For negative results, these operations differ:
- Floor division: `-5 // 2 = -3` (rounds to negative infinity)
- Truncation: `int(-5 / 2) = -2` (rounds toward zero)

**Solution:** Always use `int(operand_stack.pop() / current_number)` for division operations.

### 2. **Skipping Whitespace with `continue`**

**Pitfall:** The problem's input contains no spaces, but a common adaptation for inputs with spaces is to skip them at the top of the loop:

```python
while char_queue:
    char = char_queue.popleft()

    # Skip whitespace (breaks the end-of-input check)
    if char == " ":
        continue

    # Rest of the logic...
```

This breaks the `not char_queue` check that applies the last pending operator. If the expression ends with a space, the final number is never pushed: `"3 "` returns `0` and `"2 + 3 * 4 "` returns `5` instead of `14`.

**Solution:** The current code needs no special case for spaces. A space is neither a digit nor an operator, so it is ignored, and if it is the last character the `not char_queue` branch still applies the pending operator. If you do skip spaces, strip them first with `s.replace(" ", "")` before building the deque.

### 3. **Empty Expression or Invalid Input**

**Pitfall:** Inputs such as `""`, `"*5"`, or `"()"` are outside the problem's constraints, which guarantee a valid expression. The current code does not raise an error on them; it returns `0` for all three, which is not a meaningful answer for `"*5"`.

```python
# Outside the constraints; the current code returns 0 for each
""          # Empty string
"*5"        # Starting with operator
"()"        # Empty parentheses
```

**Solution:** For the interview problem, no extra handling is needed. If the calculator must reject malformed input, validate the string before evaluating it, for example by checking that every operator has a number or parenthesized expression on both sides.

### 4. **Division by Zero**

**Pitfall:** The code doesn't check for division by zero, which will raise an exception.

```python
"5/0"       # Will raise ZeroDivisionError
"10/(3-3)"  # Also results in division by zero
```

**Solution:** Add a check before performing division:

```python
elif operator == "/":
    if current_number == 0:
        # Handle division by zero - could return 0, infinity, or raise custom error
        raise ValueError("Division by zero")
    operand_stack.append(int(operand_stack.pop() / current_number))
```

### 5. **Character Validation**

**Pitfall:** Invalid characters in the expression aren't handled and might cause unexpected behavior.

```python
"2+3a"      # Invalid character 'a'
"2++3"      # Consecutive operators
```

**Solution:** Add character validation:

```python
if char not in "0123456789+-*/() ":
    raise ValueError(f"Invalid character: {char}")
```

These pitfalls are common in expression evaluation problems and addressing them makes the solution more robust for real-world use cases.

