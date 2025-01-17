[English](https://github.com/tangpanqing/aorm) | [简体中文](https://github.com/tangpanqing/aorm/blob/main/README_zh.md)

# Aorm
Aorm is a GoLang library for operate database.

Give a ⭐ if this project helped you!
## 🌟 Feature
- [x] Simply And Fast
- [x] Support MySQL,MSSQL,Postgres,Sqlite3 DataBase
- [x] Support Null Value When Query Or Exec
- [x] Support Auto Migrate
- [X] Support SQL Builder

## 🌟 Usage
- [Import](#import)
  - [Mysql](#mysql)
  - [Sqlite](#sqlite)
  - [MSSQL](#mssql)
  - [Postgres](#postgres)
- [Define data struct](#define-data-struct)   
- [Connect database](#connect-database)   
- [Migrate](#migrate)
- [Basic CRUD](#basic-crud)   
  - [Insert one record](#insert-one-record)
  - [Insert many record](#insert-many-record)
  - [Get one record](#get-one-record)
  - [Get many record](#get-many-record)
  - [Update record](#update-record)
  - [Delete record](#delete-record)
- [Advanced Query](#advanced-query)
  - [Table](#table)
  - [Select](#select)
  - [Where](#where)
  - [Where Operate](#where-operate)
  - [Join](#join)
  - [GroupBy](#groupBy)
  - [Having](#having)
  - [OrderBy](#orderBy)
  - [Limit and Page](#limit-and-page)
  - [Lock](#lock)
  - [Increment](#increment)
  - [Decrement](#decrement)
  - [Value](#value)
  - [Pluck](#pluck)
- [Aggregation Function](#aggregation-function)
  - [Count](#count)
  - [Sum](#sum)
  - [AVG](#avg)
  - [Min](#min)
  - [Max](#max)
- [Common](#common)
  - [Query](#query)
  - [Exec](#exec)
- [Transaction](#transaction)
- [Truncate](#truncate)
- [Utils](#utils)

### Import
```go
    import (
        "database/sql"
        _ "github.com/go-sql-driver/mysql" 
        "github.com/tangpanqing/aorm"
    )
```
`database/sql` the go std package, provide sql operate database interface   

`github.com/tangpanqing/aorm` wrapper of the sql operate, make it easy for use    

`github.com/go-sql-driver/mysql` mysql driver package, if you use other database, change it

you can download them like this

```shell
go get -u github.com/tangpanqing/aorm
```

#### Mysql
if you use `Mysql` database, you could use this driver `github.com/go-sql-driver/mysql`
```shell
go get -u github.com/go-sql-driver/mysql
```

#### Sqlite
if you use `Sqlite` database, you could use this driver `github.com/mattn/go-sqlite3`
```shell
go get -u github.com/mattn/go-sqlite3
```

#### Mssql
if you use `Mssql` database, you could use this driver `github.com/denisenkom/go-mssqldb`
```shell
go get -u github.com/denisenkom/go-mssqldb
```

#### Postgres
if you use `Postgres` database, you could use this driver `github.com/lib/pq`
```shell
go get -u github.com/lib/pq
```

### Define data struct
you should define data struct before operate database, like this
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
first, notice that like `null.Int`, `null.String`, `null.Bool`, `null.Float`, `null.Time`, which is a struct that wrapper of the `sql.NUll*` struct    

second, notice that like `aorm:` tag, this will be used when migrate data struct to database, some info you need know 

| key name       | key value | info                           | example        |
|----------------|-----------|--------------------------------|----------------|
| primary        | none      | set a primary column           | primary        |
| unique         | none      | set a unique column            | unique         |
| index          | none      | set a index column             | index          |
| auto_increment | none      | set a column auto increment    | auto_increment |
| not null       | none      | set a column allow null or not | not null       |
| type           | string    | set a column's data type       | type:double    |
| size           | int       | set a column's length or size  | size:100       |
| comment        | string    | set a column's comment         | comment:名字     |
| default        | string    | set a column's default value   | default:2      |


### Connect database
by `sql.Open` function, you can connect the database, and then you should ping test  
```go
    //replace this database param
    username := "root"
    password := "root"
    hostname := "localhost"
    port := "3306"
    dbname := "database_name"
    
    //connect
    db, err := sql.Open("mysql", username+":"+password+"@tcp("+hostname+":"+port+")/"+dbname+"?charset=utf8mb4&parseTime=True&loc=Local")
    if err != nil {
        panic(err)
    }
    defer db.Close()
    
    //ping test
    err1 := db.Ping()
    if err1 != nil {
        panic(err1)
    }
```
if you use other database, please use diff dns, for example
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

### Migrate
by `AutoMigrate` function, the table name will be `person`, underline style string with the struct name
```go
    aorm.Migrator(db).Driver("mysql").Opinion("ENGINE", "InnoDB").Opinion("COMMENT", "用户表").AutoMigrate(&Person{})
```
by `Migrate` function, You can also use other table name
```go
    aorm.Migrator(db).Driver("mysql").Opinion("ENGINE", "InnoDB").Opinion("COMMENT", "用户表").Migrate("person_1", &Person{})
```
by `ShowCreateTable` function, You can get the create table sql
```go
    showCreate := aorm.Migrator(db).Driver("mysql").ShowCreateTable("person")
    fmt.Println(showCreate)
```
like this
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

### Basic CRUD

#### Insert one record
by `Insert` function, you can insert one record from data struct
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
then get the sql and params like this
```sql
    INSERT INTO person (name,sex,age,type,create_time,money,test) VALUES (?,?,?,?,?,?,?)
    Alice false 18 0 2022-12-07 10:10:26.1450773 +0800 CST m=+0.031808801 100.15987654321 200.15987654321987
```

if you use `mssql` database or `postgres` database, please add `Driver` method, like this
```go
    id, errInsert := aorm.Use(db).Debug(true).Driver("mssql").Insert(&Person{
	......
    })
```
by `Driver` Method, aorm can and some sql suffix for get last insert id

#### Insert many record
use `InsertBatch` function, you can insert many record from data slice
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
then get the sql and params like this
```sql
    INSERT INTO person (name,sex,age,type,create_time,money,test) VALUES (?,?,?,?,?,?,?),(?,?,?,?,?,?,?)
    Alice false 18 0 2022-12-16 15:28:49.3907587 +0800 CST m=+0.022987201 100.15987654321 200.15987654321987 Bob true 18 0 2022-12-16 15:28:49.3907587 +0800 CST m=+0.022987201 100.15987654321 200.15987654321987
```

#### Get one record
by `GetOne` function, you can get one record
```go
    var person Person
    errFind := aorm.Use(db).Debug(true).Where(&Person{Id: null.IntFrom(id)}).GetOne(&person)
    if errFind != nil {
        fmt.Println(errFind)
    }
    fmt.Println(person)
```
then get the sql and params like this
```sql
    SELECT * FROM person WHERE id = ? Limit ?,?
    1 0 1
```
if you use `mssql` database, please add `Driver` method and `OrderBy` method, like this
```go
    errFind := aorm.Use(db).Debug(true).Driver("mssql").Where(&Person{Id: null.IntFrom(id)}).OrderBy("id","DESC").GetOne(&person)
```
by `Driver` Method, aorm can change some sql suffix for get record

#### Get many record
by `GetMany` function, you can get many record
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
then get the sql and params like this
```sql
    SELECT * FROM person WHERE type = ?
    0
```

#### Update record
by `Update` function, you can update record
```go
    countUpdate, errUpdate := aorm.Use(db).Debug(true).Where(&Person{Id: null.IntFrom(id)}).Update(&Person{Name: null.StringFrom("Bob")})
    if errUpdate != nil {
        fmt.Println(errUpdate)
    }
    fmt.Println(countUpdate)
```
then get the sql and params like this
```sql
    UPDATE person SET name=? WHERE id = ?
    Bob 1
```

#### Delete record
by `Delete` function, you can delete record
```go
    countDelete, errDelete := aorm.Use(db).Debug(true).Where(&Person{Id: null.IntFrom(id)}).Delete()
    if errDelete != nil {
        fmt.Println(errDelete)
    }
    fmt.Println(countDelete)
```
then get the sql and params like this
```sql
    DELETE FROM person WHERE id = ?
    1
```

### Advanced Query
#### Table
by `Table` function, you can set table name easy
```go
    aorm.Use(db).Debug(true).Table("person_1").Insert(&Person{Name: null.StringFrom("Cherry")})
```
then get the sql and params like this
```sql
    INSERT INTO person_1 (name) VALUES (?)
    Cherry
```
#### Select
by `Select` function, you can select field name easy
```go
    var listByFiled []Person
    aorm.Use(db).Debug(true).Select("name,age").Where(&Person{Age: null.IntFrom(18)}).GetMany(&listByFiled)
```
then get the sql and params like this
```sql
    SELECT name,age FROM person WHERE age = ?
    18
```
#### Where
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
then get the sql and params like this
```sql
    SELECT * FROM person WHERE type = ? AND age IN (?,?) AND money BETWEEN (?) AND (?) AND CONCAT(money,'') = ? AND name LIKE concat('%',?,'%')
    0 18 20 100.1 200.9 100.15 li
```
#### Where Operate
there are some other operates you should know

| Opt Name           | Same As      |
|--------------------|--------------|
| builder.Eq         | =            |
| builder.Ne         | !=           |
| builder.Gt         | \>           |
| builder.Ge         | \>=          |
| builder.Lt         | \<           |
| builder.Le         | \<=          |
| builder.In         | In           |
| builder.NotIn      | Not In       |
| builder.Like       | LIKE         |
| builder.NotLike    | Not Like     |
| builder.Between    | Between      |
| builder.NotBetween | Not Between  |

#### JOIN
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
then get the sql and params like this
```sql
    SELECT o.*,p.name as person_name FROM article o LEFT JOIN person p ON p.id=o.person_id WHERE o.type = ? AND p.age IN (?,?)
    0 18 20
```
some other join function like this `RightJoin`, `Join`
#### GroupBy
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
then get the sql and params like this
```sql
    SELECT age,count(age) as age_count FROM person WHERE type = ? GROUP BY age Limit ?,?
    0 0 1
```
#### Having
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
then get the sql and params like this
```sql
    SELECT age,count(age) as age_count FROM person WHERE type = ? GROUP BY age Having age_count > ?
    0 4
```
#### OrderBy
```go
    var listByOrder []Person

    var where []builder.WhereItem
    where = append(where, builder.WhereItem{Field: "type", Opt: builder.Eq, Val: 0})
	
    err := aorm.Use(db).Debug(true).
        Table("person").
        WhereArr(where).
        OrderBy("age", builder.Desc).
        GetMany(&listByOrder)
    if err != nil {
        panic(err)
    }
    fmt.Println(listByOrder)
```
then get the sql and params like this
```sql
    SELECT * FROM person WHERE type = ? Order BY age DESC
    0
```
#### Limit and Page
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
then get the sql and params like this
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
then get the sql and params like this
```sql
    SELECT * FROM person WHERE type = ? Limit ?,?
    0 20 10
```
#### Lock
by `Lock` function, you can lock the query
```go
    var itemByLock Person
    err := aorm.Use(db).Debug(true).LockForUpdate(true).Where(&Person{Id: null.IntFrom(id)}).GetOne(&itemByLock)
    if err != nil {
        panic(err)
    }
    fmt.Println(itemByLock)
```
then get the sql and params like this
```sql
    SELECT * FROM person WHERE id = ? Limit ?,?  FOR UPDATE
    2 0 1
```

#### Increment
```go
    count, err := aorm.Use(db).Debug(true).Where(&Person{Id: null.IntFrom(id)}).Increment("age", 1)
    if err != nil {
        panic(err)
    }
    fmt.Println(count)
```
then get the sql and params like this
```sql
    UPDATE person SET age=age+? WHERE id = ?
    1 2
```

#### Decrement
```go
    count, err := aorm.Use(db).Debug(true).Where(&Person{Id: null.IntFrom(id)}).Decrement("age", 2)
    if err != nil {
        panic(err)
    }
    fmt.Println(count)
```
then get the sql and params like this
```sql
    UPDATE person SET age=age-? WHERE id = ?
    2 2
```

#### Value
```go
    var name string
    errName := aorm.Use(db).Debug(true).Where(&Person{Id: null.IntFrom(id)}).Value("name", &name)
    if errName != nil {
        panic(errName)
    }
    fmt.Println(name)
```
then get the sql and params like this
```sql
    SELECT name FROM person WHERE id = ? Limit ?,?
    2 0 1
```
then print the value `Alice`

#### Pluck
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
then get the sql and params like this
```sql
    SELECT name FROM person WHERE type = ? Limit ?,?
    0 0 5
```

### Aggregation Function
#### Count
```go
    count, err := aorm.Use(db).Debug(true).Where(&Person{Age: null.IntFrom(18)}).Count("*")
    if err != nil {
        panic(err)
    }
    fmt.Println(count)
```
then get the sql and params like this
```sql
    SELECT count(*) as c FROM person WHERE age = ?
    18
```
 
#### Sum
```go
    sum, err := aorm.Use(db).Debug(true).Where(&Person{Age: null.IntFrom(18)}).Sum("age")
    if err != nil {
        panic(err)
    }
    fmt.Println(sum)
```
then get the sql and params like this
```sql
    SELECT sum(age) as c FROM person WHERE age = ?
    18
```
 
#### Avg
```go
    avg, err := aorm.Use(db).Debug(true).Where(&Person{Age: null.IntFrom(18)}).Avg("age")
    if err != nil {
        panic(err)
    }
    fmt.Println(avg)
```
then get the sql and params like this
```sql
    SELECT avg(age) as c FROM person WHERE age = ?
    18
```



#### min
```go
    min, err := aorm.Use(db).Debug(true).Where(&Person{Age: null.IntFrom(18)}).Min("age")
    if err != nil {
        panic(err)
    }
    fmt.Println(min)
```
then get the sql and params like this
```sql
    SELECT min(age) as c FROM person WHERE age = ?
    18
```



#### Max
```go
    max, err := aorm.Use(db).Debug(true).Where(&Person{Age: null.IntFrom(18)}).Max("age")
    if err != nil {
        panic(err)
    }
    fmt.Println(max)
```
then get the sql and params like this
```sql
    SELECT max(age) as c FROM person WHERE age = ?
    18
```
 
### Common
#### Query
```go
    resQuery, err := aorm.Use(db).Debug(true).Query("SELECT * FROM person WHERE id=? AND type=?", 1, 3)
    if err != nil {
        panic(err)
    }
    fmt.Println(resQuery)
```
then get the sql and params like this
```sql
    SELECT * FROM person WHERE id=? AND type=?
    1 3
```

#### Exec
```go
    resExec, err := aorm.Use(db).Debug(true).Exec("UPDATE person SET name = ? WHERE id=?", "Bob", 3)
    if err != nil {
        panic(err)
    }
    fmt.Println(resExec.RowsAffected())
```
then get the sql and params like this
```sql
    UPDATE person SET name = ? WHERE id=?
    Bob 3
```

### Transaction
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
then get the sql and params like this
```sql
    INSERT INTO person (name) VALUES (?)
    Alice
                              
    UPDATE person SET name=? WHERE id = ?
    Bob 3
```

### Truncate
```go
    count, err := aorm.Use(db).Table("person").Truncate()
    if err != nil {
        panic(err)
    }
    fmt.Println(count)
```
then get the sql and params like this
```sql
    TRUNCATE TABLE person
```

### Utils
by `Ul` or `UnderLine`, you can transform camel case string to underline case    
for example, transform `personId` to `person_id`
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
then get the sql and params like this
```sql
    SELECT o.*,p.name as person_name FROM article o LEFT JOIN person p ON p.id=o.person_id WHERE o.type = ? AND p.age IN (?,?)
    0 18 20
```

## Benchmark
https://github.com/tangpanqing/orm-benchmark

## Author

👤 **tangpanqing**

* Twitter: [@tangpanqing](https://twitter.com/tangpanqing)
* Github: [@tangpanqing](https://github.com/tangpanqing)
* Wechat: tangpanqing    
  ![wechat](./wechat.jpg)
## Show Your Support
Give a ⭐ if this project helped you!
