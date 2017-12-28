#Kotlin-泛型（十五）#
##泛型类型参数##
泛型允许您定义具有类型参数的类型。 当创建这样一个类型的实例时，类型参数被替换为特定的类型，称为类型参数。 例如，如果您有一个List类型的变量，那么知道该列表中存储了什么类型的东西是有用的。 type参数可以让你精确地指定 - 而不是“这个变量拥有一个列表”，你可以这样说：“这个变量包含一个字符串列表”。 Kotlin用于表示“字符串列表”的语法看起来与Java：List &lt;String>中的相同。 您也可以为一个类声明多个类型参数。 例如，Map类具有键类型和值类型的类型参数：Map &lt;String，Person>。 到目前为止，一切看起来都和Java一样。
就像通常的类型一样，Kotlin编译器通常可以推断出类型参数：
<pre><code>
val authors = listOf("Dmitry", "Svetlana")
</code></pre>
如果你需要创建一个空的列表，没有什么可以推断出类型参数，所以你需要明确地指定它。 在创建列表的情况下，您可以选择将类型指定为变量声明的一部分，并为创建列表的函数指定类型参数。 以下示例显示了这是如何完成的：
<pre><code>
val reads: MutableList&lt;String> = mutableListOf()
val reads = mutableListOf&lt;String>()
</code></pre>
与Java不同，Kotlin总是要求类型参数是由编译器明确指定或推断的。 因为泛型只在版本1.5中被添加到Java中，所以它必须保持与为旧版本编写的代码的兼容性，所以它允许使用不带类型参数的泛型类型（所谓的原始类型）。 例如，在Java中，可以声明List类型的变量，而不指定它包含的内容。因为Kotlin从一开始就有泛型，所以它不支持原始类型，而且类型参数必须被定义。

---
##泛型函数和属性##
如果你要编写一个可以和list一起工作的函数，而且你希望它能和任何列表一起工作（不是一个特定类型的元素列表），你需要编写一个通用函数。 泛型函数具有自己的类型参数。 这些类型参数必须用每个函数调用的特定类型参数替换。
大多数使用集合的库函数都是通用的。 例如，我们来看一下slice函数声明，如下所示。 这个函数返回一个只包含指定范围内索引元素的列表。
<pre><code>
fun &lt;T> List&lt;T>.slice(indices: IntRange): List&lt;T>
</code></pre>
函数的类型参数T用于接收器类型和返回类型; 他们都是List &lt;T>。 当你在一个特定的列表上调用这个函数时，你可以明确地指定这个类型参数。 但几乎在所有情况下，您都不需要，因为编译器推断它：
<pre><code>
>>>val letters = ('a'..'z').toList()
>>>println(letters.slice(0..2))
[a, b, c]
>>>println(letters.slice(10..13))
[k, l, m, n]
</code></pre>
再看一个例子：
<pre><code>
val authors = listOf("Dmitry", "Svetlana")
val readers = mutableListOf<String>(/* ... */)
fun &lt;T> List&lt;T>.filter(predicate: (T) -> Boolean): List&lt;T>
>>>readers.filter { it !in authors }
</code></pre>
在这种情况下，自动生成的lambda参数的类型是String。 编译器推断：在函数声明中，lambda参数具有泛型类型T（它是（T） - > Boolean中函数参数的类型）。 编译器知道T是String，因为它知道函数应该在List &lt;T>上被调用，而它的接收者，实际类型是List &lt;String>。
您可以在类，顶级函数和扩展函数的方法上声明类型参数。 在最后一种情况下，类型参数可以在接收者和参数类型中使用。
您也可以使用相同的语法声明通用扩展属性。 例如，下面是一个扩展属性，它返回列表中最后一个元素之前的元素：
<pre><code>
val &lt;T> List&lt;T>.penultimate: T
    get() = this[size - 2]
>>>println(listOf(1, 2, 3, 4).penultimate)
3
</code></pre>
你不能声明一个通用的非扩展属性
常规（非扩展）属性不能有类型参数。 不可能在一个类的属性中存储多个不同类型的值，因此声明一个通用的非扩展属性是没有意义的。 如果您尝试这样做，编译器会报告一个错误：
<pre><code>
>>>val &lt;T> x = T = TODO()
ERROR: type parameter of a property must be used in its receiver type
</code></pre>
##声明泛型类##
就像在Java中一样，通过在尖括号中的类名称和类型参数后面添加尖括号来声明Kotlin泛型类或接口。 一旦你这样做，你可以在类的主体中使用类型参数，就像其他类型一样。 让我们来看看如何在Kotlin中声明标准的Java接口List。 为了简化它，我们已经省略了大部分的方法：
<pre><code>
interface List&lt;T> {
	operator fun get(index: Int): T
	// ...
}
</code></pre>
如果您的类扩展了一个泛型类（或者实现了一个泛型接口），则必须为基类型的泛型参数提供一个类型参数。 它可以是特定类型或其他类型参数：
<pre><code>
class StringList: List&lt;String> {
	override fun get(index: Int): String = ...
}
class ArrayList&lt;T> : List&lt;T> {
	override fun get(index: Int): T = ...
}
</code></pre>
StringList类被声明为只包含String 元素，所以它使用String作为基类型的类型参数。 任何来自子类的函数都用这个适当的类型代替T，所以你有一个签名fun get（Int）：StringList中的String，而不是fun get（Int）：T
ArrayList类定义了自己的类型参数T，并将其指定为超类的类型参数。 请注意，ArrayList &lt;T>中的T与List &lt;T>中的T不同 - 它是一个新的类型参数，不需要具有相同的名称。
一个类甚至可以把自己称为一个类型参数。 实现Comparable接口的类是这种模式的经典例子。 任何可比较的元素都必须定义如何将其与相同类型的对象进行比较：
<pre><code>
interface Comparable&lt;T> {
	fun compareTo(other: T): Int
}
class String : Comparable&lt;String> {
	override fun compareTo(other: String): Int = /* ... */
}
</code></pre>
##类型参数约束##
类型参数约束使您可以限制可以用作类或函数的类型参数的类型。 例如，考虑一个计算列表中元素总和的函数。 它可以在List &lt;Int>或List &lt;Double>上使用，但不能用于List &lt;String>。 为了表达这一点，你可以定义一个类型参数约束，它指定sum的类型参数必须是一个数字。
当您将类型指定为泛型类型（也称为上限）的类型参数的约束时，泛型类型的特定实例中的相应类型参数必须是指定的类型或其子类型。 （现在，您可以将子类型视为子类的同义词，后面将突出显示其差异。）
要指定约束，需要在类型参数名称后面加上冒号，后跟类型，即类型参数的上限; 如下。 在Java中，使用关键字extends来表示相同的概念：&lt;T extends Number> T sum（List &lt;T> list）。
<pre><code>
fun &lt;T : Number> List&lt;T>.sum():T
</code></pre>
实际的类型参数（在下面的例子中为Int）应该扩展Number来允许这个函数的调用：
<pre><code>
>>>println(listOf(1, 2, 3).sum())
6
</code></pre>
一旦为类型参数T指定了一个边界，就可以使用类型T的值作为其上限的值。 例如，您可以调用用作绑定的类中定义的方法：
<pre><code>
fun &lt;T : Number> oneHalf(value: T): Double {
    return value.toDouble() / 2.0
}
>>>println(oneHalf(3))
1.5
</code></pre>
现在我们来编写一个通用函数来查找两个项目的最大值。 由于只能找到可以相互比较的项目的最大数量，因此您需要在函数的签名中指定该项目。 以下是你如何做到这一点。
<pre><code>
fun &lt;T : Comparable&lt;T>> max(first: T, second: T): T {
    return if (first > second) first else second
}
>>>println(max("Kotlin","Java"))
Kotlin
</code></pre>
当您尝试对不可比项目调用max时，代码将无法编译：
<pre><code>
>>>println(max("kotlin", 42))
ERROR: Type parameter bound for `T` is not satisfied:
	inferred type `Any` is not a subtype of `Comparable&lt;Any>`
