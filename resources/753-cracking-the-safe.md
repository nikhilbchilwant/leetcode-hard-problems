# 753. Cracking the Safe

> Source: https://algo.monster/liteproblems/753

## Problem Statement

You have a safe that's protected by a password. The password consists of `n` digits, where each digit can be any value from `0` to `k-1`.

The safe has an interesting way of checking passwords. Every time you type a digit, it checks the **most recent `n` digits** that were entered to see if they match the password.

For example, if the correct password is `"345"` (where `n=3`) and you type `"012345"`:
- After typing `0`: checks `"0"` (only 1 digit so far) - incorrect
- After typing `1`: checks `"01"` (only 2 digits so far) - incorrect  
- After typing `2`: checks `"012"` (now we have 3 digits) - incorrect
- After typing `3`: checks `"123"` (the last 3 digits) - incorrect
- After typing `4`: checks `"234"` (the last 3 digits) - incorrect
- After typing `5`: checks `"345"` (the last 3 digits) - correct! The safe opens.

Your task is to find the **shortest possible string** that, when entered, will guarantee to unlock the safe at some point. This string should contain every possible `n`-digit combination that can be formed using digits from `0` to `k-1`.

The solution uses an Eulerian circuit approach by constructing a directed [graph](/problems/graph_intro) where:
- Each node represents a string of length `n-1` using digits `0` to `k-1`
- Each edge from node `u` to node `v` is labeled with a digit `c` (from `0` to `k-1`)
- The edge represents that appending digit `c` to node `u` and taking the last `n-1` characters gives you node `v`
- Each edge essentially represents one possible `n`-digit password

Since each node has exactly `k` incoming and `k` outgoing edges, an Eulerian circuit exists in this [graph](/problems/graph_intro). Traversing this circuit gives us the minimum-length string that contains all possible passwords.

## Intuition

The key insight is recognizing that we need a string containing every possible `n`-digit password as a substring. If we think naively, we might concatenate all `k^n` possible passwords, giving us a string of length `n × k^n`. But we can do much better by overlapping passwords.

Consider passwords `"123"` and `"234"` - they share the substring `"23"`. If we're clever about ordering, we can write `"1234"` which contains both passwords in just 4 characters instead of 6. This overlap property is crucial.

To systematically find the shortest string with all overlaps, we can model this as a [graph](/problems/graph_intro) problem. Think of each `(n-1)`-digit string as a node. For example, with `n=3`, the nodes would be `"00"`, `"01"`, `"02"`, ..., `"12"`, `"23"`, etc.

An edge from node `u` to node `v` exists if we can append a digit to `u` and get `v` by taking the last `(n-1)` digits. For instance, from node `"12"`, we can go to node `"23"` by appending `"3"` (giving us `"123"`, whose last 2 digits are `"23"`). This edge represents the password `"123"`.

Now here's the beautiful part: each node has exactly `k` outgoing edges (one for each digit we can append) and exactly `k` incoming edges. This means our [graph](/problems/graph_intro) has an Eulerian circuit - a path that visits every edge exactly once and returns to the starting point.

Finding an Eulerian circuit gives us the minimum-length string because:
1. Each edge represents a unique `n`-digit password
2. Traversing each edge once means including each password exactly once
3. The overlapping is maximized because consecutive edges share `(n-1)` digits by construction

We use DFS to find this Eulerian circuit. Starting from any node (typically `"0...0"`), we explore all outgoing edges, marking them as visited. The path we take, when we record the digits on the edges, gives us our answer. We append the starting node's digits at the end to complete the circuit representation as a string.

**Learn more about [Depth-First Search](/problems/dfs_intro) and [Graph](/problems/graph_intro) patterns.**

## Solution Approach

The implementation uses DFS to find an Eulerian circuit in the [graph](/problems/graph_intro) we conceptualized. Let's walk through the key components:

**[Graph](/problems/graph_intro) Representation:**
Instead of explicitly building the graph, we use an implicit representation. Each node is represented as an integer (the decimal value of the `(n-1)`-digit string). For example, with `n=3`, the node `"12"` is represented as the integer `12`.

**Key Variables:**
- `mod = 10^(n-1)`: This helps us extract the last `(n-1)` digits when we append a new digit
- `vis`: A set to track which edges (passwords) we've already used
- `ans`: A list to collect the digits we traverse

**DFS Algorithm:**
```python
def dfs(u):
    for x in range(k):
        e = u * 10 + x
        if e not in vis:
            vis.add(e)
            v = e % mod
            dfs(v)
            ans.append(str(x))
```

The DFS works as follows:
1. From current node `u`, we try all `k` possible digits (`x` from `0` to `k-1`)
2. `e = u * 10 + x` creates the edge identifier (the full `n`-digit password). For example, if `u=12` and `x=3`, then `e=123`
3. We check if this edge/password has been visited
4. If not visited, we mark it as visited and calculate the next node: `v = e % mod`. This extracts the last `(n-1)` digits. For `e=123` and `mod=100`, we get `v=23`
5. We recursively call DFS on the new node `v`
6. **Important:** We append the digit `x` to our answer AFTER the recursive call. This is because we're building the Eulerian circuit in reverse order (post-order traversal)

