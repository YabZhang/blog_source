---
title: '[Forward]Things About Databases'
date: 2020-05-13 04:12:55
tags:
---

Origin: [Article Link](https://medium.com/@rakyll/things-i-wished-more-developers-knew-about-databases-2d0178464f78)


## Notes

1. *You are lucky if 99.999% of the time network is not a problem.*

	> Networking is unreliable!

2. ACID has many meanings

	ACID: atomicity, consistency, isolation, durability | 事务的特性
	ACID 是一个宽泛的描述和原则，并不是一个严格声明的标准和规范。很多自称“ACID”的数据库实际上是有不同的解释，比如 `MongoDB` 事务。

3. Each database has diff consistency and isolation capabilities

	* Serializable (most strict, expensive)：严格线性
	* Repeatable reads: 其他事务的提交变动，在当前未提交事务中不可见;
	* Read committed: 当前事务可以看到其他事务的提交改动；可能会发生幻读（phantom read）
	* Read uncommitted: 允许脏读(可以看到其他事务的未提交部分)

4. *Opimistic locking is an option when you can’t hold a lock*

	optimistic and pessimistic locking with SQL
	https://convincedcoder.com/2018/09/01/Optimistic-pessimistic-locking-sql/#optimistic-locking-using-where

	```sql
	# mysql 悲观锁
	start transaction;
	select … for update;  # 排他写锁
	select … lock in share mode;  # 共享读锁
	commit;
	```

5. There are anomalies other than dirty reads and data loss.

	e.g: write skew

6. My database and I don’t always agree on ordering

	数据逻辑依据其“实际”到达DB的顺序。

7. Application-level sharding can live outside the application

	应用层水平拓展，解耦出来的“分片层”，或者中间件，更加灵活；
	例如：vitess（mysql: https://vitess.io/, 还有mycat也可以使用)
	https://www.youtube.com/watch?v=OCS45iy5v1M&feature=youtu.be&t=204

8. AUTOINCREMTN can be harmful

	*  分布式系统中维护自增ID是个问题, UUID会是更好的选择;
	* 顺序自增ID可能导致分区热点访问问题（集中访问热点分片而没有打散）
	* 若字段唯一（例如唯一的用户名字段），可能是更好的选择，便于更方便的获取数据

9. Stale data can be useful and lock-free.

	近期的非实时数据其实也可以满足需要的，也会减少对资源消耗；

10. Clock skews happen between any clock sources.

	任何不同的时钟源都可能有偏差（时钟同步信息传输有耗时）。

11. Latency has many meanings.

	Network latency vs Database latency

12. Evaluate performance requirements per transaction.
13. Nested transactions can be harmful.

	嵌套事务可能导致数据异常问题。你可以使用同一个事务完成所有操作。

14. Transactions shouldn’t maintain application state.
15. Query planners can tell about databases.

	查询计划决定全表扫描 和 索引扫描

16. Online migrations are complex but possible.

	线上热切换流程： http://www.aviransplace.com/2015/12/15/safe-database-migration-pattern-without-downtime/#ixzz3vsEunxmA
	step1: 创建新的数据库表（生产环境）
	step2: 在应用代码中添加新的DAO代码(可用于访问新的数据库)
	step3: 开始对新旧库“做双写”
	step4: 开始从新数据库读（以旧库为主，尽可能用新库)，维持一段时间运行
	step5: 切换到以新库为主（首先写新库，仍然读双库）
	step6: 暂停写旧库（仍读双库）
	step7: 迁移旧库数据到新库（旧库此时为只读库）
	step8: 删除应用中旧的DAO代码

	Stripe 做online-migration https://stripe.com/blog/online-migrations
	技术难点：
        1. 变动规模大；
		2. 业务影响；
		3. 设计多套代码和系统, 影响服务准确性

    迁移模式：
        1. 双写同步新旧库数据（原有旧数据需拷贝）
		2. 切换到读新库
		3. 切换只写新库
		4. 移除旧数据的访问途径

17. Significant database growth introduces unpredictability.

	显著的数据增长会引入不确定性。stay foolish
