# Kotlin-类型系统（十二）
## 可空性 ##
可空性是Kotlin类型系统的一个功能，可帮助您避免NullPointerException错误。 作为程序的客户端，您可能已经看到类似于“发生错误：java.lang.NullPointerException”的错误消息，而没有其他详细信息。 另一个版本是一个消息，如“不幸的是，应用程序X已经停止”，这通常也隐藏了一个NullPointerException异常的原因。
现代语言（包括Kotlin）的方法是将这些问题从运行时错误转换为编译时错误。 通过支持可空性作为类型系统的一部分，编译器可以在编译期间检测许多可能的错误，并减少在运行时抛出异常的可能性。
在本节中，我们将讨论Kotlin中的可空类型：Kotlin如何标记允许为null的值，以及Kotlin提供的工具来处理这些值。 除此之外，我们将介绍关于可空类型混合Kotlin和Java代码的细节。

---
可空类型
Kotlin和Java类型系统之间的第一个也许最重要的区别是Kotlin对可空类型的明确支持。 这是什么意思？ 这是一种指示程序中哪些变量或属性被允许为空的方法。 如果变量可以为null，则调用其上的方法是不安全的，因为它可能导致NullPointerException。 Kotlin不允许这样的呼叫，从而防止许多NullPointerException的发生。
我们来看下面的Java函数：
<pre><code>
	/* Java */
    public static int strLen(String s) {
        return s.length();
    }
</code></pre>
如果使用`null`调用这个函数，则会抛出NullPointerException，在运行的时候崩溃。使用kotlin写这个函数：
<pre><code>
fun strLen(s: String) = s.length
</code></pre>
kotlin中使用null调用该函数是不允许的，编译器会报错：`ERROR: Null can not be a value of a non-null type String`
该参数被声明为String类型，而在Kotlin中，这意味着它必须始终包含一个String实例。 编译器强制执行，所以你不能传递一个包含null的参数。 这样可以保证strLen函数在运行时不会抛出NullPointerException。
如果要使用null作参数，则需要在类型名称后添加一个问号来明确标记：
<pre><code>
fun strLenSafe(s: String?) = ...
</code></pre>
您可以在任何类型之后放一个问号，以表示此类型的值可以存储空引用：String？ ，Int？ ，MyCustomType？ ，等等。
需要注意的是，没有问号的类型表示此类型的变量不能存储空引用。 这意味着默认情况下所有常规类型都不为空，除非明确标记为可空。
当一个值被标记为空之后，您接下来使用该参数的时候可执行的操作被限制，例如，你不能这样调用下列方法：
<pre><code>
fun strLen(s: String?) = s.length
</code></pre>
编译器将会报错，`ERROR: only safe (?.) or non-null asserted (!!.) calls are allowed on a nullable receiver of type kotlin.String?`
您不能将其分配给非空类型的变量：
<pre><code>
    val x: String? = null
    var y: String = x
</code></pre>
编译器将会报错，` Type mismatch: inferred type is String? but String was expected`
那你该怎么办呢？ 最重要的是将其与null进行比较。 一旦执行比较，编译器将会记住该值，并将该值视为在执行检查的范围内为非空值。 例如，此代码完全有效：
<pre><code>
fun strLen(s: String?) {
    if (s != null)
        s.length else 0
}
</code></pre>
如果使用if检查是解决可空性的唯一工具，则您的代码会很快地变得冗长。 幸运的是，Kotlin提供了一些其他工具来帮助更简洁地处理可空值。 

---
类型的含义
其他方法来应对NullPointerException错误
Java有一些工具来帮助解决NullPointerException的问题。 例如，有些人使用注释（例如@Nullable和@NotNull）来表示值的可空性。 有一些工具（例如，IntelliJ IDEA的内置代码检查），可以使用这些注释来检测可以抛出NullPointerException的位置。 但这些工具不是标准Java编译过程的一部分，因此很难确保它们始终如一地应用。 也很难注释整个代码库，包括项目使用的库，以便可以检测到所有可能的错误位置。 我们在JetBrains上的经验表明，即使在Java中广泛使用可空性注释也不能完全解决NPE的问题。
解决这个问题的另一个途径是永远不要在代码中使用空值，并且使用特殊的包装器类型，如Java 8中引入的可选类型，来表示可能被定义或不被定义的值。 这种方法有几个缺点：代码变得更加冗长，额外的包装器实例会影响运行时的性能，并且在整个生态系统中都不一致使用。 即使您在自己的代码中使用“可选”，您仍然需要处理从JDK，Android框架和其他第三方库的方法返回的空值。
现在我们来看看如何使用Kotlin中的可空类型

