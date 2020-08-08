# VUE

## 直接引入

```js
<!-- 开发环境版本，包含了有帮助的命令行警告 -->
<script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
```

## npm安装

先安装了npm镜像cnpm

```
npm install -g cnpm --registry=https://registry.npm.taobao.org
```

```shel
# 全局安装 vue-cli
$ cnpm install --global vue-cli
# 创建一个基于 webpack 模板的新项目
$ vue init webpack my-project
```

然后一路回车即可

运行vue项目

```she
$ cd my-project
$ cnpm install
$ cnpm run dev
 DONE  Compiled successfully in 4388ms
 > Listening at http://localhost:8080
```

如果npm install 失败了 ，那么删除掉node_modules文件夹，然后重新执行npm install

有时候会报错找不到node-sass，执行以下命令

**cnpm install node-sass --save**

## VUE项目目录结构



## VUE挂载

- vue会管理el选项中命中的元素及内部的子元素
- 可以其他选择器，但是建议使用id选择器（#app）
- 可以使用其他的双标签，但是不能用html和body，建议使用div，因为没有特殊的样式

```html
<div id="app">
  <p>{{ message }}</p>
  <span>{{message}}</span>
  <span>{{message}}</span>
</div>

<!-- 开发环境版本，包含了有帮助的命令行警告 -->
<script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
<script>
    new Vue({
        el: "#app",
        data:{
            message : "hening"
        }
    })
</script>
```

## data数据对象

- Vue中用到的数据都定义在data中
- data中可以定义复杂数据类型，如Array，对象，String
- 在页面渲染时，需要遵循js语法

```html
<div id="app">
  <p>{{ message }}</p>
    <p>{{person}}</p>
    <p>{{place}}</p>
    <span>{{person.name}}</span> <span>{{person.sex}}</span> <span>{{person.phone}}</span>
    <ul>
        <li>{{place[0]}}</li>
        <li>{{place[1]}}</li>
        <li>{{place[2]}}</li>
    </ul>
</div>

<!-- 开发环境版本，包含了有帮助的命令行警告 -->
<script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
<script>
    new Vue({
        el: "#app",
        data:{
            message : "hening",
            person:{
                name:'hening',
                sex:'man',
                phone:'13699448253'
            },
            place:['成都','北京','上海']
        }
    })
</script>
```



## VUE  v-text 标签

v-text 可以设置标签的内容

默认写法会替换全部的内容，如果需要拼接参数，可以在内部使用单引号拼接，如果是使用{{message}}方式，可以直接使用双引号拼接

```html
<div id="app">
   <p>{{ message }}</p>
    <p v-text="message"></p>
    <h1 v-text="person.name"></h1>
    <h1 v-text="person.sex"></h1>
    <ul>
        <li v-text="place[0]"></li>
        <li v-text="place[1]"></li>
        <li v-text="place[2]"></li>
    </ul>
</div>

<!-- 开发环境版本，包含了有帮助的命令行警告 -->
<script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
<script>
    new Vue({
        el: "#app",
        data:{
            message : "hening",
            person:{
                name:'hening',
                sex:'man',
                phone:'13699448253'
            },
            place:['成都','北京','上海']
        }
    })
</script>
```

## VUE v-html指令

v-html指定作用是这是元素的innerHTML

如果内容有html代码，那么会被解析为标签

```html
<div id="app">
   <h1 v-text="message"></h1>
   <h1 v-html="message"></h1>
</div>

<!-- 开发环境版本，包含了有帮助的命令行警告 -->
<script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
<script>
    new Vue({
        el: "#app",
        data:{
            message : "<a href='https://www.baidu.com'>baidu</a>"
        }
    })
</script>
```

## VUE v-on指令

v-on指令用于绑定事件

v-on可以简写为@事件名（如@click=“方法名”）

```html
<div id="app">
   <button v-on:click="dothing">点击事件</button>
   <button @click="dothing">点击事件简写</button>
   <button @dblclick="dblclickthing">双击事件</button>
</div>

<!-- 开发环境版本，包含了有帮助的命令行警告 -->
<script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
<script>
    new Vue({
        el: "#app",
        data:{
            message : "何宁好帅"
        },
        methods: {
            dothing:function(){
                alert("点击事件触发");
            },
            dblclickthing:function(){
                alert("双击事件触发");
            }
        }
    })
</script>
```

