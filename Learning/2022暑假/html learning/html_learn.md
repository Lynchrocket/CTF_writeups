# HTML是什么
HTML 是用来描述网页的一种`标记`语言。
* HTML 指的是超文本标记语言: HyperText Markup Language
* HTML 不是一种编程语言，而是一种标记语言；标记语言是一套标记标签 (markup tag)
* HTML 文档包含了HTML 标签及文本内容

# HTML结构
```html
<!DOCTYPE html> <!--声明为 HTML5 文档-->
<html> <!--HTML文档标记，位于网页的最前和最后-->
    <head> <!--HTML文件头标记，用来包含文件的基本信息,内容不会再浏览器中显示-->
        <!--head内容-->
    </head>
    <body> <!--网页真正显示的内容都在body标签中-->
        <!--body内容-->
    </body>
</html>
```

# HTML常用标签（body）
* 标签也叫做元素
* 标签一般成对出现，如 \<开放标签>...\</闭合标签>。（可以 \<开放标签/> 的方式简写，但是不建议）
* 对于一些自闭合标签，就无需进行闭合，如\<input>、\<img>、\<br>、\<hr>等。（\<hr>与\<hr/>等价）
* 有一些标签可闭合也可不闭合，如\<li>等，但是也建议能闭合的就闭合。
* 标签大小写不敏感，即\<br>与\<BR>效果相同，但是推荐使用小写。在未来的HTML（或XHTML）中将会<sTroNg>强制</stRonG>小写

## 注释
`<!--here is comment  -->`

## 标题
`<h1>` - `<h6>`，一共可以定义六级标题。
```html
<h1>这是一级标题</h1>
<h2>这是二级标题</h2>
<h3>这是三级标题</h3>
```
效果如下：
    <h1>这是一级标题</h1>
    <h2>这是二级标题</h2>
    <h3>这是三级标题</h3>

## 段落
`<p></p>` 。
```html
<p>段 落 一</p>
<p>段 落      二</p>
<p>段    落       三</p>
```
效果如下：
    <p>段 落 一</p>
    <p>段 落      二</p>
    <p>段    落       三</p>

注意到上面的空格，无论有多少个空格（包括换行符，在markdown里无法演示），渲染时HTML解释器会将这些空格全部减少为一个空格。

## 链接
`<a>` 。
```html
<a href="https://compass.ctfd.io/">链接</a>
```
效果如下：

<a href="https://compass.ctfd.io/">链接</a>

## 图像
`<img>` 。
```html
<img src="pic/desktop.png" width="500" height="200">
```
效果如下：

<img src="pic/desktop.png" width="500" height="200">

## 强调
`<em></em>` 或者 `<strong></strong>` 。
```html
<p>这是<em>emphasis</em> </p>
<p>这是<strong>strong importance</strong> </p>
```
效果如下：

<p>这是<em>emphasis</em> </p>
<p>这是<strong>strong importance</strong> </p>

<font color="red">注：</font>实际上有看上去一样的 `<i></i>` 和 `<b></b>`，但是分别指的是`文本格式化`中的意大利斜体（Italic）和加粗（Bold）。但是在一些地方的渲染效果可能会不一样。

<p>这是<i>Italic</i> </p>
<p>这是<b>Bold</b> </p>

## 区块和布局
大多数HTML元素可分为`块级元素`和`内联元素`。块级元素在浏览器显示时，通常会另起新行；而内联元素在显示时通常不会以新行开始。

HTML分区块有两种，`<div></div>`（块级元素） 和 `<span></span>`（内联元素）

```html
<div style="color:#FFD700">
    <h3>这是一个在 div 元素中的标题。</h3>
    <p>这是一个在 div 元素中的文本。</p>
</div>
<p>这是一个在 <span style="color:green;font-weight:bold">span</span> 元素中的文本。</p>
```
效果如下：
<div style="color:#FFD700">
    <h3>这是一个在 div 元素中的标题。</h3>
    <p>这是一个在 div 元素中的文本。</p>
</div>
<p>这是一个在 <span style="color:green;font-weight:bold">span</span> 元素中的文本。</p>

## 表单和输入
`<form></form>` 。表单表示文档中的一个区域，此区域包含交互控件，用于收集用户的输入信息，并将收集到的信息发送到 Web 服务器。

```html
<form name="input" action="html_form_action.php" method="get">
Username: <input type="text" name="usrn"><br>
Password: <input type="password" name="pwd"><br>
<input type="radio" name="sex" value="male">男<br>
<input type="radio" name="sex" value="female">女<br>
<input type="submit" value="Submit">
</form>
```

## 框架
`<iframe src="URL"></iframe>` 。在同一个浏览器窗口中显示不止一个页面。
```html
<iframe src="https://compass.ctfd.io/"></iframe>
```

