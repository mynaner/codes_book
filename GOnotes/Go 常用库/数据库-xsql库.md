# sqlx 库

>安装：go get github.com/jmoiron/sqlx

```go
// CRUD 示例
import (
	"fmt"
	_ "github.com/go-sql-driver/mysql"
	"g"
)
var db *sqlx.DB
func initDB() (err error) { 
	dsn := "root:dfanewer!k12AAA3@tcp(47.107.176.217:3306)/sql_test?charset=utf8&parseTime=True&loc=Local"
	db,err = sqlx.Connect("mysql",dsn)
	if err!=nil {
		return nil
	}
	db.SetMaxOpenConns(20)
	db.SetMaxIdleConns(10)
	return
}
type user struct {
	ID uint `db:"id"`
	Name string	`db:"name"`
	Age uint `db:"age"`
}
func main() {
	if err:=initDB();err !=nil {
		fmt.Printf("content db failed,err:%v\n",err)
		return
	}
	fmt.Println("content db success")
	defer db.Close() 
}

// 查询单行数据
func queryRowDemo()  {
	sqlStr := "select id,name,age from user where id=?"
	var u user
	err := db.Get(&u,sqlStr,1)
	if err != nil {
		fmt.Printf("get failed,err:%v\n",err)
		return
	}
	fmt.Println(u)
}
// 查询多行数据
func queryMultiRowDemo()  {
	sqlStr := "select id,name,age from user"
	var us []user
	err := db.Select(&us,sqlStr)
	if err != nil {
		fmt.Printf("get failed,err:%v\n",err)
		return
	}
	fmt.Println(us)
}
// 插入
func insertRowDemo()  {
	sqlStr := "insert into user(name,age) values (?,?)"
	ret,err := db.Exec(sqlStr,"一条小团团",26)
	if err!=nil {
		fmt.Printf("insert failed,1err:%v\n",err)
		return
	}
	theID,err := ret.RowsAffected()
	if err!=nil {
		fmt.Printf("insert failed,err:%v\n",err)
		return
	}
	fmt.Printf("insert success,the id is %d\n",theID)
}
// 更新
func updateRowDemo()  {
	sqlStr := "update user set age = ? where id=?"
	ret,err:=db.Exec(sqlStr,43,1)
	if err!=nil {
		fmt.Printf("update failed,err:%v\n",err)
		return
	}
	n,err:=ret.RowsAffected()
	if err!=nil {
		fmt.Printf("update failed,err:%v\n",err)
		return
	}
	fmt.Printf("update success,affected rows:%d\n",n)
}
// 删除
func deleteRowDemo()  {
	sqlStr := "delete from user where id = ?"
	ret,err:=db.Exec(sqlStr,2)
	if err!=nil {
		fmt.Printf("delete failed,err:%v\n",err)
		return
	}
	n,err:= ret.RowsAffected()
	if err!=nil {
		fmt.Printf("delete fialed,err:%v\n",err)
		return
	}
	fmt.Printf("delete success,affected rows:%d\n",n)
}

// NamedExec
// DB.NameExec 方法用来绑定SQL语句与结构体或map中的同名字段
func insertUserDemo()(err error)  {
	sqlStr := "insert into user(name ,age) value (:name,:age)"
	_,err =db.NamedExec(sqlStr,user{
		Name: "张三",
		Age:  24,
	})
	return
}
// NamedQuery 
func namedQuery()  {
	sqlStr := "select id,name,age from user where name = :name "
	rows,err := db.NamedQuery(sqlStr,map[string]interface{}{"name":"张三"})
	if err!=nil {
		fmt.Printf("db.NameQuery failed,err:%v\n",err)
		return
	}
	defer rows.Close()
	for rows.Next(){
		var u user
		err:=rows.StructScan(&u)
		if err!=nil {
			fmt.Printf("db.NameQuery failed,err:%v\n",err)
			continue
		}
		fmt.Println(u,"1")
	}
	u:=user{
		Name: "一条小团团",
	}
	rows,err = db.NamedQuery(sqlStr,u)
	if err!=nil {
		fmt.Printf("db.namequery failed,err:%v\n",err)
		return
	}
	fmt.Println("====================")
	defer rows.Close()
	for rows.Next()  {
		var u user
		err := rows.StructScan(&u)
		if err!=nil {
			fmt.Printf("db.NameQuery failed,err:%v\n",err)
			continue
		}
		fmt.Println(u)
	}
}


// 事物操作
// 对于事务操作，我们可以使用sqlx中提供的db.Beginx()和tx.Exec()方法
func transactionDemo2()(err error) {
	tx, err := db.Beginx() // 开启事务
	if err != nil {
		fmt.Printf("begin trans failed, err:%v\n", err)
		return err
	}
	defer func() {
		if p := recover(); p != nil {
			tx.Rollback()
			panic(p) // re-throw panic after Rollback
		} else if err != nil {
			fmt.Println("rollback")
			tx.Rollback() // err is non-nil; don't change it
		} else {
			err = tx.Commit() // err is nil; if Commit returns error update err
			fmt.Println("commit")
		}
	}()
	sqlStr1 := "Update user set age=20 where id=?"
	rs, err := tx.Exec(sqlStr1, 1)
	if err!= nil{
		return err
	}
	n, err := rs.RowsAffected()
	if err != nil {
		return err
	}
	if n != 1 {
		return errors.New("exec sqlStr1 failed")
	}
	sqlStr2 := "Update user set age=50 where i=?"
	rs, err = tx.Exec(sqlStr2, 5)
	if err!=nil{
		return err
	}
	n, err = rs.RowsAffected()
	if err != nil {
		return err
	}
	if n != 1 {
		return errors.New("exec sqlStr1 failed")
	}
	return err
}
```

