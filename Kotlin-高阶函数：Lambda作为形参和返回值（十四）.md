## Kotlin-高阶函数 ：Lambda 作为形参和返回值（十四） ##
Lambda是建立抽象的一个很好的工具，当然他们的力量不限于标准库中的集合和其他类。 在本章中，您将学习如何创建高阶函数 - 您自己的将lambda作为参数或返回它们的函数。 您将看到高阶函数如何帮助您简化代码，删除代码重复以及构建漂亮的抽象。 您还将熟悉内联函数 - 强大的Kotlin功能，可以消除与使用lambdas相关的性能开销，并在lambda表达式中实现更灵活的控制流。

---
声明高阶函数
本章的关键新思想是高阶函数的概念。 根据定义，高阶函数是将另一个函数作为参数或返回一个函数的函数。 在Kotlin中，可以使用lambdas或函数引用将函数表示为值。 因此，高阶函数是可以将lambda函数或函数引用作为参数传递给的任何函数，也可以是返回一个或两个函数的函数。
例如，过滤器标准库函数将一个谓词函数作为一个参数，因此是一个高阶函数：
<pre><code>
list.filter { x > 0 }
</code></pre>

---
函数类型
为了声明一个将lambda作为参数的函数，你需要知道如何声明这样一个参数的类型。 在我们开始之前，让我们看一个更简单的情况，并将lambda存储在局部变量中。 您已经看到了如何在不声明类型的情况下依靠Kotlin的类型推断来做到这一点：
<pre><code>
val sum = { x: Int, y: Int -> x + y }
val action = { println("42") }
</code></pre>
在这种情况下，编译器推断sum和action变量都具有函数类型。 现在让我们来看看这些变量的显式类型声明是什么样的：
<pre><code>
val sum1: (Int, Int) -> Int = { x, y -> x + y }
val action1: () -> Unit = { println("42") }
</code></pre>

 1. 采用两个Int参数并返回一个Int值的函数
 2. 不带参数且不返回值的函数
要声明一个函数类型，可以将函数参数类型放在括号中，后跟一个箭头和函数的返回类型：
<pre><code>
(Int,String)->Unit
</code></pre>
当你声明一个常规函数时，Unit返回类型可以省略，但是函数类型声明总是需要一个明确的返回类型，所以你不能在这个上下文中忽略Unit。
注意如何在lambda表达式{x，y - > x + y}中省略参数x，y的类型。 因为它们是作为变量声明的一部分在函数类型中指定的，所以不需要在lambda本身中重复它们。
就像使用其他函数一样，函数类型的返回类型可以被标记为空（null）：
<pre><code>
var canReturnNull: () -> Int? = { null }
</code></pre>
你也可以定义一个函数类型的可为空的变量。 要指定变量本身（而不是函数的返回类型）可以为空，则需要将整个函数类型定义括在括号中，并将问号放在括号后面：
<pre><code>
var funOrNull: (() -> Int)? = null
</code></pre>

---
函数类型的参数名称
您可以为函数类型的参数指定名称：
<pre><code>
fun performRequest(url: String,
                   callback: (code: Int, content: String) -> Unit) {
    /*...*/
}
>>>val url = "http://kotl.in"
>>>performRequest(url) { code, content -> /*...*/ }
>>>performRequest(url) { code, page -> /*...*/ }
</code></pre>
参数名称不影响类型匹配。 在声明lambda时，不必使用与函数类型声明中使用的名称相同的参数名称。 但是这些名称提高了代码的可读性。

---
调用作为参数的函数
现在你已经知道如何声明一个更高阶的函数了，接下来我们来讨论如何实现一个函数。
第一个示例尽可能简单，并使用与前面看到的总和lambda相同的类型声明。 该函数对两个数字2和3执行任意操作，并打印结果。
<pre><code>
fun twoAndThree(operation: (Int, Int) -> Int) { 1.
    val result = operation(2,3) 2.
    println("The result is $result")
}
>>>twoAndThree { a, b -> a + b }
The result is 5
>>> twoAndThree { a, b -> a * b }
The result is 6
</code></pre>

 1. 声明一个函数类型的参数
 2. 调用函数类型的参数
