---
title: "C++多态总结"
description: 
date: 2023-11-07T17:05:59+08:00
image: 
math: 
license: 
hidden: false
comments: true
draft: false
tags:
categories:
 - 杂谈
---

多态（Polymorphism） 多态是允许不同类的对象使用相同的接口名字，但具有不同实现的特性。

在 C++ 中，多态主要通过虚函数（Virtual Function）和抽象基类（Abstract Base Class）来实现。

虚函数允许在派生类中重写基类的方法，而抽象基类包含至少一个纯虚函数（Pure Virtual Function），不能被实例化，只能作为其他派生类的基类。

通过多态，我们可以编写更加通用、可扩展的代码，提高代码的灵活性。

### 函数签名

函数签名由函数名、参数列表构成，不包含返回值。

### **虚函数**

- **C++使用虚函数的时候，子类也要使用virtual关键字吗**

c++规定，当一个成员函数被声明为虚函数后，其派生类中的同名函数都自动成为虚函数。因此，在子类从新声明该虚函数时，可以加，也可以不加，但习惯上每一层声明函数时都加virtual,使程序更加清晰。

#### 对虚函数的探究。

写了一段代码。

```c++
class A {
 public:
  void m1() { LOG_INFO("A"); }
};

class B : public A {
 public:
  void m1() { LOG_INFO("B"); }
};

TEST(TEST_POLY, NOT_VIRTUAL) {
  B b;
  A* ap = &b;
  ap->m1();
  B* bp = &b;
  bp->m1();
}
```



在想如果不加virtual，不也能重写吗？

但是输出结果

```C++
A
B
```

但是

```cpp
class A {
 public:
  virtual void m1() { LOG_INFO("A"); }
};

class B : public A {
 public:
  void m1() override { LOG_INFO("B"); }
};

TEST(TEST_POLY, NOT_VIRTUAL) {
  B b;
  A* ap = &b;
  ap->m1();
  B* bp = &b;
  bp->m1();
}
```

输出

```
B
B
```

#### 多态的析构函数必须是虚函数

经过上面的分析，就理解这个了。

为A和B添加析构函数

```cpp
class A {
 public:
  virtual void m1() { LOG_INFO("A"); }
  ~A() { LOG_INFO("Delete A"); }
};

class B : public A {
 public:
  void m1() override { LOG_INFO("B"); }
  ~B() { LOG_INFO("Delete B"); }
};
```

然后创建两个B对象，分别用A和B的指针指向。



```c++
TEST(TEST_POLY, DESTORY) {
  A* ap = new B();
  delete ap;
  LOG_INFO("---");
  B* bp = new B();
  delete bp;
}
```

输出结果

```log
2023-11-06 17:33:48 [F:/LeetCode/solution/lab/polymorphism.cpp:13:~A] INFO  - Delete A
2023-11-06 17:33:48 [F:/LeetCode/solution/lab/polymorphism.cpp:33:TestBody] INFO  - ---
2023-11-06 17:33:48 [F:/LeetCode/solution/lab/polymorphism.cpp:19:~B] INFO  - Delete B
2023-11-06 17:33:48 [F:/LeetCode/solution/lab/polymorphism.cpp:13:~A] INFO  - Delete A
```

如果用A指针指向B，并且析构函数不是虚函数，析构只会调用A的析构函数,而不会调用这个类实际的析构函数`~B()`。而析构B时，会先调用B的析构函数，再析构基类（A）。

而如果析构函数为虚函数

```cpp
class A {
 public:
  virtual void m1() { LOG_INFO("A"); }
  virtual ~A() { LOG_INFO("Delete A"); }
};

class B : public A {
 public:
  void m1() override { LOG_INFO("B"); }
  ~B() override { LOG_INFO("Delete B"); }
};

TEST(TEST_POLY, DESTORY) {
  A* ap = new B();
  delete ap;
  LOG_INFO("---");
  B* bp = new B();
  delete bp;
}
```

输出

