# number

数值的扩展

二进制和八进制表示法

ES6 提供了二进制和八进制数值的新写法，分别用`0b`(或`0B`)和`0o`（或`0O`）表示

```纯文本
0b111110111 === 503
0o767 = 503
```


如果要将`0b`和`0o`前缀的字符串转化为十进制，要使用`Number`方法

```纯文本
Number('0b111'); // 7
Number('0o10'); // 8
```


Number.isFinite(), Number.isNaN()

`Number.isFinite()`用来检查一个数值是否为有限的

```纯文本
Number.isFinite(15); // true
Number.isFinite(Infinity); // false
```


`Number.isNaN()`用来检查一个值是否为`NaN`

```纯文本
Number.isNaN(NaN); // true
Number.isNaN('true' / 0); // true
Number.isNaN('15'); // false
```


Number.parseInt(), Number.parseFloat()

ES6 将全局方法`parseInt()`和`parseFloat()`，移植到`Number`对象上，行为完全保持不变。

```纯文本
// ES5写法
parseInt(12.5); // 12
parseFloat('123.45#'); // 123.45

// ES6写法
Number.pareInt(12.5); // 12
Number.parseFloat('123.45#'); 123.45
```


这样做的目的，是逐步减少全局性方法，使得语言逐步模块化

```纯文本
Number.parseInt === parseInt; // true
Number.parseFloat === parseFloat; // true
```


Number.isInteger()

`Number.isInteger()`用来判断一个数值是否为整数

```纯文本
Number.isInteger(25); // true
Number.isInteger(25.1); // false
```


JavaScript内部,整数和浮点数采用的是同样的存储方式，所以`25`和`25.0`被视为同一个值

```纯文本
Number.isInteger(25); // true
Number.isInteger(25.0); // true
```


Number.EPSILON

ES6 在`Number`对象上，新增一个极小的常量`Number.EPSILON`。根据规定，它表示1与大于1的最小浮点数之间的差

```纯文本
Numer.EPSILON === Math.pow(2, -52); // true
Number.EPSILON; // 2.220446049250313e-16
Number.EPSILON.toFixed(20); // "0.00000000000000022204"
```


> `Number.EPSILON`实际上是 JavaScript 能够表示的最小精度。误差如果小于这个值，就可以认为已经没有意义了，即不存在误差了


因此，`Number.EPSILON`的实质是一个可以接受的最小误差范围

```纯文本
function withinErrorMargin(left, right) {
	return Math.abs(left - right) < Number.EPSILON * Math.pow(2, 2);
}

0.1 + 0.2 === 0.3; // false
withinErrorMargin(0.1 + 0.2, 0.3); // true

1.1 + 1.3 === 2.4; // false
withinErrorMargin(1.1 + 1.3, 2.4); // true
```


Number.isSafeInteger()

JavaScript 能够准确表示的整数范围在`-2^53`到`2^53`之间（不含两个端点），超过这个范围，无法精确表示这个值

```纯文本
Math.pow(2, 53); // 9007199254740992

9007199254740992; // 9007199254740992
9007199254740993; // 9007199254740992

Math.pow(2, 53) === Math.pow(2, 53) + 1; // true
```


ES6 引入了`Number.MIN_SAFE_INTEGER`和`Number.MAX_SAFE_INTEGER`这两个常量，用来表示这个范围的上下限

```纯文本
Number.MAX_SAFE_INTEGER === Math.pow(2, 53) - 1; // true

Number.MAX_SAFE_INTEGER === 9007199254740991; // true

Number.MIN_SAFE_INTEGER === -Number.MAX_SAFE_INTEGER; // true

Number.Min_SAFE_INTEGER === -9007199254740991; // true
```


`Number.isSafeInteger()`用来判断一个整数是否在这个范围之内

```纯文本
Number.isSafeInteger('a'); // false
Number.isSafeInteger(3); // true
Number.isSafeInteger(1.2); // false

Number.isSafeInteger(Number.MIN_SAFE_INTEGER); // true
Number.isSafeInteger(Number.MAX_SAFE_INTEGER); // true
Number.isSafeInteger(Number.MIN_SAFE_INTEGER - 1); // false
Number.isSafeInteger(Number.MAX_SAFE_INTEGER + 1); // false
```


Math 对象的扩展

ES6 在 Math 对象上新增了 17 个与数学相关的方法

Math.trunc()

去除一个数的小数部分，返回整数部分

```纯文本
Math.trunc(4.1); // 4
Math.trunc(-0.123); // -0
```


Math.sign()

判断一个数是正数、负数还是零

```纯文本
Math.sign(-5); // -1
Math.sign(5); // 1
Math.sign(0); // 0
Math.sign(-0); // -0
Math.sign(NaN); // NaN
```


Math.cbrt()

计算一个数的立方根

```纯文本
Math.cbrt(-1); // -1
Math.cbrt(0); // 0
Math.cbrt(1); // 1
Math.cbrt(27); // 3
```


Math.clz32()

将参数转化为 32 位无符号整数形式，返回 32 位里面有多少个前导 0

```纯文本
Math.clz32(0); // 32
Math.clz32(1); // 31
```


