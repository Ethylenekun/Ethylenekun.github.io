---
title: HTML5
tags:
  - FrontEnd
  - HTML
categories: FrontEnd
description: HTML5的标签内容学习笔记
cover: 'https://gcore.jsdelivr.net/gh/Ethylenekun/images/img/html.jpg'
abbrlink: 98f3a6d9
---

# HTML

## 基本框架

```html
<!DOCTYPE html> <!-- HTML5文档类型声明 -->
<html lang="zh-CN"> <!-- 页面语言代表是中文 -->

<head>
  <meta charset="UTF-8"> <!-- 字符集为UTF-8 -->
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title> <!-- 页面标题 -->
</head>

<body>

</body>

</html>
```

## 标签

### 标题标签h

```html
<h1>一级标题</h1>

<h6>六级标题</h6>
```

### 段落标签p

> 段落p标签里面不要放其他块级元素

```html
<!-- 错误写法 -->
<p>
    <h1></h1>
</p>

<!-- 浏览器输出结果 -->
<p></p>
<h1></h1>
<p></p>
```

### 强调与重要性标签

| 推荐标签(有语义) |  作用  | 过时标签(无语义) |
| :--------------: | :----: | :--------------: |
|      strong      |  加粗  |        b         |
|        em        |  倾斜  |        i         |
|       ins        | 下划线 |        u         |
|       del        | 删除线 |        s         |

### 图片标签img

> - `src`：图片路径
>   - 相对路径
>     - 同一目录：`<img src="p1.png">`
>     - 下级目录：`<img src="img/p1.png">`
>     - 上级目录：`<img src="../p1.png">`
>   - 绝对路径
> - `alt`：用在图片无法正常显示

```html	
<img src="img/d1.webp" alt="d1" title="d1">
```

### 视频和音频标签

> - `src`：媒体路径
> - `controls`：播放控件
> - `autoplay`：自动播放
> - `muted`：静音
> - `loop`：循环播放
> - `poster`：视频封面

```html
<!-- 视频标签 -->
<video src="video.mp4" width="50%" controls muted poster="mi.webp" autoplay loop></video>

<!-- 音频标签 -->
<audio src="img/m1.mp3" controls autoplay loop muted></audio>
```

> 兼容性写法，浏览器会检查 <source> 元素，并且播放第一个与其自身相匹配的媒体
>
> ```html
> <video controls width="50%">
>     <source src="img/video.mp4" type="video/mp4">
>     <source src="img/video.ogg" type="video/ogg">
>     <source src="img/video.webm" type="video/webm">
>     <p>当前浏览器不支持视频格式</p>
> </video>
> ```

### 超链接标签a

> - `href`：链接地址或者下载文件等
> - `target`：打开页面方式
>   - `_self`：默认，当前窗口打开
>   - `_blank`：新窗口打开
> - `download`：若空缺则直接下载文件，添加参数后可自定义下载文件名。不添加该参数，若链接指向图片、视频等浏览器可以直接打开的文件，则不下载直接打开

```html
<!-- 打开外部链接 -->
<a href="https://www.jd.com" target="_blank">京东</a>

<!-- 打开内部html文件 -->
<a href="root.html" target="_blank">跳转root</a>

<!-- 跳转空链接 -->
<a href="#">跳转空链接</a>

<!-- 跳转下载 -->
<a href="img/888.gif" download="8.gif">文件下载</a>

<!-- 邮件链接 -->
<a href="mailto:123@qq.com">发送邮件</a>

<!-- 锚点链接,以id作为跳转路径 -->
<style>
    html {
        scroll-behavior: smooth;
    } /* 平滑滚动效果 */
</style>

<h1 id="top">top</h1>
<div style="height:3000px"></div>
<a href="#top">跳转top</a>
```

### 网站结构标签

