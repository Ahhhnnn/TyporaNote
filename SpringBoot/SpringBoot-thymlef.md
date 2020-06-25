# SpringBoot-Thymeleaf

## 引入Thymeleaf

在pom文件中因为thymeleaf依赖

```xml
<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

Springboot整合thymeleaf配置(一般配置spring.thymeleaf.cache=false即可)

```properties
#spring.thymeleaf.suffix=.html
#spring.thymeleaf.mode=HTML5
#spring.thymeleaf.encoding=UTF-8
# ;charset=<encoding> is added
#spring.thymeleaf.content-type=text/html
# set to false for hot refresh

spring.thymeleaf.cache=false
```

在html文件中因为thymeleaf约束

```html
<html lang="en" xmlns:th="http://www.thymeleaf.org">
```

创建User实体

```java
@Data
public class User {
	private String name;
	private String password;
}
```

在Controller层返回ModelAndView用于thmeleaf模板渲染

```java
@GetMapping(value = {"", "/"})
	public ModelAndView index(HttpServletRequest request) {
		ModelAndView mv = new ModelAndView();

		User user = (User) request.getSession().getAttribute("user");
		if (ObjectUtil.isNull(user)) {
			mv.setViewName("redirect:/user/login");
		} else {
			mv.setViewName("page/index");
			mv.addObject(user);
		}
		return mv;
	}
```

也可以不使用ModalAndView ，直接返回String，内容为转发或重定向的地址，上下文的内容使用一个Map存储

```java
@GetMapping("/test")
    public ModelAndView test(Map<String,Object> map){
	    ModelAndView mv = new ModelAndView();
	    map.put("user","hening");
	    mv.setViewName("page/index");
	    mv.addObject(map);
	    return mv;
    }
```

由于在ModelAndView中直接返回了User对象，在thmeleaf中可以直接操作这个对象获取属性的值

```html
<!doctype html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<header th:replace="~{common/head :: header}"></header>
<body>
<div id="app" style="margin: 20px 20%">
	欢迎登录，<span th:text="${user.name}"></span>！
</div>
</body>
</html>
```



## Thymeleaf 常用标签

### *th:fragment*

定义用于加载的模块

```html
<span th:fragment="pagination"> 
the public pagination
</span>
```

### th：**replace**,th:include



```html

<!-- 加载模板的内容： 读取加载节点的内容（不含节点名称），替换<div>的内容 -->
<div th:include="pagination::pagination">1</div>
<!-- 替换当前标签为模板中的标签： 加载的节点会整个替换掉加载他的<div>  -->
<div th:replace="pagination::pagination">2</div>
```

结果如下

```html
<!-- 加载模板的内容： 读取加载节点的内容（不含节点名称），替换<div>的内容 -->
<div> the public pagination</div>
<!-- 替换当前标签为模板中的标签： 加载的节点会整个替换掉加载他的<div>  -->
<span> the public pagination</span>
```

参数传递

```html
<div th:replace="commons/bar::#top"></div>
		<div th:replace="commons/bar::#sidebar(currUri='index',statistics = ${statistics})"></div>
```

参数接受

```html
<li th:class="${currUri == 'index'?'active':''}">
                <a href="index"><i class="ion ion-speedometer"></i><span>使用情况</span></a>
</li>
<li class="menu-header">所有文件</li>
<li th:class="${currUri == 'all'?'active':''}">
    <a href="files"><i class="ion ion-folder"></i> 全部文件</a>
</li>
<li th:class="${currUri == 'upload'?'active':''}">
    <a href="upload"><i class="ion ion-upload"></i> 上传文件</a>
</li>
```

