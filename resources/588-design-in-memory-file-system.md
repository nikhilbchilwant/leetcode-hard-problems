# 588. Design In-Memory File System

> Source: https://algo.monster/liteproblems/588

## Problem Statement

This problem asks you to design an in-memory file system data structure that supports basic file and directory operations.

The file system should support four main operations:

1. **`ls(path)`**: List the contents at a given path
   - If the path points to a file, return a list containing only that file's name
   - If the path points to a directory, return all files and subdirectories within it in lexicographic (alphabetical) order

2. **`mkdir(path)`**: Create a new directory
   - The path given will not already exist
   - If intermediate directories in the path don't exist, create them automatically (similar to `mkdir -p` in Unix)

3. **`addContentToFile(filePath, content)`**: Add content to a file
   - If the file doesn't exist, create it with the given content
   - If the file already exists, append the new content to the existing content

4. **`readContentFromFile(filePath)`**: Read and return all content from a file

The solution uses a [Trie](/problems/trie_intro) (prefix tree) data structure to represent the file system hierarchy. Each node in the Trie can represent either a file or a directory:
- Directories contain children (other files or directories)
- Files contain content and have a flag marking them as files

The path parsing works by splitting paths on `/` characters. For example, `/a/b/c` would be split into components `['', 'a', 'b', 'c']`, where the empty string represents the root directory.

Key implementation details:
- The `insert` method navigates through the path components, creating nodes as needed
- The `search` method traverses the [Trie](/problems/trie_intro) to find a specific file or directory
- File content is stored as a list of strings that get concatenated when read
- Directory listings are automatically sorted to maintain lexicographic order

## Intuition

When thinking about how to represent a file system in memory, we need a hierarchical structure that can efficiently store both files and directories. The natural choice is a tree structure since file systems are inherently tree-like - directories can contain other directories and files, forming a parent-child relationship.

A [Trie](/problems/trie_intro) (prefix tree) is particularly well-suited for this problem because:

1. **Path components are naturally shared**: Multiple paths like `/a/b/c` and `/a/b/d` share the common prefix `/a/b`. A [Trie](/problems/trie_intro) efficiently stores these shared prefixes without duplication.

2. **Easy navigation**: We can traverse from root to any file/directory by following the path components one by one, similar to how we navigate through actual file systems.

3. **Dynamic structure**: We can easily add new directories and files at any level without restructuring the entire tree.

The key insight is to treat each path component (the parts between `/`) as a step in our [Trie](/problems/trie_intro) traversal. For instance, the path `/home/user/document.txt` becomes a series of steps: root → home → user → document.txt.

To distinguish between files and directories, we use a boolean flag `isFile`. This is necessary because:
- Files can have content but cannot have children
- Directories can have children but don't have content
- When we `ls` a path, we need different behavior for files (return just the filename) vs directories (return all children)

For handling file content, using a list of strings rather than a single string is clever because it makes appending new content efficient - we just add to the list rather than creating new string objects each time. When reading, we simply join all the pieces together.

The [sorting](/problems/sorting_summary) requirement for `ls` is handled by using Python's `sorted()` function on the dictionary keys, which gives us lexicographic ordering automatically.

**Learn more about [Trie](/problems/trie_intro) and [Sorting](/problems/sorting_summary) patterns.**

## Solution Approach

The solution implements a file system using a [Trie](/problems/trie_intro) data structure with two main classes: `Trie` for the tree nodes and `FileSystem` as the main interface.

### [Trie](/problems/trie_intro) Node Structure

Each `[Trie](/problems/trie_intro)` node contains:
- `name`: Stores the filename (only for files)
- `isFile`: Boolean flag to distinguish files from directories
- `content`: List of strings to store file content
- `children`: Dictionary mapping child names to their Trie nodes

### Core Methods Implementation

**1. Insert Method**
```python
def insert(self, path, isFile):
    node = self
    ps = path.split('/')
    for p in ps[1:]:  # Skip empty string from leading '/'
        if p not in node.children:
            node.children[p] = [Trie](/problems/trie_intro)()
        node = node.children[p]
    node.isFile = isFile
    if isFile:
        node.name = ps[-1]
    return node
```
- Splits the path by `/` and traverses/creates nodes as needed
- Marks the final node as file or directory
- Returns the final node for further operations

