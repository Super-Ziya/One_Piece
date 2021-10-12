## Kotlin学习

#### 一、基本语法

##### 1、包声明

> kotlin 源文件不需要相匹配的目录和包，源文件可以放在任何文件目录
>
> test() 的全名是 com.runoob.main.test、Runoob 的全名是 com.runoob.main.Runoob
>
> 如果没有指定包，默认为 default 包

```kotlin
package com.runoob.main

import java.util.*

fun test(){}
class Runoob{}
```

- 默认导入：有多个包会默认导入到每个 Kotlin 文件中
  - kotlin.*
  - kotlin.annotation.*
  - kotlin.collections.*
  - kotlin.comparisons.*
  - kotlin.io.*
  - kotlin.ranges.*
  - kotlin.sequences.*
  - kotlin.text.*

##### 2、函数定义

- 关键字：`fun`
- 参数格式：参数：类型

```kotlin
fun sum(a:Int,b:Int):Int{//Int 参数，返回值 Int
    return a + b;
}
```

- 表达式作为函数体，返回类型自动推断：

```kotlin
fun sum(a:Int,b:Int) = a + b
//public 方法必须明确写出返回类型
public fun sum(a:Int,b:Int):Int = a + b
```

- 无返回值的函数：

```kotlin
fun printSum(a:Int,b:Int):Unit{
    print(a + b)
}
//返回值是 Unit 类型可以省略（public 方法也适用）
public fun printSum(a:Int,b:Int){
    print(a + b)
}
```

- 可变长参数函数：
  - 关键字：`vararg`

```kotlin
fun vars(vararg v:Int){
    for(vt in v){
        print(vt)
    }
}
```

- lambda（匿名函数）：

```kotlin
val sumLambda:(Int,Int) -> Int = {x,y -> x+y}
```

##### 3、定义常量与变量

- 可变变量关键字：`var`

`var <标识符> : <类型> = <初始化值>`

- 不可变变量关键字：`val`（Java 中的 final 修饰变量）

`val <标识符> : <类型> = <初始化值>`

> 常量与变量都可以没有初始化值，但是在引用前必须初始化
>
> 编译器支持自动类型判断，即声明时可以不指定类型，由编译器判断

```kotlin
val a:Int = 1
val b = 1 //系统自动推断类型 Int
val c:Int //不在声明时初始化必须提供变量类型
```

##### 4、字符串模板

- `$` ：变量名或变量值
- `$varName` ：变量值
- `${varName.fun()}` ：变量的方法返回值

```kotlin
var a = 1
val s1 = "a is $a" //模板简单名称
val s2 = "${s1.replace("is","was")},but now is $a" //模板中任意表达式
```

##### 5、NULL 检查机制

> Kotlin 的空安全设计对于声明可为空的参数，在使用时要进行空判断处理，有两种处理方式，字段后加 `!!` 像 Java 一样抛出空异常，另一种字段后加 `?` 可不做处理返回值为 null 或配合 `?:` 做空判断处理

```kotlin
var age:String? = "23" // 可为空
val ages = age!!.toInt() //抛出空指针异常
val ages1 = age?.toInt() //不做处理返回 null
val ages2 = age?.toInt() ?: -1 //age 为空返回 -1
```

> 当一个引用可能为 null 值时，对应的类型声明必须明确地标记为可为 null。
>
> 当 str 中的字符串内容不是一个整数时，返回 null：

```kotlin
fun parseInt(str:String):Int?{
	//...
}
```

- 使用返回值可为 null 的函数：

```kotlin
if(args.size < 2){
    print("Two")
    return
}
val x = parseInt(args[0])
val y = parseInt(args[1])
//直接使用 x+y 会导致错误，可能为 null
if(x != null && y != null){
    //进行过 null 值检查之后, x 和 y 的类型会被自动转换为非 null 变量
    print(x + y)
}
```

##### 6、类型检测及自动类型转换

- 运算符：`is`（相当 Java 中 `instanceof` 关键字）

```kotlin
fun getStringLength(obj:Any):Int?{
    if(obj is String){
        //类型判断以后，obj 会被系统自动转换为 String 类型
        return obj.length
    }
    return null
}

fun getStringLength(obj:Any):Int?{
    if(obj !is String)
    	return null
    //在这个分支中，obj 的类型会被自动转换为 String
    return obj.length
}

fun getStringLength(obj:Any):Int?{
    //在 && 运算符的右侧，obj 的类型会被自动转换为 String
    if(obj is String && obj.length > 0)
    	return obj.length
    return null
```

##### 7、区间

- 区间表达式由操作符：`..` 的 `rangeTo` 函数辅以 `in` 和 `!in` 形成
- 区间是为任何可比较类型定义的，但对于整型原生类型，它有一个优化的实现

```kotlin
for(i in 1..4) print(i) //输出 1234
for(i in 4..1) print(i) //没有输出

if(i in 1..10){ //等同 1 <= i && i <= 10
    println(i)
}
//使用步长
for(i in 1..4 step 2) print(i) //输出 13
for(i in 4 downTo 1 step 2) print(i) //输出 42
//使用 until 函数排除结束元素
for(i in 1 until 10){ //i in [1,10) 排除了10
    println(i)
}
```

#### 二、基本数据类型

| 类型   | 位宽度 |
| ------ | ------ |
| Double | 64     |
| Float  | 32     |
| Long   | 64     |
| Int    | 32     |
| Short  | 16     |
| Byte   | 8      |

> 不同于 Java 的是，字符不属于数值类型，是一个独立的数据类型

##### 1、字面常量

- 整型：
  - 十进制：`123`
  - 长整型 Long：`123L`
  - 16 进制：`0x0F`
  - 2 进制：`0b00001011`
  - 不支持8进制

- 浮点型：
  - Doubles（默认写法）: `123.5`, `123.5e10`
  - Floats 使用 f 或者 F 后缀：`123.5f`

