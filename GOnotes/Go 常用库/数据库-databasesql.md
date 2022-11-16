# Go操作MySQL

>安装：go get -u github.com/go-sql-driver/mysql

```go
// 初始化连接
// 定义一个全局对象db
var db *sql.DB

// 定义一个初始化数据库的函数
func initDB() (err error) {
	// DSN:Data Source Name
	dsn := "user:password@tcp(127.0.0.1:3306)/sql_test?charset=utf8mb4&parseTime=True"
	// 不会校验账号密码是否正确
	// 注意！！！这里不要使用:=，我们是给全局变量赋值，然后在main函数中使用全局变量db
	db, err = sql.Open("mysql", dsn)
	if err != nil {
		return err
	}
	// 尝试与数据库建立连接（校验dsn是否正确）
	err = db.Ping()
	if err != nil {
		return err
	}
  
  // 数值根据业务情况决定
	db.SetConnMaxLifetime(time.Second *10)	// 超时连接
	db.SetMaxIdleConns(200) // 最大连接数
	db.SetMaxOpenConns(10) // 最大空闲连接数
  
	return nil
}
func main() {
	// 调用输出化数据库的函数
	if err := initDB();err != nil {
		fmt.Printf("init db failed,err:%v\n", err)
		return
	}
  // 在错误检查后 释放资源
	defer db.Close()
}
```

```go
// CRUD
type User struct {
	id uint
	name string
	age uint
}
// 单行查询 
func queryRowDemo()  {
	sqlStr := "select id,name,age from user where id=?"
	var u User
	row := db.QueryRow(sqlStr,1)
	err:=row.Scan(&u.id,&u.name,&u.age)
	if err !=nil {
		fmt.Printf("scan failed err:%v\n",err)
		return
	}
	fmt.Printf("%#v",u)
}
// 多行查询
func queryMultiRowDemo()  {
	sqlStr := "select id,name,age from user"
	rows,err :=db.Query(sqlStr)
	if err != nil {
		fmt.Println("query failed,err",err)
		return
	}
	defer rows.Close()
	var us []User
	for rows.Next()  {
		var u User
		if err:=rows.Scan(&u.id,&u.name,&u.age);err!=nil{
			return
		}
		us = append(us,u)
	}
	fmt.Println(us)
}
// 插入数据
func insertRowDemo()  {
	sqlStr := "insert into user(name,age) value (?,?)"
	ret,err := db.Exec(sqlStr,"小杨",29)
	if err!=nil {
		fmt.Printf("insert failed, err :%v\n",err)
		return
	}
	theID,err := ret.RowsAffected() // 新插入数据的id
	if err!=nil{
		fmt.Printf("get lastinsert ID failed, err:%v\n",err)
		return
	}
	fmt.Printf("intert success,the id is %d . \n",theID)
}
// 更新数据
func updateRowDemo()  {
	sqlStr := "update user set age=? where id = ?"
	ret,err:= db.Exec(sqlStr,39,1)
	if err!=nil {
		fmt.Printf("update failed,err:%v\n",err)
		return
	}
	n,err := ret.RowsAffected() // 操作影响行数 // 如果为0 ，则没有被更改
	if err!=nil {
		fmt.Printf("update rowsAffected failed,err:%v\n",err)
		return
	}
	fmt.Printf("update success ,affected rows:%d\n",n)
}
// 删除数据
func deleteRowDemo()  {
	sqlStr := "delete from user where id=?"
	ret,err:= db.Exec(sqlStr,3)
	if err!=nil {
		fmt.Printf("delete failed,err:%v\n",err)
		return
	}
	n,err:=ret.RowsAffected() // 操作影响行数 // 如果为0 ，则没有被删除
	if err!=nil {
		fmt.Printf("delete rowsAffected failed,err:%v\n",err)
		return
	}
	fmt.Printf("delete success,affected rows%d",n)
}
```

