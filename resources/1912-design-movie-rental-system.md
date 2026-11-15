# 1912. Design Movie Rental System

> Source: https://algo.monster/liteproblems/1912

## Problem Statement

You need to design a movie rental system for a company with `n` shops. Each shop can have at most one copy of any movie, and each movie copy has a specific rental price at that shop.

The system receives initial data as a 2D array `entries` where each `entries[i] = [shop_i, movie_i, price_i]` means shop `shop_i` has a copy of movie `movie_i` available for rent at price `price_i`.

The system must support four operations:

1. **Search(movie)**: Find the 5 cheapest shops that have an unrented copy of the specified movie. Results should be sorted by:
   - Price (ascending)
   - If prices are equal, by shop ID (ascending)
   - Return fewer than 5 shops if that's all that's available
   - Return empty list if no unrented copies exist

2. **Rent(shop, movie)**: Mark a specific movie copy at a specific shop as rented (removing it from available inventory).

3. **Drop(shop, movie)**: Return a previously rented movie to a specific shop (making it available again).

4. **Report()**: Find the 5 cheapest currently rented movies across all shops. Results should be returned as `[shop, movie]` pairs sorted by:
   - Price (ascending)
   - If prices are equal, by shop ID (ascending)  
   - If shop IDs are also equal, by movie ID (ascending)
   - Return fewer than 5 if that's all that's rented
   - Return empty list if nothing is rented

The solution uses three main data structures:
- `unrented`: A dictionary mapping each movie to a sorted list of `(price, shop)` tuples for available copies
- `shopAndMovieToPrice`: A dictionary storing the price for each `(shop, movie)` combination
- `rented`: A sorted list of `(price, shop, movie)` tuples for all currently rented movies

The `SortedList` data structure automatically maintains sorted order during insertions and deletions, making it efficient to retrieve the cheapest options for both search and report operations.

## Intuition

The key insight is that we need to efficiently maintain and query two separate groups of movies: those available for rent and those currently rented. Since we frequently need to find the "cheapest 5" items from both groups, we need data structures that keep items sorted.

Let's think about what operations we need to perform:
- When searching for a movie, we need the cheapest available copies of that specific movie
- When reporting, we need the cheapest rented movies across ALL movies
- When renting/dropping, we need to move items between the "available" and "rented" groups

This suggests we need:
1. A way to group available movies by movie ID (since search is by movie ID)
2. A way to track all rented movies together (since report looks across all movies)
3. Quick access to prices for any `(shop, movie)` pair

For the available movies, we use a dictionary where each movie ID maps to its available copies. Since we always want the cheapest ones, we store these copies in a `SortedList` as `(price, shop)` tuples. Python naturally sorts tuples by comparing elements left-to-right, so `(price, shop)` automatically gives us the ordering we want.

For rented movies, we use a single `SortedList` containing `(price, shop, movie)` tuples. Again, the tuple ordering naturally handles our tie-breaking rules: first by price, then by shop, then by movie.

The `shopAndMovieToPrice` dictionary is crucial because when we rent or drop a movie, we only know the shop and movie IDs - but we need the price to properly add/remove items from our sorted lists. This dictionary gives us O(1) price lookups.

Using `SortedList` (from the sortedcontainers library) is the key efficiency trick here. It maintains sorted order while allowing O(log n) insertions and deletions, making our rent/drop operations efficient. Getting the first 5 elements is O(1) since they're always at the front of the sorted structure.

**Learn more about [Heap (Priority Queue)](/problems/heap_intro) patterns.**

## Solution Approach

The implementation uses three main data structures to efficiently handle all operations:

### Data Structures

1. **`unrented`**: A `defaultdict(SortedList)` that maps each movie ID to a sorted list of `(price, shop)` tuples representing available copies
2. **`rented`**: A `SortedList` containing `(price, shop, movie)` tuples for all currently rented movies
3. **`shopAndMovieToPrice`**: A dictionary mapping `(shop, movie)` pairs to their prices for quick lookups

### Implementation Details

**Initialization (`__init__`)**:
```python
def __init__(self, n: int, entries: List[List[int]]):
    self.unrented = collections.defaultdict(SortedList)
    self.shopAndMovieToPrice = {}
    self.rented = SortedList()
    for shop, movie, price in entries:
        self.unrented[movie].add((price, shop))
        self.shopAndMovieToPrice[(shop, movie)] = price
```
We iterate through all entries and:
- Add each movie copy to the `unrented` dictionary under its movie ID as a `(price, shop)` tuple
- Store the price in `shopAndMovieToPrice` for future reference
- Initialize `rented` as empty since no movies are initially rented

