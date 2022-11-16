

# zap 日志库

```go
// 日志
// 能够将事件记录到文件中，而不是应用程序控制台。
// 日志切割-能够根据文件大小、时间或间隔等来切割日志文件。
// 支持不同的日志级别。例如INFO，DEBUG，ERROR等。
// 能够打印基本信息，如调用文件/函数名和行号，日志时间等。

// 默认的Go Logger
// 优势: 它最大的优点是使用非常简单。我们可以设置任何io.Writer作为日志记录输出并向其发送要写入的日志。
// 劣势:
//		仅限基本的日志级别:只有一个Print选项。不支持INFO/DEBUG等多个级别。
// 		对于错误日志，它有Fatal和Panic ,Fatal日志通过调用os.Exit(1)来结束程序,Panic日志在写入日志消息之后抛出一个panic,但是它缺少一个ERROR日志级别，这个级别可以在不抛出panic或退出程序的情况下记录错误
// 		缺乏日志格式化的能力——例如记录调用者的函数名和行号，格式化日期和时间格式。等等。
// 		不提供日志切割的能力。 
```

# Uber-go Zap
>```bash
>go get -u go.uber.org/zap
>```

```go
// 它同时提供了结构化日志记录和printf风格的日志记录
// 它非常的快

// Zap 提供了两种类型的日志记录器 
// SugaredLogger:
// 			在性能很好但不是很关键的上下文中，使用SugaredLogger
// 			它比其他结构化日志记录包快4-10倍，并且支持结构化和printf风格的日志记录。
// Logger :
//			在每一微秒和每一次内存分配都很重要的上下文中，使用Logger
// 			它甚至比SugaredLogger更快，内存分配次数也更少，但它只支持强类型的结构化日志记录。
```

```go
// Logger 
// 通过调用zap.NewProduction() / zap.NewDevelopment() 或者 zap.Example 创建一个Logger
// 上面每个函数都将创建一个logger，唯一区别在于它将记录的信息不同。
// 通过Logger 调用Info/Error 等
// 默认情况下日志都会打印到应用程序等console界面 
var logger *zap.Logger 
func simpleHttpGet(url string)  {
	resp,err:=http.Get(url)
	if err!=nil {
		logger.Error("Error fetching url ..",zap.String("url",url),zap.Error(err))
		return
	}
	logger.Info("success..",zap.String("url",url),zap.String("statusCode",resp.Status))
	resp.Body.Close()
}
func InitLogger()  {
	logger,_ = zap.NewProduction()
}
func main() {
	InitLogger()
	defer logger.Sync()
	simpleHttpGet("http://www.baidu.com")
	simpleHttpGet("www.baidu.com")
}
// Sugared Logger 
// * 大部分与 Logger 实现相同
// * 唯一区别是，我们通过调用logger的.Sugar() 方法来获取一个 SugaredLoger
// * 然后使用 SugaredLogger 以 printf 格式记录语句
var sugarLogger *zap.SugaredLogger
 func simpleHttpGet(url string)  {
	sugarLogger.Debugf("Trying to hit GET request for %s",url)
	resp,err:=http.Get(url)
	if err!=nil { 
		sugarLogger.Errorf("Error fetching URL %s : Error = %s",url,err)
		return
	}
	sugarLogger.Infof("success ! statusCode = %s for URL %s️",resp.Status,url) 
	resp.Body.Close()
}
func InitLogger()  {
	logger,_ := zap.NewProduction()
	sugarLogger = logger.Sugar()
}
func main() {
	InitLogger()
	defer sugarLogger.Sync()
	simpleHttpGet("http://www.baidu.com")
	simpleHttpGet("www.baidu.com")
}
```

