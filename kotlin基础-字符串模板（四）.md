# Kotlin基础—字符串模板（四）

    fun main(args: Array<String>) {
	    val name = if (args.size > 0) args[0] else "Kotlin"
	    println("Hello $name")
    }
上述例子讲述了字符串模板功能，跟许多脚本语言一样，kotlin允许放入$来引用代码中的变量，相当于java中的字符串连接`( "Hello, " + name + "!" )`，但是相对于java来说，kotlin的字符串更加紧凑。

如果需要在字符串模板中使用$符号，可以使用`println(\$)`替换。
kotlin的字符串模板不仅仅局限于变量的引用，还可以使用大括号进行更加复杂的表达式：

    fun main(args: Array<String>) {
	    println("Hello,${if (args.size > 0) args[0] else "everyone"}")
    }