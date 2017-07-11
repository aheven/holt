# Kotlin基础—变量（三）

    val question = "The Ultimate Question of Life, the Universe, and Everything"
	val answer = 42;

上述为kotlin声明变量的方法，与java不同的是kotlin声明变量以val或var关键字开始。这里省略了类型声明，但是也可以明确声明变量的类型：

    val answer: Int = 42;
如果变量没有初始化值，则在生命过程中必须明确指出变量类型：

    val answer: Int;
    answer = 42;
kotlin中有两个关键字来生命变量：
 
 * val - 常量，用val生命的变量初始化后不能重新分配值，对应java中的最终变量
 * var - 变量，可变变量，对应的值可以改变

默认情况下，应该尽量使用val声明kotlin中的所有变量，在有需要的时候，将其更改为var。val变量必须初始化一次，如果编译器可以确保val变量只被初始化一次，则可以根据不同的条件初始化不同的值：

    val message:String;
    if (canPerformOperation()){
        message = "Success"
    }else{
        message = "Failed"
    }

值得注意的是，val引用本身是不可变的，但指向的对象可能是可变的，例如：

    val languages = arrayListOf("Java")
    languages.add("kotlin")
即使var关键字允许变量改变其值，但是变量的类型也是固定的。比如，下面的代码无法通过编译器编译：

    var answer = 42
    answer = "no answer"
