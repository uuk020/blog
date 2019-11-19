## PHP 开发工程师基础篇 - PHP数组

### 数组(Array)

数组是 PHP 中最重要的数据类型, 可以说是掌握数组, 基本上PHP一大半问题都可以解决. PHP 数组与其他编程语言数组概念不一样. 其他编程语言数组是由相同类型的元素（element）的集合所组成的数据结构, 而PHP 数组元素可以为不同类型的元素. 因此说PHP数组不是纯粹的数组, 而是哈希(字典)更为恰当.
PHP数组可以当作真正的数组，或列表（向量），散列表（是映射的一种实现），字典，集合，栈，队列以及更多可能性. 数组分为关联数组(类似字典)和索引数组(纯数组, 键值从数字开始, 一般从0开始).

- [PHP 数组底层实现](https://juejin.im/post/5b967696e51d450e452a74d8)
- [详解 PHP 数组的底层实现：HashTable](https://juejin.im/post/5bce1c35518825781f7b0935)

### 语法
- array 
  ```php
  $arr = array(1, 2, 3);
  ```
- []
  ```php
  $arr = [1, 2, 3];
  ```
### 数组键值注意事项

PHP 数组的键值仅支持字符串(string)和整型(integer). 其他类型均会转换数字或字符串. 浮点类型会被转换为整型类型的, 8.7->8. 布尔类型则会转换为整型类型的, true->1, false->0. Null会被转换为空的字符串, 数组和对象则不会转换成为字符串类型和整型.会抛出一个警告: Illegal offset type. 

注意的是字符串类型假如是有效十进制数字的话, 则会转换为整型. "8"->8. 若不想转换则在数字前添加"+"符号. 
```php
// 02不是有效十进制数字
$arr = ['1' => 'a', '+1' => 'b', '02' => 'c'];
/**
 array(3) {
  [1]=>
  string(1) "a"
  ["+1"]=>
  string(1) "b"
  ["02"]=>
  string(1) "c"
}
 */
var_dump($arr);
// 数组的键值可以不用有键值, 下面这个数组最大数字是6,所以在它后面的元素,则是在它基础上+1
$array = array(
         "a",
         "b",
    6 => "c",
         "d",
);
/**
 array(4) {
  [0]=>
  string(1) "a"
  [1]=>
  string(1) "b"
  [6]=>
  string(1) "c"
  [7]=>
  string(1) "d"
}
 */
var_dump($array);
```

### 数组基本操作(增删改查)
通过在方括号([])内指定键名来给数组赋值实现的增加和修改元素, 也可以省略键名.
- 增加数组的元素
  ```php
  // 索引数组
  $arr = [1, 2, 3];
  $arr[] = 4; // [1, 2, 3, 4]
  // 关联数组
  $arr1 = ['a' => 1, 'b' => 2];
  $arr1['c'] = 3; // ['a' => 1, 'b' => 2, 'c' => 3]
  // 混合数组
  $arr3 = ['a' => 1, 'b' => 2];
  $arr3[1] = 3; // ['a' => 1, 'b' => 2, 1 => 3]
  ```
- 删除数组的元素
  ```php
  $arr = [1, 2, 3];
  // 删除数组中值为3的, 注意的是数组不会被重新索引(索引数组)
  unset($arr[2]);
  $arr[] = 3;
  /**
  array(3) {
  [0]=>
  int(1)
  [1]=>
  int(2)
  [3]=>
  int(3)
   }
   */
  var_dump($arr);
  ```
- 改变数组的元素
  ```php
  $arr = [1, 2, 3];
  $arr[0] = 'a';
  var_dump($arr); ['a', 2, 3]
  ```
- 查询数组的元素
  ```php
  $arr = [1, 2, 3];
  var_dump($arr[0]); // 1
  ```

### 其他类型转换为数组
除了对象(Object)转换数组外, 对于对于任意 integer，float，string，boolean 和 resource 类型, 如果将一个值转换为数组, 将得到一个仅有一个元素的数组,其下标为0.
(array)$scalarValue 与 array($scalarValue) 一样. 而对象(object)类型转换为array. 其单元为该对象的属性。键名将为成员变量名，不过有几点例外：整数属性不可访问；私有变量前会加上类名作前缀；保护变量前会加上一个 '*' 做前缀。这些前缀的前后都各有一个 NULL 字符。这会导致一些不可预知的行为：
```php
class A {
    private $A; // This will become '\0A\0A'
}

class B extends A {
    private $A; // This will become '\0B\0A'
    protected $B // This will \0*\0A
    public $AA; // This will become 'AA'
}
// 会有两个键名为 'AA'，不过其中一个实际上是 '\0A\0A'。
/**
 array(4) {
  ["BA"]=>
  NULL
  ["*B"]=>
  NULL
  ["AA"]=>
  NULL
  ["AA"]=>
  NULL
}
 */
var_dump((array) new B());

```
- [整数属性](https://stackoverflow.com/questions/14547187/what-is-a-integer-property-and-whats-the-meaning-of-0a-0a)


### 数组函数分类
1. **创建数组**
   - array_combine ( array $keys , array $values ) : array — 创建一个数组，用一个数组的值作为其键名，另一个数组的值作为其值
     ```php
     $key = ['a', 'b', 'c'];
     $value = [1, 2, 3];
     print_r(array_combine($key, $value)); // ['a' => 1, 'b' => 2, 'c' => 3]
     ```
   - array_fill_keys ( array $keys , mixed $value ) : array — 使用指定的键和值填充数组
     ```php
      $keys = ['red', 'blue', 'green'];
      var_dump(array_fill_keys($keys, 1)); // ['red' => 1, 'blue' => 1, 'green' => 1]
      ```
   - array_fill ( int $start_index , int $num , mixed $value ) : array — 用给定的值填充数组
      ```php
      var_dump(array_fill(0, 0, 1)); // []
      var_dump(array_fill(1, 3, 'red')); // [1=>'red', 2=>'red', 3=>'red']
      ```
   - array — 新建一个数组
     ```php
     // array() 是语言结构, 不是常规函数
     $a = array(1, 2, 3);
     ```
   - compact ( mixed $varname1 [, mixed $... ] ) : array — 建立一个数组，包括变量名和它们的值
     ```php
     $a = 'apple';
     $b = 'banana';
     $d = 'lemon';
     $sweet = ['b', 'd'];
     $result = compact('a', $sweet); // ['a' => 'apple', 'b' => 'banana', 'd' => 'lemon']
     ```
   - range ( mixed $start , mixed $end [, number $step = 1 ] ) : array — 根据范围创建数组，包含指定的元素
     ```php
     var_dump(range(1, 5)); // [1, 2, 3, 4, 5]
     var_dump(range(0, 100, 10)); // [0, 10, 20, 30, 40, 50, 60, 70, 80, 90, 100]
     ```
     
2. **数组分割**
   - array_chunk ( array $array , int $size [, bool $preserve_keys = FALSE ] ) : array — 将一个数组分割成多个
     ```php
     $a = ['a' => 1, 'B' => 2, 3 => 'c', 'A' => 4, 'b' => 6];
     print_r(array_chunk($a, 2)); // [[1, 2], ['c', 4], [6]]
     print_r(array_chunk($a, 2, true)); // [['a' => 1, 'B' => 2], [3 => 'c', 'A' => 4], ['b' => 6]]
     ```
   - array_slice ( array $array , int $offset [, int $length = NULL [, bool $preserve_keys = FALSE ]] ) : array — 从数组中取出一段
      ```php
      $input = ['a', 'b', 'c', 'e', 'f'];
      var_dump(array_slice($a, 2)); // ['c', 'e', 'f']
      var_dump(array_slice($a, 2, 1)); // ['c']
      var_dump(array_slice($a, -1, 1)); // ['f']
      var_dump(array_slice($a, -1, 1, true)); [4 => 'f']
      var_dump(array_slice($a, 0, 6)); // ['a', 'b', 'c', 'e', 'f']
      ```
3. **数组队列或栈操作**
   - array_pop ( array &$array ) : mixed — 弹出数组最后一个单元（出栈）
      ```php
      $arr1 = ['yellow', 'green', 'red']
      var_dump(array_pop($arr1)); // 'yellow'
      ```
   - array_push ( array &$array [, mixed $... ] ) : int — 将一个或多个单元压入数组的末尾（入栈）
      ```php
      var_dump(array_push([1, 2, 3], 4, 5)) // 2 
      ```
   - array_shift ( array &$array ) : mixed — 将数组开头的单元移出数组
      ```php
      $a ['php', 'c', 'python'];
      var_dump(array_shift($a));// 'php'
      ```
   - array_unshift ( array &$array [, mixed $... ] ) : int — 在数组开头插入一个或多个单元
     ```php
     $queue = ['orange', 'banana'];
     var_dump(array_unshift($queue, 'apple')); // 3
     var_dump($queue);
     ```
4. **数组数学运算**
   - array_count_values( array $array ): array  — 统计数组中所有的值
     ```php
     print_r(array_count_value([1, 'hello', 1, 'hello', 'world'])) // [1=>2, 'hello' => 2, 'world' => 1]
     ```
   - array_product ( array $array ) : number — 计算数组中所有值的乘积
     ```php
      $a = array(2, 4, 6, 8);
      var_dump(array_product($a));// 384
      var_dump(array_product([])); // 1
     ```
   - array_reduce ( array $array , callable $callback [, mixed $initial = NULL ] ) : mixed
     ```php
     function sum ($carry, $item) {
        $carry += $item;
        return $carry;
      }
      $a = [1, 2, 3, 4, 5];
      var_dump(array_reduce($a, 'sum')); // 15
      var_dump(array_reduce($a, 'sum', 10)); // 25
     ```
   - array_sum ( array $array ) : number — 对数组中所有值求和
     ```php
     var_dump(array_sum([1, 2, 3])); // 6
     var_dump(array_sum([])); // 0
     ```
5. **数组键值操作** 
   - array_change_key_case ( array $array [, int $case = CASE_LOWER ] ) : array  — 将数组中的所有键名修改为全大写或小写
     ```php
     $a = ['a' => 1, 'B' => 2, 3 => 'c', 'A' => 4];
     print_r(array_change_key_case($a)); // ['a' => 4, 'b' => 2, 3 => 'c']
     print_r(array_change_key_case($a, CASE_UPPER)); // ['A' => 4, 'B' => 2, 3 => 'c']
     ```
   - array_key_exists — 检查数组里是否有指定的键名或索引
     ```php
     var_dump(array_key_exists('first', ['first' => 1, 'second' => 2])); // true
     ```
   - array_key_first — 获取数组第一个键值
     ```php
     // php >= 7.3.0
     var_dump(array_key_first(['a' => 1, 'b' => 2])); // 'a'
     ```
   - array_key_last — 获取数组最后一个键值
     ```php
      // php >= 7.3.0
      var_dump(array_key_last(['a' => 1, 'b' => 2])); // 'b'
     ```
   - array_keys ( array $array , mixed $search_value [, bool $strict = FALSE ] ) : array | array_keys ( array $array ) : array — 返回数组中部分的或所有的键名
      ```php
      $a = ['a' => 1, 'b' => 2, 'c' => 3];
      var_dump(array_keys($a)); // ['a', 'b', 'c']
      var_dump(array_keys($a, '1')); // 'a'
      var_dump(array_keys($a, '1', true)); // []
      ```
   - key_exists — 别名 array_key_exists
6. **数组元素操作**
   - array_pad ( array $array , int $size , mixed $value ) : array — 以指定长度将一个值填充进数组
     ```php
      $input = [12, 10, 9];
      var_dump(array_pad($input, 5, 2)); // [12, 10, 9, 2, 2]
      var_dump(array_pad($input, -5, 2)); // [2, 2, 12, 10, 9]
      var_dump(array_pad($input, 4, [1])); // [12, 10, 9, [1]]
     ```
   - array_rand — 从数组中随机取出一个或多个单元
     ```php
     var_dump(array_rand([1, 2, 3], 2)); // [1, 3]
     ```
   - array_search ( mixed $needle , array $haystack [, bool $strict = FALSE ] ) : mixed — 在数组中搜索给定的值，如果成功则返回首个相应的键名
     ```php
      $a = [1, 'blue', 'red', 'php', 0];
      var_dump(array_search('php', $a)); // 2
      var_dump(array_search(false, $a)); // 3
      var_dump(array_search(false, $a, true)); // false
     ```
   - array_values ( array $array ) : array — 返回数组中所有的值
     ```php
     $a = ['a' => 1, 'b' => 2];
     var_dump(array_value($a)); // [1, 2]
     ```
   - in_array ( mixed $needle , array $haystack [, bool $strict = FALSE ] ) : bool 检查数组中是否存在某个值
     ```php
     // 字符串比较是大小写敏感
     $os = ['Mac', 'NT', 'Irix', 'Linux', 1];
     var_dump(in_array('Linux', $os)); // true
     var_dump(in_array('1', $os)); //true
     var_dump(in_array('1a', $os)); //true
     // 开启严格类型比较
     var_dump(in_array('1', $os, true)); // false
     ```
7. **数组交集与差集, 合并**
   - array_diff_assoc ( array $array1 , array $array2 [, array $... ] ) : array — 带索引检查计算数组的差集
     ```php
     $array1 = ["a" => "green", "b" => "brown", "c" => "blue", "red"];
     $array2 = ["a" => "green", "yellow", "red"];
     print_r(array_diff_assoc($array1, $array2)); // ["b" => "brown", "c" => "blue", 0 => "red"]
     ```
   - array_diff_key — 使用键名比较计算数组的差集
     ```php
     $array1 = ['blue' => 1, 'red' => 2]; 
     $array2 = ['green' => 2, 'blue' => 1];
     print_r(array_diff_key($array1, $array2)); // ['red' => 2];
     ```
   - array_diff_uassoc ( array $array1 , array $array2 [, array $... ], callable $key_compare_func ) : array — 用用户提供的回调函数做索引检查来计算数组的差集
     ```php
     function key_compare_func($a, $b)
     {
         if ($a === $b) {
             return 0;
         }
        return ($a > $b)? 1:-1;
     }
     $array1 = array("a" => "green", "b" => "brown", "c" => "blue", "red");
     $array2 = array("a" => "green", "yellow", "red");
     $result = array_diff_uassoc($array1, $array2, "key_compare_func");
     print_r($result); // ["b" => "brown", "c" => ""]
     ```
   - array_diff_ukey ( array $array1 , array $array2 [, array $... ], callable $key_compare_func ) : array — 用回调函数对键名比较计算数组的差集
     ```php
     function key_compare_func($key1, $key2)
     {
         if ($key1 == $key2) {
             return 0;
         } else if ($key1 > $key2) {
             return 1;
         } else {
             return -1;
         }
     }
     $array1 = ['blue' => 1, 'red' => 2, 'green' => 3];
     $array2 = ['green' => 5, 'blue' => 6, 'yellow' => 8];
     var_dump(array_diff_ukey($array1, $array2, 'key_compare_func')); // ['red' => 2]
     ```
   - array_diff ( array $array1 , array $array2 [, array $... ] ) : array — 计算数组的差集
     ```php
      $array1 = ['red', 'blue', 'green'];
      $array2 = ['red', 'yellow', 'hello'];
      $array3 = ['green', 'world'];
      var_dump(array_diff($array1, $array2, $array3)); // ['blue']
     ```
   - array_intersect_assoc ( array $array1 , array $array2 [, array $... ] ) : array — 带索引检查计算数组的交集
     ```php
      $array1 = array("a" => "green", "b" => "brown", "c" => "blue", "red");
      $array2 = array("a" => "green", "b" => "yellow", "blue", "red");
      $result_array = array_intersect_assoc($array1, $array2);
      print_r($result_array); // ['a' => 'green']
     ```
   - array_intersect_key ( array $array1 , array $array2 [, array $... ] ) : array — 使用键名比较计算数组的交集
     ```php
      $array1 = array('blue'  => 1, 'red'  => 2, 'green'  => 3, 'purple' => 4);
      $array2 = array('green' => 5, 'blue' => 6, 'yellow' => 7, 'cyan'   => 8);
      var_dump(array_intersect_key($array1, $array2)); // ['blue' => 1, 'green'=>5]
     ```
   - array_intersect_uassoc — 带索引检查计算数组的交集，用回调函数比较索引
     ```php
      $array1 = ["a" => "green", "b" => "brown", "c" => "blue", "red"];
      $array2 = ["a" => "GREEN", "B" => "brown", "yellow", "red"];
      var_dump(array_intersect_uassoc($array1, $array2, 'strcasecmp')); // ['b' => 'brown']
     ```
   - array_intersect_ukey ( array $array1 , array $array2 [, array $... ], callable $key_compare_func ) : array — 用回调函数比较键名来计算数组的交集
     ```php
      $array1 = ["a" => "green", "b" => "brown", "c" => "blue", "red"];
      $array2 = ["a" => "GREEN", "B" => "brown", "yellow", "red"];
      var_dump(array_intersect_ukey($array1, $array2, 'strcasecmp')); // ['a' => 'green', 'b' => 'brown', 'red']
     ```
   - array_intersect ( array $array1 , array $array2 [, array $... ] ) : array — 计算数组的交集
     ```php
      $a = ['a' => 'green', 'red', 'blue'];
      $b = ['b' => 'green', 'yellow', 'red'];
      var_dump(array_intersect($a, $b)); // ['a' => 'green', 'red']
     ```
   - array_merge_recursive ([ array $... ] ) : array — 递归地合并一个或多个数组
     ```php
      $arr1 = ['color' => ['favorite' => 'red', 5]];
      $arr2 = ['color' => ['favorite' => 'green', 6, 7]];
      var_dump(array_merge_recursive($arr1, $arr2)); // ['color' => ['favorite' => ['red', 'green'], 5, 6, 7]]
     ```
   - array_merge ( array $array1 [, array $... ] ) : array — 合并一个或多个数组
     ```php
      $arr1 = ['color' => ['favorite' => 'red', 5]];
      $arr2 = ['color' => ['favorite' => 'green', 6, 7]];
      var_dump(array_merge($arr1, $arr2)); // ['color' => ['favorite' => 'green', 6, 7]];
      var_dump($arr1 + $arr2); // ['color' => ['favorite' => 'red', 5]]
     ```
   - array_udiff_assoc ( array $array1 , array $array2 [, array $... ], callable $value_compare_func ) : array — 带索引检查计算数组的差集，用回调函数比较数据
     ```php
     // https://segmentfault.com/q/1010000000364758/a-1020000000364791 array_udiff_assoc 和 array_diff_uassoc 区别
     /**
      * https://www.reddit.com/r/PHP/comments/1ncqc0/real_difference_between_array_udiff_assoc_and/
      */
     $item1 = [ 1 => "a", "b", "c"];
     $item2 = [-1 => 'a', -2 => 'b', -3 => 'c'];
     $item3 = [1 => ['a'], ['b'], ['c']];
     function compareKey($a, $b) {
          if(abs($a) == abs($b)) return 0;
          return (abs($a) > abs($b)) ? 1 : -1;
     }
     function compareValue($a, $b) {
         $tempA = is_array($a) ? $a[0] : $a;
       $tempB = is_array($b) ? $b[0] : $b;
       if($tempA == $tempB) return 0;
       return ($tempA > $tempB) ? 1 : -1;
     }
     print_r(array_diff_uassoc($item1, $item2, 'compareKey')); // []
     print_r(array_udiff_assoc($item1, $item2, 'compareKey')); // ['a', 'b', 'c']
     print_r(array_diff_uassoc($item1, $item3, 'compareValue')); // ['a', 'b', 'c']
     print_r(array_udiff_assoc($item1, $item3, 'compareValue')); // []
     ```
   - array_udiff_uassoc ( array $array1 , array $array2 [, array $... ], callable $value_compare_func , callable $key_compare_func ) : array — 带索引检查计算数组的差集，用回调函数比较数据和索引
     ```php
     $a = ['0.1' => 9, '0.5' => 12, 0 => 23, 1 => 4, 2 => -15];
     $b = ['0.2' => 9, '0.5' => 22, 0 => 3, 1 => 4, 2 => -15];
     function compareKey($a, $b) {
         if ($a === $b) return 0;
         return $a > $b ? 1 : -1;
     }
     function compareValue($a, $b) {
         if ($a === $b) return 0;
         return $a > $b ? 1 : -1;
     }
     print_r(array_udiff_uassoc($a, $b, 'compareValue', 'compareKey')); // [0.1 => 9, 0.5 => 12, 0 => 23]
     ```
   - array_udiff ( array $array1 , array $array2 [, array $... ], callable $value_compare_func ) : array — 用回调函数比较数据来计算数组的差集
     ```php
     $a = [[1, 2], [7, 1], [2, 9]];
     $b = [[1, 3], [8, 1], [2, 9]];
     function compare($a, $b) {
         $areaA = $a[0] * $a[1];
         $areaB = $b[0] * $b[1];
         if ($areaA == $areaB) return 0;
         return $areaA > $areaB ? 1 : -1;
     }
     print_r(array_udiff($a, $b, 'compare')); // [[1, 2], [7, 1]]
     ```
   - array_uintersect_assoc ( array $array1 , array $array2 [, array $... ], callable $value_compare_func ) : array — 带索引检查计算数组的交集，用回调函数比较数据
     ```php
     $a = ['a' => 'green', 'b' => 'brown', 'c' => 'blue', 'red'];
     $b = ['a' => 'GREEN', 'B' => 'brown', 'yellow', 'red'];
     print_r(array_uintersect_assoc($a, $b, 'strcasecmp')); // ['a' => 'green']
     ```
   - array_uintersect_uassoc ( array $array1 , array $array2 [, array $... ], callable $value_compare_func , callable $key_compare_func ) : array — 带索引检查计算数组的交集，用单独的回调函数比较数据和索引
     ```php
     $array1 = array("a" => "green", "b" => "brown", "c" => "blue", "red");
     $array2 = array("a" => "GREEN", "B" => "brown", "yellow", "red");
     print_r(array_uintersect_uassoc($array1, $array2, "strcasecmp", "strcasecmp")); // ['a' => 'green', 'b' => 'brown']
     ```
   - array_uintersect ( array $array1 , array $array2 [, array $... ], callable $value_compare_func ) : array — 计算数组的交集，用回调函数比较数据
     ```php
     $a = [[1, 2], [7, 1], [2, 9]];
     $b = [[1, 3], [8, 1], [2, 9]];
     function compare($a, $b) {
         $areaA = $a[0] * $a[1];
         $areaB = $b[0] * $b[1];
         if ($areaA == $areaB) return 0;
         return $areaA > $areaB ? 1 : -1;
     }
     print_r(array_uintersect($a, $b, 'compare')); // [[2, 9]]
     ```
8. **数组排序**
   - array_multisort ( array &$array1 [, mixed $array1_sort_order = SORT_ASC [, mixed $array1_sort_flags = SORT_REGULAR [, mixed $... ]]] ) : bool — 对多个数组或多维数组进行排序
     ```php
      // 更多用法请看 https://www.php.net/manual/en/function.array-multisort.php
      $ar1 = [10, 100, 100, 0];
      $ar2 = [1, 3, 2, 4];
      array_multisort($ar1, $ar2); // [0, 10, 100, 100] [4, 1, 2, 3]
      array_multisort($ar1, SORT_DESC, SORT_NUMERIC, $ar2, SORT_ASC, SORT_NUMERIC); // [100, 100, 10, 0] [2, 3, 1, 4]
     ```
   - arsort ( array &$array [, int $sort_flags = SORT_REGULAR ] ) : bool — 对数组进行逆向排序并保持索引关系
     ```php
     // 排序针对的是值, 而且会保存原先索引
     $fruits = array("d" => "lemon", "a" => "orange", "b" => "banana", "c" => "apple");
     arsort($fruits);
     print_r($fruits); // ['a' => 'orange', 'd' => 'lemon', 'b' => 'banana', 'c' => 'apple'];
     ```
   - asort ( array &$array [, int $sort_flags = SORT_REGULAR ] ) : bool — 对数组进行排序并保持索引关系
     ```php
     $fruits = array("d" => "lemon", "a" => "orange", "b" => "banana", "c" => "apple");
     asort($fruits);
     print_r($fruits); // ['c' => 'apple', 'b' => 'banana', 'd' => 'lemon', 'a' => 'orange'];
     ```
   - krsort ( array &$array [, int $sort_flags = SORT_REGULAR ] ) : bool — 对数组按照键名逆向排序
     ```php
     $fruits = ['d' => 'lemon', 'a' => 'orange', 'b' => 'banana', 'c' => 'apple'];
     krsort($fruits);//true
     var_dump($fruits); // ['d' => 'lemon', 'c' => 'apple', 'b' => 'banana', 'a' => 'orange']
     ```
   - ksort ( array &$array [, int $sort_flags = SORT_REGULAR ] ) : bool — 对数组按照键名排序
     ```php
     $fruits = ['d' => 'lemon', 'a' => 'orange', 'b' => 'banana', 'c' => 'apple'];
     ksort($fruits);//true
     var_dump($fruits); // ['a' => 'orange', 'b' => 'banana', 'c' => 'apple', 'd' => 'lemon']
     ```
   - rsort ( array &$array [, int $sort_flags = SORT_REGULAR ] ) : bool — 对数组逆向排序
     ```php
     $fruits = array("lemon", "orange", "banana", "apple");
     rsort($fruits); // true
     var_dump($fruits); // ['orange', 'lemon', 'banana', 'apple']
     ```
   - sort ( array &$array [, int $sort_flags = SORT_REGULAR ] ) : bool — 对数组排序
     ```php
     // https://bettercuicui.github.io/2018/04/01/PHP/php%20sort%E6%8E%92%E5%BA%8F%E5%AE%9E%E7%8E%B0%E5%8E%9F%E7%90%86/
     $fruits = array("lemon", "orange", "banana", "apple", 1);
     sort($fruits);
     var_dump($fruits); // ['apple', 'banana', 'lemon', 'orange', 1]
     // 数字比较项目, 'a1' 转换数字为 0, 所以排序顺序如下
     $arr1 = ['a1', 0, 2, '1a'];
     sort($arr1, SORT_NUMERIC);
     var_dump($arr1);  // ['a1', 0, '1a', 2]
     // 被作为字符串来比较
     $fruits = array("lemon", "orange", "banana", "apple", 1);
     sort($fruits, SORT_STRING);
     var_dump($fruits); // [1, 'apple', 'banana', 'lemon', 'orange']
     // 根据当前的区域（locale）设置来把单元当作字符串比较，可以用 setlocale() 来改变
     $fruits = array("lemon", "orange", "banana", "apple", 1);
     sort($fruits, SORT_LOCALE_STRING);
     var_dump($fruits); // [1, 'apple', 'banana', 'lemon', 'orange']
     //  和 natsort() 类似对每个单元以“自然的顺序”对字符串进行排序
     $arr = ['a1', 'A1', 'c2', 'c3', 'b4', 'b5', 'B1'];
     sort($arr, SORT_NATURAL);  // true
     var_dump($arr); // ['A1', 'B1', 'a1', 'b4', 'b5', 'c2', 'c3']
     // 能够与 SORT_STRING 或 SORT_NATURAL 合并（OR 位运算），不区分大小写排序字符串。
     $fruits = array("lemon", "orange", "banana", "apple", "Apple", 1);
     sort($fruits, SORT_FLAG_CASE|SORT_STRING);
     var_dump($fruits); // [1, "apple", "Apple", "banana", "lemon", "orange"]
     ```
   - uasort ( array &$array , callable $value_compare_func ) : bool — 使用用户自定义的比较函数对数组中的值进行排序并保持索引关联
     ```php
     // 用户自定义值比较函数, 保持索引不变
     function compare($a, $b) {
         if ($a === $b) {
             return 0;
         }
         return ($a < $b)  ? -1 : 1;
     }
     $arr = ['a' => 3, 'b' => 9, 'c' => 4, 'd' => 8, 'e' => -1];
     uasort($arr, 'compare'); // true
     var_dump($arr); // ['e' => -1, 'a' => 3, 'c' => 4, 'd' => 8, 'b' => 9]
     ```
   - uksort ( array &$array , callable $key_compare_func ) : bool — 使用用户自定义的比较函数对数组中的键名进行排序
     ```php
     // 用户自定义值比较函数, 保持索引不变
     function keyCompare($a, $b) {
         if ($a === $b) {
             return 0;
         }
         return ($a > $b)  ? -1 : 1;
     }
     $arr = ['a' => 3, 'b' => 9, 'c' => 4, 'e' => 8, 'd' => -1];
     uksort($arr, 'compare'); // true
     var_dump($arr); // ['a' => 3, 'b' => 9, 'c' => 4, 'd' => -1, 'e' => 8]
     ```
   - usort — 使用用户自定义的比较函数对数组中的值进行排序
     ```php
     // 对多维数组排序时，$ a和$ b包含对数组第一个索引的引用。
     function cmp($a, $b)
     {
        return strcmp($a["fruit"], $b["fruit"]);
     }
     $fruits = [['fruit' => 'lemons'], ['fruit' => 'apples'], ['fruit' => 'grapes']];
     usort($fruits, "cmp");
     var_dump($arr); // [['fruit' => 'apples'], ['fruit' => 'grapes'], ['fruit' => 'lemons']]
     ```
    - natcasesort ( array &$array ) : bool — 用“自然排序”算法对数组进行不区分大小写字母的排序
      ```php
      // 自然排序, 以人们认知方式排序,不区分大小写
      $arr = ['a1', 'A1', 'c2', 'c3', 'b4', 'b5', 'B1'];
      natcasesort($arr);  // true
      var_dump($arr); // ['a1', 'A1', 'B1', 'b4', 'b5', 'c2', 'c3']
      ```
    - natsort ( array &$array ) : bool — 用“自然排序”算法对数组排序
      ```php
      // 自然排序, 以人们认知方式排序,区分大小写
      $arr = ['a1', 'A1', 'c2', 'c3', 'b4', 'b5', 'B1'];
      natsort($arr);  // true
      var_dump($arr); // ['A1', 'B1', 'a1', 'b4', 'b5', 'c2', 'c3']
      ```
9. **回调函数处理数组**
    - array_filter ( array $array [, callable $callback [, int $flag = 0 ]] ) : array — 用回调函数过滤数组中的单元
      ```php
      // 若没有回调函数, 则筛选值为false
      $a = [1, 0, 'false', 'null', '0', 2, '', false];
      var_dump(array_filter($a)); // [1, "false", "null", 2]
      $b = [6, 7, 8, 9, 10, 11, 12];
      function odd($var)
      {
          return $var & 1;
      }
      var_dump(array_filter($b, 'odd')); // [7, 9, 11]
      ```
    - array_map ( callable $callback , array $array1 [, array $... ] ) : array — 为数组的每个元素应用回调函数
      ```php
      print_r(array_map('strtoupper', ['a', 'b', 'c'])); // ['A', 'B', 'C']
      function demo($n, $m) {
          return strtoupper($n) . '-' . strtoupper($m);
      }
      print_r('demo', ['a', 'b', 'c'], ['d', 'e', 'f']); // ['A-D', 'B-E', 'C-F']
      ```
    - array_walk_recursive — 对数组中的每个成员递归地应用用户函数
      ```php
      $sweet = ['a' => 'apple', 'b' => 'banana'];
      $fruits = ['sweet' => $sweet, 'sour' => 'lemon'];
      function testPrint($item, $key) {
          echo "$key holds $item\n";
      }
      /**
       * a holds apple
       * b holds banana
       * sour holds lemon
       */
      array_walk_recursive($fruits, 'testPrint');
      function testPrint2($item, $key, $userData) {
          $item = strtoupper($item);
          echo "$key holds $item and $userData\n";
      }
      /**
       * a holds apple and php
       * b holds banana and php
       * sour holds lemon and php
       */
      array_walk_recursive($fruits, 'testPrint2', 'php');
      ``` 
    - array_walk ( array &$array , callable $callback [, mixed $userdata = NULL ] ) : bool — 使用用户自定义函数对数组中的每个元素做回调处理
      ```php
      $sweet = ['a' => 'apple', 'b' => 'banana'];
      function testPrint($item, $key) {
          echo "$key holds $item\n";
      }
      /**
       * a holds apple
       * b holds banana
       */
      array_walk($sweet, 'testPrint');
      function testPrint($item, $key, $userData) {
          echo "$key holds $item and $userData\n";
      }
      /**
       * a holds apple and php
       * b holds banana and php
       * sour holds lemon and php
       */
      array_walk_recursive($fruits, 'testPrint2', 'php');
      ```
10. **数组替换**
   - array_replace_recursive ( array $array1 [, array $... ] ) : array — 使用传递的数组递归替换第一个数组的元素
     ```php
      $base = ['citrus' => ['orange'], 'berries' => ['blackberry', 'raspberry']];
      $replacements = ['citrus' => ['pineapple'], 'berries' => ['blueberry']];
      var_dump(array_replace_recursive($base, $replacements));// ['citrus' => ['pineapple'], 'berries' => ['blueberry', 'raspberry']]
     ```
   - array_replace ( array $array1 [, array $... ] ) : array — 使用传递的数组替换第一个数组的元素
     ```php
      $base = ['citrus' => ['orange'], 'berries' => ['blackberry', 'raspberry']];
      $replacements = ['citrus' => ['pineapple'], 'berries' => ['blueberry']];
      var_dump(array_replace($base, $replacements));// ['citrus' => ['pineapple'], 'berries' => ['blueberry']]
     ```
   - array_splice ( array & $input , int $offset [, int $length = count($input) [, mixed $replacement = array() ]] ) : array — 去掉数组中的某一部分并用其它值取代
     ```php
      $a = ['a', 'b', 'c'];
      array_splice($a, 1, 2, ['B', 'C']);
      var_dump($a); // ['a', 'B', 'C']
      array_splice($a, -2, 2, ['b', 'c']); 
      var_dump($a);// ['a', 'b', 'c']
      array_splice($a, 3, 0, ['d', 'e']);
      var_dump($a); // ['a', 'b', 'c', 'd', 'e']
     ```
11. **数组指针操作**
    - current ( array $array ) : mixed — 返回数组中的当前单元
      ```php
      $a = [1, 2, 3];
      var_dump(current($a)); // 1
      ```
    - end ( array &$array ) : mixed — 将数组的内部指针指向最后一个单元
      ```php
      $a = [1, 2, 3];
      var_dump(end($a)); // 3
      ```
    - key ( array $array ) : mixed — 从关联数组中取得键名
      ```php
      $arr = ['color' => 'red'];
      var_dump(key($arr)); // color
      ```
    - next ( array &$array ) : mixed — 将数组中的内部指针向前移动一位
      ```php
      $arr = [1, 2, 3];
      var_dump(next($arr)); // 2
      ```
    - pos — current 的别名
    - prev ( array &$array ) : mixed — 将数组的内部指针倒回一位
      ```php
      $arr = [1, 2, 3];
      next($arr); // 2
      var_dump(prev($arr)); // 1
      ```
    - reset ( array &$array ) : mixed — 将数组的内部指针指向第一个单元
      ```php
      $arr = [1, 2, 3];
      next($arr); // 2
      next($arr); // 3
      var_dump(reset($arr)); // 1
      ```
12. **其他**
    - array_column ( array $input , mixed $column_key [, mixed $index_key = NULL ] ) : array — 返回数组中指定的一列
      ```php
      $a = [['id' => 1, 'first_name' => 'John', 'last_name' => 'Doe'], ['id' => 2, 'first_name' => 'Wythe', 'last_name' => 'Huang']];
      print_r(array_column($a, 'first_name')); // ['John', 'Wythe']
      print_r(array_column($a, 'first_name', 'id')); // [1=> 'John', 2 => 'Wythe']
      ```
    - array_flip ( array $array ) : array — 交换数组中的键和值
      ```php
      $a = ['a' => 1, 'b' => 2, 'c' => 3];
      var_dump(array_filp($a)); // [1 => 'a', 2 => 'b', 3 => 'c']
      ```
    - array_reverse ( array $array [, bool $preserve_keys = FALSE ] ) : array  — 返回单元顺序相反的数组
      ```php
      $a = ['php', 4.0, ['green', 'red']];
      var_dump(array_reverse($a)); // [['green', 'red'], 7.1, 'php']
      var_dump(array_reverse($a, true)); // [2 => ['green', 'red'], 1 => 7.1, 0 => 'php']
      ```
    - array_unique ( array $array [, int $sort_flags = SORT_STRING ] ) : array — 移除数组中重复的值
      ```php
      $a = ['4', 1, 4, '1', 2];
      var_dump(array_unique($a)); // [0 => '4', 1 => 1, 4 => 2]
      var_dump(array_unique($a, SORT_NUMERIC));// [0 => '4', 1 => 1, 4 => 2]
      ```
    - count ( mixed $array_or_countable [, int $mode = COUNT_NORMAL ] ) : int — 计算数组中的单元数目，或对象中的属性个数
      ```php
      $a = [1, 2, 3];
      echo count($a); // 3
      $b = [$a, 4, 5, 6];
      echo count($b, COUNT_RECURSIVE) // 7
      ```
    - extract ( array &$array [, int $flags = EXTR_OVERWRITE [, string $prefix = NULL ]] ) : int — 从数组中将变量导入到当前的符号表
      ```php
      $size = 1;
      $varArray = ['color' => 'blue', 'size' => 2, 'shape' => 'sphere'];
         
      var_dump(extract($varArray)); // 3 若存在, 则覆盖
      echo "$color, $size, $shape\n"; // blue, 2, sphere
         
      extract($varArray, EXTR_SKIP); // 若存在, 则不覆盖
      echo "$color, $size, $shape\n"; // blue, 1, sphere
         
      extract($varArray, EXTR_PREFIX_SAME, 'prefix'); // 若存在, 则在变量名使用前缀
      echo "$color, $size, $shape, $prefix_size\n"; // blue, 1, sphere, 2
         
      extract($varArray, EXTR_PREFIX_ALL, 'prefix'); // 若存在, 则在变量名使用前缀
      echo "$prefix_color, $size, $prefix_shape, $prefix_size\n"; // blue, 1, sphere, 2
         
      var_dump(extract($varArray, EXTR_IF_EXISTS)); // 1 若存在, 则只覆盖已有变量
      /**
       * Notice: Undefined variable: color
       * Notice: Undefined variable: shape
       */
      echo "$color, $size, $shape\n"; // , 2, 
         
      extract($varArray, EXTR_PREFIX_IF_EXISTS, 'prefix'); // 仅当当前符号表中存在相同变量的非前缀版本时，才创建带前缀的变量名称。
      /**
       * Notice: Undefined variable: color
       * Notice: Undefined variable: shape
       */
      echo "$color, $size, $shape, $prefix_size\n"; // , 1, , 2
         
      var_dump(extract($varArray, EXTR_REFS)); // 3 变量作为引用。这实际上意味着导入的变量的值仍在引用数组参数的值, 可以与其他参数结合
      echo "$color, $size, $shape\n"; // blue, 2, sphere
      $color = 'red';
      var_dump($varArray); // ['color' => 'red', 'size' => 2, 'shape' => 'sphere'];
         
      // 变量作为引用。这实际上意味着导入的变量的值仍在引用数组参数的值, 可以与其他参数结合, 若解析变量前缀的, 则修改前缀变量, 数组变量则修改.
      $size = 1;
      $varArray = ['color' => 'blue', 'size' => 2, 'shape' => 'sphere'];
      var_dump(extract($varArray, EXTR_REFS|EXTR_PREFIX_SAME, 'prefix')); // 3
      echo "$color, $size, $shape, $prefix_size\n"; // blue, 1, sphere, 2
      $color = 'red';
      $prefix_size = 4;
      var_dump($size); // 1
      var_dump($varArray); // ['color' => 'red', 'size' => 4, 'shape' => 'sphere'];
         
      $a = [1, 2, 3];
      extract($a, EXTR_PREFIX_INVALID, 'prefix');
      echo "$prefix_0, $prefix_1, $prefix_2\n"; // 1, 2, 3
      ```
    - list ( mixed $var1 [, mixed $... ] ) : array — 把数组中的值赋给一组变量
      ```php
      // 语言结构, 并非函数
      $info = ['coffee', 'brown', 'caffeine'];
      list($drink, $color, $power) = $info;
      echo "$drink, $color, $power"; // coffee, brown, caffeine
      // 可以省略不想取的
      list($drink, , $power) = $info;
      echo "$drink, $power"; // coffee, caffeine
      // 7.0 以上是从左到右赋值, 7.0以下是从右到左赋值
      list($a[0], $a[1], $a[2]) = $info;
      // 7.0 以上
      var_dump($a); // ['coffee', 'brown', 'caffeine']
      // 7.0 以下
      var_dump($a); // ['caffeine', 'brown', 'coffee']
      // 7.1 支持索引数组
      $info = ['a' => 'coffee', 'b' => 'brown', 'c' => 'caffeine'];
      list('a' => $drink, 'c' => $power) = $info;
      echo "$drink, $power"; // coffee, caffeine
      ```
    - shuffle ( array &$array ) : bool — 打乱数组
      ```php
      $arr = [1, 2, 3, 4];
      shuffle($arr); // 洗牌算法, 打乱数组
      ```
    - sizeof 等同于 count  