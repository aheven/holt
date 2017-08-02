# Kotlin类、数据类与接口（十）
kotlin的类与接口与java的有所不同，比如kotlin接口可以包含属性，kotlin的声明默认的是final和public。此外，在默认情况下，嵌套类不是内部类，它们不包含对其外部类的隐式引用。
## 定义类结构 ##
接口：kotlin接口类似于java8，它们可以包含抽象方法的定义以及非抽象方法的实现。要声明一个接口，使用interface关键字替代class关键字。
<pre><code>
interface Clickable{
    fun click()
}
</code></pre>
实现接口：
<pre><code>
class Button : Clickable {
    override fun click() = println("I was clicked")
}
</code></pre>
kotlin使用`:`替换java中的`extends`和`implements`关键字。`override`修饰符与java中的`@Override`类似，用于标记从父类或接口中覆盖的方法和属性。与java不同的是，在Kotlin中使用override修饰符是必需的。
一个接口方法可以有一个默认的实现，java8要求使用default关键字来标记这种实现。在kotlin中，此类方法没有特殊的注释：只需要提供一个函数体。
<pre><code>
interface Clickable {
    fun click()
    fun showOff() = println("I'm clickable!")
}
</code></pre>
其中`showOff()`为函数提供了默认的实现方式。
假设另一个接口也实现了`showOff()`函数，并且具有以下实现：
<pre><code>
interface Focusable {
    fun setFocus(b: Boolean) = println("I ${if (b) "got" else "lost"} focus.")
    fun showOff() = println("I'm focusable!")
}
</code></pre>
如果需要在类中同时实现Clickable与Focusable接口，每个接口都包含showOff()方法，这时候默认实现哪一个？
实际上，在kotlin中，如果不明确实现showOff()，则会收到编译器错误。
<pre><code>
The class Button must override public open fun showOff() because it inherits many implementations of it.
</code></pre>
kotlin编译器会强制要求实现showOff()方法：
<pre><code>
class Button : Clickable,Focusable {
    override fun showOff() {
        super<Focusable>.showOff()
        super<Clickable>.showOff()
    }

    override fun click() = showOff()
}
</code></pre>

 - super由括号中的父类类型名称限定，指定要从中调用方法的父类。
如果只需要调用一个继承的实现，你可以这样写：
<pre><code>
class Button : Clickable, Focusable {
    override fun showOff() = super<Clickable>.showOff()

    override fun click() = showOff()
}
</code></pre>
在java中实现kotlin的接口，kotlin1.0设计的目标版本为Java6,它不支持接口中的默认方法。因此，如果要在java类中实现包含默认方法的接口，则必须定义自己的所有方法的实现，包括在Kotlin中具有方法体的默认方法。
Kotlin的未来版本将支持生成Java 8字节码作为选项。 如果您选择定位Java 8，Kotlin的方法体将被编译为默认方法，您将可以在Java中实现这些接口，而无需为方法提供实现。

---
final修饰符是类的默认修饰符
Java允许创建任何类的子类，并覆盖任何方法，除非已被明确标记为final关键字。 默认情况下，Java类和方法是open，Kotlin是final。
如果要允许创建类的子类，则需要使用open修饰符标记该类。
<pre><code>
open class RichButton:Clickable{
    fun disable() {}

    open fun animate() {}