---
安全调用运算符 ：“?.”
“?.”允许您将空检查和方法调用组合到单个操作中。 例如，表达式s？.toUpperCase（）等同于：`if（s！= null）s.toUpperCase（）else null`，换句话说，如果您尝试调用该方法的值不为null，则方法调用将正常执行。 如果它为空，则跳过该调用。
<pre><code>
fun strLen(s: String?) {
    if (s != null)
        s.length else 0
}
</code></pre>
  安全调用也可用于访问属性，而不仅仅是方法调用。 以下快速示例显示了具有可空属性的简单Kotlin类，并演示了使用安全调用运算符来访问该属性。
  <pre><code>
class Employee(val name: String, val manager: Employee?)
fun managerName(employee: Employee) = employee.manager?.name
&gt;&gt;&gt;println(managerName(developer))
Da Boss
&gt;&gt;&gt;println(managerName(ceo))
null
</code></pre>
  <pre><code>
  class Address(val streetAddress: String, val zipCode: Int,
              val city: String, val country: String)
class Company(val name: String, val address: Address?)
class Person(val name: String, val company: Company?)
fun Person.countryName(): String {
    val country = company?.address?.country
    return if (country != null) country else "Unknown"
}
fun main(args: Array<String>) {
    val person = Person("Dmitry", null)
    println(person.countryName())
}
&gt;&gt;&gt;Unknown
</code></pre>
几个安全访问操作可以链式访问。

---
Elvis 运算符 ：“?:”
Kotlin有一个方便的操作符来提供默认值而不是null。 它被称为Elvis操作员（或者null-coalescing运算符）。
<pre><code>
  fun foo(s: String?) = s ?: ""
</code></pre>
如果“s”为空，则结果为空字符串。
当调用该方法的对象为空时，Elvis运算符通常与安全调用运算符一起使用，以替代null以外的值。 以下是如何使用此模式来简化strLenSafe（）示例：
<pre><code>
  fun foo(s: String?) = s ?: ""
  &gt;&gt;&gt; println(strLenSafe("abc"))
  3
  &gt;&gt;&gt;  println(strLenSafe(null))
  0
</code></pre>
上一节的countryName函数现在也适用于一行：
<pre><code>
fun Person.countryName(): String = company?.address?.country ?: "Unknown"
</code></pre>
让我们看看你如何使用这个操作符来实现一个功能来打印这个人的公司地址的标签。
<pre><code>
class Address(val streetAddress: String, val zipCode: Int,
              val city: String, val country: String)
class Company(val name: String, val address: Address?)
class Person(val name: String, val company: Company?)
fun printShippingLabel(person: Person) {
    val address = person.company?.address ?: throw IllegalArgumentException("No address")
    with(address) {
        println(streetAddress)
        println("$zipCode $city, $country")
    }
}
 &gt;&gt;&gt; printShippingLabel(person)
 Elsestr. 47
80687 Munich, Germany
 &gt;&gt;&gt; printShippingLabel(person)
  java.lang.IllegalArgumentException: No address
</code></pre>

---
安全转换 ：“as?” 
as?运算符尝试将值转换为指定的类型，如果该值没有正确的类型，则返回null。
<pre><code>
class Person(val firstName: String, val lastName: String) {
    override fun equals(other: Any?): Boolean {
        val otherPerson = other as? Person ?: return false 1.
        return otherPerson.firstName == firstName && 2.
                otherPerson.lastName == lastName
    }

    override fun hashCode(): Int = firstName.hashCode() * 37 + lastName.hashCode()
}
</code></pre>

 1. 检查类型，如果不匹配则返回false
 2. 在安全检查之后，变量otherPerson被编译器自动转化为Person类型

---
非空断言 ：“!!”
非空的断言是Kotlin为处理可空类型的值而提供的最简单和最简单的工具。 它由双重感叹号表示，并将任何值转换为非空类型。 对于空值，抛出异常。
<pre><code>
fun ignoreNulls(s:String?){
    val sNotNull: String = s!!
    println(sNotNull.length)
}
&gt;&gt;&gt; ignoreNulls(null)
Exception in thread "main" kotlin.KotlinNullPointerException
</code></pre>

