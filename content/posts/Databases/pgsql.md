---
title: "pgsql"
date: 2022-11-24T19:38:37+08:00
author: ["loveyu"]
draft: false
categories: 
- databases
tags: 
- pgsql
- databases
---

# 命令行

```sql
-- 登录
psql -U username -d dbname -h hostname

-- 查看用户权限
\du

-- 查看有哪些库
\l

-- 切换库
\c dbname

-- 查看当前库有哪些表
\dt
```



# 用户管理

>登录权限（LOGIN）：允许角色登录到数据库。
>超级用户权限（SUPERUSER）：赋予角色超级用户权限，使其可以执行任何操作，包括修改系统表。
>创建数据库权限（CREATEDB）：允许角色创建新的数据库。
>创建角色权限（CREATEROLE）：允许角色创建新的数据库角色。
>创建临时表权限（TEMPORARY）：允许角色创建临时表。
>连接限制（CONNECTION LIMIT）：限制角色同时连接到数据库的最大数量。
>继承权限（INHERIT）：允许角色继承其他角色的权限。
>授权权限（GRANT）：允许角色授权给其他角色权限。
>修改其他角色权限（ALTER ANY ROLE）：允许角色修改其他角色的属性。
>删除其他角色权限（DROP ANY ROLE）：允许角色删除其他角色。
>超时限制（PASSWORD EXPIRY）：设置角色密码的过期时间。
>加密方式（ENCRYPTED）：指定角色密码是否加密存储。

```sql
-- 创建用户
create role 用户名 with superuser login password '密码';

-- 删除用户
drop role 用户名;

-- 修改用户密码
alter role 用户名 with password '新密码';

REVOKE privilege_name ON object_name FROM username;
如：
# 撤销数据库权限
revoke all privileges on database mydata from jinjin;
# 撤销表权限
revoke all privileges on all tables in schema public from jinjin;
```

# 时区管理

```sql
-- 显示当前数据库使用的时区
show time zone ;

-- 查看当前时间
select now();

-- 查看可以设置的时区有哪些
select * from pg_timezone_names;

-- 设置当前时区为PRC
set timezone = 'PRC';

-- 设置全部用户的时区
alter role all set timezone = 'PRC';
```



# 序列

```sql
CREATE SEQUENCE my_sequence
    START 1  序列从 1 开始
    INCREMENT BY 1 每次递增 1
    MINVALUE 1  序列的最小值是 1
    MAXVALUE 10000 序列的最大值是 10000
    CYCLE; 当序列到达最大值后，循环回到最小值

-- 创建了一个名为 user_id_seq 的简单序列，未指定起始值或者增量等属性，默认情况下会从 1 开始，每次增加 1。
create sequence user_id_seq;

-- 将 user_id_seq 序列的所有权修改为 pgsql 用户。只有序列的所有者（或超级用户）可以修改序列的属性。
alter sequence user_id_seq owner to pgsql;

-- 指定了 user_id_seq 序列由 users.id 列拥有。这通常用于确保序列与特定表的列相关联，以便自动管理默认值。
alter sequence user_id_seq owned by users.id;

-- 将 users 表的 id 列设置默认值为 user_id_seq 序列的下一个值。这意味着每次插入新行时，如果不提供 id 列的值，则会自动使用序列生成的下一个值。
alter table users alter column id set default nextval('user_id_seq'::regclass);

-- 向 users 表中插入一行数据，设置 username 列的值为 'a'。由于 id 列已经设置为序列 user_id_seq 的默认值，它会自动为 id 列生成一个递增的唯一值。
select setval('user_id_seq',(select max(id) from users));
```

# 错误码

## SQLSTATE 23505

>`SQLSTATE 23505`: 发生在尝试向数据库中插入一条记录时，违反了表中的唯一约束。例如，如果尝试插入一个已经在表中存在的唯一键值，就会触发 SQLSTATE 23505 错误。
>
>例如在使用gorm初始化后添加默认数据时表内已经有这条数据就会爆这个错误,只需判断是这个错误则忽略即可

```go
if len(defaultRoles) > total {
    for _, item := range defaultRoles {
        if err = db.Create(item).Error; err != nil {
            if !strings.Contains(err.Error(), "SQLSTATE 23505") {
                panic(err)
            }
        }
    }

    if err = pg.Client.Exec(`SELECT setval('roles_id_seq', (SELECT MAX(id) FROM roles))`).Error; err != nil {
        logrus.Panicf("reset roles_id_seq err: %v", err)
    }
}
```



## SQLSTATE 42723

>`SQLSTATE 42723`: 创建一个已存在的函数

```go
var (
    err error
    sql = `CREATE FUNCTION array_intersect(anyarray, anyarray)