**Search Operation**:
```python
def search(self, movie: int) -> List[int]:
    return [shop for _, shop in self.unrented[movie][:5]]
```
- Access the sorted list of available copies for the given movie
- Take the first 5 entries (or fewer if less than 5 exist)
- Extract just the shop IDs from the `(price, shop)` tuples
- The `SortedList` ensures these are already in the correct order

**Rent Operation**:
```python
def rent(self, shop: int, movie: int) -> None:
    price = self.shopAndMovieToPrice[(shop, movie)]
    self.unrented[movie].remove((price, shop))
    self.rented.add((price, shop, movie))
```
- Look up the price using our price dictionary
- Remove the `(price, shop)` tuple from the unrented list for this movie
- Add a `(price, shop, movie)` tuple to the rented list
- Both operations maintain sorted order automatically

**Drop Operation**:
```python
def drop(self, shop: int, movie: int) -> None:
    price = self.shopAndMovieToPrice[(shop, movie)]
    self.unrented[movie].add((price, shop))
    self.rented.remove((price, shop, movie))
```
- Reverse of the rent operation
- Look up the price
- Add the movie back to the unrented list
- Remove it from the rented list

**Report Operation**:
```python
def report(self) -> List[List[int]]:
    return [[shop, movie] for _, shop, movie in self.rented[:5]]
```
- Take the first 5 entries from the sorted rented list
- Extract `[shop, movie]` pairs, discarding the price
- The tuple structure `(price, shop, movie)` ensures correct sorting order

### Time Complexity
- **Initialization**: O(m log m) where m is the number of entries
- **Search**: O(1) to access the list, O(5) = O(1) to extract results
- **Rent/Drop**: O(log k) where k is the number of copies of a specific movie or rented movies
- **Report**: O(1) to access first 5 elements

The `SortedList` data structure is crucial for efficiency, providing O(log n) insertions/deletions while maintaining sorted order, making it perfect for this problem's requirements.

## Example Walkthrough

Let's walk through a small example to illustrate how the solution works.

**Initial Setup:**
```
n = 3 shops (shops 0, 1, 2)
entries = [[0, 1, 5], [0, 2, 3], [1, 1, 4], [1, 2, 6], [2, 1, 7]]
```

This means:
- Shop 0 has: Movie 1 at $5, Movie 2 at $3
- Shop 1 has: Movie 1 at $4, Movie 2 at $6  
- Shop 2 has: Movie 1 at $7

After initialization, our data structures look like:
```
unrented = {
    1: [(4, 1), (5, 0), (7, 2)]  # Movie 1 sorted by price
    2: [(3, 0), (6, 1)]          # Movie 2 sorted by price
}

shopAndMovieToPrice = {
    (0, 1): 5, (0, 2): 3,
    (1, 1): 4, (1, 2): 6,
    (2, 1): 7
}

rented = []  # Empty initially
```

**Operation 1: Search(movie=1)**
- Look at `unrented[1]` = `[(4, 1), (5, 0), (7, 2)]`
- Take first 5 (we only have 3): `[(4, 1), (5, 0), (7, 2)]`
- Extract shop IDs: `[1, 0, 2]`
- Returns: `[1, 0, 2]` (shops sorted by price: $4, $5, $7)

**Operation 2: Rent(shop=1, movie=1)**
- Look up price: `shopAndMovieToPrice[(1, 1)]` = 4
- Remove `(4, 1)` from `unrented[1]`
- Add `(4, 1, 1)` to `rented`

Now:
```
unrented[1] = [(5, 0), (7, 2)]
rented = [(4, 1, 1)]
```

**Operation 3: Rent(shop=0, movie=2)**
- Look up price: `shopAndMovieToPrice[(0, 2)]` = 3
- Remove `(3, 0)` from `unrented[2]`
- Add `(3, 0, 2)` to `rented`

Now:
```
unrented[2] = [(6, 1)]
rented = [(3, 0, 2), (4, 1, 1)]  # Automatically sorted by price
```

**Operation 4: Report()**
- Look at `rented` = `[(3, 0, 2), (4, 1, 1)]`
- Take first 5 (we only have 2): `[(3, 0, 2), (4, 1, 1)]`
- Extract `[shop, movie]` pairs: `[[0, 2], [1, 1]]`
- Returns: `[[0, 2], [1, 1]]` (sorted by price: $3, $4)

**Operation 5: Drop(shop=0, movie=2)**
- Look up price: `shopAndMovieToPrice[(0, 2)]` = 3
- Add `(3, 0)` back to `unrented[2]`
- Remove `(3, 0, 2)` from `rented`