```go
// 定制logger
// 将日志写入文件而不是终端
// * 使用zap.New(...) 方法手动传递所有配置 来创建logger
func  New(core zapcore.Core, options ...Option) *Logger
// zapcore.Core 需要三个配置
// * Encoder : 编码器(如何写入日志)。使用 NewJSONEncoder,并使用预先设置的 ProductionEncderConfig()
zapcore.NewJSONEncoder(zap.NewProductionEncoderConfig())
// * WriterSyncer : 指定日志写到哪里去，使用zapcore.AddSynnc() 函数并将打开的文件句柄传进去
file,_:=os.Create("./test.log")
writeSyncer := zapcore.AddSync(file)
// * LogLevel : 那种级别的日志将被写入

// 示例
var logger *zap.Logger
var sugarLogger *zap.SugaredLogger
func simpleHttpGet(url string)  {
	sugarLogger.Debugf("Trying to hit GET request for %s",url)
	resp,err:=http.Get(url)
	if err!=nil { 
		sugarLogger.Errorf("Error fetching URL %s : Error = %s",url,err)
		return
	}
	sugarLogger.Infof("success ! statusCode = %s for URL %s️",resp.Status,url) 
	resp.Body.Close()
}
func InitLogger()  {
	writeSyncer := getLogwriter()
	encoder := getEncoder()
	core := zapcore.NewCore(encoder,writeSyncer,zapcore.DebugLevel)
  // zap.AddCaller() 将调用的函数信息 记录到日志中
	logger := zap.New(core,zap.AddCaller())
	sugarLogger = logger.Sugar()
}
func getEncoder() zapcore.Encoder  {
	// 拿取配置 
	encoderConfig := zap.NewProductionEncoderConfig()
  // 修改时间编码器
	encoderConfig.EncodeTime = zapcore.ISO8601TimeEncoder
  // 在日志文件中使用大写字母记录日志级别
	encoderConfig.EncodeLevel = zapcore.CapitalColorLevelEncoder 
  // zapcore.NewJSONEncoder()  // json Encoder
	// zapcore.NewConsoleEncoder() //  普通Encoder
	return zapcore.NewConsoleEncoder(encoderConfig) 
}
func getLogwriter()zapcore.WriteSyncer  {
	file,_ := os.OpenFile("./test.log",os.O_CREATE|os.O_APPEND|os.O_RDWR,0744)
	return zapcore.WriteSyncer(file)
}
func main() {
	InitLogger()
	defer sugarLogger.Sync()
	simpleHttpGet("http://www.baidu.com")
	simpleHttpGet("www.baidu.com")
}
```
# Lumberjack(切割归档日志文件 )

>```bash
>go get -u github.com/natefinch/lumberjack
>// zap 本身不支持切割归档日志文件 
>```

```go
func getLogwriter()zapcore.WriteSyncer  {
	lumberjackLogger := &lumberjack.Logger{
		Filename:   "./test.log", 	// 日志位置
		MaxSize:    1,				// 在切割之前，日志文件到最大大小
		MaxAge:     5,				// 保留旧文件的最大天数
		MaxBackups: 30,				// 保留旧文件的最大个数
		LocalTime:  false,			// 切割时间是否使用计算机本地时间，默认使用UTC时间
		Compress:   false,			// 是否压缩/归档旧文件
	}
	return zapcore.AddSync(lumberjackLogger)
}
```
# 在gin中使用zap
```go
// gin 默认的中间件 
// 在使用 gin.Default() 的同时用到了gin框架内的两个默认中间件 Logger() 和 Recovery()
// Logger() : 日志输出
// Recovery() 在程序出现 panic 的时候恢复，并返回500

// 基于zap的中间件

// GinLogger 接收gin框架默认的日志
func GinLogger(logger *zap.SugaredLogger) gin.HandlerFunc {
	return func(c *gin.Context) {
		start := time.Now()
		path := c.Request.URL.Path
		query := c.Request.URL.RawQuery
		c.Next()
		cost := time.Since(start)
		logger.Info(path,
			zap.Int("status", c.Writer.Status()),
			zap.String("method", c.Request.Method),
			zap.String("path", path),
			zap.String("query", query),
			zap.String("ip", c.ClientIP()),
			zap.String("user-agent", c.Request.UserAgent()),
			zap.String("errors", c.Errors.ByType(gin.ErrorTypePrivate).String()),
			zap.Duration("cost", cost),
		)
	}
}
// GinRecovery recover掉项目可能出现的panic
func GinRecovery(logger *zap.SugaredLogger, stack bool) gin.HandlerFunc {
	return func(c *gin.Context) {
		defer func() {
			if err := recover(); err != nil {
				// Check for a broken connection, as it is not really a
				// condition that warrants a panic stack trace.
				var brokenPipe bool
				if ne, ok := err.(*net.OpError); ok {
					if se, ok := ne.Err.(*os.SyscallError); ok {
						if strings.Contains(strings.ToLower(se.Error()), "broken pipe") || strings.Contains(strings.ToLower(se.Error()), "connection reset by peer") {
							brokenPipe = true
						}
					}
				}
				httpRequest, _ := httputil.DumpRequest(c.Request, false)
				if brokenPipe {
					logger.Error(c.Request.URL.Path,
						zap.Any("error", err),
						zap.String("request", string(httpRequest)),
					)
					// If the connection is dead, we can't write a status to it.
					c.Error(err.(error)) // nolint: errcheck
					c.Abort()
					return
				}
				if stack {
					logger.Error("[Recovery from panic]",
						zap.Any("error", err),
						zap.String("request", string(httpRequest)),
						zap.String("stack", string(debug.Stack())),
					)
				} else {
					logger.Error("[Recovery from panic]",
						zap.Any("error", err),
						zap.String("request", string(httpRequest)),
					)
				}
				c.AbortWithStatus(http.StatusInternalServerError)
			}
		}()
		c.Next()
	}
}
```

