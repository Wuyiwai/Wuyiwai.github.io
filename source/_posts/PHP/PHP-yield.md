---
title: PHP yield
date: 2019-10-12 21:41:47
tags: PHP
categories: PHP
---
> 参考文档
> 1. [PHP yield](https://www.php.net/manual/zh/language.generators.overview.php)

<!--more-->

##### 有个坑
> When a generator function is called for the first time, an object of the internal Generator class is returned. This object implements the Iterator interface in much the same way as a forward-only iterator object would, and provides methods that can be called to manipulate the state of the generator, including sending values to and returning values from it.

在一个生成器方法中,当被第一次调用时就会指定返回Generator类型的对象.
```
<?php
function gen_one_to_three($flag) {
    if ($flag) {
        for ($i = 1; $i <= 3; $i++) {
            yield $i;
        }
    } else {
        echo "sss";
        return [1,2,3];
    }

}
$generator = gen_one_to_three(false);
var_dump($generator);
foreach ($generator as $value) {
    echo "$value\n";
}
```
此时不管怎么样都是返回Generator类型的对象,无关if的条件

##### 一个简单的例子
```
<?php
function gen_one_to_three() {
    for ($i = 1; $i <= 3; $i++) {
        //注意变量$i的值在不同的yield之间是保持传递的。
        yield $i;
    }
}

$generator = gen_one_to_three();
foreach ($generator as $value) {
    echo "$value\n";
}
?>

输出:
1
2
3
```
注意:在内部会为生成的值配对连续的整型索引，就像一个非关联的数组。

##### Caution
```
如果在一个表达式上下文(例如在一个赋值表达式的右侧)中使用yield，你必须使用圆括号把yield申明包围起来。 例如这样是有效的：

$data = (yield $value);
而这样就不合法，并且在PHP5中会产生一个编译错误：

$data = yield $value;
The parenthetical restrictions do not apply in PHP 7.

这个语法可以和生成器对象的Generator::send()方法配合使用。
```

##### 关于内存的例子
1. 正常使用foreach迭代数组

```
function getValues() {
    $valuesArray = [];
    // 获取初始内存使用量
    echo round(memory_get_usage() / 1024 / 1024, 2) . ' MB' . PHP_EOL;
    for ($i = 1; $i < 800000; $i++) {
        $valuesArray[] = $i;
        // 为了让我们能进行分析，所以我们测量一下内存使用量
        if (($i % 200000) == 0) {
            // 来 MB 为单位获取内存使用量
            echo round(memory_get_usage() / 1024 / 1024, 2) . ' MB'. PHP_EOL;
        }
    }
    return $valuesArray;
}
$myValues = getValues(); // 一旦我们调用函数将会在这里创建数组
foreach ($myValues as $value) {}

输出:
0.34 MB
8.34 MB
16.34 MB
32.34 MB
```

2. 使用yield返回可迭代对象
```
function getValues() {
    // 获取内存使用数据
    echo round(memory_get_usage() / 1024 / 1024, 2) . ' MB' . PHP_EOL;
    for ($i = 1; $i < 800000; $i++) {
        yield $i;
        // 做性能分析，因此可测量内存使用率
        if (($i % 200000) == 0) {
            // 内存使用以 MB 为单位
            echo round(memory_get_usage() / 1024 / 1024, 2) . ' MB'. PHP_EOL;
        }
    }
}
$myValues = getValues(); // 在循环之前都不会有动作
foreach ($myValues as $value) {} // 开始生成数据

输出:
0.34 MB
0.34 MB
0.34 MB
0.34 MB
```

##### 总结
1. yield返回的是一个继承了Iterator的可迭代对象
2. yield的好处:
    1. 一个生成器允许我们使用循环来迭代一维数组的代码，而不需要在内存中创建是一个数组，这可能会导致超出内存限制。