## VUE 监听键盘事件

@keyup.enter 监听enter键按下的时间



## VUE实现计数器 小应用

```html
<div id="app">
   <button @click="sub">
        -
   </button>

   <span>
        {{num}}
   </span>
   <button @click="add">
       +
   </button>
</div>

<!-- 开发环境版本，包含了有帮助的命令行警告 -->
<script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
<script>
    var app = new Vue({
        el: "#app",
        data:{
            num : 0
        },
        methods: {
            add:function(){
                console.log("add");
                if(this.num<10){
                    this.num++
                }else{
                    alert("不能大于10");
                }
            },
            sub:function(){
                console.log("sub");
                if(this.num>0){
                    this.num--
                }else{
                    alert("不能小于0");
                }
            }
        }
    })
</script>
```

## VUE v-show 指令

v-show 指令是根据真假切换元素的显示

原理是修改元素的display属性

指令里内容 实际上是布尔值

```html
<div id="app">
    <button @click="changeB">修改</button>
   <h1 v-show="isshow">这是一个标题用于测试v-show指令</h1>
   <h2 v-show="age>18">这是用于测试v-show指令里面的表达式</h2>
</div>

<!-- 开发环境版本，包含了有帮助的命令行警告 -->
<script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
<script>
    var app = new Vue({
        el: "#app",
        data:{
            isshow :false,
            age:15
        },
        methods: {
            changeB:function(){
                this.isshow =!this.isshow
            }
        }
    })
</script>
```

## VUE v-if指令

v-if根据表达式切换元素的显示和隐藏

原理 操作dom元素，隐藏时直接移除了dom元素

栗子和上面v-show差不多

## VUE v-bind指令

v-bind指令作用是为元素绑定属性

完整语法是v-bind：属性名 如v-bind：src，v-bind：class

简写语法是：属性名，如：src，：class

设置class时可以使用三元表达式或者对象的写法

三元表达式： :class="isactive?'activeclass':''"

简写：:class="{activeclass:isactive}" 意思是activeclass是否生效取决于isactive的值

```html
<style>
    .activeclass{
        border: 1px solid red;
    }
</style>
<div id="app">
   <a :href="baidu">这是一个超链接</a>
    <img :src="imageurl" :class="isactive?'activeclass':''" alt="">
    <img :src="imageurl" :class="{activeclass:isactive}" alt="">
</div>

<!-- 开发环境版本，包含了有帮助的命令行警告 -->
<script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
<script>
    var app = new Vue({
        el: "#app",
        data:{
            baidu:"https://www.baidu.com",
            isactive:false,
            imageurl:"/img/jiumi.jpg"
        },
        methods: {
            toogleactive:function(){
                this.active=!this.active
            }
        }
    })
</script>
```

### bind绑定class和style

可以使用字符串、对象、数组对class进行绑定

:class ="a", a为data中的一个属性

:class = {aclass:isA,bclass:isB} isA和isB都是data的属性，是一个布尔值

:class = "['aclass','bclass']" 使用数组绑定class



:style = "{color:activeColor,fontSize: fontsize+'px'}"

其中activeColor 和 fontsize都是data中的属性



## VUE 图片切换





## VUE v-for指令

v-for指令作用是根据数据生成列表结构，循环遍历数据

数组经常和v-for结合使用

语法是（item,index） in arr 可以不加index，只使用数据

this.arrObject.push({name:"triger"}); push方法可以增加数据

  this.arrObject.shift();shift方式移除最左边的元素

```html
<div id="app">
    <button @click="add">
        增加数据
    </button>
    <button @click="remove">
        移除数据
    </button>
   <ul>
       <li v-for="item in arr">
           {{item}}
       </li>
   </ul>
   <!--带索引-->
   <ul>
    <li v-for="(item,index) in arr">
        {{index+1}} {{item}}
    </li>
    <h2 v-for="item in arrObject">
        {{item.name}}
    </h2>
    <h2 v-for="(item,index) in arrObject">
       {{index+1}} {{item.name}}
    </h2>
</ul>
</div>

<!-- 开发环境版本，包含了有帮助的命令行警告 -->
<script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
<script>
    new Vue({
        el: "#app",
        data:{
            arr:['成都','上海','北京','深证'],
            arrObject:[
                {name:"dog"},
                {name:"cat"},
                {name:"hourse"}
            ]
        },
        methods: {
            add:function(){
                this.arrObject.push({name:"triger"});
                this.arr.push("日本");
            },
            remove:function(){
                this.arrObject.shift();
                this.arr.shift();
            }
        }
    })
</script>
```

