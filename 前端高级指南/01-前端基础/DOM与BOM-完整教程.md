# DOM与BOM - 完整教程

> @author erik.zhou

## 📚 技术概述

### 版本信息
- **DOM**：Document Object Model（文档对象模型）
- **BOM**：Browser Object Model（浏览器对象模型）
- **标准**：W3C DOM标准

### 学习难度
- **难度等级**：⭐⭐⭐ (中等)
- **预计学习时间**：15-25小时
- **重要程度**：⭐⭐⭐⭐⭐ (必学)

### 前置知识
- HTML基础
- CSS基础
- JavaScript基础

## 🎯 学习目标

完成本教程后，你将能够：

- [ ] 理解DOM树结构
- [ ] 掌握DOM元素选择和操作
- [ ] 掌握事件处理机制
- [ ] 理解事件冒泡和捕获
- [ ] 掌握BOM对象的使用
- [ ] 能够操作浏览器API

## 📖 目录

1. [DOM基础](#1-dom基础)
2. [选择元素](#2-选择元素)
3. [操作元素](#3-操作元素)
4. [事件处理](#4-事件处理)
5. [事件委托](#5-事件委托)
6. [BOM对象](#6-bom对象)
7. [浏览器API](#7-浏览器api)

---

## 1. DOM基础

### 1.1 DOM树结构

```javascript
// 🔥 DOM树示例
/*
document
  └── html
      ├── head
      │   ├── title
      │   └── meta
      └── body
          ├── div
          │   ├── h1
          │   └── p
          └── script
*/

// 访问文档
console.log(document);
console.log(document.documentElement); // <html>
console.log(document.head);            // <head>
console.log(document.body);            // <body>
```

### 1.2 节点类型

```javascript
// 🔥 节点类型
const element = document.getElementById('myDiv');

console.log(element.nodeType);  // 1 (元素节点)
console.log(element.nodeName);  // 'DIV'
console.log(element.nodeValue); // null

// 常见节点类型
// 1: Element (元素节点)
// 3: Text (文本节点)
// 8: Comment (注释节点)
// 9: Document (文档节点)
```

---

## 2. 选择元素

### 2.1 基本选择方法

```javascript
// 🔥 通过ID选择
const element = document.getElementById('myId');

// 🔥 通过类名选择（返回HTMLCollection）
const elements = document.getElementsByClassName('myClass');

// 🔥 通过标签名选择
const divs = document.getElementsByTagName('div');

// 🔥 通过name属性选择
const inputs = document.getElementsByName('username');
```

### 2.2 现代选择方法

```javascript
// 🔥 querySelector（返回第一个匹配的元素）
const element = document.querySelector('.myClass');
const firstDiv = document.querySelector('div');
const complexSelect = document.querySelector('div.container > p:first-child');

// 🔥 querySelectorAll（返回NodeList）
const elements = document.querySelectorAll('.myClass');
const allDivs = document.querySelectorAll('div');

// 遍历NodeList
elements.forEach(element => {
  console.log(element);
});

// 转换为数组
const elementsArray = Array.from(elements);
const elementsArray2 = [...elements];
```

### 2.3 遍历DOM树

```javascript
const parent = document.getElementById('parent');

// 🔥 子节点
console.log(parent.children);        // HTMLCollection（只包含元素节点）
console.log(parent.childNodes);      // NodeList（包含所有节点）
console.log(parent.firstElementChild); // 第一个子元素
console.log(parent.lastElementChild);  // 最后一个子元素

// 🔥 父节点
const child = document.getElementById('child');
console.log(child.parentElement);    // 父元素
console.log(child.parentNode);       // 父节点

// 🔥 兄弟节点
console.log(child.nextElementSibling);     // 下一个兄弟元素
console.log(child.previousElementSibling); // 上一个兄弟元素
```

---

## 3. 操作元素

### 3.1 修改内容

```javascript
const element = document.getElementById('myDiv');

// 🔥 innerHTML（解析HTML）
element.innerHTML = '<p>Hello <strong>World</strong></p>';

// 🔥 textContent（纯文本）
element.textContent = 'Hello World';

// 🔥 innerText（考虑样式的文本）
element.innerText = 'Hello World';

// ⚠️ innerHTML vs textContent
element.innerHTML = '<script>alert("XSS")</script>'; // 危险！
element.textContent = '<script>alert("XSS")</script>'; // 安全，显示为文本
```

### 3.2 修改属性

```javascript
const img = document.querySelector('img');

// 🔥 getAttribute/setAttribute
console.log(img.getAttribute('src'));
img.setAttribute('src', 'new-image.jpg');
img.setAttribute('alt', 'New Image');

// 🔥 直接访问属性
console.log(img.src);
img.src = 'new-image.jpg';
img.alt = 'New Image';

// 🔥 data属性
const element = document.getElementById('myDiv');
element.setAttribute('data-user-id', '123');
console.log(element.dataset.userId); // '123'

element.dataset.userName = 'zhangsan';
console.log(element.getAttribute('data-user-name')); // 'zhangsan'

// 🔥 删除属性
element.removeAttribute('data-user-id');

// 🔥 检查属性
console.log(element.hasAttribute('id')); // true
```

### 3.3 修改样式

```javascript
const element = document.getElementById('myDiv');

// 🔥 内联样式
element.style.color = 'red';
element.style.fontSize = '20px';
element.style.backgroundColor = 'blue';

// 驼峰命名
element.style.marginTop = '10px';

// 🔥 cssText（一次设置多个样式）
element.style.cssText = 'color: red; font-size: 20px; background-color: blue;';

// 🔥 类名操作
element.className = 'myClass';
element.className += ' anotherClass';

// 🔥 classList（推荐）
element.classList.add('active');
element.classList.remove('inactive');
element.classList.toggle('visible');
console.log(element.classList.contains('active')); // true

// 🔥 获取计算后的样式
const styles = window.getComputedStyle(element);
console.log(styles.color);
console.log(styles.fontSize);
```

### 3.4 创建和插入元素

```javascript
// 🔥 创建元素
const div = document.createElement('div');
div.textContent = 'Hello World';
div.className = 'myClass';

// 🔥 appendChild（末尾添加）
const parent = document.getElementById('parent');
parent.appendChild(div);

// 🔥 insertBefore（指定位置插入）
const reference = document.getElementById('reference');
parent.insertBefore(div, reference);

// 🔥 insertAdjacentHTML
element.insertAdjacentHTML('beforebegin', '<p>Before</p>');
element.insertAdjacentHTML('afterbegin', '<p>Start</p>');
element.insertAdjacentHTML('beforeend', '<p>End</p>');
element.insertAdjacentHTML('afterend', '<p>After</p>');

// 🔥 insertAdjacentElement
const newElement = document.createElement('p');
element.insertAdjacentElement('beforebegin', newElement);

// 🔥 replaceChild（替换）
const newChild = document.createElement('div');
parent.replaceChild(newChild, oldChild);

// 🔥 removeChild（删除）
parent.removeChild(child);

// 🔥 remove（删除自己）
element.remove();

// 🔥 cloneNode（克隆）
const clone = element.cloneNode(true); // true表示深克隆
```

---

## 4. 事件处理

### 4.1 添加事件监听器

```javascript
const button = document.getElementById('myButton');

// 🔥 addEventListener
button.addEventListener('click', function(event) {
  console.log('Button clicked!');
  console.log(event.target); // 触发事件的元素
});

// 箭头函数
button.addEventListener('click', (event) => {
  console.log('Button clicked!');
});

// 命名函数
function handleClick(event) {
  console.log('Button clicked!');
}

button.addEventListener('click', handleClick);

// 🔥 移除事件监听器
button.removeEventListener('click', handleClick);

// ⚠️ 匿名函数无法移除
button.addEventListener('click', function() {
  // 无法移除这个监听器
});
```

### 4.2 事件对象

```javascript
element.addEventListener('click', function(event) {
  // 🔥 事件对象属性
  console.log(event.type);           // 'click'
  console.log(event.target);         // 触发事件的元素
  console.log(event.currentTarget);  // 绑定事件的元素
  console.log(event.clientX);        // 鼠标X坐标
  console.log(event.clientY);        // 鼠标Y坐标
  console.log(event.pageX);          // 页面X坐标
  console.log(event.pageY);          // 页面Y坐标
  
  // 🔥 阻止默认行为
  event.preventDefault();
  
  // 🔥 阻止事件冒泡
  event.stopPropagation();
  
  // 🔥 立即阻止事件冒泡（包括同元素的其他监听器）
  event.stopImmediatePropagation();
});
```

### 4.3 常见事件类型

```javascript
// 🔥 鼠标事件
element.addEventListener('click', handler);       // 点击
element.addEventListener('dblclick', handler);    // 双击
element.addEventListener('mousedown', handler);   // 鼠标按下
element.addEventListener('mouseup', handler);     // 鼠标释放
element.addEventListener('mousemove', handler);   // 鼠标移动
element.addEventListener('mouseenter', handler);  // 鼠标进入
element.addEventListener('mouseleave', handler);  // 鼠标离开
element.addEventListener('mouseover', handler);   // 鼠标悬停
element.addEventListener('mouseout', handler);    // 鼠标移出

// 🔥 键盘事件
element.addEventListener('keydown', handler);     // 键盘按下
element.addEventListener('keyup', handler);       // 键盘释放
element.addEventListener('keypress', handler);    // 键盘按键（已废弃）

// 🔥 表单事件
form.addEventListener('submit', handler);         // 表单提交
input.addEventListener('input', handler);         // 输入变化
input.addEventListener('change', handler);        // 值改变
input.addEventListener('focus', handler);         // 获得焦点
input.addEventListener('blur', handler);          // 失去焦点

// 🔥 文档事件
document.addEventListener('DOMContentLoaded', handler); // DOM加载完成
window.addEventListener('load', handler);               // 页面完全加载
window.addEventListener('beforeunload', handler);       // 页面卸载前
window.addEventListener('unload', handler);             // 页面卸载

// 🔥 其他事件
window.addEventListener('resize', handler);       // 窗口大小改变
window.addEventListener('scroll', handler);       // 滚动
element.addEventListener('contextmenu', handler); // 右键菜单
```

### 4.4 事件冒泡和捕获

```javascript
// 🔥 事件冒泡（默认）
/*
<div id="outer">
  <div id="inner">
    <button id="button">Click</button>
  </button>
</div>
*/

document.getElementById('outer').addEventListener('click', () => {
  console.log('Outer clicked');
});

document.getElementById('inner').addEventListener('click', () => {
  console.log('Inner clicked');
});

document.getElementById('button').addEventListener('click', () => {
  console.log('Button clicked');
});

// 点击按钮输出：
// Button clicked
// Inner clicked
// Outer clicked

// 🔥 事件捕获
document.getElementById('outer').addEventListener('click', () => {
  console.log('Outer clicked');
}, true); // true表示捕获阶段

// 点击按钮输出：
// Outer clicked (捕获)
// Button clicked
// Inner clicked
// Outer clicked (冒泡)
```

---

## 5. 事件委托

### 5.1 事件委托基础

```javascript
// ❌ 为每个元素添加监听器（性能差）
const buttons = document.querySelectorAll('.button');
buttons.forEach(button => {
  button.addEventListener('click', handleClick);
});

// ✅ 事件委托（性能好）
const container = document.getElementById('container');
container.addEventListener('click', function(event) {
  if (event.target.classList.contains('button')) {
    handleClick(event);
  }
});
```

### 5.2 实际应用

```javascript
// 🔥 动态添加的元素也能响应事件
const list = document.getElementById('list');

list.addEventListener('click', function(event) {
  if (event.target.tagName === 'LI') {
    console.log('Clicked:', event.target.textContent);
  }
});

// 动态添加元素
const newItem = document.createElement('li');
newItem.textContent = 'New Item';
list.appendChild(newItem); // 点击也会触发事件

// 🔥 复杂的事件委托
document.getElementById('table').addEventListener('click', function(event) {
  const target = event.target;
  
  // 删除按钮
  if (target.classList.contains('delete-btn')) {
    const row = target.closest('tr');
    row.remove();
  }
  
  // 编辑按钮
  if (target.classList.contains('edit-btn')) {
    const row = target.closest('tr');
    // 编辑逻辑
  }
});
```


## 6. BOM对象

### 6.1 window对象

```javascript
// 🔥 window对象（全局对象）
console.log(window.innerWidth);   // 窗口内部宽度
console.log(window.innerHeight);  // 窗口内部高度
console.log(window.outerWidth);   // 窗口外部宽度
console.log(window.outerHeight);  // 窗口外部高度

// 🔥 打开新窗口
const newWindow = window.open('https://example.com', '_blank', 'width=800,height=600');

// 关闭窗口
newWindow.close();

// 🔥 对话框
window.alert('提示信息');
const result = window.confirm('确认吗？');
const input = window.prompt('请输入：', '默认值');

// 🔥 滚动
window.scrollTo(0, 0);           // 滚动到指定位置
window.scrollBy(0, 100);         // 相对滚动
element.scrollIntoView();        // 滚动到元素可见

// 🔥 定时器
const timeoutId = setTimeout(() => {
  console.log('延迟执行');
}, 1000);

clearTimeout(timeoutId);

const intervalId = setInterval(() => {
  console.log('重复执行');
}, 1000);

clearInterval(intervalId);
```

### 6.2 location对象

```javascript
// 🔥 location对象（URL信息）
console.log(location.href);      // 完整URL
console.log(location.protocol);  // 协议 (http:)
console.log(location.host);      // 主机名和端口
console.log(location.hostname);  // 主机名
console.log(location.port);      // 端口
console.log(location.pathname);  // 路径
console.log(location.search);    // 查询字符串 (?key=value)
console.log(location.hash);      // 锚点 (#section)

// 🔥 导航
location.href = 'https://example.com';  // 跳转
location.assign('https://example.com'); // 跳转（可后退）
location.replace('https://example.com'); // 跳转（不可后退）
location.reload();                       // 刷新
location.reload(true);                   // 强制刷新

// 🔥 解析查询参数
const params = new URLSearchParams(location.search);
console.log(params.get('id'));
console.log(params.get('name'));

// 遍历参数
params.forEach((value, key) => {
  console.log(`${key}: ${value}`);
});
```

### 6.3 navigator对象

```javascript
// 🔥 navigator对象（浏览器信息）
console.log(navigator.userAgent);    // 用户代理字符串
console.log(navigator.language);     // 语言
console.log(navigator.languages);    // 语言列表
console.log(navigator.platform);     // 平台
console.log(navigator.onLine);       // 是否在线
console.log(navigator.cookieEnabled); // Cookie是否启用

// 🔥 地理位置
navigator.geolocation.getCurrentPosition(
  (position) => {
    console.log('纬度:', position.coords.latitude);
    console.log('经度:', position.coords.longitude);
  },
  (error) => {
    console.error('获取位置失败:', error);
  }
);

// 🔥 剪贴板
navigator.clipboard.writeText('复制的文本')
  .then(() => console.log('复制成功'))
  .catch(err => console.error('复制失败:', err));

navigator.clipboard.readText()
  .then(text => console.log('读取的文本:', text))
  .catch(err => console.error('读取失败:', err));
```

### 6.4 history对象

```javascript
// 🔥 history对象（浏览历史）
console.log(history.length);  // 历史记录数量

// 🔥 导航
history.back();               // 后退
history.forward();            // 前进
history.go(-1);               // 后退1页
history.go(2);                // 前进2页

// 🔥 pushState（添加历史记录）
history.pushState(
  { page: 1 },           // 状态对象
  'Page 1',              // 标题（大多数浏览器忽略）
  '/page1'               // URL
);

// 🔥 replaceState（替换当前历史记录）
history.replaceState(
  { page: 2 },
  'Page 2',
  '/page2'
);

// 🔥 监听历史记录变化
window.addEventListener('popstate', (event) => {
  console.log('历史记录变化:', event.state);
});
```

### 6.5 screen对象

```javascript
// 🔥 screen对象（屏幕信息）
console.log(screen.width);        // 屏幕宽度
console.log(screen.height);       // 屏幕高度
console.log(screen.availWidth);   // 可用宽度
console.log(screen.availHeight);  // 可用高度
console.log(screen.colorDepth);   // 颜色深度
console.log(screen.pixelDepth);   // 像素深度
```

---

## 7. 浏览器API

### 7.1 本地存储

```javascript
// 🔥 localStorage（永久存储）
localStorage.setItem('username', 'zhangsan');
const username = localStorage.getItem('username');
localStorage.removeItem('username');
localStorage.clear();

// 存储对象
const user = { name: 'zhangsan', age: 25 };
localStorage.setItem('user', JSON.stringify(user));
const storedUser = JSON.parse(localStorage.getItem('user'));

// 🔥 sessionStorage（会话存储）
sessionStorage.setItem('token', 'abc123');
const token = sessionStorage.getItem('token');

// 🔥 监听存储变化
window.addEventListener('storage', (event) => {
  console.log('Key:', event.key);
  console.log('Old Value:', event.oldValue);
  console.log('New Value:', event.newValue);
  console.log('URL:', event.url);
});
```

### 7.2 Fetch API

```javascript
// 🔥 GET请求
fetch('https://api.example.com/data')
  .then(response => {
    if (!response.ok) {
      throw new Error('Network response was not ok');
    }
    return response.json();
  })
  .then(data => {
    console.log(data);
  })
  .catch(error => {
    console.error('Error:', error);
  });

// 🔥 POST请求
fetch('https://api.example.com/users', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    name: 'zhangsan',
    age: 25
  })
})
  .then(response => response.json())
  .then(data => console.log(data))
  .catch(error => console.error('Error:', error));

// 🔥 async/await方式
async function fetchData() {
  try {
    const response = await fetch('https://api.example.com/data');
    
    if (!response.ok) {
      throw new Error('Network response was not ok');
    }
    
    const data = await response.json();
    console.log(data);
  } catch (error) {
    console.error('Error:', error);
  }
}
```

### 7.3 IntersectionObserver

```javascript
// 🔥 监听元素是否进入视口
const observer = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      console.log('元素进入视口:', entry.target);
      // 懒加载图片
      const img = entry.target;
      img.src = img.dataset.src;
      observer.unobserve(img);
    }
  });
}, {
  root: null,           // 视口
  rootMargin: '0px',    // 边距
  threshold: 0.5        // 50%可见时触发
});

// 观察元素
const images = document.querySelectorAll('img[data-src]');
images.forEach(img => observer.observe(img));
```

### 7.4 MutationObserver

```javascript
// 🔥 监听DOM变化
const observer = new MutationObserver((mutations) => {
  mutations.forEach(mutation => {
    console.log('类型:', mutation.type);
    console.log('目标:', mutation.target);
    
    if (mutation.type === 'childList') {
      console.log('添加的节点:', mutation.addedNodes);
      console.log('删除的节点:', mutation.removedNodes);
    }
    
    if (mutation.type === 'attributes') {
      console.log('属性名:', mutation.attributeName);
      console.log('旧值:', mutation.oldValue);
    }
  });
});

// 配置观察选项
const config = {
  attributes: true,        // 监听属性变化
  childList: true,         // 监听子节点变化
  subtree: true,           // 监听后代节点
  attributeOldValue: true, // 记录旧值
  characterData: true      // 监听文本内容变化
};

// 开始观察
const target = document.getElementById('container');
observer.observe(target, config);

// 停止观察
observer.disconnect();
```

### 7.5 ResizeObserver

```javascript
// 🔥 监听元素大小变化
const observer = new ResizeObserver((entries) => {
  entries.forEach(entry => {
    console.log('宽度:', entry.contentRect.width);
    console.log('高度:', entry.contentRect.height);
    
    // 响应式处理
    if (entry.contentRect.width < 600) {
      entry.target.classList.add('mobile');
    } else {
      entry.target.classList.remove('mobile');
    }
  });
});

// 观察元素
const element = document.getElementById('responsive');
observer.observe(element);
```

### 7.6 requestAnimationFrame

```javascript
// 🔥 动画循环
function animate() {
  // 更新动画
  element.style.left = position + 'px';
  position += 1;
  
  // 继续动画
  if (position < 500) {
    requestAnimationFrame(animate);
  }
}

requestAnimationFrame(animate);

// 🔥 取消动画
const animationId = requestAnimationFrame(animate);
cancelAnimationFrame(animationId);

// 🔥 平滑滚动动画
function smoothScroll(target, duration) {
  const start = window.pageYOffset;
  const distance = target - start;
  const startTime = performance.now();
  
  function animation(currentTime) {
    const elapsed = currentTime - startTime;
    const progress = Math.min(elapsed / duration, 1);
    
    window.scrollTo(0, start + distance * progress);
    
    if (progress < 1) {
      requestAnimationFrame(animation);
    }
  }
  
  requestAnimationFrame(animation);
}

// 使用
smoothScroll(1000, 500); // 滚动到1000px，耗时500ms
```

---

## 📝 学习检查清单

- [ ] 理解DOM树结构
- [ ] 掌握元素选择方法
- [ ] 掌握DOM元素操作
- [ ] 理解事件处理机制
- [ ] 掌握事件冒泡和捕获
- [ ] 理解事件委托
- [ ] 掌握BOM对象的使用
- [ ] 了解现代浏览器API

---

## 🔗 相关资源

- [MDN DOM文档](https://developer.mozilla.org/zh-CN/docs/Web/API/Document_Object_Model)
- [MDN BOM文档](https://developer.mozilla.org/zh-CN/docs/Web/API/Window)
- [DOM标准](https://dom.spec.whatwg.org/)
- [JavaScript.info DOM](https://javascript.info/document)

---

@author erik.zhou
