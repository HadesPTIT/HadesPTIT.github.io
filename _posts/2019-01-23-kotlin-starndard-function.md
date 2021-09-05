---
title: "[Android] Kotlin standard functions: run-with-let-also-apply"
date: 2019-01-23 21:15:03 +0700
categories: [Android]
tags: [android, performance]
---

# Giới thiệu
[Standard.kt](https://github.com/JetBrains/kotlin/blob/master/libraries/stdlib/src/kotlin/util/Standard.kt) là một phần nằm trong Kotlin library. Trong đó định nghĩa một số hàm mở rộng như `let`, `apply`, `run` `with` `also` mà chắc hẳn bạn đã từng nhìn thấy trong các guide hay đã từng sử dụng chúng.

Trong bài biết này, ta sẽ cùng tìm hiểu cách để phân biệt sự khác nhau giữa chúng và khi nào thì sử dụng nhé :D

###  Scoping Functions

Một số function mà mình focus vào là *`run`, `with`, `T.run`, `T.let`, `T.also` và `T.apply`*. Ta gọi đó là những scoping function (hàm phạm vi) bởi chúng có chức năng chính là cung cấp scope nhỏ hơn từ hàm gọi nó.

Bên dưới là một ví dụ về scoping function :
~~~java
fun test() {
    var mood = "I am sad"

    run {
        val mood = "I am happy"
        println(mood) // I am happy
    }

    println(mood)  // I am sad
}
~~~
Với ví dụ này thì bạn có thể có một scope riêng biệt với giá trị mood được định nghĩa lại thành `I am happy` và nó chỉ tồn tại bên trong `run`.
Một hữu ích khác của `run` là nó trả về đối tượng cuối cùng nằm trong scope.

~~~java
    run {
        if (firstTimeView) introView else normalView
    }.show()
    
    // similarly
    
    if (firstTimeView) {
            introView.show()
    } else {
            normalView.show()
    }
~~~
Dễ thấy sử dụng `run` sẽ ngắn gọn hơn nhiều so với cách thông thường,

### 3 attributes of scoping functions
Để phân loại các scoping funtion, ta sẽ dựa vào những tiêu chí sau :

-	Normal vs. extension function
-	This vs. it argument
-	Return this vs. other type

#### 1. Normal vs. extension function

Nếu ta để ý `with` và `T.run` sẽ thấy 2 hàm này khá là giống nhau. Ví dụ bên dưới cho ra kết quả tương đương nhau :
~~~java
with(webview.settings) {
    javaScriptEnabled = true
    databaseEnabled = true
}
// similarly
webview.settings.run {
    javaScriptEnabled = true
    databaseEnabled = true
}
~~~
Tuy nhiên, khác nhau giữa chúng là một thằng là normal function (`with`), một thằng là extension function (`T.run`)
Vậy khi nào thì sử dụng `with`, khi nào thì sử dụng `T.run` ?

Hãy tưởng tượng nếu webview.settings có thể `null`, ví dụ bên trên sẽ được viết lại như sau :
~~~java
// Yack!
with(webview.settings) {
      this?.javaScriptEnabled = true
      this?.databaseEnabled = true
   }
}
// Nice.
webview.settings?.run {
    javaScriptEnabled = true
    databaseEnabled = true
}
~~~
Trong trường hợp này, `T.run` tốt hơn, bởi ta có thể check null trước khi sử dụng nó.

#### 2. This vs. it argument

Tiếp theo là `T.run` và `T.let`, cả 2 function khá giống nhau ngoại trừ cách mà chúng nhận tham số.

Hãy nhìn ví dụ bên dưới :
~~~java
stringVariable?.run {
      println("The length of this String is $length")
}
// Similarly.
stringVariable?.let {
      println("The length of this String is ${it.length}")
}
~~~
Nếu để ý vào chi tiết của `T.run`, bạn sẽ thấy `T.run` chỉ được tạo ra dưới dạng extension function gọi là T.() . Do đó tất cả bên trong scope, T có thể được gọi như `this`. Trong lập trình, phần lớn `this` đều có thể bỏ qua. Vì vậy, ta có thể sử dụng `$length` thay vì `${this.length}`.

Với `T.let`, bạn sẽ thấy `T.let` gửi chính nó vào` block: (T)`. và có thể được gọi trong function như `it`.

