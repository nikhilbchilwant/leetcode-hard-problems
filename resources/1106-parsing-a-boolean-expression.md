# 1106. Parsing A Boolean Expression

> Source: https://algo.monster/liteproblems/1106

## Problem Statement

You are given a string representing a boolean expression that needs to be evaluated to either `true` or `false`.

The expression can contain the following elements:

- `'t'` - represents the boolean value `true`
- `'f'` - represents the boolean value `false`
- `'!(subExpr)'` - represents the logical NOT operation on the inner expression `subExpr`
- `'&(subExpr₁, subExpr₂, ..., subExprₙ)'` - represents the logical AND operation on n inner expressions (where n ≥ 1)
- `'|(subExpr₁, subExpr₂, ..., subExprₙ)'` - represents the logical OR operation on n inner expressions (where n ≥ 1)

Each operation follows standard boolean logic:
- NOT (`!`) flips the boolean value (true becomes false, false becomes true)
- AND (`&`) returns true only if all sub-expressions are true
- OR (`|`) returns true if at least one sub-expression is true

The sub-expressions within AND and OR operations are separated by commas and enclosed in parentheses. The expression is guaranteed to be valid and follow the given rules.

Your task is to parse and evaluate the given boolean expression string and return the final boolean result.

For example:
- `"t"` evaluates to `true`
- `"!(f)"` evaluates to `true` (NOT false = true)
- `"&(t,f)"` evaluates to `false` (true AND false = false)
- `"|(f,t)"` evaluates to `true` (false OR true = true)

## Intuition

When we look at this boolean expression, we notice it has a nested structure with operators and parentheses. This immediately suggests we need a way to handle the innermost expressions first before evaluating the outer ones - similar to how we evaluate arithmetic expressions with parentheses.

The key insight is that whenever we encounter a closing parenthesis `')'`, we've reached the end of a complete sub-expression that needs to be evaluated. At this point, we should evaluate everything between this closing parenthesis and its corresponding operator (which comes right before the opening parenthesis).

A [stack](/problems/stack_intro) is perfect for this because:
1. We can push characters as we encounter them
2. When we hit a `')'`, we can pop backwards to collect all the boolean values that need to be evaluated
3. The operator will be right after these boolean values (going backwards in the stack)

The evaluation process becomes straightforward:
- For `'!'`, we just need to flip the single boolean value
- For `'&'`, if there's any `'f'` among the values, the result is `'f'`, otherwise `'t'`
- For `'|'`, if there's any `'t'` among the values, the result is `'t'`, otherwise `'f'`

After evaluating a sub-expression, we replace it with its result (`'t'` or `'f'`) by pushing this result back onto the [stack](/problems/stack_intro). This way, the evaluated result can participate in outer expressions as a simple boolean value.

By processing the string character by character and evaluating sub-expressions as we complete them, we gradually simplify the entire expression until we're left with a single boolean value on the [stack](/problems/stack_intro) - our final answer.

**Learn more about [Stack](/problems/stack_intro) and [Recursion](/problems/recursion_intro) patterns.**

## Solution Approach

We implement the solution using a [stack](/problems/stack_intro)-based approach to parse and evaluate the boolean expression from left to right.

**Algorithm Steps:**

1. **Initialize a [stack](/problems/stack_intro)** to store characters as we traverse the expression.

