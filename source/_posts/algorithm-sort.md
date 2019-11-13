---
title: algorithm-sort
date: 2019-10-16 15:06:10
tags:
 - 数据结构
 - 排序
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
- 最好空间复杂度$O(\log n)$，最差是$O(n)$的空间复杂度。每一次需要$O(1)$常数空间存储pivot的位置，在递归调用保存栈的时候需要空间。至多有$\log n$或者$O(n)$次，所以空间复杂度就是$O(n)$。

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

### 性能测试
自己实现的：
1万数据，大概是0.004秒
10万数据，大概是0.013秒
100万数据，大概是0.13秒


C语言qsort：
100万数据，大概是0.03到0.06秒
1000万数据，大概是0.3秒左右
1亿数据，大概是3秒左右
C++sort：
100万数据，大概是0.13秒左右
1000万数据，大概是1.4秒左右
1亿数据，大概是16秒左右

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
//首先我写了下面的代码，实际上是有问题的，
//错误示例！！！！！！！！！！！！！！！！！！！！！！！
void insert_sort(int a[], int n)
{
    int i = -1, j = -1, temp = -1;
    for(i = 1; i < n; i++)
    {
        temp = a[i];
        for(j = i - 1; j >= 0; j--)
        {
            if(a[j] > temp)
            {
                a[j+1] = a[j];
            }
            else 
            {
                //如果第j论插入应该插入在0位置时，会跳过这一步的执行。
                a[j+1] = temp;
                break;
            }
        }
    }
}
//正确示例
void insert_sort(int a[], int n)
{
    int i = -1, j = -1, temp = -1;
    for(i = 1; i < n; i++)
    {
        temp = a[i];
        for(j = i - 1; j >= 0; j--)
        {
            if(a[j] > temp)
            {
                a[j+1] = a[j];
            }
            else 
            {
                break;
            }
        }
        a[j+1] = temp;
    }
}
```

### 希尔排序
#### 思路简介
希尔排序是对插入排序的扩展。

#### 属性
- 不稳定
- 平均的时间复杂度$O(n^{1.3} )$
- 最坏的时间复杂度$O(n^2 )$
- 最好的时间复杂度$O(n )$
- 空间复杂度是$O(1)$

#### 代码
``` c
void shell_sort(int a[], int n)
{
    int i = -1, j = -1, temp = -1;
    for(int gap = n/2; gap > 0; gap/=2)
    {
        for(i = gap; i < n; i++)
        {
            temp = a[i];
            for(j = i - gap; j >= 0; j -= gap)
            {
                if(temp < a[j])
                {
                    a[j+gap] =  a[j];
                }
                else
                {
                    break;
                }
            }
            a[j+gap] = temp;
        }
    }
}
```

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


## 归并排序
### 思路
将数组分为原来的一半，一直分到每一个都只有一个元素，然后这两个有序数组合并到一块。

### 属性
- 稳定
- 最坏时间复杂度$O(n\log n)$
- 最好时间复杂度$O(n\log n)$
- 平均时间复杂度$O(n\log n)$
- 空间复杂度$O(n)$


### 代码
``` c
void merge_sort(int a[], int l, int r)
{
    if(l < r)
    {
        //int middle = (l+r)/2 may be overflow, use (r-l)/2 + l 
        int m = (r-l)/2 + l;
        merge_sort(a, l, m);
        merge_sort(a, m+1, r);
        merge(a, l , m, r);
    }

}


