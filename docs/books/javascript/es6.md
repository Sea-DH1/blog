---
sidebar: auto
---

# 深入理解ES6
在`ECMAScript6`标准定稿之前，已经开始出现了一些实验性的`转译器(Transpiler)`，例如谷歌的`Traceur`，可以将代码从`ECMAScript6`转换成`ECMAScript5`。但它们大多功能非常有限，或难以插入现有的`JavaScript`构建管道。<br/>
但是，随后出现了的新型转译器`6to5`改变了这一切。它易于安装，可以很好的集成现有的工具中，生成的代码可读，于是就像野火一样逐步蔓延开来，`6to5`也就是现在鼎鼎大名的`Babel`。

## 前言

### ECMAScript6的演变之路
::: tip
JavaScript核心的语言特性是在标准`ECMA-262`中被定义，该标准中定义的语言被称作`ECMAScript`，它是`JavaScript`的子集。
:::

* `停滞不前`：逐渐兴起的`Ajax`开创了动态`Web`引用的新时代，而自1999年第三版`ECMA-262`发布以来，`JavaScript`却没有丝毫的改变。
* `转折点`：2007年，`TC-39`委员会将大量规范草案整合在了`ECMAScript4`中，其中新增的语言特性涉足甚广，包括：模块、类、类继承、私有对象成员等众多其它的特性。
* `分歧`：然而`TC-39`组织内部对`ECMAScript4`草案产生了巨大的分歧，部分成员认为不应该一次性在第四版标准中加入过多的新功能，而来自雅虎、谷歌和微软的技术负责人则共同提交了一份`ECMAScript3.1`草案作为下一代`ECMAScript`的可选方案，其中此方案只是对现有标准进行小幅度的增量修改。行为更专注于优化属性特性、支持原生`JSON`以及为已有对象增加新的方法。
* `从未面世的ECMAScript4`：2008年，`JavaScript`创始人`Brendan Eich`宣布`TC-39`委员一方面会将合理推进`ECMAScript3.1`的标准化工作，另一方面会暂时将`ECMAScript4`标准中提出的大部分针对语法及特性的改动搁置。
* `ECMAScript5`：经过标准化的`ECMAScript3.1`最终作为`ECMA-262`第五版与2009年正式发布，同时被命名为`ECMAScript5`。
* `ECMAScript6`：在`ECMAScript5`发布后，`TC-39`委员会于2013年冻结了`ECMAScript6`的草案，不再添加新的功能。2013年`ECMAScript6`草案发布，在进过12个月各方讨论和反馈后。2015年`ECMAScript6`正式发布，并命名为`ECMAScript 2015`。

## 块级作用域绑定
::: tip 引入的目的
一直以来`JavaScript`中的变量生声明机制一直令我们感到困惑，大多数类C语言在声明变量的同时也会创建变量，而在以前的`JavaScript`中，何时创建变量要看如何声明的变量，`ES6`引入块级作用域可以让我们更好的控制作用域。
:::

### var声明和变量提升机制

问：提升机制是什么？<br/>
答：在函数作用域或全局作用域中通过关键字`var`声明的变量，无论实际上是在哪里声明的，都会被当成在当前作用域顶部声明的变量，这就是我们常说的提升机制。<br/>
以下实例代码说明了这种提升机制：
```js
function getValue (condition) {
  if (condition) {
    var value = 'value'
    return value
  } else {
    // 这里可以访问到value，只不过值为undefined
    console.log(value)
    return null
  }
}
getValue(false) // 输出undefined
```
你可以在以上代码中看到，当我们传递了`false`的值，但依然可以访问到`value`这个变量，这是因为在预编译阶段，`JavaScript`引擎会将上面函数的代码修改成如下形式：
```js
function getValue (condition) {
  var value
  if (condition) {
    value = 'value'
    return value
  } else {
    console.log(value)
    return null
  }
}
```

