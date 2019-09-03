---
title: PHP代码技巧
date: 2019-09-03 18:38:17
tags: PHP
categories: PHP
---
1. [list](https://www.php.net/manual/zh/function.list.php)-把数组中的值赋给一组变量
    - 在 PHP 7.1.0 之前的版本，list() 仅能用于数字索引的数组，并假定数字索引从 0 开始。
    - 像array()一样,这不是真正的函数，而是语言结构
    - PHP 5 里，list() 从最右边的参数开始赋值； PHP 7 里，list() 从最左边的参数开始赋值
    ```
    //一个简单的例子
    $info = array('coffee', 'brown', 'caffeine');
    list($drink, $color, $power) = $info;
    echo "$drink is $color and $power makes it special.\n";
    list($drink, , $power) = $info;
    echo "$drink has $power.\n";...
    
    ```
2. [飞船操作符 <=>](https://www.php.net/manual/en/language.operators.comparison.php)
    ```
    //一个简单的例子
    // Integers
    echo 1 <=> 1; // 0
    echo 1 <=> 2; // -1
    echo 2 <=> 1; // 1
    ```
3. [yield](https://www.php.net/manual/zh/language.generators.php)
    ```
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
    ```
4. 输出当前目录:
    ```
    echo `ls`;
    ```
5. [switch](https://www.php.net/manual/zh/control-structures.switch.php)
    - switch的continue 2;
    - 注意和其它语言不同，continue 语句作用到 switch 上的作用类似于 break。如果在循环中有一个 switch 并希望 continue 到外层循环中的下一轮循环，用 continue 2。
6. 变量名引用
    ```
    //反正我觉得这玩意儿慎用
    $foo = "Hello";
    $bar = "foo";
    echo $$bar;
    输出: Hello
    
    同理:
    function test(){
        echo 'hello world!';
    }
    $fn = 'test';
    $fn();
    输出: hello world!
    ```
7. 待更新
