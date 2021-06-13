# Viper TODO

Viper是适用于Go应用程序的完整配置解决方案。它旨在在应用程序中工作。

viper 是一个配置解决方案，拥有丰富的特性：

- 支持 JSON/TOML/YAML/HCL/envfile/Java properties 等多种格式的配置文件；
- 可以设置监听配置文件的修改，修改时自动加载新的配置；
- 从环境变量、命令行选项和`io.Reader`中读取配置；
- 从远程配置系统中读取和监听修改，如 etcd/Consul；
- 代码逻辑中显示设置键值。



## 设置默认值

viper.SetDefault()

```go
package main

import (
	"fmt"
	"github.com/spf13/viper"
)

func main() {
	viper.SetDefault("ContentDir", "content")
	viper.SetDefault("LayoutDir", "layouts")
	viper.SetDefault("Taxonomies", map[string]string{"tag": "tags", "category": "categories"})
	ContentDir := viper.Get("ContentDir")
	fmt.Println(ContentDir)
	fmt.Println(viper.Get("LayoutDir"))
}
```

## 读取配置文件



```go

```



### 忽略读取错误

如果没有读取到配置文件，那么可以设置忽略错误，继续向下走



## 写配置文件