调用作为参数传递的函数的语法与调用常规函数的语法相同：将括号放在函数名称后面，并将参数放在括号内。
作为一个更有趣的例子，让我们重新实现一个最常用的标准库函数：过滤函数。 为了简单起见，您将在String上实现筛选器函数，但是对任何元素的集合起作用的通用版本是相似的。 其声明如下：
<pre><code>
fun String.filter(predicate: (Char) -> Boolean): String{}
</code></pre>
filter函数将predicate作为参数。 predicate的类型是一个函数，它接受一个字符参数并返回一个布尔结果。 如果传递给predicate的字符需要出现在结果字符串中，则结果为true，否则为false。
下面是该功能的实现：
<pre><code>
fun String.filter(predicate: (Char) -> Boolean): String {
    val sb = StringBuilder()
    for (index in 0 until length) {
        val element = get(index)
        if (predicate(element)) { 1.
            sb.append(element)
        }
    }
    return sb.toString()
}
>>>println("ab1c".filter { it in 'a'..'z' }) 2.
abc
</code></pre>
 1. 调用作为“predicate参数传递的函数
 2. 传递一个lambda作为“predicate”的参数
过滤函数的实现非常简单。 它检查每个元素是否满足谓词，并在成功时将其添加到包含结果的StringBuilder中。

---
在 Java 中使用函数类
在Java中，函数类型被声明为常规接口。 它们中有很多，对应于不同数量的函数参数：Function0 &lt;R>（该函数不带任何参数），Function1 &lt;P1，R>（该函数带一个参数），依此类推。
使用函数类型的Kotlin函数可以很容易地从Java调用。 Java 8 lambda会自动转换为函数类型的值：
<pre><code>
fun processTheAnswer(f:(Int)->Int){
    println(f(42))
}
/* Java */
>>> processTheAnswer(number -> number + 1);
</code></pre>
在较旧的Java版本中，可以从相应的函数接口传递实现invoke方法的匿名类的实例：
<pre><code>
/* Java */
>>> processTheAnswer(new Function1&lt;Integer, Integer>() {
            @Override
            public Integer invoke(Integer number) {
                return number+1;
            }
        });
</code></pre>
在Java中，您可以轻松使用Kotlin标准库中的扩展函数，该函数将lambdas作为参数。 但是请注意，它们看起来不像Kotlin那么好 - 你必须明确地传递一个接收者作为第一个参数：
<pre><code>
/* Java */
    public static void main(String[] args){
        List<String> strings = new ArrayList&lt;>();
        strings.add("42");
        CollectionsKt.forEach(strings, new Function1&lt;String, Unit>() { 1.
            @Override
            public Unit invoke(String s) {
                System.out.println(s);
                return Unit.INSTANCE; 2.
            }
        });
    }
</code></pre>

 1. 您可以使用Java代码中的Kotlin标准库中的函数。
 2. 您必须显式返回Unit类型的值。
在Java中，你的函数或lambda可以返回Unit。 但是因为Unit类型在Kotlin中有一个值，所以你需要明确的返回它。 你不能传递返回void的lambda作为返回Unit的函数类型的参数。

---
函数类型的参数默认值和 null 值
当你声明一个函数类型的参数时，你也可以指定它的默认值。
<pre><code>
fun &lt;T> Collection&lt;T>.joinToString(separator: String = ",",
                                   prefix: String = "",
                                   postfix: String = ""
): String {
    val result = StringBuilder(prefix)
    for ((index, element) in this.withIndex()) {
        if (index > 0) result.append(separator)
        result.append(element)
    }
    result.append(postfix)
    return result.toString()
}
</code></pre>
这个实现是灵活的，但它不能让您控制转换的一个关键方面：集合中的单个值如何转换为字符串。 代码使用StringBuilder.append（o：Any？），它总是使用toString方法将对象转换为字符串。 这在许多情况下是好的，但并非总是如此。 您现在知道您可以传递一个lambda来指定如何将值转换为字符串。 但是要求所有的调用者通过这个lambda会很麻烦，因为他们中的大多数都是默认行为。 为了解决这个问题，你可以定义一个函数类型的参数，并为其指定一个默认值。
<pre><code>
fun &lt;T> Collection&lt;T>.joinToString(separator: String = ",",
                                   prefix: String = "",
                                   postfix: String = "",
                                   transform: (T) -> String = { it.toString() } 1.
): String {
    val result = StringBuilder(prefix)
    for ((index, element) in this.withIndex()) {
        if (index > 0) result.append(separator) 2.
        result.append(transform(element))
    }
    result.append(postfix)
    return result.toString()
}
>>> val letters = listOf("Alpha", "Beta")  3.
>>>> println(letters.joinToString())
Alpha, Beta
>>> println(letters.joinToString { it.toUpperCase() }) 4.
ALPHA, BETA
</code></pre>

 1. 使用默认值声明函数类型的参数
 2. 调用作为参数传递的函数
 3. 使用默认的转换功能
 4. 使用命名的参数语法来传递lambda参数
