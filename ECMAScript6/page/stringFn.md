# stringFn

字符串的新增方法

String.fromCodePoint()

ES5 提供`String.fromCharCode()`方法，用于从 Unicode 码点返回对应的字符，但是这个方法不能识别码点大于`0xFFFF`的字符。

```纯文本
String.fromCharCode(0x20BB7);
// "ஷ"
```


ES6 提供了`String.fromCodePoint()`方法，可以识别`0xFFFF`的字符。

```纯文本
String.fromCodePoint(0x20BB7);
// "𠮷"
```


String.raw()

ES6 为原生的 String 对象提供了一个`raw`方法。该方法返回一个斜杠都被转义的字符串，往往用于模版字符串的处理方法。

```纯文本
String.raw`Hi\n${2+3}!`
// 实际返回 "Hi\\n5!"，显示的是转义后的结果 "Hi\n5!"

String.raw`Hi\u000A!`;
// 实际返回 "Hi\\u000A!"，显示的是转义后的结果 "Hi\u000A!"
```


实例方法：codePointAt()

JavaScript 内部，字符以 UTF-16 的格式存储，每个字符固定为两个字节。对于那些需要4个字节存储的字符（Unicode码点大于`0xFFFF`的字符），JavaScript 会认为他们是两个字符。

```纯文本
var s = '𠮷';

s.length; // 2
s.charAt(0); // ""
s.charAt(1); // ""
s.charCodeAt(0); // 55362
s.charCodeAt(1); // 57271
```


ES6 提供了`codePointAt()`方法，能够正常处理4个字节存储的字符，返回一个字符的码点。

```纯文本
let s = '𠮷';

s.codePointAt(0)
```


实例方法：normalize()

许多欧洲语言有语调符号和重音符号。为了表示它们，Unicode 提供了两种方法。一种是直接提供带重音符号的字符；另一种是提供合成符号。<br />这两种表示方法，在视觉和语义上都等价，但 JavaScript 不能识别。

```纯文本
'\u01D1'==='\u004F\u030C' //false

'\u01D1'.length // 1
'\u004F\u030C'.length // 2
```


ES6 提供字符串实例的`normalize()`方法，用来将字符的不同表示形式统一为同样的形式，这称为 Unicode 正规化。

```纯文本
'\u01D1'.normalize() === '\u004F\u030C'.normalize()
// true
```


`normalize`方法可以接受一个参数指定`normalize`的形式，参数的四个可选值如下。

> 


1. NFC：默认参数，标准等价合成
2. NFD：标准等价分解
3. NFKC: 兼容等价合成
4. NFKD: 兼容等价分解

实例方法：includes()、startsWith()、endsWith()

传统上，JavaScript 只有`indexOf`方法，可以用来确定一个字符串是否包含在另一个字符串中。<br />ES6 又提供了三种新方法：

> includes: 返回布尔值，表示是否找到了参数字符串<br />startsWith: 返回布尔值，表示参数字符串是否在原始字符串的头部<br />endsWith: 返回布尔值，表示参数字符串是否在原始字符串的尾部


```纯文本
let s = 'hello world';

s.includes('o'); // true
s.startsWith('hello'); // true
s.endsWith('world'); // true
```


这个三个方法都支持第二个参数，表示开始搜索的位置

```纯文本
let s = 'Hello world!';

s.startsWith('world', 6) // true
s.endsWith('Hello', 5) // true
s.includes('Hello', 6) // false
```


实例方法：repeat()

`repeaet`方法返回一个新字符串，表示将原字符重复n次

```纯文本
'x'.repeat(3); // "xxx"
'hello'.repeat(2); // 'hellohello'
'na'.repeat(0); // ""
```


实例方法：padStart(), padEnd()

ES2017 引入了字符串补全长度功能。

```纯文本
'x'.padStart(5, 'ab'); // 'ababx'
'x'.padStart(4, 'ab'); // 'abax'

'x'.padEnd(4, 'ab'); // 'xabab'
'x'.padEnd(4, 'ab'); // 'xaba'
```


`padStart()`的常见用途为数值补全位数。

```纯文本
'1'.padStart(10, '0'); // "0000000001"
'12'.padStart(10, '0'); // "0000000012"
'123456'.padStart(10, '0'); // "0000123456"
```


另一个用途是提示字符串格式

```纯文本
'12'.padStart(10, 'YYYY-MM-DD'); // "YYYY-MM-12"
'09-12'.padStart(10, 'YYYY-MM-DD'); // "YYYY-09-12"
```


实例方法：trimStart(), trimEnd()

ES2019 对字符串实例新增了`trimStart()`和`trimEnd()`这两个方法。它们的行为与`trim()`一致，`trimStart`消除字符串头部的空格，`trimEnd()`消除尾部的空格。它们返回的都是新字符串，不会修改原始字符。

```纯文本
const s = '   abc  ';

s.trim(); // "abc"
s.trimStart(); // "abc  "
s.trimEnd(); // "   abc"
```


实例方法：matchAll()

`matchAll`方法返回一个正则表达式在当前字符串的所有匹配

