---
title: 从PHP 7.1.x 移植到 PHP 7.2.x的变更
date: 2019-09-18 10:12:15
tags: PHP
categories: PHP
---
> 参考文章
> 1. [从PHP 7.1.x 移植到 PHP 7.2.x](https://www.php.net/manual/zh/migration72.php)

<!--more-->

### 新特性
##### 新的对象类型object
引进了可用于逆变（contravariant）参数输入和协变（covariant）返回任何对象类型。
```
<?php
function test(object $obj) : object{
    return new SplQueue();
}

test(new StdClass());
```

##### 通过名称加载扩展
扩展文件不再需要通过文件加载 (Unix下以.so为文件扩展名，在Windows下以 .dll 为文件扩展名) 进行指定。可以在php.ini配置文件进行启用, 也可以使用 dl() 函数进行启用。

##### 允许重写抽象方法(Abstract method)
当一个抽象类继承于另外一个抽象类的时候，继承后的抽象类可以重写被继承的抽象类的抽象方法。
```
<?php
abstract class A{
    abstract function test(string $s);
}

abstract class B extends A{
    // overridden - still maintaining contravariance for parameters and covariance for return
    abstract function test($s) : int;
}
```

### 废弃的功能
##### __autoload() 方法
__autoload() 方法已被废弃， 因为和 spl_autoload_register() 相比功能较差 (因为无法链式处理多个 autoloader)， 而且也无法在两种 autoloading 样式中配合使用。

##### each() 函数
使用此函数遍历时，比普通的 foreach 更慢， 并且给新语法的变化带来实现问题。因此它被废弃了。

##### assert() 一个字符串参数
assert() 字符串参数将要求它能被 eval() 执行。 考虑到可能被执行远程代码，废弃了字符串的 assert()，最好提供 bool 的表达式。