Now:
```
unrented[2] = [(3, 0), (6, 1)]  # Automatically re-sorted
rented = [(4, 1, 1)]
```

**Operation 6: Search(movie=2)**
- Look at `unrented[2]` = `[(3, 0), (6, 1)]`
- Take first 5 (we only have 2): `[(3, 0), (6, 1)]`
- Extract shop IDs: `[0, 1]`
- Returns: `[0, 1]` (shops sorted by price: $3, $6)

This example demonstrates how:
1. The `SortedList` structures automatically maintain sort order during insertions and removals
2. The tuple structure `(price, shop)` or `(price, shop, movie)` provides natural sorting
3. The `shopAndMovieToPrice` dictionary enables quick price lookups during rent/drop operations
4. Search operations are fast because we can directly access the sorted list for a specific movie
5. Report operations are fast because all rented movies are maintained in a single sorted structure

## Python Solution

```python
from typing import List
from collections import defaultdict
from sortedcontainers import SortedList


class MovieRentingSystem:
    def __init__(self, n: int, entries: List[List[int]]):
        """
        Initialize the movie renting system.
      
        Args:
            n: Number of shops
            entries: List of [shop, movie, price] entries
        """
        # Store unrented movies: movie_id -> sorted list of (price, shop_id)
        self.unrented = defaultdict(SortedList)
      
        # Store price lookup: (shop_id, movie_id) -> price
        self.shop_and_movie_to_price = {}
      
        # Store currently rented movies: sorted list of (price, shop_id, movie_id)
        self.rented = SortedList()
      
        # Process initial entries
        for shop, movie, price in entries:
            self.unrented[movie].add((price, shop))
            self.shop_and_movie_to_price[(shop, movie)] = price

    def search(self, movie: int) -> List[int]:
        """
        Search for the 5 cheapest shops that have an unrented copy of the given movie.
      
        Args:
            movie: The movie ID to search for
          
        Returns:
            List of up to 5 shop IDs with cheapest prices
        """
        # Return shop IDs from the first 5 (price, shop) tuples
        return [shop for _, shop in self.unrented[movie][:5]]

    def rent(self, shop: int, movie: int) -> None:
        """
        Rent a movie from a specific shop.
      
        Args:
            shop: The shop ID
            movie: The movie ID to rent
        """
        # Get the price for this shop-movie combination
        price = self.shop_and_movie_to_price[(shop, movie)]
      
        # Remove from unrented inventory
        self.unrented[movie].remove((price, shop))
      
        # Add to rented inventory
        self.rented.add((price, shop, movie))

    def drop(self, shop: int, movie: int) -> None:
        """
        Return a rented movie to a specific shop.
      
        Args:
            shop: The shop ID
            movie: The movie ID to return
        """
        # Get the price for this shop-movie combination
        price = self.shop_and_movie_to_price[(shop, movie)]
      
        # Add back to unrented inventory
        self.unrented[movie].add((price, shop))
      
        # Remove from rented inventory
        self.rented.remove((price, shop, movie))

    def report(self) -> List[List[int]]:
        """
        Get the 5 cheapest rented movies.
      
        Returns:
            List of up to 5 [shop_id, movie_id] pairs
        """
        # Return shop and movie IDs from the first 5 rented entries
        return [[shop, movie] for _, shop, movie in self.rented[:5]]


# Your MovieRentingSystem object will be instantiated and called as such:
# obj = MovieRentingSystem(n, entries)
# param_1 = obj.search(movie)
# obj.rent(shop, movie)
# obj.drop(shop, movie)
# param_4 = obj.report()
```

## Time and Space Complexity

**Time Complexity:**

- `__init__(n, entries)`: `O(m * log m)` where `m` is the number of entries. Each entry insertion into the SortedList takes `O(log m)` time, and we process all `m` entries.

- `search(movie)`: `O(1)` to access the unrented movies for a specific movie, and `O(1)` to retrieve up to 5 elements from the sorted list.

- `rent(shop, movie)`: `O(log k + log r)` where `k` is the number of shops having the specific movie and `r` is the number of currently rented movies. The operation includes:
  - `O(1)` to look up the price
  - `O(log k)` to remove from `self.unrented[movie]`
  - `O(log r)` to add to `self.rented`

- `drop(shop, movie)`: `O(log k + log r)` where `k` is the number of shops having the specific movie and `r` is the number of currently rented movies. The operation includes:
  - `O(1)` to look up the price
  - `O(log k)` to add to `self.unrented[movie]`
  - `O(log r)` to remove from `self.rented`

- `report()`: `O(1)` to retrieve up to 5 elements from the sorted list.

**Space Complexity:**

