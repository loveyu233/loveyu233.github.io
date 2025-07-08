---
title: "Gorm"
date: 2020-10-04T19:38:37+08:00
author: ["loveyu"]
draft: false
categories: 
- go
tags: 
- go
- gorm
---


# 关联关系

>关联关系必须要求外键

## 关联标签

| 标签             | 描述                                     |
| :--------------- | :--------------------------------------- |
| foreignKey       | 指定当前模型的列作为连接表的外键         |
| references       | 指定引用表的列名，其将被映射为连接表外键 |
| polymorphic      | 指定多态类型，比如模型名                 |
| polymorphicValue | 指定多态值、默认表名                     |
| many2many        | 指定连接表表名                           |
| joinForeignKey   | 指定连接表的外键列名，其将被映射到当前表 |
| joinReferences   | 指定连接表的外键列名，其将被映射到引用表 |
| constraint       | 关系约束，例如：`OnUpdate`、`OnDelete`   |

## 无关联

### CRUD

```go
	DB.Where("id > ?", 2).Select("id", "name").Order("id").Find(&users)

```

## BeLongs To：属于

>` belongs to` 会与另一个模型建立了一对一的连接。 这种模型的每一个实例都“属于”另一个模型的一个实例。

```go
// `User` 属于 `Company`，`CompanyID` 是外键
type User struct {
	gorm.Model
	Name      string
	CompanyID int
	Company   Company
}

type Company struct {
	ID   int
	Name string
}
```



### 重写外键

>`foreignKey`：重写外键，可以不用外键表+ID这种形式
>
>默认外键是外键表的名称+ID，创建顺序为先创建User后创建CreditCard；DB.AutoMigrate(&User{}, &CreditCard{})

```go
type User struct {
  gorm.Model
  Name         string
  CompanyRefer int
  Company      Company `gorm:"foreignKey:CompanyRefer"`
  // 使用 CompanyRefer 作为外键
}

type Company struct {
  ID   int
  Name string
}
```



### 重写引用

>`references`：重写引用，默认引用的是ID，使用references可以更改引用；
>
>必须要在外键和被引用外键都标注`gorm:"index"`

```go
type User struct {
	gorm.Model
	Name      string
	CompanyID string  `gorm:"index"`
	Company   Company `gorm:"references:Code"` // 使用 Code 作为引用
}

type Company struct {
	ID   int
	Code string `gorm:"index"`
	Name string
}
```

### CRUD

>Select：选择要添加值的字段
>
>Omit：跳过指定字段，向未指定字段值添加值
>
>tx.RowsAffected：影响行数

```go
func TestInsert(t *testing.T) {
	tx := db.Create(&User{
		Name: "user1",
		Company: Company{
			Name: "company1",
		},
	})
	log.Println(tx.RowsAffected)

	db.Select("name").Create(&User{Name: "abc", Company: Company{Name: "abcCompany"}})
	db.Omit("name").Create(&User{Name: "abc", Company: Company{Name: "abcCompany"}})
}

func TestFind(t *testing.T) {
	user := User{}
	db.Preload("Company").First(&user)
	// {ID:1 Name:user1 CompanyID:1 Company:{ID:1 Name:company1}}
	log.Printf("%+v", user)

	users := []User{}
	db.Preload("Company").Find(&users)
	// [{ID:1 Name:user1 CompanyID:1 Company:{ID:1 Name:company1}} {ID:2 Name:user2 CompanyID:2 Company:{ID:2 Name:company2}}]
	log.Printf("%+v", users)

	userw := &User{}
	db.Preload("Company").Where("name=?", "user2").Find(&userw)
	// {ID:2 Name:user2 CompanyID:2 Company:{ID:2 Name:company2}}
	log.Printf("%+v", userw)
}

func TestUpdate(t *testing.T) {
	//修改
	tx := db.Model(&User{ID: 1}).Update("name", "asd")
	log.Println(tx.RowsAffected)
	db.Model(&User{}).Where("id", 1).Update("name", "asd")
  	//更新关联的数据，原本company_id为1，执行后为新创建的comoany的id，新创建的company数据为：Company: Company{Name: "llll"}
	db.Session(&gorm.Session{FullSaveAssociations: true}).Updates(&User{ID: 7, Company: Company{Name: "llll"}})

	//删除关联关系	原本user的id为1的company_id为1删除后为null
	db.Model(&User{ID: 1}).Association("Company").Delete(&Company{ID: 1})
	//清空关联关系	原本user的id为2的company_id为2清空后为null
	db.Model(&User{ID: 2}).Association("Company").Clear()
	//添加关联关系	原本user的id为1的company_id为null添加后为2
	db.Model(&User{ID: 1}).Association("Company").Append(&Company{ID: 2})
	//替换关联关系	原本user的id为1的company_id为2替换后为1
	db.Model(&User{ID: 1}).Association("Company").Replace(&Company{ID: 1})
}

func TestDel(t *testing.T) {
  // 效果一样
	db.Delete(&User{ID: 2})
	db.Where("id", 2).Delete(&User{})
}

```

