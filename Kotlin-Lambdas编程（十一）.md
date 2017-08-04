# Kotlin-Lambdas编程（十一）
Lambda表达式本质上是可以传递给其他函数的小块代码。
## Lambda表达式和成员引用 ##
lambdas简介：代码块作为方法参数
在代码中传递和存储行为是一个很常见的需求，在旧版本的Java中，一般通过匿名内部类的方式完成这类功能。
函数式编程为您提供了另一种解决问题的方法：将功能视为值的能力。 而不是声明类并将该类的实例传递给方法，您可以直接传递函数。 使用lambda表达式，代码更简洁。 您不需要声明函数：相反，您可以有效地将代码块直接作为方法参数传递。
<pre><code>
/* Java */
button.setOnClickListener(new OnClickListener() {
	@Override
	public void onClick(View view) {
	/* actions on click */
	}
});
</code></pre>
 在Kotlin中，与Java 8一样，您可以使用lambda：
 
<pre><code>
/* Java */
button.setOnClickListener { /* actions on click */ }
</code></pre>

---
Lambdas与集合
我们来看一个例子。 您将使用包含人员姓名和年龄信息的Person类：
<pre><code>
data class Person(val name: String, val age: Int)
</code></pre>
假设不使用Lambda，要在一个Person的集合中找到年龄最大的人，你可能会这么做：
<pre><code>
data class Person(val name: String, val age: Int)

fun findTheOldest(people: List<Person>) {
    var maxAge = 0
    var theOldest: Person? = null
    for (person in people) {
        if (person.age>maxAge){
            maxAge = person.age
            theOldest = person
        }
    }
    println(theOldest)
}

fun main(args: Array<String>) {
    val people = listOf(Person("Alice", 29), Person("Bob", 31))
    println(findTheOldest(people))
}
</code></pre>
在kotlin中，有一个更好的方法。 您可以使用库函数：
<pre><code>
val people = listOf(Person("Alice", 29), Person("Bob", 31))
println(people.maxBy { it.age })
</code></pre>
如果一个lambda只是委托一个方法或属性，它可以被一个成员引用替换：
<pre><code>
people.maxBy(Person::age)
</code></pre>
我们通常使用Java中的集合（Java 8之前）的大多数事情可以通过使用lambdas或成员引用的库函数来更好地表达。 结果代码要短得多，易于理解。