请注意，这个函数是通用的：它有一个类型参数T表示集合中元素的类型。 变换lambda将收到该类型的参数。
声明一个函数类型的默认值不需要特殊的语法，只需要将该值作为=符号后面的lambda表达式即可。
另一种方法是声明一个可为空的函数类型的参数。 请注意，您不能直接调用传递给这个参数的函数：Kotlin会拒绝编译这样的代码，因为它在这种情况下检测到空指针异常的可能性。 一种选择是明确地检查null：
<pre><code>
fun foo(callback: (() -> Unit)?) {
// ...
    if (callback != null) {
        callback()
    }
}
</code></pre>
java较早的版本利用了函数类型的变量是FunctionN接口的实现。 Kotlin标准库定义了一系列接口（Function0，Function1等），表示具有相应参数个数的函数。 每个接口定义一个invoke方法，调用它将执行该函数。 函数类型的变量是实现相应FunctionN接口的类的一个实例，调用方法包含lambda的主体。 作为常规方法，可以通过安全调用语法来调用invoke：callback？.invoke（）
以下是如何使用这种技术来重写joinToString（）函数：
<pre><code>
fun &lt;T> Collection&lt;T>.joinToString(separator: String = ",",
                                   prefix: String = "",
                                   postfix: String = "",
                                   transform: ((T) -> String)? = null
): String {
    val result = StringBuilder(prefix)
    for ((index, element) in this.withIndex()) {
        if (index > 0) result.append(separator)
        val str = transform?.invoke(element) ?: element.toString()
        result.append(str)
    }
    result.append(postfix)
    return result.toString()
}
</code></pre>

---
返回函数的函数
从另一个函数返回一个函数的要求并不像将函数传递给其他函数那样频繁，但是它仍然是有用的。 例如，设想一个程序中的一个逻辑块，可以根据程序的状态或其他条件而变化，例如根据所选的运输方式计算运输成本。 您可以定义一个函数来选择适当的逻辑变体，并将其作为另一个函数返回。 下面是代码：
<pre><code>
enum class Delivery {
    STANDARD, EXPEDITED
}

class Order(val itemCount: Int)

fun getShippingCostCalculator(delivery: Delivery): (Order) -> Double {
    if (delivery == Delivery.EXPEDITED) {
        return { order -> 6 + 2.1 * order.itemCount }
    }

    return { order -> 1.2 * order.itemCount }
}
>>>val calculator = getShippingCostCalculator(Delivery.EXPEDITED)
>>>println("Shipping costs ${calculator(Order(3))}")
Shipping costs 12.3
</code></pre>
要声明一个返回另一个函数的函数，可以指定一个函数类型作为它的返回类型。 getShippingCostCalculator返回一个函数，它接受一个Order并返回一个Double。 要返回一个函数，你需要写一个返回表达式，后跟一个lambda表达式，一个方法引用，或者一个函数类型的另一个表达式，比如一个局部变量。
让我们看看另一个例子，从函数返回函数是有用的。
假设您正在使用GUI联系人管理应用程序，并且您需要根据UI的状态确定应显示哪些联系人。 假设用户界面允许您键入一个字符串，然后只显示名称以该字符串开头的联系人; 它还可以让您隐藏没有指定电话号码的联系人。 您将使用ContactListFilters类来存储选项的状态。
<pre><code>
class ContactListFilters {
    var prefix: String = ""
    var onlyWithPhoneNumber: Boolean = false
}
</code></pre>
要从过滤用户界面中分离联系人列表显示逻辑，可以定义一个函数来创建用于过滤联系人列表的prefix。 该prefix检查前缀，并在需要时检查电话号码是否存在：
<pre><code>
class ContactListFilters {
    var prefix: String = ""
    var onlyWithPhoneNumber: Boolean = false

