# 2759. Convert JSON String to Object

> Source: https://algo.monster/liteproblems/2759

## Problem Statement

You need to implement a JSON parser that converts a valid JSON string into its corresponding JavaScript object representation. The function should parse the string character by character without using the built-in `JSON.parse` method.

The input string `str` is guaranteed to be valid JSON and will only contain:
- **Strings**: Text enclosed in double quotes (e.g., `"hello"`)
- **Numbers**: Integer or decimal values (e.g., `42`, `3.14`, `-10`)
- **Arrays**: Ordered lists enclosed in square brackets (e.g., `[1, 2, 3]`)
- **Objects**: Key-value pairs enclosed in curly braces (e.g., `{"name": "John", "age": 30}`)
- **Booleans**: `true` or `false`
- **Null**: `null`

The parser should handle nested structures, meaning arrays and objects can contain other arrays, objects, or any of the primitive types mentioned above. The string won't contain invisible characters or escape sequences that need special handling beyond basic backslash escaping.

For example:
- Input: `"{"a": 1, "b": [2, 3], "c": {"d": true}}"`
- Output: `{a: 1, b: [2, 3], c: {d: true}}`

## Intuition

When we look at JSON structure, we notice that each value type starts with a distinctive character. A string always starts with `"`, an object with `{`, an array with `[`, true with `t`, false with `f`, null with `n`, and numbers with a digit or minus sign. This observation gives us a natural way to determine what we're parsing just by looking at the current character.

The key insight is that JSON has a recursive structure - objects can contain other objects, arrays can contain arrays, and both can contain any valid JSON value. This naturally leads us to think about a recursive parsing approach where we have a main function that identifies the value type and delegates to specialized parsing functions.

We can think of parsing as "consuming" characters from left to right. As we read each character, we decide what to do based on what we're currently parsing. For instance, when parsing an object and we encounter a `}`, we know the object is complete. When parsing a string and we see a closing `"`, the string ends.

The challenge with nested structures becomes manageable when we realize that each parsing function can simply call back to the main value parser when it needs to parse a nested element. For example, when `parseArray` encounters a non-delimiter character (not `]` or `,`), it knows there's a value to parse, so it calls `parseValue`, which will figure out what type of value it is and handle it appropriately.

By maintaining a single index `i` that tracks our position in the string and advancing it as we consume characters, we ensure that each character is processed exactly once and in the correct order. Each specialized parsing function knows exactly how many characters to consume (like `true` always being 4 characters) or when to stop (like reading until the closing quote for strings).

This approach mirrors how we mentally parse JSON - we recognize patterns, handle them according to their type, and recursively process nested structures when we encounter them.

## Solution Approach

The implementation uses a **recursive descent parser** with a shared index pointer to traverse the string. Here's how each component works:

**Main Structure:**
- We maintain a global index `i` that tracks our current position in the string
- The main entry point is `parseValue()` which examines the current character and delegates to the appropriate parser

**Character-based Dispatching:**
The `parseValue()` function acts as a dispatcher by checking the first character:
- `{` → call `parseObject()`
- `[` → call `parseArray()`
- `"` → call `parseString()`
- `t` → call `parseTrue()`
- `f` → call `parseFalse()`
- `n` → call `parseNull()`
- Any other character (digit or `-`) → call `parseNumber()`

**Parsing Primitives:**

1. **Boolean and Null Parsing:** These are the simplest - we know exactly how many characters to skip:
   - `parseTrue()`: advances index by 4 characters (for "true")
   - `parseFalse()`: advances index by 5 characters (for "false")
   - `parseNull()`: advances index by 4 characters (for "null")

2. **Number Parsing:**
   - Accumulates characters into a string until hitting a delimiter (`,`, `}`, or `]`)
   - Converts the accumulated string to a number using `Number()`