- 使用下划线使数字常量易读：
  - `val oneMillion = 1_000_000`

##### 2、比较数字

> Kotlin 中没有基础数据类型，只有封装的数字类型，每定义一个变量，Kotlin 就封装了一个对象，这样可以保证不会出现空指针，数字类型也一样。
>
> 三个等号 `===` 表示比较对象地址，两个 `==` 表示比较两个值大小

##### 3、类型转换

> 较小类型不是较大类型的子类型，不能隐式转换为较大的类型

- 方法：`toXXX` 

```kotlin
val b:Byte = 1
val i:Int = b.toInt()

toByte()
toShort()
toInt()
toLong()
toFloat()
toDouble()
toChar()
```

- 自动类型转换（可以根据上下文环境推断）

```kotlin
val l = 1L + 3 // Long + Int => Long
```

##### 4、位操作符

> 对于 Int 和 Long 类型使用

```kotlin
shl(bits) //左移位 Java <<
shr(bits) //右移位 Java >>
ushr(bits) //无符号右移位 Java >>>
and(bits) //与
or(bits) //或
xor(bits) //异或
inv() //反向
```

##### 5、字符

> 和 Java 不一样，Kotlin 中的 Char 不能直接和数字操作，Char 必需是单引号 `'` 包含起来。比如普通字符 `'0'`，`'a'`

- 特殊字符可以反斜杠转义：
  - `\t、\b、\n、\r、\'、\"、\\、\$` 
- 可以显式把字符转为 Int 数字
- 当需要可空引用时，像数字、字符会被装箱，装箱操作不会保留同一性

##### 6、布尔

- 表示：`Boolean` 
- 若需要可空引用布尔会被装箱
- 内置的布尔运算：

```kotlin
|| //短路逻辑或
&& //短路逻辑与
! //逻辑非
```

##### 7、数组

- 实现：`Array` 
  - size 属性
  - get、set 方法，使用 [] 重载了 get 和 set 方法，可以通过下标获取或者设置数组对应位置的值

- 数组的创建：
  - 使用函数 `arrayOf()`
  - 使用工厂函数

```kotlin
val a = arrayOf(1,2,3)
val b = Array(3,{i -> (i*2)}) //[0,2,4]
```

> 与 Java 不同的是，Kotlin 中数组是不协变的（invariant）
>
> 数组的协变性：如果类Base是类Sub的基类，那么Base[]是Sub[]的基类

- 有各个类型的数组，省去装箱操作，效率更高

```kotlin
val x:IntArray = intArrayOf(1,2,3)

ByteArray
ShortArray
IntArray
```

##### 8、字符串

> 和 Java 一样，String 是不可变的。方括号 [] 语法可以获取字符串中的某个字符，也可以通过 for 循环来遍历

```kotlin
for(c in str){
    println(c)
}
```

- 删除多余空白：
  - 方法：`trimMargin()` 
- 多行字符串：

```kotlin
val text = """
多行字符串
多行字符串
""".trimMargin()
```

##### 9、字符串模板

> 字符串可以包含模板表达式 ，即一些小段代码，会求值并把结果合并到字符串中。 模板表达式以美元符 `$` 开头，由一个简单的名字构成

```kotlin
val i = 10
val s = "i = $i"
```

- 在原生字符串中表示字面值 `$` 字符（不支持反斜杠转义）

```kotlin
$('$')
```

##### 10、Any 与 Java Object 比较

- Any 是 Kotlin 中所有非空类型的根类型
- Any 中定义的方法：`toString()`、`equals()`、`hashCode() ` 3个
- Object 类中定义的方法：`toString()`、`equals()`、`hashCode()`、`getClass()`、`clone()`、`finalize()`、`notify()`、`notifyAll()`、`wait()`、`wait(long)`、`wait(long,int)` 11个

#### 三、条件控制

##### 1、If 表达式

- 作为表达式（不需要像 Java 的三元操作符）

`val max = if(a>b) a else b` 

- 把 if 表达式的结果赋值给一个变量

```kotlin
val max = if(a>b){
    a
}else{
    b
}
```

- 使用区间

```kotlin
val x = 5
if(x in 1..8){
    println(x)
}
```

##### 2、when 表达式

> 类似于 Java 的 switch 操作符
>
> 既可以被当做表达式使用也可以被当做语句使用，被当做表达式，符合条件的分支的值就是整个表达式的值，当做语句使用， 则忽略个别分支的值

```kotlin
when(x){
    1 -> print("x == 1")
    2 -> print("x == 2")
    else -> {//类似 Java switch 操作符中的 default
        print("x == other")
    }
}
```

- 多分支相同处理：

```kotlin
when(x){
    0,1 -> print("x == 0,1")
    else -> print("other")
}
```

- 检测值在不在区间：

```kotlin
when(x){
    in 1..10 -> print("x in the range")
    !in 10..20 -> print("x outside the range")
    else -> print("none")
}
```

- 由于智能转换，可以访问该类型的方法和属性，无需任何额外的检测

```kotlin
fun hasPrefix(x:Any) = when(x){
    is String -> x.startsWith("prefix")
    else -> false
}
```

- 采取 if-else if 链，不提供参数，所有的分支条件都是简单的布尔表达式，当一个分支的条件为真时执行该分支

```kotlin
when{
    x.isOdd() -> print("x is odd")
    x.isEven() -> print("x is even")
    else -> print("x is funny")
}
```

- 使用 `in` 运算符来判断集合内是否包含某实例

```kotlin
val items = setOf("apple","banana","kiwi")
when{
    "orange" in items -> println("juicy")
    "apple" in items -> println("apple is fine too")
}
```

#### 四、循环控制

##### 1、for 循环

> 可以循环遍历任何提供了迭代器的对象

```kotlin
for(item:Int in ints){
    //...
}
```

