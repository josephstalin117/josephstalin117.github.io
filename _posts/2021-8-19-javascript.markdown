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

查看类型
```
typeof "John"                 // Returns "string"
typeof 3.14                   // Returns "number"
typeof NaN                    // Returns "number"
typeof false                  // Returns "boolean"
typeof [1,2,3,4]              // Returns "object"
typeof {name:'John', age:34}  // Returns "object"
typeof new Date()             // Returns "object"
typeof function () {}         // Returns "function"
typeof myCar                  // Returns "undefined" *
typeof null                   // Returns "object"
```

array
```
const fruits = ["Banana", "Orange", "Apple", "Mango"];
fruits.push("Kiwi");   // Adds "Kiwi" to fruits

//foreach
const a = ["a", "b", "c"];
a.forEach((element) => {
    console.log(element);
});
```