# 让你能看懂系列：JavaScript 类型转换

----------

关于 JavaScript 的类型转换，网上有很多资料，但大多都没有引入较新的标准，说得不怎么清晰明白，甚至还有一些错误夹杂其中。我把 ECMA 262 标准中相关的部分整理了一下，并重新组织语言，重新描述步骤，把还未正式列入标准的 `BigInt` 引入进来。


#### **目 录**

```
0. 引言
1. 关于类型
	1.1 类型列表
		1.1.1 Undefined
		1.1.2 Null
		1.1.3 Boolean
		1.1.4 String
		1.1.5 Symbol
		1.1.6 Number
		1.1.7 BigInt
		1.1.8 Object
	1.2 运算符 typeof
	1.3 原始类型的值是唯一的
	1.4 包装类型
2. 显式转换
	2.1 转为 Boolean
	2.2 转为 String
	2.3 转为 Number
	2.4 转为 Object
	2.5 Object 转为原始类型
        2.6 转为 BigInt
        2.7 转为 Symbol
3. 隐式转换
	3.1 逻辑运算
		3.1.1 逻辑非运算
		3.1.2 逻辑与和逻辑或运算
	3.2 属性访问
	3.3 算术运算
		3.3.1 一元算术运算
		3.3.2 二元算术运算
	3.4 相等运算
		3.4.1 严格相等运算（===）
		3.4.2 相等运算（==）
	3.5 加法运算
	3.6 关系运算
	3.7 其他的隐式转换
4. 数组对象转为原始类型
5. 结语
```


## 0. 引言

关于 JavaScript 的类型转换，网上有很多资料，但大多都没有引入较新的标准，说得不怎么清晰明白，甚至还有一些错误夹杂其中。我把 ECMA 262 标准中相关的部分整理了一下，并重新组织语言，重新描述步骤，把还未正式列入标准的 `BigInt` 引入进来。

为了方便测试，文章中，对可能暴露在全局的变量，我都用 `var` 取代 `let` 来申明。引用函数时，为了能更直观地显示参数的类型，我通常会使用 Java 的方法签名。


## 1. 关于类型


#### 1.1 类型列表

这儿说的类型，不是指 `typeof` 运算符返回的类型，而是指语言类型（ECMAScript Language Types)，包含 `Undefined`, `Null`, `Boolean`, `String`, `Symbol`, `Number`, `BigInt` 和 `Object`。其中，`BigInt` 还处于提案阶段，未正式列入标准。


###### 1.1.1 Undefined

只有一个值，即 `undefined`。未赋值的变量的值即为 `undefined`。


###### 1.1.2 Null

只有一个值，即 `null`。


###### 1.1.3 Boolean

代表逻辑值，包含 `true` 和 `false` 两个值。


###### 1.1.4 String

代表 16 位无符号整数（即 character）的有序序列。长度介于 $[0, \  2^{53} - 1]$。长度为 0 的字符串为空字符串。具体见 UNICODE 标准。


###### 1.1.5 Symbol

代表可以作为对象属性名的非字符串的值。


###### 1.1.6 Number

在 JavaScript 中，`Number` 实际上为 64 位浮点数，具体见 IEEE 754: IEEE Standard for Floating-Point Arithmetic。

`Number` 类型能表示的数的范围为：

```
[ -Number.MAX_VALUE, -Number.MIN_VALUE ] ∪ { -0, +0 } ∪ [ Number.MIN_VALUE, -Number.MAX_VALUE ]
```

这中间很多值不能被精确表示，这是浮点数“用精度换范围”的策略。

能精确表示的整数值的范围为：

```
[ Number.MIN_SAFE_INTEGER, Number.MAX_SAFE_INTEGER ]
```

以上这些都遵循 IEEE 754 标准，引入的这些值也可以根据 IEEE 754 计算出来。这儿不作详细介绍。

`Number` 类型还包含了一些特殊值：`+0`, `-0`, `+Infinity`, `-Infinity` 和 `NaN`。在 JavaScript 中，只规定了 `NaN`，而没有规定 `qNaN` 和 `sNaN`。

`Number` 类型的值的相等运算有两个特殊情况：

```js
+0 === -0;            // => true
NaN === NaN;          // => false
```

