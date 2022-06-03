@[TOC](目录)

对于所有具体的相关操作请自行查看官方操作手册。
# 闪回
## 回收站
OceanBase提供回收站，主要用于存储用户删除的数据库和表等信息。

在当前版本中，支持进入回收站的对象有索引、表、库和租户，各模式回收站对象的支持情况如下表所示。
模式 | 索引（Index）| 表（T able）| 数据库（Dat abase）| 租户（Tenant ）
-- | -- | -- | -- | --
MySQL | 支持 | 支持 | 支持 | 支持
Oracle | 支持 | 支持 | 不支持 | 不支持
> - 直接 DROP 索引不会进入回收站，删除表时，表上的索引会随主表一起进入回收站。
> - 不能对回收站的对象做任何查询和 DML 操作，DDL 操作中也仅支持 Purge 和 Flashback 操作。
## 闪回回收对象
使用限制：
- 在恢复表时，Flashback 对象的顺序需要符合从属关系，即先 Flashback 数据库，再 Flashback 表。
- 通过 FLASHBACK 语句恢复表时，表上的索引也会恢复。
- 通过 PURGE 命令可以单独清除索引，但是 FLASHBACK 命令不支持单独恢复索引。
- 如果一张表在进入回收站前属于某个表组，通过闪回恢复后该表默认会恢复到该表组中；如果表组已经被删除，则该表恢复后会不属于任何一个表组。
## Restore Point
在很多应用系统中，用户需要查询数据库中的某个时间点，或者特定版本的数据来完成一些数据分析或汇总之类的操作。

OceanBase 数据库在 V2.2.7x 版本中提供了 Restore Point 功能，允许用户在租户上创建 Restore Point，将历史版本的数据保存下来。Restore Point 功能类似于租户的快照点，可以通过闪回查询的方式来访问特定版本的历史数据。

当前 Restore Point 功能的使用限制如下：
- 不支持物理备份。
- 不支持主备库。
- 不支持在 sys 租户下创建 Restore Point。
- 每个租户内最多可创建 10 个 Restore Point 。
- 创建 Restore Point 后，如果对创建 Restore Point 前就存在的表执行 DDL 语句，对于 MySQL 模式，系统会报 ERROR 4179 (HY000): restore point exist, create index/alter not allowed 的错误；对于 Oracle 模式，系统会报 ORA-00600: internal error code, arguments: -4179, restore point exist, create index/alter not allowed 的错误。

由于 Restore Point 功能依赖 GTS 维护全局的一致性快照，故在使用 Restore Point 时，需要开启 GTS。
## 闪回查询多数据库版本
OceanBase 数据库提供了记录级别的闪回查询（Flashback Query）功能，该功能允许用户获取某个历史版本的数据。

其中，Oracle 模式支持 AS OF SCN 和 AS OF TIMESTAMP 两种语法来查询；MySQL 模式支持通过 AS OF SNAPSHOT 语法来查询。

# 备份恢复管理