    fun getPredicate(): (Person) -> Boolean { 1.
        val startsWithPrefix = { p: Person
            ->
            p.firstName.startsWith(prefix)
        }
        if (!onlyWithPhoneNumber) {
            return startsWithPrefix 2.
        }
        return {
            startsWithPrefix(it)
                    && it.phoneNumber != null 3.
        }
    }
}

data class Person(
        val firstName: String,
        val lastName: String,
        val phoneNumber: String?
)
>>>val contacts = listOf(Person("Dmitry", "Jemerov", "123-4567"),
            Person("Svetlana", "Isakova", null))
>>>val contactListFilters = ContactListFilters()
>>>with(contactListFilters) {
        prefix = "Dm"
        onlyWithPhoneNumber = true
    }
>>>println(contacts.filter(contactListFilters.getPredicate())) 4.
[Person(firstName=Dmitry, lastName=Jemerov, phoneNumber=123-4567)]
</code></pre>

 1. 声明一个返回函数的函数
 2. 返回一个函数类型的变量
 3. 从这个函数返回一个lambda
 4. 将getPredicate返回的函数作为参数传递给“filter”
正如你所看到的，getPredicate方法返回一个函数值，作为参数传递给过滤函数。 Kotlin函数类型允许您像使用其他类型（如字符串）的值一样容易地执行此操作。
高阶函数为您提供了一个非常强大的工具来改进代码的结构并消除重复。 让我们看看lambdas如何帮助从代码中提取重复的逻辑。

---
通过 lambda 去除重复代码
函数类型和lambda表达式一起构成了一个创建可重用代码的好工具。 现在可以通过使用简洁的lambda表达式来消除以前只能通过繁琐的构造才能避免的许多类型的代码重复。
我们来看一个分析访问网站的例子。 SiteVisit类存储每次访问的路径，持续时间和用户的操作系统。 各种操作系统用枚举表示：
<pre><code>
data class SiteVisit( val
           path: String,
           val duration: Double,
           val os: OS
)

enum class OS { WINDOWS, LINUX, MAC, IOS, ANDROID }

val log = listOf(
        SiteVisit("/", 34.0, OS.WINDOWS),
        SiteVisit("/", 22.0, OS.MAC),
        SiteVisit("/login", 12.0, OS.WINDOWS),
        SiteVisit("/signup", 8.0, OS.IOS),
        SiteVisit("/", 16.3, OS.ANDROID)
)
</code></pre>
假设您需要显示Windows机器的平均访问时间。 你可以使用平均函数来解决任务。
<pre><code>
val averageWindowsDuration = log.filter {
        it.os == OS.WINDOWS
    }.map(SiteVisit::duration)
            .average()
>>>println(averageWindowsDuration)
23.0
</code></pre>
现在，假设您需要为Mac用户计算相同的统计信息。 为避免重复，可以将该平台作为参数提取。
<pre><code>
fun List<SiteVisit>.averageDurationFor(os: OS) = filter { it.os == os }.map(SiteVisit::duration).average()
>>>println(log.averageDurationFor(OS.WINDOWS))
23.0
>>>println(log.averageDurationFor(OS.MAC))
22.0
</code></pre>
请注意如何使这个功能扩展提高了可读性。 如果仅在本地上下文中有意义，则甚至可以将此函数声明为本地扩展函数。
但它不够强大。 想象一下，您对移动平台的平均访问时间感兴趣（目前您认识其中的两个：iOS和Android）。
<pre><code>
val averageWindowsDuration = log.filter {
        it.os in setOf(OS.IOS, OS.ANDROID)
    }.map(SiteVisit::duration)
            .average()
