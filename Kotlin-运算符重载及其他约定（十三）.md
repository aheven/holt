## Kotlin-运算符重载及其他约定（十三） ##

运算符重载及其他约定
Kotlin使用惯例的原则，而不是像Java那样依赖类型，因为这允许Kotlin将现有的Java类调整为Kotlin语言特性的要求。 由类实现的接口集是固定的，Kotlin不能修改现有类，以便实现其他接口。 另一方面，通过扩展功能的机制，可以定义类的新方法。 您可以将任何约定方法定义为扩展，从而适应任何现有的Java类，而不修改其代码。
首先定义一个`point`的数据类：
<code>
data class Point(val x: Int, val y: Int)
</code>

---
重载算术运算符
在Kotlin中使用约定的最简单的例子是算术运算符。 在Java中，完整的算术运算集只能与原始类型一起使用，此外，+运算符可以与String值一起使用。
但是在其他情况下，这些操作也可以方便。 例如，如果您通过BigInteger类处理数字，则使用+来比较它们更加优雅，而不是明确调用add方法。 要向集合添加元素，您可能需要使用+ =运算符。 Kotlin可以让你这样做，在这一节我们会告诉你它是如何工作的。

---
重载二元算术运算
<pre><code>
data class Point(val x: Int, val y: Int) {
    operator fun plus(other: Point): Point { 1.
        return Point(x + other.x, y + other.y) 2.
    } 
}
&gt;&gt;&gt; println(p1 + p2) 3.
Point(x=40, y=60)
</code></pre>

 1. 约定一个名为plus的方法
 2. 计算Point的值并返回一个新的Point
 3. 使用+运算符进行运算
operator关键字：您打算使用该函数作为相应约定的实现，并且您没有定义一个意外地具有匹配名称的函数。
同时，也可以将上述的代码写成拓展函数：
<pre><code>
data class Point(val x: Int, val y: Int)

