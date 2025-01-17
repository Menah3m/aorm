[English](https://github.com/tangpanqing/aorm) | [简体中文](https://github.com/tangpanqing/aorm/blob/main/README_zh.md)

# Aorm
Aorm是一个基于go语言的数据库操作库。

给个 ⭐ 吧，如果这个项目帮助到你

## 🌟 特性
- [x] 代码简洁，高性能
- [x] 支持 MySQL,MsSQL,Postgres,Sqlite3 数据库
- [x] 支持 空值查询
- [x] 支持 自动迁移
- [X] 支持 SQL 拼接

## 🌟 如何使用
- [导入](#导入)
  - [Mysql](#mysql)
  - [Sqlite](#sqlite)
  - [MSSQL](#mssql)
  - [Postgres](#postgres)
- [定义数据结构](#定义数据结构)   
- [连接数据库](#连接数据库)   
- [自动迁移](#自动迁移)
- [基本增删改查](#基本增删改查)   
  - [增加一条记录](#增加一条记录)
  - [增加多条记录](#增加多条记录)
  - [获取一条记录](#获取一条记录)
  - [获取多条记录](#获取多条记录)
  - [更新记录](#更新记录)
  - [删除记录](#删除记录)
- [高级查询](#高级查询)
  - [查询指定表](#查询指定表)
  - [查询指定字段](#查询指定字段)
  - [查询条件](#查询条件)
  - [查询条件相关操作](#查询条件相关操作)
  - [联合查询](#联合查询)
  - [分组查询](#分组查询)
  - [筛选](#筛选)
  - [排序](#排序)
  - [分页查询](#分页查询)
  - [悲观锁](#悲观锁)
  - [自增操作](#自增操作)
  - [自减操作](#自减操作)
  - [查询某字段的值](#查询某字段的值)
  - [查询某列的值](#查询某列的值)
- [聚合查询](#聚合查询)
  - [Count](#count)
  - [Sum](#sum)
  - [AVG](#avg)
  - [Min](#min)
  - [Max](#max)
- [通用操作](#通用操作)
  - [Query](#query)
  - [Exec](#exec)
- [事务操作](#事务操作)
- [清空表数据](#清空表数据)
- [工具类](#工具类)

### 导入
```go
    import (
        "database/sql"
        _ "github.com/go-sql-driver/mysql" 
        "github.com/tangpanqing/aorm"
    )
```
`database/sql` go的标准库，提供操作数据库的接口   

`github.com/tangpanqing/aorm` 对于sql操作的包装，使其更易用    

`github.com/go-sql-driver/mysql` mysql数据库的驱动包，如果你使用其他数据库，需要更改这里    

你可以通过下面的命令下载他们    

```shell
go get -u github.com/tangpanqing/aorm
```

#### Mysql
如果你使用 `Mysql` 数据库, 你或许可以使用这个驱动 `github.com/go-sql-driver/mysql`
```shell
go get -u github.com/go-sql-driver/mysql
```

#### Sqlite
如果你使用 `Sqlite` 数据库, 你或许可以使用这个驱动 `github.com/mattn/go-sqlite3`
```shell
go get -u github.com/mattn/go-sqlite3
```

#### Mssql
如果你使用 `Mssql` 数据库, 你或许可以使用这个驱动 `github.com/denisenkom/go-mssqldb`
```shell
go get -u github.com/denisenkom/go-mssqldb
```

#### Postgres
如果你使用 `Postgres` 数据库, 你或许可以使用这个驱动 `github.com/lib/pq`
```shell
go get -u github.com/lib/pq
```

### 定义数据结构
在操作数据库之前，你应该定义数据结构，如下
```go
    type Person struct {
        Id         null.Int    `aorm:"primary;auto_increment" json:"id"`
        Name       null.String `aorm:"size:100;not null;comment:名字" json:"name"`
        Sex        null.Bool   `aorm:"index;comment:性别" json:"sex"`
        Age        null.Int    `aorm:"index;comment:年龄" json:"age"`
        Type       null.Int    `aorm:"index;comment:类型" json:"type"`
        CreateTime null.Time   `aorm:"comment:创建时间" json:"createTime"`
        Money      null.Float  `aorm:"comment:金额" json:"money"`
        Test       null.Float  `aorm:"type:double;comment:测试" json:"test"`
    }
```
首先请注意一些类型,例如 `null.Int`, `null.String`, `null.Bool`, `null.Float`, `null.Time`, 这是对 `sql.NUll*` 结构的一种包装

其次请注意 `aorm:` 标签, 这将被用于数据迁移，将数据表字段与结构体属性相对应, 以下信息你需要知道

| 关键字            | 关键字的值  | 描述              | 例子             |
|----------------|--------|-----------------|----------------|
| primary        | none   | 设置某列为主键索引       | primary        |
| unique         | none   | 设置某列为唯一索引       | unique         |
| index          | none   | 设置某列为普通索引       | index          |
| auto_increment | none   | 设置某列可自增         | auto_increment |
| not null       | none   | 设置某列可空          | not null       |
| type           | string | 设置某列的数据类型       | type:double    |
| size           | int    | 设置某列的数据长度或者显示长度 | size:100       |
| comment        | string | 设置某列的备注信息       | comment:名字     |
| default        | string | 设置某列的默认值        | default:2      |

### 连接数据库
使用 `sql.Open` 方法, 你可以连接数据库, 接下来你应该`ping`数据库，保证可用
```go
    //替换这些数据库参数
    username := "root"
    password := "root"
    hostname := "localhost"
    port := "3306"
    dbname := "database_name"
    
    //连接数据库
    db, err := sql.Open("mysql", username+":"+password+"@tcp("+hostname+":"+port+")/"+dbname+"?charset=utf8mb4&parseTime=True&loc=Local")
    if err != nil {
    panic(err)
    }
    defer db.Close()
    
    //测试连接
    err1 := db.Ping()
    if err1 != nil {
        panic(err1)
    }
```
如果你使用其他数据库，请使用不同的数据库连接，如下
```go
//if content sqlite3 database
sqlite3Content, sqlite3Err := aorm.Open("sqlite3", "test.db")

//if content postgres database
psqlInfo := fmt.Sprintf("host=%s port=%d user=%s password=%s dbname=%s sslmode=disable", "localhost", 5432, "postgres", "root", "postgres")
postgresContent, postgresErr := aorm.Open("postgres", psqlInfo)

//if content mssql database
mssqlInfo := fmt.Sprintf("server=%s;database=%s;user id=%s;password=%s;port=%d;encrypt=disable", "localhost", "database_name", "sa", "root", 1433)
mssqlContent, mssqlErr := aorm.Open("mssql", mssqlInfo)
```

### 自动迁移
使用 `AutoMigrate` 方法, 表名将是结构体名字的下划线形式，如`person`
```go
    aorm.Migrator(db).Driver("mysql").Opinion("ENGINE", "InnoDB").Opinion("COMMENT", "用户表").AutoMigrate(&Person{})
```
使用 `Migrate` 方法, 你可以使用其他的表名
```go
    aorm.Migrator(db).Driver("mysql").Opinion("ENGINE", "InnoDB").Opinion("COMMENT", "用户表").Migrate("person_1", &Person{})
```
使用 `ShowCreateTable` 方法, 你可以获得创建表的sql语句
```go
    showCreate := aorm.Migrator(db).Driver("mysql").ShowCreateTable("person")
    fmt.Println(showCreate)
```
如下
```sql
    CREATE TABLE `person` (
        `id` int NOT NULL AUTO_INCREMENT,
        `name` varchar(100) COLLATE utf8mb4_general_ci NOT NULL COMMENT '名字',
        `sex` tinyint DEFAULT NULL COMMENT '性别',
        `age` int DEFAULT NULL COMMENT '年龄',
        `type` int DEFAULT NULL COMMENT '类型',
        `create_time` datetime DEFAULT NULL COMMENT '创建时间',
        `money` float DEFAULT NULL COMMENT '金额',
        `article_body` text COLLATE utf8mb4_general_ci COMMENT '文章内容',
        `test` double DEFAULT NULL COMMENT '测试',
        PRIMARY KEY (`id`),
        KEY `idx_person_sex` (`sex`),
        KEY `idx_person_age` (`age`),
        KEY `idx_person_type` (`type`)
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci COMMENT='人员表'
```

### 基本增删改查

#### 增加一条记录
使用 `Insert` 方法, 你可以增加一条记录
```go
    id, errInsert := aorm.Use(db).Debug(true).Insert(&Person{
        Name:       null.StringFrom("Alice"),
        Sex:        null.BoolFrom(false),
        Age:        null.IntFrom(18),
        Type:       null.IntFrom(0),
        CreateTime: null.TimeFrom(time.Now()),
        Money:      null.FloatFrom(100.15987654321),
        Test:       null.FloatFrom(200.15987654321987654321),
    })
    if errInsert != nil {
        fmt.Println(errInsert)
    }
    fmt.Println(id)
```
上述代码运行后得到的SQL预处理语句以及相关参数如下
```sql
    INSERT INTO person (name,sex,age,type,create_time,money,test) VALUES (?,?,?,?,?,?,?)
    Alice false 18 0 2022-12-07 10:10:26.1450773 +0800 CST m=+0.031808801 100.15987654321 200.15987654321987
```

如果你使用 `mssql` 数据库 或者 `postgres` 数据库, 请添加 `Driver` 方法, 例如
```go
    id, errInsert := aorm.Use(db).Debug(true).Driver("mssql").Insert(&Person{
	......
    })
```
使用 `Driver` 方法, aorm 将会为sql语句添加一些内容，为了获取最后插入的id

#### 增加多条记录
使用 `InsertBatch` 方法, 你可以增加多条记录
```go
    var batch []Person
    batch = append(batch, Person{
        Name:       null.StringFrom("Alice"),
        Sex:        null.BoolFrom(false),
        Age:        null.IntFrom(18),
        Type:       null.IntFrom(0),
        CreateTime: null.TimeFrom(time.Now()),
        Money:      null.FloatFrom(100.15987654321),
        Test:       null.FloatFrom(200.15987654321987654321),
    })
    
    batch = append(batch, Person{
        Name:       null.StringFrom("Bob"),
        Sex:        null.BoolFrom(true),
        Age:        null.IntFrom(18),
        Type:       null.IntFrom(0),
        CreateTime: null.TimeFrom(time.Now()),
        Money:      null.FloatFrom(100.15987654321),
        Test:       null.FloatFrom(200.15987654321987654321),
    })
    
    count, errInsertBatch := aorm.Use(db).Debug(true).InsertBatch(&batch)
    if errInsertBatch != nil {
        fmt.Println(errInsertBatch)
    }
    fmt.Println(count)
```
上述代码运行后得到的SQL预处理语句以及相关参数如下
```sql
    INSERT INTO person (name,sex,age,type,create_time,money,test) VALUES (?,?,?,?,?,?,?),(?,?,?,?,?,?,?)
    Alice false 18 0 2022-12-16 15:28:49.3907587 +0800 CST m=+0.022987201 100.15987654321 200.15987654321987 Bob true 18 0 2022-12-16 15:28:49.3907587 +0800 CST m=+0.022987201 100.15987654321 200.15987654321987
```

#### 获取一条记录
使用 `GetOne` 方法, 你可以获取一条记录
```go
    var person Person
    errFind := aorm.Use(db).Debug(true).Where(&Person{Id: null.IntFrom(id)}).GetOne(&person)
    if errFind != nil {
        fmt.Println(errFind)
    }
    fmt.Println(person)
```
上述代码运行后得到的SQL预处理语句以及相关参数如下
```sql
    SELECT * FROM person WHERE id = ? Limit ?,?
    1 0 1
```
如果你使用 `mssql` 数据库, 请添加 `Driver` 方法 和 `OrderBy` 方法, 例如
```go
    errFind := aorm.Use(db).Debug(true).Driver("mssql").Where(&Person{Id: null.IntFrom(id)}).OrderBy("id","DESC").GetOne(&person)
```
使用 `Driver` 方法, aorm 将改变sql语句，为了获取到正确的数据

#### 获取多条记录
使用 `GetMany` 方法, 你可以获取多条记录
```go
    var list []Person
    errSelect := aorm.Use(db).Debug(true).Where(&Person{Type: null.IntFrom(0)}).GetMany(&list)
    if errSelect != nil {
        fmt.Println(errSelect)
    }
    for i := 0; i < len(list); i++ {
        fmt.Println(list[i])
    }
```
上述代码运行后得到的SQL预处理语句以及相关参数如下
```sql
    SELECT * FROM person WHERE type = ?
    0
```

#### 更新记录
使用 `Update` 方法, 你可以更新记录
```go
    countUpdate, errUpdate := aorm.Use(db).Debug(true).Where(&Person{Id: null.IntFrom(id)}).Update(&Person{Name: null.StringFrom("Bob")})
    if errUpdate != nil {
        fmt.Println(errUpdate)
    }
    fmt.Println(countUpdate)
```
上述代码运行后得到的SQL预处理语句以及相关参数如下
```sql
    UPDATE person SET name=? WHERE id = ?
    Bob 1
```

#### 删除记录
使用 `Delete` 方法, 你可以删除记录
```go
    countDelete, errDelete := aorm.Use(db).Debug(true).Where(&Person{Id: null.IntFrom(id)}).Delete()
    if errDelete != nil {
        fmt.Println(errDelete)
    }
    fmt.Println(countDelete)
```
上述代码运行后得到的SQL预处理语句以及相关参数如下
```sql
    DELETE FROM person WHERE id = ?
    1
```

### 高级查询
#### 查询指定表
使用 `Table` 方法, 你可以在查询时指定表名
```go
    aorm.Use(db).Debug(true).Table("person_1").Insert(&Person{Name: null.StringFrom("Cherry")})
```
上述代码运行后得到的SQL预处理语句以及相关参数如下
```sql
    INSERT INTO person_1 (name) VALUES (?)
    Cherry
```
#### 查询指定字段
使用 `Select` 方法, 你可以在查询时指定字段
```go
    var listByFiled []Person
    aorm.Use(db).Debug(true).Select("name,age").Where(&Person{Age: null.IntFrom(18)}).GetMany(&listByFiled)
```
上述代码运行后得到的SQL预处理语句以及相关参数如下
```sql
    SELECT name,age FROM person WHERE age = ?
    18
```
#### 查询条件
使用 `WhereArr` 方法, 你可以在查询时添加更多查询条件
```go
    var listByWhere []Person
    
    var where1 []builder.WhereItem
    where1 = append(where1, builder.WhereItem{Field: "type", Opt: builder.Eq, Val: 0})
    where1 = append(where1, builder.WhereItem{Field: "age", Opt: builder.In, Val: []int{18, 20}})
    where1 = append(where1, builder.WhereItem{Field: "money", Opt: builder.Between, Val: []float64{100.1, 200.9}})
    where1 = append(where1, builder.WhereItem{Field: "money", Opt: builder.Eq, Val: 100.15})
    where1 = append(where1, builder.WhereItem{Field: "name", Opt: builder.Like, Val: []string{"%", "li", "%"}})
    
    aorm.Use(db).Debug(true).Table("person").WhereArr(where1).GetMany(&listByWhere)
    for i := 0; i < len(listByWhere); i++ {
        fmt.Println(listByWhere[i])
    }
```
上述代码运行后得到的SQL预处理语句以及相关参数如下
```sql
    SELECT * FROM person WHERE type = ? AND age IN (?,?) AND money BETWEEN (?) AND (?) AND CONCAT(money,'') = ? AND name LIKE concat('%',?,'%')
    0 18 20 100.1 200.9 100.15 li
```
#### 查询条件相关操作
还有更多的查询操作，你需要知道

| 操作名                | 等同于         |
|--------------------|-------------|
| builder.Eq         | =           |
| builder.Ne         | !=          |
| builder.Gt         | \>          |
| builder.Ge         | \>=         |
| builder.Lt         | \<          |
| builder.Le         | \<=         |
| builder.In         | In          |
| builder.NotIn      | Not In      |
| builder.Like       | LIKE        |
| builder.NotLike    | Not Like    |
| builder.Between    | Between     |
| builder.NotBetween | Not Between |

#### 联合查询
使用 `LeftJoin` 方法, 你可以使用联合查询
```go
    var list2 []ArticleVO
    
    var where2 []builder.WhereItem
    where2 = append(where2, builder.WhereItem{Field: "o.type", Opt: builder.Eq, Val: 0})
    where2 = append(where2, builder.WhereItem{Field: "p.age", Opt: builder.In, Val: []int{18, 20}})
    
    aorm.Use(db).Debug(true).
        Table("article o").
        LeftJoin("person p", "p.id=o.person_id").
        Select("o.*").
        Select("p.name as person_name").
        WhereArr(where2).
        GetMany(&list2)
```
上述代码运行后得到的SQL预处理语句以及相关参数如下
```sql
    SELECT o.*,p.name as person_name FROM article o LEFT JOIN person p ON p.id=o.person_id WHERE o.type = ? AND p.age IN (?,?)
    0 18 20
```
其他的联合查询方法还有 `RightJoin`, `Join`
#### 分组查询
使用 `GroupBy` 方法, 你可以进行分组查询
```go
    type PersonAge struct {
        Age         null.Int
        AgeCount    null.Int
    }

    var personAge PersonAge
    
    var where []builder.WhereItem
    where = append(where, builder.WhereItem{Field: "type", Opt: builder.Eq, Val: 0})

    err := aorm.Use(db).Debug(true).
        Table("person").
        Select("age").
        Select("count(age) as age_count").
        GroupBy("age").
        WhereArr(where).
        GetOne(&personAge)
    if err != nil {
        panic(err)
    }
	fmt.Println(personAge)
```
上述代码运行后得到的SQL预处理语句以及相关参数如下
```sql
    SELECT age,count(age) as age_count FROM person WHERE type = ? GROUP BY age Limit ?,?
    0 0 1
```
#### 筛选
使用 `HavingArr` 以及 `Having` 方法, 你可以对分组查询的结果进行筛选
```go
    var listByHaving []PersonAge
    
    var where3 []builder.WhereItem
    where3 = append(where3, builder.WhereItem{Field: "type", Opt: builder.Eq, Val: 0})
    
    var having []builder.WhereItem
    having = append(having, builder.WhereItem{Field: "age_count", Opt: builder.Gt, Val: 4})
    
    err := aorm.Use(db).Debug(true).
        Table("person").
        Select("age").
        Select("count(age) as age_count").
        GroupBy("age").
        WhereArr(where3).
        HavingArr(having).
        GetMany(&listByHaving)
    if err != nil {
        panic(err)
    }
    fmt.Println(listByHaving)
```
上述代码运行后得到的SQL预处理语句以及相关参数如下
```sql
    SELECT age,count(age) as age_count FROM person WHERE type = ? GROUP BY age Having age_count > ?
    0 4
```
#### 排序
使用 `OrderBy` 方法, 你可以对查询结果进行排序
```go
    var listByOrder []Person

    var where []builder.WhereItem
    where = append(where, builder.WhereItem{Field: "type", Opt: builder.Eq, Val: 0})
	
    err := aorm.Use(db).Debug(true).
        Table("person").
        WhereArr(where).
        OrderBy("age", aorm.Desc).
        GetMany(&listByOrder)
    if err != nil {
        panic(err)
    }
    fmt.Println(listByOrder)
```
上述代码运行后得到的SQL预处理语句以及相关参数如下
```sql
    SELECT * FROM person WHERE type = ? Order BY age DESC
    0
```
#### 分页查询
使用 `Limit` 或者 `Page` 方法, 你可以进行分页查询
```go
    var list3 []Person

    var where1 []builder.WhereItem
    where1 = append(where1, builder.WhereItem{Field: "type", Opt: builder.Eq, Val: 0})
	
    err1 := aorm.Use(db).Debug(true).
        Table("person").
        WhereArr(where1).
        Limit(50, 10).
        GetMany(&list3)
    if err1 != nil {
        panic(err1)
    }
    fmt.Println(list3)
```
上述代码运行后得到的SQL预处理语句以及相关参数如下
```sql
    SELECT * FROM person WHERE type = ? Limit ?,?
    0 50 10
```
```go
    var list4 []Person

    var where2 []builder.WhereItem
    where2 = append(where2, builder.WhereItem{Field: "type", Opt: builder.Eq, Val: 0})
	
    err := aorm.Use(db).Debug(true).
        Table("person").
        WhereArr(where2).
        Page(3, 10).
        GetMany(&list4)
    if err != nil {
        panic(err)
    }
    fmt.Println(list4)
```
上述代码运行后得到的SQL预处理语句以及相关参数如下
```sql
    SELECT * FROM person WHERE type = ? Limit ?,?
    0 20 10
```
#### 悲观锁
使用 `LockForUpdate` 方法, 你可以在查询时候锁住某些记录，禁止他们被修改
```go
    var itemByLock Person
    err := aorm.Use(db).Debug(true).LockForUpdate(true).Where(&Person{Id: null.IntFrom(id)}).GetOne(&itemByLock)
    if err != nil {
        panic(err)
    }
    fmt.Println(itemByLock)
```
上述代码运行后得到的SQL预处理语句以及相关参数如下
```sql
    SELECT * FROM person WHERE id = ? Limit ?,?  FOR UPDATE
    2 0 1
```

#### 自增操作
使用 `Increment` 方法, 你可以直接操作某字段增加数值
```go
    count, err := aorm.Use(db).Debug(true).Where(&Person{Id: null.IntFrom(id)}).Increment("age", 1)
    if err != nil {
        panic(err)
    }
    fmt.Println(count)
```
上述代码运行后得到的SQL预处理语句以及相关参数如下
```sql
    UPDATE person SET age=age+? WHERE id = ?
    1 2
```

#### 自减操作
使用 `Decrement` 方法, 你可以直接操作某字段减少数值
```go
    count, err := aorm.Use(db).Debug(true).Where(&Person{Id: null.IntFrom(id)}).Decrement("age", 2)
    if err != nil {
        panic(err)
    }
    fmt.Println(count)
```
上述代码运行后得到的SQL预处理语句以及相关参数如下
```sql
    UPDATE person SET age=age-? WHERE id = ?
    2 2
```

#### 查询某字段的值
使用 `Value` 方法, 你可以直接获取到某字段的值。
```go
    var name string
    errName := aorm.Use(db).Debug(true).Where(&Person{Id: null.IntFrom(id)}).Value("name", &name)
    if errName != nil {
        panic(errName)
    }
    fmt.Println(name)
```
上述代码运行后得到的SQL预处理语句以及相关参数如下
```sql
    SELECT name FROM person WHERE id = ? Limit ?,?
    2 0 1
```
打印结果为 `Alice`

#### 查询某列的值
使用 `Pluck` 方法, 你可以直接查询某列的值。
```go
    var nameList []string
    err := aorm.Use(db).Debug(true).Where(&Person{Type: null.IntFrom(0)}).Limit(0, 5).Pluck("name", &nameList)
    if err != nil {
        panic(err)
    }
    for i := 0; i < len(nameList); i++ {
        fmt.Println(nameList[i])
    }
```
上述代码运行后得到的SQL预处理语句以及相关参数如下
```sql
    SELECT name FROM person WHERE type = ? Limit ?,?
    0 0 5
```

### 聚合查询
#### Count
使用 `Count` 方法, 你可以查询出记录总数量
```go
    count, err := aorm.Use(db).Debug(true).Where(&Person{Age: null.IntFrom(18)}).Count("*")
    if err != nil {
        panic(err)
    }
    fmt.Println(count)
```
上述代码运行后得到的SQL预处理语句以及相关参数如下
```sql
    SELECT count(*) as c FROM person WHERE age = ?
    18
```
 
#### Sum
使用 `Sum` 方法, 你可以查询出符合条件的某字段之和
```go
    sum, err := aorm.Use(db).Debug(true).Where(&Person{Age: null.IntFrom(18)}).Sum("age")
    if err != nil {
        panic(err)
    }
    fmt.Println(sum)
```
上述代码运行后得到的SQL预处理语句以及相关参数如下
```sql
    SELECT sum(age) as c FROM person WHERE age = ?
    18
```
 
#### Avg
使用 `Avg` 方法, 你可以查询出符合条件的某字段平均值
```go
    avg, err := aorm.Use(db).Debug(true).Where(&Person{Age: null.IntFrom(18)}).Avg("age")
    if err != nil {
        panic(err)
    }
    fmt.Println(avg)
```
上述代码运行后得到的SQL预处理语句以及相关参数如下
```sql
    SELECT avg(age) as c FROM person WHERE age = ?
    18
```
 
#### Min
使用 `Min` 方法, 你可以查询出符合条件的某字段最小值
```go
    min, err := aorm.Use(db).Debug(true).Where(&Person{Age: null.IntFrom(18)}).Min("age")
    if err != nil {
        panic(err)
    }
    fmt.Println(min)
```
上述代码运行后得到的SQL预处理语句以及相关参数如下
```sql
    SELECT min(age) as c FROM person WHERE age = ?
    18
```

#### Max
使用 `Max` 方法, 你可以查询出符合条件的某字段最大值
```go
    max, err := aorm.Use(db).Debug(true).Where(&Person{Age: null.IntFrom(18)}).Max("age")
    if err != nil {
        panic(err)
    }
    fmt.Println(max)
```
上述代码运行后得到的SQL预处理语句以及相关参数如下
```sql
    SELECT max(age) as c FROM person WHERE age = ?
    18
```

### 通用操作
使用 `Query` 方法, 你可以执行自定义的查询
#### Query
```go
    resQuery, err := aorm.Use(db).Debug(true).Query("SELECT * FROM person WHERE id=? AND type=?", 1, 3)
    if err != nil {
        panic(err)
    }
    fmt.Println(resQuery)
```
上述代码运行后得到的SQL预处理语句以及相关参数如下
```sql
    SELECT * FROM person WHERE id=? AND type=?
    1 3
```

#### Exec
使用 `Exec` 方法, 你可以执行自定义的修改操作
```go
    resExec, err := aorm.Use(db).Debug(true).Exec("UPDATE person SET name = ? WHERE id=?", "Bob", 3)
    if err != nil {
        panic(err)
    }
    fmt.Println(resExec.RowsAffected())
```
上述代码运行后得到的SQL预处理语句以及相关参数如下
```sql
    UPDATE person SET name = ? WHERE id=?
    Bob 3
```

### 事务操作
使用 `db` 的 `Begin` 方法, 开始一个事务    
然后使用 `Commit` 方法提交事务，`Rollback` 方法回滚事务
```go
    tx, _ := db.Begin()
    
    id, errInsert := aorm.Use(tx).Insert(&Person{
        Name: null.StringFrom("Alice"),
    })
    
    if errInsert != nil {
        fmt.Println(errInsert)
        tx.Rollback()
        return
    }
    
    countUpdate, errUpdate := aorm.Use(tx).Where(&Person{
        Id: null.IntFrom(id),
    }).Update(&Person{
        Name: null.StringFrom("Bob"),
    })
    
    if errUpdate != nil {
        fmt.Println(errUpdate)
        tx.Rollback()
        return
    }
    
    fmt.Println(countUpdate)
    tx.Commit()
```
上述代码运行后得到的SQL预处理语句以及相关参数如下
```sql
    INSERT INTO person (name) VALUES (?)
    Alice
                              
    UPDATE person SET name=? WHERE id = ?
    Bob 3
```

### 清空表数据
使用 `Truncate` 方法, 你可以很方便的清空一个表    
```go
    count, err := aorm.Use(db).Table("person").Truncate()
    if err != nil {
        panic(err)
    }
    fmt.Println(count)
```
上述代码运行后得到的SQL预处理语句以及相关参数如下
```sql
    TRUNCATE TABLE person
```

### 工具类
使用 `Ul` 或者 `UnderLine` 方法, 你可以将一个字符串从驼峰写法转换成下划线写法   
例如, 转换 `personId` 成 `person_id`
```go
    var list2 []ArticleVO
    var where2 []builder.WhereItem
    where2 = append(where2, builder.WhereItem{Field: "o.type", Opt: builder.Eq, Val: 0})
    where2 = append(where2, builder.WhereItem{Field: "p.age", Opt: builder.In, Val: []int{18, 20}})
	
    aorm.Use(db).Debug(true).
        Table("article o").
        LeftJoin("person p", helper.Ul("p.id=o.personId")).
        Select("o.*").
        Select(helper.Ul("p.name as personName")).
        WhereArr(where2).
        GetMany(&list2)
```
上述代码运行后得到的SQL预处理语句以及相关参数如下
```sql
    SELECT o.*,p.name as person_name FROM article o LEFT JOIN person p ON p.id=o.person_id WHERE o.type = ? AND p.age IN (?,?)
    0 18 20
```
## 基准测试
https://github.com/tangpanqing/orm-benchmark

## 作者

👤 **tangpanqing**

* Twitter: [@tangpanqing](https://twitter.com/tangpanqing)
* Github: [@tangpanqing](https://github.com/tangpanqing)
* Wechat: tangpanqing    
  ![wechat](./wechat.jpg)
## 希望能得到你的支持
给个 ⭐ 吧，如果这个项目帮助到你