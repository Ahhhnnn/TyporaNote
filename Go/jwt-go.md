# jwt-go入门

文档：https://pkg.go.dev/github.com/dgrijalva/jwt-go#readme-what-the-heck-is-a-jwt

JSON Web Tokens.

## 创建JWT

func NewWithClaims()

需要传入一个加密方法，一个Claims接口

**自定义MyCustomClaims结构体，加一个匿名参数（jwt.StandardClaims  实现了Claims接口）**

自定义Claims结构体结构体，可以加一些自定义参数等

```go
package main

import (
	"fmt"
	"github.com/dgrijalva/jwt-go"
)

type MyCustomClaims struct {
	Foo string `json:"foo"`
	// 匿名参数
	jwt.StandardClaims
}
func main() {
	mySigningKey := []byte("AllYourBase")
	// Create the Claims
	claims := MyCustomClaims{
		"bar",
		jwt.StandardClaims{
			ExpiresAt: 15000,
			Issuer:    "test",
		},
	}
	// 创建token
	token := jwt.NewWithClaims(jwt.SigningMethodHS256, claims)
	ss, err := token.SignedString(mySigningKey)
	fmt.Printf("%v %v", ss, err)
}

```

![image-20210427230429341](assets/image-20210427230429341.png)

直接使用预制的jwt.StandardClaims

```go
package main

import (
	"fmt"
	"github.com/dgrijalva/jwt-go"
)

func main() {
	mySigningKey := []byte("AllYourBase")

	// Create the Claims
  // 发现这里加不加 & 都成正常生成token 不知道为啥？？
	claims := &jwt.StandardClaims{
		ExpiresAt: 15000,
		Issuer:    "test",
	}
	token := jwt.NewWithClaims(jwt.SigningMethodHS256, claims)
	ss, err := token.SignedString(mySigningKey)
	fmt.Printf("%v %v", ss, err)
}

```



## 解析JWT

**使用ParseWithClaims() 解析有带参数的token**

解析预制的jwt.StandardClaims

```go
token, err = jwt.ParseWithClaims(ss, claims, func(token *jwt.Token) (interface{}, error) {
		return []byte("AllYourBase"), nil
	})
	fmt.Println()
	fmt.Println("token = ",token)
```

解析自定义Claims

```go
tokenString := "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJmb28iOiJiYXIiLCJleHAiOjE1MDAwLCJpc3MiOiJ0ZXN0In0.HE7fK0xOQwFEr4WDgRWj4teRPZ6i3GLwD5YCm6Pwu_c"

type MyCustomClaims struct {
	Foo string `json:"foo"`
	jwt.StandardClaims
}

// sample token is expired.  override time so it parses as valid
	token, err := jwt.ParseWithClaims(tokenString, &MyCustomClaims{}, func(token *jwt.Token) (interface{}, error) {
		return []byte("AllYourBase"), nil
	})

	if claims, ok := token.Claims.(*MyCustomClaims); ok && token.Valid {
		fmt.Printf("%v %v", claims.Foo, claims.StandardClaims.ExpiresAt)
	} else {
		fmt.Println(err)
```



## 实战使用

项目中一般用在中间件中。

一般在登录进入系统时校验用户成功后，颁发一个token给前端（设定过期时间等），前端存在缓存中，之后每次的请求都带上这个token放在Header中。后端拿到请求后，在中间件中统一解析token，判断是否有效、是否过期等操作。

```go
token := c.Request.Header.Get("x-token")
```

**自定义中间件**：

见gin-vue-admin项目中的jwt.go

初始化路由分组、使用中间件

自定义**Claims**

```go
type CustomClaims struct {
	UUID        uuid.UUID
	ID          uint
	Username    string
	NickName    string
	AuthorityId string
	BufferTime  int64
	jwt.StandardClaims
}
```

```go
// 分组
PrivateGroup := Router.Group("")
// use 中间件，让所有的请求都走jwt校验
PrivateGroup.Use(middleware.JWTAuth()).Use(middleware.CasbinHandler())
```

