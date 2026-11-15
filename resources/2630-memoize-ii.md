# 2630. Memoize II

> Source: https://algo.monster/liteproblems/2630

## Problem Statement

You need to create a higher-order function called `memoize` that takes any function `fn` as input and returns a memoized version of that function.

A **memoized function** is an optimized version that caches its results. When called with the same inputs multiple times, it returns the cached result instead of recalculating it. This optimization technique trades memory for speed by storing previously computed results.

**Key Requirements:**

1. The input function `fn` can be any function with any number of parameters
2. The function can accept any type of values as parameters
3. Two inputs are considered identical if they are strictly equal (`===` comparison)
4. If the memoized function is called with identical inputs, it should:
   - Return the cached result from the first call
   - NOT execute the original function again

**Example Behavior:**

If you have a function that adds two numbers:
- First call: `memoizedFn(2, 3)` → executes the original function, caches result `5`, returns `5`
- Second call: `memoizedFn(2, 3)` → retrieves cached result, returns `5` without executing original function
- The original function is only called once despite multiple identical calls

The solution uses two Maps:
- `idxMap`: Assigns unique indices to each unique parameter value (handles reference types correctly)
- `cache`: Stores the computed results with keys generated from parameter indices

The caching key is created by mapping each parameter to its unique index and joining them with commas, ensuring that identical parameter combinations produce the same cache key.

## Intuition

The core challenge in memoization is creating a reliable way to identify when we've seen the same inputs before. We need to map function arguments to their computed results.

The first thought might be to use a simple object or Map with the arguments directly as keys. However, this runs into issues:
- JavaScript objects only support string keys
- Converting arguments to strings (like `JSON.stringify`) doesn't work for all types (functions, undefined, symbols)
- We need to handle reference types correctly - two different objects with the same content should be treated as different inputs per the `===` requirement

The key insight is that we need a way to uniquely identify each distinct parameter value we encounter. Since `===` is our equality criteria, we can assign each unique value a numerical ID when we first see it. This works because:
- Primitive values that are `===` equal will map to the same ID
- Reference types (objects, arrays) will each get their own ID since they're only `===` to themselves

With this ID system in place via `idxMap`, we can create cache keys by:
1. Converting each parameter to its unique ID: `params.map(getIdx)`
2. Joining these IDs into a string: `.join(',')`

This gives us a string key like `"0,1,2"` that uniquely represents a specific combination of parameters. 

The `cache` Map then stores these string keys mapped to their computed results. When the memoized function is called:
- Generate the cache key from the parameters
- If the key exists in cache, return the stored result
- If not, compute `fn(...params)`, store it, and return it

This two-Map approach elegantly handles all data types while respecting JavaScript's `===` equality semantics, ensuring that the memoized function behaves exactly like the original except it never computes the same inputs twice.

## Solution Approach

The solution implements memoization using a closure pattern with two Map data structures for efficient caching.

**Data Structures Used:**
1. `idxMap: Map<string, number>` - Maps each unique parameter value to a unique index number
2. `cache: Map<string, any>` - Stores the computed results with composite keys

**Implementation Walkthrough:**

1. **Index Assignment Function (`getIdx`):**
   ```typescript
   const getIdx = (obj: any): number => {
       if (!idxMap.has(obj)) {
           idxMap.set(obj, idxMap.size);
       }
       return idxMap.get(obj)!;
   };
   ```
   - Checks if the parameter value already has an assigned index
   - If not, assigns a new index equal to the current size of `idxMap`
   - Returns the index for this parameter value
   - This ensures each unique value (by `===` comparison) gets a consistent index

2. **Memoized Function Creation:**
   ```typescript
   return function (...params: any) {
       const key = params.map(getIdx).join(',');
       if (!cache.has(key)) {
           cache.set(key, fn(...params));
       }
       return cache.get(key)!;
   };
   ```
   - Takes any number of parameters using rest syntax `...params`
   - Generates a cache key by:
     - Mapping each parameter to its index: `params.map(getIdx)`
     - Joining indices with commas: `.join(',')`
     - Example: `[2, 3]` might become `"0,1"`
   - Checks if this key exists in the cache
   - If not cached, executes `fn(...params)` and stores the result
   - Returns the cached result

**Why This Works:**
- The closure preserves access to `idxMap` and `cache` across function calls
- Using Map instead of plain objects allows any type as keys (including objects and functions)
- The index-based key generation ensures:
  - Same parameters → same key → cached result returned
  - Different parameters → different key → function executed
- The `===` equality is naturally handled since Map uses SameValueZero equality

**Time & Space Complexity:**
- Time: O(n) per call where n is the number of parameters (for key generation)
- Space: O(m × n) where m is the number of unique parameter combinations cached and n is the average number of parameters

## Example Walkthrough

Let's walk through a concrete example with a simple sum function:

