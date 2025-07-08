---
title: "pg常用操作"
date: 2024-11-21T11:23:35+08:00
author: ["loveyu"]
draft: false
categories: 
- databases
tags: 
- go
- pgsql
- docker
---

# 删除pgsql的缓存计划

>```sql
>discard all;
>```
>
>

# text[]模糊查询

>```sql 
>select * from aaa where exists(select 1 from unnest(alias) as elem where elem like %a%)
>```
>
>

# 最近时间

>```sql
>select max(time) from abc;
>```
>
>

# 不为空

>pgsql中`<> 表示 != 不等于`
>
>```sql
>-- 这样子是不对的, 不能把select的字段用到where上, 只能是from中的表字段或者显示声明的字段才可以在where中使用
>select 
>	distinct unnest(product_type) as type 
>from 
>	products 
>where 
>	type IS NOT NULL AND type <> '';
>
>-- 正确的用法
>SELECT 
>	DISTINCT pt 
>FROM 
>	( SELECT unnest(product_type) AS pt FROM "products" ) AS subquery
>WHERE 
>	pt IS NOT NULL AND pt <> '';
>```
>
>

# 计算比例

>语法: ` sum() over()` 必须带over()
>
>```sql
>SELECT
>    trade,
>    trade_count,
>    trade_count::float / SUM(trade_count) OVER () AS weight_ratio
>FROM (
>    SELECT
>        unnest(industry_chain_position) AS trade,
>        COUNT(id) AS trade_count
>    FROM
>        products
>    WHERE
>        enterprise_id = 229501
>    GROUP BY
>        trade
>) AS subquery
>ORDER BY
>    trade_count DESC;
>```
>
>```sql
>WITH 
>    trade_counts AS (
>    SELECT
>        unnest(belong_trade) AS trade,
>        COUNT(code) AS product_total
>    FROM
>        products
>    WHERE
>        enterprise_id = 229501
>      AND
>        1 = ANY(industry_chain_position)
>    GROUP BY
>        trade
>),
>total_count AS (
>    SELECT
>        SUM(product_total) AS total_product_count
>    FROM
>        trade_counts
>)
>
>SELECT
>    trade,
>    product_total,
>    (product_total::float / total_product_count) AS proportion
>FROM
>    trade_counts,
>    total_count;
>```
>
>



# case

>语法:
>
>```sql
>CASE input_expression
>    WHEN when_expression_1 THEN result_1
>    WHEN when_expression_2 THEN result_2
>    ...
>    ELSE result_n
>END
>```
>
>

```sql
select
    transaction_time,
    (case when supplier_id = 228949 then customer_id when customer_id = 228949 then supplier_id end ) as enterprise_id,
    (case when supplier_id = 228949 then customer when customer_id = 228949 then supplier end ) as enterprise_name,
    (case when supplier_id = 228949 then '供货商' when customer_id = 228949 then '客户' end ) as category
from
    sales
where
    supplier_id = 228949 or customer_id = 228949
order by
    transaction_time desc limit 10;
```





# 分组排序

>- `PARTITION BY supplier_country`: 将结果集按照 `supplier_country` 划分为不同的组。
>- `ORDER BY weight DESC`: 在每个组内，按照 `weight` 的降序排列。
>- `ROW_NUMBER()`: 为每个组内的行分配一个唯一的行号，从1开始。这意味着每个国家的重量最大的产品会被标记为 `rn = 1`。

```sql
SELECT
    supplier_country as country,
    weight as value,
    product_name
FROM
    (
    SELECT supplier_country, weight, product_name,
           ROW_NUMBER() OVER (PARTITION BY supplier_country ORDER BY weight DESC ) AS rn
    FROM
        sales
    WHERE customer_id = 228949) AS ranked
WHERE rn = 1
limit 10;

```



# 数组jsonb搜索

>`jsonb_array_elements` 把数组转为 `element` 然后指定`key`模糊搜索`value` 

```sql
select id,customs_code from products where exists(select 1 from jsonb_array_elements(products.customs_code) as ele where ele ->> 'code' like '%xx%');
```



# Linux命令

```shell
# root身份保存文件
:w !sudo tee %

# 运行docker root权限
--privileged
```

