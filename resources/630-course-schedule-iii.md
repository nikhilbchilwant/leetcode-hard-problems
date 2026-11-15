# 630. Course Schedule III

> Source: https://algo.monster/liteproblems/630

## Problem Statement

In this problem, you are given `n` online courses, each with a specified duration and a deadline by which it should be completed. The courses must be taken one at a time, consecutively, from day 1 onwards. The goal is to find out the most number of courses you can finish before their respective deadlines.

Each course is described by two parameters: `duration` (how many days it takes to complete the course) and `lastDay` (the last day by which the course must be completed). You are provided this information in an array `courses`, where each element `courses[i]` is another array with two elements: `[duration_i, lastDay_i]`.

Your task is to pick the courses in such a way that you can complete the maximum number of them within their deadlines, keeping in mind that you cannot take more than one course at a time.

## Intuition

The solution to this problem involves [sorting](/problems/sorting_summary) and priority queues, specifically a min-heap. The first step is to sort the courses by their deadline (`lastDay`). This initial sort helps to align our course selections in the direction that we attempt to tackle courses with earlier deadlines first.

After [sorting](/problems/sorting_summary) the courses by their deadlines, we iterate through each course. As we go through the courses, we add each course's duration to a total duration accumulator (`s`) and push the negative of duration onto a min-heap (`pq`). We use a min-heap to keep track of the courses with the longest duration that we have attended to, and the reason for storing negative durations is to effectively turn the min-heap into a max-heap (since Python's `heapq` only provides a min-heap).

If at any point our total duration accumulator (`s`) exceeds the current course's last day, it means we cannot finish the current course before the deadline. To rectify this, we start removing the longest courses we have taken from our schedule (the top of the heap), until `s` no longer exceeds the last day of the current course.

By iteratively adding courses and potentially removing the longest course from our schedule whenever necessary, we ensure that we are always in a position to have completed the maximum number of courses by their respective deadlines.

The solution's efficiency comes from the fact that when we must drop a course, we drop the one with the longest duration, which has the most significant potential to free up our schedule for fitting in more courses.

The final answer is the number of courses we have in our priority queue, as this represents the maximum number of courses we can complete within their deadlines.

**Learn more about [Greedy](/problems/greedy_intro), [Sorting](/problems/sorting_summary) and [Heap (Priority Queue)](/problems/heap_intro) patterns.**

## Solution Approach

The solution takes a [greedy](/problems/greedy_intro) approach along with the use of a priority queue to maximize the number of courses taken. Here, we go through the algorithm and patterns used in the solution:

1. **[Sorting](/problems/sorting_summary)**: Initially, we sort the `courses` array by their deadlines, which is `lastDay_i` for the `i-th` course. This is done to prioritize courses with the earliest deadlines, allowing us to take courses in a sequence that respects those deadlines.

    ```python
    courses.sort(key=lambda x: x[1])
    ```

2. **Priority Queue (Min-Heap)**: We use a priority queue (min-heap) to keep track of the courses we have decided to take. Since Python's `heapq` library only supports a min-heap and we want to have the functionality of a max-heap (to easily pop the longest course when needed), we store negative durations in the heap.

    The priority queue (heap) operations used are:
    - `heappush` to add an element to the heap
    - `heappop` to remove the smallest element, which in our case will be the course with the longest duration due to the negation

    ```python
    pq = []
    ```

3. **Iterating Through the Courses**: We iterate through the sorted `courses` list, and for each course, we perform the following steps:

    a. First, we push the negative duration of the current course onto the heap and add the duration to an accumulator `s`.

        ```python
        heappush(pq, -duration)
        s += duration
        ```

    b. Then, we check if adding the current course has caused the total duration (`s`) to exceed the last day by which the course must be completed. If it does, we enter a while loop:

        - Inside the loop, we pop the course with the longest duration (which is at the top of the heap due to being the smallest negative number), and subtract from our total duration `s`. 

        - We continue this process until the total duration `s` no longer exceeds the deadline of the current course or until the heap is empty.

        ```python
        while s > last:
            s += heappop(pq)
        ```

4. **Count of Courses Taken**: After iterating through all courses and adjusting our schedule as necessary, the remaining courses in the heap represent the courses that we can complete within their respective deadlines. Hence, the length of the heap gives us the maximum number of courses that can be taken.

    ```python
    return len(pq)
    ```