</code></pre>
T的上界是一个泛型类型Comparable &lt;T>。 正如前面所见，String类扩展了Comparable &lt;String>，它使得String成为max函数的有效类型参数。
在极少数情况下，当您需要在类型参数上指定多个约束时，使用稍微不同的语法。 例如，下面的清单是确保给定的CharSequence最后有句点的通用方法。 它适用于标准的StringBuilder类和java.nio.CharBufferclass。
<pre><code>
fun &lt;T> ensureTrailingPeriod(seq: T)
        where T : CharSequence, T : Appendable {
    if (!seq.endsWith('.')) {
        seq.append('.')
    }
}
>>>val helloWorld = StringBuilder("Hello World")
>>>ensureTrailingPeriod(helloWorld)
>>>println(helloWorld)
Hello World.
</code></pre>
##让类型形参非空##
如果声明一个泛型类或函数，则可以用任何类型（包括可为空的类型）替代其类型参数。 考虑下面的例子：
<pre><code>
class Processor&lt;T> {
    fun process(value: T) {
        value?.hashCode()
    }
}
</code></pre>
在Processor函数中，即使T没有标记问号，参数值也是可以为空的。 这是因为处理器类的具体实例可以使用T的可为空类型：
<pre><code>
val nullableStringProcessor = Processor&lt;String?>()
nullableStringProcessor.process(null)
</code></pre>
如果你想保证一个非null类型将总是替代一个类型参数，你可以通过指定一个约束来实现：
<pre><code>
class Processor&lt;T : Any> {
    fun process(value: T) {
        value?.hashCode()
    }
}
</code></pre>
&lt;T：Any>约束确保T类型将始终为非空类型。 代码Processor &lt;String？>不会被编译器接受，因为类型参数String？ 不是Any的子类型（它是Any的子类型，它是一个不太具体的类型）：
<pre><code>
>>>val nullableStringProcessor = Processor&lt;String?>()
Error: Type argument is not within its bounds: should be subtype of 'Any'
</code></pre>
请注意，您可以通过将任何非空类型指定为上限，而不是仅指定Any类型，从而将类型参数设置为非null。
##运行时的泛型 ：擦除和实化类型参数##
正如您可能知道的那样，JVM上的泛型通常通过类型擦除来实现，这意味着泛型类实例的类型参数在运行时不会被保留。 在本节中，我们将讨论类型擦除对Kotlin的实际影响，以及如何通过将函数声明为内联来解决其限制。 你可以声明一个内联函数，使得它的类型参数不会被擦除（或者用Kotlin术语来说）。
##运行时的泛型 ：类型检查和转换##
就像在Java中一样，Kotlin的泛型在运行时被删除。 这意味着泛型类的实例不会携带有关用于创建该实例的类型参数的信息。 例如，如果你创建了一个List &lt;String>并且放入了一串字符串，那么在运行时你只能看到它是一个List（当然，你可以得到一个元素并检查它的类型，但是 这不会给你任何保证，因为其他元素可能有稍微不同的类型。）
考虑运行下列代码时这两个列表会发生什么：
<pre><code>
val list1: List&lt;String> = listOf("a", "b")
val list2: List&lt;Int> = listOf(1, 2, 3)
</code></pre>
即使编译器在列表中看到两个不同的类型，但在执行时它们看起来完全一样。 尽管如此，通常可以确定List &lt;String>仅包含字符串，而List &lt;Int>仅包含整数，因为编译器知道类型参数，并确保只有正确类型的元素存储在每个列表中。 （你可以通过类型转换或使用Java原始类型来访问列表来欺骗编译器，但是你需要做一些特别的工作。）
因为类型参数没有被存储，所以你不能检查它们，例如你不能检查列表是否包含字符串而不是其他对象。 作为一般规则，在检查中使用带有类型参数的类型是不可能的。 下面的代码将不能编译：
<pre><code>
>>>if (value is List&lt;String>) { ... }
ERROR: Cannot check for instance of erased type
</code></pre>
尽管在运行时很容易发现这个值是一个List，但是你不能判断它是一个字符串，人物还是别的东西：这个信息已经被删除了。 请注意，删除泛型类型信息有其好处：应用程序使用的内存总量较小，因为较少的类型信息需要保存在内存中。
正如我们前面所述，Kotlin不会让你使用一个没有指定类型参数的类型。 因此，您可能想知道如何检查该值是一个list，而不是一个set或另一个对象。 你可以通过使用特殊的星形投影语法来做到这一点：
<pre><code>
if (value is List&lt;*>) { ... }
</code></pre>
实际上，您需要为类型中的每个类型参数包含一个*。 我们将在本章后面详细讨论星形投影（包括为什么称为投影）。 现在，你可以把它看作一个未知参数的类型（或者Java的List &lt;？>类似）。 在前面的例子中，你检查一个值是否是一个List，并且你没有得到关于它的元素类型的任何信息。
请注意，你仍然可以使用正常的泛型as and as?。 但是，如果类具有正确的基本类型和错误的类型参数，则转换不会失败，因为在执行转换时，运行时不知道类型参数。 因为这个原因，编译器会在这样的一个表演上发出一个“unchecked cast”警告。 这只是一个警告，所以你可以稍后使用该值作为必要的类型，如下所示。
<pre><code>
fun printSum(c: Collection&lt;*>) {
    val intList = c as? List&lt;Int> ?: throw IllegalArgumentException("List is expected")
    println(intList.sum())
}
>>>printSum(listOf(1, 2, 3))
6
</code></pre>
一切正常编译：编译器只发出警告，这意味着这个代码是合法的。 如果您在整数或集合的列表中调用printSum函数，它将按预期工作：它在第一种情况下输出总和，并在第二种情况下抛出IllegalArgumentException。 但是如果你传入一个错误类型的值，你将在运行时得到一个ClassCastException：
<pre><code>
>>> printSum(setOf(1, 2, 3))
IllegalArgumentException: List is expected
>>> printSum(listOf("a", "b", "c"))
ClassCastException: String cannot be cast to Number
</code></pre>
让我们来讨论如果在字符串列表中调用printSum函数时抛出的异常。 您不会收到IllegalArgumentException，因为您无法检查参数是否为List &lt;Int>。因此转换成功，函数总和在这样的列表上调用。 尽管它被执行，但抛出了一个异常。 发生这种情况是因为该函数试图从列表中获取数值并将它们添加在一起。 尝试使用String作为数字会导致运行时发生ClassCastException。
请注意，Kotlin编译器足够聪明，可以在编译时检查对应的类型信息是否已知。
<pre><code>
fun printSum(c: Collection&lt;Int>) {
    if (c is List&lt;Int>){
        println(c.sum())
    }
}
</code></pre>
一般来说，Kotlin编译器负责让你知道哪些检查是危险的（禁止检查和发布警告），哪些是可能的。 你只需要知道这些警告的含义，并了解哪些操作是安全的。
正如我们已经提到的，Kotlin确实有一个特殊的构造，它允许你在函数体中使用特定的类型参数，但是这只对内联函数是可能的。 我们来看看这个功能。
##声明带实化类型参数的函数##
正如我们前面所讨论的，Kotlin泛型在运行时会被擦除，这意味着如果您有一个泛型类的实例，则无法找到创建实例时使用的类型参数。 这同样适用于函数的类型参数。 当你调用一个泛型函数时，在它的主体中，你不能确定它被调用的类型参数：
<pre><code>
>>> fun &lt;T> isA(value: Any) = value is T
Error: Cannot check for instance of erased type: T
</code></pre>
这通常是正确的，但有一种情况下可以避免这种限制：内联函数。 内联函数的类型参数可以被指定，这意味着您可以在运行时引用实际的类型参数。
我们在第8章中详细讨论了内联函数。提醒一下，如果使用inline关键字标记函数，编译器将用实际执行函数的代码替换对函数的每个调用。 如果函数使用lambdas作为参数，使内联函数可以提高性能：lambda代码也可以内联，所以不会创建匿名类。 本节展示了另一种内联函数有用的情况：它们的类型参数可以被指定。
如果将前面的isA函数声明为inline并将类型参数标记为已通过，则可以检查该值以查看它是否是T的一个实例：
<pre><code>
inline fun &lt; reified T> isA(value: Any) = value is T
>>>println(isA&lt;String>("abc"))
true
>>>println(isA&lt;String>(123))
false
</code></pre>
让我们看看使用实体类型参数的一些不太重要的例子。 其中一个最简单的例子就是filterIsInstance标准库函数。 该函数接受一个集合，选择指定类的实例，并仅返回那些实例。 这是如何使用它。
<pre><code>
>>>val items = listOf("one", 2, "three")
>>>println(items.filterIsInstance&lt;String>())
[one, three]
</code></pre>
你说你只对字符串感兴趣，通过指定&lt;String>作为函数的类型参数。 函数的返回类型因此将是List &lt;String>。 在这种情况下，type参数在运行时是已知的，filterIsInstance（）使用它来检查列表中的哪些值是指定为类型参数的类的实例。
下面是来自Kotlin标准库的filterIsInstance声明的简化版本：
<pre><code>
inline fun &lt;reified T> Iterable&lt;*>.filterIsInstance(): List&lt;T> {
    val destination = mutableListOf<T>()
    for (element in this) {
        if (element is T) {
            destination.add(element)
        }
    }
    return destination
}
</code></pre>

 - “reified”声明这个类型参数不应该被删除。
 - 您可以检查元素是否是指定为类型参数的类的实例

