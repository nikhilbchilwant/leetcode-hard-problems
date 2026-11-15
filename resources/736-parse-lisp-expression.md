# 736. Parse Lisp Expression

> Source: https://algo.monster/liteproblems/736

## Problem Statement

This problem asks you to evaluate a string expression written in a Lisp-like syntax and return its integer value.

The expression can be one of the following types:

1. **Integer**: A simple integer value (positive or negative)

2. **Variable**: A variable name that starts with a lowercase letter followed by zero or more lowercase letters or digits

3. **Let Expression**: Has the format `"(let v1 e1 v2 e2 ... vn en expr)"` where:
   - `let` is the keyword
   - `v1, v2, ..., vn` are variable names
   - `e1, e2, ..., en` are expressions whose values get assigned to the corresponding variables
   - `expr` is the final expression whose value becomes the result of the entire let expression
   - Variables are assigned sequentially, and each assignment can use previously assigned variables in the same let expression

4. **Add Expression**: Has the format `"(add e1 e2)"` where:
   - `add` is the keyword
   - `e1` and `e2` are expressions to be evaluated
   - The result is the sum of the evaluated values of `e1` and `e2`

5. **Mult Expression**: Has the format `"(mult e1 e2)"` where:
   - `mult` is the keyword
   - `e1` and `e2` are expressions to be evaluated
   - The result is the product of the evaluated values of `e1` and `e2`

**Scope Rules**:
- When evaluating a variable, the innermost scope (closest enclosing parentheses) is checked first for the variable's value
- If not found, outer scopes are checked sequentially
- Variables defined in a let expression are only valid within that expression's scope

**Constraints**:
- The keywords `"add"`, `"let"`, and `"mult"` are reserved and will never be used as variable names
- All expressions are guaranteed to be valid

For example, `"(let x 2 (mult x (let x 3 y 4 (add x y))))"` would evaluate to `14` because:
- Outer `x` is assigned `2`
- Inner let assigns `x = 3` and `y = 4`
- `(add x y)` uses the inner scope values: `3 + 4 = 7`
- `(mult x 7)` uses the outer scope `x = 2`: `2 * 7 = 14`

## Intuition

The key insight is that this problem is essentially asking us to build an interpreter for a simple programming language. When we see nested parentheses and variable scoping, we should think about recursive evaluation - each expression can contain other expressions, so we need to evaluate them from the inside out.

The natural approach is to parse and evaluate the expression as we go through it character by character. When we encounter an opening parenthesis `(`, we know we're entering a new expression that needs to be evaluated recursively. The type of expression is determined by the keyword that follows (`let`, `add`, or `mult`).

For handling variable scopes, we need a data structure that can track multiple values for the same variable name at different scope levels. A dictionary mapping variable names to a [stack](/problems/stack_intro) of values works perfectly here. When we enter a `let` expression and assign a value to a variable, we push the new value onto that variable's stack. When we exit the `let` expression, we pop the value off, effectively restoring the previous scope's value.

The parsing strategy follows these patterns:
- **Variables**: Start with a lowercase letter, so we read characters until we hit a space or closing parenthesis
- **Integers**: May start with a minus sign, followed by digits
- **Expressions**: Always start with `(` and we recursively evaluate based on the keyword

The recursive nature of `evaluate_expression()` mirrors the recursive structure of the expressions themselves. Each call to `evaluate_expression()` handles one complete expression and returns its integer value. For `let` expressions, we need to carefully manage the scope by tracking which variables we've modified so we can restore them after evaluation.

The pointer-based parsing (using `index` to track our position) allows us to efficiently move through the string without creating substrings unnecessarily, and the nonlocal declaration lets nested functions modify this shared parsing position.

**Learn more about [Stack](/problems/stack_intro) and [Recursion](/problems/recursion_intro) patterns.**

## Solution Approach

The solution uses a recursive parsing and evaluation approach with a global index pointer to track the current position in the expression string.

**Data Structures:**
- `scope`: A `defaultdict(list)` where each key is a variable name and the value is a [stack](/problems/stack_intro) (list) of values. This handles nested scopes by pushing/popping values.
- `index`: A global index pointer to track the current parsing position
- `length`: The length of the expression string

**Helper Functions:**

1. **`parse_variable()`**: Extracts a variable name starting from the current position
   - Records the starting position `start = index`
   - Advances `index` until we hit a space or closing parenthesis
   - Returns the substring `expression[start:index]` as the variable name

