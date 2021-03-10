

# JWT

> github.com/dgrijalva/jwt-go

```go
// 在以前的项目中，通常使用的是Cookie-Session模式实现用户认证。
// 相关流程大致如下：
// 1，用户在浏览器端填写用户名和密码，并发送给服务端
// 2，服务端对用户名和密码校验通过后会生成一份保存当前用户相关信息的session数据和一个与之对应的标识（通常称为session_id）
// 3，服务端返回响应时将上一步的session_id写入用户浏览器的Cookie
// 4，后续用户来自该浏览器的每次请求都会自动携带包含session_id的Cookie
// 5，服务端通过请求中的session_id就能找到之前保存的该用户那份session数据，从而获取该用户的相关信息。

// 这种方案依赖于客户端（浏览器）保存Cookie，并且需要在服务端存储用户的session数据。


// 在移动互联网时代，我们的用户可能使用浏览器也可能使用APP来访问我们的服务，我们的web应用可能是前后端分开部署在不同的端口，有时候我们还需要支持第三方登录，这下Cookie-Session的模式就有些力不从心了。


// JWT就是一种基于Token的轻量级认证模式，服务端认证通过后，会生成一个JSON对象，经过签名后得到一个Token（令牌）再发回给用户，用户后续请求只需要带上这个Token，服务端解密之后就能获取该用户的相关信息了。
```

## 生成JWT和解析JWT

```go
package jwt
import (
	"errors"
	"github.com/dgrijalva/jwt-go"
	"time"
)

// 定义JWT的过期时间，这里以2小时为例：
const TokenExpireDuration = time.Hour * 2

// 要定义Secret
var mySecret = []byte("天涯明月刀")

// MyClaims 自定义声明结构体并内嵌jwt.StandardClaims
// jwt包自带的jwt.StandardClaims只包含了官方字段
// 我们这里需要额外记录一个username字段，所以要自定义结构体
// 如果想要保存更多信息，都可以添加到这个结构体中
type MyClaims struct {
	UserID   int64  `json:"user_id"`
	Username string `json:"username"`
	jwt.StandardClaims
}

// GenToken 生成JWT
func GenToken(userID int64, username string) (string, error) {
  // // 创建一个我们自己的声明
	c := MyClaims{
		UserID:   userID, // 自定义字段
		Username: username, // 自定义字段
		StandardClaims: jwt.StandardClaims{
			ExpiresAt: time.Now().Add(TokenExpireDuration).Unix(), //过期时间
			Issuer:    "clooe",                                    // 签发人 可以是app名称
		},
	}
	// 使用指定刀签名方法创建签名对象
	token := jwt.NewWithClaims(jwt.SigningMethodHS256, c)
	// 使用指定刀mySecret 签名并获得完整刀编码后刀字符串token
	return token.SignedString(mySecret)
}


// ParseToken 解析token
func ParseToken(tokenString string) (*MyClaims, error) {
	// 解析token
	var mc = new(MyClaims)
	token, err := jwt.ParseWithClaims(tokenString, mc, func(token *jwt.Token) (i interface{}, err error) {
		return mySecret, nil
	})
	if err != nil {
		return nil, err
	}
	if token.Valid {
		return mc, nil
	}
	return nil, errors.New("invalid token")
}
```



# jwt中间件(在gin中使用)

```go
package middlewares
import (
	"errors"
	"github.com/gin-gonic/gin"
	"go.uber.org/zap"
	"strings"
	"web_app/pkg/jwt" // 这是引用上面定义的jwt包
	"web_app/pkg/response" // gin 自定义的返回响应
)
const CtxUserIDKey = "userID"
// JWTAuthMiddleware 基于JWT的认证中间件
func JWTAuthMiddleware() func(c *gin.Context) {
	return func(c *gin.Context) {
		// 客户端携带Token放在请求头 Header 的 Authorization 中，并使用Bearer开头
		authHeader := c.Request.Header.Get("Authorization")
		// 空格分割
		parts := strings.SplitN(authHeader, " ", 2)
		// parts 长度为2 parts[0] 为 Bearer
		if !(len(parts) == 2 && parts[0] == "Bearer") {
			zap.L().Error("Permission Denied", zap.Error(errors.New("没有权限")))
			response.ResponseUnauthorizedError(c)
			c.Abort()
			return
		}
		// parts[1] 为token字段 ，解析
		mc, err := jwt.ParseToken(parts[1])
		if err != nil {
			zap.L().Error("Permission Denied", zap.Error(err))
			response.ResponseUnauthorizedError(c)
			c.Abort()
			return
		}
		// 将请求的信息保存到上下文中
		c.Set(CtxUserIDKey, mc.UserID)
		c.Next()
	}
}

```

