# 3749. Evaluate Valid Expressions 🔒

> Source: https://algo.monster/liteproblems/3749

## Problem Statement

You are given a string `expression` that represents a nested mathematical expression written in a simplified form. Your task is to fully evaluate this expression and return the resulting integer.

An expression is considered **valid** if it satisfies one of the following two conditions:

1. It is an integer **literal** (which may be negative).
2. It follows the format `op(a,b)`, where:
   - `op` is one of the four operators: `"add"`, `"sub"`, `"mul"`, or `"div"`.
   - `a` and `b` are themselves each **valid** expressions (they can also be nested expressions of the same form).

The four **operations** are defined as follows:

- `add(a,b) = a + b` — returns the sum of `a` and `b`.
- `sub(a,b) = a - b` — returns the difference of `a` and `b`.
- `mul(a,b) = a * b` — returns the product of `a` and `b`.
- `div(a,b) = a / b` — returns the (integer) quotient of `a` divided by `b`.

Because `a` and `b` may themselves be expressions of the form `op(a,b)`, the input can be deeply nested. You must evaluate the innermost expressions first and work outward, combining the results according to the operators until the entire expression is reduced to a single integer.

Return an integer representing the **result** after fully evaluating the expression.

## Intuition

The key observation is that the structure of this expression is **recursive by nature**. A valid expression is either a simple integer, or an operator applied to two sub-expressions `op(a,b)`. Crucially, both `a` and `b` follow the exact same rule — they too are valid expressions. This self-similar definition is a strong hint that a **recursive** approach fits perfectly.

When we look at a string like `add(2,mul(3,4))`, we can think of it as a tree:

- The outer expression is `add` applied to `2` and `mul(3,4)`.
- The right operand `mul(3,4)` is itself an expression that must be evaluated first.

To evaluate the whole thing, we naturally need to evaluate the inner parts first, then combine their results — which is exactly what recursion gives us for free.

The main challenge is **parsing**: as we scan the string from left to right, we need to know where each piece begins and ends. A single integer can span multiple characters (and may start with `-`), an operator is always `3` letters long, and the operands are separated by a comma and wrapped in parentheses. Since sub-expressions can be nested to any depth, we cannot simply split the string by commas — a comma inside a nested call would confuse us.

This leads us to design a parsing function that not only returns the **value** of the expression it parsed, but also returns the **position where it stopped**. By returning the next unprocessed index, the caller knows exactly where to continue reading — for example, after parsing the first operand `a`, we know where the comma is, so we can skip it and start parsing `b`.

So the idea becomes:

- If the current character is a digit or a `-`, we are looking at an integer literal. Scan forward while we still see digits, convert the substring to an integer, and report back the value along with the stopping index.
- Otherwise, the current character is the start of an operator. Read the `3`-letter operator, skip the opening `(`, recursively parse the first operand `a`, skip the comma `,`, recursively parse the second operand `b`, and skip the closing `)`. Finally, apply the operator to `a` and `b`.

By having each recursive call hand back both the computed value and the index to resume from, the whole expression unwinds cleanly from the inside out, and the result of `parse(0)` gives us the final answer.

**Learn more about [Stack](/problems/stack_intro), [Math](/problems/math-basics) and [Divide and Conquer](/problems/divide_and_conquer_intro) patterns.**

## Solution Approach

### Solution 1: Recursion

We define a recursive function `parse(i)` that parses the sub-expression starting from index `i` and returns a pair: the **computed result** and the **next unprocessed index**. The final answer is simply `parse(0)[0]`.

The implementation of `parse(i)` works as follows:

**Step 1: Handle an integer literal.**

If the character at the current position `i` is a digit or a negative sign `-`, then we are reading an integer. We start a pointer `j` at `i`:

- If `expression[j]` is `-`, we move `j` forward by one to skip the sign.
- We then keep advancing `j` while `expression[j]` is a digit.

