---
title: "DB|可串行化加锁规则"
description: 本文简要介绍基于锁的serialize数据库事务隔离级别的实现
date: 2023-06-20T21:54:53+08:00
image: 
math: 
license: 
hidden: false
comments: true
draft: false
tags:
  - Database
categories:
  - 数据库
---

## 2PL封锁协议

锁升级：从共享锁升级为排他锁。发生在共享阶段。

锁降级：从排他锁降级为共享锁。发生在缩减阶段。

一个事务通过lock-S(Q)获得Q上的共享锁，通过lock-X(Q)获得Q上的排他锁

一个简单可串行化的实现：

- 当要read(Q)时，产生lock-S(Q)指令。
- 当要write(Q)时,检查是否有S锁，如果有，进行upgrade操作。删除该锁，然后加到队列等待头部。否则进行lock-X(Q)
- 事务提交或中止后，释放所有锁。

## No-wait策略

当有一个新的加锁请求时，如果不兼容，那么直接abort该事务。

比如：事务T1已经在I表上有了S锁。此时事务T2申请S锁，可以直接授予。若申请X锁，直接abort。

## 具体实现

### 加锁

对于可串行化事务，首先需要判断是否在收缩阶段加锁

```C++
if(txn->get_state()==TransactionState::SHRINKING) {
  txn->set_state(TransactionState::ABORTED);
  // 如果在收缩阶段加锁，抛出异常
  throw TransactionAbortException(txn->get_transaction_id(),AbortReason::LOCK_ON_SHRINKING);
}
```

然后在lock_manager中寻找到需要加锁项的lock request queue，遍历该请求队列。

如果有其他事务拿到了该锁，并且锁不兼容，直接abort掉该事务

```C++
if(request->txn_id_ != txn_id &&!CheckCompatible(request->lock_mode_,lock_mode)) {
  // 采用no-wait策略
  txn->set_state(TransactionState::ABORTED);
  lock_request_queue->latch_.unlock();
  throw TransactionAbortException(txn->get_transaction_id(),AbortReason::DEADLOCK_PREVENTION);
}
```

如果该事务（通过txn_id相同判断）已经有了被授予的锁，判断是否是同种类型的锁，如果不是，做出升级请求。

如果请求队列没有该事务，则新增一个请求。

接下来就是判断在什么时候加锁。

对于一个加锁对象的请求队列来说，比如已经有了五项已经授予锁的请求，第六项要被授予锁，必须和前五项是兼容的。

写一个伪代码如下

```C
for(request in queue):
	if(request is granted) // 之前被授予了锁的请求
		check 兼容性矩阵
    else  //遇到了第一个正在等待加锁的请求
    	return request.txn==txn_now // 如果在队列中第一个等待加锁的是当前事务，才能够加锁
```

### 解锁

解锁操作较为简单。

首先要通过解锁对象，找到锁表中对应的队列。然后在队列中，寻找到该事务的请求，如果是可串行化，还需要将事务从growing变为shrinking状态。

最后，从队列中移除该请求。并且从事务对象中移除该锁。

如果队列中没有找到该请求，抛出异常。