- 遍历数组：
  - 这种"在区间上遍历"会编译成优化的实现而不会创建额外对象

```kotlin
for(i in array.indices){
    print(array[i])
}
```

- 用库函数 withIndex：

```kotlin
for((index,value) in array.withIndex()){
    println("at $index is $value")
}
```

##### 2、while 与 do..while 循环

> 与 Java 类似

##### 3、返回与跳转

- return、break、continue
- break 与 continue 可以使用标签标记表达式，限制 break 跳转到标签指定的循环后面的执行点，continue 继续标签指定的循环的下一次迭代

```kotlin
loop@ for(i in 1..100){
    for(j in 1..100){
        if(...) break@loop
    }
}
```

- 标签处返回

  - 从 lambda 表达式中返回

  ```kotlin
  fun foo(){
      ints.forEach lit@{
          if(it == 0) return@lit
          print(it)
      }
  }
  ```

  - 隐式标签：
    - 标签与接收 lambda 的函数同名

  ```kotlin
  fun foo(){
      ints.forEach {
          if(it == 0) return@forEach
          print(it)
      }
  }
  ```

  - 匿名函数内部的 return 语句将从该匿名函数自身返回

  ```kotlin
  fun foo(){
      ints.forEach(fun(value:Int){
          if(value == 0) return
          print(value)
      })
  }
  ```

  - 返回值
    - 从标签处 @a 返回值 1

  ```kotlin
  return@a 1
  ```

#### 五、类和对象

##### 1、类定义

- 类可以包含：构造函数和初始化代码块、函数、属性、内部类、对象声明
- 关键字：`class` 

```kotlin
class Runoob{
    //...
}
```

##### 2、类属性

- 关键字 `var` 声明为可变的，关键字 `val` 声明为不可变
- 创建类实例（没有 new 关键字）

```kotlin
val site = Runoob()
```

- 引用属性

```kotlin
site.name
```

- 可以有一个主构造器（类头部的一部分，位于类名后），以及一个或多个次构造器

```kotlin
class Person constructor(firstName:String){}
//主构造器没有注解、没有任何可见度修饰符时constructor关键字可以省略：
class Person(firstName:String){}
```

- `getter` 和 `setter` 
  - 后端变量关键字：`field`，只能用于属性的访问器

```kotlin
var allByDefault:Int? //错误: 需要一个初始化语句, 默认实现了 getter 和 setter 方法
var initialized = 1 //类型为 Int, 默认实现了 getter 和 setter
val simple:Int? //类型为 Int，默认实现 getter，必须在构造函数中初始化
val inferredType = 1 //类型为 Int 类型,默认实现 getter

var lastName:String = "huang"
	get() = field.toUpperCase() //将变量赋值后转换为大写
	set

var no:Int = 100
	get() = field //后端变量
	set(value){
        if(value < 10){
            field = value
        }else{
            field = -1
        }
    }

var heiht:Float = 145.4f
	private set
```

- 延迟初始化
  - 非空属性必须在定义的时候初始化
  - 关键字：`lateinit` 

```kotlin
lateinit var subject:TestSubject

@SetUp fun setup(){
    subject = TestSubject()
}
```

##### 3、主构造器

> 主构造器中不能包含任何代码，初始化代码可以放在初始化代码段中，初始化代码段使用 init 关键字作为前缀
>
> 主构造器的参数可以在初始化代码段中使用，也可以在类主体 n 定义的属性初始化代码中使用

```kotlin
class Person constructor(firstName:String){
    init{
        println("FirstName is $firstName")
    }
}
```

- 通过主构造器来定义属性并初始化属性值

```kotlin
class Perple(val firstNameL:String,val lastName:String){
    //...
}
```

##### 4、次构造函数

> 需加前缀 `constructor`

```kotlin
class Person{
    constructor(parent:Person){
        parent.children.add(this)
    }
}
```

- 同一个类中代理另一个构造函数使用 this 关键字

```kotlin
class Person(val name:String){
    constructor(name:String,age:Int) : this(name){
        //初始化
    }
}
```

> 一个非抽象类没有声明构造函数（主构造函数或次构造函数），会产生一个没有参数的构造函数。构造函数是 public，不想类有公共的构造函数，得声明一个空的主构造函数

```kotlin
class DontCreateMe private constructor(){}
```

> 在 JVM 虚拟机中，如果主构造函数所有参数都有默认值，编译器会生成一个附加的无参的构造函数，这个构造函数会直接使用默认值。这使得 Kotlin 更简单的使用像 Jackson 或 JPA 这样使用无参构造函数来创建类实例的库

##### 5、抽象类

- 关键字：`abstract` 
- 无需对抽象类或抽象成员标注 open 注解

```kotlin
abstract class Derived:Base(){
    override abstract fun f()
}
```

##### 6、嵌套类

```kotlin
class Outer{
    private val bar:Int = 1
    class Nested{
        fun foo() = 2
    }
}
```

##### 7、内部类

- 关键字：`inner` 

> 内部类会带有一个对外部类的对象的引用，所以内部类可以访问外部类成员属性和成员函数

```kotlin
class Outer{
    private val bar:Int = 1
    var v = "成员变量"
    inner class Inner{
        fun foo() = bar //访问外部类成员
        fun innerTest(){
            var o = this@Outer //获取外部类的成员变量，@Outer 是一个 代指 this 来源的标签
            println(o.v)
        }
    }
}
```

##### 8、匿名内部类

```kotlin
class Test{
    fun setInterFace(test:TestInterFace){
        test.test()
    }
}

interface TestInterFace{
    fun test()
}

fun main(args:Array<String>){
    var test = Test()
    
    test.setInterFace(object:TestInterFace{
        override fun test(){
            println("测试")
        }
    })
}
```

##### 9、类的修饰符

- 类属性修饰符

```kotlin
abstract //抽象类
final //类不可继承，默认属性
enum //枚举类
open //类可继承
annotation //注解类
```

