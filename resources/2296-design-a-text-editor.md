# 2296. Design a Text Editor

> Source: https://algo.monster/liteproblems/2296

## Problem Statement

You need to design a text editor that simulates a cursor's behavior with the following operations:

**Core Functionality:**
- **Add text** at the cursor position
- **Delete text** to the left of the cursor (like pressing backspace)
- **Move cursor** left or right within the text

**Key Constraints:**
- The cursor position is always valid: `0 <= cursor.position <= currentText.length`
- Deletion only affects characters to the left of the cursor
- The cursor cannot move beyond the text boundaries

**Class Methods to Implement:**

1. `TextEditor()`: Initialize an empty text editor

2. `addText(string text)`: Insert the given `text` at the current cursor position. After insertion, the cursor moves to the right of the newly added text.

3. `deleteText(int k)`: Delete up to `k` characters to the left of the cursor. Returns the actual number of characters deleted (which may be less than `k` if there aren't enough characters).

4. `cursorLeft(int k)`: Move the cursor left by up to `k` positions. Returns the last `min(10, len)` characters to the left of the cursor after moving, where `len` is the number of characters to the left of the cursor.

5. `cursorRight(int k)`: Move the cursor right by up to `k` positions. Returns the last `min(10, len)` characters to the left of the cursor after moving, where `len` is the number of characters to the left of the cursor.

**Solution Strategy:**

The solution uses two stacks to efficiently manage text on both sides of the cursor:
- `left` [stack](/problems/stack_intro): stores characters to the left of the cursor
- `right` stack: stores characters to the right of the cursor

This design makes cursor movements and text operations efficient by simply transferring elements between stacks. When moving left, characters pop from the `left` [stack](/problems/stack_intro) and push to the `right` stack. When moving right, the opposite occurs. Text addition pushes to the `left` stack, and deletion pops from the `left` stack.

## Intuition

Think about how a cursor works in a real text editor - it divides the text into two parts: everything before the cursor and everything after the cursor. When you type, new characters appear right where the cursor is, pushing everything after the cursor to the right. When you press backspace, you delete what's immediately before the cursor.

This natural division suggests we can treat the text as two separate segments. But how should we store these segments? If we use simple strings or arrays, moving the cursor would require shifting many elements. For example, moving the cursor left means taking characters from the "before" segment and moving them to the "after" segment.

The key insight is recognizing that cursor movements follow a Last-In-First-Out pattern. When you move the cursor left, the last character before the cursor becomes the first character after the cursor. When you move right, the first character after the cursor becomes the last character before the cursor. This LIFO behavior is exactly what stacks provide!

By using two stacks:
- The `left` [stack](/problems/stack_intro) naturally maintains characters before the cursor, with the top being the character immediately to the left
- The `right` stack maintains characters after the cursor, with the top being the character immediately to the right

This makes all operations elegant:
- **Adding text**: Just push to the `left` [stack](/problems/stack_intro) - `O(n)` where n is the text length
- **Deleting**: Simply pop from the `left` stack - `O(k)` for k deletions  
- **Moving cursor left**: Pop from `left` and push to `right` - `O(k)` for k moves
- **Moving cursor right**: Pop from `right` and push to `left` - `O(k)` for k moves

The beauty of this approach is that the cursor position is implicitly defined by the boundary between the two stacks, eliminating the need to track it explicitly or shift elements when it moves.

**Learn more about [Stack](/problems/stack_intro) and [Linked List](/problems/linked_list_cycle) patterns.**

## Solution Approach

The implementation uses two stacks to represent the text editor state:
- `self.left`: [Stack](/problems/stack_intro) containing characters to the left of the cursor
- `self.right`: Stack containing characters to the right of the cursor

Let's walk through each method implementation:

### 1. `__init__()`
```python
def __init__(self):
    self.left = []
    self.right = []
```
Initialize two empty lists that will serve as our stacks. The cursor starts at position 0, meaning all text (initially none) is to the right of the cursor.

### 2. `addText(text)`
```python
def addText(self, text: str) -> None:
    self.left.extend(list(text))
```
Add text at the cursor position by pushing all characters onto the `left` [stack](/problems/stack_intro). Using `extend()` with `list(text)` efficiently adds all characters at once. After this operation, the cursor is positioned after the newly added text.

**Time Complexity**: `O(|text|)` where `|text|` is the length of the input string.

### 3. `deleteText(k)`
```python
def deleteText(self, k: int) -> int:
    k = min(k, len(self.left))
    for _ in range(k):
        self.left.pop()
    return k
```
Delete up to `k` characters to the left of the cursor. First, we calculate the actual number of characters to delete using `min(k, len(self.left))` to handle cases where `k` exceeds available characters. Then pop from the `left` [stack](/problems/stack_intro) `k` times and return the actual deletion count.

**Time Complexity**: `O(k)` for `k` deletions.

### 4. `cursorLeft(k)`
```python
def cursorLeft(self, k: int) -> str:
    k = min(k, len(self.left))
    for _ in range(k):
        self.right.append(self.left.pop())
    return ''.join(self.left[-10:])
```
Move the cursor left by transferring characters from the `left` [stack](/problems/stack_intro) to the `right` stack. Each character popped from `left` becomes the new top of `right`, maintaining the correct order. After moving, return the last 10 characters (or fewer if less than 10 exist) to the left of the cursor using slice notation `self.left[-10:]`.

**Time Complexity**: `O(k)` for moving the cursor plus `O(min(10, len(self.left)))` for building the return string.

### 5. `cursorRight(k)`
```python
def cursorRight(self, k: int) -> str:
    k = min(k, len(self.right))
    for _ in range(k):
        self.left.append(self.right.pop())
    return ''.join(self.left[-10:])
```
Move the cursor right by transferring characters from the `right` [stack](/problems/stack_intro) to the `left` stack. This is the reverse of `cursorLeft`. Characters move from `right` to `left`, effectively moving the cursor rightward. Return the last 10 characters to the left of the new cursor position.

**Time Complexity**: `O(k)` for moving the cursor plus `O(min(10, len(self.left)))` for building the return string.

### Example Walkthrough
Consider the sequence: `addText("hello")`, `cursorLeft(3)`, `deleteText(2)`:

1. After `addText("hello")`: `left = ['h','e','l','l','o']`, `right = []`
2. After `cursorLeft(3)`: `left = ['h','e']`, `right = ['o','l','l']` (note the reverse order in right)
3. After `deleteText(2)`: `left = []`, `right = ['o','l','l']`

The final text would be "llo" with the cursor at position 0.

## Example Walkthrough

Let's trace through a sequence of operations to see how the two-stack approach works:

**Initial state**: Empty editor
- `left = []`, `right = []`
- Text: "" (cursor at position 0)

**Step 1**: `addText("abc")`
- Push 'a', 'b', 'c' to left stack
- `left = ['a','b','c']`, `right = []`
- Text: "abc|" (cursor at position 3)

**Step 2**: `cursorLeft(2)`
- Pop 'c' from left, push to right
- Pop 'b' from left, push to right
- `left = ['a']`, `right = ['c','b']`
- Text: "a|bc" (cursor at position 1)
- Returns: "a" (last 10 chars to the left)

**Step 3**: `addText("xy")`
- Push 'x', 'y' to left stack
- `left = ['a','x','y']`, `right = ['c','b']`
- Text: "axy|bc" (cursor at position 3)

**Step 4**: `deleteText(1)`
- Pop 'y' from left
- `left = ['a','x']`, `right = ['c','b']`
- Text: "ax|bc" (cursor at position 2)
- Returns: 1 (deleted 1 character)

**Step 5**: `cursorRight(1)`
- Pop 'c' from right, push to left
- `left = ['a','x','c']`, `right = ['b']`
- Text: "axc|b" (cursor at position 3)
- Returns: "axc" (last 10 chars to the left)

Notice how the cursor position is always the boundary between the two stacks. Characters to the left are in order in the left stack, while characters to the right are in reverse order in the right stack (top element is closest to cursor).

## Python Solution

```python
class TextEditor:
    def __init__(self) -> None:
        """
        Initialize the text editor with two stacks to represent text on both sides of cursor.
        The cursor position is between the left and right stacks.
        """
        self.left = []   # Stack for characters to the left of cursor
        self.right = []  # Stack for characters to the right of cursor

    def addText(self, text: str) -> None:
        """
        Add text at the current cursor position.
      
        Args:
            text: String to be added at cursor position
        """
        # Convert string to list of characters and add to left of cursor
        self.left.extend(list(text))

    def deleteText(self, k: int) -> int:
        """
        Delete at most k characters to the left of cursor.
      
        Args:
            k: Maximum number of characters to delete
          
        Returns:
            The actual number of characters deleted
        """
        # Delete min(k, available characters) from left of cursor
        k = min(k, len(self.left))
        for _ in range(k):
            self.left.pop()
        return k

    def cursorLeft(self, k: int) -> str:
        """
        Move cursor k positions to the left.
      
        Args:
            k: Number of positions to move left
          
        Returns:
            String of up to 10 characters to the left of cursor after movement
        """
        # Move min(k, available characters) from left stack to right stack
        k = min(k, len(self.left))
        for _ in range(k):
            self.right.append(self.left.pop())
      
        # Return last 10 characters to the left of cursor
        return ''.join(self.left[-10:])

    def cursorRight(self, k: int) -> str:
        """
        Move cursor k positions to the right.
      
        Args:
            k: Number of positions to move right
          
        Returns:
            String of up to 10 characters to the left of cursor after movement
        """
        # Move min(k, available characters) from right stack to left stack
        k = min(k, len(self.right))
        for _ in range(k):
            self.left.append(self.right.pop())
      
        # Return last 10 characters to the left of cursor
        return ''.join(self.left[-10:])


# Your TextEditor object will be instantiated and called as such:
# obj = TextEditor()
# obj.addText(text)
# param_2 = obj.deleteText(k)
# param_3 = obj.cursorLeft(k)
# param_4 = obj.cursorRight(k)
```

## Time and Space Complexity

**Time Complexity:**
- `__init__()`: `O(1)` - Simply initializes two empty lists.
- `addText(text)`: `O(n)` where `n` is the length of the input text. The `extend()` operation with `list(text)` iterates through all characters.
- `deleteText(k)`: `O(k)` - Pops at most `k` characters from the `left` stack. Each pop operation is `O(1)`, so total is `O(k)`.
- `cursorLeft(k)`: `O(k)` - Transfers at most `k` characters from `left` to `right` stack. Each pop and append operation is `O(1)`, plus the `join` operation on the last 10 characters is `O(1)` since it's bounded by a constant.
- `cursorRight(k)`: `O(k)` - As confirmed by the reference answer, we pop characters from the `right` stack up to `k` times and push them onto the `left` stack. Each pop and append operation is `O(1)`, and returning the last 10 characters with `join` is `O(1)`.

**Space Complexity:**
- Overall space complexity: `O(m)` where `m` is the total number of characters stored in the text editor.
- The `left` and `right` stacks together store all the characters in the editor.
- Each method uses `O(1)` additional space except `addText` which temporarily creates a list from the input string, using `O(n)` extra space where `n` is the length of the input text.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Order When Moving Cursor Right**
A common mistake is forgetting that the `right` stack stores characters in reverse order. When moving the cursor right, characters must be popped from `right` and pushed to `left`, not the other way around.

**Incorrect Implementation:**
```python
def cursorRight(self, k: int) -> str:
    k = min(k, len(self.right))
    for _ in range(k):
        # Wrong: This would reverse the text order
        self.left.extend(self.right[:k])  
        del self.right[:k]
    return ''.join(self.left[-10:])
```

**Correct Implementation:**
```python
def cursorRight(self, k: int) -> str:
    k = min(k, len(self.right))
    for _ in range(k):
        # Correct: Pop from right and append to left one by one
        self.left.append(self.right.pop())
    return ''.join(self.left[-10:])
```

### 2. **Inefficient Character-by-Character Addition**
When implementing `addText`, developers might add characters one by one, which is less efficient than bulk operations.

**Inefficient Implementation:**
```python
def addText(self, text: str) -> None:
    for char in text:
        self.left.append(char)  # O(n) iterations with function call overhead
```

**Efficient Implementation:**
```python
def addText(self, text: str) -> None:
    self.left.extend(list(text))  # Single operation, more efficient
```

### 3. **Forgetting to Cap Movement/Deletion Operations**
A critical mistake is not limiting operations to available characters, which can cause index errors or incorrect behavior.

**Problematic Implementation:**
```python
def deleteText(self, k: int) -> int:
    # This will crash if k > len(self.left)
    for _ in range(k):
        self.left.pop()
    return k
```

**Correct Implementation:**
```python
def deleteText(self, k: int) -> int:
    k = min(k, len(self.left))  # Cap k to available characters
    for _ in range(k):
        self.left.pop()
    return k
```

### 4. **Returning Wrong Slice for Display**
The methods `cursorLeft` and `cursorRight` should return the last 10 characters to the left of the cursor, not the first 10.

**Wrong Implementation:**
```python
def cursorLeft(self, k: int) -> str:
    k = min(k, len(self.left))
    for _ in range(k):
        self.right.append(self.left.pop())
    return ''.join(self.left[:10])  # Wrong: Returns first 10, not last 10
```

**Correct Implementation:**
```python
def cursorLeft(self, k: int) -> str:
    k = min(k, len(self.left))
    for _ in range(k):
        self.right.append(self.left.pop())
    return ''.join(self.left[-10:])  # Correct: Returns last 10 characters
```

### 5. **Memory Inefficiency with String Concatenation**
Building the return string inefficiently can impact performance, especially if done repeatedly.

**Less Efficient:**
```python
def cursorLeft(self, k: int) -> str:
    # ... cursor movement code ...
    result = ""
    for char in self.left[-10:]:
        result += char  # String concatenation creates new objects
    return result
```

**More Efficient:**
```python
def cursorLeft(self, k: int) -> str:
    # ... cursor movement code ...
    return ''.join(self.left[-10:])  # Single join operation
```