---
“let”函数
假设有下列函数
<pre><code>
fun sendEmailTo(email: String) { /*...*/ }
</code></pre>
您不能将可空类型的值传递给此函数：
<pre><code>
>>> val email: String? = ...
>>> sendEmailTo(email)
ERROR: Type mismatch: inferred type is String? but String was expected
</code></pre>
您必须明确地检查该值是否不为空
<pre><code>
if (email != null) sendEmailTo(email)
</code></pre>
但是你可以另一种方式：使用let函数，并通过安全调用来调用它。 所有的let函数都将它所调用的对象转换为lambda的参数。 如果将其与安全调用语法组合在一起，则可以将您调用let的nullable类型的对象转换为非空类型
只有当电子邮件值非空时，才会调用let函数，因此您可以使用该邮件作为lambda的非空参数：
<pre><code>
fun sendEmailTo(email: String) {
    println("Sending email to $email")
}
&gt;&gt;&gt;var email: String? = "yole@example.com"
&gt;&gt;&gt;email?.let { sendEmailTo(it) }
Sending email to yole@example.com
&gt;&gt;&gt; email = null
&gt;&gt;&gt; email?.let { sendEmailTo(it) }
</code></pre>
因此，你不必为这种类型的值创建一个单独的变量，你可以将以下代码：
<pre><code>
val person: Person? = getTheBestPersonInTheWorld()
if (person != null) sendEmailTo(person.email)
</code></pre>
去掉变量改写成：
<pre><code>
getTheBestPersonInTheWorld()?.let { sendEmailTo(it.email) }
</code></pre>
当您需要检查多个值为null时，可以使用嵌套的let调用来处理它们。 但是在大多数情况下，这样的代码最终会比较冗长，难以跟踪。 通常使用常规if语句可以一起检查所有的值。

---
延迟初始化的属性
许多框架在创建对象实例后调用的专用方法初始化对象。 例如，在Android中，活动初始化在onCreate方法中发生。 JUnit要求您将初始化逻辑放在使用@Before注释的方法中。
但是，如果构造函数中没有初始化器，则不能留下非空属性，并且只能以特殊方法初始化它。 Kotlin通常要求您初始化构造函数中的所有属性，如果属性具有非空类型，则必须提供非空的初始值设定值。 如果您无法提供该值，则必须使用可空类型。 如果这样做，对该属性的每次访问都需要一个空检查或一个!!操作符：
<pre><code>
class MyService {
    fun performAction(): String = "foo"
}

class MyTest {
    private var myService: MyService? = null 1.

    @Before fun setUp() {
        myService = MyService() 2.
    }

    @Test fun testAction() {
        Assert.assertEquals("foo", myService!!.performAction()) 3.
    }
}
</code></pre>

 1. 声明可空类型的属性，使用null初始化它
 2. 在setUp方法中提供一个真正的初始化器
 3. 你必须关心可空性：使用！ 要么 ？。
这看起来很丑陋，特别是如果您多次访问该属性。 要解决这个问题，可以将myService属性声明为较早的初始化。 这是通过应用lateinit修饰符来完成的：
<pre><code>
class MyService {
    fun performAction(): String = "foo"
}

class MyTest {
    private lateinit var myService: MyService? = null 1.

    @Before fun setUp() {
        myService = MyService() 2.
    }

    @Test fun testAction() {
        Assert.assertEquals("foo", myService.performAction()) 3.
    }
}
</code></pre>

 1. 声明不带初始化器的非空类型的属性
 2. 初始化setUp方法中的属性
 3. 访问属性，而不需要额外的空值检查
请注意， late-initialized属性始终为var，因为您需要在构造函数之外更改其值。 但是，您不再需要在构造函数中初始化它，即使该属性具有非空类型。 如果您在初始化之前访问该属性，那么您将收到一个异常`lateinit property myService has not been initialized`

---
可空类性的扩展
定义可空类型的扩展函数是处理空值的另一个强有力的方法。 而不是确保变量在方法调用之前不能为空，您可以允许使用null作为接收方的调用，并在函数中处理null。 这仅适用于扩展功能; 通过对象实例调度常规方法调用，因此在实例为空时不能执行：
<pre><code>
fun String.isNullOrBlank(): Boolean = this == null || this.isBlank()
</code></pre>
请注意，我们之前讨论的let函数也可以在可空接收器上调用，但不会检查null的值。 如果在不使用安全呼叫运算符的情况下以空类型调用它，则lambda参数也将为空。因此，如果要使用let检查非空的参数，则必须使用safe-call操作符？ 