```
2023-11-06 17:35:27 [F:/LeetCode/solution/lab/polymorphism.cpp:19:~B] INFO  - Delete B
2023-11-06 17:35:27 [F:/LeetCode/solution/lab/polymorphism.cpp:13:~A] INFO  - Delete A
2023-11-06 17:35:27 [F:/LeetCode/solution/lab/polymorphism.cpp:33:TestBody] INFO  - ---
2023-11-06 17:35:27 [F:/LeetCode/solution/lab/polymorphism.cpp:19:~B] INFO  - Delete B
2023-11-06 17:35:27 [F:/LeetCode/solution/lab/polymorphism.cpp:13:~A] INFO  - Delete A
```

### 为什么要优先调用虚继承?

TODO

### 为什么不能用虚函数为模板函数

例如

```cpp
class C {
 public:
  template <typename T>
  virtual void func();
};
```

报错`'virtual' cannot be specified on member function templates`。（注意模板函数中是能用虚函数的，但是虚函数不能有模板）

```cpp
class C {
 public:
  template <typename T>
  virtual void func(){};

};
TEST(TEST_POLY, VIRTUAL) {
  C c;
  c.func<int>();
  c.func<long>();
}
```

比如上面的情况，因为虚函数表是一一对应的关系，

### 模板实例化

实例化函数：

```cpp
void foo(T value) {}
template void foo<int>(int);  // 实例化 foo<int>
```

实例化类：

```cpp
template <typename T>
class C0 {};
template class C0<int>;
```

实例化模板类中的模板函数：

```cpp
template <typename T>
class C {
 public:
  template <typename U>
  void foo(){};
};
// 显式实例化模板类中的模板函数
template void C<int>::foo<long>();
```



模板实例化都是`template`，然后跟限定符，参数列表。

比如

```cpp
template<class T1>
class Tmp1 {
  public:
  template<class T2>
  class Tmp2 {

  };
};
template class Tmp1<A>::Tmp2<B>;
```

`template class Tmp1<A>::Tmp2<B>;`中，第一个`class`指的是要实例化`Tmp2`这个类，这个类的标识符是` Tmp1<A>::Tmp2`,并在后面给出参数`B`

### 空类的大小

想到很多好玩的：

1. 空类的大小
2. 继承空类的空类大小
3. 继承空类，但是有成员变量的大小。
4. 继承空类且有一个虚函数的类大小
5. 空类，且有一个普通函数

```C++
class Empty {};
template <class E>
class Empty2 {
 public:
  E e;
};
class Empty4 : Empty {
  virtual void func(){};
};
TEST(TEST_EMPTY, SAMPLE) {
  Empty e1;
  LOG_INFO("%zu", sizeof(e1));
  Empty2<Empty> e2{};
  LOG_INFO("%zu", sizeof(e2));
  Empty2<int> e3{};
  LOG_INFO("%zu", sizeof(e3));
  Empty4 e4;
  LOG_INFO("%zu", sizeof(e4));
  Empty5 e5;
  LOG_INFO("%zu", sizeof(e5));
}
```

输出

```
1
1
4
8
1
```

为什么空类有大小呢？

假如内存栈上面有两个连续的空类，为了区分这两个类，不能让它们有相同的地址。

例如

```cpp
TEST(TEST_EMPTY, SAMPLE2) {
  Empty e1;
  LOG_INFO("%p", &e1);
  Empty e2;
  LOG_INFO("%p", &e2);
}
```

输出

```
2023-11-07 00:11:36 [F:/LeetCode/solution/lab/polymorphism.cpp:72:TestBody] INFO  - 0000000000cdf88f
2023-11-07 00:11:36 [F:/LeetCode/solution/lab/polymorphism.cpp:74:TestBody] INFO  - 0000000000cdf88e
```

可以看到，两个类实例在栈上且相差1（地址从高到低）

如果在heap上就不太一样。

```cpp
Empty e1;
Empty e2;
TEST(TEST_EMPTY, SAMPLE2) {
  LOG_INFO("%p", &e1);
  LOG_INFO("%p", &e2);
}

2023-11-07 00:13:35 [F:/LeetCode/solution/lab/polymorphism.cpp:72:TestBody] INFO  - 00000000005b6050
2023-11-07 00:13:35 [F:/LeetCode/solution/lab/polymorphism.cpp:73:TestBody] INFO  - 00000000005b6051
```

一个地址高，一个地址低。



参考[Empty Class（空类）的作用_python 空class-CSDN博客](https://blog.csdn.net/u014613043/article/details/51323808)， 空类可以很优雅地实现一些概念、约束。