---
lambda表达式的语法
![lambda表达式语法](https://github.com/aheven/holt/blob/master/image/lambda%E8%A1%A8%E8%BE%BE%E5%BC%8F%E8%AF%AD%E6%B3%95.png)
Kotlin中的lambda表达式始终被大括号包围。 请注意，参数周围没有括号。 箭头将参数列表与lambda的主体分开。
您可以将lambda表达式存储在变量中，然后像普通函数一样处理此变量（使用相应的参数调用它）：
<pre><code>
fun main(args: Array<String>) {
    val sum = { x: Int, y: Int -> x + y }
    println(sum(5, 6))
}
</code></pre>
如果你想，你可以直接调用lambda表达式：
<pre><code>
{ println(42) }()
</code></pre>
但是这样的语法是不可读的，并没有什么意义（它等同于直接执行lambda体）。 如果需要在块中包含一段代码，可以使用执行传递给它的lambda的库函数run：
<pre><code>
run { println(42) }
</code></pre>
回到寻找年龄最大的人的例子，如果你在不使用任何语法快捷方式的情况下重写这个实例：
<pre><code>
people.maxBy({ person -> person.age })
</code></pre>
花括号中的代码是一个lambda表达式，并将其作为参数传递给函数。 lambda表达式具有Person类型的一个参数，并返回其年龄。
但是这个代码是冗长的。 首先，有太多的标点符号，这会伤害可读性。 其次，该类型可以从上下文推断出来，因此省略。 最后，在这种情况下，您不需要为lambda参数指定一个名称。
 在Kotlin中，句法约定允许您将lambda表达式从括号中移出，如果它是函数调用中的最后一个参数。 在这个例子中，lambda是唯一的参数，因此可以放在括号之后：
 <pre><code>
 people.maxBy() { p: Person -> p.age }
</code></pre>
当lambda是函数的唯一参数时，还可以从调用中删除空的括号：
 <pre><code>
 people.maxBy { p: Person -> p.age }
</code></pre>
根据类型推断，可以省略参数类型
 <pre><code>
 people.maxBy { p -> p.age }
</code></pre>
与局部变量一样，如果可以推断出lambda参数的类型，则不需要明确指定它。使用maxBy函数，参数类型始终与集合元素类型相同。 编译器知道你在一个Person对象的集合中调用maxBy，所以它可以理解lambda参数也将是Person类型。
最后一个简化是用默认参数名称替换参数：it。如果上下文期望只有一个参数的lambda，则可以生成此名称，并且可以推断其类型：
 <pre><code>
 people.maxBy { it.age }
</code></pre>
it惯例是缩短你的代码，但你不应该滥用它。 特别地，在嵌套的lambdas的情况下，最好明确地声明每个lambda的参数; 否则很难理解它所指的含义。
如果将lambda存储在变量中，则没有可以推断参数类型的上下文，因此必须明确指定它们：
 <pre><code>
    val people = listOf(Person("Alice", 29), Person("Bob", 31))
    val getAge = { p: Person -> p.age }
    println(people.maxBy(getAge))
</code></pre>
到目前为止，你只看到一些由一个表达式或语句组成的lambdas的例子。 但是，lambdas并不限于这么小的大小，并且可以包含多个语句。 在这种情况下，最后一个表达式是结果：
 <pre><code>
 fun main(args: Array<String>) {
    val sum = { x: Int, y: Int ->
        println("Computing the sum of $x and $y")
        x + y
    }
    println(sum(5,8))
}
</code></pre>

---
范围内访问变量
您知道，当您在方法中声明一个匿名内部类时，可以从类中引用该方法的参数和局部变量。 在lambdas中，你可以做同样的事情。 如果在函数中使用lambda，则可以访问该函数的参数以及在lambda之前声明的局部变量。
 <pre><code>
 fun printMessagesWithPrefix(message: Collection<String>, prefix: String) {
    message.forEach {
        println("$prefix $it")
    }
}
fun main(args: Array<String>) {
    val errors = listOf("403 Forbidden", "404 Not Found")
    printMessagesWithPrefix(errors, "Error:")
}
</code></pre>
Kotlin和Java之间的一个重要区别是，在Kotlin中，你并不局限于访问最终变量。 您还可以从lambda中修改变量。
 <pre><code>
 fun printProblemCounts(responses: Collection<String>) {
    var clientErrors = 0
    var serverErrors = 0
    responses.forEach {
        if (it.startsWith("4")) {
            clientErrors++
        }else if (it.startsWith("5")){
            serverErrors++
        }
    }
    println("$clientErrors client errors,$serverErrors server errors")
}
fun main(args: Array<String>) {
    val responses = listOf("200 OK", "418 I'm a teapot", "500 Internal Server Error")
    printProblemCounts(responses)
}
</code></pre>
一个重要的注意事项是，如果将lambda用作事件处理程序或以其他方式异步执行，则仅当执行lambda时，才会对局部变量进行修改。 例如，以下代码不是计算按钮点击次数的正确方法：
<pre><code>
fun tryToCountButtonClicks(button: Button): Int {
    var clicks = 0
    button.onClick { clicks++ }
    return clicks
}
</code></pre>

---
成员引用
您已经看到了lambdas如何允许您将一段代码作为参数传递给函数。
但是，如果您需要作为参数传递的代码已经被定义为一个函数呢？
当然，您可以传递一个调用该函数的lambda，但这样做有点多余。 你能直接传递函数吗？
在Kotlin中，就像Java 8一样，如果将函数转换为值，您可以这样做。 您使用::运算符：
<pre><code>
fun main(args: Array<String>) {
    val getAge = Person::age
    println(getAge(Person("tom",15)))
}
</code></pre>
此表达式称为成员引用，它提供了一种简短的语法，用于创建仅调用一个方法或访问属性的函数值。 双冒号将类的名称与您需要引用的成员的名称（方法或属性）分离，如图5.2所示。
![成员引用语法](https://github.com/aheven/holt/blob/master/image/%E6%88%90%E5%91%98%E5%BC%95%E7%94%A8%E8%AF%AD%E6%B3%95.png)
这是比lambdas更简洁的一种表达式，它等价于：
<pre><code>
val getAge = { p: Person -> p.age }
</code></pre>
请注意，无论您是引用函数还是属性，不应在方法引用中将括号括在其名称之后。
成员引用与调用该函数的lambda具有相同的类型，因此可以互换使用两个：
<pre><code>
people.maxBy(Person::age)
</code></pre>
您可以引用一个在顶层声明的函数（并且不是类的成员），以及：
<pre><code>
fun salute() = println("Salute!")

fun main(args: Array<String>) {
    run(::salute)
}
</code></pre>
在这种情况下，您省略了类名，并以::开头。 成员引用:: salute作为参数传递给库函数run，它调用相应的函数。
提供一个成员引用而不是使用一个lambda代表一个函数，在使用几个参数是情况下很方便的：
<pre><code>
val action = { person: Person, message: String -> sendEmail(person, message) } 1.
val nextAction = ::sendEmail 2.
</code></pre>

 1. 这个lambda委托给sendEmail函数。
 2. 您可以使用成员引用替代lambda
您可以存储或推迟使用构造函数引用创建类的实例的操作。 构造函数引用是通过在双冒号后指定类名来形成的：
<pre><code>
fun main(args: Array<String>) {
    val createPerson = ::Person
    val person = createPerson("Alice", 19)
    println(person)
}
</code></pre>
创建“Person”实例的操作将保存为一个值。
请注意，您也可以以相同的方式引用扩展功能：
<pre><code>
	fun Person.isAdult() = age > 21
    val predicate = Person::isAdult
</code></pre>
虽然isAdult不是Person类的成员，但您可以通过引用访问它，就像您可以作为实例上的成员一样访问它：person.isAdult（）

---
## 关于集合的api ##
 filter 和 map 函数
filter和map函数构成操纵集合的基础。
 接下来还是从Person类开始：
 <pre><code>
 data class Person(val name: String, val age: Int)
</code></pre>
filter函数过滤掉不满足条件的元素
 <pre><code>
	 val list = listOf(1, 2, 3, 4)
    list.filter { it % 2 == 0 }
</code></pre>
如果你想只保留30岁以上的人，可以使用filter：
 <pre><code>
	 val people = listOf(Person("Alice", 29), Person("Bob", 31))
    people.filter { it.age > 30 }
</code></pre>
filter函数可以从集合中删除不需要的元素，但不会更改元素。
map函数将给定函数应用于集合中的每个元素，并将结果收集到新的集合中。
<pre><code>
	val list = listOf(1, 2, 3, 4)
    list.map { it * it }
</code></pre>
如果你想打印一个名单列表，而不是一个人的列表，你可以使用map函数转换列表
<pre><code>
	val people = listOf(Person("Alice", 29), Person("Bob", 31))
    people.map { it.name }
</code></pre>
请注意，此示例可以使用成员引用很好地重写：
<pre><code>
    people.map(Person::name)
</code></pre>
可以很容易地链接filter和map函数。 例如，我们打印30岁以上的人的姓名：
<pre><code>
	val people = listOf(Person("Alice", 29), Person("Bob", 31))
    people.filter { it.age > 30 }.map(Person::name)
</code></pre>
现在，让我们说你需要团队中年龄最大的人的名字，并返回所有年龄最大的人的列表。 使 lambdas编写这样的代码很容易：
<pre><code>
people.filter {
        it.age == people.maxBy(Person::age)?.age
    }
</code></pre>
但请注意，此代码重复了找到每个人的最大年龄的过程，所以如果收集中有100个人，搜索最大年龄将被执行100次！以下解决方案改进，并计算最大年龄只有一次：
<pre><code>
	val maxAge = people.maxBy(Person::age)!!.age
    people.filter {
        it.age == maxAge
    }
</code></pre>
你也可以在map中使用filter函数
<pre><code>
	val numbers = mapOf(0 to "zero", 1 to "one")
    println(numbers.mapValues { it.value.toUpperCase() })
</code></pre>

---
all, any, count和find
另一个常见的任务是检查集合中的所有元素是否与特定条件匹配。 在kotlin中，这是通过all和any函数来实现的。 count函数检查有多少个元素满足条件，find函数返回第一个匹配元素。
为了演示这些功能，我们来定义一个val canBeInClub27来检查一个人年纪小于等于27：
<pre><code>
val canBeInClub27 = { p: Person -> p.age &lt;= 27 }
</code></pre>
查找是否所有元素都满足条件：
<pre><code>
	val people = listOf(Person("Alice", 27), Person("Bob", 31))
    people.all(canBeInClub27)
</code></pre>
检查任意元素满足条件：
<pre><code>
	people.any(canBeInClub27)
</code></pre>
如果你想知道有多少元素满足条件，使用count：
<pre><code>
	println(people.count(canBeInClub27))
</code></pre>
注意`people.count(canBeInClub27)`的效率要高于`people.filter(canBeInClub27).size`
要查找满足条件的元素，使用find函数：
<pre><code>
	 println(people.find(canBeInClub27))
</code></pre>
如果有多个，则返回第一个匹配元素，如果没有满足谓词则返回null。一个与find类似的方法是`firstOrNull`

---
将list转换成map
想象一下，您需要根据一些参数将所有元素分成不同的组。 例如，您想要将同龄的人聚集在一起。 groupBy功能可以为您做到这一点：
<pre><code>
    val people = listOf(Person("Alice", 31), Person("Bob", 29), Person("Carol", 31))
    println(people.groupBy(Person::age))
    &gt;&gt;&gt;{31=[Person(name=Alice, age=31), Person(name=Carol, age=31)], 29=[Person(name=Bob, age=29)]}
</code></pre>
上述代码为将人的年龄作为键，将Person作为值的map
另一个例子，使用第一个字符串对list进行分组：
<pre><code>
	val list = listOf("a", "ab", "b")
    val groupBy = list.groupBy(String::first)
    println(groupBy)
    &gt;&gt;&gt;{a=[a, ab], b=[b]}
</code></pre>
请注意，这里first不是String类的成员，它是一个扩展函数。 不过，您可以作为成员参考访问它。

---
flatMap和flatten
写一个书籍的类：
<pre><code>
	class Book(val title: String, val authors: List&lt;String>)
</code></pre>
每本书都是由一位或多位作者撰写的。 您可以计算库中所有作者的集合：
<pre><code>
	list.flatMap(Book::authors).toSet()
</code></pre>
flatMap函数有两个步骤：首先根据作为参数给出的函数将每个元素转换（或映射）到一个集合，然后将几个列表合并到一个列表中：
<pre><code>
	val strings = listOf("abcd", "def")
    println(strings.flatMap(String::toList))
    &gt;&gt;&gt;[a, b, c, d, e, f]
</code></pre>
当您遇到必须组合成一个元素的集合的集合时，您可能会想到flatMap。 请注意，如果您不需要转换任何东西，只需要平铺这样的集合，就可以使用flatten函数：listOfLists.flatten（）

---
## sequences##
在上一节中，您看到了几个链接集合函数的示例，如map和filter。 这些函数将密切地创建中间集合，这意味着每个步骤的中间结果都存储在临时列表中。 序列给你一个替代的方式来执行这样的计算，避免创建中间临时对象。
有下面这样的例子：
<pre><code>
people.map(Person::name).filter { it.startsWith("A") }
</code></pre>
在kotlin标准函数中，所有的map和filter函数返回值都是list，上述代码意味着这个通话链将创建两个列表：一个用于保存过滤器功能的结果，另一个用于过滤的结果。当源列表包含两个元素时，这不是一个问题，但是如果您有一个百万元素，它的效率要低得多。
为了使其更有效率，您可以转换操作，使其使用序列而不是直接使用集合：
<pre><code>
people.asSequence() 1.
            .map(Person::name) 2.
            .filter { it.startsWith("A") } 2.
            .toList() 3.
</code></pre>

 1. 将初始集合转换为Sequence
 2. 序列支持与集合相同的API。
 3. 将生成的序列转换回列表
上述代码中没有创建用于存储元素的中间集合，因此，大量 元素的时候效率会明显好转。
Sequence接口的优点在于其实现方式。 序列中的元素被懒加载。 因此，您可以使用序列来有效地执行集合元素上的操作链，而无需创建集合来保存处理的中间结果。
您可以通过调用扩展方法asSequence将任何集合转换为序列。 您调用toList进行后向转换。

---
执行sequences操作
sequences中间部分总是实行懒加载，比如下面的例子：
<pre><code>
listOf(1, 2, 3, 4).asSequence().map { print("map{$it}");it*it }
            .filter {  print("filter($it) "); it % 2 == 0 }
</code></pre>
执行此代码段将不会向控制台打印任何内容。 这意味着map和filter转换将被推迟，只有当结果被执行时才会执行：
<pre><code>
listOf(1, 2, 3, 4).asSequence().map { print("map{$it}");it*it }
            .filter {  print("filter($it) "); it % 2 == 0 }.toList()
&gt;&gt;&gt;map{1}filter(1) map{2}filter(4) map{3}filter(9) map{4}filter(16) 
</code></pre>
在这个例子中需要注意的一个重要事情是执行计算的顺序。 初始化方法是首先调用每个元素上的map函数，然后在生成的序列的每个元素上调用filter函数。 这就是map和filter对集合的作用，而不是序列。 对于序列，所有操作都顺序应用于每个元素：第一个元素被处理（映射，然后过滤），然后处理第二个元素，依此类推。
下图说明了用渴望（使用集合）和懒惰（使用序列）方式评估此代码之间的区别。
![集合与序列链式处理](https://github.com/aheven/holt/blob/master/image/%E9%9B%86%E5%90%88%E4%B8%8E%E5%BA%8F%E5%88%97%E9%93%BE%E5%BC%8F%E5%A4%84%E7%90%86.png)

---
创建sequences
以前的示例使用相同的方法来创建序列：在集合中调用asSequence（）。另一种可能性是使用generateSequence（）函数。 该函数计算给定前一个序列的序列中的下一个元素。
例如，以下是如何使用generateSequence（）来计算最多100个自然数的总和：
<pre><code>
fun main(args: Array<String>) {
    val naturalNumbers = generateSequence(0) { it + 1 }
    val numbersTo100 = naturalNumbers.takeWhile { it &lt;= 100 }
    println(numbersTo100.sum())
}
</code></pre>
当获得结果“和”时，执行所有的延迟操作。
请注意，本示例中的naturalNumbers和numbersTo100是延迟计算的两个序列。 在调用终端操作（在这种情况下为总和）之前，不会评估这些序列中的实际数字。

---
## 使用java功能接口 ##
使用Lambdas与Kotlin库是不错的，但您使用的大多数API可能是用Java编写的，而不是Kotlin。 好消息是，Kotlin lambdas可以与Java API完全互操作; 在本节中，您将看到如何正常工作。
将lambda作为参数传递：
<pre><code>
button.setOnClickListener { /* actions on click */ }
/* Java */
public class Button {
public void setOnClickListener(OnClickListener l) { ... }
}
</code></pre>
OnClickListener接口声明：

<pre><code>
/* Java */
public interface OnClickListener
{ void onClick(View v);
}
</code></pre>
在Java（Java 8之前）中，您必须创建一个匿名类的新实例，将其作为参数传递给setOnClickListener方法：
<pre><code>
	/* Java */
    button.setOnClickListener(new OnClickListener() {
        @Override
        public void onClick(View v) {
            ...
        }
    }
</code></pre>
在Kotlin，你可以传递一个lambda：
<pre><code>
button.setOnClickListener { view -> ... }
</code></pre>
这是因为OnClickListener接口只有一个抽象方法。 这样的接口称为功能接口或SAM接口，其中SAM代表单一抽象方法。 Java API充满了诸如Runnable和Callable之类的功能接口以及与它们一起工作的方法。 Kotlin允许您在调用将功能接口作为参数的Java方法时使用lambdas，确保您的Kotlin代码保持干净和惯用

---
将lambda作为参数传递给Java方法
您可以将lambda传递给任何需要功能接口的Java方法。
<pre><code>
/* Java */
void postponeComputation(int delay, Runnable computation);
</code></pre>
在Kotlin中，您可以调用它并传递一个lambda作为参数。 编译器将自动将其转换为Runnable的实例：
<pre><code>
postponeComputation(1000) { println(42) }
</code></pre>
您可以通过创建一个明确实现Runnable的匿名对象来实现相同的效果：
<pre><code>
postponeComputation(1000, object : Runnable
{ 	override fun run() {
		println(42)
	}
})
</code></pre>
在大多数情况下，kotlin将自动将lambda转换为function的实例，不需要任何代码进行处理。 但有些情况下您需要显式执行转换。

---
SAM 构造方法 ：显式地把 lambda 转换成函数式接口
暂时搞不懂这一节

---
## 带接收者的 lambda ：“with”与“apply” ##
“with”函数
许多语言都有特殊的语句可用于在同一对象上执行多个操作，而不重复其名称。 Kotlin也有这个功能，但它使用with库函数来表示，而不是一个声明。
<pre><code>
fun alphabet(): String {
    val result = StringBuilder()
    for (letter in 'A'..'Z') {
        result.append(letter)
    }
    result.append("\nNow I know the alphabet!")
    return result.toString()
}
</code></pre>
在上述示例中，在每个调用中都用到了result名称，所以代码中存在许多result名称。要解决这个问题，需要用到with函数。
with函数是将某对象作为函数的参数，在函数块内可以通过 this 指代该对象。返回值为函数块的最后一行或指定return表达式。比如重写上面的示例：
<pre><code>
fun alphabet1(): String {
    val stringBuilder = StringBuilder()
    return with(stringBuilder) { 1.
        for (letter in 'A'..'Z') {
            this.append(letter) 2.
        }
        append("\nNow I know the alphabet1!") 3.
        this.toString() // 或return this.toString() 4.
    }
}
</code></pre>

 1. 指定您在其上调用方法的接收器值
 2. 调用接收器值的方法，但通过显式的“this”
 3. 调用方法，省略“this”
 4. 从lambda返回一个值
with结构看起来像一个特殊的构造，但它是一个函数，它接受两个参数：stringBuilder（在这种情况下）和一个lambda。 将lambda放在括号外的约定在这里工作，整个调用看起来像是语言的内置功能。 或者，您可以使用（stringBuilder，{...}）编写，但它的可读性较差。
让我们重构初始的stringBuilder变量：
<pre><code>
fun alphabet2() = with(StringBuilder()) {
    for (letter in 'A'..'Z') {
        append(letter)
    }
    append("\nNow I know the alphabet2!")
    toString()
}
</code></pre>
这个函数现在只返回一个表达式，所以使用expression-body语法重写它。 您创建一个新的StringBuilder实例，并将其作为参数直接传递，然后引用它。

---
“apply”函数
with是带有返回值执行lambda代码的结果，结果是lambda中的最后一个表达式。 但有时您希望调用返回接收器对象，而不是执行lambda的结果。 这就是使用apply函数的地方。
apply函数功能与with函数基本一样，唯一的区别就是with函数返回的是lambda计算后的值，而apply函数返回lambda接收的对象：
<pre><code>
fun alphabet3() = StringBuilder().apply {
    for (letter in 'A'..'Z') {
        append(letter)
    }
    append("\nNow I know the alphabet3!")
}.toString()
</code></pre>
正如你所看到的，apply是一个扩展功能。 该函数的接收者成为传递给参数的lambda的接收者。 执行apply的结果是StringBuilder，所以你调用toString将它转换成String。
当您使用预初始化属性创建对象的实例时，很有用的一个例子就是这样。 在Java中，通常通过单独的Builder对象来实现; 在Kotlin中，您可以使用任何对象的应用程序，无需任何特殊支持。
要了解如何在这种情况下使用apply（），我们来看一个创建一个具有一些自定义属性的Android TextView组件的例子：
<pre><code>
fun createViewWithCustomAttributes(context: Context) =
	TextView(context).apply{ 
	text = "Sample
	Text" textSize = 20.0
	setPadding(10, 0, 0, 0)
}
</code></pre>
apply函数允许您使用紧凑的方式封装函数。 您创建一个新的TextView实例，并立即将其传递给apply函数。 在要应用的lambda中，TextView实例成为接收者，因此可以调用方法并设置属性。 执行lambda后，应用返回
实例已经初始化了; 它将成为createViewWithCustomAttributes函数的结果。
buildString函数是一个优雅处理string的方案，用于在StringBuilder的帮助下创建一个任务：
<pre><code>
fun alphabet4() = buildString {
    for (letter in 'A'..'Z') {
        append(letter)
    }
    append("\nNow I know the alphabet4!")
}
</code></pre>

##  总结##

 - Lambdas允许您将代码块作为参数传递给方法
 - Kotlin可以让Lambdas传递给括号外的方法，并引用一个单独的lambda参数。
 - lambda中的代码可以访问和修改包含对lambda调用的函数中的变量。
 - 您可以通过使用::前缀函数的名称来创建对方法，构造函数和属性的引用，并将这些引用传递给方法而不是lambdas。
 - 集合的最常见操作可以在不手动迭代的情况下执行
 - 过滤元素，使用filter，map，all ，any 等函数
 - Sequences（序列）允许您组合集合上的多个操作，而不创建集合以保存中间结果。
 - 您可以将lambdas作为参数传递给使用Java功能接口（具有单个抽象方法（也称为SAM接口）的接口）作为参数的方法。
 - 带有接收器的Lambdas-----您可以在其中直接调用特殊接收器对象上的方法。
 - 使用with标准库函数允许您在同一对象上调用多个方法，而不重复对该对象的引用。 apply函数可以使用构建器样式的API构建和初始化任何对象。