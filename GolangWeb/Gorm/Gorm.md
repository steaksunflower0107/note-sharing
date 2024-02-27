# GORM框架

ORM：Object Relational Mapping 对象关系映射

go语言支持的Gorm、Xorm

## 连接MySQL

### 连接数据库

要注意的是，gorm不能创建数据库，需要先手动创建。

导入的包：

```go
import (
   "fmt"
   _ "github.com/go-sql-driver/mysql" // "_"表示代码不直接使用包，但是底层链接要使用
    /*
    具体过程是："_"导入会跨包调用init函数，会在main函数执行之前使用，其和main是函数是唯二的小写也能被跨包调用的函数
    在这里的作用是，是注册了MySQL驱动
    */
   "github.com/jinzhu/gorm"
)
```

#### 1.定义全局结构体(表)

```go
type Student struct {
   Id   int // 默认主键，主键的引入是为了提升查询速度
   Name string
   Age  int
}
```

#### 2.连接数据库

```go
GlobalConn, err := gorm.Open("mysql", "root:sf20020107@tcp(127.0.0.1:3306)/ihome?charset=utf8mb4&parseTime=True&loc=Local")
if err != nil {
   fmt.Println("数据库链接失败", err)
   return
}
defer conn.Close()
```

连接数据库 -- 格式：用户名：密码@协议(IP:port)/数据库名?字符

#### 3.借助gorm创建表

```go
GlobalConn.AutoMigrate(new(Student)).Error
```

AutoMigrate返回的是一个*DB类型，Error是其字段。

需要注意的是，该方法默认创建的表是复数类型，即会给表自动添加"s"，若想避免这一点，可以使用`GlobalConn.SingularTable(true)`创建非复数表名的表。

### 连接池

#### 1.创建全局连接池句柄

```go
var GlobalConn *gorm.DB
```

#### 2.设置初始数与最大连接数

```go
GlobalConn.DB().SetMaxIdleConns(10)
GlobalConn.DB().SetMaxOpenConns(100)
```

#### 3.使用连接池句柄操作数据库

### MySQL 8小时时区问题

MySQL默认使用的时间的是：美国东8区时间，和北京时间是差了8小时的。

因此，在连接数据库时，要加入`&parseTime=True&loc=Local`。

由此，MySQL中与时间相关的字段就会使用北京时间啦。



## Gorm操作数据库

### Gorm创建数据

使用`create`函数。

要注意的是必须要传入一个地址值，即要加`&`，否则会报错。

```go
func InsertData() {
   // 先创建数据 ---> 创建对象
   var stu Student
   stu.Name = "jack"
   stu.Age = 100

   // 插入数据
   fmt.Println(GlobalConn.Create(&stu).Error)
}
```

#### 创建记录

```go
user := User{Name: "Jinzhu", Age: 18, Birthday: time.Now()}

result := db.Create(&user) // 通过数据的指针来创建

user.ID             // 返回插入数据的主键
result.Error        // 返回 error
result.RowsAffected // 返回插入记录的条数
```

#### 批量插入

要有效地插入大量记录，就将一个 slice 传递给 Create 方法。 将切片数据传递给 Create 方法，GORM 将生成一个单一的 SQL 语句来插入所有数据，并回填主键的值，钩子方法也会被调用。

如：

```go
var users = []User{{Name: "jinzhu1"}, {Name: "jinzhu2"}, {Name: "jinzhu3"}}
db.Create(&users)

for _, user := range users {
  user.ID // 1,2,3
}
```

使用`CreateInBatches`，指定创建的数量：

```go
var 用户 = []User{name: "jinzhu_1"}, ...., {Name: "jinzhu_10000"}}

// 数量为 100
db.CreateInBatches(用户, 100)

```

#### 创建钩子

GORM 允许用户定义的钩子有 `BeforeSave`, `BeforeCreate`, `AfterSave`, `AfterCreate` 创建记录时将调用这些钩子方法。

比如一个钩子函数：