## VUE v-model 双向数据绑定

v-model指令的作用是便捷的设置和获取表单元素的值

绑定的数据会和表单元素的值相关联

```html
<div id="app">
  <input type="text" name="test" v-model="message">
  <h1>{{message}}</h1>
  <button @click="doclick">点击</button>
</div>

<!-- 开发环境版本，包含了有帮助的命令行警告 -->
<script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
<script>
    var app=new Vue({
        el: "#app",
        data:{
            message : "何宁好帅啊啊啊"
        },
        methods: {
            doclick:function(){
                alert(this.message);
            }
        }
    })
</script>
```

## VUE生命周期





## 数据存储

LocalStorage  以Key -Value存储数据

```js
window.localStorage.getItem('todos' || '[]')
```

使用watch监视缓存数据

```js
watch:{
    todos :{
        deep:true,//深度监视
        //回调函数
        handler:function(value){//最新的值
            //如果存储JSON JSON.stringify(value)
            window.localStorage.setItem(key,value)
        }
    }
}
```

## 数据存储优化

编写一个工具类，来暴露存储的方法

StroageUtil

```js
/*
使用localstroage存储数据的工具模块
1、函数
2、对象
需要向外暴露1个功能还是多个功能
1个功能暴露函数，多个暴露对象
*/
export default {
    
}
```









## VUE 案列，记事本

常用方法

this.things.splice(index);  根据下标删除数组的元素

总结:

- 列表可以使用v-for指令结合数据生成
- v-on指令结合事件修饰符可以对事件进行限制如.enter敲回车才触发
- v-on绑定事件可以传参数
- 通过v-model可以快速的设置和获取表单元素的值
- 基于数据的开发方式



## VUE 使用js-Cookie实现Cookie存储

安装js-Cookie

```js
npm i js-cookie --save
```

在需要使用的地方进行引入

```js
import Cookies from 'js-cookie'

//使用js-cookie
// Create a cookie, valid across the entire site:
Cookies.set('name', 'value');

// Create a cookie that expires 7 days from now, valid across the entire site:
Cookies.set('name', 'value', { expires: 7 });

// Create an expiring cookie, valid to the path of the current page:
Cookies.set('name', 'value', { expires: 7, path: '' });


// Read cookie:
Cookies.get('name'); // => 'value'
Cookies.get('nothing'); // => undefined

// Read all visible cookies:
Cookies.get(); // => { name: 'value' }


// Delete cookie:
Cookies.remove('name');

// Delete a cookie valid to the path of the current page:
Cookies.set('name', 'value', { path: '' });
Cookies.remove('name'); // fail!
Cookies.remove('name', { path: '' }); // removed!
```



## VUE-网络应用-Axios

引入axios

通过cdn方式引入

```js
<script src="https://unpkg.com/axios/dist/axios.min.js"></script>
```

axios的使用

axios使用链式请求

axios.get(url).then(function(response){})

axios.post(url,{key:value}).then(function(){})

```html
<div id="app">
    <button id="get">get请求</button>
    <button id="post">post请求</button>
</div>


<!-- 开发环境版本，包含了有帮助的命令行警告 -->
<script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
<script src="https://unpkg.com/axios/dist/axios.min.js"></script>

<script>
    document.querySelector('#get').onclick = function(){
        console.log('get请求')
        axios.get('../mock/data.json').then(function(response){
            console.log(response)
        },function(err){
            console.log(err)
        })
    }

    document.querySelector('#post').onclick = function(){
        console.log('post请求')
        axios.post('../mock/data.json',{
            username:'hening'
        }).then(function(response){
            console.log(response)
        },function(err){
            console.log(err)
        })
    }
</script>
```

### axios 与 vue

配置默认访问的前缀

```
Vue.prototype.$axios = axios    //把axios挂载到vue的原型中，在vue中每个组件都可以使用axios发送请求
axios.defaults.baseURL = 'http://localhost:8081' 
axios.defaults.withCredentials=true;
```

