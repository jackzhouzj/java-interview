# React Hooks - 完整教程

> @author erik.zhou

## 📚 技术概述

### 版本信息
- **React版本**：18.x
- **最新稳定版**：18.2.0
- **推荐版本**：18.2+

### 学习难度
- **难度等级**：⭐⭐⭐⭐ (较难)
- **预计学习时间**：20-30小时
- **重要程度**：⭐⭐⭐⭐⭐ (必学)

### 前置知识
- JavaScript ES6+
- React基础
- 函数式编程概念
- 闭包和作用域

## 🎯 学习目标

完成本教程后，你将能够：

- [ ] 理解Hooks的设计理念
- [ ] 掌握所有内置Hooks的使用
- [ ] 能够编写自定义Hooks
- [ ] 理解Hooks的最佳实践
- [ ] 掌握Hooks的性能优化
- [ ] 能够解决常见的Hooks问题

## 📖 目录

1. [Hooks简介](#1-hooks简介)
2. [useState](#2-usestate)
3. [useEffect](#3-useeffect)
4. [useContext](#4-usecontext)
5. [useReducer](#5-usereducer)
6. [useCallback](#6-usecallback)
7. [useMemo](#7-usememo)
8. [useRef](#8-useref)
9. [自定义Hooks](#9-自定义hooks)
10. [最佳实践](#10-最佳实践)

---

## 1. Hooks简介

### 1.1 什么是Hooks

Hooks是React 16.8引入的新特性，让你在不编写class的情况下使用state和其他React特性。

**核心优势**：
- 🔥 **逻辑复用**：更容易复用状态逻辑
- 🔥 **代码简洁**：减少样板代码
- 🔥 **更好的组织**：相关逻辑可以放在一起
- 🔥 **更容易理解**：避免this的困扰

### 1.2 Hooks规则

```typescript
// ✅ 正确：在函数组件顶层调用
function MyComponent() {
  const [count, setCount] = useState(0);
  useEffect(() => {
    // ...
  });
  return <div>{count}</div>;
}

// ❌ 错误：在条件语句中调用
function MyComponent() {
  if (condition) {
    const [count, setCount] = useState(0); // 错误！
  }
}

// ❌ 错误：在循环中调用
function MyComponent() {
  for (let i = 0; i < 10; i++) {
    useEffect(() => {}); // 错误！
  }
}

// ❌ 错误：在普通函数中调用
function notAComponent() {
  const [count, setCount] = useState(0); // 错误！
}
```

---

## 2. useState

### 2.1 基础用法

```typescript
import { useState } from 'react';

function Counter() {
  // 🔥 声明state变量
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>
        Increment
      </button>
    </div>
  );
}
```

### 2.2 多个state变量

```typescript
function UserProfile() {
  const [name, setName] = useState('');
  const [age, setAge] = useState(0);
  const [email, setEmail] = useState('');

  return (
    <form>
      <input 
        value={name} 
        onChange={(e) => setName(e.target.value)} 
      />
      <input 
        type="number"
        value={age} 
        onChange={(e) => setAge(Number(e.target.value))} 
      />
      <input 
        type="email"
        value={email} 
        onChange={(e) => setEmail(e.target.value)} 
      />
    </form>
  );
}
```

### 2.3 对象和数组state

```typescript
// 🔥 对象state
function UserForm() {
  const [user, setUser] = useState({
    name: '',
    email: '',
    age: 0
  });

  const updateUser = (field: string, value: any) => {
    setUser(prev => ({
      ...prev,
      [field]: value
    }));
  };

  return (
    <input 
      value={user.name}
      onChange={(e) => updateUser('name', e.target.value)}
    />
  );
}

// 🔥 数组state
function TodoList() {
  const [todos, setTodos] = useState<string[]>([]);

  const addTodo = (todo: string) => {
    setTodos(prev => [...prev, todo]);
  };

  const removeTodo = (index: number) => {
    setTodos(prev => prev.filter((_, i) => i !== index));
  };

  return (
    <ul>
      {todos.map((todo, index) => (
        <li key={index}>
          {todo}
          <button onClick={() => removeTodo(index)}>Delete</button>
        </li>
      ))}
    </ul>
  );
}
```

### 2.4 函数式更新

```typescript
function Counter() {
  const [count, setCount] = useState(0);

  // ✅ 推荐：使用函数式更新
  const increment = () => {
    setCount(prev => prev + 1);
  };

  // ❌ 不推荐：直接使用当前值
  const incrementBad = () => {
    setCount(count + 1);
  };

  // 多次更新
  const incrementThree = () => {
    setCount(prev => prev + 1);
    setCount(prev => prev + 1);
    setCount(prev => prev + 1);
  };

  return <button onClick={incrementThree}>+3</button>;
}
```

---

## 3. useEffect

### 3.1 基础用法

```typescript
import { useState, useEffect } from 'react';

function DataFetcher() {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);

  // 🔥 组件挂载后执行
  useEffect(() => {
    fetch('https://api.example.com/data')
      .then(res => res.json())
      .then(data => {
        setData(data);
        setLoading(false);
      });
  }, []); // 空依赖数组：只在挂载时执行

  if (loading) return <div>Loading...</div>;
  return <div>{JSON.stringify(data)}</div>;
}
```

### 3.2 依赖数组

```typescript
function SearchResults({ query }: { query: string }) {
  const [results, setResults] = useState([]);

  // 🔥 当query变化时重新执行
  useEffect(() => {
    if (!query) return;

    fetch(`https://api.example.com/search?q=${query}`)
      .then(res => res.json())
      .then(setResults);
  }, [query]); // query变化时执行

  return (
    <ul>
      {results.map(item => (
        <li key={item.id}>{item.name}</li>
      ))}
    </ul>
  );
}
```

### 3.3 清理函数

```typescript
function Timer() {
  const [seconds, setSeconds] = useState(0);

  useEffect(() => {
    // 设置定时器
    const interval = setInterval(() => {
      setSeconds(prev => prev + 1);
    }, 1000);

    // 🔥 清理函数：组件卸载时执行
    return () => {
      clearInterval(interval);
    };
  }, []);

  return <div>Seconds: {seconds}</div>;
}

// WebSocket示例
function ChatRoom({ roomId }: { roomId: string }) {
  useEffect(() => {
    const ws = new WebSocket(`ws://example.com/${roomId}`);

    ws.onmessage = (event) => {
      console.log('Message:', event.data);
    };

    // 清理WebSocket连接
    return () => {
      ws.close();
    };
  }, [roomId]);

  return <div>Chat Room: {roomId}</div>;
}
```

### 3.4 多个useEffect

```typescript
function UserProfile({ userId }: { userId: string }) {
  const [user, setUser] = useState(null);
  const [posts, setPosts] = useState([]);

  // 🔥 分离不同的副作用
  // 获取用户信息
  useEffect(() => {
    fetch(`/api/users/${userId}`)
      .then(res => res.json())
      .then(setUser);
  }, [userId]);

  // 获取用户文章
  useEffect(() => {
    fetch(`/api/users/${userId}/posts`)
      .then(res => res.json())
      .then(setPosts);
  }, [userId]);

  // 设置页面标题
  useEffect(() => {
    if (user) {
      document.title = `${user.name}'s Profile`;
    }
  }, [user]);

  return (
    <div>
      {user && <h1>{user.name}</h1>}
      {posts.map(post => (
        <article key={post.id}>{post.title}</article>
      ))}
    </div>
  );
}
```

---

## 4. useContext

### 4.1 基础用法

```typescript
import { createContext, useContext, useState } from 'react';

// 🔥 创建Context
interface ThemeContextType {
  theme: 'light' | 'dark';
  toggleTheme: () => void;
}

const ThemeContext = createContext<ThemeContextType | undefined>(undefined);

// Provider组件
function ThemeProvider({ children }: { children: React.ReactNode }) {
  const [theme, setTheme] = useState<'light' | 'dark'>('light');

  const toggleTheme = () => {
    setTheme(prev => prev === 'light' ? 'dark' : 'light');
  };

  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}

// 自定义Hook
function useTheme() {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error('useTheme must be used within ThemeProvider');
  }
  return context;
}

// 使用Context
function ThemedButton() {
  const { theme, toggleTheme } = useTheme();

  return (
    <button 
      onClick={toggleTheme}
      style={{ 
        background: theme === 'light' ? '#fff' : '#333',
        color: theme === 'light' ? '#333' : '#fff'
      }}
    >
      Toggle Theme
    </button>
  );
}

// App组件
function App() {
  return (
    <ThemeProvider>
      <ThemedButton />
    </ThemeProvider>
  );
}
```

---

## 5. useReducer

### 5.1 基础用法

```typescript
import { useReducer } from 'react';

// 🔥 定义state类型和action类型
interface State {
  count: number;
}

type Action = 
  | { type: 'increment' }
  | { type: 'decrement' }
  | { type: 'reset' }
  | { type: 'set'; payload: number };

// Reducer函数
function counterReducer(state: State, action: Action): State {
  switch (action.type) {
    case 'increment':
      return { count: state.count + 1 };
    case 'decrement':
      return { count: state.count - 1 };
    case 'reset':
      return { count: 0 };
    case 'set':
      return { count: action.payload };
    default:
      return state;
  }
}

function Counter() {
  const [state, dispatch] = useReducer(counterReducer, { count: 0 });

  return (
    <div>
      <p>Count: {state.count}</p>
      <button onClick={() => dispatch({ type: 'increment' })}>+</button>
      <button onClick={() => dispatch({ type: 'decrement' })}>-</button>
      <button onClick={() => dispatch({ type: 'reset' })}>Reset</button>
      <button onClick={() => dispatch({ type: 'set', payload: 10 })}>
        Set to 10
      </button>
    </div>
  );
}
```

### 5.2 复杂状态管理

```typescript
interface TodoState {
  todos: Todo[];
  filter: 'all' | 'active' | 'completed';
}

type TodoAction =
  | { type: 'add'; payload: string }
  | { type: 'toggle'; payload: number }
  | { type: 'delete'; payload: number }
  | { type: 'setFilter'; payload: 'all' | 'active' | 'completed' };

function todoReducer(state: TodoState, action: TodoAction): TodoState {
  switch (action.type) {
    case 'add':
      return {
        ...state,
        todos: [
          ...state.todos,
          { id: Date.now(), text: action.payload, completed: false }
        ]
      };
    case 'toggle':
      return {
        ...state,
        todos: state.todos.map(todo =>
          todo.id === action.payload
            ? { ...todo, completed: !todo.completed }
            : todo
        )
      };
    case 'delete':
      return {
        ...state,
        todos: state.todos.filter(todo => todo.id !== action.payload)
      };
    case 'setFilter':
      return { ...state, filter: action.payload };
    default:
      return state;
  }
}
```

---

## 6. useCallback

### 6.1 基础用法

```typescript
import { useState, useCallback } from 'react';

function SearchForm() {
  const [query, setQuery] = useState('');

  // 🔥 缓存函数，避免子组件不必要的重渲染
  const handleSearch = useCallback((searchTerm: string) => {
    console.log('Searching for:', searchTerm);
    // 执行搜索逻辑
  }, []); // 空依赖：函数永远不变

  return (
    <div>
      <input 
        value={query}
        onChange={(e) => setQuery(e.target.value)}
      />
      <SearchButton onSearch={handleSearch} />
    </div>
  );
}

// 子组件使用React.memo优化
const SearchButton = React.memo(({ 
  onSearch 
}: { 
  onSearch: (term: string) => void 
}) => {
  console.log('SearchButton rendered');
  return <button onClick={() => onSearch('test')}>Search</button>;
});
```

### 6.2 带依赖的useCallback

```typescript
function ProductList({ category }: { category: string }) {
  const [products, setProducts] = useState([]);

  // 🔥 当category变化时，函数会重新创建
  const fetchProducts = useCallback(async () => {
    const response = await fetch(`/api/products?category=${category}`);
    const data = await response.json();
    setProducts(data);
  }, [category]); // category变化时重新创建

  useEffect(() => {
    fetchProducts();
  }, [fetchProducts]);

  return (
    <div>
      {products.map(product => (
        <div key={product.id}>{product.name}</div>
      ))}
    </div>
  );
}
```

---

## 7. useMemo

### 7.1 基础用法

```typescript
import { useState, useMemo } from 'react';

function ExpensiveComponent({ items }: { items: number[] }) {
  const [filter, setFilter] = useState('');

  // 🔥 缓存计算结果
  const filteredItems = useMemo(() => {
    console.log('Filtering items...');
    return items.filter(item => 
      item.toString().includes(filter)
    );
  }, [items, filter]); // 只在items或filter变化时重新计算

  const sum = useMemo(() => {
    console.log('Calculating sum...');
    return filteredItems.reduce((acc, item) => acc + item, 0);
  }, [filteredItems]);

  return (
    <div>
      <input 
        value={filter}
        onChange={(e) => setFilter(e.target.value)}
        placeholder="Filter..."
      />
      <p>Sum: {sum}</p>
      <ul>
        {filteredItems.map((item, index) => (
          <li key={index}>{item}</li>
        ))}
      </ul>
    </div>
  );
}
```

---

## 8. useRef

### 8.1 访问DOM元素

```typescript
import { useRef, useEffect } from 'react';

function AutoFocusInput() {
  const inputRef = useRef<HTMLInputElement>(null);

  useEffect(() => {
    // 🔥 组件挂载后自动聚焦
    inputRef.current?.focus();
  }, []);

  return <input ref={inputRef} />;
}
```

### 8.2 保存可变值

```typescript
function Timer() {
  const [count, setCount] = useState(0);
  const intervalRef = useRef<number>();

  const start = () => {
    if (intervalRef.current) return;
    
    intervalRef.current = window.setInterval(() => {
      setCount(c => c + 1);
    }, 1000);
  };

  const stop = () => {
    if (intervalRef.current) {
      clearInterval(intervalRef.current);
      intervalRef.current = undefined;
    }
  };

  useEffect(() => {
    return () => stop(); // 清理
  }, []);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={start}>Start</button>
      <button onClick={stop}>Stop</button>
    </div>
  );
}
```

---

## 9. 自定义Hooks

### 9.1 useLocalStorage

```typescript
function useLocalStorage<T>(key: string, initialValue: T) {
  const [storedValue, setStoredValue] = useState<T>(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      console.error(error);
      return initialValue;
    }
  });

  const setValue = (value: T | ((val: T) => T)) => {
    try {
      const valueToStore = value instanceof Function 
        ? value(storedValue) 
        : value;
      setStoredValue(valueToStore);
      window.localStorage.setItem(key, JSON.stringify(valueToStore));
    } catch (error) {
      console.error(error);
    }
  };

  return [storedValue, setValue] as const;
}

// 使用
function App() {
  const [name, setName] = useLocalStorage('name', '');
  
  return (
    <input 
      value={name}
      onChange={(e) => setName(e.target.value)}
    />
  );
}
```

### 9.2 useFetch

```typescript
function useFetch<T>(url: string) {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<Error | null>(null);

  useEffect(() => {
    const fetchData = async () => {
      try {
        setLoading(true);
        const response = await fetch(url);
        const json = await response.json();
        setData(json);
      } catch (err) {
        setError(err as Error);
      } finally {
        setLoading(false);
      }
    };

    fetchData();
  }, [url]);

  return { data, loading, error };
}

// 使用
function UserProfile({ userId }: { userId: string }) {
  const { data, loading, error } = useFetch<User>(
    `/api/users/${userId}`
  );

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;
  if (!data) return null;

  return <div>{data.name}</div>;
}
```

---

## 10. 最佳实践

### 10.1 避免过度优化

```typescript
// ❌ 不必要的优化
function SimpleComponent({ name }: { name: string }) {
  const greeting = useMemo(() => `Hello, ${name}!`, [name]);
  return <div>{greeting}</div>;
}

// ✅ 简单计算不需要useMemo
function SimpleComponent({ name }: { name: string }) {
  const greeting = `Hello, ${name}!`;
  return <div>{greeting}</div>;
}
```

### 10.2 正确使用依赖数组

```typescript
// ✅ 包含所有依赖
function Component({ userId }: { userId: string }) {
  const [user, setUser] = useState(null);

  useEffect(() => {
    fetchUser(userId).then(setUser);
  }, [userId]); // 正确：包含userId

  return <div>{user?.name}</div>;
}
```

---

## 📝 学习检查清单

- [ ] 理解Hooks的设计理念
- [ ] 掌握useState的使用
- [ ] 掌握useEffect和清理函数
- [ ] 理解useContext的使用场景
- [ ] 掌握useCallback和useMemo的优化
- [ ] 能够编写自定义Hooks
- [ ] 了解Hooks的最佳实践

---

## 🔗 相关资源

- [React Hooks官方文档](https://react.dev/reference/react)
- [React Hooks FAQ](https://react.dev/learn)
- [useHooks](https://usehooks.com/)
- [React Hooks Cheatsheet](https://react-hooks-cheatsheet.com/)

---

@author erik.zhou