**2. Search Method**
```python
def search(self, path):
    node = self
    if path == '/':
        return node
    ps = path.split('/')
    for p in ps[1:]:
        if p not in node.children:
            return None
        node = node.children[p]
    return node
```
- Handles root directory as special case
- Traverses the tree following path components
- Returns `None` if path doesn't exist

### FileSystem Operations

**1. ls Operation**
```python
def ls(self, path: str) -> List[str]:
    node = self.root.search(path)
    if node is None:
        return []
    if node.isFile:
        return [node.name]
    return sorted(node.children.keys())
```
- Searches for the node at given path
- For files: returns single-element list with filename
- For directories: returns sorted list of all children names

**2. mkdir Operation**
```python
def mkdir(self, path: str) -> None:
    self.root.insert(path, False)
```
- Simply inserts the path with `isFile=False`
- The insert method handles creating intermediate directories automatically

**3. addContentToFile Operation**
```python
def addContentToFile(self, filePath: str, content: str) -> None:
    node = self.root.insert(filePath, True)
    node.content.append(content)
```
- Inserts the path as a file (`isFile=True`)
- Appends content to the node's content list
- Works for both new files and existing files

**4. readContentFromFile Operation**
```python
def readContentFromFile(self, filePath: str) -> str:
    node = self.root.search(filePath)
    return ''.join(node.content)
```
- Finds the file node
- Joins all content pieces into a single string

### Time Complexity Analysis

- `ls`: O(p + n log n) where p is path length and n is number of children (due to [sorting](/problems/sorting_summary))
- `mkdir`: O(p) where p is path length
- `addContentToFile`: O(p) for path traversal
- `readContentFromFile`: O(p + c) where c is total content length

### Space Complexity

O(total number of nodes + total content size) for storing the entire file system structure and file contents.

## Example Walkthrough

Let's walk through a sequence of operations to see how the file system works:

**Initial State**: Empty file system with just a root node `/`

**Step 1: `mkdir("/a/b/c")`**
- Path splits into: `['', 'a', 'b', 'c']`
- Starting from root, we traverse/create:
  - Root → create child 'a' → create child 'b' → create child 'c'
- Each node is marked as directory (`isFile=False`)

Tree structure:
```
/
└── a/
    └── b/
        └── c/
```

**Step 2: `addContentToFile("/a/b/file1.txt", "Hello")`**
- Path splits into: `['', 'a', 'b', 'file1.txt']`
- Traverse: Root → 'a' → 'b' → create 'file1.txt'
- Mark 'file1.txt' as file (`isFile=True`)
- Set `name = "file1.txt"` and append "Hello" to content list

Tree structure:
```
/
└── a/
    └── b/
        ├── c/
        └── file1.txt (content: ["Hello"])
```

**Step 3: `ls("/a/b")`**
- Search for node at path `/a/b`
- Node exists and `isFile=False`, so return sorted children keys
- Children are: `{'c': Trie(), 'file1.txt': Trie()}`
- Return: `["c", "file1.txt"]` (already in alphabetical order)

**Step 4: `addContentToFile("/a/b/file1.txt", " World")`**
- Path already exists as a file
- Append " World" to existing content list
- Content becomes: `["Hello", " World"]`

**Step 5: `readContentFromFile("/a/b/file1.txt")`**
- Search for node at path `/a/b/file1.txt`
- Join content list: `"Hello" + " World" = "Hello World"`
- Return: `"Hello World"`

**Step 6: `ls("/a/b/file1.txt")`**
- Search finds the node
- Node has `isFile=True`, so return just the filename
- Return: `["file1.txt"]`

This example demonstrates:
- How directories are created recursively
- File content is stored as a list for efficient appending
- The `ls` command behaves differently for files vs directories
- The Trie structure efficiently shares common path prefixes (here `/a/b` is shared between the directory `c` and `file1.txt`)

## Python Solution