这时需要用到 `Object.is()` 函数来检测：

```js
Object.is(+0, -0);    // => false
Object.is(NaN, NaN);  // => true
```


###### 1.1.7 BigInt

代表长整数。`BigInt` 类型还只是提案，但已成为事实标准，当前大多数浏览器、Node.js 皆已实现此类型。详见提案：BigInt: Arbitrary precision integers in JavaScript。

```js
0n;
99999999999999999999999999999999999999999n;
```


###### 1.1.8 Object

这儿的 `Object`，包含 `Array` 和 `Function`。


#### 1.2 运算符 `typeof`

运算符 `typeof` 和语言类型不一样的地方：

```js
typeof null;          // => "object"

typeof function(){};  // => "function"
```

其他类型的表现和其语言类型一致：

```js
typeof undefined;    // => "undefined"
 
typeof true;         // => "boolean"
 
typeof "";           // => "string" 

typeof Symbol();     // => "symbol"
 
typeof 0;            // => "number"

typeof 0n;           // => "bigint"

typeof {};           // => "object"
typeof [];           // => "object"
```


#### 1.3 原始类型的值是唯一的

原始类型（Primitive Type），包含 `Undefined`, `Null`, `Boolean`, `String`, `Symbol`, `Number` 和 `BigInt`，他们的值都是唯一的。

所谓唯一的，就是说，所有的 `undefined`，所有的 `null`，所有的 `true`，所有的 `1`，所有的 `"abc"`，所有的 `99999999999n`，都是同一个值。

举例来说：

```js
Symbol() === Symbol();                 // => false

Symbol("a") === Symbol("a");           // => false

Symbol.for("a") === Symbol.for("a");   // => true
```

上面的代码中，两个 `Symbol()`、两个 `Symbol("a")` 不是同一个值，但两个 `Symbol.for("a")` 是同一值。


#### 1.4 包装类型

**包装类型的实例是对象，而不是原始类型的值**。

```js
typeof new Boolean();               // => "object"

new Boolean() === new Boolean();    // => false
```

先说下 `Object()` 和 `valueOf()` 函数。`Object(T t)` 函数可以对原始类型的值进行包装：如果参数是对象，则返回该对象，否则返回该原始类型的值的包装对象（未讨论参数缺省及为 `null` 和 `undefined` 的情况）。包装类型的 `valueOf()` 函数，可以返回其对应的原始类型的值。

```js
var x = Object(0);
Object(x) === x;     // => true，返回对象自身
x.valueOf();         // => 0, 拆箱
```

要特别说明的是 `Symbol` 和 `BigInt` 类型。对于 `Symbol` 和 `BigInt` 函数，其不支持 `new` 运算符，但并不是说这两种类型不能被包装，用上面介绍的 `Object()` 函数即可实现包装功能，而且可以用 `valueOf()` 函数进行拆箱操作：

```js
var x = Symbol();
var y = Object(x);    // 包装对象

y.constructor === Symbol;                       // => true
Object.getPrototypeOf(y) === Symbol.prototype;  // => true
y.valueOf() === x;                              // => true
```

在原始类型上调用方法，实际上就会进行一次装箱操作。


## 2. 显式转换

JavaScript 类型转换的环境是多样的。在本小节中，为了使代码清晰，如无特殊情况，我用各类型的构造函数的普通调用（不用 `new` 运算符）来讨论，这些函数包含 `Boolean`, `String`, `Number`, `Object`, `Symbol` 和 `BigInt`。至于调用 `toString()` 和 `valueOf()` 等方法，这些都比较直观，直接调用方法就行，不再赘述。


#### 2.1 转为 Boolean

`Boolean` 的转换行为比较单纯，只有当参数为 `undefined`, `null`, `false`, `""`, `+0`, `-0`, `NaN` 和 `0n` 才返回 `false`，否则返回 `true`。

```js
Boolean(undefined);   // => false

Boolean(null);        // => false

Boolean(Boolean b);   // => b

Boolean(Number n);    // 如果参数为 ±0 或 NaN，返回 false, 否则返回 true

Boolean(BigInt bi);   // 如果参数为 0n，返回 false, 否则返回 true
 
Boolean(String s);    // 如果参数为空字符串，返回 false, 否则返回 true

Boolean(Symbol s);    // => true

Boolean(Object o);    // => true
```


