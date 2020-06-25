# VScode配置VUE模板

1、文件 ==> 首选项 ==> 用户代码片段 ==> 输入 `vue 然后回车`，打开vue.json文件

在文件中添加以下模板

```js
"Print to console": {
	    "prefix": "vue",
	    "body": [
	      "<template>",
	      "  <div class=\"page\">\n",
	      "  </div>",
	      "</template>\n",
	      "<script>",
	      "export default {",
			  "  name: '',",
		    "  components: {},",
	      "  data() {",
	      "    return {\n",
	      "    }",
	      "  }",
	      "}",
	      "</script>\n",
	      "<style scoped lang=\"stylus\">",
	      "</style>",
	      "$2"
	    ],
	    "description": "Log output to console"
	  }
```



2、保存后重启vscode

3、新建一个vue文件，然后输入vue然后点击tab键就可以自动创建模板