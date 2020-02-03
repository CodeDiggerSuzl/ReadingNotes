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

"🌱派生":和可以指针的方式从另一个类型"派生":
```go
type Foo struct{
    *Base
    ...
}
```
这个同样可以实现"派生"的效果,但是在创建`Foo`实例的时候,需要**提供一个 Base 类的指针**.