为什么物化只适用于内联函数

这个怎么用？ 为什么你允许在内联函数中写元素是T，而不是在普通的类或函数中？ 正如我们在第8.2节中讨论的那样，编译器将实现内联函数的字节码插入每个被调用的地方。 每次使用一个具体化的类型参数调用该函数时，编译器都知道该特定调用中用作类型参数的确切类型。 因此，编译器可以生成引用特定类用作类型参数的字节码。 实际上，对于显示的filterIsInstance&lt;String>调用，生成的代码将等同于以下内容：
<pre><code>
for (element in this) {
        if (element is String) {
            destination.add(element)
        }
    }
</code></pre>
因为生成的字节码引用了特定的类，而不是类型参数，所以它不受在运行时发生的类型参数删除的影响。
请注意，不能从Java代码调用具有指定类型参数的内联函数。 Kotlin代码不会调用它们，而是直接在调用站点内联，所以它们是以一种特殊的方式编译的，而不是常规的方法。因为Java不支持内联，所以这些函数是不可访问的。
<pre><code>
//code in java
System.out.println(CollectionsKt.filterIsInstance(list,Integer.class));
</code></pre>
内联函数可以具有多个指定类型参数，除了指定类型参数外，还可以具有非指定类型参数。 请注意，filterIsInstance（）函数被标记为内联，即使它没有任何lambda参数。 在第8.2.4节中，我们讨论了当函数具有lambda参数并且lambda函数与函数一起内联时，将函数标记为内联函数仅具有性能优点。 
为了确保良好的性能，您仍然需要跟踪标记为内嵌的函数的大小。 如果函数变大，最好将不依赖于指定类型参数的代码提取到单独的非内联函数中。
##使用实化类型参数代替类引用##
通用类型参数的一个常见用例是为接受java.lang.Class类型的参数的API构建适配器。 这种API的一个例子是来自JDK的ServiceLoader，它接受表示接口或抽象类的java.lang.Class，并返回实现该接口的服务类的实例。 让我们看看如何使用通用类型参数来使这些API更简单的调用。
要使用ServiceLoader的标准Java API加载服务，请使用以下调用：
<pre><code>
val serviceLoader = ServiceLoader.load(Service::class.java)
</code></pre>
:: class.java语法显示了如何获得对应于Kotlin类的java.lang.Class。 这与Java中的Service.class完全相同。
现在让我们使用一个具有指定类型参数的函数来重写这个例子：
<pre><code>
val serviceLoader = loadService&lt;Service>()
</code></pre>
更短，不是吗？ 如您所见，要加载的服务的类现在被指定为loadService函数的类型参数。 将类指定为类型参数比较容易阅读，因为它比您需要使用的:: class.java语法短。
接下来，我们来看看如何定义这个loadService函数：
<pre><code>
inline fun &lt;reified T> loadService() = ServiceLoader.load(T::class.java)
</code></pre>
您可以在常规类上使用的指定类型参数上使用相同的:: class.java语法。 使用这个语法可以为您提供java.lang.Class，它对应于指定为类型参数的类，您可以正常使用它。