**Final Assembly:**
```python
dfs(0)
ans.append("0" * (n - 1))
return "".join(ans)
```

We start the DFS from node `0` (representing `"00...0"` with `n-1` zeros). After the DFS completes, we append `n-1` zeros to represent the starting node itself. Since we collected digits in reverse order during DFS, joining them gives us the correct Eulerian circuit.

**Why This Works:**
The algorithm ensures we visit each edge (password) exactly once through the `vis` set. The post-order collection of digits ensures that when we reverse the path (implicitly done by collecting in post-order), we get a valid sequence where each consecutive `n` digits form a unique password that appears in our [graph](/problems/graph_intro).

## Example Walkthrough

Let's walk through a small example with `n=2` and `k=2` (binary digits: 0 and 1).

**Step 1: Identify all possible passwords**
With `n=2` and `k=2`, we have 2² = 4 possible passwords: `"00"`, `"01"`, `"10"`, `"11"`

**Step 2: Construct the graph conceptually**
- Nodes represent all 1-digit strings: `"0"` and `"1"` 
- Edges represent 2-digit passwords:
  - From node `"0"`:
    - Append `0` → password `"00"` → leads to node `"0"` (last 1 digit of "00")
    - Append `1` → password `"01"` → leads to node `"1"` (last 1 digit of "01")
  - From node `"1"`:
    - Append `0` → password `"10"` → leads to node `"0"` (last 1 digit of "10")
    - Append `1` → password `"11"` → leads to node `"1"` (last 1 digit of "11")

**Step 3: Execute DFS to find Eulerian circuit**
Starting from node `0`:

```
Initial: u=0, vis={}, ans=[]

DFS(0):
  Try x=0: e=00, not visited
    Add 00 to vis
    Next node v = 00 % 10 = 0
    DFS(0):
      Try x=0: e=00, already visited, skip
      Try x=1: e=01, not visited
        Add 01 to vis
        Next node v = 01 % 10 = 1
        DFS(1):
          Try x=0: e=10, not visited
            Add 10 to vis
            Next node v = 10 % 10 = 0
            DFS(0):
              Try x=0: e=00, already visited, skip
              Try x=1: e=01, already visited, skip
              Return
            Append "0" to ans → ans=["0"]
          Try x=1: e=11, not visited
            Add 11 to vis
            Next node v = 11 % 10 = 1
            DFS(1):
              Try x=0: e=10, already visited, skip
              Try x=1: e=11, already visited, skip
              Return
            Append "1" to ans → ans=["0","1"]
          Return
        Append "1" to ans → ans=["0","1","1"]
      Return
    Append "0" to ans → ans=["0","1","1","0"]
  Try x=1: e=01, already visited, skip
  Return
```

**Step 4: Build final string**
- After DFS: `ans = ["0","1","1","0"]`
- Append starting node (n-1 = 1 zero): `ans = ["0","1","1","0","0"]`
- Join to get final string: `"01100"`

**Step 5: Verify the result**
Let's check that `"01100"` contains all passwords:
- Position 0-1: `"01"` ✓
- Position 1-2: `"11"` ✓
- Position 2-3: `"10"` ✓
- Position 3-4: `"00"` ✓

All 4 possible passwords are present! The string length is 5, which is optimal (we need at least n + k^n - 1 = 2 + 4 - 1 = 5 characters to contain all 4 two-digit passwords with maximum overlap).

## Python Solution

```python
import sys


class Solution:
    def crackSafe(self, n: int, k: int) -> str:
        """
        Find the minimum string that contains all possible n-digit passwords using digits 0 to k-1.
        Uses Hierholzer's algorithm to find an Eulerian path in a de Bruijn graph.
      
        Args:
            n: Length of each password
            k: Number of available digits (0 to k-1)
      
        Returns:
            A string containing all possible n-digit passwords as substrings
        """
        def dfs(current_node: int) -> None:
            """
            Depth-first search to traverse all edges in the graph exactly once.
          
            Args:
                current_node: Current (n-1)-digit node in the de Bruijn graph
            """
            # Try all possible digits to append
            for digit in range(k):
                # Create an edge by appending the digit to current node
                # Edge represents an n-digit combination
                edge = current_node * 10 + digit
              
                # Check if this edge has been visited
                if edge not in visited_edges:
                    visited_edges.add(edge)
                  
                    # Calculate the next node by removing the leftmost digit
                    # This creates an (n-1)-digit node for the next iteration
                    next_node = edge % modulo
                  
                    # Recursively visit the next node
                    dfs(next_node)
                  
                    # Append the digit to result after visiting all edges from next_node
                    # (Hierholzer's algorithm: build path in reverse)
                    result.append(str(digit))
      
        # The DFS can be up to k^n = 4096 calls deep, above Python's default limit of 1000
        sys.setrecursionlimit(10000)

        # Calculate modulo for extracting last (n-1) digits
        modulo = 10 ** (n - 1)
      
        # Set to track visited edges (n-digit combinations)
        visited_edges = set()
      
        # List to store the result digits in reverse order
        result = []
      
        # Start DFS from node 0 (representing "00...0" with n-1 zeros)
        dfs(0)
      
        # Append the initial (n-1) zeros for the starting node
        result.append("0" * (n - 1))
      
        # Join the result list to form the final string
        return "".join(result)
```