#### 2.2 转为 String

```js
String(undefined);      // => "undefined"

String(null);           // => "null"

String(Boolean b);      // 如果参数为 false，返回 "false"；否则返回 "true"

String(String s);       // => s
 
String(Number n);       // 如果值参数为 NaN, 返回 "NaN"
                        // 如果参数为 +0 或 -0，返回 "0"
                        // 返回值包含参数的字符串（包含 "Infinity"）
                        // 如果值小于 0，返回值前导 "-"
                        // 值是否用科学计数法表示的情况比较复杂，这儿不作介绍
 
String(BigInt bi);      // 返回包含十进制数字序列的字符串
 
String(Object o);       // 见 2.5 Object 转为原始类型
```

`Symbol` 是不支持隐式转换为 `String` 的，会抛出 `TypeError`。

当参数为 `Symbol` 时，`String()` 函数会特殊处理：

```js
String(Symbol s);      // => "Symbol(" + Description + ")"
                       // Description 即生成 Symbol 时所用的字符串，如：
                       // "a" 即 Symbol("a") 的 Description

new String(Symbol s);  // 抛出 TypeError
```


#### 2.3 转为 Number

```js
Number(undefined);    // => NaN

Number(null);         // => 0

Number(Boolean b);    // 如果参数为 false，返回 0, 否则返回 1

Number(String s);     // 只支持基础平面（BMP）的字符，如包含代理字符（无论成不成对），返加 NaN
                      // 关于基础平面、代理对，见 UNICODE 标准
                      // 忽略参数首尾的空白字符，如果是空符串，返回 0
                      // 如果不能转换，返加 NaN
                      // 只能返回能表示的最接近的值
                      // 如果值太大，返回 Infinity 或 -Infinity
 
Number(Symbol s);     // 抛出 TypeError

Number(Number n);     // => n

Number(BigInt bi);    // 只能返回能表示的最接近的值
                      // 如果值太大，返回 Infinity 或 -Infinity
 
Number(Object o);     // 见 2.5 Object 转为原始类型
```

`BigInt` 不支持隐式转换为 `Nubmer`，会抛出 `TypeError`。但 `BigInt` 也是可以执行算术、比较和相等运算且不运算时不会转换类型的，见 3.3 算术运算 和 3.4 相等运算。

```js
// BigInt 的隐式转换
1n + 5;     // TypeError
isNaN(0n);  // TypeError
```

支持的字符串的语法，我作了一个归纳：

```js
// 空白字符串序列
/^\s*$/

// 二进制整数值，0b 或 0B 打头，不能为负值
/^\s*0[bB][01]+\s*$/

// 八进制整数值，0o 或 0O 打头，不能为负值
/^\s*0[oO][0-7]+\s*$/

// 十六进制整数值，0x 或 0X 打头，不能为负值
/^\s*0[xX][0-9a-fA-F]+\s*$/

// +Infinity 和 -Infinity
/^\s*[\+\-]?Infinity\s*$/

// 十进制
// 可以用符号打头
// 数字部分可以用小数点打头
// 支持科学计数法
/^\s*[\+\-]?((\d+)|(\.\d+)|(\d+\.\d+))([eE][\+\-]?\d+)?\s*$/
```

关于能表示的最接近的值，我举个例子：

```js
Number.MAX_SAFE_INTEGER;          // => 9007199254740991
Number.MAX_SAFE_INTEGER + 1;      // => 9007199254740992
Number.MAX_SAFE_INTEGER + 2;      // => 9007199254740992
Number.MAX_SAFE_INTEGER + 3;      // => 9007199254740994
// 以上可知，9007199254740993 不能被 64 位浮点数精确表示

Number("9007199254740993");        // => 9007199254740992
Number(9007199254740993n);         // => 9007199254740992
```

不仅仅是超出安全范围的整数，绝大多数小数也不能被浮点数精确表示。


#### 2.4 转为 Object

`Object()` 操作，会对基础类型进行装箱操作，引用类型则直接返回。见 1.4 包装类型。