## 其他
除了以上常见的HTML标签，还有许多其他标签。可以在最后的参考链接里查阅。

## 尝试一下
> see case1.html

# HTML属性
* 添加附加信息
* 描述于开始标签
* 以 “名称=值” 形式 

```html
<a href="https://compass.ctfd.io/">链接</a>
```

# HTML头部
HTML头部标签包含了关于文档的概要信息，也即元信息（meta-information）。

## 文档标题
`<title></title>` 。title在HTML/XHTML文档中是必须的。

## 文档链接
`<base>` 。该标签描述了基本的链接地址。

浏览器一般会从当前文档的 URL 中提取相应的元素来填写相对 URL 中的空白。但使用 \<base> 标签后，浏览器将不再使用当前文档的 URL，而使用 \<base> 标签指定的基本 URL 来解析所有的相对 URL。这其中包括 \<a>、\<img>、\<link>、\<form> 标签中的 URL。

例如：以下代码定义了页面的的基准url为 http://www.w3school.com.cn；故img实际的src为：http://www.w3school.com.cn/i/eg_smile.gif。

```html
<head>
    <meta charset="UTF-8">
    <title></title>
    <base href="http://www.w3school.com.cn" target="_blank" />
</head>
<body>
    <div>
	<img src="i/eg_smile.gif" />
	<a href="i/eg_smile.gif">链接一</a>
    </div>		
</body>
```

## 外部链接
`<link>` 。该标签定义了文档与外部资源之间的关系，通常用于链接到样式表。
```html
<head> 
    <link rel="stylesheet" type="text/css" href="theme.css" />
</head>
```

## 样式文件引用
`<style></style>` 。该标签定义了HTML文档的样式文件引用地址，需要指定样式文件来渲染HTML文档。
```html
<head>
    <style type="text/css">
        h1 {color:red;}
        p {color:blue;}
    </style>
</head>
<body>
    <h1>这是一个标题</h1>
    <p>这是一个段落。</p>
</body>
```

## 元数据
`<meta>` 。提供关于HTML文档的元数据（如页面的描述、
关键词、作者等等）。

```html
<head>
    <meta name="description" content="Hello HTML">
</head>
```

## 脚本
`<script></script>` 。用于加载脚本文件。

`<noscript></noscript>` 提供无法使用脚本时的替代内容。
只有在浏览器不支持脚本或者禁用脚本时，才会显示 \<noscript> 元素中的内容：

## 尝试一下
> see case2.html

# DOM
当网页被加载时，浏览器会创建页面的文档对象模型（Document Object Model）。HTML DOM 模型被构造为对象的树：

![DOM](pic/DOM.gif)

# 信息隐藏
HTML 中的部分标签用于元信息展示、注释等功能，并不用于内容的显示。另一方面，一些属性具有修改浏览器显示样式的功能，在 CTF 中常被用来进行信息隐藏。

```html
标签
<!--...-->    定义注释
<!DOCTYPE>    定义文档类型
<head>        定义关于文档的信息
<meta>        定义关于HTML文档的元信息
<iframe>      定义内联框架

属性
hidden        隐藏元素
```

# XSS
导致 XSS 漏洞的原因是嵌入在 HTML 中的其它动态语言，但是 HTML 为恶意注入提供了输入口。

```html
<script>                      定义客户端脚本
<img src=>                    规定显示图像的 URL
<body background=>            规定文档背景图像URL
<body onload=>                body 标签的事件属性
<input onfocus= autofocus>    form 表单的事件属性
<button onclick=>             击键的事件属性
<link href=>                  定义外部资源链接
<object data=>                定义引用对象数据的 URL
<svg onload=>                 定义 SVG 资源引用
```

# HTML编码
HTML 编码是一种用于表示问题字符已将其安全并入 HTML 文档的方案。HTML 定义了大量 HTML 实体来表示特殊的字符。

|实体表示法|数字表示法（十进制）|数字表示法（十六进制）|特殊字符|
|---|---|---|---|
|&quot|&#34|&#22|"|
|&apos|&#39|&#27|'|
|&amp|&#38|&#26|&|
|&nbsp|&#32|&#20| (空格)|
|&lt|&#60|&#3c|<|
|&gt|&#62|&#3e|>|
|...|...|...|...|

# 参考链接

[HTML: HyperText Markup Language](https://developer.mozilla.org/en-US/docs/Web/HTML)

[HTML参考手册](https://www.w3school.com.cn/tags/tag_doctype.asp)

[HTML基础](https://www.wenjiangs.com/doc/0pkvndr4kh)

[HTML5 Security Cheatsheet](http://html5sec.org/)