---
类型参数的可空性
默认情况下，Kotlin中的函数和类的所有类型参数都为空。 类型参数可以替换任何类型，包括可空类型; 在这种情况下，使用type参数作为类型的声明允许为空，即使类型参数T不以问号结尾。 请考虑以下示例：
<pre><code>
fun <T> printHashCode(t: T) {
    println(t?.hashCode()) 1.
}
fun main(args: Array<String>) {
    printHashCode(null) 2.
}
</code></pre>

 1. 您必须使用安全呼叫，因为“t”可能为空。
 2. “T”被推定为“Any”。
要使类型参数不为null，您需要为其指定非空的上限。 这将拒绝作为参数的可空值：
<pre><code>
fun <T : Any> printHashCode(t: T) {
    println(t.hashCode())
}
&gt;&gt;&gt; printHashCode(null)
Error: Type parameter bound for `T` is not satisfied
&gt;&gt;&gt; printHashCode(42)
42
</code></pre>

---
可空性和 Java 
首先，正如我们提到的，有时Java代码包含有关可空性的信息，使用注释表示。 当这个信息存在于代码中时，Kotlin使用它。
因此，Java中的@Nullable String被视为String？ ，@NotNull String视为String
<pre><code>
/* Java */
public class Person {
    private final String name;

    public Person(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }
}
</code></pre>
getName可以返回null吗？ 在这种情况下，Kotlin编译器不知道String类型的可空性，所以你必须自己处理它。 如果您确定name不为null，则可以像通常的方式一样通过Java取消引用它，而不需要额外的检查。 但是下面是例外的情况：
<pre><code>
fun yellAt(person: Person) {
    println(person.name.toUpperCase() + "!!!")
}
java.lang.IllegalArgumentException: Parameter specified as non-null is null: method toUpperCase, parameter $receiver
</code></pre>
toUpperCase（）调用的接收者person.name为null，因此抛出异常。使用Java API时要小心。 大多数库没有注释，所以您可以将所有类型解释为非空，但这可能会导致错误。 为了避免错误，您应该检查您使用的Java方法的文档。
在Kotlin中覆盖Java方法时，您可以选择是否将参数和返回类型声明为可空或非空。 例如：
<pre><code>
/* Java */
interface StringProcessor {
    void process(String value);
}
</code></pre>
在Kotlin中，以下两个实现将被编译器接受：
<pre><code>
class StringPrinter : StringProcessor {
    override fun process(value: String) {
        println(value)
    }
}

class NullableStringPrinter : StringProcessor {
    override fun process(value: String?) {
        value.let { println(it) }
    }
}
</code></pre>
## 基本数据类型和其他基本类型 ##
 与Java不同，Kotlin不区分原始类型和包装器。
 基本数据类型 ：Int、Boolean 及其他
 如你所知，Java对原始类型和引用类型进行了区分。 原始类型的变量（如int）直接保存其值。 引用类型的变量（如String）包含对包含对象的内存位置的引用。
 原始类型的值可以更有效地存储和传递，但是您不能调用此类值的方法或将它们存储在集合中。 Java提供了在需要对象的情况下封装原始类型的特殊封装类型（如java.lang.Integer）。 因此，要定义整数的集合，您不能说集合<int>; 您必须使用Collection &lt;Integer>。
 Kotlin不区分原始类型和包装类型。 您始终使用相同的类型（例如Int）：
 <pre><code>
val i: Int = 1
val list: List&lt;Int> = listOf(1, 2, 3)
</code></pre>
这很方便。 此外，您可以调用数字类型值的方法。 例如，考虑下列代码，它使用coerceIn标准库函数将该值限制在指定的范围内：
 <pre><code>
 fun showProgress(progress: Int) {
    val percent = progress.coerceIn(0, 100)
    println("We're $percent% done!")
}
&gt;&gt;&gt;println(showProgress(114))
We're 100% done!
</code></pre>
如果原始和引用类型是相同的，那意味着Kotlin表示所有数字作为对象？ 这不会是非常低效吗？ 事实上,kotlin并不会这么做。
在运行时，数字类型以最有效的方式表示。
在大多数情况下 - 对于变量，属性，参数和返回类型 - Kotlin的Int类型被编译为Java原语类型int。 唯一不可能的情况是通用类，如集合。 用作泛型类型参数的原始类型将被编译为相应的Java包装器类型。
例如，如果Int类型用作集合的类型参数，则集合将存储java.lang.Integer的实例，相应的包装器类型。
与Java基本类型对应的完整的类型列表如下所示：

 - 整数类型— Byte , Short , Int , Long
 - 浮点数类型— Float , Double
 - 字符类型— Char
 - 布尔类型— Boolean

