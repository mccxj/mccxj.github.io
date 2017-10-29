---
layout: post
comments: true
title: "tidb源码学习之auto_increment"
description: "对tidb中auto_increment实现的研究"
categories: ["tidb", "auto_increment", "源码"]
---

### 自增ID

TiDB 的自增 ID (Auto Increment ID) 只保证自增且唯一，并不保证连续分配。
TiDB 目前采用批量分配的方式，所以如果在多台TiDB上同时插入数据，分配的自增 ID 会不连续。

https://github.com/pingcap/tidb/tree/master/meta/autoid/autoid.go

### 基本接口

```
// Allocator is an auto increment id generator.
// Just keep id unique actually.
type Allocator interface {
	// Alloc allocs the next autoID for table with tableID.
	// It gets a batch of autoIDs at a time. So it does not need to access storage for each call.
	Alloc(tableID int64) (int64, error)
	// Rebase rebases the autoID base for table with tableID and the new base value.
	// If allocIDs is true, it will allocate some IDs and save to the cache.
	// If allocIDs is false, it will not allocate IDs.
	Rebase(tableID, newBase int64, allocIDs bool) error
}
```

接口很简单，一个是根据tableID分配一个自增ID，而且分配是批量进行的，所以还有另一个是用来重新设置批量获取的起始点。

```
type allocator struct {
	mu    sync.Mutex
	base  int64
	end   int64
	store kv.Storage
	dbID  int64
}
```

从实现类的结构可以看出实现的基本思路。mu是用于并发分配操作的锁，base表示最后一个已分配的ID，end表示最后一个未分配的ID。
当base和end相等的时候，就表示当前自增ID已经使用完，需要重新发起一次批量申请了。store表示后端的存储引擎，dbID表示表所在的数据库。

```
		err := kv.RunInNewTxn(alloc.store, true, func(txn kv.Transaction) error {
			m := meta.NewMeta(txn)
			var err1 error
			newBase, err1 = m.GetAutoTableID(alloc.dbID, tableID)
			if err1 != nil {
				return errors.Trace(err1)
			}
			newEnd, err1 = m.GenAutoTableID(alloc.dbID, tableID, step)
			if err1 != nil {
				return errors.Trace(err1)
			}
			return nil
		})
```

至于如何从store获取到当前的未分配的自增ID,首先通过GetAutoTableID获取到当前最后一个已分配的ID，然后通过GenAutoTableID获取一个批次的ID。
