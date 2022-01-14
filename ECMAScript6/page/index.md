# index

简介

ECMAScript6.0 （以下简称 ES6）是于2015年6月发布的 JavaScript 语言的新一代标准

ECMAScript 历史

- ECMAScript1.0 1997年
- ECMAScript2.0 1998年6月
- ECMAScript3.0 1999年12月
- ECMAScript4.0 2000年，最终没有通过
- ECMAScript5.0 2009年12月
- ECMAScript6.0 2015年6月

> ES4 没有通过?


因为这个版本太激进，对 ES3 做了彻底升级，导致标准委员会的一些成员不愿意接受。2008年7月，终止 ES4 的开发，将其中涉及现有功能改善的一小部分发布为 ESCMAScript3.1。会后不久，ECMAScript3.1 就改名为 ECMAScript5。2009年12月，ECMAScript5.0 正式发布。

关系

- JavaScript: ES + DOM + BOM
- TypeScript: ES6 + 类型系统
- ECMAScript: 语言标准
- ECMAScript6: 最新的语言标准（ES2015、ES2016、ES2017...）
- ECMAScript2015: 2015年的 ES6 版本（ES6 的第一个版本）

> ES6 标准在每年的6月份正式发布一次，作为当年的正式版本


支持

各大浏览器的最新版本，对 ES6 的支持可以[查看](https://kangax.github.io/compat-table/es6/)

Node 是 JavaScript 的服务器运行环境。它对 ES6 的支持度更高。

查看 Node 已经实现的 ES6 特性

```纯文本
$ node --v8-options | grep harmony
```


Babel 转码器

[Babel](https://babeljs.io/) 是一个广泛使用的 ES6 转码器，可以将 ES6 代码转为 ES5 代码，从而实现在现有环境执行

