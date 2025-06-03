---
title: "mysql日志"
date: 2022-05-24T19:38:37+08:00
author: ["loveyu"]
draft: false
categories: 
- databases
tags: 
- databases
- mysql
---

# UndoLog

>悔做日志（Undo Log）是数据库事务管理的一个关键组成部分，尤其是在支持事务的存储引擎（如 MySQL 的 InnoDB 存储引擎）中。Undo Log 主要用于记录事务对数据的修改操作的反向操作，以便在事务回滚时撤销这些修改。在 InnoDB 存储引擎中，Undo Log 用于实现事务的原子性（Atomicity）和隔离性（Isolation）。

## 功能

1. **事务回滚**：当一个事务需要回滚时（例如，事务执行过程中遇到错误或用户手动执行 ROLLBACK 命令），Undo Log 中存储的反向操作会被用来撤销事务对数据的所有更改，从而实现事务的原子性。

2. **多版本并发控制（MVCC）**：InnoDB 存储引擎使用多版本并发控制（MVCC）技术来实现事务的隔离性。在 MVCC 模型中，不同事务对相同数据的修改不会直接覆盖数据，而是在 Undo Log 中生成一个数据的旧版本。这使得在同一时间点，不同事务可以看到数据的不同版本，从而实现事务的隔离性。

## 工作流程

1. 当一个事务开始执行时，InnoDB 会为该事务分配一个新的 Undo Log。如果该事务修改了数据，InnoDB 会将修改前的数据存储在 Undo Log 中。

2. 如果事务正常提交，Undo Log 中的记录会在一段时间后被清理。这个过程通常在下一个检查点（Checkpoint）时发生，因为检查点会确保所有相关的数据更改已经被持久化到磁盘。

3. 如果事务需要回滚，InnoDB 会使用 Undo Log 中的记录来撤销事务所做的所有更改。这样，数据可以恢复到事务开始前的状态，从而实现事务的原子性。

4. 在并发事务的情况下，Undo Log 中存储的数据旧版本用于实现多版本并发控制。当一个事务需要读取数据时，InnoDB 会根据该事务的隔离级别和 Undo Log 中的记录来确定应该返回哪个版本的数据。

# BinaryLog

>binlog 用于记录数据库执行的写入性操作(不包括查询)信息，以二进制的形式保存在磁盘中。由MySQL 的 Server 层实现的，所有引擎都可以使用
>
>BinLog是逻辑日志，记录的是这个语句的原始逻辑，比如“给 ID=2 这一行的 c 字段加 1 ”。
>
>binlog 是可以追加写入的。“追加写”是指 binlog 文件写到一定大小后会切换到下一个，并不会覆盖以前的日志。
>
>使用场景：主从复制和数据恢复
>
>查看BinLog日志模式: `select @@binlog_format` 

### 数据模式

#### RowLevel行模式

> Row Level 模式会记录每一行数据被修改的形式，在slave端再对相同的数据进行修改。这种模式日志记录最为完整，但是日志量大。日志格式如下：

1. insert 语句的 binlog 里会记录所有的字段信息，这些信息可以用来精确定位刚刚被插入的那一行。这时，你直接把 insert 语句转成 delete 语句，删除掉这被误插入的一行数据就可以了。

2. 如果执行的是 update 语句的话，binlog 里面会记录修改前整行的数据和修改后的整行数据。所以，如果你误执行了 update 语句的话，只需要把这个 event 前后的两行信息对调一下，再去数据库里面执行，就能恢复这个更新操作了。



#### Statement Level（默认）

>记录每一条修改数据的sql，slave在复制的时候sql进程会解析成和原来master端执行过的相同的sql来再次执行。这种模式能够减少减少bin-log日志量，但是在修改数据的时候使用了某些定的函数或者功能的时候会出现问题（主备不一致）。
>
>这种格式的日志是`存在风险`的，比如以上诉delete语句为例，因为带了limit，可能会导致主备不一致，比如，在主库上delete语句选择了索引a，删除的是索引a找到的第一个满足条件的行，如果在备库上选择的是索引b，那么删除的是索引b找到第一个满足条件的行。

#### Mixed 自动模式

> MySQL会根据执行的每一条具体的sql语句来区分对待记录的日志格式，也就是在Statement和Row之间选择一种。MySQL 自己会判断这条 SQL 语句是否可能引起主备不一致，如果有可能，就用 row 格式，否则就用 statement 格式（一般如果sql语句是update或者delete等修改数据的语句，那么还是会记录所有行的变更。）

