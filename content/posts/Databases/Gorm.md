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