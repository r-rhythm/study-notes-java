# 二分查找 Binary Search

==注意: 使用二分查找的前提时需要确定数组已经是有序的==, 如果不确定, 则可用先排序后在使用二分查找

```java
public class BinarySearchTest {  
    public static void main(String[] args) throws InstantiationException, IllegalAccessException {  
          
        int[] array = {1, 2, 3, 4, 5};  
        int target = 5;  
          
        int result = binarySearch(array, target);  
        System.out.println(result);  
    }  
      
    /**  
     * 二分查找  
     *  
     * @param arr  
     * @param target  
     * @return  
     */  
    public static int binarySearch(int[] arr, int target) {  
        // 确定最左及最右  
        int left = 0;  
        int right = arr.length - 1;  
          
        while (left <= right) {  
            int mid = left + (right - left) / 2;  
              
            if (arr[mid] == target) {  
                return mid;  
            } else if (arr[mid] < target) {  
                left = mid + 1;  
            } else {  
                right = mid - 1;  
            }  
        }  
        return -1;  
    }  
}
```