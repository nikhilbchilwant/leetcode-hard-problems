# 2102. Sequentially Ordinal Rank Tracker

> Source: https://algo.monster/liteproblems/2102

## Problem Statement

You need to build a tracking system for scenic locations that can:
1. Add locations one at a time (each with a unique name and an integer score)
2. Query for the i-th best location, where i equals the number of times the system has been queried

**Ranking Rules:**
- Locations are ranked by score (higher score = better ranking)
- If two locations have equal scores, the one with the lexicographically smaller name ranks higher

**System Operations:**

The `SORTracker` class needs three methods:
- `SORTracker()`: Initialize an empty tracking system
- `add(name, score)`: Add a new scenic location with given name and score
- `get()`: Return the i-th best location's name, where i is the cumulative query count

**Key Behavior of `get()`:**
- First call returns the 1st best location
- Second call returns the 2nd best location  
- Third call returns the 3rd best location
- And so on...

The problem guarantees that the number of queries will never exceed the number of locations added.

**Example Flow:**
If you add locations A(score=5), B(score=3), C(score=5), D(score=7):
- Ranking would be: D(7) → A(5) → C(5) → B(3) 
- Note: A ranks before C because "A" < "C" lexicographically
- First `get()` returns "D"
- Second `get()` returns "A"  
- Third `get()` returns "C"
- Fourth `get()` returns "B"

## Intuition

The core challenge is maintaining a dynamically sorted collection of locations while efficiently retrieving the i-th best element where i increments with each query.

Since we need to:
1. Keep locations sorted by score (descending) and name (ascending for ties)
2. Access elements by index position
3. Add new locations that automatically get placed in the correct sorted position

A sorted data structure like `SortedList` is ideal. It maintains order automatically when elements are added and allows O(log n) insertion and O(1) index access.

The clever trick is storing `(-score, name)` tuples instead of `(score, name)`. Why? Python's default tuple comparison works left-to-right:
- First compares the first elements
- If equal, compares the second elements

By negating the score, we convert the natural ascending sort into descending order for scores. When scores are equal (negated values are equal), it naturally falls back to comparing names alphabetically, which is exactly what we want.

For tracking which element to return, we maintain a counter `i` that starts at -1. Each `get()` call increments `i` by 1, so:
- First call: i becomes 0 (return 0th element - the best)
- Second call: i becomes 1 (return 1st element - second best)
- And so on...

This approach elegantly handles both the sorting requirements and the sequential access pattern without complex logic or multiple data structures.

**Learn more about [Data Stream](/problems/median_of_data_stream) and [Heap (Priority Queue)](/problems/heap_intro) patterns.**

## Solution Approach

The solution uses a `SortedList` data structure to maintain locations in sorted order and a counter variable to track query positions.

**Data Structure Setup:**
- `self.sl = SortedList()`: Creates an ordered container that automatically maintains sorted order
- `self.i = -1`: Initialize query counter to -1 (will become 0 on first `get()` call)

**Adding Locations (`add` method):**
```python
def add(self, name: str, score: int) -> None:
    self.sl.add((-score, name))
```
- Store each location as a tuple `(-score, name)`
- Negating the score achieves descending order sorting:
  - Location with score 10 becomes `(-10, name)`
  - Location with score 5 becomes `(-5, name)`
  - Since -10 < -5, the higher score appears first
- The `SortedList` automatically inserts the tuple in the correct position based on:
  1. First: Negative score (ascending, which gives us descending by original score)
  2. Second: Name (ascending alphabetically when scores tie)

**Retrieving Locations (`get` method):**
```python
def get(self) -> str:
    self.i += 1
    return self.sl[self.i][1]
```
- Increment the counter `i` to track cumulative queries
- Access the i-th element using index: `self.sl[self.i]`
- Return the name part of the tuple: `[1]` gets the second element (name) from `(-score, name)`

**Time Complexity:**
- `add()`: O(log n) for inserting into sorted position
- `get()`: O(1) for index-based access

**Space Complexity:**
- O(n) where n is the number of locations stored

This approach elegantly handles the ranking requirements without manual sorting or complex comparisons, using Python's tuple comparison and the `SortedList`'s automatic ordering.

## Example Walkthrough

Let's walk through a small example to illustrate how the solution works:

**Initial Setup:**
- Create a `SORTracker` with an empty `SortedList` and `i = -1`

**Step 1: Add "park" with score 4**
- Call: `add("park", 4)`
- We store `(-4, "park")` in the SortedList
- SortedList contents: `[(-4, "park")]`

**Step 2: Add "beach" with score 6**
- Call: `add("beach", 6)`
- We store `(-6, "beach")` in the SortedList
- The tuple `(-6, "beach")` is automatically inserted before `(-4, "park")` because -6 < -4
- SortedList contents: `[(-6, "beach"), (-4, "park")]`

**Step 3: First get() call**
- Increment `i` from -1 to 0
- Access `self.sl[0]` which is `(-6, "beach")`
- Return the name part: `"beach"`

**Step 4: Add "lake" with score 6**
- Call: `add("lake", 6)`
- We store `(-6, "lake")` in the SortedList
- Since -6 == -6 (same negated score), Python compares the names
- "beach" < "lake" alphabetically, so the order is maintained
- SortedList contents: `[(-6, "beach"), (-6, "lake"), (-4, "park")]`

**Step 5: Second get() call**
- Increment `i` from 0 to 1
- Access `self.sl[1]` which is `(-6, "lake")`
- Return the name part: `"lake"`