简化Android上的startActivity功能

如果您是Android开发人员，则可能会发现另一个更熟悉的示例：显示Activity。 不要将活动的类作为java.lang.Class传递，也可以使用一个具体的类型参数：
<pre><code>
inline fun &lt;reified T : Activity> Context.startActivity() { 1.
    val intent = Intent(this, T::class.java) 2.
    startActicity(intent)
}

startActivity&lt;DetailActivity>() 3.
</code></pre>

 - 类型参数被标记为“reified”
 - 以T :: class的形式访问类型参数的类
 - 调用该方法来启动一个activity
##实化类型参数的限制##
指定类型参数是一个方便的工具，但它们也有一定的限制。 有些是固有的概念，而其他的则由当前的实现来决定，并可能在未来版本的Kotlin中放松。
更具体地说，下面是如何使用reified的类型参数：

 - 在类型检查和强制转换中（is, !is, as, as?）
 - 要使用Kotlin反射API，我们将在第10章讨论（:: class）
 - 为了得到相应的java.lang.Class（:: class.java）
 - 作为调用其他函数的类型参数
 
你不能在下列情景中使用：

 - 创建具有类型参数的类的新实例的时候
 - 调用类型参数类的伴生对象上的方法
 - 在使用特定类型参数调用函数时，使用非特定类型参数作为类型参数
 - 实体的类，属性或非内联函数的类型参数没有被`reified`标记

最后一个约束导致了一个有趣的结果：由于被指定的类型参数只能用在内联函数中，所以使用一个被指定的类型参数意味着该函数以及所有传递给它的lambda被内联。 如果因为内联函数使用它们而不能内联lambda，或者出于性能原因不希望内联函数被内联，则可以使用8.2.2节中介绍的noinline修饰符将它们标记为非内联。

##变型 ：泛型和子类型化##
方差概念描述了具有相同基本类型和不同类型参数的类型如何相互关联：例如，List &lt;String>和List &lt;Any>。 首先，我们将讨论为什么这个关系总体上是重要的，然后我们将看看它在Kotlin中是如何表达的。 在编写自己的泛型类或函数时，理解差异是至关重要的：它可以帮助您创建不会以不方便的方式限制用户的API，并且不会破坏类型安全性期望。

##为什么存在变型 ：给函数传递实参##
想象一下，你有一个函数，它将一个List &lt;Any>作为参数。 将List &lt;String>类型的变量传递给此函数是否安全？ 将astring传递给期望Any的函数是绝对安全的，因为String类扩展了Any。 但是当Any和String成为List接口的类型参数时，就不再那么清楚了。
例如，让我们考虑一个打印列表内容的函数。
<pre><code>
fun printContents(list: List&lt;Any>) {
    println(list.joinToString())
}
>>>printContents(listOf("abc","def"))
abc, def
</code></pre>
它看起来像一个字符串列表在这里工作正常。 该函数将每个元素都视为Any，并且因为每个字符串都是Any，所以它是完全安全的。
现在让我们来看看另一个函数，它修改列表（因此将MutableList作为参数）。
<pre><code>
fun addAnswer(list: MutableList&lt;String>){
    list.add(42.toString())
}

>>> val strings = mutableListOf("abc", "bac")
>>> addAnswer(strings)
>>> println(strings.maxBy { it.length })
ClassCastException: Integer cannot be cast to String
</code></pre>
你声明一个类型为MutableList &lt;String>的变量字符串。 然后你尝试把它传递给函数。 如果编译器接受它，您可以将一个整数添加到字符串列表中，当您尝试以字符串的形式访问列表的内容时，会导致运行时异常。 因此，这个调用不能编译。 这个例子表明，当MutableList &lt;Any>被期望时，传递一个MutableList &lt;String>作为参数是不安全的。 Kotlin编译器正确地禁止了这一点。
现在，您可以回答将某个字符串列表传递给期望列出任何对象的函数是否安全的问题。 如果函数添加或替换列表中的元素是不安全的，因为这会造成类型不一致的可能性。 否则是安全的（我们将在本章后面详细讨论为什么）。 在Kotlin中，这可以通过选择正确的界面来轻松控制，取决于列表是否可变。 如果一个函数接受一个只读列表，你可以传递一个更具体的元素类型的列表。 如果列表是可变的，你不能这样做。
在本章的后面，我们将推广任何泛型类的相同问题，而不仅仅是List。 你也会明白为什么两个接口List和MutableList的类型参数是不同的。 但在此之前，我们需要讨论类型和子类型的概念。
##类、类型和子类型##
在最简单的情况下，对于非泛型类，类的名称可以直接用作类型。 例如，如果您编写var x：String，则声明一个可以存放String类实例的变量。 但是请注意，同样的类名也可以用来声明一个可以为null的类型：var x：String ?. 这意味着每个Kotlin类可以用来构造至少两种类型。
泛型类的类型变得更加复杂。 为了得到一个有效的类型，你必须用一个特定的类型替换这个类的类型参数。 List不是一个类型（它是一个类），但是下列所有替换都是有效的类型：List &lt;Int>，List &lt;String>，List &lt;List &lt;String >>等等。 每个泛型类产生可能无限数量的类型。
为了让我们讨论类型之间的关系，你需要熟悉术语子类型。 类型B是类型A的子类型，只要需要类型A的值就可以使用类型B的值。 例如，Int是Number的子类型，但Int不是String的子类型。
例如，仅当表达式的类型是函数参数类型的子类型时才允许将表达式传递给函数。 另外，当值类型是变量类型的子类型时，只允许将值存储在变量中。
<pre><code>
fun test(i: Int) {
    val n: Number = i 1.
    f(i) 2.
}

