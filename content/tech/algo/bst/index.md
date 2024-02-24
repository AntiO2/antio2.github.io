---
title: "算法导论|第12章 二叉搜索树"
description: 
date: 2023-10-28T18:47:10+08:00
image: 
math: 
license: 
hidden: false
comments: true
draft: false
tags:
  - 算法导论
categories:
  - 算法
---



## 二叉搜索树.begin()

**定义：**

对于树中

- 对于任何节点x

  - x的左子树中的所有节点的key值，不大于x。
  - x的右子树中的所有key不小于`x.key`

  

**性质**

- 二叉搜索树中序遍历是有序的。

## 二叉搜索树的操作

**查找**

要查找关键字k

```c++
x = root
while x!=NIL and k != x.key {
	if k < x.key
        x = x.left
    else
        x = x.right
}
return x
```



**MAX和MIN**

分别沿着左右子树寻找



**后继和前驱**



按照中序遍历的顺序，一个结点的后继寻找：

- 如果右子树不为空，返回右子树中最小的。
- 否则向父辈寻找。
  - 直到找到作为父结点左子树根的结点。返回该根的值。

```c++
if x.right!=nullptr
    return tree-min(x.right)
auto y = x.p
while y!=nullptr&&y.right ==x
    x = y
    y = y.p
return y
```

如果最后y是nullptr,说明x一直在右子树中。没有后继结点。

相同的，要寻找前驱

```c++
if x.left!=nullptr
    return tree-max(x.left)
// 如果没有左子树
auto y = x.p
while y!=nullptr&&y.left ==x
    x = y
    y = y.p
return y
```

## 插入和删除

```C++
/**
 * 
 * @param t 要插入的新值
 */
void insert(T t) requires std::totally_ordered<T> {
  TreeNode<T>* node = new TreeNode{t};  // 要插入的新结点
  TreeNode<T>* y = nullptr;
  TreeNode<T>* x = root_;  // x是node应当插入的位置。
  while (x != nullptr) {
    y = x;
    if (x->val < t) {
      x = x->right;
    } else {
      x = x->left;
    }
  }
  node->parent = y;
  if (node->val < y->val) {
    y->left = node;
  } else {
    y->right = node;
  }
}
```

插入新的叶结点。



删除结点z分三种情况

- 若z为叶子结点，直接删除，并修改父结点对应的子结点为nullptr
- 若z只有一个孩子，则提升该孩子。
- 若z有两个孩子，找到z的后继y（因为保证有两个孩子，所以后继一定是右子树中的最小结点）。让y代替z的位置。

```C++
 /**
  * 让v代替u的位置
  * @param u
  * @param v
  */
 void transplant(TreeNode<T>* u, TreeNode<T>* v) {
   if (u->parent == nullptr) {
     // u本来是根节点的情况。
     root_ = v;
   } else if (u == u->parent->left) {
     u->parent->left = v;
   } else {
     u->parent->right = v;
   }
   if (v != nullptr) {
     v->parent = u;
   }
 }

public:
 TreeNode<T>* min_of_tree(TreeNode<T>* n) {
   if (n == nullptr) {
     return nullptr;
   }
   while (n->left != nullptr) {
     n = n->left;
   }
   return n;
 }
 void delete_node(TreeNode<T>* n) {
   if (n == nullptr) {
     return;
   }
   if (n->left == nullptr) {
     transplant(n, n->right);  // 这里有两种情况：
     // 1. n.right 为nullptr
     // 2. n.right不为nullptr。实际上是一样的，
   } else {
     if (n->right == nullptr) {
       transplant(n, n->left);
     } else {
       auto y = min_of_tree(n->right);  // 找到后继结点。
       if (y->parent != n) {
         // 感觉这里不判断也行，没有影响。
         transplant(y, y->right);
         y->right = n->right;
         y->right->parent = y;
       }
       transplant(n, y);
       y->left = n->left;
       y->left->parent = y;
     }
   }
 }
```