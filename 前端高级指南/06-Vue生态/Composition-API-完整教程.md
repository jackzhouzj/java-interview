# Composition API - 完整教程

> @author erik.zhou

## 📚 技术概述

### 版本信息
- **Vue版本**：3.x
- **API类型**：Composition API
- **推荐语法**：`<script setup>`

### 学习难度
- **难度等级**：⭐⭐⭐⭐ (较难)
- **预计学习时间**：15-25小时
- **重要程度**：⭐⭐⭐⭐⭐ (必学)

### 前置知识
- Vue 3基础
- JavaScript ES6+
- TypeScript基础（推荐）

## 🎯 学习目标

完成本教程后，你将能够：

- [ ] 理解Composition API的设计理念
- [ ] 掌握setup函数和`<script setup>`
- [ ] 掌握响应式API
- [ ] 理解生命周期钩子
- [ ] 掌握依赖注入
- [ ] 能够编写组合式函数
- [ ] 理解Composition API最佳实践

## 📖 目录

1. [Composition API简介](#1-composition-api简介)
2. [setup函数](#2-setup函数)
3. [响应式API](#3-响应式api)
4. [生命周期钩子](#4-生命周期钩子)
5. [依赖注入](#5-依赖注入)
6. [组合式函数](#6-组合式函数)
7. [最佳实践](#7-最佳实践)

---

## 1. Composition API简介

### 1.1 为什么需要Composition API

```typescript
// ❌ Options API的问题：逻辑分散
export default {
  data() {
    return {
      // 用户相关
      userName: '',
      userAge: 0,
      // 文章相关
      postTitle: '',
      postContent: ''
    };
  },
  methods: {
    // 用户相关
    fetchUser() {},
    updateUser() {},
    // 文章相关
    fetchPost() {},
    updatePost() {}
  },
  computed: {
    // 用户相关
    userInfo() {},
    // 文章相关
    postInfo() {}
  }
};

// ✅ Composition API：逻辑组织更清晰
import { ref, computed } from 'vue';

// 用户相关逻辑
const userName = ref('');
const userAge = ref(0);
const userInfo = computed(() => `${userName.value}, ${userAge.value}`);
const fetchUser = () => {};
const updateUser = () => {};

// 文章相关逻辑
const postTitle = ref('');
const postContent = ref('');
const postInfo = computed(() => `${postTitle.value}: ${postContent.value}`);
const fetchPost = () => {};
const updatePost = () => {};
```

### 1.2 核心优势

**🔥 优势**：
- **更好的逻辑复用**：通过组合式函数
- **更好的类型推导**：TypeScript支持更好
- **更灵活的代码组织**：相关逻辑放在一起
- **更小的打包体积**：Tree-shaking友好

---

## 2. setup函数

### 2.1 基本用法

```vue
<script lang="ts">
import { ref, defineComponent } from 'vue';

export default defineComponent({
  // 🔥 setup函数
  setup(props, context) {
    const count = ref(0);
    
    const increment = () => {
      count.value++;
    };
    
    // 返回给模板使用
    return {
      count,
      increment
    };
  }
});
</script>

<template>
  <div>
    <p>{{ count }}</p>
    <button @click="increment">增加</button>
  </div>
</template>
```

### 2.2 `<script setup>`语法糖

```vue
<script setup lang="ts">
// 🔥 自动导出，无需return
import { ref } from 'vue';

const count = ref(0);

const increment = () => {
  count.value++;
};
</script>

<template>
  <div>
    <p>{{ count }}</p>
    <button @click="increment">增加</button>
  </div>
</template>
```

### 2.3 defineProps和defineEmits

```vue
<script setup lang="ts">
// 🔥 定义props
interface Props {
  title: string;
  count?: number;
}

const props = defineProps<Props>();

// 带默认值
const propsWithDefaults = withDefaults(defineProps<Props>(), {
  count: 0
});

// 🔥 定义emits
interface Emits {
  (e: 'update', value: number): void;
  (e: 'delete', id: string): void;
}

const emit = defineEmits<Emits>();

// 触发事件
const handleUpdate = () => {
  emit('update', props.count! + 1);
};

const handleDelete = () => {
  emit('delete', '123');
};
</script>

<template>
  <div>
    <h1>{{ title }}</h1>
    <p>{{ count }}</p>
    <button @click="handleUpdate">更新</button>
    <button @click="handleDelete">删除</button>
  </div>
</template>
```

### 2.4 defineExpose

```vue
<script setup lang="ts">
import { ref } from 'vue';

const count = ref(0);
const message = ref('Hello');

const increment = () => {
  count.value++;
};

// 🔥 暴露给父组件
defineExpose({
  count,
  increment
});
</script>

<!-- 父组件 -->
<script setup lang="ts">
import { ref } from 'vue';
import ChildComponent from './ChildComponent.vue';

const childRef = ref<InstanceType<typeof ChildComponent>>();

const handleClick = () => {
  // 访问子组件暴露的属性和方法
  console.log(childRef.value?.count);
  childRef.value?.increment();
};
</script>

<template>
  <ChildComponent ref="childRef" />
  <button @click="handleClick">调用子组件方法</button>
</template>
```

---

## 3. 响应式API

### 3.1 ref()

```typescript
import { ref, Ref } from 'vue';

// 🔥 基本类型
const count = ref(0);
const message = ref('Hello');
const isActive = ref(true);

// 访问和修改
console.log(count.value); // 0
count.value++;

// 🔥 对象类型
interface User {
  name: string;
  age: number;
}

const user = ref<User>({
  name: 'zhangsan',
  age: 25
});

// 修改属性
user.value.name = 'lisi';

// 替换整个对象
user.value = {
  name: 'wangwu',
  age: 30
};

// 🔥 数组类型
const list = ref<number[]>([1, 2, 3]);
list.value.push(4);

// 🔥 类型注解
const count: Ref<number> = ref(0);
```

### 3.2 reactive()

```typescript
import { reactive } from 'vue';

// 🔥 创建响应式对象
interface State {
  count: number;
  message: string;
  user: {
    name: string;
    age: number;
  };
}

const state = reactive<State>({
  count: 0,
  message: 'Hello',
  user: {
    name: 'zhangsan',
    age: 25
  }
});

// 直接访问属性
console.log(state.count);
state.count++;

// 嵌套对象
state.user.name = 'lisi';

// ⚠️ 不能替换整个对象
// state = reactive({ count: 1 }); // 错误！
```

### 3.3 computed()

```typescript
import { ref, computed } from 'vue';

const firstName = ref('Zhang');
const lastName = ref('San');

// 🔥 只读计算属性
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
console.log(fullName.value); // 'Zhang San'
fullNameWritable.value = 'Li Si';
```

### 3.4 readonly()

```typescript
import { reactive, readonly } from 'vue';

const state = reactive({
  count: 0
});

// 🔥 创建只读代理
const readonlyState = readonly(state);

// 可以读取
console.log(readonlyState.count); // 0

// 不能修改
// readonlyState.count++; // 警告！
```

### 3.5 watch()

```typescript
import { ref, watch } from 'vue';

const count = ref(0);
const message = ref('Hello');

// 🔥 侦听单个ref
watch(count, (newValue, oldValue) => {
  console.log(`count: ${oldValue} -> ${newValue}`);
});

// 🔥 侦听多个数据源
watch([count, message], ([newCount, newMessage], [oldCount, oldMessage]) => {
  console.log('count或message变化了');
});

// 🔥 侦听getter函数
const user = ref({ name: 'zhangsan', age: 25 });

watch(
  () => user.value.name,
  (newName, oldName) => {
    console.log(`name: ${oldName} -> ${newName}`);
  }
);

// 🔥 深度侦听
watch(
  user,
  (newUser, oldUser) => {
    console.log('user变化了');
  },
  { deep: true }
);

// 🔥 立即执行
watch(
  count,
  (newValue) => {
    console.log('count:', newValue);
  },
  { immediate: true }
);

// 🔥 停止侦听
const stop = watch(count, (newValue) => {
  console.log('count:', newValue);
});

// 停止侦听
stop();
```

### 3.6 watchEffect()

```typescript
import { ref, watchEffect } from 'vue';

const count = ref(0);
const message = ref('Hello');

// 🔥 自动追踪依赖
watchEffect(() => {
  console.log(`count: ${count.value}, message: ${message.value}`);
});

// 🔥 清理副作用
watchEffect((onCleanup) => {
  const timer = setTimeout(() => {
    console.log('延迟执行');
  }, 1000);
  
  // 清理函数
  onCleanup(() => {
    clearTimeout(timer);
  });
});

// 🔥 停止侦听
const stop = watchEffect(() => {
  console.log('count:', count.value);
});

stop();
```

---

## 4. 生命周期钩子

### 4.1 生命周期对比

```typescript
// Options API -> Composition API
// beforeCreate -> setup()
// created -> setup()
// beforeMount -> onBeforeMount
// mounted -> onMounted
// beforeUpdate -> onBeforeUpdate
// updated -> onUpdated
// beforeUnmount -> onBeforeUnmount
// unmounted -> onUnmounted
// errorCaptured -> onErrorCaptured
// renderTracked -> onRenderTracked
// renderTriggered -> onRenderTriggered
// activated -> onActivated
// deactivated -> onDeactivated
```

### 4.2 使用生命周期钩子

```vue
<script setup lang="ts">
import {
  onBeforeMount,
  onMounted,
  onBeforeUpdate,
  onUpdated,
  onBeforeUnmount,
  onUnmounted,
  onErrorCaptured,
  onRenderTracked,
  onRenderTriggered,
  onActivated,
  onDeactivated
} from 'vue';

// 🔥 组件挂载前
onBeforeMount(() => {
  console.log('组件即将挂载');
});

// 🔥 组件挂载后
onMounted(() => {
  console.log('组件已挂载');
  // 访问DOM、发起请求等
});

// 🔥 组件更新前
onBeforeUpdate(() => {
  console.log('组件即将更新');
});

// 🔥 组件更新后
onUpdated(() => {
  console.log('组件已更新');
});

// 🔥 组件卸载前
onBeforeUnmount(() => {
  console.log('组件即将卸载');
  // 清理定时器、取消请求等
});

// 🔥 组件卸载后
onUnmounted(() => {
  console.log('组件已卸载');
});

// 🔥 捕获错误
onErrorCaptured((err, instance, info) => {
  console.error('捕获到错误:', err);
  return false; // 阻止错误继续传播
});

// 🔥 keep-alive激活
onActivated(() => {
  console.log('组件被激活');
});

// 🔥 keep-alive停用
onDeactivated(() => {
  console.log('组件被停用');
});
</script>
```

---

## 5. 依赖注入

### 5.1 provide和inject

```vue
<!-- 祖先组件 -->
<script setup lang="ts">
import { provide, ref } from 'vue';

// 🔥 提供数据
const theme = ref('dark');
provide('theme', theme);

// 提供方法
const updateTheme = (newTheme: string) => {
  theme.value = newTheme;
};
provide('updateTheme', updateTheme);

// 提供对象
provide('user', {
  name: 'zhangsan',
  age: 25
});
</script>

<!-- 后代组件 -->
<script setup lang="ts">
import { inject, Ref } from 'vue';

// 🔥 注入数据
const theme = inject<Ref<string>>('theme');
const updateTheme = inject<(theme: string) => void>('updateTheme');

// 带默认值
const user = inject('user', { name: 'guest', age: 0 });

// 使用
console.log(theme?.value);
updateTheme?.('light');
</script>
```

### 5.2 应用级provide

```typescript
// main.ts
import { createApp } from 'vue';
import App from './App.vue';

const app = createApp(App);

// 🔥 应用级provide
app.provide('globalConfig', {
  apiUrl: 'https://api.example.com',
  timeout: 5000
});

app.mount('#app');
```

---

## 6. 组合式函数

### 6.1 创建组合式函数

```typescript
// composables/useCounter.ts
import { ref, computed } from 'vue';

// 🔥 组合式函数
export function useCounter(initialValue = 0) {
  const count = ref(initialValue);
  
  const double = computed(() => count.value * 2);
  
  const increment = () => {
    count.value++;
  };
  
  const decrement = () => {
    count.value--;
  };
  
  const reset = () => {
    count.value = initialValue;
  };
  
  return {
    count,
    double,
    increment,
    decrement,
    reset
  };
}

// 使用
<script setup lang="ts">
import { useCounter } from './composables/useCounter';

const { count, double, increment, decrement, reset } = useCounter(10);
</script>
```

### 6.2 useFetch示例

```typescript
// composables/useFetch.ts
import { ref, Ref } from 'vue';

interface UseFetchReturn<T> {
  data: Ref<T | null>;
  error: Ref<Error | null>;
  loading: Ref<boolean>;
  refetch: () => Promise<void>;
}

export function useFetch<T>(url: string): UseFetchReturn<T> {
  const data = ref<T | null>(null);
  const error = ref<Error | null>(null);
  const loading = ref(false);
  
  const fetchData = async () => {
    loading.value = true;
    error.value = null;
    
    try {
      const response = await fetch(url);
      if (!response.ok) {
        throw new Error('Failed to fetch');
      }
      data.value = await response.json();
    } catch (err) {
      error.value = err as Error;
    } finally {
      loading.value = false;
    }
  };
  
  // 立即执行
  fetchData();
  
  return {
    data,
    error,
    loading,
    refetch: fetchData
  };
}

// 使用
<script setup lang="ts">
import { useFetch } from './composables/useFetch';

interface User {
  id: number;
  name: string;
}

const { data, error, loading, refetch } = useFetch<User>('/api/user/1');
</script>

<template>
  <div v-if="loading">加载中...</div>
  <div v-else-if="error">错误: {{ error.message }}</div>
  <div v-else-if="data">
    <p>{{ data.name }}</p>
    <button @click="refetch">刷新</button>
  </div>
</template>
```

### 6.3 useLocalStorage示例

```typescript
// composables/useLocalStorage.ts
import { ref, watch, Ref } from 'vue';

export function useLocalStorage<T>(
  key: string,
  defaultValue: T
): Ref<T> {
  // 从localStorage读取初始值
  const storedValue = localStorage.getItem(key);
  const data = ref<T>(
    storedValue ? JSON.parse(storedValue) : defaultValue
  ) as Ref<T>;
  
  // 监听变化，同步到localStorage
  watch(
    data,
    (newValue) => {
      localStorage.setItem(key, JSON.stringify(newValue));
    },
    { deep: true }
  );
  
  return data;
}

// 使用
<script setup lang="ts">
import { useLocalStorage } from './composables/useLocalStorage';

const theme = useLocalStorage('theme', 'light');
const user = useLocalStorage('user', { name: '', age: 0 });
</script>
```

---

## 7. 最佳实践

### 7.1 命名规范

```typescript
// ✅ 组合式函数以use开头
export function useCounter() {}
export function useFetch() {}
export function useLocalStorage() {}

// ✅ ref变量使用名词
const count = ref(0);
const user = ref({});

// ✅ 函数使用动词
const increment = () => {};
const fetchData = () => {};
```

### 7.2 组织代码

```vue
<script setup lang="ts">
// 1. 导入
import { ref, computed, watch, onMounted } from 'vue';
import { useRouter } from 'vue-router';

// 2. Props和Emits
const props = defineProps<{ id: string }>();
const emit = defineEmits<{ (e: 'update'): void }>();

// 3. 组合式函数
const router = useRouter();

// 4. 响应式数据
const count = ref(0);
const message = ref('');

// 5. 计算属性
const double = computed(() => count.value * 2);

// 6. 方法
const increment = () => {
  count.value++;
};

// 7. 侦听器
watch(count, (newValue) => {
  console.log('count:', newValue);
});

// 8. 生命周期
onMounted(() => {
  console.log('mounted');
});
</script>
```

---

## 📝 学习检查清单

- [ ] 理解Composition API的设计理念
- [ ] 掌握setup函数和`<script setup>`
- [ ] 掌握响应式API（ref、reactive、computed等）
- [ ] 理解生命周期钩子
- [ ] 掌握依赖注入（provide/inject）
- [ ] 能够编写组合式函数
- [ ] 了解Composition API最佳实践

---

## 🔗 相关资源

- [Vue 3 Composition API文档](https://cn.vuejs.org/guide/extras/composition-api-faq.html)
- [VueUse](https://vueuse.org/) - 组合式函数集合
- [Vue Composition API RFC](https://github.com/vuejs/rfcs/blob/master/active-rfcs/0013-composition-api.md)

---

@author erik.zhou
