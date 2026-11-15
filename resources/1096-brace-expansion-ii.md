# 1096. Brace Expansion II

> Source: https://algo.monster/liteproblems/1096

## Problem Statement

This problem asks you to parse and evaluate a special grammar that represents sets of lowercase words. Given an expression string, you need to return all possible words that can be generated from it, sorted in lexicographical order.

The grammar has three rules:

1. **Single letters**: Any lowercase letter by itself represents a set containing just that letter. For example, `"a"` represents the set `{"a"}`.

2. **Union with braces and commas**: When expressions are enclosed in braces and separated by commas like `{expr1,expr2,...}`, this represents the union of all the sets. For example:
   - `"{a,b,c}"` represents `{"a", "b", "c"}`
   - `"{{a,b},{b,c}}"` represents `{"a", "b", "c"}` (duplicates are removed in the union)

3. **Concatenation**: When expressions are placed next to each other, they are concatenated. This means taking every possible combination where you pick one word from the first set and one from the second set, then concatenate them. For example:
   - `"{a,b}{c,d}"` represents all combinations: `{"ac", "ad", "bc", "bd"}`
   - `"a{b,c}{d,e}f{g,h}"` breaks down as:
     - Start with `"a"`
     - Concatenate with either `"b"` or `"c"`
     - Concatenate with either `"d"` or `"e"`
     - Concatenate with `"f"`
     - Concatenate with either `"g"` or `"h"`
     - This gives 8 total combinations: `{"abdfg", "abdfh", "abefg", "abefh", "acdfg", "acdfh", "acefg", "acefh"}`

## Intuition

The key insight is to process the expression from the inside out, similar to how we evaluate mathematical expressions with parentheses. When we see nested braces like `"a{b,{c,d}e}"`, we need to first resolve the innermost braces before handling the outer ones.

Think of the expression as layers of an onion. We peel away one layer at a time by finding the innermost brace pair (a pair with no other braces inside it) and expanding it. For example, in `"{a,b}{c,d}"`, both brace pairs are at the same "depth" and contain no nested braces, so we can expand either one first.

The recursive approach works by:
1. Finding the last `}` in the expression
2. Finding the corresponding `{` that opens this brace (the nearest one to the left)
3. This guarantees we've found an innermost brace pair with no nested braces inside

Once we identify an innermost brace pair like `{a,b,c}`, we expand it by creating multiple versions of the expression - one for each option inside the braces. If our expression is `"x{a,b,c}y"`, we create three new expressions: `"xay"`, `"xby"`, and `"xcy"`.

We continue this process recursively on each generated expression until no braces remain. At that point, we have a simple concatenated string that we add to our result set. Using a set automatically handles duplicates for us, which is important since different expansion paths might lead to the same final string.

The beauty of this approach is its simplicity - by always working with the innermost braces first, we avoid the complexity of parsing nested structures. Each recursive call either finds another brace pair to expand or adds a final result to our set. The sorted return ensures we meet the problem's requirement for lexicographical ordering.

**Learn more about [Stack](/problems/stack_intro), [Breadth-First Search](/problems/bfs_intro) and [Backtracking](/problems/backtracking) patterns.**

## Solution Approach

The implementation uses a recursive depth-first search approach with a set to store unique results. Let's walk through the key components:

**Data Structure:**
- A `set` called `s` to store all unique expanded strings (automatically handles duplicates)

**Main Algorithm - The `dfs` function:**

1. **Base Case Detection:** 
   - First, check if there are any closing braces `}` in the expression using `j = exp.find('}')`
   - If `j == -1` (no braces found), we have a fully expanded string, so add it to the set and return

2. **Finding Innermost Braces:**
   - Find the first closing brace position: `j = exp.find('}')`
   - Find the corresponding opening brace by searching backwards from position `j-1`: `i = exp.rfind('{', 0, j - 1)`
   - This guarantees we found an innermost pair since `rfind` searches from right to left, finding the closest `{` to our `}`

3. **String Decomposition:**
   - Split the expression into three parts:
     - `a = exp[:i]` - everything before the opening brace
     - The content between braces: `exp[i + 1 : j]`
     - `c = exp[j + 1:]` - everything after the closing brace