Once we stop, the substring `expression[i:j]` is the full number. We convert it with `int(...)` and return both the value and the index `j`:

```python
if expression[i].isdigit() or expression[i] == "-":
    j = i
    if expression[j] == "-":
        j += 1
    while j < len(expression) and expression[j].isdigit():
        j += 1
    return int(expression[i:j]), j
```

**Step 2: Read the operator.**

Otherwise, position `i` is the start of an operator like `add`, `sub`, `mul`, or `div`. We scan forward with `j` until we hit the opening parenthesis `(`. The substring `expression[i:j]` is the operator string `op`. After reading it, we skip the `(` by doing `j += 1`:

```python
j = i
while expression[j] != "(":
    j += 1
op = expression[i:j]
j += 1
```

**Step 3: Parse the two operands.**

We recursively call `parse(j)` to obtain the first operand `val1` and the updated index. Since the two operands are separated by a comma `,`, we skip it with `j += 1`, then recursively call `parse(j)` again to obtain the second operand `val2`. Finally, we skip the closing parenthesis `)` with another `j += 1`:

```python
val1, j = parse(j)
j += 1
val2, j = parse(j)
j += 1
```

Notice how returning the next index from each recursive call lets the caller resume exactly where the child left off — this is what allows us to correctly walk past commas and parentheses even when the operands are deeply nested.

**Step 4: Apply the operator.**

Based on the operator `op`, we combine `val1` and `val2`. A `match` statement maps each operator to its operation, where division uses integer (floor) division `//`:

```python
match op:
    case "add":
        res = val1 + val2
    case "sub":
        res = val1 - val2
    case "mul":
        res = val1 * val2
    case "div":
        res = val1 // val2
return res, j
```

We return the computed `res` together with the final index `j`.

**Driving the recursion.**

Because the entire string is one valid expression beginning at index `0`, calling `parse(0)` evaluates everything from the inside out, and `parse(0)[0]` gives the final integer result.

In terms of complexity, every character of the string is visited a constant number of times across the recursive calls, so the time complexity is `O(n)`, where `n` is the length of `expression`. The space complexity is `O(n)` as well, due to the recursion [stack](/problems/stack_intro), which can grow proportionally to the nesting depth of the expression.

## Example Walkthrough

Let's trace the solution approach using the small example `expression = "add(2,mul(3,4))"`. We want to evaluate this fully, expecting the result `2 + (3 * 4) = 14`.

Recall that `parse(i)` returns a pair `(value, next_index)`. We kick things off with `parse(0)`.

**Indexing the string for reference:**

```
Index:  0  1  2  3  4  5  6  7  8  9 10 11 12 13 14
Char:   a  d  d  (  2  ,  m  u  l  (  3  ,  4  )  )
```

---

**Call 1: `parse(0)`** — the outermost expression.

- Character at index `0` is `'a'`, not a digit or `-`, so we read an operator (Step 2).
- Scan forward until we hit `'('` at index `3`. The operator is `expression[0:3] = "add"`. Skip `'('`, so `j = 4`.
- Now parse the two operands (Step 3).

**Call 2: `parse(4)`** — the first operand of `add`.

- Character at index `4` is `'2'`, a digit, so we read an integer literal (Step 1).
- Advance `j` while seeing digits: from index `4`, next char (index `5`) is `','`, so we stop. The number is `expression[4:5] = "2"`.
- Return `(2, 5)`.

Back in **Call 1**: `val1 = 2`, `j = 5`. Skip the comma `','`, so `j = 6`. Parse the second operand.

**Call 3: `parse(6)`** — the second operand of `add`.

- Character at index `6` is `'m'`, an operator (Step 2).
- Scan forward until `'('` at index `9`. The operator is `expression[6:9] = "mul"`. Skip `'('`, so `j = 10`.
- Parse the two operands of `mul`.

**Call 4: `parse(10)`** — first operand of `mul`.