```go
func (u *User) BeforeCreate(tx *gorm.DB) (err error) {
  u.UUID = uuid.New()

    if u.Role == "admin" {
        return errors.New("invalid role")
    }
    return
}
```

#### 跳过钩子

若想跳过钩子函数的执行，可以使用 `SkipHooks` 会话模式：

```go
DB.Session(&gorm.Session{SkipHooks: true}).Create(&user)

DB.Session(&gorm.Session{SkipHooks: true}).Create(&users)

DB.Session(&gorm.Session{SkipHooks: true}).CreateInBatches(users, 100)
```

#### 根据Map创建

GORM 支持根据 `map[string]interface{}` 和 `[]map[string]interface{}{}` 创建记录，例如：

```go
db.Model(&User{}).Create(map[string]interface{}{
  "Name": "jinzhu", "Age": 18,
})

// batch insert from `[]map[string]interface{}{}`
db.Model(&User{}).Create([]map[string]interface{}{
  {"Name": "jinzhu_1", "Age": 18},
  {"Name": "jinzhu_2", "Age": 20},
})
```



### Gorm查询数据

#### 检索单个对象

有3种方法：

- `First(&Student)`：获取Student表中的第一条数据

  相当于`SELECT * FROM users ORDER BY id LIMIT 1;`

  - 最简单版本：

  - ```go
    func SearchData() {
       var stu Student
       GlobalConn.First(&stu)
       fmt.Println(stu)
    }
    ```

- `Last(&Student)`：获取Student表中的最后一条数据

​       相当于`SELECT * FROM users ORDER BY id DESC LIMIT 1;`

- `Take(&Student)`：获取Student表中的一条数据

​		相当于`SELECT * FROM users LIMIT 1;`

#### 根据主键检索

1.`db.First(&user, 10)`
相当于`SELECT * FROM users WHERE id = 10;`

2.`db.First(&user, "10")`
相当于 `SELECT * FROM users WHERE id = 10;`

3.`db.Find(&users, []int{1,2,3})`
相当于`SELECT * FROM users WHERE id IN (1,2,3);`

#### 检索全部对象

1.获取全部记录：
`result := db.Find(&users)`
相当于`SELECT * FROM users;`

2.返回找到的记录数，相当于 `len(users)`

```go
result := db.First(&user)
result.RowsAffected 
result.Error        // returns error
```

#### 条件查询

##### String条件

1.获取第一条匹配的记录
`db.Where("name = ?", "jinzhu").First(&user)`
相当于`SELECT * FROM users WHERE name = 'jinzhu' ORDER BY id LIMIT 1;`

2.获取全部匹配的记录
`db.Where("name <> ?", "jinzhu").Find(&users)`
相当于`SELECT * FROM users WHERE name <> 'jinzhu';`

3.IN关键字
`db.Where("name IN ?", []string{"jinzhu", "jinzhu 2"}).Find(&users)`
相当于`SELECT * FROM users WHERE name IN ('jinzhu','jinzhu 2');`

3.LIKE关键字
`db.Where("name LIKE ?", "%jin%").Find(&users)`
相当于`SELECT * FROM users WHERE name LIKE '%jin%';`

4.AND关键字
`db.Where("name = ? AND age >= ?", "jinzhu", "22").Find(&users)`
相当于`SELECT * FROM users WHERE name = 'jinzhu' AND age >= 22;`

5.Time相关
`db.Where("updated_at > ?", lastWeek).Find(&users)`
相当于`SELECT * FROM users WHERE updated_at > '2000-01-01 00:00:00';`

6.BETWEEN关键字
`db.Where("created_at BETWEEN ? AND ?", lastWeek, today).Find(&users)`
相当于`SELECT * FROM users WHERE created_at BETWEEN '2000-01-01 00:00:00' AND '2000-01-08 00:00:00';`

##### Struct & Map条件

Struct：

`db.Where(&User{Name: "jinzhu", Age: 20}).First(&user)`
相当于`SELECT * FROM users WHERE name = "jinzhu" AND age = 20 ORDER BY id LIMIT 1;`