注意事项：

axios中的this上下文对象已经被改变，无法通过this修改data的数据

需要把this保存起来，然后通过保存的值间接修改data的值

```html
<div id="app">
    <button id="get" @click='getJoke'>get请求</button>
    <p>{{joke}}</p>
</div>

<!--获取笑话接口 get: https://autumnfish.cn/api/joke-->
<!--获取笑话接口 get: https://autumnfish.cn/api/joke-->
<!-- 开发环境版本，包含了有帮助的命令行警告 -->
<script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
<script src="https://unpkg.com/axios/dist/axios.min.js"></script>
<script>
    var vue = new Vue({
        el:'#app',
        data:{
            joke:'给你讲一个笑话吧！！'
        },
        methods: {
            getJoke:function(){
                console.log(this.joke)
                var that = this
                axios.get('https://autumnfish.cn/api/joke').then(function(response){
                    console.log(response.data)
                    that.joke=response.data
                },function(err){
                    console.log(err)
                })
            }
        }
    })
</script>
```



## Axios 跨域问题

当在vue中使用axios访问后端资源时（前后端分离），一定会出现跨域问题

![image-20200416222554077](C:\Users\YMXD\AppData\Roaming\Typora\typora-user-images\image-20200416222554077.png)

在后端写一个Filter

```java
/**
 * 解决跨域的过滤器
 * @author hening
 */
public class CorsFilter implements Filter {

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {

    }

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        HttpServletResponse response =(HttpServletResponse)servletResponse;
        HttpServletRequest request=(HttpServletRequest)servletRequest;
        String origin = request.getHeader("Origin");
        response.setHeader("Access-Control-Allow-Origin",origin);
        response.setHeader("Access-Control-Allow-Methods","POST,GET,OPTIONS,DELETE");
        response.setHeader("Access-Control-Max-Age","3600");
        response.setHeader("Access-Control-Allow-Headers","x-requested-with,Authorization");
        response.setHeader("Access-Control-Allow-Credentials","true");
        String method = request.getMethod();
        if(method.equalsIgnoreCase("OPTIONS")){
            servletResponse.getOutputStream().write("success".getBytes(StandardCharsets.UTF_8));
        }else {
            filterChain.doFilter(servletRequest,servletResponse);
        }

        System.out.println("*******************Do Filter Ok!");
    }

    @Override
    public void destroy() {

    }
}
```



添加filter

```java
@Configuration
public class FilterConfig {

    @Bean
    public FilterRegistrationBean testFilterRegistration() {

        FilterRegistrationBean registration = new FilterRegistrationBean();
        registration.setFilter(new CorsFilter());
        registration.addUrlPatterns("/*");
        registration.setName("CorsFilter");
        registration.setOrder(1);
        return registration;
    }
}
```

然后就能正常访问后端资源了

![image-20200416224012669](C:\Users\YMXD\AppData\Roaming\Typora\typora-user-images\image-20200416224012669.png)



## Axios携带Cooike访问接口

全局配置(main.js)

```js
//配置基础URL
axios.defaults.baseURL = 'http://localhost:8888/hn-cloud' 
//全局配置默认携带Cookie
axios.defaults.withCredentials=true;
```

将后端接口返回的tokne 存储到Cookie中，方便下次访问时携带

```js
console.log('用户'+res.data.username+' id=' +res.data.id + 'token='+res.data.token+'登录成功')
Cookies.set('JSESSIONID', res.data.token, { expires: 7, path: '' }); //7天过期
```

## Axios 全局拦截

一般新建一个js文件用于配置axios的全局拦截，包括请求前拦截，加上一些参数、响应后拦截，如果返回状态码为200表示成功，不拦截，如果为401表示token过期，跳转到登录页面，如果是其他状态码则进行提示，如果后端报错了，那么进行错误拦截，给出提示。





## VUE-Router

在使用vue init webpack my-project 初始化一个vue项目的时候，就可以自动选择添加router，如果没有选择需要后续手动添加router组件

--save命令可以帮助我们保存配置package的dependcies节点中写入依赖

可以试用-S简写 ：npm i element-ui -S

-dev是保存到dev配置文件中

```shell
npm install vue-router --save-dev
```

如果在一个模块化工程中使用它，必须要通过 Vue.use() 明确地安装路由功能：

