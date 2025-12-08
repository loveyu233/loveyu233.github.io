---
date: 2025-12-08T11:44:11+08:00
title: 高级sql用法
author: ["loveyu"]
draft: false
categories: 
- go
tags: 
- go
- gorm
---


# 指定字段不能为空字符串

```sql
ALTER TABLE student
    ADD CONSTRAINT chk_col_not_blank CHECK (CHAR_LENGTH(TRIM(guid)) > 0);
    
--- 删除
ALTER TABLE student DROP CHECK chk_col_not_blank;
```



# 设置字段的默认值为一段sql

```sql
ALTER TABLE xtj_stars.leave
    ADD COLUMN start_slot INT UNSIGNED GENERATED ALWAYS AS (
        TO_DAYS(start_date) * 4 +
        CASE
            WHEN start_period = '早上' THEN 1
            WHEN start_period = '上午' THEN 2
            WHEN start_period = '下午' THEN 3
            WHEN start_period = '晚上' THEN 4
            END
        ) STORED,
    ADD COLUMN end_slot   INT UNSIGNED GENERATED ALWAYS AS (
        TO_DAYS(end_date) * 4 +
        CASE
            WHEN end_period = '早上' THEN 1
            WHEN end_period = '上午' THEN 2
            WHEN end_period = '下午' THEN 3
            WHEN end_period = '晚上' THEN 4
            END
        ) STORED;
```



# 递归查询所有的子节点

>从 table_name 表中以 id = 85 的记录为起点，向下递归查找其所有未被软删除（deleted_at IS NULL）的子节点（子、孙、曾孙……）

```sql
WITH RECURSIVE descendants AS (SELECT id, parent_id
                               FROM table_name
                               WHERE id = 85
                                 AND deleted_at IS NULL
                               UNION ALL
                               SELECT sc.id, sc.parent_id
                               FROM table_name sc
                                        JOIN descendants d ON sc.parent_id = d.id
                               WHERE sc.deleted_at IS NULL)
SELECT id
FROM descendants;
 
-- 查找多个同时返回root节点
WITH RECURSIVE descendants (id, parent_id, root_id) AS (SELECT id, parent_id, id AS root_id
                                                        FROM table_name
                                                        WHERE id IN (1, 2, 3)
                                                          AND deleted_at IS NULL
                                                        UNION ALL
                                                        SELECT sc.id, sc.parent_id, d.root_id
                                                        FROM table_name sc
                                                                 JOIN descendants d ON sc.parent_id = d.id
                                                        WHERE sc.deleted_at IS NULL)
SELECT id, root_id
FROM descendants
ORDER BY root_id, id;
```



# 递归查询所有的父节点

>沿着 parent_id 向“上”查找给定节点集合（roots，id IN (?)）的所有上级（父、祖父、曾祖父……）

```sql
WITH RECURSIVE
    roots AS (SELECT id
              FROM table_name
              WHERE id IN (?)
                AND deleted_at IS NULL),
    ancestors AS (SELECT id, parent_id
                  FROM table_name
                  WHERE id IN (SELECT id FROM roots)
                    AND deleted_at IS NULL
                  UNION ALL
                  SELECT sc.id, sc.parent_id
                  FROM table_name sc
                           JOIN ancestors a ON sc.id = a.parent_id
                  WHERE sc.deleted_at IS NULL)
SELECT DISTINCT id
FROM ancestors;
```

