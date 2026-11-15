# 158. Read N Characters Given read4 II - Call Multiple Times

> Source: https://algo.monster/liteproblems/158

## Problem Statement

This problem asks you to implement a `read` method that can read `n` characters from a file, but you can only access the file through a provided `read4` method. The key challenge is that your `read` method may be called multiple times.

**Understanding read4:**
- `read4(buf4)` reads up to 4 consecutive characters from the file
- It stores these characters in the `buf4` array
- It returns the actual number of characters read (0-4)
- When the file ends, it returns fewer than 4 characters or 0

**Your task - implement read(buf, n):**
- Read exactly `n` characters from the file (or fewer if the file ends)
- Store the characters in the provided `buf` array
- Return the actual number of characters read

**The main challenge:** Since `read` can be called multiple times, you need to handle the case where:
1. A previous call to `read4` may have fetched more characters than needed
2. Those extra characters should be available for the next `read` call
3. You need to maintain state between multiple calls to `read`

For example:
- File contains: "abcdefg"
- First call: `read(buf, 5)` → reads "abcde", returns 5
- Second call: `read(buf, 2)` → reads "fg", returns 2

The solution uses instance variables to maintain state:
- `buf4`: A buffer to store characters from `read4`
- `i`: Current position in `buf4`
- `size`: Number of valid characters in `buf4`

## Intuition

The key insight is that `read4` always gives us chunks of 4 characters, but we might need any number of characters. This creates a mismatch - what if we only need 2 characters but `read4` gives us 4? Those extra 2 characters shouldn't be lost because the next call to `read` might need them.

Think of it like buying eggs that only come in cartons of 4, but your recipes might need any number of eggs. If a recipe needs 2 eggs, you still have to buy a carton of 4. You'd naturally save the remaining 2 eggs for your next recipe rather than throwing them away.

This leads us to realize we need a "buffer" - a temporary storage space between `read4` and our `read` method. When `read4` gives us more characters than currently needed, we store them. When `read` is called again, we first check our buffer before calling `read4` again.

The state we need to track:
- **What we have stored**: We need our own buffer (`buf4`) to hold characters from `read4`
- **Where we are in that buffer**: A pointer (`i`) to track which character to read next from our buffer
- **How much valid data is in the buffer**: A counter (`size`) to know how many characters in our buffer are actually valid

The flow becomes:
1. When `read` is called, first check if we have leftover characters from a previous `read4` call (when `i < size`)
2. Use those leftover characters first
3. Only when we've used all buffered characters (`i == size`), call `read4` for more
4. Reset our position (`i = 0`) when we get fresh data
5. Keep copying characters until we've either read `n` characters or the file ends

This approach elegantly handles the mismatch between the fixed 4-character chunks from `read4` and the variable number of characters requested by `read`.

## Solution Approach

The solution uses a class-based approach with instance variables to maintain state between multiple calls to `read`.

**Data Structures:**
- `self.buf4`: A list of size 4 to store characters retrieved from `read4`
- `self.i`: An integer tracking the current position in `buf4` (which character to read next)
- `self.size`: An integer storing the number of valid characters currently in `buf4`

**Implementation walkthrough:**

1. **Initialization (`__init__`):**
   ```python
   self.buf4 = [None] * 4
   self.i = self.size = 0
   ```
   - Create a buffer of size 4 to match `read4`'s output
   - Initialize both position pointer and size to 0 (empty buffer state)

2. **Main read logic:**
   ```python
   j = 0  # tracks position in output buffer
   while j < n:
   ```
   - Use `j` to track how many characters we've written to the output buffer
   - Continue until we've read `n` characters or the file ends

3. **Check if we need new data from read4:**
   ```python
   if self.i == self.size:
       self.size = read4(self.buf4)
       self.i = 0
       if self.size == 0:
           break
   ```
   - When `self.i == self.size`, we've consumed all buffered characters
   - Call `read4` to get up to 4 new characters
   - Reset position to 0 for the fresh buffer
   - If `read4` returns 0, the file has ended, so break

4. **Copy characters from buffer to output:**
   ```python
   while j < n and self.i < self.size:
       buf[j] = self.buf4[self.i]
       self.i += 1
       j += 1
   ```
   - Copy characters one by one from `buf4` to output `buf`
   - Increment both pointers: `self.i` (position in `buf4`) and `j` (position in output)
   - Stop when either we've read `n` characters or exhausted the current buffer

5. **Return the count:**
   ```python
   return j
   ```
   - Return the actual number of characters read