```javascript
function sum(a, b) {
    console.log("Computing sum...");
    return a + b;
}

const memoizedSum = memoize(sum);
```

**Initial State:**
- `idxMap`: empty Map
- `cache`: empty Map

**First Call: `memoizedSum(2, 3)`**

1. Parameters: `[2, 3]`
2. Generate cache key:
   - Process parameter `2`:
     - `2` not in `idxMap`
     - Add to `idxMap`: `{2 → 0}`
     - Return index: `0`
   - Process parameter `3`:
     - `3` not in `idxMap`
     - Add to `idxMap`: `{2 → 0, 3 → 1}`
     - Return index: `1`
   - Join indices: `"0,1"`
3. Check cache with key `"0,1"`:
   - Not found in cache
   - Execute `sum(2, 3)` → prints "Computing sum...", returns `5`
   - Store in cache: `{"0,1" → 5}`
4. Return: `5`

**Second Call: `memoizedSum(2, 3)`**

1. Parameters: `[2, 3]`
2. Generate cache key:
   - Process parameter `2`:
     - Found in `idxMap` at index `0`
   - Process parameter `3`:
     - Found in `idxMap` at index `1`
   - Join indices: `"0,1"`
3. Check cache with key `"0,1"`:
   - Found! Value is `5`
   - No function execution (no console output)
4. Return: `5`

**Third Call: `memoizedSum(3, 2)`** (different order)

1. Parameters: `[3, 2]`
2. Generate cache key:
   - Parameter `3` → index `1` (already in idxMap)
   - Parameter `2` → index `0` (already in idxMap)
   - Join indices: `"1,0"` (different from `"0,1"`)
3. Check cache with key `"1,0"`:
   - Not found in cache
   - Execute `sum(3, 2)` → prints "Computing sum...", returns `5`
   - Store in cache: `{"0,1" → 5, "1,0" → 5}`
4. Return: `5`

**Final State:**
- `idxMap`: `{2 → 0, 3 → 1}`
- `cache`: `{"0,1" → 5, "1,0" → 5}`

This example demonstrates how:
- Each unique parameter value gets a consistent index
- The same parameters in the same order produce the same cache key
- Different parameter orders create different cache keys (as required by `===` comparison)
- The original function only executes when there's a cache miss

## Python Solution

```python
from typing import Any, Callable, Dict, Tuple

def memoize(fn: Callable) -> Callable:
    """
    Creates a memoized version of the provided function.
    The memoized function caches results based on the arguments passed.
  
    Args:
        fn: The function to be memoized
      
    Returns:
        A memoized version of the input function
    """
    # Dictionary to store unique identifiers for each unique object/primitive argument
    argument_index_map: Dict[Any, int] = {}
  
    # Dictionary to store cached results with serialized argument keys
    result_cache: Dict[str, Any] = {}
  
    def get_argument_index(argument: Any) -> int:
        """
        Gets or creates a unique index for an argument value.
        This allows handling of object references as distinct cache keys.
      
        Args:
            argument: The argument to get an index for
          
        Returns:
            The unique index for this argument
        """
        # Use id() for unhashable types, otherwise use the value itself as key
        try:
            key = argument
            # Try to hash the argument to check if it's hashable
            hash(argument)
        except TypeError:
            # For unhashable types (like lists, dicts), use object id
            key = id(argument)
      
        # If this argument hasn't been seen before, assign it a new index
        if key not in argument_index_map:
            argument_index_map[key] = len(argument_index_map)
      
        # Return the index for this argument
        return argument_index_map[key]
  
    def memoized_function(*params: Any) -> Any:
        """
        The memoized wrapper function that caches results.
      
        Args:
            *params: Variable arguments passed to the original function
          
        Returns:
            The result of the function call (either cached or newly computed)
        """
        # Create a unique cache key by mapping each parameter to its index and joining
        cache_key: str = ','.join(str(get_argument_index(param)) for param in params)
      
        # If result is not cached, compute and store it
        if cache_key not in result_cache:
            result_cache[cache_key] = fn(*params)
      
        # Return the cached result
        return result_cache[cache_key]
  
    # Return the memoized function
    return memoized_function


# Example usage:
# call_count = 0
# def add(a, b):
#     global call_count
#     call_count += 1
#     return a + b
#
# memoized_fn = memoize(add)
# print(memoized_fn(2, 3))  # 5
# print(memoized_fn(2, 3))  # 5 (retrieved from cache, function not called again)
# print(call_count)  # 1 (function was only called once)
```

## Time and Space Complexity

**Time Complexity:**
- For the `getIdx` function: `O(1)` average case for Map operations (both `has` and `set`/`get`)
- For the memoized function call:
  - First call with specific arguments: `O(n)` where `n` is the number of parameters
    - Creating the key requires mapping over `n` parameters: `O(n)`
    - Each `getIdx` call is `O(1)` average
    - Joining the array into a string: `O(n)`
    - Checking cache and setting value: `O(1)` average for Map operations
    - Plus the time complexity of the original function `fn`
  - Subsequent calls with same arguments: `O(n)` for key generation, but the original function is not executed