严格来说，`undefined` 和 `null` 并不能转换为对象。在标准中，有一个 `ToObject()` 抽象运算，也是要抛出错误的。这里列出，求个全，供参考。

```js
Object(undefined);      // => {}

Object(null);           // => {}

Object(Boolean b);      // 装箱

Object(Number b);       // 装箱

Object(String s);       // 装箱

Object(Symbol s);       // 装箱，见 1.4 包装类型

Object(Number b);       // 装箱

Object(BigInt bi);      // 装箱，见 1.4 包装类型

Object(Object o);       // => o
```


#### 2.5 Object 转为原始类型

`Object` 转为原始类型内容较多，且显示转换和隐式转换很难分开讨论，单列一小节于此。

先说下期望类型（Preferred Type）。期望类型有三种，分别是 `"default"`, `"string"` 和 `"number"`：

```js
// 期望类型为 "default"
obj + 0;                      // 加法运算符：+
obj == 5;                     // 相等运算符：==, !=，操作数为 null 或 undefined 除外
 
// 期望类型为 "string"
String(obj);                  // String()
[1, 2, 3, 4, 5][obj];         // 属性访问
 
// 期望类型为 "number"
Number(obj);                  // Number()
+obj;                         // 一元算术运算符：+, -, ++, --
obj - 0;                      // 二元算术运算符： -, *, /, %, **
obj < 2;                      // 关系运算符： >, >=, <, <=
```

在 ES 2018 版标准中，期望类型为 `"number"` 时，操作数会被转换为 `Number`。但在引入了 `BigInt` 后，当期望类型为 `"number"` 时，转换后为的值为 `Number` 或 `BigInt`，都是符合预期的。因为 `BigInt` 也可能进行算术、比较、相等运算。

关于转换方法，我作了一个整理：


>&nbsp;
>1. 取得对象 `o` 的期望类型 `type`
>&nbsp;
>2. 检测对象 `o` 是否包含 `Symbol.toPrimitive` 属性
>&nbsp;
>3. 如果 `o[Symbol.toPrimitive]` 存在，
>&nbsp;
>3.1 调用 `o[Symbol.toPrimitive](type)`
>&nbsp;
>3.2 返回值如果是原始类型，则返回，否则，抛出 `TypeError`
>&nbsp;
>4. 如果 `o[Symbol.toPrimitive] `不存在
>&nbsp;
>4.1 如果 `type` 是 `"string"`, 依次调用 `o.toString()` 和 `o.valueOf()`，直到返回原始类型的值
>&nbsp;
>4.2 如果 `type` 是 `"number"` 或 `"default"`, 依次调用 `o.valueOf()` 和 `o.toString()`，直到返回原始类型的值
>&nbsp;
>4.3 如果都以上两个函数都没有返回原始类型的值，抛出 `TypeError`
>&nbsp;


先讨论 **`Symbol.toPrimitive` 属性存在的情况**。


```js
var o = {
  [Symbol.toPrimitive](type) {
    switch (type) {
      case "number":
        return 1;
      case "string":
        return 3;
      case "default":
        return 5;
    }
  },
  toString: () => 7,
  valueOf:  () => 9,
};

// 期望类型为 "default"
o + 0;                  // => 5
o == 5;                 // => true

// 期望类型为 "string"
String(o);             // => "3"
[1, 2, 3, 4, 5][o];    // => 4

// 期望类型为 "number"
Number(o);             // => 1
o - 0;                 // => 1
o < 2;                 // => true
```

再说下 `Date` 构造函数，其参数有不同的形式，当以 `new Date(T t)` 的形式调用时，参数的期望类型为 `"default"`。

```js
// Date() 函数的参数有不同的形式
new Date();
new Date(100);
new Date(2019, 0, 1);
new Date("Tue Jan 01 2000 00:00:00 GMT+0800 (China Standard Time)");
new Date("test");     // => Invalid Date


var o = {
  [Symbol.toPrimitive](type) {
    switch (type) {
      case "number":
        return Date.now();
      case "string":
        return Date.now().toString();
      case "default":
        return "Tue Jan 01 2000 00:00:00 GMT+0800 (China Standard Time)";
    }
  },
};

// 期望类型为 "default"
new Date(o);      // => Sat Jan 01 2000 00:00:00 GMT+0800 (China Standard Time)


// 注意：Date(T t) 和 new Date(T t) 行为不一样
```

