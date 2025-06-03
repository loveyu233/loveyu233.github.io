---
title: "查询mysql连接信息"
date: 2025-06-03T15:52:30+08:00
author: ["loveyu"]
draft: false
categories: 
- databases
tags: 
- mysql
---

# MySQL 连接信息查看完整指南

## 概述

在MySQL数据库管理中，查看当前连接信息是监控数据库性能、排查问题和管理连接的重要手段。本文档详细介绍了各种查看连接信息的SQL命令及其返回结果的含义。

## 常用查看连接信息的SQL命令

### 1. 基础连接查看命令

#### 查看所有连接
```sql
SHOW PROCESSLIST;
```
显示当前所有连接的基本信息，SQL语句会被截断到前100个字符。

#### 查看完整连接信息
```sql  
SHOW FULL PROCESSLIST;
```
与上面命令功能相同，但显示完整的SQL语句，不会被截断。

### 2. 通过系统表查询

#### 基础查询
```sql
SELECT * FROM INFORMATION_SCHEMA.PROCESSLIST;
```
通过系统表获取详细的连接信息，支持WHERE条件过滤。

#### 查看当前用户连接
```sql
SELECT * FROM INFORMATION_SCHEMA.PROCESSLIST WHERE USER = USER();
```

#### 查看特定数据库连接
```sql
SELECT * FROM INFORMATION_SCHEMA.PROCESSLIST WHERE DB = '数据库名';
```

### 3. 连接统计信息

```sql
-- 当前连接数
SHOW STATUS LIKE 'Threads_connected';

-- 当前活跃连接数
SHOW STATUS LIKE 'Threads_running';

-- 历史最大连接数
SHOW STATUS LIKE 'Max_used_connections';
```

## 返回字段详细说明

### PROCESSLIST 核心字段

| 字段名 | 数据类型 | 说明 | 示例值 |
|--------|----------|------|--------|
| **Id** | 整数 | 连接的唯一标识符（线程ID），可用于KILL命令 | 1, 2, 3... |
| **User** | 字符串 | 发起连接的MySQL用户名 | root, app_user |
| **Host** | 字符串 | 客户端主机名或IP地址及端口号 | localhost:3306, 192.168.1.100:52341 |
| **db** | 字符串 | 当前连接使用的数据库名，NULL表示未选择数据库 | my_database, NULL |
| **Command** | 字符串 | 当前连接正在执行的命令类型 | Query, Sleep, Connect |
| **Time** | 整数 | 当前状态持续的时间（秒） | 0, 30, 120 |
| **State** | 字符串 | 连接的详细状态信息 | Sending data, NULL |
| **Info** | 文本 | 正在执行的SQL语句，NULL表示无查询 | SELECT * FROM users |

### Command 字段常见值

| 值 | 含义 | 说明 |
|----|------|------|
| **Query** | 正在执行查询 | 连接正在处理SQL语句 |
| **Sleep** | 连接空闲 | 等待客户端发送新命令 |
| **Connect** | 正在连接 | 客户端正在建立连接 |
| **Quit** | 正在断开连接 | 连接正在关闭 |
| **Init DB** | 切换数据库 | 正在执行USE database命令 |
| **Field List** | 获取字段列表 | 获取表结构信息 |
| **Statistics** | 获取统计信息 | 收集或计算统计数据 |

### State 字段常见值

| 状态 | 含义 | 解释 |
|------|------|------|
| **NULL** 或 空 | 空闲状态 | 连接未执行任何操作 |
| **Sending data** | 发送数据 | 正在向客户端发送查询结果 |
| **Waiting for table level lock** | 等待表级锁 | 查询被表锁阻塞 |
| **Copying to tmp table** | 复制到临时表 | 创建临时表处理复杂查询 |
| **Sorting result** | 排序结果 | 正在对查询结果排序 |
| **Opening tables** | 打开表 | 正在打开和初始化表 |
| **Locked** | 被锁定 | 连接被某种锁机制阻塞 |
| **Statistics** | 计算统计信息 | 优化器正在收集表统计信息 |