**Why this works:**
- The outer loop ensures we keep trying to read until we get `n` characters or hit EOF
- The state variables (`self.i` and `self.size`) persist between calls, preserving unused characters
- The inner loop efficiently copies available characters without calling `read4` unnecessarily
- The algorithm correctly handles edge cases like EOF and partial reads

**Example execution:**
- File: "abcdefg", first `read(buf, 3)`:
  - `read4` returns 4, `buf4 = ['a','b','c','d']`
  - Copy 3 characters to output: ['a','b','c']
  - State after: `self.i = 3`, `self.size = 4`
- Second `read(buf, 2)`:
  - `self.i < self.size`, so use buffered 'd'
  - Need more, call `read4`, get ['e','f','g'], size = 3
  - Copy 'e' to complete the request
  - State after: `self.i = 1`, `self.size = 3`

## Example Walkthrough

Let's walk through a concrete example to understand how the solution handles multiple `read` calls.

**Setup:** File contains "abcdefg" (7 characters)

**First call: `read(buf, 5)`**

Initial state: `self.i = 0`, `self.size = 0`, `self.buf4 = [None, None, None, None]`

1. Start with `j = 0` (output position)
2. Check: `self.i == self.size` (0 == 0)? Yes, buffer is empty
3. Call `read4(self.buf4)`:
   - Reads "abcd" into `self.buf4 = ['a','b','c','d']`
   - Returns 4, so `self.size = 4`
   - Reset `self.i = 0`
4. Copy loop: while `j < 5` and `self.i < 4`:
   - Copy 'a': `buf[0] = 'a'`, `self.i = 1`, `j = 1`
   - Copy 'b': `buf[1] = 'b'`, `self.i = 2`, `j = 2`
   - Copy 'c': `buf[2] = 'c'`, `self.i = 3`, `j = 3`
   - Copy 'd': `buf[3] = 'd'`, `self.i = 4`, `j = 4`
   - Stop: `self.i` reached `self.size`
5. Still need 1 more character (`j < 5`), so continue outer loop
6. Check: `self.i == self.size` (4 == 4)? Yes, buffer exhausted
7. Call `read4(self.buf4)`:
   - Reads "efg" into `self.buf4 = ['e','f','g',?]`
   - Returns 3, so `self.size = 3`
   - Reset `self.i = 0`
8. Copy 'e': `buf[4] = 'e'`, `self.i = 1`, `j = 5`
9. Stop: `j` reached `n` (5)
10. Return 5

**State after first call:** `self.i = 1`, `self.size = 3`, `self.buf4 = ['e','f','g',?]`
*Note: We have 'f' and 'g' still buffered!*

**Second call: `read(buf, 3)`**

Starting state: `self.i = 1`, `self.size = 3` (leftover data from previous call)

1. Start with `j = 0`
2. Check: `self.i == self.size` (1 == 3)? No, we have buffered data!
3. Copy loop: while `j < 3` and `self.i < 3`:
   - Copy 'f': `buf[0] = 'f'`, `self.i = 2`, `j = 1`
   - Copy 'g': `buf[1] = 'g'`, `self.i = 3`, `j = 2`
   - Stop: `self.i` reached `self.size`
4. Still need 1 more character (`j < 3`), continue outer loop
5. Check: `self.i == self.size` (3 == 3)? Yes, buffer exhausted
6. Call `read4(self.buf4)`:
   - No more data in file
   - Returns 0, so `self.size = 0`
   - Break from loop (EOF reached)
7. Return 2

**Result:** Second call returns 2 with `buf = ['f','g']`, correctly using the leftover characters from the first `read4` call and detecting EOF.

This example demonstrates the key aspects:
- Buffering excess characters between calls
- Maintaining state with `self.i` and `self.size`
- Efficiently reusing buffered data before calling `read4` again
- Properly handling EOF when fewer characters remain than requested

## Python Solution

