---
title: CS50 Week3
author: XuJinyao
date: 2025-10-31 23:28:09
tags:
- CSDIY
- CS50
categories:
- Programming
description: 
- 这篇文章讲解了CS50在Week3里提到的三种排序算法
---
# WEEK 3

## 概述

在CS50的第三周，我们主要了解到了三种算法：冒泡排序（bubble sort），选择排序（selection sort），以及归并排序（merge sort），还有递归的思想，并且对时间复杂度有了一个大概的了解。下面我们来总结一下这三种算法的特点以及代码实现。

btw，其实在C++中，我们不用自己实现排序算法，因为C++已经内置了排序函数，也就是std::sort()，并且sort()使用的是快速排序算法，其时间复杂度为O(nlogn)。 
## 冒泡排序（Bubble Sort）

冒泡排序是一种简单的排序算法。主要思想是通过对相邻元素进行比较和交换，使得较大的元素逐渐向上移动，直到最后一个元素。

### 特点

- 时间复杂度：O(n^2)
- 空间复杂度：O(1)
- 稳定性：稳定

### 代码实现

```C
#include <stdio.h>

void bubble_sort(int arr[], int n) {
    int i, j;
    for (i = 0; i < n - 1; i++) {
        for (j = 0; j < n - i - 1; j++) {
            if (arr[j] > arr[j + 1]) {
                int temp = arr[j];
                arr[j] = arr[j + 1];
                arr[j + 1] = temp;
            }
        }
    }
}
```

## 选择排序（Selection Sort）

选择排序是一种简单直观的排序算法。主要思想是通过遍历数组，找到最小的元素，将其放在数组的第一个位置，然后再从剩下的元素中找到最小的元素，将其放在数组的第二个位置，以此类推。

### 特点

- 时间复杂度：O(n^2)
- 空间复杂度：O(1)
- 稳定性：不稳定

### 代码实现

```C
#include <stdio.h>

void selection_sort(int arr[], int n) {
    int i, j, min_idx;
    for (i = 0; i < n - 1; i++) {
        min_idx = i;
        for (j = i + 1; j < n; j++) {
            if (arr[j] < arr[min_idx]) {
                min_idx = j;
            }
        }    
        int temp = arr[i];
        arr[i] = arr[min_idx];
        arr[min_idx] = temp;
    }
}
```

## 归并排序（Merge Sort）

归并排序是一种分治法的排序算法。主要思想是将数组分成两半，分别排序，然后再将排序好的两半合并成一个有序数组。

### 特点

- 时间复杂度：O(nlogn)
- 空间复杂度：O(n)
- 稳定性：稳定

### 代码实现

```C    
#include <stdio.h>

void merge(int arr[], int l, int m, int r) {
    int i, j, k;
    int n1 = m - l + 1;
    int n2 = r - m;
    int L[n1], R[n2];
    for (i = 0; i < n1; i++) {
        L[i] = arr[l + i];
    }    
    for (j = 0; j < n2; j++) {
        R[j] = arr[m + 1 + j];
    }    
    i = 0;
    j = 0;
    k = l;
    while (i < n1 && j < n2) {
        if (L[i] <= R[j]) {
            arr[k] = L[i];
            i++;
        } else {
            arr[k] = R[j];
            j++;
        }
        k++;
    }    
    while (i < n1) {
        arr[k] = L[i];
        i++;
        k++;
    }    
    while (j < n2) {
        arr[k] = R[j];
        j++;
        k++;
    }
}

void merge_sort(int arr[], int l, int r) {
    if (l < r) {
        int m = l + (r - l) / 2;
        merge_sort(arr, l, m);
        merge_sort(arr, m + 1, r);
        merge(arr, l, m, r);
    }
}
```


## 总结

在Week3视频的最后，David通过动画演示直观地展现了三种排序算法的过程，可以很清晰地看到归并排序的时间是最快的，这主要是因为归并排序的分治法的思想，使得它可以将数组分成两半，分别排序，然后再将排序好的两半合并成一个有序数组，这样就使得时间复杂度降低到O(nlogn)。而选择排序和冒泡排序的时间复杂度都为O(n^2)，这主要是因为它们的比较和交换操作，使得它们的效率较低。
其实在写函数的时候，我们也可以利用递归思想，也就是分治法来优化代码，同样来自David，例如我们要写一个打印Mario里的砖块（阶梯型）的函数，我们可以这样思考：每次打印下一层时，其实就是比上一层砖块多了一个砖块。让我们来看看下面的代码。
```C
void draw(int n)
{
    if (n <= 0)
    {
        return;
    }

    draw(n - 1);

    for (int i = 0; i < n; i++)
    {
        printf("#");
    }
    printf("\n");
}
```