经过以上示例，我们可以发现：变量`value`的声明被提升至函数作用域的顶部，而初始化操作依旧留在原处执行，正因为`value`变量只是声明了而没有赋值，因此以上代码才会打印出`undefined`。


### 块级声明
::: tip
块级声明用于声明在指定的作用于之外无妨访问的变量，块级作用域存在于：函数内部和块中。
:::

`let`声明：
* `let`声明和`var`声明的用法基本相同。
* `let`声明的变量不会被提升。
* `let`不能在同一个作用域中重复声明已经存在的变量，会报错。
* `let`声明的变量作用域范围仅存在于当前的块中，程序进入快开始时被创建，程序退出块时被销毁。

根据`let`声明的规则，改动以上代码后像下面这样：
```js
function getValue (condition) {
  if (condition) {
    // 变量value只存在于这个块中。
    let value = 'value'
    return value
  } else {
    // 访问不到value变量
    console.log(value)
    return null
  }
}
```

`const`声明：`const`声明和`let`声明大多数情况是相同的，唯一的本质区别在于，`const`是用来声明常量的，其声明后的变量值不能再被修改，即意味着：`const`声明必须进行初始化。
```js
const MAX_ITEMS = 30
// 报错
MAX_ITEMS = 50
```

::: warning 注意
我们说的`const`变量值不可变，需要分两种类型来说：
* 值类型：变量的值不能改变。
* 引用类型：变量的地址不能改变，值可以改变。
:::
```js
const num = 23
const arr = [1, 2, 3, 4]
const obj = {
  name: 'why',
  age: 23
}

// 报错
num = 25

// 不报错
arr[0] = 11
obj.age = 32
console.log(arr) // [11, 2, 3, 4]
console.log(obj) // { name: 'why', age: 32 }

// 报错
arr = [4, 3, 2, 1]
```

### 暂时性死区
因为`let`和`const`声明的变量不会进行声明提升，所以在`let`和`const`变量声明之前任何访问(即使是`typeof`也不行)此变量的操作都会引发错误：
```js
if (condition) {
  // 报错
  console.log(typeof value)
  let value = 'value'
}
```
问：为什么会报错？<br/>
答：`JavaScript`引擎在扫描代码发现变量声明时，要么将它们提升至作用域的顶部(`var`声明)，要么将声明放在`TDZ`(暂时性死区)中(`let`和`const`)。访问`TDZ`中的变量会触发错误，只有执行变量声明语句之后，变量才会从`TDZ`中移出，随后才能正常访问。

### 全局块作用域绑定
我们都知道：如果我们在全局作用域下通过`var`声明一个变量，那么这个变量会挂载到全局对象`window`上：
```js
var name = 'why'
console.log(window.name) // why
```
但如果我们使用`let`或者`const`在全局作用域下创建一个新的变量，这个变量不会添加到`window`上。
```js
const name = 'why'
console.log('name' in window) // false
```

### 块级绑定最佳实践的进化
在`ES6`早期，人们普遍认为应该默认使用`let`来代替`var`，这是因为对于开发者而言，`let`实际上与他们想要的`var`一样，直接替换符合逻辑。但随着时代的发展，另一种做法也越来越普及：默认使用`const`，只有确定变量的值会在后续需要修改时才会使用`let`声明，因为大部分变量在初始化后不应再改变，而预料以外的变量值改变是很多`bug`的源头。

## 字符串

### 模块字面量
::: tip
模板字面量是扩展`ECMAScript`基础语法的语法糖，其提供了一套生成、查询并操作来自其他语言里内容的`DSL`，且可以免受`XSS`注入攻击和`SQL`注入等等。
:::

在`ES6`之前，`JavaScript`一直以来缺少许多特性：
* **多行字符串**：一个正式的多行字符串的概念。
* **基本的字符串格式化**：将变量的值嵌入字符串的能力。
* **HTML转义**：向`HTML`插入经过安全转换后的字符串的能力。

