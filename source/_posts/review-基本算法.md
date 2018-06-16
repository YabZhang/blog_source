---
title: review-基本算法
description: 基础算法
date: 2017-08-05 21:06:23
tags:
---

本篇主要是复习下基本的算法。

### basic algo.

谈到基本算法肯定就是排序和查找了，那就具体来看下这些基本的排序和查找算法；

#### sort

基本的排序算法包括：
１. 冒泡排序
２. 选择排序
３. 插入排序
４. 希尔排序
５. 归并排序
６. 快排

排序算法分为稳定和非稳定两类。
稳定排序算法的复杂度不会因序列有所改变；
而非稳定排序的算法复杂度会因特定的原因导致算法执行效率下降，复杂度增加；
比较型排序算法的Big-O复杂度一般不会低于`O(nlogn)`；

* 冒泡排序

冒泡排序是稳定排序，复杂度`O(n^2)`;
排序时，依次遍历序列，两两比较。若后者大于前者则交换顺序，否则继续下一组比较；
当一轮遍历完成后，最大的元素就会被交换到序列的最后一位，整个过程就像冒泡一样；
之后继续进行下一轮遍历，比较和交换位置。具体过程同上；
若某一次遍历完成后，并没有元素需要交换位置，那么排序就完成了；
遍历过程，最少一次，最多n次;

{% codeblock lang:python %}
def bubble_sort(seq):
    done = None
    for i in range(len(seq) - 1):
        done = True
        for j in range(1, len(seq) - i):
            if seq[j] < seq[j - 1]:
                seq[j], seq[j - 1] = seq[j - 1], seq[j]
                done = False
        if done:
            break
    return seq
{% endcodeblock %}

* 选择排序

选择排序是稳定排序,复杂度`O(n^2)`;
选择排序就像字面意思一样，每次从剩余列表中选择出最小的元素排在前面；
具体的过程就像小学做操的时候，每次排队，老师都找到最矮的排在前面，全部过程结束后，就排好了队；
这里在排序时，也是每次都找到剩余部分最小的元素，然后将最小的元素与其应该在的位置上的元素交换即可；

{% codeblock lang:python %}
def selection_sort(seq):
    smallest = None
    for i in range(len(seq) - 1):
        smallest = i
        for j in range(i + 1, len(seq)):
            if seq[j] < seq[smallest]:
                smallest = j
        seq[i], seq[smallest] = seq[smallest], seq[i]
    return seq
{% endcodeblock %}

* 插入排序

插入排序是稳定排序，复杂度`O(n^2)`;
插入排序时先分出一部分序列是拍好序的，然后每次从未排序的部分取出元素插入到已排序的部分中应该在的位置；
一直持续这个过程，直到所有的未排序全部都插入到已排序列中，排序完成；
在代码实现中，选择以交换的方式来移动位置；

{% codeblock lang:python %}
def insert_sort(seq):
    for i in range(1, len(seq)):
        for j in range(i, 0, -1):
            if seq[j] < seq[j - 1]:
                seq[j], seq[j - 1] = seq[j - 1], seq[j]
            else:
                break
    return seq
{% endcodeblock %}

* 希尔排序

希尔排序是插入排序的升级版，目的是为了减少排序时的移位操作；
具体的做法是先把待排序列分成一定间隔的元素组成的子序列，使用插入排序完成子序列的排序；
然后进一步减少分组数目，重新分配子序列，再次重复上述过程；
最后分组数目减少为１时，就与插入排序相同了；但移位操作可以有效减少；
最坏复杂度为`O(n^2)`；

{% codeblock lang:python %}
def shell_sort(seq)
    sub_cnt = len(seq) // 2
    while sub_cnt > 0:  # 最小为１个分组
        for start in range(sub_cnt):
            for i in range(start + sub_cnt, len(seq), sub_cnt):
                for j in range(i, start, -sub_cnt):
                    if seq[j] < seq[j - sub_cnt]:
                        seq[j], sq[j - sub_cnt] = seq[sub_cnt], seq[j]
        sub_cnt = sub_cnt // 2  # 减少分组
    return sub_cnt
{% endcodeblock%}