- 访问权限修饰符

```kotlin
private //同一文件中可见
protected //同一文件或子类可见
public //所有调用的地方可见
internal //同一模块可见
```

#### 六、继承

> Kotlin 中所有类都继承该 Any 类，它是所有类的超类，对于没有超类型声明的类是默认超类

- Any 提供三个函数
  - `equals()`
  - `hashCode()`
  - `toString()`
- 类要被继承，可用 `open` 修饰

```kotlin
open class Base(p:Int) //基类

class Derived(p:Int) : Base(p)
```

##### 1、构造函数

- 子类有主构造函数， 则基类必须在主构造函数中立即初始化

```kotlin
open class Person(var name:String,var age:Int){} //基类

class Student(name:String,age:Int,var no:String,var score:Int) : Person(name,age){}
```

- 子类没有主构造函数，则必须在每一个二级构造函数中用 super 关键字初始化基类，或者在代理另一个构造函数
  - 初始化基类时，可以调用基类的不同构造方法

```kotlin
class Student : Person{
    constructor(ctx:Context) : super(ctx){}
    constructor(ctx:Context,attrs:ArrtibuteSet) : super(ctx,attrs){}
}
```

##### 2、重写

> 在基类中，使用 fun 声明函数默认为 final 修饰，不能被子类重写
>
> 如果允许子类重写该函数，要添加 open 修饰, 子类重写方法用 override 关键词

```kotlin
open class Person{
    open fun study(){
        println("毕业了")
    }
}

class Student : Person(){
    override fun study(){
        println("大学")
    }
}
```

> 如果有多个相同的方法（继承或者实现自其他类），必须要重写该方法，使用 super 范型去选择性地调用父类的实现

```kotlin
open class A{
    open fun f(){print("A")}
}

interface B{
    fun f(){print("B")}
}

class C() : A(),B{
    override fun f(){
        super<A>,f()
        super<B>,f()
    }
}
```

##### 3、属性重写

> 属性重写使用 override 关键字，属性必须具有兼容类型，每一个声明的属性都可以通过初始化程序或者 getter 方法被重写

```kotlin
open class Foo{
    open val x:Int get{...}
}

class Bar1 : Foo(){
    override val x:Int = ...
}
```

> 可以用 var 属性重写 val 属性，反过来不行。因为 val 属性本身定义了 getter 方法，重写为 var 属性会在衍生类中额外声明一个 setter 方法
>
> 可以在主构造函数中使用 override 关键字作为属性声明的一部分

```kotlin
interface Foo{
    val count:Int
}

class Bar1(override val count:Int) : Foo

class Bar2 : Foo{
    override var count:Int = 0
}
```

#### 七、接口

- 关键字：`interface` 
- 允许方法有默认实现

```kotlin
interface MyInterface{
    fun bar()
    fun foo(){
        print("foo")
    }
}
```

##### 1、实现接口

> 一个类或者对象可以实现一个或多个接口

```kotlin
class Child : MyInterface{
    override fun bar(){...}
}
```

##### 2、接口属性

> 接口中的属性只能是抽象的，不允许初始化值，
>
> 接口不会保存属性值，实现接口时，必须重写属性

```kotlin
interface MyInterface{
    var name:String
}

class MyImpl : MyInterface{
    override var name:String = "runoob"
}
```

#### 八、扩展

> Kotlin 可以对一个类的属性和方法进行扩展，且不需要继承或使用 Decorator 模式
>
> 扩展是一种静态行为，对被扩展的类代码本身不会造成任何影响

##### 1、扩展函数

 ```kotlin
fun receiverType.functionName(params){
    body
}
//receiverType：函数的扩展对象
//functionName：扩展函数名称
//params：扩展函数的参数，可为 NULL

class User(var name:String)

fun User.Print(){
    print("name is $name")
}
 ```

> 扩展函数是静态解析，不是接收者类型的虚拟成员，在调用扩展函数时，具体被调用的的是哪一个函数，由调用函数的的对象表达式来决定的，而不是动态的类型决定的
>
> 若扩展函数和成员函数一致，则使用该函数时，会优先使用成员函数

##### 2、扩展一个空对象

> 在扩展函数内， 可以通过 this 来判断接收者是否为 NULL，这样，即使接收者为 NULL，也可以调用扩展函数

```kotlin
fun Any?.toString():String{
    if(this == null) return "null"
    return toString()
}
```

##### 3、扩展属性

```kotlin
val <T> List<T>.lastIndex:Int
	get() = size - 1
```

> 扩展属性允许定义在类或者 kotlin 文件中，不允许定义在函数中
>
> 初始化属性因为属性没有后端字段（backing field），所以不允许被初始化，只能由显式提供的 getter/setter 定义
>
> 扩展对象只能被声明为 val

```kotlin
val Foo.bar = 1 //错误，扩展属性不能有初始化器
```

##### 4、伴生对象的扩展

- 通过 "类名." 形式调用伴生对象，伴生对象声明的扩展函数，通过用类名限定符来调用
- 伴生对象弥补kotlin没有static的缺陷
- `object` 关键字定义一个类同时创建一个实例

```kotlin
class MyClass{
    companion object{} //被称为 Companion
}

fun MyClass.Companion.foo(){
    println("伴随对象的扩展函数")
}

val MyClass.Companion.no:Int
	get() = 10

fun main(args:Array<String>)
```

##### 5、扩展作用域

- 通常扩展函数或属性定义在顶级包下

```kotlin
package foo.bar
fun Baz.goo(){...}
```

- 使用定义包之外的扩展，需要 `import` 导入扩展的函数名

```kotlin
package com.example.usage

import foo.bar.goo //导入所有名为 goo 的扩展
import foo.bar.* //从 foo.bar 导入一切

fun usage(baz:Baz){
    baz.goo()
}
```