fun f(s: String) {
    /*....*/
}
</code></pre>

 - 编译，因为Int是Number的子类型
 - 不会编译，因为Int不是String的子类型
在简单情况下，子类型意味着基本上与子类相同的东西。 例如，Int类是Number的子类，因此Int类型是Number类型的子类型。 如果一个类实现了一个接口，那么它的类型就是接口类型的一个子类型：String是CharSequence的子类型。
可空类型提供了子类型与子类不相同的一个示例。 非空类型是其可为空的版本的子类型，但它们都对应于一个类。 您始终可以将非空类型的值存储在可为空的类型的变量中，但不能将空值类型的值存储（对于非空类型的变量，null不是可接受的值）。
当我们开始讨论泛型时，子类和子类型之间的区别变得尤为重要。 上一节中提到的将List &lt;String>类型的变量传递给期望List &lt;Any>的函数是否安全的问题现在可以用子类型的形式进行重新表述：List &lt;String> List &lt;Any>的子类型？ 你已经知道为什么把MutableList &lt;String>当作MutableList &lt;Any>的子类型是不安全的。 显然，MutableList &lt;Any>也不是MutableList &lt;String>的子类型。
如果对于任何两个不同的类型A和B，MutableList &lt;A>不是MutableList &lt;B>的子类型或超类型，则通用类（例如MutableList）称为不变类型。 在Java中，所有的类都是不变的（即使这些类的具体用法可以被标记为非不变的，就像你将会看到的那样）。
在上一节中，您看到了子类型规则不同的类：List。 Kotlin中的List接口表示一个只读集合。 如果A是B的子类型，则List &lt;A>是List &lt;B>的子类型。 这样的类或接口被称为协变。
##协变 ：保留子类型化关系##
一个协变类是一个泛型类（我们将使用Producer &lt;T>作为例子），其中以下成立：如果A是B的一个子类型，那么Producer &lt;A>是Producer &lt;B>的一个子类型。 子类型被保留。 例如，Producer &lt;Cat>是Producer &lt;Animal>的一个子类型，因为Cat是Animal的一个子类型。
在Kotlin中，要将类声明为对某个类型参数是协变的，可以将out关键字放在类型参数的名称之前：
<pre><code>
interface Producer&lt;out T> {
    fun produce(): T
}
</code></pre>
 - 这个类在T上被声明为协变。

将类的类型参数标记为协变可以将该类的值作为函数参数传递，并在类型参数与函数定义中的类型参数不完全匹配时返回值。 例如，设想一个以Herd类为代表的喂养一组动物的功能。 Herd类的类型参数标识了群中动物的类型。
<pre><code>
open class Animal {
    fun feed() {/*...*/
    }
}

class Herd&lt;T : Animal> {
    val size: Int get() = ...
    operator fun get(i: Int): T {
        ...
    }
}

fun feedAll(animals: Herd&lt;Animal>) {
    for (i in 0 until animals.size) {
        animals[i].feed()
    }
}
</code></pre>
类型参数没有声明为协变。
假设你的代码的用户有一群猫，需要照顾他们：
<pre><code>
class Cat : Animal() { 1.
    fun cleanLitter() {
        ...
    }
}

fun takeCareOfCats(cats: Herd&lt;Cat>) {
    for (i in 0 until cats.size) {
        cats[i].cleanLitter()
         feedAll(cats) 2.
    }
}
</code></pre>

 1. A Cat is an Animal.
 2. Error: inferred type is Herd&lt;Cat>, but Herd&lt;Animal> was expected
不幸的是，这些猫仍然饥肠辘辘的：如果你试图传递给feedAll方法，编译过程中会出现类型不匹配的错误。 因为你不在Herd类的T型参数上使用任何方差注解，所以猫群不是动物群的子类。 你可以使用明确的转换来解决这个问题，但是这种方法是冗长的，容易出错的，而且几乎从来都不是一个处理类型不匹配问题的正确方法。
由于Herd类具有与List类似的API，并且不允许其客户端添加或更改群中的动物，因此可以使其协变并相应地更改调用代码。
<pre><code>
class Herd<out T : Animal> {
    val size: Int get() = ...
    operator fun get(i: Int): T {
        ...
    }
}

fun takeCareOfCats(cats: Herd&lt;Cat>) {
    for (i in 0 until cats.size) {
        cats[i].cleanLitter()
         feedAll(cats) 
    }
}
</code></pre>

 - T参数现在是协变的。
 - 你不需要强转

你不能做任何类的协变：这将是不安全的。 使类对某个类型参数进行协变约束了类中这个类型参数的可能用法。 为了保证类型安全，只能在所谓的外出位置使用，这意味着类可以给出类型T的值，但不能把它们带入。
关于类的类型参数的out关键字要求所有使用T的方法只有T在out位置而不在in位置。 这个关键字限制了T的使用，这保证了相应子类型关系的安全性。
作为一个例子，考虑一下Herd类。 它只在一个地方使用类型参数T：get方法的返回值。
<pre><code>
class Herd&lt;out T : Animal> {
    val size: Int
        get()
        = ...