## Has One：拥有

>`has one` 与另一个模型建立一对一的关联，但它和一对一关系有些许不同。 这种关联表明一个模型的每个实例都包含或拥有另一个模型的一个实例。

例如：用户拥有一张信用卡

```go
// User 有一张 CreditCard，UserID 是外键
type User struct {
	gorm.Model
	CreditCard CreditCard
}

type CreditCard struct {
	gorm.Model
	Number string
	UserID uint
}
```



### 重写外健

```go
type User struct {
	gorm.Model
	CreditCard CreditCard `gorm:"foreignKey:SuiBian"`
}

type CreditCard struct {
	gorm.Model
	Number  string
	SuiBian uint
}
```



### 重写引用

```go
type User struct {
	gorm.Model
	Abc        int        `gorm:"index"`
	CreditCard CreditCard `gorm:"foreignkey:UserAbc;references:Abc"`
}

type CreditCard struct {
	gorm.Model
	Number  string
	UserAbc int
}
```

>references：重写引用，默认引用的是ID，使用references可以更改引用；has one 的被引用的字段必须要有 `gorm:"index"`不然会报错

## Has Many：一对多

>`has many` 与另一个模型建立了一对多的连接。 不同于 `has one`，拥有者可以有零或多个关联模型。
>
>例如，您的应用包含 user 和 credit card 模型，且每个 user 可以有多张 credit card。

```go
// User 有多张 CreditCard，UserID 是外键
type User struct {
  gorm.Model
  CreditCards []CreditCard
}

type CreditCard struct {
  gorm.Model
  Number string
  UserID uint
}
```



### 重写外键

```go
type User struct {
  gorm.Model
  CreditCards []CreditCard `gorm:"foreignKey:UserRefer"`
}

type CreditCard struct {
  gorm.Model
  Number    string
  UserRefer uint
}
```



### 重写引用

>外键和被引用外键都必须标注`gorm:"index"`

```go
type User struct {
	gorm.Model
	MemberNumber string       `gorm:"index"`
	CreditCards  []CreditCard `gorm:"foreignKey:UserNumber;references:MemberNumber"`
}

type CreditCard struct {
	gorm.Model
	Number     string
	UserNumber string `gorm:"index"`
}
```

### 多态关联

>GORM 为 `has one` 和 `has many` 提供了多态关联支持，它会将拥有者实体的表名、主键都保存到多态类型的字段中。

```go
type Dog struct {
  ID   int
  Name string
  Toys []Toy `gorm:"polymorphic:Owner;"`
}

type Toy struct {
  ID        int
  Name      string
  OwnerID   int
  OwnerType string
}

db.Create(&Dog{Name: "dog1", Toys: []Toy{{Name: "toy1"}, {Name: "toy2"}}})
// INSERT INTO `dogs` (`name`) VALUES ("dog1")
// INSERT INTO `toys` (`name`,`owner_id`,`owner_type`) VALUES ("toy1","1","dogs"), ("toy2","1","dogs")

```



> 可以使用标签 `polymorphicValue` 来更改多态类型的值，例如：

```go
type Dog struct {
  ID   int
  Name string
  Toys []Toy `gorm:"polymorphic:Owner;polymorphicValue:master"`
}

type Toy struct {
  ID        int
  Name      string
  OwnerID   int
  OwnerType string
}

db.Create(&Dog{Name: "dog1", Toy: []Toy{{Name: "toy1"}, {Name: "toy2"}}})
// INSERT INTO `dogs` (`name`) VALUES ("dog1")
// INSERT INTO `toys` (`name`,`owner_id`,`owner_type`) VALUES ("toy1","1","master"), ("toy2","1","master")
```





### 自引用 Has Many

```go
type User struct {
  gorm.Model
  Name      string
  ManagerID *uint
  Team      []User `gorm:"foreignkey:ManagerID"`
}
```





### 外键约束

> 可以通过为标签 `constraint` 配置 `OnUpdate`、`OnDelete` 实现外键约束，在使用 GORM 进行迁移时它会被创建，例如

```go
type User struct {
  gorm.Model
  CreditCards []CreditCard `gorm:"constraint:OnUpdate:CASCADE,OnDelete:SET NULL;"`
}

type CreditCard struct {
  gorm.Model
  Number string
  UserID uint
}
```





## Many To Many：多对多

> Many to Many 会在两个 model 中添加一张连接表。
>
> 例如，您的应用包含了 user 和 language，且一个 user 可以说多种 language，多个 user 也可以说一种 language。
>
> 当使用 GORM 的 `AutoMigrate` 为 `User` 创建表时，GORM 会自动创建连接表

```go
// User 拥有并属于多种 language，`user_languages` 是连接表
type User struct {
  gorm.Model
  Languages []Language `gorm:"many2many:user_languages;"`
}

type Language struct {
  gorm.Model
  Name string
}
```



### 反向引用