**Step 6: Add "mountain" with score 4**
- Call: `add("mountain", 4)`
- We store `(-4, "mountain")` in the SortedList
- Both `(-4, "mountain")` and `(-4, "park")` have the same negated score
- "mountain" < "park" alphabetically, so it's inserted before "park"
- SortedList contents: `[(-6, "beach"), (-6, "lake"), (-4, "mountain"), (-4, "park")]`

**Step 7: Third get() call**
- Increment `i` from 1 to 2
- Access `self.sl[2]` which is `(-4, "mountain")`
- Return the name part: `"mountain"`

**Final Ranking Explanation:**
1. "beach" (score 6) - highest score, lexicographically first among score 6
2. "lake" (score 6) - highest score, lexicographically second among score 6
3. "mountain" (score 4) - lower score, lexicographically first among score 4
4. "park" (score 4) - lower score, lexicographically second among score 4

The negation trick converts our "higher score is better" requirement into something that works with ascending sort order, while the tuple comparison automatically handles the lexicographic tiebreaker.

## Python Solution

```python
from sortedcontainers import SortedList

class SORTracker:
    def __init__(self):
        # Initialize a sorted list to maintain locations in sorted order
        # Stores tuples of (-score, name) for automatic sorting
        self.sorted_locations = SortedList()
      
        # Track the current index for get() queries
        # Starts at -1 since it's incremented before each retrieval
        self.current_index = -1

    def add(self, name: str, score: int) -> None:
        """
        Add a new location with its score to the tracker.
      
        Args:
            name: The name of the location
            score: The score of the location
        """
        # Store as (-score, name) tuple for sorting:
        # - Negative score ensures higher scores come first
        # - Name as second element provides lexicographic tiebreaking
        self.sorted_locations.add((-score, name))

    def get(self) -> str:
        """
        Get the next best location based on previous queries.
      
        Returns:
            The name of the (i+1)-th best location where i is the number
            of times get() has been called previously.
        """
        # Increment index to get the next best location
        self.current_index += 1
      
        # Return the name (second element) of the tuple at current index
        return self.sorted_locations[self.current_index][1]


# Your SORTracker object will be instantiated and called as such:
# obj = SORTracker()
# obj.add(name, score)
# param_2 = obj.get()
```

## Time and Space Complexity

**Time Complexity:**
- `__init__()`: `O(1)` - Simply initializes an empty SortedList and sets a counter.
- `add(name, score)`: `O(log n)` - The SortedList maintains sorted order using a balanced tree structure internally. Adding an element requires finding the correct position and inserting, which takes logarithmic time where `n` is the number of elements already in the list.
- `get()`: `O(log n)` - Accessing an element by index in a SortedList takes `O(log n)` time due to its tree-based implementation, even though it might appear to be `O(1)` like a regular list access.

**Space Complexity:**
- `O(n)` - The space complexity is linear with respect to the number of attractions added. The SortedList stores all `n` attractions that have been added, where each entry contains a tuple of `(-score, name)`. The counter variable `i` uses constant additional space.

The code uses a SortedList to maintain attractions sorted by score (in descending order due to negation) and name (lexicographically as a tiebreaker). The negation of the score ensures that higher scores come first when sorted in ascending order.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Score Ordering - Forgetting to Negate**
A frequent mistake is storing the score as-is instead of negating it, which results in ascending score order (worst locations first).

**Incorrect Implementation:**
```python
def add(self, name: str, score: int) -> None:
    self.sorted_locations.add((score, name))  # Wrong: sorts in ascending order
```

**Why it fails:** With positive scores, the SortedList sorts in ascending order, so a location with score 3 would appear before score 7, giving us the worst locations first.

**Correct Solution:**
```python
def add(self, name: str, score: int) -> None:
    self.sorted_locations.add((-score, name))  # Correct: negating for descending order
```

### 2. **Index Management Error - Starting at 0**
Another common error is initializing the query counter to 0 instead of -1.

**Incorrect Implementation:**
```python
def __init__(self):
    self.sorted_locations = SortedList()
    self.current_index = 0  # Wrong: starts at 0

def get(self) -> str:
    result = self.sorted_locations[self.current_index][1]
    self.current_index += 1  # Incrementing after access
    return result
```

**Why it fails:** This pattern increments after accessing, which works but is less clean. More critically, if someone forgets to increment or places it incorrectly, it causes off-by-one errors.

**Correct Solution:**
```python
def __init__(self):
    self.current_index = -1  # Start at -1

def get(self) -> str:
    self.current_index += 1  # Increment first, then access
    return self.sorted_locations[self.current_index][1]
```

### 3. **Using Standard List with Manual Sorting**
Attempting to use a regular list and sorting after each addition is inefficient.

**Inefficient Implementation:**
```python
def __init__(self):
    self.locations = []
    self.current_index = -1

def add(self, name: str, score: int) -> None:
    self.locations.append((score, name))
    self.locations.sort(key=lambda x: (-x[0], x[1]))  # O(n log n) every time!
```

**Why it's problematic:** This requires O(n log n) time for every addition, whereas SortedList only needs O(log n) for insertion.

**Correct Solution:** Use SortedList which maintains order automatically with O(log n) insertion.

### 4. **Misunderstanding Lexicographic Ordering with Scores**
Some might try complex custom comparisons when the tuple structure handles it automatically.

**Overcomplicated Implementation:**
```python
def add(self, name: str, score: int) -> None:
    # Trying to manually handle comparison logic
    position = 0
    for i, (s, n) in enumerate(self.sorted_locations):
        if score > -s or (score == -s and name < n):
            position = i
            break
    self.sorted_locations.insert(position, (-score, name))
```

**Correct Solution:** Trust Python's tuple comparison - it naturally compares element by element in order, handling both score and name comparisons correctly.

