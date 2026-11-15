# 2585. Number of Ways to Earn Points

> Source: https://algo.monster/liteproblems/2585

## Problem Statement

You are given a test that contains `n` different types of questions. Each question type is described by two values:
- `count[i]`: the number of questions available of type `i`
- `marks[i]`: the points earned for solving each question of type `i`

Your goal is to find the number of ways to earn **exactly** `target` points by solving questions from the test.

The input is provided as:
- An integer `target` representing the exact score you need to achieve
- A 2D array `types` where `types[i] = [count[i], marks[i]]`

Important notes:
- Questions of the same type are indistinguishable. For example, if there are 3 questions of type 1, solving questions #1 and #2 is considered the same as solving questions #1 and #3.
- You can choose to solve any number of questions from each type (from 0 up to the available count)
- The answer should be returned modulo `10^9 + 7` since it can be very large

For example, if you have `types = [[2, 5], [3, 2]]` and `target = 10`:
- Type 1 has 2 questions worth 5 points each
- Type 2 has 3 questions worth 2 points each
- You could achieve exactly 10 points by:
  - Solving 2 questions of type 1: 2 × 5 = 10 points
  - Solving 0 questions of type 1 and 5 questions of type 2: 0 × 5 + 5 × 2 = 10 points (but this is invalid since type 2 only has 3 questions)
  - Other valid combinations...

The problem asks you to count all valid ways to select questions that sum to exactly the target score.

## Intuition

This problem asks us to count the number of ways to achieve an exact score, which immediately suggests a counting problem that can be solved with [dynamic programming](/problems/dynamic_programming_intro). The key insight is that we need to make decisions about how many questions to solve from each type, and these decisions build upon each other.

Think of it as a knapsack-like problem where instead of maximizing value, we're counting the number of ways to reach an exact "weight" (the target score). Each question type represents a group of identical items, and we can take anywhere from 0 to `count[i]` items from each group.

The natural approach is to process one question type at a time. For each type, we decide how many questions to solve from it (0, 1, 2, ... up to the available count). Each choice affects our remaining target score and builds upon the ways we could achieve scores using previous question types.

This leads us to define our state as `f[i][j]` = number of ways to get exactly `j` points using only the first `i` types of questions. 

Why does this work? Because:
1. We can break down the problem into smaller subproblems (using fewer question types)
2. The solution to a larger problem depends on solutions to smaller problems
3. We can systematically build up from "0 types of questions" to "all n types"

For each new question type we consider, we examine all possible numbers of questions we could solve from that type. If we solve `k` questions of type `i` (each worth `marks[i]` points), we get `k × marks[i]` points from this type. The remaining points `j - k × marks[i]` must come from the previous types, which we've already computed.

The transition naturally becomes: for each possible score `j` and each valid number of questions `k` from the current type, add the number of ways to achieve score `j - k × marks[i]` using previous types to our current count for score `j`.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

We implement the [dynamic programming](/problems/dynamic_programming_intro) solution using a 2D array `f` where `f[i][j]` represents the number of ways to achieve exactly `j` points using the first `i` types of questions.

**Initialization:**
- Create a 2D array `f` of size `(n+1) × (target+1)` initialized with zeros
- Set `f[0][0] = 1` as there's exactly one way to get 0 points with 0 question types (solve nothing)
- All other values start at 0

**State Transition:**
For each question type `i` from 1 to `n`:
1. Extract `count` and `marks` from `types[i-1]` (using 0-indexed array)
2. For each possible score `j` from 0 to `target`:
   - For each possible number of questions `k` from 0 to `count`:
     - Check if we can afford `k` questions: `j >= k * marks`
     - If yes, add the ways to achieve score `j - k * marks` using the previous `i-1` types:
       ```
       f[i][j] = (f[i][j] + f[i-1][j - k * marks]) % mod
       ```

The formula captures the idea that to get `j` points using `i` question types, we can:
- Solve 0 questions of type `i` and get `j` points from types 1 to `i-1`
- Solve 1 question of type `i` (getting `marks` points) and get `j - marks` points from types 1 to `i-1`
- Solve 2 questions of type `i` (getting `2 * marks` points) and get `j - 2 * marks` points from types 1 to `i-1`
- ... and so on up to `count` questions

