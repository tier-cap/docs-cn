---
title: 系统变量
aliases: ['/docs-cn/dev/system-variables/','/docs-cn/dev/reference/configuration/tidb-server/mysql-variables/','/docs-cn/dev/tidb-specific-system-variables/','/docs-cn/dev/reference/configuration/tidb-server/tidb-specific-variables/','/zh/tidb/dev/tidb-specific-system-variables/']
---

# 系统变量

TiDB 系统变量的行为与 MySQL 相似，变量的作用范围可以是会话级别有效 (Session Scope) 或全局范围有效 (Global Scope)。其中：

- 对 `SESSION` 作用域变量的更改，设置后**只影响当前会话**。
- 对 `GLOBAL` 作用域变量的更改，设置后立即生效。如果该变量也有 `SESSION` 作用域，已经连接的所有会话 (包括当前会话) 将继续使用会话当前的 `SESSION` 变量值。
- 要设置变量值，可使用 [`SET` 语句](/sql-statements/sql-statement-set-variable.md)。

```sql
# 以下两个语句等价地改变一个 Session 变量
SET tidb_distsql_scan_concurrency = 10;
SET SESSION tidb_distsql_scan_concurrency = 10;

# 以下两个语句等价地改变一个 Global 变量
SET @@global.tidb_distsql_scan_concurrency = 10;
SET  GLOBAL tidb_distsql_scan_concurrency = 10;
```

> **注意：**
>
> 部分 `GLOBAL` 作用域的变量会持久化到 TiDB 集群中。文档中的变量有一个“是否持久化到集群”的说明，可以为“是”或者“否”。
> 
> - 对于持久化到集群的变量，当该全局变量被修改后，会通知所有 TiDB 服务器刷新其系统变量缓存。在集群中增加一个新的 TiDB 服务器时，或者重启现存的 TiDB 服务器时，都将自动使用该持久化变量。
> - 对于不持久化到集群的变量，对变量的修改只对当前连接的 TiDB 实例生效。如果需要保留设置过的值，需要在 `tidb.toml` 配置文件中声明。
> 
> 此外，由于应用和连接器通常需要读取 MySQL 变量，为了兼容这一需求，在 TiDB 中，部分 MySQL 的变量既可读取也可设置。例如，尽管 JDBC 连接器不依赖于查询缓存 (query cache) 的行为，但仍然可以读取和设置查询缓存。

> **注意：**
>
> 变量取较大值并不总会带来更好的性能。由于大部分变量对单个连接生效，设置变量时，还应考虑正在执行语句的并发连接数量。
>
> 确定安全值时，应考虑变量的单位：
>
> * 如果单位为线程，安全值通常取决于 CPU 核的数量。
> * 如果单位为字节，安全值通常小于系统内存的总量。
> * 如果单位为时间，单位可能为秒或毫秒。
>
> 单位相同的多个变量可能会争夺同一组资源。

## 变量参考

### `allow_auto_random_explicit_insert` <span class="version-mark">从 v4.0.3 版本开始引入</span>

- 作用域：SESSION | GLOBAL
- 是否持久化到集群：是
- 默认值：`OFF`
- 是否允许在 `INSERT` 语句中显式指定含有 `AUTO_RANDOM` 属性的列的值。

### `auto_increment_increment`

- 作用域：SESSION | GLOBAL
- 是否持久化到集群：是
- 默认值：`1`
- 范围：`[1, 65535]`
- 控制 `AUTO_INCREMENT` 自增值字段的自增步长。该变量常与 `auto_increment_offset` 一起使用。

### `auto_increment_offset`

- 作用域：SESSION | GLOBAL
- 是否持久化到集群：是
- 默认值：`1`
- 范围：`[1, 65535]`
- 控制 `AUTO_INCREMENT` 自增值字段的初始值。该变量常与 `auto_increment_increment` 一起使用。示例如下：

```sql
mysql> CREATE TABLE t1 (a int not null primary key auto_increment);
Query OK, 0 rows affected (0.10 sec)

mysql> set auto_increment_offset=1;
Query OK, 0 rows affected (0.00 sec)

mysql> set auto_increment_increment=3;
Query OK, 0 rows affected (0.00 sec)

mysql> INSERT INTO t1 VALUES (),(),(),();
Query OK, 4 rows affected (0.04 sec)
Records: 4  Duplicates: 0  Warnings: 0

mysql> SELECT * FROM t1;
+----+
| a  |
+----+
|  1 |
|  4 |
|  7 |
| 10 |
+----+
4 rows in set (0.00 sec)
```

### `autocommit`

