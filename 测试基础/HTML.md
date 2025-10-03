### HTML介绍

HTML：超文本标记语言，有一套标记标签组成，是用来描述网页的一种语言

**Web前端三大核心技术：**
- HTML：负责网页的架构
- CSS：负责网页的样式、美化
- JS：负责网页的行为

**HTML标签**

- 单标签\<标签名\>
- 双标签\<标签名\> 内容 \</标签名\>

**标签属性**

描述某一特征
- 属性格式：属性名=“属性值”
```html
<a href= "http://www.jd.com">京东</a>
<!-- 单标签 <a 属性名=“属性值”>
双标签 <a 属性名=“属性值>内容</a> -->
```

#### HTML骨架标签

```html
<!DOCTYPE html> <!-- 文档类型声明 -->
<html>
	<head>
		<meta charset="UTF-8">
		<title>标题</title>
	</head>
	<body>
	<!-- 编写代码区域 -->

	</body>
</html>
```

 1. html：根标签，所有的内容都应该放到html标签中
 2. head：头部标签
 3. body：身体标签（代码编写区域）

**关于注释**

国际通用快捷键：ctrl+/
测试点：前端页面发布之前，需要检查所有注释或去除注释

### 标签

#### **标题标签**

HTML标题是通过`<h1> - <h6>`等标签进行定义的
- h1最大，h6最小
```html
<h1>我是h1</h1>
<h6>我是h6</h6>
```

#### 段落标签

特点：语义化、独占一块（换行）
```html
<p>我是段落</p>
```

#### 超链接标签

通过`<a>`标签进行定义

```html
<body>
	<a href="http://www.baidu.com" target="_blank">百度</a>
</body>
```

- 属性：
href：跳转地址或文件
target：打开方式 新窗口打开 `新窗口：target="_blank"`

#### 图片标签

网页中插入图片就要使用图片标签，HTML图片是通过`<img`标签进行定义的

```html
<body>
	<img src="logo.ipg"title="你好"alt="logo"width="104"height="142"/>
<body>
```

属性：
- src:图片路径
- title:光标悬停显示文字
- alt:图片未加载时显示文字
- width:图片宽度
- height:图片高度
测试点：必须有title属性（悬停和未加载显示）

#### 换行与空格

- 换行：`<br />`
- 空格：`&nbsp;`
```html
<body>
	<!--1、换行-->
	你好<br />世界！

	<!--2、空格-->
	学习&nbsp;软件测试！
<body>
```

#### 布局标签

 布局：设置页面布局，便于排版
 - 大盒子：`div` 独占一行
 - 小盒子：`span`

#### 列表标签

列表标签常用`li`元素（分为：有序和无序）
```html
<body>
	<!--有序列表-->
	<ol>
		<li>北京</li>
		<li>上海</li>
	</ol>
	<!--无序列表-->
	<ul>
		<li>测试</li>
		<li>开发</li>
	</ul>

	<style>
	//css标签
	</style>
	
	<script>
	//js标签
	</script>
<body>
```

1. script：js标签
2. style：css标签
3. link：外部加载css标签

#### input、form(表单)标签

- 文本框：`<input type="text" />`
- 密码框：`<input type="password" />`

```html
<body>
	<!--
		form
			作用：将页面输入的数据提交到后台或指定页面
			属性：
				action:指定将数据提交到哪个页面
				method:提交参数的方法(get、post)
					get:查询使用
						1.参数在url明文显示
						2.提交速度快
						3.提交参数有长度限制
					post:提交数据、注册、登录
						1.非明文显示
						2.提交速度慢
						3.提交参数的长度无限制
	-->
	<form action="xxx.html"method="get">
		用户名： <input type="text" />
		<br />
		密码框：<input type="password" />
		<br />
		<!-- 
		单选效果：
		1.相同一组的radio才能做单选
		2.设置相同组名name属性值的为一组
		-->
		性别：
		//单选按钮
		<input type="radio" name="one"/>男
		<input type="radio" name="one"/>女
		<br />
		爱好：
		//复选框
		<input type="checkbox" />赚钱
		<input type="checkbox" />吃饭
		<input type="checkbox" />喝水
		<input type="checkbox" />呃呃
		<br />
		//按钮
		<!--
			按钮测试点：统一使用value进行赋值
			普通按钮默认没有执行效果，需要配合js来实现
		-->
		<input type="submit" /> //提交
		<input type="reset" /> //重置
		<input type="button" value="点我" />	//普通
	</form>
	
</body>
	
```