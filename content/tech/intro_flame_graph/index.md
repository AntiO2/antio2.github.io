---
title: "火焰图介绍"
description: 
date: 2023-12-17T17:37:17+08:00
image: https://antio2-1258695065.cos.ap-chengdu.myqcloud.com/img/blogperf.svg?x=782.9&y=341
math: 
license: 
hidden: false
comments: true
draft: false
tags:
categories: 
  - 杂谈
---

## 0x1 准备工作

### 安装perf

我用的WSL2，perf折腾了一会。

使用命令`sudo apt-get install linux-source`下载对应内核源码到`/usr/src/linux-source-*.tar.bz2`中并解压

然后在`/usr/src/linux-source-5.15.0/tools/perf`中使用make编译安装perf

### 下载火焰图工具

```shell
git clone git@github.com:brendangregg/FlameGraph.git
```

### 测试程序

这里我随便写了个递归调用的程序。主要的系统调用是一些文件操作。

```cpp
/**
 * @author Anti
 * @date 2023-12-16
 */

#include <filesystem>
#include <fstream>
#include <iostream>
#include <random>
#include <thread>
#include <vector>
#include <iostream>
#include <cstdlib>
#include <unistd.h>
#include <fcntl.h>
#include <sys/types.h>
#include <sys/stat.h>
#include "fmt/core.h"
#include "gtest/gtest.h"
#include "logger.h"
void C(int depth,int max) {
  if(depth>=max) {
    return ;
  }
  for(int i = 0; i < 100;i++) {
    int n = std::rand()%50+10;
    for(int i = 0 ; i< n;i++) {
      int fileDescriptor = open("example.txt", O_WRONLY | O_CREAT, S_IRUSR | S_IWUSR);
      if (fileDescriptor == -1) {
        std::cerr << "无法打开文件！" << std::endl;
        return;
      }
      const char *message = "Perf Lab";
      ssize_t bytesWritten = write(fileDescriptor, message, strlen(message));
      if (bytesWritten == -1) {
        std::cerr << "写入文件时出错！" << std::endl;
        return;
      }

      fsync(fileDescriptor);

      // 关闭文件
      int closeResult = close(fileDescriptor);
      if (closeResult == -1) {
        std::cerr << "关闭文件时出错！" << std::endl;
        return;
      }
    }
  }
  C(depth+1,max);
}
void B() {
  int n = std::rand()%100+50;
  std::cout<<"Thread "<<std::this_thread::get_id()<< "Launch "<<n <<"Workers"<<std::endl;
  for(int i = 0; i < n;i++) {
    C(0,i);
  }
  }
class Runner {
 private:

  void A() {
    const int n = 10;
    std::vector<std::unique_ptr<std::thread>> tl;
    LOG_INFO("Launch %d thread",n);
    for(int i = 0; i < n; i++) {
      tl.emplace_back(std::make_unique<std::thread>(B));
    }
    for(auto&&t :tl) {
      t->join();
    }
  }
 public:
  void run() {
    A();
  }
};
TEST(PERF,T1) {
  Runner r;
  r.run();
}
```

## 0x2 生成火焰图

[CPU Flame Graphs (brendangregg.com)](https://www.brendangregg.com/FlameGraphs/cpuflamegraphs.html#Instructions)

1. 首先运行测试程序

`sudo perf record -F 100 -g  ./perf_test`

- `-F 100` 表示采样频率为100hz。也就是说每秒采样100次CPU的调用并记录相应的信息。
- `-g` 表示记录堆栈

完成后，生成`perf.data`文件,将该文件拷贝至`~`

2. 生成火焰图

`sudo perf script | FlameGraph/stackcollapse-perf.pl > out.perf-folded`生成折叠后的调用栈。这个的用处是将每次调用时，相同的调用栈给合并起来，相当于对原始数据的一次处理。



` sudo FlameGraph/flamegraph.pl out.perf-folded > perf.svg` 生成火焰图。

然后使用浏览器打开这个火焰图

![antio2-1258695065.cos.ap-chengdu.myqcloud.com/img/blogperf.svg](https://antio2-1258695065.cos.ap-chengdu.myqcloud.com/img/blogperf.svg)

**Click: https://antio2-1258695065.cos.ap-chengdu.myqcloud.com/img/blogperf.svg**
（可以使用鼠标在上面点击查看不同的调用栈）

## 0x2 分析火焰图

按照上面的火焰图来说，可以查找平顶函数。一般是从上往下找到第一个系统调用的函数。这里的瓶颈就是`do_fsync`



**Click: https://antio2-1258695065.cos.ap-chengdu.myqcloud.com/img/blogperf.svg?x=782.9&y=341**
![image-20231217002444887](https://antio2-1258695065.cos.ap-chengdu.myqcloud.com/img/blogimage-20231217002444887.png)

比如这个截图里面，明显`syscall`中，占用时间`sys_fsync>ksys_write>sys_close`。`fsync`操作可能是性能瓶颈

---

暂时写这么多，想到再写。