    operator fun get(i: Int): T {
        ...
    }
}
</code></pre>

重申，类型参数T的out关键字意味着两件事情：

 - 保存子类型（Producer &lt;Cat>是Producer &lt;Animal>的子类型）。
 - T只能在返回值返回的时候使用。

现在让我们看看List &lt;T>接口。 List在Kotlin中是只读的，所以它有一个方法get返回一个T类型的元素，但是没有定义任何在列表中存储类型T的方法。 因此，它也是协变的：
<pre><code>
interface List&lt;out T> : Collection&lt;T> {
    operator fun get(index: Int): T
// ...
}
</code></pre>

 - 只定义返回T的方法的只读接口（所以T处于输出位置）

请注意，类型参数不仅可以直接用作参数类型或返回类型，还可以用作其他类型的类型参数。 例如，包含一个返回List &lt;T>的方法subList。
<pre><code>
interface List&lt;out T> : Collection&lt;T> {
    fun subList(fromIndex: Int, toIndex: Int): List&lt;T>
    //...
}
</code></pre>

 - 这个T处于return的状态是可行的

在这种情况下，函数subList中的T被用于out位置。 我们在这里不会深入细节。 如果您对确定哪个位置已关闭以及位于哪个位置的确切算法感兴趣，则可以在Kotlin语言文档中找到此信息。
请注意，您不能将MutableList &lt;T>声明为其类型参数的协变，因为它包含将T类型的值作为参数并返回这些值的方法（因此，T出现在in和out的位置）。
<pre><code>
interface MutableList&lt;T>
    : List&lt;T>, MutableCollection&lt;T> {
    override fun add(element: T): Boolean
}
</code></pre>

 - MutableList不能被声明为T上的协变...
 - ...因为T用在“in”的位置

编译器强制执行此限制。 如果该类声明为协变，它将报错：Type parameter T is declared as 'out' but occurs in
'in' position.
请注意，构造函数的参数既不在in也不在out的位置。 即使将一个类型参数声明为out，仍然可以在构造函数参数声明中使用它：
<pre><code>
class Herd&lt;out T: Animal>(vararg animals: T) { ... }
</code></pre>
如果你将它作为一个更通用的类型的实例工作，那么方差可以防止类实例被滥用：你不能调用潜在的危险方法。 构造函数不是一个可以稍后调用的方法（创建实例之后），因此它不会有潜在的危险。
但是，如果将val或var关键字与构造函数参数一起使用，则还要声明一个getter和一个setter（如果该属性是可变的）。 因此，类型参数用于只读属性的内外位置，以及可变属性的内外位置：
<pre><code>
class Herd&lt;T: Animal>(var leadAnimal: T, vararg animals: T) { ... }
</code></pre>
在这种情况下，T不能被标记为out，因为这个类包含一个setAnterward属性，它使用T in in position。
还要注意，位置规则只覆盖了一个类的外部可见（public和protected）API。 私有方法的参数既不在in也不在out位置。 差异规则保护一个类别免受外部客户的滥用，并且不会在类别本身的执行中发挥作用：
<pre><code>
class Herd&lt;out T: Animal>(private var leadAnimal: T, vararg animals: T) { ... }
</code></pre>
现在可以安全地使用协变，因为leadAnimal属性已经被私有。
##逆变 ：反转子类型化关系##
反变换的概念与协变是双重的：对于一个逆变类，子类型关系与作为类型参数的类的子类型关系是相反的。 我们从一个例子开始：`Comparator`接口。 这个接口定义了一个比较两个给定对象的方法：
<pre><code>
interface Comparator&lt;in T> {
    fun compare(e1: T, e2: T): Int { ... }
}
</code></pre>
你可以看到这个接口的方法只消耗了T类型的值。这意味着T仅用于in位置，因此它的声明可以以in关键字为前缀。
为特定类型的值定义的比较器当然可以比较该类型的任何子类型的值。 例如，如果您有比较器&lt;Any>，则可以使用它来比较任何特定类型的值。
<pre><code>
>>> val anyComparator = Comparator&lt;Any> {
    ... e1, e2 -> e1.hashCode() - e2.hashCode()
    ... }
>>> val strings: List&lt;String> = ...
>>> strings.sortedWith(anyComparator)
</code></pre>
sortedWith函数需要一个Comparator &lt;String>（一个可以比较字符串的比较器），并且可以安全地传递一个可以比较更一般类型的比较器。 如果您需要对某种类型的对象进行比较，则可以使用比较器来处理该类型或其任何超类型。 这意味着Comparator &lt;Any>是Comparator &lt;String>的子类型，其中Any是String的超类型。 两种不同类型比较器之间的子类型关系与这些类型之间的子类型关系相反。
现在你已经准备好了反转的完整定义。 与类型参数相反的类是泛型类（让我们以消费者<T>为例），其中以下成立：如果A是超类型B，则消费者&lt;A>是消费者&lt;B>的子类型。 类型参数A和B改变了地方，所以我们说子类型是颠倒过来的。 例如，Consumer &lt;Animal>是Consumer &lt;Cat>的子类型。
一个类或接口可以在一个类型参数上是协变的，而在另一个类型参数上是逆变的。 经典的例子是功能界面。 下面的函数接口声明需要一个参数：
<pre><code>
interface Function1&lt;in P, out R> {
    operator fun invoke(p: P): R
}
</code></pre>
Kotlin符号（P） - > R是表达Function1 &lt;P，R>的另一种更可读的形式。 您可以看到P（参数类型）仅用于in位置，用in关键字标记，而R（返回类型）仅用于out位置，并用out关键字标记。 这意味着函数类型的子类型在第一个类型参数中被颠倒过来，并保留在第二个类型参数中。 例如，如果你有一个更高级的函数来枚举Cat类，你可以通过一个lambda枚举任何动物。
<pre><code>
fun enumerateCats(f: (Cat) -> Number) { ... }
fun Animal.getIndex(): Int = ...

>>> enumerateCats(Animal::getIndex)
</code></pre>
请注意，到目前为止，在所有的例子中，类的方差直接在其声明中指定，并适用于所有使用类的地方。 Java不支持这一点，而是使用通配符来指定类的特定用途的差异。 我们来看看这两种方法之间的区别，看看如何在Kotlin中使用第二种方法。