### INFORMATION_SCHEMA.PROCESSLIST 额外字段

| 字段名 | 说明 |
|--------|------|
| **TIME_MS** | 状态持续时间的毫秒精度版本 |
| **STAGE** | 查询执行的当前阶段信息 |
| **MAX_STAGE** | 查询执行的最大阶段数 |
| **PROGRESS** | 当前阶段的完成进度（百分比）|

## 连接状态统计说明

### 重要统计指标

| 统计项 | 命令 | 含义 | 用途 |
|--------|------|------|------|
| **当前连接数** | `SHOW STATUS LIKE 'Threads_connected'` | 当前已连接的线程总数 | 监控当前连接负载 |
| **活跃连接数** | `SHOW STATUS LIKE 'Threads_running'` | 当前正在运行的线程数（非Sleep状态） | 监控数据库活跃度 |
| **历史最大连接数** | `SHOW STATUS LIKE 'Max_used_connections'` | 自MySQL启动以来的最大同时连接数 | 了解连接峰值，规划容量 |

## 实际应用示例

### 性能监控查询

```sql
-- 查找长时间运行的查询（超过60秒）
SELECT Id, User, Host, DB, Time, State, Info 
FROM INFORMATION_SCHEMA.PROCESSLIST 
WHERE COMMAND = 'Query' AND TIME > 60
ORDER BY TIME DESC;

-- 查找占用CPU的活跃连接
SELECT Id, User, Host, DB, Time, State, LEFT(Info, 50) AS Query_Preview
FROM INFORMATION_SCHEMA.PROCESSLIST 
WHERE COMMAND != 'Sleep'
ORDER BY TIME DESC;

-- 统计各用户的连接数
SELECT User, COUNT(*) AS Connection_Count
FROM INFORMATION_SCHEMA.PROCESSLIST 
GROUP BY User
ORDER BY Connection_Count DESC;
```

### 问题排查查询

```sql
-- 查找被锁定的连接
SELECT * FROM INFORMATION_SCHEMA.PROCESSLIST 
WHERE State LIKE '%lock%' OR State LIKE '%Lock%';

-- 查找特定数据库的所有连接
SELECT Id, User, Host, Command, Time, State 
FROM INFORMATION_SCHEMA.PROCESSLIST 
WHERE DB = 'your_database_name';

-- 查找来自特定IP的连接
SELECT * FROM INFORMATION_SCHEMA.PROCESSLIST 
WHERE Host LIKE '192.168.1.%';
```

### 连接管理操作

```sql
-- 终止特定连接（谨慎使用）
KILL 123;  -- 123是连接ID

-- 终止特定用户的所有连接（批量操作，极其谨慎）
SELECT CONCAT('KILL ', Id, ';') AS kill_command
FROM INFORMATION_SCHEMA.PROCESSLIST 
WHERE User = 'problematic_user' AND Command = 'Sleep';
```

## 最佳实践建议

### 1. 定期监控
- 建议定期检查长时间运行的查询
- 监控连接数是否接近最大连接限制
- 观察Sleep状态连接的累积情况

### 2. 性能优化
- Time值较大的Query状态连接需要重点关注
- 频繁出现的等锁状态需要优化查询或索引
- 合理设置连接超时参数

### 3. 安全考虑
- 定期检查异常的连接来源
- 监控特权用户的连接活动
- 及时清理异常或可疑连接

### 4. 故障排除
- 结合慢查询日志分析长时间运行的查询
- 使用EXPLAIN分析有问题的SQL语句
- 监控锁等待和死锁情况

## 注意事项

1. **权限要求**：查看连接信息需要PROCESS权限
2. **性能影响**：频繁执行SHOW PROCESSLIST对性能影响很小
3. **KILL命令**：使用KILL命令终止连接时要格外谨慎
4. **连接限制**：注意max_connections参数设置，避免连接数耗尽
5. **监控频率**：建议结合监控工具定期自动检查连接状态

通过掌握这些连接信息查看和分析技能，可以更好地管理MySQL数据库，及时发现和解决性能问题。