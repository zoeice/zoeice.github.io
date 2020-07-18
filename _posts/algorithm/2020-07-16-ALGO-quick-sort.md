---
layout:     post
title:      快速排序
description: 快速排序（英语：Quicksort），又称分区交换排序，简称快排，一种排序算法，最早由东尼·霍尔提出。 
date:       2016-08-24 11:06:00
author:     zoeice
image: https://zoeice-blog.oss-cn-shanghai.aliyuncs.com/post-bg-quick-sort.jpeg
optimized_image: https://zoeice-blog.oss-cn-shanghai.aliyuncs.com/post-bg-quick-sort.jpeg?x-oss-process=image/resize,w_380
catalog: true
category: ALGO
tags:
    - 排序
    - Typescript
---

**快速排序**（英语：Quicksort），又称**分区交换排序**，简称**快排**，一种排序算法，最早由东尼·霍尔提出。
- 平均时间复杂度为 O(*n* log *n*)
- 最坏时间复杂度为 O(n^2)   *这样的情况不常见*
- 最优时间复杂度为 O(*n* log *n*)
![排序示例](https://zoeice-blog.oss-cn-shanghai.aliyuncs.com/gif/Sorting_quicksort_anim.gif)
实际上，快排通常明显比大多数排序算法更快

#### 算法
快速排序使用**分治法**（Divide and conquer）策略来把一个序列（list）分为较小和较大的2个子序列，然后递归地排序两个子序列。

步骤为：
1. **挑选基准值**：从数列中挑出一个元素，称为“基准”（pivot），
2. **分割**：重新排序数列，所有比基准值小的元素摆放在基准前面，所有比基准值大的元素摆在基准后面（与基准值相等的数可以到任何一边）。在这个分割结束之后，对基准值的排序就已经完成，
3. **递归排序子序列**：递归地将小于基准值元素的子序列和大于基准值元素的子序列排序。

其中 **选取基准值** 的方式差异对排序的时间性能有决定性影响。

下面用Typescript实现个demo看看
```js
// 成员互换
function swap(arr: Array<number>, i: number, j: number) {
    const temp = arr[i];
    arr[i] = arr[j];
    arr[j] = temp;
}

// 原地分割算法
function partition(arr: Array<number>, left: number, right: number): number {
    var pivotIndex = left;
    var index = pivotIndex + 1;
    for (var i = index; i <= right; i++) {
        if (arr[i] < arr[pivotIndex]) {
            swap(arr, i, index);
            index++;
        }
    }
    swap(arr, pivotIndex, index - 1);
    return index - 1;
}

// 快速排序
function quickSort(arr: Array<number>, left: number, right: number): Array<number> {
    var partiationIndex;
    if (left < right) {
        partiationIndex = partition(arr, left, right);
        quickSort(arr, left, partiationIndex - 1);
        quickSort(arr, partiationIndex + 1, right);
    }
    return arr;
}
```

然后随机生成一个数组用来测试排序算法：
```
function generateRandomArray(size: number): Array<number> {
    if(size > 10000) size = 10000
    let array = []
    for (let i = 0; i < size; i++) {
        let num = Math.floor(Math.random() * size)
        array.push(num)
    }
    return array
}

let array = generateRandomArray(100)
console.log('test', quickSort(array, 0, array.length))
```
运行结果：
```
[0, 0, 2, 3, 3, 3, 3, 7, 12, 12, 12, 12, 13, 15, 17, 18, 18, 18, 19, 19, 20, 20, 22, 23, 23, 23, 23, 25, 26, 26, 27, 28, 29, 29, 31, 32, 34, 34, 35, 36, 36, 37, 39, 44, 44, 45, 46, 46, 46, 47, 49, 52, 53, 53, 54, 54, 56, 58, 60, 60, 61, 61, 61, 61, 62, 63, 63, 64, 64, 65, 65, 65, 67, 68, 71, 72, 72, 74, 76, 78, 78, 79, 80, 80, 80, 83, 84, 84, 84, 85, 85, 86, 87, 93, 93, 94, 94, 94, 95, 98]
```