##使用点变型 ：在类型出现的地方指定变型##
在类声明中指定方差注释的功能很方便，因为注释适用于所有使用类的地方。 这被称为声明站点差异。 如果您熟悉Java的通配符类型（？extends和？super），那么您会意识到Java处理差异的方式不同。 在Java中，每次使用具有类型参数的类型时，还可以指定是否可以使用其子类型或超类型替换此类型参数。 这被称为使用地点差异。

Kotlin与Java通配符中声明站点的差异

在Java中，为了创建符合用户期望的API，库编写者必须始终使用通配符：Function&lt;? super T, ? extends
R>。 如果您检查Java 8标准库的源代码，则可以在函数接口的每个用法中找到通配符。 例如，下面是如何声明Stream.map方法：
<pre><code>
/* Java */
public interface Stream&lt;T> {
		&lt;R> Stream&lt;R> map(Function&lt;? super T, ? extends R> mapper);
}
//在声明中指定一次方差使得代码更加简洁和优雅。
</code></pre>
Kotlin也支持使用，即使在类声明中不能声明为协变或逆变时，也可以指定特定类型参数的变化。 让我们看看这是如何工作的。
您已经看到，像MutableList这样的许多接口在一般情况下不是协变或逆变的，因为它们既可以产生也可以使用由它们的类型参数指定的类型的值。 但是，特定函数中的这种类型的变量通常只能用于其中一个角色：作为生产者或作为消费者。 例如，考虑这个简单的功能。
<pre><code>
fun &lt;T> copyData(source: MutableList&lt;T>, destination: MutableList&lt;T>) {
    destination += source
}
</code></pre>
为了使这个函数能够处理不同类型的列表，你可以引入第二个通用参数。
<pre><code>
fun &lt;T : R, R> copyData(source: MutableList&lt;T>, destination: MutableList&lt;R>) {
    destination += source
}
>>>val ints = mutableListOf(1, 2, 3)
>>>val anyItems = mutableListOf&lt;Any>()
>>>copyData(ints, anyItems)
</code></pre>
您声明两个通用参数，表示源和目标列表中的元素类型。 为了能够将元素从一个列表复制到另一个列表中，源元素类型应该是目标列表中元素的一个子类型。
但是Kotlin提供了一个更优雅的方式来表达这一点。 当一个函数的实现只调用在out（或者只在in）位置具有type参数的方法时，你可以利用它并在函数定义中为类型参数的特定用法添加方差注释。
<pre><code>
fun &lt;T> copyData(source: MutableList&lt;out T>, destination: MutableList&lt;T>) {
    destination += source
}

fun &lt;T> copyData(source: MutableList&lt;T>, destination: MutableList&lt;in T>) {
    destination += source
}

fun &lt;T> copyData(source: MutableList&lt;out T>, destination: MutableList&lt;in T>) {
    destination += source
}
>>>val ints = mutableListOf(1, 2, 3)
>>>val anyItems = mutableListOf&lt;Any>()
>>>copyData(ints, anyItems)
</code></pre>
使用地点预测可以帮助扩大可接受类型的范围。 现在我们来讨论一下极端情况：什么时候所有可能的类型参数都可以被接受。
##星号投影 ：使用 * 代替类型参数##
在本章前面谈到类型检查和强制转换的时候，我们提到了特殊的星型投影语法，可以用来表示没有关于泛型参数的信息。 例如，一个未知类型的元素列表使用List &lt;\*>这样的语法来表示。 我们来详细研究星形投影的语义。
首先，请注意，MutableList &lt;\*>与MutableList &lt;Any？>不一样（这里MutableList <T>在T上是不变的）。 MutableList &lt;Any？>是一个列表，您知道可以包含任何类型的元素。 另一方面，MutableList &lt;\*>是一个包含特定类型元素的列表，但不知道它是什么类型。 该列表是作为特定类型的元素（如String（不能创建新的ArrayList &lt;\*>））的元素列表创建的，创建它的代码期望它只包含该类型的元素。 因为你不知道类型是什么，你不能把任何东西放到列表中，因为你放在那里的任何值都可能违背调用代码的期望。 但可以从列表中获取元素，因为您确实知道存储在那里的所有值都将匹配Any类型，即所有Kotlin类型的超类型：
<pre><code>
>>>val list: MutableList&lt;Any?> = mutableListOf('a', 1, "qwe")
>>>val chars = mutableListOf('d', 'b', 'c')
>>>val unknownElements: MutableList&lt;*> = if (Random().nextBoolean()) list else chars

>>>unknownElements.add(42)
Error: Out-projected type 'MutableList&lt;*>' prohibits the use of 'public abstract fun add(element: E): Boolean defined in kotlin.collections.MutableList'
>>>println(unknownElements.first())
a
</code></pre>

 - MutableList&lt;\*>与MutableList&lt;Any？>不一样。
 - 编译器禁止你调用这个方法。
 - 获取元素是安全的，可以正常获取到元素。
为什么编译器将MutableList &lt;\*>引用为“out-projected”类型？ 在这种情况下，MutableList &lt;\*>充当MutableList &lt;out Any>>：当您对元素的类型一无所知时，获取Any的元素是安全的。 类型，但将元素放入列表中并不安全。 谈到Java通配符，Kotlin中的MyType &lt;\*>对应于Java的MyType&lt;？>。
对于逆变类型参数，如Consumer &lt;in T>，星型投影相当于&lt;in Nothing>。 实际上，你不能在这样的星型投影中调用任何签名中有T的方法。 如果类型参数是逆变的，它只作为一个消费者，而且，正如我们前面所讨论的那样，你并不知道它可以消费什么。 因此，你不能给它任何消耗。
当关于类型参数的信息不重要时，可以使用星型投影语法：不要使用引用签名中的类型参数的任何方法，或者只读取数据，而不关心其数据 具体类型。 例如，您可以实现以List <*>作为参数的printFirst函数：
<pre><code>
fun printFirst(list: List&lt;*>) {
    if (list.isNotEmpty()) {
        println(list.first())
    }
}
>>>printFirst(listOf("Svetlana", "Dmitry"))
Svetlana
</code></pre>
 - first（）现在返回Any？，但在这种情况下就足够了。

