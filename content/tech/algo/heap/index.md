---
title: "堆排序、STL源码阅读"
description: 
date: 2023-10-30T18:02:33+08:00
image: 
math: 
license: 
hidden: false
comments: true
draft: false
tags:
  - C++
  - 算法
  - 堆
categories:	
  - 算法
---

CLRS的伪代码和STL实现有些出入。



堆是通过数组容器实现。但是可以看成一个近似的完全二叉树。这使得堆具有空间原址性。只需要常数个元素空间存储临时数据。

> - Parent(i) = i/2 = i >> 1
> - Left-child(i) = 2i = i << 1
> - Right-child(i) = 2i + 1 = i << 1 | 1

这里可以通过内联函数实现。

[【精选】【C++】 内联函数详解（搞清内联的本质及用法）_c++内联函数_赵大宝字的博客-CSDN博客](https://blog.csdn.net/qq_35902025/article/details/127912415)

## MAX-HEAPIFY

维护堆的性质，将其下推。

这里我参考着C++的priority_queue看的。

在优先队列中，初始化会调用make_heap函数，然后一直调用__adjust_heap函数。

```C++
     const _DistanceType __len = __last - __first;
     _DistanceType __parent = (__len - 2) / 2;
     while (true)
{
  _ValueType __value = _GLIBCXX_MOVE(*(__first + __parent));
  std::__adjust_heap(__first, __parent, __len, _GLIBCXX_MOVE(__value),
                   __comp);
  if (__parent == 0)
    return;
  __parent--;
}
```

这里的`__parent`应该是最后一个未调整的父结点的索引值。

通过`__first + __parent`可以获取容器中对应结点的迭代器。

书上的伪代码意思是：

- 传入数组A和要调整的根节点i。
- 如果i没有子结点，或者比自己的子结点都大，结束递归。
- 否则选出子结点中较大的一个，记为largest
- 交换A[i]和A[largest]的值
- 递归调用MAX-HEAPIFY(A,largest)。

但是STL里面并没有直接SWAP。

而是在传入之间。用了`_ValueType __value = _GLIBCXX_MOVE(*(__first + __parent));`,将调整根节点的值通过`std::move()`移出，然后把被移出的这个位置叫做hole。

```C++
__adjust_heap(_RandomAccessIterator __first, _Distance __holeIndex,
_Distance __len, _Tp __value, _Compare __comp)
{
  const _Distance __topIndex = __holeIndex;
  _Distance __secondChild = __holeIndex;
```

但是STL中用的迭代的方式。

```C++
     while (__secondChild < (__len - 1) / 2)
{
  __secondChild = 2 * (__secondChild + 1);
  if (__comp(__first + __secondChild,
            __first + (__secondChild - 1)))
    __secondChild--;
  *(__first + __holeIndex) = _GLIBCXX_MOVE(*(__first + __secondChild));
  __holeIndex = __secondChild;
}
```

这里while的条件是，当前的根节点有两个子结点，然后分别判断哪个子结点小，将小的结点移动到hole上。然后将移动的索引值为新的hole。

最后如果还有单一的左结点的话。将左节点移动到Hole上。

```C++
     if ((__len & 1) == 0 && __secondChild == (__len - 2) / 2)
{
  __secondChild = 2 * (__secondChild + 1);
  *(__first + __holeIndex) = _GLIBCXX_MOVE(*(__first
                                        + (__secondChild - 1)));
  __holeIndex = __secondChild - 1;
}
```

一开始我还在想：这怎么没有父结点和子结点的比较过程呢？想了一下才反应过来，Hole的位置是没有值的，所以直接将`Comp(second_child, second_child-1)==true`的值放到hole上就行了。

最后，除了Value（一开始挖的洞）都已经满足堆的性质了，再将Value添加到堆中。也就是

```C++
std::__push_heap(__first, __holeIndex, __topIndex,
   _GLIBCXX_MOVE(__value), __cmp);
```

其中，topIndex是这个函数一开始的根，没有变过，holeIndex经过多次迭代，应该是最后一个叶结点的位置了。

 这个过程的时间复杂度取决于“树”的高度。

```C++
 template<typename _RandomAccessIterator, typename _Distance, typename _Tp,
   typename _Compare>
   _GLIBCXX20_CONSTEXPR
   void
   __push_heap(_RandomAccessIterator __first,
       _Distance __holeIndex, _Distance __topIndex, _Tp __value,
       _Compare& __comp)
   {
     _Distance __parent = (__holeIndex - 1) / 2;
     while (__holeIndex > __topIndex && __comp(__first + __parent, __value))
{
  *(__first + __holeIndex) = _GLIBCXX_MOVE(*(__first + __parent));
  __holeIndex = __parent;
  __parent = (__holeIndex - 1) / 2;
}
     *(__first + __holeIndex) = _GLIBCXX_MOVE(__value);
   }
```

push Heap就是从hole开始，然后往上迭代，比如如果父节点较小（大根堆的情况），就将父节点move到hole上，此时父结点变为hole。类似于一个值向上冒泡的过程，直到遇见top或者，父节点比它大。

## 建堆

也即是之前提到的，`std::__make_heap(__first, __last, __comp);`

这节通过循环不变式证明了建堆能够使整个堆满足堆的性质。

时间复杂度求和的计算有点看不懂。微积分感觉忘完了。。。😓

建堆的时间复杂度为O(n), 我的直观理解是：虽然一次adjust heap的时间复杂度为O(h)，也即是O(logn)。但是多次连续adjust heap不是印象中的O(nlogn),因为高度较低的根数量要少一些。

比如一共有10层（从根到叶我分别叫做1-10层）。然后第9层的h=2，一共有$2^9$ 个结点。第2层的h=9，一共有2个结点。因为是求的$\sum\limits_{h=0}^{lgn}\frac{n}{2^{h+1}}O(h)$, 最后算出来是O(n)

## 堆排序

如何通过堆来获得一个有序的数组，且不使用新的空间呢。

观察到堆顶总是最大的。

假如现在数组有1-i的元素。将对根和i交换，heapify (1~(i-1)).重复这个过程，得到从小到大排列的元素。

```C++
   __pop_heap(_RandomAccessIterator __first, _RandomAccessIterator __last,
       _RandomAccessIterator __result, _Compare& __comp)
   {
     _ValueType __value = _GLIBCXX_MOVE(*__result);
     *__result = _GLIBCXX_MOVE(*__first);
     std::__adjust_heap(__first, _DistanceType(0),
               _DistanceType(__last - __first),
               _GLIBCXX_MOVE(__value), __comp);
   }
```

比如在pop_heap的实现中，先将result暂存到value中，然后将堆顶值存到result中。之后将该Value加入堆中，并堆化。

```C++
 void  __sort_heap(_RandomAccessIterator __first, _RandomAccessIterator __last,
       _Compare& __comp)
   {
     while (__last - __first > 1)
{
  --__last;
  std::__pop_heap(__first, __last, __last, __comp);
}
   }
```

sort_heap就是不停地将

- 堆顶的元素pop到容器末尾（假设为10）
- 原本的堆顶为hole
- 调用adjust heap，维护堆，并将hole移动到新的末尾（这里就为9）
- 将value push_heap。value尝试向上冒泡。

## 优先队列

就是通过之前讲的`pop_heap`和`push_heap`实现的。

```c++
     void
     pop()
     {
__glibcxx_requires_nonempty();
std::pop_heap(c.begin(), c.end(), comp);
c.pop_back();
     }
```

```C++
     void
     push(value_type&& __x)
     {
c.push_back(std::move(__x));
std::push_heap(c.begin(), c.end(), comp);
     }
```