---
title: "[Android] Kotlin Sealed Class"
date: 2019-01-23 21:15:03 +0700
categories: [Android]
tags: [android, kotlin]
---

### Introduction

`Sealed class` được dùng để đại diện cho các class phân cấp bị hạn chế. Trong đó một giá trị chỉ có thể có một kiểu từ một tập có giới hạn và không thể có bất kì loại nào khác. Về mặt nào đó, `sealed class` là một phần mở rộng của `enum class` chỉ khác ở chỗ mỗi `enum` constant chỉ có một instance trong khi mỗi subclass của `sealed class` có thể có nhiều instance như một class thường vì vậy có thể chứa state trong các class này.

Giờ hãy cùng tìm hiểu cách implement một `sealed class` như thế nào nhé ;D

### Implementing Kotlin sealed classes

Để khai báo một sealed class, Bạn cần thêm modifier **sealed** trước tên của class.
~~~java
sealed class A {
    class B : A()
    class C : A()
}
~~~
Một sealed class là **abstract** và không thể được khởi tạo. Ví dụ :
~~~java
fun main(args: Array<String>) 
{
    var a = A() //compiler error. Class A cannot be instantiated.
}
~~~
Mặc định, các constructors của `sealed class` có modifier là `private`.

Tất cả các subclass của sealed class phải được khai báo trong cùng một file.

Sealed class rất quan trọng trong việc đảm bảo `type safety` bằng cách hạn chế tập các kiểu chỉ định tại thời điểm biên dịch (compile-time).
~~~java

sealed class A {

    class B : A()

    class C : A() 
    {
        class E : A() //this works.
    }

    init {
        println("Sealed class A")
    }

}

class D : A() //this works
{
    class F: A() //This won't work. Since sealed class is defined in another scope.
}
~~~

**Sealed class constructors**

~~~java
sealed class A(var name: String){
    
    class B : A("B")
    
    class C : A("C")
}

class D : A("D")
~~~

~~~java
fun main(args: Array<String>) {
    
    var b = A.B()
    var d = D()
}
~~~

**Data class & object in sealed class**

~~~java

sealed class A {

    class B : A()
    
    class C : A()
    
    object D : A()
    {
         fun name()
         {
             println("Object D")
         }
    }
    
    data class E(var name: String) : A()
}
~~~
~~~java
fun main(args: Array<String>) {

    val e = A.E("Anupam")
    println(e) //prints E(name=Anupam)

    var d = A.D
    d.name() //prints Object D
}
~~~

### Difference between enum and sealed classes

`Sealed class` cho phép ta tạo các thể hiện với nhiều kiểu khác nhau trong khi `enum` sử dụng cùng loại cho tất cả enum constants. 

Với enum bạn không thể làm được như dưới đây :
~~~java
enum class Months(string: String){
    January("Jan"), February(2),      // error
}
~~~
Bởi enum chỉ cho phép single type cho các hằng mà thôi.
Đây là cách mà sealed class giải quyết vấn đề này :
~~~java
sealed class Months {
    class January(var shortHand: String) : Months()
    class February(var number: Int) : Months()
    class March(var shortHand: String, var number: Int) : Months()
}
~~~
Giả sử bạn có một newfeed app, bạn có thể tạo các kiểu lớp khác nhau cho bài Post như Status, Image hay Video chẳng hạn.
~~~java
sealed class Post
{
    class Status(var text: String) : Post()
    class Image(var url: String, var caption: String) : Post()
    class Video(var url: String, var timeDuration: Int, var encoding: String): Post()
}
~~~
### Sealed classes and when

`Sealed class` thường được sử dụng với các câu lệnh `when` do mỗi lớp con và kiểu của nó biểu diễn một case. Hơn nữa, bởi sealed class giới hạn các kiểu nên khối `else` coi như bỏ.
~~~java
sealed class Shape {
    class Circle(var radius: Float): Shape()
    class Square(var length: Int): Shape()
    class Rectangle(var length: Int, var breadth: Int): Shape()
}

fun eval(e: Shape) =
        when (e) {
            is Shape.Circle -> println("Circle area is ${3.14*e.radius*e.radius}")
            is Shape.Square -> println("Square area is ${e.length*e.length}")
            is Shape.Rectangle -> println("Rectagle area is ${e.length*e.breadth}")
        }
~~~
Cùng thử tính toán với hàm `eval` xem :
~~~java
fun main(args: Array<String>) {

    var circle = Shape.Circle(4.5f)
    var square = Shape.Square(4)
    var rectangle = Shape.Rectangle(4,5)

    eval(circle)       //Circle area is 63.585
    eval(square)       //Square area is 16
    eval(rectangle)    //Rectangle area is 20
    //eval(x) //compile-time error.

}
~~~
**Note** : 
Modifier `is` kiểm tra nếu class đó thuộc một trong các kiểu đã cho và chỉ yêu cầu cho các class.
Với các objects thì sao ?
~~~java

sealed class Shape{
    class Circle(var radius: Float): Shape()
    class Square(var length: Int): Shape()
    object Rectangle: Shape()
    {
        var length: Int = 0
        var breadth : Int = 0
    }
}

fun eval(e: Shape) =
        when (e) {
            is Shape.Circle -> println("Circle area is ${3.14*e.radius*e.radius}")
            is Shape.Square -> println("Square area is ${e.length*e.length}")
            Shape.Rectangle -> println("Rectangle area is ${Shape.Rectangle.length*Shape.Rectangle.breadth}")
        }
~~~
### Conclusion
Hi vọng bài viết có thể giúp các bạn hiểu rõ hơn về sealed class và khi nào thì sử dụng nó một cách hợp lý.
Cảm ơn các bạn đã đọc bài viết. Happy coding ^^

### References
1. [Kotlin Docs](https://kotlinlang.org/docs/reference/sealed-classes.html)
2. [Kotlin Sealed Class](https://www.journaldev.com/18719/kotlin-sealed-class#difference-between-enum-and-sealed-classes)