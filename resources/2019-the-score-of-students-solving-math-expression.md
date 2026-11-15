# 2019. The Score of Students Solving Math Expression

> Source: https://algo.monster/liteproblems/2019

## Problem Statement

You are given a string `s` containing only digits `0-9`, addition symbols `'+'`, and multiplication symbols `'*'`, representing a valid [math](/problems/math-basics) expression with single-digit numbers (e.g., `3+5*2`). This expression was given to `n` elementary school students who were instructed to evaluate it following these specific rules:

1. First, compute all multiplications from left to right
2. Then, compute all additions from left to right

For example, for the expression `3+5*2`:
- First, calculate `5*2 = 10`
- Then, calculate `3+10 = 13`

You are also given an integer array `answers` of length `n`, containing the answers submitted by the students in no particular order.

Your task is to grade these answers according to the following scoring system:

- **5 points**: If the student's answer equals the correct answer (following the proper order of operations)
- **2 points**: If the student's answer is incorrect but could be obtained by applying the operators in a different order while still performing correct arithmetic (e.g., calculating `(3+5)*2 = 16` instead of `3+(5*2) = 13`)
- **0 points**: If the student's answer cannot be obtained through any valid interpretation of the expression

The problem asks you to return the total sum of points earned by all students.

The key challenge is to find all possible results that can be obtained by evaluating the expression with different orders of operations (different ways of parenthesizing), while still performing correct arithmetic for each operation. This helps identify which incorrect answers deserve partial credit (2 points) for using valid arithmetic but wrong operation order.

## Intuition

The core insight is that we need to find all possible results that can be obtained by evaluating the expression with different parenthesizations. When students apply operators in the wrong order, they're essentially adding parentheses in different places.

For example, with `3+5*2`:
- Correct order: `3+(5*2) = 13`
- Wrong order could be: `(3+5)*2 = 16`

This naturally leads us to think about **interval [dynamic programming](/problems/dynamic_programming_intro)**, where we can break down the expression into smaller subexpressions and combine their results.

The key observation is that any way of evaluating an expression can be viewed as:
1. Choosing a position to split the expression into two parts
2. Evaluating each part independently
3. Combining the results with the operator at the split position

For instance, with digits at positions `[i, j]`, we can split at any position `k` where `i ≤ k < j`, compute all possible results for `[i, k]` and `[k+1, j]`, then combine them using the operator between positions `k` and `k+1`.

This recursive structure suggests using [dynamic programming](/problems/dynamic_programming_intro) where `f[i][j]` stores all possible results we can get from evaluating digits from position `i` to `j` with any parenthesization.

We build up from smaller intervals to larger ones:
- Single digits: `f[i][i]` contains just the digit itself
- Larger intervals: For `f[i][j]`, we try all split points `k` and combine results from `f[i][k]` and `f[k+1][j]`

By computing all possible results this way, we can identify which student answers deserve partial credit - they're the ones that appear in `f[0][m-1]` (all possible results for the entire expression) but aren't the correct answer.

The constraint of limiting results to `≤ 1000` helps keep the set of possible values manageable and prevents the solution from becoming too computationally expensive.

