+++
title = "ES6: ArrayBuffer, TypedArray & Blob"
description = "ES6: ArrayBuffer, TypedArray & Blob"
tags = [
    "Blob"
]
date = "2017-11-28"
categories = [
    "Development",
    "nodejs",
]
thumbnail = "images/meteor-guide/whatsapp2.png"
+++

ES6提供了ArrayBuffer和TypedArray，让前端也可以直接操作编辑二进制数据， 网页中的类型为file的input标签， 也可以通过FileReader转化为二进制， 然后再做编辑。

<!--more-->

ArrayBuffer : 代表内存之中的一段二进制数据， 通过它我们可以直接创建二进制对象，然后使用相关的方法和属性。

new ArrayBuffer(32)， 从内存中申请32个字节.

把ArrayBuffer转换为可以编辑的TypedArray， 然后修改typedArray的内容， 接着再把二进制的数据转化为blob类型的数据，再把blob对象转化为一个url数据， 接着就可以把blob文件下载下来：

``` 
var ab = new ArrayBuffer(32)
var iA = new Int8Array(ab)
iA[0] = 97;//把二进制的数据的首位改为97 ，97为小写字母a的ascll码；
var blob = new Blob([iA], {type: "application/octet-binary"});//把二进制的码转化为blob类型
var url = URL.createObjectURL(blob);
window.open(url)

```