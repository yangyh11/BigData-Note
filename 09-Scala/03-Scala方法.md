**1.****定义****方法**

/** 1.无参方法 */ def hello() = {  println("hello Scala") } /** 2.有参方法 */ def max(a: Int, b: Int) = {  if (a > b)    a  else    b } 

调用：

hello() println(fun3(5)) 

**2.****递归****方法，必须显式指返回值类型**

def fun3(num: Int): Int = {  if (num == 1)    num  else    num * fun3(num - 1) } 

调用：

println(fun3(5)) 

**3.参数****有默认值****的方法**

def fun4(name: String = "李四", age: Int = 19): Unit = {  println(s"我的名字是$name,今年$age 岁了") } 

调用：

fun4() fun4(age = 20) 

**4.****可变****参数的方法**

def fun5(elements: Int*) = {      println(elements)      // for循环      for (elem <- elements) {        print(s"$elem ")      } } 

调用：

fun5(2, 4, 6, 7, 4) 

**5.****匿名函数****,可以将匿名函数赋给一个变量**

// 1.有参的匿名函数 val myFun1 = (a: Int) => {  println(a) } // 2.无参数的匿名函数 val myFun2 = () => {  println("我爱Scala") } // 3.有返回值的匿名方法 val myFun3 = (a: Int, b: Int) => {  a + b } 

调用：

myFun1(1) myFun2() println(myFun3(2, 4)) 

**6.****嵌套****方法**

def fun7(num: Int) = {  def innerFun(a: Int, b: Int): Int = {    if (a == 1)      b    else      innerFun(a - 1, a * b)  }   innerFun(num, 1) } 

调用：

println(fun7(5)) 

**7.****偏应用****函数，是一种表达式**

def log(date: Date, s: String) = {  println(s"date is $date,log is $s") } 

调用：

val date = new Date() log(date, "aaa") log(date, "bbb") log(date, "ccc") log(date, "ddd") val logWithDate = log(date, _: String) logWithDate("aaa") logWithDate("bbb") logWithDate("ccc") 

**8.****高阶****函数**

**1.函数的参数是函数**

def highFun1(a: Int, b: Int, f: (Int, Int) => Int) = {  f(a, b) } 

调用：调用这个高阶函数,参数可以传递不同的函数

val f = (x: Int, y: Int) => x + y val sum = highFun1(10, 20, f) println(sum) val multiply = highFun1(10, 20, (x: Int, y: Int) => x * y) println(multiply) 

**2.函数的返回是函数**

def highFun2(): (Int, Int) => Int = {  def f2(a: Int, b: Int) = {    a + b  }  f2 } 

调用：

val num = highFun2() // 返回的是一个函数 println(num) println(num(1, 2)) 

**3.函数的参数和返回都是函数**

def highFun3(f: (Int, Int) => Int, a: Int, b: Int): (Int, Int) => Int = {  val num = f(a, b)   def f2 = (x: Int, y: Int) => {    x + y + num  }  f2 } 

调用：

val n = highFun3((i: Int, j: Int) => { i * j}, 3, 4)(1, 10) // 可以不写参数类型 val n = highFun3((i, j) => { i * j}, 3, 4)(1, 10) // 如果函数的参数在方法体中只使用了一次，那么可以写成_表示 val n = highFun3(_ * _, 3, 4)(1, 10) println(n) 

**9.函数，高阶函数的简化**

def curriedFun(a: Int, b: Int)(c: Int, d: Int) = {  a + b + c + d } 

调用：

println(curriedFun(1, 2)(3, 4)) 