2. **Traverse each character `c` in the expression:**
   - If `c` is one of `'t'`, `'f'`, `'!'`, `'&'`, or `'|'`, push it directly onto the [stack](/problems/stack_intro)
   - If `c` is a comma `','` or opening parenthesis `'('`, skip it (we don't need them for evaluation)
   - If `c` is a closing parenthesis `')'`, it signals the end of a sub-expression that needs evaluation

3. **When encountering `')'`, evaluate the sub-expression:**
   - Initialize counters: `t = 0` and `f = 0` to count true and false values
   - Pop elements from the [stack](/problems/stack_intro) while the top element is either `'t'` or `'f'`:
     - If popped element is `'t'`, increment `t`
     - If popped element is `'f'`, increment `f`
   - After collecting all boolean values, pop the operator from the stack
   - Based on the operator, compute the result:
     - For `'!'`: Result is `'t'` if `f > 0` (NOT false = true), else `'f'`
     - For `'&'`: Result is `'f'` if `f > 0` (any false makes AND false), else `'t'`
     - For `'|'`: Result is `'t'` if `t > 0` (any true makes OR true), else `'f'`
   - Push the computed result back onto the stack

4. **After processing the entire expression**, only one character remains in the [stack](/problems/stack_intro) - this is our final result. Return `True` if it's `'t'`, otherwise `False`.

**Example Walkthrough:**

For expression `"&(t,!(f))"`：
- Push `'&'` → [stack](/problems/stack_intro): `['&']`
- Skip `'('`
- Push `'t'` → stack: `['&', 't']`
- Skip `','`
- Push `'!'` → stack: `['&', 't', '!']`
- Skip `'('`
- Push `'f'` → stack: `['&', 't', '!', 'f']`
- Encounter first `')'`: 
  - Pop `'f'` (f=1)
  - Pop `'!'` (operator)
  - Evaluate: NOT false = true, push `'t'`
  - Stack: `['&', 't', 't']`
- Encounter second `')'`:
  - Pop `'t'`, `'t'` (t=2)
  - Pop `'&'` (operator)
  - Evaluate: true AND true = true, push `'t'`
  - Stack: `['t']`
- Return `True` since stack[0] is `'t'`

The time complexity is `O(n)` where n is the length of the expression, as we process each character once. The space complexity is also `O(n)` for the [stack](/problems/stack_intro) storage.

## Example Walkthrough

Let's walk through the expression `"|(f,&(t,f))"` step by step to see how our stack-based solution works.

**Initial expression:** `"|(f,&(t,f))"`

**Step-by-step execution:**

1. **Character `'|'`**: Push onto stack
   - Stack: `['|']`

2. **Character `'('`**: Skip (we don't need opening parentheses)
   - Stack: `['|']`

3. **Character `'f'`**: Push onto stack
   - Stack: `['|', 'f']`

4. **Character `','`**: Skip (we don't need commas)
   - Stack: `['|', 'f']`

5. **Character `'&'`**: Push onto stack
   - Stack: `['|', 'f', '&']`

6. **Character `'('`**: Skip
   - Stack: `['|', 'f', '&']`

7. **Character `'t'`**: Push onto stack
   - Stack: `['|', 'f', '&', 't']`

8. **Character `','`**: Skip
   - Stack: `['|', 'f', '&', 't']`

9. **Character `'f'`**: Push onto stack
   - Stack: `['|', 'f', '&', 't', 'f']`

10. **Character `')'`**: Time to evaluate the inner expression!
    - Pop `'f'` → f = 1
    - Pop `'t'` → t = 1
    - Pop `'&'` (operator)
    - Evaluate: `&(t,f)` = true AND false = false
    - Push result `'f'` back
    - Stack: `['|', 'f', 'f']`

11. **Character `')'`**: Time to evaluate the outer expression!
    - Pop `'f'` → f = 1
    - Pop `'f'` → f = 2
    - Pop `'|'` (operator)
    - Evaluate: `|(f,f)` = false OR false = false
    - Push result `'f'` back
    - Stack: `['f']`

**Final Result:** The stack contains `'f'`, so we return `False`.

This example demonstrates how:
- We process characters left to right
- Closing parentheses trigger evaluation of sub-expressions
- Inner expressions are evaluated first (the `&(t,f)` before the outer `|`)
- Each evaluation replaces a complex sub-expression with a simple boolean value
- The process continues until we have a single boolean result

## Python Solution

```python
class Solution:
    def parseBoolExpr(self, expression: str) -> bool:
        """
        Evaluates a boolean expression string containing operators and boolean values.
      
        Args:
            expression: A string containing boolean expression with:
                       - 't' for true, 'f' for false
                       - '!' for NOT, '&' for AND, '|' for OR
                       - Parentheses for grouping
      
        Returns:
            Boolean result of the evaluated expression
        """
        stack = []
      
        for char in expression:
            # Push operators and boolean values onto the stack
            if char in 'tf!&|':
                stack.append(char)
          
            # When we encounter a closing parenthesis, evaluate the sub-expression
            elif char == ')':
                # Count true and false values in the current sub-expression
                true_count = 0
                false_count = 0
              
                # Pop all boolean values until we reach an operator
                while stack[-1] in 'tf':
                    if stack[-1] == 't':
                        true_count += 1
                    else:  # stack[-1] == 'f'
                        false_count += 1
                    stack.pop()
              
                # Pop the operator and evaluate based on its type
                operator = stack.pop()
              
                # Evaluate based on the operator type
                if operator == '!':
                    # NOT operator: returns opposite of the single operand
                    result = 't' if false_count > 0 else 'f'
                elif operator == '&':
                    # AND operator: returns false if any operand is false
                    result = 'f' if false_count > 0 else 't'
                elif operator == '|':
                    # OR operator: returns true if any operand is true
                    result = 't' if true_count > 0 else 'f'
              
                # Push the result back onto the stack
                stack.append(result)
          
            # Skip opening parentheses and commas (they don't affect evaluation)
            # These characters are implicitly ignored by not handling them
      
        # The final result is the only element left in the stack
        return stack[0] == 't'
```

## Time and Space Complexity

**Time Complexity:** `O(n)`, where `n` is the length of the expression string.

The algorithm iterates through each character in the expression exactly once. For each character:
- Characters `'t'`, `'f'`, `'!'`, `'&'`, `'|'` are pushed to the stack in `O(1)` time
- Character `')'` triggers evaluation where we pop elements from the stack. Each `'t'` or `'f'` character is pushed once and popped once throughout the entire execution
- The evaluation operations (counting `t` and `f`, and determining the result) are all `O(1)` per character processed

Since each character is processed at most twice (once when pushed, once when popped during evaluation), the total time complexity is `O(n)`.

**Space Complexity:** `O(n)`, where `n` is the length of the expression string.

The space is used by the stack `stk`. In the worst case, the stack could contain:
- All operands and operators before encountering any closing parenthesis
- For deeply nested expressions like `&(&(&(...)))`, the stack grows proportionally to the input size
- The maximum stack size is bounded by the number of characters in the expression (excluding commas and opening parentheses which aren't stored)

Therefore, the space complexity is `O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Operator Precedence Assumption**
A common mistake is assuming that operators need to be evaluated in a specific order or that nested expressions require special handling. Developers might overcomplicate the solution by trying to build an expression tree or implement operator precedence rules.

**Why it happens:** In traditional expression evaluation, we often deal with operator precedence (e.g., multiplication before addition). Here, the parentheses explicitly define the evaluation order, making the problem simpler.

**Solution:** Trust the stack-based approach. The closing parenthesis naturally triggers evaluation at the right time, handling nesting automatically without explicit precedence rules.

### 2. **Forgetting to Handle Single-Operand Cases**
Some developers might assume that AND and OR operations always have multiple operands, but the problem states n ≥ 1, meaning single operands are valid:
- `&(t)` should return `true`
- `|(f)` should return `false`

**Why it happens:** Test cases often show multiple operands, leading to assumptions about minimum operand counts.

**Solution:** The counting approach handles this naturally - if there's only one `'t'` and no `'f'` values, AND returns true; if there's only one `'f'` and no `'t'` values, OR returns false.

### 3. **Stack Boundary Check Errors**
When popping from the stack, forgetting to check if the element exists or using incorrect conditions can cause IndexError or infinite loops.

**Incorrect approach:**
```python
while stack and stack[-1] != '(' :  # Wrong! Looking for opening parenthesis
    if stack[-1] == 't':
        true_count += 1
    stack.pop()
```

**Why it happens:** Confusion about what's actually on the stack - we never push opening parentheses, so we shouldn't look for them.

**Solution:** Check for boolean values specifically:
```python
while stack[-1] in 'tf':  # Correct - only pop boolean values
    if stack[-1] == 't':
        true_count += 1
    else:
        false_count += 1
    stack.pop()
```

### 4. **Mishandling the NOT Operator**
The NOT operator always has exactly one operand, but developers might:
- Try to apply NOT to multiple values
- Forget that `!(expression)` can have a complex expression inside, not just a single boolean

**Why it happens:** The NOT operator seems simple, but `!(expression)` evaluates the entire expression first, then negates it.

**Solution:** The stack approach handles this correctly - by the time we process the closing parenthesis after `!`, the inner expression has already been evaluated to a single boolean value.

### 5. **Character Filtering Confusion**
Some might explicitly handle commas and opening parentheses with unnecessary logic:

**Overcomplicated approach:**
```python
if char == ',' or char == '(':
    continue  # Explicit but unnecessary
```

**Why it happens:** Trying to be explicit about every character in the expression.

**Solution:** Simply don't handle these characters - the if-elif structure naturally ignores them:
```python
if char in 'tf!&|':
    stack.append(char)
elif char == ')':
    # evaluate
# Commas and '(' are implicitly ignored
```