**Implementation Details:**
- We use modulo `10^9 + 7` at each addition to prevent integer overflow
- The outer loop iterates through question types (1 to `n`)
- The middle loop iterates through all possible scores (0 to `target`)
- The inner loop iterates through all valid numbers of questions for the current type (0 to `count`)

**Time Complexity:** `O(n × target × max_count)` where `max_count` is the maximum number of questions of any single type

**Space Complexity:** `O(n × target)` for the DP table

The final answer is `f[n][target]`, representing the number of ways to achieve exactly `target` points using all `n` question types.

## Example Walkthrough

Let's walk through a small example with `types = [[2, 3], [1, 4]]` and `target = 6`.

This means:
- Type 1: 2 questions worth 3 points each
- Type 2: 1 question worth 4 points

We'll build our DP table `f[i][j]` where `i` represents using the first `i` question types and `j` represents the score.

**Step 1: Initialize**
Create a table `f` of size 3×7 (since we have 2 question types and target=6):
```
f[0][0] = 1 (one way to get 0 points with 0 types)
All other entries = 0
```

Initial state:
```
    j: 0  1  2  3  4  5  6
i=0:   1  0  0  0  0  0  0
i=1:   0  0  0  0  0  0  0
i=2:   0  0  0  0  0  0  0
```

**Step 2: Process Type 1 (count=2, marks=3)**
For each score j from 0 to 6, consider taking k questions (k can be 0, 1, or 2):

