---
layout: post
title: 使用四种JS符号输出数字123
date: 2017-11-15 11:00:00 +0800
tags:
- javascript
---

一个师弟（现就读于NYU 👏 ）今天问了我个JS的问题，题目如下:

```
Search for JS documentation.
Play in your browser's console(seeing is believing).

Challenge: How to get numerical value 123 in JS in one command with no more than 50
characters, if you can use 4 types of characters: []!+

Hint: +!![] gives you the number 1
```

意思就是如何使用`[`、`]`、 `!`、 `+`这四种符号在一条不超过50个字符的一条命令中输出数字123。

这个题目的提示非常重要: `+!![]`会输出数字1

几番尝试后得出了一种解法:

```
+[+!![] + [+!![] + +!![]] + [+!![] + +!![] + +!![]]]
```

分开解释一下就能很快明白这句命令的含义：

```
# 对任意对象进行双重否定得到的结果都是true，这里的[]代表空数组对象
> !![]
< true
# 对任意对象(除false和0之外)取否得到的结果都为false
> ![]
< false
# 对true进行+操作相当于将bool值转化为Int值，true的值相当于1，这里相当于类型转换
> +true
< 1
# 因此将一个数字与true相加就相当于在数字的值上进行加一操作
> 1.1 + true
< 2.1
# + false则相当于将false转化0
> +false
< 0
# []为一个数组(Array)对象，两个数组对象相加有相应的规则
> [1] + [1]
< "11"
> [1] + 1
< "11"
# 对字符串或字符转数组进行+操作
> +"11"
< 11
> +["111"]
< 111
# 空数组+操作与false＋操作结果相同
> +[]
< 0
```

由以上的例子我们可以将之前的结果一步步进行转化:

```
+[+!![] + [+!![] + +!![]] + [+!![] + +!![] + +!![]]]
# 两次取反得到结果true
+[+true + [+true + +true] + [+true + +true + +true]]
# +true 得到结果1
+[1 + [1 + 1] + [1 + 1 + 1]]
# 将数字进行相加
+[1 + [2] + [3]]
# 数组相加
+["123"]
# 最终结果
123
```