##### 6、扩展声明为成员

> 在一个类内部你可以为另一个类声明扩展
>
> 在扩展中，有个多个隐含的接受者，其中扩展方法定义所在类的实例称为分发接受者，扩展方法的目标类型的实例称为扩展接受者

```kotlin
class D{
    fun bar(){println("D")}
}

class C{
    fun bar(){println("C")}
    fun D.foo(){
        bar() //调用 D.bar
        baz() //调用 C.baz
    }
    fun caller(d:D){
        d.foo() //调用扩展函数
    }
}
```

> 在 C 类内创建了 D 类的扩展。此时 C 为分发接受者，D 为扩展接受者
>
> 在扩展函数中，可以调用派发接收者的成员函数
>
> 调用某一函数，而该函数在分发接受者和扩展接受者均存在，以扩展接收者优先，引用分发接收者的成员可以使用限定的 this 语法

```kotlin
class D{
    fun bar(){println("D")}
}

class C{
    fun bar(){println("C")}
    fun D.foo(){
        bar() //调用 D.bar，扩展接收者优先
        this@C.baz() //调用 C.baz
    }
    fun caller(d:D){
        d.foo() //调用扩展函数
    }
}
```

> 以成员形式定义的扩展函数，可以声明为 open，可以在子类中覆盖，在这类扩展函数的派发过程中，针对分发接受者是虚拟的（virtual），但针对扩展接受者仍然是静态的

```kotlin
open class D{}

class D1:D{}

open class C{
    open fun D.foo(){
        println("D.foo")
    }
    open fun D1.foo(){
        println("D1.foo")
    }
    fun caller(d:D){
        d.foo //调用扩展函数
    }
}

class C1:C{
	override fun D.foo(){
        println("C1 D.foo")
    }
    override fun D1.foo(){
        println("C1 D1.foo")
    }
}

fun main(args:Array<String>){
    C().caller(D()) //输出 D.foo
    C1().caller(D()) //输出 C1 D.foo，分发接收者虚拟解析
    C().caller(D1()) //输出 D.foo，扩展接收者静态解析
}
```

#### 九、数据类与密封类

##### 1、数据类

- 关键字：`data` 

> 编译器会自动从主构造函数中根据所有声明的属性提取以下函数：

- `equals()` / `hashCode()`
- `toString()` 格式如 `"User(name=John, age=42)"`
- `componentN() functions` 对应于属性，按声明顺序排列
- `copy()` 函数

> 若这些函数在类中已被明确定义，或者从超类中继承而来，就不再会生成
>
> 为保证生成代码的一致性及有意义，数据类需要满足以下条件：

- 主构造函数至少包含一个参数
- 所有的主构造函数的参数必须标识为 `val` 或者 `var` 
- 数据类不可以声明为 `abstract`, `open`, `sealed` 或者 `inner`
- 数据类不能继承其他类 （但是可以实现接口）

##### 2、复制

> 复制使用 `copy()` 函数，可以使用该函数复制对象并修改部分属性

```kotlin
data class User(val name:String,val age:Int)

fun main(args:Array<String>){
    val jack = User(name = "jack",age = 1)
    val olderJack = jack.copy(age = 2)
} 
```

##### 3、数据类及解构声明

> 组件函数允许数据类在解构声明中使用：

```kotlin
val jane = User("Jane",35)
val (name,age) = jane
```

##### 4、密封类

> 密封类用来表示受限的类继承结构：一个值为有限几种的类型, 不能有任何其他类型时
>
> 枚举类的扩展：枚举类型的值集合是受限的，每个枚举常量只存在一个实例，而密封类的一个子类可以有可包含状态的多个实例
>
> 密封类可以有子类，但是所有的子类都必须要内嵌在密封类中
>

- 使用 `sealed` 修饰
  - `sealed` 不能修饰 `interface`、`abstract class`（会报 warning，但不会出现编译错误）

```kotlin
sealed class Expr
data class Const(val number: Double): Expr()
data class Sum(val e1: Expr,val e2: Expr): Expr()
object NotANumber : Expr()

fun eval(expr: Expr): Double = when(expr){
    is Const -> expr.number
    is Sum -> eval(expr.e1) + eval(expr.e2)
    NotANumber -> Double.NaN
    //使用密封类的好处：不需要else子句，已经覆盖了所有情况
}
```

#### 十、泛型

> 泛型，即 "参数化类型"，将类型参数化，可以用在类，接口，方法上

```kotlin
class Box<T>(t:T){
    var value = t
}
//创建类实例要指定参数类型
val box: Box<Int> = Box<Int>(1)
val box = Box(1) //编译器会进行类型推断
```

> 定义泛型类型变量，可以完整地写明类型参数，如果编译器可以自动推定类型参数，也可以省略类型参数、泛型参数
>
> Kotlin 泛型函数的声明与 Java 相同，类型参数要放在函数名的前面

```kotlin
fun <T> boxInt(value: T) = Box(value)

val box = boxInt<Int>(1)
val box = boxInt(1) //编译器会进行类型推断
```

##### 1、泛型约束

> Kotlin 中使用 : 对泛型的类型上限进行约束
>
> 默认上界是 `Any?` 

```kotlin
//Comparable 的子类可以代替 T
fun <T: Comparable<T>> sort(list: List<T>){
    //...
}

sort(listOf(1,2,3)) //正确，Int 是 Comparable<Int> 的子类型
sort(listOf(HashMap<Int,String>())) //错误，HashMap<Int,String> 不是 Comparable<HashMap<Int,String>> 的子类型
```

> 对于多个上界约束条件，可调用 `where` 子句

```kotlin
fun<T> copyWhenGreater(list: List<T>,threshold: T): List<String> where T: CharSequence,T: Comparable<T>{
    return list.filter{it > threshold}.map{it.toString()}
}
```

##### 2、型变

> Kotlin 中没有通配符类型，有声明处型变与类型投影