.---
可空的基本数据类型 ：Int?、Boolean? 及其他
Kotlin中的Nullable类型不能由Java原语类型表示，因为null只能存储在Java引用类型的变量中。 这意味着每当在Kotlin中使用原始类型的可空版本时，它将被编译为相应的包装器类型。
例如，查询一个人年纪更大的例子：
 <pre><code>
 data class Person(val name: String, val age: Int? = null) {
    fun isOlderThan(other: Person): Boolean? {
        if (age == null || other.age == null) return null
        return age > other.age
    }
}
&gt;&gt;&gt; println(Person("Sam", 35).isOlderThan(Person("Amy", 42)))
false
&gt;&gt;&gt;println(Person("Sam", 35).isOlderThan(Person("Jane")))
null
</code></pre>
请注意在这里适用的常规可空性规则。 你不能只比较Int的两个值？ ，因为其中一个可能为空。 相反，您必须检查这两个值都不为空。 之后，编译器允许您正常使用它们。
在Person类中声明的age属性的值存储为java.lang.Integer。 但是，如果您正在使用Java中的类，这个细节才是重要的。 要在Kotlin中选择正确的类型，您只需要考虑null是变量或属性的可能值。

---
数字转换
Kotlin和Java之间的一个重要区别就是处理数值转换的方式。 Kotlin不会自动将数字从一种类型转换为另一种类型，即使其他类型较大。 例如，以下代码将不会在Kotlin中编译：
<pre><code>
val i = 1
val l: Long = i
Error: type mismatch
</code></pre>
相反，您需要明确应用转换：
<pre><code>
val i = 1
val l: Long = i.toLong()
</code></pre>
转换函数是为每个基本类型（Boolean除外）定义的：toByte（），toShort（），toChar（）等等。 这些函数支持在两个方向上进行转换：将较小的类型扩展为较大的类型，如Int.toLong（），并将较大的类型截断为较小的类型，如Long.toInt（）
如果您在代码中同时使用不同的数字类型，则必须显式转换变量以避免意外行为。
请注意，当您编写数字字面值时，通常不需要使用转换函数。 一种可能性是使用特殊语法来明确地标记常量的类型，如42L或42.0f。 即使不使用它，如果使用数字字面值来初始化已知类型的变量，或者将其作为参数传递给方法，则自动应用必要的转换。 此外，算术运算符重载以接受所有适当的数字类型。
例如，以下代码正常工作，没有任何明确的转换：
<pre><code>
fun foo(l: Long) = println(l)
&gt;&gt;&gt; val b: Byte = 1
&gt;&gt;&gt; val l = b + 1L
&gt;&gt;&gt; foo(42)
42
</code></pre>
Kotlin标准库提供了一组类似的扩展函数，用于将字符串转换为基本类型（toInt，toByte，toBoolean等）：
<pre><code>
>>> println("42".toInt())
42
</code></pre>
这些函数中的每一个都尝试将字符串的内容解析为相应的类型，如果解析失败，则会抛出NumberFormatException。

---
“Any”和“Any?”：根类型
类似于Object是Java中类层次结构的根，Any类型是Kotlin中所有不可为空的类型的超类型。 但是在Java中，Object只是所有引用类型的超类型，而原始类型不是层次结构的一部分。 这意味着每当需要Object时，您必须使用包装器类型，例如java.lang.Integer。 在Kotlin中，Any是所有类型的超类型，包括基本类型，如Int
请注意，Any是不可为空的类型，所以类型为Any的变量不能保存值为null。 如果您需要一个可以在Kotlin中保存任何值的变量，包括null，则必须使用Any？ 类型。
所有Kotlin类都有以下三种方法：toString（），equals（）和hashCode（）。 这些方法是从Any继承的。 在java.lang.Object上定义的其他方法（如wait和notify）在Any上不可用，但是如果手动将值转换为java.lang.Object，则可以调用它们。

