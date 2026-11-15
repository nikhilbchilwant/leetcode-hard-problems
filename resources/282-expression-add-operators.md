# 282. Expression Add Operators

> Source: https://algo.monster/liteproblems/282

## Problem Statement

You are given a string `num` consisting only of digits and an integer `target`. Your task is to find all possible ways to insert the binary operators `'+'`, `'-'`, and `'*'` between the digits of `num` such that when the resulting expression is evaluated, it equals the `target` value.

Key requirements:
- You can insert operators between any adjacent digits in the string
- The three allowed operators are addition (`+`), subtraction (`-`), and multiplication (`*`)
- Numbers in the resulting expressions cannot have leading zeros (e.g., "05" is not valid, but "0" alone is valid)
- A number can span multiple consecutive digits (you don't have to put an operator between every digit)
- Return all valid expressions that evaluate to the target

For example, if `num = "123"` and `target = 6`, possible valid expressions include:
- `"1+2+3"` which evaluates to 6
- `"1*2*3"` which evaluates to 6

The solution uses a depth-first search (DFS) approach with backtracking. It explores all possible ways to partition the string and insert operators, keeping track of:
- The current position in the string (`u`)
- The previous operand (`prev`) - needed for handling multiplication precedence
- The current evaluation result (`curr`)
- The expression path built so far

## Intuition

When we look at this problem, we need to explore all possible ways to place operators between digits. Think of it like placing dividers in the string - at each position between digits, we can either place an operator or continue building the current number.

The key insight is that this is a decision tree problem. At each step, we face multiple choices:
1. How many digits should form the next number? (1 digit, 2 digits, 3 digits, etc.)
2. Which operator should we use before this number? (+, -, or *)

For example, with `"123"`, we could have:
- `"1+23"`, `"12+3"`, `"123"` (different ways to partition)
- For each partition, we have 3 operator choices

This screams backtracking - we need to try a choice, explore further, and if it doesn't work out, backtrack and try another choice.

The tricky part is handling multiplication because of operator precedence. When we encounter `2+3*4`, we can't just evaluate left to right. We need to "undo" the last addition and apply multiplication first. 

To handle this, we track two values:
- `curr`: The running total of our expression
- `prev`: The last operand we added/subtracted

When we multiply, we need to adjust: `curr - prev + (prev * next)`. This removes the effect of the last operation and applies multiplication correctly.

For leading zeros, we need a simple check: if we're trying to form a multi-digit number starting with '0', we should stop - that's invalid.

The recursive nature of trying all possibilities, combined with the need to track state (current value, previous operand, position in string), naturally leads us to a DFS backtracking solution where we build the expression character by character and evaluate it simultaneously.

## Solution Approach

The solution uses DFS with backtracking to explore all possible expressions. Let's walk through the implementation:

**Core DFS Function Parameters:**
- `u`: Current index in the string where we're deciding what to do next
- `prev`: The previous operand (needed for handling multiplication precedence)
- `curr`: Current evaluation result of the expression so far
- `path`: The expression string we're building

**Main Algorithm Flow:**

1. **Base Case**: When `u == len(num)`, we've processed the entire string. If `curr == target`, we found a valid expression and add it to our answer list.

2. **Recursive Exploration**: From position `u`, we try forming numbers of different lengths:
   ```python
   for i in range(u, len(num)):
   ```
   This loop considers taking 1 digit, 2 digits, 3 digits, etc. as the next number.

3. **Leading Zero Check**: 
   ```python
   if i != u and num[u] == '0':
       break
   ```
   If we're trying to form a multi-digit number starting with '0', we stop. Single '0' is allowed.

4. **Extract Next Number**:
   ```python
   next = int(num[u : i + 1])
   ```
   Convert the substring to an integer.

5. **Handle First Number**: When `u == 0`, there's no operator to place before the first number:
   ```python
   dfs(i + 1, next, next, path + str(next))
   ```

6. **Try All Three Operators**: For subsequent numbers, we recursively try each operator:
 
   - **Addition**: 
     ```python
     dfs(i + 1, next, curr + next, path + "+" + str(next))
     ```
     Simple - just add to current value.
 
   - **Subtraction**:
     ```python
     dfs(i + 1, -next, curr - next, path + "-" + str(next))
     ```
     Store negative value as `prev` for potential future multiplication.
 
   - **Multiplication**:
     ```python
     dfs(i + 1, prev * next, curr - prev + prev * next, path + "*" + str(next))
     ```
     The tricky part: `curr - prev + prev * next`
     - First, undo the last operation by subtracting `prev`
     - Then add `prev * next` to apply multiplication with correct precedence
     - Update `prev` to `prev * next` for potential future operations

**Example Walkthrough** with `"123"`, target = `6`:
- Start with `dfs(0, 0, 0, "")`
- Try `"1"` as first number: `dfs(1, 1, 1, "1")`
  - Try `"1+2"`: `dfs(2, 2, 3, "1+2")`
    - Try `"1+2+3"`: `dfs(3, 3, 6, "1+2+3")` → Success!
  - Try `"1*2"`: `dfs(2, 2, 2, "1*2")`
    - Try `"1*2*3"`: `dfs(3, 6, 6, "1*2*3")` → Success!

The backtracking happens automatically through the recursion - when we return from a recursive call, we're back to the previous state and can try the next option.

## Example Walkthrough

Let's walk through a small example with `num = "105"` and `target = 5`.

**Starting State**: `dfs(0, 0, 0, "")`

**Step 1: Choose first number "1"**
- Extract "1" from position 0
- Since it's the first number, no operator needed
- Call: `dfs(1, 1, 1, "1")`

**Step 2: From "1", try different options at position 1**

*Option 2a: Add "0"*
- Call: `dfs(2, 0, 1, "1+0")`
- From here, try adding "5": `dfs(3, 5, 6, "1+0+5")` → 6 ≠ 5, backtrack
- Try subtracting "5": `dfs(3, -5, -4, "1+0-5")` → -4 ≠ 5, backtrack
- Try multiplying "5": `dfs(3, 0, 1, "1+0*5")` → 1 ≠ 5, backtrack

*Option 2b: Subtract "0"*
- Call: `dfs(2, 0, 1, "1-0")`
- From here, try adding "5": `dfs(3, 5, 6, "1-0+5")` → 6 ≠ 5, backtrack
- Try subtracting "5": `dfs(3, -5, -4, "1-0-5")` → -4 ≠ 5, backtrack
- Try multiplying "5": `dfs(3, 0, 1, "1-0*5")` → 1 ≠ 5, backtrack

*Option 2c: Multiply "0"*
- Call: `dfs(2, 0, 0, "1*0")`
- From here, try adding "5": `dfs(3, 5, 5, "1*0+5")` → **5 = 5, Found solution!**

*Option 2d: Try "05" as next number*
- Since "05" starts with '0' and has multiple digits, this is invalid - skip

**Step 3: Back to start, try "10" as first number**
- Extract "10" from position 0
- Call: `dfs(2, 10, 10, "10")`
- From here, try subtracting "5": `dfs(3, -5, 5, "10-5")` → **5 = 5, Found solution!**

**Step 4: Try "105" as single number**
- Extract "105" from position 0
- Call: `dfs(3, 105, 105, "105")` → 105 ≠ 5, backtrack

**Final Result**: `["1*0+5", "10-5"]`

The key insight illustrated here is how multiplication precedence is handled. When we compute `1*0+5`:
1. First we have `curr = 1, prev = 1`
2. We multiply by 0: `curr = 1 - 1 + 1*0 = 0, prev = 0`
3. We add 5: `curr = 0 + 5 = 5, prev = 5`

This shows how the algorithm correctly evaluates `1*0` first before adding 5, respecting operator precedence.

## Python Solution

```python
class Solution:
    def addOperators(self, num: str, target: int) -> List[str]:
        """
        Find all possible ways to add operators (+, -, *) between digits of num to reach target.
      
        Args:
            num: String containing only digits
            target: Target value to reach
          
        Returns:
            List of valid expressions that evaluate to target
        """
        result = []
      
        def backtrack(index: int, prev_operand: int, current_eval: int, expression: str) -> None:
            """
            Recursively build expressions by trying different operators and operands.
          
            Args:
                index: Current position in the num string
                prev_operand: Previous operand value (needed for multiplication precedence)
                current_eval: Current evaluation result of the expression
                expression: Current expression string being built
            """
            # Base case: reached end of string
            if index == len(num):
                if current_eval == target:
                    result.append(expression)
                return
          
            # Try all possible numbers starting from current index
            for i in range(index, len(num)):
                # Skip numbers with leading zeros (except single digit 0)
                if i != index and num[index] == '0':
                    break
              
                # Extract current number as operand
                current_operand = int(num[index:i + 1])
              
                # First number in expression (no operator needed)
                if index == 0:
                    backtrack(i + 1, current_operand, current_operand, str(current_operand))
                else:
                    # Try addition
                    backtrack(i + 1, 
                             current_operand, 
                             current_eval + current_operand, 
                             expression + "+" + str(current_operand))
                  
                    # Try subtraction
                    backtrack(i + 1, 
                             -current_operand, 
                             current_eval - current_operand, 
                             expression + "-" + str(current_operand))
                  
                    # Try multiplication (need to handle precedence)
                    # Undo the previous operation and apply multiplication first
                    backtrack(i + 1, 
                             prev_operand * current_operand, 
                             current_eval - prev_operand + prev_operand * current_operand, 
                             expression + "*" + str(current_operand))
      
        # Start the recursive exploration
        backtrack(0, 0, 0, "")
        return result
```

## Time and Space Complexity

**Time Complexity:** `O(n * 3^n)`

The time complexity analysis breaks down as follows:
- At each position in the string, we can choose to extend the current number or place an operator before starting a new number
- For a string of length `n`, we have at most `n-1` positions where we can place operators
- At each position where we place an operator, we have 3 choices: `+`, `-`, or `*`
- In the worst case, we explore all possible combinations of operators between digits
- For each valid expression generated, we perform operations that take `O(n)` time (string concatenation and integer parsing)
- The total number of expressions we can generate is bounded by `O(3^n)` since we make up to 3 recursive calls for each position
- Therefore, the overall time complexity is `O(n * 3^n)`

**Space Complexity:** `O(n)`

The space complexity analysis:
- The recursion depth is at most `O(n)` where `n` is the length of the input string
- Each recursive call uses `O(1)` space for its local variables (`u`, `prev`, `curr`, `next`)
- The `path` parameter accumulates the expression string, which can be at most `O(n)` in length (considering operators and digits)
- The `ans` list stores all valid expressions, but this is considered as output space and typically not counted in space complexity analysis
- The call stack space dominates, giving us `O(n)` space complexity

## Common Pitfalls

### 1. **Incorrect Multiplication Precedence Handling**

**The Pitfall:** 
The most common mistake is not properly handling multiplication precedence. Many initial attempts incorrectly evaluate expressions left-to-right without considering that multiplication should be performed before addition/subtraction.

**Incorrect Approach:**
```python
# WRONG: Simply applying operators left to right
if operator == '*':
    current_eval = current_eval * next_number  # This is wrong!
```

For example, with expression "2+3*4":
- Wrong evaluation: (2+3)*4 = 20
- Correct evaluation: 2+(3*4) = 14

**The Solution:**
Track the previous operand and use the formula `curr - prev + prev * next`:
```python
# CORRECT: Undo last operation, then apply multiplication
backtrack(i + 1, 
         prev_operand * current_operand,  # New previous operand
         current_eval - prev_operand + prev_operand * current_operand,  # Adjusted evaluation
         expression + "*" + str(current_operand))
```

### 2. **Leading Zeros Not Properly Handled**

**The Pitfall:**
Forgetting to check for leading zeros can generate invalid expressions like "05+2" or "00+3".

**Incorrect Approach:**
```python
# WRONG: No check for leading zeros
for i in range(index, len(num)):
    current_operand = int(num[index:i + 1])
    # Process without checking...
```

**The Solution:**
Add a check that breaks the loop when encountering a multi-digit number starting with '0':
```python
# CORRECT: Check for leading zeros
for i in range(index, len(num)):
    if i != index and num[index] == '0':
        break  # Stop if trying to form multi-digit number with leading zero
    current_operand = int(num[index:i + 1])
```

### 3. **Incorrect Previous Operand for Subtraction**

**The Pitfall:**
When performing subtraction, forgetting to store the negative value as the previous operand leads to incorrect multiplication handling later.

**Incorrect Approach:**
```python
# WRONG: Storing positive value for subtraction
backtrack(i + 1, current_operand, current_eval - current_operand, expression + "-" + str(current_operand))
```

This causes problems with expressions like "6-2*3":
- After `6-2`, `current_eval = 4`
- Wrong (prev stored as `2`): `4 - 2 + 2*3 = 8`
- Correct (prev stored as `-2`): `4 - (-2) + (-2)*3 = 0`, which matches `6 - 6 = 0`

**The Solution:**
Store the negative value as the previous operand:
```python
# CORRECT: Store negative value for subtraction
backtrack(i + 1, 
         -current_operand,  # Negative value as previous operand
         current_eval - current_operand, 
         expression + "-" + str(current_operand))
```

### 4. **Integer Overflow with Large Numbers**

**The Pitfall:**
Python integers never overflow, but in Java or C++ the operands and partial results can exceed 32 bits. With up to 10 digits, a single operand can be `9999999999`, and products such as `99999*99999` also exceed `2^31 - 1`.

**Example Problem:**
```python
num = "9999999999"  # one operand of 9,999,999,999 does not fit in a 32-bit int
```

**The Solution:**
In Java or C++, keep `prev`, `curr`, and the parsed operand in 64-bit integers (`long` / `long long`). Skipping long operands would drop valid expressions such as `"9999999999-9999999998"`.

### 5. **Not Handling Edge Cases Properly**

**The Pitfall:**
Missing edge cases like single digit strings or when the target is 0.

**Common Edge Cases to Consider:**
- `num = "0", target = 0` → Should return ["0"]
- `num = "00", target = 0` → Should return ["0+0", "0-0", "0*0"]
- `num = "105", target = 5` → Should handle the '0' in the middle correctly

**The Solution:**
The current implementation handles these correctly, but always test with:
```python
# Test edge cases
assert "0" in solution.addOperators("0", 0)
assert len(solution.addOperators("00", 0)) == 3
assert "10-5" in solution.addOperators("105", 5)
```

