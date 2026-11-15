# 1153. String Transforms Into Another String

> Source: https://algo.monster/liteproblems/1153

## Problem Statement

You are given two strings `str1` and `str2` of the same length. Your task is to determine whether you can transform `str1` into `str2` by performing zero or more conversions.

In each conversion, you can select one character in `str1` and convert **all** occurrences of that character to any other lowercase English character. For example, if `str1 = "aabcc"`, you could convert all `'a'`s to `'d'`, resulting in `"ddbcc"`.

The key constraints are:
- When you convert a character, you must convert **all** occurrences of that character at once
- You can only convert to lowercase English letters
- You can perform multiple conversions in sequence
- Each character can only be mapped to one other character (you cannot map `'a'` to both `'b'` and `'c'`)

Return `true` if it's possible to transform `str1` into `str2` through such conversions, and `false` otherwise.

For example:
- `str1 = "aabcc"`, `str2 = "ccdee"` → `true` (convert `'a'→'c'`, `'b'→'d'`, `'c'→'e'`)
- `str1 = "leetcode"`, `str2 = "codeleet"` → `false` (impossible transformation)

## Intuition

To understand if we can transform `str1` into `str2`, let's think about what each conversion means. When we convert all occurrences of a character `'a'` to `'b'`, we're essentially creating a mapping: `'a' → 'b'`. This mapping must be consistent - we cannot map `'a'` to different characters.

The first observation is straightforward: if `str1` and `str2` are already equal, no conversion is needed, so we return `true`.

The critical insight comes from considering when a transformation is impossible. Imagine `str2` contains all 26 lowercase letters. In this case, we need 26 distinct mappings to reach `str2`. However, if we're transforming characters, we might create conflicts. For instance, if we need to map both `'a' → 'z'` and `'b' → 'z'` (because different positions require it), we'd have two characters mapping to the same target, which could work. But the real problem arises when we need to use all 26 letters as targets - we have no "spare" letter to use as an intermediate step for chained conversions.

Think of it like a parking lot puzzle where you need one empty space to move cars around. If all 26 letter positions are occupied in `str2`, we have no room to maneuver our transformations without creating conflicts.

For the main checking logic, we need to ensure consistency: each character in `str1` can only map to one character in `str2`. As we traverse both strings position by position, we record what each character in `str1` should transform to. If we encounter the same character in `str1` later but it needs to map to a different character in `str2`, the transformation is impossible.

This leads us to use a hash table to track mappings: for each character in `str1`, we store what it should transform to. If we see a contradiction (same source character needing to map to different targets), we return `false`.

**Learn more about [Graph](/problems/graph_intro) patterns.**

## Solution Approach

Let's walk through the implementation step by step:

**Step 1: Check if strings are already equal**
```python
if str1 == str2:
    return True
```
If the strings are identical, no transformation is needed.

**Step 2: Check if all 26 letters are used in `str2`**
```python
if len(set(str2)) == 26:
    return False
```
We convert `str2` to a set to count unique characters. If `str2` contains all 26 lowercase letters, transformation is impossible because we have no "spare" letter to use as an intermediate step for conversions.

**Step 3: Build and validate character mappings**
```python
d = {}
for a, b in zip(str1, str2):
    if a not in d:
        d[a] = b
    elif d[a] != b:
        return False
```

We use a hash table `d` to track the mapping from each character in `str1` to its corresponding character in `str2`.

The algorithm works as follows:
- We iterate through both strings simultaneously using `zip(str1, str2)`
- For each pair of characters `(a, b)` where `a` is from `str1` and `b` is from `str2`:
  - If character `a` hasn't been seen before (`a not in d`), we record its mapping: `d[a] = b`
  - If character `a` has been seen before, we check if its previous mapping matches the current requirement
  - If `d[a] != b`, it means character `a` needs to map to different characters at different positions, which is impossible (one-to-many mapping), so we return `False`

**Step 4: Return success**
```python
return True
```
If we complete the iteration without finding any conflicts, the transformation is possible.

The time complexity is `O(n)` where `n` is the length of the strings, as we traverse them once. The space complexity is `O(1)` since the hash table can store at most 26 mappings (one for each lowercase letter).

## Example Walkthrough

Let's walk through the solution with `str1 = "aabcc"` and `str2 = "ccdee"`.

**Step 1: Check if strings are equal**
- `"aabcc" == "ccdee"` → False, so we continue.

**Step 2: Check if str2 uses all 26 letters**
- `set(str2) = {'c', 'd', 'e'}` → 3 unique characters
- Since 3 ≠ 26, we have room to perform transformations, so we continue.

**Step 3: Build character mappings**

We'll iterate through both strings position by position:

| Position | str1[i] | str2[i] | Action | Hash table `d` |
|----------|---------|---------|--------|----------------|
| 0 | 'a' | 'c' | 'a' not in d, add mapping | {'a': 'c'} |
| 1 | 'a' | 'c' | 'a' in d, check: d['a']='c' matches 'c' ✓ | {'a': 'c'} |
| 2 | 'b' | 'd' | 'b' not in d, add mapping | {'a': 'c', 'b': 'd'} |
| 3 | 'c' | 'e' | 'c' not in d, add mapping | {'a': 'c', 'b': 'd', 'c': 'e'} |
| 4 | 'c' | 'e' | 'c' in d, check: d['c']='e' matches 'e' ✓ | {'a': 'c', 'b': 'd', 'c': 'e'} |

**Step 4: Return result**
- No conflicts were found during the mapping process
- Return `True`

The transformation is possible with the mappings: 'a'→'c', 'b'→'d', 'c'→'e'.

---

Now let's see a failing example with `str1 = "ab"` and `str2 = "aa"`:

**Step 1:** `"ab" ≠ "aa"`, continue.

**Step 2:** `set("aa") = {'a'}`, only 1 unique character (not 26), continue.

**Step 3:** Build mappings:

| Position | str1[i] | str2[i] | Action | Hash table `d` |
|----------|---------|---------|--------|----------------|
| 0 | 'a' | 'a' | 'a' not in d, add mapping | {'a': 'a'} |
| 1 | 'b' | 'a' | 'b' not in d, add mapping | {'a': 'a', 'b': 'a'} |

Although we built the mappings without conflict, this would require both 'a' and 'b' to map to 'a'. While our hash table check passes (no one-to-many mapping), this represents a many-to-one mapping which is actually valid for this problem. The transformation would succeed, returning `True`.

Actually, let's consider `str1 = "aa"` and `str2 = "ab"` instead:

| Position | str1[i] | str2[i] | Action | Hash table `d` |
|----------|---------|---------|--------|----------------|
| 0 | 'a' | 'a' | 'a' not in d, add mapping | {'a': 'a'} |
| 1 | 'a' | 'b' | 'a' in d, check: d['a']='a' ≠ 'b' ✗ | Conflict! |

**Step 4:** Return `False` due to conflict - 'a' cannot map to both 'a' and 'b'.

## Python Solution

```python
class Solution:
    def canConvert(self, str1: str, str2: str) -> bool:
        # If both strings are identical, no conversion needed
        if str1 == str2:
            return True

        # If str2 uses all 26 letters, we cannot convert
        # (we need at least one unused character as intermediate for conversion chains)
        if len(set(str2)) == 26:
            return False

        # Dictionary to store character mappings from str1 to str2
        char_mapping = {}

        # Check if a consistent one-to-one mapping exists
        for char1, char2 in zip(str1, str2):
            if char1 not in char_mapping:
                # Create new mapping for this character
                char_mapping[char1] = char2
            elif char_mapping[char1] != char2:
                # Conflict: char1 already maps to a different character
                return False

        # All characters have consistent mappings
        return True
```

## Time and Space Complexity

The time complexity is `O(n)`, where `n` is the length of the strings `str1` and `str2`. This is because:
- The initial string comparison `str1 == str2` takes `O(n)` time
- Converting `str2` to a set with `set(str2)` requires `O(n)` time to iterate through all characters
- The main loop with `zip(str1, str2)` iterates through both strings once, performing `O(1)` dictionary operations for each character pair, resulting in `O(n)` total time

The space complexity is `O(C)`, where `C` is the size of the character set (in this problem, `C = 26` for lowercase English letters). This is because:
- The dictionary `d` can store at most `C` unique character mappings (one for each distinct character in `str1`)
- The set created from `str2` can contain at most `C` unique characters, requiring `O(C)` space
- Since `C` is constant (26), the space complexity can also be considered `O(1)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### Pitfall: Misunderstanding the "all 26 letters" check

A common mistake is thinking that if `str2` has all 26 letters, we can still perform the transformation by carefully ordering our conversions. However, this overlooks the fundamental issue of **conversion cycles**.

**Why this matters:**
When `str2` uses all 26 letters, every letter is "occupied" as a target. If we have a cycle in our mapping (like `a→b`, `b→c`, `c→a`), we need a temporary "parking spot" - an unused letter to break the cycle. Without any unused letters, breaking cycles becomes impossible.

**Example of the pitfall:**
```python
# Incorrect approach - missing the 26-letter check
def canConvert(str1, str2):
    if str1 == str2:
        return True

    char_mapping = {}
    for char1, char2 in zip(str1, str2):
        if char1 not in char_mapping:
            char_mapping[char1] = char2
        elif char_mapping[char1] != char2:
            return False

    return True  # Wrong! Doesn't check if str2 has all 26 letters
```

**Test case that breaks the incorrect solution:**
- `str1 = "abcdefghijklmnopqrstuvwxyz"`
- `str2 = "bcdefghijklmnopqrstuvwxyza"`

This creates a cycle where each letter maps to the next one, with 'z' mapping back to 'a'. The incorrect solution would return `True`, but the transformation is actually impossible since we need an extra letter to break the cycle.

**Solution:**
Always include the check for whether `str2` contains all 26 unique characters:
```python
if len(set(str2)) == 26:
    return False
```

This ensures we have at least one "spare" letter available for breaking conversion cycles when needed.

