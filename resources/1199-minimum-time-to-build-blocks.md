# 1199. Minimum Time to Build Blocks

> Source: https://algo.monster/liteproblems/1199

## Problem Statement

You have a list of blocks where `blocks[i] = t` means the i-th block requires `t` units of time to build. Each block must be built by exactly one worker.

Starting with just one worker, you have two options at any point:
1. **Split a worker**: A worker can split into two workers, which takes `split` units of time. If multiple workers split simultaneously, they do so in parallel (the time cost is still just `split`, not multiplied).
2. **Build a block**: A worker can build one block and then go home (that worker is no longer available).

Your goal is to find the minimum time needed to build all blocks.

**Example scenario**:
- If you have blocks `[1, 2]` and `split = 1`:
  - The single worker first splits (takes 1 time unit), creating 2 workers
  - Then both workers build their blocks in parallel
  - Total time = 1 (split) + max(1, 2) = 3

The challenge is determining the optimal strategy for when to split workers versus when to assign them to build blocks, minimizing the total time required to complete all blocks.

## Intuition

The key insight is to think about this problem in reverse. Instead of thinking forward about splitting workers, we can think backward about merging blocks.

Consider what happens when we have multiple blocks to build:
- With one block: No splitting needed, just build it directly
- With two blocks: We must split once to get 2 workers, then they build in parallel. Total time = `split + max(block1, block2)`
- With more blocks: The pattern becomes complex if we think forward

The reverse perspective reveals something elegant: When we split a worker to build two blocks, the total time is `split + max(block_i, block_j)`. This is equivalent to "merging" two blocks into one super-block that takes `split + max(block_i, block_j)` time to build.