// 这个代码的话，有问题，每次都声明一个N的数组，时间和空间消耗都很大，所以需要进行改善，只需要声明一个r-l+1的临时数组就行了。
void merge(int a[], int l, int m, int r)
{
    //!!!错误
    //int res[N] = { 0 };
    int res[r-l+1] = {0};
    int i = l, j = m+1, k = 0;
    // merge
    while((i <= m) && (j <= r))
    {
        if(a[i] < a[j])
        {
            res[k++] = a[i++];
        }
        else
        {
            res[k++] = a[j++];
        }
    }

    // copy left
    for( ; i <= m; )
    {
        res[k++] = a[i++];
    }

    for( ; j <= r; )
    {
        res[k++] = a[j++];
    }

    // copy to a
    for(int t = l; t <=r; t++)
    {
        a[t] = res[t-l];
    }
}
```

### 性能
100万，0.1秒左右
10万，0.02左右
1万，0.003左右

## 非比较排序

### 计数排序
#### 思路
假设输入的$n$个数都在$0-k$之间，对于输入的每个元素$x$，统计比它小的值或者和它相等的值的个数，那么这个值在排序后的位置也就确定了。

#### 排序过程
输入数组：[2, 5, 3, 0, 2, 3, 0, 3]
排序后的数组应该是：[0, 0, 2, 2, 3, 3, 3, 5]
1. 输入待排序数组a：
     0, 1, 2, 3, 4, 5, 6, 7
a = [2, 5, 3, 0, 2, 3, 0, 3], n=8
2. 使用数组c统计出现的次数：
     0, 1, 2, 3, 4, 5
c = [2, 0, 2, 3, 0, 1], k=5
3. 对c进行操作，计算每个值应该在的位置
     0, 1, 2, 3, 4, 5
c = [2, 2, 4, 7, 7, 8], k=5
4. 根据数组c和数组a给出排序后的数组output:
遍历数组的每一个值，给出他们应该在哪个位置
a[0] = 2, c[a[0]] = c[2] = 4, c[2]=3, output[4] = a[0] = 2;
a[1] = 5，c[a[1]] = c[5] = 8, c[5]=7, output[8] = a[1] = 5;
a[2] = 5，c[a[2]] = c[3] = 7, c[3]=6, output[7] = a[2] = 3; 7中放的是第一个3
a[3] = 5，c[a[3]] = c[0] = 2, c[0]=1, output[2] = a[3] = 0;
a[4] = 5，c[a[4]] = c[2] = 3, c[2]=2, output[3] = a[4] = 2;
a[5] = 5，c[a[5]] = c[3] = 6, c[3]=5, output[6] = a[5] = 3;
a[6] = 5，c[a[6]] = c[0] = 1, c[0]=0, output[1] = a[6] = 0;
a[7] = 5，c[a[7]] = c[3] = 5, c[3]=4, output[5] = a[7] = 3;
**正序遍历是不稳定的，倒序遍历是稳定的。上面就是正序的，可以发现不稳定**

#### 属性
- 稳定
- 不基于比较
- 时间复杂度是$O(n+k)$
- 空间复杂度是$O(n+k)$

#### 代码
``` c
void counting_sort(int a[], int n, int k)
{
    int output[n];
    int freq[k];
    memset(freq, 0, sizeof(freq));

    // freq[i] contains the number of elements equal to i
    for(int i = 0; i < n; i++)
    {
        freq[a[i]] ++;
    }
    
    // freq[i] contains the number of elements equal or less to i
    for(int i = 1; i <= k; i++)
    {
        freq[i] = freq[i] + freq[i-1];
    }
    
    for(int i = n - 1; i >= 0 ; i--)
    {
        output[freq[a[i]]--] = a[i];
    }
               
    for(int i = 0; i < n; i++)
    {
        a[i] = output[i+1];
    }
}

```

### 基数排序
#### 思路
借助多关键字排序的思想对单逻辑关键字排序。简单来说，对于十进制数字，依次按照个十百千万上每个位上的数字进行排序，假设有$d$位，需要分别对这$d$位进行排序。对每一位进行排序时，可以使用计数排序，因为$k$不大。

#### 排序过程
给出一组待排序数字：[329, 457, 657, 839, 436, 726, 255]
先按照个位数进行排序，
再按照十位数进行排序，
最后按照百位数进行排序

#### 属性
- 稳定
- 时间复杂度$O(d(n+k))$
- 空间复杂度$O(n+k)$

#### 代码
``` c
void bucket_sort(int a[], int n)
{
    int max = get_max(a, n);
    printf("max=%d\n", max);
    for(int exp=1; max/exp > 0; exp*=10)
    {
        counting_sort(a, n, exp);
    }
}


int get_max(int a[], int n)
{
    int max = a[0];
    for(int i = 1; i < n ;i++)
    {
        if(a[i] > max)
            max = a[i];
    }
    return max;
}


