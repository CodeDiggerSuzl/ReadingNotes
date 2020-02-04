# 第 3 章: 面向对象编程 (OOP)
go 中没有沿用面向对象中的诸多概念, 比如✨继承, 虚函数, 构造函数, 析构函数, 隐藏的 `this` 指针.

优雅之处在于: 整个类型系统通过接口类型串联.
## 3.1 ✨类型系统

** 类型系统是一个语言的类型体系结构,✨是一门编程语言的地基 **, 典型的类型系统包括:
1. 基础类型,`byte,int,bool,float` 等
2. 符合类型, 数组, 结构体, 指针等
3. 可以指向任意对象的类型 (Any 类型)
4. 值语义和引用语义
5. 面向对象, 即所有面向对象特征 (比如成员方法) 的类型
6. 接口

类型系统就是描述是这些内容如何在一个语言中被关联.

☕️ 以 Java 为类型讲解类型系, 在 Java 中 2 种类型系统:

- 一种是值类型: 主要是基本类型 (byte,short,int,char...);
- ✨一种是以 `Object` 为根的对象类型, 使用 `new`.
  - Java 中的 Any 类型就是整个对象类型系统的根  -`java.lang.Object`
  - 只用对象类型系统的实例才能被 `Any` 类型引用.
  - 值类型要想被 `Any` 引用, 需要 `boxing`(装箱) 过程.
  - 只有对象类型系统中的类型才可以实现接口, 具体方法是: 让该类型从要实现的接口继承.

🐭 go 语言类型中
- 大多数系统都是值语义, 都可以包含对应的操作方法. 可以给任何类型 (包括内置系统 * 但是不包括指针类型 *)"添加" 新方法.
- 在实现某个接口的时候, 无需从该接口继承, 只需要实现该接口的所有方法.
- 任何类型都可以被 `Any` 类型引用. `Any` 类型就是空接口, 即 `interface{}`

### 3.1.1 为类型添加方法

为 `int` 类型添加 `Less` 方法
```go
func main() {
	var a Integer = 1
	if a.Less(2) {
		fmt.Println("a less 2")
	}
}

type Integer int

func (a Integer) Less(b Integer) bool {
	return a < b
}
```

在 C++ 中, 面向对象的继承和多态知识在 C 语言中的基础上添加的语法糖 (参见 < 深度探索 C++ 对象模型> 真本书)

面向对象的实现方式:
```go
func (a Integer) Less(b Integer) bool {
	return a < b
}
```
面向过程
```go
func Integer_Less(a Integer,b Integer) bool {
    return a < b
}
```

面向对象的用法: `a.Less(2)`

面向过程的用法: `Integer_Less(a,2)`


🦉面向对象的实现方式: ** 面向对象只是换了一种语法形式而已 **,C++ 之所以让人迷惑就是隐藏了 `this` 指针, Java 和 C# 中的 `this` 和 Python 中的 `self` 作用是完全一样的.

💥在 Go 语言中没有隐藏的 `this` 指针含义是:
1. 方法施加的目标 (也就是 "对象") 显示传递, 没有被隐藏起来.
2. 方法施加的目标 (也就是 "对象") 不需要非得是指针, 也不用非得叫 `this`

在 Java 中,
```Java
class Integer{
    paivate int val;
    public boolean Less(Integer b){
        return this.val < b.val;
    }
}
```
✨ 这个 `this` 是从哪里来的: 这个主要是因为 `Less() 方法隐藏了第一个参数 `Integer* this`.

### 3.1.2 值语义和引用语义
值语义和引用语义真的差别在赋值.

```go
b = a
b.Modify()
```
如果 `Modify` 方法没有修改 a 的值就是值传递, 否则就是引用语义

Go 语言中的大多是值语义:
1. 基本类型
2. 符合类型: `array`,`struct`(结构体),`pointer`

go 中有 4 个类型特殊
1. slice: 指向 array 区间

    数组切内部指向数组的指针, 但是赋值仍是值语义.
2. map

   map 本质是一个字典指针
3. channel: goroutine 直接的通信设施

    channel 和 map 类似, 本质上是一个指针. 将他们设计成引用类型而不是值类型的原因是: ** 完整赋值一个 channel 和 map 不是常规需求.**
4. ✨interface: 对一组满足某个契约的类型的抽象

### 3.1.3 结构体

struct 和其他语言中的 class 含有相同的地位, 但是只保留了 ** 组合 ** 这个最基本的特性.

## 3.2 初始化
初始化一个 struct: Rect.

1. 使用 `new`

   `var a := new(Rect)`
2. `&Rect{}`

    全部初始化要顺序初始化

没有初始化的属性: 会被设置成默认值

go 中没有构造函数: 对象的创建交由一个全局的创建函数来完成, 以 `NewXXX` 开头, 表示构造函数.

```go
func NewRect(len,width float){
    return &Rect{len,width}
}
```

## 3.3 匿名组合

匿名组合: 在 go 中使用组合的文法来实现继承, 称之为匿名组合.

基础 `struct`, 定义一个 Base 类, 实现了 `Foo()` 成员方法,
```go
type Base struct{
    Name string
}