Map：

`db.Where(map[string]interface{}{"name: "jinzhu", Age: 20}).Find(&users)`

相当于`SELECT * FROM users WHERE name = "jinzhu" AND age = 20;`

主键切片条件(In)：

`db.Where([]int64{20, 21, 22}).Find(&users)`

相当于SELECT * FROM users WHERE id IN (20, 21, 22);

##### 注意点

当使用结构作为条件查询时，**GORM 只会查询非零值字段**。这意味着如果您的字段值为 `0`、`''`、`false` 或其他零值，该字段不会被用于构建查询条件，例如：

`db.Where(&User{Name: "jinzhu", Age: 0}).Find(&users)`
相当于`SELECT * FROM users WHERE name = "jinzhu";`

解决的方法是，可以通过构建Map来构建查询条件：

`db.Where(map[string]interface{}{"Name": "jinzhu", "Age": 0}).Find(&users)`

相当于` SELECT * FROM users WHERE name = "jinzhu" AND age = 0;`

#### 内联条件

**用法与`Where`类似。**

1.根据主键获取记录，如果是非整型主键

`db.First(&user, "id = ?", "string_primary_key")`
相当于`SELECT * FROM users WHERE id = 'string_primary_key';`

2.Plain SQL
`db.Find(&user, "name = ?", "jinzhu")`
相当于`SELECT * FROM users WHERE name = "jinzhu";`

`db.Find(&users, "name <> ? AND age > ?", "jinzhu", 20)`
相当于SELECT * FROM users WHERE name <> "jinzhu" AND age > 20;

3.Struct
`db.Find(&users, User{Age: 20})`
相当于`SELECT * FROM users WHERE age = 20;`

4.Map
`db.Find(&users, map[string]interface{}{"age": 20})`
相当于`SELECT * FROM users WHERE age = 20;`

#### Not条件

**在db后调用Not函数，类似直接取反**。

1.`db.Not("name = ?", "jinzhu").First(&user)`
相当于`SELECT * FROM users WHERE NOT name = "jinzhu" ORDER BY id LIMIT 1;`

2.Not In
`db.Not(map[string]interface{}{"name": []string{"jinzhu", "jinzhu 2"}}).Find(&users)`
相当于`SELECT * FROM users WHERE name NOT IN ("jinzhu", "jinzhu 2");`

3.Struct
`db.Not(User{Name: "jinzhu", Age: 18}).First(&user)`
相当于`SELECT * FROM users WHERE name <> "jinzhu" AND age <> 18 ORDER BY id LIMIT 1;`

4.不在主键切片中的记录
`db.Not([]int64{1,2,3}).First(&user)`
相当于`SELECT * FROM users WHERE id NOT IN (1,2,3) ORDER BY id LIMIT 1;`

#### Or条件

**用Or函数连接句子**。

1.`db.Where("role = ?", "admin").Or("role = ?", "super_admin").Find(&users)`
相当于`SELECT * FROM users WHERE role = 'admin' OR role = 'super_admin';`

2.Struct
`db.Where("name = 'jinzhu'").Or(User{Name: "jinzhu 2", Age: 18}).Find(&users)`
相当于`SELECT * FROM users WHERE name = 'jinzhu' OR (name = 'jinzhu 2' AND age = 18);`

3.Map
`db.Where("name = 'jinzhu'").Or(map[string]interface{}{"name": "jinzhu 2", "age": 18}).Find(&users)`
相当于`SELECT * FROM users WHERE name = 'jinzhu' OR (name = 'jinzhu 2' AND age = 18);`

#### 选择特定字段

**使用Select函数来选择字段，字段间用`,`分割开来。**

1.`db.Select("name", "age").Find(&users)`
相当于`SELECT name, age FROM users;`

2.`db.Select([]string{"name", "age"}).Find(&users)`
相当于`SELECT name, age FROM users;`

3.`db.Table("users").Select("COALESCE(age,?)", 42).Rows()`
相当于`SELECT COALESCE(age,'42') FROM users;`