RETURNS anyarray
language sql
as $FUNCTION$
SELECT ARRAY(
    SELECT UNNEST($1)
    INTERSECT
    SELECT UNNEST($2)
);
$FUNCTION$;`
)

if err = db.Exec(sql).Error; err != nil {
    if !strings.Contains(err.Error(), "SQLSTATE 42723") {
        panic(err)
    }
}
```

# 关键字

## any

>返回any数组内满足全部条件的数据
>
>例如下面sql语句就相当于,遍历any的数组然后判断满足uuid = array[i] 且 user_id = 1 且 view = true 的数据

```sql
select *
from test
where user_id = 1 and uuid = any (array [1001,1002,1003,1004]) and view = true;
```

## conflict

>`ON CONFLICT` 是用于处理插入数据时发生唯一性冲突的关键字。
>
>```sql
>INSERT INTO table_name (column1, column2, ...)
>VALUES (value1, value2, ...)
>ON CONFLICT (conflict_target) DO UPDATE SET column1 = value1, column2 = value2, ...
>```
>
>**conflict_target**：指定可能引起唯一性冲突的列或约束，可以是单个列名或列名的组合，也可以是约束名称。
>
>处理方式:
>
>- `DO NOTHING`：当发生冲突时不执行任何操作。
>- `DO UPDATE SET column1 = value1, column2 = value2, ...`：当发生冲突时执行更新操作，将指定列更新为相应的值
>
>示例:
>
>- INSERT INTO users (id, name) VALUES (1, 'Alice') ON CONFLICT (id) DO UPDATE SET name = 'Alice';
>  - 如果 `users` 表中已存在 `id` 为 1 的记录，那么将更新该记录的 `name` 为 'Alice'；如果不存在，则插入新的记录。
>- INSERT INTO users (id, name) VALUES (1, 'Alice') ON CONFLICT (id) DO NOTHING;
>  - 如果`id`为1的记录已存在，则`不执行任何操作`；如果不存在，则插入新记录

```go
// 向 `sonar_contact_logs` 表中插入数据，如果 `(user_id, uuid)` 存在冲突（即已存在对应记录），则执行更新操作将 `view` 字段设置为 `true`。如果 `(user_id, uuid)` 不存在冲突，则插入新的记录。具体的值会通过占位符 `?` 动态传入。
err = errors.Join(tx.
					Clauses(clause.OnConflict{
						Columns: []clause.Column{{Name: "user_id"}, {Name: "uuid"}},
						DoUpdates: clause.Assignments(map[string]any{
							"view": true,
						}),
					}).
					Create(&model.SonarContactLog{
						UserId:   op.Id,
						Username: op.Username,
						Uuid:     newList[idx],
						View:     true,
					}).Error)
// 对应的sql语句
sql := "
INSERT INTO sonar_contact_logs (user_id, username, uuid, view)
VALUES (?, ?, ?, ?)
ON CONFLICT (user_id, uuid) DO UPDATE SET view = true;
"
```

## Like / iLike / ~

>`~ 和 LIKE 和 ILIKE` 操作符可以**模糊匹配字符串**，LIKE是一般用法，ILIKE匹配时则**不区分字符串的大小写，~** 波浪号则可以使用**正则匹配**

## JOSNB

>1. **存储格式**：
>   - json数据类型存储输入文本的精准拷贝，即原始的JSON文本格式，保留了语法上不明显的空格和JSON对象内部的键的顺序。
>   - jsonb数据类型则存储在一种分解好的二进制格式中，不保留原始JSON文本的空格和键的顺序。
>2. **处理效率**：
>   - json类型在处理时需要每次重新解析输入文本，因此在处理速度上较慢。
>   - jsonb类型由于存储在二进制格式中，不需要解析，因此在处理速度上更快。
>3. **索引支持**：
>   - json类型在列上无法直接创建索引，但可以在列上建函数索引。
>   - jsonb类型的列上可以直接创建索引，并且支持GIN索引，这可以高效地从jsonb内部的key/value中搜索数据。
>4. **数据保留特性**：
>   - json类型保留重复的对象键，所有的键/值对都会被保留。
>   - jsonb类型不保留重复的对象键，如果在输入中指定了重复的键，只有最后一个值会被保留。

| 操作符 | 右操作数类型 | 描述                                            | 例子                                             | 例子结果     |
| :----- | :----------- | :---------------------------------------------- | :----------------------------------------------- | :----------- |
| ->     | int          | 获得 JSON 数组元素（索引从 0 开始，负整数结束） | ‘[{“a”:“foo”},{“b”:“bar”},{“c”:“baz”}]’::json->2 | {“c”:“baz”}  |
| ->     | text         | 通过键获得 JSON 对象域                          | ‘{“a”: {“b”:“foo”}}’::json->‘a’                  | {“b”:“foo”}  |
| ->>    | int          | 以文本形式获得 JSON 数组元素                    | ‘[1,2,3]’::json->>2                              | 3            |
| ->>    | text         | 以文本形式获得 JSON 对象域                      | ‘{“a”:1,“b”:2}’::json->>‘b’                      | 2            |
| #>     | text[]       | 获取在指定路径的 JSON 对象                      | ‘{“a”: {“b”:{“c”: “foo”}}}’::json#>‘{a,b}’       | {“c”: “foo”} |
| #>>    | text[]       | 以文本形式获取在指定路径的 JSON 对象            | ‘{“a”:[1,2,3],“b”:[4,5,6]}’::json#>>‘{a,2}’      | 3            |