By using the [greedy](/problems/greedy_intro) strategy of always picking the next course that can be finished before its deadline after the courses already scheduled, and by adjusting the schedule when necessary by dropping the longest course, the algorithm ensures we optimize for the maximum number of courses taken.

## Example Walkthrough

To illustrate the solution approach, let's use a small example of the courses array:

```
courses = [[100, 200], [70, 150], [120, 210]]
```

1. **Sorting by Deadlines**

   First, we sort the courses by their deadlines:

   - `courses.sort(key=lambda x: x[1])`
 
   After sorting:

   ```
   courses = [[70, 150], [100, 200], [120, 210]]
   ```

   Now, our courses array is sorted by the last day on which the courses need to be completed, enabling us to focus on the courses with the earliest deadlines first.

2. **Initializing the Priority Queue**

   We create a priority queue (min-heap) to help us keep track of the courses we're taking.

   - `pq = []`
 
   Initially, the priority queue is empty.

3. **Iterating Through the Courses**

   We go through each course and follow a series of steps.
 
   a. For the first course `[70, 150]`:
   - We add the negative course duration to the heap and update the accumulator, `s`:
     ```
     heappush(pq, -70)
     s = 70
     ```
 
   b. The total duration `s` does not exceed the last day of the current course (150), so we move to the next course.
 
   a. For the second course `[100, 200]`:
   - We add the negative course duration to the heap and update the accumulator, `s`:
     ```
     heappush(pq, -100)
     s = 170
     ```
 
   b. The total duration `s` does not exceed the last day of the current course (200), so we move to the next course.
 
   a. For the third course `[120, 210]`:
   - We add the negative course duration to the heap and update the accumulator, `s`:
     ```
     heappush(pq, -120)
     s = 290
     ```
 
   b. Now the total duration `s` exceeds the last day of the current course (210). We need to adjust our schedule:
 
   - We remove the longest course from the heap (the most negative value, `-120`) and subtract its duration:
     ```
     s += heappop(pq)    # Removes -120, giving s = 290 - 120 = 170
     ```

   - Now `s = 170` does not exceed the deadline 210, so the adjustment stops. The heap holds `[-100, -70]`.

4. **Count of Courses Taken**:

   After iterating through all the courses and adjusting the schedule, the number of courses left in the queue is the number we can complete within their respective deadlines. So the final answer is:

   - `return len(pq)`

   The heap contains the courses with durations 70 and 100, so we get the final result:

   ```
   Maximum number of courses = 2
   ```

   Taking the 70-day course (finishing on day 70) and then the 100-day course (finishing on day 170) meets both deadlines. All three courses together need 290 days, which is past every deadline.

This example highlights the greedy decision-making process: by sorting the courses by deadlines and using a priority queue to manage durations, dropping the longest course if the total duration exceeds a course's deadline, we optimize for the maximum number of courses that can be completed.

## Python Solution

```python
from heapq import heappush, heappop

class Solution:
    def scheduleCourse(self, courses: List[List[int]]) -> int:
        # Sort the courses based on their end day
        courses.sort(key=lambda x: x[1])
      
        # Initialize a max heap to keep track of the longest courses we have taken
        max_heap = []
      
        # This variable will hold the total duration of all courses we have taken so far
        total_duration = 0
      
        # Iterate over each course
        for duration, last_day in courses:
            # Add the current course to the max heap and increment the total duration
            heappush(max_heap, -duration)
            total_duration += duration
          
            # If the total duration exceeds the last day of the current course,
            # remove the longest course from our schedule
            while total_duration > last_day:
                # heappop returns a negative number because we are using max heap,
                # so we add it back to reduce the total duration
                total_duration += heappop(max_heap)
      
        # The number of courses in the max heap is the number of courses we can take
        return len(max_heap)
```

## Time and Space Complexity

The time complexity of the above code is `O(n log n)` for sorting the courses, where `n` is the number of courses. The insertion and deletion operations in the priority queue take `O(log k)`, where `k` is the number of courses currently in the queue. Since each course can be pushed onto and popped from the queue once, in the worst case the total cost for these operations is `O(n log k)`. Assuming the queue could have at most `n` courses, the total time complexity of the algorithm becomes `O(n log n)` due to the initial sort dominating the overall complexity.

The space complexity of the code is `O(n)` which is due to the priority queue storing at most `n` courses at any time.

**Learn more about [how to find time and space complexity quickly using problem constraints](/problems/runtime_summary).**

