

## background

```css
{
background-image:url('img_tree.png');
  
background-repeat:no-repeat;
background-position:right top;
}
```

| Property                                                     | 描述                                         |
| ------------------------------------------------------------ | -------------------------------------------- |
| [background](https://www.runoob.com/cssref/css3-pr-background.html) | 简写属性，作用是将背景属性设置在一个声明中。 |
| [background-attachment](https://www.runoob.com/cssref/pr-background-attachment.html) | 背景图像是否固定或者随着页面的其余部分滚动。 |
| [background-image](https://www.runoob.com/cssref/pr-background-image.html) | 把图像设置为背景。                           |
| [background-position](https://www.runoob.com/cssref/pr-background-position.html) | 设置背景图像的起始位置。                     |
| [background-repeat](https://www.runoob.com/cssref/pr-background-repeat.html) | 设置背景图像是否及如何重复。                 |

### linear-gradient() 函数

以下实例演示了从头部开始的线性渐变，从红色开始，转为黄色，再到蓝色:

```css
#grad {
  background-image: linear-gradient(red, yellow, blue);
}
/* 渐变轴为45度，从蓝色渐变到红色 */
linear-gradient(45deg, blue, red);

/* 从右下到左上、从蓝色渐变到红色 */
linear-gradient(to left top, blue, red);

/* 从下到上，从蓝色开始渐变、到高度40%位置是绿色渐变开始、最后以红色结束 */
linear-gradient(0deg, blue, green 40%, red);
```

```css
background-image: linear-gradient(135deg, #72EDF1 10%, #5151E6 100%);
```

+ 135deg:表示角度



## Text

### 文本对齐方式

```css
h1 {text-align:center;}
p.date {text-align:right;}
p.main {text-align:justify;}
```

### 文本缩进

```css
p {text-indent:50px;}
```



## 盒子模型Box Model

![image-20200314202957960](C:\Users\YMXD\AppData\Roaming\Typora\typora-user-images\image-20200314202957960.png)

- **Margin(外边距)** - 清除边框外的区域，外边距是透明的。

- **Border(边框)** - 围绕在内边距和内容外的边框。

- **Padding(内边距)** - 清除内容周围的区域，内边距是透明的。

- **Content(内容)** - 盒子的内容，显示文本和图像。

  

## CSS margin(外边距)

![image-20200314203621291](C:\Users\YMXD\AppData\Roaming\Typora\typora-user-images\image-20200314203621291.png)

## CSS padding（填充）

![image-20200314203750305](C:\Users\YMXD\AppData\Roaming\Typora\typora-user-images\image-20200314203750305.png)

在CSS中，它可以指定不同的侧面不同的填充：

```css
padding-top:25px;
padding-bottom:25px;
padding-right:50px;
padding-left:50px;
```

也可以简写代码

```css
padding:25px 50px;
```

不同的简写代表不同的内容，Padding属性，可以有一到四个值。顺序为 上、右、下、左

 **padding:25px 50px 75px 100px;**

- 上填充为25px
- 右填充为50px
- 下填充为75px
- 左填充为100px

 **padding:25px 50px 75px;**

- 上填充为25px
- 左右填充为50px
- 下填充为75px

 **padding:25px 50px;**

- 上下填充为25px
- 左右填充为50px

 **padding:25px;**

- 所有的填充都是25px

## CSS Display(显示) 与 Visibility（可见性）

隐藏元素 - display:none或visibility:hidden
隐藏一个元素可以通过把display属性设置为"none"，或把visibility属性设置为"hidden"。但是请注意，这两种方法会产生不同的结果。

visibility:hidden可以隐藏某个元素，但隐藏的元素仍需占用与未隐藏之前一样的空间。也就是说，该元素虽然被隐藏了，但仍然会影响布局。



## CSS Display - 块和内联元素

h1,p,div标签为典型的块元素，在前后都是换行符。

li{display:inline;}变为内联元素

span,a 标签内联元素只需要必要的宽度，不强制换行。

span {display:block;} 变为块级元素

```css
li{display:inline;}
span {display:block;}
```

## CSS Position(定位)

### static 定位

HTML 元素的默认值，即没有定位，遵循正常的文档流对象。

静态定位的元素不会受到 top, bottom, left, right影响。



### fixed 定位

元素的位置相对于浏览器窗口是固定位置。

即使窗口是滚动的它也不会移动：

### relative 定位

相对定位元素的定位是相对其正常位置。会受到 top, bottom, left, right影响。

### absolute 定位

绝对定位的元素的位置相对于最近的已定位父元素，如果元素没有已定位的父元素，那么它的位置相对于<html>:



## CSS 布局 - Overflow

CSS overflow 属性用于控制内容溢出元素框时显示的方式。

CSS overflow 属性可以控制内容溢出元素框时在对应的元素区间内添加滚动条。

| 值      | 描述                                                     |
| :------ | :------------------------------------------------------- |
| visible | 默认值。内容不会被修剪，会呈现在元素框之外。             |
| hidden  | 内容会被修剪，并且其余内容是不可见的。                   |
| scroll  | 内容会被修剪，但是浏览器会显示滚动条以便查看其余的内容。 |
| auto    | 如果内容被修剪，则浏览器会显示滚动条以便查看其余的内容。 |
| inherit | 规定应该从父元素继承 overflow 属性的值。                 |

## CSS Float(浮动)

Float（浮动），会使元素向左或向右移动，其周围的元素也会重新排列。

Float指定一个盒子（元素）是否可以浮动。

##  CSS 布局-元素居中对齐

### 元素居中对齐

要水平居中对齐一个元素(如 <div>), 可以使用 **margin: auto;**。

设置到元素的宽度将防止它溢出到容器的边缘。

元素通过指定宽度，并将两边的空外边距平均分配：

```css
.center {
    margin: auto;
    width: 50%;
    border: 3px solid green;
    padding: 10px;
}
```

### 文本居中对齐

如果仅仅是为了文本在元素内居中对齐，可以使用 **text-align: center;**

###  图片居中对齐

要让图片居中对齐, 可以使用 **margin: auto;** 并将它放到 **块** 元素中:

如果不标识为块元素，那么不会生效**margin: auto;**对块元素生效

```css
img {
    display: block;
    margin: auto;
    width: 40%;
}
```



### 左右对齐 - 使用定位方式

我们可以使用 **position: absolute;** 属性来对齐元素:

使用绝对定位，并且距离right0px，即对齐在右边

```css
.right {
    position: absolute;
    right: 0px;
    width: 300px;
    border: 3px solid #73AD21;
    padding: 10px;
}
```

### 左右对齐 - 使用 float 方式

```css
.right {
    float: right;
    width: 300px;
    border: 3px solid #73AD21;
    padding: 10px;
}
```

### 垂直居中对齐 - 使用 padding

```css
.center {
    padding: 70px 0;
    border: 3px solid green;
}
```

如果要水平和垂直都居中，可以使用 **padding** 和 **text-align: center**:

```css
.center {
    padding: 70px 0;
    border: 3px solid green;
    text-align: center;
}
```

## justify-content属性

在弹性盒对象的 <div> 元素中的各项周围留有空白：

```css
div
{
    display: flex;
    justify-content: space-around;
}
```

属性值：

| flex-start    | 默认值。项目位于容器的开头。                     | [测试 »](https://www.runoob.com/try/playit.php?f=playcss_justify-content&preval=flex-start) |
| ------------- | ------------------------------------------------ | ------------------------------------------------------------ |
| flex-end      | 项目位于容器的结尾。                             | [测试 »](https://www.runoob.com/try/playit.php?f=playcss_justify-content&preval=flex-end) |
| center        | 项目位于容器的中心。                             | [测试 »](https://www.runoob.com/try/playit.php?f=playcss_justify-content&preval=center) |
| space-between | 项目位于各行之间留有空白的容器内。               | [测试 »](https://www.runoob.com/try/playit.php?f=playcss_justify-content&preval=space-between) |
| space-around  | 项目位于各行之前、之间、之后都留有空白的容器内。 |                                                              |



## align-items 属性

居中对齐弹性盒的各项 <div> 元素：

```css
div
{
    display: flex;
    align-items:center;
}
```



##  border-radius 属性

给div元素添加圆角的边框

```css
div
{
    border:2px solid;
    border-radius:25px;
}
```

## box-shadow 属性

向 div 元素添加阴影：

```css
div
{
    box-shadow: 10px 10px 5px #888888;
}
```

| 值         | 说明                                                         |
| :--------- | :----------------------------------------------------------- |
| *h-shadow* | 必需的。水平阴影的位置。允许负值                             |
| *v-shadow* | 必需的。垂直阴影的位置。允许负值                             |
| *blur*     | 可选。模糊距离                                               |
| *spread*   | 可选。阴影的大小                                             |
| *color*    | 可选。阴影的颜色。在[CSS颜色值](https://www.runoob.com/cssref/css_colors_legal.aspx)寻找颜色值的完整列表 |
| inset      | 可选。从外层的阴影（开始时）改变阴影内侧阴影                 |

## transform 属性

旋转 div 元素

```css
transform:rotate(7deg);
```

缩放元素

`scale()` 用于修改元素的大小。可以通过向量形式定义的缩放值来放大或缩小元素，同时可以在不同的方向设置不同的缩放值。

语法：scale(sx)，scale(sx, sy)

*sx* 表示缩放向量的横坐标。

*sy* 表示缩放向量的纵坐标

```
.location-container button:hover {
    transform: scale(1.05);
}
```

### translate 属性

重新定位在水平和/或垂直方向上的元件

![image-20200408220624963](C:\Users\YMXD\AppData\Roaming\Typora\typora-user-images\image-20200408220624963.png)

该变换的特征在于二维向量。它的坐标定义了元素在每个方向上移动了多少。

```css
/* Single <length-percentage> values */
transform: translate(200px);
transform: translate(50%);

/* Double <length-percentage> values */
transform: translate(100px, 200px);
transform: translate(100px, 50%);
transform: translate(30%, 200px);
transform: translate(30%, 50%);
transform: translate(-50%,-50%);
```



### transition属性

transition: 200ms; 表示缩放的时间，在一定时间间隔内完成缩放

##  cursor: 属性

属性定义鼠标指针悬浮在元素上方显示的鼠标光标。

常用：pointer

![image-20200404144452050](C:\Users\YMXD\AppData\Roaming\Typora\typora-user-images\image-20200404144452050.png)



## calc函数

calc() 此 [CSS](https://developer.mozilla.org/zh-CN/docs/Web/CSS) 函数允许在声明 CSS 属性值时执行一些计算。

语法：width: calc(100% - 80px);  用于根据计算设置元素的width

此 calc()函数用一个表达式作为它的参数，用这个表达式的结果作为值。这个表达式可以是任何如下操作符的组合，采用标准操作符处理法则的简单表达式。

```css
left: calc(51% - 160px);
```



## box-sizing 属性

 **`box-sizing`** 属性定义了 [user agent](https://developer.mozilla.org/zh-CN/docs/Glossary/User_agent) 应该如何计算一个元素的总宽度和总高度。

border-box:告诉浏览器：你想要设置的边框和内边距的值是包含在width内的，不会出现超出width区域

```css
box-sizing: border-box;
```



### line-height 

设置行间距的属性



## CSS 组合选择符

CSS组合选择符包括各种简单选择符的组合方式。

在 CSS3 中包含了四种组合方式:

- 后代选择器(以空格分隔)
- 子元素选择器(以大于号分隔）
- 相邻兄弟选择器（以加号分隔）
- 普通兄弟选择器（以破折号分隔）

## CSS伪元素

CSS伪元素是用来添加一些选择器的特殊效果。 

### "first-line" 伪元素

用于向文本的首行设置特殊样式。**"first-line" 伪元素只能用于块级元素。**

```css
p:first-line 
{
    color:#ff0000;
    font-variant:small-caps;
}
```

### :first-letter 伪元素

"first-letter" 伪元素用于向文本的首字母设置特殊样式：

```css
p:first-letter 
{
    color:#ff0000;
    font-size:xx-large;
}
```

### CSS - :before 伪元素

":before" 伪元素可以在元素的内容前面插入新内容。

下面的例子在每个 <h1>元素前面插入一幅图片：

content属性：配合：before 和：after使用，用于添加内容

```css
h1:before 
{
    content:url(smiley.gif);
}
```

## CSS 媒体类型

媒体类型允许你指定文件将如何在不同媒体呈现。该文件可以以不同的方式显示在屏幕上，在纸张上，或听觉浏览器等等。 

```css
@media screen
{
    p.test {font-family:verdana,sans-serif;font-size:14px;}
}
@media print
{
    p.test {font-family:times,serif;font-size:10px;}
}
@media screen,print
{
    p.test {font-weight:bold;}
}
```

## CSS 属性 选择器

具有特定属性的HTML元素样式不仅仅是class和id。

### 属性选择器

把包含标题（title）的所有元素变为蓝色

```css
[title]
{
    color:blue;
}
```

### 属性和值选择器

改变了标题title='runoob'元素的边框样式:

```css
[title=runoob]
{
    border:5px solid green;
}
```

### 属性和值的选择器 - 多值

包含指定值的title属性的元素样式的例子，使用（~）分隔属性和值:

```css
[title~=hello] { color:blue; }
```



## CSS 网页布局

网页布局有很多种方式，一般分为以下几个部分：**头部区域、菜单导航区域、内容区域、底部区域**。

![image-20200315124353342](C:\Users\YMXD\AppData\Roaming\Typora\typora-user-images\image-20200315124353342.png)

常用网页布局demo

```html
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
<style>
* {
  box-sizing: border-box;
}
 
body {
  font-family: Arial;
  padding: 10px;
  background: #f1f1f1;
}
 
/* 头部标题 */
.header {
  padding: 30px;
  text-align: center;
  background: white;
}
 
.header h1 {
  font-size: 50px;
}
 
/* 导航条 */
.topnav {
  overflow: hidden;
  background-color: #333;
}
 
/* 导航条链接 */
.topnav a {
  float: left;
  display: block;
  color: #f2f2f2;
  text-align: center;
  padding: 14px 16px;
  text-decoration: none;
}
 
/* 链接颜色修改 */
.topnav a:hover {
  background-color: #ddd;
  color: black;
}
 
/* 创建两列 */
/* Left column */
.leftcolumn {   
  float: left;
  width: 75%;
}
 
/* 右侧栏 */
.rightcolumn {
  float: left;
  width: 25%;
  background-color: #f1f1f1;
  padding-left: 20px;
}
 
/* 图像部分 */
.fakeimg {
  background-color: #aaa;
  width: 100%;
  padding: 20px;
}
 
/* 文章卡片效果 */
.card {
  background-color: white;
  padding: 20px;
  margin-top: 20px;
}
 
/* 列后面清除浮动 */
.row:after {
  content: "";
  display: table;
  clear: both;
}
 
/* 底部 */
.footer {
  padding: 20px;
  text-align: center;
  background: #ddd;
  margin-top: 20px;
}
 
/* 响应式布局 - 屏幕尺寸小于 800px 时，两列布局改为上下布局 */
@media screen and (max-width: 800px) {
  .leftcolumn, .rightcolumn {   
    width: 100%;
    padding: 0;
  }
}
 
/* 响应式布局 -屏幕尺寸小于 400px 时，导航等布局改为上下布局 */
@media screen and (max-width: 400px) {
  .topnav a {
    float: none;
    width: 100%;
  }
}
</style>
</head>
<body>

<div class="header">
  <h1>我的网页</h1>
  <p>重置浏览器大小查看效果。</p>
</div>

<div class="topnav">
  <a href="#">链接</a>
  <a href="#">链接</a>
  <a href="#">链接</a>
  <a href="#" style="float:right">链接</a>
</div>

<div class="row">
  <div class="leftcolumn">
    <div class="card">
      <h2>文章标题</h2>
      <h5>2019 年 4 月 17日</h5>
      <div class="fakeimg" style="height:200px;">图片</div>
      <p>一些文本...</p>
      <p>菜鸟教程 - 学的不仅是技术，更是梦想！菜鸟教程 - 学的不仅是技术，更是梦想！菜鸟教程 - 学的不仅是技术，更是梦想！菜鸟教程 - 学的不仅是技术，更是梦想！</p>
    </div>
    <div class="card">
      <h2>文章标题</h2>
      <h5>2019 年 4 月 17日</h5>
      <div class="fakeimg" style="height:200px;">图片</div>
      <p>一些文本...</p>
      <p>菜鸟教程 - 学的不仅是技术，更是梦想！菜鸟教程 - 学的不仅是技术，更是梦想！菜鸟教程 - 学的不仅是技术，更是梦想！菜鸟教程 - 学的不仅是技术，更是梦想！</p>
    </div>
  </div>
  <div class="rightcolumn">
    <div class="card">
      <h2>关于我</h2>
      <div class="fakeimg" style="height:100px;">图片</div>
      <p>关于我的一些信息..</p>
    </div>
    <div class="card">
      <h3>热门文章</h3>
      <div class="fakeimg"><p>图片</p></div>
      <div class="fakeimg"><p>图片</p></div>
      <div class="fakeimg"><p>图片</p></div>
    </div>
    <div class="card">
      <h3>关注我</h3>
      <p>一些文本...</p>
    </div>
  </div>
</div>

<div class="footer">
  <h2>底部区域</h2>
</div>

</body>
</html>
```