>>>println(averageWindowsDuration)
12.15
</code></pre>
现在代表平台的一个简单的参数不能完成这项工作。 您也可能希望以更复杂的条件查询日志，例如“iOS的注册页面的平均访问持续时间是多少？ Lambdas可以提供帮助。 您可以使用函数类型将所需条件提取到参数中。
<pre><code>
fun List<SiteVisit>.averageDurationFor(predicate: (SiteVisit) -> Boolean) = filter(predicate).map(SiteVisit::duration).average()
>>>println(log.averageDurationFor { it.os in setOf&lt;OS>(OS.ANDROID, OS.IOS) })
12.15
println(log.averageDurationFor { it.os == OS.IOS && it.path == "/signup" })
8.0
</code></pre>
正如你所看到的，函数类型可以帮助消除代码重复。 如果您想复制和粘贴一段代码，可能会避免重复。 =使用lambda表达式，不仅可以提取重复的数据，还可以提取行为。

---
内联函数 ：消除 lambda 带来的运行时开销
您可能已经注意到，传递lambda参数给Kotlin中的函数的简写语法看起来与常规语句（如if和for）的语法类似。
当我们使用`with`，`apply`函数时，运行速度是怎样的？
在第五章中，我们解释说lambda通常编译为匿名类。
但是这意味着每次使用lambda表达式时都会创建一个额外的类; 如果lambda捕获了一些变量，那么每个调用都会创建一个新的对象。
这会引入运行时开销，导致使用lambda的实现效率低于直接执行相同代码的函数。
这个问题出现了：是否有可能告诉编译器生成与Java语句一样高效的代码，但是可以将重复的逻辑提取到库函数中？ 事实上，Kotlin编译器允许你这样做。 如果使用inline关键字标记函数，编译器将不会在使用此函数时生成函数调用，而是会用实现该函数的实际代码替换对该函数的每个调用。 让我们来看看这是如何详细工作，并看具体的例子。

