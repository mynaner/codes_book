

# CRUD

```go
// 定义结构体模型
type User struct {
	ID int64
	Name string `gorm:"default:'小样'"`
	Age int64
}
func main() {
	db,err:=gorm.Open("mysql", "root:root1234@(127.0.0.1:13306)/db1?charset=utf8&parseTime=True&loc=Local")
	if err !=nil {
		panic(err)
	}
	defer db.Close()
	// 自动迁移 在不影响之前数据的情况下自动建表或增加表字段
	db.AutoMigrate(&User{}) 
	user1:= User{ Age:  11 }
  // 使用 NewRecord() 查询主键是否存在
	fmt.Println(db.NewRecord(&User{}))
  // 创建记录Create()
  db.Debug().Create(&user1) // Debug() 可以打印出sql语句
}
```

```go
// 创建 
// 通过tag字段定义表字段的默认值
type User struct {
	ID int64
	Name string `gorm:"default:'小样'"`
	Age int64
}
// 注意：通过tag定义字段的默认值，在创建记录时候生成的 SQL 语句会排除没有值或值为 零值 的字段。 
// 在将记录插入到数据库后，Gorm会从数据库加载那些字段的默认值。
// 所有字段的零值, 比如0, "",false或者其它零值，都不会保存到数据库内，但会使用他们的默认值。 
// 可以使用 指针 或  实现 Scanner/Valuer接口 避免

// 指针方式实现零值
type User struct {
	ID int64
	Name *string `gorm:"default:'小样'"`
	Age int64
}
user1:= User{ Name:new(string), Age:  11}
// Scanner/Valuer 接口方式实现零值
type User struct {
	ID int64
	Name sql.NullString `gorm:"default:'小样'"`
	Age int64
}
user1:= User{ Name:sql.NullString{ String: "", Valid:  true, }, Age:  11}


// 拓展创建选项
// PostgreSQL数据库中可以使用下面的方式实现合并插入, 有则更新, 无则插入
// 为Instert语句添加扩展SQL选项
db.Set("gorm:insert_option", "ON CONFLICT").Create(&product)
// INSERT INTO products (name, code) VALUES ("name", "code") ON CONFLICT;
```

```go
// 查询
var user User
// 根据主键查询第一条数据
db.Debug().First(&user)
//  SELECT * FROM `users`  WHERE `users`.`deleted_at` IS NULL ORDER BY `users`.`id` ASC LIMIT 1
// 根据主键查最后一条记录
db.Debug().Last(&user)
// SELECT * FROM `users`  WHERE `users`.`deleted_at` IS NULL ORDER BY `users`.`id` DESC LIMIT 1
// 随机获取一条记录
db.Debug().Take(&user)
// SELECT * FROM `users`  WHERE `users`.`deleted_at` IS NULL AND `users`.`id` = 1 LIMIT 1
// 查询指定的某条记录(仅当主键为整型时可用)
db.Debug().First(&user,1)
//  SELECT * FROM `users`  WHERE `users`.`deleted_at` IS NULL AND ((`users`.`id` = 1)) ORDER BY `users`.`id` ASC LIMIT 1  
var users []User
// 查询所有数据
db.Debug().Find(&users)
// SELECT * FROM `users`  WHERE `users`.`deleted_at` IS NULL 


// Or条件
db.Where("role = ?", "admin").Or("role = ?", "super_admin").Find(&users)
//// SELECT * FROM users WHERE role = 'admin' OR role = 'super_admin';

// Struct
db.Where("name = 'jinzhu'").Or(User{Name: "jinzhu 2"}).Find(&users)
//// SELECT * FROM users WHERE name = 'jinzhu' OR name = 'jinzhu 2';

// Map
db.Where("name = 'jinzhu'").Or(map[string]interface{}{"name": "jinzhu 2"}).Find(&users)
//// SELECT * FROM users WHERE name = 'jinzhu' OR name = 'jinzhu 2';
```

```go

// Where 条件查询
// 查询满足条件的第一条数据
//  SELECT * FROM `users`  WHERE `users`.`deleted_at` IS NULL AND ((name='张三')) ORDER BY `users`.`id` ASC LIMIT 1
db.Debug().Where("name=?","张三").First(&user)
// 查询满足条件的所有数据
//  SELECT * FROM `users`  WHERE `users`.`deleted_at` IS NULL AND ((name='张三'))
db.Debug().Where("name=?","张三").Find(&users)
// <> 不等于, 查询 不满足条件的数据
// SELECT * FROM `users`  WHERE `users`.`deleted_at` IS NULL AND ((name <> '张三 '))
db.Debug().Where("name <> ?","张三 ").Find(&users)
// in , 满足于多个
// SELECT * FROM `users`  WHERE `users`.`deleted_at` IS NULL AND ((name in ('张三','小张')))
db.Debug().Where("name in (?)",[]string{"张三","小张"}).Find(&users)
// like 模糊查询 	%张 | %张% | 张%
// SELECT * FROM `users`  WHERE `users`.`deleted_at` IS NULL AND ((name like '%张%'))
db.Debug().Where("name like ?","%张%").Find(&users)
// AND 多条件查询
// SELECT * FROM `users`  WHERE `users`.`deleted_at` IS NULL AND ((name like '%张%' AND age <= '22'))
db.Debug().Where("name like ? AND age <= ?", "%张%", "22").Find(&users)
// Time
// SELECT * FROM `users`  WHERE `users`.`deleted_at` IS NULL AND ((updated_at < '2020-11-12 16:19:07'))
db.Debug().Where("updated_at < ?",time.Now()).Find(&users)
// BETWEEN
// SELECT * FROM `users`  WHERE `users`.`deleted_at` IS NULL AND ((created_at BETWEEN '12120-110-110' AND '2020-11-12 17:33:14')) 
db.Debug().Where("created_at BETWEEN ? AND ?",time.Now().Format("2020-10-10"),time.Now()).Find(&users)
```

