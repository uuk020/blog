## 浅析PHP闭包(下)

PHP中, 闭包函数(匿名函数)是由匿名类实现的, 因此闭包是匿名类的对象实例. 查看PHP手册-[闭包](http://php.net/manual/en/class.closure.php), 匿名类有哪些方法
```PHP
Closure {
    /* Methods */
    private __construct ( void )
    public static bind ( Closure $closure , object $newthis [, mixed $newscope = "static" ] ) : Closure
    public bindTo ( object $newthis [, mixed $newscope = "static" ] ) : Closure
    public call ( object $newthis [, mixed $... ] ) : mixed
    public static fromCallable ( callable $callable ) : Closure
}
```

一一分析这些方法作用:

### private __construct
私有的构造方法, 说明匿名类无法实例化. 只能通过闭包函数(匿名函数)来实例化匿名类. 而不是new Closure()来创建闭包
```PHP
$closure = function () {
    echo 'this is closure';
};
```

## public call
public call ( object $newthis [, mixed $... ] )  **仅支持PHP 7**. 暂时将闭包绑定到newthis，并用任何给定的参数调用它。
```PHP
class Value
{
    protected $value;
	public function __construct($value)
	{
	    $this->value = $value;
	}
	public function getValue()
	{
        return $this->value;
	}
}
$three = new Value(3);
$closure = function($delta) {
    var_dump($this->getValue() + $delta);
};
$closure->call($three, 4); // 7
```
**newthis** 在调用期间将闭包绑定到的对象中. 因为闭包是对象, 当把闭包绑定到另一个类的对象, 在闭包函数中可以调用另一个类对象的方法和属性. 当然可以在闭包中实例化另一个对象, 这样做代码会增加, 对代码组织上有所繁琐.
```PHP
class Value
{
    protected $value;
	public function __construct($value)
	{
	    $this->value = $value;
	}
	public function getValue()
	{
        return $this->value;
	}
}
$closure = function($delta) {
    $three = new Value(3);
    var_dump($three->getValue() + $delta);
};
$closure(3); // 6
```
### Closure::fromCallable
public static Closure::fromCallable ( callable $callable ) : Closure 将可调用函数转换为闭包函数(匿名函数), 用途如下:
```PHP
$a = array_map(Closure::fromCallable('intval'), [2.2, 1.3]);
print_r($a); // [2, 1]
$b = array_map(Closure::fromCallable('strtoupper'), ['a', 'b']);
print_r($b); // ['A', 'B']
```
可以将PHP系统内函数转为闭包函数, 减少代码冗余. 这个方法支持PHP7.1以上

### Closure::bindTo与Closure::bind
Closure::bindTo与Closure::bind 没有什么区别. 简单来说,bind是bindTo静态方法版, 参数有所不同, 少了个匿名类的对象.
```PHP
public Closure::bindTo ( object $newthis [, mixed $newscope = 'static' ] ) : Closure
public static Closure::bind ( Closure $closure , object $newthis [, mixed $newscope = "static" ] ) : Closure
```
Closure::bind： 复制一个闭包，绑定指定的$this对象和类作用域。
Closure::bindTo： 复制当前闭包对象，绑定指定的$this对象和类作用域。
参数和返回值说明：
closure：表示需要绑定的闭包对象。
newthis：表示需要绑定到闭包对象的对象，或者NULL创建未绑定的闭包。
newscope：表示想要绑定给闭包的类作用域，或者 'static' 表示不改变。如果传入一个对象，则使用这个对象的类型名。 类作用域用来决定在闭包中 $this 对象的 私有、保护方法 的可见性。与闭包相关联的类范围，或保持当前的“static”。如果给定了对象，将使用对象的类型。这决定了绑定对象的受保护方法和私有方法的可见性
该方法成功时返回一个新的 Closure 对象，失败时返回FALSE。
```PHP
class Car
{
    private static $benz = 'BENZ';
	private $bmw = 'BMW';
	public $audi = 'AUDI';
	public static $mini = 'MINI';
}
$car = new Car();
// 访问公有属性
$public = function() {
    return 'this is the ' . $this->audi;
};
$bindBenz = $public->bindTo($car);
echo $bindBenz(); // this is the AUDI
// 访问私有属性
$private = function() {
   return 'this is the ' . $this->bmw;
};
$bindBmw = $private->bindTo($car);
// Fatal error: Uncaught Error: Cannot access private property Car::$bmw
echo $bindBmw();
// 访问公有静态属性
$publicStatic = function() {
   return 'this is the ' . self::$mini;
};
// Uncaught Error: Access to undeclared static property: Closure::$mini
$bindMini = $publicStatic->bindTo($publicStatic);
```
通过以上例子,可以说明bindTo只有一个参数newthis, 闭包函数体的 *$this* 只能访问绑定对象的公有属性和方法.而只有第二个参数会是怎么样?通过例子来说明
```PHP
class Car
{
    private static $benz = 'BENZ';
	private $bmw = 'BMW';
	public $audi = 'AUDI';
	public static $mini = 'MINI';
	private static $honda = 'HONDA';
}
$car = new Car();
// $this访问公有属性
$public = function() {
    return 'this is the ' . $this->audi;
};
$bindBenz = $public->bindTo(null, $car);
// Fatal error: Uncaught Error: Using $this when not in object context
echo $bindBenz();
// self访问公有静态属性
$publicStatic = function() {
    return 'this is the ' . self::$mini;
};
$bindMini = $publicStatic->bindTo(null, $car);
$bindMini(); // this is the MINI
// $this访问私有有属性
$private = function() {
    return 'this is the ' . $this->bmw;
};
$bindBmw = $private->bindTo(null, $car);
// Fatal error: Uncaught Error: Using $this when not in object context
echo $bindBmw();
// self访问私有静态属性
$privateStatic = function() {
    return 'this is the ' . self::$honda;
};
$bindHonda = $private->bindTo(null, $car);
// this is the honda
echo $bindHonda();
```
$bindHonda = $private->bindTo(null, $car) 与 $bindHonda = $private->bindTo(null, 'Car'); 没有什么区别. 只有第二参数时候, 决定匿名类的作用域, 也就是self, parent, static关键字访问绑定对象的类静态属性和静态方法(公有,受保护和私有).两个参数同时存在, $this可以访问到绑定对象的私有属性和私有方法. 也包含受保护的属性和方法.
```PHP
class Car
{
    private static $benz = 'BENZ';
	private $bmw = 'BMW';
	public $audi = 'AUDI';
	public static $mini = 'MINI';
	private static $honda = 'HONDA';
}
$car = new Car();
// 访问所有属性
$demo = function() {
    echo 'this is the public property ' . $this->bmw . "\n";
    echo 'this is the ' . self::$honda;
};
$test = $demo->bindTo($car, 'Car');
/**
 * this is the public property BMW
 * this is the HONDA
 */
echo $test();
```
以上代码都在PHP7.3.2测试, 这就是我对闭包理解.

[PHP手册-闭包](http://php.net/manual/en/class.closure.php)
[The new Closure::fromCallable() in PHP 7.1](https://josephsilber.com/posts/2016/07/13/closure-from-callable-in-php-7-1#table-of-contents)
[PHP闭包分析](https://learnku.com/articles/4625/php-closure-closure)