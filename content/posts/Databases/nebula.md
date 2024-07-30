---
title: "nebula"
date: 2024-03-24T19:38:37+08:00
author: ["loveyu"]
draft: false
categories: 
- databases
tags: 
- nebula
- databases
---

>[官网文档](https://docs.nebula-graph.com.cn/3.6.0/2.quick-start/3.quick-start-on-premise/4.nebula-graph-crud/#_10)

# 初始化

## 添加Storage 主机

>刚部署好不添加主机`有可能`会报错

```shell
ADD HOSTS "storaged0":9779,"storaged1":9779,"storaged2":9779
```



# 图

## 创建

> 必须指定vid_type,partition_num:分区数，通常为全集群硬盘数量的 5 倍;replica_factor：指定集群中副本的数量，通常生产环境为 3，测试环境为 1
>
> 如果报错提示`[ERROR (-1005)]: Host not enough!`，请检查是否已添加 Storage 主机。

```shell
CREATE SPACE [IF NOT EXISTS] <graph_space_name> (
[partition_num = <partition_number>,]
[replica_factor = <replica_number>,]
vid_type = {FIXED_STRING(<N>) | INT64}
)
[COMMENT = '<comment>'];

# 实例
CREATE SPACE IF NOT EXISTS hzy (partition_num = 1, replica_factor=1, vid_type=fixed_string(30)) COMMENT='测试图';
```

```shell
nebula> CREATE SPACE basketballplayer(partition_num=15, replica_factor=1, vid_type=fixed_string(30));
```



## 查看

```shell
SHOW SPACES;
```

## 检查分片分布

>如果 **Leader distribution** 分布不均匀，请执行命令`BALANCE LEADER`重新分配。

```shell
SHOW HOSTS
```

## 使用

```shell
USE space_name;
```



# Tag/Edge type

## 创建

```shell
CREATE {TAG | EDGE} [IF NOT EXISTS] {<tag_name> | <edge_type_name>}
    (
      <prop_name> <data_type> [NULL | NOT NULL] [DEFAULT <default_value>] [COMMENT '<comment>']
      [{, <prop_name> <data_type> [NULL | NOT NULL] [DEFAULT <default_value>] [COMMENT '<comment>']} ...] 
    )
    [TTL_DURATION = <ttl_duration>]
    [TTL_COL = <prop_name>]
    [COMMENT = '<comment>'];

# 实例
create tag if not exists tag1 (name string not null default 'default_value' comment '测试tag1');
create edge if not exists dege1 (start int comment '测试dege1');
```

```shell
nebula> CREATE TAG player(name string, age int);

nebula> CREATE TAG team(name string);

nebula> CREATE EDGE follow(degree int);

nebula> CREATE EDGE serve(start_year int, end_year int);
```



## 插入数据

### 点

>`vid`是 Vertex ID 的缩写，`vid`在一个图空间中是唯一的。

```shell
INSERT VERTEX [IF NOT EXISTS] [tag_props, [tag_props] ...]
VALUES <vid>: ([prop_value_list])

tag_props:
  tag_name ([prop_name_list])

prop_name_list:
   [prop_name [, prop_name] ...]

prop_value_list:
   [prop_value [, prop_value] ...]  

# 实例
insert vertex tag1(name) values "v1":("v1-name");
insert vertex tag1(name),tag2(age) values "v2":("v2-name",14);
```

```shell
nebula> INSERT VERTEX player(name, age) VALUES "player100":("Tim Duncan", 42);

nebula> INSERT VERTEX player(name, age) VALUES "player101":("Tony Parker", 36);

nebula> INSERT VERTEX player(name, age) VALUES "player102":("LaMarcus Aldridge", 33);

nebula> INSERT VERTEX team(name) VALUES "team203":("Trail Blazers"), "team204":("Spurs");
```



### 边

```shell
INSERT EDGE [IF NOT EXISTS] <edge_type> ( <prop_name_list> ) VALUES 
<src_vid> -> <dst_vid>[@<rank>] : ( <prop_value_list> )
[, <src_vid> -> <dst_vid>[@<rank>] : ( <prop_value_list> ), ...];

<prop_name_list> ::=
[ <prop_name> [, <prop_name> ] ...]

<prop_value_list> ::=
[ <prop_value> [, <prop_value> ] ...]

# 实例
insert edge dege1(start) values "v1" -> "v2":(95);
```

```shell
nebula> INSERT EDGE follow(degree) VALUES "player101" -> "player100":(95);

nebula> INSERT EDGE follow(degree) VALUES "player101" -> "player102":(90);

nebula> INSERT EDGE follow(degree) VALUES "player102" -> "player100":(75);

nebula> INSERT EDGE serve(start_year, end_year) VALUES "player101" -> "team204":(1999, 2018),"player102" -> "team203":(2006,  2015);
```



## 查询

### GO

>[GO](https://docs.nebula-graph.com.cn/3.6.0/3.ngql-guide/7.general-query-statements/3.go/) 语句可以根据指定的条件遍历数据库。`GO`语句从一个或多个点开始，沿着一条或多条边遍历，返回`YIELD`子句中指定的信息。

```shell
GO [[<M> TO] <N> {STEP|STEPS}] FROM <vertex_list>
OVER <edge_type_list> [{REVERSELY | BIDIRECT}]
[ WHERE <conditions> ]
YIELD [DISTINCT] <return_list>
[{ SAMPLE <sample_list> | <limit_by_list_clause> }]
[ | GROUP BY {<col_name> | expression> | <position>} YIELD <col_name>]
[ | ORDER BY <expression> [{ASC | DESC}]]
[ | LIMIT [<offset>,] <number_rows>];
```

>从 VID 为`player101`的球员开始，沿着边`follow`找到连接的球员。

```shell
GO FROM "player101" OVER follow YIELD id($$)
```

>从 VID 为`player101`的球员开始，沿着边`follow`查找年龄大于或等于 35 岁的球员，并返回他们的姓名和年龄，同时重命名对应的列。
>
>| YIELD | 指定该查询需要返回的值或结果。 |
>| :---: | :----------------------------: |
>|  $$   |         表示边的终点。         |
>|   \   |       表示换行继续输入。       |
>
>

```shell
go from "player101" over follow where properties($$).age >= 35 yield properties($$).name as Teamate,properties($$).age as Age;
```

>从 VID 为`player101`的球员开始，沿着边`follow`查找连接的球员，然后检索这些球员的球队。为了合并这两个查询请求，可以使用管道符或临时变量。
>
>|  $^  |                       表示边的起点。                       |
>| :--: | :--------------------------------------------------------: |
>|  \|  | 组合多个查询的管道符，将前一个查询的结果集用于后一个查询。 |
>|  $-  |             表示管道符前面的查询输出的结果集。             |
>
>1. `GO FROM "player101"`：这是起始点，从图数据库中的节点 "player101" 开始查询。
>2. `OVER follow YIELD dst(edge) AS id`：在从起始节点出发后，沿着名为 "follow" 的边进行遍历，将沿途经过的目标节点（即边的终点）的标识符保存为变量 `id`。这个变量会在后续的查询中使用。
>3. `GO FROM $-.id OVER serve`：这一部分使用了前面查询得到的目标节点的标识符 `$-.id`，表示从上一步得到的目标节点作为起始节点。然后，沿着名为 "serve" 的边进行遍历。
>4. `YIELD properties($$).name AS Team, properties($^).name AS Player`：在遍历过程中，使用 `YIELD` 关键字来返回结果。`properties($$).name` 表示从当前节点出发，获取边的属性，然后获取属性名为 "name" 的值作为 Team；`properties($^).name` 表示从当前节点出发，获取边的起始节点的属性，然后获取属性名为 "name" 的值作为 Player。

```shell
GO FROM "player101" OVER follow YIELD dst(edge) AS id | \
        GO FROM $-.id OVER serve YIELD properties($$).name AS Team, \
        properties($^).name AS Player;
```



### FETCH

>[FETCH](https://docs.nebula-graph.com.cn/3.6.0/3.ngql-guide/7.general-query-statements/4.fetch/) 语句可以获得点或边的属性。

```shell
# tag属性
FETCH PROP ON {<tag_name>[, tag_name ...] | *}
<vid> [, vid ...]
YIELD <return_list> [AS <alias>];

# 边属性
FETCH PROP ON <edge_type> <src_vid> -> <dst_vid>[@<rank>] [, <src_vid> -> <dst_vid> ...]
YIELD <output>;
```

>查询 VID 为`player100`的球员的属性。

```shell
fetch prop on player "player100" yield properties(vertex);
```

>查询点`player101`到`player102`的边属性

```shell
FETCH PROP ON follow "player101"->"player102" yield properties(edge);
```

### LOOKUP

>[LOOKUP](https://docs.nebula-graph.com.cn/3.6.0/3.ngql-guide/7.general-query-statements/5.lookup/) 语句是基于[索引](https://docs.nebula-graph.com.cn/3.6.0/2.quick-start/3.quick-start-on-premise/4.nebula-graph-crud/#_12)的，和`WHERE`子句一起使用，查找符合特定条件的数据。

```shell
LOOKUP ON {<vertex_tag> | <edge_type>}
[WHERE <expression> [AND <expression> ...]]
YIELD <return_list> [AS <alias>];

<return_list>
    <prop_name> [AS <col_alias>] [, <prop_name> [AS <prop_alias>] ...];
```

>`player.name`必须是已经创建索引

```shell
LOOKUP ON player WHERE player.name == "Tony Parker" \
        YIELD properties(vertex).name AS name, properties(vertex).age AS age;
```



### MATCH

>[MATCH](https://docs.nebula-graph.com.cn/3.6.0/3.ngql-guide/7.general-query-statements/2.match/) 语句是查询图数据最常用的，可以灵活的描述各种图模式，但是它依赖[索引](https://docs.nebula-graph.com.cn/3.6.0/2.quick-start/3.quick-start-on-premise/4.nebula-graph-crud/#_12)去匹配 NebulaGraph 中的数据模型，性能也还需要调优。

```shell
MATCH <pattern> [<clause_1>]  RETURN <output>  [<clause_2>];
```

>要查询的必须是索引

```shell
MATCH (v:player{name:"Tony Parker"}) RETURN v;
```



## 修改

```shell
# UPDATE点
UPDATE VERTEX <vid> SET <properties to be updated>
[WHEN <condition>] [YIELD <columns>];

# UPDATE边
UPDATE EDGE ON <edge_type> <source vid> -> <destination vid> [@rank] 
SET <properties to be updated> [WHEN <condition>] [YIELD <columns to be output>];

# UPSERT点或边
UPSERT {VERTEX <vid> | EDGE <edge_type>} SET <update_columns>
[WHEN <condition>] [YIELD <columns>];
```

>修改点`player100`的` player.name='Tim'` 

```shell
update vertex "player100" set player.name='Tim';
```

>修改点`"player101"->"player102"`的属性`degree=92` 

```shell
update edge on follow "player101"->"player102" set degree=92
```

>根据条件修改

```shell
INSERT VERTEX player(name,age) VALUES "player111":("David West", 38);

UPSERT VERTEX "player111" SET player.name = "David", player.age = $^.player.age + 11 \
        WHEN $^.player.name == "David West" AND $^.player.age > 20 \
        YIELD $^.player.name AS Name, $^.player.age AS Age;
```

## 删除

```shell
# 删除点
DELETE VERTEX <vid1>[, <vid2>...]

# 删除边
DELETE EDGE <edge_type> <src_vid> -> <dst_vid>[@<rank>]
[, <src_vid> -> <dst_vid>...]
```

```shell
DELETE VERTEX "player111", "team203";

DELETE EDGE follow "player101" -> "team204";
```

# 索引

>为没有指定长度的变量属性创建索引时，需要指定索引长度。在 utf-8 编码中，一个中文字符占 3 字节，请根据变量属性长度设置合适的索引长度。例如 10 个中文字符，索引长度需要为 30。详情请参见[创建索引](https://docs.nebula-graph.com.cn/3.6.0/3.ngql-guide/14.native-index-statements/1.create-native-index/)。

```shell
# 创建索引
CREATE {TAG | EDGE} INDEX [IF NOT EXISTS] <index_name>
ON {<tag_name> | <edge_name>} ([<prop_name_list>]) [COMMENT = '<comment>'];

# 创建索引
REBUILD {TAG | EDGE} INDEX <index_name>;
```



## 创建

>为 name 属性创建索引 player_index_1。

```shell
create tag index if not exists player_index_1 on player(name(20));
```

## 重建

>重建索引确保能对已存在数据生效。

```shell
REBUILD TAG INDEX player_index_1;
```