4. **Expansion Process:**
   - Split the content between braces by commas: `exp[i + 1 : j].split(',')`
   - For each option `b` in the comma-separated list:
     - Create a new expression: `a + b + c`
     - Recursively call `dfs` on this new expression

5. **Final Processing:**
   - After all recursive calls complete, the set `s` contains all unique expanded strings
   - Return the sorted list: `sorted(s)`

**Example Walkthrough with `"{a,b}{c,d}"`:**

- Initial call: `dfs("{a,b}{c,d}")`
- Finds `j=4` (first `}`), `i=0` (corresponding `{`)
- Splits into: `a=""`, options=`["a","b"]`, `c="{c,d}"`
- Creates two recursive calls:
  - `dfs("a{c,d}")` → finds inner braces, expands to `dfs("ac")` and `dfs("ad")`
  - `dfs("b{c,d}")` → finds inner braces, expands to `dfs("bc")` and `dfs("bd")`
- Base cases add `"ac"`, `"ad"`, `"bc"`, `"bd"` to the set
- Returns `["ac", "ad", "bc", "bd"]` after sorting

The elegance of this solution lies in its simplicity - by always processing the innermost braces first, we avoid complex parsing logic and naturally handle all nested structures through recursion.

## Example Walkthrough

Let's trace through the expression `"k{a,e}l{l{p,q}}"` step by step to see how the recursive solution works:

**Initial Call:** `dfs("k{a,e}l{l{p,q}}")`
- Find last `}`: position 14
- Find matching `{`: position 10 (using rfind from position 13)
- This gives us the innermost brace pair `{p,q}`
- Split: `a = "k{a,e}l{l"`, options = `["p", "q"]`, `c = "}"`
- Create two recursive calls:
  - `dfs("k{a,e}l{lp}")`
  - `dfs("k{a,e}l{lq}")`

**First Branch:** `dfs("k{a,e}l{lp}")`
- Find last `}`: position 9
- Find matching `{`: position 8
- This identifies `{lp}` as innermost
- Split: `a = "k{a,e}l"`, options = `["lp"]`, `c = ""`
- Create one recursive call:
  - `dfs("k{a,e}llp")`

**Continuing:** `dfs("k{a,e}llp")`
- Find last `}`: position 5
- Find matching `{`: position 1
- This identifies `{a,e}` as innermost
- Split: `a = "k"`, options = `["a", "e"]`, `c = "llp"`
- Create two recursive calls:
  - `dfs("kallp")` - no braces, add to set
  - `dfs("kellp")` - no braces, add to set

**Second Branch:** `dfs("k{a,e}l{lq}")`
- Following similar logic:
  - Expands `{lq}` → `dfs("k{a,e}llq")`
  - Expands `{a,e}` → `dfs("kallq")` and `dfs("kellq")`
  - Both base cases add to set

**Final Result:**
- Set contains: `{"kallp", "kellp", "kallq", "kellq"}`
- After sorting: `["kallp", "kallq", "kellp", "kellq"]`

The key insight is that by always finding the last `}` and its matching `{`, we guarantee we're working with innermost braces that contain no nested structures, making the expansion straightforward.

## Python Solution

```python
class Solution:
    def braceExpansionII(self, expression: str) -> List[str]:
        def dfs(current_expression):
            # Find the first closing brace
            closing_brace_index = current_expression.find('}')
          
            # Base case: no more braces to expand
            if closing_brace_index == -1:
                result_set.add(current_expression)
                return
          
            # Find the matching opening brace for the first closing brace
            # Search backwards from just before the closing brace
            opening_brace_index = current_expression.rfind('{', 0, closing_brace_index - 1)
          
            # Extract parts: before the brace, inside the brace, and after the brace
            prefix = current_expression[:opening_brace_index]
            suffix = current_expression[closing_brace_index + 1:]
            brace_content = current_expression[opening_brace_index + 1:closing_brace_index]
          
            # Expand each option within the braces
            for option in brace_content.split(','):
                # Recursively process the expression with this option substituted
                dfs(prefix + option + suffix)
      
        # Initialize result set to store unique expanded strings
        result_set = set()
      
        # Start the depth-first search expansion
        dfs(expression)
      
        # Return sorted list of all unique expanded strings
        return sorted(result_set)
```

## Time and Space Complexity

