---
layout: post
title: "由mergeSort引发的一些思考"
subtitle: 'mergeSort thinking'
author: "markzhang"
header-style: text
tags:
  - algorithm
---

重新梳理一下归并排序以及一些相关的东西。

对于归并排序大家如果需要回忆下是个什么东西的话，可以点击这个[链接](https://visualgo.net/zh/sorting)，里面有各种排序的动画演示以及讲解，比我再用文字赘述一遍要好得多，功能相当强大。
![](/openBlog/img/merge-animation.gif)

先给出归并排序的js代码实现：
```javascript
function mergeSort(arr, l, r) {
    if (l === r) {
        return;
    }
    let mid = Math.floor((r + l) / 2);
    mergeSort(arr, l, mid);
    mergeSort(arr, mid + 1, r);
    merge(arr, l, mid, r);
}

function merge(arr, l, mid, r) {
    let leftIndex = l;
    let rightIndex = mid + 1;
    let helpArr = [];
    while(leftIndex <= mid && rightIndex <= r) {
        let leftItem = arr[leftIndex];
        let rightItem = arr[rightIndex];
        if (leftItem < rightItem) {
            helpArr.push(leftItem);
            leftIndex++;
        } else {
            helpArr.push(rightItem);
            rightIndex++;
        }
    }

    // 这俩循环只会进去一个，因为经过上面的比较，要么左边部分走完了，要么右边部分走完了
    while(leftIndex <= mid) {
        helpArr.push(arr[leftIndex]);
        leftIndex++;
    }

    while(rightIndex <= r) {
        helpArr.push(arr[rightIndex]);
        rightIndex++;
    }

    for (let index = 0; index < helpArr.length; index++) {
        arr[l] = helpArr[index];
        l++;
    }
}
```
如何估计归并排序的时间复杂度呢？

由于上面采用了递归写法，我们使用master公式对递归进行时间复杂度估算，以下是公式详情。
<p>T(n) = a*T(n/b) + O(n^d)<br/>
（1）、log(b, a) > d => 复杂度为O(n^log(b, a))<br/>
（2）、log(b, a) = d => 复杂度为O(n^d*logn)<br/>
（3）、log(b, a) < d => 复杂度为O(n^d)</p>

a代表递归的次数，由于在mergeSort中调用了两次mergeSort，所以归并排序中a = 2。<br/>
b代表样本量被划分几份，由于我们对样本量是一分为二将数组分为left和right部分，所以归并排序中b = 2。<br/>
O(n^d)代表其他操作的时间复杂度，所以在归并排序中主要是merge这个函数，相当于是执行了一次数组遍历，则为O(n)。<br/>
a = 2，b = 2，d = 1根据master公式，复杂度为nlogn。

我们知道冒泡排序、选择排序、插入排序的时间复杂度都是O(n^2)，当样本量比较大的时候，n^2比之nlogn差了可不是一星半点。这是为什么呢？因为在其他三种排序中，会浪费元素之间的比较，比如冒泡排序冒泡比较一轮只定位了一个元素，下一轮冒泡又只定位一个元素，会浪费元素之间的相互比较；而归并排序通过分治，由小到大进行比较合并的过程中，上一次比较合并的元素不会再次发生比较，有序的区域成规模增长，这样就不会浪费比较，节省了时间。

由归并排序引入数组小和问题和数组逆序对问题。
![](/openBlog/img/arr-small-sum.png)

根据小和的题目要求，我们思考一下可以发现，在归并排序过程中，left和right部分进行比较合并的时候，其实就可以找到左边部分比右边部分小的数，意思就是说我们可以很方便的在merge这个函数执行过程中来计算数组的小和且会快很多，因为合并的时候左右两遍都是有序的，如果一个数比右边的第一个数字小，我们可以得知这个数字肯定比右边全部的数字都小。

举个例子，比如left = [1,2,3]，right = [4,5,6]，1小于4，说明右边三个数都比1大，假如说小和等于sum，那么sum就要加1 * 3。

代码实现一下小和：
```javascript
function smallSum(arr) {
    if (!arr || arr.length < 2) {
        return 0;
    }
    return mergeSort(arr, 0, arr.length - 1);
}

function mergeSort(arr, l, r) {
    if (l === r) {
        return 0;
    }
    let mid = Math.floor((l + r)/2);
    return mergeSort(arr, l, mid)
            + mergeSort(arr, mid + 1, r)
            + merge(arr, l, mid, r)
}

function merge(arr, l, mid, r) {
    let leftIndex = l;
    let rightIndex = mid + 1;
    let helpArr = [];
    let sum = 0;
    while(leftIndex <= mid && rightIndex <= r) {
        let leftItem = arr[leftIndex];
        let rightItem = arr[rightIndex];
        if (leftItem < rightItem) {
            /**相对于归并排序增加的部分**/
            let tempSum = (r - rightIndex + 1) * leftItem
            sum += tempSum;
            /***************************/
            helpArr.push(leftItem);
            leftIndex++;
        } else {
            helpArr.push(rightItem);
            rightIndex++;
        }
    }

    /**这部分和归并排序merge函数一样**/

    return sum;
}
```

对于小和都知道如何使用归并排序进行求解之后，逆序对其实和小和是一样的，只是反过来了而已，以下直接贴出代码。
```javascript
function inversePairs(arr) {
    if (!arr || arr.length < 2) {
        return [];
    }
    return mergeSort(arr, 0, arr.length - 1);
}

function mergeSort(arr, l, r) {
    if (l === r) {
        return [];
    }
    let mid = Math.floor((l + r)/2);
    return [
        ...mergeSort(arr, l, mid),
        ...mergeSort(arr, mid + 1, r),
        ...merge(arr, l, mid, r)
    ];
}

function merge(arr, l, mid, r) {
    let leftIndex = l;
    let rightIndex = mid + 1;
    let helpArr = [];
    let res = [];
    while(leftIndex <= mid && rightIndex <= r) {
        let leftItem = arr[leftIndex];
        let rightItem = arr[rightIndex];
        if (leftItem < rightItem) {
            helpArr.push(leftItem);
            leftIndex++;
        } else {
            /**相对于归并排序增加的部分**/
            res.push([leftItem, rightItem]);
            /***************************/
            helpArr.push(rightItem);
            rightIndex++;
        }
    }

    /**这部分和归并排序merge函数一样**/

    return res;
}
```

以上是对归并排序这部分内容进行的一些回顾和总结，希望能加深自己对它的理解，能在其他更多的地方将其运用上；如果有不正确的地方，大家可以踊跃指出，我将及时改正。