---
Unit 类型 ：Kotlin 的“void”
Kotlin中的Unit类型与Java中的void相同。 它可以用作没有返回类型的函数：
<pre><code>
fun f(): Unit { ... }
</code></pre>
在语法上，它和java同样可以省略：
<pre><code>
fun f() { ... }
</code></pre>
那么Kotlin的Unit与Java的void有何区别呢？
Unit是一个完整的类型，并且，与void不同，它可以用作类型参数。当您覆盖返回泛型参数并使其返回Unit类型的值的函数时：
<pre><code>
interface Processor<T> {
    fun progress(): T
}
class NoResultProcessor:Processor<Unit>{
    override fun progress() {1.
			2.
    }
}
</code></pre>

 1. 返回单位，但省略类型说明
 2. 你不需要明确的返回值

---
Nothing 类型 ：“这个函数永不返回” 
对于Kotlin的一些功能，“返回值”的概念没有任何意义，因为它们从未成功完成。
当分析调用这样的函数的代码时，知道该函数将永远不会正常终止是有用的。 为了表示这一点，Kotlin使用了一种叫做Nothing的特殊返回类型：
<pre><code>
fun fail(message:String):Nothing{
    throw IllegalAccessException(message)
}
&gt;&gt;&gt; fail("Error occurred")
java.lang.IllegalStateException: Error occurred
</code></pre>
请注意，返回Nothing的功能可以在Elvis操作员的右侧使用，以执行前提条件检查：
<pre><code>
    val address = company.address ?: fail("No address")
    println(address.city)
</code></pre>
## 集合与数组 ##
您已经看到使用各种集合API的许多代码示例，并且您知道Kotlin构建在Java集合库中，并通过扩展功能添加的功能来增强它。
可空性和集合
Kotlin完全支持类型参数的可空性。 就像变量的类型可以有一个？ 附加的字符表示变量可以保持为null，用作类型参数的类型可以以相同的方式进行标记。
要看看它是如何工作的，我们来看一个函数的例子，该函数从一个文件中读取行列表，并尝试将每一行解析为一个数字：
<pre><code>
fun readNumber(reader: BufferedReader): List&lt;Int?> {
    val result = ArrayList&lt;Int?>() 1.
    for (line in reader.lineSequence()) {
        try {
            val number = line.toInt()
            result.add(number) 2.
        } catch(e: NumberFormatException) {
            result.add(null) 3.
        }
    }
    return result
}
</code></pre>

 1. 创建可空的Int值的列表
 2. 向列表中添加一个整数（非空值）
 3. 将null添加到列表中，因为当前行不能被解析为整数
您可以看到，List &lt;Int？>是一个可以容纳Int类型值的列表 ：换句话说，Int或null。
要查看如何处理可空值的列表，我们来编写一个函数来将所有有效数字加在一起，并分别对无效数进行计数：
<pre><code>
fun addValidNumbers(numbers: List&lt;Int?>) {
    var sumOfValidNumbers = 0
    var invalidNumbers = 0
    for (number in numbers) {
        if (number != null)
            sumOfValidNumbers += number
        else
            invalidNumbers++
    }
    println("Sum of valid numbers: $sumOfValidNumbers")
    println("Invalid numbers: $invalidNumbers")
}
&gt;&gt;&gt;val reader = BufferedReader(StringReader("1\nabc\n42"))
&gt;&gt;&gt;val numbers = readNumbers(reader)
&gt;&gt;&gt; addValidNumbers(numbers)
Sum of valid numbers: 43
Invalid numbers: 1
</code></pre>
集合过滤null值是很常见的操作，因此kotlin库提供了`filterNotNull`来执行它，以下为简化的例子：
<pre><code>
fun addValidNumbers2(numbers: List&lt;Int?>){
    val validNumbers = numbers.filterNotNull()
    println("Sum of valid numbers: ${validNumbers.sum()}")
    println("Invalid numbers: ${numbers.size - validNumbers.size}")
}
</code></pre>
当然，过滤也会影响集合的类型。 validNumbers的类型为List <Int>，因为过滤确保集合不包含任何空元素。

