### JavaScript 基础

#### JavaScript 使用

---

`<script>` 标签

- 在HTML中，JS代买必须位于这个标签之中

  - 注释：就是JS例子也许会使用 `type`属性：<script type="text/javascript">

  - type 属性不是必须的，JS是HTML的默认脚本语言

---

JavaScript 函数和事件

<head> 或 <body> 中的JavaScript

- 能在HTML文档中任意位置防止任意数量的脚本
- **提示：**把脚本置于 `<body>` 元素的底部，可改善显示速度，因为脚本编译会拖慢显示。

---

外部脚本

- 脚本文件可放置在外部

- 使用脚本标签引入
  <script src = "myScript.js"></script>

  - JavaScript文件的扩展名是 .js
  - 使用 src 属性引入

外部引用

- 可通过完整的URL或相对于当前网页的路径引用外部脚本

- ```html
  <script src="https://www.w3school.com.cn/js/myScript1.js"></script>
  ```



#### JavaScript 输出

JavaScript 不提供任何内建形式的打印或显示函数。显示方案

- 使用 window.alert() 写入警告框
- 使用 document.write() 写入HTML输出
  - 在HTML文档完全加载后使用将 删除所有已有的HTML
  - 此方法一般仅用于测试
- 使用 innerHTML 写入HTML元素
- 使用 console.log() 写入浏览器控制台



#### JavaScript 语句

在HTML中，JavaScript语句是由 web浏览器 “执行”的“指令”

JavaScript 程序

- 计算机程序是由计算机“执行”的一系列“指令”。

- 在编程语言中，这些编程指令被称为语句。

- JavaScript 程序就是一系列的编程语句。

- 注释：在 HTML 中，JavaScript 程序由 web 浏览器执行。

---

分号 `;`

- 用来分隔 JS 语句

代码块 `{   }`

- 用花括号包裹代码块

---

JavaScript 关键词

| 关键词        | 描述                                              |
| :------------ | :------------------------------------------------ |
| break         | 终止 switch 或循环。                              |
| continue      | 跳出循环并在顶端开始。                            |
| debugger      | 停止执行 JavaScript，并调用调试函数（如果可用）。 |
| do ... while  | 执行语句块，并在条件为真时重复代码块。            |
| for           | 标记需被执行的语句块，只要条件为真。              |
| function      | 声明函数。                                        |
| if ... else   | 标记需被执行的语句块，根据某个条件。              |
| return        | 退出函数。                                        |
| switch        | 标记需被执行的语句块，根据不同的情况。            |
| try ... catch | 对语句块实现错误处理。                            |
| var           | 声明变量。                                        |



#### JavaScript 语法

---

JavaScript 值

- 定义了两种类型的值：混合值和变量值
  - 混合值被称为 字面量：literal
    - 数值、字符串等
  - 变量值被称为 变量
    - 使用 `var` 定义变量
    - 使用 `=` 为变量赋值
- Value = undefined
  - 被声明的变量，如果没有赋初始值，它的值将是 `undefined`



#### JavaScript Let

- ES 2015 中引入了两个重要的新关键词：
  - let
  - const
- 这两个关键字在 JavaScript 中提供了块作用域（*Block Scope*）变量（和常量）。
- 在 ES2015 之前，JavaScript 只有两种类型的作用域：*全局作用域*和*函数作用域*。
  - 在函数外用 var 声明的变量具有 全局作用域
  - 在函数内用 var 声明的变量具有 函数作用域

- 块作用域：let

  - JavaScript 块作用域

    - 通过 var 关键词声明的变量没有块作用域。

    - 在块 {} 内声明的变量可以从块之外进行访问。

  - 可以使用 `let` 关键词声明拥有块作用域的变量。

    - 在块 *{}* 内声明的变量无法从块外访问

HTML中的全局变量

- 使用 JavaScript 的情况下，全局作用域是 JavaScript 环境。

- 在 HTML 中，全局作用域是 window 对象。

- 通过 `var` 关键词定义的全局变量属于 window 对象

- 通过 `let` 关键词定义的全局变量不属于 window 对象

- ```js
  var carName = "porsche";
  // 此处的代码可使用 window.carName
  
  let carName = "porsche";
  // 此处的代码不可使用 window.carName
  ```

- 在相同的作用域，或在相同的块中，通过 `let` 重新声明一个 `var` 变量是不允许的

- 通过 `var` 声明的变量会*提升*到顶端。如果您不了解什么是提升（Hoisting），请学习我们的提升这一章。

  - 您可以在声明变量之前就使用它

- 通过 `let` 定义的变量不会被提升到顶端。

  - 在声明 `let` 变量之前就使用它会导致 ReferenceError。

  - 变量从块的开头一直处于“暂时死区”，直到声明为止



#### JavaScript Const

- Const：通过 const 定义的变量不能重新赋值
- JavaScript `const` 变量必须在声明时赋值

不是真正的常数

