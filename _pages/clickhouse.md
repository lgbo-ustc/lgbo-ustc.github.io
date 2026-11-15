---
permalink: /clickhouse/
title: "ClickHouse 社区贡献"
layout: single
author_profile: false
share: false
comments: false
related: false
read_time: false
---

2021 年起参与 ClickHouse 开源社区，累计提交 80 个 PR，其中 58 个已合入，贡献集中在 Hive 外部表集成、Join 执行、窗口函数与查询执行优化等方向。

## Hive 外部表集成

- 新增 `hive` 表函数，配合 `StorageHive` 直接查询 Hive Metastore 管理的 HDFS/S3 表（#34946）
- 分区键与列裁剪：全为分区键时不做文件读取，扫描时只读所需列（#37103、#34954）
- Hive 文件缓存支持多磁盘，限制单表读取分区数（#37279、#37281）
- 支持 struct 类型读取及 Parquet/ORC 字段级裁剪（#38118、#44484）
- 支持对 `StorageHive` 表执行 ALTER（#38214）
- 引入 HiveMetastore 连接池，优化随机读 readbuffer（#34940、#34957）
- 修复 Snappy 多块解码越界、Hive 查询中 `IN` 误用、HDFS/S3 集群 join 列名歧义、`IHiveFile` 跨线程共享崩溃等问题（#35116、#34945、#35748、#38887），并完善集成测试稳定性（#38260）

## Join 执行与 grace_hash

- 并发 HashJoin：右表构建拆为多实例并行，提升大右表 LEFT/INNER join 的 CPU 利用率（#36415）
- grace_hash 支持 RIGHT/FULL join，按需预分配哈希表并修复数据提前释放问题（#49483、#51013、#49816、#50875、#51382）
- 支持混合 join 条件（`in`、函数表达式），覆盖 ANY/SEMI/ANTI 及不等值 join（#64281、#64775、#60920）
- 引入查询级 spill 调度器，统一 spill 触发策略（当前用于 grace join，#72728）
- 修复 cross join 类型不匹配、full/right join 中 `JoinUsedFlags` 复用、`have_compressed` 信息丢失（#66413、#49585、#69404）
- 重构 HashJoin，拆分过大的编译单元（#64818）

## 窗口函数

- 新增 `ntile`、`percent_rank` 窗口函数（#46256、#62747）
- 扩展 `first_value`/`last_value` 聚合函数（#46467）
- 修复 lag/lead、percent_rank 崩溃（#68262、#67661）
- 降低部分窗口函数的内存占用（#65647）

## 查询执行与优化

- 修复短路执行在 `and`/`or` 中对非函数列误用 `maskedExecute`（#73965）
- 新增 `stringCompare` 字符串比较函数（#74070）
- Parquet 写入支持 page index，优化 Parquet/ORC struct 字段读取（#65475、#56117）
- JSONEachRow 在必需字段读完后跳过剩余字段（#62210）
- JSON 函数复用 JSONPath 生成器、将 JSON 元素直接序列化进列 buffer（#54735、#54613、#48768）

## 类型与函数修复

- 修复 `ColumnTuple::insertManyFrom` 未更新 `column_length`（#66626）
- 修复 `groupArray` 返回结果不应为 Nullable（#48593）
- 修复删除不存在列时的异常（#35393）

> 以上仅列已合入的 PR。
