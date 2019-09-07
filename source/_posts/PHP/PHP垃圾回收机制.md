---
title: PHP垃圾回收机制
date: 2019-09-07 15:30:05
tags: PHP 
categories: PHP
---
> 参考文章:
> 1. [PHP垃圾回收机制](https://www.php.net/manual/zh/features.gc.php)

#### 引用计数基本知识
每个php变量存在一个叫"zval"的变量容器中。一个zval变量容器，除了包含变量的类型和值，还包括两个字节的额外信息。第一个是"is_ref"，是个bool值，用来标识这个变量是否是属于引用集合(reference set)。通过这个字节，php引擎才能把普通变量和引用变量区分开来，由于php允许用户通过使用&来使用自定义引用，zval变量容器中还有一个内部引用计数机制，来优化内存使用。第二个额外字节是"refcount"，用以表示指向这个zval变量容器的变量(也称符号即symbol)个数。所有的符号存在一个符号表中，其中每个符号都有作用域(scope)，那些主脚本(比如：通过浏览器请求的的脚本)和每个函数或者方法也都有作用域。
```
//zval定义
typedef struct _zval_struct {
    zvalue_value value;
    zend_uint refcount;
    zend_uchar type;
    zend_uchar is_ref;
  } zval;
  
//zvalue_value定义
typedef union _zvalue_value {
    long lval;
    double dval;
    struct {
        char *val;
        int len;
    } str;
    HashTable *ht;
    zend_object_value obj;
} zvalue_value;
```

当一个变量被赋常量值时，就会生成一个zval变量容器，如下例这样：

##### Example #1 生成一个新的zval容器

```
<?php
$a = "new string";
?>
```

在上例中，新的变量a，是在当前作用域中生成的。并且生成了类型为 string 和值为new string的变量容器。在额外的两个字节信息中，"is_ref"被默认设置为 FALSE，因为没有任何自定义的引用生成。"refcount" 被设定为 1，因为这里只有一个变量使用这个变量容器. 注意到当"refcount"的值是1时，"is_ref"的值总是FALSE. 如果你已经安装了» Xdebug，你能通过调用函数 xdebug_debug_zval()显示"refcount"和"is_ref"的值。

##### Example #2 显示zval信息

```
<?php
xdebug_debug_zval('a');
?>
输出：
a: (refcount=1, is_ref=0)='new string'
```

把一个变量赋值给另一变量将增加引用次数(refcount).

##### Example #3 增加一个zval的引用计数

```
<?php
$a = "new string";
$b = $a;
xdebug_debug_zval( 'a' );
?>
输出：
a: (refcount=2, is_ref=0)='new string'
```

这时，引用次数是2，因为同一个变量容器被变量 a 和变量 b关联.当没必要时，php不会去复制已生成的变量容器。变量容器在”refcount“变成0时就被销毁. 当任何关联到某个变量容器的变量离开它的作用域(比如：函数执行结束)，或者对变量调用了函数 unset()时，”refcount“就会减1，下面的例子就能说明:

##### Example #4 减少引用计数

```
<?php
$a = "new string";
$c = $b = $a;
xdebug_debug_zval( 'a' );
unset( $b, $c );
xdebug_debug_zval( 'a' );
?>
输出：

a: (refcount=3, is_ref=0)='new string'
a: (refcount=1, is_ref=0)='new string'
```

如果我们现在执行 unset($a);，包含类型和值的这个变量容器就会从内存中删除。

复合类型(Compound Types)
当考虑像 array和object这样的复合类型时，事情就稍微有点复杂. 与 标量(scalar)类型的值不同，array和 object类型的变量把它们的成员或属性存在自己的符号表中。这意味着下面的例子将生成三个zval变量容器。

##### Example #5 Creating a array zval


```
<?php
$a = array( 'meaning' => 'life', 'number' => 42 );
xdebug_debug_zval( 'a' );
?>
以上例程的输出类似于：

a: (refcount=1, is_ref=0)=array (
   'meaning' => (refcount=1, is_ref=0)='life',
   'number' => (refcount=1, is_ref=0)=42
)
```

图示:
![一个简单数组的zval](https://raw.githubusercontent.com/Wuyiwai/Source/master/php-gc-1.png)

这三个zval变量容器是: a，meaning和 number。增加和减少”refcount”的规则和上面提到的一样. 下面, 我们在数组中再添加一个元素,并且把它的值设为数组中已存在元素的值:

##### Example #6 添加一个已经存在的元素到数组中

```
<?php
$a = array( 'meaning' => 'life', 'number' => 42 );
$a['life'] = $a['meaning'];
xdebug_debug_zval( 'a' );
?>
输出：
a: (refcount=1, is_ref=0)=array (
   'meaning' => (refcount=2, is_ref=0)='life',
   'number' => (refcount=1, is_ref=0)=42,
   'life' => (refcount=2, is_ref=0)='life'
)
```

图示：
![带有引用的简单数组的zval](https://raw.githubusercontent.com/Wuyiwai/Source/master/php-gc-2.png)

从以上的xdebug输出信息，我们看到原有的数组元素和新添加的数组元素关联到同一个"refcount"2的zval变量容器. 尽管 Xdebug的输出显示两个值为'life'的 zval 变量容器，其实是同一个。 函数xdebug_debug_zval()不显示这个信息，但是你能通过显示内存指针信息来看到。

删除数组中的一个元素，就是类似于从作用域中删除一个变量. 删除后,数组中的这个元素所在的容器的“refcount”值减少，同样，当“refcount”为0时，这个变量容器就从内存中被删除，下面又一个例子可以说明：

##### Example #7 从数组中删除一个元素

```
<?php
$a = array( 'meaning' => 'life', 'number' => 42 );
$a['life'] = $a['meaning'];
unset( $a['meaning'], $a['number'] );
xdebug_debug_zval( 'a' );
?>
以上例程的输出类似于：

a: (refcount=1, is_ref=0)=array (
   'life' => (refcount=1, is_ref=0)='life'
)
```

现在，当我们添加一个数组本身作为这个数组的元素时，事情就变得有趣，下个例子将说明这个。例中我们加入了引用操作符，否则php将生成一个复制。

##### Example #8 把数组作为一个元素添加到自己

```
<?php
$a = array( 'one' );
$a[] =& $a;
xdebug_debug_zval( 'a' );
?>
以上例程的输出类似于：

a: (refcount=2, is_ref=1)=array (
   0 => (refcount=1, is_ref=0)='one',
   1 => (refcount=2, is_ref=1)=...
)
```

图示：
![自引用(curcular reference,自己是自己的一个元素)的数组的zval](https://raw.githubusercontent.com/Wuyiwai/Source/master/php-gc-3.png)

能看到数组变量 (a) 同时也是这个数组的第二个元素(1) 指向的变量容器中“refcount”为 2。上面的输出结果中的"..."说明发生了递归操作, 显然在这种情况下意味着"..."指向原始数组。

跟刚刚一样，对一个变量调用unset，将删除这个符号，且它指向的变量容器中的引用次数也减1。所以，如果我们在执行完上面的代码后，对变量$a调用unset, 那么变量 $a 和数组元素 "1" 所指向的变量容器的引用次数减1, 从"2"变成"1". 下例可以说明:

##### Example #9 Unsetting $a

```
(refcount=1, is_ref=1)=array (
   0 => (refcount=1, is_ref=0)='one',
   1 => (refcount=1, is_ref=1)=...
)
```

图示：
![Zvals after removal of array with a circular reference demonstrating the memory leak](https://raw.githubusercontent.com/Wuyiwai/Source/master/php-gc-4.png)

#### 清理变量容器的问题(Cleanup Problems)
尽管不再有某个作用域中的任何符号指向这个结构(就是变量容器)，由于数组元素“1”仍然指向数组本身，所以这个容器不能被清除 。因为没有另外的符号指向它，用户没有办法清除这个结构，结果就会导致内存泄漏。庆幸的是，php将在脚本执行结束时清除这个数据结构，但是在php清除之前，将耗费不少内存。如果你要实现分析算法，或者要做其他像一个子元素指向它的父元素这样的事情，这种情况就会经常发生。当然，同样的情况也会发生在对象上，实际上对象更有可能出现这种情况，因为对象总是隐式的被引用。

如果上面的情况发生仅仅一两次倒没什么，但是如果出现几千次，甚至几十万次的内存泄漏，这显然是个大问题。这样的问题往往发生在长时间运行的脚本中，比如请求基本上不会结束的守护进程(deamons)或者单元测试中的大的套件(sets)中。后者的例子：在给巨大的eZ(一个知名的PHP Library) 组件库的模板组件做单元测试时，就可能会出现问题。有时测试可能需要耗用2GB的内存，而测试服务器很可能没有这么大的内存。

#### 回收周期(Collecting Cycles)
传统上，像以前的 php 用到的引用计数内存机制，无法处理循环的引用内存泄漏。然而 5.3.0 PHP 使用文章» [引用计数系统中的同步周期回收(Concurrent Cycle Collection in Reference Counted Systems)](http://researcher.watson.ibm.com/researcher/files/us-bacon/Bacon01Concurrent.pdf)中的同步算法，来处理这个内存泄漏问题。

对算法的完全说明有点超出这部分内容的范围，将只介绍其中基础部分。首先，我们先要建立一些基本规则，如果一个引用计数增加，它将继续被使用，当然就不再在垃圾中。如果引用计数减少到零，所在变量容器将被清除(free)。就是说，仅仅在引用计数减少到非零值时，才会产生垃圾周期(garbage cycle)。其次，在一个垃圾周期中，通过检查引用计数是否减1，并且检查哪些变量容器的引用次数是零，来发现哪部分是垃圾。

![垃圾回收算法](https://raw.githubusercontent.com/Wuyiwai/Source/master/php-gc-5.png)

为避免不得不检查所有引用计数可能减少的垃圾周期，这个算法把所有可能根(possible roots 都是zval变量容器),放在根缓冲区(root buffer)中(用紫色来标记，称为疑似垃圾)，这样可以同时确保每个可能的垃圾根(possible garbage root)在缓冲区中只出现一次。仅仅在根缓冲区满了时，才对缓冲区内部所有不同的变量容器执行垃圾回收操作。看上图的步骤 A。

在步骤 B 中，模拟删除每个紫色变量。模拟删除时可能将不是紫色的普通变量引用数减"1"，如果某个普通变量引用计数变成0了，就对这个普通变量再做一次模拟删除。每个变量只能被模拟删除一次，模拟删除后标记为灰（原文说确保不会对同一个变量容器减两次"1",不对的吧）。

在步骤 C 中，模拟恢复每个紫色变量。恢复是有条件的，当变量的引用计数大于0时才对其做模拟恢复。同样每个变量只能恢复一次，恢复后标记为黑，基本就是步骤 B 的逆运算。这样剩下的一堆没能恢复的就是该删除的蓝色节点了，在步骤 D 中遍历出来真的删除掉。

算法中都是模拟删除、模拟恢复、真的删除，都使用简单的遍历即可（最典型的深搜遍历）。复杂度为执行模拟操作的节点数正相关，不只是紫色的那些疑似垃圾变量。

现在，你已经对这个算法有了基本了解，我们回头来看这个如何与PHP集成。默认的，PHP的垃圾回收机制是打开的，然后有个 php.ini 设置允许你修改它：**zend.enable_gc** 。

当垃圾回收机制打开时，每当根缓存区存满时，就会执行上面描述的循环查找算法。根缓存区有固定的大小，可存10,000个可能根,当然你可以通过修改PHP源码文件Zend/zend_gc.c中的常量GC_ROOT_BUFFER_MAX_ENTRIES，然后重新编译PHP，来修改这个10,000值。当垃圾回收机制关闭时，循环查找算法永不执行，然而，可能根将一直存在根缓冲区中，不管在配置中垃圾回收机制是否激活。

当垃圾回收机制关闭时，如果根缓冲区存满了可能根，更多的可能根显然不会被记录。那些没被记录的可能根，将不会被这个算法来分析处理。如果他们是循环引用周期的一部分，将永不能被清除进而导致内存泄漏。

即使在垃圾回收机制不可用时，可能根也被记录的原因是，相对于每次找到可能根后检查垃圾回收机制是否打开而言，记录可能根的操作更快。不过垃圾回收和分析机制本身要耗不少时间。

**除了修改配置zend.enable_gc ，也能通过分别调用gc_enable() 和 gc_disable()函数来打开和关闭垃圾回收机制。调用这些函数，与修改配置项来打开或关闭垃圾回收机制的效果是一样的。即使在可能根缓冲区还没满时，也能强制执行周期回收。你能调用gc_collect_cycles()函数达到这个目的。这个函数将返回使用这个算法回收的周期数。**

允许打开和关闭垃圾回收机制并且允许自主的初始化的原因，是由于你的应用程序的某部分可能是高时效性的。在这种情况下，你可能不想使用垃圾回收机制。当然，对你的应用程序的某部分关闭垃圾回收机制，是在冒着可能内存泄漏的风险，因为一些可能根也许存不进有限的根缓冲区。因此，就在你调用gc_disable()函数释放内存之前，先调用gc_collect_cycles()函数可能比较明智。因为这将清除已存放在根缓冲区中的所有可能根，然后在垃圾回收机制被关闭时，可留下空缓冲区以有更多空间存储可能根。

#### 总结
1. 循环引用造成的内存泄漏
    1. 如果引用计数减少到零，所在变量容器将被清除.
    2. 如果一个zval 的引用计数减少后还大于0，那么它会进入垃圾周期。其次，在一个垃圾周期中，通过检查引用计数是否减1，并且检查哪些变量容器的引用次数是零，来发现哪部分是垃圾。
    3. 循环引用基本上只会出现在 数组和对象中，对象是因为它的本身就是引用
2. 对象和数组的回收
    1. 垃圾收集器:把变量的zend_refcount信息放到回收池,满了会回收.
    2. 垃圾回收算法:
        1. 遍历回收池中的每一个变量，根据每一个变量，再遍历每一个成员，如果成员还有嵌套的话继续遍历。然后把所有成员的 做模拟的 refcount -1。如果此时外部的变量的 引用次数为 0 。那么可以视为垃圾，清楚。如果大于0，那么恢复引用次数，并从垃圾回收池中取出。
        2. 原理:如果你这个变量不是垃圾，那么它的所有成员变量的引用减一之后，必然不会是总变量的引用为0。
3. todo:装个xdebug模拟一下