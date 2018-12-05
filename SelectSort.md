# Select Sort
选择排序

    /**
     * 选择排序
     * 1. 选择排序的算法时间平均复杂度为O(n²)。
     * 2. 选择排序空间复杂度为 O(1)。
     * 3. 选择排序为不稳定排序。
     */
    public class Select {
      public static void sort(int[] arr) {
        int min;
        for (int i = 0; i < arr.length; i++) {
          min = i;
          for (int j = i+1; j < arr.length; j++) {
            if (arr[min] > arr[j]) {
              min = j;
            }
          }
          if (min != i) {
            Utils.swap(arr, min, i);
          }
        }

        for (int a: arr) {
          System.out.println(a);
        }
      }
    }
