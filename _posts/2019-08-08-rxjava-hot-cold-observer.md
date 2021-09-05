---
title: "[RxJava] HOT & COLD Observable"
date: 2019-08-08 21:15:03 +0700
categories: [Android]
tags: [android, rxjava]
---

## Introduction

Để hiểu được concept về HOT & COLD Observable, hãy nhìn vào những gì mà thằng `Producer` sản xuất ra. Bạn có thể hiểu đơn giản là như thế này :

> Khi dữ liệu được tạo ra bên ngoài Observable, ta gọi đó là HOT Observable.<br/>
> Khi dữ liệu được tạo ra bởi chính Observable, ta gọi đó là COLD Observable.
>

OK, giờ hãy cùng tìm hiểu tại sao lại nói như vậy nhé. 

## Just vs FromCallable ?

Mình sẽ lấy ra 1 ví dụ rất đơn giản về việc sử dụng `just` để cấp value cho `Single` Producer.

~~~java
Single.just(1).subscribe{ it -> print(it) } // Kết quả : 1
~~~
Tuy nhiên, có một function khác trông khá giống, và cũng cho ra kết quả tương tự đó là : `fromCallable`

~~~java
Single.fromCallable{ 1 }.subscribe{ it -> print(it) } // Kêt quả : 1
~~~
Mặc dù cùng cho ra 1 kết quả, nhưng `just` và `fromCallable` hoàn toàn khác nhau. Hãy cùng xem sự khác biệt.

## Given Externally vs Generated Internally

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/xjo6fxrbq_8_HHuzh9ZJHYbd8rgQn_Jrww.png)

`just` sử dụng value mà Single đã gửi trước đến cho nó. Trong khi `fromCallable` tạo ra value bên trong Single.

Trong ví dụ này, ta sẽ tạo ra 1 số ngẫu nhiên mỗi khi nó được gọi để thấy sự khác biệt :

~~~java
println("From Just")
val justSingle = Single.just(Random.nextInt())
justSingle.subscribe{ it -> println(it) }
justSingle.subscribe{ it -> println(it) }

println("\nFrom Callable")
val callableSingle = Single.fromCallable { Random.nextInt() }
callableSingle.subscribe{ it -> println(it) }
callableSingle.subscribe{ it -> println(it) }
~~~
Ta giữ lại thằng Single và dùng bằng 2 thằng Consumer ( được subscribe 2 lần ). Kết quả như sau :

~~~txt
From Just
801570614
801570614

From Callable
1251601849
2033593269
~~~

Bạn sẽ nhận thấy giá trị được phát ra bởi `just` giữ nguyên cho dù có nhiêu subcriber đi chăng nữa. Bởi giá trị được tạo ra bên ngoài Observable, và Observable chỉ giữ lại giá trị này cho các thằng Consumer sau mà thôi.

Tuy nhiên, đối với `fromCallable`, nó được tạo ra bên trong Observable. Vì vậy mỗi khi có ai đó subscribe, một số ngẫu nhiên mới lại được tạo ra.

## Executed Immediate vs Executed Lazily

* `just` có giá trị ngay cả khi chưa có ai subscribe nó.
* `fromCallable` chỉ thực hiện tạo giá trị khi có ai đó subscribe nó. Đây được gọi là `lazy initialization`. Điều này rất hữu ích trong lập trình khi có thể tiết kiệm một số tài nguyên khi chưa dùng tới.

Hãy cùng nhìn ví dụ bên dưới.

~~~java
fun main() {
    println("From Just")
    val justSingle = Single.just(getRandomMessage())
    println("start subscribing")
    justSingle.subscribe{ it -> println(it) }
   
    println("\nFrom Callable")
    val callableSingle = Single.fromCallable { getRandomMessage() }
    println("start subscribing")
    callableSingle.subscribe{ it -> println(it) }
}

fun getRandomMessage(): Int {
    println("-Generating-")
    return Random.nextInt()
}
~~~
Kết quả :

~~~
From Just
-Generating-
start subscribing
1778320787

From Callable
start subscribing
-Generating-
1729786515
~~~

Với `just` thì việc `Generating` xảy ra trước khi  `start subscribing`. Với `fromCallable` thì phải `start subscribing` sau đó mới `Generating`.

## An important comparison: Hot vs Cold Observable

Từ ví dụ về just và fromCallable. Mình sẽ có 

|       just    | fromCallable  |
| ------------- |:-------------:| 
| Giá trị được lấy từ bên ngoài     | Sinh ra từ bên trong | 
| Tạo ra giá trị TRƯỚC KHI đăng kí      | Tạo ra giá trị SAU KHI đăng kí      |   

Điều này tương tự với HOT & COLD observable

| HOT Observable        | COLD Observable           |
| ------------- |:-------------:| 
| Giá trị được lấy từ bên ngoài     | Sinh ra từ bên trong | 
| Tạo ra giá trị TRƯỚC KHI đăng kí      | Tạo ra giá trị SAU KHI đăng kí      |   

## Is one better than the other?

Vậy ta nên sử dụng cái nào ?

Điều đó phụ thuộc vào từng hoàn cảnh thường thì COLD Observable thường sẽ được sử dụng hơn, trừ khi :
* Bạn muốn đảm bảo nhiều người đăng kí (subscribers) nhận được cùng một dữ liệu.
* Bạn tạo ra một instance mới của một thứ gì đó mỗi lần Observable thực thi, ví dụ như websocket connection đi. Bạn không muốn tạo một connection mới cho mỗi subscriber nhưng thay vào đó chỉ chia sẻ cho những người đăng kí thôi. Di chuyển việc khởi tạo connection bên ngoài Observable sẽ khiến nó trở thành HOT và xử lý được vấn đề này.

## Conclusion

Hi vọng qua bài viết này có thể giúp bạn hiểu hơn cách hoạt động của Observable và ý nghĩa của COLD & HOT Observable. 
Happy coding !!!

## Reference 

* [Understand Hot & Cold Observable](https://medium.com/@luukgruijs/understanding-hot-vs-cold-observables-62d04cf92e03)
* [HOT & COLD Observable with just & fromCallable](https://medium.com/@elye.project/rxjava-2-understanding-hot-vs-cold-with-just-vs-fromcallable-3c463f9f68c9)
