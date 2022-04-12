# Merge Sort
# 归并排序 
----
## Average Time Complexity  
 **O(n㏒n)**  
Merge Sort belongs to the category of Divide and Conquer algorithms.
These algorithms operate by breaking down large problems into smaller,
more easily solvable problems.  
归并排序属于分治算法分类，这些算法通过把大的问题拆分成更小的，更容易解决的问题操作。  
The idea of the merge sort algorithm is to divide the array in half
over and over again util each piece is only one item long.Then those
items are put back together(merged) in sort-order.  
归并排序算法的思想是把数组元素一次次的对半拆分，直到每一块只有一个元素。然后再归并这些元素成有序的。  
for example:
 * the array is [31,4,55,1,4,2,42]
 * step 1: [31,4,55,1] ,[4,2,42] divided into 2 parts
 * step 2: [31,4][55,1][4,2][42] divided into 4 parts
 * step 3:[31] [4] [55] [1] [4] [2] [42] divided into single item
 * then we need to merge them back into sort-order:
 * step5: [4,31],[1,55],[2,4],[42] merge into sort-order
 * step6: [1,4,31,55] [2,4,42] merge again
 * step7: [1,2,3,31,42,55]  
 
merge sort is useful for sorting linked lists,as the merge operations
can be implemented without extra space for linked list.
for arrays,the algorithm needs extra temporary storage space for each
half during each iteration.  
归并排序在linked list的排序中非常有用，因为合并操作可以不分配额外存储空间。
对于数组来说，该算法需要在每一部分迭代中单独分配临时的存储空间。  

## Java Implementation
```java
public class MergeSort implements Isort {
    @Override
    public void sortElement(int[] elements) {

        internalSort(elements, 0, elements.length - 1);


    }

    /**
     * 递归调用，先调用内层，再调用外层
     *
     * @param elements
     * @param left
     * @param right
     */
    private void internalSort(int[] elements, int left, int right) {

        if (left < right) { // break into single item
            int middle = (left + right) / 2; //middle index
            internalSort(elements, left, middle);
            internalSort(elements, middle + 1, right);
            merge(elements, left, middle, right);

        }
    }

    /**
     * merge two sorted sub array
     * 合并两个已经排序的数组
     *
     * @param elements
     * @param left
     * @param middle
     * @param right
     */
    private void merge(int[] elements, int left, int middle, int right) {
        int leftSize = middle - left + 1;
        int rightSize = right - middle;
        int[] leftArr = new int[leftSize];
        int[] rightArr = new int[rightSize];
        for (int i = 0; i < leftSize; i++) {  //left array
            leftArr[i] = elements[left + i];
        }
        for (int j = 0; j < rightSize; j++) {  //right array
            rightArr[j] = elements[middle + j + 1];
        }
        int i = 0, j = 0; // i is left array index and j is right array index
        int k = left;    // k is whole array index
        while (i < leftSize && j < rightSize) {
            if (leftArr[i] <= rightArr[j]) {
                elements[k] = leftArr[i];
                i++;
            } else {
                elements[k] = rightArr[j];
                j++;
            }
            k++;

        }
        while (i < leftSize) { // if left array have some items not set to elements
            elements[k] = leftArr[i];
            i++;
            k++;
        }
        while (j < rightSize) {   // if right array have some items not set to element,set it iteratively
            elements[k] = rightArr[j];
            j++;
            k++;
        }


    }

    public static void main(String[] args) {
        int[] test = new int[]{31, 4, 55, 1, 4, 2, 42};
        int[] test2 = new int[]{31, 4, 55, 1};
        /**
         *         step 1 31 4  55 1
         *         step 2  31 4 55 1
         */

        new MergeSort().sortElement(test);
        System.out.println("sorted array is  = " + Arrays.toString(test));
    }

}
```