#### Order

**指定从数据库检索记录的排序方式。**

**使用Order函数，字段名 排序方式、字段间用`,`隔开。**

1.`db.Order("age desc, name").Find(&users)`
相当于`SELECT * FROM users ORDER BY age desc, name;`

2.多个 order
`db.Order("age desc").Order("name").Find(&users)`
`SELECT * FROM users ORDER BY age desc, name;`

3.`db.Clauses(clause.OrderBy{
  Expression: clause.Expr{SQL: "FIELD(id,?)", Vars: []interface{}{[]int{1, 2, 3}}, WithoutParentheses: true},
}).Find(&User{})`
相当于`SELECT * FROM users ORDER BY FIELD(id,1,2,3)`

#### Limit & Offset

**`Limit`函数用于设定获取记录的最大数量，`Offset`函数用于设定在开始返回记录之前要跳过的记录数量。**

1.`db.Limit(3).Find(&users)`
相当于`SELECT * FROM users LIMIT 3;`

2.通过 -1 消除 Limit 条件
`db.Limit(10).Find(&users1).Limit(-1).Find(&users2)`
相当于`SELECT * FROM users LIMIT 10; (users1)`

3.`db.Offset(3).Find(&users)`
相当于`SELECT * FROM users OFFSET 3;`

4.`db.Limit(10).Offset(5).Find(&users)`
相当于`SELECT * FROM users OFFSET 5 LIMIT 10;`

5.通过 -1 消除 Offset 条件
`db.Offset(10).Find(&users1).Offset(-1).Find(&users2)`
相当于`SELECT * FROM users OFFSET 10; (users1)`

#### Group & Having

定义结构体：

```go
type result struct {
  Date  time.Time
  Total int
}
```

1.

```go
db.Model(&User{}).Select("name, sum(age) as total").Where("name LIKE ?", "group%").Group("name").First(&result)

```

相当于`SELECT name, sum(age) as total FROM `users` WHERE name LIKE "group%" GROUP BY name`

2.

```go
db.Model(&User{}).Select("name, sum(age) as total").Group("name").Having("name = ?", "group").Find(&result)
```


相当于`SELECT name, sum(age) as total FROM users GROUP BY name HAVING name = "group"`

#### Distinct

**用于在模型中选择不相同的值。**

`db.Distinct("name", "age").Order("name, age desc").Find(&results)`

#### Joins

直接在Joins函数中写原生SQL语句。

```go
db.Model(&User{}).Select("users.name, emails.email").Joins("left join emails on emails.user_id = users.id").Scan(&result{})

```

相当于`SELECT users.name, emails.email FROM `users` left join emails on emails.user_id = users.id`

带参数的多表连接：

```go
db.Joins("JOIN emails ON emails.user_id = users.id AND emails.email = ?", "jinzhu@example.org").Joins("JOIN credit_cards ON credit_cards.user_id = users.id").Where("credit_cards.number = ?", "411111111111").Find(&user)
```

#### Scan

**扫描结果写入到struct，用法与`Find`类似。**

```go
type Result struct {
  Name string
  Age  int
}
```

```go
var result Result
db.Table("users").Select("name", "age").Where("name = ?", "Antonio").Scan(&result)
```

**写原生 SQL语句**：
`db.Raw("SELECT name, age FROM users WHERE name = ?", "Antonio").Scan(&result)`

### Gorm更改数据

#### 保存所有字段

**使用`Save`函数， 会保存所有的字段，即使字段是零值**。

```go
db.First(&user)

user.Name = "jinzhu 2"
user.Age = 100
db.Save(&user)
```

相当于`UPDATE users SET name='jinzhu 2', age=100, birthday='2016-01-01', updated_at = '2013-11-17 21:34:10' WHERE id=111;`

#### 更新单个列

**使用`Update`函数。**

当使用 `Update` 更新单个列时，你需要指定条件，否则会返回`ErrMissingWhereClause` 错误，查看 `Block Global Updates` 获取详情。当使用了 Model 方法，且该对象主键有值，该值会被用于构建条件，例如：

