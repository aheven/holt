# Kotlin基础—总结（八）
1. fun关键字用于声明一个函数，val关键字代表不可变变量，var关键字代表可变变量。
2. 字符串模板可以帮助我们避免类似于java的复杂字符串连接，用\$变量名或\${ }表达式将其注入到字符串中
3. 数据对象类（JavaBean）在kotlin中以更加简洁的形式存在
4. if关键字现在是一个具有返回值的表达式
5. when表达式类似于java中的switch，但是功能更加强大
6. 在检查完一个变量的类型后（is）,不必再显式地转换变量，编译器会智能转换
7. for、while、do-while循环和java依旧类似，但是在kotlin中，for循环显得更加智能，特别是对map或需要集合下标的时候
8. 简洁的语法1..5创建一个范围，范围和进度(step|downto)允许在for循环中使用，还可以是使用in或!in检查一个变量是否在一个范围内
9. kotlin中的异常处理方式与java的处理方式十分相似，但是kotlin不需要在使用一个方法时一定要抛出异常，比如`Integer.parseInt(line)`在使用的时候不强制抛出`NumberFormatException`