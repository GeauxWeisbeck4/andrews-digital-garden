---
id: 01JADMF5154GVVZ8VTDNB668RH
title: Insertion Sort
tags:
  - sorting
  - algorithms
  - dsa
  - roadmaps
  - programming
  - insertion-sort
modified: 2024-10-22T14:28:44-04:00
---
# Insertion Sort

Insertion sort is a simple sorting algorithm that builds the final sorted array (or list) one item at a time. It’s much less efficient on large lists than more advanced algorithms like quicksort, heapsort, or merge sort. Still, it provides several advantages such as it’s easy to understand the algorithm, it performs well with small lists or lists that are already partially sorted and it can sort the list as it receives it. The algorithm iterates, consuming one input element each repetition and growing a sorted output list. At each iteration, it removes one element from the input data, finds the location it belongs within the sorted list and inserts it there. It repeats until no input elements remain.

Free Resources

---

- [articleInsertion Sort - W3Schools](https://www.w3schools.com/dsa/dsa_algo_insertionsort.php)
- [articleInsertion Sort Visualization](https://www.hackerearth.com/practice/algorithms/sorting/insertion-sort/visualize/)

## Tutorial
Insertion sort is based on the idea that one element from the input elements is consumed in each iteration to find its correct position i.e, the position to which it belongs in a sorted array.

It iterates the input elements by growing the sorted array at each iteration. It compares the current element with the largest value in the sorted array. If the current element is greater, then it leaves the element in its place and moves on to the next element else it finds its correct position in the sorted array and moves it to that position. This is done by shifting all the elements, which are larger than the current element, in the sorted array to one position ahead

**Implementation**

```
void insertion_sort ( int A[ ] , int n) 
{
     for( int i = 0 ;i < n ; i++ ) {
    /*storing current element whose left side is checked for its 
             correct position .*/

      int temp = A[ i ];    
      int j = i;

       /* check whether the adjacent element in left side is greater or
            less than the current element. */

          while(  j > 0  && temp < A[ j -1]) {

           // moving the left side element to one position forward.
                A[ j ] = A[ j-1];   
                j= j - 1;

           }
         // moving current element to its  correct position.
           A[ j ] = temp;       
     }  
}
```

Take array A[]=[7,4,5,2].

![enter image description here](https://he-s3.s3.amazonaws.com/media/uploads/46bfac9.png)

Since 7 is the first element has no other element to be compared with, it remains at its position. Now when on moving towards 4, 7 is the largest element in the sorted list and greater than 4. So, move 4 to its correct position i.e. before 7. Similarly with 5, as 7 (largest element in the sorted list) is greater than 5, we will move 5 to its correct position. Finally for 2, all the elements on the left side of 2 (sorted list) are moved one position forward as all are greater than 2 and then 2 is placed in the first position. Finally, the given array will result in a sorted array.

**Time Complexity:**

In worst case,each element is compared with all the other elements in the sorted array. For N elements, there will be N2 comparisons. Therefore, the time complexity is ON ^2

## Practice

You have been given an _A_ array consisting of _N_ integers. All the elements in this array are guaranteed to be unique. For each position _i_ in the array _A_ you need to find the position A[i] should be present in, if the array was a sorted array. You need to find this for each _i_ and print the resulting solution.

**Input Format**:

The first line contains a single integer _N_ denoting the size of array _A_. The next line contains _N_ space separated integers denoting the elements of array _A_.

**Output Format**:

Print _N_ space separated integers on a single line , where the _I_th integer denotes the position of A[i] if this array were sorted.

**Constraints**:

1≤N≤100

1≤A[i]≤100

SAMPLE INPUT

[](https://he-s3.s3.amazonaws.com/media/hackathon/question-for-new-practice-section/problems/b746c0e2-0-sample-input-b74632e.txt?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIA6I2ISGOYMPJGUFGY%2F20241022%2Fap-southeast-1%2Fs3%2Faws4_request&X-Amz-Date=20241022T182714Z&X-Amz-Expires=3600&X-Amz-SignedHeaders=host&X-Amz-Signature=68a175f54034e8bb0d07494cdb2189f1baac78a4d90ace5bd5da3de2a17076a3) 

5
1 2 3 4 5

SAMPLE OUTPUT

[](https://he-s3.s3.amazonaws.com/media/hackathon/question-for-new-practice-section/problems/bc8a3d54-0-sample-output-bc8997f.txt?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIA6I2ISGOYMPJGUFGY%2F20241022%2Fap-southeast-1%2Fs3%2Faws4_request&X-Amz-Date=20241022T182714Z&X-Amz-Expires=3600&X-Amz-SignedHeaders=host&X-Amz-Signature=d88f62f4d9938108f84f04b78d286616e8e49067d78aefde0e39cd72b9f12f39) 

1 2 3 4 5