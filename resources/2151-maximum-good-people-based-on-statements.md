# 2151. Maximum Good People Based on Statements

> Source: https://algo.monster/liteproblems/2151

## Problem Statement

You have `n` people, and each person can be either a **good person** (always tells the truth) or a **bad person** (might tell the truth or might lie).

You're given a 2D array `statements` of size `n x n` where `statements[i][j]` represents what person `i` says about person `j`:
- `0` means person `i` says person `j` is bad
- `1` means person `i` says person `j` is good  
- `2` means person `i` makes no statement about person `j`

No one makes statements about themselves, so `statements[i][i] = 2` for all `i`.

The key insight is that **good people always tell the truth**, while **bad people can say anything** (truth or lies).

Your task is to find the **maximum number of people who can be good** such that all statements are logically consistent.

For example, if person `i` is good and says person `j` is bad (`statements[i][j] = 0`), then person `j` must actually be bad. But if person `i` is bad, their statement can be ignored since bad people might lie.

## Intuition

The key insight is recognizing that **good people create constraints while bad people don't**. If someone is good, everything they say must be true. If someone is bad, we can completely ignore what they say.

This means we need to try different assumptions about who is good and who is bad, then check if our assumption is consistent. Think of it like a logic puzzle where we say "let's assume persons 0, 2, and 3 are good and everyone else is bad" - can this work given what the good people say?

Since we have a small number of people (at most 15), we can try all possible combinations. With `n` people, there are `2^n` ways to assign them as good or bad. We can represent each combination as a binary number where bit `i` indicates if person `i` is good (1) or bad (0).

For each combination (mask), we only need to validate statements from people we assumed are good. If person `i` is good (bit `i` is 1 in our mask) and they say person `j` is good (`statements[i][j] = 1`), then person `j` must actually be good in our assumption (bit `j` must be 1). Similarly, if they say person `j` is bad (`statements[i][j] = 0`), then bit `j` must be 0.

If any statement from a good person contradicts our assumption, this combination is invalid. Otherwise, we count how many people are good in this valid combination.

The brute force approach works because:
1. We can enumerate all possibilities efficiently with bit manipulation
2. Checking each combination is fast - just verify statements from assumed good people
3. The small constraint (`n ≤ 15`) makes `2^n` combinations manageable

**Learn more about [Backtracking](/problems/backtracking) patterns.**

## Solution Approach

The solution uses **bit masking** to enumerate all possible combinations of good and bad people, then validates each combination.

**Algorithm Overview:**
1. Generate all possible assignments of people as good (1) or bad (0) using numbers from `1` to `2^n - 1`
2. For each assignment (mask), check if it's valid
3. Track the maximum count of good people across all valid assignments

**Implementation Details:**

The main function iterates through all possible masks from `1` to `2^n - 1`:
```python
return max(check(i) for i in range(1, 1 << len(statements)))
```
Here, `1 << len(statements)` equals `2^n`, giving us all possible combinations.

The `check` function validates a specific mask and returns the count of good people if valid:

1. **Extract good people from mask**: For each person `i`, check if bit `i` is set using `mask >> i & 1`. If it's 1, person `i` is assumed good.

2. **Validate statements from good people**: 
   ```python
   if mask >> i & 1:  # Person i is good
       for j, x in enumerate(row):
           if x < 2 and (mask >> j & 1) != x:
               return 0
   ```
   - If person `i` is good, check all their statements (`row` = `statements[i]`)
   - For each statement about person `j` where `x < 2` (not a "no statement" case):
     - If `x = 1` (says j is good), then `mask >> j & 1` must be 1
     - If `x = 0` (says j is bad), then `mask >> j & 1` must be 0
   - If any statement contradicts our assumption, return 0 (invalid)

3. **Count good people**: If all statements from good people are consistent, count the number of 1-bits in the mask (number of good people).

**Why this works:**
- We only validate statements from people we assume are good because bad people can lie
- Each mask represents a complete hypothesis about who is good/bad
- By checking all `2^n` possibilities, we guarantee finding the maximum valid configuration