void counting_sort(int a[], int n, int exp)
{
    int count[10];
    int output[n+1];
    memset(count, 0, sizeof(count));

    //329, 457, 657, 839, 436, 726, 255
    // 1.count
    for(int i = 0; i < n; i ++)
    {
        int temp = (a[i] / exp) % 10;
        count[temp] ++;
    }

    // 2.accumulate count
    for(int i = 1; i < 10; i++)
    {
        count[i] = count[i] + count[i-1];
    }

    // 3.sort
    for(int i = n - 1; i >= 0; i--)
    {
        int temp = (a[i] / exp) % 10;
        output[count[temp]--] = a[i];
    }

    // 4. copy
    for(int i = 0; i < n; i++)
    {
        a[i] = output[i+1];
        printf("%d, ", a[i]);
    }
    printf("\n");
}
```

### 桶排序
#### 思路
计数排序是桶排序的一个特例，计数排序中使用的桶的个数和max-min+1的值相同，而通排序中桶的个数要小于等于max-min+1，等于max-min+1时，桶排序就退化成了计数排序。

#### 属性
- 稳定
- 最好的时间复杂度是$O(n)$
- 最坏的时间复杂度是$O(n^2 )$
- 平均时间复杂度是$O(n+k)$
- 空间复杂度是$O(n+k)$

#### 代码
```
int get_min(int a[], int n)
{
    int min = a[0];
    for(int i=1; i < n; i++)
    {
        if(a[i] < min)
        {
            min = a[i];
        }
    }
    return min;
}

int get_max(int a[], int n)
{
    int max = a[0];
    for(int i=1; i < n; i++)
    {
        if(a[i] > max)
        {
            max = a[i];
        }
    }
    return max;
}


void bucket_sort(int a[], int n, int bucket_number)
{
    // 1.创建n个桶
    std::vector<int> b[bucket_number];

    int max = get_max(a, n);
    int min = get_min(a, n);

    // 2.每个桶的大小
    int bucket_size = (max - min + 1) / bucket_number;
    for(int i = 0 ; i< n; i++)
    {
        int bucket_index = (a[i] - min) / bucket_size;
        b[bucket_index].push_back(a[i]);
    }

    for(int i = 0; i < bucket_number; i++)
    {
        sort(b[i].begin(), b[i].end());
    }
    int count = 0;
    for(int i = 0; i < bucket_number; i++)
    {
        for(int j = 0; j < b[i].size(); j++)
        {
            a[count++] = b[i][j];
        }
    }
}
```

## 快速排序vs归并排序
- 辅助空间：快排可以使用in-place方法实现，在每一个排序过程中不需要额外的空间，但是快排的递归实现，需要保存栈调用（是常数），平均情况下是$O(\log n)$，最坏情况下是$O(n)$；[使用尾递归的快排最坏情况下空间复杂度也是$O(\log n)$](https://www.geeksforgeeks.org/quicksort-tail-call-optimization-reducing-worst-case-space-log-n/)。而归并排序需要临时数组存储归并后的排序数组，[需要$O(n)$的空间复杂度](https://cs.stackexchange.com/a/35510)。
- 时间复杂度：快排在最坏情况下的时间复杂度是$O(n^2 )$，但是可以使用随机选择pivot的方式避免。
- 归并排序更适合大的数据结构，mergesore是稳定排序，可以修改成适合链表等数据结构的算法，以及内存和网络上的排序。因为在链表中，插入的时间和空间复杂度都是$O(1)$，因此链表的归并排序可以不需要额外的辅助空间。
- 快排和归并的平均时间复杂度都是$O(n\log n)$，但是因为归并排序需要分配以及销毁临时数组，所以要更慢一些。
- 快排是cache friendly的，对于数据来说，他有locality of reference。（[即是否很大可能性从cache中读取大量元素](https://stackoverflow.com/a/70631/8939281)）。


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
4.https://www.geeksforgeeks.org/radix-sort/
5.https://www.geeksforgeeks.org/quicksort-better-mergesort/
6.https://cs.stackexchange.com/a/35510
7.https://www.geeksforgeeks.org/quicksort-tail-call-optimization-reducing-worst-case-space-log-n/