- Overall: `O(n + T)` for first call where `T` is the time complexity of the original function, and `O(n)` for cached calls

**Space Complexity:**
- `idxMap`: Stores up to `U` unique parameter values across all function calls, where `U` is the total number of unique objects/primitives passed as arguments: `O(U)`
- `cache`: Stores up to `C` entries where `C` is the number of unique argument combinations: `O(C × R)` where `R` is the average space for storing return values
- Key strings: Each key string takes `O(n × d)` space where `n` is the number of parameters and `d` is the average number of digits in the indices
- Overall: `O(U + C × (n × d + R))` where:
  - `U` = number of unique parameter values
  - `C` = number of unique argument combinations
  - `n` = number of parameters per function call
  - `d` = average digits in indices
  - `R` = average space for return values

## Common Pitfalls

### 1. **Handling Unhashable Arguments**
The Python implementation attempts to handle unhashable types (like lists or dictionaries) by using `id()`, but this creates a critical flaw: two lists with identical content will be treated as different arguments because they have different memory addresses.

**Problem Example:**
```python
def sum_list(lst):
    return sum(lst)

memoized_sum = memoize(sum_list)
print(memoized_sum([1, 2, 3]))  # Computes: 6
print(memoized_sum([1, 2, 3]))  # Computes again: 6 (not cached!)
```

**Solution:**
Convert unhashable types to hashable representations:
```python
def get_argument_index(argument: Any) -> int:
    try:
        # For lists and tuples, convert to tuple (hashable)
        if isinstance(argument, list):
            key = tuple(argument)
        elif isinstance(argument, dict):
            # Convert dict to sorted tuple of items
            key = tuple(sorted(argument.items()))
        elif isinstance(argument, set):
            key = frozenset(argument)
        else:
            key = argument
        hash(key)  # Test if hashable
    except TypeError:
        # Fall back to id() only for truly unhashable objects
        key = id(argument)
  
    if key not in argument_index_map:
        argument_index_map[key] = len(argument_index_map)
    return argument_index_map[key]
```

### 2. **Memory Leaks with Unbounded Cache Growth**
The cache has no size limit and will grow indefinitely, potentially causing memory issues in long-running applications.

**Problem Example:**
```python
# This will create millions of cache entries
for i in range(1000000):
    memoized_fn(i, i+1)  # Each unique pair creates a new cache entry
```

**Solution:**
Implement an LRU (Least Recently Used) cache with size limits:
```python
from collections import OrderedDict

def memoize(fn: Callable, max_size: int = 128) -> Callable:
    cache = OrderedDict()
  
    def memoized_function(*params: Any) -> Any:
        cache_key = create_cache_key(params)
      
        if cache_key in cache:
            # Move to end (most recently used)
            cache.move_to_end(cache_key)
            return cache[cache_key]
      
        result = fn(*params)
        cache[cache_key] = result
      
        # Remove oldest entry if cache exceeds max size
        if len(cache) > max_size:
            cache.popitem(last=False)
      
        return result
  
    return memoized_function
```

### 3. **Nested Mutable Arguments**
The solution doesn't handle nested mutable structures properly. Changes to nested objects after caching can lead to incorrect results.

**Problem Example:**
```python
def process_data(config):
    return config['value'] * 2

memoized_process = memoize(process_data)
config = {'value': 5}
print(memoized_process(config))  # 10

config['value'] = 10  # Modifying the object
print(memoized_process(config))  # Still returns 10 (incorrect!)
```

**Solution:**
Create deep copies of mutable arguments or use immutable data structures:
```python
import copy

def get_argument_index(argument: Any) -> int:
    # Create immutable representation for mutable types
    if isinstance(argument, (list, dict, set)):
        # Create a deep copy and convert to immutable form
        key = str(copy.deepcopy(argument))
    else:
        key = argument
  
    # Rest of the implementation...
```

### 4. **Function Arguments with Default Values**
Memoization doesn't account for default parameter values, treating explicit and default arguments differently.

**Problem Example:**
```python
def multiply(a, b=1):
    return a * b

memoized_multiply = memoize(multiply)
print(memoized_multiply(5))      # Calls function: 5
print(memoized_multiply(5, 1))   # Calls function again: 5 (should be cached!)
```

**Solution:**
Normalize arguments by including default values:
```python
import inspect

def memoize(fn: Callable) -> Callable:
    sig = inspect.signature(fn)
  
    def memoized_function(*args, **kwargs):
        # Bind arguments to get normalized form with defaults
        bound = sig.bind(*args, **kwargs)
        bound.apply_defaults()
      
        # Create cache key from all parameters (including defaults)
        cache_key = str(bound.arguments)
      
        # Rest of caching logic...
```

