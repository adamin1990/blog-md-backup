# Insertion Sort 
# 插入排序 
---
Insertion Sort is a simple sorting algorithm that works the way
we sort playing cards in our hands.We sort the first two cards and then
place the third card in the appropriate position within the first two,
and then the fourth is positioned within the first three,and so until
the whole hand is sorted.  
插入排序也是一种简单的排序算法，它的原理跟我们玩扑克牌排序一样。首先排序前两张牌，然后在他们中合适的未知插入第三张牌，然后在前三张合适的位置插入第四张，直到手里的牌全部有序。  
During an iteration,an element of the list is inserted into the sorted
portion of the array to its left.So,basically,for each iteration,we
have an array of sorted elements to the left,and an array of other
elements still to be sorted to the right.  
在一次迭代中，一个列表中的元素被插入到排序部分数组的左侧，因此每次迭代，我们左侧会有一个有序的数组，右侧有一个需要被排序的数组。  
for example :
 * the array is [4,1,5,2]
 * step1:[1,4,5,2],start with second element 1 and position it in the
 * "array" of the first two elements.now the left sorted array is [1,4]
 * step2:[1,4,5,2] the next element is 5,insert it properly two the left
 * array,then the left sorted array is [1,4,5]
 * step3: [1,2,4,5]the next element is 2,insert it properly two the left array  

## Java Implementation
## Java实现
```java
public class InsertionSort implements Isort{
    @Override
    public void sortElement(int[] elements) {
        int length = elements.length;

        for(int i=0;i<length;i++){
              for(int j=i;j>0;j--){
                  if(elements[j]<elements[j-1]){
                      swap(elements,j,j-1);
                  }else {
                      break;
                  }
              }

        }

    }

    private void swap(int[] elements, int j, int i) {
        int temp=elements[j];
        elements[j]=elements[i];
        elements[i]=temp;
    }

    private int[] insertIntoArray(int[] elements, int insertPosition, int targetPos) {
        System.out.println("insertIntoArray() = " + insertPosition+"-------"+targetPos);
        int target=elements[targetPos];
        List<Integer> ints = Arrays.stream(elements).boxed().collect(Collectors.toList());
        ints.remove(targetPos);
        ints.add(insertPosition,target);
       return ints.stream().mapToInt(value -> value).toArray();
    }

    public static void main(String[] args) {
     int [] test=new int[]{4,5,2,8};  // 2,3,4,1
     new InsertionSort().sortElement(test);
        System.out.println("Arrays.toString(test) = " + Arrays.toString(test));
    }
}