```go
// User 拥有并属于多种 language，`user_languages` 是连接表
type User struct {
  gorm.Model
  Languages []*Language `gorm:"many2many:user_languages;"`
}

type Language struct {
  gorm.Model
  Name string
  Users []*User `gorm:"many2many:user_languages;"`
}
```





分别查询两张表

```go
// Retrieve user list with edger loading languages
func GetAllUsers(db *gorm.DB) ([]User, error) {
    var users []User
    err := db.Model(&User{}).Preload("Languages").Find(&users).Error
    return users, err
}

// Retrieve language list with edger loading users
func GetAllLanguages(db *gorm.DB) ([]Language, error) {
    var languages []Language
    err := db.Model(&Language{}).Preload("Users").Find(&languages).Error
    return languages, err
}
```



### 重写外键

> 对于 `many2many` 关系，连接表会同时拥有两个模型的外键，例如：

```go
type User struct {
  gorm.Model
  Languages []Language `gorm:"many2many:user_languages;"`
}

type Language struct {
  gorm.Model
  Name string
}

// Join Table: user_languages
//   foreign key: user_id, reference: users.id
//   foreign key: language_id, reference: languages.id

```

>若要重写它们，可以使用标签 `foreignKey`、`references`、`joinforeignKey`、`joinReferences`。当然，不需要使用全部的标签，你可以仅使用其中的一个重写部分的外键、引用。

```go
type User struct {
    gorm.Model
    Profiles []Profile `gorm:"many2many:user_profiles;foreignKey:Refer;joinForeignKey:UserReferID;References:UserRefer;joinReferences:ProfileRefer"`
    Refer    uint      `gorm:"index:,unique"`
}

type Profile struct {
    gorm.Model
    Name      string
    UserRefer uint `gorm:"index:,unique"`
}

// Which creates join table: user_profiles
//   foreign key: user_refer_id, reference: users.refer
//   foreign key: profile_refer, reference: profiles.user_refer
```



### 自引用 Many2Many

```go
type User struct {
  gorm.Model
    Friends []*User `gorm:"many2many:user_friends"`
}

// Which creates join table: user_friends
//   foreign key: user_id, reference: users.id
//   foreign key: friend_id, reference: users.id
```



### 自定义连接表

>`连接表` 可以是一个全功能的模型，支持 `Soft Delete`、`钩子`、更多的字段，就跟其它模型一样。您可以通过 `SetupJoinTable` 指定它，例如：
>
>**注意：** 自定义连接表要求外键是复合主键或复合唯一索引

```go
type Person struct {
  ID        int
  Name      string
  Addresses []Address `gorm:"many2many:person_addresses;"`
}

type Address struct {
  ID   uint
  Name string
}

type PersonAddress struct {
  PersonID  int `gorm:"primaryKey"`
  AddressID int `gorm:"primaryKey"`
  CreatedAt time.Time
  DeletedAt gorm.DeletedAt
}

func (PersonAddress) BeforeCreate(db *gorm.DB) error {
  // ...
}

// Change model Person's field Addresses' join table to PersonAddress
// PersonAddress must defined all required foreign keys or it will raise error
err := db.SetupJoinTable(&Person{}, "Addresses", &PersonAddress{})
```



### 外键约束

>你可以通过为标签 `constraint` 配置 `OnUpdate`、`OnDelete` 实现外键约束，在使用 GORM 进行迁移时它会被创建，例如：
>
>也可以在删除记录时通过 `Select` 来删除 many2many 关系的记录

```go
type User struct {
  gorm.Model
  Languages []Language `gorm:"many2many:user_speaks;"`
}

type Language struct {
  Code string `gorm:"primarykey"`
  Name string
}

// CREATE TABLE `user_speaks` (`user_id` integer,`language_code` text,PRIMARY KEY (`user_id`,`language_code`),CONSTRAINT `fk_user_speaks_user` FOREIGN KEY (`user_id`) REFERENCES `users`(`id`) ON DELETE SET NULL ON UPDATE CASCADE,CONSTRAINT `fk_user_speaks_language` FOREIGN KEY (`language_code`) REFERENCES `languages`(`code`) ON DELETE SET NULL ON UPDATE CASCADE);
```



### 复合外键

>如果您的模型使用了 [复合主键](https://gorm.io/zh_CN/docs/composite_primary_key.html)，GORM 会默认启用复合外键。
>
>您也可以覆盖默认的外键、指定多个外键，只需用逗号分隔那些键名，例如：

```go
type Tag struct {
  ID     uint   `gorm:"primaryKey"`
  Locale string `gorm:"primaryKey"`
  Value  string
}

type Blog struct {
  ID         uint   `gorm:"primaryKey"`
  Locale     string `gorm:"primaryKey"`
  Subject    string
  Body       string
  Tags       []Tag `gorm:"many2many:blog_tags;"`
  LocaleTags []Tag `gorm:"many2many:locale_blog_tags;ForeignKey:id,locale;References:id"`
  SharedTags []Tag `gorm:"many2many:shared_blog_tags;ForeignKey:id;References:id"`
}
```









