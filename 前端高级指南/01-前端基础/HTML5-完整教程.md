# HTML5 - 完整教程

> @author erik.zhou

## 📚 技术概述

### 版本信息
- **HTML版本**：HTML5
- **最新标准**：Living Standard
- **推荐使用**：HTML5

### 学习难度
- **难度等级**：⭐⭐ (简单)
- **预计学习时间**：15-20小时
- **重要程度**：⭐⭐⭐⭐⭐ (必学)

### 前置知识
- 计算机基础知识
- 文本编辑器使用
- 浏览器基本操作

## 🎯 学习目标

完成本教程后，你将能够：

- [ ] 理解HTML5的基本结构
- [ ] 掌握常用HTML标签
- [ ] 能够编写语义化的HTML
- [ ] 理解表单和验证
- [ ] 掌握多媒体元素
- [ ] 了解HTML5新特性

## 📖 目录

1. [HTML基础](#1-html基础)
2. [文档结构](#2-文档结构)
3. [文本标签](#3-文本标签)
4. [列表和表格](#4-列表和表格)
5. [表单](#5-表单)
6. [语义化标签](#6-语义化标签)
7. [多媒体](#7-多媒体)
8. [Canvas和SVG](#8-canvas和svg)
9. [本地存储](#9-本地存储)
10. [最佳实践](#10-最佳实践)

---

## 1. HTML基础

### 1.1 什么是HTML

HTML（HyperText Markup Language）是用于创建网页的标准标记语言。

**核心特点**：
- 🔥 **标记语言**：使用标签定义内容
- 🔥 **结构化**：定义网页的结构和内容
- 🔥 **语义化**：标签具有语义含义
- 🔥 **跨平台**：所有浏览器都支持

### 1.2 第一个HTML页面

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>我的第一个网页</title>
</head>
<body>
    <h1>Hello, World!</h1>
    <p>这是我的第一个HTML页面。</p>
</body>
</html>
```

### 1.3 HTML标签结构

```html
<!-- 🔥 标签的基本结构 -->
<标签名 属性名="属性值">内容</标签名>

<!-- 示例 -->
<a href="https://example.com" target="_blank">链接文本</a>

<!-- 自闭合标签 -->
<img src="image.jpg" alt="图片描述">
<br>
<hr>
<input type="text">
```

---

## 2. 文档结构

### 2.1 DOCTYPE声明

```html
<!-- HTML5文档类型声明 -->
<!DOCTYPE html>

<!-- 旧版本（不推荐） -->
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN">
```

### 2.2 html标签

```html
<!-- 🔥 设置语言 -->
<html lang="zh-CN">
    <!-- 中文页面 -->
</html>

<html lang="en">
    <!-- 英文页面 -->
</html>
```

### 2.3 head标签

```html
<head>
    <!-- 字符编码 -->
    <meta charset="UTF-8">
    
    <!-- 视口设置（响应式必需） -->
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    
    <!-- SEO相关 -->
    <meta name="description" content="页面描述">
    <meta name="keywords" content="关键词1, 关键词2">
    <meta name="author" content="作者名">
    
    <!-- 页面标题 -->
    <title>页面标题</title>
    
    <!-- 引入CSS -->
    <link rel="stylesheet" href="style.css">
    
    <!-- 网站图标 -->
    <link rel="icon" href="favicon.ico">
    
    <!-- 引入JavaScript -->
    <script src="script.js" defer></script>
</head>
```

---

## 3. 文本标签

### 3.1 标题标签

```html
<!-- 🔥 六级标题 -->
<h1>一级标题</h1>
<h2>二级标题</h2>
<h3>三级标题</h3>
<h4>四级标题</h4>
<h5>五级标题</h5>
<h6>六级标题</h6>

<!-- ✅ 正确使用：按层级使用 -->
<h1>网站标题</h1>
<h2>章节标题</h2>
<h3>小节标题</h3>

<!-- ❌ 错误使用：跳级使用 -->
<h1>标题</h1>
<h4>子标题</h4>
```

### 3.2 段落和文本

```html
<!-- 段落 -->
<p>这是一个段落。</p>

<!-- 换行 -->
<p>第一行<br>第二行</p>

<!-- 水平线 -->
<hr>

<!-- 文本格式化 -->
<strong>粗体（强调）</strong>
<b>粗体（样式）</b>
<em>斜体（强调）</em>
<i>斜体（样式）</i>
<mark>高亮文本</mark>
<small>小号文本</small>
<del>删除线</del>
<ins>下划线</ins>
<sub>下标</sub>
<sup>上标</sup>

<!-- 引用 -->
<blockquote>
    这是一段引用文本。
</blockquote>

<q>这是一个短引用</q>

<!-- 代码 -->
<code>console.log('Hello');</code>

<pre>
    保留格式的文本
    包括空格和换行
</pre>
```

### 3.3 链接

```html
<!-- 🔥 基本链接 -->
<a href="https://example.com">外部链接</a>

<!-- 在新标签页打开 -->
<a href="https://example.com" target="_blank">新标签页打开</a>

<!-- 内部链接 -->
<a href="/about.html">关于我们</a>

<!-- 锚点链接 -->
<a href="#section1">跳转到章节1</a>
<h2 id="section1">章节1</h2>

<!-- 邮件链接 -->
<a href="mailto:example@email.com">发送邮件</a>

<!-- 电话链接 -->
<a href="tel:+8613800138000">拨打电话</a>

<!-- 下载链接 -->
<a href="file.pdf" download>下载PDF</a>
```

---

## 4. 列表和表格

### 4.1 列表

```html
<!-- 🔥 无序列表 -->
<ul>
    <li>列表项1</li>
    <li>列表项2</li>
    <li>列表项3</li>
</ul>

<!-- 有序列表 -->
<ol>
    <li>第一项</li>
    <li>第二项</li>
    <li>第三项</li>
</ol>

<!-- 自定义列表 -->
<dl>
    <dt>术语1</dt>
    <dd>术语1的描述</dd>
    <dt>术语2</dt>
    <dd>术语2的描述</dd>
</dl>

<!-- 嵌套列表 -->
<ul>
    <li>项目1
        <ul>
            <li>子项目1.1</li>
            <li>子项目1.2</li>
        </ul>
    </li>
    <li>项目2</li>
</ul>
```

### 4.2 表格

```html
<!-- 🔥 基本表格 -->
<table>
    <thead>
        <tr>
            <th>姓名</th>
            <th>年龄</th>
            <th>城市</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>张三</td>
            <td>25</td>
            <td>北京</td>
        </tr>
        <tr>
            <td>李四</td>
            <td>30</td>
            <td>上海</td>
        </tr>
    </tbody>
    <tfoot>
        <tr>
            <td colspan="3">共2条记录</td>
        </tr>
    </tfoot>
</table>

<!-- 合并单元格 -->
<table>
    <tr>
        <td rowspan="2">合并行</td>
        <td>单元格1</td>
    </tr>
    <tr>
        <td>单元格2</td>
    </tr>
    <tr>
        <td colspan="2">合并列</td>
    </tr>
</table>
```

---

## 5. 表单

### 5.1 基本表单

```html
<!-- 🔥 表单结构 -->
<form action="/submit" method="POST">
    <!-- 文本输入 -->
    <label for="username">用户名：</label>
    <input type="text" id="username" name="username" required>
    
    <!-- 密码输入 -->
    <label for="password">密码：</label>
    <input type="password" id="password" name="password" required>
    
    <!-- 提交按钮 -->
    <button type="submit">提交</button>
</form>
```

### 5.2 输入类型

```html
<form>
    <!-- 文本 -->
    <input type="text" placeholder="请输入文本">
    
    <!-- 密码 -->
    <input type="password" placeholder="请输入密码">
    
    <!-- 邮箱 -->
    <input type="email" placeholder="请输入邮箱">
    
    <!-- 数字 -->
    <input type="number" min="0" max="100" step="1">
    
    <!-- 日期 -->
    <input type="date">
    
    <!-- 时间 -->
    <input type="time">
    
    <!-- 颜色 -->
    <input type="color">
    
    <!-- 文件 -->
    <input type="file" accept="image/*">
    
    <!-- 搜索 -->
    <input type="search" placeholder="搜索...">
    
    <!-- URL -->
    <input type="url" placeholder="https://example.com">
    
    <!-- 电话 -->
    <input type="tel" placeholder="+86 138 0013 8000">
</form>
```

### 5.3 选择控件

```html
<form>
    <!-- 🔥 单选按钮 -->
    <input type="radio" id="male" name="gender" value="male">
    <label for="male">男</label>
    
    <input type="radio" id="female" name="gender" value="female">
    <label for="female">女</label>
    
    <!-- 复选框 -->
    <input type="checkbox" id="agree" name="agree">
    <label for="agree">我同意条款</label>
    
    <!-- 下拉列表 -->
    <select name="city">
        <option value="">请选择城市</option>
        <option value="beijing">北京</option>
        <option value="shanghai">上海</option>
        <option value="guangzhou">广州</option>
    </select>
    
    <!-- 多行文本 -->
    <textarea name="message" rows="5" cols="30" placeholder="请输入留言"></textarea>
</form>
```

### 5.4 表单验证

```html
<form>
    <!-- 必填 -->
    <input type="text" required>
    
    <!-- 最小/最大长度 -->
    <input type="text" minlength="3" maxlength="20">
    
    <!-- 模式匹配 -->
    <input type="text" pattern="[0-9]{11}" title="请输入11位手机号">
    
    <!-- 自定义验证消息 -->
    <input type="email" required 
           oninvalid="this.setCustomValidity('请输入有效的邮箱地址')"
           oninput="this.setCustomValidity('')">
</form>
```

---

## 6. 语义化标签

### 6.1 HTML5语义化标签

```html
<!-- 🔥 页面结构 -->
<header>
    <nav>
        <ul>
            <li><a href="/">首页</a></li>
            <li><a href="/about">关于</a></li>
        </ul>
    </nav>
</header>

<main>
    <article>
        <header>
            <h1>文章标题</h1>
            <time datetime="2024-01-01">2024年1月1日</time>
        </header>
        
        <section>
            <h2>章节标题</h2>
            <p>章节内容...</p>
        </section>
        
        <aside>
            <h3>相关文章</h3>
            <ul>
                <li><a href="#">相关文章1</a></li>
            </ul>
        </aside>
    </article>
</main>

<footer>
    <p>&copy; 2024 版权所有</p>
</footer>
```

### 6.2 语义化标签说明

```html
<!-- 页眉 -->
<header>网站头部或文章头部</header>

<!-- 导航 -->
<nav>导航链接</nav>

<!-- 主要内容 -->
<main>页面主要内容（每页只能有一个）</main>

<!-- 文章 -->
<article>独立的文章内容</article>

<!-- 章节 -->
<section>文档中的章节</section>

<!-- 侧边栏 -->
<aside>侧边栏或附加信息</aside>

<!-- 页脚 -->
<footer>页脚信息</footer>

<!-- 图片容器 -->
<figure>
    <img src="image.jpg" alt="图片">
    <figcaption>图片说明</figcaption>
</figure>

<!-- 详情/摘要 -->
<details>
    <summary>点击展开</summary>
    <p>详细内容...</p>
</details>
```

---

## 7. 多媒体

### 7.1 图片

```html
<!-- 🔥 基本图片 -->
<img src="image.jpg" alt="图片描述">

<!-- 响应式图片 -->
<img src="image.jpg" 
     alt="图片描述"
     width="800"
     height="600"
     loading="lazy">

<!-- 多分辨率图片 -->
<img srcset="image-320w.jpg 320w,
             image-640w.jpg 640w,
             image-1280w.jpg 1280w"
     sizes="(max-width: 320px) 280px,
            (max-width: 640px) 600px,
            1200px"
     src="image-640w.jpg"
     alt="响应式图片">

<!-- picture元素 -->
<picture>
    <source media="(min-width: 800px)" srcset="large.jpg">
    <source media="(min-width: 400px)" srcset="medium.jpg">
    <img src="small.jpg" alt="图片">
</picture>
```

### 7.2 音频

```html
<!-- 🔥 音频播放器 -->
<audio controls>
    <source src="audio.mp3" type="audio/mpeg">
    <source src="audio.ogg" type="audio/ogg">
    您的浏览器不支持音频播放。
</audio>

<!-- 自动播放（静音） -->
<audio autoplay muted loop>
    <source src="audio.mp3" type="audio/mpeg">
</audio>
```

### 7.3 视频

```html
<!-- 🔥 视频播放器 -->
<video controls width="640" height="360">
    <source src="video.mp4" type="video/mp4">
    <source src="video.webm" type="video/webm">
    <track src="subtitles.vtt" kind="subtitles" srclang="zh" label="中文">
    您的浏览器不支持视频播放。
</video>

<!-- 视频属性 -->
<video controls 
       autoplay 
       muted 
       loop 
       poster="thumbnail.jpg"
       preload="metadata">
    <source src="video.mp4" type="video/mp4">
</video>
```

---

## 8. Canvas和SVG

### 8.1 Canvas

```html
<!-- Canvas画布 -->
<canvas id="myCanvas" width="400" height="300"></canvas>

<script>
const canvas = document.getElementById('myCanvas');
const ctx = canvas.getContext('2d');

// 绘制矩形
ctx.fillStyle = '#FF0000';
ctx.fillRect(10, 10, 100, 50);

// 绘制圆形
ctx.beginPath();
ctx.arc(200, 150, 50, 0, 2 * Math.PI);
ctx.fillStyle = '#0000FF';
ctx.fill();
</script>
```

### 8.2 SVG

```html
<!-- 内联SVG -->
<svg width="200" height="200">
    <!-- 矩形 -->
    <rect x="10" y="10" width="100" height="50" fill="red"/>
    
    <!-- 圆形 -->
    <circle cx="150" cy="100" r="50" fill="blue"/>
    
    <!-- 线条 -->
    <line x1="0" y1="0" x2="200" y2="200" stroke="black" stroke-width="2"/>
    
    <!-- 文本 -->
    <text x="50" y="180" font-size="20" fill="green">SVG文本</text>
</svg>
```

---

## 9. 本地存储

### 9.1 localStorage

```html
<script>
// 🔥 存储数据
localStorage.setItem('username', 'zhangsan');

// 读取数据
const username = localStorage.getItem('username');

// 删除数据
localStorage.removeItem('username');

// 清空所有数据
localStorage.clear();

// 存储对象
const user = { name: 'zhangsan', age: 25 };
localStorage.setItem('user', JSON.stringify(user));

// 读取对象
const storedUser = JSON.parse(localStorage.getItem('user'));
</script>
```

### 9.2 sessionStorage

```html
<script>
// sessionStorage用法与localStorage相同
// 区别：关闭标签页后数据会被清除

sessionStorage.setItem('token', 'abc123');
const token = sessionStorage.getItem('token');
</script>
```

---

## 10. 最佳实践

### 10.1 语义化HTML

```html
<!-- ✅ 好的做法：使用语义化标签 -->
<article>
    <header>
        <h1>文章标题</h1>
    </header>
    <section>
        <p>文章内容...</p>
    </section>
</article>

<!-- ❌ 不好的做法：过度使用div -->
<div class="article">
    <div class="header">
        <div class="title">文章标题</div>
    </div>
    <div class="content">
        <div>文章内容...</div>
    </div>
</div>
```

### 10.2 可访问性

```html
<!-- 🔥 为图片添加alt属性 -->
<img src="logo.png" alt="公司Logo">

<!-- 使用label关联表单 -->
<label for="email">邮箱：</label>
<input type="email" id="email" name="email">

<!-- ARIA属性 -->
<button aria-label="关闭对话框">×</button>

<!-- 跳过导航链接 -->
<a href="#main-content" class="skip-link">跳到主内容</a>
```

### 10.3 性能优化

```html
<!-- 延迟加载图片 -->
<img src="image.jpg" alt="图片" loading="lazy">

<!-- 异步加载脚本 -->
<script src="script.js" async></script>

<!-- 延迟加载脚本 -->
<script src="script.js" defer></script>

<!-- 预加载资源 -->
<link rel="preload" href="font.woff2" as="font" type="font/woff2" crossorigin>
```

---

## 📝 学习检查清单

- [ ] 理解HTML文档结构
- [ ] 掌握常用HTML标签
- [ ] 能够创建表单
- [ ] 理解语义化标签
- [ ] 掌握多媒体元素
- [ ] 了解本地存储API
- [ ] 遵循HTML最佳实践

---

## 🔗 相关资源

- [MDN HTML文档](https://developer.mozilla.org/zh-CN/docs/Web/HTML)
- [HTML5规范](https://html.spec.whatwg.org/)
- [Can I Use](https://caniuse.com/)
- [W3C验证器](https://validator.w3.org/)

---

@author erik.zhou