可见，`Symbol.toPrimitive` 这个属性存在，`toString()` 和 `valueOf()` 就只能打酱油了。如果 `Symbol.toPrimitive` 属性存在，且值不为函数会怎么样？当然是抛出 `TypeError` 错误了，这相当于调用一个非函数的值。

再来说 **`Symbol.toPrimitive` 这个属性不存在的情况**。

如果 `Symbol.toPrimitive` 属性不存在，那么就轮到 `toString()` 和 `valueOf()` 登场了：如果期望类型如果为 `"default"`, 会被修改为 `"number"`。对于 `"number"` 的期望类型，会先调用 `valueOf()` 方法，再调用 `toString()` 方法；对于 `"string"` 的期望类型，会先调用 `toString()` 方法，再调用 `valueOf()` 方法，直到返回原始类型，如果都未返回原始类型，会抛出 `TypeError` 错误。

上面的例子改造一下：

```js
var o = {
  toString: () => 1,
  valueOf:  () => 3,
};

// 期望类型为 "default"，valueOf() 先被调用：
o + 0;                // => 3
o == 3;               // => true

// 期望类型为 "string"，toString() 先被调用：
String(o);            // => "1"
[1, 2, 3, 4, 5][o];   // => 2

// 期望类型为 "number"，valueOf() 先被调用：
Number(o);            // => 3
o - 0;                // => 3
o < 2;                // => false
```


#### 2.6 转为 BigInt

转为 `BigInt`，和转为 `Number` 类似。`BigInt()` 的参数可以是 `Boolean`，`String`，`Number` 或 `BigInt`。

```js
BigInt(null);             // TypeError
 
BigInt(undefined);        // TypeError 

BigInt(true);             // => 1n
BigInt(false);            // => 0n 

BigInt(String s);         // 行为和 Number(String s) 相似，但只支持整数
                          // 空字符串或空白字符串转为 0n
                          // 支持二进制、八进制、十六进制，也不能带符号
                          // 十进制可以带符号，但不支持科学计数法
                          // 转换出错抛出 SyntaxError
 
BigInt(Symbol s);         // TypeError
 
BigInt(Number n);         // 只能是整数，否则抛出 RangeError
                          // 如 n 为 -0, 行为 和 n = 0 一样
 
BigInt(BigInt bi);        // => bi
 
BigInt(Object o);         // 见 2.5 Object 转为原始类型
```

转为 `BigInt` 类型，只会出现在两种情况下：一是显示转换，二是对象转为原始类型时。

```js
BigInt(11) % 3n;                // => 2n

1n + { valueOf: () => 1n };     // => 2n
```


#### 2.7 转为 Symbol

`Symbol` 的行为与其他类型是不一样的。`Symbol()` 函数的参数如果是 `undefined`，相当于忽略 `Description`，否则，会将参数转为字符串，并作为 `Description`。

```js
Symbol();                // => Symbol()

Symbol(undefined);       // => Symbol()

Symbol(T t);             // 相当于 Symbol(String(t))，其中 T 不为 Undefined

Symbol(Symbol());        // TypeError，相当于 Symbol(String(Symbol()))
```

转为 `Symbol` 的环境，只有两种情况：一是显示转换（其实也不是转换，只是生成一个 `Symbol` 而已），二是对象转为原始类型。

```js
var s = Symbol();
var o ={
  s: 1,
  toString: () => s,
};
o[o];                 // => 1
```


## 3. 隐式转换

JavaScript 涉及到的隐式类型转换的环境，大致有以下方面。

#### 3.1 逻辑运算

###### 3.1.1 逻辑非运算
执行逻辑非运算时，隐式转换为 `Boolean`。

```js
!NaN;     // => true

!![];     // => true
```


###### 3.1.2 逻辑与和逻辑或运算

逻辑与（`&&`）和逻辑或（`||`），相当与管道运算。此运算不进行转换，但会进行逻辑计算。运算时，会一直往后计算操作数，直到逻辑与遇到假值，或逻辑或遇到真值，否则一直执行到最后，然后返回最后计算的操作数。

