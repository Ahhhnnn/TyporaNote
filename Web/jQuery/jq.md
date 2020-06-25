## jQuery 选择器

jQuery 选择器允许您对 HTML 元素组或单个元素进行操作。

jQuery 选择器基于元素的 id、类、类型、属性、属性值等"查找"（或选择）HTML 元素。 它基于已经存在的 [CSS 选择器](https://www.runoob.com/cssref/css-selectors.html)，除此之外，它还有一些自定义的选择器。

jQuery 中所有选择器都以美元符号开头：$()。

```js
$(document).ready(function(){
  $("button").click(function(){
    $("p").hide();
  });
});
```

### #id 选择器

```js
$(document).ready(function(){
  $("button").click(function(){
    $("#test").hide();
  });
})
```

### class 选择器

```js
$(document).ready(function(){
  $("button").click(function(){
    $(".test").hide();
  });
})
```



## Jquery 常见取值&赋值操作

```js
$(" #test ").val()
$(" input[ name='test' ] ").val()
$(" input[ type='text' ] ").val()
$(" input[ type='text' ]").attr("value")

.html() 取出或设置html内容
.text() 取出或设置text内容
.attr() 取出或设置某个属性的值
.attr(“属性名”,”属性值”) 设置标签的属性值
.attr(“属性名”)获取标签的属性值
.width() 取出或设置某个元素的宽度
.height() 取出或设置某个元素的高度
.val() 取出某个表单元素的值
```



## jQuery 事件

页面对不同访问者的响应叫做事件。

事件处理程序指的是当 HTML 中发生某些事件时所调用的方法。如在元素上移动鼠标，选取单选按钮，点击元素

![image-20200318230718860](C:\Users\YMXD\AppData\Roaming\Typora\typora-user-images\image-20200318230718860.png)

```js
$("p").click(function(){
    // 动作触发后执行的代码!!
});
```



## jQuery 效果- 隐藏和显示

```js
$("#hide").click(function(){
  $("p").hide();
});
 
$("#show").click(function(){
  $("p").show();
});
```

带回调函数和参数的方法

```js
$(document).ready(function(){
  $(".hidebtn").click(function(){
    $("div").hide(1000,"linear",function(){
      alert("Hide() 方法已完成!");
    });
  });
});
```

toggle()来切换 hide() 和 show() 方法。

```
$("button").click(function(){
  $("p").toggle();
});
```



## jQuery 效果- 动画

jQuery animate() 方法允许您创建自定义的动画。

**语法：**

$(*selector*).animate({*params*}*,speed,callback*);



## jQuery 方法



### .dblclick 双击事件

```js
$('.folders').dblclick(function () {
			let id = $(this).children("td").children("span").html();
			var location = window.location.href;
			let strings = location.split("?");
			if (id != ""){
				window.location.href = strings[0]+"?fId="+id;
			}else{
				return;
			}
		});
```

### window.location.href :当前url

"http://localhost:8080/moti-cloud/files?fId=0"

```
window.location.href
```



### contextmenu() 方法

单击右键触发 contextmenu 事件

```js
<div id="target">
    右键单击这里
</div>
<script>
$(function () { 
    $( "#target" ).contextmenu(function() {
        alert( "处理程序.contextmenu()被调用。" );
    });
})
</script>
```

```js
$(".folders").contextMenu({
		width: 100, // width
		itemHeight: 30, // 菜单项height
		bgColor: "#fff", // 背景颜色
		color: "#333", // 字体颜色
		fontSize: 12, // 字体大小
		hoverBgColor: "#3498db", // hover背景颜色
		target: function(ele) { // 当前元素
			console.log(ele);
		},
		menu: [{ // 菜单项
			text: " 打开",
			callback: function() {
				let id = $('#tarFolder').html();
				var location = window.location.href;
				let strings = location.split("?");
				if (id != ""){
					window.location.href = strings[0]+"?fId="+id;
				}else{
					return;
				}
			}
		},
			{
				text: " 返回上一级",
				callback: function() {
					toPreFolder();
				}
			},
			{
				text: " 重命名",
				callback: function() {
					let id = $('.flag td span').html();
					let name = $('.flag td a').html();
					let html = $('.flag td').eq(1).html($('' +
							"<form id='updateFolderForm' action='updateFolder' method='post'>" +
							"<input id='updateFolderName' name='fileFolderName' autocomplete='off' type='text' onblur='checkUpdateFolder()' value='"+name+"'>" +
							"<input type='hidden' name='fileFolderId' value='"+id+"'>" +
							"</form>" +
							''));
				}
			},
			{
				text: " 新建文件夹",
				callback: function() {
					addFolderElement();
				}
			},
			{
				text: " 清空并删除",
				callback: function() {
					let id = $('#tarFolder').html();
					var location = window.location.href;
					let strings = location.split("moti-cloud");
					if (id != ""){
						window.location.href = strings[0]+"moti-cloud/deleteFolder?fId="+id;
					}else{
						return;
					}
				}
			}
		]

	});
```

### $(function(){}) 和 $(document).ready(function(){})

这两个方法的效果都是一样的，都是在dom文档树加载完之后执行一个函数,（注意，这里面的ready 是 DOM树加载完成，不是onload的页面资源加载完成的）。

document.ready 和 window.onload 的区别是：上面定义的document.ready方法在DOM树加载完成后就会执行，而window.onload是在页面资源（比如图片和媒体资源，它们的加载速度远慢于DOM的加载速度）加载完成之后才执行。也就是说$(document).ready要比window.onload先执行。



## jQuery DOM 操作

jQuery 提供一系列与 DOM 相关的方法，这使访问和操作元素和属性变得很容易。

三个简单实用的用于 DOM 操作的 jQuery 方法：

- text() - 设置或返回所选元素的文本内容
- html() - 设置或返回所选元素的内容（包括 HTML 标记）
- val() - 设置或返回表单字段的值

```js
$("#btn1").click(function(){
  alert("Text: " + $("#test").text());
});
$("#btn2").click(function(){
  alert("HTML: " + $("#test").html());
});
```

### 获取属性 - attr()

获取href属性的值

```html
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<script src="https://cdn.staticfile.org/jquery/1.10.2/jquery.min.js">
</script>
<script>
$(document).ready(function(){
  $("button").click(function(){
    alert($("#runoob").attr("href"));
  });
});
</script>
</head>

<body>
<p><a href="//www.runoob.com" id="runoob">菜鸟教程</a></p>
<button>显示 href 属性的值</button>
</body>
</html>
```

回调函数

text()、html() 以及 val()，同样拥有回调函数。回调函数有两个参数：被选元素列表中当前元素的下标，以及原始（旧的）值。然后以函数新值返回您希望使用的字符串。

```html
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<script src="https://cdn.staticfile.org/jquery/1.10.2/jquery.min.js">
</script>
<script>
$(document).ready(function(){
  $("#btn1").click(function(){
    $("#test1").text(function(i,origText){
      return "旧文本: " + origText + " 新文本: Hello world! (index: " + i + ")"; 
    });
  });

  $("#btn2").click(function(){
    $("#test2").html(function(i,origText){
      return "旧 html: " + origText + " 新 html: Hello <b>world!</b> (index: " + i + ")"; 
    });
  });

});
</script>
</head>

<body>
<p id="test1">这是一个有 <b>粗体</b> 字的段落。</p>
<p id="test2">这是另外一个有 <b>粗体</b> 字的段落。</p>
<button id="btn1">显示 新/旧 文本</button>
<button id="btn2">显示 新/旧 HTML</button>
</body>
</html>
```

## jQuery - 获取并设置 CSS 类

jQuery 拥有若干进行 CSS 操作的方法。我们将学习下面这些：

- addClass() - 向被选元素添加一个或多个类
- removeClass() - 从被选元素删除一个或多个类
- toggleClass() - 对被选元素进行添加/删除类的切换操作
- css() - 设置或返回样式属性

```js
$("button").click(function(){
  $("h1,h2,p").addClass("blue");
  $("div").addClass("important");
});
```



首个匹配元素的 background-color 值：

```js
$("p").css("background-color");
```

## jQuery 尺寸

### jQuery 尺寸方法

jQuery 提供多个处理尺寸的重要方法：

- width()
- height()
- innerWidth()
- innerHeight()
- outerWidth()
- outerHeight()

![image-20200320200013865](C:\Users\YMXD\AppData\Roaming\Typora\typora-user-images\image-20200320200013865.png)

## jQuery 遍历

------

### 什么是遍历？

jQuery 遍历，意为"移动"，用于根据其相对于其他元素的关系来"查找"（或选取）HTML 元素。以某项选择开始，并沿着这个选择移动，直到抵达您期望的元素为止。

下图展示了一个家族树。通过 jQuery 遍历，您能够从被选（当前的）元素开始，轻松地在家族树中向上移动（祖先），向下移动（子孙），水平移动（同胞）。这种移动被称为对 DOM 进行遍历。



## jQuery 遍历 - 祖先

parent() 方法返回被选元素的直接父元素。

该方法只会向上一级对 DOM 树进行遍历。

```js
$(document).ready(function(){
  $("span").parent().css({"color":"red","border":"2px solid red"});
});
```

parents() 方法返回被选元素的所有祖先元素，它一路向上直到文档的根元素 (<html>)。

```js
$(document).ready(function(){
  $("span").parents();
});
```

您也可以使用可选参数来过滤对祖先元素的搜索。span的祖先元素并且是li元素

```js
$(document).ready(function(){
  $("span").parents("ul");
});
```

parentsUntil() 方法返回介于两个给定元素之间的所有祖先元素。

下面的例子返回介于 <span> 与 <div> 元素之间的所有祖先元素：

```js
$(document).ready(function(){
  $("span").parentsUntil("div");
});
```

## jQuery 遍历 - 后代

children() 方法返回被选元素的所有直接子元素。

该方法只会向下一级对 DOM 树进行遍历。

```js
$(document).ready(function(){
  $("div").children();
});
```

您也可以使用可选参数来过滤对子元素的搜索。

下面的例子返回类名为 "1" 的所有 <p> 元素，并且它们是 <div> 的直接子元素：

```js
$(document).ready(function(){
  $("div").children("p.1");
});
```

find() 方法返回被选元素的后代元素，一路向下直到最后一个后代。

下面的例子返回属于 <div> 后代的所有 <span> 元素：

```js
$(document).ready(function(){
  $("div").find("span").css({"color":"red","border":"2px solid red"});
});
```

返回所有后代元素

```js
$(document).ready(function(){
  $("div").find("*");
});
```

## jQuery 遍历- 过滤

三个最基本的过滤方法是：first(), last() 和 eq()，它们允许您基于其在一组元素中的位置来选择一个特定的元素。

其他过滤方法，比如 filter() 和 not() 允许您选取匹配或不匹配某项指定标准的元素。

first() 方法返回被选元素的首个元素。

```js
$(document).ready(function(){
  $("div p").first();
})
```



last() 方法返回被选元素的最后一个元素。

```js
$(document).ready(function(){
  $("div p").last();
});
```

eq() 方法返回被选元素中带有指定索引号的元素。 参数为索引

```
$(document).ready(function(){
  $("p").eq(1);
});
```



## Jquery Ajax请求

jq 实现跨域请求只需要在参数中加上  dataType: "jsonp", //指定服务器返回的数据类型

```js
 $.ajax({
                url:"https://api.asilu.com/weather/?city="+$("#city").val(),
                dataType: "jsonp", //指定服务器返回的数据类型
                success:function(result){
                    console.log(result);
                    console.log(result.city)
                    weather(result);
            }});
```

对比普通跨域请求

如果是在图片或者script中使用跨域连接，是可以正常请求数据的，因此可以使用这一点

注意：url后跟了一个callback，会在调用后将返回值回调到指定方法中。

```js
var script = document.createElement("script");
            script.src = `https://api.asilu.com/weather/?city=${city.value}&callback=weather`;
            
            //插入到页面中去
document.body.appendChild(script);
```

