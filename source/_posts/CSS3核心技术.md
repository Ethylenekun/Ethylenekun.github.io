---
title: CSS3核心技术
tags:
  - FrontEnd
  - CSS
categories: FrontEnd
description: CSS3的核心技术学习笔记
cover: 'https://gcore.jsdelivr.net/gh/Ethylenekun/images/img/CSS.jpg'
abbrlink: 78adfe16
---
# CSS3核心技术

> CSS注释
>
> ```css
> /* CSS注释 */
> ```

## CSS分类

### 位置分类

- 内联样式表(行内样式表)

  ```html
  <div style="width:500px;height:500px;background-color:pink;"></div>
  ```

- 内部样式表

  ```html
  <head>
    <style>
      div {
        width: 500px;
        height: 500px;
        background-color: pink;
      }
    </style>
  </head>
  ```

- 外部样式表

  ```html
  <!-- 在html中引入外部css文件 -->
  <link rel="stylesheet" href="style.css">
  ```

  ```css
  div {
    width: 200px;
    height: 200px;
    background-color: pink;
  }
  ```

---

## 选择器

### 基础选择器

- 标签选择器

  ```css
  p{
      color:red;
  }
  ```

- 类选择器

  ```html
    <div class="d1">Lorem ipsum dolor sit amet.</div>
    <div class="d2">Lorem ipsum dolor sit amet.</div>
  ```

  ```css
  .d2 {
    color: red;
  }
  ```

- id选择器

  ```html
    <div id="d1">Lorem ipsum dolor sit amet.</div>
    <div id="d2">Lorem ipsum dolor sit amet.</div>
  ```

  ```css
  #d1 {
    color: red;
  }
  ```

- 通配符选择器

  ```css
  *{
      margin:0;
      padding:0;
  }
  ```

### 关系选择器

> 子代/后代选择器HTML示例

```html
<div>
    <span>0</span>	
    <p>
        1 <span>2</span> 3
    </p>
</div>
```

- 后代选择器

  ```css
  /* 0 2会显示为红色 */
  div span {
      color:red;
  } 
  ```

- 子代选择器

  ```css
  /* 0会显示为红色 */
  div>span {
    color: red;
  }
  ```

> 相邻/通用兄弟选择器HTML示例

```html
  <div>0</div>
  <p>1</p>
  <p>2</p>
  <p>3</p>
```

- 相邻兄弟选择器

  ```css
  /* 1会显示为红色 */
  div+p {
      color:red;
  }
  ```

- 通用兄弟选择器

  ```css
  /* 1 2 3会显示为红色 */
  div~p {
    color: red;
  }
  ```

### 分组选择器

> 或称**并集选择器**

```css
p,div {
    color:red;
}
```

### 伪类选择器

#### 状态伪类

- 链接伪类

  > 伪类顺序规则：`a:link -> a:visited -> a:hover -> a:active`

  | 链接伪类    | 作用                               |
  | ----------- | ---------------------------------- |
  | `a:link`    | 未访问链接的默认样式               |
  | `a:visited` | 已访问链接的样式                   |
  | `a:hover`   | 鼠标悬停在链接上时的反馈           |
  | `a:active`  | 链接被点击时的瞬时状态(按下到松开) |

- 行为伪类

  > 或称动态伪类

  | 动态伪类 | 作用                 |
  | -------- | -------------------- |
  | `:hover` | 鼠标经过元素         |
  | `:focus` | 获得焦点的元素(表单) |

#### 结构伪类

| 结构伪类        | 作用                                        | 语法                                                         |
| --------------- | ------------------------------------------- | ------------------------------------------------------------ |
| `:first-child`  | 一组兄弟元素中的第一个元素                  |                                                              |
| `:last-child`   | 一组兄弟元素中的最后一个元素                |                                                              |
| `:nth-child(n)` | 一组兄弟元素列表中第n个元素(n是从0开始计算) | `:nth-child(3n)`: 3的倍数，第3.6.9...个元素<br />`:nth-child(n+2)`: 第2个以及以后的元素<br />`:nth-child(-n+3)`: 前面3个元素 |

> `:first-child`和`:first-of-type`**区别**
>
> - `:first-child`选择的是父元素下的第一个子元素，无论其元素类型如何
> - `:first-of-type`选择的是父元素下的第一个具有指定元素类型的子元素，并不要求它是父元素的第一个子元素

#### 表单伪类

| 表单伪类    | 作用                   |
| ----------- | ---------------------- |
| `:disabled` | 表单禁用               |
| `:checked`  | 选中状态，单选复选按钮 |

### 伪元素选择器

- 选择特定部分