```go
// Struct & Map查询
// Struct
// SELECT * FROM users WHERE name = "jinzhu" AND age = 20 LIMIT 1;
db.Where(&User{Name: "jinzhu", Age: 20}).First(&user)
// Map
// SELECT * FROM users WHERE name = "jinzhu" AND age = 20;
db.Where(map[string]interface{}{"name": "jinzhu", "age": 20}).Find(&users)
// 主键的切片
// SELECT * FROM users WHERE id IN (20, 21, 22);
db.Where([]int64{20, 21, 22}).Find(&users)
// 当通过结构体进行查询时，GORM将会只通过非零值字段查询，这意味着如果你的字段值为0，''，false或者其他零值时，将不会被用于构建查询条件
// 使用指针或实现 Scanner/Valuer 接口来避免
```

```go
// Not 条件
db.Not("name", "jinzhu").First(&user)
//// SELECT * FROM users WHERE name <> "jinzhu" LIMIT 1;

// Not In
db.Not("name", []string{"jinzhu", "jinzhu 2"}).Find(&users)
//// SELECT * FROM users WHERE name NOT IN ("jinzhu", "jinzhu 2");

// Not In slice of primary keys
db.Not([]int64{1,2,3}).First(&user)
//// SELECT * FROM users WHERE id NOT IN (1,2,3);

db.Not([]int64{}).First(&user)
//// SELECT * FROM users;

// Plain SQL
db.Not("name = ?", "jinzhu").First(&user)
//// SELECT * FROM users WHERE NOT(name = "jinzhu");

// Struct
db.Not(User{Name: "jinzhu"}).First(&user)
//// SELECT * FROM users WHERE name <> "jinzhu";
```

```go
// 立即执行方法
// Immediate methods ，立即执行方法是指那些会立即生成SQL语句并发送到数据库的方法, 他们一般是CRUD方法，比如：
// Create, First, Find, Take, Save, UpdateXXX, Delete, Scan, Row, Rows…
// 这有一个基于上面链式方法代码的立即执行方法的例子：
tx.Find(&user)
// 生成的SQL语句如下：
// SELECT * FROM users where name = 'jinzhu' AND age = 30 AND active = 1;

// 内联条件
// 根据主键获取记录 (只适用于整形主键)
db.First(&user, 23)
//// SELECT * FROM users WHERE id = 23 LIMIT 1;
// 根据主键获取记录, 如果它是一个非整形主键
db.First(&user, "id = ?", "string_primary_key")
//// SELECT * FROM users WHERE id = 'string_primary_key' LIMIT 1;

// Plain SQL
db.Find(&user, "name = ?", "jinzhu")
//// SELECT * FROM users WHERE name = "jinzhu";

db.Find(&users, "name <> ? AND age > ?", "jinzhu", 20)
//// SELECT * FROM users WHERE name <> "jinzhu" AND age > 20;

// Struct
db.Find(&users, User{Age: 20})
//// SELECT * FROM users WHERE age = 20; 

// Map
db.Find(&users, map[string]interface{}{"age": 20})
//// SELECT * FROM users WHERE age = 20;
```

```go
// 范围
// Scopes，Scope是建立在链式操作的基础之上的。
// 基于它，你可以抽取一些通用逻辑，写出更多可重用的函数库。
func AmountGreaterThan1000(db *gorm.DB) *gorm.DB {
  return db.Where("amount > ?", 1000)
}
func PaidWithCreditCard(db *gorm.DB) *gorm.DB {
  return db.Where("pay_mode_sign = ?", "C")
}
func PaidWithCod(db *gorm.DB) *gorm.DB {
  return db.Where("pay_mode_sign = ?", "C")
}
func OrderStatus(status []string) func (db *gorm.DB) *gorm.DB {
  return func (db *gorm.DB) *gorm.DB {
    return db.Scopes(AmountGreaterThan1000).Where("status IN (?)", status)
  }
}
db.Scopes(AmountGreaterThan1000, PaidWithCreditCard).Find(&orders)
// 查找所有金额大于 1000 的信用卡订单
db.Scopes(AmountGreaterThan1000, PaidWithCod).Find(&orders)
// 查找所有金额大于 1000 的 COD 订单
db.Scopes(AmountGreaterThan1000, OrderStatus([]string{"paid", "shipped"})).Find(&orders)
// 查找所有金额大于 1000 且已付款或者已发货的订单
```

```go
// 多个立即执行方法
// Multiple Immediate Methods，在 GORM 中使用多个立即执行方法时，后一个立即执行方法会复用前一个立即执行方法的条件 (不包括内联条件) 。
db.Where("name LIKE ?", "jinzhu%").Find(&users, "id IN (?)", []int{1, 2, 3}).Count(&count)
```

