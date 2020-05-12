---
title: '[Forward]MySQL Sharding at Quora'
date: 2020-05-13 04:29:22
tags:
---

Origin: [Article Link](https://www.quora.com/q/quoraengineering/MySQL-sharding-at-Quora)

## Notes

1. Vertical sharding

	把一些并发压力的数据表独立到新的数据库实例；

	ZK 保存数据表的主从分片位置，便于热切换；
	使用 mysqldump + binlog 做数据迁移和同步；

2. Horizontal Sharding

	* build vs buy
	* logical db level vs tabel level
	* range-based vs hash-based
	* manging metadata of shards: ZK
	* shardng column for a table: 根据查询效率评估
