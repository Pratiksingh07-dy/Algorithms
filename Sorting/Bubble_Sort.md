Bubble Sort
📌 Introduction

Bubble Sort is one of the simplest comparison-based sorting algorithms. It repeatedly compares adjacent elements in an array and swaps them if they are in the wrong order.

The algorithm continues making passes through the array until no more swaps are required, indicating that the array is sorted.

The name Bubble Sort comes from the way larger elements "bubble up" to the end of the array after each pass.

🎯 Problem Statement

Given an unsorted array, arrange the elements in ascending order by repeatedly swapping adjacent elements when they are in the wrong order.

Example

Input:

[5, 3, 8, 4, 2]

Output:

[2, 3, 4, 5, 8]
⚙️ How Bubble Sort Works

Consider the array:

[5, 3, 8, 4, 2]
Pass 1

Compare 5 and 3

[3, 5, 8, 4, 2]

Compare 5 and 8

[3, 5, 8, 4, 2]

Compare 8 and 4

[3, 5, 4, 8, 2]

Compare 8 and 2

[3, 5, 4, 2, 8]

Largest element (8) reaches its correct position.

Pass 2

Compare 3 and 5

[3, 5, 4, 2, 8]

Compare 5 and 4

[3, 4, 5, 2, 8]

Compare 5 and 2

[3, 4, 2, 5, 8]
Pass 3

Compare 3 and 4

[3, 4, 2, 5, 8]

Compare 4 and 2

[3, 2, 4, 5, 8]
Pass 4

Compare 3 and 2

[2, 3, 4, 5, 8]

Array is now sorted.

🔍 Algorithm
Traverse the array from left to right.
Compare adjacent elements.
Swap if the left element is greater than the right element.
After each pass, the largest unsorted element reaches its correct position.
Repeat until the array becomes sorted.

📝 Pseudocode
for i = 0 to n-1
    for j = 0 to n-i-2
        if arr[j] > arr[j+1]
            swap(arr[j], arr[j+1])

💻 Python Implementation
def bubble_sort(arr):
    n = len(arr)

    for i in range(n):
        swapped = False

        for j in range(0, n - i - 1):

            if arr[j] > arr[j + 1]:
                arr[j], arr[j + 1] = arr[j + 1], arr[j]
                swapped = True

        if not swapped:
            break

    return arr


arr = [5, 3, 8, 4, 2]

print("Original Array:", arr)
print("Sorted Array:", bubble_sort(arr))


📊 Time Complexity Analysis
Case	     Time Complexity
Best Case	     O(n)
Average Case     O(n²)
Worst Case	     O(n²)

Explanation

Best Case
Occurs when the array is already sorted. With the optimization using the swapped flag, the algorithm stops after one pass.

Average Case
The algorithm performs multiple comparisons and swaps.

Worst Case
Occurs when the array is sorted in reverse order.

💾 Space Complexity
Metric	Complexity
Auxiliary Space	O(1)

Bubble Sort is an in-place sorting algorithm because it does not require additional memory proportional to the input size.

⭐ Characteristics
Property	Value
Stable	Yes
In-Place	Yes
Adaptive	Yes (Optimized Version)
Comparison Based	Yes
Recursive	No

✅ Advantages
Easy to understand and implement.
Requires no extra memory.
Stable sorting algorithm.
Useful for educational purposes.
Can detect already sorted arrays efficiently using optimization.

❌ Disadvantages
Poor performance on large datasets.
Large number of comparisons.
Not suitable for production-level applications.
Much slower than Merge Sort, Quick Sort, and Heap Sort.

🌍 Real-World Use Cases
Bubble Sort is rarely used in production systems, but it is valuable for:
Learning sorting concepts.
Understanding algorithm optimization.
Teaching time complexity analysis.
Small datasets where simplicity is preferred.


🔥 Interview Questions
1. Is Bubble Sort stable?
Yes. Equal elements maintain their relative order after sorting.

2. Is Bubble Sort an in-place algorithm?
Yes. It uses constant extra memory O(1).

3. Why is Bubble Sort called Bubble Sort?
Because larger elements gradually move toward the end of the array, similar to bubbles rising to the surface.

4. Can Bubble Sort achieve O(n) complexity?
Yes. When the optimized version uses a swapped flag and the array is already sorted.

5. Is Bubble Sort used in industry?
Rarely. More efficient algorithms such as Quick Sort, Merge Sort, and Tim Sort are preferred.

📚 Key Takeaways
Bubble Sort repeatedly swaps adjacent elements.
Largest element reaches its correct position after each pass.
Best Case: O(n)
Average Case: O(n²)
Worst Case: O(n²)
Space Complexity: O(1)
Stable and In-Place sorting algorithm.
Primarily used for learning and understanding sorting fundamentals.