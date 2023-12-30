# Go 结构体

Go 通过类型别名（alias types）和结构体的形式支持用户自定义类型，或者叫定制类型.

一个带属性的结构体试图表示一个现实世界中的实体。结构体是复合类型（composite types），当需要定义一个类型，它由一系列属性组成，每个属性都有自己的类型和值的时候，就应该使用结构体，它把数据聚集在一起。然后可以访问这些数据，就好像它是一个独立实体的一部分。

结构体也是**值类型**，因此可以通过 new 函数来创建

组成结构体类型的那些数据称为 字段（fields）。每个字段都有一个类型和一个名字；在一个结构体中，字段名字必须是唯一的

结构体定义的一般方式:

```go
type identifier struct {
  field1 type1
  field2 type2
  ...
}

```

`type T struct {a, b int}` 也是合法的语法, 更适用于简单的结构体

结构体里的字段都有 名字，像 field1、field2 等，如果字段在代码中从来也不会被用到，那么可以命名它为 _

结构体的字段可以是任何类型，甚至是结构体本身，也可以是函数或者接口。可以声明结构体类型的一个变量

使用 new 函数给一个新的结构体变量分配内存，它返回指向已分配内存的指针：`var t *T = new(T)`，如果需要可以把这条语句放在不同的行

```go
package main
import "fmt"

type struct1 struct {
  i1  int
  f1  float32
  str string
}

func main() {
  ms := new(struct1)
  ms.i1 = 10
  ms.f1 = 15.5
  ms.str = "wythe"
  fmt.Println(ms) // &{10 15.5 Wythe}
}

```

就像在面向对象语言所作的那样，可以使用点号符给字段赋值：`structname.fieldname = value`。
同样的，使用点号符可以获取结构体字段的值：`structname.fieldname`。

混合字面量语法（composite literal syntax）`&struct1{a, b, c}` 是一种简写，底层仍然会调用 `new ()`，这里值的顺序必须按照字段顺序来写。

Go 语言中，结构体和它所包含的数据在内存中是以连续块的形式存在的，即使结构体中嵌套有其他的结构体，这在性能上带来了很大的优势。

```go
type Rect1 struct {Min, Max Point }
type Rect2 struct {Min, Max *Point }

```

![Untitled](Go%20%E7%BB%93%E6%9E%84%E4%BD%93%20012add16f8a74fdcaff748f613c40eff/Untitled.png)

Go 中的类型转换遵循严格的规则。当为结构体定义了一个 alias 类型时，此结构体类型和它的 alias 类型都有相同的底层类型

结构体中的字段除了有名字和类型外，还可以有一个可选的标签（tag）：它是一个附属于字段的字符串，可以是文档或其他的重要标记。标签的内容不可以在一般的编程中使用，只有包 reflect 能获取它

```go
package main

import(
  "fmt"
  "reflect"
)

type TagType struct {
  field1 bool   "An important answer"
  field2 string "The name of the thing"
  field3 int    "How much there are"
}

// An important answer
// The name of the thing
// How much there are
func main() {
   tt := TagType{true, "Barak Obama", 1}
   for i := 0; i < 3; i++ {
     refTag(tt, i)
   }
 }

 func refTag(tt TagType, ix int) {
   ttType := reflect.Typeof(tt)
   ixField := ttType.Field(tt)
   fmt.Printf("%v\n", ixField.Tag)
 }

```

结构体可以包含一个或多个 匿名（或内嵌）字段，即这些字段没有显式的名字，只有字段的类型是必须的，此时类型就是字段的名字。匿名字段本身可以是一个结构体类型，即 结构体可以包含内嵌结构体

可以粗略地将这个和面向对象语言中的继承概念相比较，它被用来模拟类似继承的行为。Go 语言中的继承是通过内嵌或组合来实现的，所以可以说，在 Go 语言中，相比较于继承，组合更受青睐。

```go
package main

import "fmt"

type innerS struct {
    in1 int
    in2 int
}

type outerS struct {
    b    int
    c    float32
    int  // anonymous field
    innerS //anonymous field
}

func main() {
    outer := new(outerS)
    outer.b = 6
    outer.c = 7.5
    outer.int = 60
    outer.in1 = 5
    outer.in2 = 10

    fmt.Printf("outer.b is: %d\n", outer.b)
    fmt.Printf("outer.c is: %f\n", outer.c)
    fmt.Printf("outer.int is: %d\n", outer.int)
    fmt.Printf("outer.in1 is: %d\n", outer.in1)
    fmt.Printf("outer.in2 is: %d\n", outer.in2)

    // 使用结构体字面量
    outer2 := outerS{6, 7.5, 60, innerS{5, 10}}
    fmt.Println("outer2 is:", outer2)
}

```

在一个结构体中对于每一种数据类型只能有一个匿名字段。

同样地结构体也是一种数据类型，所以它也可以作为一个匿名字段来使用. 内嵌结构体甚至可以来自其他包。内层结构体被简单的插入或者内嵌进外层结构体。这个简单的 “继承” 机制提供了一种方式，使得可以从另外一个或一些类型继承部分或全部实现。

当两个字段拥有相同的名字（可能是继承来的名字）时

1. 外层名字会覆盖内层名字（但是两者的内存空间都保留），这提供了一种重载字段或方法的方式；
2. 如果相同的名字在同一级别出现了两次，如果这个名字被程序使用了，将会引发一个错误（不使用没关系）。没有办法来解决这种问题引起的二义性，必须由程序员自己修正

```go
package main

import "fmt"

type A struct {a int}

type B struct {a, b int}

type C struct {A; B}

type D struct {B; b float32}

func main() {
  var c C
}

```