1.条件更新

```go
db.Model(&User{}).Where("active = ?", true).Update("name", "hello")
```

相当于`UPDATE users SET name='hello', updated_at='2013-11-17 21:34:10' WHERE active=true;`

2.User 的 ID 是 `111`

```go
db.Model(&user).Update("name", "hello")
```

相当于`UPDATE users SET name='hello', updated_at='2013-11-17 21:34:10' WHERE id=111;`

3.根据条件和 model 的值进行更新

```go
db.Model(&user).Where("active = ?", true).Update("name", "hello")
```

相当于`UPDATE users SET name='hello', updated_at='2013-11-17 21:34:10' WHERE id=111 AND active=true;`

#### 更新多列

**`Update`函数与Map和struct配合使用。**

`Updates` 方法支持 `struct` 和 `map[string]interface{}` 参数。当使用 `struct` 更新时，默认情况下，GORM 只会更新非零值的字段。

1.根据 `struct` 更新属性，只会更新非零值的字段

```go
db.Model(&user).Updates(User{Name: "hello", Age: 18, Active: false})
```

相当于`UPDATE users SET name='hello', age=18, updated_at = '2013-11-17 21:34:10' WHERE id = 111;`

2.根据 `map` 更新属性

```go
db.Model(&user).Updates(map[string]interface{}{"name": "hello", "age": 18, "actived": false})
```

相当于`UPDATE users SET name='hello', age=18, actived=false, updated_at='2013-11-17 21:34:10' WHERE id=111;`

#### 更新选定字段

**选定和不选定分别使用`Select`和`Omit`函数。**

如果想要在更新时选定、忽略某些字段，可以使用 `Select`、`Omit`。

1.Select

```go
db.Model(&user).Select("name").Updates(map[string]interface{}{"name": "hello", "age": 18, "actived": false})
```

相当于`UPDATE users SET name='hello' WHERE id=111;`

2.Omit

```go
db.Model(&user).Omit("name").Updates(map[string]interface{}{"name": "hello", "age": 18, "actived": false})
```

相当于`UPDATE users SET age=18, actived=false, updated_at='2013-11-17 21:34:10' WHERE id=111;`

3.struct

```go
db.Model(&result).Select("Name", "Age").Updates(User{Name: "new_name", Age: 0})
```

相当于`UPDATE users SET name='new_name', age=0 WHERE id=111;`

#### 更新Hook

##### Hook

Hook是在**创建、查询、更新、删除等操作之前、之后**调用的函数。

如果已经为模型定义了指定的方法，Hook会在创建、更新、查询、删除时自动被调用。如果在任何回调后返回了错误，GORM 将停止后续的操作并回滚事务。

钩子方法的函数签名应该是 `func(*gorm.DB) error`。

命名中有Before、After等来标识。

示例：

```go
func (u *User) BeforeCreate(tx *gorm.DB) (err error) {
  u.UUID = uuid.New()

  if !u.IsValid() {
    err = errors.New("can't save invalid data")
  }
  return
}

func (u *User) AfterCreate(tx *gorm.DB) (err error) {
  if u.ID == 1 {
    tx.Model(u).Update("role", "admin")
  }
  return
}

func (u *User) AfterCreate(tx *gorm.DB) (err error) {
  if !u.IsValid() {
    return errors.New("rollback invalid user")
  }
  return nil
}
```

##### 更新操作中Hook的使用

对于更新操作，GORM 支持 `BeforeSave`、`BeforeUpdate`、`AfterSave`、`AfterUpdate` 钩子。

```go
func (u *User) BeforeUpdate(tx *gorm.DB) (err error) {
    if u.Role == "admin" {
        return errors.New("admin user not allowed to update")
    }
    return
}
```

#### 批量更新

若未通过`Model`指定记录的主键，则GORM会执行批量更新。

1.根据struct更新

```go
db.Model(User{}).Where("role = ?", "admin").Updates(User{Name: "hello", Age: 18})
```

