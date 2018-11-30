# BubbleSort
冒泡排序

    package com.droid;
    public class BubbleSort {
      public static void main(String[] args) {
        int[] arr = { 6, 4, 2, 1, 8, 3, 7, 9, 5 };
        sort(arr);
        print(arr);
      }

      private static void sort(int[] arr) {
        if (arr == null)
          return;
        // 定义一个标记 isSort，当其值为 true 的时候代表已经有序。
        boolean isSort;
        for (int i = 0; i < arr.length - 1; i++) {
          isSort = true;
          for (int j = 1; j < arr.length - i; j++) {
            if (arr[j - 1] > arr[j]) {
              swap(arr, j - 1, j);
              isSort = false;
            }
          }
          if (isSort)
            break;
        }
      }

      /**
       * 数据交换
       * 
       * @param arr
       * @param i
       * @param j
       */
      private static void swap(int[] arr, int i, int j) {
        int temp = arr[i];
        arr[i] = arr[j];
        arr[j] = temp;
      }

      /**
       * 打印
       * 
       * @param arr
       */
      private static void print(int[] arr) {
        for (int anArr : arr) {
          System.out.print(anArr + " ");
        }
      }
    }
#### 冒泡排序总结：

###### 1. 冒泡排序的算法时间平均复杂度为 O(n²)。
###### 2. 空间复杂度为 O(1)。
###### 3. 冒泡排序为稳定排序。

###### 稳定排序：通俗地讲就是能保证排序前两个相等的数据其在序列中的先后位置顺序与排序后它们两个先后位置顺序相同。