3. **String Parsing:**
   - Skips the opening quote and reads until the closing quote
   - Handles escaped characters by checking for backslash `\` and including the next character as-is

**Parsing Complex Structures:**

1. **Array Parsing (`parseArray()`):**
   - Skips the opening `[`
   - Loops until finding `]`:
     - Skips commas between elements
     - Calls `parseValue()` recursively for each element
     - Pushes each parsed value to the result array
   - Skips the closing `]`

2. **Object Parsing (`parseObject()`):**
   - Skips the opening `{`
   - Loops until finding `}`:
     - Skips commas between key-value pairs
     - Calls `parseString()` to get the key (keys are always strings in JSON)
     - Skips the colon `:` separator
     - Calls `parseValue()` recursively to get the value
     - Assigns the key-value pair to the result object
   - Skips the closing `}`

**Key Design Patterns:**

1. **Mutual Recursion:** `parseValue()` calls specific parsers, which in turn call `parseValue()` for nested structures
2. **Lookahead Pattern:** Each parser examines the current character to decide what to do next
3. **State Management:** The shared index `i` maintains parsing state across all functions
4. **Incremental Consumption:** Each parser advances the index only for characters it processes

The algorithm's time complexity is `O(n)` where `n` is the length of the input string, as each character is visited exactly once. The space complexity is `O(d)` where `d` is the maximum depth of nesting in the JSON structure, due to the recursive call stack.

## Example Walkthrough

Let's walk through parsing the JSON string: `{"a":1,"b":[2,true]}`

**Initial State:** `i = 0`, string = `{"a":1,"b":[2,true]}`

**Step 1:** Call `parseValue()`
- Current character at index 0 is `{`
- Dispatch to `parseObject()`

**Step 2:** Inside `parseObject()`
- Skip opening `{`, now `i = 1`
- Create empty object `result = {}`
- Current character is `"`, not `}`, so continue

**Step 3:** Parse first key-value pair
- Call `parseString()` for the key
  - Skip opening `"`, now `i = 2`
  - Read character `a`
  - Hit closing `"` at index 3
  - Skip closing quote, now `i = 4`
  - Return key = `"a"`
- Skip colon `:`, now `i = 5`
- Call `parseValue()` for the value
  - Current character is `1`
  - Dispatch to `parseNumber()`
  - Accumulate `1` until hitting `,` at index 6
  - Return value = `1`
- Set `result["a"] = 1`
- Current character is `,`, skip it, now `i = 7`

**Step 4:** Parse second key-value pair
- Call `parseString()` for the key
  - Skip opening `"`, now `i = 8`
  - Read character `b`
  - Hit closing `"` at index 9
  - Skip closing quote, now `i = 10`
  - Return key = `"b"`
- Skip colon `:`, now `i = 11`
- Call `parseValue()` for the value
  - Current character is `[`
  - Dispatch to `parseArray()`

**Step 5:** Inside `parseArray()`
- Skip opening `[`, now `i = 12`
- Create empty array `arr = []`
- Current character is `2`, not `]`, so continue
- Call `parseValue()`
  - Current character is `2`
  - Dispatch to `parseNumber()`
  - Accumulate `2` until hitting `,` at index 13
  - Return `2`
- Push `2` to array: `arr = [2]`
- Skip comma `,`, now `i = 14`
- Call `parseValue()`
  - Current character is `t`
  - Dispatch to `parseTrue()`
  - Skip 4 characters ("true"), now `i = 18`
  - Return `true`
- Push `true` to array: `arr = [2, true]`
- Current character is `]`
- Skip closing `]`, now `i = 19`
- Return `arr = [2, true]`

**Step 6:** Back in `parseObject()`
- Set `result["b"] = [2, true]`
- Current character is `}`
- Skip closing `}`, now `i = 20`
- Return `result = {a: 1, b: [2, true]}`

**Final Result:** `{a: 1, b: [2, true]}`

The parser successfully navigated through the nested structure by:
1. Recognizing each value type by its first character
2. Recursively calling `parseValue()` for nested elements
3. Maintaining a single index that advances through the string
4. Each specialized parser knowing exactly when to stop (delimiters or fixed lengths)

## Python Solution

```python
def jsonParse(str_input: str):
    """
    Custom JSON parser implementation
    Parses a JSON string and returns the corresponding Python value

    Args:
        str_input: The JSON string to parse

    Returns:
        The parsed Python value
    """
    length = len(str_input)
    current_index = [0]  # Using list to maintain reference in nested functions

    def parse_true():
        """
        Parses the boolean value 'true'
        Advances the index by 4 characters

        Returns:
            The boolean value True
        """
        current_index[0] += 4  # Skip 'true'
        return True

    def parse_false():
        """
        Parses the boolean value 'false'
        Advances the index by 5 characters

        Returns:
            The boolean value False
        """
        current_index[0] += 5  # Skip 'false'
        return False

    def parse_null():
        """
        Parses the null value
        Advances the index by 4 characters

        Returns:
            None
        """
        current_index[0] += 4  # Skip 'null'
        return None

    def parse_number():
        """
        Parses a numeric value from the string
        Continues reading until a delimiter (comma, closing brace, or closing bracket) is found

        Returns:
            The parsed number
        """
        number_string = ''

        # Read characters until we hit a delimiter
        while current_index[0] < length:
            current_char = str_input[current_index[0]]

            # Check for delimiters that indicate end of number
            if current_char in [',', '}', ']']:
                break

            number_string += current_char
            current_index[0] += 1

        # Convert string to number (handles both int and float)
        return float(number_string) if '.' in number_string else int(number_string)

    def parse_array():
        """
        Parses an array from the string
        Handles nested values recursively

        Returns:
            The parsed array (list)
        """
        result_array = []
        current_index[0] += 1  # Skip opening '['

        while current_index[0] < length:
            current_char = str_input[current_index[0]]

            # Check for array end
            if current_char == ']':
                current_index[0] += 1  # Skip closing ']'
                break

            # Skip commas between elements
            if current_char == ',':
                current_index[0] += 1
                continue

            # Parse the next value and add to array
            parsed_value = parse_value()
            result_array.append(parsed_value)

        return result_array

    def parse_string():
        """
        Parses a string value from the JSON
        Handles escape sequences with backslash

        Returns:
            The parsed string
        """
        result_string = ''
        current_index[0] += 1  # Skip opening '"'

        while current_index[0] < length:
            current_char = str_input[current_index[0]]

            # Check for string end
            if current_char == '"':
                current_index[0] += 1  # Skip closing '"'
                break

            # Handle escape sequences
            if current_char == '\\':
                current_index[0] += 1  # Skip the backslash
                result_string += str_input[current_index[0]]  # Add the escaped character
            else:
                result_string += current_char

            current_index[0] += 1

        return result_string

    def parse_object():
        """
        Parses an object from the JSON string
        Extracts key-value pairs recursively

        Returns:
            The parsed object (dictionary)
        """
        result_object = {}
        current_index[0] += 1  # Skip opening '{'

        while current_index[0] < length:
            current_char = str_input[current_index[0]]

            # Check for object end
            if current_char == '}':
                current_index[0] += 1  # Skip closing '}'
                break

            # Skip commas between key-value pairs
            if current_char == ',':
                current_index[0] += 1
                continue

            # Parse key (always a string)
            key = parse_string()
            current_index[0] += 1  # Skip the ':' separator

            # Parse the corresponding value
            value = parse_value()
            result_object[key] = value

        return result_object

    def parse_value():
        """
        Main parsing function that determines the type of value to parse
        Based on the first character encountered

        Returns:
            The parsed value of appropriate type
        """
        current_char = str_input[current_index[0]]

        # Determine parsing strategy based on first character
        if current_char == '{':
            return parse_object()
        if current_char == '[':
            return parse_array()
        if current_char == '"':
            return parse_string()
        if current_char == 't':
            return parse_true()
        if current_char == 'f':
            return parse_false()
        if current_char == 'n':
            return parse_null()

        # Default to parsing as number
        return parse_number()

    # Start parsing from the beginning
    return parse_value()
```

## Time and Space Complexity

**Time Complexity: `O(n)`**

The algorithm processes each character in the input string exactly once. The variable `i` serves as a pointer that moves forward through the string without backtracking. Each parsing function (`parseTrue`, `parseFalse`, `parseNull`, `parseNumber`, `parseString`, `parseArray`, `parseObject`) advances the pointer `i` by the number of characters it consumes. Since every character is visited at most once during the entire parsing process, the time complexity is linear with respect to the length of the input string `n`.

**Space Complexity: `O(d + m)`**

The space complexity consists of two main components:

1. **Recursion Stack Space: `O(d)`** - The parser uses recursive calls for nested structures (objects and arrays). The maximum depth of recursion equals the maximum nesting depth `d` of the JSON structure. Each recursive call adds a new frame to the call stack.

2. **Output Space: `O(m)`** - The parsed result requires space proportional to the size of the data being stored. This includes:
   - All primitive values (numbers, booleans, null)
   - All string values stored in the output
   - All object and array containers

   Where `m` represents the total size of the parsed output structure.

The temporary string variables used in `parseNumber` and `parseString` contribute `O(k)` space where `k` is the length of the longest individual string or number, but this is bounded by `O(n)` and typically much smaller than the output space.

Therefore, the total space complexity is `O(d + m)`, where `d` is the maximum nesting depth and `m` is the size of the output data structure.

## Common Pitfalls

### 1. **Index Management with Nested Functions**
One of the most common mistakes is incorrectly managing the shared index across nested functions. Using a simple integer variable won't work because integers are immutable in Python, and nested functions would create local copies instead of modifying the shared state.

**Incorrect approach:**
```python
def jsonParse(str_input: str):
    current_index = 0  # This won't work!

    def parse_string():
        current_index += 1  # UnboundLocalError!
        # ...
```

**Solution:**
Use a mutable container (list) to maintain the reference:
```python
current_index = [0]  # Mutable list
current_index[0] += 1  # Modify the shared value
```

### 2. **Whitespace Handling**
The current implementation assumes no whitespace in the JSON string. Real JSON often contains spaces, tabs, and newlines that need to be skipped.

**Problem scenario:**
```python
input_str = '{ "key" : "value" }'  # Spaces around colon
# Parser will fail trying to parse the space as a value
```

**Solution:**
Add a helper function to skip whitespace:
```python
def skip_whitespace():
    while current_index[0] < length and str_input[current_index[0]] in ' \t\n\r':
        current_index[0] += 1

# Call it before parsing values and after structural characters
def parse_object():
    result_object = {}
    current_index[0] += 1  # Skip '{'
    skip_whitespace()  # Skip any whitespace after '{'

    while current_index[0] < length:
        skip_whitespace()  # Skip whitespace before checking character
        current_char = str_input[current_index[0]]
        # ...
```

### 3. **Number Parsing Edge Cases**
The current number parser doesn't validate the number format and might incorrectly parse invalid JSON numbers.

**Problem scenarios:**
- Leading zeros: `"01"` (invalid in JSON)
- Multiple decimal points: `"1.2.3"`
- Incomplete numbers: `"1e"` (scientific notation without exponent)

**Solution:**
Use a more robust number parsing approach:
```python
def parse_number():
    start = current_index[0]

    # Handle negative sign
    if str_input[current_index[0]] == '-':
        current_index[0] += 1

    # Parse integer part
    if str_input[current_index[0]] == '0':
        current_index[0] += 1
    else:
        while current_index[0] < length and str_input[current_index[0]].isdigit():
            current_index[0] += 1

    # Parse decimal part if exists
    if current_index[0] < length and str_input[current_index[0]] == '.':
        current_index[0] += 1
        while current_index[0] < length and str_input[current_index[0]].isdigit():
            current_index[0] += 1

    # Parse exponent if exists
    if current_index[0] < length and str_input[current_index[0]] in 'eE':
        current_index[0] += 1
        if current_index[0] < length and str_input[current_index[0]] in '+-':
            current_index[0] += 1
        while current_index[0] < length and str_input[current_index[0]].isdigit():
            current_index[0] += 1

    number_str = str_input[start:current_index[0]]
    return float(number_str) if '.' in number_str or 'e' in number_str.lower() else int(number_str)
```

### 4. **Empty Container Handling**
The parser might fail on empty arrays `[]` or objects `{}` because it doesn't check for immediate closing brackets.

**Solution:**
Check for empty containers before attempting to parse elements:
```python
def parse_array():
    result_array = []
    current_index[0] += 1  # Skip '['

    # Check for empty array
    if current_index[0] < length and str_input[current_index[0]] == ']':
        current_index[0] += 1
        return result_array

    # Continue with regular parsing...
```

### 5. **String Escape Sequences**
The current implementation only handles basic backslash escaping but doesn't properly handle all JSON escape sequences like `\n`, `\t`, `\"`, `\\`, etc.

**Solution:**
Implement proper escape sequence handling:
```python
def parse_string():
    result_string = ''
    current_index[0] += 1  # Skip opening '"'

    escape_map = {
        'n': '\n',
        't': '\t',
        'r': '\r',
        'b': '\b',
        'f': '\f',
        '"': '"',
        '\\': '\\',
        '/': '/'
    }

    while current_index[0] < length:
        current_char = str_input[current_index[0]]

        if current_char == '"':
            current_index[0] += 1
            break

        if current_char == '\\':
            current_index[0] += 1
            next_char = str_input[current_index[0]]
            result_string += escape_map.get(next_char, next_char)
        else:
            result_string += current_char

        current_index[0] += 1

    return result_string
```

