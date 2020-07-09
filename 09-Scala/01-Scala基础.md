**Scala的六大特征：**

1.无缝混编

2.类型推测

3.并发和分布式

4.特质 trait

5.模式匹配

6.高阶函数：函数的参数是函数，，函数的返回时函数

**一、数据类型**

Scala与Java有着相同的数据类型（Java的8种基本数据类型和String类型）

Scala数据类型首字母大写，scala中数据类型都是对象，没有Java中的原生类型。

Byte Short Int Long Float Double Char Boolean String 

Scala特有的数据类型

Unit：表示无值，用法与Java中的void一样。 Null：空值或者空引用。 Nothing：是所有其他类型的子类型，表示没有值。 Any：是所有类型的超类，任何实例都属于Any类型。 AnyRef：所用引用类型的超类。 AnyVal：所有值类的超类。 

**二、==,eq()与equals()的区别**

**==：值相等返回true，不管类型**

Scala中的==与Java中不同，它是比较值是否相等，无论对象是否是相同的类型

List(1, 2, 3) == List(1, 2, 3) // true 1 == 1.0 // true 

**equals：类型相同，值相同返回true**

**同类型**

与==作用相同，都是比较值是否相等

"hello".equals("hello") // true 

**不同类型**

返回false

1.equals(1.0) // false 

**eq：引用比较,同一个对象返回true**

val str = new String("hello") "hello".eq(str) // false val str2 = "hello" "hello".eq(str2) // true 