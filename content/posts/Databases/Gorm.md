---
title: "Gorm"
date: 2023-07-21T11:18:37+08:00
author: ["loveyu"]
draft: false
categories: 
- go
tags: 
- go
- gorm
---

# 一对多

>gorm:"column:a;unique"
>
>``unique:`` 唯一约束 
>
>gorm:"foreignKey:C;references:A"
>
>``foreignKey:`` 设置外键
>
>``references:`` 重写引用
>
>1. 引用的字段必须是唯一约束,不然使用``AutoMigrate`` 创建表会报错, 这里的引用意思是查询Stu时会查询出Demo中C和A相同的数据
>
>2. ``D []Demo`` 不会创建为表字段,不用设置``gorm:"-"``
>
>3. 在查询时必须使用``Preload("外键字段名称,这里为D")`` 加载,不然查询出的结果中没有该数据
>
>4. 如果有多重引用例如Demo中的E则``Preload("D.E")``
>
>5. ```go 
>    opt.DB.Model(project).
>
>  		Preload("Lists").
>  		Preload("Lists.HTTPS").Preload("Lists.GRPCS").
>  		Where("id = ?", 1).Take(project)
>  ```

```go
type Stu struct {
	A string `json:"a,omitempty" gorm:"column:a;unique"`
	B string `json:"b,omitempty" gorm:"column:b"`
	D []Demo `json:"d,omitempty" gorm:"foreignKey:C;references:A"`
}

type Demo struct {
	C string `json:"c,omitempty" gorm:"column:c"`
	D string `json:"d,omitempty" gorm:"column:d"`
}
```



# 多态关联

>``gorm:"polymorphic:Owner;polymorphicValue:master"`` 使用``polymorphicValue``更改多态类型的值,这里指定为``master``则Type字段的值就为``master`` 

```go
type Comment struct {
	ID              uint        `json:"id,omitempty" gorm:"column:id;unique"`
	Body            string      `json:"body,omitempty" gorm:"column:body"`
	CommentableID   uint        `json:"commentable_id,omitempty" gorm:"column:commentable_id"`
	CommentableType string      `json:"commentable_type,omitempty" gorm:"column:commentable_type"`
	Commentable     interface{} `json:"commentable,omitempty" gorm:"-"`
}

type Post struct {
	ID       uint       `json:"id,omitempty" gorm:"column:id;unique"`
	Title    string     `json:"title,omitempty" gorm:"column:title"`
	Comments []*Comment `gorm:"polymorphic:Commentable;" json:"comments,omitempty"`
}

type Video struct {
	ID       uint       `json:"id,omitempty" gorm:"column:id;unique"`
	Title    string     `json:"title,omitempty" gorm:"column:title"`
	Comments []*Comment `gorm:"polymorphic:Commentable;" json:"comments,omitempty"`
}

============== 创建数据 ==================
	opt.DB.AutoMigrate(&Comment{}, &Post{}, &Video{})

	post := &Post{
		Title: "帖子1",
	}
	c := &Comment{
		Body: "评论2",
	}
	post.Comments = append(post.Comments, c)
	opt.DB.Create(post)
============== 添加数据 ==================
	post := new(Post)
	opt.DB.Preload("Comments").Where("id = ?", 1).Find(post)
	for i := range post.Comments {
		t.Log(post.Comments[i])
	}
	post.Comments = append(post.Comments, &Comment{Body: "新的评论"})

	opt.DB.Save(post)
```



# Save和Create区别

>``Create:``只能用于创建新的数据
>
>``Save:`` 会根据结构体中的主键字段是否存在而做到创建或修改操作



# Preload和Joins区别

>``Preload:``  用于预加载关联的数据，并将其填充到查询的结果中。它会在单个查询中同时获取主表和关联表的数据。通过 `Preload` 可以避免 N+1 查询问题，提高查询效率。
>
>`Joins` 方法用于进行内连接（inner join）操作，通过关联表之间的关系将查询结果合并为单个结果集。它允许你以更自定义的方式编写 JOIN 语句。
>
>``N+1 查询问题:`` 是指在某个关联查询的场景下，如果你使用常规的查询方式，可能会导致对关联表进行多次单独的查询，从而导致性能下降。



# 标签