- Character `'3'` is a digit. Advance until non-digit: index `11` is `','`. Number is `"3"`.
- Return `(3, 11)`.

Back in **Call 3**: `val1 = 3`, `j = 11`. Skip the comma, so `j = 12`. Parse the second operand.

**Call 5: `parse(12)`** — second operand of `mul`.

- Character `'4'` is a digit. Advance until non-digit: index `13` is `')'`. Number is `"4"`.
- Return `(4, 13)`.

Back in **Call 3**: `val2 = 4`, `j = 13`. Skip the closing `')'`, so `j = 14`. Apply the operator (Step 4):

- `op = "mul"` → `res = 3 * 4 = 12`.
- Return `(12, 14)`.

Back in **Call 1**: `val2 = 12`, `j = 14`. Skip the closing `')'`, so `j = 15`. Apply the operator:

- `op = "add"` → `res = 2 + 12 = 14`.
- Return `(14, 15)`.

---

**Final result:** `parse(0)[0] = 14`.

Notice how the recursion naturally evaluated the inner `mul(3,4)` *before* combining it with `2`. Each call handed back both its computed value **and** the index where it stopped — this is what let the parent call skip past commas and parentheses correctly, even though `mul(3,4)` spans multiple characters and sits inside the outer `add(...)`.

## Python Solution

```python
from typing import Tuple


class Solution:
    def evaluateExpression(self, expression: str) -> int:
        def parse(index: int) -> Tuple[int, int]:
            # Case 1: the token is a number (possibly negative)
            if expression[index].isdigit() or expression[index] == "-":
                end = index
                # Skip the leading minus sign for negative numbers
                if expression[end] == "-":
                    end += 1
                # Advance through all consecutive digits
                while end < len(expression) and expression[end].isdigit():
                    end += 1
                # Return the parsed integer and the next position
                return int(expression[index:end]), end

            # Case 2: the token is an operator expression like "add(...)"
            end = index
            # Read the operator name until the opening parenthesis
            while expression[end] != "(":
                end += 1
            operator = expression[index:end]

            # Skip the "(" and parse the first operand
            end += 1
            value1, end = parse(end)

            # Skip the separating space and parse the second operand
            end += 1
            value2, end = parse(end)

            # Skip the closing ")"
            end += 1

            # Apply the operator to the two operands
            result = 0
            match operator:
                case "add":
                    result = value1 + value2
                case "sub":
                    result = value1 - value2
                case "mul":
                    result = value1 * value2
                case "div":
                    result = value1 // value2

            # Return the computed result and the next position
            return result, end

        # Parse from the beginning and return only the final value
        return parse(0)[0]
```

**Notes on the changes:**

- **Type annotation**: Replaced the tuple-literal annotation `(int, int)` with the canonical `Tuple[int, int]` and added the necessary import. In Python 3.9+ you could also write `tuple[int, int]` without the import.
- **Naming**: `i` → `index`, `j` → `end` (the moving cursor/end pointer), `op` → `operator`, `val1`/`val2` → `value1`/`value2`, `res` → `result` for readability.
- **`match` statement**: Kept as-is since it's valid Python 3.10+ syntax. If you need to support earlier versions, this could be rewritten using `if/elif` branches.

**Alternative perspective** — if you target Python versions below 3.10, replace the `match` block with:

```python3
            if operator == "add":
                result = value1 + value2
            elif operator == "sub":
                result = value1 - value2
            elif operator == "mul":
                result = value1 * value2
            elif operator == "div":
                result = value1 // value2
```

## Time and Space Complexity

**Time Complexity:** `O(n)`, where `n` is the length of the expression string.

The `parse` function processes the expression by scanning through it character by character. Although `parse` is recursive, each character in the expression is visited a constant number of times across all recursive calls:

- When parsing a number (digit or `-` followed by digits), the inner `while` loop advances `j` through consecutive digit characters, and each such character is consumed exactly once.
- When parsing an operator expression, the `while expression[j] != "("` loop scans the operator name characters once, then the function recurses on the two sub-expressions (`val1` and `val2`), advancing the index `j` forward without revisiting earlier characters.