    override fun click() {
    }
}
</code></pre>
请注意，如果覆盖基类或接口的成员，则默认情况下，覆盖成员也将被标记为open。
如果你想改变这一点，并禁止你的类的子类覆盖你的实现，你可以明确地将覆盖成员标记为final
<pre><code>
open class RichButton:Clickable{
    override fun click() {
    }
}
</code></pre>
在Kotlin中，和Java一样，你可以声明一个类抽象，这个类不能被实例化。
![访问修饰符](https://github.com/aheven/holt/blob/master/image/%E8%AE%BF%E9%97%AE%E4%BF%AE%E9%A5%B0%E7%AC%A6.png)
抽象类通常包含没有实现的抽象成员，必须在子类中被覆盖。 抽象成员总是打开的，所以你不需要使用一个明确的open修饰符。示例：
<pre><code>
abstract class Animated{
    abstract fun animate()

    open fun stopAnimating() {
    }

    fun animateTwice() {
    }
}
</code></pre>
与Java一样，类可以实现多接口，但它只能扩展一个类。

---
可见性修饰符：默认为public
![可见修饰符](https://github.com/aheven/holt/blob/master/image/%E5%8F%AF%E8%A7%81%E4%BF%AE%E9%A5%B0%E7%AC%A6.png)
请注意Java和Kotlin中protected修饰符的行为差异。 在Java中，您可以从同一个包中访问受保护的成员，但Kotlin不允许。 在Kotlin中，可见性规则很简单，受保护的成员只能在类及其子类中可见。 还要注意，类的扩展功能不能访问其私有或受保护的成员。
Kotlin和Java之间的可见性规则有一个区别是外部类不会在Kotlin中看到其内部（或嵌套）类的私有成员。

---
内嵌和嵌套类：默认嵌套
在Java中与Kotlin中，您可以在另一个类中声明一个类。 这样做可以用于封装帮助类或将代码放置在更接近使用的位置。 不同的是，Kotlin嵌套类无权访问外部类实例，除非您特别要求。
kotlin中的嵌套类不持有对外部类的引用，以下是一个嵌套类的示例：
<pre><code>
class Outter{
    class Nested{
        fun execute(){
            println("nested run")
        }
    }

    fun outFun(){
        Nested().execute()
    }
}

fun main(args: Array<String>) {
    Outter.Nested().execute()
}
</code></pre>
kotlin中没有显式修饰符的嵌套类与Java中的静态嵌套类相同。
如果需要在内部类中持有外部的引用，需要使用`inner`修饰符:
![Java和Kotlin中的嵌套和内部类之间的对应关系](https://github.com/aheven/holt/blob/master/image/Java%E5%92%8CKotlin%E4%B8%AD%E7%9A%84%E5%B5%8C%E5%A5%97%E5%92%8C%E5%86%85%E9%83%A8%E7%B1%BB%E4%B9%8B%E9%97%B4%E7%9A%84%E5%AF%B9%E5%BA%94%E5%85%B3%E7%B3%BB.png)
在kotlin中引用外部类实例与java也有区别，kotlin中，使用 this@Outer语法可以从内部类中获取外部实例
<pre><code>
class Outer {
    inner class Inner {
        fun getOuterReference(): Outer = this@Outer
    }
}
</code></pre>

---
密封类：定义受限类层次结构
密封类通常与when语句使用，下面开始分析下列代码
<pre><code>
interface Expr
class Num(val value: Int) : Expr
class Sum(val left: Expr, val right: Expr) : Expr

fun eval(e: Expr): Int =
        when (e) {
            is Num -> e.value
            is Sum -> eval(e.right) + eval(e.left)
            else ->
                throw IllegalArgumentException("Unknown expression")
        }
</code></pre>
当使用when语句来判断表达式时，kotlin编译器会强制检查默认选项，在上面的例子中，else中的不能返回任何有意义的东西，所以只能抛出一个异常。始终不得不添加默认分支（else分支）是不方便的，此外，如果添加了一个新的`Expr`子类，kotlin不会检测到内容的变化，那么默认分支将会被选中，因此这可能造成程序上的微小错误。
kotlin为上述问题提供了方案，使用`sealed`修饰符标记父类，所有直接子类必须在父类中：
<pre><code>
sealed class Expr {1.
    class Num(val value: Int) : Expr() {}2.
    class Sum(val left: Expr, val right: Expr) : Expr()
}

fun eval(e: Expr): Int =
        when (e) {3.
            is Expr.Num -> e.value
            is Expr.Sum -> eval(e.right) + eval(e.left)
        }
</code></pre>

 1. 将类标记为密封类
 2. 列出所有的子类为嵌套类
 3. `when`表达式列出了所有可能的情况，所以不需要`else`分支
注意：密封类默认是开放的，因此不需要`open`修饰符，密封类不能在类定义继承者。data class不能作为密封类。
## 声明与非平凡构造或属性的类 ##
在Java中，您知道，类可以声明一个或多个构造函数。 Kotlin是类似的，另外还有一个变化：它区分主构造函数（通常是初始化一个类的主要简洁方法，并且在类体之外被声明）和辅助构造函数（在类中声明）。
它还允许您在初始化程序块中添加其他初始化逻辑。 首先，我们将看一下声明主构造函数和初始化程序块的语法，然后我们将解释如何声明多个构造函数。 之后，我们将再讨论一下属性。
初始化类：主构造函数和初始化程序块
声明一个简单类的方法：
<pre><code>
class User(val username: String)
</code></pre>
通常，一个类中的所有声明都放在大括号内。上述代码中由括号括起来的代码块称为主构造函数，它有两个功能：指定构造函数参数和定义由这些参数初始化的属性。
下面的代码是一更明确的方式实现了上述代码：
<pre><code>
class User constructor(_nickname: String) {1.
    val nickname: String

    init {2.
        nickname = _nickname
    }
}
</code></pre>

 1. 包含一个参数的主构造函数
 2. 初始化代码块
`constructor`关键字开始声明一个主构造函数或次构造函数
`init`关键字引入一个初始化代码块
在此示例中，不需要将初始化代码放在初始化程序块中，因为它可以与`nickname`属性的声明组合。如果主构造函数上没有注释或可见性修饰符，那么也可以省略构造函数关键字：
<pre><code>
class User(_nickname: String) {
    val nickname: String = _nickname
}
</code></pre>
前面两个例子通过在类的body中使用val关键字声明了该属性。如果属性由相应的构造函数参数初始化，则可以通过在参数之前添加val关键字来简化代码。 这将替换类中的属性定义：
<pre><code>
class User(val  nickname: String)
</code></pre>
“val”表示为构造函数参数生成相应的属性。
您可以为构造函数参数声明默认值，就像函数参数一样：
<pre><code>
class User(val nickname: String, val isSubscribed: Boolean = false)
</code></pre>
如果所有构造函数参数都具有默认值，则编译器将生成一个附加构造函数，而不使用所有默认值的参数。 这使得使用Kotlin更容易使用通过无参数构造函数实例化类的库。
<pre><code>
class User(val nickname: String? = null, val isSubscribed: Boolean = false)

fun main(args: Array<String>) {
    val user = User()
    println(user.toString())
}
</code></pre>
如果你的类有一个超类，主构造函数也需要初始化超类。 您可以通过在基类列表中的超类引用之后提供超类构造函数来执行此操作：
<pre><code>
open class User(val nickname: String)

class TwitterUser(nickname: String) : User(nickname)
</code></pre>
如果您没有为类声明任何构造函数，那么将为您生成不执行任何操作的默认构造函数：
<pre><code>
open class Button
</code></pre>
如果您继承Button类并且不提供任何构造函数，则必须显式调用超类的构造函数，即使它没有任何参数：
<pre><code>
class RadioButton : Button()
</code></pre>
这就是为什么在超类名称之后需要空括号。 注意与接口的区别：接口没有构造函数。
如果想确保类不能被其他代码实例化，则必须私有化主构造函数：
<pre><code>
class Secretive private constructor()
</code></pre>
或者，您可以在类的正文中以更常规的方式声明它：
<pre><code>
class Secretive{
    private constructor()
}
</code></pre>

---
辅助构造器-以不同的方式初始化超类
一般来说，具有多个构造函数的类在Kotlin代码中不如Java中常见。 在Java中需要重载构造函数的大多数情况下，Kotlin都支持默认参数值。注意，不要声明多个辅助构造函数来重载并提供参数的默认值。 而是直接指定默认值。
<pre><code>
open class View {
    constructor(ctx: Context) {
// some code
    }

    constructor(ctx: Context, attr: AttributeSet) {
// some code
    }
}
</code></pre>
`View`类并没有声明一个主构造参数（正如你所知道的那样，因为类头中的类名没有括号），而是声明了两个辅助构造函数。 使用constructor关键字引入辅助构造函数。您可以根据需要声明任意多个辅助构造函数。
如果你想扩展这个类，你可以声明相同的构造函数：
<pre><code>
class MyButton : View {
    constructor(ctx: Context) : super(ctx) {
    }

    constructor(ctx: Context, attr: AttributeSet) : super(ctx, attr) {
    }
}
</code></pre>
这里定义两个构造函数，每个构造函数使用super（）关键字调用超类的对应构造函数。
与Java一样，您还可以使用this（）关键字从构造函数中调用自己类的另一个构造函数。
<pre><code>
class MyButton : View {
    constructor(ctx: Context) : this(ctx, MY_STYLE) {
    }

    constructor(ctx: Context, attr: AttributeSet) : super(ctx, attr) {
    }
}
</code></pre>

---
在接口中声明属性
在Kotlin中，一个接口可以包含抽象属性声明。 这是一个这样声明的接口定义的例子：
<pre><code>
interface User {
    val nickname: String
}
</code></pre>
这表明，实现`User`接口的类需要提供一种获取昵称的值的方法。以下为抽象属性声明的示例：
<pre><code>
class PrivateUser(override val username: String) : User

class SubscribingUser(val email: String) : User {
    override val username: String
        get() = email.substringBefore("@")
}

class FacebookUser(val accountId:Int):User{
    override val username: String = getFacebookName(accountId)
}
</code></pre>
对于PrivateUser，您可以使用简洁的语法在主构造函数中直接声明属性。 此属性实现了User的抽象属性，因此您将其标记为override
对于FacebookUser，您可以将值在初始化代码块中实现nickname属性。
注意SubscribingUser和FacebookUser中不同的实现。 虽然它们看起来类似，但是第一个属性具有一个定制的getter，它可以在每次访问时计算substringBefore，而在FacebookUser中的属性具有一个备份字段，用于存储类初始化期间计算的数据。

---
自定义属性中的`field`字段
使用计算每个访问权值的自定义访问器存储值和属性的属性。 现在我们来看看如何组合这两个并实现存储一个值的属性，并提供在访问或修改该值时执行的额外的逻辑。 为了支持这一点，您需要能够从其访问者访问该属性的备份字段。
<pre><code>
class User(val name: String) {
    var address: String = "unspecified"
        set(value) {
            println("""Address was changed for $name:"$field" -> "$value".""".trimIndent())
            field = value
        }
}

fun main(args: Array<String>) {
    val user = User("Alice")
    user.address = "Elsenheimerstraße 47, 80687 München"
    println(user.address)
}
</code></pre>

 1. 读取支持字段值
 2. 更新备份字段值

---
更改访问者的可见性
默认情况下，访问者的可见性与属性相同。 但是，如果需要，可以通过在get或set关键字之前放置一个可见性修饰符来更改此值。 要看看如何使用它，我们来看一个例子：
<pre><code>
class LengthCounter{
    var counter:Int = 0
    private set(value) {}
}
</code></pre>
您不能在类之外更改此属性。

---
## 编译器生成的方法：数据类和类委派##
Java平台定义了许多需要存在于许多类中的方法，通常以机械方式实现，如equals（），hashCode（）和toString（））。
幸运的是，Java IDE可以自动生成这些方法，因此您通常不需要手动编写它们。 但在这种情况下，您的代码库包含样板代码。 Kotlin编译器向前迈进了一步：它可以在后台执行机械代码生成，而不会使您的源代码文件与结果混淆。

---
通用对象方法
<pre><code>
class Client(val name: String, val postalCode: Int)
</code></pre>
重写上面代码的toString()方法：
<pre><code>
class Client(val name: String, val postalCode: Int) {
    override fun toString() = "Client(name=$name,postalCode=$postalCode)"
}
</code></pre>
在Kotlin中，==检查对象是否相等，而不是引用。
在Kotlin中，==是比较两个对象的默认方式：它通过编译器调用equals来比较它们的值。 因此，如果在类中覆盖了equals，则可以使用==来安全地比较其实例。
重写上面代码的hashcode与equals方法，可以通过编译器自动生成：
<pre><code>
class Client(val name: String, val postalCode: Int) {
    override fun toString() = "Client(name=$name,postalCode=$postalCode)"
    override fun equals(other: Any?): Boolean {
        if (this === other) return true
        if (other?.javaClass != javaClass) return false

        other as Client

        if (name != other.name) return false
        if (postalCode != other.postalCode) return false

        return true
    }

    override fun hashCode(): Int {
        var result = name.hashCode()
        result = 31 * result + postalCode
        return result
    }
}
</code></pre>

---
data class：自动生成通用方法的实现
好消息是，您不必在Kotlin中生成toString(),hashcode(),equals()这些方法。 如果您将修改器数据添加到类中，则会自动为您生成所有必需的方法：
<pre><code>
data class Client(val name: String, val postalCode: Int)
</code></pre>

 - equals（）用于比较实例
 - hashCode（）用于使用它们作为基于哈希的容器（如HashMap）中的键
 - toString（）用于生成以声明顺序显示所有字段的字符串表示形式
 请注意，在主构造函数中未声明的属性不参与相等检查和哈希码计算。
 为了更容易将数据类用作不可变对象，Kotlin编译器为它们生成一个方法：允许您复制类的实例，更改某些属性的值的方法。 创建副本通常是修改实例的好选择：复制具有单独的生命周期，并且不能影响引用原始实例的代码中的位置。 这是copy（）方法：
 <pre><code>
 fun main(args: Array<String>) {
    val client = Client("Tom", 0)
    val client1 = Client("jack", 0)
    val client2 = Client("kate", 0)
    val newClient = client.copy(name = "admin")
    val listOf = mutableListOf<Client>(client, client1, client2)
    listOf.replace(1,newClient)
    println(listOf)
}
fun <E> MutableList<E>.replace(index: Int, e: E) {
    removeAt(index)
    add(index, e)
}
</code></pre>

---
类委托：
通常，您需要添加行为到另一个类，即使它不是设计为扩展。 一种常用的实现方法就是Decorator模式。
模式的本质是创建一个新类，实现与原始类相同的接口，并将原始类的实例存储为一个字段。 不需要修改原始类的行为的方法被转发到原始的类实例。
这种方法的一个缺点是它需要相当大量的样板代码（就像IntelliJ IDEA的IDE具有专门的功能来为您生成代码）。 例如，这是一个装饰器需要多少代码，它实现与Collection一样简单的接口，即使您不修改任何行为：
 <pre><code>
 class DelegatingCollection<T> : Collection<T> {
    private val innerList = arrayListOf<T>()
    override val size: Int
        get() = innerList.size
    override fun contains(element: T): Boolean = innerList.contains(element)
    override fun containsAll(elements: Collection<T>): Boolean =  innerList.containsAll(elements)
    override fun isEmpty(): Boolean = innerList.isEmpty()
    override fun iterator(): Iterator<T> = innerList.iterator()
}
</code></pre>
好消息是，kotlin包括一级支持授权作为一种语言功能。 无论何时实现一个接口，您都可以说使用by关键字将接口的实现委托给另一个对象。 以下是如何使用此方法重写上一个示例：
 <pre><code>
 class DelegatingCollection2<T>(innerList: Collection<T> = ArrayList<T>()) : Collection<T> by innerList {}
</code></pre>
你可以看到，类中的所有方法实现都已经消失了。 编译器将生成它们，实现方式类似于DelegatingCollection示例。
当您需要更改某些方法的行为时，您可以覆盖它们，您的代码将被调用。下面是一个使用委托的示例：
 <pre><code>
 class CountingSet<T>(val innerSet: MutableCollection<T> = HashSet<T>()) : MutableCollection<T> by innerSet {
    var objectsAdded = 0
    override fun add(element: T): Boolean {
        objectsAdded++
        return innerSet.add(element)
    }
    override fun addAll(c: Collection<T>): Boolean {
        objectsAdded += c.size
        return innerSet.addAll(c)
    }
}
fun main(args: Array<String>) {
    val cset = CountingSet<Int>()
    cset.addAll(listOf(1, 1, 2))
    println("${cset.objectsAdded} objects were added, ${cset.size} remain")
}

</code></pre>
## 声明一个类并创建一个与object关键字组合的实例 ##
`object`关键字出现在Kotlin中具有相同的核心思想：该关键字同时定义一个类并创建该类的实例（换句话说，同一个对象）。 我们来看看使用它们的不同情况：

 - 单例模式
 - 伴生对象可以包含与此类相关的工厂方法和其他方法，但不需要调用类实例。 他们的成员可以通过类名访问。
 - 使用对象表达式替代Java匿名内部类。

---
单例：更简单创建单例的方式
 <pre><code>
 object Payroll{
    val allEmployees = arrayListOf<Person>()
    fun calculateSalary() {
        for (person in allEmployees) {
        }
    }
}
</code></pre>
您可以看到，对象声明与object关键字一起引入。 对象声明在单个语句中有效地定义了该类的类和变量。跟普通类一样，object声明可以包含属性，方法，初始化程序块等的声明。唯一的区别是object不允许有构造函数（主要或次要构造函数）。
与常规类的实例不同，object声明都是在定义之前被创建的，而不是通过代码中其他地方的构造函数调用，因此定义一个object的构造函数是没有意义的。
object类中的属性或函数在其他类中如下调用：
 <pre><code>
    Payroll.calculateSalary()
    Payroll.allEmployees.add(Person("tom"))
</code></pre>
object声明也可以从类和接口继承，这通常使用在当您使用的框架需要您实现接口但您的实现不包含任何状态时。
接下来实现一个比较文件路径的比较器：
 <pre><code>
 object CaseInsensitiveFileComparator : Comparator<File> {
    override fun compare(o1: File, o2: File): Int {
        return o1.path.compareTo(o2.path, ignoreCase = true)
    }
}
fun main(args: Array<String>) {
    println(CaseInsensitiveFileComparator.compare(File("/User"), File("/user")))
}
</code></pre>
你可以在普通类的实例中的任意地方使用object对象，例如，可以将此对象作为参数传递给使用比较器的函数：
 <pre><code>
    val files = listOf(File("/Z"), File("/a"))
    println(files.sortedWith(CaseInsensitiveFileComparator))
</code></pre>
在这里，您使用sortedWith函数，该函数返回根据指定的比较器排序的列表。
就像Singleton模式一样，对象声明并不总是适用于大型软件系统。对于具有很少或没有依赖关系的小代码而言，对于与系统的许多其他部分进行交互的大型组件而言，这些代码并不是理想的。 主要原因是您无法控制对象的实例化，并且不能为构造函数指定参数。
这意味着您不能在单元测试或软件系统的不同配置中替换对象本身或对象所依赖的其他类的实现。 如果你需要这个能力，你应该使用常规Kotlin类和一个依赖注入框架，如Guice（https://github.com/google/guice），就像Java一样。
也可以在类中声明object，这样的object对象也只有一个实例。例如：
 <pre><code>
 data class Person(val name: String) {
    object NameComparator : Comparator<Person> {
        override fun compare(o1: Person, o2: Person) = o1.name.compareTo(o2.name)
    }
}
fun main(args: Array<String>) {
    val persons = listOf(Person("Bob"), Person("Alice"))
    println(persons.sortedWith(Person.NameComparator))
}
</code></pre>
在java中使用kotlin的object对象：
Kotlin中的一个对象被编译为具有静态域的类，该静态域保存其单个实例，它始终命名为INSTANCE。 如果您在Java中实现了Singleton模式，那么您可能会手动完成相同的操作。
因此，要使用Java代码中的Kotlin对象，可以访问静态INSTANCE字段：
 <pre><code>
 CaseInsensitiveFileComparator.INSTANCE.compare(file1, file2);
</code></pre>

---
伴生对象：用于工厂方法和静态成员
kotlin类中不能有静态成员;kotlin没有java中的static关键字。
作为替代，Kotlin依赖于包级别的功能（在许多情况下可以替代Java的静态方法）和object对象声明（在其他情况下替换Java静态方法以及静态字段）。
在大多数情况下，建议您使用包级别的功能。 但是包级别的函数不能访问类的私有成员。
因此，如果您需要编写一个可以调用的函数，而不需要一个类实例，但需要访问类的内部，那么可以将其作为对象声明的一个成员编写。 这种功能的一个例子是工厂方法。
在类中使用`companion`建立伴生对象：
 <pre><code>
 class A {
    companion object {
        fun bar() {
            println("Companion object called")
        }
    }
}
fun main(args: Array<String>) {
    A.bar()
}
</code></pre>
创建工厂方法来使用伴生对象是实例：
 <pre><code>
 class User(val name: String) {
    companion object {1.
        fun newSubscribingUser(email: String) = User(email.substringBefore('@'))2.
        fun newFacebookUser(accountId: Int) = User(getFacebookName(accountId))3.
    }
}
</code></pre>

 1. 声明伴生对象
 2. 根据email创建user对象
 3. 通过Facebook帐号创建user对象
你可以通过类名调用伴生对象的方法：
 <pre><code>
 fun main(args: Array<String>) {
    val subscribingUser = User.newSubscribingUser("bob@gmail.com")
    val facebookUser = User.newFacebookUser(4)
}
</code></pre>

---
伴生对象作为常规object
伴生对象是在类中声明的常规对象。 它可以命名，实现一个接口，或者具有扩展功能或属性。
 <pre><code>
class Person(val name: String) {
    companion object Loader {
        fun fromJSON(jsonText: String): Person = ...
    }
}
</code></pre>
你可以使用两种方式调用：
 <pre><code>
  person = Person.Loader.fromJSON("{name: 'Dmitry'}")
  person.name
  person2 = Person.fromJSON("{name: 'Brent'}")
  person2.name
</code></pre>

---
伴生对象接口
 <pre><code>
 interface JSONFactory<T>{
    fun fromJSON(name:String):T
}
class Person(val name:String){
    companion object : JSONFactory<Person> {
        override fun fromJSON(name: String): Person = ...
    }
}
</code></pre>
在java代码中使用伴生对象：
 <pre><code>
 /* Java */
Person.Companion.fromJSON("...")
</code></pre>
如果伴随对象具有名称，则使用此名称而不是Companion
 <pre><code>
 /* Java */
Person.CompanionName.fromJSON("...")
</code></pre>
你可能需要在kotlin代码中使用java的静态成员，可以通过相应成员上的@JvmStatic注释来实现此目的。 如果要声明静态字段，请在顶级属性或对象中声明的属性中使用@JvmField注释。

---
伴生对象拓展
 <pre><code>
 class Person(val firstName: String, val lastName: String){
    companion object {
    }
}
fun Person.Companion.fromJSON(json: String):Person{
    return Person("","")
}
fun main(args: Array<String>) {
    Person.fromJSON(json)
}
</code></pre>

---
匿名内部类
`object`关键字不仅可以用于声明命名的类似单例的对象，还可以用于声明匿名对象。 匿名对象取代了Java对匿名内部类的使用。
<pre><code>
 window.addMouseListener(object : MouseAdapter() {
        override fun mouseClicked(e: MouseEvent) {
// ...
        }

        override fun mouseEntered(e: MouseEvent) {
// ...
        }
    }
    )
</code></pre>
对象表达式声明一个类并创建该类的实例，但它不为类或实例分配名称。因为您将使用该对象作为函数调用中的参数。 如果您需要为对象分配名称，则可以将其存储在变量中：
<pre><code>
val listener = object : MouseAdapter() {
override fun mouseClicked(e: MouseEvent) { ... }
override fun mouseEntered(e: MouseEvent) { ... }
}
</code></pre>
与Java匿名内部类不同，Kotlin匿名对象可以实现多个接口或没有接口（尽管后者不太有用）。
匿名对象不是单例。 每次执行一个对象表达式时，都会创建一个新的对象实例。
就像Java的匿名类一样，对象表达式中的代码可以访问创建它的函数中的变量。您还可以从对象表达式中修改变量的值。
<pre><code>
fun countClicks(window: Window)
{ var clickCount = 0
    window.addMouseListener(object : MouseAdapter()
    { override fun mouseClicked(e: MouseEvent) {
        clickCount++
    }
    })
// ...
}
</code></pre>
注意，当您需要覆盖匿名对象中的多个方法时，对象表达式最为有用。 如果只需要实现单方法接口（如Runnable），则可以依靠Kotlin对SAM转换的支持（将函数文字转换为具有单一抽象方法的接口实现），并将您的实现写为函数 文字（lambda）。

---
## 总结##

 - kotlin接口与java类似，但可以包含默认的实现和属性
 - 默认情况下，类的修饰符都是`final`和`public`
 - 要实现继承（非final），需要将类标记为open
 - `internal`关键字修饰类表示该类在同一模块（module）中可见
 - 默认情况下，嵌套类不是内部类，使用关键字`inner`来储存对外部类的引用
 - 密封类（sealed class）的子类只能写在密封类中，以嵌套类的形式存在
 - 初始化程序块和辅助构造函数为初始化类实例提供了灵活性
 - 在set()，get()方法中可以使用field字段作为属性的备用字段
 - 数据类(data class)让编译器生成的equals（），hashCode（），toString（），copy（）和其他方法。
 - 类委托避免了重复写大量不相关的代码
 - kotlin使用`object`关键字声明单例
 - Companion对象（以及包级别的函数和属性）替代了Java的静态方法和字段定义。
 - 伴随对象与其他对象一样，可以实现接口或具有扩展功能或属性。
 - 对象表达式是Kotlin替代Java的匿名内部类，具有增加的功能，例如实现多个接口并修改创建对象的范围中定义的变量的能力。