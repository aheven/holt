# Kotlin函数声明与调用（九）
你可以使用以下的方式创建集合
<pre><code>
val list = listOf<Int>(1, 7, 53, 8)
val map = mapOf(1 to "one", 7 to "seven", 53 to "fifty-three")
</code></pre>
值得注意的是kotlin没有自己的集合类，kotlin使用标准的java集合类，这么做是为了完美地与java协同合作。但是在kotlin的标准库中，你可以使用集合类做更多的事情：
<pre><code>
fun main(args: Array<String>) {
    val list = listOf<Int>(1, 7, 53, 8)
    val set = setOf<Int>(1, 14, 2, 8)
    println(list.last())
    println(set.max())
}
</code></pre>
***
使函数更加容易调用：
现在需要一个需求，将集合中的元素添加到StringBuilder中，元素与元素之间有一个分隔符，开头和结尾都有不同符号标记，但是这些符号都是可以自定义的。
<pre><code>
fun <T> joinToString(collection: Collection<T>,
                     separator: String,
                     prefix: String,
                     postfix: String): String {
    val result = StringBuilder(prefix)
    for ((index, element) in collection.withIndex()) {
        if (index > 0) result.append(separator)
        result.append(element)
    }
    result.append(postfix)
    return result.toString()
}
</code></pre>
现在我们尝试着调用这个函数

<pre><code>
  val list = listOf(1, 2, 3)
  println(joinToString(list, "; ", "(", ")"))
</code></pre>
***
## 命名参数-解决函数调用的可读性 ##
在kotlin中可以使用下面的方式调用函数，增加函数调用的可读性
<pre><code>
	val list = listOf<Int>(1, 7, 53, 8)
    println(joinToString(list, separator = "; ", prefix = "(", postfix = ")"))
</code></pre>
当使用上述方法调用函数时，可以指定要传递给函数的一些参数名称。如果在调用中指定了参数的名称，那么还应该为之后的所有参数都指定名称，防止混淆。不幸的是，在kotlin中调用Java编写的方法时，不能指定命名参数，包括JDK和Android框架中的方法。
***
## 默认参数值 ##
另一个java中的常见问题是一些类中过多的重载方法，为了方便api用户，或者其他原因，重载的出现使java代码出现了大量重复，参数名称和类型不断反复地出现，你必须在每次重载中复制大部分文档，同时，如果你调用了省略某些参数的函数，则你并不能确认默认的值是哪个。
在kotlin中，你可以经常性避免使用重载，因为可以在函数声明的时候制定参数的默认值。
比如将上面的`joinToString`方法制定默认值：
<pre><code>
fun <T> joinToString(collection: Collection<T>,
                     separator: String = ",",
                     prefix: String = "",
                     postfix: String = ""): String 
</code></pre>
则你可以使用一下方式调用函数：
<pre><code>
>>> joinToString(list, ", ", "", "")
1, 2, 3
>>> joinToString(list)
1, 2, 3
>>> joinToString(list, "; ")
1; 2; 3
>>> joinToString(list, prefix = "# ")
/# 1, 2, 3
</code></pre>
鉴于java没有默认参数值的概念，当使用java调用kotlin的具有默认参数值的函数时，必须显式制定所有参数，如果经常需要调用这个函数，则可以在kotlin的方法前加上@JvmOverloads，这指示编译器生成java的重载函数。
例如，使用@JvmOverloads注释joinToString()，则会生成以下方法：
<pre><code>
/* Java */
String joinToString(Collection<T> collection, String separator,String prefix, String postfix);
String joinToString(Collection<T> collection, String separator,String prefix);
String joinToString(Collection<T> collection, String separator);
String joinToString(Collection<T> collection);
</code></pre>
每个参数都会使用省略的默认参数值
***
## 摆脱静态工具类-顶级函数和属性 ##
我们都知道，Java作为面向对象语言，需要将代码写成类的方法。 通常，这样做很好; 但实际上，几乎每一个大型项目最终都会出现很多不属于任何一个类的代码。 有时，一个操作可以与两个不同的类的对象一起工作，这些对象同样重要。 有时候有一个主要的对象，但你不想通过添加操作作为一个实例方法来膨胀它的API。一个典型的例子就是java中的Collections类。
在kotlin中，不需要再创建这些本身没有意义的类，kotlin中可以将这些函数放在任何类之外的源文件的顶层，如果需要从其它包调用他们，同样需要导入他们，但额外的嵌套不需要再使用了。
将joinToString函数放入strings包中：
<pre><code>
package strings
fun joinToString(...): String { ... }
</code></pre>
将kotlin代码转成java的代码为：