- j=0: Only k=0 works (0×3=0). f[1][0] = f[0][0] = 1
- j=1: No valid k (can't make 1 with multiples of 3)
- j=2: No valid k
- j=3: k=1 works (1×3=3). f[1][3] = f[0][3-3] = f[0][0] = 1
- j=4: No valid k
- j=5: No valid k  
- j=6: k=2 works (2×3=6). f[1][6] = f[0][6-6] = f[0][0] = 1

After processing Type 1:
```
    j: 0  1  2  3  4  5  6
i=0:   1  0  0  0  0  0  0
i=1:   1  0  0  1  0  0  1
i=2:   0  0  0  0  0  0  0
```

**Step 3: Process Type 2 (count=1, marks=4)**
For each score j from 0 to 6, consider taking k questions (k can be 0 or 1):

- j=0: k=0 works. f[2][0] = f[1][0] = 1
- j=1: No valid k
- j=2: No valid k
- j=3: k=0 works. f[2][3] = f[1][3] = 1
- j=4: k=1 works (1×4=4). f[2][4] = f[1][4-4] = f[1][0] = 1
- j=5: No valid k
- j=6: k=0 works. f[2][6] = f[1][6] = 1

But wait! For j=6, we should also check k=1:
- k=1 would need j-4 = 2 points from Type 1, but f[1][2] = 0

So f[2][6] = f[1][6] = 1

Final table:
```
    j: 0  1  2  3  4  5  6
i=0:   1  0  0  0  0  0  0
i=1:   1  0  0  1  0  0  1
i=2:   1  0  0  1  1  0  1
```

**Answer:** f[2][6] = 1

There's exactly 1 way to achieve 6 points: solve 2 questions of Type 1 (2×3=6 points) and 0 questions of Type 2.

Let's verify: Could we do 0 questions of Type 1 and some Type 2? We'd need 6 points from Type 2, but 1×4=4, which isn't enough. Could we do 1 question of Type 1 and some Type 2? We'd have 3 points from Type 1 and need 3 more, but 1×4=4 is too much. So indeed, the only way is 2 questions of Type 1.

## Python Solution

```python
class Solution:
    def waysToReachTarget(self, target: int, types: List[List[int]]) -> int:
        """
        Calculate the number of ways to reach exactly the target score.
      
        Args:
            target: The target score to achieve
            types: List of [count, marks] where count is max questions of this type
                   and marks is the score per question
      
        Returns:
            Number of ways to reach the target score modulo 10^9 + 7
        """
        num_types = len(types)
        MOD = 10**9 + 7
      
        # dp[i][j] represents number of ways to achieve score j using first i types
        dp = [[0] * (target + 1) for _ in range(num_types + 1)]
      
        # Base case: one way to achieve score 0 with 0 types
        dp[0][0] = 1
      
        # Process each question type
        for type_idx in range(1, num_types + 1):
            max_questions, points_per_question = types[type_idx - 1]
          
            # For each possible score
            for score in range(target + 1):
                # Try using 0 to max_questions of current type
                for questions_used in range(max_questions + 1):
                    points_from_current = questions_used * points_per_question
                  
                    # Check if we can achieve this score
                    if score >= points_from_current:
                        remaining_score = score - points_from_current
                        dp[type_idx][score] = (dp[type_idx][score] + 
                                              dp[type_idx - 1][remaining_score]) % MOD
      
        return dp[num_types][target]
```

## Time and Space Complexity

The time complexity is `O(n × target × count)` where:
- `n` is the number of question types (length of the `types` array)
- `target` is the target score we want to achieve
- `count` represents the maximum number of questions we can solve for each type

This is because we have three nested loops:
1. The outer loop iterates through all `n` question types
2. The middle loop iterates through all possible scores from `0` to `target`
3. The inner loop iterates through the number of questions we can take from the current type (from `0` to `count`)

The space complexity is `O(n × target)` because we create a 2D DP table `f` with dimensions `(n + 1) × (target + 1)` to store the number of ways to achieve each score using the first `i` types of questions.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Integer Overflow Without Proper Modulo Application**

**The Pitfall:**
A common mistake is forgetting to apply the modulo operation during intermediate calculations, or applying it only at the very end. This can cause integer overflow even in languages that handle large integers well, and definitely causes issues in languages with fixed integer sizes.

**Incorrect approach:**
```python
# Wrong - accumulating without modulo
for questions_used in range(max_questions + 1):
    if score >= questions_used * points_per_question:
        dp[type_idx][score] += dp[type_idx - 1][score - questions_used * points_per_question]

# Applying modulo only at the end
dp[type_idx][score] %= MOD
```

**Why it's problematic:**
- The intermediate sum can grow extremely large before the modulo is applied
- In some test cases with large counts and many types, the number of ways can exceed standard integer limits

**The Solution:**
Apply modulo operation immediately after each addition:
```python
dp[type_idx][score] = (dp[type_idx][score] + 
                       dp[type_idx - 1][remaining_score]) % MOD
```

### 2. **Breaking Early in the Inner Loop**

**The Pitfall:**
Some might try to optimize by breaking the inner loop when `questions_used * points_per_question > score`:

```python
# Wrong optimization
for questions_used in range(max_questions + 1):
    points_from_current = questions_used * points_per_question
    if points_from_current > score:
        break  # This seems logical but is unnecessary
    dp[type_idx][score] = (dp[type_idx][score] + 
                           dp[type_idx - 1][score - points_from_current]) % MOD
```

**Why it can be problematic:**
- While not incorrect, it adds complexity without significant performance gain
- The condition check `if score >= points_from_current` already handles invalid cases
- Breaking early might make the code harder to understand and maintain

**The Solution:**
Keep the simple condition check without early termination. The clean approach in the original solution is preferred for clarity and correctness.

### 3. **Space Optimization Pitfall**

**The Pitfall:**
Attempting to optimize space by using a 1D array instead of 2D, but updating it incorrectly:

```python
# Wrong space optimization
dp = [0] * (target + 1)
dp[0] = 1

for type_idx in range(len(types)):
    max_questions, points_per_question = types[type_idx]
    for score in range(target + 1):  # Wrong direction!
        for questions_used in range(1, max_questions + 1):
            if score >= questions_used * points_per_question:
                dp[score] = (dp[score] + dp[score - questions_used * points_per_question]) % MOD
```

**Why it's problematic:**
- When iterating forward through scores, we might use already-updated values from the current iteration
- This leads to counting the same question type multiple times

**The Solution:**
If space optimization is needed, iterate through scores in reverse order:
```python
dp = [0] * (target + 1)
dp[0] = 1

for type_idx in range(len(types)):
    max_questions, points_per_question = types[type_idx]
    # Iterate in reverse to avoid using updated values
    for score in range(target, -1, -1):
        for questions_used in range(1, max_questions + 1):
            if score >= questions_used * points_per_question:
                dp[score] = (dp[score] + dp[score - questions_used * points_per_question]) % MOD
```

This ensures we only use values from the previous iteration, not the current one.