```python
from typing import List, Optional


class TrieNode:
    """
    Trie node representing a file or directory in the file system.
    """

    def __init__(self):
        # Name of the file (only used for files, not directories)
        self.name: Optional[str] = None
        # Flag to indicate if this node represents a file
        self.is_file: bool = False
        # Content of the file (list of strings that will be concatenated)
        self.content: List[str] = []
        # Children nodes (subdirectories or files)
        self.children: dict[str, 'TrieNode'] = {}

    def insert(self, path: str, is_file: bool) -> 'TrieNode':
        """
        Insert a new file or directory into the trie.

        Args:
            path: The full path to insert (e.g., "/a/b/c")
            is_file: True if inserting a file, False for directory

        Returns:
            The node representing the inserted path
        """
        node = self
        # Split path by '/' and skip the empty string from leading '/'
        path_components = path.split('/')

        # Traverse/create the path in the trie
        for component in path_components[1:]:
            if component not in node.children:
                node.children[component] = TrieNode()
            node = node.children[component]

        # Mark as file if needed and store the file name
        node.is_file = is_file
        if is_file:
            node.name = path_components[-1]

        return node

    def search(self, path: str) -> Optional['TrieNode']:
        """
        Search for a node at the given path.

        Args:
            path: The full path to search for

        Returns:
            The node at the path, or None if not found
        """
        node = self

        # Handle root directory case
        if path == '/':
            return node

        # Split path and traverse the trie
        path_components = path.split('/')
        for component in path_components[1:]:
            if component not in node.children:
                return None
            node = node.children[component]

        return node


class FileSystem:
    """
    In-memory file system implementation using a Trie data structure.
    """

    def __init__(self):
        # Initialize root directory
        self.root = TrieNode()

    def ls(self, path: str) -> List[str]:
        """
        List the contents of a directory or return file name if it's a file.

        Args:
            path: The path to list

        Returns:
            List of file/directory names in sorted order, or single file name
        """
        node = self.root.search(path)

        # Path doesn't exist
        if node is None:
            return []

        # If it's a file, return just the file name
        if node.is_file:
            return [node.name]

        # If it's a directory, return sorted list of children
        return sorted(node.children.keys())

    def mkdir(self, path: str) -> None:
        """
        Create a new directory path (including intermediate directories).

        Args:
            path: The directory path to create
        """
        self.root.insert(path, is_file=False)

    def addContentToFile(self, filePath: str, content: str) -> None:
        """
        Add content to a file. Creates the file if it doesn't exist.

        Args:
            filePath: The path to the file
            content: The content to append to the file
        """
        node = self.root.insert(filePath, is_file=True)
        node.content.append(content)

    def readContentFromFile(self, filePath: str) -> str:
        """
        Read all content from a file.

        Args:
            filePath: The path to the file to read

        Returns:
            The concatenated content of the file
        """
        node = self.root.search(filePath)
        return ''.join(node.content)


# Your FileSystem object will be instantiated and called as such:
# obj = FileSystem()
# param_1 = obj.ls(path)
# obj.mkdir(path)
# obj.addContentToFile(filePath, content)
# param_4 = obj.readContentFromFile(filePath)
```

## Time and Space Complexity

### Time Complexity

**`ls(path)`**: `O(m + n*log(n))` where `m` is the length of the path (number of directories in the path) and `n` is the number of files/directories in the target directory.
- Searching the path takes `O(m)` time to traverse through the trie
- If it's a file, returning the name is `O(1)`
- If it's a directory, sorting the children keys takes `O(n*log(n))`

**`mkdir(path)`**: `O(m)` where `m` is the length of the path (number of directories in the path).
- The insert operation traverses through each component of the path, creating nodes as needed
- Splitting the path takes `O(p)` where `p` is the string length of path
- Traversing and creating nodes takes `O(m)` where `m` is the number of path components

**`addContentToFile(filePath, content)`**: `O(m)` where `m` is the length of the file path (number of directories in the path).
- The insert operation takes `O(m)` to traverse/create the path
- Appending content to the list is `O(1)`

**`readContentFromFile(filePath)`**: `O(m + c)` where `m` is the length of the file path and `c` is the total length of content stored in the file.
- Searching the path takes `O(m)`
- Joining all content strings takes `O(c)` where `c` is the total character count

### Space Complexity

**Overall Space**: `O(n*k + c)` where:
- `n` is the total number of nodes (files and directories) in the trie
- `k` is the average length of file/directory names
- `c` is the total content stored across all files

**Per Operation Space**:
- **`ls(path)`**: `O(n)` for storing and returning the sorted list of children
- **`mkdir(path)`**: `O(m*k)` for creating new trie nodes along the path, where `k` is the average name length
- **`addContentToFile(filePath, content)`**: `O(m*k)` for potentially creating new nodes, plus the content is stored permanently
- **`readContentFromFile(filePath)`**: `O(c)` for creating the joined string of all content pieces

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Path Splitting Leading to Empty String Issues**