相当于：

```sql
UPDATE users SET name='hello', age=18 WHERE role = 'admin;
```

2.根据map更新

```go
db.Table("users").Where("id IN ?", []int{10, 
```

相当于：

```sql
UPDATE users SET name='hello', age=18 WHERE id IN (10, 11);
```

#### 阻止全局更新

如果在没有任何条件的情况下执行批量更新，默认情况下，GORM 不会执行该操作，并返回 `ErrMissingWhereClause` 错误。条件指的是`WHERE`等关键字。

对此，可以加一些条件，或者使用原生 SQL，或者启用 `AllowGlobalUpdate` 模式，例如：

```go
db.Model(&User{}).Update("name", "jinzhu").Error // 会报错误：gorm.ErrMissingWhereClause，因为没有指定条件
```

加上条件：

```go
db.Model(&User{}).Where("1 = 1").Update("name", "jinzhu")
```

则相当于：

```sql
UPDATE users SET `name` = "jinzhu" WHERE 1=1
```

原生sql：

```go
db.Exec("UPDATE users SET name = ?", "jinzhu")
```

相当于：

```sql
UPDATE users SET name = "jinzhu"
```

启用 `AllowGlobalUpdate` 模式：

```go
db.Session(&gorm.Session{AllowGlobalUpdate: true}).Model(&User{}).Update("name", "jinzhu")
```

相当于：

```sql
UPDATE users SET `name` = "jinzhu"
```

#### 更新的记录数

通过 `RowsAffected` 得到更新的记录数。

如：

```go
result := db.Model(User{}).Where("role = ?", "admin").Updates(User{Name: "hello", Age: 18})

result.RowsAffected		// 获取更新的记录数
```

### Gorm删除数据

#### 删除一条记录

主要使用的函数就是`Delete`啦。

1.删除Email的ID为10的字段

```go
db.Delete(&email)
```

相当于：

```sql
DELETE from emails where id = 10;
```

2.条件删除

```go
db.Where("name = ?", "jinzhu").Delete(&email)
```

相当于：

```sql
DELETE from emails where id = 10 AND name = "jinzhu";
```

#### 根据主键删除

GORM 允许通过内联条件指定主键来检索对象，但只支持整型数值，因为 string 可能导致 SQL 注入。

```go
db.Delete(&User{}, 10)
```

相当于：

```sql
DELETE FROM users WHERE id = 10;
```

字符串指定，这种情况就可能导致sql注入：

```go
db.Delete(&User{}, "10")
```

再举一个正面例子：

```go
db.Delete(&users, []int{1,2,3})
```

相当于：

```sql
DELETE FROM users WHERE id IN (1,2,3);
```

#### Delete Hook

对于删除操作，GORM 支持 `BeforeDelete`、`AfterDelete` Hook，在删除记录时会调用这些方法。

如：

```go
func (u *User) BeforeDelete(tx *gorm.DB) (err error) {
    if u.Role == "admin" {
        return errors.New("admin user not allowed to delete")
    }
    return
}
```

#### 批量删除

如果指定的值不包括主属性，那么 GORM 会执行批量删除，它将删除所有匹配的记录。

如：

```go
db.Where("email LIKE ?", "%jinzhu%").Delete(Email{})
db.Delete(&Email{}, "email LIKE ?", "%jinzhu%")
```

都相当于：

```sql
DELETE from emails where email LIKE "%jinzhu%";
```

#### 阻止全局删除

更前面讲到的阻止全局更新时类似的。

如果在没有任何条件的情况下执行批量删除，GORM 不会执行该操作，并返回 ErrMissingWhereClause 错误。

对此，你必须加一些条件，或者使用原生 SQL，或者启用 AllowGlobalUpdate 模式，例如：

```go
db.Delete(&User{}).Error // gorm.ErrMissingWhereClause

db.Where("1 = 1").Delete(&User{})
// DELETE FROM `users` WHERE 1=1

db.Exec("DELETE FROM users")
// DELETE FROM users

db.Session(&gorm.Session{AllowGlobalUpdate: true}).Delete(&User{})
// DELETE FROM users
```