**Learn more about [Stack](/problems/stack_intro), [Memoization](/problems/memoization_intro), [Math](/problems/math-basics) and [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The solution uses interval [dynamic programming](/problems/dynamic_programming_intro) to find all possible results from different parenthesizations of the expression.

**Step 1: Calculate the Correct Answer**

First, we implement a helper function `cal(s)` that computes the correct result following the proper order of operations:
- Process multiplications first from left to right
- Then process additions from left to right

```python
def cal(s: str) -> int:
    res, pre = 0, int(s[0])
    for i in range(1, n, 2):
        if s[i] == "*":
            pre *= int(s[i + 1])
        else:
            res += pre
            pre = int(s[i + 1])
    res += pre
    return res
```

The variable `pre` accumulates multiplication results, and when we encounter an addition, we add `pre` to `res` and start a new accumulation.

**Step 2: Initialize the DP Table**

We create a 2D DP table `f[i][j]` where each cell contains a set of possible results for evaluating digits from position `i` to `j`:

```python
m = (n + 1) >> 1  # number of digits
f = [[set() for _ in range(m)] for _ in range(m)]
```

For base cases (single digits), `f[i][i]` contains just the digit itself:
```python
for i in range(m):
    f[i][i] = {int(s[i << 1])}
```

Note: `s[i << 1]` maps the i-th digit to its position in string `s` (digits are at even indices).

**Step 3: Fill the DP Table**

We iterate through intervals from largest `i` to smallest, and for each `i`, iterate `j` from `i` to `m-1`:

```python
for i in range(m - 1, -1, -1):
    for j in range(i, m):
        for k in range(i, j):
            for l in f[i][k]:
                for r in f[k + 1][j]:
                    if s[k << 1 | 1] == "+" and l + r <= 1000:
                        f[i][j].add(l + r)
                    elif s[k << 1 | 1] == "*" and l * r <= 1000:
                        f[i][j].add(l * r)
```

For each interval `[i, j]`:
- Try all split points `k` where `i ≤ k < j`
- Combine results from `f[i][k]` and `f[k+1][j]`
- The operator at position `k` is at index `k * 2 + 1` in string `s` (operators are at odd indices)
- Only add results ≤ 1000 to prevent explosion of possibilities

**Step 4: Calculate Final Score**

Count occurrences of each answer and assign points:

```python
cnt = Counter(answers)
ans = cnt[x] * 5  # 5 points for correct answer
for k, v in cnt.items():
    if k != x and k in f[0][m - 1]:
        ans += v << 1  # 2 points for valid but wrong order
```

- Students with the correct answer `x` get 5 points
- Students whose answers appear in `f[0][m-1]` (all possible results) but aren't `x` get 2 points
- All other answers get 0 points (implicitly)

The time complexity is `O(m^3 * P)` where `m` is the number of digits and `P` is the maximum number of possible results per interval. The space complexity is `O(m^2 * P)` for storing all possible results.

## Example Walkthrough

Let's walk through the expression `"2+3*4"` with student answers `[14, 20, 11, 8]`.

**Step 1: Calculate the Correct Answer**

Following proper order of operations (multiplication first, then addition):
- First: `3 * 4 = 12`
- Then: `2 + 12 = 14`
- Correct answer = `14`

**Step 2: Find All Possible Results Using DP**

The expression has 3 digits at positions 0, 2, 4 in the string (digits at even indices).
Operators '+' and '*' are at positions 1, 3 (odd indices).

Initialize DP table `f[i][j]` for digit indices 0, 1, 2:
- `f[0][0] = {2}` (just digit '2')
- `f[1][1] = {3}` (just digit '3')
- `f[2][2] = {4}` (just digit '4')

**Building up intervals:**

For interval `[0,1]` (digits "2" and "3" with operator '+'):
- Split at k=0: combine `f[0][0]={2}` and `f[1][1]={3}` with '+'
- Result: `2 + 3 = 5`
- `f[0][1] = {5}`

For interval `[1,2]` (digits "3" and "4" with operator '*'):
- Split at k=1: combine `f[1][1]={3}` and `f[2][2]={4}` with '*'
- Result: `3 * 4 = 12`
- `f[1][2] = {12}`

For interval `[0,2]` (entire expression):
- Split at k=0: combine `f[0][0]={2}` and `f[1][2]={12}` with '+'
  - Result: `2 + 12 = 14` (this is `2+(3*4)`)
- Split at k=1: combine `f[0][1]={5}` and `f[2][2]={4}` with '*'
  - Result: `5 * 4 = 20` (this is `(2+3)*4`)
- `f[0][2] = {14, 20}`

**Step 3: Grade Student Answers**

All possible valid results: `f[0][2] = {14, 20}`

Grading each answer:
- `14`: This is the correct answer → **5 points**
- `20`: This appears in `f[0][2]` but isn't correct (student calculated `(2+3)*4`) → **2 points**
- `11`: Not in `f[0][2]` (no valid interpretation gives 11) → **0 points**
- `8`: Not in `f[0][2]` (no valid interpretation gives 8) → **0 points**

**Total Score: 5 + 2 + 0 + 0 = 7 points**

This example shows how the DP approach systematically finds all possible results (14 and 20) by trying different parenthesizations, allowing us to identify which wrong answers deserve partial credit for using valid arithmetic with incorrect operator precedence.

## Python Solution

```python
class Solution:
    def scoreOfStudents(self, s: str, answers: List[int]) -> int:
        def calculate_correct_answer(expression: str) -> int:
            """
            Calculate the correct result of the expression following order of operations.
            Multiplication is performed before addition.
            """
            result = 0
            current_term = int(expression[0])
          
            # Process operators and operands pairs
            for i in range(1, expression_length, 2):
                operator = expression[i]
                next_number = int(expression[i + 1])
              
                if operator == "*":
                    # Multiply with current term
                    current_term *= next_number
                else:  # operator == "+"
                    # Add current term to result and start new term
                    result += current_term
                    current_term = next_number
          
            # Add the last term
            result += current_term
            return result
      
        expression_length = len(s)
        correct_answer = calculate_correct_answer(s)
      
        # Number of operands in the expression
        num_operands = (expression_length + 1) >> 1  # Equivalent to // 2
      
        # dp[i][j] stores all possible results for subexpression from operand i to j
        # considering different evaluation orders (ignoring operator precedence)
        dp = [[set() for _ in range(num_operands)] for _ in range(num_operands)]
      
        # Initialize base case: single operands
        for i in range(num_operands):
            operand_position = i << 1  # Convert operand index to string position (i * 2)
            dp[i][i] = {int(s[operand_position])}
      
        # Fill DP table for all subexpressions
        for start in range(num_operands - 1, -1, -1):
            for end in range(start, num_operands):
                # Try splitting at each possible position
                for split_point in range(start, end):
                    # Get operator position in string
                    operator_position = split_point << 1 | 1  # (split_point * 2 + 1)
                    operator = s[operator_position]
                  
                    # Combine results from left and right subexpressions
                    for left_value in dp[start][split_point]:
                        for right_value in dp[split_point + 1][end]:
                            if operator == "+":
                                new_value = left_value + right_value
                                # Only keep values <= 1000 to avoid overflow
                                if new_value <= 1000:
                                    dp[start][end].add(new_value)
                            else:  # operator == "*"
                                new_value = left_value * right_value
                                # Only keep values <= 1000 to avoid overflow
                                if new_value <= 1000:
                                    dp[start][end].add(new_value)
      
        # Count occurrences of each answer
        answer_counts = Counter(answers)
      
        # Calculate total score
        total_score = answer_counts[correct_answer] * 5  # 5 points for correct answer
      
        # Award 2 points for each wrong but possible answer
        for answer_value, count in answer_counts.items():
            if answer_value != correct_answer and answer_value in dp[0][num_operands - 1]:
                total_score += count << 1  # count * 2
      
        return total_score
```

## Time and Space Complexity

The time complexity is `O(n^3 × M)`, where `n` is the length of string `s` and `M` is the maximum possible value bound (1000 in this case).

- The outer two loops iterate through all possible substrings: `O(m^2)` where `m = (n+1)/2` is the number of operands, which is `O(n^2)`.
- For each substring `[i, j]`, we iterate through all possible split points `k`: `O(m)` which is `O(n)`.
- For each split point, we iterate through all values in `f[i][k]` and `f[k+1][j]`. In the worst case, each set can contain up to `M` different values.
- The nested iteration through two sets gives `O(M^2)` operations for each split point.
- However, due to the constraint that we only add values when `l + r <= 1000` or `l * r <= 1000`, the actual number of valid combinations is bounded by `O(M)` rather than `O(M^2)`.
- Total: `O(n^2 × n × M) = O(n^3 × M)`

The space complexity is `O(n^2 × M)`:

- We have a 2D array `f` of size `m × m` where `m = (n+1)/2`, so `O(n^2)` cells.
- Each cell contains a set that can have at most `M` elements (bounded by 1000).
- The `Counter` for answers takes `O(|answers|)` space.
- Total: `O(n^2 × M)`

Note: The reference answer suggests `O(n^3 × M^2)` for time complexity, which would be the case if we didn't have the upper bound constraint (≤ 1000) that effectively limits the number of valid combinations.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Overflow and Memory Issues with Large Intermediate Results**

One of the most common pitfalls in this problem is not properly handling the potential explosion of intermediate results in the DP table. Without proper constraints, the sets in the DP table can grow exponentially large, leading to:
- Memory limit exceeded errors
- Time limit exceeded due to processing massive sets
- Integer overflow in calculations

**Example of the Problem:**
Consider an expression like `9*9*9*9*9`. Without limits:
- First two 9s: {81}
- First three 9s: {729}
- First four 9s: {6561}
- All five 9s: {59049}

But with different parenthesizations, many more intermediate values are possible, and the set sizes can explode.

**Solution:**
The code correctly addresses this by limiting results to 1000:
```python
if operator == "+":
    new_value = left_value + right_value
    if new_value <= 1000:  # Critical constraint
        dp[start][end].add(new_value)
else:  # operator == "*"
    new_value = left_value * right_value
    if new_value <= 1000:  # Critical constraint
        dp[start][end].add(new_value)
```

This constraint is safe because:
- The correct answer is guaranteed to be ≤ 1000 (per problem constraints)
- Any valid alternative interpretation will also be ≤ 1000
- Values > 1000 cannot be valid student answers worth partial credit

### 2. **Incorrect Index Mapping Between Operators and Operands**

Another common mistake is incorrectly mapping between:
- Operand indices (0, 1, 2, ...) 
- Operator indices (0, 1, 2, ...)
- String positions (operands at even indices, operators at odd indices)

**Example of the Problem:**
For string `"3+5*2"`:
- Operands: position 0 (3), position 2 (5), position 4 (2)
- Operators: position 1 (+), position 3 (*)

Incorrect mapping could lead to:
```python
# Wrong: Using i directly instead of i*2
dp[i][i] = {int(s[i])}  # Would get '+' or '*' instead of digit

# Wrong: Incorrect operator position
operator = s[k * 2]  # Would get a digit instead of operator
```

**Solution:**
The code uses bit operations for efficient and correct mapping:
```python
# Operand at index i is at position i*2 in string
operand_position = i << 1  # i * 2

# Operator after operand k is at position k*2+1 in string  
operator_position = split_point << 1 | 1  # split_point * 2 + 1
```

### 3. **Not Handling the Correct Answer Calculation Properly**

A subtle pitfall is incorrectly implementing the standard order of operations in the `calculate_correct_answer` function.

**Example of the Problem:**
For `"3+5*2"`, incorrect implementations might:
```python
# Wrong: Evaluating left to right without precedence
result = 3
result = result + 5  # 8
result = result * 2  # 16 (wrong!)

# Wrong: Not accumulating multiplication chains
for i in range(1, n, 2):
    if s[i] == "*":
        result *= int(s[i + 1])  # Multiplies with running sum
    else:
        result += int(s[i + 1])
```

**Solution:**
The code correctly uses a `current_term` variable to accumulate multiplication chains:
```python
result = 0
current_term = int(expression[0])

for i in range(1, expression_length, 2):
    if expression[i] == "*":
        current_term *= int(expression[i + 1])  # Accumulate multiplications
    else:  # "+"
        result += current_term  # Add completed term
        current_term = int(expression[i + 1])  # Start new term

result += current_term  # Don't forget the last term!
```

This ensures that `3+5*2` is evaluated as `3+(5*2)=13`, not `(3+5)*2=16`.