```js
import Vue from 'vue'
import VueRouter from 'vue-router'

Vue.use(VueRouter);
```

创建自己的一个组件 Hening.vue

```js
<template>
  <div class="page">
      <h1>这是何宁第一个组件</h1>
      <h2>{{msg}}</h2>
  </div>
</template>

<script>
export default {
  name: 'Hening',
  data () {
    return {
      msg: 'Welcome to Your Vue.js App'
    }
  }
}
</script>

```

在路由的配置文件中index.js中引入自己的组件，并配置路由

```js
import Vue from 'vue'
import Router from 'vue-router'
import HelloWorld from '@/components/HelloWorld'
import Hening from '../components/Hening'

Vue.use(Router)

export default new Router({
  routes: [
    {
      path: '/helloworld',
      name: 'HelloWorld',
      component: HelloWorld
    },
    {
      path: '/hening',
      name: 'Hening',
      component: Hening
    }
  ]
})

```

在main.js中配置路由

```js
// The Vue build version to load with the `import` command
// (runtime-only or standalone) has been set in webpack.base.conf with an alias.
import Vue from 'vue'
import App from './App'
import router from './router'

Vue.config.productionTip = false

/* eslint-disable no-new */
new Vue({
  el: '#app',
  router,
  components: { App },
  template: '<App/>'
})

```



然后在App.vue组件中使用路由，通过点击可以路由到我们的组件

```js
<template>
  <div id="app">
    <img src="./assets/logo.png">
    <div>
      <router-link to="/helloworld">helloworld</router-link>
      <router-link to="/hening">hening</router-link>
    </div>
    <router-view></router-view>
  </div>
</template>

<script>
export default {
  name: 'App'
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>

```



![image-20200322152116136](C:\Users\YMXD\AppData\Roaming\Typora\typora-user-images\image-20200322152116136.png)

## router指令

<router-link to="/helloworld">helloworld</router-link> 相当于一个超链接，将点击这个超链接的请求转发到/helloworld上

<router-view></router-view>显示页面，将<router-link>标签转发的页面进行显示





## VUE - 结合elementui

创建一个vue工程

```shell
# 进入工程目录
cd hello-vue
# 安装 vue-router 如果在创建工程师已经安装，这里可以不用安装
npm install vue-router --save-dev
# 安装 element-ui
npm i element-ui -S
# 安装依赖
npm install
# 安装 SASS 加载器
cnpm install sass-loader node-sass --save-dev
# 启动测试
npm run dev


#问题解决
npm install sass-loader --save;

npm install node-sass --save;

#然后运行npm run start就可以 
```

在cscode使用cnpm失败

win10系统中搜索PowerShell，以管理员身份运行Windos PowerShell

打开后运行*set-ExecutionPolicy RemoteSigned，然后更改权限为A*



## VUE 项目打包

执行npm  run build 命令即可打包

会在当前目录下生成一个dist文件夹，里面包含了一个主入口文件index.html 已经各种静态资源

![image-20200329164617044](C:\Users\YMXD\AppData\Roaming\Typora\typora-user-images\image-20200329164617044.png)

直接打开index.html页面后报错

Failed to load resource: net::ERR_FILE_NOT_FOUND

这是因为打包时候引入静态资源路径不对

修改webpack.prod.conf.js 配置文件，在output项哪里增加一行  publicPath: './'即可

```js
output: {
    path: config.build.assetsRoot,
    filename: utils.assetsPath('js/[name].[chunkhash].js'),
    chunkFilename: utils.assetsPath('js/[id].[chunkhash].js'),
    publicPath: './'
  },
```

然后再次执行 npm run build 命令即可完成打包，然后正常访问index.html页面



## VUE项目中引入Jquery

 ### 安装jquery依赖

npm install jquery --save



### 修改配置文件

修改webpack.base.conf.js

增加一行

const webpack = require('webpack')

![image-20200415195900716](C:\Users\YMXD\AppData\Roaming\Typora\typora-user-images\image-20200415195900716.png)

修改同一文件下module_exports

增加plugins 模块

 // 增加一个plugins

```js
 // 增加一个plugins
  plugins: [
    new webpack.ProvidePlugin({
        $: "jquery",
        jQuery: "jquery"
    })
 ],
```