operator fun Point.plus(other: Point): Point {
    return Point(x + other.x, y + other.y)
}
</code></pre>
表7.1列出了可以定义的所有二进制运算符和相应的函数名：
![可重载的二元运算符函数名](https://github.com/aheven/holt/blob/master/image/%E5%8F%AF%E9%87%8D%E8%BD%BD%E4%BA%8C%E8%BF%9B%E5%88%B6%E7%AE%97%E6%9C%AF%E8%BF%90%E7%AE%97%E7%AC%A6.png)
您自己的类型的操作符始终使用与标准数字类型相同的优先级。
定义运算符时，不需要为两个操作数使用相同的类型：
<pre><code>
operator fun Point.times(scale: Double): Point {
    return Point((x * scale).toInt(), (y * scale).toInt())
}
&gt;&gt;&gt;println(p1 * 1.5)
Point(x=15, y=30)
</code></pre>
运算符函数的返回类型也可以不同于任一操作数类型：
<pre><code>
operator fun Char.times(count: Int): String {
    return toString().repeat(count)
}
&gt;&gt;&gt;println('a'.times(3))
aaa
</code></pre>

---
重载复合赋值运算符
通常，当您定义一个运算符，如加号时，Kotlin不仅支持+操作，还支持+ =。 运算符如+ =， - =等等于复合分配运算符。 这里有一个例子：
<pre><code>
var point = Point(1, 2)
point += Point(3, 4)
&gt;&gt;&gt;println(point)
Point(x=4, y=6)
</code></pre>
在某些情况下，重载复合赋值运算符定义将修改由其使用的变量引用的对象的+ =操作是有意义的，但不会重新分配引用。 一个这样的情况是将一个元素添加到可变集合中：
<pre><code>
    val numbers = ArrayList<Int>()
    numbers += 42
    &gt;&gt;&gt;println(numbers[0])
    42
</code></pre>
如果您使用Unit返回类型定义名为plusAssign的函数，则在使用+ =运算符时，Kotlin将调用它。
Kotlin标准库在可变集合上定义了一个函数plusAssign，前面的例子使用它：
<pre><code>
operator fun <T> MutableCollection<T>.plusAssign(element: T) {
    this.add(element)
}
</code></pre>
Kotlin标准库支持集合的两种方法。 +和-operator总是返回一个新的集合。 + =和 - =操作符通过在可变集合中进行修改，并通过返回修改的副本在只读集合上进行操作。 （这意味着+ =和 - =只能与只读集合一起使用，如果引用它的变量被声明为var。）作为这些运算符的操作数，可以使用单个元素或其他集合与匹配的元素类型：
<pre><code>
val list = arrayListOf(1,2,3)
list += 4  1.
&gt;&gt;&gt;println(list)
val newList = list + listOf(7,8)  2.
&gt;&gt;&gt;println(newList)
[1, 2, 3, 4]
[1, 2, 3, 4, 7, 8]
</code></pre>

 1. +=添加list元素
 2. +返回一个包含所有元素的新列表。

---
重载一元运算符
重载一元运算符的过程与以前看到的一样：用预定义的名称声明一个函数（成员或扩展名），并用修饰符运算符标记。 我们来看一个例子：
<pre><code>
operator fun Point.unaryMinus(): Point {
    return Point(-x, -y)
}

&gt;&gt;&gt;val point = Point(10, 20)
&gt;&gt;&gt;println(-point)
Point(x=-10, y=-20)
</code></pre>
注：一元运算符没有参数
所有可以重载的一元运算符：
![可重载的一元运算符函数名](https://github.com/aheven/holt/blob/master/image/%E4%B8%80%E5%85%83%E7%AE%97%E6%95%B0%E7%AC%A6%E5%88%97%E8%A1%A8.png)
当定义inc和dec函数来重载递减运算符时，编译器自动支持相同的语义，增加运算符与常规数字类型。 考虑这个例子，它重载BigDecimal类的++运算符：
<pre><code>
operator fun BigDecimal.inc() = this + BigDecimal.ONE

fun main(args: Array<String>) {
    var bd = BigDecimal.ZERO
    println(bd++)
    println(++bd)
}

0
2
</code></pre>

---
等号运算符 ：“equals"
kotlin重载默认调用对象的`equals`方法，注意，如果使用`===`用来检查两个对象地址是否相等，类似于java中的`==`，===操作符不能重载。
equals不能作为扩展来实现，因为从该继承的实现始终优先于扩展。

---
排序运算符 ：compareTo
在Java中，类可以实现Comparable接口，以便在比较值的算法中使用，例如查找最大值或排序。 该接口中定义的compareTo方法用于确定一个对象是否大于另一个对象。 但是在Java中，调用此方法没有速记语法。 使用<和>可以比较原始类型的值。 所有其他类型都需要您明确写入element1.compareTo（element2）。
Kotlin支持相同的Comparable界面。 但是，该接口中定义的compareTo方法可以通过惯例调用，并且比较运算符（<，>，<=和> =）的使用将转换为compareTo的调用。 compareTo的返回类型必须为Int。 表达式p1 &lt;p2等价于p1.compareTo（p2）&lt;0。 其他比较运算符的工作方式完全相同。

---
集合与区间的约定
使用集合的一些最常见的操作是通过索引获取和设置元素，以及检查元素是否属于集合。 所有这些操作都通过操作符语法进行支持：通过索引获取或设置元素，使用语法a [b]（称为索引运算符）。 可以使用in运算符来检查元素是在集合还是范围内，还可以迭代集合。 您可以为自己的类添加用作集合的操作。 现在我们来看看用于支持这些操作的惯例。

---
通过下标来访问元素 ：“get”和“set” 
找集合中可以通过`map[key] = value`来引用元素，你也可以使用方括号引用点坐标：p [0]访问X坐标，p [1]访问Y坐标。 以下是实现和使用它的方法：
<pre><code>
operator fun Point.get(index: Int): Int {
    return when (index) {
        0 -> x
        1 -> y
        else ->
            throw IndexOutOfBoundsException("Invalid coordinate $index")
    }
}
&gt;&gt;&gt;val point1 = Point(1, 2)
&gt;&gt;&gt;println(point1[1])
2
</code></pre>

---
“in”的约定
集合支持的另一个运算符是in运算符，用于检查对象是否属于集合。 相应的函数叫做包含。 我们来实现它，以便您可以使用in运算符来检查点是否属于一个矩形：
<pre><code>
data class Rectangle(val upperLest: Point, val lowerRight: Point) {
    operator fun contains(p: Point): Boolean {
        return p.x in upperLest.x until lowerRight.x &&
                p.y in upperLest.y until lowerRight.y
    }
}
&gt;&gt;&gt;val rect = Rectangle(Point(10, 20), Point(50, 50))
&gt;&gt;&gt;println(Point(20, 30) in rect)
true
println(Point(5, 5) in rect)
false
</code></pre>
注意：in运算符是不包括终点的，如果您使用10到20的数字（包括20）建立常规（关闭）范围。开放范围为10..20，此范围包括所有10到20，包括从10到19的数字，但不包括20。 矩形类通常被定义为使其底部和右侧坐标不是矩形的一部分，因此在这里使用开放范围是合适的。

---
rangeTo 的约定
要创建一个范围，您使用..语法：例如，1..10枚举从1到10的所有数字。您符合2.4.4节中的范围，实际上，`...`方法实际上是调用'rangeTo '函数
rangeTo函数返回一个范围。 您可以为自己的类定义此运算符。 但是，如果您的类实现了Comparable接口，那么您不需要这样做：您可以通过Kotlin标准库创建一系列可比较的元素。
库定义了可以在任何可比元素上调用的rangeTo函数：
<pre><code>
operator fun &lt;T: Comparable<T>> T.rangeTo(that: T): ClosedRange&lt;T>
</code></pre>
<pre><code>
&gt;&gt;&gt;val now = LocalDate.now()
&gt;&gt;&gt;val vacation = now..now.plusDays(10)
&gt;&gt;&gt;println(now.plusWeeks(1) in vacation)
true
</code></pre>
<pre><code>
&gt;&gt;&gt; val n = 9
&gt;&gt;&gt;  println(0..(n + 1))
0..10
</code></pre>
另请注意，表达式0..n.forEach {}不会编译，因为您必须用括号括起范围表达式以调用其上的方法：
<pre><code>
&gt;&gt;&gt;  (0..n).forEach { print(it) }
0123456789
</code></pre>

---
在“for”循环中使用“iterator”的约定
正如我们在第2章中讨论过的，Kotlin中的循环在操作符中使用与范围检查相同的循环。 但是它的含义在这种情况下是不同的：它被用来执行迭代。 这意味着一个诸如for（x in list）{...}的语句将被转换为list.iterator（）的调用，然后在其上重复调用hasNext和next方法，就像在Java中一样。
请注意，在Kotlin中，它也是一个约定，这意味着迭代器方法可以被定义为扩展。 这就解释了为什么可以遍历一个常规的Java字符串：标准库定义了CharSequence的扩展函数迭代器，它是String的超类：
<pre><code>
operator fun CharSequence.iterator(): CharIterator
&gt;&gt;&gt;   for (c in "abc") {}
</code></pre>
您可以为自己的类定义迭代器方法。 例如，定义以下方法可以迭代日期：
<pre><code>
operator fun ClosedRange<LocalDate>.iterator(): Iterator<LocalDate> =
        object : Iterator<LocalDate> { 1.
            var current = start
            override fun hasNext(): Boolean {
                return current &lt;= endInclusive 2.
            }

            override fun next(): LocalDate = current.apply { 3.
                current = plusDays(1) 4.
            }
        }
&gt;&gt;&gt;  val newYear = LocalDate.ofYearDay(2017, 1)
&gt;&gt;&gt;   val daysOff = newYear.minusDays(1)..newYear
&gt;&gt;&gt;    for (dayOff in daysOff) { println(dayOff) } 5.
2016-12-31
2017-01-01
</code></pre>

 1. 此对象通过LocalDate元素实现迭代器。
 2. 注意compareTo惯例用于日期
 3. 在更改之前返回当前日期
 4. 将当前日期增加一天
 5. 当相应的迭代器功能可用时迭代超过daysOff

---
解构声明和组件函数
解构声明。 此功能允许您解压缩单个复合值并将其存储在多个单独的变量中。
<pre><code>
&gt;&gt;&gt;   val p = Point(10, 20)
&gt;&gt;&gt;    val (x, y) = p
&gt;&gt;&gt;     println(x)
10
&gt;&gt;&gt;     println(y)
20
</code></pre>
对于数据类，编译器为主构造函数中声明的每个属性生成一个组件函数，如下所示：
<pre><code>
class Point(val x: Int, val y: Int) {
    operator fun component1(): Int = x
    operator fun component2(): Int = y
}
</code></pre>

---
解构声明和循环
解析声明不仅可以作为顶级语句，还可以在其他可以声明变量的地方工作，例如在循环中。 
<pre><code>
fun printEntries(map: Map&lt;String, String>) {
    for ((key, value) in map) {
        println("$key -> $value")
    }
}
&gt;&gt;&gt; val map = mapOf("Oracle" to "Java", "JetBrains" to "Kotlin")
&gt;&gt;&gt; printEntries(map)
Oracle -> Java
JetBrains -> Kotlin
</code></pre>

---
重用属性访问的逻辑 ：委托属性
我们来看一个依赖于约定的另外一个功能，它是Kotlin中最独特和最强大的功能之一：委托属性。 此功能允许您轻松实现更复杂的属性，而不是在备份字段中存储值，而不会在每个访问器中复制逻辑。 例如，属性可以将它们的值存储在数据库表，浏览器会话，map等中。
此功能的基础是委托：一种设计模式，其中对象而不是执行任务将该任务委派给另一个辅助对象。 辅助对象被称为委托。 这里，这个模式被应用于一个属性，它也可以将其访问器的逻辑委托给一个帮助对象。 您可以手动实现（稍后会看到示例）或使用更好的解决方案：Kotlin为此功能提供语言支持。 我们将从一般的解释开始，然后看具体的例子。

---
委托属性的基本操作
委托属性的一般语法如下：
<pre><code>
class Foo {
var p: Type by Delegate()
}
</code></pre>
属性p将其访问器的逻辑委托给另一个对象：在这种情况下，是Delegate类的新实例。 通过评估by关键字后面的表达式获得该对象，该表达式可以满足属性委托的约定规则。
<pre><code>
class Foo {
  private val delegate = Delegate() 1.
  var p: Type 2.
  set(value: Type) = delegate.setValue(..., value)
  get() = delegate.getValue(...)
}
</code></pre>

 1. 這個幫助屬性是由編譯器生成的。
 2. p屬性的生成訪問器在“委託”上調用getValue和setValue方法。
按照慣例，Delegate類必須具有getValue和setValue方法（後者僅適用於可變屬性）。 像往常一樣，他們可以是成員或擴展。 為了簡化說明，我們省略了它們的參數; 確切的簽名將在本章後面介紹。 在一個簡單的表單中，Delegate類可能如下所示：
<pre><code>
class Delegate {
	operator fun getValue(...) { ... }
	operator fun setValue(..., value: Type) { ... }
}
class Foo {
	var p: Type by Delegate()
}
>>> val foo = Foo()
>>> val oldValue = foo.p
>>> foo.p = newValue
</code></pre>

---
使用委托属性 ：惰性初始化和“by lazy()”
延迟初始化是一种常见模式，需要在第一次访问时按需创建一部分对象。 这在初始化过程消耗大量资源时非常有用，并且在使用该对象时并不总是需要数据。
<pre><code>
>>>val lazyValue: String by lazy {
        println("computed!")
        "Hello"
    }
>>>println(lazyValue)
>>>println(lazyValue)
computed!
Hello
Hello
</code></pre>
lazy函数返回一个对象，该对象具有一个名为getValue的方法，并带有正确的签名，因此您可以将它与by关键字一起使用来创建一个委托属性。 lazy的参数是一个lambda，它调用来初始化该值。 懒惰函数默认是线程安全的; 如果需要，可以指定其他选项来告诉它使用哪个锁，或者如果在多线程环境中从不使用该类，则完全绕过同步。

---
实现委托属性
为了看到如何实现委托属性，我们再举一个例子：当对象的属性发生变化时通知监听器的任务。 这在很多不同的情况下都很有用：例如，当用户界面中显示对象，并且当对象发生更改时，您想自动更新UI。 Java有这样的通知的标准机制：PropertyChangeSupport和PropertyChangeEvent类。
让我们看看如何在Kotlin中使用它们，而不先使用委托属性，然后将代码重构为委托属性。
PropertyChangeSupport类管理监听器的列表并为其分派PropertyChangeEvent事件。 为了使用它，你通常将这个类的一个实例作为bean类的一个字段存储，并委托给它的属性改变处理。
为避免将此字段添加到每个类，您将创建一个小型助手类，该类将存储PropertyChangeSupport实例并跟踪属性更改侦听器。 稍后，您的类将扩展此帮助程序类以访问changeSupport：
<pre><code>
open class PropertyChangeAware {
    protected val changeSupport = PropertyChangeSupport(this)

    fun addPropertyChangeListener(listener: PropertyChangeListener) {
        changeSupport.addPropertyChangeListener(listener)
    }

    fun removePropertyChangeListener(listener: PropertyChangeListener) {
        changeSupport.removePropertyChangeListener(listener)
    }
}
</code></pre>
现在我们来编写Person类。 您将定义一个只读属性（该名称通常不会更改）和两个可写属性：年龄和工资。
当人的年龄或工资发生变化时，班级将通知其听众：
<pre><code>
class Person(val name: String, age: Int, salary: Int) : PropertyChangeAware() {
    var age: Int = age
        set(newValue) {
            val oldValue = field
            field = newValue
            changeSupport.firePropertyChange("age", oldValue, newValue)
        }

    var salary: Int = salary
        set(newValue) {
            val oldValue = field
            field = newValue
            changeSupport.firePropertyChange("salary", oldValue, newValue)
        }
}
>>>val person = Person("Dmitry", 34, 2000)
>>>person.addPropertyChangeListener(PropertyChangeListener { evt ->
        println("Property ${evt.propertyName} changed from ${evt.oldValue} to ${evt.newValue}")
    })
>>> p.age = 35
Property age changed from 34 to 35
>>> p.salary = 2100
Property salary changed from 2000 to 2100
</code></pre>
在setters中有很多重复的代码。 让我们尝试提取一个将存储属性值并激发必要通知的类：
<pre><code>
class ObservableProperty(val propName: String, var propValue: Int, val changeSupport: PropertyChangeSupport) {
    fun getValue(): Int = propValue
    fun setValue(newValue: Int) {
        val oldValue = propValue
        propValue = newValue
        changeSupport.firePropertyChange(propName, oldValue, propValue)
    }
}

class Person(val name: String, age: Int, salary: Int) : PropertyChangeAware() {
    val _age = ObservableProperty("age", age, changeSupport)
    var age: Int
        set(value) = _age.setValue(value)
        get() = _age.getValue()

    val _salary = ObservableProperty("salary", salary, changeSupport)
    var salary: Int
        set(value) = _salary.setValue(value)
        get() = _salary.getValue()
}
</code></pre>
现在您已经接近了解Kotlin委派的属性是如何工作的。 您创建了一个存储属性值的类，并在修改后自动触发属性更改通知。 您删除了逻辑中的重复，但相当多的样板需要为每个属性创建ObservableProperty实例，并将getter和setter委托给它。
kotlin的属性委托功能可以让你摆脱该样板。 但在这之前，您需要更改ObservableProperty方法的签名以符合Kotlin约定所要求的签名：
<pre><code>
class ObservableProperty(var propValue: Int, val changeSupport: PropertyChangeSupport) {
    operator fun getValue(person: Person, property: KProperty&lt;*>): Int = propValue

    operator fun setValue(person: Person, property: KProperty&lt;*>, newValue: Int) {
        val oldValue = propValue
        propValue = newValue
        changeSupport.firePropertyChange(property.name, oldValue, newValue)
    }
}
</code></pre>
与以前的版本相比，此代码有以下更改：

 - 现在将getValue（）和setValue（）函数标记为运算符，按照惯例所使用的所有函数都需要这样做。
 - 您可以为这些函数添加两个参数：一个用于接收属性获取或设置的实例，另一个用于表示属性本身。 该属性表示为KProperty类型的对象。
 - 您从主构造函数中删除名称属性，因为您现在可以通过KProperty访问属性名称。
你终于可以使用Kotlin的委托属性了。 看看代码变得更短了吗？
<pre><code>
class Person(val name: String, age: Int, salary: Int) : PropertyChangeAware() {
    var age: Int by ObservableProperty(age, changeSupport)

    var salary: Int by ObservableProperty(salary, changeSupport)
}
</code></pre>
通过by关键字，Kotlin编译器会自动执行在以前版本的代码中手动执行的操作。 将此代码与以前版本的Person类进行比较：使用委托属性时生成的代码非常相似。 右边的对象称为委托。 Kotlin自动将代理存储在隐藏属性中，并在访问或修改主属性时调用代理上的getValue和setValue。
您可以使用Kotlin标准库中的observable属性支持，而不是手工实现它。 事实证明，标准库已经包含一个类似于ObservableProperty的类。 标准库类没有耦合到你在这里使用的PropertyChangeSupport类，所以你需要传递一个lambda表达式来告诉它如何报告属性值的变化。
<pre><code>
class Person(val name: String, age: Int, salary: Int) : PropertyChangeAware() {
    private val observer = { property: KProperty&lt;*>, oldValue: Int, newValue: Int ->
        changeSupport.firePropertyChange(property.name, oldValue, newValue)
    }

    var age: Int by Delegates.observable(age, observer)

    var salary: Int by Delegates.observable(salary, observer)
}
</code></pre>
by的右边表达式不一定是一个新的实例创建。 它也可以是一个函数调用，另一个属性或任何其他表达式，只要该表达式的值是编译器可以使用正确的参数类型调用getValue和setValue的对象。 和其他约定一样，getValue和setValue既可以是对象本身声明的，也可以是扩展函数。
请注意，我们仅向您展示了如何使用Int类型的委托属性来简化示例。 委托属性机制是完全通用的，也适用于任何其他类型。

---
委托属性的变换规则
让我们总结委托属性如何工作。 假设你有一个具有委托属性的类：
<pre><code>
class Foo {
	var c: Type by MyDelegate()
}
val foo = Foo()
</code></pre>
MyDelegate的实例将存储在一个隐藏的属性中，我们将其称为<delegate>。 编译器也将使用KProperty类型的对象来表示属性。 我们将这个对象称为<property>。
编译器生成以下代码：
<pre><code>
class Foo {
    private val &lt;delegate> = MyDelegate()
    var c: Type
        set(value: Type) = &lt;delegate>.setValue(c, &lt;property>, value)
    get() = &lt;delegate>.getValue(c, &lt;property>)
}
</code></pre>
因此，在对属性的每次访问中，都会调用相应的getValue和setValue方法。

---
在 map 中保存属性值
委托属性起作用的另一种常见模式是具有与其关联的动态定义的一组属性的对象。 这样的对象有时被称为expando对象。 例如，考虑一个联系人管理系统，它允许你存储关于你的联系人的任意信息。 系统中的每个人都有一些必要的属性（例如名字）以特殊的方式处理，以及任何数量的附加属性（每个人都可以不同）（例如，最小的孩子的生日）。
实现这种系统的一种方法是将人员的所有属性存储在map中，并提供用于访问需要特殊处理的信息的属性。
这是一个例子：
<pre><code>
class Person {
    private val _attributes = hashMapOf&lt;String, String>()

    fun setAttribute(attrName: String, value: String) {
        _attributes[attrName] = value
    }

    val name: String
    get() = _attributes["name"]!!
}
>>> val p = Person()
>>> val data = mapOf("name" to "Dmitry", "company" to "JetBrains")
>>>for ((attrName, value) in data) {
...  p.setAttribute(attrName, value)
Dmitry
</code></pre>
在这里，您使用通用API将数据加载到对象中（在真实的项目中，这可能是JSON反序列化或类似的东西），然后使用特定的API来访问一个属性的值。 改变这个使用委托的属性是微不足道的; 您可以直接在by关键字后面放置map：
<pre><code>
class Person {
    private val _attributes = hashMapOf&lt;String, String>()

    fun setAttribute(attrName: String, value: String) {
        _attributes[attrName] = value
    }

    val name: String by _attributes
}
</code></pre>
这是有效的，因为标准库在标准的Map和MutableMap接口上定义了getValue和setValue扩展函数。 该属性的名称将自动用作将值存储在map中的键。 和前面的例子一样，p.name隐藏了_attributes.getValue（p，prop）的调用，而这又被实现为_attributes [prop.name]

---
## 总结 ##

 - Kotlin允许你用相应的名字定义函数来重载一些标准的操作，但是你不能定义你自己的操作符。
 - 比较运算符被映射到equals和compareTo方法的调用。
 - 通过定义名为get，set和contains的函数，您可以支持[]和在运算符中使您的类与Kotlin集合类似。
 - 创建范围并遍历集合也可以通过约定和数组来工作。
 - 解构声明允许你通过解开一个对象来初始化多个变量，这对于返回函数中的多个值很方便。 他们自动处理数据类，并且可以通过定义名为componentN（）的函数来支持自己的类。
 - 委托属性允许您重用逻辑来控制如何存储，初始化，访问和修改属性值，这是构建框架的强大工具。
 - lazy标准库函数提供了一个简单的方法来实现懒惰的初始化属性。
 - Delegates.observable函数允许您添加属性更改的观察者。
 - 委托属性可以使用任何map作为属性委托，为处理具有可变属性集的对象提供了一种灵活的方法。