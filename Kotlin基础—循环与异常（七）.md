# Kotlin基础—循环与异常（七）
kotlin的while循环与java的while循环没有任何区别
<pre><code>
while (condition) {
/*...*/
}
do {
/*...*/
} while (condition)
</code></pre>

* 第一条只有满足了true才会执行语句
* 第二条，循环体无条件执行一次，之后才根据condition的值去判断是否继续执行。

***
kotlin使用范围的概念替换for循环中的变量初始化等定义
<pre><code>
fun fizzBuzz(i: Int) = when {
    i % 15 == 0 -> "FizzBuzz "
    i % 3 == 0 -> "Fizz "
    i % 5 == 0 -> "Buzz "
    else -> "$i "
}

fun main(args: Array<String>) {
    for (i in 0..100){
        println(fizzBuzz(i))
    }
}
</code></pre>
其中..表示kotlin中范围的概念，该表达式表示在i在0到100的范围内循环遍历（这里的范围总是包含左右两边的数字）。除了..，for循环中还有其他的关键字
<pre><code>
for (i in 100 downTo 0 step 2){
        println(fizzBuzz(i))
    }
</code></pre>

 1. downTo-i从100遍历以降序遍历到0
 2. step-步长，循环时可用，设置每次循环的增加或减少的量
上面的表达式表示100到0的偶数
for循环在map中的使用如下所示：
<pre><code>
fun main(args: Array&lt; String&gt;) {	
    val treeMap = TreeMap&lt; Char, String&gt;() 1.
    for (c in 'A'..'Z'){2.
        treeMap[c] = Integer.toBinaryString(c.toInt())3.4.
    }

    for ((key,value) in treeMap){5.
        println("$key = $value")
    }
}
</code></pre>
1.	使用treeMap，以便按键的顺序进行排序
2.	开始迭代从A到Z的字符
3.	将ASCII码转换为二进制码
4.	通过c键存储map中的值
5.	迭代map，将map的键和值分配给两个变量

可以使用相同的语法对集合进行序列的跟踪
<pre><code>
	val list = arrayListOf("10", "11", "1001")
    for ((index, element) in list.withIndex()) {
        println("$index:$element")
    }
</code></pre>

可以使用in关键字来检查一个值是否在范围内，例如：

    fun isLetter(c:Char) = c in 'a'..'z' || c in 'A'..'Z'
    fun isNotDigit(c: Char) = c !in '0'..'9'
***
<pre><code>
fun main(args: Array<String>) {
    val reader = BufferedReader(StringReader("1"))
    println(readNumber(reader))
}
fun readNumber(reader: BufferedReader): Int? {
    try {
        val line = reader.readLine()
        return Integer.parseInt(line)
    } catch (e: NumberFormatException) {
        return null
    } finally {
        reader.close()
    }
}
</pre></code>

kotlin的异常机制和java中的异常机制是一样的，与java不同的是,kotlin不强制进行异常检查，因此这代表你可以忽略异常。这种机制是因为java的机制通常需要大量无意义的代码去进行忽略处理异常，而且在规避异常错误的同时常常会引发另外的错误。