```go
// 别人封装好的： https://github.com/gin-contrib/zap 
import (
	"github.com/gin-contrib/zap"
	"github.com/gin-gonic/gin"
	"github.com/natefinch/lumberjack"
	"go.uber.org/zap"
	"go.uber.org/zap/zapcore"
	"net/http"
	"time"
)

var logger *zap.Logger

func InitLogger() {
	writeSyncer := getLogwriter()
	encoder := getEncoder()
	core := zapcore.NewCore(encoder, writeSyncer, zapcore.DebugLevel)
	// zap.AddCaller() 将调用的函数信息 记录到日志中
	logger = zap.New(core, zap.AddCaller())
}
func getEncoder() zapcore.Encoder {
	// 拿取配置
	encoderConfig := zap.NewProductionEncoderConfig()
	// 修改时间编码器
	encoderConfig.EncodeTime = zapcore.ISO8601TimeEncoder
	// 在日志文件中使用大写字母记录日志级别
	encoderConfig.EncodeLevel = zapcore.CapitalColorLevelEncoder
	// zapcore.NewJSONEncoder()  // json Encoder
	// zapcore.NewConsoleEncoder() //  普通Encoder
	return zapcore.NewConsoleEncoder(encoderConfig)
}
func getLogwriter() zapcore.WriteSyncer {
	lumberjackLogger := &lumberjack.Logger{
		Filename:   "./test.log", // 日志位置
		MaxSize:    1,            // 在切割之前，日志文件到最大大小
		MaxAge:     5,            // 保留旧文件的最大天数
		MaxBackups: 30,           // 保留旧文件的最大个数
		LocalTime:  false,        // 切割时间是否使用计算机本地时间，默认使用UTC时间
		Compress:   false,        // 是否压缩/归档旧文件
	}
	return zapcore.AddSync(lumberjackLogger)
}
func main() {
	InitLogger()
	r := gin.New()
	//logger, _ := zap.NewProduction()
	r.Use(ginzap.Ginzap(logger, time.RFC3339, true))
	r.GET("/hello", func(c *gin.Context) {
		c.String(http.StatusOK, "hello world")
	})
	r.Run()
}
```

### 在gin框架中使用