- 声明处型变

  - 修饰符：`in`、`out` 
  - `out` 使一个类型参数协变，协变类型参数只能用作输出，可以作为返回值类型但是无法作为入参类型

  ```kotlin
  class Runoob<out A>(val a: A){
      fun foo(): A{
          return a
      }
  }
  
  fun main(args: Array<String>){
      var strCo: Runoob<String> = Runoob("a")
      var anyCo: Runoob<Any> = Runoob<Any>("b")
      anyCo = strCo
      println(anyCo.foo()) //输出 a
  }
  ```

  - `in` 使得一个类型参数逆变，逆变类型参数只能用作输入，可以作为入参的类型但是无法作为返回值的类型

  ```kotlin
  class Runoob<in A>(a: A){
      fun foo(a: A){}
  }
  ```

- 星号投射

  > 不知道类型参数的任何信息，但仍然希望能够安全地使用它。“安全地使用”指对泛型类型定义一个类型投射，要求这个泛型类型的所有的实体实例都是这个投射的子类型

  - 假如类型定义为 `Foo<out T>`，T 的上界为 TUpper，`Foo<*>` 等价于 `Foo<out TUpper>`，表示当 T 未知时，可以安全地从 `Foo<*>` 中读取TUpper 类型的值
  - 假如类型定义为 `Foo<in T>`，`Foo<*>` 等价于 `Foo<in Nothing>`，表示当 T 未知时，不能安全地向 `Foo<*>` 写入任何东西
  - 假如类型定义为 `Foo<T>`，T 的上界为 TUpper , 对于读取值的场合，`Foo<*>` 等价于 `Foo<out TUpper>`，对于写入值的场合，等价于 `Foo<in Nothing>`

  > 如果一个泛型类型中存在多个类型参数，每个类型参数都可单独投射，假设类型定义为 `interface Function<in T, out U>` 

  - `Function<*, String>`，代表 `Function<in Nothing, String>` 
  - `Function<Int, *>`，代表 `Function<Int, out Any?>` 
  - `Function<*, *>`，代表 `Function<in Nothing, out Any?>` 

  > 星号投射与 Java 的原生类型非常类似，但可以安全使用

#### 十一、枚举类

```kotlin
enum class Color{
    RED,BLACK,BLUE
}
```

##### 1、枚举类初始化

> 每个枚举都是枚举类的实例，可以被初始化

```kotlin
enum class Color(val rgb: Int){
    RED(0xFF0000),
    GREEN(0x00FF00),
    BLUE(0x0000FF)
}
```

> 默认名称为枚举字符名，值从0开始
>
> 若需指定值，可以使用其构造函数

```kotlin
enum class Shape(value: Int){
    ovel(100),
    rectangle(200)
}
```

> 支持以声明自己的匿名类及相应的方法、以及覆盖基类的方法
>
> 如果枚举类定义任何成员，要使用分号将成员定义中的枚举常量定义分隔开

```kotlin
enum class ProtocolState{
    WAITING{
        override fun signal() = TALKING
    },
    TALKING{
        override fun signal() = WAITING
    };
    
    abstract fun signal(): ProtocolState
}
```

##### 2、使用枚举常量

> Kotlin 中的枚举类具有合成方法，允许遍历定义的枚举常量，并通过其名称获取枚举常数

```kotlin
EnumClass.valueOf(value: String): EnumClass //转换指定 name 的枚举值，匹配不成功会抛出 IllegalArgumentException
EnumClass.values(): Array<EnumClass> //以数组形式，返回枚举值

val name: String //获取枚举名称
val ordinal: Int //获取枚举值在所有枚举数组中定义的顺序

enum class Color{
    RED,BLACK,BLUE
}

fun main(args: Aray<String>){
    var color: Color = Color.BLUE
    println(Color.values())
    println(Color.valueOf("RED"))
    println(color.name)
    println(color.ordinal)
}
```

> 自 Kotlin 1.1 起，可以使用 `enumValues()` 和 `enumValueOf()` 函数以泛型的方式访问枚举类中的常量 

```kotlin
enum class RGB{RED,GREEN,BLUE}

inline fun <reified T: Enum<T>> printAllValues(){
    print(enumValues<T>().joinToString{it.name})
}

fun main(args: Array<String>){
    printAllValues<RGB>() //输出 RED,GREEN,BLUE
}
```

#### 十二、对象表达式和对象声明

> Kotlin 用对象表达式和对象声明来实现创建一个对某个类做了轻微改动的类的对象，且不需要去声明一个新的子类

##### 1、对象表达式

> 通过对象表达式实现一个匿名内部类的对象用于方法的参数中

```kotlin
window.addMouseListener(object: MouseAdapter(){
    override fun mouseClicked(e: MouseEvent){
        //...
    }
    override fun mouseEntered(e: MouseEvent){
        //...
    }
})
```

> 对象可以继承于某个基类，或者实现其他接口
>
> 如果超类型有一个构造函数，则必须传递参数给它。多个超类型和接口可以用逗号分隔
>
> 通过对象表达式可以越过类的定义直接得到一个对象

```kotlin
open class A(x: Int){
    public open val y: Int = x
}

interface B{...}

val ab: A = object: A(1),B{
    override val y = 15
}
```

> 匿名对象可以用作只在本地和私有作用域中声明的类型
>
> 如果使用匿名对象作为公有函数的返回类型或者用作公有属性的类型，那么该函数或属性的实际类型会是匿名对象声明的超类型，如果没有声明任何超类型，就会是 Any，在匿名对象中添加的成员将无法访问

```kotlin
class C{
    //私有函数，返回类型是匿名对象类型
    private fun foo() = object{
        val x: String = "x"
    }
    //公有函数，返回类型是 Any
    fun publicFoo() = object{
        val x: String = "x"
    }
    
    fun bar(){
        val x1 = foo.x //正确
        val x2 = publicFoo().x //错误，未能解析引用的“x”
    }
}
```

