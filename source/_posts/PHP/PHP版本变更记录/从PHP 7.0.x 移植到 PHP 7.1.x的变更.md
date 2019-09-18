---
title: 从PHP 7.0.x 移植到 PHP 7.1.x的变更
date: 2019-09-17 16:18:52
tags: PHP
categories: PHP
---
> 参考文档
> 1. [从PHP 7.0.x 移植到 PHP 7.1.x](https://www.php.net/manual/zh/migration71.php)

### 新特性
##### 可为空（Nullable）类型
参数以及返回值的类型现在可以通过在类型前加上一个问号使之允许为空。 当启用这个特性时，传入的参数或者函数返回的结果**要么是给定的类型，要么是 null** 。
```
<?php

function testReturn(): ?string{
    return 'elePHPant';
}
var_dump(testReturn());

function testReturn(): ?string{
    return null;
}
var_dump(testReturn());

function test(?string $name){
    var_dump($name);
}

test('elePHPant');
test(null);
test();

以上例程会输出：
string(10) "elePHPant"
NULL
string(10) "elePHPant"
NULL
Uncaught Error: Too few arguments to function test(), 0 passed in...
```

##### 强推Symmetric array destructuring
短数组语法（[]）现在作为list()语法的一个备选项，可以用于将数组的值赋给一些变量（包括在foreach中）。

```
<?php
$data = [
    [1, 'Tom'],
    [2, 'Fred'],
];

// list() style
list($id1, $name1) = $data[0];

// [] style
[$id1, $name1] = $data[0];

// list() style
foreach ($data as list($id, $name)) {
    // logic here with $id and $name
}

// [] style
foreach ($data as [$id, $name]) {
    // logic here with $id and $name
}
```

##### 多异常捕获处理
一个catch语句块现在可以通过管道字符(|)来实现多个异常的捕获。 这对于需要同时处理来自不同类的不同异常时很有用。
```
<?php
try {
    // some code
} catch (FirstException | SecondException $e) {
    // handle first and second exceptions
}
```

##### list现在支持键名
现在list()和它的新的[]语法支持在它内部去指定键名。这意味着它可以将任意类型的数组 都赋值给一些变量（与短数组语法类似）
```
<?php
$data = [
    ["id" => 1, "name" => 'Tom'],
    ["id" => 2, "name" => 'Fred'],
];

// list() style
list("id" => $id1, "name" => $name1) = $data[0];

// [] style
["id" => $id1, "name" => $name1] = $data[0];

// list() style
foreach ($data as list("id" => $id, "name" => $name)) {
    // logic here with $id and $name
}

// [] style
foreach ($data as ["id" => $id, "name" => $name]) {
    // logic here with $id and $name
}
```

##### 支持为负的字符串偏移量
现在所有支持偏移量的字符串操作函数 都支持接受负数作为偏移量，包括通过[]或{}操作字符串下标。在这种情况下，一个负数的偏移量会被理解为一个从字符串结尾开始的偏移量。


```
<?php
var_dump("abcdef"[-2]);
var_dump(strpos("aabbcc", "b", -3));
以上例程会输出：

string (1) "e"
int(3)
Negative string and array offsets are now also supported in the simple variable parsing syntax inside of strings.

<?php
$string = 'bar';
echo "The last character of '$string' is '$string[-1]'.\n";
?>
以上例程会输出：

The last character of 'bar' is 'r'.
```

### 新增的函数
- [is_iterable ( mixed $var ) : bool](https://www.php.net/manual/zh/function.is-iterable.php)