| 伪元素选择器     | 作用                        |
| ---------------- | --------------------------- |
| `::first-line`   | 选择首行                    |
| `::first-letter` | 选择首字母                  |
| `::placeholder`  | 选择input或者textarea占位符 |

- 插入内容

| 伪元素选择器 | 作用                               | 语法                                     |
| ------------ | ---------------------------------- | ---------------------------------------- |
| `::before`   | 元素内插入伪元素，作为第一个元素   | div::before{<br />content:"开始";<br />} |
| `::after`    | 元素内插入伪元素，作为最后一个元素 |                                          |

### 属性选择器

| 属性选择器   | 作用                                 |
| ------------ | ------------------------------------ |
| `[属性]`     | 匹配包含指定属性的元素               |
| `[属性=值]`  | 匹配属性值等于指定值的元素           |
| `[属性^=值]` | 匹配属性值以指定字符串开头的元素     |
| `[属性$=值]` | 匹配属性值以指定字符串结尾的元素     |
| `[属性*=值]` | 匹配属性值任意位置包含指定子串的元素 |

---

## CSS优先级

![image-20251218140948992](https://gcore.jsdelivr.net/gh/Ethylenekun/images/img/image-20251218140948992.png)

---

## 文本样式

### 字体样式

- `color`：设置字体颜色
  - 关键字：red、green、blue等
  - 十六进制：#F00、#CCC、#F3F3F3
  - rgba：rgb(255,255,255,0) ,指红绿蓝和透明度

- `font-family`：字体族
  - serif：衬线字体
  - sans-serif：无衬线字体(推荐)
- `font-size`：字体大小
- `font-style`：字体风格
  - normal：正常
  - italic：斜体

- `font-weight`：字体粗细
  - 100：细
  - 400：正常
  - 700：加粗
- `text-decoration`：字体装饰
  - none：取消字体装饰
  - underline：下划线
  - overline：上划线
  - line-through：删除线 

> font简写：`font: font-style font-weight font-size/line-height font-family;`\
>
> 其中`font-size`和`font-family `是必须写

### 文本布局

- `text-align`：控制文本、图片在块级元素的水平对齐

  - 参数：left / center / right / justify(**两端对齐**)

- `text-indent`：块级元素中的首行缩进

  - 2em：相对单位，相对于当前元素的文本大小

- `letter-spacing`：字间距

- `line-height`：文本每行之间的高

   ```css
    p {
    	line-height:1.5; /* 当前字体大小的1.5倍 */   
    }
   ```

  ```css
    /* 文字垂直居中：盒子高度等于line-height */
    p{
        height:20px;
        line-height:20px;
    }
  ```

---

## 盒子模型

### 边框

> 边框语法

- `border: 1px solid red;`

- `border-top/right/bottom/left: 1px solid red;`

-   ` border-color: red yellow green blue;`

- `border-style: solid dashed dotted;`

- `border-width: 1px 20px;`

- `border-top/right/bottom/left-width/style/color`

> 圆角边框：`border-radius` 顺序 - 左上角 右上角 右下角 左下角；

- 胶囊圆角：长方形设置圆角为宽度的一半
- 圆形： 正方形设置圆角为高度一半或者50%

| 圆角写法                               | 作用                                   |
| -------------------------------------- | -------------------------------------- |
| `border-radius: 10px;`                 |                                        |
| `border-radius: 10px 20px;`            | 左上 右下是10px，右上 左下 20px        |
| `border-radius ：10px 20px 30px;`      | 左上是10px，右上 左下是20px，右下30px  |
| `border-radius： 10px 20px 30px 40px;` | 左上10px，右上20px，右下30px，左下40px |

### 内边距

> 内边距语法

- `padding: 10px 20px 30px 40px;`
- `padding-left: 10px;`

### 外边距

> 注意点
>
> 1. **行内元素左右外边距和内边距生效，上下外边距无效，上下内边距生效但不影响行高或布局**
>
> 2. **行内元素设置宽度和高度也无效**

> **块级元素水平区中，只要实现左右外边距都为`auto`即可**

```css
div {
    margin: 0 auto;
    
    /* margin: auto */
    
    /* 
    margin-left: auto;
    margin-right: auto;
    */
}
```

> **外边距折叠**

- 并列关系(兄弟）的区块元素
- 两个上下外边距将合并为一个外边距。其大小等于最大的单个外边距

> **外边距塌陷**

- 嵌套关系(父子)的区块元素
- 给子盒子设置上下外边距会让父盒子塌陷移动

```css
/* 解决方法 */

.father-box {
    border-top: 1px solid transparent; /* 父盒子添加上边框 */
    /* padding-top:1px 父盒子添加上内边距 */
    /* overflow: hidden 父盒子添加该属性 */
}
```

### 尺寸计算

> `box-sizing`：用于定于元素的盒子模型计算方式，控制元素的`width`和`height`是否包含`padding`和`border`

|    属性值     |                             描述                             |
| :-----------: | :----------------------------------------------------------: |
| `content-box` | 默认值。元素的 `width` 和 `height` 仅包含内容区域，不包含 `padding` 和 `border`<br />理解： `width` = 内容的宽度 |
| `border-box`  | 元素的 `width` 和 `height` 包含内容、`padding`和 `border`<br/>理解： `width` = `border` + `padding` + 内容的宽度 |

### 盒子背景

> 复合写法(顺序无关)：`background: 颜色 图片 重复 固定 位置/尺寸 ;`

|          属性           |         作用         |                        常用值                        |
| :---------------------: | :------------------: | :--------------------------------------------------: |
|   `background-color`    |   设置元素背景颜色   |           颜色名称、十六进制、RGB、透明度            |
|   `background-image`    |     设置背景图片     |                   `url(image.jpg)`                   |
|   `background-repeat`   | 控制背景图片是否重复 |    repeat（默认）、no-repeat、repeat-x、repeat-y     |
|  `background-position`  |   定位背景图片位置   |        x y （如 center top， 或者px、%单位）         |
|    `background-size`    |   调整背景图片尺寸   | 默认auto、cover（覆盖）、contain（包含） 或者跟px、% |
| `background-attachment` |  背景是否随页面滚动  |          scroll（默认滚动的）、fixed(固定）          |

> 注意点：`background-attachment`设置为`fixed`时，图片的位置是相对于**视窗**的


### 背景渐变

| 属性/方法                                         | 描述                     | 示例代码                                                  |
| ------------------------------------------------- | ------------------------ | --------------------------------------------------------- |
| `linear-gradient(方向, 颜色1 位置,颜色2 位置...)` | 线性渐变（方向可控）     | `background: linear-gradient(to right, #ff6b6b, #4ecdc4)` |
| `radial-gradient(形状,颜色1,颜色2... )`           | 径向渐变（形状和位置可控 | `radial-gradient(circle, #ff9a9e, #fad0c4)`               |

> 文字渐变

```css
.content {
    background: linear-gradient(to right, pink, red); /* 设置背景渐变 */
	-webkit-background-clip: text; /* 兼容性写法，照顾谷歌老版本浏览器 -webkit */
	background-clip: text; /* 文字范围裁剪背景 */
    -webkit-text-fill-color: transparent; /* 文本填充颜色设置为透明 */
}
```

### 盒子阴影

> 语法：`box-shadow: X轴偏移量 Y轴偏移量 模糊半径 扩散半径 颜色;`
>
> - 多个属性用空格隔开
> - X轴偏移量和Y轴偏移量是必须要写，其余可以省略采取默认值。
> -  默认是外阴影，如果改为内阴影要写 inset

```css
.box:hover {
    box-shadow: 0 0 10px 10px rgba(0, 0, 0, 0.5);
}
```

---

## 过渡

> 语法：` transition :过渡属性 过渡时间;`
>
> - 属性值中间空格隔开
> - 过渡属性如果都要变化可以写 `all`
> - 过渡时间单位是秒s，比如 0.2s 等

```css
.box {
    transition: all 0.3s;
}
```

---

## 样式初始化

```css
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}; /* 去除内外边距,设置盒子模型 */

ul,
ol {
    list-style: none; /* 去除列表样式 */
}

a {
    text-decoration: none; /* 去除链接的下划线 */
}

input,button {
    outline: none;
}
```

> 推荐`Normalize.css`
>
> [官网地址](https://necolas.github.io/normalize.css/)

---

## 文字溢出

```css
.content {
	white-space: nowrap; /* 文本一行显示,不换行 */
    overflow: hidden; /* 溢出部分隐藏 */
    text-overflow: ellipsis; /* 文本溢出显示为省略号 */
}
```

### 多行文本溢出

```css
.content {
    display: -webkit-box; /* 旧版弹性盒子布局 */
    -webkit-box-orient: vertical; /* 文本垂直排列 */
    -webkit-line-clamp: 2; /* 限制显示行数 */
    overflow: hidden; /* 隐藏溢出内容 */
    text-overflow: ellipsis; /* 文本溢出显示为省略号 */
}
```

## 字体图标

[iconfont图标库](https://www.iconfont.cn/)

```css
<link rel="stylesheet" href="iconfont.css">
```

```html
<div class="iconfont icon-xihuan"></div> <!-- 必须引入iconfont类 -->
```

