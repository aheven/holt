# Kotlin基础—类、属性和包（五）

在java中，声明一个简单的JavaBean对象如下：

    public class Person {
    private final String name;

    public Person(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }
    }
同样的功能在kotlin中描述为：

    class Person(val name: String)
在现代语言中，许多语言都提供了比java更简洁的语言来声明javaBean形式的数据类，这种类型的类通常只包含类不包含其他逻辑代码。
请注意，在从java转换到kotlin的过程中，修饰符public消失了，这是因为在kotlin中默认的修饰符就是可见的public，所以可以省略它。
***
在Java中，数据通常存储在字段当中，而一般是私有的，我们通过setter/getter方法获取/设置数据。这种字段和其访问器的组合我们就称之为属性。在kotlin中，声明一个属性和声明一个变量是同样的形式，val为不可变量，var为可变变量。

    class Person(val name: String,
             var isMarried: Boolean)
	
* val-只读属性，生成了一个字段和一个getter方法
* var-可变属性，生成了一个字段和一个setter方法和一个getter方法

在kotlin中调用属性的方法如下：

    fun main(args: Array<String>) {
	    val person = Person("Bob", true)
	    println(person.name)
	    println(person.isMarried)
    }

* 在创建对象的时候不需要在使用new关键字
* kotlin中可以直接访问属性，但是实际上调用的是属性的setter/getter方法
***
大多数情况下，属性都具有存储该值的相应字段，但是，如果该值是动态的，这时候我们可以通过自定义属性来实现它，比如，声明一个矩形，判断矩形时候为正方形的时候不需要将正方形信息存储为一个单独的字段，因为可以直接比较正方形的宽高来判断矩形是否是正方形。

    class Rectangle(val width: Int, val height: Int) {
    val isSquare: Boolean
        get() {
            return width == height
        }
     }
上述代码还可以简写成：

    class Rectangle(val width: Int, val height: Int) {
	    val isSquare: Boolean get() = width == height
    }
***
kotlin的包机制与java是一致的，下面是一个显示了源文件和包声明语法的示例：

    package goemetry.shapes
    import java.util.Random
    class Rectangle(val height: Int, val width: Int) {
	    val isSquare: Boolean get() = width == height
	    fun createRandomRectangle(): Rectangle {
        val random = Random();
        return Rectangle(random.nextInt(), random.nextInt())
     }
    }
Kotlin 不支持 import static 语法。在kotlin中，可以将多个类放到同一个文件中，并可以随意命名，然而在大多情况下，遵循java的包结构依然是一个很好的做法。但是在kotlin类如果小的情况下，可以将相同功能的类放到同一个文件下。