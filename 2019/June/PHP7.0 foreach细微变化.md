## PHP7.0 foreach细微变化

>  When used in the default by-value mode, foreach will now operate on a copy of the array being iterated rather than the array itself. This means that changes to the array made during iteration will not affect the values that are iterated. 

这句话大概的是说: 当默认使用通过值遍历数组时，foreach 实际操作的是数组的迭代副本，而非数组本身。这意味着在迭代期间对数组所做的更改不会影响迭代的值。PHP 5也是这样的, 只是实现这个逻辑与PHP 7 不同.
```
$arr = [0];
foreach ($arr as $k => $v)
{
    debug_zval_dump($arr);
}
```
打印出来的refcount为3, 说明在foreach中复制数组了, 导致refcount为3. 进一步验证. 
```
$arr = [0];
foreach ($arr as $v) {
	$copy = $arr;
    debug_zval_dump($arr);
}
```
假设数组在循环中复制了, 那么refcount应该为4. 其打印结果跟我猜想一样. 说明数组在foreach进行复制了. 而且不受数组的长度影响. 因为数组长度为2时候, 还是打印4.在PHP5  foreach靠的是数组指针在移动从而达到迭代数组的值.
```
$arr = [0, 1];
foreach ($arr as $v) {
	$copy = $arr;
    debug_zval_dump($arr);
}
```
*注意引用的一些坑*


 



[foreach是如何运行的][1]

[深入理解PHP原理之变量分离/引用(Variables Separation) ][2]

[当我们使用foreach时，内部究竟发生了什么(PHP5)？][3]


  [1]: https://stackoverflow.com/questions/10057671/how-does-php-foreach-actually-work
  [2]: http://www.laruence.com/2008/09/19/520.html
  [3]: https://segmentfault.com/a/1190000005018918