**Time Complexity:** `O(n * 2^m * m)` where `n` is the length of the expression and `m` is the number of choice points (comma-separated options within braces).

The algorithm recursively expands each brace group from innermost to outermost. In the worst case:
- Finding `}` with `find()`: `O(n)` per recursive call
- Finding `{` with `rfind()`: `O(n)` per recursive call  
- String slicing and concatenation `a + b + c`: `O(n)` per recursive call
- Splitting by comma: `O(n)` per recursive call
- Each brace group with `k` comma-separated options creates `k` recursive branches
- With `m` total choice points across all brace groups, this creates up to `2^m` total recursive paths (product of all choices)
- Each recursive call does `O(n)` work
- Total: `O(n * 2^m * m)` considering the depth of recursion

**Space Complexity:** `O(n * 2^m)` where `n` is the length of the expression and `m` is the number of choice points.

The space is consumed by:
- Recursion stack depth: `O(m)` in the worst case (one level per brace group)
- String storage in each recursive call: `O(n)` per call
- Set `s` storing unique results: up to `2^m` unique strings, each of length up to `O(n)`
- Total: `O(n * 2^m)` dominated by the storage of all possible expanded strings in the set

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Brace Matching with Nested Structures**

The most critical pitfall is incorrectly matching opening and closing braces when there are nested structures. Consider the expression `"{{a,b},{c,d}}e"`:

**Wrong Approach:**
```python
# If we naively find the first '{' and first '}'
opening = expression.find('{')  # finds position 0
closing = expression.find('}')  # finds position 6
# This would incorrectly pair the outermost '{' with the first inner '}'
```

**Why the Current Solution Works:**
The solution cleverly avoids this by:
1. First finding the closing brace using `find('}')`
2. Then searching **backwards** from that position using `rfind('{', 0, j-1)`
3. This ensures we always get the innermost brace pair

### 2. **Handling Adjacent Concatenations Without Braces**

Another pitfall occurs when trying to handle expressions like `"ab{c,d}ef"` where letters are directly concatenated without braces.

**Potential Issue:**
If the algorithm tried to process concatenations separately from brace expansion, it would need complex logic to determine where one "unit" ends and another begins.

**How It's Avoided:**
The current solution treats everything outside braces as literal strings that get carried along (`prefix` and `suffix`), naturally handling concatenation through the recursive structure.

### 3. **Memory Issues with Large Expressions**

For expressions with many nested braces and options, the set can grow exponentially.

**Example Problem:**
```python
# This expression generates 2^10 = 1024 unique strings
expression = "{a,b}{c,d}{e,f}{g,h}{i,j}{k,l}{m,n}{o,p}{q,r}{s,t}"
```

**Mitigation Strategy:**
While the current solution handles this correctly, for very large inputs you might need to:
- Check expression length limits early
- Consider iterative approaches with explicit stack management
- Monitor memory usage during recursion

### 4. **Edge Cases with Empty Options**

A subtle pitfall is handling expressions with empty options or consecutive commas:

**Problematic Input:**
```python
expression = "{a,,b}"  # Empty string between commas
expression = "{,a,b}"  # Empty string at start
```

**Solution Enhancement:**
```python
def dfs(current_expression):
    closing_brace_index = current_expression.find('}')
  
    if closing_brace_index == -1:
        # Only add non-empty results
        if current_expression:  # Add this check
            result_set.add(current_expression)
        return
  
    opening_brace_index = current_expression.rfind('{', 0, closing_brace_index - 1)
    prefix = current_expression[:opening_brace_index]
    suffix = current_expression[closing_brace_index + 1:]
    brace_content = current_expression[opening_brace_index + 1:closing_brace_index]
  
    for option in brace_content.split(','):
        # Skip empty options if needed
        if option or (not prefix and not suffix):  # Allow empty only if it's the entire result
            dfs(prefix + option + suffix)
```

### 5. **Assumption About Input Validity**

The solution assumes well-formed input (balanced braces, proper comma placement). For production code, validation would be important:

**Adding Input Validation:**
```python
def validate_expression(expression):
    # Check for balanced braces
    brace_count = 0
    for char in expression:
        if char == '{':
            brace_count += 1
        elif char == '}':
            brace_count -= 1
            if brace_count < 0:
                return False
    return brace_count == 0
```

