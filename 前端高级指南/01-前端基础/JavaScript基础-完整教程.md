# JavaScript基础 - 完整教程

> @author erik.zhou

## 📚 技术概述

### 版本信息
- **JavaScript版本**：ECMAScript 2023 (ES14)
- **推荐学习版本**：ES6+ (ES2015及以上)
- **浏览器支持**：现代浏览器全面支持

### 学习难度
- **难度等级**：⭐⭐⭐ (中等)
- **预计学习时间**：40-50小时
- **重要程度**：⭐⭐⭐⭐⭐ (必学)

### 前置知识
- HTML基础
- CSS基础
- 基本的编程概念

## 🎯 学习目标

完成本教程后，你将能够：

- [ ] 掌握JavaScript基础语法
- [ ] 理解数据类型和类型转换
- [ ] 掌握函数和作用域
- [ ] 理解对象和原型
- [ ] 掌握数组操作方法
- [ ] 理解异步编程基础
- [ ] 能够处理错误和调试

## 📖 目录

1. [JavaScript简介](#1-javascript简介)
2. [变量和数据类型](#2-变量和数据类型)
3. [运算符](#3-运算符)
4. [控制流程](#4-控制流程)
5. [函数](#5-函数)
6. [对象](#6-对象)
7. [数组](#7-数组)
8. [字符串](#8-字符串)
9. [错误处理](#9-错误处理)
10. [异步编程基础](#10-异步编程基础)

---

## 1. JavaScript简介

### 1.1 什么是JavaScript

JavaScript是一种高级的、解释型的编程语言，是Web开发的核心技术之一。

**核心特点**：
- 🔥 **动态类型**：变量类型在运行时确定
- 🔥 **单线程**：一次只执行一个任务
- 🔥 **事件驱动**：响应用户交互
- 🔥 **原型继承**：基于原型的面向对象

### 1.2 在HTML中使用JavaScript

```html
<!-- 内联JavaScript -->
<button onclick="alert('Hello!')">点击我</button>

<!-- 内部JavaScript -->
<script>
  console.log('Hello, World!');
</script>

<!-- 外部JavaScript -->
<script src="script.js"></script>

<!-- 🔥 推荐：在body底部引入 -->
<body>
  <!-- 页面内容 -->
  <script src="script.js"></script>
</body>
```


## 2. 变量和数据类型

### 2.1 变量声明

```javascript
// 🔥 var（不推荐，函数作用域）
var name = 'zhangsan';

// 🔥 let（推荐，块级作用域）
let age = 25;
age = 26; // 可以重新赋值

// 🔥 const（推荐，块级作用域，不可重新赋值）
const PI = 3.14159;
// PI = 3.14; // 错误！不能重新赋值

// const对象的属性可以修改
const user = { name: 'zhangsan' };
user.name = 'lisi'; // ✅ 可以
user.age = 25;      // ✅ 可以
// user = {};       // ❌ 不可以
```

### 2.2 数据类型

```javascript
// 🔥 基本数据类型（Primitive Types）

// 1. Number（数字）
let integer = 42;
let float = 3.14;
let negative = -10;
let infinity = Infinity;
let notANumber = NaN;

// 2. String（字符串）
let str1 = 'Hello';
let str2 = "World";
let str3 = `Hello ${str2}`; // 模板字符串

// 3. Boolean（布尔值）
let isTrue = true;
let isFalse = false;

// 4. Undefined（未定义）
let undefinedVar;
console.log(undefinedVar); // undefined

// 5. Null（空值）
let nullVar = null;

// 6. Symbol（符号，ES6）
let sym = Symbol('description');

// 7. BigInt（大整数，ES2020）
let bigInt = 9007199254740991n;

// 🔥 引用数据类型（Reference Types）

// 1. Object（对象）
let obj = { name: 'zhangsan', age: 25 };

// 2. Array（数组）
let arr = [1, 2, 3, 4, 5];

// 3. Function（函数）
let func = function() { return 'Hello'; };
```

### 2.3 类型检测

```javascript
// typeof运算符
console.log(typeof 42);           // "number"
console.log(typeof 'hello');      // "string"
console.log(typeof true);         // "boolean"
console.log(typeof undefined);    // "undefined"
console.log(typeof null);         // "object" (历史遗留问题)
console.log(typeof {});           // "object"
console.log(typeof []);           // "object"
console.log(typeof function(){}); // "function"

// 🔥 更准确的类型检测
console.log(Array.isArray([]));              // true
console.log(Object.prototype.toString.call([])); // "[object Array]"
```

### 2.4 类型转换

```javascript
// 🔥 转换为字符串
String(123);        // "123"
(123).toString();   // "123"
123 + '';           // "123"

// 🔥 转换为数字
Number('123');      // 123
parseInt('123');    // 123
parseFloat('3.14'); // 3.14
+'123';             // 123

// 🔥 转换为布尔值
Boolean(1);         // true
Boolean(0);         // false
Boolean('');        // false
Boolean('hello');   // true
!!value;            // 转换为布尔值

// ⚠️ 隐式类型转换
console.log('5' + 3);    // "53" (字符串拼接)
console.log('5' - 3);    // 2 (数字运算)
console.log('5' * '2');  // 10 (数字运算)
console.log(true + 1);   // 2
console.log(false + 1);  // 1
```

---

## 3. 运算符

### 3.1 算术运算符

```javascript
let a = 10, b = 3;

console.log(a + b);  // 13 加法
console.log(a - b);  // 7  减法
console.log(a * b);  // 30 乘法
console.log(a / b);  // 3.333... 除法
console.log(a % b);  // 1  取余
console.log(a ** b); // 1000 幂运算 (ES2016)

// 自增自减
let x = 5;
console.log(x++);    // 5 (先使用后加)
console.log(x);      // 6
console.log(++x);    // 7 (先加后使用)
console.log(x--);    // 7 (先使用后减)
console.log(--x);    // 5 (先减后使用)
```

### 3.2 比较运算符

```javascript
// 🔥 相等性比较
console.log(5 == '5');   // true  (值相等，类型转换)
console.log(5 === '5');  // false (值和类型都要相等)
console.log(5 != '5');   // false
console.log(5 !== '5');  // true

// ✅ 推荐使用 === 和 !==

// 大小比较
console.log(5 > 3);      // true
console.log(5 < 3);      // false
console.log(5 >= 5);     // true
console.log(5 <= 3);     // false
```

### 3.3 逻辑运算符

```javascript
// 🔥 逻辑与 (&&)
console.log(true && true);   // true
console.log(true && false);  // false

// 🔥 逻辑或 (||)
console.log(true || false);  // true
console.log(false || false); // false

// 🔥 逻辑非 (!)
console.log(!true);          // false
console.log(!false);         // true

// 短路求值
let user = null;
let name = user && user.name; // undefined (user为null，不继续执行)

let defaultName = user || 'Guest'; // 'Guest'

// 🔥 空值合并运算符 (??) ES2020
let value = null;
console.log(value ?? 'default'); // 'default'
console.log(0 ?? 'default');     // 0 (0不是null/undefined)
```

### 3.4 赋值运算符

```javascript
let x = 10;

x += 5;  // x = x + 5  (15)
x -= 3;  // x = x - 3  (12)
x *= 2;  // x = x * 2  (24)
x /= 4;  // x = x / 4  (6)
x %= 4;  // x = x % 4  (2)
x **= 3; // x = x ** 3 (8)
```

### 3.5 三元运算符

```javascript
// 🔥 条件 ? 值1 : 值2
let age = 18;
let status = age >= 18 ? '成年' : '未成年';
console.log(status); // '成年'

// 嵌套三元运算符（不推荐，可读性差）
let score = 85;
let grade = score >= 90 ? 'A' : 
            score >= 80 ? 'B' : 
            score >= 70 ? 'C' : 'D';
```

---

## 4. 控制流程

### 4.1 条件语句

```javascript
// 🔥 if语句
let age = 18;

if (age >= 18) {
  console.log('成年人');
}

// if...else
if (age >= 18) {
  console.log('成年人');
} else {
  console.log('未成年人');
}

// if...else if...else
let score = 85;

if (score >= 90) {
  console.log('优秀');
} else if (score >= 80) {
  console.log('良好');
} else if (score >= 60) {
  console.log('及格');
} else {
  console.log('不及格');
}
```

### 4.2 switch语句

```javascript
// 🔥 switch语句
let day = 3;
let dayName;

switch (day) {
  case 1:
    dayName = '星期一';
    break;
  case 2:
    dayName = '星期二';
    break;
  case 3:
    dayName = '星期三';
    break;
  case 4:
    dayName = '星期四';
    break;
  case 5:
    dayName = '星期五';
    break;
  case 6:
  case 7:
    dayName = '周末';
    break;
  default:
    dayName = '无效的日期';
}

console.log(dayName); // '星期三'
```

### 4.3 循环语句

```javascript
// 🔥 for循环
for (let i = 0; i < 5; i++) {
  console.log(i); // 0, 1, 2, 3, 4
}

// while循环
let i = 0;
while (i < 5) {
  console.log(i);
  i++;
}

// do...while循环
let j = 0;
do {
  console.log(j);
  j++;
} while (j < 5);

// 🔥 for...of循环（遍历数组）
let arr = [1, 2, 3, 4, 5];
for (let item of arr) {
  console.log(item);
}

// 🔥 for...in循环（遍历对象属性）
let obj = { name: 'zhangsan', age: 25 };
for (let key in obj) {
  console.log(key, obj[key]);
}

// break和continue
for (let i = 0; i < 10; i++) {
  if (i === 3) continue; // 跳过3
  if (i === 7) break;    // 在7处停止
  console.log(i); // 0, 1, 2, 4, 5, 6
}
```

---

## 5. 函数

### 5.1 函数声明

```javascript
// 🔥 函数声明
function greet(name) {
  return `Hello, ${name}!`;
}

console.log(greet('zhangsan')); // "Hello, zhangsan!"

// 🔥 函数表达式
const greet2 = function(name) {
  return `Hello, ${name}!`;
};

// 🔥 箭头函数 (ES6)
const greet3 = (name) => {
  return `Hello, ${name}!`;
};

// 简写形式
const greet4 = name => `Hello, ${name}!`;

// 多个参数
const add = (a, b) => a + b;
```

### 5.2 函数参数

```javascript
// 默认参数
function greet(name = 'Guest') {
  return `Hello, ${name}!`;
}

console.log(greet());          // "Hello, Guest!"
console.log(greet('zhangsan')); // "Hello, zhangsan!"

// 剩余参数
function sum(...numbers) {
  return numbers.reduce((total, num) => total + num, 0);
}

console.log(sum(1, 2, 3, 4, 5)); // 15

// 解构参数
function printUser({ name, age }) {
  console.log(`${name} is ${age} years old`);
}

printUser({ name: 'zhangsan', age: 25 });
```

### 5.3 作用域和闭包

```javascript
// 🔥 全局作用域
let globalVar = 'global';

function test() {
  // 函数作用域
  let localVar = 'local';
  console.log(globalVar); // 可以访问
  console.log(localVar);  // 可以访问
}

// console.log(localVar); // 错误！无法访问

// 🔥 块级作用域
if (true) {
  let blockVar = 'block';
  const blockConst = 'const';
  var functionVar = 'function'; // var没有块级作用域
}

// console.log(blockVar);    // 错误！
// console.log(blockConst);  // 错误！
console.log(functionVar);    // 可以访问

// ⚠️ 闭包
function createCounter() {
  let count = 0;
  
  return {
    increment() {
      count++;
      return count;
    },
    decrement() {
      count--;
      return count;
    },
    getCount() {
      return count;
    }
  };
}

const counter = createCounter();
console.log(counter.increment()); // 1
console.log(counter.increment()); // 2
console.log(counter.getCount());  // 2
console.log(counter.decrement()); // 1
```

### 5.4 高阶函数

```javascript
// 🔥 函数作为参数
function operate(a, b, operation) {
  return operation(a, b);
}

const add = (x, y) => x + y;
const multiply = (x, y) => x * y;

console.log(operate(5, 3, add));      // 8
console.log(operate(5, 3, multiply)); // 15

// 🔥 函数作为返回值
function createMultiplier(multiplier) {
  return function(number) {
    return number * multiplier;
  };
}

const double = createMultiplier(2);
const triple = createMultiplier(3);

console.log(double(5)); // 10
console.log(triple(5)); // 15
```


## 6. 对象

### 6.1 创建对象

```javascript
// 🔥 对象字面量
const person = {
  name: 'zhangsan',
  age: 25,
  city: 'Beijing'
};

// 构造函数
function Person(name, age) {
  this.name = name;
  this.age = age;
}

const person2 = new Person('lisi', 30);

// Object.create()
const personProto = {
  greet() {
    console.log(`Hello, I'm ${this.name}`);
  }
};

const person3 = Object.create(personProto);
person3.name = 'wangwu';
```

### 6.2 访问和修改属性

```javascript
const user = {
  name: 'zhangsan',
  age: 25,
  'favorite color': 'blue'
};

// 🔥 点表示法
console.log(user.name);  // 'zhangsan'
user.age = 26;

// 🔥 方括号表示法
console.log(user['name']); // 'zhangsan'
console.log(user['favorite color']); // 'blue'

// 动态属性名
const prop = 'age';
console.log(user[prop]); // 26

// 添加新属性
user.email = 'zhangsan@example.com';

// 删除属性
delete user.age;
```

### 6.3 对象方法

```javascript
const calculator = {
  value: 0,
  
  add(num) {
    this.value += num;
    return this;
  },
  
  subtract(num) {
    this.value -= num;
    return this;
  },
  
  multiply(num) {
    this.value *= num;
    return this;
  },
  
  getValue() {
    return this.value;
  }
};

// 🔥 方法链式调用
const result = calculator
  .add(10)
  .multiply(2)
  .subtract(5)
  .getValue();

console.log(result); // 15
```

### 6.4 对象遍历

```javascript
const user = {
  name: 'zhangsan',
  age: 25,
  city: 'Beijing'
};

// 🔥 for...in循环
for (let key in user) {
  console.log(key, user[key]);
}

// Object.keys()
const keys = Object.keys(user);
console.log(keys); // ['name', 'age', 'city']

// Object.values()
const values = Object.values(user);
console.log(values); // ['zhangsan', 25, 'Beijing']

// Object.entries()
const entries = Object.entries(user);
console.log(entries); 
// [['name', 'zhangsan'], ['age', 25], ['city', 'Beijing']]

// 遍历entries
for (let [key, value] of Object.entries(user)) {
  console.log(`${key}: ${value}`);
}
```

### 6.5 对象解构

```javascript
const user = {
  name: 'zhangsan',
  age: 25,
  city: 'Beijing'
};

// 🔥 基本解构
const { name, age } = user;
console.log(name); // 'zhangsan'
console.log(age);  // 25

// 重命名
const { name: userName, age: userAge } = user;
console.log(userName); // 'zhangsan'

// 默认值
const { name, country = 'China' } = user;
console.log(country); // 'China'

// 剩余属性
const { name: n, ...rest } = user;
console.log(rest); // { age: 25, city: 'Beijing' }
```

### 6.6 对象扩展运算符

```javascript
const user = {
  name: 'zhangsan',
  age: 25
};

// 🔥 复制对象
const userCopy = { ...user };

// 合并对象
const address = {
  city: 'Beijing',
  country: 'China'
};

const fullUser = { ...user, ...address };
console.log(fullUser);
// { name: 'zhangsan', age: 25, city: 'Beijing', country: 'China' }

// 覆盖属性
const updatedUser = { ...user, age: 26 };
console.log(updatedUser); // { name: 'zhangsan', age: 26 }
```

---

## 7. 数组

### 7.1 创建数组

```javascript
// 🔥 数组字面量
const arr1 = [1, 2, 3, 4, 5];

// Array构造函数
const arr2 = new Array(5); // 创建长度为5的空数组
const arr3 = new Array(1, 2, 3); // [1, 2, 3]

// Array.of()
const arr4 = Array.of(1, 2, 3); // [1, 2, 3]

// Array.from()
const arr5 = Array.from('hello'); // ['h', 'e', 'l', 'l', 'o']
const arr6 = Array.from({ length: 5 }, (_, i) => i); // [0, 1, 2, 3, 4]
```

### 7.2 数组基本操作

```javascript
const fruits = ['apple', 'banana', 'orange'];

// 访问元素
console.log(fruits[0]); // 'apple'
console.log(fruits[fruits.length - 1]); // 'orange'

// 修改元素
fruits[1] = 'grape';

// 🔥 添加元素
fruits.push('mango');        // 末尾添加
fruits.unshift('strawberry'); // 开头添加

// 🔥 删除元素
fruits.pop();    // 删除末尾元素
fruits.shift();  // 删除开头元素

// 🔥 splice（删除、插入、替换）
const numbers = [1, 2, 3, 4, 5];
numbers.splice(2, 1);        // 从索引2删除1个元素: [1, 2, 4, 5]
numbers.splice(2, 0, 3);     // 在索引2插入3: [1, 2, 3, 4, 5]
numbers.splice(2, 1, 10);    // 替换索引2的元素: [1, 2, 10, 4, 5]
```

### 7.3 数组遍历

```javascript
const numbers = [1, 2, 3, 4, 5];

// 🔥 for循环
for (let i = 0; i < numbers.length; i++) {
  console.log(numbers[i]);
}

// 🔥 for...of循环
for (let num of numbers) {
  console.log(num);
}

// 🔥 forEach方法
numbers.forEach((num, index) => {
  console.log(`Index ${index}: ${num}`);
});

// forEach不能break或return
```

### 7.4 数组方法

```javascript
const numbers = [1, 2, 3, 4, 5];

// 🔥 map（映射）
const doubled = numbers.map(num => num * 2);
console.log(doubled); // [2, 4, 6, 8, 10]

// 🔥 filter（过滤）
const evens = numbers.filter(num => num % 2 === 0);
console.log(evens); // [2, 4]

// 🔥 reduce（归约）
const sum = numbers.reduce((total, num) => total + num, 0);
console.log(sum); // 15

// 🔥 find（查找第一个）
const found = numbers.find(num => num > 3);
console.log(found); // 4

// 🔥 findIndex（查找索引）
const index = numbers.findIndex(num => num > 3);
console.log(index); // 3

// 🔥 some（是否有满足条件的）
const hasEven = numbers.some(num => num % 2 === 0);
console.log(hasEven); // true

// 🔥 every（是否全部满足条件）
const allPositive = numbers.every(num => num > 0);
console.log(allPositive); // true

// 🔥 includes（是否包含）
console.log(numbers.includes(3)); // true

// 🔥 indexOf（查找索引）
console.log(numbers.indexOf(3)); // 2

// 🔥 slice（切片，不改变原数组）
const sliced = numbers.slice(1, 4);
console.log(sliced); // [2, 3, 4]

// 🔥 concat（连接数组）
const arr1 = [1, 2];
const arr2 = [3, 4];
const combined = arr1.concat(arr2);
console.log(combined); // [1, 2, 3, 4]

// 🔥 join（转换为字符串）
console.log(numbers.join('-')); // "1-2-3-4-5"

// 🔥 reverse（反转）
const reversed = [...numbers].reverse();
console.log(reversed); // [5, 4, 3, 2, 1]

// 🔥 sort（排序）
const unsorted = [3, 1, 4, 1, 5, 9, 2, 6];
const sorted = [...unsorted].sort((a, b) => a - b);
console.log(sorted); // [1, 1, 2, 3, 4, 5, 6, 9]
```

### 7.5 数组解构

```javascript
const numbers = [1, 2, 3, 4, 5];

// 🔥 基本解构
const [first, second] = numbers;
console.log(first);  // 1
console.log(second); // 2

// 跳过元素
const [a, , c] = numbers;
console.log(a); // 1
console.log(c); // 3

// 剩余元素
const [head, ...tail] = numbers;
console.log(head); // 1
console.log(tail); // [2, 3, 4, 5]

// 默认值
const [x, y, z = 0] = [1, 2];
console.log(z); // 0
```

### 7.6 数组扩展运算符

```javascript
const arr1 = [1, 2, 3];
const arr2 = [4, 5, 6];

// 🔥 复制数组
const copy = [...arr1];

// 🔥 合并数组
const combined = [...arr1, ...arr2];
console.log(combined); // [1, 2, 3, 4, 5, 6]

// 🔥 在数组中插入元素
const inserted = [...arr1, 10, ...arr2];
console.log(inserted); // [1, 2, 3, 10, 4, 5, 6]

// 🔥 数组转参数
const numbers = [1, 2, 3];
console.log(Math.max(...numbers)); // 3
```

---

## 8. 字符串

### 8.1 字符串基础

```javascript
// 创建字符串
const str1 = 'Hello';
const str2 = "World";
const str3 = `Hello ${str2}`; // 模板字符串

// 字符串长度
console.log(str1.length); // 5

// 访问字符
console.log(str1[0]);        // 'H'
console.log(str1.charAt(0)); // 'H'
```

### 8.2 字符串方法

```javascript
const str = 'Hello, World!';

// 🔥 查找
console.log(str.indexOf('World'));     // 7
console.log(str.lastIndexOf('o'));     // 8
console.log(str.includes('World'));    // true
console.log(str.startsWith('Hello'));  // true
console.log(str.endsWith('!'));        // true

// 🔥 提取
console.log(str.slice(0, 5));          // 'Hello'
console.log(str.substring(0, 5));      // 'Hello'
console.log(str.substr(0, 5));         // 'Hello' (已废弃)

// 🔥 替换
console.log(str.replace('World', 'JavaScript')); // 'Hello, JavaScript!'
console.log(str.replaceAll('o', '0'));           // 'Hell0, W0rld!'

// 🔥 大小写转换
console.log(str.toUpperCase()); // 'HELLO, WORLD!'
console.log(str.toLowerCase()); // 'hello, world!'

// 🔥 去除空格
const padded = '  hello  ';
console.log(padded.trim());      // 'hello'
console.log(padded.trimStart()); // 'hello  '
console.log(padded.trimEnd());   // '  hello'

// 🔥 分割
const csv = 'apple,banana,orange';
console.log(csv.split(','));     // ['apple', 'banana', 'orange']

// 🔥 重复
console.log('ha'.repeat(3));     // 'hahaha'

// 🔥 填充
console.log('5'.padStart(3, '0')); // '005'
console.log('5'.padEnd(3, '0'));   // '500'
```

### 8.3 模板字符串

```javascript
const name = 'zhangsan';
const age = 25;

// 🔥 变量插值
const message = `My name is ${name} and I'm ${age} years old.`;

// 🔥 表达式
const price = 100;
const tax = 0.1;
console.log(`Total: ${price * (1 + tax)}`); // 'Total: 110'

// 🔥 多行字符串
const html = `
  <div>
    <h1>${name}</h1>
    <p>Age: ${age}</p>
  </div>
`;

// 🔥 标签模板
function highlight(strings, ...values) {
  return strings.reduce((result, str, i) => {
    return result + str + (values[i] ? `<mark>${values[i]}</mark>` : '');
  }, '');
}

const highlighted = highlight`Name: ${name}, Age: ${age}`;
console.log(highlighted);
// "Name: <mark>zhangsan</mark>, Age: <mark>25</mark>"
```

---

## 9. 错误处理

### 9.1 try...catch

```javascript
// 🔥 基本用法
try {
  // 可能出错的代码
  const result = riskyOperation();
  console.log(result);
} catch (error) {
  // 处理错误
  console.error('Error:', error.message);
} finally {
  // 无论是否出错都会执行
  console.log('Cleanup');
}
```

### 9.2 抛出错误

```javascript
// 🔥 throw语句
function divide(a, b) {
  if (b === 0) {
    throw new Error('Division by zero');
  }
  return a / b;
}

try {
  const result = divide(10, 0);
} catch (error) {
  console.error(error.message); // 'Division by zero'
}

// 自定义错误类
class ValidationError extends Error {
  constructor(message) {
    super(message);
    this.name = 'ValidationError';
  }
}

function validateAge(age) {
  if (age < 0) {
    throw new ValidationError('Age cannot be negative');
  }
  if (age > 150) {
    throw new ValidationError('Age is too high');
  }
  return true;
}
```

---

## 10. 异步编程基础

### 10.1 setTimeout和setInterval

```javascript
// 🔥 setTimeout（延迟执行）
setTimeout(() => {
  console.log('Executed after 1 second');
}, 1000);

// 取消定时器
const timeoutId = setTimeout(() => {
  console.log('This will not execute');
}, 1000);
clearTimeout(timeoutId);

// 🔥 setInterval（重复执行）
let count = 0;
const intervalId = setInterval(() => {
  count++;
  console.log(`Count: ${count}`);
  
  if (count === 5) {
    clearInterval(intervalId);
  }
}, 1000);
```

### 10.2 回调函数

```javascript
// 🔥 回调函数模式
function fetchData(callback) {
  setTimeout(() => {
    const data = { id: 1, name: 'zhangsan' };
    callback(data);
  }, 1000);
}

fetchData((data) => {
  console.log('Data received:', data);
});

// ⚠️ 回调地狱
fetchUser(userId, (user) => {
  fetchPosts(user.id, (posts) => {
    fetchComments(posts[0].id, (comments) => {
      console.log(comments);
    });
  });
});
```

### 10.3 Promise基础

```javascript
// 🔥 创建Promise
const promise = new Promise((resolve, reject) => {
  setTimeout(() => {
    const success = true;
    
    if (success) {
      resolve('Success!');
    } else {
      reject('Error!');
    }
  }, 1000);
});

// 使用Promise
promise
  .then(result => {
    console.log(result); // 'Success!'
  })
  .catch(error => {
    console.error(error);
  })
  .finally(() => {
    console.log('Done');
  });

// Promise链
fetch('https://api.example.com/data')
  .then(response => response.json())
  .then(data => {
    console.log(data);
    return data.id;
  })
  .then(id => {
    return fetch(`https://api.example.com/details/${id}`);
  })
  .then(response => response.json())
  .then(details => {
    console.log(details);
  })
  .catch(error => {
    console.error('Error:', error);
  });
```

---

## 📝 学习检查清单

- [ ] 掌握JavaScript基础语法
- [ ] 理解数据类型和类型转换
- [ ] 掌握运算符和控制流程
- [ ] 理解函数和作用域
- [ ] 掌握对象和数组操作
- [ ] 理解闭包概念
- [ ] 掌握字符串处理方法
- [ ] 能够处理错误
- [ ] 理解异步编程基础

---

## 🔗 相关资源

- [MDN JavaScript文档](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript)
- [JavaScript.info](https://javascript.info/)
- [You Don't Know JS](https://github.com/getify/You-Dont-Know-JS)
- [Eloquent JavaScript](https://eloquentjavascript.net/)

---

@author erik.zhou
