---
date: 2025-12-16
title: 内核编程中的rhashtable
category: blog
tags:
- c
- linux
- rhashtable
- kernel
description:  内核开发中的rhashtable使用方法。
---

> 最近在搞内核模块开发，在写一个简单的ACL匹配规则，适用于/32和/128的ip匹配。因此倒腾了一下rhashtable。

## rhashtable是什么

`rhashtable` 是 **Linux 内核中的可伸缩并发哈希表实现**，全称是 *Resizable Hash Table*，有几个主要特点：

- **高并发**
- **读多写少**
- **需要动态扩容**



*源码详见*

```c
/**
 * struct rhashtable - Hash table handle
 * @tbl: Bucket table
 * @key_len: Key length for hashfn
 * @max_elems: Maximum number of elements in table
 * @p: Configuration parameters
 * @rhlist: True if this is an rhltable
 * @run_work: Deferred worker to expand/shrink asynchronously
 * @mutex: Mutex to protect current/future table swapping
 * @lock: Spin lock to protect walker list
 * @nelems: Number of elements in table
 */
struct rhashtable {
	struct bucket_table __rcu	*tbl;
	unsigned int			key_len;
	unsigned int			max_elems;
	struct rhashtable_params	p;
	bool				rhlist;
	struct work_struct		run_work;
	struct mutex                    mutex;
	spinlock_t			lock;
	atomic_t			nelems;
};
```

主要由3个文件组成。

1. https://elixir.bootlin.com/linux/v5.10.246/source/include/linux/rhashtable-types.h
2. https://elixir.bootlin.com/linux/v5.10.246/source/include/linux/rhashtable.h
3. https://elixir.bootlin.com/linux/v5.10.246/source/lib/rhashtable.c

## 初始化

## 查找

`rhashtable` 的key查找原理是RCU 保护下，通过 key → hash → bucket → 链表遍历完成无锁查找，并在重哈希期间保证一致性。

`rhashtable` 并不存独立 key，key 将作为对象结构体中的一部分进行保存。

查找时对key的定位可以理解为

```c
obj_key = (void *)obj + key_offset;
memcmp(obj_key, key, key_len);// 如果结构体有padding进行内存对齐，千万要置零！！
```



## 添加

## 删除