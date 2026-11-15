# 1948. Delete Duplicate Folders in System

> Source: https://algo.monster/liteproblems/1948

## Problem Statement

You have a file system with many duplicate folders due to a bug. Given a 2D array `paths` where each `paths[i]` represents an absolute path to a folder (e.g., `["one", "two", "three"]` represents `/one/two/three`), you need to identify and remove all duplicate folder structures.

Two folders are considered **identical** if they contain the exact same set of non-empty subfolders with the same structure, regardless of their location in the file system. When folders are identified as identical, both folders and all their subfolders must be marked for deletion.

For example, consider these two folder structures:
- `/a` with subfolders `/a/x`, `/a/x/y`, `/a/z`  
- `/b` with subfolders `/b/x`, `/b/x/y`, `/b/z`

These folders `/a` and `/b` are identical because they have the same subfolder structure (`x`, `x/y`, and `z`). Therefore, all these paths would be marked for deletion:
- `/a`, `/a/x`, `/a/x/y`, `/a/z`
- `/b`, `/b/x`, `/b/x/y`, `/b/z`

However, if `/b` had an additional subfolder `/b/w`, then `/a` and `/b` would no longer be identical. Note that even in this case, `/a/x` and `/b/x` could still be identical if they have the same internal structure.

The deletion process runs only once - any folders that become identical after the initial deletion are not removed.

Your task is to return the paths of all remaining folders after deleting all marked folders. The paths can be returned in any order.

## Intuition

To identify duplicate folder structures, we need a way to represent each folder's complete subfolder structure in a comparable format. Think of it like creating a "fingerprint" for each folder based on its contents.

The key insight is that two folders are identical if they have the same subfolders with the same names and those subfolders are also structurally identical. This suggests a recursive approach - we need to compare not just immediate children, but the entire subtree structure.

We can represent each folder's structure as a string that captures both the names of its subfolders and their internal structures. For example, if a folder has subfolders `x` and `z`, and `x` has a subfolder `y`, we could represent this as something like `"x(y())z()"`. The parentheses capture the hierarchical structure.

To build this representation efficiently, we can:
1. First construct a tree ([trie](/problems/trie_intro)) from all the paths to understand the folder structure
2. Traverse the tree from bottom to top (post-order), building string representations for each subtree
3. Use these string representations as keys to identify duplicates - if two different folders generate the same string representation, they have identical structures

The beauty of this approach is that by processing folders bottom-up, we ensure that when we create a folder's representation, we already know the representations of all its subfolders. This allows us to detect duplicates at any level of the hierarchy.

Once we identify which folders are duplicates (by finding matching string representations), we mark them for deletion. Finally, we traverse the tree again to collect only the paths that weren't marked for deletion.

**Learn more about [Trie](/problems/trie_intro) patterns.**

## Solution Approach

The solution uses a **[Trie](/problems/trie_intro) data structure** combined with **DFS traversal** to efficiently identify and remove duplicate folder structures.

### Step 1: Build the [Trie](/problems/trie_intro) Structure

First, we create a `[Trie](/problems/trie_intro)` class with:
- `children`: A dictionary mapping folder names to their child nodes
- `deleted`: A boolean flag to mark nodes for deletion

We insert all paths into the [trie](/problems/trie_intro):
```python
root = Trie()
for path in paths:
    cur = root
    for name in path:
        if cur.children[name] is None:
            cur.children[name] = Trie()
        cur = cur.children[name]
```

### Step 2: Generate Unique Signatures for Each Subtree

We use a DFS function `dfs(node)` that:
1. Returns an empty string for leaf nodes (folders with no subfolders)
2. For non-leaf nodes, creates a string representation by:
   - Recursively getting signatures for all child nodes
   - Combining each child's name with its signature in format: `"name(child_signature)"`
   - Sorting these representations alphabetically to ensure consistent ordering
   - Concatenating them into a single string