与使用站点差异的情况一样，您可以选择引入泛型类型参数：
<pre><code>
fun &lt;T> printFirst(list: List&lt;T>) {
    if (list.isNotEmpty()) {
        println(list.first())
    }
}
</code></pre>
 - first（）现在返回一个值T.
使用星型投影的语法更加简洁，但是只有当您对泛型类型参数的确切值不感兴趣时才有效：您只使用生成值的方法，而且您可以使用这些值，因为它们具有 Any？ 类型。
现在让我们来看看另一个使用星型投影的类型，以及在使用它时可能遇到的常见陷阱。 假设您需要验证用户输入，并声明一个接口FieldValidator。 它只包含在位的类型参数，所以可以声明为逆变。 而且的确，使用可以验证任何元素的验证器是正确的，当验证器的字符串是预期的（这就是声明它是逆变让你这样做）。 你还声明了两个验证器来处理String和Int输入。
<pre><code>
interface FieldValidator&lt;in T> {
    fun validate(input: T): Boolean
}

object DefaultStringValidator : FieldValidator&lt;String> {
    override fun validate(input: String): Boolean = input.isNotEmpty()
}

object DefaultIntValidator : FieldValidator&lt;Int> {
    override fun validate(input: Int): Boolean = input >= 0
}
</code></pre>
现在想象一下，你想要将所有的验证器存储在同一个容器中，根据输入的类型获得正确的验证器。 您的第一次尝试可能会使用map来存储它们。 您需要存储任何类型的验证器，所以您需要为KVlass（它代表一个Kotlin类，第10章将详细介绍KClass）声明一个映射到FieldValidator <*>（可能指任何类型的验证器）：
<pre><code>
>>>val validators = mutableMapOf&lt;KClass&lt;*>, FieldValidator&lt;*>>()
>>>validators[String::class] = DefaultStringValidator
>>>validators[Int::class] = DefaultIntValidator
</code></pre>
一旦你这样做，你可能会遇到困难，当试图使用验证。 您无法使用FieldValidator <*>类型的验证程序验证字符串。 这是不安全的，因为编译器不知道它是什么类型的验证器：
<pre><code>
>>>validators[String::class]!!.validate("")
Error:Out-projected type 'FieldValidator&lt;*>' prohibits the use of 'public abstract fun validate(input: T): Boolean defined in FieldValidator'
</code></pre>
 在这种情况下，这个错误意味着给一个未知类型的验证器赋予一个特定类型的值是不安全的。 解决这个问题的方法之一就是将验证器明确地转换为您需要的类型。 这是不安全的，不建议，但我们在这里显示它作为一个快速的技巧，使您的代码编译，以便您可以重构它：
<pre><code>
>>>val fieldValidator = validators[String::class]!! as FieldValidator&lt;String>
>>>println(fieldValidator.validate(""))
false
</code></pre>
编译器发出关于未经检查的强制转换的警告。 但是，请注意，此代码将仅在验证时失败，而不是在执行强制转换时，因为在运行时所有通用类型信息都将被擦除：
<pre><code>
>>>val fieldValidator = validators[Int::class]!! as FieldValidator&lt;String>
>>>println(fieldValidator.validate(""))
java.lang.ClassCastException:
	java.lang.String cannot be cast to java.lang.Number
	at DefaultIntValidator.validate
</code></pre>
解决方案使用相同的验证器映射，但是将所有对它的访问封装为两个通用方法，负责只有正确的验证器注册和返回。 此代码还会发出关于未经检查的强制转换（同一个）的警告，但此处对象Validators将控制对map的所有访问，从而保证不会有人错误地更改map。
<pre><code>
object Validators {
    private val validators = mutableMapOf&lt;KClass&lt;*>, FieldValidator&lt;*>>()  

    fun &lt;T : Any> registerValidator(kClass: KClass&lt;T>, fieldValidator: FieldValidator&lt;T>) {
        validators[kClass] = fieldValidator 
    }

    operator fun &lt;T : Any> get(kClass: KClass&lt;T>): FieldValidator&lt;T> = validators[kClass] as? FieldValidator&lt;T>
            ?: throw IllegalArgumentException("No validator for ${kClass.simpleName}")
}
>>>Validators.registerValidator(String::class, DefaultStringValidator)
>>>Validators.registerValidator(Int::class, DefaultIntValidator)
>>>println(Validators[String::class].validate("Kotlin"))
true
>>>println(Validators[Int::class].validate(42))
true
</code></pre>
现在你有一个类型安全的API。 所有不安全的逻辑都隐藏在课堂的正文中; 并通过本地化，保证它不能被错误地使用。 编译器禁止你使用不正确的验证器，因为Validators对象总是给你正确的验证器实现。
请注意，此模式可以轻松扩展以存储任何自定义泛型类。 将不安全的代码本地化在一个单独的地方防止误用，并使得容器的使用安全。
你现在应该更好地理解星型投影类型和安全处理它们的方法。 请注意，这里描述的模式不是特定于Kotlin; 你也可以在Java中使用相同的方法。
Java泛型和差异通常被认为是语言中最棘手的部分。 在Kotlin，我们努力想出一个更容易理解和更易于使用的设计，同时保持与Java的互操作性。
##总结##

 - Kotlin的泛型与Java中的泛型非常相似：用相同的方式声明泛型函数或类。
 - 和Java一样，泛型类型的类型参数只在编译时才存在。
 - 您不能使用带有类型参数的类型与is运算符一起使用，因为在运行时会删除类型参数。
 - 内联函数的类型参数可以被标记为通用的，这允许您在运行时使用它们来执行检查并获取java.lang.Class实例。
 - 方差（Variance）是一种指定类型的类型参数是否可以替代其子类或超类的方法。
 - 如果参数只用于out位置，则可以将类声明为类型参数的协变。
 - 相反的情况则适用于逆变情况：如果仅在位置中使用类，则可以在类型参数上声明类为逆变。
 - Kotlin中的只读接口List被声明为协变，这意味着List &lt;String>是List &lt;Any>的一个子类型。
 - 函数接口在其第一个类型参数中声明为逆变形，第二个函数接口声明为协变，这使（Animal） - > Int成为（Cat） - > Number的子类型。
 - Kotlin允许您为泛型类（声明站点方差）和特定类型（使用站点方差）指定方差。
 - 当确切类型参数未知或不重要时，可以使用星形投影语法。