> 在对象表达中可以访问作用域中的其他变量

```kotlin
fun countClicks(window: JComponent){
    var clickCount = 0
    var enterCount = 0
    
    window.addMouseListener(object: MouseAdapter(){
        override fun mouseClicked(e: MouseEvent){
            clickCount++
        }
        
        override fun mouseEntered(e: MouseEvent){
            enterCount++
        }
    })
}
```

##### 2、对象声明

- 关键字：`object` 
- Kotlin 中可以通过对象声明来获得一个单例

```kotlin
object DataProviderManager{
    fun registerDataProvider(provide: DataProvider){
        //...
    }
    
   	val allDataProviders: Collection<DataProvider>
    	get() = //...
}
//引用该对象，直接使用其名称
DataProviderManager.registerDataProvider(...)
```

> 可以定义一个变量来获取这个对象，当定义两个不同的变量来获取这个对象时，并不能得到两个不同的变量，也就是这种方式获得一个单例
>
> 对象可以有超类型

```kotlin
object DefaultListener: MouseAdapter(){
    override fun mouseClicked(e: MouseEvent){
        //...
    }
    override fun mouseEntered(e: MoiseEvent){
        //...
    }
}
```

> 与对象表达式不同，当对象声明在另一个类的内部时，这个对象并不能通过外部类的实例访问到该对象，而只能通过类名来访问，同样该对象也不能直接访问到外部类的方法和变量

```kotlin
class Site{
    var name = "ziya"
    object Desktop{
        var url = "www.ziya.com"
        fun showName(){
            print{"ziya is $name"} //
        }
    }
}

fun main(args: Array<String>){
    var site = Site()
    site.Desktop.url //错误，不能通过外部类的实例访问到该对象
    Site.Desktop.url //正确
}
```

##### 3、伴生对象

- 关键字：`companion` 
- 可以直接通过外部类访问到对象的内部元素

```kotlin
class Myclass{
    companion object Factory{
        fun create(): MyClass = MyClass()
    }
}

val instance = MyClass.create() //访问到对象的内部元素
```

> 可以省略掉该对象的对象名，使用 Companion 替代需要声明的对象名

```kotlin
class MyClass{
    companion object{}
}

val x = MyClass.Companion
```

> 一个类里面只能声明一个内部关联对象，即关键字 companion 只能使用一次
>
> 请伴生对象的成员像 Java 的静态成员，但在运行时仍然是真实对象的实例成员，还可以实现接口

- 对象表达式和对象声明之间的语义差异
  - 对象表达式是在使用他们的地方立即执行的
  - 对象声明是在第一次被访问到时延迟初始化的
  - 伴生对象的初始化是在相应的类被加载（解析）时，与 Java 静态初始化器的语义相匹配

#### 十三、委托

> 委托模式是一种设计模式。在委托模式中，有两个对象参与处理同一个请求，接受请求的对象将请求委托给另一个对象来处理
>
> Kotlin 直接支持委托模式，更加优雅，简洁

- 关键字：`by` 

##### 1、类委托

> 类委托：一个类中定义的方法实际是调用另一个类的对象的方法来实现的
>

```kotlin
interface Base{
    fun print()
}

//被委托的类
class BaseImpl(val x: Int): Base{
    override fun print(){print(x)}
}

//建立委托类
//by 子句表示将 b 保存在 Derived 对象实例内部，编译器会生成继承自 Base 接口的所有方法, 并将调用转发给 b
class Derived(b: Base): Base by b

fun main(args: Array<String>){
    val b = BaseImpl(10)
    Derived(b).print() //输出10
}
```

##### 2、属性委托

> 属性委托指一个类的某个属性值不是在类中直接进行定义，而是将其托付给一个代理类，从而实现对该类的属性统一管理

```kotlin
val/var <属性名>: <类型> by <表达式>
//表达式：委托代理类，属性的 get()、set() 方法将被委托给这个对象的 getValue() 和 setValue() 方法，属性委托不必实现任何接口, 但必须提供 getValue()、setValue() 函数
```

- 定义被委托的类

  > 该类需要包含 `getValue()` 方法和 `setValue()` 方法，参数 `thisRef` 为进行委托的类的对象，`prop` 为进行委托的属性的对象

```kotlin
import kotlin.reflect.KProperty

class Example{
    var p: String by Delegate()
}
//委托的类
class Delegate{
    operator fun getValue(thisRef: Any?,property: KProperty<*>): String{
        return "$thisRef, ${property.name}"
    }
    operator fun setValue(thisRef: Any?,property: KProperty<*>,value: String){
        println("$thisRef, ${property.name}, $value")
    }
}

fun main(args: Array<String>){
    val e = Example()
    e.p = "ziya" //调用 setValue() 函数
    println(e.p) //调用 getValue() 函数
}
```

##### 3、标准委托

- 延迟属性 Lazy

  > `lazy()` 接受一个 Lambda 表达式作为参数，返回一个 `Lazy <T>` 实例的函数，返回的实例可以作为实现延迟属性的委托： 第一次调用 `get()` 会执行已传递给 `lazy()` 的 Lambda 表达式并记录结果， 后续调用 `get()` 只是返回记录的结果

  ```kotlin
  val lazyValue: String by lazy{
      println("ziya?") //第一次调用输出，第二次调用不执行
      "ziya!"
  }
  
  fun main(args: Array<String>){
      println(lazyValue) //第一次执行，输出ziya? ziya!
      println(lazyValue) //第二次执行，输出返回值 ziya!
  }
  ```

##### 4、可观察属性 Observable

> observable 可以用于实现观察者模式
>
> `Delegates.observable()` 接受两个参数：初始化值、属性值变化事件的响应器（handler）
>
> 在属性赋值后会执行事件的响应器（handler），有三个参数：被赋值的属性、旧值和新值