## Time and Space Complexity

The time complexity is `O(k^n)`, and the space complexity is `O(k^n)`.

**Time Complexity Analysis:**
- The algorithm performs a depth-first search (DFS) on a De Bruijn graph where each node represents a string of length `n-1` with `k` possible characters
- The total number of edges in this graph equals the number of possible strings of length `n`, which is `k^n`
- Each edge is visited exactly once due to the `vis` set tracking visited edges
- Each edge visit involves constant time operations (set lookup, set insertion, modulo operation, string append)
- Therefore, the time complexity is `O(k^n)`

**Space Complexity Analysis:**
- The `vis` set stores all visited edges, which can contain up to `k^n` elements (all possible n-length strings)
- The `ans` list stores the result string, which has length `k^n + (n-1)` in the worst case
- The recursion stack depth is at most `k^n` in the worst case when traversing the entire graph (which is why the solution raises Python's recursion limit)
- The dominant factor is `O(k^n)` for both the visited set and the recursion stack
- Therefore, the space complexity is `O(k^n)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Edge Representation for k > 10**

The solution uses a base-10 representation (`u * 10 + x`), which is correct because the problem guarantees `k ≤ 10`, so every digit fits in one decimal place. If the code is adapted to a variant with `k > 10`, digits can be two-digit numbers (like 10, 11, etc.), and the edge calculation breaks down.

**Problem Example:**
- If `n=2, k=12`, and we're at node representing "11", trying to append digit "10":
- Base-10 code: `edge = 11 * 10 + 10 = 120`
- This incorrectly represents the sequence "11" + "10" → "1110" as 120
- The modulo operation `120 % 10 = 0` gives wrong next node

**Solution:**
Use base-k arithmetic instead of base-10:
```python
def crackSafe(self, n: int, k: int) -> str:
    def dfs(current_node: int) -> None:
        for digit in range(k):
            # Use base-k representation
            edge = current_node * k + digit  # Changed from * 10
          
            if edge not in visited_edges:
                visited_edges.add(edge)
                next_node = edge % modulo
                dfs(next_node)
                result.append(str(digit))
  
    # Use k^(n-1) instead of 10^(n-1)
    modulo = k ** (n - 1)  # Changed from 10 ** (n - 1)
    visited_edges = set()
    result = []
  
    dfs(0)
    result.append("0" * (n - 1))
    return "".join(result)
```

### 2. **Recursion Depth for Large Inputs**

The recursive DFS goes one call deeper for each edge on the current path, so the depth can approach `k^n`. With `k^n` up to 4096, this exceeds Python's default recursion limit of 1000 (for example, `n = 4, k = 8` raises `RecursionError`). The solution raises the limit with `sys.setrecursionlimit(10000)`. Another option is an iterative DFS with an explicit stack.

**Iterative alternative:**
```python
def crackSafe(self, n: int, k: int) -> str:
    modulo = k ** (n - 1)
    visited_edges = set()
    result = []

    # Each stack entry is (node, digit used to reach it)
    stack = [(0, None)]
    # Track next digit to try for each node
    next_digit = [0] * modulo

    while stack:
        u, digit_in = stack[-1]
        if next_digit[u] < k:
            digit = next_digit[u]
            next_digit[u] += 1
            edge = u * k + digit
            if edge not in visited_edges:
                visited_edges.add(edge)
                stack.append((edge % modulo, digit))
        else:
            # All edges from u have been explored: record the digit that led here
            stack.pop()
            if digit_in is not None:
                result.append(str(digit_in))

    result.append("0" * (n - 1))
    return "".join(result)
```

Storing the incoming digit on the stack matters. Recomputing it from the previous node fails when `n = 1`, because every edge then leads back to the same node and the digit cannot be recovered.

### 3. **Edge Case: n = 1**

When `n = 1`, the graph degenerates to a single node with k self-loops. The current approach still works but is unnecessarily complex.

**Solution:**
Add special handling for n = 1:
```python
def crackSafe(self, n: int, k: int) -> str:
    if n == 1:
        # Simply concatenate all digits from 0 to k-1
        return "".join(str(i) for i in range(k))
  
    # Rest of the original algorithm...
```

### 4. **Missing Validation for k = 1**

When `k = 1`, there's only one possible digit (0), so the only possible password is "000...0" (n zeros).

**Solution:**
Add special case handling:
```python
def crackSafe(self, n: int, k: int) -> str:
    if k == 1:
        return "0" * n
  
    # Rest of the algorithm...
```