**Time Complexity:** `O(2^n × n^2)` - we check `2^n` masks, and for each mask, we potentially examine `n×n` statements.

**Space Complexity:** `O(1)` - only using variables for counting and iteration.

## Example Walkthrough

Let's walk through a small example with `n = 3` people and the following statements:
```
statements = [[2, 1, 2],
              [1, 2, 2],
              [2, 2, 2]]
```

This means:
- Person 0 says person 1 is good (statements[0][1] = 1)
- Person 1 says person 0 is good (statements[1][0] = 1)
- Person 2 makes no statements about anyone

**Step 1: Try different masks (combinations)**

We'll check masks from 1 to 7 (binary: 001 to 111). Each bit position represents a person (rightmost bit is person 0).

**Mask = 3 (binary: 011)**
- Person 0 is good (bit 0 = 1)
- Person 1 is good (bit 1 = 1)  
- Person 2 is bad (bit 2 = 0)

Validation:
- Person 0 is good and says person 1 is good → Check if person 1 is actually good in our mask → YES (bit 1 = 1) ✓
- Person 1 is good and says person 0 is good → Check if person 0 is actually good in our mask → YES (bit 0 = 1) ✓
- Person 2 is bad, so we ignore their statements

This mask is valid! Count of good people = 2.

**Mask = 5 (binary: 101)**
- Person 0 is good (bit 0 = 1)
- Person 1 is bad (bit 1 = 0)
- Person 2 is good (bit 2 = 1)

Validation:
- Person 0 is good and says person 1 is good → Check if person 1 is actually good in our mask → NO (bit 1 = 0) ✗

This mask is invalid because person 0 (who we assume is good) says person 1 is good, but in our mask person 1 is bad. This is a contradiction!

**Mask = 7 (binary: 111)**
- All three people are good

Validation:
- Person 0 is good and says person 1 is good → Check if person 1 is good → YES ✓
- Person 1 is good and says person 0 is good → Check if person 0 is good → YES ✓
- Person 2 is good but makes no statements → Nothing to validate ✓

This mask is valid! Count of good people = 3.

**Step 2: Return the maximum**

After checking all masks from 1 to 7, we find that masks 3 and 7 are valid with counts of 2 and 3 good people respectively. The maximum is 3.

The key insight: When we assume someone is good, their statements become constraints we must satisfy. When we assume someone is bad, we completely ignore what they say since they could be lying.

## Python Solution

```python
class Solution:
    def maximumGood(self, statements: List[List[int]]) -> int:
        """
        Find the maximum number of good people based on their statements.
      
        Args:
            statements: 2D array where statements[i][j] represents what person i says about person j
                       0 = person j is bad, 1 = person j is good, 2 = no statement
      
        Returns:
            Maximum number of people that can be good
        """
      
        def check_validity(bitmask: int) -> int:
            """
            Check if a given configuration of good/bad people is valid.
          
            Args:
                bitmask: Binary representation where bit i = 1 means person i is good
          
            Returns:
                Number of good people if configuration is valid, 0 otherwise
            """
            good_count = 0
          
            # Check each person's statements
            for person_idx, person_statements in enumerate(statements):
                # Check if current person is assumed to be good (bit is 1)
                if (bitmask >> person_idx) & 1:
                    # If person is good, all their statements must be true
                    for target_idx, statement_value in enumerate(person_statements):
                        # Skip if no statement made (value is 2)
                        if statement_value < 2:
                            # Get the assumed state of the target person
                            target_is_good = (bitmask >> target_idx) & 1
                          
                            # Check if statement matches the assumed state
                            if target_is_good != statement_value:
                                # Configuration is invalid
                                return 0
                  
                    good_count += 1
          
            return good_count
      
        # Try all possible configurations (excluding all bad scenario)
        n = len(statements)
        max_good_people = 0
      
        for configuration in range(1, 1 << n):
            max_good_people = max(max_good_people, check_validity(configuration))
      
        return max_good_people
```

## Time and Space Complexity