而在`ECMAScript 6`中，通过模板字面量的方式多以上问题进行了填补，一个最简单的模板字面量的用法如下：
```js
const message = `hello,world!`
console.log(message) // hello,world!
console.log(typeof message) // string
```
一个需要注意的地方就是，如果我们需要在字符串中使用反撇号，需要使用`\`来进行转义，如下：
```js
const message = `\`hello\`,world!`
console.log(message) // `hello`,world!
```

#### 多行字符串
自`JavaScript`诞生起，开发者们就一直在尝试和创建多行字符串，以下是`ES6`之前的方法：
::: tip
在字符串的新行最前方加上`\`可以承接上一行代码，可以利用这个小`bug`来创建多行字符串。
:::
```js
const message = 'hello\
,world!'
console.log(message) // hello,world
```

在`ES6`之后，我们可以使用模板字面量，在里面直接还行就可以创建多行字符串，如下：
::: tip
在模板字面量中，即反撇号中所有空白字符都属于字符串的一部分。
:::
```js
const message = `hello
,world!`
console.log(message) // hello
                     // ,world!
```

#### 字符串占位符
::: tip
模板字面量于普通字符串最大的区别是模板字符串中的占位符功能，其中占位符中的内容，可以是任意合法的`JavaScript`表达式，例如：变量，运算式，函数调用，设置是另外一个模板字面量。
:::
```js
const age = 23
const name = 'why'
const message = `Hello ${name}, you are ${age} years old!`
console.log(message) // Hello why, you are 23 years old!
```
模板字面量嵌套：
```js
const name = 'why'
const message = `Hello, ${`my name is ${name}`}.`
console.log(message) // Hello, my name is why.
```

#### 标签模板
::: tip
标签指的是在模板字面量第一个反撇号前方的标注的字符串，每一个模板标签都可以执行模板字面量上的转换并返回最终的字符串值。
:::
```js
// tag就是`Hello world!`模板字面量的标签模板
const message = tag`Hello world!`
```
标签可以是一个函数，标签函数通常使用不定参数特性来定义占位符，从而简化数据处理的过程，就像下面这样：
```js
function tag(literals, ...substitutions) {
  // 返回一个字符串
}
const name = 'why'
const age = 23
const message = tag`${name} is ${age} years old!`
```
其中`literals`是一个数组，它包含：
* 第一个占位符前的空白字符串：""。
* 第一个、第二个占位符之间的字符串：" is "。
* 第二个占位符后的字符串：" years old!"

`substitutions`也是一个数组：
* 数组第一项为：`name`的值，即：`why`。
* 数组第二项为：`age`的值，即：`23`。

通过以上规律我们可以发现：
* `literals[0]`始终代表字符串的开头。
* `literals`总比`substitutions`多一个。

我们可以通过以上这种模式，将`literals`和`substitutions`这两个数组交织在一起重新组成一个字符串，来模拟模板字面量的默认行为，像下面这样：
```js
function tag(literals, ...substitutions) {
  let result = ''
  for (let i = 0; i< substitutions.length; i++) {
    result += literals[i]
    result += substitutions[i]
  }

  // 合并最后一个
  result += literals[literals.length - 1]
  return result
}
const name = 'why'
const age = 23
const message = tag`${name} is ${age} years old!`
console.log(message) // why is 23 years old!
```

#### 原生字符串信息
::: tip
通过模板标签可以访问到字符串转义被转换成等价字符串前的原生字符串。
:::
```js
const message1 = `Hello\nworld`
const message2 = String.raw`Hello\nworld`
console.log(message1) // Hello
                      // world
console.log(message2) // Hello\nworld
```

## 函数

### 形参默认值

### 无命名参数

### 增强的Function构造函数

### 展开运算符

### 函数name属性

### 函数的多种用途

### 块级函数

### 箭头函数

### 尾调用优化

## 对象的扩展

## 解构

## Symbol及其Symbol属性

## Set和Map集合

## 迭代器(Iterator)和生成器(Generator)

## JavaScript中的类

## 数据的改进

## Promise和异步编程

## 代理(Proxy)和反射(Reflect)API

## 用模块封装代码