### BinLog写入机制

>当一个事务执行完成并提交时，MySQL 会将事务中的更改写入 binlog。在事务提交之前，这些更改被缓存在 binlog 缓存中。当事务提交时，MySQL 会将缓存中的 binlog 事件刷新到磁盘。

每个线程有自己 binlog cache，但是共用同一份 binlog 文件。

write，指的就是指把日志写入到文件系统的 page cache，并没有把数据持久化到磁盘，所以速度比较快。

fsync，才是将数据持久化到磁盘的操作。一般情况下，我们认为 fsync 才占磁盘的 IOPS。

write 和 fsync 的时机，是由参数 sync_binlog 控制。

- sync_binlog=0 的时候，表示每次提交事务都只 write，不 fsync。
- sync_binlog=1 的时候，表示每次提交事务都会执行 fsync。
- sync_binlog=N(N>1) 的时候，表示每次提交事务都 write，但累积 N 个事务后才 fsync。

将 sync_binlog 设置为 N，对应的风险是：如果主机发生异常重启，会丢失最近 N 个事务的 binlog 日志。

# RedoLog

>在 MySQL 中，当使用 InnoDB 作为存储引擎时，重做日志（redo log）是一种非常重要的日志。redo log 主要用于确保事务的持久性（Durability），并在数据库崩溃时提供故障恢复能力。

1. InnoDB 引擎特有,记录的是“在某个数据页上做了什么修改”

2. redo log 是循环写的，空间固定会用完

3. Innodb 引擎使用 WAL（Write-Ahead Logging）技术来提升效率，技术的关键是先写日志再写磁盘。当有一条记录需要更新的时候，InnoDB 引擎就会先把记录写到 redo log，并更新内存，这个时候更新就算完成了。同时，InnoDB 引擎会在适当的时候，将这个操作记录更新到磁盘里面。

4. crash-safe：有了 redo log，InnoDB 就可以保证即使数据库发生异常重启，之前提交的记录都不会丢失，这个能力称为 crash-safe。

5. 检查点（Checkpoints）：为了避免在数据库崩溃时回放过多的 redo log 数据，InnoDB 使用检查点（checkpoints）机制。检查点是 InnoDB 数据文件和 redo log 文件之间的同步点。InnoDB 会定期将 redo log 中的数据刷新到数据文件中，并将检查点信息更新到磁盘。在数据库崩溃时，只需要回放检查点之后的 redo log 数据即可。检查点有助于减少故障恢复时间。

6. 为什么有 binlog 还要 redo log：binlog 不具备奔溃恢复的能力，这是因为 binlog 没有 check point，无法确定从哪里开始重放日志。

>redo log 是固定大小的，比如可以配置为一组 4 个文件，每个文件的大小是 1GB，那么日志总共就可以记录 4GB 的操作。从头开始写，写到末尾就又回到开头循环写，如下图所示。

## 状态

1. 存在 redo log buffer 中，物理上是在 MySQL 进程内存中，就是图中的红色部分；

2. 写到磁盘 (write)，但是没有持久化（fsync)，物理上是在文件系统的 page cache 里面，也就是图中的黄色部分；

3. 持久化到磁盘，对应的是 hard disk，也就是图中的绿色部分

> InnoDB 提供了 innodb_flush_log_at_trx_commit 参数用于控制 redo log 的写入策略

1. 设置为 0 的时候，表示每次事务提交时都只是把 redo log 留在 redo log buffer 中

2. 设置为 1 的时候，表示每次事务提交时都将 redo log 直接持久化到磁盘

3. 设置为 2 的时候，表示每次事务提交时都只是把 redo log 写到 page cache

>InnoDB 有一个后台线程，每隔 1 秒，就会把 redo log buffer 中的日志，调用 write 写到文件系统的 page cache，然后调用 fsync 持久化到磁盘。

## 两段提交与崩溃恢复

>在 MySQL 中，当使用 InnoDB 存储引擎并启用二进制日志（binlog）时，会使用两段提交协议来确保数据的一致性。在这种情况下，两段提交协议涉及到重做日志（redo log）和二进制日志（binlog）的同步。

两段提交协议在 MySQL InnoDB 存储引擎中的运作如下：

