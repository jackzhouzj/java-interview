# ES6+新特性 - 完整教程

> @author erik.zhou

## 📚 技术概述

### 版本信息
- **ES6 (ES2015)**：2015年发布，重大更新
- **ES7-ES14**：2016-2023年持续更新
- **推荐学习**：ES6+所有新特性

### 学习难度
- **难度等级**：⭐⭐⭐ (中等)
- **预计学习时间**：20-30小时
- **重要程度**：⭐⭐⭐⭐⭐ (必学)

### 前置知识
- JavaScript基础
- 函数和对象
- 数组操作

## 🎯 学习目标

完成本教程后，你将能够：

- [ ] 掌握let/const和块级作用域
- [ ] 理解箭头函数和this绑定
- [ ] 掌握解构赋值
- [ ] 理解Promise和async/await
- [ ] 掌握模块化
- [ ] 理解Class语法
- [ ] 掌握新的数据结构

## 📖 目录

1. [let和const](#1-let和const)
2. [箭头函数](#2-箭头函数)
3. [模板字符串](#3-模板字符串)
4. [解构赋值](#4-解构赋值)
5. [扩展运算符](#5-扩展运算符)
6. [Promise](#6-promise)
7. [async/await](#7-asyncawait)
8. [Class](#8-class)
9. [模块化](#9-模块化)
10. [新的数据结构](#10-新的数据结构)

---

## 1. let和const

### 1.1 块级作用域

```javascript
// 🔥 var：函数作用域
function testVar() {
  if (true) {
    var x = 10;
  }
  console.log(x); // 10（可以访问）
}

// 🔥 let：块级作用域
function testLet() {
  if (true) {
    let y = 10;
  }
  // console.log(y); // 错误！无法访问
}

// for循环中的区别
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100);
}
// 输出：3, 3, 3

for (let j = 0; j < 3; j++) {
  setTimeout(() => console.log(j), 100);
}
// 输出：0, 1, 2
```

### 1.2 const常量

```javascript
// 🔥 const声明常量
const PI = 3.14159;
// PI = 3.14; // 错误！不能重新赋值

// const对象
const user = { name: 'zhangsan' };
user.name = 'lisi';  // ✅ 可以修改属性
user.age = 25;       // ✅ 可以添加属性
// user = {};        // ❌ 不能重新赋值

// const数组
const arr = [1, 2, 3];
arr.push(4);         // ✅ 可以修改数组
// arr = [];         // ❌ 不能重新赋值

// 冻结对象
const frozenUser = Object.freeze({ name: 'zhangsan' });
// frozenUser.name = 'lisi'; // 严格模式下报错
```

### 1.3 暂时性死区

```javascript
// ⚠️ 暂时性死区（TDZ）
console.log(x); // undefined（var会提升）
var x = 10;

// console.log(y); // 错误！TDZ
let y = 20;

// console.log(z); // 错误！TDZ
const z = 30;
```

---

## 2. 箭头函数

### 2.1 基本语法

```javascript
// 🔥 传统函数
function add(a, b) {
  return a + b;
}

// 🔥 箭头函数
const add = (a, b) => a + b;

// 单个参数可以省略括号
const double = x => x * 2;

// 无参数需要空括号
const greet = () => 'Hello!';

// 多行函数体需要大括号和return
const multiply = (a, b) => {
  const result = a * b;
  return result;
};

// 返回对象需要用括号包裹
const createUser = (name, age) => ({ name, age });
```

### 2.2 this绑定

```javascript
// ⚠️ 箭头函数没有自己的this
const obj = {
  name: 'zhangsan',
  
  // 传统函数：this指向obj
  sayHello: function() {
    console.log(`Hello, ${this.name}`);
  },
  
  // 箭头函数：this继承外层作用域
  sayHi: () => {
    console.log(`Hi, ${this.name}`); // undefined
  },
  
  // 实际应用
  delayedGreet: function() {
    setTimeout(() => {
      // 箭头函数继承delayedGreet的this
      console.log(`Hello, ${this.name}`);
    }, 1000);
  }
};

obj.sayHello(); // "Hello, zhangsan"
obj.sayHi();    // "Hi, undefined"
obj.delayedGreet(); // "Hello, zhangsan"
```

### 2.3 不适合使用箭头函数的场景

```javascript
// ❌ 对象方法
const person = {
  name: 'zhangsan',
  sayName: () => {
    console.log(this.name); // undefined
  }
};

// ❌ 原型方法
Person.prototype.sayName = () => {
  console.log(this.name); // undefined
};

// ❌ 构造函数
const Person = (name) => {
  this.name = name; // 错误！箭头函数不能作为构造函数
};

// ❌ 需要arguments对象
const sum = () => {
  // console.log(arguments); // 错误！箭头函数没有arguments
};

// ✅ 使用剩余参数代替
const sum = (...args) => {
  return args.reduce((total, num) => total + num, 0);
};
```

---

## 3. 模板字符串

### 3.1 基本用法

```javascript
const name = 'zhangsan';
const age = 25;

// 🔥 变量插值
const message = `My name is ${name} and I'm ${age} years old.`;

// 🔥 表达式
const price = 100;
const tax = 0.1;
console.log(`Total: ${price * (1 + tax)}`);

// 🔥 多行字符串
const html = `
  <div class="user">
    <h1>${name}</h1>
    <p>Age: ${age}</p>
  </div>
`;

// 🔥 嵌套模板
const list = ['apple', 'banana', 'orange'];
const listHtml = `
  <ul>
    ${list.map(item => `<li>${item}</li>`).join('')}
  </ul>
`;
```

### 3.2 标签模板

```javascript
// 🔥 标签函数
function highlight(strings, ...values) {
  return strings.reduce((result, str, i) => {
    const value = values[i] ? `<mark>${values[i]}</mark>` : '';
    return result + str + value;
  }, '');
}

const name = 'zhangsan';
const age = 25;
const result = highlight`Name: ${name}, Age: ${age}`;
console.log(result);
// "Name: <mark>zhangsan</mark>, Age: <mark>25</mark>"

// 实用示例：SQL查询
function sql(strings, ...values) {
  // 防止SQL注入
  const escaped = values.map(v => 
    typeof v === 'string' ? v.replace(/'/g, "''") : v
  );
  
  return strings.reduce((query, str, i) => {
    return query + str + (escaped[i] || '');
  }, '');
}

const userId = 1;
const query = sql`SELECT * FROM users WHERE id = ${userId}`;
```

---

## 4. 解构赋值

### 4.1 数组解构

```javascript
// 🔥 基本解构
const [a, b, c] = [1, 2, 3];
console.log(a, b, c); // 1 2 3

// 跳过元素
const [first, , third] = [1, 2, 3];
console.log(first, third); // 1 3

// 剩余元素
const [head, ...tail] = [1, 2, 3, 4, 5];
console.log(head); // 1
console.log(tail); // [2, 3, 4, 5]

// 默认值
const [x = 0, y = 0] = [1];
console.log(x, y); // 1 0

// 交换变量
let m = 1, n = 2;
[m, n] = [n, m];
console.log(m, n); // 2 1
```

### 4.2 对象解构

```javascript
// 🔥 基本解构
const user = { name: 'zhangsan', age: 25, city: 'Beijing' };
const { name, age } = user;
console.log(name, age); // 'zhangsan' 25

// 重命名
const { name: userName, age: userAge } = user;
console.log(userName, userAge); // 'zhangsan' 25

// 默认值
const { name, country = 'China' } = user;
console.log(country); // 'China'

// 剩余属性
const { name: n, ...rest } = user;
console.log(rest); // { age: 25, city: 'Beijing' }

// 嵌套解构
const person = {
  name: 'zhangsan',
  address: {
    city: 'Beijing',
    country: 'China'
  }
};

const { address: { city, country } } = person;
console.log(city, country); // 'Beijing' 'China'
```

### 4.3 函数参数解构

```javascript
// 🔥 对象参数解构
function printUser({ name, age, city = 'Unknown' }) {
  console.log(`${name}, ${age}, ${city}`);
}

printUser({ name: 'zhangsan', age: 25 });
// "zhangsan, 25, Unknown"

// 数组参数解构
function sum([a, b]) {
  return a + b;
}

console.log(sum([1, 2])); // 3
```

---

## 5. 扩展运算符

### 5.1 数组扩展运算符

```javascript
// 🔥 复制数组
const arr1 = [1, 2, 3];
const arr2 = [...arr1];

// 合并数组
const arr3 = [4, 5, 6];
const combined = [...arr1, ...arr3];
console.log(combined); // [1, 2, 3, 4, 5, 6]

// 插入元素
const inserted = [...arr1, 10, ...arr3];
console.log(inserted); // [1, 2, 3, 10, 4, 5, 6]

// 数组转参数
const numbers = [1, 5, 3, 9, 2];
console.log(Math.max(...numbers)); // 9

// 字符串转数组
const chars = [...'hello'];
console.log(chars); // ['h', 'e', 'l', 'l', 'o']
```

### 5.2 对象扩展运算符

```javascript
// 🔥 复制对象
const user = { name: 'zhangsan', age: 25 };
const userCopy = { ...user };

// 合并对象
const address = { city: 'Beijing', country: 'China' };
const fullUser = { ...user, ...address };
console.log(fullUser);
// { name: 'zhangsan', age: 25, city: 'Beijing', country: 'China' }

// 覆盖属性
const updatedUser = { ...user, age: 26 };
console.log(updatedUser); // { name: 'zhangsan', age: 26 }

// 添加属性
const userWithEmail = { ...user, email: 'zhangsan@example.com' };
```

### 5.3 剩余参数

```javascript
// 🔥 函数剩余参数
function sum(...numbers) {
  return numbers.reduce((total, num) => total + num, 0);
}

console.log(sum(1, 2, 3, 4, 5)); // 15

// 结合普通参数
function multiply(multiplier, ...numbers) {
  return numbers.map(num => num * multiplier);
}

console.log(multiply(2, 1, 2, 3)); // [2, 4, 6]
```


## 6. Promise

### 6.1 Promise基础

```javascript
// 🔥 创建Promise
const promise = new Promise((resolve, reject) => {
  setTimeout(() => {
    const success = true;
    
    if (success) {
      resolve('操作成功');
    } else {
      reject('操作失败');
    }
  }, 1000);
});

// 使用Promise
promise
  .then(result => {
    console.log(result); // '操作成功'
    return result + '!';
  })
  .then(result => {
    console.log(result); // '操作成功!'
  })
  .catch(error => {
    console.error(error);
  })
  .finally(() => {
    console.log('完成');
  });
```

### 6.2 Promise链式调用

```javascript
// 🔥 Promise链
function fetchUser(userId) {
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve({ id: userId, name: 'zhangsan' });
    }, 1000);
  });
}

function fetchPosts(userId) {
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve([
        { id: 1, title: 'Post 1' },
        { id: 2, title: 'Post 2' }
      ]);
    }, 1000);
  });
}

// 链式调用
fetchUser(1)
  .then(user => {
    console.log('User:', user);
    return fetchPosts(user.id);
  })
  .then(posts => {
    console.log('Posts:', posts);
  })
  .catch(error => {
    console.error('Error:', error);
  });
```

### 6.3 Promise静态方法

```javascript
// 🔥 Promise.all（全部成功才成功）
const promise1 = Promise.resolve(1);
const promise2 = Promise.resolve(2);
const promise3 = Promise.resolve(3);

Promise.all([promise1, promise2, promise3])
  .then(results => {
    console.log(results); // [1, 2, 3]
  });

// 🔥 Promise.race（第一个完成的）
Promise.race([promise1, promise2, promise3])
  .then(result => {
    console.log(result); // 1（第一个完成的）
  });

// 🔥 Promise.allSettled（等待全部完成）
const promises = [
  Promise.resolve(1),
  Promise.reject('error'),
  Promise.resolve(3)
];

Promise.allSettled(promises)
  .then(results => {
    console.log(results);
    // [
    //   { status: 'fulfilled', value: 1 },
    //   { status: 'rejected', reason: 'error' },
    //   { status: 'fulfilled', value: 3 }
    // ]
  });

// 🔥 Promise.any（第一个成功的）
Promise.any(promises)
  .then(result => {
    console.log(result); // 1（第一个成功的）
  });
```

---

## 7. async/await

### 7.1 基本用法

```javascript
// 🔥 async函数返回Promise
async function fetchData() {
  return 'data';
}

fetchData().then(data => console.log(data)); // 'data'

// 🔥 await等待Promise
async function getData() {
  const data = await fetchData();
  console.log(data); // 'data'
}

// 实际示例
async function fetchUser(userId) {
  const response = await fetch(`/api/users/${userId}`);
  const user = await response.json();
  return user;
}
```

### 7.2 错误处理

```javascript
// 🔥 try...catch处理错误
async function fetchUserSafe(userId) {
  try {
    const response = await fetch(`/api/users/${userId}`);
    
    if (!response.ok) {
      throw new Error('Failed to fetch user');
    }
    
    const user = await response.json();
    return user;
  } catch (error) {
    console.error('Error:', error.message);
    return null;
  }
}

// 使用
const user = await fetchUserSafe(1);
```

### 7.3 并发请求

```javascript
// ❌ 串行执行（慢）
async function fetchDataSerial() {
  const user = await fetchUser(1);      // 等待1秒
  const posts = await fetchPosts(1);    // 再等待1秒
  const comments = await fetchComments(1); // 再等待1秒
  // 总共3秒
  return { user, posts, comments };
}

// ✅ 并行执行（快）
async function fetchDataParallel() {
  const [user, posts, comments] = await Promise.all([
    fetchUser(1),
    fetchPosts(1),
    fetchComments(1)
  ]);
  // 总共1秒（同时执行）
  return { user, posts, comments };
}

// 🔥 顺序执行多个异步操作
async function processItems(items) {
  const results = [];
  
  for (const item of items) {
    const result = await processItem(item);
    results.push(result);
  }
  
  return results;
}

// 🔥 并行执行多个异步操作
async function processItemsParallel(items) {
  const promises = items.map(item => processItem(item));
  return await Promise.all(promises);
}
```

---

## 8. Class

### 8.1 类的基本语法

```javascript
// 🔥 定义类
class Person {
  // 构造函数
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }
  
  // 实例方法
  sayHello() {
    console.log(`Hello, I'm ${this.name}`);
  }
  
  // Getter
  get info() {
    return `${this.name}, ${this.age}`;
  }
  
  // Setter
  set info(value) {
    const [name, age] = value.split(', ');
    this.name = name;
    this.age = parseInt(age);
  }
  
  // 静态方法
  static create(name, age) {
    return new Person(name, age);
  }
}

// 使用类
const person = new Person('zhangsan', 25);
person.sayHello(); // "Hello, I'm zhangsan"
console.log(person.info); // "zhangsan, 25"

const person2 = Person.create('lisi', 30);
```

### 8.2 类的继承

```javascript
// 🔥 继承
class Student extends Person {
  constructor(name, age, grade) {
    super(name, age); // 调用父类构造函数
    this.grade = grade;
  }
  
  // 重写方法
  sayHello() {
    super.sayHello(); // 调用父类方法
    console.log(`I'm in grade ${this.grade}`);
  }
  
  // 新方法
  study() {
    console.log(`${this.name} is studying`);
  }
}

const student = new Student('wangwu', 18, 12);
student.sayHello();
// "Hello, I'm wangwu"
// "I'm in grade 12"
student.study(); // "wangwu is studying"
```

### 8.3 私有字段

```javascript
// 🔥 私有字段（ES2022）
class BankAccount {
  #balance = 0; // 私有字段
  
  constructor(initialBalance) {
    this.#balance = initialBalance;
  }
  
  deposit(amount) {
    this.#balance += amount;
  }
  
  withdraw(amount) {
    if (amount <= this.#balance) {
      this.#balance -= amount;
      return true;
    }
    return false;
  }
  
  getBalance() {
    return this.#balance;
  }
}

const account = new BankAccount(1000);
account.deposit(500);
console.log(account.getBalance()); // 1500
// console.log(account.#balance); // 错误！无法访问私有字段
```

---

## 9. 模块化

### 9.1 导出模块

```javascript
// 🔥 命名导出
// math.js
export const PI = 3.14159;

export function add(a, b) {
  return a + b;
}

export function subtract(a, b) {
  return a - b;
}

// 或者统一导出
const PI = 3.14159;
function add(a, b) { return a + b; }
function subtract(a, b) { return a - b; }

export { PI, add, subtract };

// 🔥 默认导出
// calculator.js
export default class Calculator {
  add(a, b) {
    return a + b;
  }
}

// 或者
class Calculator {
  add(a, b) {
    return a + b;
  }
}

export default Calculator;
```

### 9.2 导入模块

```javascript
// 🔥 导入命名导出
import { PI, add, subtract } from './math.js';

console.log(PI); // 3.14159
console.log(add(1, 2)); // 3

// 重命名导入
import { add as sum } from './math.js';

// 导入全部
import * as math from './math.js';
console.log(math.PI);
console.log(math.add(1, 2));

// 🔥 导入默认导出
import Calculator from './calculator.js';

const calc = new Calculator();

// 混合导入
import Calculator, { PI, add } from './module.js';
```

### 9.3 动态导入

```javascript
// 🔥 动态导入（返回Promise）
async function loadModule() {
  const module = await import('./math.js');
  console.log(module.add(1, 2));
}

// 条件导入
if (condition) {
  import('./module-a.js').then(module => {
    module.doSomething();
  });
} else {
  import('./module-b.js').then(module => {
    module.doSomething();
  });
}

// 按需加载
button.addEventListener('click', async () => {
  const { default: Chart } = await import('./chart.js');
  const chart = new Chart();
  chart.render();
});
```

---

## 10. 新的数据结构

### 10.1 Set

```javascript
// 🔥 Set（集合，值唯一）
const set = new Set([1, 2, 3, 3, 4]);
console.log(set); // Set(4) { 1, 2, 3, 4 }

// 添加元素
set.add(5);
set.add(5); // 重复的不会添加

// 删除元素
set.delete(3);

// 检查是否存在
console.log(set.has(2)); // true

// 大小
console.log(set.size); // 4

// 清空
set.clear();

// 遍历
const numbers = new Set([1, 2, 3, 4, 5]);

numbers.forEach(num => console.log(num));

for (let num of numbers) {
  console.log(num);
}

// 数组去重
const arr = [1, 2, 2, 3, 3, 4];
const unique = [...new Set(arr)];
console.log(unique); // [1, 2, 3, 4]
```

### 10.2 Map

```javascript
// 🔥 Map（键值对，键可以是任意类型）
const map = new Map();

// 设置值
map.set('name', 'zhangsan');
map.set('age', 25);
map.set(1, 'number key');
map.set({ id: 1 }, 'object key');

// 获取值
console.log(map.get('name')); // 'zhangsan'

// 检查是否存在
console.log(map.has('age')); // true

// 删除
map.delete('age');

// 大小
console.log(map.size); // 3

// 清空
map.clear();

// 初始化
const user = new Map([
  ['name', 'zhangsan'],
  ['age', 25],
  ['city', 'Beijing']
]);

// 遍历
user.forEach((value, key) => {
  console.log(`${key}: ${value}`);
});

for (let [key, value] of user) {
  console.log(`${key}: ${value}`);
}

// 获取所有键
console.log([...user.keys()]); // ['name', 'age', 'city']

// 获取所有值
console.log([...user.values()]); // ['zhangsan', 25, 'Beijing']

// 获取所有条目
console.log([...user.entries()]);
// [['name', 'zhangsan'], ['age', 25], ['city', 'Beijing']]
```

### 10.3 WeakSet和WeakMap

```javascript
// 🔥 WeakSet（弱引用Set，只能存储对象）
const weakSet = new WeakSet();

let obj1 = { id: 1 };
let obj2 = { id: 2 };

weakSet.add(obj1);
weakSet.add(obj2);

console.log(weakSet.has(obj1)); // true

// 当对象没有其他引用时，会被垃圾回收
obj1 = null; // obj1会被垃圾回收

// 🔥 WeakMap（弱引用Map，键只能是对象）
const weakMap = new WeakMap();

let key = { id: 1 };
weakMap.set(key, 'value');

console.log(weakMap.get(key)); // 'value'

// 当键对象没有其他引用时，会被垃圾回收
key = null; // 键值对会被垃圾回收

// 实际应用：存储DOM元素的私有数据
const privateData = new WeakMap();

class Component {
  constructor(element) {
    privateData.set(element, {
      clickCount: 0
    });
  }
  
  handleClick(element) {
    const data = privateData.get(element);
    data.clickCount++;
  }
}
```

### 10.4 Symbol

```javascript
// 🔥 Symbol（唯一标识符）
const sym1 = Symbol('description');
const sym2 = Symbol('description');

console.log(sym1 === sym2); // false（每个Symbol都是唯一的）

// 作为对象属性
const id = Symbol('id');
const user = {
  name: 'zhangsan',
  [id]: 123
};

console.log(user[id]); // 123
console.log(user.id);  // undefined

// Symbol属性不会被遍历
console.log(Object.keys(user)); // ['name']

// 获取Symbol属性
console.log(Object.getOwnPropertySymbols(user)); // [Symbol(id)]

// 全局Symbol
const globalSym1 = Symbol.for('app.id');
const globalSym2 = Symbol.for('app.id');

console.log(globalSym1 === globalSym2); // true

// 获取Symbol的key
console.log(Symbol.keyFor(globalSym1)); // 'app.id'
```

---

## 📝 学习检查清单

- [ ] 掌握let/const和块级作用域
- [ ] 理解箭头函数和this绑定
- [ ] 掌握模板字符串
- [ ] 掌握解构赋值
- [ ] 掌握扩展运算符
- [ ] 理解Promise和链式调用
- [ ] 掌握async/await
- [ ] 理解Class语法和继承
- [ ] 掌握模块化导入导出
- [ ] 了解Set、Map等新数据结构

---

## 🔗 相关资源

- [ES6入门教程](https://es6.ruanyifeng.com/)
- [MDN JavaScript文档](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript)
- [ECMAScript规范](https://tc39.es/ecma262/)
- [Can I Use](https://caniuse.com/)

---

@author erik.zhou
