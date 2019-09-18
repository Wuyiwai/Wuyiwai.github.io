---
title: PHP从5.6x 移植到 7.0x的变更
date: 2019-09-17 15:23:54
tags: PHP
categories: PHP
---
> 参考文章
> 1. [PHP5.6到PHP7.0的变更](https://www.php.net/manual/zh/migration70.php)

### 不向后兼容的变更
#### 错误和异常处理相关的变更
在PHP 7 中，很多致命错误以及可恢复的致命错误，都被转换为异常来处理了。 这些异常继承自 **Error** 类，此类实现了 **Throwable** 接口 （所有异常都实现了这个基础接口）。

这也意味着，当发生错误的时候，以前代码中的一些错误处理的代码将无法被触发。 因为在 PHP 7 版本中，已经使用抛出异常的错误处理机制了。 （如果代码中没有捕获 Error 异常，那么会引发致命错误）。

PHP 7 中的错误处理的更完整的描述，请参见 [PHP 7 错误处理](https://www.php.net/manual/zh/language.errors.php7.php)。 本迁移指导主要是列出对兼容性有影响的变更。

##### set_exception_handler() 不再保证收到的一定是 Exception 对象

抛出 Error 对象时，如果 set_exception_handler() 里的异常处理代码声明了类型 Exception ，将会导致 fatal error。

想要异常处理器同时支持 PHP5 和 PHP7，应该删掉异常处理器里的类型声明。如果代码仅仅是升级到 PHP7，则可以把类型 Exception 替换成 Throwable。


```
<?php
// PHP 5 时代的代码将会出现问题
function handler(Exception $e) { ... }
set_exception_handler('handler');

// 兼容 PHP 5 和 7
function handler($e) { ... }

// 仅支持 PHP 7
function handler(Throwable $e) { ... }
?>
```

##### 间接使用变量、属性和方法的变化
表达式 | PHP 5 的解析方式 | PHP 7 的解析方式
---|---|---
$$foo['bar']['baz'] | ${$foo['bar']['baz']} | ($$foo)['bar']['baz'] |
$foo->$bar['baz'] | $foo->{$bar['baz']} | ($foo->$bar)['baz'] |
$foo->$bar['baz'] () | $foo->{$bar['baz']}() | ($foo->$bar)['baz'] () |
Foo::$bar['baz'] () | Foo::{$bar['baz']}() | (Foo::$bar)['baz'] () |

##### list() 不再以反向的顺序来进行赋值
list() 现在会按照变量定义的顺序来给他们进行赋值，而非反过来的顺序。 通常来说，这只会影响list() 与数组的[]操作符一起使用的案例，如下所示：
```
<?php
list($a[], $a[], $a[]) = [1, 2, 3];
var_dump($a);
?>
Output of the above example in PHP 5:[3,2,1]

Output of the above example in PHP 7:[1,2,3]
```

#### foreach的变化
##### foreach不再改变内部数组指针
在PHP7之前，当数组通过 foreach 迭代时，数组指针会移动。现在开始，不再如此，见下面代码
```
<?php
$array = [0, 1, 2];
foreach ($array as &$val) {
    var_dump(current($array));
}
?>
Output of the above example in PHP 5:
int(1)
int(2)
bool(false) //这个例子存疑，需要验证

Output of the above example in PHP 7:
int(0)
int(0)
int(0)
```

##### foreach 通过值遍历时，操作的值为数组的副本。
当默认使用通过值遍历数组时，foreach 实际操作的是数组的迭代副本，而非数组本身。这就意味着，foreach 中的操作不会修改原数组的值。

这其实也意味着有一定的内存成本。yield使用时可以减轻这种迭代成本。

##### 十六进制字符串不再被认为是数字
含十六进制字符串不再被认为是数字。例如：
```
<?php
var_dump("0x123" == "291");
var_dump(is_numeric("0x123"));
var_dump("0xe" + "0x1");
var_dump(substr("foo", "0x1"));
?>

Output of the above example in PHP 5:
bool(true)
bool(true)
int(15)
string(2) "oo"

Output of the above example in PHP 7:
bool(false)
bool(false)
int(0)

Notice: A non well formed numeric value encountered in /tmp/test.php on line 5
string(3) "foo"
```
filter_var() 函数可以用于检查一个 string 是否含有十六进制数字,并将其转换为integer:
```
<?php
$str = "0xffff";
$int = filter_var($str, FILTER_VALIDATE_INT, FILTER_FLAG_ALLOW_HEX);
if (false === $int) {
    throw new Exception("Invalid integer!");
}
var_dump($int); // int(65535)
?>
```

##### 所有 ext/mysql 函数已被删掉
如何选择不同的 MySQL API，详情请见 选择 [MySQL API](https://www.php.net/manual/zh/mysqlinfo.api.choosing.php)。

##### yield 变更为右联接运算符，可以通过括号来消除歧义
```
<?php
echo yield -1;
// 在之前版本中会被解释为：
echo (yield) - 1;
// 现在，它将被解释为：
echo yield (-1);

yield $foo or die;
// 在之前版本中会被解释为：
yield ($foo or die);
// 现在，它将被解释为：
(yield $foo) or die;
?>
```

##### INI 文件中 # 注释格式被移除
在 INI 文件中，不再支持以 # 开始的注释行， 请使用 ;（分号）来表示注释。 此变更适用于 php.ini 以及用 parse_ini_file() 和 parse_ini_string() 函数来处理的文件。

##### 在数值溢出的时候，内部函数将会失败
将浮点数转换为整数的时候，如果浮点数值太大，导致无法以整数表达的情况下， 在之前的版本中，内部函数会直接将整数截断，并不会引发错误。 在 PHP 7.0 中，如果发生这种情况，会引发 E_WARNING 错误，并且返回 NULL。

##### 相等的元素在排序时的顺序问题
由于内部排序算法进行了提升， 可能会导致对比时被视为相等的多个元素之间的顺序不稳定。

在对比时被视为相等的多个元素之间的排序顺序是不可信赖的，即使是相等的两个元素， 他们的位置也可能被排序算法所改变。

### 新特性
##### 标量类型声明
标量类型声明 有两种模式: 强制 (默认) 和 严格模式。 现在可以使用下列类型参数（无论用强制模式还是严格模式）： 字符串(string), 整数 (int), 浮点数 (float), 以及布尔值 (bool)。它们扩充了PHP5中引入的其他类型：类名，接口，数组和 回调类型。

```
<?php
// Coercive mode
function sumOfInts(int ...$ints)
{
    return array_sum($ints);
}

var_dump(sumOfInts(2, '3', 4.1));
以上例程会输出：

int(9)
```

要使用严格模式，一个 declare 声明指令必须放在文件的顶部。这意味着严格声明标量是基于文件可配的。 这个指令不仅影响参数的类型声明，也影响到函数的返回值声明（参见 返回值类型声明, 内置的PHP函数以及扩展中加载的PHP函数）

##### 返回值类型声明
PHP 7 增加了对**返回类型声明**的支持。类似于**参数类型声明**，返回类型声明**指明了函数返回值的类型**。可用的类型与参数声明中可用的类型相同。

```
<?php

function arraysSum(array ...$arrays): array
{
    return array_map(function(array $array): int {
        return array_sum($array);
    }, $arrays);
}

print_r(arraysSum([1,2,3], [4,5,6], [7,8,9]));
以上例程会输出：

Array
(
    [0] => 6
    [1] => 15
    [2] => 24
)
```

##### null合并运算符

如果变量存在且值不为NULL， 它就会返回自身的值，否则返回它的第二个操作数。
```
<?php
// Fetches the value of $_GET['user'] and returns 'nobody'
// if it does not exist.
$username = $_GET['user'] ?? 'nobody';
// This is equivalent to:
$username = isset($_GET['user']) ? $_GET['user'] : 'nobody';

// Coalesces can be chained: this will return the first
// defined value out of $_GET['user'], $_POST['user'], and
// 'nobody'.
$username = $_GET['user'] ?? $_POST['user'] ?? 'nobody';
?>
```

##### 太空船操作符
太空船操作符用于比较两个表达式。当$a小于、等于或大于$b时它分别返回-1、0或1。 比较的原则是沿用 PHP 的常规比较规则进行的。

```
<?php
// 整数
echo 1 <=> 1; // 0
echo 1 <=> 2; // -1
echo 2 <=> 1; // 1

// 浮点数
echo 1.5 <=> 1.5; // 0
echo 1.5 <=> 2.5; // -1
echo 2.5 <=> 1.5; // 1
 
// 字符串
echo "a" <=> "a"; // 0
echo "a" <=> "b"; // -1
echo "b" <=> "a"; // 1
?>
```

##### 通过define()定义常量数组
Array 类型的常量现在可以通过 define() 来定义。在 PHP5.6 中仅能通过 const 定义。
```
<?php
define('ANIMALS', [
    'dog',
    'cat',
    'bird'
]);

echo ANIMALS[1]; // 输出 "cat"
?>
```

##### Group use declarations
从同一 namespace 导入的类、函数和常量现在可以通过单个 use 语句 一次性导入了。
```
<?php

// PHP 7 之前的代码
use some\namespace\ClassA;
use some\namespace\ClassB;
use some\namespace\ClassC as C;

use function some\namespace\fn_a;
use function some\namespace\fn_b;
use function some\namespace\fn_c;

use const some\namespace\ConstA;
use const some\namespace\ConstB;
use const some\namespace\ConstC;

// PHP 7+ 及更高版本的代码
use some\namespace\{ClassA, ClassB, ClassC as C};
use function some\namespace\{fn_a, fn_b, fn_c};
use const some\namespace\{ConstA, ConstB, ConstC};
?>
```

##### Generator delegation
生成器yield from,值得关注.

现在，只需在最外层生成其中使用 yield from， 就可以把一个生成器自动委派给其他的生成器， Traversable 对象或者 array。
[传送门](https://www.php.net/manual/zh/language.generators.syntax.php#control-structures.yield.from)
```
<?php

function gen()
{
    yield 1;
    yield 2;

    yield from gen2();
}

function gen2()
{
    yield 3;
    yield 4;
}

foreach (gen() as $val)
{
    echo $val, PHP_EOL;
}

?>
```

##### 新加的整数除法函数intdiv()
```
<?php

var_dump(intdiv(10, 3));
?>
以上例程会输出：

int(3)
```

### 值得注意的变更的函数
- dirname() 增加了可选的第二个参数, depth, 获取当前目录向上 depth 级父目录的名称
- substr() 现在当 start 的值与 string 的长度相同时将返回一个空字符串。

### 值得关注的新的类和接口
##### Exception 等级
- Throwable
- Error
- TypeError
- ParseError
- AssertionError
- ArithmeticError
- DivisionByZeroError

### 移除的扩展
- ereg
- mssql
- mysql
- sybase_ct