Since the index `j` only moves forward and never backtracks, the total work done is proportional to the number of characters processed, giving an overall time complexity of `O(n)`.

**Space Complexity:** `O(n)`, where `n` is the length of the expression string.

The space is dominated by the recursion call stack. In the worst case, the expression is deeply nested (e.g., `add(1, add(1, add(1, ...)))`), and the depth of recursion is proportional to the level of nesting, which can be on the order of `O(n)`. Each recursive frame uses only `O(1)` additional space (storing indices and intermediate values), so the total auxiliary space is `O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### Pitfall 1: Mishandling Division for Negative Operands

The most common mistake is assuming that `//` in Python performs the same integer division as the problem intends. The problem typically specifies **truncation toward zero** (the same behavior as C/Java integer division), but Python's `//` operator performs **floor division** (rounding toward negative infinity). These differ whenever the result is negative.

**Example of the discrepancy:**

```python
-7 // 2   # Python floor division → -4
# Truncation toward zero would give → -3
```

So an expression like `div(-7,2)` would yield `-4` with the current code, while a problem expecting truncation wants `-3`.

**Solution:** Use truncating division explicitly via `int(value1 / value2)` or the `math.trunc` / `operator.floordiv` alternatives carefully. The cleanest fix:

```python
case "div":
    # Truncate toward zero instead of flooring toward -infinity
    result = int(value1 / value2)
```

But `int(a / b)` risks floating-point inaccuracy for very large integers. A fully integer-safe version is:

```python
case "div":
    # Integer division truncating toward zero
    quotient = abs(value1) // abs(value2)
    if (value1 < 0) != (value2 < 0):
        quotient = -quotient
    result = quotient
```

Always confirm which rounding rule the problem demands before choosing `//`.

---

### Pitfall 2: Confusing the Operand Separator

The solution skips a **single character** between the two operands with `end += 1`, assuming the separator is exactly one character (a comma `,`). However, the prose comment in the code says *"Skip the separating space"*, which hints at a mismatch between the assumed separator and the actual one.

If the input uses `op(a, b)` (comma **plus** a space) instead of `op(a,b)`, then skipping only one character leaves the cursor sitting on the space, and the next `parse` call sees a space rather than a digit/operator — silently misparsing or crashing.

**Solution:** Skip the comma robustly rather than blindly advancing one position:

```python
# Skip the comma and any following whitespace
end += 1  # skip ','
while end < len(expression) and expression[end] == " ":
    end += 1
value2, end = parse(end)
```

Keep the comment and the code in agreement — decide on the exact input format and handle it explicitly.

---

### Pitfall 3: Negative Sign vs. Subtraction Confusion

The number-parsing branch triggers whenever `expression[index] == "-"`. This is correct **only** because operators are alphabetic words (`sub`, etc.), so a bare `-` can only be a negative-number sign in this grammar. If you ever extend the grammar to allow the `-` infix operator, this branch would incorrectly consume the operator as part of a number.

**Solution:** In the current grammar no change is needed, but document the assumption clearly. If extending, distinguish a leading `-` (unary, part of a literal) from a binary minus by checking the parsing context:

```python
# A '-' is a negative literal ONLY at the start of an operand position.
# Ensure callers always invoke parse() at an operand boundary.
```

---

### Pitfall 4: Recursion Depth on Deeply Nested Input

For extremely deep nesting (e.g., thousands of nested `add(...)` calls), the recursive `parse` may exceed Python's default recursion limit (1000), raising `RecursionError`.

**Solution:** Either raise the limit defensively, or convert to an explicit stack-based iterative parser:

```python
import sys
sys.setrecursionlimit(10**6)
```

The iterative alternative uses an explicit stack to hold partial operators and operands, eliminating reliance on the call stack entirely — preferable when input depth is unbounded.

