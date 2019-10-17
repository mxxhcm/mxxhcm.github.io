---
title: algorithm-sort
date: 2019-10-16 15:06:10
tags:
 - 数据结构
 - 排序
 - 冒泡排序
 - 快速排序
 - 选择排序
 - 堆排序
 - 插入排序
 - 希尔排序
 - 计数排序
 - 归并排序
 - 桶排序
 - 基数排序
categories: 数据结构
mathjax: ture
---

## 排序分类
### 内部排序
#### 基于比较的排序
交换排序
- 冒泡排序
- 快速排序

插入排序
- 简单插入排序
- 希尔排序

选择排序
- 简单选择排序
- 堆排序

归并排序

#### 非比较排序
- 计数排序
- 桶排序
- 基数排序

### 外部排序
- 多路合并
- 置换选择排序

## 交换排序
### bubble sort
#### 思路
在第$i$趟排序过程中，最后$i-1$个值是有序的，对剩余的$n-i+1$个元素，不断的交换两个相邻位置的值，使得后面的值大于前面的值，最后这$n-i+1$个元素中的最大值跑到了倒数第$i$个位置，就像冒泡一样。。
核心思想就是只交换两个相邻的值。

#### 属性
- 稳定
- 最坏时间复杂度$O(n^2 )$
- 最好时间复杂度$O(n )$，在数组正序的情况下，只进行一轮排序，$n-1$次比较，在数组倒序的情况下，进行$n-1$轮排序，进行很多次比较，是$O(n^2 )$。
- 平均时间复杂度$O(n^2 )$
- 空间复杂度$O(n)$

#### 代码
``` c
void bubble_sort(int a[], int n)
{
    for(int i = 0; i < n - 1; i++)
    {
        for(int j = 0; j < n - i - 1; j++)
        {
            if(a[j] > a[j+1])
            {
                int temp = a[j];
                a[j] = a[j+1];
                a[j+1] = temp;
            }
        }
    }
}
```

### quick sort
#### 思路
把一个数组分成两部分，然后在将这两部分的每一部分继续分下去，一直分解到每一个部分只有一个元素，这样子就有序了。

#### 属性
- 不稳定
- 最坏时间复杂度$O(n^2 )$，在数组基本有序的情况下退化成冒泡排序了。
- 最好时间复杂度$O(n\log n )$
- 平均时间复杂度$O(n\log n )$
- 空间复杂度$O(n\log n)$

#### 时间复杂度计算
##### 最坏情况下
数组有序，扫描一遍将它分解成了N-1和1，N-1分解称了N-2, 1
对于$N$个元素的数组，partition的时间复杂度是：
$T(N) = N - 1 + T(N-1)$，即扫描一遍需要比较的次数
$T(N-1) = N - 2 + T(N-2)$
$T(N-2) = N - 3 + T(N-3)$
$\cdots $
$T(3) = 2 + T(2)$
$T(2) = 1 + T(1)$
$T(1) = 0$
合计就是：$N-1 + N-2 + \cdots + 2 + 1 + 0= O(N^2 )$

##### 平均情况下
$T(N) = 2T(\frac{N}{2}) + N$ 做了近似
$\frac{T(N)}{N}) = \frac{T(\frac{N}{2})}{\frac{N}{2}} + 1$
$\frac{T(\frac{N}{2})}{\frac{N}{2}} =\frac{T(\frac{N}{4})}{\frac{N}{4}} + 1 $
$\cdots$
$\frac{4}{4} =\frac{T(4)}{4} + 1$
$\frac{2}{2} =\frac{T(1)}{1} + 1$
而
$T(1) = 0$
$\frac{T(2)}{2} = 1$
$\frac{T(4)}{4} = T(2) + T(1) = 2$
$\frac{T(8)}{8} = T(4) + T(2) + T(1) = 3$
$\frac{T(16)}{16} = T(8) T(4) + T(2) + T(1) = 4$
所以
$\frac{T(N)}{N} = \log N$
$T(N) = N\log N$