- 关键字 `const` 有一定的误导性

  - 它没有定义常量值。它定义了对值的常量引用
  - 不能更改常量原始值，但我们可以更改常量对象的属性

- 提升：Hoisting

  - 通过 `const` 定义的变量不会被提升到顶端。

  - `const` 变量不能在声明之前使用



#### JavaScript 运算符

JavaScript 比较运算符

| 运算符 | 描述       |
| :----- | :--------- |
| ==     | 等于       |
| ===    | 等值等型   |
| ?      | 三元运算符 |

JavaScript 类型运算符

| 运算符     | 描述                                |
| :--------- | :---------------------------------- |
| typeof     | 返回变量的类型。                    |
| instanceof | 返回 true，如果对象是对象类型的实例 |



#### JavaScript 数据类型

字符串值、数值、布尔值、数组、对象

- JavaScript 拥有动态类型

  - 数组 []

    - ```javascript
      var cars = ["Porsche", "Volvo", "BMW"];
      ```

  - 对象 {}

    - ```js
      var person = {firstName:"Bill", lastName:"Gates", age:62, eyeColor:"blue"};
      ```

- typeof 运算符
  - 返回变量或表达式的类型

- Undefined
  - 在 JavaScript 中，没有值的变量，其值是 `undefined`。typeof 也返回 `undefined`。

- null

  - 在 JavaScript 中，`null` 是 "nothing"。它被看做不存在的事物。

  - 不幸的是，在 JavaScript 中，`null` 的数据类型是对象。
  - 您可以把 `null` 在 JavaScript 中是对象理解为一个 bug。它本应是 `null`。

  - 您可以通过设置值为 `null` 清空对象

- Undefined 与 Null 的区别

​		`Undefined` 与 `null` 的值相等，但类型不相等：

```
typeof undefined              // undefined
typeof null                   // object
null === undefined            // false
null == undefined             // true
```



#### JavaScript 事件

HTML事件是发生在 HTML 元素上的“事情”

当在HTML页面中使用JavaScript时，JS能够应对这些事情

---

常见的HTML事件

| 事件        | 描述                         |
| :---------- | :--------------------------- |
| onchange    | HTML 元素已被改变            |
| onclick     | 用户点击了 HTML 元素         |
| onmouseover | 用户把鼠标移动到 HTML 元素上 |
| onmouseout  | 用户把鼠标移开 HTML 元素     |
| onkeydown   | 用户按下键盘按键             |
| onload      | 浏览器已经完成页面加载       |





#### JavaScript 字符串方法

---

字符串方法和属性

- 字符串长度 `str.length`

- 查找字符串中的字符串：

  - indexOf('str')
  - lastIndexOf()

- 检索字符串中的字符串

  - search()
  - 与 indexOf() 区别
    - search() 方法无法设置第二个开始位置参数
    - indexOf() 方法无法设置更强大的搜索之（正则表达式）

- 提取部分字符串：

  - slice(start, end)
  - substring(start, end)
  - substr(start, length)

- 替换字符串内容

  - replace()

- 转换为大写和小写

  - toUpperCase()
  - toLowerCase()

- concat() 方法

  - 连接两个或多个字符串

- 删除字符串两端空白

  - trim()

- 提取字符串

  - charAt()
  - charCodeAt()

- 属性访问：Property Access

- 把字符串转换为数组

  - 可以通过 split() 把字符串转换为数组

  - ```js
    var txt = "a,b,c,d,e";   // 字符串
    txt.split(",");          // 用逗号分隔
    txt.split(" ");          // 用空格分隔
    txt.split("|");          // 用竖线分隔
    ```



#### JavaScript 字符串模板

- 模板字面量使用反引号 \`\` 来定义字符串。作用：
  - 字符串内使用引号，不再需要转义
  - 多行字符串
  - 插值 ${...}
  - 变量替换
  - 表达式替换
  - HTML模板



#### JavaScript 数字

JavaScript 数值始终是 64 位 浮点数



NaN：非数值

- 属于JS保留词，指示某个数不是合法数
- `NaN` 是数，`typeof NaN` 返回 `number`



Infinity：是JS在计算数时超出最大可能范围时返回的值



#### JavaScript 数字方法

- toString() 方法

  - 以字符串返回数值

- toExponential() 方法

  - 返回字符串值，包含已被四舍五入并使用指数计数法的数字

- toFixed() 方法

  - 返回字符串，包含了指定 位数 小数的数字

- toPrecision() 方法

  - 返回字符串，包含指定长度的数字

- valueOf() 方法

  - 以数值返回数值

  - 在 JavaScript 中，数字可以是原始值（typeof = number）或对象（typeof = object）。

    在 JavaScript 内部使用 `valueOf()` 方法可将 Number 对象转换为原始值。

    没有理由在代码中使用它。

    所有 JavaScript 数据类型都有 `valueOf()` 和 `toString()` 方法。

- 把变量转换为数值：全局JS方法
  - Number()
  - parseInt()
  - parseFloat()