<pre><code>
/* Java */
package strings;
public class JoinKt {
	public static String joinToString(...) { ... }
}
</code></pre>

* JoinKt对应于join.kt，上一个例子的文件名
可以看到kotlin编译器生成的类名称对应于包含该函数的文件的文件名称。文件中所有的顶级函数都被编译成该类的静态方法。因此，从Java调用此方法与调用任何其他静态方法一样简单：
<pre><code>
import strings.JoinKt;
...
JoinKt.joinToString(list, ", ", "", "")
</code></pre>
如果要更改包含Kotlin顶级函数的生成类的名称，在该文件头部添加一个@JvmName注释。 将它放在文件的开头，在包名之前：
<pre><code>
@file:JvmName("StringFunctions")
package strings
fun joinToString(...): String { ... }
</code></pre>
现在函数可以调用如下：
<pre><code>
/* Java */
import strings.StringFunctions;
StringFunctions.joinToString(list, ", ", "", "");
</code></pre>

顶级属性和顶级函数一样放在文件的顶层
<pre><code>
var opCount = 0 1.
fun performOperation()
{ opCount++2.
// ...
}
fun reportOperationCount() {
println("Operation performed $opCount times")3.
}
</code></pre>

 1. 包级属性声明
 2. 更改属性的值
 3. 读取属性的值

