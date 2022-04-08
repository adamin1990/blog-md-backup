## LEETCODE PROBLEMS ---TWO SUM (两数之和)
------
### Description(问题描述)

  Give an array of integers **nums** and an integer **target**,return **indices** of the two numbers such that they add up to **target**.  
  给定一个整型数组　**nums**和一个整型目标值　**target**,要求返回两个数的下标，满足两数之和是　**target**.


### Example(实例)
```
Input: nums =[2,4,5,6] target: 7
Output: [0.3]
```
### Solutions(writen in java) 

#### Solution 1 : use for loop  (解法1：使用for循环)

```
   public int[] twoSum(int[] nums, int target) {
        int length = nums.length;
        int[] res = new int[2];
        for (int i = 0; i < length; i++) {
            int j = i + 1;
            if (j < length) {
                for (int k = i + 1; k < length; k++) {
                    if (nums[i] + nums[k] == target) {
                        res[0] = i;
                        res[1] = k;
                        return res;
                    }
                }


            }


        }
        return res;
    }
```
#### Solution 2: use ArrayList（解法2：使用ArrayList）

```
   public static int[] twoSum2(int[] nums, int target) {
        int[] res = new int[2];
        ArrayList<Integer> list = new ArrayList<Integer>();
        for (int i = 0; i < nums.length; i++) {
            if (list.contains(nums[i])) {
                res[0] = list.indexOf(nums[i]);
                res[1] = i;
            }
            list.add(target - nums[i]);
        }
        return res;
    }
    
```

#### Solution 3: use Hashmap (解法3：使用Hashmap)

```
   public static int[] twoSum3(int[] nums, int target) {
        int[] res = new int[2];
        Map<Integer, Integer> data = new HashMap<>();
        for (int i = 0; i < nums.length; i++) {
            if (data.containsKey(nums[i])) {
                res[0] = data.get(nums[i]);
                res[1] = i;
            }
            data.put(target - nums[i], i);
        }
        return res;
    }
```