Why does [greedy](/problems/greedy_intro) selection work here? The blocks with longer build times should participate in as few merges as possible because each merge adds a `split` cost. By always merging the two smallest blocks first:
1. We minimize the impact of the `split` overhead on larger blocks
2. Larger blocks get merged later (or not at all if they're the last one), reducing their participation in costly merge operations

Think of it like a tournament bracket in reverse - we want the "strongest" (longest) blocks to have the shortest path to the final, while the "weakest" (shortest) blocks can afford to go through multiple rounds of merging.

Using a min heap allows us to efficiently extract the two smallest blocks at each step, merge them into a new block with time `split + max(block_i, block_j)` (which simplifies to `split + block_j` since `block_j` is the larger one after popping from the heap), and continue until only one block remains.

**Learn more about [Greedy](/problems/greedy_intro), [Math](/problems/math-basics) and [Heap (Priority Queue)](/problems/heap_intro) patterns.**

## Solution Approach

The implementation uses a **[greedy](/problems/greedy_intro) algorithm with a min heap** to efficiently merge blocks from smallest to largest.

**Step-by-step implementation:**

1. **Initialize the min heap**: Convert the `blocks` list into a min heap using `heapify(blocks)`. This operation takes O(n) time and transforms the list in-place into a heap structure where the smallest element is always at the root.

2. **Merge blocks iteratively**: While there's more than one block in the heap:
   - Extract the smallest block using `heappop(blocks)` - this is discarded since it gets absorbed into the merge
   - Extract the second smallest block using `heappop(blocks)`
   - Create a new merged block with time `second_smallest + split`
   - Push this new block back into the heap using `heappush(blocks, second_smallest + split)`

3. **Return the final block**: After all merges, only one block remains in the heap - this represents the minimum total time needed.

**Why the algorithm works:**

- When we pop two blocks and only keep the larger one plus `split`, we're simulating the scenario where one worker splits and two workers build these blocks in parallel
- The time for parallel execution is determined by the slower (larger) block
- By always merging the smallest blocks first, we ensure larger blocks participate in fewer merges, minimizing the accumulation of `split` costs

**Time Complexity**: O(n log n) where n is the number of blocks
- Initial heapify: O(n)
- We perform n-1 merge operations
- Each merge involves 2 pops and 1 push, each taking O(log n)

**Space Complexity**: O(1) as we modify the input list in-place (the heap is built on the original list)

## Example Walkthrough

Let's walk through the solution with blocks = `[3, 1, 4, 2]` and split = `2`.

**Initial Setup:**
- Convert the array into a min heap: `[1, 2, 4, 3]` (heap representation)
- Visual: We start with 4 blocks that need to be built

**Iteration 1:**
- Pop smallest: 1 (discarded)
- Pop second smallest: 2
- New block time: 2 + 2 (split) = 4
- Push 4 back to heap
- Heap now: `[3, 4, 4]`
- *Interpretation: One worker splits (2 time), creating two workers who build blocks of size 1 and 2 in parallel. Total time for this group: 2 + max(1,2) = 4*

**Iteration 2:**
- Pop smallest: 3 (discarded)
- Pop second smallest: 4
- New block time: 4 + 2 (split) = 6
- Push 6 back to heap
- Heap now: `[4, 6]`
- *Interpretation: A worker handling the merged block (size 4) splits, and the two resulting workers build blocks of size 3 and 4 in parallel. Total time: 2 + max(3,4) = 6*

**Iteration 3:**
- Pop smallest: 4 (discarded)
- Pop second smallest: 6
- New block time: 6 + 2 (split) = 8
- Push 8 back to heap
- Heap now: `[8]`
- *Interpretation: The initial worker splits, with one branch eventually handling the block of size 4, and the other handling the complex merged block of size 6*

**Result:** The final block has time 8, which is the minimum time needed to build all blocks.

This represents the strategy where:
1. First split creates 2 workers (time: 2)
2. One worker builds block 4, the other splits again (time: 2)
3. After second split, one worker splits again while other builds block 3 (time: 2)
4. After third split, workers build blocks 1 and 2 in parallel (time: max(1,2) = 2)
5. Total: Following the critical path = 2 + 2 + 2 + 2 = 8

## Python Solution

```python
from typing import List
import heapq

class Solution:
    def minBuildTime(self, blocks: List[int], split: int) -> int:
        """
        Calculate minimum time to build all blocks with workers that can split.

        Strategy: Use a min-heap to greedily pair the smallest build times.
        When a worker splits, it creates a new worker, taking 'split' time.
        The optimal approach is to pair smallest blocks together under split operations.

        Args:
            blocks: List of integers representing time to build each block
            split: Time required for a worker to split into two workers

        Returns:
            Minimum time to build all blocks
        """
        # Convert blocks list into a min-heap (in-place operation)
        heapq.heapify(blocks)

        # Keep merging until only one time value remains
        while len(blocks) > 1:
            # Remove the smallest build time (we pair it with the second smallest)
            heapq.heappop(blocks)

            # Take the second smallest and add split time
            # This represents the total time when these two blocks are built in parallel
            # after one worker splits
            second_smallest = heapq.heappop(blocks)
            merged_time = second_smallest + split

            # Push the merged time back to heap
            heapq.heappush(blocks, merged_time)

        # The last remaining value is the minimum total time
        return blocks[0]
```

## Time and Space Complexity

The time complexity is `O(n × log n)`, where `n` is the number of blocks.

- The `heapify(blocks)` operation takes `O(n)` time to build a min-heap from the list.
- The while loop runs `n - 1` iterations (until only one element remains in the heap).
- In each iteration:
  - `heappop(blocks)` is called twice, each taking `O(log n)` time
  - `heappush(blocks, ...)` is called once, taking `O(log n)` time
  - Total per iteration: `O(log n)`
- Overall time complexity: `O(n) + O((n-1) × log n) = O(n × log n)`

The space complexity is `O(n)`, where `n` is the number of blocks.

- The heap is created in-place by modifying the input list `blocks`, so no additional space proportional to `n` is needed for a separate data structure.
- However, the heap operations themselves use `O(log n)` space for the call stack during the heapify and heap operations.
- Since the input list itself takes `O(n)` space and we're considering the total space used, the space complexity is `O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### Pitfall 1: Misunderstanding the Parallel Execution Model

**The Problem**: A common misconception is thinking that when workers split and build blocks in parallel, the total time should be `split + block1 + block2`. This leads to incorrect implementations that sum all block times.

**Why It's Wrong**: When multiple workers operate in parallel, they work simultaneously, not sequentially. The total time for parallel operations is determined by the slowest worker, not the sum of all work times.

**Incorrect Implementation Example**:
```python
# WRONG: This adds both block times
merged_time = first_smallest + second_smallest + split
```

**Correct Understanding**: When two workers build blocks in parallel after a split, the time is `split + max(block1, block2)`. Since we always pop the smallest first and discard it, keeping only the larger block time plus split achieves this.

### Pitfall 2: Not Handling Edge Cases Properly

**The Problem**: Failing to handle special cases like empty block lists or single blocks can cause runtime errors or incorrect results.

**Solution**: Add proper validation:
```python
def minBuildTime(self, blocks: List[int], split: int) -> int:
    # Handle edge cases
    if not blocks:
        return 0
    if len(blocks) == 1:
        return blocks[0]

    # Continue with main algorithm...
    heapq.heapify(blocks)
    # ... rest of the code
```

### Pitfall 3: Modifying the Input List Without Consideration

**The Problem**: The current implementation modifies the input `blocks` list in-place. If the caller needs the original list preserved, this causes unexpected side effects.

**Solution**: Create a copy if preserving the original is important:
```python
def minBuildTime(self, blocks: List[int], split: int) -> int:
    # Create a copy to avoid modifying the original
    blocks_heap = blocks.copy()  # or blocks[:]
    heapq.heapify(blocks_heap)

    while len(blocks_heap) > 1:
        heapq.heappop(blocks_heap)
        second_smallest = heapq.heappop(blocks_heap)
        heapq.heappush(blocks_heap, second_smallest + split)

    return blocks_heap[0]
```

### Pitfall 4: Incorrect Greedy Strategy

**The Problem**: Some might think pairing the smallest with the largest block is optimal (to "balance" the work), leading to implementations using both min and max values.

**Why It's Wrong**: The optimal strategy is to always merge the two smallest values. This ensures that larger block times (which take longer) participate in fewer merge operations, minimizing the accumulation of split costs.

**Visual Example**:
- Blocks: [1, 1, 3, 5], split = 1
- Correct (merge smallest): Total time = 8
- Wrong (merge min-max): Would result in suboptimal time > 8

