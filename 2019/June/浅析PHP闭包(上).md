## 浅析PHP闭包(上)

> Anonymous functions, also known as closures, allow the creation of functions which have no specified name.

在PHP中, 闭包函数, 也叫匿名函数. 允许创建一个没有函数名的函数. 与JavaScript的闭包函数不一样.

> JavaScript闭包是由函数以及创建该函数的词法环境组合而成。这个环境包含了这个闭包创建时所能访问的所有局部变量

```JavaScript
function makeFunc() {
    var name = "Mozilla";
    function displayName() {
        alert(name);
    }
    return displayName;
}
var myFunc = makeFunc();
myFunc();
```

简单来说, JavaScript闭包就是能够读取外部函数的内部变量. 

在PHP中, 简单分成全局作用域和函数作用域. 函数作用域无法读取全局作用域下局部变量. 但可以通过关键字**global**读取, 也可以通过**超全局变量**访问. 
```PHP
$a = 'global';
$b = 'from $GLOBALS';
function makeFunc()
{
    global $a;
    $b = $GLOBALS['b'];
    echo '来自global关键字 ' . $a; // 'global'
    echo "\n";
    echo '来自超全局变量 ' . $b; // from $GLOBALS
}
makeFunc();
```
闭包函数(匿名函数) 跟普通函数一样方法访问全局变量, 除此之外, 还可以通过关键字**use** 
```PHP
$message = 'use';
$example = function () use ($message) {
    var_dump($message);
};
$example();
```
PHP闭包有什么用? 个人理解是简化代码, 用于函数或方法的回调. 利用回调,可以在运行时, 没有直接关系插入的功能插入函数或类中, 赋予其他人在你不知道情景下扩展你的代码.

```PHP
$map = array_map(function($v) {
    return $v + 1;
}, [1,2]);
print_r($map); // [2, 3]
```