**Time Complexity:** `O(n² * 2^n)` where `n` is the number of people (length of statements array).

- The outer loop iterates through all possible masks from 1 to `2^n - 1`, which gives us `O(2^n)` iterations
- For each mask, the `check` function is called:
  - It iterates through all `n` people to check if they are marked as "good" in the current mask
  - For each "good" person (bit set to 1), it iterates through their `n` statements about other people
  - In the worst case, all people could be marked as good, resulting in `O(n²)` operations per mask
- Overall: `O(2^n) * O(n²) = O(n² * 2^n)`

**Space Complexity:** `O(1)` excluding the input array.

- The algorithm uses only a constant amount of extra space for variables like `mask`, `cnt`, `i`, `j`, and `x`
- The recursive call stack is not used as this is an iterative solution
- The generator expression `max(check(i) for i in range(...))` evaluates one value at a time and doesn't store all results simultaneously
- Therefore, the space complexity is `O(1)` auxiliary space

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Starting iteration from 0 instead of 1**
A common mistake is iterating through all masks starting from 0:
```python
# WRONG - includes the case where everyone is bad
for configuration in range(0, 1 << n):  
    max_good_people = max(max_good_people, check_validity(configuration))
```

**Why it's wrong:** When `configuration = 0`, all bits are 0, meaning everyone is assumed to be bad. In this case, there are no good people making statements, so any configuration would be "valid" by default (no statements to verify). This could incorrectly return 0 as a valid answer when there might be valid configurations with good people.

**Solution:** Start from 1 to ensure at least one person is good:
```python
# CORRECT
for configuration in range(1, 1 << n):
    max_good_people = max(max_good_people, check_validity(configuration))
```

### 2. **Incorrectly checking if a person is good in the bitmask**
Beginners often make errors with bit manipulation:
```python
# WRONG - checks if entire mask equals 1
if bitmask & 1:  

# WRONG - forgets to shift before checking
if bitmask & person_idx:  
```

**Solution:** Use proper bit shifting and masking:
```python
# CORRECT - shifts right by person_idx positions, then checks least significant bit
if (bitmask >> person_idx) & 1:
```

### 3. **Validating statements from bad people**
A critical error is checking statements from people assumed to be bad:
```python
# WRONG - validates everyone's statements
for person_idx, person_statements in enumerate(statements):
    for target_idx, statement_value in enumerate(person_statements):
        if statement_value < 2:
            target_is_good = (bitmask >> target_idx) & 1
            if target_is_good != statement_value:
                return 0
```

**Why it's wrong:** Bad people can lie, so their statements don't need to be consistent with reality. Only good people's statements must be validated.

**Solution:** Only validate statements from people marked as good:
```python
# CORRECT
if (bitmask >> person_idx) & 1:  # Only check if person is good
    for target_idx, statement_value in enumerate(person_statements):
        # ... validation logic
```

### 4. **Using wrong comparison for statement validation**
The statement value directly represents whether the target is good (1) or bad (0):
```python
# WRONG - inverted logic
if statement_value == 0 and target_is_good == 1:
    return 0
if statement_value == 1 and target_is_good == 0:
    return 0

# WRONG - treating statement_value as a boolean incorrectly
if bool(statement_value) != bool(target_is_good):
    return 0
```

**Solution:** Direct comparison since both use the same encoding (1 = good, 0 = bad):
```python
# CORRECT
if target_is_good != statement_value:
    return 0
```

### 5. **Not handling the "no statement" case properly**
Forgetting to skip when `statement_value == 2`:
```python
# WRONG - tries to validate even "no statement" cases
for target_idx, statement_value in enumerate(person_statements):
    target_is_good = (bitmask >> target_idx) & 1
    if target_is_good != statement_value:  # This fails when statement_value = 2
        return 0
```

**Solution:** Skip validation when no statement is made:
```python
# CORRECT
if statement_value < 2:  # Only process actual statements (0 or 1)
    target_is_good = (bitmask >> target_idx) & 1
    if target_is_good != statement_value:
        return 0
```