* 归并排序

归并排序使用了分治的思想，把问题分解为子问题，然后递归这个过程。
直到最终的子问题容易解决，然后把结果合并；最后通过合并子问题的结果，得到原有问题的解答；
归并排序是稳定排序，算法复杂度为`O(nlogn)`,需要额外的存储空间`O(n)`；

排序过程，先选择一个基准元素，与基准元素比较把列表分为两自列表：小于基准元素和大于等于基准元素。
之后对两自列表重复递归上述过程；最后可以得到长度为１子列表，可以认为这样的列表是已排序的；
然后把排好序的列表收集起来，最终得到排好序的列表；

{% codeblock lang:python %}
def merge_sort(seq):
    if len(seq) <= 1:
        return seq

    mid = len(seq) // 2
    left = merge_sort(seq[:mid])
    right = merge_sort(seq[mid:])

    cnt, m, n = [0] * 3
    while m < len(left) and n < len(right):
        if left[m] <= right[n]:
            seq[cnt] = left[m]
            m += 1
        else:
            seq[cnt] = right[n]
            n += 1
        cnt += 1

    if m < len(left):
        while m < len(left) and cnt < len(seq):
            seq[cnt] = left[m]
            cnt += 1
            m += 1
    elif n < len(right):
        while n < len(right) and cnt < len(seq):
            seq[cnt] = right[n]
            cnt += 1
            n += 1
    return seq
{% endcodeblock %}

* 快排

快排是非稳定排序，一般情况时算法的复杂度为`O(n)`；
但若是在最坏情况下，算法复杂度会退化到`O(n^2)`；
快排的过程，每一次迭代会从序列中选出一个基准元素，然后以此基准把序列分为较小(左)和较大(右)的两部分；
然后再分别对这两部分重复此过程；最后得到排好的序列；

最坏情况出现在每次选择基准元素均选中了最小或最大的元素，这样序列就不能成功地分为两部分；
复杂度就会退化为`O(n^2)`的复杂度；所以一般选择使用随机数来选择基准元素，尽量避免最坏情况的出现；

采用原地快排可以节省额外空间的使用；

{% codeblock lang:python %}
import random
def quick_sort(seq, start, end):
    if end - start < 1:
        return
    base = random.randint(start, end)
    seq[start], seq[base] = seq[base], seq[start]

    pivot, cnt = start, start
    for i in range(start + 1, end + 1):
        if seq[i] < seq[pivot]:
            cnt += 1
            seq[cnt] = seq[i]
    seq[pivot], seq[cnt] = seq[cnt], seq[pivot]

    quick_sort(seq, start, cnt - 1)
    quick_sort(seq, cnt + 1, end)
    return seq
{% endcodeblock %}

#### search

基本的查找算法：顺序查找和二分查找；

* 顺序查找

顾名思义，就是按照顺序查找；复杂度`O(n)`

{% codeblock lang:python %}
def sequential_search(seq, item):
    for i in range(len(seq)):
        if seq[i] == item:
            return i
    return -1
{% endcodeblock %}

* 二分查找

二分查找，算法复杂度为`O(logn)`；
要求序列是有序的；

{% codeblock lang:python %}
def bin_search(seq, item):
    start, end = 0, len(seq) - 1
    while start <= end:
        mid = (start + end) // 2
        if seq[mid] > item:
            end = mid - 1
        elif seq[mid] < item:
            start = mid + 1
        else:
            return mid
    return -1
{% endcodeblock %}

### the end


基本算法：

６种基本排序算法；
比较型排序算法最快不会小于`O(nlogn)`；

两种基本查找算法，其中二分查找要求序列有序。