---
只读集合与可变集合
将Kotlin的集合设计从Java中分离出来的一个重要特征是它分离了用于访问集合中的数据和修改数据的接口。 这种区别起始于使用集合kotlin.Collection的最基本的接口。 使用此接口，可以迭代集合中的元素，获取其大小，检查它是否包含某个元素，以及执行从集合读取数据的其他操作。 但这个界面没有添加或删除元素的任何方法。
如果需要修改集合中的数据，可以使用kotlin.MutableCollection接口。 它扩展了常规kotlin.Collection并提供了添加和删除元素，清除集合等的方法
作为一般规则，您应该在代码中使用只读集合。 仅当代码将修改集合时才使用可变集合。
例如，您可以清楚地看到，以下copyElements函数将修改目标集合而不是源集合：
<pre><code>
fun <T> copyElements(source: Collection<T>, target: MutableCollection<T>) {
    for (item in source){ 1.
        target.add(item) 2.
    }
}
&gt;&gt;&gt;val source: Collection<Int> = arrayListOf(3, 5, 7)
&gt;&gt;&gt; val target: MutableCollection<Int> = arrayListOf(1)
&gt;&gt;&gt; copyElements(source, target)
&gt;&gt;&gt;println(target)
[1, 3, 5, 7]
</code></pre>

 1. 循环遍历源集合中的所有项目
 2. 将项目添加到可变目标集合