| 标签名                 | 说明                                                         |
| :--------------------- | :----------------------------------------------------------- |
| column                 | 指定 db 列名                                                 |
| type                   | 列数据类型，推荐使用兼容性好的通用类型，例如：所有数据库都支持 bool、int、uint、float、string、time、bytes 并且可以和其他标签一起使用，例如：`not null`、`size`, `autoIncrement`… 像 `varbinary(8)` 这样指定数据库数据类型也是支持的。在使用指定数据库数据类型时，它需要是完整的数据库数据类型，如：`MEDIUMINT UNSIGNED not NULL AUTO_INCREMENT` |
| serializer             | 指定将数据序列化或反序列化到数据库中的序列化器, 例如: `serializer:json/gob/unixtime` |
| size                   | 定义列数据类型的大小或长度，例如 `size: 256`                 |
| primaryKey             | 将列定义为主键                                               |
| unique                 | 将列定义为唯一键                                             |
| default                | 定义列的默认值                                               |
| precision              | 指定列的精度                                                 |
| scale                  | 指定列大小                                                   |
| not null               | 指定列为 NOT NULL                                            |
| autoIncrement          | 指定列为自动增长                                             |
| autoIncrementIncrement | 自动步长，控制连续记录之间的间隔                             |
| embedded               | 嵌套字段                                                     |
| embeddedPrefix         | 嵌入字段的列名前缀                                           |
| autoCreateTime         | 创建时追踪当前时间，对于 `int` 字段，它会追踪时间戳秒数，您可以使用 `nano`/`milli` 来追踪纳秒、毫秒时间戳，例如：`autoCreateTime:nano` |
| autoUpdateTime         | 创建/更新时追踪当前时间，对于 `int` 字段，它会追踪时间戳秒数，您可以使用 `nano`/`milli` 来追踪纳秒、毫秒时间戳，例如：`autoUpdateTime:milli` |
| index                  | 根据参数创建索引，多个字段使用相同的名称则创建复合索引，查看 [索引](https://gorm.io/zh_CN/docs/indexes.html) 获取详情 |
| uniqueIndex            | 与 `index` 相同，但创建的是唯一索引                          |
| check                  | 创建检查约束，例如 `check:age > 13`，查看 [约束](https://gorm.io/zh_CN/docs/constraints.html) 获取详情 |
| <-                     | 设置字段写入的权限， `<-:create` 只创建、`<-:update` 只更新、`<-:false` 无写入权限、`<-` 创建和更新权限 |
| ->                     | 设置字段读的权限，`->:false` 无读权限                        |
| -                      | 忽略该字段，`-` 表示无读写，`-:migration` 表示无迁移权限，`-:all` 表示无读写迁移权限 |
| comment                | 迁移时为字段添加注释                                         |

# OnConflict

> 数据库为pgsql时gorm指定字段为primaryKey则会自动创建对应序列并绑定

>`uniqueIndex:user_privilege_unique"` 
>
>`Columns:   []clause.Column{{Name: "user_id"}, {Name: "department_id"}},` 这里指定的字段的`索引必须一样` 

```go
type UserPrivilege struct {
	Id        uint64 `json:"id" gorm:"primaryKey;column:id"`
	CreatedAt int64  `json:"created_at" gorm:"column:created_at;autoCreateTime:milli"`
	UpdatedAt int64  `json:"updated_at" gorm:"column:updated_at;autoUpdateTime:milli"`
	DeletedAt int64  `json:"deleted_at" gorm:"index;column:deleted_at;default:0"`

	UserID       string                   `json:"user_id" gorm:"column:user_id;uniqueIndex:user_privilege_unique"`
	DepartmentID string                   `json:"department_id" gorm:"column:department_id;uniqueIndex:user_privilege_unique"`
}
if err = pg.ClientV2(tools.Timeout()).
			Model(&model.UserPrivilege{}).
			Debug().
			Clauses(clause.OnConflict{
				Columns:   []clause.Column{{Name: "user_id"}, {Name: "department_id"}},
				DoUpdates: clause.Assignments(map[string]any{"privilege_ids": userPrivilege.PrivilegeIDs}),
			}).
			Create(&userPrivilege).Error; err != nil {
			return response.Resp500(c, err.Error())
		}
```

