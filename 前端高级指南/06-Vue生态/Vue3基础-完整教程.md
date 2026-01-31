# Vue 3基础 - 完整教程

> @author erik.zhou

## 📚 技术概述

### 版本信息
- **Vue版本**：3.x
- **最新稳定版**：3.4+
- **推荐版本**：3.4+

### 学习难度
- **难度等级**：⭐⭐⭐ (中等)
- **预计学习时间**：25-35小时
- **重要程度**：⭐⭐⭐⭐⭐ (必学)

### 前置知识
- HTML/CSS基础
- JavaScript ES6+
- 基本的编程概念

## 🎯 学习目标

完成本教程后，你将能够：

- [ ] 理解Vue 3的核心概念
- [ ] 掌握模板语法和指令
- [ ] 能够开发Vue组件
- [ ] 理解响应式系统
- [ ] 掌握组件通信
- [ ] 理解生命周期钩子
- [ ] 能够使用Vue 3开发应用

## 📖 目录

1. [Vue 3简介](#1-vue-3简介)
2. [创建应用](#2-创建应用)
3. [模板语法](#3-模板语法)
4. [响应式基础](#4-响应式基础)
5. [计算属性和侦听器](#5-计算属性和侦听器)
6. [Class和Style绑定](#6-class和style绑定)
7. [条件渲染](#7-条件渲染)
8. [列表渲染](#8-列表渲染)
9. [事件处理](#9-事件处理)
10. [表单输入绑定](#10-表单输入绑定)

---

## 1. Vue 3简介

### 1.1 什么是Vue 3

Vue 3是一个渐进式JavaScript框架，用于构建用户界面。

**核心特点**：
- 🔥 **响应式系统**：自动追踪依赖，高效更新
- 🔥 **组件化**：可复用的组件系统
- 🔥 **声明式渲染**：模板语法简洁直观
- 🔥 **性能优化**：比Vue 2更快更小

### 1.2 Vue 3新特性

```typescript
// 🔥 Composition API
import { ref, computed } from 'vue';

export default {
  setup() {
    const count = ref(0);
    const double = computed(() => count.value * 2);
    
    return { count, double };
  }
};

// 🔥 <script setup>语法糖
<script setup>
import { ref, computed } from 'vue';

const count = ref(0);
const double = computed(() => count.value * 2);
</script>

// 🔥 Teleport组件
<Teleport to="body">
  <div class="modal">模态框</div>
</Teleport>

// 🔥 Suspense组件
<Suspense>
  <template #default>
    <AsyncComponent />
  </template>
  <template #fallback>
    <div>加载中...</div>
  </template>
</Suspense>
```

---

## 2. 创建应用

### 2.1 使用Vite创建项目

```bash
# 🔥 创建Vue 3项目
npm create vite@latest my-vue-app -- --template vue

# 使用TypeScript
npm create vite@latest my-vue-app -- --template vue-ts

# 进入项目
cd my-vue-app

# 安装依赖
npm install

# 启动开发服务器
npm run dev
```

### 2.2 项目结构

```
my-vue-app/
├── public/
│   └── favicon.ico
├── src/
│   ├── assets/          # 静态资源
│   ├── components/      # 组件
│   ├── App.vue          # 根组件
│   └── main.ts          # 入口文件
├── index.html
├── package.json
├── tsconfig.json
└── vite.config.ts
```

### 2.3 main.ts入口文件

```typescript
// 🔥 创建Vue应用
import { createApp } from 'vue';
import App from './App.vue';

const app = createApp(App);

// 全局配置
app.config.errorHandler = (err) => {
  console.error('全局错误:', err);
};

// 全局属性
app.config.globalProperties.$api = 'https://api.example.com';

// 挂载应用
app.mount('#app');
```

---

## 3. 模板语法

### 3.1 文本插值

```vue
<template>
  <!-- 🔥 双大括号插值 -->
  <p>{{ message }}</p>
  
  <!-- 表达式 -->
  <p>{{ count + 1 }}</p>
  <p>{{ ok ? 'YES' : 'NO' }}</p>
  <p>{{ message.split('').reverse().join('') }}</p>
  
  <!-- 调用函数 -->
  <p>{{ formatDate(date) }}</p>
</template>

<script setup lang="ts">
import { ref } from 'vue';

const message = ref('Hello Vue 3!');
const count = ref(0);
const ok = ref(true);
const date = ref(new Date());

const formatDate = (date: Date) => {
  return date.toLocaleDateString();
};
</script>
```

### 3.2 原始HTML

```vue
<template>
  <!-- 🔥 v-html指令 -->
  <div v-html="rawHtml"></div>
  
  <!-- ⚠️ 注意XSS攻击 -->
  <div v-html="userInput"></div> <!-- 危险！ -->
</template>

<script setup lang="ts">
import { ref } from 'vue';

const rawHtml = ref('<span style="color: red">红色文本</span>');
const userInput = ref('<script>alert("XSS")</script>'); // 危险！
</script>
```

### 3.3 属性绑定

```vue
<template>
  <!-- 🔥 v-bind指令 -->
  <img v-bind:src="imageSrc" v-bind:alt="imageAlt">
  
  <!-- 简写 -->
  <img :src="imageSrc" :alt="imageAlt">
  
  <!-- 动态属性名 -->
  <button :[attributeName]="value">按钮</button>
  
  <!-- 绑定多个属性 -->
  <div v-bind="objectOfAttrs"></div>
</template>

<script setup lang="ts">
import { ref } from 'vue';

const imageSrc = ref('/logo.png');
const imageAlt = ref('Logo');
const attributeName = ref('disabled');
const value = ref(true);

const objectOfAttrs = ref({
  id: 'container',
  class: 'wrapper'
});
</script>
```

---

## 4. 响应式基础

### 4.1 ref()

```vue
<script setup lang="ts">
import { ref } from 'vue';

// 🔥 ref创建响应式数据
const count = ref(0);
const message = ref('Hello');

// 访问值需要.value
console.log(count.value); // 0

// 修改值
count.value++;
message.value = 'Hi';

// 对象和数组
const user = ref({
  name: 'zhangsan',
  age: 25
});

user.value.name = 'lisi';

const list = ref([1, 2, 3]);
list.value.push(4);
</script>

<template>
  <!-- 模板中自动解包，不需要.value -->
  <p>{{ count }}</p>
  <p>{{ message }}</p>
  <p>{{ user.name }}</p>
</template>
```

### 4.2 reactive()

```vue
<script setup lang="ts">
import { reactive } from 'vue';

// 🔥 reactive创建响应式对象
const state = reactive({
  count: 0,
  message: 'Hello'
});

// 直接访问属性
console.log(state.count); // 0

// 修改属性
state.count++;
state.message = 'Hi';

// 嵌套对象
const user = reactive({
  name: 'zhangsan',
  address: {
    city: 'Beijing',
    country: 'China'
  }
});

user.address.city = 'Shanghai';

// ⚠️ 不能替换整个对象
// state = reactive({ count: 1 }); // 错误！失去响应性
</script>

<template>
  <p>{{ state.count }}</p>
  <p>{{ state.message }}</p>
  <p>{{ user.address.city }}</p>
</template>
```

### 4.3 ref vs reactive

```typescript
// ✅ ref：适合基本类型和需要重新赋值的场景
const count = ref(0);
count.value = 10; // 可以重新赋值

// ✅ reactive：适合对象和不需要重新赋值的场景
const state = reactive({ count: 0 });
state.count = 10; // 修改属性

// ❌ reactive不能重新赋值
// state = reactive({ count: 10 }); // 错误！

// 🔥 推荐使用ref
const user = ref({
  name: 'zhangsan',
  age: 25
});

// 可以重新赋值
user.value = {
  name: 'lisi',
  age: 30
};
```

---

## 5. 计算属性和侦听器

### 5.1 计算属性

```vue
<script setup lang="ts">
import { ref, computed } from 'vue';

const firstName = ref('Zhang');
const lastName = ref('San');

// 🔥 计算属性（只读）
const fullName = computed(() => {
  return `${firstName.value} ${lastName.value}`;
});

// 🔥 可写计算属性
const fullNameWritable = computed({
  get() {
    return `${firstName.value} ${lastName.value}`;
  },
  set(value: string) {
    const names = value.split(' ');
    firstName.value = names[0];
    lastName.value = names[1];
  }
});

// 使用
fullNameWritable.value = 'Li Si';
console.log(firstName.value); // 'Li'
console.log(lastName.value);  // 'Si'
</script>

<template>
  <p>{{ fullName }}</p>
  <input v-model="fullNameWritable">
</template>
```

### 5.2 侦听器

```vue
<script setup lang="ts">
import { ref, watch, watchEffect } from 'vue';

const count = ref(0);
const message = ref('Hello');

// 🔥 watch侦听单个数据源
watch(count, (newValue, oldValue) => {
  console.log(`count从${oldValue}变为${newValue}`);
});

// 🔥 watch侦听多个数据源
watch([count, message], ([newCount, newMessage], [oldCount, oldMessage]) => {
  console.log('count或message变化了');
});

// 🔥 watch侦听对象
const user = ref({ name: 'zhangsan', age: 25 });

watch(
  () => user.value.name,
  (newName, oldName) => {
    console.log(`name从${oldName}变为${newName}`);
  }
);

// 深度侦听
watch(
  user,
  (newUser, oldUser) => {
    console.log('user对象变化了');
  },
  { deep: true }
);

// 🔥 watchEffect自动追踪依赖
watchEffect(() => {
  console.log(`count: ${count.value}, message: ${message.value}`);
});

// 立即执行
watch(
  count,
  (newValue) => {
    console.log('count变化:', newValue);
  },
  { immediate: true }
);
</script>
```

---

## 6. Class和Style绑定

### 6.1 绑定Class

```vue
<template>
  <!-- 🔥 对象语法 -->
  <div :class="{ active: isActive, 'text-danger': hasError }"></div>
  
  <!-- 绑定对象 -->
  <div :class="classObject"></div>
  
  <!-- 🔥 数组语法 -->
  <div :class="[activeClass, errorClass]"></div>
  
  <!-- 数组中使用对象 -->
  <div :class="[{ active: isActive }, errorClass]"></div>
  
  <!-- 组件上使用 -->
  <MyComponent :class="{ active: isActive }" />
</template>

<script setup lang="ts">
import { ref, computed } from 'vue';

const isActive = ref(true);
const hasError = ref(false);

const classObject = computed(() => ({
  active: isActive.value,
  'text-danger': hasError.value
}));

const activeClass = ref('active');
const errorClass = ref('text-danger');
</script>
```

### 6.2 绑定Style

```vue
<template>
  <!-- 🔥 对象语法 -->
  <div :style="{ color: activeColor, fontSize: fontSize + 'px' }"></div>
  
  <!-- 绑定对象 -->
  <div :style="styleObject"></div>
  
  <!-- 🔥 数组语法 -->
  <div :style="[baseStyles, overridingStyles]"></div>
  
  <!-- 自动添加前缀 -->
  <div :style="{ display: ['-webkit-box', '-ms-flexbox', 'flex'] }"></div>
</template>

<script setup lang="ts">
import { ref } from 'vue';

const activeColor = ref('red');
const fontSize = ref(30);

const styleObject = ref({
  color: 'red',
  fontSize: '30px'
});

const baseStyles = ref({
  color: 'red'
});

const overridingStyles = ref({
  fontSize: '30px'
});
</script>
```


## 7. 条件渲染

### 7.1 v-if

```vue
<template>
  <!-- 🔥 v-if -->
  <div v-if="type === 'A'">A</div>
  <div v-else-if="type === 'B'">B</div>
  <div v-else>C</div>
  
  <!-- template包裹多个元素 -->
  <template v-if="ok">
    <h1>标题</h1>
    <p>段落</p>
  </template>
  
  <!-- v-if vs v-show -->
  <div v-if="show">v-if（条件渲染）</div>
  <div v-show="show">v-show（切换display）</div>
</template>

<script setup lang="ts">
import { ref } from 'vue';

const type = ref('A');
const ok = ref(true);
const show = ref(true);
</script>
```

---

## 8. 列表渲染

### 8.1 v-for

```vue
<template>
  <!-- 🔥 遍历数组 -->
  <ul>
    <li v-for="(item, index) in items" :key="item.id">
      {{ index }}: {{ item.text }}
    </li>
  </ul>
  
  <!-- 🔥 遍历对象 -->
  <ul>
    <li v-for="(value, key, index) in user" :key="key">
      {{ index }}. {{ key }}: {{ value }}
    </li>
  </ul>
  
  <!-- 🔥 遍历数字 -->
  <span v-for="n in 10" :key="n">{{ n }}</span>
  
  <!-- template包裹 -->
  <template v-for="item in items" :key="item.id">
    <li>{{ item.text }}</li>
    <li class="divider"></li>
  </template>
  
  <!-- ⚠️ v-if和v-for不要一起使用 -->
  <!-- ❌ 不推荐 -->
  <li v-for="item in items" v-if="item.isActive" :key="item.id">
    {{ item.text }}
  </li>
  
  <!-- ✅ 推荐：使用计算属性 -->
  <li v-for="item in activeItems" :key="item.id">
    {{ item.text }}
  </li>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue';

interface Item {
  id: number;
  text: string;
  isActive: boolean;
}

const items = ref<Item[]>([
  { id: 1, text: 'Item 1', isActive: true },
  { id: 2, text: 'Item 2', isActive: false },
  { id: 3, text: 'Item 3', isActive: true }
]);

const user = ref({
  name: 'zhangsan',
  age: 25,
  city: 'Beijing'
});

const activeItems = computed(() => {
  return items.value.filter(item => item.isActive);
});
</script>
```

### 8.2 数组变更检测

```vue
<script setup lang="ts">
import { ref } from 'vue';

const items = ref([1, 2, 3, 4, 5]);

// 🔥 变更方法（会触发视图更新）
items.value.push(6);           // 添加
items.value.pop();             // 删除最后一个
items.value.shift();           // 删除第一个
items.value.unshift(0);        // 添加到开头
items.value.splice(2, 1);      // 删除
items.value.sort();            // 排序
items.value.reverse();         // 反转

// 🔥 替换数组
items.value = items.value.filter(item => item > 2);
items.value = items.value.map(item => item * 2);

// ⚠️ 直接修改索引不会触发更新（Vue 2的问题，Vue 3已解决）
items.value[0] = 100; // Vue 3中可以触发更新
</script>
```

---

## 9. 事件处理

### 9.1 监听事件

```vue
<template>
  <!-- 🔥 内联处理器 -->
  <button @click="count++">增加</button>
  
  <!-- 🔥 方法处理器 -->
  <button @click="increment">增加</button>
  
  <!-- 传递参数 -->
  <button @click="say('hello')">打招呼</button>
  
  <!-- 访问事件对象 -->
  <button @click="handleClick">点击</button>
  <button @click="handleClickWithArg('test', $event)">点击</button>
  
  <!-- 多个处理器 -->
  <button @click="one($event), two($event)">点击</button>
</template>

<script setup lang="ts">
import { ref } from 'vue';

const count = ref(0);

const increment = () => {
  count.value++;
};

const say = (message: string) => {
  console.log(message);
};

const handleClick = (event: MouseEvent) => {
  console.log(event.target);
};

const handleClickWithArg = (arg: string, event: MouseEvent) => {
  console.log(arg, event);
};

const one = (event: MouseEvent) => {
  console.log('one');
};

const two = (event: MouseEvent) => {
  console.log('two');
};
</script>
```

### 9.2 事件修饰符

```vue
<template>
  <!-- 🔥 .stop - 阻止事件冒泡 -->
  <div @click="handleParent">
    <button @click.stop="handleChild">点击</button>
  </div>
  
  <!-- 🔥 .prevent - 阻止默认行为 -->
  <form @submit.prevent="handleSubmit">
    <button type="submit">提交</button>
  </form>
  
  <!-- 🔥 .capture - 捕获模式 -->
  <div @click.capture="handleCapture">
    <button>点击</button>
  </div>
  
  <!-- 🔥 .self - 只在元素自身触发 -->
  <div @click.self="handleSelf">
    <button>点击</button>
  </div>
  
  <!-- 🔥 .once - 只触发一次 -->
  <button @click.once="handleOnce">只触发一次</button>
  
  <!-- 🔥 .passive - 提升滚动性能 -->
  <div @scroll.passive="handleScroll">滚动区域</div>
  
  <!-- 修饰符可以链式调用 -->
  <button @click.stop.prevent="handleClick">点击</button>
</template>

<script setup lang="ts">
const handleParent = () => console.log('parent');
const handleChild = () => console.log('child');
const handleSubmit = () => console.log('submit');
const handleCapture = () => console.log('capture');
const handleSelf = () => console.log('self');
const handleOnce = () => console.log('once');
const handleScroll = () => console.log('scroll');
const handleClick = () => console.log('click');
</script>
```

### 9.3 按键修饰符

```vue
<template>
  <!-- 🔥 按键修饰符 -->
  <input @keyup.enter="handleEnter">
  <input @keyup.tab="handleTab">
  <input @keyup.delete="handleDelete">
  <input @keyup.esc="handleEsc">
  <input @keyup.space="handleSpace">
  <input @keyup.up="handleUp">
  <input @keyup.down="handleDown">
  <input @keyup.left="handleLeft">
  <input @keyup.right="handleRight">
  
  <!-- 🔥 系统修饰键 -->
  <input @keyup.ctrl="handleCtrl">
  <input @keyup.alt="handleAlt">
  <input @keyup.shift="handleShift">
  <input @keyup.meta="handleMeta">
  
  <!-- 组合键 -->
  <input @keyup.ctrl.enter="handleCtrlEnter">
  
  <!-- 🔥 .exact修饰符 -->
  <button @click.ctrl.exact="handleCtrlClick">Ctrl+点击</button>
  
  <!-- 🔥 鼠标按键修饰符 -->
  <button @click.left="handleLeftClick">左键</button>
  <button @click.right="handleRightClick">右键</button>
  <button @click.middle="handleMiddleClick">中键</button>
</template>

<script setup lang="ts">
const handleEnter = () => console.log('enter');
const handleTab = () => console.log('tab');
const handleDelete = () => console.log('delete');
const handleEsc = () => console.log('esc');
const handleSpace = () => console.log('space');
const handleUp = () => console.log('up');
const handleDown = () => console.log('down');
const handleLeft = () => console.log('left');
const handleRight = () => console.log('right');
const handleCtrl = () => console.log('ctrl');
const handleAlt = () => console.log('alt');
const handleShift = () => console.log('shift');
const handleMeta = () => console.log('meta');
const handleCtrlEnter = () => console.log('ctrl+enter');
const handleCtrlClick = () => console.log('ctrl+click');
const handleLeftClick = () => console.log('left click');
const handleRightClick = () => console.log('right click');
const handleMiddleClick = () => console.log('middle click');
</script>
```

---

## 10. 表单输入绑定

### 10.1 v-model基础

```vue
<template>
  <!-- 🔥 文本输入 -->
  <input v-model="text" placeholder="输入文本">
  <p>{{ text }}</p>
  
  <!-- 🔥 多行文本 -->
  <textarea v-model="message" placeholder="输入多行文本"></textarea>
  <p>{{ message }}</p>
  
  <!-- 🔥 复选框 -->
  <input type="checkbox" v-model="checked">
  <p>{{ checked }}</p>
  
  <!-- 多个复选框 -->
  <input type="checkbox" value="apple" v-model="checkedFruits">
  <input type="checkbox" value="banana" v-model="checkedFruits">
  <input type="checkbox" value="orange" v-model="checkedFruits">
  <p>{{ checkedFruits }}</p>
  
  <!-- 🔥 单选按钮 -->
  <input type="radio" value="male" v-model="gender">
  <input type="radio" value="female" v-model="gender">
  <p>{{ gender }}</p>
  
  <!-- 🔥 选择框 -->
  <select v-model="selected">
    <option disabled value="">请选择</option>
    <option value="a">A</option>
    <option value="b">B</option>
    <option value="c">C</option>
  </select>
  <p>{{ selected }}</p>
  
  <!-- 多选 -->
  <select v-model="multiSelected" multiple>
    <option value="a">A</option>
    <option value="b">B</option>
    <option value="c">C</option>
  </select>
  <p>{{ multiSelected }}</p>
</template>

<script setup lang="ts">
import { ref } from 'vue';

const text = ref('');
const message = ref('');
const checked = ref(false);
const checkedFruits = ref<string[]>([]);
const gender = ref('');
const selected = ref('');
const multiSelected = ref<string[]>([]);
</script>
```

### 10.2 v-model修饰符

```vue
<template>
  <!-- 🔥 .lazy - 在change事件后同步 -->
  <input v-model.lazy="text">
  
  <!-- 🔥 .number - 转换为数字 -->
  <input v-model.number="age" type="number">
  
  <!-- 🔥 .trim - 去除首尾空格 -->
  <input v-model.trim="message">
  
  <!-- 组合使用 -->
  <input v-model.lazy.trim="text">
</template>

<script setup lang="ts">
import { ref } from 'vue';

const text = ref('');
const age = ref(0);
const message = ref('');
</script>
```

---

## 📝 学习检查清单

- [ ] 理解Vue 3的核心概念
- [ ] 掌握模板语法和指令
- [ ] 掌握响应式系统（ref和reactive）
- [ ] 理解计算属性和侦听器
- [ ] 掌握Class和Style绑定
- [ ] 掌握条件渲染和列表渲染
- [ ] 掌握事件处理和修饰符
- [ ] 掌握表单输入绑定

---

## 🔗 相关资源

- [Vue 3官方文档](https://cn.vuejs.org/)
- [Vue 3 Playground](https://play.vuejs.org/)
- [Vue Mastery](https://www.vuemastery.com/)
- [Vue School](https://vueschool.io/)

---

@author erik.zhou