// Foo 方法
func (a *Base) Foo(){

}
```

继承类继承 `Base`,
```go
type BaseExt struct{
    Base
    ...
}
func(e *BaseExt)Foo{
    e.Base.Foo()
}
```
`BaseExt` 没有改写 `Base` 成员方法, 这就实现了


另外: `BaseExt` 中 `Base` 位置显示了类的内存布局.`Base` 和其他成员放置的位置显示了内存布局.

"🌱派生": 和可以指针的方式从另一个类型 "派生":
```go
type Foo struct{
    *Base
    ...
}
```
这个同样可以实现 "派生" 的效果, 但是在创建 `Foo` 实例的时候, 需要 ** 提供一个 Base 类的指针 **.

下面的 struct 匿名组合了一个 `log.Logger` 指针:
```go
type Jop struct{
    Command string
    *log.Logger
}
```
在合适复制后, 在 Job 类型所有的成员方法都可以使用 log.Logger 提供的方法, 如:
```go
func (job *Job)Start(){
    job.Log("do sth")
}
```

对于 Job 的实现者, 根本就不知道 `log.Logger` 类型的存在. 这就是匿名组合的魅力.

🍄需要注意的是:
1. 无论怎么样的组合方法, 被组合类型所包含的方法虽然升级成了外部这个组合类型的方法, 但是其实他们被组合的方法调用时的接收者并没有改变.
2. 即使调用方法是 `job.Log()` 但是 `Log()` 函数的接受者还是 `log.Logger` 这个指针, 无法访问 `Job` 中的其他成员变量.(因为被组合的类型不知道自己会被什么类型组合, 也就无法访问那个未知的 "组合者" 的功能)

🌺 同名问题:
1. 组合类型和被组合类型都包含同样的名字的成员, 但是所有组合类型只会访问到最外边的成员, 内部成员会被隐藏起来
2. 当都为指针时

    ```go
    type Logger struct{
        Name string
    }
    type Y struct{
        *Logger
        *log.Logger
        Name string
    }
    ```
    这样的话, 会出现问题. ** 因为之前说过, 匿名组合类型相当于以其类型名称 (出去包名) 作为成员变量的名字.** 这样的话, Y 中就包含了两个 `Logger` 成员, 虽然类型不同, 但是预期会受到编译错误.

    但是只有当没有使用这两个 `Logger` 任何一个的时候才不会出现问题.

## 3.4 可见性

要想使得某个符号对其他包可见 (package) 的话, 需要定义为大写字母开头. 否则小写.

Go 中可访问性是包一级, 而不是类型一级的.

## 3.5 接口
Go 语言接口是整个类型系统的基石.

### 3.5.1 其他语言的接口

💥 Go 中的接口和其他语言中的接口概念不同.

其他语言中的接口是作为不同组件之间的契约存在的, 对契约的实现是强制的, 必须声明的确实现了某个接口, 这就是 "侵入式" 接口,

### 3.5.2 非侵入式接口
- 其他语言中 (java): 实现一个接口就要实现其全部方法.
- go 中: 实现所有方法就是实现了该接口.

一个类 `example` 实现了 `a()`,`b()`,`c()` 三个方法, 但是可能 `interface1` 包含 a,b,`interface2` 包含 c 方法, 这样 `example` 就实现了两个接口. 并且可以相互赋值

```go
val ex1 interface1 = new(example)
val ex2 interface2 = new(example)
```

好处的三点:
1. 标准库中不用再写类图的继承树图
2. 不用纠结接口需要拆的多细, 实现类的时候只需要关系自己应该提供哪些方法.
3. 不用引包

### 3.5.3 接口赋值
1. 将对象实例赋值给接口

    以上面的 Integer 类型进行举例.
    ```go
    var a Integer = 1
    var b Interface1 = &a
    ```
    当接口中为指针类型的时候, 使用 `&a` 进行赋值. 当不是指针的时候,`a` 或者 `&a` 都可以. 尽量使用 `&a` 进行复制.原因见: Page75

    **🦍 [为什么可以将接口的实例赋值给接口 ?](https://stackoverflow.com/questions/13511203/why-cant-i-assign-a-struct-to-an-interface)**
    > 当您有一个实现接口的结构时，指向该结构的指针也会自动实现该接口。 这就是为什么在函数原型中永远没有* SomeInterface的原因，因为这不会向SomeInterface添加任何内容，并且在变量声明中不需要这种类型（请参阅此相关问题）。

    > 接口值不是具体结构的值（因为它的大小可变，所以不可能），但是它是一种指针（更精确地说，是指向结构的指针和指向类型的指针 ）。 拉斯·考克斯（Russ Cox）正是在这里描述它：`https://research.swtch.com/interfaces`


2. 将一个接口赋值给另一个接口

   只要两个接口拥有相同的方法, 无论顺序是否相同, 都可以互相赋值.

   另外: 并不要求两个接口完全相等, 一个接口是另一个的子集也可以.

### 3.5.4 接口查询 `page77`

接口查询的语法:
```go
var file WriterInterface = ...
if file5,ok := file.(Interface2);ok{
    ...
}
```
判断 `file` 接口指向的实例是否实现了 `Interface2` 接口.


还可以询问接口它指向的对象是否为某个类型:

```go
if file2,ok := file.(*File);ok{
    ...
}
```
判读 `file` 接口它指向的对象是否是为 `(*File)` 类型.

查询对象类型只是一个接口查询的一个特例.接口是对一组类型的公共特性的抽象.

### 3.5.5 类型查询
类型查询:
```go
var v1  interface{} = ...
switch v := v1.(type){
    case int:
    case string:
    ...
}
```
`fmt.Println()`中采用的穷举法,对一般的情况判断是否实现了`String()`方法,否则利用反射功能遍历对象的所用成员变量进行打印.

### 3.5.6 接口组合
```go
type ReadWriter interface{
    Reader
    Writer
}
```

### 3.5.7 Any 类型
`interface {}`,典型类型为 `fmt.Println`
```go
func Println(args ...interface{})
```

## 3.6 [Simple Music Player](ch3-SMP.md)

实现基本的命令行程序,实现数据 🎵音乐的管理功能.