One of the most common pitfalls is mishandling the path splitting, particularly with the empty string that results from splitting a path that starts with `/`.

**The Problem:**
When you split a path like `/a/b/c` by `/`, you get `['', 'a', 'b', 'c']`. The first element is an empty string. If you don't skip it properly, you might:
- Create an unwanted empty-named node in the trie
- Cause index out of bounds errors
- Return incorrect results for root directory operations

**Incorrect Implementation:**
```python
def insert(self, path, is_file):
    node = self
    ps = path.split('/')
    for p in ps:  # Wrong! This includes the empty string
        if p not in node.children:
            node.children[p] = TrieNode()
        node = node.children[p]
    # ...
```

**Correct Implementation:**
```python
def insert(self, path, is_file):
    node = self
    ps = path.split('/')
    for p in ps[1:]:  # Skip the first empty string
        if p not in node.children:
            node.children[p] = TrieNode()
        node = node.children[p]
    # ...
```

### 2. **Not Handling Root Directory as a Special Case**

The root directory `/` needs special handling in the search method.

**The Problem:**
When searching for `/`, splitting it results in `['', '']`. Skipping the first element still leaves one empty string, so the loop looks for a child named `''`, finds none, and returns `None`. Then `ls("/")` returns an empty list instead of the root's contents.

**Incorrect Implementation:**
```python
def search(self, path):
    node = self
    ps = path.split('/')
    for p in ps[1:]:  # For "/", ps[1:] is [''], and '' is not a child
        if p not in node.children:
            return None
        node = node.children[p]
    return node
```

**Correct Implementation:**
```python
def search(self, path):
    node = self
    if path == '/':  # Handle root specially
        return node
    ps = path.split('/')
    for p in ps[1:]:
        if p not in node.children:
            return None
        node = node.children[p]
    return node
```

### 3. **Overwriting Existing File Content Instead of Appending**

The problem specifically states that `addContentToFile` should append content if the file exists, not replace it.

**The Problem:**
Using assignment instead of append would overwrite existing content.

**Incorrect Implementation:**
```python
def addContentToFile(self, filePath, content):
    node = self.root.insert(filePath, True)
    node.content = [content]  # Wrong! This overwrites existing content
```

**Correct Implementation:**
```python
def addContentToFile(self, filePath, content):
    node = self.root.insert(filePath, True)
    node.content.append(content)  # Correctly appends to existing content
```

### 4. **Not Preserving File State When Re-inserting**

When `addContentToFile` is called on an existing file, the insert method shouldn't reset the file's properties.

**The Problem:**
If insert always resets the content list or other properties, you'll lose existing data.

**Better Implementation with State Preservation:**
```python
def insert(self, path, is_file):
    node = self
    ps = path.split('/')
    for p in ps[1:]:
        if p not in node.children:
            node.children[p] = TrieNode()
        node = node.children[p]

    # Only set is_file if creating new node or converting directory to file
    if not node.is_file:  # Preserve existing file state
        node.is_file = is_file
        if is_file:
            node.name = ps[-1]

    return node
```

### 5. **Forgetting to Sort Directory Listings**

The problem requires directory contents to be returned in lexicographic order.

**Incorrect Implementation:**
```python
def ls(self, path):
    node = self.root.search(path)
    if node is None:
        return []
    if node.is_file:
        return [node.name]
    return list(node.children.keys())  # Wrong! Not sorted
```

**Correct Implementation:**
```python
def ls(self, path):
    node = self.root.search(path)
    if node is None:
        return []
    if node.is_file:
        return [node.name]
    return sorted(node.children.keys())  # Correctly sorted
```

### 6. **Memory Inefficiency with Content Storage**

Storing content as a list of strings and joining them every read can be inefficient for frequently read files.

**Alternative Approach for Optimization:**
```python
class TrieNode:
    def __init__(self):
        self.content_builder = []  # For building content
        self.cached_content = None  # Cache the joined content

    def add_content(self, content):
        self.content_builder.append(content)
        self.cached_content = None  # Invalidate cache

    def get_content(self):
        if self.cached_content is None:
            self.cached_content = ''.join(self.content_builder)
        return self.cached_content
```

These pitfalls are subtle but critical for correct implementation. The key is careful handling of edge cases, particularly around path parsing and root directory operations.