|  标签   |             功能             |
| :-----: | :--------------------------: |
| header  |       网页页眉(头部）        |
|   nav   |            导航栏            |
|  main   | 网页内容，每个页面只能有一个 |
|  aside  |            侧边栏            |
| article |           文章相关           |
| section |             分块             |
| footer  |       页面页脚（底部）       |

![image-20251225155403584](https://gcore.jsdelivr.net/gh/Ethylenekun/images/img/image-20251225155403584.png)

### 无语义标签

![image-20251225155511846](https://gcore.jsdelivr.net/gh/Ethylenekun/images/img/image-20251225155511846.png)

### 列表标签

> - 无序列表ul
> - 有序列表ol
> - 描述列表dl

```html
<!-- 无序列表嵌套 -->
<ul>
    <li>
        1
        <ul>
            <li>1.1</li>
            <li>1.2</li>
            <li>1.3</li>
        </ul>
    </li>
    <li>2</li>
    <li>3</li>
</ul>
```
> 效果与下方markdown效果相同
- 1
  - 1.1
  - 1.2
  - 1.3
- 2
- 3

```html
<!-- 描述列表 -->
<dl>
    <dt>浙江</dt>
    <dd>杭州</dd>
    <dd>宁波</dd>

    <dt>江苏</dt>
    <dd>南京</dd>
    <dd>无锡</dd>
</dl>
```

### 表格标签

|  标签   |      功能      |
| :-----: | :------------: |
|  table  |  表格容器标签  |
| caption |    表格标题    |
|   tr    |     行标签     |
|   td    |   单元格标签   |
|   th    | 表头单元格标签 |
|  thead  |    表头区域    |
|  tbody  |  表格内容区域  |
|  tfoot  |  表格底部区域  |

```html
<table>
    <caption>表格标题</caption>
    <thead>
        <tr>
            <th>姓名</th>
            <th>年龄</th>
            <th>性别</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>A</td>
            <td>20</td>
            <td>male</td>
        </tr>
        <tr>
            <td>B</td>
            <td>25</td>
            <td>female</td>
        </tr>
    </tbody>
    <tfoot>
        <tr>
            <td colspan="3">合计2人</td>
        </tr>
    </tfoot>
</table>
```

> 添加表格的边框以及合并效果

```css
th,
td {
    border: 1px solid black;
    width: 80px;
    text-align: center;
}

table {
    border-collapse: collapse; /* 表格边框合并 */
}
```

### 表单控件

- 表单容器`form`标签
  - `action`：在提交表单时，应该把所收集的数据送给（URL）去处理
  - `method`：表单提交的方式为`get`或`post`等

```html
<form action="" method="get">
    <input type="text">
</form>
```

- 控件标签

  - `input`标签

    `name`：元素名称，用于表单提交作为键值，又是单选和复选相关联的名称

    |  type类型  |                             参数                             |
    | :--------: | :----------------------------------------------------------: |
    |   `text`   | `placeholder`：提示信息<br />`maxlenght`：允许的最长字符数<br />`accesskey`：使元素获得焦点的快捷键<br />`autocomplete`：控制自动填充行为，可选值为`on`或者`off` |
    | `password` |                             同上                             |
    |  `radio`   |                     `checked`：默认选中                      |
    | `checkbox` |                             同上                             |
    |  `button`  |                                                              |
    |   `file`   | `multiple`：多文件上传<br />`accept`：选择文件类型，`.xlsx,.xls` |

  - `button`标签

  - `textarea`标签

  - `select`标签

- 辅助`label`标签

```html
<!-- 表单实例 -->
<form action="" method="get">

    用户名：<input type="text" placeholder="Enter your username" name="uname" maxlength="10" autocomplete="off">
    <br>

    密码: <input type="password" name="pwd" autocomplete="off">
    <br>

    <label>
        <input type="radio" name="gender" value="male">男
    </label>

    <input type="radio" name="gender" value="female" id="female" checked>
    <label for="female">女</label>
    <br>

    <label>
        <input type="checkbox" name="hobby" value="1">1
    </label>
    <label>
        <input type="checkbox" name="hobby" value="2" checked>2
    </label>
    <label>
        <input type="checkbox" name="hobby" value="3">3
    </label>
    <br>

    <input type="file" multiple accept=".xlsx,.xls">
    <br>

    <textarea name="textarea"></textarea>
    <br>

    <select name="level">
        <option value="a">A</option>
        <option value="b" selected>B</option>
        <option value="c">C</option>
    </select>
    <br>

    <input type="button" value="提交">
    <button type="reset">重置</button>
</form>
```

## 字符实体

|  字符   |   实体    |         说明         |
| :-----: | :-------: | :------------------: |
| &nbsp;  | `&nbsp;`  |      不换行空格      |
|  &lt;   |  `&lt;`   |         小于         |
|  &gt;   |  `&gt;`   |         大于         |
|  &amp;  |  `&amp;`  | 实体或字符引用的开头 |
| &copy;  | `&copy;`  |       版权符号       |
| &trade; | `&trade;` |       商标符号       |

