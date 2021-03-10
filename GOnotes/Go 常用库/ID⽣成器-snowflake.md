# 分布式**ID**⽣成器

```go
// 全局唯⼀性：不能出现有重复的ID标识，这是基本要求。
// 递增性：确保⽣成ID对于⽤户或业务是递增的。
// ⾼可⽤性：确保任何时候都能⽣成正确的ID。
// ⾼性能性：在⾼并发的环境下依然表现良好。
```

###  snowflflake算法


```go
snowflflake-64bit
+--------------------------------------------------------------------------+
| 1 Bit Unused | 41 Bit Timestamp |  10 Bit NodeID  |   12 Bit Sequence ID |
+--------------------------------------------------------------------------+
// 第⼀位 占⽤1bit，其值始终是0，没有实际作⽤。
// 时间戳 占⽤41bit，单位为毫秒，总共可以容纳约69年的时间。
// ⼯作机器id 占⽤10bit，其中⾼位5bit是数据中⼼ID，低位5bit是⼯作节点ID，最多可以容纳1024个节点。
// 序列号 占⽤12bit，⽤来记录同毫秒内产⽣的不同id。每个节点每毫秒0开始不断累加，最多可以累加到4095，同⼀毫秒⼀共可以产⽣4096个ID。
// SnowFlake算法在 同⼀毫秒的ID数量 = 1024 X 4096 = 4194304
```

### snowflflake 的Go实现

```
> go get github.com/bwmarrin/snowflake
```

```go
var node *snowflake.Node
func Init(startTime string, machineID int64) (err error) {
	fmt.Println(startTime, machineID)
	var st time.Time
	st, err = time.Parse("2006-01-02", startTime)
	if err != nil {
		return
	}
	snowflake.Epoch = st.UnixNano() / 1000000
	node, err = snowflake.NewNode(machineID)
	return
}
func GenID() int64 {
	return node.Generate().Int64()
}
```

