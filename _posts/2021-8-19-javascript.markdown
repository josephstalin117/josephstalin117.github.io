---
layout: post
title:  "javascript"
date:   2021-8-19 20:35:35 +0800
categories: command
---

正则分组

```
//world hello
var str = 'hello world';
var pattern = /([a-z]+)\s([a-z]+)/;
var n_str = str.replace(pattern, "$2 $1");

```

array
```
const fruits = ["Banana", "Orange", "Apple", "Mango"];
fruits.push("Kiwi");   // Adds "Kiwi" to fruits
```