```sql
select * from tags where 'a' = any (ids);

select * from tags where name ->> 'name' like '%a%';


select *
from tags
where name -> 'one' ->> 'two' like '%a%';


select jsonb_array_length(name -> 'arr') as len
from tags;

// {"arr": [1, 2, 3, 4, 5], "one": {"two": "1234"}, "name": null, "test": "abc"}
// jsonb_set 有则更新没有添加
update tags set name = jsonb_set(name,'{name}','"lisi"') where id = 1;

update tags set name = jsonb_set(name,'{one,two}','"1234"') where id = 1;



select tags.name -> 'arr' from tags;

```

## Array

| 操作符                                                       |
| ------------------------------------------------------------ |
| =	等于	ARRAY[1,2,3] = ARRAY[1,2,3]	true             |
| <>	不等于	ARRAY[1,2,3] <> ARRAY[1,2,4]	true         |
| <	小于	ARRAY[1,2,3] < ARRAY[1,2,4]	true             |
| >	大于	ARRAY[1,4,3] > ARRAY[1,2,4]	true             |
| <=	小于或等于	ARRAY[1,2,3] <= ARRAY[1,2,3]	true     |
| >=	大于或等于	ARRAY[1,4,3] >= ARRAY[1,4,3]	true     |
| @>	包含	ARRAY[1,4,3] @> ARRAY[3,1]	true             |
| <@	被包含于	ARRAY[2,7] <@ ARRAY[1,7,4,2,6]	true     |
| &&	重叠(有共同元素)	ARRAY[1,4,3] && ARRAY[2,1]	true |
| \|\|         数组与数组连接	ARRAY[1,2,3]  \|\| ARRAY[4,5,6]	{1,2,3,4,5,6} |

array_append	向数组的末尾添加元素	array_append(ARRAY[1,2], 3)	{1,2,3}
array_prepend	向数组的开头添加函数	array_prepend(1, ARRAY[2,3])	{1,2,3}
array_cat	连接两个数组	array_cat(ARRAY[1,2,3], ARRAY[4,5])	{1,2,3,4,5}
array_replace	用新值替换每个等于给定值的数组元素	array_replace(ARRAY[1,2,5,4], 5, 3)	{1,2,3,4}
array_remove	删除某个给定值的数组元素	array_remove(ARRAY[1,2,3,4],4)	{1,2,3}
[start:end]	通过下标范围数组元素	select (ARRAY[‘a’,‘b’,‘c’,‘d’])[1:2]	{a,b}
array_position	数组中指定元素出现的位置	array_position(ARRAY[1,8,3,7], 8)	2
array_dims	返回数组维数的文本表示	array_dims(ARRAY[[1,2,3], [4,5,6]])	[1:2][1:3]
array_lower	返回数组维数的下界	array_lower(ARRAY[1,2,3,4], 1)	1
array_upper	返回数组维数的上界	array_upper(ARRAY[1,2,3,4], 1)	4
array_ndims	返回数组的维数	array_ndims(ARRAY[[1,2,3], [4,5,6]])	2
array_length	返回数组维度的长度	array_length(array[1,2,3], 1)	3
cardinality	返回数组中的总元素数量，如果数组是空的则为0	cardinality(ARRAY[1,2,3,5])	4
array_to_string	将数组转换为字符串，使用分隔符连接数组元素	select array_to_string(ARRAY[1, 2, 3], ‘,’)	1,2,3
string_to_array	使用指定的分隔符把字符串分隔成数组元素	select string_to_array(‘a,b,c’,‘,’)	{a,b,c}
array_agg	把多个值合并到一个数组中	SELECT id, array_agg(label) FROM t_label group by id	{label1,label2}
unnest	分解一个数组为一组行	select unnest(ARRAY[1,2,3])	1     2    3

```sql
-- 添加
UPDATE messages SET status = status || '{uuid1,uuid2}' WHERE id = 1;

update messages SET status = array_append(status,'uuid3') where id = 1;
-- 删除
UPDATE messages SET cat_user_ids = array_remove(cat_user_ids, '3') WHERE id = 1;

-- 判断元素是否在某一个字段内
select * from messages where '1'::text = any(cat_user_ids);
```



```sql
select version();

select * from tags where array['a']<@ ids;

alter table tags add column ns text[];

alter table tags alter column ns type text, alter column new set default 'default';

alter table tags rename column ns to new;

alter table tags drop column ns;
```