- 作用域：SESSION | GLOBAL
- 是否持久化到集群：是
- 默认值：`ON`
- 用于设置在非显式事务时是否自动提交事务。更多信息，请参见[事务概述](/transaction-overview.md#自动提交)。

### character_set_client

- 作用域：SESSION | GLOBAL
- 是否持久化到集群：是
- 默认值：`utf8mb4`
- 这个变量表示从客户端发出的数据所用的字符集。有关更多 TiDB 支持的字符集和排序规则，参阅[字符集和排序规则](/character-set-and-collation.md)文档。如果需要更改字符集，建议使用 [`SET NAMES`](/sql-statements/sql-statement-set-names.md) 语句。

### character_set_connection

- 作用域：SESSION | GLOBAL
- 是否持久化到集群：是
- 默认值：`utf8mb4`
- 若没有为字符串常量指定字符集，该变量表示这些字符串常量所使用的字符集。

### character_set_database

- 作用域：SESSION | GLOBAL
- 是否持久化到集群：是
- 默认值：`utf8mb4`
- 该变量表示当前默认在用数据库的字符集，**不建议设置该变量**。选择新的默认数据库后，服务器会更改该变量的值。

### character_set_results

- 作用域：SESSION | GLOBAL
- 是否持久化到集群：是
- 默认值：`utf8mb4`
- 该变量表示数据发送至客户端时所使用的字符集。

### character_set_server

- 作用域：SESSION | GLOBAL
- 是否持久化到集群：是
- 默认值：`utf8mb4`
- 当 `CREATE SCHEMA` 中没有指定字符集时，该变量表示这些新建的表结构所使用的字符集。

### `cte_max_recursion_depth`

- 作用域：SESSION | GLOBAL
- 是否持久化到集群：是
- 默认值：`1000`
- 范围：`[0, 4294967295]`
- 这个变量用于控制公共表表达式的最大递归深度。

### `datadir`

- 作用域：NONE
- 默认值：/tmp/tidb
- 这个变量表示数据存储的位置，位置可以是本地路径。如果数据存储在 TiKV 上，则可以是指向 PD 服务器的路径。
- 如果变量值的格式为 `ip_address:port`，表示 TiDB 在启动时连接到的 PD 服务器。

### `ddl_slow_threshold`

- 作用域：INSTANCE
- 默认值：`300`
- 单位：毫秒
- 耗时超过该阈值的 DDL 操作会被输出到日志。

### `default_authentication_plugin`

- 作用域：GLOBAL
- 是否持久化到集群：是
- 默认值：`mysql_native_password`
- 可选值：`mysql_native_password`，`caching_sha2_password`
- 服务器和客户端建立连接时，这个变量用于设置服务器对外通告的默认身份验证方式。如要了解该变量的其他可选值，参见[可用的身份验证插件](/security-compatibility-with-mysql.md#可用的身份验证插件)。

### `foreign_key_checks`

- 作用域：SESSION | GLOBAL
- 是否持久化到集群：是
- 默认值：`OFF`
- 为保持兼容，TiDB 对外键检查返回 `OFF`。

### `hostname`

- 作用域：NONE
- 默认值：（系统主机名）
- 这个变量一个只读变量，表示 TiDB server 的主机名。

### `init_connect`

- 作用域：GLOBAL
- 是否持久化到集群：是
- 默认值：""
- 用户首次连接到 TiDB 服务器时，`init_connect` 特性允许 TiDB 自动执行一条或多条 SQL 语句。如果你有 `CONNECTION_ADMIN` 或者 `SUPER` 权限，这些 SQL 语句将不会被自动执行。如果这些语句执行报错，你的用户连接将被终止。

### `innodb_lock_wait_timeout`

- 作用域：SESSION | GLOBAL
- 是否持久化到集群：是
- 默认值：`50`
- 范围：`[1, 3600]`
- 单位：秒
- 悲观事务语句等锁时间。

### `interactive_timeout`

- 作用域：SESSION | GLOBAL
- 是否持久化到集群：是
- 默认值：`28800`
- 范围：`[1, 31536000]`
- 单位：秒
- 该变量表示交互式用户会话的空闲超时。交互式用户会话是指使用 `CLIENT_INTERACTIVE` 选项调用 [`mysql_real_connect()`](https://dev.mysql.com/doc/c-api/5.7/en/mysql-real-connect.html) API 建立的会话（例如：MySQL shell 客户端）。该变量与 MySQL 完全兼容。

### `last_plan_from_binding` <span class="version-mark">从 v4.0 版本开始引入</span>

- 作用域：SESSION
- 默认值：`OFF`
- 该变量用来显示上一条执行的语句所使用的执行计划是否来自 binding 的[执行计划](/sql-plan-management.md)。

### `last_plan_from_cache` <span class="version-mark">从 v4.0 版本开始引入</span>

- 作用域：SESSION
- 默认值：`OFF`
- 这个变量用来显示上一个 `execute` 语句所使用的执行计划是不是直接从 plan cache 中取出来的。

### `license`

- 作用域：NONE
- 默认值：`Apache License 2.0`
- 这个变量表示 TiDB 服务器的安装许可证。

### `max_execution_time`

- 作用域：SESSION | GLOBAL
- 是否持久化到集群：是
- 默认值：`0`
- 范围：`[0, 2147483647]`
- 单位：毫秒
- 语句最长执行时间。默认值 (0) 表示无限制。

> **注意：**
>
> `max_execution_time` 目前对所有类型的语句生效，并非只对 `SELECT` 语句生效，与 MySQL 不同（只对`SELECT` 语句生效）。实际精度在 100ms 级别，而非更准确的毫秒级别。

### `plugin_dir`

- 作用域：INSTANCE
- 默认值：""
- 指定加载插件的目录。

### `plugin_load`

- 作用域：INSTANCE
- 默认值：""
- 指定 TiDB 启动时加载的插件，多个插件之间用逗号（,）分隔。

### `port`

- 作用域：NONE
- 默认值：`4000`
- 范围：`[0, 65535]`
- 使用 MySQL 协议时 tidb-server 监听的端口。

### `rand_seed1`

- 作用域：SESSION
- 默认值：`0`
- 范围：`[0, 2147483647]`
- 该变量用于为 SQL 函数 `RAND()` 中使用的随机值生成器添加种子。
- 该变量的行为与 MySQL 兼容。

### rand_seed2

- 作用域：SESSION
- 默认值：`0`
- 范围：`[0, 2147483647]`
- 该变量用于为 SQL 函数 `RAND()` 中使用的随机值生成器添加种子。
- 该变量的行为与 MySQL 兼容。

### skip_name_resolve <span class="version-mark">从 v5.2.0 版本开始引入</span>

- 作用域：GLOBAL
- 是否持久化到集群：是
- 默认值：`OFF`
- 该变量控制 `tidb-server` 实例是否将主机名作为连接握手的一部分来解析。
- 当 DNS 不可靠时，可以启用该变量来提高网络性能。

> **注意：**
>
> 当 `skip_name_resolve` 设置为 `ON` 时，身份信息中包含主机名的用户将无法登录服务器。例如：
>
> ```sql
> CREATE USER 'appuser'@'apphost' IDENTIFIED BY 'app-password';
> ```
>
> 该示例中，建议将 `apphost` 替换为 IP 地址或通配符（`%`）。

### `socket`

- 作用域：NONE
- 默认值：""
- 使用 MySQL 协议时，tidb-server 所监听的本地 unix 套接字文件。

### `sql_mode`

- 作用域：SESSION | GLOBAL
- 是否持久化到集群：是
- 默认值：`ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION`
- 这个变量控制许多 MySQL 兼容行为。详情见 [SQL 模式](/sql-mode.md)。

### `sql_select_limit` <span class="version-mark">从 v4.0.2 版本开始引入</span>

- 作用域：SESSION | GLOBAL
- 是否持久化到集群：是
- 默认值：`18446744073709551615`
- 范围：`[0, 18446744073709551615]`
- `SELECT` 语句返回的最大行数。

### `system_time_zone`

- 作用域：NONE
- 默认值：（随系统）
- 该变量显示首次引导启动 TiDB 时的系统时区。另请参阅 [`time_zone`](#time_zone)。

### `tidb_allow_batch_cop` <span class="version-mark">从 v4.0 版本开始引入</span>

- 作用域：SESSION | GLOBAL
- 是否持久化到集群：是
- 默认值：`1`
- 范围：`[0, 2]`
- 这个变量用于控制 TiDB 向 TiFlash 发送 coprocessor 请求的方式，有以下几种取值：

    * 0：从不批量发送请求
    * 1：aggregation 和 join 的请求会进行批量发送
    * 2：所有的 cop 请求都会批量发送

### `tidb_allow_fallback_to_tikv` <span class="version-mark">从 v5.0 版本开始引入</span>

- 作用域：SESSION | GLOBAL
- 是否持久化到集群：是
- 默认值：""
- 这个变量表示将 TiKV 作为备用存储引擎的存储引擎列表。当该列表中的存储引擎发生故障导致 SQL 语句执行失败时，TiDB 会使用 TiKV 作为存储引擎再次执行该 SQL 语句。目前支持设置该变量为 "" 或者 "tiflash"。如果设置该变量为 "tiflash"，当 TiFlash 返回超时错误（对应的错误码为 ErrTiFlashServerTimeout）时，TiDB 会使用 TiKV 作为存储引擎再次执行该 SQL 语句。

### `tidb_allow_function_for_expression_index` <span class="version-mark">从 v5.2.0 版本开始引入</span>

- 作用域：NONE
- 默认值：`lower, md5, reverse, tidb_shard, upper, vitess_hash`
- 这个变量用于显示创建表达式索引所允许使用的函数。

### `tidb_allow_mpp` <span class="version-mark">从 v5.0 版本开始引入</span>

- 作用域：SESSION | GLOBAL
- 是否持久化到集群：是
- 默认值：`ON`
- 这个变量用于控制是否使用 TiFlash 的 MPP 模式执行查询，可以设置的值包括：
    - 0 或 OFF，代表从不使用 MPP 模式
    - 1 或 ON，代表由优化器根据代价估算选择是否使用 MPP 模式（默认）

MPP 是 TiFlash 引擎提供的分布式计算框架，允许节点之间的数据交换并提供高性能、高吞吐的 SQL 算法。MPP 模式选择的详细说明参见[控制是否选择 MPP 模式](/tiflash/use-tiflash.md#控制是否选择-mpp-模式)。

### `tidb_allow_remove_auto_inc` <span class="version-mark">从 v2.1.18 和 v3.0.4 版本开始引入</span>

- 作用域：SESSION
- 默认值：`OFF`
- 这个变量用来控制是否允许通过 `ALTER TABLE MODIFY` 或 `ALTER TABLE CHANGE` 来移除某个列的 `AUTO_INCREMENT` 属性。默认 (`OFF`) 为不允许。

### `tidb_analyze_version` <span class="version-mark">从 v5.1.0 版本开始引入</span>

- 作用域：SESSION | GLOBAL
- 是否持久化到集群：是
- 默认值：`2`
- 范围：`[1, 2]`
- 这个变量用于控制 TiDB 收集统计信息的行为。
- 在 v5.3.0 及之后的版本中，该变量的默认值为 `2`，作为实验特性启用，具体可参照[统计信息简介](/statistics.md)文档。如果从 v5.3.0 之前版本的集群升级至 v5.3.0 及之后的版本，`tidb_analyze_version` 的默认值不发生变化。

### `tidb_auto_analyze_end_time`

- 作用域：GLOBAL
- 是否持久化到集群：是
- 默认值：`23:59 +0000`
- 这个变量用来设置一天中允许自动 ANALYZE 更新统计信息的结束时间。例如，只允许在凌晨 1:00 至 3:00 之间自动更新统计信息，可以设置如下：

    - `tidb_auto_analyze_start_time='01:00 +0000'`
    - `tidb_auto_analyze_end_time='03:00 +0000'`

### `tidb_auto_analyze_ratio`

- 作用域：GLOBAL
- 是否持久化到集群：是
- 默认值：`0.5`
- 这个变量用来设置 TiDB 在后台自动执行 [`ANALYZE TABLE`](/sql-statements/sql-statement-analyze-table.md) 更新统计信息的阈值。`0.5` 指的是当表中超过 50% 的行被修改时，触发自动 ANALYZE 更新。可以指定 `tidb_auto_analyze_start_time` 和 `tidb_auto_analyze_end_time` 来限制自动 ANALYZE 的时间

> **注意：**
>
> 只有在 TiDB 的启动配置文件中开启了 `run-auto-analyze` 选项，该 TiDB 才会触发 `auto_analyze`。

### `tidb_auto_analyze_start_time`

- 作用域：GLOBAL
- 是否持久化到集群：是
- 默认值：`00:00 +0000`
- 这个变量用来设置一天中允许自动 ANALYZE 更新统计信息的开始时间。例如，只允许在凌晨 1:00 至 3:00 之间自动更新统计信息，可以设置如下：

    - `tidb_auto_analyze_start_time='01:00 +0000'`
    - `tidb_auto_analyze_end_time='03:00 +0000'`

### `tidb_backoff_lock_fast`

- 作用域：SESSION | GLOBAL
- 是否持久化到集群：是
- 默认值：`10`
- 范围：`[1, 2147483647]`
- 这个变量用来设置读请求遇到锁的 backoff 时间。

### `tidb_backoff_weight`

- 作用域：SESSION | GLOBAL
- 是否持久化到集群：是
- 默认值：`2`
- 范围：`[0, 2147483647]`
- 这个变量用来给 TiDB 的 `backoff` 最大时间增加权重，即内部遇到网络或其他组件 (TiKV, PD) 故障时，发送重试请求的最大重试时间。可以通过这个变量来调整最大重试时间，最小值为 1。

    例如，TiDB 向 PD 取 TSO 的基础超时时间是 15 秒，当 `tidb_backoff_weight = 2` 时，取 TSO 的最大超时时间为：基础时间 \* 2 等于 30 秒。

    在网络环境较差的情况下，适当增大该变量值可以有效缓解因为超时而向应用端报错的情况；而如果应用端希望更快地接到报错信息，则应该尽量减小该变量的值。

### `tidb_batch_insert`

- 作用域：SESSION
- 默认值：`OFF`
- 该变量允许 `tidb_dml_batch_size` 变量作用于 `INSERT` 语句。
- 只有该变量值为 `OFF` 才符合 ACID 原则。将该变量修改为任何其他值，会破坏 TiDB 的原子性和隔离性保证，因为每个 `INSERT` 语句都会被分割成较小的事务。

### `tidb_broadcast_join_threshold_count` <span class="version-mark">从 v5.0 版本开始引入</span>

- 作用域：SESSION | GLOBAL
- 是否持久化到集群：是
- 默认值：`10240`
- 范围：`[0, 9223372036854775807]`
- 单位为行数。如果 join 的对象为子查询，优化器无法估计子查询结果集大小，在这种情况下通过结果集行数判断。如果子查询的行数估计值小于该变量，则选择 Broadcast Hash Join 算法。否则选择 Shuffled Hash Join 算法。

### `tidb_broadcast_join_threshold_size` <span class="version-mark">从 v5.0 版本开始引入</span>

- 作用域：SESSION | GLOBAL
- 是否持久化到集群：是
- 默认值：`104857600` (100 MiB)
- 范围：`[0, 9223372036854775807]`
- 单位：字节
- 如果表大小（字节数）小于该值，则选择 Broadcast Hash Join 算法。否则选择 Shuffled Hash Join 算法。

### `tidb_build_stats_concurrency`

- 作用域：SESSION | GLOBAL
- 是否持久化到集群：是
- 默认值：`4`
- 这个变量用来设置 ANALYZE 语句执行时并发度。
- 当这个变量被设置得更大时，会对其它的查询语句执行性能产生一定影响。

### `tidb_capture_plan_baselines` <span class="version-mark">从 v4.0 版本开始引入</span>

- 作用域：GLOBAL
- 是否持久化到集群：是
- 默认值：`OFF`
- 这个变量用于控制是否开启[自动捕获绑定](/sql-plan-management.md#自动捕获绑定-baseline-capturing)功能。该功能依赖 Statement Summary，因此在使用自动绑定之前需打开 Statement Summary 开关。
- 开启该功能后会定期遍历一次 Statement Summary 中的历史 SQL 语句，并为至少出现两次的 SQL 语句自动创建绑定。

### `tidb_check_mb4_value_in_utf8`

- 作用域：INSTANCE
- 默认值：`ON`
- 设置该变量为 `ON` 可强制只存储[基本多文种平面 (BMP)](https://zh.wikipedia.org/zh-hans/Unicode字符平面映射) 编码区段内的 `utf8` 字符值。若要存储 BMP 区段外的 `utf8` 值，推荐使用 `utf8mb4` 字符集。
- 早期版本的 TiDB 中 (v2.1.x)，`utf8` 检查更为宽松。如果你的 TiDB 集群是从早期版本升级的，推荐关闭该变量，详情参阅[升级与升级后常见问题](/faq/upgrade-faq.md)。

### `tidb_checksum_table_concurrency`

- 作用域：SESSION
- 默认值：`4`
- 这个变量用来设置 `ADMIN CHECKSUM TABLE` 语句执行时扫描索引的并发度。当这个变量被设置得更大时，会对其它的查询语句执行性能产生一定影响。

### `tidb_config`

- 作用域：SESSION
- 默认值：""
- 这个变量是一个只读变量，用来获取当前 TiDB Server 的配置信息。

### `tidb_constraint_check_in_place`

- 作用域：SESSION | GLOBAL
- 是否持久化到集群：是
- 默认值：`OFF`
- 该变量仅适用于乐观事务模型。当这个变量设置为 `OFF` 时，唯一索引的重复值检查会被推迟到事务提交时才进行。这有助于提高性能，但对于某些应用，可能导致非预期的行为。详情见[约束](/constraints.md)。

    - 乐观事务模型下将 `tidb_constraint_check_in_place` 设置为 0：

        {{< copyable "sql" >}}

        ```sql
        create table t (i int key);
        insert into t values (1);
        begin optimistic;
        insert into t values (1);
        ```

        ```
        Query OK, 1 row affected
        ```

        {{< copyable "sql" >}}

        ```sql
        tidb> commit; -- 事务提交时才检查
        ```

        ```
        ERROR 1062 : Duplicate entry '1' for key 'PRIMARY'
        ```

    - 乐观事务模型下将 `tidb_constraint_check_in_place` 设置为 1：

        {{< copyable "sql" >}}

        ```sql
        set @@tidb_constraint_check_in_place=1;
        begin optimistic;
        insert into t values (1);
        ```

        ```
        ERROR 1062 : Duplicate entry '1' for key 'PRIMARY'
        ```

悲观事务模式中，始终默认执行约束检查。

### `tidb_current_ts`

- 作用域：SESSION
- 默认值：`0`
- 这个变量是一个只读变量，用来获取当前事务的时间戳。

### `tidb_ddl_error_count_limit`

- 作用域：GLOBAL
- 是否持久化到集群：是
- 默认值：`512`
- 范围：`[0, 9223372036854775807]`
- 这个变量用来控制 DDL 操作失败重试的次数。失败重试次数超过该参数的值后，会取消出错的 DDL 操作。

### `tidb_ddl_reorg_batch_size`

- 作用域：GLOBAL
- 是否持久化到集群：是
- 默认值：`256`
- 范围：`[32, 10240]`
- 这个变量用来设置 DDL 操作 `re-organize` 阶段的 batch size。比如 `ADD INDEX` 操作，需要回填索引数据，通过并发 `tidb_ddl_reorg_worker_cnt` 个 worker 一起回填数据，每个 worker 以 batch 为单位进行回填。

    - 如果 `ADD INDEX` 操作时有较多 `UPDATE` 操作或者 `REPLACE` 等更新操作，batch size 越大，事务冲突的概率也会越大，此时建议调小 batch size 的值，最小值是 32。
    - 在没有事务冲突的情况下，batch size 可设为较大值，最大值是 10240，这样回填数据的速度更快，但是 TiKV 的写入压力也会变大。

### `tidb_ddl_reorg_priority`

- 作用域：SESSION
- 默认值：PRIORITY_LOW
- 这个变量用来设置 `ADD INDEX` 操作 `re-organize` 阶段的执行优先级，可设置为 `PRIORITY_LOW`/`PRIORITY_NORMAL`/`PRIORITY_HIGH`。

### `tidb_ddl_reorg_worker_cnt`

- 作用域：GLOBAL
- 是否持久化到集群：是
- 默认值：`4`
- 范围：`[1, 256]`
- 这个变量用来设置 DDL 操作 `re-organize` 阶段的并发度。

### `tidb_disable_txn_auto_retry`

- 作用域：SESSION | GLOBAL
- 是否持久化到集群：是
- 默认值：`ON`
- 这个变量用来设置是否禁用显式的乐观事务自动重试，设置为 `ON` 时，不会自动重试，如果遇到事务冲突需要在应用层重试。

    如果将该变量的值设为 `OFF`，TiDB 将会自动重试事务，这样在事务提交时遇到的错误更少。需要注意的是，这样可能会导致数据更新丢失。

    这个变量不会影响自动提交的隐式事务和 TiDB 内部执行的事务，它们依旧会根据 `tidb_retry_limit` 的值来决定最大重试次数。

    关于是否需要禁用自动重试，请参考[重试的局限性](/optimistic-transaction.md#重试的局限性)。

    该变量只适用于乐观事务，不适用于悲观事务。悲观事务的重试次数由 [`max_retry_count`](/tidb-configuration-file.md#max-retry-count) 控制。

### `tidb_distsql_scan_concurrency`

- 作用域：SESSION | GLOBAL
- 是否持久化到集群：是
- 默认值：`15`
- 范围：`[1, 256]`
- 这个变量用来设置 scan 操作的并发度。
- AP 类应用适合较大的值，TP 类应用适合较小的值。对于 AP 类应用，最大值建议不要超过所有 TiKV 节点的 CPU 核数。
- 若表的分区较多可以适当调小该参数，避免 TiKV 内存溢出 (OOM)。

### `tidb_dml_batch_size`

- 作用域：SESSION | GLOBAL
- 是否持久化到集群：是
- 默认值：`0`
- 范围：`[0, 2147483647]`
- 这个变量的值大于 `0` 时，TiDB 会将 `INSERT` 或 `LOAD DATA` 等语句在更小的事务中批量提交。这样可减少内存使用，确保大批量修改时事务大小不会达到 `txn-total-size-limit` 限制。
- 要使 `tidb_dml_batch_size` 作用于 `INSERT` 语句，你还需要将系统变量 [`tidb_batch_insert`](#tidb_batch_insert) 设置为 `ON`。
- 只有变量值为 `0` 时才符合 ACID 要求。否则无法保证 TiDB 的原子性和隔离性要求。

### `tidb_enable_1pc` <span class="version-mark">从 v5.0 版本开始引入</span>

- 作用域：SESSION | GLOBAL
- 是否持久化到集群：是
- 默认值：`ON`
- 指定是否在只涉及一个 Region 的事务上启用一阶段提交特性。比起传统两阶段提交，一阶段提交能大幅降低事务提交延迟并提升吞吐。

> **注意：**
>
> - 对于新创建的集群，默认值为 ON。对于升级版本的集群，如果升级前是 v5.0 以下版本，升级后默认值为 `OFF`。
> - 启用 TiDB Binlog 后，开启该选项无法获得性能提升。要获得性能提升，建议使用 [TiCDC](/ticdc/ticdc-overview.md) 替代 TiDB Binlog。
> - 启用该参数仅意味着一阶段提交成为可选的事务提交模式，实际由 TiDB 自行判断选择最合适的提交模式进行事务提交。

### `tidb_enable_amend_pessimistic_txn` <span class="version-mark">从 v4.0.7 版本开始引入</span>

- 作用域：SESSION | GLOBAL
- 是否持久化到集群：是
- 默认值：`OFF`
- 这个变量用于控制是否开启 `AMEND TRANSACTION` 特性。在[悲观事务模式](/pessimistic-transaction.md)下开启该特性后，如果该事务相关的表存在并发 DDL 操作和 SCHEMA VERSION 变更，TiDB 会尝试对该事务进行 amend 操作，修正该事务的提交内容，使其和最新的有效 SCHEMA VERSION 保持一致，从而成功提交该事务而不返回 `Information schema is changed` 报错。该特性对以下并发 DDL 变更生效：

    - `ADD COLUMN` 或 `DROP COLUMN` 类型的 DDL 操作。
    - `MODIFY COLUMN` 或 `CHANGE COLUMN` 类型的 DDL 操作，且只对增大字段长度的操作生效。
    - `ADD INDEX` 或 `DROP INDEX` 类型的 DDL 操作，且操作的索引列须在事务开启之前创建。

> **注意：**
>
> 目前该特性可能造成事务语义的变化，且与 TiDB Binlog 存在部分不兼容的场景，可以参考[事务语义行为区别](https://github.com/pingcap/tidb/issues/21069)和[与 TiDB Binlog 兼容问题汇总](https://github.com/pingcap/tidb/issues/20996)了解更多关于该特性的使用注意事项。

### `tidb_enable_async_commit` <span class="version-mark">从 v5.0 版本开始引入</span>

- 作用域：SESSION | GLOBAL
- 是否持久化到集群：是
- 默认值：`ON`
- 该变量控制是否启用 Async Commit 特性，使事务两阶段提交的第二阶段于后台异步进行。开启本特性能降低事务提交的延迟。

> **注意：**
>
> - 对于新创建的集群，默认值为 ON。对于升级版本的集群，如果升级前是 v5.0 以下版本，升级后默认值为 `OFF`。
> - 启用 TiDB Binlog 后，开启该选项无法获得性能提升。要获得性能提升，建议使用 [TiCDC](/ticdc/ticdc-overview.md) 替代 TiDB Binlog。
> - 启用该参数仅意味着 Async Commit 成为可选的事务提交模式，实际由 TiDB 自行判断选择最合适的提交模式进行事务提交。

### `tidb_enable_auto_increment_in_generated`

- 作用域：SESSION | GLOBAL
- 是否持久化到集群：是
- 默认值：`OFF`
- 这个变量用于控制是否允许在创建生成列或者表达式索引时引用自增列。

### `tidb_enable_cascades_planner`

> **警告：**
>
> 目前 cascades planner 为实验特性，不建议在生产环境中使用。

- 作用域：SESSION | GLOBAL
- 是否持久化到集群：是
- 默认值：`OFF`
- 这个变量用于控制是否开启 cascades planner。

### `tidb_enable_chunk_rpc` <span class="version-mark">从 v4.0 版本开始引入</span>

- 作用域：SESSION
- 默认值：`ON`
- 这个变量用来设置是否启用 Coprocessor 的 `Chunk` 数据编码格式。

### `tidb_enable_clustered_index` <span class="version-mark">从 v5.0 版本开始引入</span>

- 作用域：SESSION | GLOBAL
- 是否持久化到集群：是
- 默认值：`INT_ONLY`
- 可选值：`OFF`，`ON`，`INT_ONLY`
- 这个变量用于控制默认情况下表的主键是否使用[聚簇索引](/clustered-indexes.md)。“默认情况”即不显式指定 `CLUSTERED`/`NONCLUSTERED` 关键字的情况。可设置为 `OFF`/`ON`/`INT_ONLY`。
    - `OFF` 表示所有主键默认使用非聚簇索引。
    - `ON` 表示所有主键默认使用聚簇索引。
    - `INT_ONLY` 此时的行为受配置项 `alter-primary-key` 控制。如果该配置项取值为 `true`，则所有主键默认使用非聚簇索引；如果该配置项取值为 `false`，则由单个整数类型的列构成的主键默认使用聚簇索引，其他类型的主键默认使用非聚簇索引。

### `tidb_enable_collect_execution_info`

- 作用域：INSTANCE
- 默认值：`ON`
- 这个变量用于控制是否同时将各个执行算子的执行信息记录入 slow query log 中。

### `tidb_enable_column_tracking` <span class="version-mark">从 v5.4.0 版本开始引入</span>

> **警告：**
>
> 收集 `PREDICATE COLUMNS` 的统计信息目前为实验特性，不建议在生产环境中使用。

- 作用域：GLOBAL
- 是否持久化到集群：是
- 默认值：`OFF`
- 这个变量用于控制是否开启 TiDB 对 `PREDICATE COLUMNS` 的收集。关闭该变量后，之前收集的 `PREDICATE COLUMNS` 会被清除。详情见[收集部分列的统计信息](/statistics.md#收集部分列的统计信息)。

### `tidb_enable_enhanced_security`

- 作用域：NONE
- 默认值：`OFF`
- 这个变量表示所连接的 TiDB 服务器是否启用了安全增强模式 (SEM)。若要改变该变量值，你需要在 TiDB 服务器的配置文件中修改 `enable-sem` 项的值，并重启 TiDB 服务器。
- 安全增强模式受[安全增强式 Linux](https://zh.wikipedia.org/wiki/安全增强式Linux) 等系统设计的启发，削减拥有 MySQL `SUPER` 权限的用户能力，转而使用细粒度的 `RESTRICTED` 权限作为替代。这些细粒度的 `RESTRICTED` 权限如下：
    - `RESTRICTED_TABLES_ADMIN`：能够写入 `mysql` 库中的系统表，能查看 `information_schema` 表上的敏感列。
    - `RESTRICTED_STATUS_ADMIN`：能够在 `SHOW STATUS` 命令中查看敏感内容。
    - `RESTRICTED_VARIABLES_ADMIN`：能够在 `SHOW [GLOBAL] VARIABLES` 和 `SET` 命令中查看和设置包含敏感内容的变量。
    - `RESTRICTED_USER_ADMIN`：能够阻止其他用户更改或删除用户帐户。

### `tidb_restricted_read_only` <span class="version-mark">从 v5.2.0 版本开始引入</span>

- 作用域：GLOBAL
- 是否持久化到集群：是
- 默认值：`0`
- 可选值：`0` 和 `1`
- 该变量可以控制整个集群的只读状态。开启后（即该值为 `1`），整个集群中的 TiDB 服务器都将进入只读状态，只有 `SELECT`、`USE`、`SHOW` 等不会修改数据的语句才能被执行，其他如 `INSERT`、`UPDATE` 等语句会被拒绝执行。
- 该变量开启只读模式只保证整个集群最终进入只读模式，当变量修改状态还没被同步到其他 TiDB 服务器时，尚未同步的 TiDB 仍然停留在非只读模式。
- 在变量开启时，正在执行的 SQL 语句不会受影响，只对新执行的 SQL 语句进行是否只读的检查。
- 在变量开启时，对于尚未提交的事务：
    - 如果有尚未提交的只读事务，可正常提交该事务。
    - 如果尚未提交的事务为非只读事务，在事务内执行写入的 SQL 语句会被拒绝。
    - 如果尚提交的事务已经有数据改动，其提交也会被拒绝。
- 当集群开启只读模式后，所有用户（包括 `SUPER` 用户）都无法执行可能写入数据的 SQL 语句，除非该用户被显式地授予了 `RESTRICTED_REPLICA_WRITER_ADMIN` 权限。
- 拥有 `RESTRICTED_VARIABLES_ADMIN` 或 `SUPER` 权限的用户可以修改该变量。如果用户开启了[安全增强模式 (Security Enhanced Mode)](/system-variables.md#tidb_enable_enhanced_security)，则只有 `RESTRICTED_VARIABLES_ADMIN` 权限的用户才能修改该变量。

### `tidb_enable_fast_analyze`

> **警告：**
>
> 目前快速分析功能为实验特性，不建议在生产环境中使用。

- 作用域：SESSION | GLOBAL
- 是否持久化到集群：是
- 默认值：`OFF`
- 这个变量用来控制是否启用统计信息快速分析功能。默认值 0 表示不开启。
- 快速分析功能开启后，TiDB 会随机采样约 10000 行的数据来构建统计信息。因此在数据分布不均匀或者数据量比较少的情况下，统计信息的准确度会比较低。这可能导致执行计划不优，比如选错索引。如果可以接受普通 `ANALYZE` 语句的执行时间，则推荐关闭快速分析功能。

### `tidb_enable_index_merge` <span class="version-mark">从 v4.0 版本开始引入</span>

> **注意：**
>
> - 当集群从 v4.0.0 以下版本升级到 v5.4.0 及以上版本时，该变量开关默认关闭，防止升级后计划发生变化导致回退。
> - 当集群从 v4.0.0 及以上版本升级到 v5.4.0 及以上版本时，该变量开关保持升级前的状态。
> - 对于 v5.4.0 及以上版本的新建集群，该变量开关默认开启。

- 作用域：SESSION | GLOBAL
- 是否持久化到集群：是
- 默认值：`ON` 
- 这个变量用于控制是否开启 index merge 功能。

### tidb_enable_legacy_instance_scope <span class="version-mark">从 v6.0.0 版本开始引入</span>

- 作用域：SESSION | GLOBAL
- 是否持久化到集群：是
- 默认值：`ON` 
- 这个变量用于允许使用 `SET SESSION` 对 `INSTANCE` 作用域的变量进行设置，用法同 `SET GLOBAL`。
- 为了兼容之前的 TiDB 版本，该变量值默认为 `ON`。

### `tidb_enable_list_partition` <span class="version-mark">从 v5.0 版本开始引入</span>

> **警告：**
>
> 目前 List 分区和 List COLUMNS 分区类型为实验特性，不建议在生产环境中使用。

- 作用域：SESSION | GLOBAL
- 是否持久化到集群：是
- 默认值：`OFF`
- 这个变量用来设置是否开启 `LIST (COLUMNS) TABLE PARTITION` 特性。

### `tidb_enable_mutation_checker`（从 v6.0.0 版本开始引入）

- 作用域：SESSION | GLOBAL
- 是否持久化到集群：是
- 默认值：`OFF`
- 这个变量用于设置是否开启 mutation checker。mutation checker 是一项在 DML 语句执行过程中进行的数据索引一致性校验，校验报错会回滚当前语句。开启该校验会导致 CPU 使用轻微上升。详见[数据索引一致性报错](/troubleshoot-data-inconsistency-errors.md)。

### `tidb_enable_noop_functions` <span class="version-mark">从 v4.0 版本开始引入</span>

- 作用域：SESSION | GLOBAL
- 是否持久化到集群：是
- 默认值：`OFF`
- 默认情况下，用户尝试将某些语法用于尚未实现的功能时，TiDB 会报错。若将该变量值设为 `ON`，TiDB 则自动忽略此类功能不可用的情况，即不会报错。若用户无法更改 SQL 代码，可考虑将变量值设为 `ON`。
- 启用 `noop` 函数可以控制以下行为：
    * `get_lock` 和 `release_lock` 函数
    * `LOCK IN SHARE MODE` 语法
    * `SQL_CALC_FOUND_ROWS` 语法
    * `START TRANSACTION READ ONLY` 和 `SET TRANSACTION READ ONLY` 语法
    * `tx_read_only`、`transaction_read_only`、`offline_mode`、`super_read_only`、`read_only` 以及 `sql_auto_is_null` 系统变量

> **警告：**
>
> 该变量只有在默认值 `OFF` 时，才算是安全的。因为设置 `tidb_enable_noop_functions=1` 后，TiDB 会自动忽略某些语法而不报错，这可能会导致应用程序出现异常行为。例如，允许使用语法 `START TRANSACTION READ ONLY` 时，事务仍会处于读写模式。

### `tidb_enable_paging` <span class="version-mark">从 v5.4.0 版本开始引入</span>

- 作用域：SESSION | GLOBAL
- 是否持久化到集群：是
- 默认值：`OFF`
- 这个变量用于控制 `IndexLookUp` 算子是否使用分页 (paging) 方式发送 Coprocessor 请求。
- 适用场景：对于使用 `IndexLookUp` 和 `Limit` 并且 `Limit` 无法下推到 `IndexScan` 上的读请求，可能会出现读请求的延迟高、TiKV 的 Unified read pool CPU 使用率高的情况。在这种情况下，由于 `Limit` 算子只需要少部分数据，开启 `tidb_enable_paging`，能够减少处理数据的数量，从而降低延迟、减少资源消耗。
- 开启 `tidb_enable_paging` 后，`Limit` 无法下推且数量小于 `960` 的 `IndexLookUp` 请求会使用 paging 方式发送 Coprocessor 请求。`Limit` 的值越小，优化效果会越明显。

### `tidb_enable_parallel_apply` <span class="version-mark">从 v5.0 版本开始引入</span>

- 作用域：SESSION | GLOBAL
- 是否持久化到集群：是
- 默认值：0
- 这个变量用于控制是否开启 Apply 算子并发，并发数由 `tidb_executor_concurrency` 变量控制。Apply 算子用来处理关联子查询且默认无并发，所以执行速度较慢。打开 Apply 并发开关可增加并发度，提高执行速度。目前默认关闭。

### `tidb_enable_pseudo_for_outdated_stats` <span class="version-mark">从 v5.3.0 版本开始引入</span>

- 作用域：SESSION | GLOBAL
- 是否持久化到集群：是
- 默认值：`ON`
- 这个变量用来控制优化器在一张表上的统计信息过期时的行为。
- 统计信息过期的判断标准：最近一次对某张表执行 `ANALYZE` 获得统计信息后，该表数据被修改的行数大于该表总行数的 80%，便可判定该表的统计信息已过期。该比例可通过 [`pseudo-estimate-ratio`](/tidb-configuration-file.md#pseudo-estimate-ratio) 配置参数调整。
- 默认情况下（即该变量值为 `ON` 时），某张表上的统计信息过期后，优化器认为该表上除总行数以外的统计信息不再可靠，转而使用 pseudo 统计信息。将该变量值设为 `OFF` 后，即使统计信息过期，优化器也仍会使用该表上的统计信息。
- 如果表数据修改较频繁，没有及时对表执行 `ANALYZE`，但又希望执行计划保持稳定，可以将该变量值设为 `OFF`。

### `tidb_enable_rate_limit_action`

- 作用域：SESSION | GLOBAL
- 是否持久化到集群：是
- 默认值：`ON`
- 这个变量控制是否为读数据的算子开启动态内存控制功能。读数据的算子默认启用 [`tidb_distsql_scan_concurrency`](/system-variables.md#tidb_distsql_scan_concurrency) 所允许的最大线程数来读取数据。当单条 SQL 语句的内存使用每超过 [`tidb_mem_quota_query`](/system-variables.md#tidb_mem_quota_query) 一次，读数据的算子会停止一个线程。
- 当读数据的算子只剩 1 个线程且当单条 SQL 语句的内存使用继续超过 [`tidb_mem_quota_query`](/system-variables.md#tidb_mem_quota_query) 时，该 SQL 语句会触发其它的内存控制行为，例如[落盘](/tidb-configuration-file.md#oom-use-tmp-storage)。

### `tidb_enable_slow_log`

- 作用域：INSTANCE
- 默认值：`ON`
- 这个变量用于控制是否开启 slow log 功能。

### `tidb_enable_stmt_summary` <span class="version-mark">从 v3.0.4 版本开始引入</span>

- 作用域：GLOBAL
- 是否持久化到集群：是
- 默认值：`ON`
- 这个变量用来控制是否开启 statement summary 功能。如果开启，SQL 的耗时等执行信息将被记录到系统表 `information_schema.STATEMENTS_SUMMARY` 中，用于定位和排查 SQL 性能问题。

### `tidb_enable_strict_double_type_check` <span class="version-mark">从 v5.0 版本开始引入</span>

- 作用域：SESSION | GLOBAL
- 是否持久化到集群：是
- 默认值：`ON`
- 这个变量用来控制是否可以用 `DOUBLE` 类型的无效定义创建表。该设置的目的是提供一个从 TiDB 早期版本升级的方法，因为早期版本在验证类型方面不太严格。
- 该变量的默认值 `ON` 与 MySQL 兼容。

例如，由于无法保证浮点类型的精度，现在将 `DOUBLE(10)` 类型视为无效。将 `tidb_enable_strict_double_type_check` 更改为 `OFF` 后，将会创建表。如下所示：

```sql
CREATE TABLE t1 (id int, c double(10));
ERROR 1149 (42000): You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use
SET tidb_enable_strict_double_type_check = 'OFF';
Query OK, 0 rows affected (0.00 sec)
CREATE TABLE t1 (id int, c double(10));
Query OK, 0 rows affected (0.09 sec)
```

> **注意：**
>
> 该设置仅适用于 `DOUBLE` 类型，因为 MySQL 允许为 `FLOAT` 类型指定精度。从 MySQL 8.0.17 开始已弃用此行为，不建议为 `FLOAT` 或 `DOUBLE` 类型指定精度。

### `tidb_enable_table_partition`

- 作用域：SESSION | GLOBAL
- 是否持久化到集群：是
- 默认值：`ON`
- 可选值：`OFF`，`ON`，`AUTO`
- 这个变量用来设置是否开启 `TABLE PARTITION` 特性。目前变量支持以下三种值：
    - 默认值 `ON` 表示开启 TiDB 当前已实现了的分区表类型，目前 Range partition、Hash partition 以及 Range column 单列的场景会生效。
    - `AUTO` 目前作用和 `ON` 一样。
    - `OFF` 表示关闭 `TABLE PARTITION` 特性，此时语法还是保持兼容，只是创建的表并不是真正的分区表，而是普通的表。

### `tidb_enable_telemetry` <span class="version-mark">从 v4.0.2 版本开始引入</span>

- 作用域：GLOBAL
- 是否持久化到集群：是
- 默认值：`ON`
- 这个变量用于动态地控制 TiDB 遥测功能是否开启。设置为 `OFF` 可以关闭 TiDB 遥测功能。当所有 TiDB 实例都设置 [`enable-telemetry`](/tidb-configuration-file.md#enable-telemetry-从-v402-版本开始引入) 为 `false` 时将忽略该系统变量并总是关闭 TiDB 遥测功能。参阅[遥测](/telemetry.md)了解该功能详情。

### `tidb_enable_top_sql` <span class="version-mark">从 v5.4.0 版本开始引入</span>

> **警告：**
>
> Top SQL 目前是实验性功能，不建议在生产环境中使用。

- 作用域：GLOBAL
- 是否持久化到集群：是
- 默认值：`OFF`
- 这个变量用控制是否开启 [Top SQL 特性](/dashboard/top-sql.md)。

### `tidb_enable_tso_follower_proxy` <span class="version-mark">从 v5.3.0 版本开始引入</span>

- 作用域：GLOBAL
- 是否持久化到集群：是
- 默认值：`OFF`
- 这个变量用来开启 TSO Follower Proxy 特性。当该值为 `OFF` 时，TiDB 仅会从 PD leader 获取 TSO。开启该特性之后，TiDB 在获取 TSO 时会将请求均匀地发送到所有 PD 节点上，通过 PD follower 转发 TSO 请求，从而降低 PD leader 的 CPU 压力。
- 适合开启 TSO Follower Proxy 的场景：
    * PD leader 因高压力的 TSO 请求而达到 CPU 瓶颈，导致 TSO RPC 请求的延迟较高。
    * 集群中的 TiDB 实例数量较多，且调高 [`tidb_tso_client_batch_max_wait_time`](/system-variables.md#tidb_tso_client_batch_max_wait_time-从-v530-版本开始引入) 并不能缓解 TSO RPC 请求延迟高的问题。

> **注意：**
>
> 如果 PD leader 的 TSO RPC 延迟升高，但其现象并非由 CPU 使用率达到瓶颈而导致（可能存在网络等问题），此时，打开 TSO Follower Proxy 可能会导致 TiDB 的语句执行延迟上升，从而影响集群的 QPS 表现。

### `tidb_enable_vectorized_expression` <span class="version-mark">从 v4.0 版本开始引入</span>

- 作用域：SESSION | GLOBAL
- 是否持久化到集群：是
- 默认值：`ON`
- 这个变量用于控制是否开启向量化执行。

### `tidb_enable_window_function`

- 作用域：SESSION | GLOBAL
- 是否持久化到集群：是
- 默认值：`ON`
- 这个变量用来控制是否开启窗口函数的支持。默认值 1 代表开启窗口函数的功能。
- 由于窗口函数会使用一些保留关键字，可能导致原先可以正常执行的 SQL 语句在升级 TiDB 后无法被解析语法，此时可以将 `tidb_enable_window_function` 设置为 `OFF`。

### `tidb_enforce_mpp` <span class="version-mark">从 v5.1 版本开始引入</span>

- 作用域：SESSION
- 默认值：`OFF`（表示关闭）。如需修改此变量的默认值，请配置 [`performance.enforce-mpp`](/tidb-configuration-file.md#enforce-mpp) 参数。
- 这个变量用于控制是否忽略优化器代价估算，强制使用 TiFlash 的 MPP 模式执行查询，可以设置的值包括：
    - 0 或 OFF，代表不强制使用 MPP 模式（默认）
    - 1 或 ON，代表将忽略代价估算，强制使用 MPP 模式。注意：只有当 `tidb_allow_mpp=true` 时该设置才生效。

MPP 是 TiFlash 引擎提供的分布式计算框架，允许节点之间的数据交换并提供高性能、高吞吐的 SQL 算法。MPP 模式选择的详细说明参见[控制是否选择 MPP 模式](/tiflash/use-tiflash.md#控制是否选择-mpp-模式)。

### `tidb_evolve_plan_baselines` <span class="version-mark">从 v4.0 版本开始引入</span>

- 作用域：SESSION | GLOBAL
- 是否持久化到集群：是
- 默认值：`OFF`
- 这个变量用于控制是否启用自动演进绑定功能。该功能的详细介绍和使用方法可以参考[自动演进绑定](/sql-plan-management.md#自动演进绑定-baseline-evolution)。
- 为了减少自动演进对集群的影响，可以进行以下配置：

    - 设置 `tidb_evolve_plan_task_max_time`，限制每个执行计划运行的最长时间，其默认值为 600s；
    - 设置`tidb_evolve_plan_task_start_time` 和 `tidb_evolve_plan_task_end_time`，限制运行演进任务的时间窗口，默认值分别为 `00:00 +0000` 和 `23:59 +0000`。

### `tidb_evolve_plan_task_end_time` <span class="version-mark">从 v4.0 版本开始引入</span>

- 作用域：GLOBAL
- 是否持久化到集群：是
- 默认值：`23:59 +0000`
- 这个变量用来设置一天中允许自动演进的结束时间。

### `tidb_evolve_plan_task_max_time` <span class="version-mark">从 v4.0 版本开始引入</span>

- 作用域：GLOBAL
- 是否持久化到集群：是
- 默认值：`600`
- 范围：`[-1, 9223372036854775807]`
- 单位：秒
- 该变量用于限制自动演进功能中，每个执行计划运行的最长时间。

### `tidb_evolve_plan_task_start_time` <span class="version-mark">从 v4.0 版本开始引入</span>

- 作用域：GLOBAL
- 是否持久化到集群：是
- 默认值：`00:00 +0000`
- 这个变量用来设置一天中允许自动演进的开始时间。

### `tidb_executor_concurrency` <span class="version-mark">从 v5.0 版本开始引入</span>

- 作用域：SESSION | GLOBAL
- 是否持久化到集群：是
- 默认值：`5`
- 范围：`[1, 256]`

变量用来统一设置各个 SQL 算子的并发度，包括：

- `index lookup`
- `index lookup join`
- `hash join`
- `hash aggregation`（partial 和 final 阶段）
- `window`
- `projection`

`tidb_executor_concurrency` 整合了已有的系统变量，方便管理。这些变量所列如下：

+ `tidb_index_lookup_concurrency`
+ `tidb_index_lookup_join_concurrency`
+ `tidb_hash_join_concurrency`
+ `tidb_hashagg_partial_concurrency`
+ `tidb_hashagg_final_concurrency`
+ `tidb_projection_concurrency`
+ `tidb_window_concurrency`

v5.0 后，用户仍可以单独修改以上系统变量（会有废弃警告），且修改只影响单个算子。后续通过 `tidb_executor_concurrency` 的修改也不会影响该算子。若要通过 `tidb_executor_concurrency` 来管理所有算子的并发度，需要将以上所列变量的值设置为 `-1`。

对于从 v5.0 之前的版本升级到 v5.0 的系统，如果用户对上述所列变量的值没有做过改动（即 `tidb_hash_join_concurrency` 值为 `5`，其他值为 `4`），则会自动转为使用 `tidb_executor_concurrency` 来统一管理算子并发度。如果用户对上述变量的值做过改动，则沿用之前的变量对相应的算子做并发控制。

### `tidb_expensive_query_time_threshold`

- 作用域：INSTANCE
- 默认值：`60`
- 范围：`[10, 2147483647]`
- 单位：秒
- 这个变量用来控制打印 expensive query 日志的阈值时间，默认值是 60 秒。expensive query 日志和慢日志的差别是，慢日志是在语句执行完后才打印，expensive query 日志可以把正在执行中的语句且执行时间超过阈值的语句及其相关信息打印出来。

### `tidb_force_priority`

- 作用域：INSTANCE
- 默认值：`NO_PRIORITY`
- 这个变量用于改变 TiDB server 上执行的语句的默认优先级。例如，你可以通过设置该变量来确保正在执行 OLAP 查询的用户优先级低于正在执行 OLTP 查询的用户。
- 可设置为 `NO_PRIORITY`、`LOW_PRIORITY`、`DELAYED` 或 `HIGH_PRIORITY`。

### `tidb_gc_concurrency` <span class="version-mark">从 v5.0 版本开始引入</span>

- 作用域：GLOBAL
- 是否持久化到集群：是
- 默认值：`-1`
- 范围：`[1, 256]`
- 这个变量用于指定 GC 在[Resolve Locks（清理锁）](/garbage-collection-overview.md#resolve-locks清理锁)步骤中线程的数量。默认值 `-1` 表示由 TiDB 自主判断运行 GC 要使用的线程的数量。

### `tidb_gc_enable` <span class="version-mark">从 v5.0 版本开始引入</span>

- 作用域：GLOBAL
- 是否持久化到集群：是
- 默认值：`ON`
- 这个变量用于控制是否启用 TiKV 的垃圾回收 (GC) 机制。如果不启用 GC 机制，系统将不再清理旧版本的数据，因此会有损系统性能。

### `tidb_gc_life_time` <span class="version-mark">从 v5.0 版本开始引入</span>

- 作用域：GLOBAL
- 是否持久化到集群：是
- 默认值：`10m0s`
- 范围：`[10m0s, 8760h0m0s]`
- 这个变量用于指定每次进行垃圾回收 (GC) 时保留数据的时限。变量值为 Go 的 Duration 字符串格式。每次进行 GC 时，将以当前时间减去该变量的值作为 safe point。

> **Note:**
>
> - 在数据频繁更新的场景下，将 `tidb_gc_life_time` 的值设置得过大（如数天甚至数月）可能会导致一些潜在的问题，如：
>     - 占用更多的存储空间。
>     - 大量的历史数据可能会在一定程度上影响系统性能，尤其是范围的查询（如 `select count(*) from t`）。
> - 如果一个事务的运行时长超过了 `tidb_gc_life_time` 配置的值，在 GC 时，为了使这个事务可以继续正常运行，系统会保留从这个事务开始时间 `start_ts` 以来的数据。例如，如果 `tidb_gc_life_time` 的值配置为 10 分钟，且在一次 GC 时，集群正在运行的事务中最早开始的那个事务已经运行了 15 分钟，那么本次 GC 将保留最近 15 分钟的数据。

### `tidb_gc_run_interval` <span class="version-mark">从 v5.0 版本开始引入</span>

- 作用域：GLOBAL
- 是否持久化到集群：是
- 默认值：`10m0s`
- 范围：`[10m0s, 8760h0m0s]`
- 这个变量用于指定垃圾回收 (GC) 运行的时间间隔。变量值为 Go 的 Duration 字符串格式，如`"1h30m"`、`"15m"`等。

### `tidb_gc_scan_lock_mode` <span class="version-mark">从 v5.0 版本开始引入</span>

> **警告：**
>
> Green GC 目前是实验性功能，不建议在生产环境中使用。

- 作用域：GLOBAL
- 是否持久化到集群：是
- 默认值：`LEGACY`
- 可设置为：`PHYSICAL`，`LEGACY`
    - `LEGACY`：使用旧的扫描方式，即禁用 Green GC。
    - `PHYSICAL`：使用物理扫描方式，即启用 Green GC。
- 这个变量用于指定垃圾回收 (GC) 的 Resolve Locks（清理锁）步骤中扫描锁的方式。当变量值设置为 `LEGACY` 时，TiDB 以 Region 为单位进行扫描。当变量值设置为 `PHYSICAL` 时，每个 TiKV 节点分别绕过 Raft 层直接扫描数据，可以有效地缓解在启用 [Hibernate Region](/tikv-configuration-file.md#hibernate-regions) 功能时，GC 唤醒全部 Region 的影响，从而提升 Resolve Locks（清理锁）这个步骤的执行速度。

### `tidb_general_log`

- 作用域：GLOBAL
- 是否持久化到集群：否
- 默认值：`OFF`
- 这个变量用来设置是否在[日志](/tidb-configuration-file.md#logfile)里记录所有的 SQL 语句。该功能默认关闭。如果系统运维人员在定位问题过程中需要追踪所有 SQL 记录，可考虑开启该功能。
- 通过查询 `"GENERAL_LOG"` 字符串可以定位到该功能在日志中的所有记录。日志会记录以下内容：
    - `conn`：当前会话对应的 ID
    - `user`：当前会话用户
    - `schemaVersion`：当前 schema 版本
    - `txnStartTS`：当前事务的开始时间戳
    - `forUpdateTS`：事务模式为悲观事务时，SQL 语句的当前时间戳。悲观事务内发生写冲突时，会重试当前执行语句，该时间戳会被更新。重试次数由 [`max-retry-count`](/tidb-configuration-file.md#max-retry-count) 配置。事务模式为乐观事务时，该条目与 `txnStartTS` 等价。
    - `isReadConsistency`：当前事务隔离级别是否是读已提交 (RC)
    - `current_db`：当前数据库名
    - `txn_mode`：事务模式。可选值：`OPTIMISTIC`（乐观事务模式），或 `PESSIMISTIC`（悲观事务模式）
    - `sql`：当前查询对应的 SQL 语句

### `tidb_hash_join_concurrency`

> **警告：**
>
> 从 v5.0 版本开始，该变量被废弃。请使用 [`tidb_executor_concurrency`](#tidb_executor_concurrency-从-v50-版本开始引入) 进行设置。

- 作用域：SESSION | GLOBAL
- 是否持久化到集群：是
- 默认值：`-1`
- 范围：`[1, 256]`
- 这个变量用来设置 hash join 算法的并发度。
- 默认值 `-1` 表示使用 `tidb_executor_concurrency` 的值。

### `tidb_hashagg_final_concurrency`

> **警告：**
>
> 从 v5.0 版本开始，该变量被废弃。请使用 [`tidb_executor_concurrency`](#tidb_executor_concurrency-从-v50-版本开始引入) 进行设置。

- 作用域：SESSION | GLOBAL
- 是否持久化到集群：是
- 默认值：`-1`
- 范围：`[1, 256]`
- 这个变量用来设置并行 hash aggregation 算法 final 阶段的执行并发度。对于聚合函数参数不为 distinct 的情况，HashAgg 分为 partial 和 final 阶段分别并行执行。
- 默认值 `-1` 表示使用 `tidb_executor_concurrency` 的值。

### `tidb_hashagg_partial_concurrency`

> **警告：**
>
> 从 v5.0 版本开始，该变量被废弃。请使用 [`tidb_executor_concurrency`](#tidb_executor_concurrency-从-v50-版本开始引入) 进行设置。

- 作用域：SESSION | GLOBAL
- 是否持久化到集群：是
- 默认值：`-1`
- 范围：`[1, 256]`
- 这个变量用来设置并行 hash aggregation 算法 partial 阶段的执行并发度。对于聚合函数参数不为 distinct 的情况，HashAgg 分为 partial 和 final 阶段分别并行执行。
- 默认值 `-1` 表示使用 `tidb_executor_concurrency` 的值。

### `tidb_ignore_prepared_cache_close_stmt`（从 v6.0 版本开始引入）

- 作用域：SESSION | GLOBAL
- 是否持久化到集群：是
- 默认值：`OFF`
- 这个变量用来设置是否忽略关闭 Prepared Statement 的指令。
- 如果变量值设为 `ON`，Binary 协议的 `COM_STMT_CLOSE` 信号和文本协议的 [`DEALLOCATE PREPARE`](/sql-statements/sql-statement-deallocate.md) 语句都会被忽略。

### `tidb_index_join_batch_size`

- 作用域：SESSION | GLOBAL
- 是否持久化到集群：是
- 默认值：`25000`
- 范围：`[1, 2147483647]`
- 这个变量用来设置 index lookup join 操作的 batch 大小，AP 类应用适合较大的值，TP 类应用适合较小的值。

### `tidb_index_lookup_concurrency`

> **警告：**
>
> 从 v5.0 版本开始，该变量被废弃。请使用 [`tidb_executor_concurrency`](#tidb_executor_concurrency-从-v50-版本开始引入) 进行设置。

- 作用域：SESSION | GLOBAL
- 是否持久化到集群：是
- 默认值：`-1`
- 范围：`[1, 256]`
- 这个变量用来设置 index lookup 操作的并发度，AP 类应用适合较大的值，TP 类应用适合较小的值。
- 默认值 `-1` 表示使用 `tidb_executor_concurrency` 的值。

### `tidb_index_lookup_join_concurrency`

> **警告：**
>
> 从 v5.0 版本开始，该变量被废弃。请使用 [`tidb_executor_concurrency`](#tidb_executor_concurrency-从-v50-版本开始引入) 进行设置。

- 作用域：SESSION | GLOBAL
- 是否持久化到集群：是
- 默认值：`-1`
- 范围：`[1, 256]`
- 这个变量用来设置 index lookup join 算法的并发度。
- 默认值 `-1` 表示使用 `tidb_executor_concurrency` 的值。

### `tidb_index_lookup_size`

- 作用域：SESSION | GLOBAL
- 是否持久化到集群：是
- 默认值：`20000`
- 范围：`[1, 2147483647]`
- 这个变量用来设置 index lookup 操作的 batch 大小，AP 类应用适合较大的值，TP 类应用适合较小的值。

### `tidb_index_serial_scan_concurrency`

- 作用域：SESSION | GLOBAL
- 是否持久化到集群：是
- 默认值：`1`
- 范围：`[1, 256]`
- 这个变量用来设置顺序 scan 操作的并发度，AP 类应用适合较大的值，TP 类应用适合较小的值。

### `tidb_init_chunk_size`

- 作用域：SESSION | GLOBAL
- 是否持久化到集群：是
- 默认值：`32`
- 范围：`[1, 32]`
- 这个变量用来设置执行过程中初始 chunk 的行数。默认值是 32，可设置的范围是 1～32。

### `tidb_isolation_read_engines` <span class="version-mark">从 v4.0 版本开始引入</span>

- 作用域：SESSION
- 默认值：`tikv,tiflash,tidb`
- 这个变量用于设置 TiDB 在读取数据时可以使用的存储引擎列表。

### `tidb_log_file_max_days` <span class="version-mark">从 v5.3.0 版本开始引入</span>

- 作用域：SESSION
- 默认值：`0`
- 范围：`[0, 2147483647]`
- 这个变量可以调整当前 TiDB 实例上日志的最大保留天数。默认值是实例配置文件中指定的值，见配置项 [`max-days`](/tidb-configuration-file.md#max-days)。此变量只影响当前 TiDB 实例上的配置，重启后丢失，且配置文件不受影响。

### `tidb_low_resolution_tso`

- 作用域：SESSION
- 默认值：`OFF`
- 这个变量用来设置是否启用低精度 TSO 特性。开启该功能之后，新事务会使用一个每 2s 更新一次的 TS 来读取数据。
- 主要场景是在可以容忍读到旧数据的情况下，降低小的只读事务获取 TSO 的开销。

### `tidb_max_chunk_size`

- 作用域：SESSION | GLOBAL
- 是否持久化到集群：是
- 默认值：`1024`
- 范围：`[32, 2147483647]`
- 这个变量用来设置执行过程中一个 chunk 最大的行数，设置过大可能引起缓存局部性的问题。

### `tidb_max_delta_schema_count`

- 作用域：GLOBAL
- 是否持久化到集群：是
- 默认值：`1024`
- 范围：`[100, 16384]`
- 这个变量用来设置缓存 schema 版本信息（对应版本修改的相关 table IDs）的个数限制，可设置的范围 100 - 16384。此变量在 2.1.18 及之后版本支持。

### `tidb_mem_quota_apply_cache` <span class="version-mark">从 v5.0 版本开始引入</span>

- 作用域：SESSION | GLOBAL
- 是否持久化到集群：是
- 默认值：`33554432` (32 MiB)
- 范围：`[0, 9223372036854775807]`
- 单位：字节
- 这个变量用来设置 `Apply` 算子中局部 Cache 的内存使用阈值。
- `Apply` 算子中局部 Cache 用来加速 `Apply` 算子的计算，该变量可以设置 `Apply` Cache 的内存使用阈值。设置变量值为 `0` 可以关闭 `Apply` Cache 功能。

### `tidb_mem_quota_binding_cache`（从 v6.0 版本开始引入）

- 作用域：GLOBAL
- 是否持久化到集群：是
- 默认值：`67108864` (64 MiB)
- 范围：`[0, 2147483647]`
- 单位：字节
- 这个变量用来设置存放 `binding` 的缓存的内存使用阈值。
- 如果一个系统创建或者捕获了过多的绑定，导致绑定所使用的内存空间超过该阈值，TiDB 会在日志中增加警告日志进行提示。这种情况下，缓存无法存放所有可用的绑定，并且无法保证哪些绑定存在于缓存中，因此，可能存在一些查询无法使用可用绑定的情况。此时，可以调大该变量的值，从而保证所有可用绑定都能正常使用。修改变量值以后，需要执行命令 `admin reload bindings` 重新加载绑定，确保变更生效。

### `tidb_mem_quota_query`

- 作用域：SESSION
- 默认值：`1073741824` (1 GiB)
- 范围：`[-1, 9223372036854775807]`
- 单位：字节
- 这个变量用来设置一条查询语句的内存使用阈值。
- 如果一条查询语句执行过程中使用的内存空间超过该阈值，会触发 TiDB 启动配置文件中 OOMAction 项所指定的行为。该变量的初始值由配置项 [`mem-quota-query`](/tidb-configuration-file.md#mem-quota-query) 配置。

### `tidb_memory_usage_alarm_ratio`

- 作用域：INSTANCE
- 默认值：`0.8`
- TiDB 内存使用占总内存的比例超过一定阈值时会报警。该功能的详细介绍和使用方法可以参考 [`memory-usage-alarm-ratio`](/tidb-configuration-file.md#memory-usage-alarm-ratio-从-v409-版本开始引入)。
- 该变量的初始值可通过 [`memory-usage-alarm-ratio`](/tidb-configuration-file.md#memory-usage-alarm-ratio-从-v409-版本开始引入) 进行配置。

### `tidb_metric_query_range_duration` <span class="version-mark">从 v4.0 版本开始引入</span>

- 作用域：SESSION
- 默认值：`60`
- 范围：`[10, 216000]`
- 单位：秒
- 这个变量设置了查询 `METRIC_SCHEMA` 时生成的 Prometheus 语句的 range duration。

### `tidb_metric_query_step` <span class="version-mark">从 v4.0 版本开始引入</span>

- 作用域：SESSION
- 默认值：`60`
- 范围：`[10, 216000]`
- 单位：秒
- 这个变量设置了查询 `METRIC_SCHEMA` 时生成的 Prometheus 语句的 step。

### `tidb_multi_statement_mode` <span class="version-mark">从 v4.0.11 版本开始引入</span>

- 作用域：SESSION | GLOBAL
- 是否持久化到集群：是
- 默认值：`OFF`
- 可选值：`OFF`，`ON`，`WARN`
- 该变量用于控制是否在同一个 `COM_QUERY` 调用中执行多个查询。
- 为了减少 SQL 注入攻击的影响，TiDB 目前默认不允许在同一 `COM_QUERY` 调用中执行多个查询。该变量可用作早期 TiDB 版本的升级路径选项。该变量值与是否允许多语句行为的对照表如下：

| 客户端设置         | `tidb_multi_statement_mode` 值 | 是否允许多语句 |
|------------------------|-----------------------------------|--------------------------------|
| Multiple Statements = ON  | OFF                               | 允许                            |
| Multiple Statements = ON  | ON                                | 允许                            |
| Multiple Statements = ON  | WARN                              | 允许                            |
| Multiple Statements = OFF | OFF                               | 不允许                             |
| Multiple Statements = OFF | ON                                | 允许                            |
| Multiple Statements = OFF | WARN                              | 允许 + 警告提示        |

> **注意：**
>
> 只有默认值 `OFF` 才是安全的。如果用户业务是专为早期 TiDB 版本而设计的，那么需要将该变量值设为 `ON`。如果用户业务需要多语句支持，建议用户使用客户端提供的设置，不要使用 `tidb_multi_statement_mode` 变量进行设置。

>
> * [go-sql-driver](https://github.com/go-sql-driver/mysql#multistatements) (`multiStatements`)
> * [Connector/J](https://dev.mysql.com/doc/connector-j/8.0/en/connector-j-reference-configuration-properties.html) (`allowMultiQueries`)
> * PHP [mysqli](https://dev.mysql.com/doc/apis-php/en/apis-php-mysqli.quickstart.multiple-statement.html) (`mysqli_multi_query`)

### `tidb_opt_agg_push_down`

- 作用域：SESSION
- 默认值：`OFF`
- 这个变量用来设置优化器是否执行聚合函数下推到 Join，Projection 和 UnionAll 之前的优化操作。当查询中聚合操作执行很慢时，可以尝试设置该变量为 ON。

### `tidb_opt_correlation_exp_factor`

- 作用域：SESSION | GLOBAL
- 是否持久化到集群：是
- 默认值：`1`
- 范围：`[0, 2147483647]`
- 当交叉估算方法不可用时，会采用启发式估算方法。这个变量用来控制启发式方法的行为。当值为 0 时不用启发式估算方法，大于 0 时，该变量值越大，启发式估算方法越倾向 index scan，越小越倾向 table scan。

### `tidb_opt_correlation_threshold`

- 作用域：SESSION | GLOBAL
- 是否持久化到集群：是
- 默认值：`0.9`
- 这个变量用来设置优化器启用交叉估算 row count 方法的阈值。如果列和 handle 列之间的顺序相关性超过这个阈值，就会启用交叉估算方法。
- 交叉估算方法可以简单理解为，利用这个列的直方图来估算 handle 列需要扫的行数。

### `tidb_opt_distinct_agg_push_down`

- 作用域：SESSION
- 默认值：`OFF`
- 这个变量用来设置优化器是否执行带有 `Distinct` 的聚合函数（比如 `select count(distinct a) from t`）下推到 Coprocessor 的优化操作。当查询中带有 `Distinct` 的聚合操作执行很慢时，可以尝试设置该变量为 `1`。

在以下示例中，`tidb_opt_distinct_agg_push_down` 开启前，TiDB 需要从 TiKV 读取所有数据，并在 TiDB 侧执行 `distinct`。`tidb_opt_distinct_agg_push_down` 开启后，`distinct a` 被下推到了 Coprocessor，在 `HashAgg_5` 里新增里一个 `group by` 列 `test.t.a`。

```sql
mysql> desc select count(distinct a) from test.t;
+-------------------------+----------+-----------+---------------+------------------------------------------+
| id                      | estRows  | task      | access object | operator info                            |
+-------------------------+----------+-----------+---------------+------------------------------------------+
| StreamAgg_6             | 1.00     | root      |               | funcs:count(distinct test.t.a)->Column#4 |
| └─TableReader_10        | 10000.00 | root      |               | data:TableFullScan_9                     |
|   └─TableFullScan_9     | 10000.00 | cop[tikv] | table:t       | keep order:false, stats:pseudo           |
+-------------------------+----------+-----------+---------------+------------------------------------------+
3 rows in set (0.01 sec)

mysql> set session tidb_opt_distinct_agg_push_down = 1;
Query OK, 0 rows affected (0.00 sec)

mysql> desc select count(distinct a) from test.t;
+---------------------------+----------+-----------+---------------+------------------------------------------+
| id                        | estRows  | task      | access object | operator info                            |
+---------------------------+----------+-----------+---------------+------------------------------------------+
| HashAgg_8                 | 1.00     | root      |               | funcs:count(distinct test.t.a)->Column#3 |
| └─TableReader_9           | 1.00     | root      |               | data:HashAgg_5                           |
|   └─HashAgg_5             | 1.00     | cop[tikv] |               | group by:test.t.a,                       |
|     └─TableFullScan_7     | 10000.00 | cop[tikv] | table:t       | keep order:false, stats:pseudo           |
+---------------------------+----------+-----------+---------------+------------------------------------------+
4 rows in set (0.00 sec)
```

### tidb_opt_enable_correlation_adjustment

- 作用域：SESSION | GLOBAL
- 是否持久化到集群：是
- 默认值：`ON`
- 这个变量用来控制优化器是否开启交叉估算。

### `tidb_opt_insubq_to_join_and_agg`

- 作用域：SESSION | GLOBAL
- 是否持久化到集群：是
- 默认值：`ON`
- 这个变量用来设置是否开启优化规则：将子查询转成 join 和 aggregation。

    例如，打开这个优化规则后，会将下面子查询做如下变化：

    {{< copyable "sql" >}}

    ```sql
    select * from t where t.a in (select aa from t1);
    ```

    将子查询转成如下 join：

    {{< copyable "sql" >}}

    ```sql
    select t.* from t, (select aa from t1 group by aa) tmp_t where t.a = tmp_t.aa;
    ```

    如果 t1 在列 `aa` 上有 unique 且 not null 的限制，可以直接改写为如下，不需要添加 aggregation。

    {{< copyable "sql" >}}

    ```sql
    select t.* from t, t1 where t.a=t1.aa;
    ```

### `tidb_opt_limit_push_down_threshold`

- 作用域：SESSION | GLOBAL
- 是否持久化到集群：是
- 默认值：`100`
- 范围：`[0, 2147483647]`
- 这个变量用来设置将 Limit 和 TopN 算子下推到 TiKV 的阈值。
- 如果 Limit 或者 TopN 的取值小于等于这个阈值，则 Limit 和 TopN 算子会被强制下推到 TiKV。该变量可以解决部分由于估算误差导致 Limit 或者 TopN 无法被下推的问题。

### `tidb_opt_prefer_range_scan` <span class="version-mark">从 v5.0 版本开始引入</span>

- 作用域：SESSION | GLOBAL
- 是否持久化到集群：是
- 默认值：`OFF`
- 将该变量值设为 `ON` 后，优化器总是偏好区间扫描而不是全表扫描。
- 在以下示例中，`tidb_opt_prefer_range_scan` 开启前，TiDB 优化器需要执行全表扫描。`tidb_opt_prefer_range_scan` 开启后，优化器选择了索引区间扫描。

```sql
explain select * from t where age=5;
+-------------------------+------------+-----------+---------------+-------------------+
| id                      | estRows    | task      | access object | operator info     |
+-------------------------+------------+-----------+---------------+-------------------+
| TableReader_7           | 1048576.00 | root      |               | data:Selection_6  |
| └─Selection_6           | 1048576.00 | cop[tikv] |               | eq(test.t.age, 5) |
|   └─TableFullScan_5     | 1048576.00 | cop[tikv] | table:t       | keep order:false  |
+-------------------------+------------+-----------+---------------+-------------------+
3 rows in set (0.00 sec)

set session tidb_opt_prefer_range_scan = 1;

explain select * from t where age=5;
+-------------------------------+------------+-----------+-----------------------------+-------------------------------+
| id                            | estRows    | task      | access object               | operator info                 |
+-------------------------------+------------+-----------+-----------------------------+-------------------------------+
| IndexLookUp_7                 | 1048576.00 | root      |                             |                               |
| ├─IndexRangeScan_5(Build)     | 1048576.00 | cop[tikv] | table:t, index:idx_age(age) | range:[5,5], keep order:false |
| └─TableRowIDScan_6(Probe)     | 1048576.00 | cop[tikv] | table:t                     | keep order:false              |
+-------------------------------+------------+-----------+-----------------------------+-------------------------------+
3 rows in set (0.00 sec)
```

### `tidb_opt_write_row_id`

- 作用域：SESSION
- 默认值：`OFF`
- 这个变量用来设置是否允许 `INSERT`、`REPLACE` 和 `UPDATE` 操作 `_tidb_rowid` 列，默认是不允许操作。该选项仅用于 TiDB 工具导数据时使用。

### `tidb_partition_prune_mode` <span class="version-mark">从 v5.1 版本开始引入</span>

> **警告：**
>
> 目前分区表动态裁剪模式为实验特性，不建议在生产环境中使用。

- 作用域：SESSION | GLOBAL
- 是否持久化到集群：是
- 默认值：static
- 这个变量用来设置是否开启分区表动态裁剪模式。关于动态裁剪模式的详细说明请参阅[分区表动态裁剪模式](/partitioned-table.md#动态裁剪模式)。

### `tidb_persist_analyze_options` <span class="version-mark">从 v5.4.0 版本开始引入</span>

- 作用域：GLOBAL
- 是否持久化到集群：是
- 默认值：`ON`
- 这个变量用于控制是否开启 [ANALYZE 配置持久化](/statistics.md#analyze-配置持久化)特性。

### `tidb_placement_mode`（从 v6.0.0 版本开始引入）

- 作用域：SESSION | GLOBAL
- 是否持久化到集群：是
- 默认值：`STRICT`
- 可选值：`STRICT`，`IGNORE`
- 该变量用于控制 DDL 语句是否忽略 [Placement Rules in SQL](/placement-rules-in-sql.md) 指定的放置规则。变量值为 `IGNORE` 时将忽略所有放置规则选项。
- 该变量可由逻辑转储或逻辑恢复工具使用，确保即使绑定了不合适的放置规则，也始终可以成功创建表。这类似于 mysqldump 将 `SET FOREIGN_KEY_CHECKS=0;` 写入每个转储文件的开头部分。

### `tidb_pprof_sql_cpu` <span class="version-mark">从 v4.0 版本开始引入</span>

- 作用域：INSTANCE
- 默认值：`0`
- 范围：`[0, 1]`
- 这个变量用来控制是否在 profile 输出中标记出对应的 SQL 语句，用于定位和排查性能问题。

### `tidb_projection_concurrency`

> **警告：**
>
> 从 v5.0 版本开始，该变量被废弃。请使用 [`tidb_executor_concurrency`](#tidb_executor_concurrency-从-v50-版本开始引入) 进行设置。

- 作用域：SESSION | GLOBAL
- 是否持久化到集群：是
- 默认值：`-1`
- 范围：`[-1, 256]`
- 这个变量用来设置 `Projection` 算子的并发度。
- 默认值 `-1` 表示使用 `tidb_executor_concurrency` 的值。

### `tidb_query_log_max_len`

- 作用域：INSTANCE
- 默认值：`4096` (4 KiB)
- 范围：`[-1, 9223372036854775807]`
- 单位：字节
- 最长的 SQL 输出长度。当语句的长度大于 query-log-max-len，将会被截断输出。

示例：

{{< copyable "sql" >}}

```sql
SET tidb_query_log_max_len = 20;
```

### `tidb_rc_read_check_ts`（从 v6.0.0 版本开始引入）

> **警告：**
>
> - 该特性与 [`replica-read`](#tidb_replica_read-从-v40-版本开始引入) 尚不兼容，开启 `tidb_rc_read_check_ts` 的读请求无法使用 [`replica-read`](#tidb_replica_read-从-v40-版本开始引入)，请勿同时开启两项特性。
> - 如果客户端使用游标操作，建议不开启 `tidb_rc_read_check_ts` 这一特性，避免前一批返回数据已经被客户端使用而语句最终会报错的情况。

- 作用域：SESSION | GLOBAL
- 是否持久化到集群：是
- 默认值：`OFF`
- 该变量用于优化时间戳的获取，适用于悲观事务 `READ-COMMITTED` 隔离级别下读写冲突较少的场景，开启此变量可以避免获取全局 timestamp 带来的延迟和开销，并优化事务内读语句延迟。
- 如果读写冲突较为严重，开启此功能会增加额外开销和延迟，造成性能回退。更详细的说明，请参考[读已提交隔离级别 (Read Committed) 文档](/transaction-isolation-levels.md#读已提交隔离级别-read-committed)。

### `tidb_read_staleness` <span class="version-mark">从 v5.4.0 版本开始引入</span>

- 作用域：SESSION
- 默认值：`0`
- 范围 `[-2147483648, 0]`
- 这个变量用于设置当前会话允许读取的历史数据范围。设置后，TiDB 会从参数允许的范围内选出一个尽可能新的时间戳，并影响后继的所有读操作。比如，如果该变量的值设置为 `-5`，TiDB 会在 5 秒时间范围内，保证 TiKV 拥有对应历史版本数据的情况下，选择尽可能新的一个时间戳。

### `tidb_record_plan_in_slow_log`

- 作用域：INSTANCE
- 默认值：`ON`
- 这个变量用于控制是否在 slow log 里包含慢查询的执行计划。

### `tidb_redact_log`

- 作用域：SESSION | GLOBAL
- 是否持久化到集群：是
- 默认值：`OFF`
- 这个变量用于控制在记录 TiDB 日志和慢日志时，是否将 SQL 中的用户信息遮蔽。
- 将该变量设置为 `1` 即开启后，假设执行的 SQL 为 `insert into t values (1,2)`，在日志中记录的 SQL 会是 `insert into t values (?,?)`，即用户输入的信息被遮蔽。

### `tidb_regard_null_as_point` <span class="version-mark">从 v5.4.0 版本开始引入</span>

- 作用域：SESSION | GLOBAL
- 是否持久化到集群：是
- 默认值：`ON`
- 这个变量用来控制优化器是否可以将包含 null 的等值条件作为前缀条件来访问索引。
- 该变量默认开启。开启后，该变量可以使优化器减少需要访问的索引数据量，从而提高查询的执行速度。例如，在有多列索引 `index(a, b)` 且查询条件为 `a<=>null and b=1` 的情况下，优化器可以同时使用查询条件中的 `a<=>null` 和 `b=1` 进行索引访问。如果关闭该变量，因为 `a<=>null and b=1` 包含 null 的等值条件，优化器不会使用 `b=1` 进行索引访问。

### `tidb_replica_read` <span class="version-mark">从 v4.0 版本开始引入</span>

- 作用域：SESSION
- 是否持久化到集群：是
- 默认值：`leader`
- 可选值：`leader`，`follower`，`leader-and-follower`
- 这个变量用于控制 TiDB 读取数据的位置，有以下三个选择：

    * leader：只从 leader 节点读取
    * follower：只从 follower 节点读取
    * leader-and-follower：从 leader 或 follower 节点读取

更多细节，见 [Follower Read](/follower-read.md)。

### `tidb_retry_limit`

- 作用域：SESSION | GLOBAL
- 是否持久化到集群：是
- 默认值：`10`
- 范围：`[-1, 9223372036854775807]`
- 这个变量用来设置乐观事务的最大重试次数。一个事务执行中遇到可重试的错误（例如事务冲突、事务提交过慢或表结构变更）时，会根据该变量的设置进行重试。注意当 `tidb_retry_limit = 0` 时，也会禁用自动重试。该变量仅适用于乐观事务，不适用于悲观事务。

### `tidb_row_format_version`

- 作用域：GLOBAL
- 是否持久化到集群：是
- 默认值：`2`
- 范围：`[1, 2]`
- 控制新保存数据的表数据格式版本。TiDB v4.0 中默认使用版本号为 2 的[新表数据格式](https://github.com/pingcap/tidb/blob/master/docs/design/2018-07-19-row-format.md)保存新数据。

- 但如果从 4.0.0 之前的版本升级到 4.0.0，不会改变表数据格式版本，TiDB 会继续使用版本为 1 的旧格式写入表中，即**只有新创建的集群才会默认使用新表数据格式**。

- 需要注意的是修改该变量不会对已保存的老数据产生影响，只会对修改变量后的新写入数据使用对应版本格式保存。

### `tidb_scatter_region`

- 作用域：GLOBAL
- 是否持久化到集群：是
- 默认值：`OFF`
- TiDB 默认会在建表时为新表分裂 Region。开启该变量后，会在建表语句执行时，同步打散刚分裂出的 Region。适用于批量建表后紧接着批量写入数据，能让刚分裂出的 Region 先在 TiKV 分散而不用等待 PD 进行调度。为了保证后续批量写入数据的稳定性，建表语句会等待打散 Region 完成后再返回建表成功，建表语句执行时间会是该变量关闭时的数倍。
- 如果建表时设置了 `SHARD_ROW_ID_BITS` 和 `PRE_SPLIT_REGIONS`，建表成功后会均匀切分出指定数量的 Region。

### `tidb_skip_ascii_check` <span class="version-mark">从 v5.0 版本开始引入</span>

- 作用域：SESSION | GLOBAL
- 是否持久化到集群：是
- 默认值：`OFF`
- 这个变量用来设置是否校验 ASCII 字符的合法性。
- 校验 ASCII 字符会损耗些许性能。当你确认输入的字符串为有效的 ASCII 字符时，可以将其设置为 `ON`。

### `tidb_skip_isolation_level_check`

- 作用域：SESSION | GLOBAL
- 是否持久化到集群：是
- 默认值：`OFF`
- 开启这个开关之后，如果对 `tx_isolation` 赋值一个 TiDB 不支持的隔离级别，不会报错，有助于兼容其他设置了（但不依赖于）不同隔离级别的应用。

```sql
tidb> set tx_isolation='serializable';
ERROR 8048 (HY000): The isolation level 'serializable' is not supported. Set tidb_skip_isolation_level_check=1 to skip this error
tidb> set tidb_skip_isolation_level_check=1;
Query OK, 0 rows affected (0.00 sec)

tidb> set tx_isolation='serializable';
Query OK, 0 rows affected, 1 warning (0.00 sec)
```

### `tidb_skip_utf8_check`

- 作用域：SESSION | GLOBAL
- 是否持久化到集群：是
- 默认值：`OFF`
- 这个变量用来设置是否校验 UTF-8 字符的合法性。
- 校验 UTF-8 字符会损耗些许性能。当你确认输入的字符串为有效的 UTF-8 字符时，可以将其设置为 `ON`。

### `tidb_slow_log_threshold`

- 作用域：INSTANCE
- 默认值：`300`
- 范围：`[-1, 9223372036854775807]`
- 单位：毫秒
- 输出慢日志的耗时阈值。当查询大于这个值，就会当做是一个慢查询，输出到慢查询日志。默认为 300 ms。

示例：

{{< copyable "sql" >}}

```sql
set tidb_slow_log_threshold = 200;
```

### `tidb_slow_query_file`

- 作用域：SESSION
- 默认值：""
- 查询 `INFORMATION_SCHEMA.SLOW_QUERY` 只会解析配置文件中 `slow-query-file` 设置的慢日志文件名，默认是 "tidb-slow.log"。但如果想要解析其他的日志文件，可以通过设置 session 变量 `tidb_slow_query_file` 为具体的文件路径，然后查询 `INFORMATION_SCHEMA.SLOW_QUERY` 就会按照设置的路径去解析慢日志文件。更多详情可以参考 [SLOW_QUERY 文档](/identify-slow-queries.md)。

### `tidb_snapshot`

- 作用域：SESSION
- 默认值：""
- 这个变量用来设置当前会话期待读取的历史数据所处时刻。比如当设置为 `"2017-11-11 20:20:20"` 时或者一个 TSO 数字 "400036290571534337"，当前会话将能读取到该时刻的数据。

### `tidb_stats_load_sync_wait` <span class="version-mark">从 v5.4.0 版本开始引入</span>

> **警告：**
>
> 统计信息同步加载目前为实验性特性，不建议在生产环境中使用。

- 作用域：SESSION | GLOBAL
- 是否持久化到集群：是
- 默认值：`0`
- 单位：毫秒
- 范围：`[0, 2147483647]`
- 这个变量用于控制是否开启统计信息的同步加载模式（默认为 `0` 代表不开启，即为异步加载模式），以及开启的情况下，SQL 执行同步加载完整统计信息等待多久后会超时。更多信息，请参考[统计信息的加载](/statistics.md#统计信息的加载)。

### `tidb_stats_load_pseudo_timeout` <span class="version-mark">从 v5.4.0 版本开始引入</span>

> **警告：**
>
> 统计信息同步加载目前为实验性特性，不建议在生产环境中使用。

- 作用域：GLOBAL
- 是否持久化到集群：是
- 默认值：`OFF`
- 这个变量用于控制统计信息同步加载超时后，SQL 是执行失败（`OFF`），还是退回使用 pseudo 的统计信息（`ON`）。

### `tidb_stmt_summary_history_size` <span class="version-mark">从 v4.0 版本开始引入</span>

- 作用域：GLOBAL
- 是否持久化到集群：是
- 默认值：`24`
- 范围：`[0, 255]`
- 这个变量设置了 [statement summary tables](/statement-summary-tables.md) 的历史记录容量。

### `tidb_stmt_summary_internal_query` <span class="version-mark">从 v4.0 版本开始引入</span>

- 作用域：GLOBAL
- 是否持久化到集群：是
- 默认值：`OFF`
- 这个变量用来控制是否在 [statement summary tables](/statement-summary-tables.md) 中包含 TiDB 内部 SQL 的信息。

### `tidb_stmt_summary_max_sql_length` <span class="version-mark">从 v4.0 版本开始引入</span>

- 作用域：GLOBAL
- 是否持久化到集群：是
- 默认值：`4096`
- 范围：`[0, 2147483647]`
- 这个变量控制 [statement summary tables](/statement-summary-tables.md) 显示的 SQL 字符串长度。

### `tidb_stmt_summary_max_stmt_count` <span class="version-mark">从 v4.0 版本开始引入</span>

- 作用域：GLOBAL
- 是否持久化到集群：是
- 默认值：`3000`
- 范围：`[1, 32767]`
- 这个变量设置了 [statement summary tables](/statement-summary-tables.md) 在内存中保存的语句的最大数量。

### `tidb_stmt_summary_refresh_interval` <span class="version-mark">从 v4.0 版本开始引入</span>

- 作用域：GLOBAL
- 是否持久化到集群：是
- 默认值：`1800`
- 范围：`[1, 2147483647]`
- 单位：秒
- 这个变量设置了 [statement summary tables](/statement-summary-tables.md) 的刷新时间。

### `tidb_top_sql_max_meta_count` <span class="version-mark">从 v6.0.0 版本开始引入</span>

- 作用域：GLOBAL
- 是否持久化到集群：是
- 默认值：`5000`
- 范围：`[1, 10000]`
- 这个变量用于控制 [Top SQL](/dashboard/top-sql.md) 每分钟最多收集 SQL 语句类型的数量。

### `tidb_top_sql_max_time_series_count` <span class="version-mark">从 v6.0.0 版本开始引入</span>

- 作用域：GLOBAL
- 是否持久化到集群：是
- 默认值：`100`
- 范围：`[1, 5000]`
- 这个变量用于控制 [Top SQL](/dashboard/top-sql.md) 每分钟保留消耗负载最大的前多少条 SQL（即 Top N) 的数据。

> **注意：**
>
> TiDB Dashboard 中的 Top SQL 页面目前只显示消耗负载最多的 5 类 SQL 查询，这与 `tidb_top_sql_max_time_series_count` 的配置无关。

### `tidb_store_limit` <span class="version-mark">从 v3.0.4 和 v4.0 版本开始引入</span>

- 作用域：GLOBAL
- 是否持久化到集群：是
- 默认值：`0`
- 范围：`[0, 9223372036854775807]`
- 这个变量用于限制 TiDB 同时向 TiKV 发送的请求的最大数量，0 表示没有限制。

### `tidb_sysdate_is_now`（从 v6.0.0 版本开始引入）

- 作用域：SESSION | GLOBAL
- 是否持久化到集群：是
- 默认值：`OFF`
- 这个变量用于控制 `SYSDATE` 函数能否替换为 `NOW` 函数，其效果与 MYSQL 中的 [`sysdate-is-now`](https://dev.mysql.com/doc/refman/8.0/en/server-options.html#option_mysqld_sysdate-is-now) 一致。

### `tidb_table_cache_lease`（从 v6.0.0 版本开始引入）

- 作用域：GLOBAL
- 是否持久化到集群：是
- 默认值：`3`
- 范围：`[1, 10]`
- 单位：秒
- 这个变量用来控制[缓存表](/cached-tables.md)的 lease 时间，默认值是 3 秒。该变量值的大小会影响缓存表的修改。在缓存表上执行修改操作后，最长可能出现 `tidb_table_cache_lease` 变量值时长的等待。如果业务表为只读表，或者能接受很高的写入延迟，则可以将该变量值调大，从而增加缓存的有效时间，减少 lease 续租的频率。

### `tidb_tmp_table_max_size` <span class="version-mark">从 v5.3 版本开始引入</span>

- 作用域：SESSION | GLOBAL
- 是否持久化到集群：是
- 默认值：`67108864`
- 范围：`[1048576, 137438953472]`
- 单位：字节
- 这个变量用于限制单个[临时表](/temporary-tables.md)的最大大小，临时表超出该大小后报错。

### `tidb_tso_client_batch_max_wait_time` <span class="version-mark">从 v5.3.0 版本开始引入</span>

- 作用域：GLOBAL
- 是否持久化到集群：是
- 默认值：`0`
- 范围：`[0, 10]`
- 单位：毫秒
- 这个变量用来设置 TiDB 向 PD 请求 TSO 时进行一次攒批操作的最大等待时长。默认值为 `0`，即不进行额外的等待。
- 在向 PD 获取 TSO 请求时，TiDB 使用的 PD Client 会一次尽可能多地收集同一时刻的 TSO 请求，将其攒批合并成一个 RPC 请求后再发送给 PD，从而减轻 PD 的压力。
- 将这个变量值设置为非 0 后，TiDB 会在每一次攒批结束前进行一个最大时长为其值的等待，目的是为了收集到更多的 TSO 请求，从而提高攒批效果。
- 适合调高这个变量值的场景：
    * PD leader 因高压力的 TSO 请求而达到 CPU 瓶颈，导致 TSO RPC 请求的延迟较高。
    * 集群中 TiDB 实例的数量不多，但每一台 TiDB 实例上的并发量较高。
- 在实际使用中，推荐将该变量尽可能设置为一个较小的值。

> **注意：**
>
> 如果 PD leader 的 TSO RPC 延迟升高，但其现象并非由 CPU 使用率达到瓶颈而导致（可能存在网络等问题），此时，调高 `tidb_tso_client_batch_max_wait_time` 可能会导致 TiDB 的语句执行延迟上升，影响集群的 QPS 表现。

### `tidb_txn_assertion_level`（从 v6.0.0 版本开始引入）

- 作用域：SESSION | GLOBAL
- 是否持久化到集群：是
- 默认值：`OFF`
- 可选值：`OFF`，`FAST`，`STRICT`
- 这个变量用于设置 assertion 级别。assertion 是一项在事务提交过程中进行的数据索引一致性校验，它对正在写入的 key 是否存在进行检查。如果不符则说明数据索引不一致，会导致事务 abort。详见[数据索引一致性报错](/troubleshoot-data-inconsistency-errors.md)。

    - `OFF`: 关闭该检查。
    - `FAST`: 开启大多数检查项，对性能几乎无影响。
    - `STRICT`: 开启全部检查项，当系统负载较高时，对悲观事务的性能有较小影响。

### `tidb_txn_mode`

- 作用域：SESSION | GLOBAL
- 是否持久化到集群：是
- 默认值：`pessimistic`
- 可选值：`pessimistic`，`optimistic`
- 这个变量用于设置事务模式。TiDB v3.0 支持了悲观事务，自 v3.0.8 开始，默认使用[悲观事务模式](/pessimistic-transaction.md)。
- 但如果从 3.0.7 及之前的版本升级到 >= 3.0.8 的版本，不会改变默认事务模式，即**只有新创建的集群才会默认使用悲观事务模式**。
- 将该变量设置为 "optimistic" 或 "" 时，将会使用[乐观事务模式](/optimistic-transaction.md)。

### `tidb_use_plan_baselines`（从 v4.0 版本开始引入）

- 作用域：SESSION | GLOBAL
- 是否持久化到集群：是
- 默认值：`ON`
- 这个变量用于控制是否开启执行计划绑定功能，默认打开，可通过赋值 `OFF` 来关闭。关于执行计划绑定功能的使用可以参考[执行计划绑定文档](/sql-plan-management.md#创建绑定)。

### `tidb_wait_split_region_finish`

- 作用域：SESSION
- 默认值：`ON`
- 由于打散 Region 的时间可能比较长，主要由 PD 调度以及 TiKV 的负载情况所决定。这个变量用来设置在执行 `SPLIT REGION` 语句时，是否同步等待所有 Region 都打散完成后再返回结果给客户端。
    - 默认 `ON` 代表等待打散完成后再返回结果
    - `OFF` 代表不等待 Region 打散完成就返回。
- 需要注意的是，在 Region 打散期间，对正在打散 Region 上的写入和读取的性能会有一定影响，对于批量写入、导数据等场景，还是建议等待 Region 打散完成后再开始导数据。

### `tidb_wait_split_region_timeout`

- 作用域：SESSION
- 默认值：`300`
- 范围：`[1, 2147483647]`
- 单位：秒
- 这个变量用来设置 `SPLIT REGION` 语句的执行超时时间，默认值是 300 秒，如果超时还未完成，就返回一个超时错误。

### `tidb_window_concurrency` <span class="version-mark">从 v4.0 版本开始引入</span>

> **警告：**
>
> 从 v5.0 版本开始，该变量被废弃。请使用 [`tidb_executor_concurrency`](#tidb_executor_concurrency-从-v50-版本开始引入) 进行设置。

- 作用域：SESSION | GLOBAL
- 是否持久化到集群：是
- 默认值：`-1`
- 范围：`[1, 256]`
- 这个变量用于设置 window 算子的并行度。
- 默认值 `-1` 表示使用 `tidb_executor_concurrency` 的值。

### `time_zone`

- 作用域：SESSION | GLOBAL
- 是否持久化到集群：是
- 默认值：`SYSTEM`
- 数据库所使用的时区。这个变量值可以写成时区偏移的形式，如 '-8:00'，也可以写成一个命名时区，如 'America/Los_Angeles'。
- 默认值 `SYSTEM` 表示时区应当与系统主机的时区相同。系统的时区可通过 [`system_time_zone`](#system_time_zone) 获取。

### `timestamp`

- 作用域：SESSION
- 默认值：`0`
- 一个 Unix 时间戳。变量值非空时，表示 `CURRENT_TIMESTAMP()`、`NOW()` 等函数的时间戳。该变量通常用于数据恢复或数据复制。

### `transaction_isolation`

- 作用域：SESSION | GLOBAL
- 是否持久化到集群：是
- 默认值：`REPEATABLE-READ`
- 可选值：`READ-UNCOMMITTED`，`READ-COMMITTED`，`REPEATABLE-READ`，`SERIALIZABLE`
- 这个变量用于设置事务隔离级别。TiDB 为了兼容 MySQL，支持可重复读 (`REPEATABLE-READ`)，但实际的隔离级别是快照隔离。详情见[事务隔离级别](/transaction-isolation-levels.md)。

### `tx_isolation`

这个变量是 `transaction_isolation` 的别名。

### `version`

- 作用域：NONE
- 默认值：`5.7.25-TiDB-(tidb version)`
- 这个变量的值是 MySQL 的版本和 TiDB 的版本，例如 '5.7.25-TiDB-v4.0.0-beta.2-716-g25e003253'。

### `version_comment`

- 作用域：NONE
- 默认值：(string)
- 这个变量的值是 TiDB 版本号的其他信息，例如 'TiDB Server (Apache License 2.0) Community Edition, MySQL 5.7 compatible'。

### `version_compile_os`

- 作用域：NONE
- 默认值：(string)
- 这个变量值是 TiDB 所在操作系统的名称。

### `version_compile_machine`

- 作用域：NONE
- 默认值：(string)
- 这个变量值是运行 TiDB 的 CPU 架构的名称。

### `wait_timeout`

- 作用域：SESSION | GLOBAL
- 是否持久化到集群：是
- 默认值：`28800`
- 范围：`[0, 31536000]`
- 单位：秒
- 这个变量表示用户会话的空闲超时。`0` 代表没有时间限制。

### `warning_count`

- 作用域：SESSION
- 默认值：`0`
- 这个只读变量表示之前执行语句中出现的警告数。

### `windowing_use_high_precision`

- 作用域：SESSION | GLOBAL
- 是否持久化到集群：是
- 默认值：`ON`
- 这个变量用于控制计算窗口函数时是否采用高精度模式。
