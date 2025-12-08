---
title: "Gorm Gen"
date: 2025-07-23T09:11:49+08:00
author: ["loveyu"]
draft: false
categories:
- go
tags:
- gorm
- gen
- sql

---

# 约束冲突

>1. **Columns** ([]clause.Column)
>   - 作用：指定可能引发冲突的列名
>   - 意义：当设置为 nil 时，GORM 会尝试自动检测可能导致冲突的列
>2. **Where** (clause.Where)
>   - 作用：为冲突操作添加 WHERE 条件
>   - 意义：限制哪些冲突行会被处理（PostgreSQL 特有）
>3. **TargetWhere** (clause.Where)
>   - 作用：指定目标约束的 WHERE 条件
>   - 意义：帮助更精确地识别冲突（PostgreSQL 特有）
>4. **OnConstraint** (string)
>   - 作用：直接指定约束名称
>   - 意义：当你知道确切的约束名时，可以直接指定而不是让 GORM 推断
>5. **DoNothing** (bool)
>   - 作用：是否在冲突时不执行任何操作
>   - 意义：设置为 true 时，遇到冲突直接忽略，不插入也不更新（类似 INSERT IGNORE）
>6. **DoUpdates** (interface{})
>   - 作用：指定冲突时要更新的列和值
>   - 意义：可以是一个结构体或 map，指定要更新的字段
>7. **UpdateAll** (bool)
>   - 作用：是否更新所有列（冲突时）
>   - 意义：设置为 true 时，会更新所有列（除了主键）

```go
table.Clauses(clause.OnConflict{
  Columns:      nil,
  Where:        clause.Where{},
  TargetWhere:  clause.Where{},
  OnConstraint: "",
  DoNothing:    false,
  DoUpdates:    nil,
  UpdateAll:    false,
})

// 冲突时不执行任何操作
table.Clauses(clause.OnConflict{DoNothing: true}).Create(&data)

// 冲突时更新特定字段
table.Clauses(clause.OnConflict{
    Columns:   []clause.Column{{Name: "unique_field"}},
    DoUpdates: clause.Assignments(map[string]interface{}{"update_field": "new_value"}),
}).Create(&data)

// 冲突时更新所有字段
table.Clauses(clause.OnConflict{UpdateAll: true}).Create(&data)

// 指定约束名称处理冲突
table.Clauses(clause.OnConflict{
    OnConstraint: "unique_constraint_name",
    DoUpdates:    clause.Assignments(map[string]interface{}{"field": "value"}),
}).Create(&data)
```



# 自定义字段

>`field.NewUnsafeFieldRaw(表名称,字段名称)`可以用在`select,where`中

```go
simpleTable, err := table.
		Select(field.NewUnsafeFieldRaw("id+year_field").As("number")).
		Where(field.NewUnsafeFieldRaw("date(timestamp_field) = ?", time.Now().Format("2006-01-02"))).First()
```



>`field.Newxxx(表名称,字段名称)`表名称可以为空,为空则为当前表,该方法生成的sql是`会加上单引号`,很多场景使用都会有问题
>
>例如下边这个生成的sql为:` SELECT * FROM `simple_table` WHERE `date(timestamp_field)` = '2025-07-23 09:22:04.337' ORDER BY `simple_table`.`id` LIMIT 1`会报错:`Error 1054 (42S22): Unknown column 'date(timestamp_field)' in 'where clause'` 

```go
first, err := table.Where(field.NewTime("", "date(timestamp_field)").Eq(time.Now())).First()
	if err != nil {
		t.Error(err)
	}
```



# 表子语句

>当from的对象不是一个特定表而是一个sql执行的结果时使用
>
>子语句sql需要加上别名

```go
users, err := gen.Table(u.Select(u.Name, u.Age).As("u")).Where(u.Age.Eq(18)).Find()
// SELECT * FROM (SELECT `name`,`age` FROM `users`) as u WHERE `age` = 18
```





# 查询并操作

## FirstOrInit

>如果没有找到记录，可以使用包含更多的属性的结构体初始化 user，`Attrs` 不会被用于生成查询 SQL
>
>两种使用方式
>
>1. Attrs中使用field.Attrs(结构体),在结构体中写入值
>2. Attrs中使用表.字段.Value(值)

```go
u.WithContext(ctx).Attrs(field.Attrs(&model.User{Age: 20})).Where(u.Name.Eq("non_existing")).FirstOrInit()

u.WithContext(ctx).Attrs(u.Age.Value(20).Where(u.Name.Eq("non_existing")).FirstOrInit()

```

>不管是否找到记录，`Assign` 都会将属性赋值给 struct，但这些属性不会被用于生成查询 SQL，也不会被保存到数据库
>
>`Assign和Attrs区别就是 Assign无论是否找到记录都会给提前预输入的值写入,Attrs则是只有在没有找到记录的时候才会写入` 