```go
// MySQL 预处理

// 普通SQL语句执行过程：
// 客户端对SQL语句进行占位符替换得到完整的SQL语句。
// 客户端发送完整SQL语句到MySQL服务端
// MySQL服务端执行完整的SQL语句并将结果返回给客户端。

// 预处理执行过程：
// 把SQL语句分成两部分，命令部分与数据部分。
// 先把命令部分发送给MySQL服务端，MySQL服务端进行SQL预处理。
// 然后把数据部分发送给MySQL服务端，MySQL服务端对SQL语句进行占位符替换。
// MySQL服务端执行完整的SQL语句并将结果返回给客户端。

// 为什么要预处理？
// 优化MySQL服务器重复执行SQL的方法，可以提升服务器性能，提前让服务器编译，一次编译多次执行，节省后续编译的成本。
// 避免SQL注入问题。

// 实现MySQL预处理器
// database/sql中使用下面的Prepare方法来实现预处理操作。
func (db *DB) Prepare(query string) (*Stmt, error)
// Prepare方法会先将sql语句发送给MySQL服务端，返回一个准备好的状态用于之后的查询和命令。返回值可以同时执行多个查询和命令。

// 查询操作的预处理器
func prepareQueryDemo()  {
	sqlStr := "select id,name,age from user where id> ?"
	stmt,err:= db.Prepare(sqlStr)
	if err!=nil {
		fmt.Printf("prepare failed,err:%v\n",err)
		return
	}
	defer stmt.Close()
	rows,err := stmt.Query(0)
	if err!=nil {
		fmt.Printf("prepare failed,err:%v\n",err)
	}
	var us []User
	for rows.Next()  {
		var u User
		if err:=rows.Scan(&u.id,&u.name,&u.age);err!=nil{
			return
		}
		us = append(us,u)
	}
	fmt.Println(us)
}
// 插入，更新和删除的预处理都类似
func prepareInsertDemo()  {
	sqlStr := "insert into user(name,age) values(?,?)"
	stmt,err:=db.Prepare(sqlStr)
	if err!=nil {
		fmt.Printf("prepare failed,err:%v\n",err)
		return
	}
	defer stmt.Close()
	_,err = stmt.Exec("小样",20)
	if err!=nil {
		fmt.Printf("prepare failed,err:%v\n",err)
		return
	}
	_,err = stmt.Exec("dx",12)
	if err!=nil {
		fmt.Printf("prepare failed,err:%v\n",err)
		return
	}
	fmt.Printf("insert success")
}
```

***避免SQL注入问题，我们任何时候都不应该自己拼接SQL语句！***

```go
// 不同的数据库中，SQL语句使用的占位符语法不尽相同。
// 数据库				占位符语法
// MySQL				?
// PostgreSQL		$1, $2等
// SQLite				? 和$1
// Oracle				:name
```

```go
// Go实现MySQL事务

// 什么是事物？
// 事务：一个最小的不可再分的工作单元；通常一个事务对应一个完整的业务(例如银行账户转账业务，该业务就是一个最小的工作单元)，同时这个完整的业务需要执行多次的DML(insert、update、delete)语句共同联合完成。A转账给B，这里面就需要执行两次update操作。
// 在MySQL中只有使用了Innodb数据库引擎的数据库或表才支持事务。事务处理可以用来维护数据库的完整性，保证成批的SQL语句要么全部执行，要么全部不执行。

// 事务的ACID
// 通常事务必须满足4个条件（ACID）：
// 原子性（Atomicity，或称不可分割性）: 一个事务（transaction）中的所有操作，要么全部完成，要么全部不完成，不会结束在中间某个环节。事务在执行过程中发生错误，会被回滚（Rollback）到事务开始前的状态，就像这个事务从来没有执行过一样。
// 一致性（Consistency）:在事务开始之前和事务结束以后，数据库的完整性没有被破坏。这表示写入的资料必须完全符合所有的预设规则，这包含资料的精确度、串联性以及后续数据库可以自发性地完成预定的工作。
// 隔离性（Isolation，又称独立性）:数据库允许多个并发事务同时对其数据进行读写和修改的能力，隔离性可以防止多个事务并发执行时由于交叉执行而导致数据的不一致。事务隔离分为不同级别，包括读未提交（Read uncommitted）、读提交（read committed）、可重复读（repeatable read）和串行化（Serializable）。
// 持久性（Durability）: 事务处理结束后，对数据的修改就是永久的，即便系统故障也不会丢失。

// 事务相关方法 
func (db *DB) Begin() (*Tx, error) // 开始事务
func (tx *Tx) Commit() error // 提交事务
func (tx *Tx) Rollback() error // 回滚事务
// 事务操作示例
func transactionDemo() {
	tx, err := db.Begin() // 开启事务
	if err != nil {
		if tx != nil {
			tx.Rollback() // 回滚
		}
		fmt.Printf("begin trans failed, err:%v\n", err)
		return
	}
	sqlStr1 := "Update user set age=30 where id=?"
	ret1, err := tx.Exec(sqlStr1, 2)
	if err != nil {
		tx.Rollback() // 回滚
		fmt.Printf("exec sql1 failed, err:%v\n", err)
		return
	}
	affRow1, err := ret1.RowsAffected()
	if err != nil {
		tx.Rollback() // 回滚
		fmt.Printf("exec ret1.RowsAffected() failed, err:%v\n", err)
		return
	}

	sqlStr2 := "Update user set age=40 where id=?"
	ret2, err := tx.Exec(sqlStr2, 3)
	if err != nil {
		tx.Rollback() // 回滚
		fmt.Printf("exec sql2 failed, err:%v\n", err)
		return
	}
	affRow2, err := ret2.RowsAffected()
	if err != nil {
		tx.Rollback() // 回滚
		fmt.Printf("exec ret1.RowsAffected() failed, err:%v\n", err)
		return
	}

	fmt.Println(affRow1, affRow2)
	if affRow1 == 1 && affRow2 == 1 {
		fmt.Println("事务提交啦...")
		tx.Commit() // 提交事务
	} else {
		tx.Rollback()
		fmt.Println("事务回滚啦...")
	}

	fmt.Println("exec trans success!")
}
```

