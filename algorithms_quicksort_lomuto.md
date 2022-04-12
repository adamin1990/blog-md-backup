# QuickSort using Lomuto partition scheme
# 使用lomuto方案的快速排序 
------

 QuickSort is another algorithm form Divide and Conquer category.It operates by breaking down large problems into smaller,more easily solvable problems.

 快速排序是分治算法分类中的另一种算法，它通过把大问题拆分成更小，更容易解决的问题操作。

 The idea of QuickSort is the following: a pivot element is picked.The versions of QuickSort differ in the way of pivot picking.First,last,median,or a randomly selected element is a candidate to be picked, as the pivot.
 快速排序的思想如下：选择一个元素作为指点，快速排序的版本因选择支点pivot的方式不同尔不同，第一个，最后一个，中间位置，或者随机选择，都可以成为选择支点的候选方式。  

The primary part of the sort process is partitioning.Once the pivot is
picked,the array is partitioned into two halves-one half containing all
 the elements less that the pivot and the other half containing the elements
 greater than the pivot.The equal ones can remain either side.Then the
 same process occurs to the remaining halves of the array recursively,
 eventually resulting in a sorted array.  

 排序进程的基本是分割。当支点选择了，数组将被分割成两部分，一部分包含了所有小于支点的元素，另一部分包含了大于支点的元素，与支点相等的元素可以在任意一侧。然后同样的进程在剩下的部分递归
 调用，最终产生的是一个有序数组。

For example:
  the array is [2,0,7,4,3]
 * choose 3(last element) as the pivot.
 * 选择最后一个元素3作为支点
 * after doing 4 comparisons we get the two halves:
 * 进行四次比较后，得到了两部分：
 * [2,0](3)[7,4]
 * Now the same process repeats for the two halves.
 * 然后对该两部分进行同样的流程
 * We choose 0 as a pivot for the lesser half,and 4 for the greater half.
 * 我们选择0作为左侧部分的支点，4作为右侧的支点
 * after a comparison for each half,we get [(0)[2]](3)[(4)[7]]
 * witch is a sorted sequence.
 * 对比后得到一个有序序列。  

 Advantages and Disadvantages  
 优缺点
 * QuickSort operates in-place,so it doesn't require extra storage.
 * 快速排序原地操作，不需要额外存储
 * The choice of pivot makes a big difference,as an unsuccessful pivot
 selection can decreases the algorithm's speed significantly.
 * 支点的选择大有不同，因为不成功的支点选择可以明显的降低算法的速度。
 * A variation of QuickSort is the 3-Way QuickSort,it divide the
  sequence into three pieces:smaller ,greater,and equal.This makes it more
 * convenient for data with high redundancy(repetitions).
 * 快速排序的一个变种叫3路快速排序，它把序列分成了三部分，比支点小的，比支点大的，与支点
 * 相等的。这使得有大量重复数据时很方便。
 * The randomized version of QuickSort is the most efficient.It choose
 * the pivot randomly,thus avoiding worst cases for particular patterns  

## Java Implementation

```java
 public void sortElement(int[] elements) {
         quickSortLomuto(elements,0,elements.length-1);

    }

    private void quickSortLomuto(int[] elements, int low, int high) {
        if(low<high) {
            int pivotIndex = partition(elements, low, high); //计算pivot的实际未知，并把low到high内的元素分布到pivot的两侧
            quickSortLomuto(elements, low, pivotIndex - 1); //before pivot
            quickSortLomuto(elements, pivotIndex + 1, high);  //after pivot
        }
    }

    /**
     * lomuto方式选取high，即最后一个作为pivot
     * @param elements
     * @param low
     * @param high
     * @return
     */
    private int partition(int[] elements, int low, int high) {
        int left=low-1; //左侧索引，当下面循环内元素小于pivot时候，右移left一次，然后调换元素
        int pivot=elements[high];
        for(int j=low;j<high;j++ ){
            if(elements[j]<pivot){
                left++;
                swap(elements,left,j); //把j值放到left索引位置
            }

        }
        swap(elements,left+1,high); //把pivot放到正确的位置
        return left+1;
    }

    private void swap(int[] elements, int left, int j) {
        int temp=elements[left];
        elements[left]=elements[j];
        elements[j]=temp;
    }
```  