```js
1 && 2 && 3;        // => 3
 
1 && 2 && "" && 3;  // => ""
 
1 || 2 || 3;        // => 1
 
0 || false || 5;    // => 5
```

逻辑与常用作空检测，逻辑或常用来取真值，都非常节约代码。


#### 3.2 属性访问

属性访问，包含点语法( `obj.key` ) 和方括号语法( `obj[key]` )。本小节讨论的是方括号语法，此语法也可能产生隐式的类型转换。

>&nbsp;
>1. 如果操作数为 Object，以 "string" 为期望类型将其转换为原始类型。
>&nbsp;
>2. 如果操作数为 `String` 或 `Symbol`，则返回。
>&nbsp;
>3. 如果操作数不为 `String` 或 `Symbol`，转换为 `String` 并返回。
>&nbsp;


例：

```js
var s = Symbol();
var o1 = {
  valueOf: () => 1,
  toString: () => s,
};
var o2 = {
  valueOf: () => s,
  toString: () => 1,
};

var d = {
  "1": "111",
  [s]: "sss",
};

d[o1];                // => "sss"
d[o2];                // => "111"
```


#### 3.3 算术运算

###### 3.3.1 一元算术运算

一元算术运算（`+, -, ++, --`），有如下转换规则：

>&nbsp;
>1. 如果为一元 `+` 运算，则期望类型为 `Number`。
>&nbsp;
>2. 如果不为一元 `+` 运算，则期望类型为 `Number` 或 `BigInt`。
>&nbsp;

例：

```js
+true;                      // => 1
+{ valueOf: () => -1 };     // => -1
+1n;                        // TypeError

-true;                      // => -1
-{ valueOf: () => -1 };     // => 1
-1n;                        // => -1n
```


###### 3.3.2 二元算术运算

操作数参与二元算术运算（`-, *, /, %, **`）时，操作数会转为 `Number` 或 `BigInt`。

>&nbsp;
>1. 如果操作数为 `Object`，以 `"number"` 为期望类型将其转换为原始类型。
>&nbsp;
>2. 如果操作数分别为 `Number` 或 `BigInt`，抛出 `TypeError`。
>&nbsp;
>3. 如果操作数都为 `Number` 或 `BigInt`，执行算术运算。
>&nbsp;
>4. 将非 `Number` 或 `BigInt` 的操作数转为 `Number` 后执行算术运算。
>&nbsp;
>5. 对于两个 `BitInt` 参与的二元算术运算，有如下限定：
>&nbsp;
>5.1 除法运算( `/`)，相当于整除。
>&nbsp;
>5.2 除法运算( `/` )和求余运算( `%` )中，除数不能为 `0n`，否则抛出 `RangeError`。
>&nbsp;
>5.3 幂运算( `**` )中，指数不能为负。否则抛出 `RangeError`。
>&nbsp;

例：

```js
2 ** { valueOf: () => "10" };    // => 1024
2n ** { valueOf: () => 10n };    // => 1024n

2 ** { valueOf: () => 10n };     // TypeError
2n ** { valueOf: () => 10 };     // TypeError


7n / 3n;     // => 2n
7n % 3n;     // => 1n

7n / 0n;     // RangeError
7n % 0n;     // RangeError
7n ** -1n;   // RangeError
```




#### 3.4 相等运算

本小节详细讨论严格相等运算( `===` ) 和相等运算( `==` )，`!==` 和 `!=` 为相反运算，不再赘诉。

###### 3.4.1 严格相等运算（===）

严格相等运算（`===`）不会出现类型转换，记下来的原因，一来可以与相等运算( `==` )对比，二来相等运算也会涉及到此运算。

对于运算 `x === y`，有

>&nbsp;
>1. 当 `x` 和 `y` 类型不相同，返回 false。
>&nbsp;
>2. 当 `x` 和 `y` 类型相同：
>&nbsp;
>2.1. 当类型为对象时，判断是否为同一个引用。
>&nbsp;
>2.2 当类型为 `Number` 时，`NaN` 和 `NaN` 不相等，`+0` 和 `-0` 相等，否则判断值是否相等。
>&nbsp;
>2.3 当类型为其他原始类型，直接判断值是否相等。
>&nbsp;


