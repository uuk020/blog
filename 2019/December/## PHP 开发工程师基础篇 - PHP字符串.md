## PHP 开发工程师基础篇 - PHP字符串
### 字符串(String)
字符串是一系列字符的集合. 如 "abc". 在 PHP 中, 一个字符代表一个字节, 一个字节(Byte)有8比特(bit). PHP 仅支持 256 字符集, 因此 PHP 本身不支持 Unicode 字符集.
> This means that PHP only supports a 256-character set, and hence does not offer **native** Unicode support.

那么 PHP 是如何做到支持 Unicode 字符集的, 答案是在于 PHP 底层字符串实现中, PHP 的字符串是一个由字节组成的数组再加上一个整数指明缓冲区长度.并没有如何将字节转换字符的信息. 那么字符串如何编码和解码, 这些全部由使用者来决定编码方式. 

字符串会被按照该脚本文件相同的编码方式来编码。因此如果一个脚本的编码是 ISO-8859-1，则其中的字符串也会被编码为 ISO-8859-1，以此类推。不过这并不适用于激活了 Zend Multibyte 时；此时脚本可以是以任何方式编码的（明确指定或被自动检测）然后被转换为某种内部编码，然后字符串将被用此方式编码存入由字节组成的数组.

> The answer is that string will be encoded in whatever fashion it is encoded in the script file. Thus, if the script is written in ISO-8859-1, the string will be encoded in ISO-8859-1 and so on. However, this does not apply if Zend Multibyte is enabled; in that case, the script may be written in an arbitrary encoding (which is explicitly declared or is detected) and then converted to a certain internal encoding, which is then the encoding that will be used for the string literals.

通过例子说明一下, 比较易懂.
```php
// unicode 字符集下 utf-8 编码方式
$s = '严';
var_dump(strlen($s)); // 3

$len = strlen($s); // 3
/**
 * int(228) 对应二进制为 11100100
 * int(184) 对应二进制为 10111000
 * int(165) 对应二进制为 10100101
 */
for ($i = 0; $i < $len; $i++) {
    var_dump(ord($s[$i]));
}
```
"严"这个字怎么会是3个, 而不是一个呢, 由于 strlen 返回的是字节数, 而非字符数. PHP 底层字符串实现方式. 是由字节组成的数组的. 因此可以得出 PHP 对于"严"在 utf-8 编码方式的底层实现是由 3 个字节组成数组, 而这些字节不超过256的值, 这就是 PHP 编码方式. 而解码则是通过文件编码方式取对应字节数目来找对应字面字符.