---
内联函数如何运作
当你将一个函数声明为内联函数时，它的内体被内联 - 换句话说，它被直接替换而不是函数调用。 让我们看一个例子来理解结果代码。
8.14中的函数可以用来确保共享资源不被多个线程同时访问。 该函数锁定一个Lock对象，执行给定的代码块，然后释放该锁。
<pre><code>
inline fun &lt;T> synchronized(lock: Lock, action: () -> T): T {
    lock.lock()
    try {
        return action()
    } finally {
        lock.unlock()
    }
}
</code></pre>
调用这个函数的语法看起来就像在Java中使用synchronized语句一样。 不同的是，Java同步语句可以与任何对象一起使用，而这个函数需要传递一个Lock实例。 这里显示的定义只是一个例子。 Kotlin标准库定义了一个不同版本的synchronized，它接受任何对象作为参数。
由于您已将同步函数声明为内联，因此每次对其调用所生成的代码与Java中的同步语句的代码相同。
<pre><code>
fun foo(l: Lock)
{ println("Before
sync") synchronized(l)
{
println("Action")
}
println("After sync")
}
</code></pre>
请注意，内联应用于lambda表达式以及synchronized函数的实现。 从lambda生成的字节码成为调用函数定义的一部分，不包含在实现函数接口的匿名类中。
请注意，也可以调用一个内联函数，并从一个变量传递一个函数类型的参数：
<pre><code>
class LockOwner(val lock: Lock) {
	fun runUnderLock(body: () -> Unit)
		{ synchronized(lock, body)
	}
}
</code></pre>
在这种情况下，lambda的代码在内联函数被调用的位置不可用，因此它不是内联的。 只有内联的身体; lambda像往常一样被调用。
同步函数是如果你有两个使用内联函数在不同的位置有不同的lambda，那么每个调用站点将被内联独立。 内联函数的代码将被复制到您使用它的两个位置，并将不同的lambda代入它。

---
内联函数的限制
由于执行内联的方式，并不是每个使用lambdas的函数都可以内联。 函数内联时，作为参数传递的lambda表达式的主体被直接替换为结果代码。 这限制了函数体中相应参数的可能用法。 如果调用这个参数，这样的代码可以很容易地内联。 但是，如果参数存储在某个地方供进一步使用，则lambda表达式的代码不能内联，因为必须有一个包含此代码的对象。
通常，如果直接调用该参数，或者将该参数作为参数传递给另一个内联函数，则该参数可以内联。 否则，编译器将禁止将参数内联，并显示一条错误消息，指出“非法使用内联参数”。
例如，在序列上工作的各种函数返回代表相应序列操作的类的实例，并接收lambda作为构造函数参数。以下是Sequence.map函数的定义：
<pre><code>
fun &lt;T, R> Sequence<T>.map(transform: (T) -> R): Sequence<R> {
    return TransformingSequence(this, transform)
}
</code></pre>
map函数不直接调用作为transform参数传递的函数。 相反，它将此函数传递给将其存储在属性中的类的构造函数。 为了支持这一点，作为变换参数传递的lambda需要被编译成标准的非内联表示，作为实现函数接口的匿名类。
如果你有一个函数需要两个或更多lambdas作为参数，你可以选择仅内联一些。 当一个lambda需要包含很多代码或者以不允许内联的方式使用时，这是有意义的。 您可以使用noinline修饰符标记接受这种不可内联lambda表达式的参数：
<pre><code>
inline fun foo(inlined: () -> Unit, noinline notInlined: () -> Unit) {
		// ...
}
</code></pre>

---
内联集合操作
让我们考虑Kotlin标准库函数对集合的性能。 标准库中的大部分集合函数都将lambda表达式作为参数。 直接实现这些操作会更有效率，而不是使用标准库函数吗？
例如，让我们来比较一下你可以过滤一个列表的方式：
<pre><code>
data class Person(val name: String, val age: Int)

val person = listOf(Person("Alice", 29), Person("Bob", 31))

>>>println(person.filter { it.age &lt; 30 })
</code></pre>
前面的代码可以在不使用lambda表达式的情况下重写，如下所示：
<pre><code>
>>> val result = mutableListOf<Person>()
>>> for (person in people) {
>>> if (person.age &lt; 30) result.add(person)
>>> }
>>> println(result)
[Person(name=Alice, age=29)]
</code></pre>
在Kotlin中，过滤函数被声明为内联。 这意味着过滤器函数的字节码以及传递给它的lambda的字节码将被内联到过滤器被调用的地方。 因此，使用过滤器的第一个版本生成的字节码与为第二个版本生成的字节码大致相同。 您可以安全地使用集合上的惯用操作，而Kotlin对内联函数的支持可确保您无需担心性能。
现在想象一下，你应用两个操作过滤器和map链：
<pre><code>
>>> println(people.filter { it.age > 30 }
... .map(Person::name))
[Bob]
</code></pre>
这个例子使用一个lambda表达式和一个成员引用。 再一次，过滤器和map都声明为内联，所以它们的内体被内联，并且没有额外的类或对象被创建。 但是代码创建一个中间集合来存储过滤列表的结果。 从过滤器函数生成的代码将元素添加到该集合，并从映射生成的代码从中读取。
如果要处理的元素的数量很大，并且中间集合的开销成为问题，则可以使用序列来替代，方法是将一个asSequence调用添加到链中。 但正如你在上一节看到的，用于处理序列的lambda表达式不是内联的。 每个中间序列被表示为在其字段中存储lambda的对象，并且终端操作引起通过每个中间序列的调用链被执行。 因此，即使对序列的操作是懒惰的，也不应该努力将asSequence调用插入到代码中的每个收集操作链中。 

---
决定何时将函数声明成内联
现在您已经了解了inline关键字的好处，您可能想要在整个代码库中使用它，试图使其运行速度更快。 事实证明，这不是一个好主意。 使用inline关键字只能将lambdas作为参数的函数提高性能。
对于常规的函数调用，Java虚拟机已经提供了强大的内联支持。 它分析你的代码的执行情况，并在内部调用时提供最大的好处。 这自动发生在机器代码级别。 在字节码中，每个函数的实现只重复一次，并不需要在函数被调用的每个地方复制，就像Kotlin的内联函数一样。
更重要的是，如果直接调用函数，栈跟踪更清晰。
另一方面，用lambda参数内联函数是有益的。 首先，通过内联避免的开销更重要。 您不仅可以保存调用，还可以为每个lambda创建额外的类，为lambda实例创建一个对象。 其次，JVM目前还不够聪明，总是通过调用和lambda进行内联。
但是，在决定是否使用内联修饰符时，仍然应该注意代码的大小。 如果你想内联的函数很大，把代码复制到每个调用站点可能代价很大。 在这种情况下，您应该尝试将与lambda参数无关的代码抽取到单独的非内联函数中。
您可以自己验证Kotlin标准库中的内联函数总是很小。

---
使用内联 lambda 管理资源 
lambda可以删除重复代码的一个常见模式是资源管理：在操作之前获取资源并在之后释放资源。 这里的资源可能意味着许多不同的东西：文件，锁，数据库事务等等。 实现这种模式的标准方法是使用try / finally语句，其中在try块之前获取资源并在finally块中释放资源。
在本节前面，您看到了一个如何将try / finally语句的逻辑封装在函数中的示例，并将该资源作为lambda函数传递给该函数。 这个例子展示了synchronized函数，它与Java中的synchronized语句具有相同的语法：它将锁对象作为参数。 Kotlin标准库定义了另一个名为withLock的函数，它为同一个任务提供了一个更为习惯的API：它是Lock接口的扩展方法。 以下是如何使用它：
<pre><code>
val l: Lock = ...
l.withLock {
// access the resource protected by this lock
}
</code></pre>
以下是Kotlin库中定义的withLock函数的方法：
<pre><code>
fun &lt;T> Lock.withLock(action: () -> T): T
	{ lock()
	try {
		return action()
		} finally {
	unlock()
	}
}
</code></pre>
文件是使用此模式的另一种常见类型的资源。 Java 7甚至为这种模式引入了特殊的语法：try-with-resources语句。 以下清单显示了使用此语句从文件中读取第一行的Java方法。
<pre><code>
    static String readFirstLineFromFile(String path) throws IOException {
        try (BufferedReader br =
                     new BufferedReader(new FileReader(path))) {
            return br.readLine();
        }
    }
</code></pre>
Kotlin没有相同的语法，因为通过带有lambda参数的函数可以完成相同的任务。 该函数被称为use，并包含在Kotlin标准库中。
<pre><code>
    static String readFirstLineFromFile(String path) throws IOException {
        try (BufferedReader br =
                     new BufferedReader(new FileReader(path))) {
            return br.readLine();
        }
    }
</code></pre>
use函数是在可关闭资源上调用的扩展函数; 它接收一个lambda作为参数。 该函数调用lambda并确保资源是关闭的，无论lambda是正常完成还是引发异常。 当然，使用函数是内联的，所以它的使用不会导致任何性能开销。

---
高阶函数中的控制流
当你开始使用lambdas替换循环等命令式的代码结构时，你很快就会遇到返回表达式的问题。 在循环的中间放置一个return语句是不容易的。 但是如果将循环转换为使用过滤器之类的函数呢？ 如何在这种情况下返回工作？ 我们来看一些例子。

---
lambda 中的返回语句 ：从一个封闭的函数返回
<pre><code>
fun lookForAlice(peoples: List&lt;Person>) {
    for (people in peoples) {
        if (people.name == "Alice") {
            println("Found")
            return
        }
    }
    println("Alice is not found")
}
>>>lookForAlice(people)
Found
</code></pre>
使用forEach迭代重写此代码是否安全？ 返回声明是否意味着同样的事情？ 是的，使用forEach函数是安全的，如下所示。
<pre><code>
fun lookForAlice(peoples: List&lt;Person>) {
    peoples.forEach {
        if (it.name == "Alice"){
            println("Found")
            return
        }
    }
    println("Alice is not found")
}
</code></pre>
如果在lambda表达式中使用return关键字，它将从您调用lambda的函数返回，而不仅仅是从lambda本身返回。 这样的返回语句被称为非本地返回，因为它返回的块比包含return语句的块大。
请注意，只有以lambda作为参数的函数被内联，才能从外部函数返回。 在前面的示例中，forEach函数的主体与lambda的主体一起内联，因此编译返回表达式以使其从封闭函数返回很容易。 在传递给非内联函数的lambda表达式中使用返回表达式是不允许的。 非内联函数可以将传递给它的lambda保存到一个变量中，并在函数已经返回时执行它，所以当周围函数返回时，lambda影响已经太晚了。

---
从 lambda 返回 ：使用标签返回
你也可以写一个lambda表达式的本地返回值。 lambda中的局部返回类似于for循环中的break表达式。 它停止lambda的执行并继续执行从lambda被调用的代码。 要区分本地return和非本地return，请使用标签。 您可以标记要返回的lambda表达式，然后在return关键字之后引用此标签。
<pre><code>
fun lookForAlice(peoples: List<Person>) {
    peoples.forEach label@ {
        if (it.name == "Alice"){
            return@label
        }
    }
    println("Alice might be somewhere")
}
>>>lookForAlice(person)
Alice might be somewhere
</code></pre>
要标记一个lambda表达式，在lambda的开始大括号之前放置标签名称（可以是任何标识符），后跟@字符。 要从lambda返回，请在返回关键字后面加上@字符，后跟标签名称。
或者，将此lambda作为参数的函数的名称可以用作标签。
<pre><code>
fun lookForAlice(peoples: List<Person>) {
    peoples.forEach {
        if (it.name == "Alice") {
            return@forEach
        }
    }
    println("Alice might be somewhere")
}
</code></pre>
请注意，如果您明确指定了lambda表达式的标签，则使用函数名称的标签不起作用。 一个lambda表达式不能有多个标签。

---
匿名函数 ：默认使用局部返回
匿名函数是编写传递给函数的代码块的不同方式。
我们从一个例子开始。
<pre><code>
fun lookForAlice(peoples: List<Person>) {
    peoples.forEach(fun(person) {
        if (person.name == "Alice") return
        println("${person.name} is not Alice")
    })
}
>>> lookForAlice(people)
Bob is not Alice
</code></pre>
您可以看到，匿名函数看起来与常规函数类似，只是省略了名称和参数类型。 这是另一个例子。
<pre><code>
person.filter(fun(person): Boolean {
        return person.age &lt; 30
    })
</code></pre>
匿名函数遵循与指定返回类型的常规函数相同的规则。 块体的匿名函数（如列表8.26中的函数）要求显式指定返回类型。 如果使用表达式体，则可以省略返回类型。
<pre><code>
person.filter(fun(person): Boolean = person.age &lt; 30)
</code></pre>
在匿名函数中，没有标签的返回表达式将从匿名函数中返回，而不是从包含的函数返回。 规则很简单：返回从使用fun关键字声明的最近函数返回。 Lambda表达式不使用fun关键字，所以lambda中的返回从外部函数返回。 匿名函数确实有趣; 因此，在前面的例子中，匿名函数是最接近的匹配函数。 因此，返回表达式从匿名函数返回，而不是从封闭函数返回
![匿名函数与Lambda返回 ](https://github.com/aheven/holt/blob/master/image/%E5%8C%BF%E5%90%8D%E5%87%BD%E6%95%B0%E8%BF%94%E5%9B%9E%E4%B8%8Elambda%E8%BF%94%E5%9B%9E.png)
请注意，尽管匿名函数看起来与常规函数声明类似，但它只是lambda表达式的另一种语法形式

## 总结 ##

 - 函数类型允许您声明一个变量，参数或函数返回值，该值保存对函数的引用。
 - 高阶函数将其他函数作为参数或返回。 您可以通过使用函数类型作为函数参数或返回值的类型来创建此类函数。
 - 当一个内联函数被编译时，它的字节码以及传递给它的lambda的字节码被直接插入到调用函数的代码中，与直接编写的类似代码相比，这确保了调用的发生。
 - 高阶函数便于单个组件部分内的代码重用，让您构建强大的通用库。
 - 内联函数允许您使用放置在从封闭函数返回的lambda表达式中的非本地返回返回表达式。
 - 匿名函数为具有不同规则的lambda表达式提供了另一种语法来解析返回表达式。 如果您需要编写一个具有多个退出点的代码块，则可以使用它们。