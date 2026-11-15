# 301. Remove Invalid Parentheses

> Source: https://algo.monster/liteproblems/301

## Problem Statement

You are given a string `s` that contains parentheses (both opening '(' and closing ')') along with letters. The goal is to remove the minimum number of parentheses to make the string valid.

A string is considered valid when:
- Every opening parenthesis '(' has a corresponding closing parenthesis ')'
- Every closing parenthesis ')' has a corresponding opening parenthesis '(' that comes before it
- The parentheses are properly balanced

The task requires you to:
1. Find all possible valid strings that can be formed by removing the minimum number of parentheses
2. Return all unique valid strings (no duplicates)
3. The answer can be returned in any order

For example:
- If `s = "()())()"`, you need to remove 1 parenthesis. Possible answers: `["(())()","()()()"]`
- If `s = "(a)())()"`, you need to remove 1 parenthesis. Possible answers: `["(a())()","(a)()()"]`
- If `s = ")("`, you need to remove 2 parentheses. Answer: `[""]`

## Intuition

The key insight is to think of this problem as exploring a state space where each state is a string, and we want to find all valid strings that are reachable with the minimum number of character removals.

Imagine starting with the original string and considering all possible ways to remove parentheses. Each removal creates a new string (a new state). If we remove one character, we get strings that are one step away. If we remove two characters, we get strings that are two steps away, and so on.

Since we want the minimum number of removals, we need to explore this state space level by level - first check all strings with 0 removals (just the original), then all strings with 1 removal, then 2 removals, and so forth. This is exactly what BFS does - it explores nodes level by level, guaranteeing we find the closest valid strings first.

However, the provided solution actually uses DFS with a clever optimization. Instead of blindly exploring all possibilities, it first calculates exactly how many left `(` and right `)` parentheses need to be removed. This is done by:
1. Scanning the string once to identify unmatched parentheses
2. For each `(`, increment a counter
3. For each `)`, if there's a matching `(` (counter > 0), decrement the counter; otherwise, it's an unmatched `)`
4. After scanning, we know exactly `l` left and `r` right parentheses must be removed

With this knowledge, the DFS explores only combinations that remove exactly `l` left parentheses and `r` right parentheses. During the traversal, it maintains counts of left and right parentheses (`lcnt`, `rcnt`) to ensure we never have more `)` than `(` at any point (which would make the string invalid).

The pruning conditions `n - i < l + r` (not enough characters left to remove) and `lcnt < rcnt` (invalid state) help eliminate unnecessary branches early, making the search efficient despite using DFS instead of BFS.

## Solution Approach

The implementation uses a DFS approach with intelligent pruning to find all valid strings with minimum removals.

**Step 1: Calculate Required Removals**

First, we determine exactly how many left and right parentheses need to be removed:
```python
l = r = 0
for c in s:
    if c == '(':
        l += 1
    elif c == ')':
        if l:
            l -= 1  # Found a matching left parenthesis
        else:
            r += 1  # Unmatched right parenthesis
```
- `l`: count of unmatched left parentheses to remove
- `r`: count of unmatched right parentheses to remove

**Step 2: DFS Exploration**

The DFS function explores all ways to remove exactly `l` left and `r` right parentheses:
```python
def dfs(i, l, r, lcnt, rcnt, t):
```
Parameters:
- `i`: current index in the string
- `l`, `r`: remaining left and right parentheses to remove
- `lcnt`, `rcnt`: count of left and right parentheses in the current built string
- `t`: the string being built

**Step 3: Base Cases and Pruning**

1. **Base case**: When we've processed all characters (`i == n`):
   - If `l == 0` and `r == 0` (removed exact required amount), add to result set

2. **Pruning conditions**:
   - `n - i < l + r`: Not enough characters left to remove required parentheses
   - `lcnt < rcnt`: Invalid state where we have more `)` than `(` so far

**Step 4: Recursive Choices**

At each position, we have two choices:

1. **Remove current character** (if it's a parenthesis we need to remove):
   - If `s[i] == '('` and `l > 0`: skip this `(` and decrement `l`
   - If `s[i] == ')'` and `r > 0`: skip this `)` and decrement `r`

2. **Keep current character**:
   - Add to the built string `t`
   - Update `lcnt` or `rcnt` if it's a parenthesis
   - Continue to next position

**Step 5: Collecting Results**

- Use a `set` to automatically handle duplicates (different removal combinations might produce the same valid string)
- Convert to list before returning

The algorithm efficiently explores only the necessary branches by:
- Pre-calculating exact removal requirements
- Maintaining validity checks during construction
- Pruning impossible branches early
- Using a set to handle duplicate results automatically

## Example Walkthrough

Let's walk through the example `s = "()())()"` step by step.

**Step 1: Calculate Required Removals**

We scan through the string to find unmatched parentheses:
- Index 0: '(' → `l = 1` (unmatched left)
- Index 1: ')' → `l = 0` (matched with previous '(')
- Index 2: '(' → `l = 1` (unmatched left)
- Index 3: ')' → `l = 0` (matched with previous '(')
- Index 4: ')' → `r = 1` (no unmatched '(' to pair with)
- Index 5: '(' → `l = 1` (unmatched left)
- Index 6: ')' → `l = 0` (matched with previous '(')

Final counts: `l = 0`, `r = 1` (we need to remove 1 right parenthesis)

**Step 2: DFS Exploration**

Starting DFS with `dfs(0, 0, 1, 0, 0, "")`, so exactly one `)` must be removed. Since `l = 0`, a `(` is never removed; at each `)` the search can either remove it (while `r > 0`) or keep it. Pruning stops any branch where the kept prefix has more `)` than `(`.

**Branch A: remove the `)` at index 1**
- Keep `(` at index 0: `"("`, lcnt=1, rcnt=0
- Remove `)` at index 1: still `"("`, r=0
- Keep `(` at index 2: `"(("`, lcnt=2, rcnt=0
- Keep `)` at index 3: `"(()"`, lcnt=2, rcnt=1
- Keep `)` at index 4: `"(())"`, lcnt=2, rcnt=2
- Keep `(` at index 5 and `)` at index 6: `"(())()"`
- End of string with l=0, r=0 → add `"(())()"`

**Branch B: remove the `)` at index 3**
- Keep indices 0-2: `"()("`, lcnt=2, rcnt=1
- Remove `)` at index 3: still `"()("`, r=0
- Keep `)` at index 4: `"()()"`, lcnt=2, rcnt=2
- Keep indices 5-6: `"()()()"`
- End of string with l=0, r=0 → add `"()()()"`

**Branch C: remove the `)` at index 4**
- Keep indices 0-3: `"()()"`, lcnt=2, rcnt=2
- Remove `)` at index 4: still `"()()"`, r=0
- Keep indices 5-6: `"()()()"`
- End of string → add `"()()()"` again; the set keeps only one copy

**Pruned branches**
- Keeping all of indices 0-4 gives `"()())"` with lcnt=2, rcnt=3, so the next call sees `lcnt < rcnt` and returns
- Removing the `)` at index 6 is never reached with `r = 1`, because every path that keeps index 4 was already pruned and every other path has used up its one removal

**Step 3: Final Results**

The algorithm finds two valid strings with exactly 1 removal:
- `"()()()"` - reached by removing the ')' at index 3 or the one at index 4
- `"(())()"` - removed the ')' at index 1

These are collected in a set (avoiding duplicates) and returned as `["()()()","(())()"]`.

## Python Solution

```python
class Solution:
    def removeInvalidParentheses(self, s: str) -> List[str]:
        def dfs(index: int, left_to_remove: int, right_to_remove: int, 
                left_count: int, right_count: int, current_string: str) -> None:
            """
            Recursively explore all possible ways to remove invalid parentheses.
          
            Args:
                index: Current position in the original string
                left_to_remove: Number of '(' that still need to be removed
                right_to_remove: Number of ')' that still need to be removed
                left_count: Count of '(' in the current valid string being built
                right_count: Count of ')' in the current valid string being built
                current_string: The string being constructed
            """
            # Base case: reached end of string
            if index == string_length:
                # If all invalid parentheses have been removed, add to result
                if left_to_remove == 0 and right_to_remove == 0:
                    valid_results.add(current_string)
                return
          
            # Pruning conditions:
            # 1. Not enough characters left to remove required parentheses
            # 2. More closing than opening parentheses (invalid state)
            if string_length - index < left_to_remove + right_to_remove or left_count < right_count:
                return
          
            # Option 1: Remove current '(' if it needs to be removed
            if s[index] == '(' and left_to_remove > 0:
                dfs(index + 1, left_to_remove - 1, right_to_remove, 
                    left_count, right_count, current_string)
          
            # Option 2: Remove current ')' if it needs to be removed
            elif s[index] == ')' and right_to_remove > 0:
                dfs(index + 1, left_to_remove, right_to_remove - 1, 
                    left_count, right_count, current_string)
          
            # Option 3: Keep current character (always explore this option)
            # Update counts if the character is a parenthesis
            new_left_count = left_count + (1 if s[index] == '(' else 0)
            new_right_count = right_count + (1 if s[index] == ')' else 0)
            dfs(index + 1, left_to_remove, right_to_remove, 
                new_left_count, new_right_count, current_string + s[index])
      
        # Step 1: Calculate how many left and right parentheses need to be removed
        left_to_remove = 0
        right_to_remove = 0
      
        for char in s:
            if char == '(':
                left_to_remove += 1
            elif char == ')':
                if left_to_remove > 0:
                    # This ')' can be paired with a previous '('
                    left_to_remove -= 1
                else:
                    # This ')' has no matching '(', needs to be removed
                    right_to_remove += 1
      
        # Initialize result set (using set to avoid duplicates)
        valid_results = set()
        string_length = len(s)
      
        # Start DFS exploration
        dfs(0, left_to_remove, right_to_remove, 0, 0, '')
      
        # Convert set to list for return
        return list(valid_results)
```

## Time and Space Complexity

**Time Complexity:** `O(2^n)`

The algorithm uses a DFS approach where at each position, we have up to 2 choices:
- Skip the current character (when it's a parenthesis that needs to be removed)
- Include the current character

In the worst case, we explore all possible subsequences of the string. Although there are pruning conditions (`n - i < l + r` and `lcnt < rcnt`) that help reduce the search space, the worst-case scenario still requires exploring an exponential number of possibilities. Each branch of the recursion tree can split into at most 2 branches, and the maximum depth is `n`, resulting in `O(2^n)` time complexity.

**Space Complexity:** `O(n)`

The space complexity consists of:
- **Recursion stack:** The maximum depth of recursion is `n` (the length of the string), contributing `O(n)` to space complexity
- **String building:** The parameter `t` in each recursive call can be at most length `n`, contributing `O(n)` 
- **Result storage:** The `ans` set stores valid strings. In the worst case, there could be multiple valid results, but each result string has length at most `n`. The number of valid results is typically much smaller than `2^n` due to the constraints of valid parentheses
- **Variables:** The variables `l`, `r`, `lcnt`, `rcnt` use `O(1)` space

The dominant factor is the recursion stack depth and the string building during recursion, giving us an overall space complexity of `O(n)`.

## Common Pitfalls

### 1. **Incorrect Handling of Multiple Removal Options**

**Pitfall**: When a parenthesis can be removed, the search must try BOTH removing it and keeping it. A common mistake is attaching the "keep" call to the removal branches with `else`, so a removable parenthesis is never kept.

**Incorrect Code**:
```python
if s[index] == '(' and left_to_remove > 0:
    dfs(index + 1, left_to_remove - 1, right_to_remove,
        left_count, right_count, current_string)
elif s[index] == ')' and right_to_remove > 0:
    dfs(index + 1, left_to_remove, right_to_remove - 1,
        left_count, right_count, current_string)
else:  # WRONG: keep only when no removal was possible
    dfs(index + 1, left_to_remove, right_to_remove,
        new_left_count, new_right_count, current_string + s[index])
```

**Why it's wrong**: With `r = 1` for `"()())()"`, the first `)` is always removed and never kept, so only `"(())()"` is found and `"()()()"` is missed. Using `elif` between the two removal options is fine (a character is either `(` or `)`); what matters is that the keep call runs unconditionally.

**Correct Approach**:
```python
# Option 1: Remove current '(' if it needs to be removed
if s[index] == '(' and left_to_remove > 0:
    dfs(index + 1, left_to_remove - 1, right_to_remove, 
        left_count, right_count, current_string)

# Option 2: Remove current ')' if it needs to be removed (elif works the same here)
if s[index] == ')' and right_to_remove > 0:
    dfs(index + 1, left_to_remove, right_to_remove - 1, 
        left_count, right_count, current_string)

# Option 3: Always try keeping the current character
new_left_count = left_count + (1 if s[index] == '(' else 0)
new_right_count = right_count + (1 if s[index] == ')' else 0)
dfs(index + 1, left_to_remove, right_to_remove, 
    new_left_count, new_right_count, current_string + s[index])
```

### 2. **Forgetting Early Termination Check**

**Pitfall**: Not checking if `right_count > left_count` during DFS traversal. This is not only slower: it lets invalid strings through, because removing the right number of parentheses does not guarantee that the rest is balanced in order. For `s = "())("` (one `(` and one `)` to remove), removing indices 0 and 1 leaves `")("`, which this version adds to the results.

**Incorrect Code**:
```python
def dfs(index, left_to_remove, right_to_remove, left_count, right_count, current_string):
    if index == string_length:
        if left_to_remove == 0 and right_to_remove == 0:
            valid_results.add(current_string)
        return
  
    # Missing the validity check!
    if string_length - index < left_to_remove + right_to_remove:
        return
```

**Correct Code**:
```python
def dfs(index, left_to_remove, right_to_remove, left_count, right_count, current_string):
    if index == string_length:
        if left_to_remove == 0 and right_to_remove == 0:
            valid_results.add(current_string)
        return
  
    # Include both pruning conditions
    if string_length - index < left_to_remove + right_to_remove or left_count < right_count:
        return
```

### 3. **Using List Instead of Set for Results**

**Pitfall**: Using a list to store results and trying to manually check for duplicates, which is inefficient and error-prone.

**Incorrect Approach**:
```python
valid_results = []  # Using list

def dfs(...):
    if index == string_length:
        if left_to_remove == 0 and right_to_remove == 0:
            if current_string not in valid_results:  # O(n) check each time!
                valid_results.append(current_string)
        return
```

**Correct Approach**:
```python
valid_results = set()  # Using set for automatic deduplication

def dfs(...):
    if index == string_length:
        if left_to_remove == 0 and right_to_remove == 0:
            valid_results.add(current_string)  # O(1) add with automatic dedup
        return

# Convert to list only at the end
return list(valid_results)
```

### 4. **Incorrect Initial Count Calculation**

**Pitfall**: Not properly tracking which parentheses can be paired during the initial scan.

**Incorrect Code**:
```python
# Simply counting all parentheses
left_to_remove = s.count('(')
right_to_remove = s.count(')')
```

**Correct Code**:
```python
left_to_remove = 0
right_to_remove = 0

for char in s:
    if char == '(':
        left_to_remove += 1
    elif char == ')':
        if left_to_remove > 0:
            left_to_remove -= 1  # This ')' pairs with a previous '('
        else:
            right_to_remove += 1  # This ')' has no match, must be removed
```

