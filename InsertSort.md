#Insert Sort
插入排序
    /**
     * 插入排序
     * 1. 插入排序的算法时间平均复杂度为 O(n²)。
     * 2. 插入排序空间复杂度为 O(1)。
     * 3. 插入排序为稳定排序。
     * 4. 插入排序对于近乎有序的数组来说效率更高，插入排序可用来优化高级排序算法
     */
    public class Insert {
      public static void sort(int[] arr) {
        for (int i = 1; i < arr.length; i++) {
          int value = arr[i];
          int j = i - 1;
          for (; j >= 0 ; j--) {
            if (arr[j] > value) {
              arr[j+1] = arr[j];
            }
            else {
              break;
            }
          }
          arr[j+1] = value;
        }

        for (int a: arr) {
          System.out.println(a);
        }
      }
    }