###### 3.4.2 相等运算（==）

对于 `x == y`，有

>&nbsp;
>1. 如果类型相同，返回 `x === y`
>&nbsp;
>2. 如果类型不相同：
>&nbsp;
>2.1 `null` 和 `undefined` 相等。
>&nbsp;
>2.2 `null` 或 `undefined` 和其他类型的值都不相等。
>&nbsp;
>2.3 如果一个操作数为 `Object`, 以 `"default"` 为期望类型转为原始类型，再执行 `==` 比较。
>&nbsp;
>2.4 如果一个操作数为 `Symbol`，执行 `===` 比较。
>&nbsp;
>2.5 如果操作数分别为 `BigInt` 和 `String`，把 `BigInt` 转为 `String`，再执行 `===` 比较。
>&nbsp;
>2.6 如果操作数分别为 `BigInt` 和 `Number`，判断数值是否相等。
>&nbsp;
>2.7 把非 `Number` 或 `BigInt` 的操作数转为 `Number` 后进行数值比较。
>&nbsp;


数值比较，即比较 `Number` 或 `BigInt` 数值的大小：

```js
2n == 1;                                      // => false; 
0n == -0;                                     // => true;

BigInt("9".repeat(1000)) == Infinity;         // => false
BigInt("9".repeat(1000)) == Number.MAX_VALUE; // => false
```

来几个刺激的例子：

```js
// Example 1
var o = {
  valueOf: () => null,
};
o == "null";          // => false, 实际为 null == "null"，规则 2.2
o == null;            // => false，规则 2.2
o + 1;                // => 1，实际为 0 + 1


// Example 2
var o = {
  valueOf: () => "0xf",
};
o == 15;             // => true，实际为 "0xf" == 15


// Example 3
var o = Object.assign(new Number(3), {
  valueOf: () => 1,
});
o == 1;              // => true
[1,2,3,4][o];        // => 4，调用的是 toString()，见 3.2 属性运算


// Example 4
9007199254740993n == 9007199254740992;       // => false, 右边实际为 9007199254740992
9007199254740993n == "9007199254740993";     // => true，规则 2.3.2


// Example 5
Number("9".repeat(500)) == "Infinity";   // => true, 规则 2.3.4
BigInt("9".repeat(500)) == "Infinity";   // => false，规则 2.3.2
```


#### 3.5 加法运算

本小节说的加法运算( `+` )，不包含一元运算（如 `+5`）。运算符( `+` )既可以执行加法算术运算，也可以执行字符串连接运算。遵循如下规则：

>&nbsp;
>1. 如果操作数为 `Object`，以 `"default"` 为期望类型将其转换为原始类型。
>&nbsp;
>2. 当其中一个操作数为 `String` 时，执行字符串连接运算。
>&nbsp;
>3. 当两个操作数都为 `Number` 或 `BigInt` 时，执行算术运算。
>&nbsp;
>4. 当两个操作数分别为 `Number` 或 `BigInt` 时，抛出 `TypeError`。
>&nbsp;
>5. 将非 `Number` 或 `BigInt` 的操作数转为 `Number` 后，执行算术运算。
>&nbsp;

例：

```js
1n + 2;                             // TypeError 
1n + "2";                           // => "12" 

1 + true;                           // => 2
"1" + true;                         // => "1true"

1 + { valueOf: () => 2 };           // => 3
1n + { valueOf: () => 2n };         // => 3n

{ valueOf: () => 5 } + null;        // => 5
{ valueOf: () => "5" } + null;      // => "5null"

{ valueOf: () => 5 } + undefined;   // => NaN
{ valueOf: () => "5" } + undefined; // => "5undefined"
```


#### 3.6 关系运算

本小节讨论的关系运算，包含 `>, >=, <, <=`，遵循如下规则：

>&nbsp;
>1. 如果操作数为 `Object`，以 `"number"` 为期望类型将其转换为原始类型。
>&nbsp;
>2. 如果两个操作数都为 `String` 时，执行字符串比较。
>&nbsp;
>3. 如两个操作数都为或分别为 `Number` 或 `BigInt` 时，执行数值比较。
>&nbsp;
>4. 将非 `Number` 或 `BigInt` 的操作数转为 `Number` 后，执行数值比较。