左移运算符（`<<`）与`Math.clz32`方法直接相关

```纯文本
Math.clz32(0); // 32
Math.clz32(1); // 31
Math.clz32(1 << 1); // 30
Math.clz32(1 << 2); // 29
Math.clz32(1 << 29); // 2
```


Math.imul()

返回两个数以 32 位带符号整数形式相乘的结果，返回的也是一个 32 位的带符号整数

```纯文本
Math.imul(2, 4); // 8
Math.imul(-1, 8); // -8
Math.imul(-2, -2); // 4
```


Math.fround()

返回一个数的 32 位单精度浮点数形式

```纯文本
Math.fround(0); // 0
Math.fround(1); // 1
Math.fround(2 ** 24 - 1); // 16777215
```


Math.hypot()

返回所有参数平方和的平方根

```纯文本
Math.hypot(3, 4); // 5
```


指数运算符

ES2016 新增了一个指数运算符（`**`）

```纯文本
2 ** 2; // 4
2 ** 3; // 8
```


这个运算符的一个特点是右结合

```纯文本
2 ** 3 ** 2； // 512
// 相当于 2 ** (3 ** 2)
```


指数运算符可以与等号结合，形成一个新的赋值运算符（`**=`）

```纯文本
let a = 2;

a **= 3; // 8
```


BigInt 数据类型

JavaScript 所有数字都保存成 64 位浮点数，这给数值的表示带来了两大限制。一是数值的精度只能到 53 个二进制，大于这个范围的整数，JavaScript 是无法精确表示的。二是大于或者等于 2 的 1024次方的数值，JavaScript 无法表示，只返回`Infinity`

```纯文本
Math.pow(2, 53) === Math.pow(2, 53) + 1; // true

Math.pow(2, 1024); // Infinity
```


ES2020 引入了一种新的数据类型 BigInt（大整数），来解决这个问题。BigInt 只用来表示整数，没有位数的限制，任何位数的整数都可以精度表示

```纯文本
const a = 2172141653n;
const b = 15346349309n;

a * b; // 33334444555566667777n

// 普通整数无法保持精度
Number(a) * Number(b); // 33334444555566670000
```


为了与 Number 类型区别，BigInt 类型的数据必须添加后缀`n`

```纯文本
123; // Number
123n; // BigInt

// BigInt 的运算
1n + 2n; // 3n
```


BigInt 同样可以使用各种进制表示，都要加上后缀`n`

```纯文本
0b1101n; // 二进制
0o777n; // 八进制
0xFFn; 	// 十六进制
```


BigInt 与普通整数是两种值，它们之间并不想等

```纯文本
42n === 42; // false
```


typeof 运算符对于 BigInt 类型的数据返回 `bigint`

```纯文本
typeof 123n; // "bigint"
```


BigInt 可以使用负号（`-`），但不能用正号（`+`）

```纯文本
-42n; // -42n
+42n; // TypeError: Cannot convert a BigInt value to a number
```


计算70的阶乘

```纯文本
let p = 1n;

for (let i = 1n; i <= 70n; i ++){
	p *= i;
}
p; // 11978571669969891796072783721689098736458938142546425857555362864628009582789845319680000000000000000n
```


BigInt 对象

JavaScript 原生提供`BigInt`对象，可以用作构造函数生成 BigInt 类型的数值。

```纯文本
BigInt(123); // 123n
BigInt('123'); // 123n
BigInt(false); // 0n
BigInt(true); 1n
```


`BigInt()`构造函数必须有参数，没有或者是小数，都会报错

```纯文本
BigInt(); // TypeError
BigInt(12.2); // RangeError
```


BigInt 对象继承了 Object 对象的两个实例方法

`BigInt.prototype.toString()`<br />`BigInt.prototype.valueOf()`

它还继承了 Number 对象的一个实例方法

`BigInt.prototype.toLocaleString()`

此外，还提供了三个静态方法

`BigInt.asUintN(width, BigInt)`：给定的 BigInt 转为 0 到 2width - 1 之间对应的值<br />`BigInt.asIntN(width, BigInt)`：给定的 BigInt 转为 -2width - 1 到 2width - 1 - 1 之间对应的值<br />`BigInt.parseInt(string[, radix])`：将一个字符串转换成指定进制的 BigInt

```纯文本
const max = 2n ** (64n - 1n) - 1n;

BigInt.asIntN(64, max); // 9223372036854775807n
BigInt.asIntN(64, max + 1n); // -9223372036854775808n
BigInt.asUintN(64, max + 1n); // 9223372036854775808n
```


可以使用`Boolean()`、`Number()`和`String()`将 BigInt 转化为布尔值、数值和字符串

```纯文本
Boolean(0n); // false
Boolean(1n); // true
Number(1n); // 1
String(1n); // "1"
```


在数学运算方面，BigInt 类型的`+`、`-`、`*`和`**`这个四个二元运算符，与 Number 类型的行为一致。除法运算`/`会舍弃小数部分，返回一个整数.

```纯文本
9n / 5n; // 1n
```


BigInt 不能与普通数值进行混合运算

```纯文本
1n + 1.4; // TypeError
```


