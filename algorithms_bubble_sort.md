# Bubble Sort
# 冒泡排序  
------
Bubble sort is an algorithm that is used to sort a list of elements,for example
elements in an array. This algorithm compares two adjacent elements and then swaps
them if they are not in order.
The process is repeated util no more swapping is needed.  
冒泡排序是对一系列元素排序的一种算法，比如数组中的元素。该算法对比相邻的元素，如果顺序不对则交换它们。
该过程一直重复，直到不需要交换为止  
The main advantage of Bubble Sort is the simplicity of the algorithm.Also,it does not
need any additional storage space,as it operation in-place.  
冒泡排序的主要优点是算法简单，并且不需要额外的存储空间。  
In terms of complexity, Bubble Sort is not optimal,as it requires multiple iterations
over the array.In the worst scenario,where all elements need to be swapped,it will
require (n-1)+(n-2)+(n-3)+....+3+2+1=n(n-1)/2 swaps(n is the number of elements).  
在复杂度上讲，冒泡排序并不是最优的，因为它需要在数组上进行多次迭代，最差的情况下，所有的元素需要交换，总共需要 (n-1)+(n-2)+(n-3)+....+3+2+1=n(n-1)/2次交换（n是数组元素的个数。  
Example:
 * if the array is [1,4,3,5]
 * step 1: [1,4,3,5] 1,4 is sorted and no need swap
 * step 2: [1,3,4,5] swap 4 and 3
 * step 3: [1,3,4,5] 4,5 is sorted and no need swap

## java实现 
## java implementation
``` java
public class BubbleSort implements Isort{


    @Override
    public void sortElement(int[] elements) {
        int length=elements.length;
        for (int i=0;i<length;i++){
            System.out.println("sort out index = " + i);
            boolean swaped=false;
            for(int j=0;j<length-i-1;j++){
                if(elements[j]>elements[j+1]){ //swap
                    int temp=elements[j];
                    elements[j]=elements[j+1];
                    elements[j+1]=temp;
                    swaped=true;
                }
            }
            if(!swaped){
                break;
            }

        }


    }

    public static void main(String[] args) {
        int[] test1=new int[]{1,4,3,5};
        int[] test2=new int[]{9,8,7,6,5};
        new BubbleSort().sortElement(test2);
        System.out.println("test1 = " + Arrays.toString(test2));

    }
}
```