来个**刺激**点的例子：

```js
var o = {
  valueOf:  ((x) => {
    return () => Math.random() > .5 ? x++ : x--;
  })(0),
};

o > o;              // => ？
o >= o && o <= o;   // => ？
```

字符串比较，需逐位比较每个字符（character）的大小，字符大的，大于字符小的；长度大的，大于长度小的。

```js
"a" > "b";                              // => false
"a".charCodeAt(0) > "b".charCodeAt(0);  // => false

"a" > " b";                             // => true
"a".charCodeAt(0) > " b".charCodeAt(0); // => true

"abc" > "ab";                           // => true
```

数值比较时，`NaN` 参与的运算返回 `false`；`+0` 和 `-0` 被认为是相等的；`BitInt` 可以和 `Number` 进行比较。

```js
NaN > 1;         // => false
NaN < 1;         // => false
 
+0 > -0;         // => false
0n <= -0;        // => true

BigInt("9".repeat(1000)) > Number.MAX_VALUE;     // => true
BigInt("9".repeat(1000)) < Infinity;             // => true
```


#### 3.7 其他的隐式转换

在 JavaScript 中，还存在另外一些隐式转换的环境。

```js
// 作为条件语句或循环语句的条件，可能会隐式转换为 Boolean
if (T) {}
for ( ; T; ) {}
while (T) {}
do {} while(T)


// 期望参数为某种类型的内置函数，也可能产生隐式转换
isNaN(1n);                    // TypeError 
isNaN(Symbol());              // TypeError

 
// 期望参数为整数的内置函数
"322324".charAt(1.9);         // => "2", 1.9 被转为 1
"322324".charAt(1n);          // TypeError


// 按位运算也可能产生隐式转换
"1.1" << 2;                   // => 4，1.1被转为 1
1n << 2;                      // TypeError
```

这些转换，都大同小异，不再深入讨论。


## 4. 数组对象转为原始类型

上文提到，数组对象的语言类型也是对象，所以**数组对象转为原始类型时，也脱离不了上面的规则**。

比较特殊的是，数组对象的 `valueOf()` 方法返回自身，所以转为原始类型时就只能调用 `toString()` 方法，而 `toString()` 方法相当于把各元素转为 `String` 后再用 `","` 连接起来。值为 `undefined` 或 `null` 的元素会被转为空字符串。

通常，这并没有什么特别的，但如果数组对象只包含 1 个元素，就相当于把该元素转为 `String` 并返回；如果数组元素不包含任何元素，就相当于返回一个空字符串。如果需要转为 `Number` 时，再把该字符串转为数字——**是先转为字符串，再转为数字的**。

```js
String(Array(2));   // => ","

[] == 0;            // => true
  
[[[null]]] == "";   // => true
 
[1] + 1;            // => "11", 相当于 "1" + 1

[1] + 1n;           // => "11"，相当于 "1" + 1n

[1n] - 1;           // => 0，相当于 "1" - 1

[1n] - 1n;          // TypeError，相当于 1 - 1n

[Symbol()] - 0;     // TypeError
```


## 5. 结语

JavaScript 类型转换，说起来复杂，其本质也就那样。一定要记住，**JavaScript 的类型转换不具备传递性**，这点非常具有迷惑性。

```js
Boolean(" ");          // => true
" " == true;           // => false


1 - 0;              // => 1
[1] - 0;            // => 1
true - 0;           // => 1
[true] - 0;         // => NaN
```

只要了解了上面的规则，并谨记“**类型转换，一事一议**”的原则，这东西也没有那么复杂。

JavaScript 作为动态类型的语言，用起来有时很爽，有时很不爽。文中就有些例子，用起来是非常得不愉快。但吐槽归吐槽，用的时候还得用。理解了过后，大多数时候还是很直观，很节约代码的。



##

**参考文献**：

ECMA-262: ECMAScript 2018 Language Specification

BigInt: Arbitrary precision integers in JavaScript. https://github.com/tc39/proposal-bigint

IEEE 754-2008: IEEE Standard for Floating-Point Arithmetic

The Unicode Standard Version 12.0: Core Specification