```kotlin
import kotlin.properties.Delegates

class User{
    var name: String by Delegates.observable("ziya"){
        prop,old,new -> println("旧值$old -> 新值$new")
    }
}

fun main(args: Array<String>){
    val user = User()
    user.name = "new ziya" //输出：旧值ziya -> 新值new ziya
    user.name = "new new ziya" //输出：旧值new ziya -> 新值new new ziya
}
```

##### 5、属性存储在映射中

> 经常出现在像解析 JSON 或者做其他"动态"事情的应用中

```kotlin
class Site(val map: Map<String,Any?>){
    val name: String by map
    val url: String by map
}

fun main(args: Array<String>){
    //构造函数接收一个映射参数
    val site = Site(mapOf(
    	"name" to "ziya"
    	"url" to "www.ziya.com"
    ))
}
```

- 如果使用 var 属性

```kotlin
class Site(val map: MultableMap<String, Any?>){
    val name: String by map
    val url: String by map
}

fun main(args: Array<String>){
 	var map: MultableMap<String, Any?> = multableMapOf{
        "name" to "ziya"
        "url" to "www.ziya.com"
    }
    
    val site = Site(map)
    println(site.name) //输出 ziya
    println(site.url) //输出 www.ziya.com
    
    map.put("name","new ziya")
    map.put("url","new www")
    println(site.name) //输出 new ziya
    println(site.url) //输出 new www
}
```

##### 6、Not Null

> 适用于那些无法在初始化阶段就确定属性值的场合
>
> 如果属性在赋值前被访问会抛出异常

```kotlin
class Foo{
    var notNullBar: String by Delegates.notNUll<String>()
}

foo.notNullBar = "bar"
```

##### 7、局部委托属性

> 将局部变量声明为委托属性。 例如，你可以使一个局部变量惰性初始化

```kotlin
fun example(computeFoo: () -> Foo){
    val memoizedFoo by lazy(computeFoo)
    
    if(someCondition && memoizedFoo.isValid()){
        memoizedFoo.doSomething()
    }
}
```

##### 8、属性委托要求

- 对于只读属性（val），委托必须提供 `getValue()` 函数。该函数接受以下参数：
  - thisRef：与属性所有者类型（对于扩展属性，指被扩展的类型）相同或者是它的超类型
  - property：必须是类型 `KProperty<*>` 或其超类型

  > 这个函数必须返回与属性相同的类型（或其子类型）

- 对于一个值可变属性（var），除 `getValue()` 函数外，委托还必须提供 `setValue()` 函数，这个函数接受以下参数：
  - property：必须是类型 `KProperty<*>` 或其超类型
  - new value：必须和属性同类型或者是它的超类型

##### 9、翻译规则

> 在每个委托属性的实现的背后，Kotlin 编译器都会生成辅助属性并委托给它
>
> 例如对于属性 prop，生成隐藏属性 prop$delegate，而访问器的代码只是简单地委托给这个附加属性
>
> Kotlin 编译器在参数中提供了关于 prop 的所有必要信息：第一个参数 this 引用到外部类 C 的实例，而 this::prop 是 KProperty 类型的反射对象，该对象描述 prop 自身

```kotlin
class C{
    var prop: Type by MyDelegate()
}
//编译器生成的相应代码
class C{
    private val prop$delegate = MyDelegate()
    var prop: Type
    	get() = prop$delegate.getValue(this,this::prop)
    	set(value: Type) = prop$delegate.setValue(this,this::prop,value)
}
```

##### 10、提供委托

> 通过定义 `provideDelegate` 操作符，可以扩展创建属性实现所委托对象的逻辑
>
>  如果 `by` 右侧所使用的对象将 provideDelegate 定义为成员或扩展函数，那么会调用该函数来创建属性委托实例
>
> provideDelegate 一个可能的使用场景是在创建属性时（而不仅在其 getter 或 setter 中）检查属性一致性

- 在绑定之前检查属性名称

  >provideDelegate 的参数与 getValue 相同
  >
  >在创建 MyUI 实例期间，为每个属性调用 provideDelegate 方法，并立即执行必要的验证
  >
  >如果没有这种拦截属性与其委托之间的绑定的能力，为了实现相同的功能， 你必须显式传递属性名

```kotlin
class ResourceLoader<T>(id: ResourceID<T>){
    operator fun provideDelegate(
    	thisRef: MyUrl
        prop: KProperty<*>
    ): ReadOnlyProperty<MyUI,T>{
        checkProperty(thisRef,prop.name)
        //创建委托
    }
    private fun checkProperty(thisRef: MyUrl,name: String){...}
}

fun <T> bindResource(id: ResourceID<T>): ResourceLoader<T>{...}

class MyUrl{
    val image by bindResource(ResourceID.image_id)
    val text by bindResource(ResourceID.text_id)
}
```

- 检查属性名称而不使用 provideDelegate 功能

```kotlin
class MyUrl{
    val image by bindResource(ResourceID.image_id,"image")
    val text by bindResource(ResourceID.text_id,"text")
}

fun <T> MyUrl.bindResource(
	id: ResourceID<T>,
    propertyName: String
): ReadOnlyProperty<MyUrl,T>{
    checkProperty(this,propertyName)
    //创建委托
}
```

> 在生成的代码中，会调用 provideDelegate 方法来初始化辅助的 prop$delegate 属性
>
> provideDelegate 方法只影响辅助属性的创建，不会影响为 getter 或 setter 生成的代码

```kotlin
class C{
    var prop: Type by MyDelegate()
}
//当 provideDelegate 功能可用时编译器生成的代码
class C{
    //调用 provideDelegate 来创建额外的 delegate 属性
    private val prop$delegate = MyDelegate().provideDelegate(this,this::prop)
    val prop: Type
    	get() = prop$delegate.getValue(this,this::prop)
}
```