```go
package logger

import (
	"github.com/gin-gonic/gin"
	"github.com/natefinch/lumberjack"
	"go.uber.org/zap"
	"go.uber.org/zap/zapcore"
	"net"
	"net/http"
	"net/http/httputil"
	"os"
	"runtime/debug"
	"strings"
	"time"
	"web_app/settings"
)

func Init(conf *settings.LogConfig,mode string) (err error) {
	writeSyncer := getLogWriter(
		conf.FileName,
		conf.MaxSize,
		conf.MaxBackups,
		conf.MaxAge,
	)
	encoder := getEncoder()
	var l = new(zapcore.Level)
	err = l.UnmarshalText([]byte(conf.Level))
	if err != nil {
		return
	}
	var core zapcore.Core
	if mode == "dev" {
		consoleEncoder:=zapcore.NewConsoleEncoder(zap.NewDevelopmentEncoderConfig())
		core = zapcore.NewTee(zapcore.NewCore(encoder,writeSyncer,l),zapcore.NewCore(consoleEncoder,zapcore.Lock(os.Stdout),zapcore.DebugLevel))
	}else {
		core = zapcore.NewCore(encoder, writeSyncer, l)
	}
	lg := zap.New(core, zap.AddCaller())
	zap.ReplaceGlobals(lg)
	return
}

func getEncoder() zapcore.Encoder {
	config := zap.NewProductionEncoderConfig()
	config.EncodeTime = zapcore.ISO8601TimeEncoder
	config.TimeKey = "time"
	config.EncodeLevel = zapcore.CapitalLevelEncoder
	config.EncodeDuration = zapcore.SecondsDurationEncoder
	config.EncodeCaller = zapcore.ShortCallerEncoder
	return zapcore.NewJSONEncoder(config)
}

func getLogWriter(filename string, maxSize, maxBackUp, maxAge int) zapcore.WriteSyncer {
	lumberjackLogger := &lumberjack.Logger{
		Filename:   filename,
		MaxSize:    maxSize,
		MaxAge:     maxAge,
		MaxBackups: maxBackUp,
	}
	return zapcore.AddSync(lumberjackLogger)
}

// GinLogger 接收gin框架默认的日志
func GinLogger() gin.HandlerFunc {
	return func(c *gin.Context) {
		start := time.Now()
		path := c.Request.URL.Path
		query := c.Request.URL.RawQuery
		c.Next()
		cost := time.Since(start)
		zap.L().Info(path,
			zap.Int("status", c.Writer.Status()),
			zap.String("method", c.Request.Method),
			zap.String("path", path),
			zap.String("query", query),
			zap.String("ip", c.ClientIP()),
			zap.String("user-agent", c.Request.UserAgent()),
			zap.String("errors", c.Errors.ByType(gin.ErrorTypePrivate).String()),
			zap.Duration("cost", cost),
		)
	}
}

// GinRecovery recover掉的项目可能出现panic,并使用zap记录相关日志
func GinRecovery(stack bool) gin.HandlerFunc {
	return func(c *gin.Context) {
		defer func() {
			if err := recover(); err != nil {
				// Check for a broken connection, as it is not really a
				// condition that warrants a panic stack trace.
				var brokenPipe bool
				if ne, ok := err.(*net.OpError); ok {
					if se, ok := ne.Err.(*os.SyscallError); ok {
						if strings.Contains(strings.ToLower(se.Error()), "broken pipe") || strings.Contains(strings.ToLower(se.Error()), "connection reset by peer") {
							brokenPipe = true
						}
					}
				}

				httpRequest, _ := httputil.DumpRequest(c.Request, false)
				if brokenPipe {
					zap.L().Error(c.Request.URL.Path,
						zap.Any("error", err),
						zap.String("request", string(httpRequest)),
					)
					// If the connection is dead, we can't write a status to it.
					c.Error(err.(error)) // nolint: errcheck
					c.Abort()
					return
				}

				if stack {
					zap.L().Error("[Recovery from panic]",
						zap.Any("error", err),
						zap.String("request", string(httpRequest)),
						zap.String("stack", string(debug.Stack())),
					)
				} else {
					zap.L().Error("[Recovery from panic]",
						zap.Any("error", err),
						zap.String("request", string(httpRequest)),
					)
				}
				c.AbortWithStatus(http.StatusInternalServerError)
			}
		}()
		c.Next()
	}
}

```

