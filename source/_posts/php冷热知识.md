---
title: php冷热知识
date: 2018-02-23 17:33:02
tags: [php,学习,知识点]
categories: php开发
---
## call_user_func_array()
****
> 调用回调函数，并把一个数组参数作为回调函数的参数

**说明**

`mixed call_user_func_array ( callable $callback , array $param_arr )`

把第一个参数作为回调函数（callback）调用，把参数数组作（param_arr）为回调函数的的参数传入。

**参数 **

|参数名称|说明|
|--|--|
|callback|被调用的回调函数|
|param_arr|要被传入回调函数的数组，这个数组得是索引数组|

**返回值**

返回回调函数的结果。如果出错的话就返回FALSE

例子：

~~~
<?php
function foobar($arg, $arg2) {
    echo __FUNCTION__, " got $arg and $arg2\n";
}
class foo {
    function bar($arg, $arg2) {
        echo __METHOD__, " got $arg and $arg2\n";
    }
}


// Call the foobar() function with 2 arguments
call_user_func_array("foobar", array("one", "two"));

// Call the $foo->bar() method with 2 arguments
$foo = new foo;
call_user_func_array(array($foo, "bar"), array("three", "four"));
?>
~~~

**Example #2 call_user_func_array()使用命名空间的情况**

~~~
<?php

namespace Foobar;

class Foo {
    static public function test($name) {
        print "Hello {$name}!\n";
    }
}

// As of PHP 5.3.0
call_user_func_array(__NAMESPACE__ .'\Foo::test', array('Hannes'));

// As of PHP 5.3.0
call_user_func_array(array(__NAMESPACE__ .'\Foo', 'test'), array('Philip'));
~~~

**Example #3 把完整的函数作为回调传入call_user_func_array()**

~~~
<?php

$func = function($arg1, $arg2) {
    return $arg1 * $arg2;
};

var_dump(call_user_func_array($func, array(2, 4))); /* As of PHP 5.3.0 */
~~~

Example #4 传引用

~~~
<?php
function mega(&$a){
    $a = 55;
    echo "function mega \$a=$a\n";
}
$bar = 77;
call_user_func_array('mega',array(&$bar));
echo "global \$bar=$bar\n";
~~~

> Note:
PHP 5.4之前，如果param_arr里面的参数是引用传值，那么不管原函数默认的各个参数是不是引用传值，都会以引用方式传入到回调函数。虽然以引用传值这种方式来传递参数给回调函数，不会发出不支持的警告，但是不管怎么说，这样做还是不被支持的。并且在PHP 5.4里面被去掉了。而且，这也不适用于内部函数，for which the function signature is honored。如果回调函数默认设置需要接受的参数是引用传递的时候，按值传递，结果将会输出一个警告。call_user_func() 将会返回 FALSE

> Note:
在函数中注册有多个回调内容时(如使用 call_user_func() 与 call_user_func_array())，如在前一个回调中有未捕获的异常，其后的将不再被调用。

## call_user_func()

call_user_func — 把第一个参数作为回调函数调用

> 说明 
mixed call_user_func ( callable $callback [, mixed $parameter [, mixed $... ]] )
第一个参数 callback 是被调用的回调函数，其余参数是回调函数的参数。

***PHP 中 call_user_func 函数 和 call_user_func_array 函数的区别***

> 它们的第一个参数都是被调用的回调函数，call_user_func() 还可以有多个参数，它们都是回调函数的参数，call_user_func_array() 只有两个参数，第二个参数是要被传入回调函数的数组，这个数组得是索引数组。

所以它们最大的区别就是：

. 如果传递一个数组给 call_user_func_array()，数组的每个元素的值都会当做一个参数传递给回调函数，数组的 key 回调掉。
. 如果传递一个数组给 call_user_func()，整个数组会当做一个参数传递给回调函数，数字的 key 还会保留住。

比如有个如下的回调函数：

~~~
function test_callback(){
	$args	= func_get_args();
	$num	= func_num_args();
	echo $num."个参数：";
	echo "<pre>";
	print_r($args);
	echo "</pre>";
}
~~~

然后我们分别使用 call_user_func 函数 和 call_user_func_array 函数进行回调：

~~~
$args = array (
	'foo'	=> 'bar',
	'hello'	=> 'world',
	0	=> 123
);

call_user_func('test_callback', $args);
call_user_func_array('test_callback', $args);
~~~

最后输出结果：

~~~
1 个参数：
Array
(
    [0] => Array
        (
            [foo] => bar
            [hello] => world
            [0] => 123
        )
)

3个参数：
Array
(
    [0] => bar
    [1] => world
    [2] => 123
)
~~~

## func_get_args()、func_get_arg()与func_num_args()

> func_get_args():返回一个包含函数参数列表的数组。 
func_get_arg():返回指定的参数值。 
func_num_args():返回调用函数的传入参数个数,类型是整型。

## spl_autoload_register

> spl_autoload_register — 注册给定的函数作为 __autoload 的实现

将函数注册到SPL __autoload函数队列中。如果该队列中的函数尚未激活，则激活它们。

如果在你的程序中已经实现了__autoload()函数，它必须显式注册到__autoload()队列中。因为 spl_autoload_register()函数会将Zend Engine中的__autoload()函数取代为spl_autoload()或spl_autoload_call()。

如果需要多条 autoload 函数，spl_autoload_register() 满足了此类需求。 它实际上创建了 autoload 函数的队列，按定义时的顺序逐个执行。相比之下， __autoload() 只可以定义一次。

**参数**
*autoload_function*
欲注册的自动装载函数。如果没有提供任何参数，则自动注册 autoload 的默认实现函数spl_autoload()。

*throw*
此参数设置了 autoload_function 无法成功注册时， spl_autoload_register()是否抛出异常。

*prepend*
如果是 true，spl_autoload_register() 会添加函数到队列之首，而不是队列尾部。

Example #1 spl_autoload_register() 作为 __autoload() 函数的替代

~~~
<?php

// function __autoload($class) {
//     include 'classes/' . $class . '.class.php';
// }

function my_autoloader($class) {
    include 'classes/' . $class . '.class.php';
}

spl_autoload_register('my_autoloader');

// 或者，自 PHP 5.3.0 起可以使用一个匿名函数
spl_autoload_register(function ($class) {
    include 'classes/' . $class . '.class.php';
});
~~~

**Example #2 class 未能加载的 spl_autoload_register() 例子**

~~~
<?php
namespace Foobar;
class Foo {
    static public function test($name) {
        print '[['. $name .']]';
    }
}
spl_autoload_register(__NAMESPACE__ .'\Foo::test'); // 自 PHP 5.3.0 起
new InexistentClass;
~~~