---
Kotlin 集合和 Java
下图显示了显示了可用于创建不同类型的集合的函数：
![集合创建函数](https://github.com/aheven/holt/blob/master/image/%E9%9B%86%E5%90%88%E5%88%9B%E5%BB%BA%E5%87%BD%E6%95%B0.png)
当您需要调用Java方法并将集合作为参数传递时，可以直接执行此操作，无需任何额外的步骤。
例如，如果您有一个以java.util.Collection为参数的Java方法，则可以将任何Collection或MutableCollection值作为参数传递给该参数。
如果您正在编写一个收集并将其传递给Java的Kotlin函数，那么您有责任为参数使用正确的类型，具体取决于您调用的Java代码是否会修改集合。

---
作为平台类型的集合
第一个示例中，Java接口表示处理文件中的文本的对象：
<pre><code>
public interface FileContentProcessor {
    void processContents(File path, byte[] binaryContents, List<String> textContents);
}
</code></pre>
这个接口的Kotlin实现需要做出以下选择：

 - 该列表将为空，因为一些文件是二进制文件，并且它们的内容不能被表示为文本。
 - 列表中的元素不为空，因为文件中的行从不为空。
 - 列表将是只读的，因为它表示文件的内容，这些内容不会被修改。
这个实现应该这么写：
<pre><code>
class FileIndexer:FileContentProcessor {
    override fun processContents(path: File, binaryContents: ByteArray?, textContents: MutableList<String>?) {
    }
}
</code></pre>
 在这里，接口的实现将文本表单中的一些数据解析成对象列表，将这些对象附加到输出列表，并通过将消息添加到单独的列表来分析时报告检测到的错误：
 <pre><code>
 public interface DataParser<T> {
    void parseData(String input, List<T> output, List<String> errors);
}
</code></pre>
在这种情况下的选择是不同的：
 - 列表<String>将不为空，因为调用者总是需要接收错误消息。
 - 列表中的元素将为空，因为输出列表中的每个项目都不会有相关的错误消息。
 - 列表<String>将是可变的，因为实现代码需要添加元素。
 <pre><code>
 class PersonParser:DataParser<Person>{
    override fun parseData(input: String, output: MutableList<Person>, errors: MutableList&lt;String?>) {
    }
}
</code></pre>
注意同样的Java类型 - List <String>是由两种不同的Kotlin类型表示的：List &lt;String>？ （可空的字符串列表），另一个是MutableList &lt;String？>（可变字符串的可变列表）。

---
对象和基本数据类型的数组
 <pre><code>
 fun main(args: Array<String>) {
    for (i in args.indices){
        println("Argument $i is: ${args[i]}")
    }
}
</code></pre>
正如你所看到的，Kotlin中的一个数组是一个类型参数，类型被指定为相应的类型参数。
要在Kotlin中创建一个数组，您有以下几种可能性：

 - arrayOf（）函数创建一个包含指定为该函数参数的元素的数组。
 - arrayOfNulls（）函数创建一个包含null元素的给定大小的数组。当然，它只能用于创建元素类型为空的数组;
 - Array（）构造函数接受数组的大小和一个lambda函数，并通过调用lambda来初始化每个数组元素。 这是如何使用非空元素类型初始化数组，而不必明确地传递每个元素。
作为一个简单的例子，下面是如何使用Array（）函数创建一个字符串数组从“a”到“z”
<pre><code>
&gt;&gt;&gt; val letters = Array(26) { i -> ('a' + i).toString() }
&gt;&gt;&gt; println(letters.joinToString(","))
a,b,c,d,e,f,g,h,i,j,k,l,m,n,o,p,q,r,s,t,u,v,w,x,y,z
</code></pre>
在Kotlin代码中创建数组的最常见的情况之一是当您需要调用采用数组的Java方法时，或者使用带有vararg参数的Kotlin方法。 在这些情况下，您通常将数据存储在集合中，您只需将其转换为数组即可。 您可以使用toTypedArray（）方法来执行此操作：
<pre><code>
&gt;&gt;&gt;  val strings = listOf("a", "b", "c")
&gt;&gt;&gt;  println("%s/%s/%s".format(*strings.toTypedArray()))
a/b/c
</code></pre>
 - 当预期出现vararg参数时，扩展运算符（*）用于传递数组。
 与其他类型一样，数组类型的类型参数始终成为对象类型。 因此，如果您声明像Array <Int>，它将成为一个数组的整数（它的Java类型将是java.lang.Integer []）。 如果您需要创建一个不使用包装的原始数据类型的数组，那么您必须对原始类型的数组使用一个专门的类。
为了表示原始类型的数组，Kotlin提供了一些单独的类，每个类型一个。 例如，int类型的值的数组称为IntArray。 对于其他类型，Kotlin提供ByteArray，CharArray，BooleanArray等。 所有这些类型都被编译为常规的Java基元类型数组，如int []，byte []，char []等等。 因此，这种数组中的值以最有效的方式可以无需装箱即可存储。
要创建一个基本类型的数组，您有以下选项：
 - 该类型的构造函数需要一个size参数，并返回一个使用相应原始类型（通常为零）的默认值初始化的数组。
 - 工厂函数（IntArray的intArrayOf等等对于其他数组类型）将可变数量的值作为参数，并创建一个包含这些值的数组。
 - 另一个构造函数需要一个大小和一个lambda来初始化每个元素。
以下是前两个选项用于创建一个包含五个零的整数数组：
<pre><code>
	val fiveZeros = IntArray(5)
    val fiveZerosToo = intArrayOf(0, 0, 0, 0, 0)
</code></pre>
以下是如何使用接受lambda的构造函数：
<pre><code>
&gt;&gt;&gt;   val squares = IntArray(5) { i -> (i+1) * (i+1) }
&gt;&gt;&gt;   println(squares.joinToString())
1, 4, 9, 16, 25
</code></pre>
或者，如果您有一个包含原始类型的框的值的数组或集合，则可以使用相应的转换函数将它们转换为该基元类型的数组，例如toIntArray（）
接下来，我们来看一下你可以用数组做的一些事情。 除了基本操作（获取数组的长度和获取和设置元素）之外，Kotlin标准库还支持与集合一样的数组扩展函数。
您在第5章中看到的所有功能（过滤器，映射等）都适用于数组，包括基本类型的数组。 （请注意，这些函数的返回值是列表，而不是数组。）
我们来看看我们如何使用forEachIndexed函数和一个lambda来重写本节的初始示例。 为数组的每个元素调用传递给该函数的lambda，并接收两个参数：元素的索引和元素本身。
<pre><code>
fun main(args: Array<String>) {
    args.forEachIndexed { index, element -> println("Argument $index is: $element") }
}
</code></pre>
## 总结 ##
 - Kotlin在编译时检测到可能的NullPointerException错误。
 - Kotlin提供诸如安全呼叫（？），Elvis操作符（？：），not- null assertions（!!）以及简单地处理可空类型的let函数等工具。
 -  as?运算符提供了一种简单的方法来将值转换为类型，并在具有不同类型的情况下处理该值。
 - 来自Java的类型被解释为Kotlin中的平台类型，允许开发人员将它们视为可空或非空
 - 表示基本数字的类型（如Int），通常编译为Java原始类型。
 - 可空的原始类型（如Int？）对应于Java中的包装原始类型（如java.lang.Integer）。
 - Any类型是所有其他类型的超类型，类似于Java的Object。 Unit类似于void。
 - Nothing类型用作不能正常终止的函数的返回类型。
 - Kotlin使用标准Java类作为集合，并通过区分只读和可变集合来增强它们。
 - 当您扩展Java类或在Kotlin中实现Java接口时，您需要仔细考虑参数的可空性和可变性。
 - Kotlin的Array类看起来像一个常规的泛型类，但它被编译为Java数组。
 - 原始类型的数组由IntArray等特殊类表示。