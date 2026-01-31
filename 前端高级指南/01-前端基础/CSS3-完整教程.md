# CSS3 - 完整教程

> @author erik.zhou

## 📚 技术概述

### 版本信息
- **CSS版本**：CSS3
- **最新标准**：CSS3 + CSS4模块
- **推荐使用**：CSS3

### 学习难度
- **难度等级**：⭐⭐⭐ (中等)
- **预计学习时间**：25-35小时
- **重要程度**：⭐⭐⭐⭐⭐ (必学)

### 前置知识
- HTML基础
- 基本的设计概念
- 浏览器使用

## 🎯 学习目标

完成本教程后，你将能够：

- [ ] 掌握CSS选择器和优先级
- [ ] 理解盒模型和布局
- [ ] 熟练使用Flexbox和Grid
- [ ] 掌握CSS动画和过渡
- [ ] 能够实现响应式设计
- [ ] 了解CSS最佳实践

## 📖 目录

1. [CSS基础](#1-css基础)
2. [选择器](#2-选择器)
3. [盒模型](#3-盒模型)
4. [布局](#4-布局)
5. [Flexbox](#5-flexbox)
6. [Grid](#6-grid)
7. [定位](#7-定位)
8. [文本和字体](#8-文本和字体)
9. [颜色和背景](#9-颜色和背景)
10. [动画和过渡](#10-动画和过渡)
11. [响应式设计](#11-响应式设计)
12. [最佳实践](#12-最佳实践)

---

## 1. CSS基础

### 1.1 CSS引入方式

```html
<!-- 🔥 外部样式表（推荐） -->
<link rel="stylesheet" href="style.css">

<!-- 内部样式表 -->
<style>
    body {
        background-color: #f0f0f0;
    }
</style>

<!-- 内联样式（不推荐） -->
<p style="color: red;">红色文本</p>
```

### 1.2 CSS语法

```css
/* 基本语法 */
选择器 {
    属性: 值;
    属性: 值;
}

/* 示例 */
h1 {
    color: blue;
    font-size: 24px;
    margin-bottom: 10px;
}

/* 多个选择器 */
h1, h2, h3 {
    font-family: Arial, sans-serif;
}

/* 注释 */
/* 这是单行注释 */

/*
这是
多行注释
*/
```

---

## 2. 选择器

### 2.1 基础选择器

```css
/* 🔥 元素选择器 */
p {
    color: black;
}

/* 类选择器 */
.container {
    width: 1200px;
}

/* ID选择器 */
#header {
    background-color: #333;
}

/* 通配符选择器 */
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

/* 属性选择器 */
input[type="text"] {
    border: 1px solid #ccc;
}

a[href^="https"] {
    color: green; /* 以https开头 */
}

a[href$=".pdf"] {
    color: red; /* 以.pdf结尾 */
}

a[href*="example"] {
    color: blue; /* 包含example */
}
```

### 2.2 组合选择器

```css
/* 后代选择器 */
div p {
    color: blue;
}

/* 子选择器 */
div > p {
    color: red;
}

/* 相邻兄弟选择器 */
h1 + p {
    font-weight: bold;
}

/* 通用兄弟选择器 */
h1 ~ p {
    color: gray;
}
```

### 2.3 伪类选择器

```css
/* 🔥 链接伪类 */
a:link { color: blue; }
a:visited { color: purple; }
a:hover { color: red; }
a:active { color: orange; }

/* 结构伪类 */
li:first-child { font-weight: bold; }
li:last-child { font-style: italic; }
li:nth-child(2) { color: red; }
li:nth-child(odd) { background: #f0f0f0; }
li:nth-child(even) { background: white; }
li:nth-child(3n) { color: blue; }

/* 表单伪类 */
input:focus { border-color: blue; }
input:disabled { opacity: 0.5; }
input:checked { background: green; }
input:valid { border-color: green; }
input:invalid { border-color: red; }

/* 其他伪类 */
p:not(.special) { color: gray; }
div:empty { display: none; }
```

### 2.4 伪元素选择器

```css
/* 🔥 before和after */
.quote::before {
    content: '"';
    font-size: 2em;
}

.quote::after {
    content: '"';
    font-size: 2em;
}

/* 首字母和首行 */
p::first-letter {
    font-size: 2em;
    font-weight: bold;
}

p::first-line {
    color: blue;
}

/* 选中文本 */
::selection {
    background: yellow;
    color: black;
}
```

### 2.5 选择器优先级

```css
/* 优先级计算：
   内联样式: 1000
   ID选择器: 100
   类/属性/伪类: 10
   元素/伪元素: 1
*/

/* 优先级示例 */
p { color: black; }              /* 1 */
.text { color: blue; }           /* 10 */
#main { color: red; }            /* 100 */
div p { color: green; }          /* 2 */
.container .text { color: purple; } /* 20 */

/* !important（慎用） */
p {
    color: red !important;
}
```

---

## 3. 盒模型

### 3.1 标准盒模型

```css
/* 🔥 盒模型组成：content + padding + border + margin */
.box {
    width: 300px;           /* 内容宽度 */
    height: 200px;          /* 内容高度 */
    padding: 20px;          /* 内边距 */
    border: 5px solid #000; /* 边框 */
    margin: 10px;           /* 外边距 */
}

/* 实际占用宽度 = 300 + 20*2 + 5*2 + 10*2 = 370px */
```

### 3.2 box-sizing

```css
/* 🔥 border-box（推荐） */
* {
    box-sizing: border-box;
}

.box {
    width: 300px;
    padding: 20px;
    border: 5px solid #000;
}
/* 实际占用宽度 = 300px（包含padding和border） */

/* content-box（默认） */
.box {
    box-sizing: content-box;
    width: 300px;
}
/* 实际占用宽度 = 300 + padding + border */
```

### 3.3 边距和内边距

```css
/* 四个方向 */
.box {
    margin-top: 10px;
    margin-right: 20px;
    margin-bottom: 10px;
    margin-left: 20px;
}

/* 简写 */
.box {
    margin: 10px 20px 10px 20px; /* 上 右 下 左 */
    margin: 10px 20px;            /* 上下 左右 */
    margin: 10px;                 /* 四个方向 */
}

/* 水平居中 */
.box {
    width: 800px;
    margin: 0 auto;
}

/* 负边距 */
.box {
    margin-top: -10px; /* 向上移动 */
}
```

### 3.4 边框

```css
/* 🔥 边框属性 */
.box {
    border-width: 2px;
    border-style: solid;
    border-color: #000;
}

/* 简写 */
.box {
    border: 2px solid #000;
}

/* 单独设置 */
.box {
    border-top: 1px solid red;
    border-right: 2px dashed blue;
    border-bottom: 3px dotted green;
    border-left: 4px double orange;
}

/* 圆角 */
.box {
    border-radius: 10px;              /* 四个角 */
    border-radius: 10px 20px;         /* 左上右下 右上左下 */
    border-radius: 10px 20px 30px 40px; /* 左上 右上 右下 左下 */
}

/* 圆形 */
.circle {
    width: 100px;
    height: 100px;
    border-radius: 50%;
}
```

---

## 4. 布局

### 4.1 display属性

```css
/* 🔥 常用display值 */
.block {
    display: block;        /* 块级元素 */
}

.inline {
    display: inline;       /* 行内元素 */
}

.inline-block {
    display: inline-block; /* 行内块元素 */
}

.none {
    display: none;         /* 隐藏元素 */
}

.flex {
    display: flex;         /* 弹性布局 */
}

.grid {
    display: grid;         /* 网格布局 */
}
```

### 4.2 浮动布局

```css
/* 浮动 */
.left {
    float: left;
    width: 200px;
}

.right {
    float: right;
    width: 200px;
}

/* 清除浮动 */
.clearfix::after {
    content: "";
    display: block;
    clear: both;
}

/* 或使用overflow */
.container {
    overflow: hidden;
}
```

---

## 5. Flexbox

### 5.1 Flex容器

```css
/* 🔥 创建Flex容器 */
.container {
    display: flex;
}

/* 主轴方向 */
.container {
    flex-direction: row;         /* 水平（默认） */
    flex-direction: row-reverse; /* 水平反向 */
    flex-direction: column;      /* 垂直 */
    flex-direction: column-reverse; /* 垂直反向 */
}

/* 换行 */
.container {
    flex-wrap: nowrap;   /* 不换行（默认） */
    flex-wrap: wrap;     /* 换行 */
    flex-wrap: wrap-reverse; /* 反向换行 */
}

/* 简写 */
.container {
    flex-flow: row wrap; /* direction + wrap */
}
```

### 5.2 主轴对齐

```css
/* 🔥 justify-content */
.container {
    justify-content: flex-start;    /* 起点对齐 */
    justify-content: flex-end;      /* 终点对齐 */
    justify-content: center;        /* 居中 */
    justify-content: space-between; /* 两端对齐 */
    justify-content: space-around;  /* 环绕对齐 */
    justify-content: space-evenly;  /* 均匀分布 */
}
```

### 5.3 交叉轴对齐

```css
/* align-items */
.container {
    align-items: stretch;    /* 拉伸（默认） */
    align-items: flex-start; /* 起点对齐 */
    align-items: flex-end;   /* 终点对齐 */
    align-items: center;     /* 居中 */
    align-items: baseline;   /* 基线对齐 */
}

/* align-content（多行） */
.container {
    flex-wrap: wrap;
    align-content: flex-start;
    align-content: flex-end;
    align-content: center;
    align-content: space-between;
    align-content: space-around;
}
```

### 5.4 Flex项目

```css
/* 🔥 flex-grow（放大比例） */
.item {
    flex-grow: 1; /* 默认0 */
}

/* flex-shrink（缩小比例） */
.item {
    flex-shrink: 1; /* 默认1 */
}

/* flex-basis（基础大小） */
.item {
    flex-basis: 200px; /* 默认auto */
}

/* 简写 */
.item {
    flex: 1;           /* flex-grow: 1 */
    flex: 0 1 200px;   /* grow shrink basis */
}

/* 单独对齐 */
.item {
    align-self: flex-start;
    align-self: flex-end;
    align-self: center;
}

/* 排序 */
.item {
    order: 1; /* 默认0，数值越小越靠前 */
}
```

### 5.5 Flex实战

```css
/* 🔥 水平垂直居中 */
.container {
    display: flex;
    justify-content: center;
    align-items: center;
    height: 100vh;
}

/* 等分布局 */
.container {
    display: flex;
}
.item {
    flex: 1;
}

/* 圣杯布局 */
.container {
    display: flex;
    min-height: 100vh;
    flex-direction: column;
}
.header, .footer {
    flex: 0 0 auto;
}
.main {
    flex: 1;
    display: flex;
}
.sidebar {
    flex: 0 0 200px;
}
.content {
    flex: 1;
}
```

---

## 6. Grid

### 6.1 Grid容器

```css
/* 🔥 创建Grid容器 */
.container {
    display: grid;
}

/* 定义列 */
.container {
    grid-template-columns: 200px 200px 200px;
    grid-template-columns: 1fr 1fr 1fr;        /* 等分 */
    grid-template-columns: 1fr 2fr 1fr;        /* 比例 */
    grid-template-columns: repeat(3, 1fr);     /* 重复 */
    grid-template-columns: repeat(auto-fill, 200px); /* 自动填充 */
}

/* 定义行 */
.container {
    grid-template-rows: 100px 200px 100px;
    grid-template-rows: repeat(3, 100px);
}

/* 间距 */
.container {
    gap: 20px;              /* 行列间距 */
    row-gap: 20px;          /* 行间距 */
    column-gap: 10px;       /* 列间距 */
}
```

### 6.2 Grid对齐

```css
/* 🔥 容器内对齐 */
.container {
    justify-items: start;   /* 水平对齐 */
    justify-items: end;
    justify-items: center;
    justify-items: stretch; /* 默认 */
    
    align-items: start;     /* 垂直对齐 */
    align-items: end;
    align-items: center;
    align-items: stretch;
}

/* 整体对齐 */
.container {
    justify-content: start;
    justify-content: end;
    justify-content: center;
    justify-content: space-between;
    justify-content: space-around;
    justify-content: space-evenly;
    
    align-content: start;
    align-content: end;
    align-content: center;
    align-content: space-between;
}
```

### 6.3 Grid项目

```css
/* 🔥 指定位置 */
.item {
    grid-column-start: 1;
    grid-column-end: 3;
    grid-row-start: 1;
    grid-row-end: 2;
}

/* 简写 */
.item {
    grid-column: 1 / 3;  /* 起始 / 结束 */
    grid-row: 1 / 2;
}

/* 跨越 */
.item {
    grid-column: span 2; /* 跨越2列 */
    grid-row: span 3;    /* 跨越3行 */
}

/* 区域 */
.container {
    grid-template-areas:
        "header header header"
        "sidebar content content"
        "footer footer footer";
}

.header { grid-area: header; }
.sidebar { grid-area: sidebar; }
.content { grid-area: content; }
.footer { grid-area: footer; }
```

### 6.4 Grid实战

```css
/* 🔥 响应式网格 */
.container {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
    gap: 20px;
}

/* 卡片布局 */
.container {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 20px;
}

@media (max-width: 768px) {
    .container {
        grid-template-columns: repeat(2, 1fr);
    }
}

@media (max-width: 480px) {
    .container {
        grid-template-columns: 1fr;
    }
}
```

---

## 7. 定位

### 7.1 position属性

```css
/* 🔥 静态定位（默认） */
.static {
    position: static;
}

/* 相对定位 */
.relative {
    position: relative;
    top: 10px;
    left: 20px;
}

/* 绝对定位 */
.absolute {
    position: absolute;
    top: 0;
    right: 0;
}

/* 固定定位 */
.fixed {
    position: fixed;
    bottom: 20px;
    right: 20px;
}

/* 粘性定位 */
.sticky {
    position: sticky;
    top: 0;
}
```

### 7.2 z-index

```css
/* 层叠顺序 */
.layer1 {
    position: relative;
    z-index: 1;
}

.layer2 {
    position: relative;
    z-index: 2;
}

.layer3 {
    position: relative;
    z-index: 3;
}
```

---

## 8. 文本和字体

### 8.1 文本样式

```css
/* 🔥 文本属性 */
.text {
    color: #333;
    font-size: 16px;
    font-weight: bold;
    font-style: italic;
    font-family: Arial, sans-serif;
    line-height: 1.6;
    text-align: center;
    text-decoration: underline;
    text-transform: uppercase;
    letter-spacing: 2px;
    word-spacing: 5px;
    text-indent: 2em;
}

/* 文本溢出 */
.ellipsis {
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
}

/* 多行省略 */
.multi-line-ellipsis {
    display: -webkit-box;
    -webkit-line-clamp: 3;
    -webkit-box-orient: vertical;
    overflow: hidden;
}
```

### 8.2 Web字体

```css
/* 🔥 引入字体 */
@font-face {
    font-family: 'MyFont';
    src: url('font.woff2') format('woff2'),
         url('font.woff') format('woff');
    font-weight: normal;
    font-style: normal;
}

.custom-font {
    font-family: 'MyFont', sans-serif;
}

/* Google Fonts */
@import url('https://fonts.googleapis.com/css2?family=Roboto:wght@400;700&display=swap');

.roboto {
    font-family: 'Roboto', sans-serif;
}
```

---

## 9. 颜色和背景

### 9.1 颜色

```css
/* 🔥 颜色表示方式 */
.color {
    /* 关键字 */
    color: red;
    
    /* 十六进制 */
    color: #ff0000;
    color: #f00;
    
    /* RGB */
    color: rgb(255, 0, 0);
    
    /* RGBA */
    color: rgba(255, 0, 0, 0.5);
    
    /* HSL */
    color: hsl(0, 100%, 50%);
    
    /* HSLA */
    color: hsla(0, 100%, 50%, 0.5);
}
```

### 9.2 背景

```css
/* 🔥 背景属性 */
.background {
    background-color: #f0f0f0;
    background-image: url('image.jpg');
    background-repeat: no-repeat;
    background-position: center center;
    background-size: cover;
    background-attachment: fixed;
}

/* 简写 */
.background {
    background: #f0f0f0 url('image.jpg') no-repeat center/cover;
}

/* 多重背景 */
.multi-bg {
    background:
        url('front.png') no-repeat center,
        url('back.jpg') no-repeat center/cover;
}

/* 渐变背景 */
.gradient {
    /* 线性渐变 */
    background: linear-gradient(to right, red, blue);
    background: linear-gradient(45deg, red, yellow, green);
    
    /* 径向渐变 */
    background: radial-gradient(circle, red, blue);
}
```

---

## 10. 动画和过渡

### 10.1 过渡

```css
/* 🔥 transition */
.button {
    background-color: blue;
    transition: background-color 0.3s ease;
}

.button:hover {
    background-color: red;
}

/* 多个属性 */
.box {
    transition: width 0.3s, height 0.3s, background-color 0.5s;
}

/* 所有属性 */
.box {
    transition: all 0.3s ease-in-out;
}

/* 延迟 */
.box {
    transition: transform 0.3s ease 0.1s;
}
```

### 10.2 动画

```css
/* 🔥 定义动画 */
@keyframes slide {
    from {
        transform: translateX(0);
    }
    to {
        transform: translateX(100px);
    }
}

/* 或使用百分比 */
@keyframes bounce {
    0% {
        transform: translateY(0);
    }
    50% {
        transform: translateY(-50px);
    }
    100% {
        transform: translateY(0);
    }
}

/* 应用动画 */
.animated {
    animation-name: slide;
    animation-duration: 1s;
    animation-timing-function: ease;
    animation-delay: 0s;
    animation-iteration-count: infinite;
    animation-direction: alternate;
    animation-fill-mode: forwards;
}

/* 简写 */
.animated {
    animation: slide 1s ease 0s infinite alternate forwards;
}
```

### 10.3 Transform

```css
/* 🔥 2D变换 */
.transform {
    /* 平移 */
    transform: translate(50px, 100px);
    transform: translateX(50px);
    transform: translateY(100px);
    
    /* 缩放 */
    transform: scale(1.5);
    transform: scale(2, 0.5);
    
    /* 旋转 */
    transform: rotate(45deg);
    
    /* 倾斜 */
    transform: skew(10deg, 20deg);
    
    /* 组合 */
    transform: translate(50px, 100px) rotate(45deg) scale(1.5);
}

/* 3D变换 */
.transform-3d {
    transform: rotateX(45deg);
    transform: rotateY(45deg);
    transform: rotateZ(45deg);
    transform: translate3d(50px, 100px, 0);
}

/* 变换原点 */
.transform {
    transform-origin: center center;
    transform-origin: top left;
    transform-origin: 50% 50%;
}
```

---

## 11. 响应式设计

### 11.1 媒体查询

```css
/* 🔥 基本媒体查询 */
@media (max-width: 768px) {
    .container {
        width: 100%;
    }
}

/* 多个条件 */
@media (min-width: 768px) and (max-width: 1024px) {
    .container {
        width: 750px;
    }
}

/* 常用断点 */
/* 手机 */
@media (max-width: 480px) {
    /* 样式 */
}

/* 平板 */
@media (min-width: 481px) and (max-width: 768px) {
    /* 样式 */
}

/* 桌面 */
@media (min-width: 769px) {
    /* 样式 */
}
```

### 11.2 响应式单位

```css
/* 🔥 相对单位 */
.responsive {
    /* em：相对于父元素字体大小 */
    font-size: 1.5em;
    
    /* rem：相对于根元素字体大小 */
    font-size: 1.5rem;
    
    /* vw：视口宽度的1% */
    width: 50vw;
    
    /* vh：视口高度的1% */
    height: 100vh;
    
    /* vmin：vw和vh中较小的值 */
    font-size: 5vmin;
    
    /* vmax：vw和vh中较大的值 */
    font-size: 5vmax;
    
    /* 百分比 */
    width: 50%;
}
```

---

## 12. 最佳实践

### 12.1 命名规范

```css
/* 🔥 BEM命名法 */
.block {}
.block__element {}
.block--modifier {}

/* 示例 */
.card {}
.card__title {}
.card__content {}
.card--featured {}

/* 语义化命名 */
.header {}
.nav {}
.main {}
.sidebar {}
.footer {}
```

### 12.2 代码组织

```css
/* 按功能组织 */
/* 1. 重置样式 */
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

/* 2. 全局样式 */
body {
    font-family: Arial, sans-serif;
    line-height: 1.6;
}

/* 3. 布局 */
.container {}
.row {}
.col {}

/* 4. 组件 */
.button {}
.card {}
.modal {}

/* 5. 工具类 */
.text-center { text-align: center; }
.mt-10 { margin-top: 10px; }
```

### 12.3 性能优化

```css
/* ✅ 使用高效选择器 */
.class {}
#id {}

/* ❌ 避免过度嵌套 */
div > ul > li > a {}

/* ✅ 使用transform代替position */
.animated {
    transform: translateX(100px);
}

/* ❌ 避免 */
.animated {
    left: 100px;
}

/* ✅ 使用will-change提示浏览器 */
.animated {
    will-change: transform;
}
```

---

## 📝 学习检查清单

- [ ] 掌握CSS选择器和优先级
- [ ] 理解盒模型
- [ ] 熟练使用Flexbox和Grid
- [ ] 掌握定位和浮动
- [ ] 能够实现动画和过渡
- [ ] 理解响应式设计
- [ ] 遵循CSS最佳实践

---

## 🔗 相关资源

- [MDN CSS文档](https://developer.mozilla.org/zh-CN/docs/Web/CSS)
- [CSS Tricks](https://css-tricks.com/)
- [Can I Use](https://caniuse.com/)
- [Flexbox Froggy](https://flexboxfroggy.com/)
- [Grid Garden](https://cssgridgarden.com/)

---

@author erik.zhou
