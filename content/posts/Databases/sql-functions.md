---
title: "sql常用函数"
date: 2025-04-27T14:11:37+08:00
author: ["loveyu"]
draft: false
categories: 
- databases
tags: 
- mysql
- pgsql
---

# SQL常用函数大全

SQL提供了丰富的函数集合，可以用于数据处理、分析和转换。本指南整理了常见的SQL函数类别及其用法示例。

## 目录
- [1. 窗口函数](#1-窗口函数)
- [2. 聚合函数](#2-聚合函数)
- [3. 字符串函数](#3-字符串函数)
- [4. 日期和时间函数](#4-日期和时间函数)
- [5. 数值函数](#5-数值函数)
- [6. 条件函数](#6-条件函数)
- [7. 类型转换函数](#7-类型转换函数)
- [8. JSON函数](#8-json函数)
- [9. 全文搜索函数](#9-全文搜索函数)
- [10. 加密和安全函数](#10-加密和安全函数)
- [11. 系统和元数据函数](#11-系统和元数据函数)
- [12. 常见复合SQL示例](#12-常见复合sql示例)

## 1. 窗口函数

窗口函数允许对结果集中的行进行计算，同时保留每行的独立性。

### 基本语法

```sql
<窗口函数> OVER ([PARTITION BY <分区列>] [ORDER BY <排序列>] [<窗口子句>])
```

### 排名函数

```sql
-- ROW_NUMBER(): 为每行分配唯一序号
SELECT name, score, ROW_NUMBER() OVER (ORDER BY score DESC) AS row_num
FROM students;

-- RANK(): 相同值获得相同排名，跳过下一个排名
SELECT name, score, RANK() OVER (ORDER BY score DESC) AS rank
FROM students;

-- DENSE_RANK(): 相同值获得相同排名，不跳过排名
SELECT name, score, DENSE_RANK() OVER (ORDER BY score DESC) AS dense_rank
FROM students;

-- NTILE(): 将分区数据分成指定数量的组
SELECT name, score, NTILE(4) OVER (ORDER BY score DESC) AS quartile
FROM students;
```

### 聚合窗口函数

```sql
SELECT 
    department,
    name,
    salary,
    AVG(salary) OVER (PARTITION BY department) AS dept_avg,
    SUM(salary) OVER (PARTITION BY department) AS dept_sum,
    COUNT(*) OVER (PARTITION BY department) AS dept_count
FROM employees;
```

### 偏移函数

```sql
-- LAG(): 获取前面行的值
SELECT date, sales, LAG(sales) OVER (ORDER BY date) AS prev_day_sales
FROM daily_sales;

-- LEAD(): 获取后面行的值
SELECT date, sales, LEAD(sales) OVER (ORDER BY date) AS next_day_sales
FROM daily_sales;
```

### 分析函数

```sql
-- FIRST_VALUE(): 窗口中第一个值
SELECT date, product, sales,
       FIRST_VALUE(sales) OVER (PARTITION BY product ORDER BY date) AS first_sale
FROM product_sales;

-- LAST_VALUE(): 窗口中最后一个值
SELECT date, product, sales,
       LAST_VALUE(sales) OVER (
           PARTITION BY product 
           ORDER BY date 
           ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
       ) AS last_sale
FROM product_sales;
```

### 窗口子句

```sql
-- 计算3天移动平均值
SELECT date, sales,
       AVG(sales) OVER (
           ORDER BY date 
           ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING
       ) AS moving_avg
FROM daily_sales;
```

## 2. 聚合函数

聚合函数对一组值执行计算并返回单个值。

```sql
-- SUM(): 计算总和
SELECT SUM(salary) FROM employees;

-- AVG(): 计算平均值
SELECT AVG(price) FROM products;

-- MAX(): 计算最大值
SELECT MAX(score) FROM exams;

-- MIN(): 计算最小值
SELECT MIN(date_registered) FROM users;

-- COUNT(): 计数
SELECT COUNT(*) FROM orders;
SELECT COUNT(DISTINCT customer_id) FROM orders;

-- GROUP_CONCAT(): 将组值连接为字符串(MySQL)
SELECT department, GROUP_CONCAT(name) FROM employees GROUP BY department;

-- STRING_AGG(): 将组值连接为字符串(PostgreSQL/SQL Server)
SELECT department, STRING_AGG(name, ', ') FROM employees GROUP BY department;
```

## 3. 字符串函数

处理和操作文本数据的函数。

```sql
-- CONCAT(): 连接字符串
SELECT CONCAT(first_name, ' ', last_name) AS full_name FROM users;

-- UPPER()/LOWER(): 转换大小写
SELECT UPPER(name) FROM products;
SELECT LOWER(email) FROM customers;

-- SUBSTRING(): 提取子字符串
SELECT SUBSTRING(description, 1, 100) FROM articles;

-- LEFT()/RIGHT(): 从左/右提取字符
SELECT LEFT(phone, 3) AS area_code FROM contacts;
SELECT RIGHT(reference, 6) AS ref_code FROM orders;

-- TRIM()/LTRIM()/RTRIM(): 去除空格
SELECT TRIM(name) FROM categories;
SELECT LTRIM(code) FROM products; -- 去除左侧空格
SELECT RTRIM(notes) FROM comments; -- 去除右侧空格

-- REPLACE(): 替换字符
SELECT REPLACE(phone, '-', '') FROM contacts;

-- LENGTH()/CHAR_LENGTH(): 字符串长度
SELECT LENGTH(password) FROM users;
SELECT CHAR_LENGTH(description) FROM products;

-- POSITION()/INSTR(): 查找子字符串位置
SELECT POSITION('SQL' IN course_name) FROM courses;
SELECT INSTR(email, '@') FROM users;

-- REGEXP/RLIKE: 正则表达式匹配
SELECT * FROM emails WHERE email REGEXP '^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$';
```

## 4. 日期和时间函数

处理日期和时间值的函数。

```sql
-- 获取当前日期/时间
SELECT CURRENT_DATE();
SELECT CURRENT_TIMESTAMP();
SELECT NOW();

-- 提取日期部分
SELECT YEAR(order_date) FROM orders;
SELECT MONTH(birth_date) FROM employees;
SELECT DAY(created_at) FROM logs;
SELECT HOUR(login_time) FROM user_sessions;
SELECT MINUTE(start_time) FROM meetings;
SELECT SECOND(timestamp) FROM events;
SELECT DAYNAME(event_date) FROM calendar; -- 星期几名称
SELECT MONTHNAME(birth_date) FROM employees; -- 月份名称
SELECT DAYOFWEEK(date) FROM dates; -- 一周的第几天(1=星期日到7=星期六)
SELECT DAYOFYEAR(date) FROM events; -- 一年的第几天(1-366)
SELECT WEEKOFYEAR(date) FROM events; -- 一年的第几周

-- 日期计算
SELECT DATE_ADD(start_date, INTERVAL 30 DAY) FROM subscriptions;
SELECT DATE_SUB(end_date, INTERVAL 7 DAY) FROM projects;
SELECT DATEDIFF(end_date, start_date) AS duration FROM projects;
SELECT TIMESTAMPDIFF(HOUR, start_time, end_time) AS hours FROM sessions;

-- 日期格式化
SELECT DATE_FORMAT(order_date, '%Y-%m-%d') FROM orders;
SELECT DATE_FORMAT(timestamp, '%H:%i:%s') FROM logs;
SELECT DATE_FORMAT(birth_date, '%d %M %Y') FROM employees;

-- 日期/时间部分提取
SELECT EXTRACT(YEAR FROM date) FROM events;
SELECT EXTRACT(MONTH FROM date) FROM events;
SELECT EXTRACT(DAY FROM date) FROM events;
SELECT EXTRACT(HOUR FROM timestamp) FROM logs;

-- 日期/时间构造
SELECT MAKEDATE(2023, 45) AS day_45_of_2023;
SELECT MAKETIME(14, 30, 0) AS time_2_30_pm;
```

## 5. 数值函数

处理数字的函数。

```sql
-- 四舍五入
SELECT ROUND(price, 2) FROM products;

-- 向上/向下取整
SELECT CEILING(rating) FROM reviews;
SELECT FLOOR(amount) FROM transactions;

-- 截断小数位
SELECT TRUNCATE(value, 2) FROM measurements;

-- 绝对值
SELECT ABS(temperature_change) FROM climate_data;

-- 取余
SELECT MOD(employee_id, 10) FROM employees;

-- 随机数
SELECT RAND() * 100;

-- 数学函数
SELECT SQRT(area) FROM squares;
SELECT POWER(base, exponent) FROM calculations;
SELECT LOG(value) FROM data_points; -- 自然对数
SELECT LOG10(value) FROM data_points; -- 以10为底对数
SELECT EXP(value) FROM exponential_data; -- e的幂
SELECT PI(); -- π值
SELECT SIN(angle), COS(angle), TAN(angle) FROM triangles; -- 三角函数
```

## 6. 条件函数

基于条件返回值的函数。

```sql
-- IF()条件表达式(MySQL)
SELECT product_name, IF(stock > 0, '有库存', '无库存') AS stock_status
FROM products;

-- CASE条件表达式
SELECT 
    order_id,
    CASE 
        WHEN amount < 100 THEN '小额订单'
        WHEN amount BETWEEN 100 AND 1000 THEN '中额订单'
        ELSE '大额订单'
    END AS order_size
FROM orders;

-- COALESCE(): 返回第一个非NULL值
SELECT COALESCE(nickname, username, email, '匿名用户') AS display_name 
FROM users;

-- NULLIF(): 如果两个表达式相等返回NULL，否则返回第一个表达式
SELECT NULLIF(divider, 0) FROM calculations; -- 避免除以零错误

-- IFNULL()/NVL(): 替换NULL值
SELECT IFNULL(phone, '未提供') FROM contacts; -- MySQL
SELECT NVL(phone, '未提供') FROM contacts; -- Oracle

-- IIF(): 条件表达式(SQL Server)
SELECT IIF(price > 100, '高价', '低价') FROM products;
```

## 7. 类型转换函数

在不同数据类型之间转换的函数。

```sql
-- CAST(): 显式类型转换
SELECT CAST(price_string AS DECIMAL(10,2)) FROM import_data;
SELECT CAST(employee_id AS CHAR) FROM employees;
SELECT CAST('2023-01-01' AS DATE);

-- CONVERT(): 类型转换(SQL Server/MySQL语法)
SELECT CONVERT(VARCHAR, order_date, 120) FROM orders; -- SQL Server
SELECT CONVERT(order_date, CHAR) FROM orders; -- MySQL

-- 隐式转换
SELECT '价格: ' || price FROM products; -- PostgreSQL字符串连接会自动转换
SELECT CONCAT('价格: ', price) FROM products; -- MySQL

-- 特定类型转换
SELECT TO_DATE('2023-01-01', 'YYYY-MM-DD'); -- Oracle/PostgreSQL
SELECT STR_TO_DATE('01/01/2023', '%d/%m/%Y'); -- MySQL
```

## 8. JSON函数

处理JSON数据类型的函数(现代数据库)。

```sql
-- JSON_EXTRACT(): 提取JSON值(MySQL)
SELECT JSON_EXTRACT(data, '$.name') FROM documents;

-- ->和->>操作符(MySQL/PostgreSQL)
SELECT data->'$.name' FROM documents; -- 返回JSON值
SELECT data->>'$.name' FROM documents; -- 返回文本值

-- JSON_SET(): 设置JSON值(MySQL)
SELECT JSON_SET(preferences, '$.theme', 'dark') FROM user_settings;

-- JSON_REMOVE(): 移除JSON键(MySQL)
SELECT JSON_REMOVE(metadata, '$.temporary') FROM files;

-- JSON_CONTAINS(): 检查JSON是否包含值(MySQL)
SELECT * FROM products WHERE JSON_CONTAINS(attributes, '"red"', '$.colors');

-- JSON_ARRAY(): 创建JSON数组(MySQL)
SELECT JSON_ARRAY('apple', 'banana', 'orange');

-- JSON_OBJECT(): 创建JSON对象(MySQL)
SELECT JSON_OBJECT('name', username, 'age', age) FROM users;

-- PostgreSQL的JSON函数
SELECT jsonb_pretty(data) FROM documents; -- 格式化JSON
SELECT jsonb_set(data, '{theme}', '"dark"') FROM user_settings; -- 设置值
SELECT jsonb_array_elements(data->'tags') FROM documents; -- 展开JSON数组
```

## 9. 全文搜索函数

提供高级文本搜索功能的函数。

```sql
-- MySQL全文搜索
CREATE FULLTEXT INDEX idx_article ON articles(title, content);
SELECT * FROM articles 
WHERE MATCH(title, content) AGAINST('关键词' IN NATURAL LANGUAGE MODE);
SELECT * FROM articles 
WHERE MATCH(title, content) AGAINST('+必须 -排除' IN BOOLEAN MODE);

-- PostgreSQL全文搜索
SELECT * FROM documents 
WHERE to_tsvector('chinese', content) @@ to_tsquery('chinese', '关键词');
SELECT ts_rank(to_tsvector('english', content), to_tsquery('english', 'search')) AS rank
FROM documents
ORDER BY rank DESC;

-- SQL Server全文搜索
SELECT * FROM articles 
WHERE CONTAINS(content, '关键词');
SELECT * FROM articles 
WHERE FREETEXT(content, '相关搜索短语');
```

## 10. 加密和安全函数

处理密码和敏感数据的函数。

```sql
-- MD5哈希(不推荐用于密码)
SELECT MD5(password) FROM legacy_users;

-- SHA哈希系列
SELECT SHA1(data) FROM files;
SELECT SHA2(password, 256) FROM users;

-- 生成UUID/GUID
SELECT UUID() AS unique_id; -- MySQL
SELECT NEWID() AS unique_id; -- SQL Server

-- 加密函数
SELECT AES_ENCRYPT(credit_card, 'secret_key') FROM payments; -- MySQL
SELECT ENCRYPT(credit_card, 'salt') FROM payments; -- PostgreSQL

-- 解密函数
SELECT AES_DECRYPT(encrypted_data, 'secret_key') FROM secure_data; -- MySQL
```

## 11. 系统和元数据函数

提供关于数据库和当前会话的信息的函数。

```sql
-- 用户信息
SELECT CURRENT_USER();
SELECT USER();
SELECT SESSION_USER();

-- 数据库信息
SELECT VERSION();
SELECT DATABASE();
SELECT SCHEMA();

-- 连接信息
SELECT CONNECTION_ID();

-- 字符集信息
SELECT CHARSET('文本');
SELECT COLLATION('文本');
```

## 12. 常见复合SQL示例

结合多种函数的实际应用示例。

### 销售报表生成

```sql
SELECT 
    YEAR(order_date) AS year,
    MONTH(order_date) AS month,
    MONTHNAME(order_date) AS month_name,
    COUNT(*) AS total_orders,
    SUM(amount) AS total_sales,
    ROUND(AVG(amount), 2) AS avg_order_value,
    MAX(amount) AS largest_order,
    MIN(amount) AS smallest_order,
    COUNT(DISTINCT customer_id) AS unique_customers,
    ROUND(SUM(amount) / COUNT(DISTINCT customer_id), 2) AS avg_customer_spend
FROM orders
WHERE order_date BETWEEN DATE_SUB(CURRENT_DATE(), INTERVAL 1 YEAR) AND CURRENT_DATE()
GROUP BY YEAR(order_date), MONTH(order_date), MONTHNAME(order_date)
ORDER BY year DESC, month DESC;
```

### 用户活跃度分析

```sql
SELECT 
    user_id,
    COUNT(*) AS login_count,
    MIN(login_time) AS first_login,
    MAX(login_time) AS last_login,
    DATEDIFF(MAX(login_time), MIN(login_time)) AS days_active,
    ROUND(COUNT(*) / NULLIF(DATEDIFF(MAX(login_time), MIN(login_time)), 0), 2) AS logins_per_day,
    CASE 
        WHEN DATEDIFF(CURRENT_DATE(), MAX(login_time)) <= 7 THEN '活跃用户'
        WHEN DATEDIFF(CURRENT_DATE(), MAX(login_time)) <= 30 THEN '一般用户'
        WHEN DATEDIFF(CURRENT_DATE(), MAX(login_time)) <= 90 THEN '非活跃用户'
        ELSE '休眠用户' 
    END AS user_status
FROM user_logins
GROUP BY user_id
HAVING COUNT(*) > 1
ORDER BY logins_per_day DESC;
```

### 动态透视表

```sql
SELECT 
    DATE_FORMAT(created_at, '%Y-%m') AS month,
    SUM(CASE WHEN status = 'completed' THEN 1 ELSE 0 END) AS completed_orders,
    SUM(CASE WHEN status = 'cancelled' THEN 1 ELSE 0 END) AS cancelled_orders,
    SUM(CASE WHEN status = 'pending' THEN 1 ELSE 0 END) AS pending_orders,
    COUNT(*) AS total_orders,
    ROUND(SUM(CASE WHEN status = 'completed' THEN 1 ELSE 0 END) / COUNT(*) * 100, 2) AS completion_rate,
    ROUND(SUM(CASE WHEN status = 'cancelled' THEN 1 ELSE 0 END) / COUNT(*) * 100, 2) AS cancellation_rate
FROM orders
GROUP BY DATE_FORMAT(created_at, '%Y-%m')
ORDER BY month;
```

### 获取每个分类的最新产品

```sql
WITH latest_products AS (
    SELECT 
        category_id,
        product_id,
        product_name,
        created_at,
        ROW_NUMBER() OVER (PARTITION BY category_id ORDER BY created_at DESC) AS rn
    FROM products
    WHERE status = 'active'
)
SELECT 
    c.category_name,
    p.product_id,
    p.product_name,
    p.created_at
FROM latest_products p
JOIN categories c ON p.category_id = c.category_id
WHERE p.rn = 1;
```

### 数据清洗和标准化

```sql
SELECT 
    id,
    TRIM(UPPER(SUBSTRING(product_code, 1, 3))) AS category_code,
    CASE 
        WHEN price_str REGEXP '^[0-9]+(\.[0-9]+)?$' THEN 
            CAST(price_str AS DECIMAL(10,2))
        ELSE NULL
    END AS clean_price,
    COALESCE(NULLIF(TRIM(description), ''), 'No description available') AS display_description,
    CASE 
        WHEN stock < 0 THEN 0
        ELSE stock
    END AS corrected_stock,
    CASE 
        WHEN stock < 10 THEN 'Low stock'
        WHEN stock < 50 THEN 'Medium stock'
        ELSE 'High stock'
    END AS stock_level,
    DATE_FORMAT(STR_TO_DATE(date_str, '%m/%d/%Y'), '%Y-%m-%d') AS standardized_date
FROM raw_product_data;
```