![utf-8编码方式](https://ftp.bmp.ovh/imgs/2019/11/d7c05e365e7b566f.png)

而这些数值代表什么意思, 则是 utf-8 的编码方式了. 严的 Unicode 是4E25（100111000100101），根据上表，可以发现4E25处在第三行的范围内（0000 0800 - 0000 FFFF），因此严的 UTF-8 编码需要三个字节，即格式是1110xxxx 10xxxxxx 10xxxxxx。然后，从严的最后一个二进制位开始，依次从后向前填入格式中的x，多出的位补0。这样就得到了，严的 UTF-8 编码是11100100 10111000 10100101，转换成十六进制就是E4B8A5。

- [字符串以及为什么说 PHP 不支持 Unicode](https://blog.csdn.net/qq_26525289/article/details/95295913)
- [字符集与编码](https://juejin.im/post/5c847cb5f265da2dd63929c2)
- [字符编码笔记：ASCII，Unicode 和 UTF-8](http://www.ruanyifeng.com/blog/2007/10/ascii_unicode_and_utf-8.html)

### 非二进制安全
字符串由什么值来组成并无限制; 特别的, 其值为 0（“NUL bytes”）的字节可以处于字符串任何位置(不过有几个函数，在本手册中被称为非“二进制安全”的，也许会把 NUL 字节之后的数据全都忽略). 在C语言里字符串都以 \0 作为结束符, 而 PHP 底层是由 C 语言编写的, 因此某些函数遇到 \0 的 会忽略之后字节的, 从而影响程序逻辑和结果.
```php
$string1 = "Hello";
$string2 = "Hello\0World";
// 非二进制安全, \0 在 ascii 表是 NUL '\0' 
var_dump(strcoll($string1, $string2)); // 0
// 二进制安全 
var_dump(strcmp($string1, $string2)); // -6
```
- [为什么 php 手册里经常说某个函数是二进制安全的](https://segmentfault.com/a/1190000020318269)
- [In PHP what does it mean by a function being binary-safe?](https://stackoverflow.com/questions/3264514/in-php-what-does-it-mean-by-a-function-being-binary-safe)

### 语法
1. 单引号
2. 双引号
3. heredoc 语法结构
4. nowdoc 语法结构(自 PHP 5.3.0 起)

**单引号**

定义一个字符串的最简单的方法是用单引号把它包围起来(字符 '), 单引号无法解析变量和转义字符, 要表达单引号本身则在单引号前面加个反斜杠(\\), 要表达反斜杠(\\)则在反斜杠前加个反斜杠(\\)

```php
// this is string
echo 'this is string';
// I'm your father
echo 'I\'m your father';
// C:\*.*?
echo 'C:\\*.*?';
```

**双引号**

用双引号(")把字符包围起来, PHP 双引号会把特殊字符解释以下转义序列. 

![转义序列](https://ftp.bmp.ovh/imgs/2019/12/8dfea8b49a342985.jpg)

双引号还有一个特点,就是会解析变量.变量解析有两个解析语法, 一个是简单和一个复杂的.

- 简单解析语法:

  简单的解析方式当 PHP 解析器遇到一个美元符号（$）时，它会和其它很多解析器一样，去组合尽量多的标识以形成一个合法的变量名. 可用花括号限制解析范围.
  ```php
  $juice = 'apple';
  $juices = 'apple\s';
  // He drank some apple juice
  echo "He drank some $juice juice." . PHP_EOL;
  // He drank some apple\s juice
  echo "He drank some $juices juice" . PHP_EOL;
  // He drank some apples juice
  echo "He drank some {$juice}s juice" . PHP_EOL;
  ```
  类似的，一个 array 索引或一个 object 属性也可被解析。数组索引要用方括号（]）来表示索引结束的边际，对象属性则是和上述的变量规则相同。
- 复杂解析语法
  
  这并不因为语法复杂而被称为复杂，而是因为它允许使用复杂的表达式. 

  任何具有 string 表达的标量变量，数组单元或对象属性都可使用此语法。只需简单地像在 string 以外的地方那样写出表达式，然后用花括号 { 和 } 把它括起来即可。由于 { 无法被转义，只有 $ 紧挨着 { 时才会被识别。可以用 {\$ 来表达 {$。
  ```php
  echo "This works too: {$obj->values[3]->name}";
  echo "This works too: {$obj->values[3]->name}";
  echo "This is the value of the var named by the return value of getName(): {${getName()}}";
  ```
**Heredoc**

第三种表达字符串的方法是用 heredoc 句法结构：<<<。在该运算符之后要提供一个标识符，然后换行。接下来是字符串 string 本身，最后要用前面定义的标识符作为结束标志。

结束时所引用的标识符必须在该行的第一列，而且，标识符的命名也要像其它标签一样遵守 PHP 的规则：只能包含字母、数字和下划线，并且必须以字母和下划线作为开头。

Heredoc 结构就象是没有使用双引号的双引号字符串，这就是说在 heredoc 结构中单引号不用被转义，但是上文中列出的转义序列还可以使用
```php
$str = <<<EOD
Example of string
spanning multiple lines
using heredoc syntax.
EOD;
$juice = 'apple';
$str1 = <<<EOD
He drank some $juice juice.
EOD;
// 运用在函数中
function foo()
{
    static $bar = <<<LABEL
Nothing in here...
LABEL;
}
// 运用在类中
class foo
{
    const BAR = <<<FOOBAR
Constant example
FOOBAR;

    public $baz = <<<FOOBAR
Property example
FOOBAR;
}
```

**Nowdoc**

Nowdoc 结构很象 heredoc 结构，但是 nowdoc 中不进行解析操作。这种结构很适合用于嵌入 PHP 代码或其它大段文本而无需对其中的特殊字符进行转义

一个 nowdoc 结构也用和 heredocs 结构一样的标记 <<<， 但是跟在后面的标识符要用单引号括起来，即 <<<'EOT'。Heredoc 结构的所有规则也同样适用于 nowdoc 结构，尤其是结束标识符的规则。
```php
$str = <<<'EOD'
Example of string
spanning multiple lines
using nowdoc syntax.
EOD;
```
### 单引号比双引号快?
在 PHP 7 之前, 假如字符串没有存在变量替换, 单引号和双引号几乎一样的. 如果存在变量替换的时候是单引号比双引号要快. PHP 7 之后, 却是不同, 双引号与单引号几乎一样性能.
```php
$singleCode = 'this is single string';
$doubleCode = "this is single string";
echo $singleCode;
echo $doubleCode;
```
![PHP7.3.0 字符串分析](https://ftp.bmp.ovh/imgs/2019/12/3e78ae49ddeef6b6.jpg)

第0到第3条op line, 可以看出在没有使用变量替换的情况下，双引号的和单引号所产生的Opcodes是一样的.

```php
$var = 'String';
$single_quotes_var = 'This is a '.$var;
$double_quotes_var = "This is a $var";
 
echo $single_quotes_var;
echo $double_quotes_var;
```
![PHP7.3.0 字符串分析1](https://ftp.bmp.ovh/imgs/2019/12/d22e6d23db0d5d6d.jpg)
第1到2条与第4到5条, Opcodes条数是一样的. PHP 7 把之前INIT_STRING, ADD_STRING, ADD_VAR 的步骤优化成一条. 优化成FAST_CONCAT

- [风雪之隅-PHP的单引号和双引号](http://www.laruence.com/2008/08/19/338.html)
- [风雪之隅-深入理解PHP原理之Opcodes](http://www.laruence.com/2008/06/18/221.html)

### 存取和修改字符串中的字符

string 中的字符可以通过一个从 0 开始的下标，用类似 array 结构中的方括号包含对应的数字来访问和修改，比如 $str[42]。可以把 string 当成字符组成的 array. 也可以使用花括号来访问和修改 注意的是字符串操作最好针对单字节编码操作, 因为字符串在 PHP
底层是字节组成的数组, 用花括号/方括号操作多字节字符串很不安全. 

在PHP7.1.0, 还支持负字符串偏移量

用超出字符串长度的下标写入将会拉长该字符串并以空格填充。非整数类型下标会被转换成整数。
```php
$str = 'This is a test.';
$first = $str[0];
$third = $str{2};
$last = $str[-1];
echo $first . PHP_EOL; // T
echo $third . PHP_EOL; // i
echo $last . PHP_EOL; // .
$len = strlen($str);
$str[$len+1] = 'a';
echo $str . PHP_EOL; // This is a test. a
$str1 = "我";
echo ord($str1[0]); // 230
```