2. **`parse_integer()`**: Parses an integer value
   - Checks for negative sign and sets `sign = -1` if present
   - Accumulates digits using `value = value * 10 + int(expression[index])`
   - Returns `sign * value` as the final integer

3. **`evaluate_expression()`**: The main recursive evaluation function
   - **Base cases (no opening parenthesis):**
     - If starts with lowercase letter → it's a variable, return `scope[parse_variable()][-1]` (top of the [stack](/problems/stack_intro))
     - Otherwise → it's an integer, return `parse_integer()`
 
   - **Recursive cases (starts with `(`):**
     - Skip the opening parenthesis by incrementing `index`
     - Check the first character to determine expression type:
   
     **For `let` expressions** (starts with 'l'):
     - Skip "let " by advancing `index += 4`
     - Create `variables_assigned` list to track variables modified in this scope
     - Loop to parse variable-value pairs:
       - Parse variable name using `parse_variable()`
       - If next character is `)`, this variable is actually the final expression to evaluate
       - Otherwise, append variable to `variables_assigned` list
       - Recursively evaluate the value expression with `evaluate_expression()`
       - Push the value onto the variable's stack: `scope[var_name].append(evaluate_expression())`
       - Check if next token is a variable (lowercase letter) to continue pairing
       - If not, evaluate the final expression
     - Clean up scope by popping all modified variables: `scope[var_name].pop()` for each `var_name` in `variables_assigned`
   
     **For `add` or `mult` expressions**:
     - Determine operation type: `is_addition = expression[index] == "a"`
     - Skip the operation keyword ("add " is 4 chars, "mult " is 5 chars)
     - Recursively evaluate first operand: `operand1 = evaluate_expression()`
     - Skip the space between operands
     - Recursively evaluate second operand: `operand2 = evaluate_expression()`
     - Calculate result: `result = operand1 + operand2` for add, `result = operand1 * operand2` for mult
   
     - Skip the closing parenthesis before returning

**Execution Flow:**
1. Initialize `index = 0`, `length = len(expression)`, and empty `scope`
2. Call `evaluate_expression()` which recursively processes the entire expression
3. The `index` automatically advances as we parse, shared across all function calls via `nonlocal`
4. Each recursive call to `evaluate_expression()` returns an integer value
5. The scope [stack](/problems/stack_intro) ensures variables are properly scoped and restored after each `let` expression

The elegance of this solution lies in how the recursive structure of the code mirrors the recursive structure of the expressions, and how the [stack](/problems/stack_intro)-based scope management naturally handles the nested variable scoping rules.

## Example Walkthrough

Let's trace through the expression `"(let x 2 (add x 3))"` step by step:

**Initial State:**
- `expression = "(let x 2 (add x 3))"`
- `index = 0`, `scope = {}` (empty)

**Step 1: First eval() call**
- `expression[0] = '('` → Enter recursive case
- `index = 1` (skip opening parenthesis)
- `expression[1] = 'l'` → This is a `let` expression
- `index = 5` (skip "let ")

**Step 2: Parse first variable-value pair**
- Call `parse_variable()` at position 5:
  - `start = 5`
  - `expression[5] = 'x'` → valid variable character
  - `index = 6` 
  - `expression[6] = ' '` → stop parsing
  - Return `"x"`
- `var_name = "x"`, add to `variables_assigned = ["x"]`
- `index = 7` (skip space)
- Call `evaluate_expression()` to get value for `x`:
  - `expression[7] = '2'` → It's an integer
  - Call `parse_integer()`: returns `2`
- `scope["x"].append(2)` → `scope = {"x": [2]}`
- `index = 8` (after parsing "2")

**Step 3: Parse final expression**
- `index = 9` (skip space)
- `expression[9] = '('` → Not a variable, so this is the final expression
- Call `evaluate_expression()` for `"(add x 3)"`:
  - `expression[9] = '('` → Enter recursive case
  - `index = 10` (skip opening parenthesis)
  - `expression[10] = 'a'` → This is an `add` expression
  - `index = 14` (skip "add ")

**Step 4: Evaluate add expression**
- First operand - call `evaluate_expression()`:
  - `expression[14] = 'x'` → It's a variable
  - Call `parse_variable()`: returns `"x"`
  - Look up `scope["x"][-1] = 2`
  - Return `2`
- `index = 16` (skip "x ")
- Second operand - call `evaluate_expression()`:
  - `expression[16] = '3'` → It's an integer
  - Call `parse_integer()`: returns `3`
