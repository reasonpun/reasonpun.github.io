---
title: 15 Javascript codes you will always need
date: 2022-03-20 09:53:08
categories:
- JavaScript
tags:
- JavaScript
- 编程技巧
---

### 随机排列数组

使用排序和随机方法对数组进行排序非常容易。

```
const shuffleArray = (arr) => arr.sort(() => 0.5 - Math.random());
console.log(shuffleArray([1, 2, 3, 4]));
// Result: [ 1, 4, 3, 2 ]
```

<!--more-->

### 检查日期是否有效

使用以下代码段检查给定日期是否有效。

```
const isDateValid = (...val) => !Number.isNaN(new Date(...val).valueOf());
isDateValid("December 17, 1995 03:24:00");
// Result: true
```

### 复制到剪贴板

使用 navigator.clipboard.writeText 轻松将任何文本复制到剪贴板。

```
const copyToClipboard = (text) => navigator.clipboard.writeText(text);
copyToClipboard("Hello World");
```

### 查找一年中的哪一天

查找给定日期的哪一天。

```
const dayOfYear = (date) =>
  Math.floor((date - new Date(date.getFullYear(), 0, 0)) / 1000 / 60 / 60 / 24);
dayOfYear(new Date());
// Result: 272
```

### 大写字符串

Javascript 没有内置的大写函数，因此我们可以为此目的使用以下代码。

```
const capitalize = str => str.charAt(0).toUpperCase() + str.slice(1)
capitalize("follow for more")
// Result: Follow for more
```

### 求两天之间的天数

使用以下代码段查找给定日期之间的天数。

```
const dayDif = (date1, date2) => Math.ceil(Math.abs(date1.getTime() - date2.getTime()) / 86400000)
dayDif(new Date("2020-10-21"), new Date("2021-10-22"))
// Result: 366
```

### 清除所有 Cookie

您可以通过使用 document.cookie 访问 cookie 并清除它来轻松清除存储在网页上的所有 cookie。

```
const clearCookies = document.cookie.split(';').forEach(cookie => document.cookie = cookie.replace(/^ +/, '').replace(/=.*/, `=;expires=${new Date(0).toUTCString()};path=/`));
```

### 生成随机十六进制

您可以使用 Math.random 和 padEnd 属性生成随机十六进制颜色。

```
const randomHex = () => `#${Math.floor(Math.random() * 0xffffff).toString(16).padEnd(6, "0")}`;
console.log(randomHex());
// Result: #92b008
```

### 从数组中删除重复项

您可以使用 JavaScript 中的 Set 轻松删除重复项。这可是非常管用的一个大招！

```
const removeDuplicates = (arr) => [...new Set(arr)];
console.log(removeDuplicates([1, 2, 3, 3, 4, 4, 5, 5, 6]));
// Result: [ 1, 2, 3, 4, 5, 6 ]
```

### 从 URL 获取查询参数

你可以轻松取到URL中的参数，不管是window.location还是goole.com?search=easy&page=3。

```
const getParameters = (URL) => {
  URL = JSON.parse('{"' + decodeURI(URL.split("?")[1]).replace(/"/g, '\\"').replace(/&/g, '","').replace(/=/g, '":"') +'"}');
  return JSON.stringify(URL);
};
getParameters(window.location)
// Result: { search : "easy", page : 3 }
```

### 借助Date记录时间

我们可以从给定日期以小时::分钟::秒的格式记录时间。

```
const timeFromDate = date => date.toTimeString().slice(0, 8);
console.log(timeFromDate(new Date(2021, 0, 10, 17, 30, 0))); 
// Result: "17:30:00"
```

### 检查一个数字是偶数还是奇数

```
const isEven = num => num % 2 === 0;
console.log(isEven(2)); 
// Result: True
```

### 求数字的平均值

使用 reduce 方法找到多个数字之间的平均值。

```
const average = (...args) => args.reduce((a, b) => a + b) / args.length;
average(1, 2, 3, 4);
// Result: 2.5
```

### 检查数组是否为空

检查数组是否为空的简单单行程序将返回 true 或 false。

```
const isNotEmpty = arr => Array.isArray(arr) && arr.length > 0;
isNotEmpty([1, 2, 3]);
// Result: true
```

### 获取选定的文本

使用内置的 getSelectionproperty 获取用户选择的文本。

```
const getSelectedText = () => window.getSelection().toString();
getSelectedText();
```

### 检测暗模式

使用以下代码检查用户的设备是否处于暗模式。

```
const isDarkMode = window.matchMedia && window.matchMedia('(prefers-color-scheme: dark)').matches
console.log(isDarkMode) // Result: True or False
```

