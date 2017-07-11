# Kotlin基础—函数定义（二）

    fun main(args: Array<String>) {
	    println("Hello,world!")
    }

从上面的基本代码可以看出：

* `fun`字段用于声明一个函数
* 参数类型写在参数名的后面
* 函数可以在文件的顶部声明，不需要把它放在`class`内部
* 数组类型是一个类，与Java不同的是，数组在kotlin中并没有特殊的语法进行声明数组类型
* 使用`println()`替换`System.out.println()`，kotlin标准库为java库提供了许多包装器，使使用更加简便，`println()`函数只是其中之一。
* 跟许多现代化语言一样，kotlin在行末尾不需要；号

  ***
    fun max(a: Int, b: Int): Int {
        return if (a > b) a else b
    }

在kotlin中，fun修饰函数，其后跟函数名，参数在函数名的括号中显示，在括号的尾部，声明函数的返回值，没有声明函数返回值则默认为Void，可以省略不写。
![函数声明示例图](https://github.com/aheven/holt/blob/master/image/%E5%87%BD%E6%95%B0%E5%A3%B0%E6%98%8E%E7%A4%BA%E4%BE%8B%E5%9B%BE.png)
你可以简化上面的代码，因为上面函数中方法体中只有单个表达式，因此可以将整个表达式作为函数体的整体，删除大括号和返回语句：

    fun max(a: Int, b: Int): Int = if (a > b) a else b
因为kotlin在编译的时候能够自动推断类型，因此上面的代码可以写得更加简单

    fun max(a: Int, b: Int) = if (a > b) a else b

*tip:必须注意的是，只有具有表达式的函数才允许省略返回类型，对于具有函数体块体的函数，必须指定返回值并显示返回语句*

***
