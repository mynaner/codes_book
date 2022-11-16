# GORM

```sql
-- 进入数据库
-- mysql -uroot -p000000 //  mysql -u用户名 -p密码
-- 创建数据库 
create database web;
-- 查看当前服务器所有数据库
SHOW DATABASES;
-- 删除数据库语句 
DROP DATABASE web_01; 
-- 切换数据库 
use web;
-- 查看当前的数据库
select database(); 
-- 删除指定表 
drop table employees;
-- 查看当前数据库所有的表
show tables; 
-- 查看表结构 
desc employe;
-- 查询整个user表
SELECT * FROM user;
```

> 安装: go get -u github.com/jinzhu/gorm

```go
// 连接数据库

// 连接MySQL
import "github.com/jinzhu/gorm"
import _ "github.com/jinzhu/gorm/dialects/mysql"
func main() {
	db,err:=gorm.Open("mysql","user:password@(location)/dbname?charset=utf8mb4&parseTime=true&loc=Local")
	if err !=nil {
		panic(err)
	}
	db.Close()
}
// 连接 PostgresSQL
import (
  "github.com/jinzhu/gorm"
  _ "github.com/jinzhu/gorm/dialects/postgres"
)
func main() {
  db, err := gorm.Open("postgres", "host=myhost port=myport user=gorm dbname=gorm password=mypassword")
  defer db.Close()
}
// 连接Sqlite3
import (
  "github.com/jinzhu/gorm"
  _ "github.com/jinzhu/gorm/dialects/sqlite"
)
func main() {
  db, err := gorm.Open("sqlite3", "/tmp/gorm.db")
  defer db.Close()
}
// 连接SQL Server
import (
  "github.com/jinzhu/gorm"
  _ "github.com/jinzhu/gorm/dialects/mssql"
)
func main() {
  db, err := gorm.Open("mssql", "sqlserver://username:password@localhost:1433?database=dbname")
  defer db.Close()
}
```

```go
// docker 快捷创建MySQL实例
// 在本地的13306端口运行一个名为mysql8019，root用户名密码为root1234的MySQL容器环境:
docker run --name mysql8019 -p 13306:3306 -e MYSQL_ROOT_PASSWORD=root1234 -d mysql:8.0.19

// 启动mysql镜像
docker run -itd --name mysql3306 -p 3306:3306 -e MYSQL_ROOT_PASSWORD=root1234 mysql

// 在另外启动一个MySQL Client连接上面的MySQL环境，密码为上一步指定的密码root1234:
docker run -it --network host --rm mysql mysql -h127.0.0.1 -P13306 --default-character-set=utf8mb4 -uroot -p

```

```go
// GORM 基本示例
package main
import (
	"fmt"
	"github.com/jinzhu/gorm"
	_ "github.com/jinzhu/gorm/dialects/mysql"
)
import _ "github.com/jinzhu/gorm/dialects/mysql"
type UserInfo struct {
	ID uint
	Name string
	Gender string
	Happy string
}
func main() {
	db,err:=gorm.Open("mysql", "root:root1234@(127.0.0.1:13306)/db1?charset=utf8mb4&parseTime=True&loc=Local")
	if err !=nil {
		panic(err)
	}
	defer db.Close()
	// 自动迁移 在不影响之前数据的情况下自动建表或增加表字段
	db.AutoMigrate(&UserInfo{}) 
	//新建数据
  user1:=UserInfo{ 1, "张三", "男", "足球"}
	user2:=UserInfo{ 2, "里斯", "女", "足球"}
	db.Create(&user1)
	db.Create(&user2)
	// 查询
	user3 := UserInfo{}
	db.First(&user3) // 必须为指针 // 查询表第一条数据
	fmt.Println(user3)
	user4 := UserInfo{}
	db.Find(&user4,"id=?","1") // 条件查询
	fmt.Println(user4)
	// 更新
	db.Model(&user4).Update("happy","ddd")
	// 删除
	db.Delete(&user4)
}
```