- `index = 17` (after parsing "3")
- Calculate: `result = 2 + 3 = 5`
- `index = 18` (skip closing parenthesis)
- Return `5`

**Step 5: Clean up let expression**
- Back in the `let` expression, `result = 5`
- Pop modified variables: `scope["x"].pop()` → `scope = {"x": []}`
- `index = 19` (skip closing parenthesis)
- Return `5`

**Final Result: 5**

The key points illustrated:
1. The `index` advances as we parse, shared across all recursive calls
2. Variables are pushed onto scope stacks when assigned in `let`
3. Variable lookups use the top of the stack (`scope[var_name][-1]`)
4. Scope cleanup happens after evaluating the final expression by popping all modified variables
5. Each `evaluate_expression()` call handles one complete expression and returns its integer value

## Python Solution

```python
from collections import defaultdict

class Solution:
    def evaluate(self, expression: str) -> int:
        """
        Evaluates a Lisp-like expression and returns the result.
        Supports let, add, and mult operations with variable scoping.
        """
      
        def parse_variable():
            """Parse a variable name from the current position."""
            nonlocal index
            start = index
            # Continue until we hit a space or closing parenthesis
            while index < length and expression[index] not in " )":
                index += 1
            return expression[start:index]
      
        def parse_integer():
            """Parse an integer value from the current position."""
            nonlocal index
            sign = 1
            value = 0
          
            # Handle negative numbers
            if expression[index] == "-":
                sign = -1
                index += 1
          
            # Build the integer digit by digit
            while index < length and expression[index].isdigit():
                value = value * 10 + int(expression[index])
                index += 1
          
            return sign * value
      
        def evaluate_expression():
            """Recursively evaluate the expression from the current position."""
            nonlocal index
          
            # Base cases: not an expression (either variable or integer)
            if expression[index] != "(":
                if expression[index].islower():
                    # It's a variable, return its current value from scope
                    var_name = parse_variable()
                    return scope[var_name][-1]
                else:
                    # It's an integer literal
                    return parse_integer()
          
            # Skip opening parenthesis
            index += 1
          
            # Handle 'let' expression
            if expression[index] == "l":
                # Skip "let " (4 characters)
                index += 4
                variables_assigned = []
              
                while True:
                    var_name = parse_variable()
                  
                    # Check if this is the final expression (no assignment)
                    if expression[index] == ")":
                        # Return the value of the variable
                        result = scope[var_name][-1]
                        break
                  
                    # This variable gets assigned a value
                    variables_assigned.append(var_name)
                    index += 1  # Skip space
                  
                    # Evaluate the value to assign
                    scope[var_name].append(evaluate_expression())
                    index += 1  # Skip space
                  
                    # Check if next token is not a variable (final expression)
                    if not expression[index].islower():
                        result = evaluate_expression()
                        break
              
                # Clean up scope - remove temporary variable bindings
                for var in variables_assigned:
                    scope[var].pop()
          
            # Handle 'add' or 'mult' expression
            else:
                is_addition = expression[index] == "a"
                # Skip "add " (4 chars) or "mult " (5 chars)
                index += 4 if is_addition else 5
              
                # Evaluate first operand
                operand1 = evaluate_expression()
                index += 1  # Skip space
              
                # Evaluate second operand
                operand2 = evaluate_expression()
              
                # Perform the operation
                result = operand1 + operand2 if is_addition else operand1 * operand2
          
            # Skip closing parenthesis
            index += 1
            return result
      
        # Initialize parsing state
        index = 0
        length = len(expression)
        # Use defaultdict to maintain variable scope stack
        scope = defaultdict(list)
      
        # Start evaluation
        return evaluate_expression()
```

## Time and Space Complexity

**Time Complexity: `O(n)`**

The algorithm performs a single pass through the expression string, where `n` is the length of the input expression. Each character is visited at most once during the parsing process:
- `parse_variable()` and `parse_integer()` advance the `index` through characters without backtracking
- `evaluate_expression()` recursively processes each part of the expression exactly once
- For `let` expressions, even though we may have multiple variable assignments, each character in those assignments is still processed only once
- The operations for add and mult are constant time

Since we traverse each character at most once and perform constant-time operations (like dictionary lookups and arithmetic operations), the overall time complexity is linear.

**Space Complexity: `O(n)`**

The space complexity consists of several components:
- **Recursion Stack**: In the worst case, we could have deeply nested expressions, leading to a recursion depth of `O(n)` (consider an expression like `(add (add (add ...)))`).
- **Scope Dictionary**: The `scope` dictionary uses a defaultdict with lists to store variable values. In the worst case with nested `let` expressions, we might store `O(n)` total values across all variables, as each character could theoretically be part of a variable name or value.
- **String Slicing**: The `parse_variable()` function creates substring slices which could be up to `O(n)` in total across all calls.

