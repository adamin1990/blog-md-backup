# Selection Sort 
# 选择排序  
-----
Selection Sort is a simple sorting algorithm that find the smallest
element in the array and swaps it with the element in the first position,
then find the second smallest element in and swaps it with the element in
the second position,and continues in this way until the entire array
is sorted.  
选择排序是一种简单的排序算法，首先查找数组中最小的元素，跟数组元素第一位交换，然后查找第二小的元素，跟数组第二位元素交换，依此类推，直到整个数组元素有序。  
The algorithm is efficient for smaller lists,but very inefficient for
larger lists,Also,it does not require any additional storage space,as
it operate in-place.  
当列表元素较少时，选择排序很高效，但是当有大量元素时，就非常不高效了。  
for example:
 * the array is [4,1,5,2]
 * step 1: [1,4,5,2] swap 4,1,because 1 is the smallest
 * step 2: [1,2,5,4] swap 4,2 because 2 is the second smallest
 * step 3: [1,2,4,5] swap 5,4 because 4 is the third smallest  

## Java Implementation
## Java 实现  
------
```java
public class SelectionSort implements Isort{
    @Override
    public void sortElement(int[] elements) {
        int length =elements.length;
        for (int i=0;i<length-1;i++){ //最后一个元素后面没有数据了，所以没必要包含在循环体内
            int ele=elements[i];
            int swapIndex=-1;
            boolean swap=false;
            for (int j=i+1;j<length;j++){
                System.out.println("jvalue = " + j+"---"+ele);
                if(ele>elements[j]){ // save the smaller value and index
                    ele=elements[j];
                    swapIndex=j;
                    swap=true ;
                }
            }
            if(swap&&swapIndex>-1){  // swap the values
                int temp=elements[i];
                elements[i]=ele;
                elements[swapIndex]=temp;
            }
        }

    }

    public static void main(String[] args) {

        int [] test1=new int[]{4,1,5,2};
        int [] test2=new int[]{0,9,8,7,6,1,2,3,4,5,11};
        new SelectionSort().sortElement(test2);
        System.out.println("sortValues = " + Arrays.toString(test2));

    }
}
```