```python
def dfs(node: [Trie](/problems/trie_intro)) -> str:
    if not node.children:
        return ""
    subs: List[str] = []
    for name, child in node.children.items():
        subs.append(f"{name}({dfs(child)})")
    s = "".join(sorted(subs))
```

For example, a folder with subfolders `x` (containing `y`) and `z` (empty) would generate: `"x(y())z()"`

### Step 3: Identify and Mark Duplicates

We maintain a global dictionary `g` that maps each unique signature to the first node that generated it:
- If a signature already exists in `g`, we've found a duplicate
- Mark both the current node and the previously seen node as deleted
- Otherwise, add the signature and node to the dictionary

```python
if s in g:
    node.deleted = g[s].deleted = True
else:
    g[s] = node
```

### Step 4: Collect Remaining Paths

Finally, we perform another DFS traversal `dfs2(node)` to collect all non-deleted paths:
- Skip any node marked as `deleted`
- Build paths by maintaining a running list of folder names
- Add complete paths to the result when we reach valid nodes
- Use backtracking (append/pop) to explore all paths

```python
def dfs2(node: [Trie](/problems/trie_intro)) -> None:
    if node.deleted:
        return
    if path:
        ans.append(path[:])
    for name, child in node.children.items():
        path.append(name)
        dfs2(child)
        path.pop()
```

The time complexity is `O(n × m)` where `n` is the number of paths and `m` is the average path length, as we need to traverse all paths twice. The space complexity is also `O(n × m)` for storing the [trie](/problems/trie_intro) and the signature dictionary.

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

**Input:** `paths = [["a","x"], ["a","x","y"], ["a","z"], ["b","x"], ["b","x","y"], ["b","z"], ["b","w"]]`

**Step 1: Build the Trie**

We construct a trie from all paths:
```
root
├── a
│   ├── x
│   │   └── y
│   └── z
└── b
    ├── x
    │   └── y
    ├── z
    └── w
```

**Step 2: Generate Signatures (Bottom-up DFS)**

Starting from leaf nodes and working up:

- Node `y` under `/a/x`: No children → signature = `""`
- Node `y` under `/b/x`: No children → signature = `""`
- Node `z` under `/a`: No children → signature = `""`
- Node `z` under `/b`: No children → signature = `""`
- Node `w` under `/b`: No children → signature = `""`

- Node `x` under `/a`: Has child `y` with signature `""`
  - Create: `"y()"`
  - Signature = `"y()"`

- Node `x` under `/b`: Has child `y` with signature `""`
  - Create: `"y()"`
  - Signature = `"y()"`

- Node `a`: Has children `x` (signature `"y()"`) and `z` (signature `""`)
  - Create: `["x(y())", "z()"]`
  - Sort alphabetically: `["x(y())", "z()"]`
  - Signature = `"x(y())z()"`

- Node `b`: Has children `x` (signature `"y()"`), `z` (signature `""`), and `w` (signature `""`)
  - Create: `["x(y())", "z()", "w()"]`
  - Sort alphabetically: `["w()", "x(y())", "z()"]`
  - Signature = `"w()x(y())z()"`

**Step 3: Identify Duplicates**

As we generate signatures, we check for duplicates:

- `/a/x/y` signature `""`: First occurrence, store in dictionary
- `/b/x/y` signature `""`: Already exists! Mark both as deleted ✗
- `/a/z` signature `""`: Already exists! Mark both as deleted ✗
- `/b/z` signature `""`: Already exists! Mark both as deleted ✗
- `/b/w` signature `""`: Already exists! Mark all as deleted ✗
- `/a/x` signature `"y()"`: First occurrence, store in dictionary
- `/b/x` signature `"y()"`: Already exists! Mark both as deleted ✗
- `/a` signature `"x(y())z()"`: First occurrence, store in dictionary
- `/b` signature `"w()x(y())z()"`: First occurrence, store in dictionary

**Step 4: Mark Deletions**