1. **第一阶段（准备阶段）**：当一个事务执行完成并准备提交时，InnoDB 首先将事务的所有修改记录到 redo log 中，并将 redo log 设置为 PREPARED 状态。此时，事务的所有更改已经记录在 redo log 中，但尚未写入 binlog。
2. **第二阶段（提交阶段）**：在 redo log 成功刷新到磁盘并设置为 PREPARED 状态后，MySQL 将事务的所有更改记录到 binlog。一旦 binlog 写入成功，事务就可以正式提交。此时，InnoDB 会将 redo log 设置为 COMMITTED 状态，并将事务的更改应用到数据文件。

通过两段提交协议，MySQL 可以确保在事务提交过程中，redo log 和 binlog 的同步。在数据库崩溃时，可以通过回放 redo log 和 binlog 来恢复数据的一致性。如果在第一阶段失败，事务会被回滚；如果在第二阶段失败，事务已经被记录在 redo log 中，因此可以在数据库恢复后继续完成提交。

### 过程

**重做日志恢复（Redo Log Recovery）**：当数据库启动时，InnoDB 存储引擎会先进行 redo log 的恢复。这一步骤会检查 redo log 中的所有记录，找到最近的检查点（Checkpoint），从检查点开始，将所有 PREPARED 或 COMMITTED 状态的事务重做（即应用到数据文件）。这样可以确保已提交事务的更改被正确应用到数据文件。在这个过程中，XID 用于确定事务的状态，以便决定是否需要重做。

- 如果碰到既有 prepare、又有 commit 的 redo log，就直接提交；
- 如果碰到只有 parepare、而没有 commit 的 redo log，就拿着 XID 去 binlog 找对应的事务。

**悔做日志恢复（Undo Log Recovery）**：在 redo log 恢复完成后，InnoDB 会进行悔做日志（undo log）恢复。悔做日志用于存储事务的回滚信息。悔做日志恢复的目的是回滚那些在数据库崩溃时尚未提交的事务。InnoDB 会根据悔做日志中的 XID 回滚未提交的事务，以确保数据的一致性。

**二进制日志恢复（Binlog Recovery）**：在 redo log 和 undo log 恢复完成后，如果启用了二进制日志（binlog），则会进行 binlog 恢复。在这个阶段，MySQL 会从最后一个完整的 binlog 备份开始，逐条执行 binlog 中的事件，将所有已提交事务的更改应用到数据文件。这样可以确保那些在 redo log 恢复阶段尚未被应用的事务得到正确的恢复。在这个过程中，XID 用于标识事务的开始和结束，以确保事务的原子性。

# RelayLog

>Relay log（中继日志）是 MySQL 主从复制过程中的一个关键组件。在主从复制配置中，主服务器（Master）会将其二进制日志（Binary Log，简称 binlog）中的更改发送给从服务器（Slave）。从服务器接收到这些更改后，会将它们写入到本地的 Relay log。然后，从服务器会根据 Relay log 中的记录应用这些更改到自己的数据文件。这样，从服务器的数据可以保持与主服务器的数据同步。

Relay log 的作用主要有以下几点：

1. **缓冲区**：Relay log 在从服务器上充当一个缓冲区，存储从主服务器接收到的 binlog 事件。这样，在从服务器繁忙或者主服务器产生 binlog 事件的速度远大于从服务器应用 binlog 事件的速度（例如，在高写入负载的情况下）时，从服务器仍然可以保持与主服务器的数据同步。
2. **容错**：Relay log 可以在从服务器宕机或连接中断情况下提供容错能力。当从服务器恢复正常工作或重新连接到主服务器时，从服务器可以从 Relay log 的末尾处继续应用未处理的 binlog 事件，从而确保数据的一致性。
3. **多从服务器支持**：主服务器可以将 binlog 事件发送给多个从服务器。这些从服务器可以独立地根据自己的 Relay log 应用这些事件，从而实现多个从服务器与主服务器数据的同步。

Relay log 的工作流程如下：

1. 从服务器上运行的 IO 线程负责连接到主服务器，并从主服务器请求 binlog 事件。
2. IO 线程将接收到的 binlog 事件写入从服务器上的 Relay log。
3. 从服务器上运行的 SQL 线程负责读取 Relay log 中的 binlog 事件，并根据这些事件应用更改到从服务器的数据文件。
4. 当从服务器成功应用了 Relay log 中的所有事件，从服务器会删除已处理的 Relay log 文件，以节省磁盘空间。