```go
// sqlx.In

// 拼接语句实现批量插入
// 实现批量插入
func beachInsertUsers(user []user) (err error)  {
	// 存放 (?,?) 的slice
	valueStrings := make([]string,0,len(user))
	// 存放 values 的 slice
	valueArgs := make([]interface{},0,len(user)*2)
	// 遍历user准备相关数据
	for _,u:=range user {
		// 此处占位符要与插入的值个数对应
		valueStrings = append(valueStrings,"( ? , ? )")
		valueArgs = append(valueArgs,u.Name)
		valueArgs = append(valueArgs,u.Age)
	}
	// 拼接执行具体语句
	stmt := fmt.Sprintf("insert into user(name,age) values %s",strings.Join(valueStrings,","))
	fmt.Println(stmt)
	_,err = db.Exec(stmt,valueArgs...)
	return
}


// 使用 sqlx.In 实现批量插入
// 需要结构体实现 driver.Valuer 接口

func (u user)Value() (driver.Value,error)  {
	return []interface{}{u.Name,u.Age},nil
}

func beachInsertUsers2(users []interface{})(err error)  {
	query,args,_ := sqlx.In(
		"insert  into user(name,age) values (?),(?)", // 有几个参数就几个 (?)
		users...,  // 结构体 必须实现 driver.Valuer 接口
	)
	_,err = db.Exec(query,args...)
	fmt.Println(query,args,err)
	return
}

// BatchInsertUsers3 使用NamedExec实现批量插入
// 该功能目前有人已经推了 不知道 作者有没有发 release，测试的时候还在报错
func BatchInsertUsers3(users []*User) error {
	_, err := db.NamedExec("INSERT INTO user (name, age) VALUES (:name, :age)", users)
	log.Println(err)
	return err
}
```

```go
// sqlx.In的查询示例 

// QueryByIDs 根据给定ID查询
func QueryByIDs(ids []int)(users []User, err error){
	// 动态填充id
	query, args, err := sqlx.In("SELECT name, age FROM user WHERE id IN (?)", ids)
	if err != nil {
		return
	}
	// sqlx.In 返回带 `?` bindvar的查询语句, 我们使用Rebind()重新绑定它
	query = DB.Rebind(query)

	err = DB.Select(&users, query, args...)
	return
}

// in查询和FIND_IN_SET函数
// 查询id在给定id集合的数据并维持给定id集合的顺序。
// QueryAndOrderByIDs 按照指定id查询并维护顺序
func QueryAndOrderByIDs(ids []int)(users []User, err error){
	// 动态填充id
	strIDs := make([]string, 0, len(ids))
	for _, id := range ids {
		strIDs = append(strIDs, fmt.Sprintf("%d", id))
	}
	query, args, err := sqlx.In("SELECT name, age FROM user WHERE id IN (?) ORDER BY FIND_IN_SET(id, ?)", ids, strings.Join(strIDs, ","))
	if err != nil {
		return
	}

	// sqlx.In 返回带 `?` bindvar的查询语句, 我们使用Rebind()重新绑定它
	query = DB.Rebind(query)

	err = DB.Select(&users, query, args...)
	return
}
//  可以先使用IN查询，然后通过代码按给定的ids对查询结果进行排序。
```