The dominant factor is the potential recursion depth and the storage for variable scopes, both of which can be proportional to the input size in pathological cases, resulting in `O(n)` space complexity.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Scope Management - Not Using Stack for Variables**

**Pitfall**: A common mistake is using a simple dictionary instead of a stack-based approach for variable scope. This fails when the same variable is redefined in nested let expressions.

**Problematic Code**:
```python
# Wrong approach - using simple dictionary
scope = {}  # This won't handle nested scopes correctly

def evaluate_expression():
    if expression[index] == "l":  # let expression
        # ...
        old_value = scope.get(var_name)  # Try to save old value
        scope[var_name] = evaluate_expression()  # Overwrite
        # ...
        if old_value is not None:
            scope[var_name] = old_value  # Restore - but what if it was undefined?
        else:
            del scope[var_name]  # This gets messy
```

**Why it fails**: Consider `"(let x 2 (let x 3 x))"`. The inner let should return 3, and after it completes, x should still be 2 in the outer scope. Simple dictionary overwrites lose the outer scope's value.

**Solution**: Use a stack (list) for each variable:
```python
from collections import defaultdict
scope = defaultdict(list)  # Each variable has a stack of values

# When assigning:
scope[var_name].append(value)  # Push new value

# When accessing:
return scope[var_name][-1]  # Get most recent value

# When leaving scope:
scope[var_name].pop()  # Remove the temporary binding
```

### 2. **Parsing the Final Expression in Let - Ambiguity Problem**

**Pitfall**: Incorrectly determining when we've reached the final expression in a let statement. The final expression could be a variable name, an integer, or another expression.

**Problematic Code**:
```python
# Wrong - assumes pairs always come in complete sets
while index < length and expression[index] != ")":
    var = parse_variable()
    index += 1  # skip space
    value = evaluate_expression()  # This might be the final expression!
    scope[var].append(value)
```

**Why it fails**: For `"(let x 2 x)"`, after parsing "x" and "2", the next "x" is the final expression to evaluate, not a new variable to assign.

**Solution**: After parsing a potential variable name, check if the next character is `)` or if we're about to parse a value:
```python
var_name = parse_variable()

# Check if this is the final expression (no assignment follows)
if expression[index] == ")":
    # This is the final expression - just evaluate the variable
    result = scope[var_name][-1]
    break

# Otherwise, it's a variable assignment
variables_assigned.append(var_name)
index += 1  # Skip space
scope[var_name].append(evaluate_expression())
```

### 3. **Global Index Management - Race Conditions in Recursion**

**Pitfall**: Using local index variables or incorrectly managing the global index pointer can cause parsing to get out of sync.

**Problematic Code**:
```python
def parse_variable(index):  # Passing index as parameter
    start = index
    while index < length and expression[index] not in " )":
        index += 1
    return expression[start:index], index  # Returning new index

# Usage becomes error-prone:
var, index = parse_variable(index)  # Easy to forget to update index
```

**Solution**: Use `nonlocal` to share the index across all parsing functions:
```python
def evaluate(self, expression: str) -> int:
    def parse_variable():
        nonlocal index  # Share the same index variable
        start = index
        while index < length and expression[index] not in " )":
            index += 1  # Automatically updates the shared index
        return expression[start:index]
  
    index = 0  # Single shared index
    length = len(expression)
```

### 4. **Not Handling Sequential Variable Dependencies in Let**

**Pitfall**: Evaluating all variable assignments before adding them to scope, which prevents later variables from using earlier ones in the same let expression.

**Problematic Code**:
```python
# Wrong - evaluate all values first, then assign
values = []
for i in range(0, len(vars), 2):
    values.append(evaluate_expression())  # Can't use previous vars!

for i, var in enumerate(vars):
    scope[var].append(values[i])
```

**Why it fails**: For `"(let x 2 y x y)"`, y should be assigned the value of x (which is 2), but if we evaluate all values first, x won't be in scope when evaluating the expression for y.

**Solution**: Assign variables immediately after evaluating their values:
```python
while True:
    var_name = parse_variable()
    # ... check if final expression ...
  
    variables_assigned.append(var_name)
    index += 1
    # Immediately add to scope so next variables can use it
    scope[var_name].append(evaluate_expression())
```