#### 代码
``` c
int partition(int a[], int low, int high)
{
    int pivot = a[low];
    while(low < high)
    {
        while(low < high && a[high] >= pivot)
        {
            high --;
        }
        a[low] = a[high];
        while(low < high && a[low] <= pivot)
        {
            low ++;
        }
        a[high] = a[low];
    }
    a[low] =  pivot;
    return low;
}


int qsort(int a[], int low, int high)
{
    if( low < high)
    {
        int pivot = partition(a, low, high);
        qsort(a, low,  pivot - 1);
        qsort(a, pivot + 1, high);
    }
}


void quicksort(int a[], int n)
{
    qsort(a, 0, n-1);
}
```

## 插入排序
### 简单插入排序insert sort
#### 思路分析
在第$i$趟排序过程中，前面$i-1$个值是有序的，将第i个数字插入前面有序的长为$i-1$的序列中，构成长为$i$的有序序列。

#### 特点
- 稳定
- 最坏时间复杂度$O(n^2 )$
- 最好时间复杂度$O(n )$，在数组有序的情况下
- 平均时间复杂度$O(n^2 )$
- 空间复杂度$O(n)$

#### 代码
``` c
void insert_sort(int a[], int n)
{
    for(int i = 1; i < n; i++)
    {
        
        int temp = a[i];
        for(int j = i - 1; j >= 0; j--)
        {
            if(a[j] > temp)
            {
                a[j+1] = a[j];
            }
            else 
            {
                a[j+1] = temp;
                break;
            }
        }
    }
}
```

### 希尔排序
## 选择排序
### 简单选择排序
#### 思路简介
在第$i$趟排序过程中，前面$i-1$个值有序，从后面$n-i+1$个值中选择第$i$小的数，记下下标min_index，如果$i$和min_index不相等的话，交换它们的值。

#### 属性
- 不稳定
- 最坏时间复杂度$O(n^2 )$
- 最好时间复杂度$O(n^2 )$
- 平均时间复杂度$O(n^2 )$
- 空间复杂度$O(n)$

#### 代码
``` c
void selection_sort(int a[], int n)
{
    for(int i = 0; i < n - 1; i++)
    {
        int min_index = i;
        for(int j = i; j < n; j++)
        {
            if(a[j] < a[min_index]) 
                min_index = j;
        }
        if(min_index != i)
        {
            int temp = a[min_index];
            a[min_index] = a[i];
            a[i] = temp;
        }
    }
}
```


## heapsort
1.https://www.geeksforgeeks.org/heap-sort/
2.http://www.techgeekbuzz.com/heap-sort-in-c/
3.http://www.techgeekbuzz.com/heap-sort-in-c/
4.https://www.zentut.com/c-tutorial/c-heapsort/

## 简单排序
常见的三大简单排序：冒泡排序，简单选择排序，简单插入排序。
冒泡排序：
- 优点:比较简单，空间复杂度较低，是稳定的；
- 缺点:时间复杂度太高，效率慢；

选择排序：
- 优点：一轮比较只需要一次交换；
- 缺点：效率慢，不稳定（举个例子2，2，1， 假设我们每趟排序都选择第一个元素作为比较值，第一趟排序会交换第一个2和1，两个2的位置已经变了）。

冒泡排序和简单选择排序的区别和联系：
- 冒泡排序每趟都是比较无序序列中相邻位置的两个数，而选择排序每趟都是选择无序序列中的最小数；
- 冒泡排序每一轮比较后，位置不对都需要换位置，选择排序每一趟排序都只需要一次交换；
- 冒泡排序是通过数去找位置，选择排序是给定位置去找数；
 

## 参考文献
1.https://www.quora.com/What-is-the-time-complexity-of-quick-sort/answer/Lakshmi-Narayana-217
2.https://www.cnblogs.com/Good-good-stady-day-day-up/p/9055698.html
3.https://www.cnblogs.com/onepixel/p/7674659.html