```go
// GORM Model 定义
// 在代码中定义模型（Models）与数据库中的数据进行映射
// Models 通常是正常定义的结构体，基本的Go类型或者他们的指针
// 同时也支持sql.Scanner 及 driver.Valuer 接口(interface)

// gorm.Model
// 内置的结构体，包含了 ID，CreatedAt，UpdateAt，DeletedAt 四个字段
type Model struct {
	ID        uint `gorm:"primary_key"`
	CreatedAt time.Time
	UpdatedAt time.Time
	DeletedAt *time.Time `sql:"index"`
}

// 模型定义示例
type User struct {
	gorm.Model
	Name         string
	Age          sql.NullInt64
	Birthday     *time.Time
	Email        string  `gorm:"type:varchar(100);unique_index"`
	Role         string  `gorm:"size:255"` // 设置字段大小为255
	MemberNumber *string `gorm:"unique;not null"` // 设置会员号（member number）唯一并且不为空
	Num          int     `gorm:"AUTO_INCREMENT"` // 设置 num 为自增类型
	Address      string  `gorm:"index:addr"` // 给address字段创建名为addr的索引
	IgnoreMe     int     `gorm:"-"` // 忽略本字段
}

// 支持的结构体标记
// Column						指定列名
// Type							指定列数据类型
// Size							指定列大小, 默认值255
// PRIMARY_KEY			将列指定为主键
// UNIQUE						将列指定为唯一
// DEFAULT					指定列默认值
// PRECISION				指定列精度
// NOT NULL					将列指定为非 NULL
// AUTO_INCREMENT		指定列是否为自增类型
// INDEX						创建具有或不带名称的索引, 如果多个索引同名则创建复合索引
// UNIQUE_INDEX	和 INDEX 类似，只不过创建的是唯一索引
// EMBEDDED					将结构设置为嵌入
// EMBEDDED_PREFIX	设置嵌入结构的前缀
// -	忽略此字段

// 关联相关标记（tags） 
// MANY2MANY							指定连接表
// FOREIGNKEY							设置外键
// ASSOCIATION_FOREIGNKEY	设置关联外键
// POLYMORPHIC						指定多态类型
// POLYMORPHIC_VALUE			指定多态值
// JOINTABLE_FOREIGNKEY		指定连接表的外键
// ASSOCIATION_JOINTABLE_FOREIGNKEY	指定连接表的关联外键
// SAVE_ASSOCIATIONS			是否自动完成 save 的相关操作
// ASSOCIATION_AUTOUPDATE	是否自动完成 update 的相关操作
// ASSOCIATION_AUTOCREATE	是否自动完成 create 的相关操作
// ASSOCIATION_SAVE_REFERENCE	是否自动完成引用的 save 的相关操作
// PRELOAD								是否自动完成预加载的相关操作
```

```go
// 主键，表名，列名的约定
// 主键(Primary Key)
type User struct{
	ID uint			// 名为ID的字段会默认作为表的主键
	Name string
}
type Animal struct {
	AnimalID uint `gorm:"primary_key"` // 使用 AnimalID 作为主键
	Name string
	Age int64
}

// 表名(Table Name)
// 表名默认就是结构体名称的复数
type User struct{} // 表明就是 `users`
// 自定义表名
func (User) TableName() string {
  return "frofiles"
}
func (u User) TableName() string{
  if u.Role == "admin"{
    return "admin_users"
  }else{
    return "users"
  }
}
// 禁用默认表名的复数形式。如果设置为true 则没有复数形式
db.SingularTable(true)

// 通过Table()指定表名
// 使用User结构体创建
// 使用User结构体创建名为`deleted_users`的表
db.Table("deleted_users").CreateTable(&User{})

var deleted_users []User
db.Table("deleted_users").Find(&deleted_users)
//// SELECT * FROM deleted_users;

db.Table("deleted_users").Where("name = ?", "jinzhu").Delete()
//// DELETE FROM deleted_users WHERE name = 'jinzhu';

db.Table("deleted_users").Delete(&User{},"name=?","xx")

// 支持更改默认表名称规则
gorm.DefaultTableNameHandler = func(db *gorm.DB, defaultTableName string) string {
  return "prefix_"+defaultTableName
}

// 列名(Column Name)
// 列名默认由字段名称进行下划线分割来生成的
type User struct {
	ID uint				// column name is `id`
	Name string			// column name is `name`
	Birthday time.Time  // column name is `birthday`
	CreateAt time.Time	// column name is `create_at`
}
// 使用结构体tag 指定 列名
type Animal struct {
	AnimalID int64			`gorm:"column:'bast_id'"`
	Birthday time.Time 	`gorm:"column:'day_of_the_bast'"`
	Age uint 						`gorm:"column:'age_of_the_bast'"`
}

// 时间戳跟踪
// CreateAt :如果模型有CreateAt字段,该字段的值将会是初次创建记录的时间
db.Create(&user) // `CreatedAt` 将会是当前时间

// 可以使用`Update`方法来改变`CreateAt`的值
db.Model(&user).Update("CreateAt",time.Now())

// UpdatedAt 该字段的值将会是每次更新记录的时间
db.Save(&User) // `UpdateAt` 将会是当前时间
db.Model(&user).Update("name","jinzhu") // `UpdatedAt` 将会是当前时间
// DeletedAt
// 如果模型有DeleteAt 字段，调用Delete删除该记录时，将会设置DeletedAt字段为当前时间，而不是直接将记录从数据库删除
```

