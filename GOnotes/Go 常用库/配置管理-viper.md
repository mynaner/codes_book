# viper 配置管理

>```bash
>go get github.com/spf13/viper
>```

```go
// viper 完整配置解决方案
// 能处理所有类型的配置需求和格式，持以下特性：
// * 设置默认值
// * 从JSON、TOML、YAML、HCL、envfile和Java properties格式的配置文件读取配置信息
// * 实时监控和重新读取配置文件（可选）
// * 从环境变量中读取
// * 从远程配置系统（etcd或Consul）读取并监控配置变化
// * 从命令行参数读取配置
// * 从buffer读取配置
// * 显式配置值

// Viper能够为你执行下列操作：
// *查找、加载和反序列化JSON、TOML、YAML、HCL、INI、envfile和Java properties格式的配置文件。
// *提供一种机制为你的不同配置选项设置默认值。
// *提供一种机制来通过命令行参数覆盖指定选项的值。
// *提供别名系统，以便在不破坏现有代码的情况下轻松重命名参数。
// *当用户提供了与默认值相同的命令行或配置文件时，可以很容易地分辨出它们之间的区别。

// Viper会按照下面的优先级。每个项目的优先级都高于它下面的项目:
// *显示调用Set设置值
// *命令行参数（flag）
// *环境变量
// *配置文件
// *key/value存储
// *默认值

// 重要： 目前Viper配置的键（Key）是大小写不敏感的
```

```go
// 存值 
// 默认值
viper.SetDefault("name", "张三")
// 读取配置文件
// 支持JSON、TOML、YAML、HCL、envfile和Java properties格式的配置文件
viper.SetConfigFile("./config.yaml") // 指定配置文件路径
viper.SetConfigName("config") // 配置文件名称(无扩展名)
viper.SetConfigType("yaml") // 如果配置文件的名称中没有扩展名，则需要配置此项
viper.AddConfigPath("/etc/appname/")   // 查找配置文件所在的路径
viper.AddConfigPath("$HOME/.appname")  // 多次调用以添加多个搜索路径
viper.AddConfigPath(".")               // 还可以在工作目录中查找配置
err := viper.ReadInConfig() // 查找并读取配置文件
if err != nil { // 处理读取配置文件的错误
	panic(fmt.Errorf("Fatal error config file: %s \n", err))
}
```