After identifying duplicates:
- `/a/x` and `/b/x` are marked as deleted (same signature `"y()"`)
- All their subfolders are automatically deleted when we skip deleted nodes
- `/a` and `/b` have different signatures due to `/b/w`, so they remain

**Step 5: Collect Remaining Paths**

Traversing the trie, skipping deleted nodes:
- Start at root
- Visit `/a` (not deleted) → add `["a"]` to result
  - Try `/a/x` → deleted, skip entire subtree
  - Try `/a/z` → deleted, skip
- Visit `/b` (not deleted) → add `["b"]` to result
  - Try `/b/x` → deleted, skip entire subtree
  - Try `/b/z` → deleted, skip
  - Visit `/b/w` (not deleted) → add `["b","w"]` to result

**Output:** `[["a"], ["b"], ["b","w"]]`

The key insight: `/a/x` and `/b/x` had identical structures (both containing only `y`), so they were removed along with all their subfolders. The parent folders `/a` and `/b` survived because `/b` had an extra subfolder `w`, making their structures different.

## Python Solution

```python
from collections import defaultdict
from typing import Dict, List

class Trie:
    def __init__(self):
        # Dictionary to store child nodes, where key is folder name
        self.children: Dict[str, "Trie"] = {}
        # Flag to mark if this subtree should be deleted as duplicate
        self.deleted: bool = False


class Solution:
    def deleteDuplicateFolder(self, paths: List[List[str]]) -> List[List[str]]:
        # Build the trie structure from all paths
        root = Trie()
        for path in paths:
            current_node = root
            for folder_name in path:
                # Create new node if it doesn't exist
                if folder_name not in current_node.children:
                    current_node.children[folder_name] = Trie()
                current_node = current_node.children[folder_name]
      
        # Dictionary to store serialized subtree signatures and their first occurrence
        signature_to_node: Dict[str, Trie] = {}
      
        def serialize_and_mark_duplicates(node: Trie) -> str:
            """
            Serialize the subtree rooted at node and mark duplicate subtrees.
            Returns a unique string representation of the subtree structure.
            """
            # Empty folder has empty signature
            if not node.children:
                return ""
          
            # Build signature from all child subtrees
            subtree_signatures: List[str] = []
            for folder_name, child_node in node.children.items():
                child_signature = serialize_and_mark_duplicates(child_node)
                # Format: foldername(child_subtree_signature)
                subtree_signatures.append(f"{folder_name}({child_signature})")
          
            # Sort to ensure same structure gives same signature
            current_signature = "".join(sorted(subtree_signatures))
          
            # Check if this subtree structure has been seen before
            if current_signature in signature_to_node:
                # Mark both occurrences for deletion
                node.deleted = True
                signature_to_node[current_signature].deleted = True
            else:
                # First occurrence of this structure
                signature_to_node[current_signature] = node
          
            return current_signature
      
        def collect_remaining_paths(node: Trie, current_path: List[str], result: List[List[str]]) -> None:
            """
            Traverse the trie and collect all non-deleted paths.
            """
            # Skip deleted subtrees
            if node.deleted:
                return
          
            # Add current path to result (except for root which has empty path)
            if current_path:
                result.append(current_path[:])
          
            # Recursively process all children
            for folder_name, child_node in node.children.items():
                current_path.append(folder_name)
                collect_remaining_paths(child_node, current_path, result)
                current_path.pop()
      
        # Step 1: Identify and mark duplicate subtrees
        serialize_and_mark_duplicates(root)
      
        # Step 2: Collect all non-deleted paths
        result: List[List[str]] = []
        collect_remaining_paths(root, [], result)
      
        return result
```

## Time and Space Complexity

**Time Complexity:** `O(N * M * L + N * M * log(M))`

Where:
- `N` = number of paths
- `M` = average number of folders in each path (depth)
- `L` = average length of folder names

Breaking down the operations:

1. **Building the Trie:** `O(N * M)` - iterating through all paths and inserting each folder
2. **First DFS (serialization):** 
   - Each node is visited once: `O(total nodes)`
   - At each node, we create a serialization string by iterating through children and sorting them
   - Sorting at each node: `O(K * log(K))` where `K` is the number of children
   - String concatenation: `O(K * L)` for each node
   - Overall: `O(N * M * L + N * M * log(M))` in the worst case
3. **Second DFS (collecting results):** `O(N * M)` - visiting non-deleted nodes

The dominant term is `O(N * M * L + N * M * log(M))` due to the serialization and sorting operations.

**Space Complexity:** `O(N * M * L)`

Breaking down the space usage:

1. **Trie structure:** `O(N * M)` - storing all nodes
2. **HashMap `g` for duplicate detection:** `O(N * M * L)` - storing serialized strings as keys
3. **Recursion stack depth:** `O(M)` - maximum depth of the folder structure
4. **Result storage:** `O(N * M)` - storing the output paths
5. **Temporary strings during serialization:** `O(M * L)` per recursive call

The dominant term is `O(N * M * L)` due to storing the serialized representations in the hashmap.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Handling of Empty Folders**
A critical pitfall is misunderstanding what constitutes an "empty" folder. In this problem, a folder is considered empty (leaf node) only if it has NO subfolders at all. However, developers often mistakenly treat folders that appear in the input paths as having content.

**Problem Example:**
```python
# Given paths: [["a"], ["a", "x"], ["b"], ["b", "x"]]
# Incorrect assumption: /a and /b are non-empty because they appear as paths
# Correct understanding: /a/x and /b/x are empty (leaf nodes)
```

**Solution:** Always check `node.children` to determine if a folder is empty, not whether it appears as a complete path in the input.

### 2. **Signature Collision Due to Poor Serialization**
Without proper delimiters in the serialization format, different folder structures can generate identical signatures.

**Problem Example:**
```python
# Without delimiters:
# /a/bc with empty child → signature: "bc"
# /ab/c with empty child → signature: "abc"  
# These are different but might generate the same signature!

# Incorrect serialization:
signature = name + child_signature  # No delimiter!
```

**Solution:** Always use delimiters like parentheses in the format `"name(child_signature)"` to ensure unique signatures:
```python
subtree_signatures.append(f"{folder_name}({child_signature})")
```

### 3. **Forgetting to Sort Child Signatures**
The order of children in the trie depends on the input order, but identical structures should generate the same signature regardless of order.

**Problem Example:**
```python
# /a has children: x, y
# /b has children: y, x (different order)
# Without sorting: signatures would be "x()y()" vs "y()x()" - incorrectly different!
```

**Solution:** Always sort the child signatures before concatenation:
```python
current_signature = "".join(sorted(subtree_signatures))
```

### 4. **Modifying the Trie During First Traversal**
Attempting to delete nodes immediately during the duplicate detection phase can corrupt the tree structure and cause missed duplicates.

**Problem Example:**
```python
# Wrong approach - deleting immediately:
if current_signature in signature_to_node:
    # DON'T do this:
    del node.children  # This breaks the traversal!
```

**Solution:** Use a two-phase approach:
1. First pass: Mark nodes with `deleted = True` flag
2. Second pass: Collect only non-deleted paths

### 5. **Including Root Path in Results**
The root node represents the file system root ("/") and should never be included in the output.

**Problem Example:**
```python
# Wrong - might add empty path for root:
def collect_paths(node, path, result):
    result.append(path[:])  # This adds [] for root!
```

**Solution:** Check if the current path is non-empty before adding:
```python
if current_path:  # Only add non-empty paths
    result.append(current_path[:])
```

### 6. **Not Creating a Copy of Path When Adding to Results**
Python lists are mutable references. Without copying, all results would point to the same list object.

**Problem Example:**
```python
# Wrong - all results will be the same (last) path:
result.append(current_path)  # Adds reference, not copy!
```

**Solution:** Always create a copy when adding to results:
```python
result.append(current_path[:])  # Creates a shallow copy
```

