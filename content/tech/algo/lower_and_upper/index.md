---
title: "lower bound与upper bound的C++实现"
description: 
date: 2020-11-17T17:53:59+08:00
image: 
math: 
license: 
hidden: false
comments: true
draft: false
tags:
  - 算法
  - C++
categories:
  - 算法
---

## 定义：

如果有一串序列，总是满足当`i < j`, 有`func(a[j],a[i])==false`。

lower bound是第一个可以插入且保持序列性质的位置。

upper bound是最后一个插入且保持序列性质的位置。

比如有序列`1,3,3,4,5`,此时`func = less<int>{}`, 总是满足后一个数不小于前一个数。

下标1是第一个可以插入3的位置，下标3是最后一个可以插入3的位置。

## lower bound

如果在默认情况下，查找第一个大于等于target的数。

也就是说，要查找到第一个满足`target <= *iter`的`iter`

但是实际上，我们的compare通常只有小于操作而没有小于等于。

也就是说，要找到最后一个不满足`target<=*iter`的iter的下一个。也就是要求满足`*iter < target`的最后一个的下一个。等于满足`cmp(*iter,target)==true`的最后一个的下一个。

当

- `cmp(*mid,target)==true`, 对任意mid之前的，都是true,对于任意mid之后的，可能是true,可能是false。因为mid不包含答案，故`left=mid+1`
- `cmp(*mid,target)==false`, 对任意mid之前的，可能是false，也可能是true。对于任意mid之后的，答案一定为false。因为mid可能被包含，故`right = mid`

然后仿照STL实现了一下

```cpp
template <typename _ForwardIterator, typename _Tp, typename _Compare>
_ForwardIterator anti_lower_bound(_ForwardIterator first, _ForwardIterator last,
                                  const _Tp& __val, _Compare __comp) {
  // 检查概念
  // concept requirements
  __glibcxx_function_requires(_ForwardIteratorConcept<_ForwardIterator>)
      __glibcxx_function_requires(
          _BinaryPredicateConcept<
              _Compare, _Tp,
              typename iterator_traits<_ForwardIterator>::value_type>) auto
          len = std::distance(first, last);
  auto begin = first;
  while (len > 0) {

    auto half = len / 2;
    auto mid = first;

    std::advance(mid, half);  // 获得中心点位置。
    LOG_INFO("%s",
             fmt::format("l:{} r: {} mid: {}", std::distance(begin, first),
                         std::distance(begin, first + len),
                         std::distance(begin, mid))
                 .c_str());
    if (__comp(*mid, __val)) {
      first = mid + 1;
      len = len - half - 1;
    } else {
      len = half;
    }
  }
  return first;
}

TEST(LOWER_BOUND, S1) {
  std::vector nums{1, 3, 5, 7, 9, 9, 11};
  auto iter = anti_lower_bound(nums.begin(), nums.end(), 8, std::less<int>{});
  LOG_INFO("%d %lld", *iter, std::distance(nums.begin(), iter));
}
```

## upper bound

找到第一个大于target的。也就是，找到第一个`cmp(val,*iter)==true` 的iter

- 对于`cmp(val,*mid)==true` ， 对于之前的，可能是true,也可能是false。且mid需要保留。对于之后的，全都是true，不需要考虑。
- 对于`cmp(val,*mid)==false` , 对于mid之前的元素，只有可能是false，对于mid之后的，可能是false或true。所以mid不需要保留

```cpp
TEST(LOWER_BOUND, S1) {
  std::vector nums{1, 3, 5, 7, 9, 9, 11};
  auto iter = anti_lower_bound(nums.begin(), nums.end(), 8, std::less<int>{});
  LOG_INFO("%d %lld", *iter, std::distance(nums.begin(), iter));
}

template <typename _ForwardIterator, typename _Tp, typename _Compare>
_ForwardIterator anti_upper_bound(_ForwardIterator first, _ForwardIterator last,
                                  const _Tp& __val, _Compare __comp) {
  // 检查概念
  // concept requirements
  __glibcxx_function_requires(_ForwardIteratorConcept<_ForwardIterator>)
      __glibcxx_function_requires(
          _BinaryPredicateConcept<
              _Compare, _Tp,
              typename iterator_traits<_ForwardIterator>::value_type>) auto
          len = std::distance(first, last);
  auto begin = first;
  while (len > 0) {

    auto half = len / 2;
    auto mid = first;

    std::advance(mid, half);  // 获得中心点位置。
    LOG_INFO("%s",
             fmt::format("l:{} r: {} mid: {}", std::distance(begin, first),
                         std::distance(begin, first + len),
                         std::distance(begin, mid))
                 .c_str());
    if (__comp(__val, *mid)) {
      len = half;
    } else {
      first = mid + 1;
      len = len - half - 1;
    }
  }
  return first;
}

TEST(UPPER_BOUND, S1) {
  std::vector nums{1, 3, 5, 7, 9, 9, 11};
  auto iter = anti_upper_bound(nums.begin(), nums.end(), 9, std::less<int>{});
  LOG_INFO("%d %lld", *iter, std::distance(nums.begin(), iter));
}
```