顶级属性的值在java中将被储存为一个静态字段
顶级属性还允许您在代码中定义常量：
<pre><code>
val UNIX_LINE_SEPARATOR = "\n"
</code></pre>
通常可以使用const来修饰他：
<pre><code>
const val UNIX_LINE_SEPARATOR = "\n"
</code></pre>
它等价与java代码中的：
<pre><code>
/* Java */
public static final String UNIX_LINE_SEPARATOR = "\n";
</code></pre>
***
## 扩展函数 ##
扩展函数概念：它是一个可以被称为类成员但是被定义在该类之外的函数。
例如添加一个字符串最后一个字符的函数
<pre><code>
fun String.lastChar() = this.get(this.length - 1)
</code></pre>
可以看到，拓展函数就是将需要被拓展的类放在要添加的函数的名称之前：
![扩展函数定义图](https://github.com/aheven/holt/blob/master/image/%E6%89%A9%E5%B1%95%E5%87%BD%E6%95%B0%E5%AE%9A%E4%B9%89.png)
然后可以使用与普通类成员方法来调用它：
<pre><code>
>>> println("Kotlin".lastChar())
n
</code></pre>
在扩展函数中，可以直接访问要扩展的类的方法和属性。但是，扩展功能不允许打破封装，即与在类中定义方法不同，扩展函数无法访问该类的私有或受保护的成员。
当定义了一个扩展函数时，它不会在整个项目中自动变得可用，扩展函数需要被导入后才能使用，就像导入其它类一样，这有利于避免意外的名称冲突：
<pre><code>
import strings.lastChar|import strings.*
val c = "Kotlin".lastChar()
</code></pre>
可以用as关键字更改要导入的类或函数的名称：
<pre><code>
import strings.lastChar as last
val c = "Kotlin".last()
</code></pre>
从java中调用拓展函数：
在jvm下，扩展函数以静态方法的形式出现，扩展对象作为第一个参数传递：
<pre><code>
/* Java */
char c = StringUtilKt.lastChar("Java");
</code></pre>
接下来改造joinToString函数，将其改成扩展函数：
<pre><code>
fun  &ltT> Collection &ltT>.joinToString(1.
	separator: String = ", ",2.
	prefix: String = "",2.
	postfix: String = ""2.
	): String {
	val result = StringBuilder(prefix)
	for ((index, element) in this.withIndex())3.
	{ if (index > 0)
	result.append(separator)
	result.append(element)
	}
	result.append(postfix)
return result.toString()
}
</code></pre>

 1. 在Collection &ltT>上声明扩展功能
 2. 分配参数的默认值
 3. this指当前集合的对象

现在你可以像一个类的成员一样调用joinToString：
<pre><code>
>>> val list = arrayListOf(1, 2, 3)
>>> println(list.joinToString(" "))
1 2 3
</code></pre>
拓展函数是无法被覆盖的，这是因为扩展功能不是类的一部分; 它们被外部声明。 即使您可以使用与基类及其子类相同的名称和参数类型定义扩展函数，所调用的函数依然取决于声明的变量的静态类型，而不是存储在该变量中的值的运行时类型。
以下示例显示在View和Button类上声明的两个showOff扩展函数：
<pre><code>
fun View.showOff() = println("I'm a view!")
fun Button.showOff() = println("I'm a button!")
>>> val view: View = Button()
>>> view.showOff()
I'm a view!
</code></pre>
根据其java形式可以看出为什么扩展函数是无法被覆盖的原因：
<pre><code>
/* Java */
>>> View view = new Button();
>>> ExtensionsKt.showOff(view);
I'm a view!
</code></pre>
如你所见，覆盖不适用于扩展函数：Kotlin静态地解析它们。如果类内部拥有与扩展函数相同名字的成员函数，那么成员函数始终是优先的。
## 扩展属性 ##
将扩展函数`getLastChar()`修改为扩展属性：
<pre><code>
val String.lastChar: Char
    get() = get(length - 1)
</code></pre>
如果您在StringBuilder上定义相同的属性，则可以将其设置为var，因为可以修改StringBuilder的内容：
<pre><code>
var StringBuilder.lastChar: Char
	get() = get(length - 1)
	set(value: Char) {
	this.setCharAt(length - 1, value)
	}
</code></pre>
***
## vararg-接受任意数量参数的函数 ##
`val list = listOf(2, 3, 5, 7, 11)`的函数声明为：

<pre><code>
public fun <T> listOf(vararg elements: T): List<T> {}
</code></pre>

Kotlin和Java之间的另一个区别是当需要传递的参数已经包装在数组中时调用该函数的语法。 在Java中，您可以按原样传递数组，而Kotlin则要求您显式解包数组。
 从技术上讲，这个特性叫做扩展运算符，但实际上就是将*字符放在相应的参数之前：
 <pre><code>
 fun main(args: Array<String>) {
	val list = listOf("args: ", *args)
	println(list)
}
</code></pre>
扩展运算符允许您在单个调用中组合来自数组的值和一些固定值。 但这在Java中并不受支持。
##infix-中缀标记法##
下面是一个创建map的方法：
 <pre><code>
 val map = mapOf(1 to "one", 7 to "seven", 53 to "fifty-three")
</code></pre>
使用中缀标记法(infix notation)来调用函数, 但函数需要满足以下条件:

 - 是成员函数, 或者是扩展函数
 - 只有单个参数
 - 使用 infix 关键字标记
map中的`1 to "one"`使用了中缀标记法标识，其声明如下
 <pre><code>
 infix fun Any.to(other: Any) = Pair(this, other)
</code></pre>
再看一个简单的例子：
 <pre><code>
 fun main(args: Array<String>) {
    val person: Person = Person("Jone", 20)
    // 使用中缀标记法调用扩展函数
    person printName "AA-BB"
    // 上面的语句等价于
    person.printName("AA-BB")
}
class Person(var name: String, var age: Int) {
    // 使用infix 关键字标记，该函数可被中缀标记法法调用
    infix fun printName(addr: String) {
        println("addr: $addr, name: $name")
    }
}
</code></pre>
##字符串和正则表达式##
在java中，String.split()是一个很常用的方法，但是很多时候`12.345-6.A”.split（"."）`并不能得到我们想要的数据，这是因为java将.当成一个正则表达式的参数，并根据表达式对字符串进行分割成多个字符串。
Kotlin隐藏了容易混淆的方法，并提供了一些名为split的重载扩展名作为替换，它们具有不同的参数。 采用正则表达式的值需要一个Regex类型的值，而不是String。 这样可以确保传递给方法的字符串是否被解释为纯文本或正则表达式。
 <pre><code>
 println("12.345-6.A".split("\\.|-".toRegex()))
</code></pre>
但是对于这样一个简单的情况，您不需要使用正则表达式。 Kotlin中的分割扩展函数的其他重载将以任意数量的分隔符作为纯文本字符串：
 <pre><code>
 //指定多个分隔符
 println("12.345-6.A".split(".","-"))
</code></pre>
下面是一个将文件的完整路径名解析为目录，文件名和扩展名的例子：
 <pre><code>
 fun main(args: Array<String>) {
    parsePath("/Users/yole/kotlin-book/chapter.adoc")
}
fun parsePath(path: String) {
    val directory = path.substringBeforeLast("/")
    val fullName = path.substringAfterLast("/")
    val fileName = fullName.substringBeforeLast(".")
    val extension = fullName.substringAfterLast(".")
    println("Dir: $directory, name: $fileName, ext: $extension")
}
</code></pre>
可以看到，kotlin更容易在不使用正则表达式的情况下解析字符串。kotlin同样支持正则表达式对字符串进行解析，以下是用正则表达式实现的函数：
<pre><code>
fun parsePathRegexp(path: String) {
    val regex = """(.+)/(.+)\.(.+)""".toRegex()
    val matchResult = regex.matchEntire(path)
    if (matchResult != null) {
        val (directory, filename, extension) = matchResult.destructured
        println("Dir: $directory, name: $filename, ext: $extension")
    }
}
</code></pre>
在这个例子中，正则表达式是用三重引号的字符串写的。 在这样的字符串中，您不需要转义任何字符，包括反斜杠，因此您可以使用\对点符号进行编码。 而不是 \\。
三重引号字符串的目的不仅在于避免转义字符。 这样的字符串字面量可以包含任何字符，包括换行符。例如：
<pre><code>
val kotlinLogo = """| //
.|//
.|/ \"""
>>> println(kotlinLogo.trimMargin("."))
| //
|//
|/ \
</code></pre>
三重字符串包含换行符，因此不能使用\ n等特殊字符。
###本地函数和扩展-使代码更加简洁###
许多开发人员认为，良好代码的最重要素质之一是缺乏重复性。 这个原则甚至有一个特殊的名字：不要重复（DRY）。 但是当你用Java编写的时候，遵循这个原则并不总是微不足道的。 在许多情况下，可以使用IDE的Extract Method重构功能将更长的方法分解成更小的块，然后重新使用这些块。 但是，这样可以使代码更难理解，因为您最终可以使用许多小型方法，而且没有明确的关系。 你可以更进一步地将提取的方法组合成一个内部类，这样可以保持结构，但是这种方法需要大量的样板。
kotlin支持本地函数：在其它函数里面的函数。
<pre><code>
class User(val id: Int, val name: String, val address: String) {
    fun User.validateBeforeSave() {
        fun validate(value: String, fieldName: String) {
            if (value.isEmpty()) {
                throw IllegalArgumentException(
                        "Can't save user $id: empty $fieldName")
            }
        }
        validate(name, "Name")
        validate(address, "Address")
    }

    fun saveUser(user: User){
        user.validateBeforeSave()
        // Save user to the database
    }
}
</code></pre>

* 一个本地函数可以访问本地外部函数的本地变量，所以在上面的例子，id可以被访问。
* 扩展函数也可以被声明为本地函数，因此您可以进一步将User.validateBeforeSave（）作为本地函数放在saveUser（）中。但是深度嵌套的本地功能通常很难阅读; 因此，作为一般规则，我们不建议使用多个级别的嵌套。
***
## 总结 ##
* Kotlin没有定义自己的集合类，而是使用更丰富的API来扩展Java集合类。
* 定义函数参数的默认值减少了定义重载函数，并且在调用的时候使得代码变得更加可读。
* 函数和属性可以直接在文件中声明，而不仅仅是类的成员，允许更灵活的代码结构。
* 扩展函数和属性可以扩展任何类的API，包括外部库中定义的类，而不需要修改源代码，没有运行时开销。
* Infix调用提供了一个更加简洁的语法来调用类似操作符的方法。
* Kotlin为正则表达式和纯字符串提供了大量方便的字符串处理函数。
* 三重引用的字符串提供了一种干净的方式来编写需要在Java中进行大量字符转义和字符串连接的表达式。
* 本地函数可以更清晰地构建代码，并消除重复。