`O(m)` where `m` is the total number of entries. The space is used for:
- `self.unrented`: stores all unrented movie-shop pairs, up to `O(m)` entries
- `self.shopAndMovieToPrice`: stores all shop-movie combinations with their prices, exactly `O(m)` entries
- `self.rented`: stores currently rented movies, at most `O(m)` entries

The total space complexity is `O(m)` as we store each entry in multiple data structures but the total is still proportional to the number of entries.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Tuple Ordering for Sorting**

**Pitfall**: One of the most common mistakes is using the wrong order of elements in tuples, which leads to incorrect sorting behavior.

**Wrong Implementation**:
```python
# Incorrect: Using (shop, price) instead of (price, shop)
self.unrented[movie].add((shop, price))  # Wrong order!

# This would sort by shop ID first, then price - opposite of what we want
```

**Why it's problematic**: Python tuples are compared element by element from left to right. If we put shop ID before price, the system would return shops sorted by their ID rather than by price, completely breaking the "cheapest first" requirement.

**Correct Solution**:
```python
# Correct: Price must come first for proper sorting
self.unrented[movie].add((price, shop))  # Price first, shop second

# For rented items, we need even more careful ordering
self.rented.add((price, shop, movie))  # Sorts by price, then shop, then movie
```

### 2. **Not Handling the SortedList Import Dependency**

**Pitfall**: Forgetting that `SortedList` is not a built-in Python data structure and requires the `sortedcontainers` library.

**Problem Code**:
```python
from sortedcontainers import SortedList  # This might fail if not installed
```

**Solution**: If `sortedcontainers` is not available, you can implement a workaround using built-in structures:

```python
import heapq
from collections import defaultdict

class MovieRentingSystem:
    def __init__(self, n: int, entries: List[List[int]]):
        # Use regular lists and sort when needed
        self.unrented = defaultdict(list)
        self.shop_and_movie_to_price = {}
        self.rented = []
      
        for shop, movie, price in entries:
            heapq.heappush(self.unrented[movie], (price, shop))
            self.shop_and_movie_to_price[(shop, movie)] = price
  
    def search(self, movie: int) -> List[int]:
        # Create a sorted copy without modifying the heap
        temp = sorted(self.unrented[movie])
        return [shop for _, shop in temp[:5]]
  
    def rent(self, shop: int, movie: int) -> None:
        price = self.shop_and_movie_to_price[(shop, movie)]
        # Mark as rented instead of removing (lazy deletion)
        # Would need additional tracking to handle this properly
        heapq.heappush(self.rented, (price, shop, movie))
```

### 3. **Forgetting to Store Prices for Later Lookup**

**Pitfall**: Not maintaining the `shop_and_movie_to_price` dictionary, making it impossible to retrieve prices during rent/drop operations.

**Wrong Implementation**:
```python
def __init__(self, n: int, entries: List[List[int]]):
    self.unrented = defaultdict(SortedList)
    self.rented = SortedList()
  
    for shop, movie, price in entries:
        self.unrented[movie].add((price, shop))
        # Forgot to store: self.shop_and_movie_to_price[(shop, movie)] = price
      
def rent(self, shop: int, movie: int) -> None:
    # Now we can't get the price!
    price = ???  # No way to retrieve this
```

**Correct Solution**: Always store the price mapping during initialization:
```python
self.shop_and_movie_to_price[(shop, movie)] = price
```

### 4. **Assuming Movies are Always Available for Operations**

**Pitfall**: Not considering edge cases where a movie might not exist or operations might be called on already rented/unrented movies.

**Defensive Solution**:
```python
def rent(self, shop: int, movie: int) -> None:
    # Add validation
    if (shop, movie) not in self.shop_and_movie_to_price:
        return  # Movie doesn't exist at this shop
  
    price = self.shop_and_movie_to_price[(shop, movie)]
  
    # Check if actually available to rent
    if (price, shop) in self.unrented[movie]:
        self.unrented[movie].remove((price, shop))
        self.rented.add((price, shop, movie))
```

### 5. **Inefficient Search Without Early Termination**

**Pitfall**: Creating unnecessary copies or sorting entire lists when only 5 elements are needed.

**Inefficient**:
```python
def search(self, movie: int) -> List[int]:
    # Sorts entire list even if we only need 5 elements
    all_shops = sorted([(price, shop) for price, shop in self.unrented[movie]])
    return [shop for _, shop in all_shops[:5]]
```

**Efficient Solution**: The SortedList maintains order, so slicing `[:5]` is O(1) and doesn't require re-sorting:
```python
return [shop for _, shop in self.unrented[movie][:5]]
```

