# Kotlin基础—枚举与“when”（六）
在kotlin中声明枚举
<pre><code>
enum class Color(val r: Int, val g: Int, val b: Int) { 1.
    RED(255, 0, 0), ORANGE(255, 165, 0),
    YELLOW(255, 255, 0), GREEN(0, 255, 0), BLUE(0, 0, 255), 2.
    INDIGO(75, 0, 130), VIOLET(238, 130, 238); 3.

    fun rgb() = (r * 256 + g) * 256 + b 4.
}
</code></pre>

 1. 声明枚举常量的属性
 2. 指定每个常量创建时的属性值
 3. 这里的分号是必须的
 4. 定义枚举类上的方法

接下来使用一个枚举的例子
<pre><code>
fun getMnemonic(color: Color) =
        when (color) {
            Color.RED -> "Richard"
            Color.ORANGE -> "Of"
            Color.YELLOW -> "York"
            Color.GREEN -> "Gave"
            Color.BLUE -> "Battle"
            Color.INDIGO -> "In"
            Color.VIOLET -> "Vain"
        }
</code></pre>
在when中，可以在同一个分支使用多个值：
<pre><code>
fun getWarmth(color: Color) =
        when (color) {
            Color.RED, Color.ORANGE, Color.YELLOW -> "warm"
            Color.GREEN -> "neutral"
            Color.BLUE, Color.INDIGO, Color.VIOLET -> "cold"
        }
</code></pre>
在when中，可以使用复杂对象
<pre><code>
fun mix(c1:Color,c2:Color) =
        when(setOf<Color>(c1,c2)){
            setOf(Color.RED, Color.YELLOW) -> Color.ORANGE
            setOf(Color.YELLOW, Color.BLUE) -> Color.GREEN
            setOf(Color.BLUE, Color.VIOLET) -> Color.INDIGO
            else -> {
                throw Exception("Dirty color")
            }
        }
</code></pre>
上述代码中set是一个集合，如果set中包含相同的对象，则when会判断两个set是相等的。这主要是因为set的特性，在set中，set集合是无需且不可重复的。但是上述的代码其实是比较低效的，它仅仅创建为了检查便创建了两个set,可以通过无参数的when来优化它的性能：
<pre><code>
fun mixOptimized(c1: Color, c2: Color) =
        when {
            (c1 == Color.RED && c2 == Color.YELLOW)
                    || (c1 == Color.YELLOW && c2 == Color.RED) ->
                Color.ORANGE
            (c1 == Color.YELLOW && c2 == Color.BLUE) ||
                    (c1 == Color.BLUE && c2 == Color.YELLOW) ->
                Color.GREEN
            (c1 == Color.BLUE && c2 == Color.VIOLET) ||
                    (c1 == Color.VIOLET && c2 == Color.BLUE) ->
                Color.INDIGO
            else -> throw Exception("Dirty color")
        }
</code></pre>
如果没有为when语句提供任何表达式，则分支语句的条件为任何返回Boolean值的表达式，mixOptimized的功能和前面mix的功能是一样的，它的优点是不会产生任何对象，缺点是比较难读。
下面是一个计算(a+b)+c的例子，从中可以发现when还可以判断数据的类型，在类型判断后，在分支中再次使用不需要进行转换可以直接使用

<pre><code>
interface Expr

class Num(val value: Int) : Expr
class Sum(val left: Expr, val right: Expr) : Expr

fun evalWithLogging(expr: Expr): Int =
        when (expr) {
            is Num ->
                expr.value
            is Sum -> {
                val left = evalWithLogging(expr.left)
                val right = evalWithLogging(expr.right)
                left + right
            }
            else -> {
                throw IllegalArgumentException("Unknown expression")
            }
        }

fun main(args: Array<String>) {
    println(evalWithLogging(Sum(Sum(Num(-1), Num(2)), Num(4))))
}
</code></pre>