```python
# The read4 API is already defined for you.
# def read4(buf4: List[str]) -> int:

from typing import List


class Solution:
    def __init__(self):
        # Internal buffer to store characters read from read4
        self.internal_buffer = [''] * 4
        # Current index in the internal buffer
        self.buffer_index = 0
        # Number of valid characters in the internal buffer
        self.buffer_size = 0

    def read(self, buf: List[str], n: int) -> int:
        """
        Read n characters from file using read4 API.

        Args:
            buf: Output buffer to store the read characters
            n: Number of characters to read

        Returns:
            The actual number of characters read
        """
        chars_read = 0

        while chars_read < n:
            # If we've consumed all characters in the internal buffer,
            # read more from the file using read4
            if self.buffer_index == self.buffer_size:
                self.buffer_size = read4(self.internal_buffer)
                self.buffer_index = 0

                # If read4 returns 0, we've reached end of file
                if self.buffer_size == 0:
                    break

            # Copy characters from internal buffer to output buffer
            # until we've read n characters or exhausted the internal buffer
            while chars_read < n and self.buffer_index < self.buffer_size:
                buf[chars_read] = self.internal_buffer[self.buffer_index]
                self.buffer_index += 1
                chars_read += 1

        return chars_read
```

## Time and Space Complexity

**Time Complexity:** `O(n)`

The algorithm reads at most `n` characters from the file. The main loop iterates through each character that needs to be read:
- The outer `while` loop continues until `j` reaches `n` or there are no more characters to read
- For each iteration, we either call `read4()` (which is `O(1)`) or copy characters from the internal buffer to the output buffer
- Each character is processed exactly once, either being read from the file via `read4()` or copied from the internal buffer
- The total number of `read4()` calls is at most `⌈n/4⌉`, and each call is `O(1)`
- The total number of character copies is exactly `min(n, available_characters)`

Therefore, the overall time complexity is `O(n)`.

**Space Complexity:** `O(1)`

The algorithm uses a fixed amount of extra space:
- `self.buf4`: A buffer array of size 4 to store characters read from `read4()`
- `self.i`: An integer to track the current position in the internal buffer
- `self.size`: An integer to store the number of characters returned by the last `read4()` call
- `j`: An integer counter for the output buffer position

All these variables use constant space regardless of the input size `n`, resulting in `O(1)` space complexity.

## Common Pitfalls

### 1. **Not Maintaining State Between Calls**
The most critical pitfall is treating each `read` call as independent. If you create local variables for the buffer instead of instance variables, you'll lose leftover characters between calls.

**Incorrect approach:**
```python
def read(self, buf: List[str], n: int) -> int:
    buf4 = [''] * 4  # Local variable - WRONG!
    # This creates a new buffer each time, losing previous data
```

**Solution:** Always use instance variables (`self.buf4`, `self.i`, `self.size`) to preserve state across multiple calls.

### 2. **Incorrectly Resetting Buffer Position**
A common mistake is resetting the buffer index at the wrong time or forgetting to reset it after calling `read4`.

**Incorrect approach:**
```python
if self.i == self.size:
    self.size = read4(self.buf4)
    # Forgot to reset self.i = 0 - WRONG!
    # Will continue reading from wrong position
```

**Solution:** Always reset `self.i = 0` immediately after calling `read4` to start reading from the beginning of the newly filled buffer.

### 3. **Overwriting Characters in Output Buffer**
Starting from the wrong index in the output buffer `buf` is a frequent error, especially when not properly tracking the write position.

**Incorrect approach:**
```python
def read(self, buf: List[str], n: int) -> int:
    while j < n:
        buf[self.i] = self.buf4[self.i]  # Using wrong index - WRONG!
        # Should use j for buf, not self.i
```

**Solution:** Use a separate counter (`j` or `chars_read`) specifically for tracking the position in the output buffer.

### 4. **Not Handling Partial Reads Correctly**
When `read4` returns fewer than 4 characters (at end of file), failing to handle this case properly can cause index out of bounds errors or infinite loops.

**Incorrect approach:**
```python
while j < n:
    self.size = read4(self.buf4)
    # Assuming size is always 4 - WRONG!
    for k in range(4):  # This will fail when size < 4
        buf[j] = self.buf4[k]
```

**Solution:** Always check the actual return value of `read4` and only process that many characters:
```python
if self.size == 0:  # Handle EOF
    break
while j < n and self.i < self.size:  # Respect actual size
    buf[j] = self.buf4[self.i]
```

### 5. **Buffer Overflow When Copying Characters**
Not properly bounding the copy operation can lead to writing beyond the requested `n` characters.

**Incorrect approach:**
```python
while self.i < self.size:  # Only checking buffer bounds - WRONG!
    buf[j] = self.buf4[self.i]
    j += 1  # Could exceed n
```

**Solution:** Always include both conditions in the copy loop:
```python
while j < n and self.i < self.size:  # Check both bounds
    buf[j] = self.buf4[self.i]
    self.i += 1
    j += 1
```