Từ đặc điểm của` T.let` và `T.run`, có vẻ như `T.run` tốt hơn `T.let`, nhưng `T.let` có một số ưu điểm như sau : 
-	`T.let` giúp phân biệt rõ ràng hơn biến thành viên và bên ngoài
-	Trong 1 số trường hợp this không thể bỏ qua, “it” ngắn hơn “this”
-	T.let cho phép đặt tên chuyển đổi, do đó bạn có thể đổi “it” sang một cái tên khác dễ hiểu hơn.
~~~java
stringVariable?.let {
      nonNullString ->
      println("The non null string is $nonNullString")
}
~~~

#### 3.Return this vs. other type

Giờ hãy để ý đến `T.let` và `T.also`, cả 2 đều giống nhau nếu ta nhìn vào phần thân của chúng :

~~~java
stringVariable?.let {
      println("The length of this String is ${it.length}")
}
// Exactly the same as below
stringVariable?.also {
      println("The length of this String is ${it.length}")
}
~~~

Tuy nhiên, sự khác biệt giữa chúng là đối tượng trả về. `T.let` cho phép trả về một kiểu giá trị khác trong khi `T.also` trả về chính nó T.

Cả 2 đều hữu ích đối với các hàm chuỗi (chaining function) khi mà `T.let` cho phép bạn triển khai các operation liên tục và `T.also` cho phép bạn biểu diễn trên cùng một variable. 
Dưới đây là một minh họa :
~~~java

val original = "abc"
// Evolve the value and send to the next chain
original.let {
    println("The original String is $it") // "abc"
    it.reversed() // evolve it as parameter to send to next let
}.let {
    println("The reverse String is $it") // "cba"
    it.length  // can be evolve to other type
}.let {
    println("The length of the String is $it") // 3
}
// Wrong
// Same value is sent in the chain (printed answer is wrong)
original.also {
    println("The original String is $it") // "abc"
    it.reversed() // even if we evolve it, it is useless
}.also {
    println("The reverse String is ${it}") // "abc"
    it.length  // even if we evolve it, it is useless
}.also {
    println("The length of the String is ${it}") // "abc"
}
// Corrected for also (i.e. manipulate as original string
// Same value is sent in the chain 
original.also {
    println("The original String is $it") // "abc"
}.also {
    println("The reverse String is ${it.reversed()}") // "cba"
}.also {
    println("The length of the String is ${it.length}") // 3
}

~~~

`T.also` có vẻ  không có tác dụng ở trên, vì ta có thể kết hợp chúng thành một khối chức năng duy nhất. Tuy nhiên, suy nghĩ kĩ một chút, chúng ta có thể thấy nó có một số điểm mạnh như:
- Tách rời rất rõ ràng trên cùng một đối tượng, tức là làm cho phần chức năng nhỏ hơn.
- Có thể tự thao tác trước khi được sử dụng, tạo ra một hoạt động xây dựng chuỗi.

Thử kết hợp `T.let` và `T.also` xem nào :

~~~java
// Normal approach
fun makeDir(path: String): File  {
    val result = File(path)
    result.mkdirs()
    return result
}

// Improved approach
fun makeDir(path: String) = path.let{ File(it) }.also{ it.mkdirs() }
~~~

### Looking at all attributes

Qua các tiêu chí trên, ta có thể thấy được khá đầy đủ về cách các function `run` `with` `let` `also` hoạt động. Vậy còn `T.apply` thì sao ?
- `T.apply` là một extension function
- `T.apply` gửi `this` như một tham số
- `T.apply` trả về `this`.

Dưới đây là ví dụ :

~~~java
// Normal approach
fun createInstance(args: Bundle) : MyFragment {
    val fragment = MyFragment()
    fragment.arguments = args
    return fragment
}
// Improved approach
fun createInstance(args: Bundle) 
              = MyFragment().apply { arguments = args }

~~~

~~~java
// Normal approach
fun createIntent(intentData: String, intentAction: String): Intent {
    val intent = Intent()
    intent.action = intentAction
    intent.data=Uri.parse(intentData)
    return intent
}
// Improved approach, chaining
fun createIntent(intentData: String, intentAction: String) =
        Intent().apply { action = intentAction }
                .apply { data = Uri.parse(intentData) }
                
 ~~~
 
###  Function selections

Dựa vào các đặc trưng và tính năng của từng loại mà bạn hãy quyết định sử dụng chúng một cách hợp lý nhé :D

![](https://images.viblo.asia/62508328-1287-48d0-b74f-2060a30dcbe3.png)


### References

1.  [https://medium.com/@elye.project/mastering-kotlin-standard-functions-run-with-let-also-and-apply-9cd334b0ef84](https://medium.com/@elye.project/mastering-kotlin-standard-functions-run-with-let-also-and-apply-9cd334b0ef84)