```go
// User not found, initialize it with give conditions and Assign attributes
u.WithContext(ctx).Assign(field.Attrs(map[string]interface{}{"age": 20})).Where(u.Name.Eq("non_existing")).FirstOrInit()
// user -> User{Name: "non_existing", Age: 20}

// Found user with `name` = `gen`, update it with Assign attributes
u.WithContext(ctx).Assign(field.Attrs(&model.User{Name: "gen_assign"}).Select(dal.User.ALL)).Where(u.Name.Eq("gen")).FirstOrInit()

// SELECT * FROM USERS WHERE name = gen' ORDER BY id LIMIT 1;
// user -> User{ID: 111, Name: "gen", Age: 20}
```



## FirstOrCreate

>如果没有找到记录则会创建该记录,与约束冲突刚好相反,约束冲突是存在记录冲突后创建,FirstOrCreate则是查询不到这条记录创建
>
>与FirstOrInit一样,FirstOrCreate也存在Attrs和Assign两个方法,区别是
>
>`Assign无论是否找到记录都会把预输入值写入并创建该记录,如果是没有找到该记录则会创建,如果有改记录则会修改,Attrs只有找不到才会创建该记录`

```go
// SELECT * FROM users WHERE name = 'non_existing' ORDER BY id LIMIT 1;
// INSERT INTO "users" (name, age) VALUES ("non_existing", 20);
u.WithContext(ctx).Attrs(field.Attrs(&model.User{Age: 20})).Where(u.Name.Eq("non_existing")).FirstOrCreate()


u.WithContext(ctx).Assign(field.Attrs(&model.User{Age: 20})).Where(u.Name.Eq("non_existing")).FirstOrCreate()
// SELECT * FROM users WHERE name = 'non_existing' ORDER BY id LIMIT 1;
// INSERT INTO "users" (name, age) VALUES ("non_existing", 20);
// user -> User{ID: 112, Name: "non_existing", Age: 20}

// Found user with `name` = `gen`, update it with Assign attributes
u.WithContext(ctx).Assign(field.Attrs(&model.User{Age: 20})).Where(u.Name.Eq("gen")).FirstOrCreate()
// SELECT * FROM users WHERE name = 'gen' ORDER BY id LIMIT 1;
// UPDATE users SET age=20 WHERE id = 111;
// user -> User{ID: 111, Name: "gen", Age: 20}
```



# 零值

>如果查询中需要使用到零值,直接使用结构体是不行的需要使用map

```go
u.WithContext(ctx).Where(field.Attrs(&User{Name: "gen", Age: 0})).Find()
// SELECT * FROM users WHERE name = "gen";

u.WithContext(ctx).Where(field.Attrs(map[string]interface{}{"name": "gen", "age": 0})).Find()
// SELECT * FROM users WHERE name = "gen" AND age = 0;
```



# 选择条件

>如果使用结构体查询时默认使用所有非零值,可以使用`select,omit`传入字段名来指定需要使用的字段
>
>`select选择使用的字段,omit选择不使用的字段`

```go
u.WithContext(ctx).Where(field.Attrs(&User{Name: "gen"}).Select(u.Name,u.Age)).Find()
// SELECT * FROM users WHERE name = "gen" AND age = 0;

u.WithContext(ctx).Where(field.Attrs(&User{Name: "gen"}).Select(u.Age)).Find()
// SELECT * FROM users WHERE age = 0;
```



# 多字段in

```go
users, err := u.WithContext(ctx).Where(u.WithContext(ctx).Columns(u.ID, u.Name).In(field.Values([][]interface{}{{1, "modi"}, {2, "zhangqiang"}}))).Find()
// SELECT * FROM `users` WHERE (`id`, `name`) IN ((1,'humodi'),(2,'tom'));
```



# json查询

>查询table表中的字段包含value的数据
>
>JSONArrayQuery(字段名称).Contains(包含的值)

```go
table.Where(gen.Cond(datatypes.JSONArrayQuery(table.Arr.ColumnName().String()).Contains(9))...).Find()
```

>查询JSONFieldjson字段下的age等于22的数据
>
>`查询数组用的是JSONArrayQuery,查询json字段属性用的是JSONQuery` 

```go
table.Where(gen.Cond(datatypes.JSONQuery(table.JSONField.ColumnName().String()).Equals(22, "age"))...).Find()
```

>`table.Where(gen.Cond(datatypes.JSONQuery(table.JSONField.ColumnName().String()).Likes("wa%", "name"))...).Find()`使用Like模糊查询执行的sql为 [SELECT * FROM `simple_table` WHERE JSON_EXTRACT(`json_field`,'$.name') LIKE 'wa%'] 无法查询出数据,原因是`JSON_EXTRACT返回的是json不是字符串无法进行like操作`,需要转为字符串然后再进行like,目前没找到转字符串方法
>
>改为使用自定义字段查询就可以查询到需要的值
>
>`JSON_UNQUOTE()`转json为字符串

```go
table.Where(field.NewUnsafeFieldRaw("JSON_UNQUOTE(JSON_EXTRACT(`json_field`, '$.username')) like ?", "wa%")).Find()
// 对应sql: SELECT * FROM `simple_table` WHERE JSON_UNQUOTE(JSON_EXTRACT(`json_field`, '$.username')) like 'wa%'
```