#### 软删除

**如果模型包含了一个 `gorm.deletedat` 字段（`gorm.Model` 已经包含了该字段)，它将自动获得软删除的能力！**

**拥有软删除能力的模型调用 `Delete` 时，记录不会被从数据库中真正删除**。但 GORM 会将 `DeletedAt` 置为当前时间作为删除标记， 并且你不能再通过正常的查询方法(比如`Find`、`First`)找到该记录。



如果不想引入`gorm.Model`，可以加入`Deleted gorm.DeletedAt`字段。

```go
type User struct {
  ID      int
  Deleted gorm.DeletedAt
  Name    string
}
```

#### 查找被软删除的记录

方法是使用`Unscoped`，在该函数后加上正常的查询语句。

比如：

```go
db.Unscoped().Where("age = 20").Find(&users)
```

相当于：

```sql
SELECT * FROM users WHERE age = 20;
```

#### 永久删除

加上`Unscoped`函数后，还可以实现永久删除！

比如：

```go
db.Unscoped().Delete(&order)
```

相当于：

```sql
DELETE FROM orders WHERE id=10;
```

#### 删除标记

1.使用 `unix` 时间戳作为删除标记

```go
import "gorm.io/plugin/soft_delete"

type User struct {
  ID        uint
  Name      string
  DeletedAt soft_delete.DeletedAt
}

// Query
SELECT * FROM users WHERE deleted_at = 0;

// Delete
UPDATE users SET deleted_at = /* current unix second */ WHERE ID = 1;
```

当使用唯一字段软删除时，你应该创建一个复合索引基于 unix 时间戳的 `DeletedAt` 字段，例如:

```go
import "gorm.io/plugin/soft_delete"

type User struct {
  ID        uint
  Name      string                `gorm:"uniqueIndex:udx_name"`
  DeletedAt soft_delete.DeletedAt `gorm:"uniqueIndex:udx_name"`
}
```

2.使用 1 / 0 作为删除标记

```go
import "gorm.io/plugin/soft_delete"

type User struct {
  ID    uint
  Name  string
  IsDel soft_delete.DeletedAt `gorm:"softDelete:flag"`
}

// Query
SELECT * FROM users WHERE is_del = 0;

// Delete
UPDATE users SET is_del = 1 WHERE ID = 1;
```

## Gorm设置表属性

### 修改表的字段大小

```go
type Student struct {
	Id 	  int
	Name  string	`gorm:"size:50"`	// string 对应的是 varchar，默认大小255，可以在创建表时指定大小
	Age	  int
}
```

### 增加默认值

通过`default`来设定字段的默认值。

```go
type Student struct {
	Id 	  int
	Name  string	`gorm:"size:50;default:`jack`"`	// string 对应的是 varchar，默认大小255，可以在创建表时指定大小
	Age	  int
}
```

### 指定字段非空

通过`not null`来指定

```go
type Student struct {
	Id 	  int
	Name  string	`gorm:"size:50;default:`jack`"`	// string 对应的是 varchar，默认大小255，可以在创建表时指定大小
	Age	  int	`gorm:"not null"`
}
```

### 设定时间属性

MySQL数据库默认有3种时间：

- date
- datetime
- timeStamp: 时间戳     --->     在gorm中，只有timeStamp

当要使用MySQL数据库中的其他类型时，使用`type`关键字去设置



```go
type Student struct {
	Id 	  int
	Name  string	 `gorm:"size:50;default:`jack`"`	// string 对应的是 varchar，默认大小255，可以在创建表时指定大小
	Age	  int		 `gorm:"not null"`
    Join  time.Time  `gorm:"type:timestamp"`	// 设定该字段为时间戳类型
}
```

### 重要结论

当表属性修改后，只会在第一次将表建入到数据库中对应属性才会生效，或者在更改了某个字段的属性后，新加入的字段才会生效而已经在数据库中的数据是不会生效的！




