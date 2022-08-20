---
title: "[RxJava] Concat & Merge operator"
date: 2019-08-09 21:15:03 +0700
categories: [Android]
tags: [android, rxjava, concat, merge]
---

### Introduction

Hi mọi người, hôm nay chúng ta sẽ cùng tìm hiểu một số operator đơn giản và hữu ích trong Rx là Concat và Merge nhé.

### Concat

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/z18ftu4pkr_9_rx_concat.PNG)

Concat ghép nối đầu ra của các Observables rồi trả về một Observable duy nhất. Nó sẽ duy trì thứ tự của các Observable khi phát ra item. Tức là nó sẽ phát ra tất cả item của Observable thứ nhất, sau đó sẽ đến các item của Observable thứ hai và cứ như thế đến Observable cuối cùng. 

Giờ mình sẽ lấy ví dụ nhé :

~~~java
Observable.concat(
    Observable.just(1, 2, 3),
    Observable.just(4, 5, 6)
    ).subscribe(object : Observer<Int> {
        override fun onSubscribe(d: Disposable) {
        }
        override fun onNext(s: Int) {
            println(s)
        }
        override fun onError(e: Throwable) {
        }
        override fun onComplete() {
        }
    }
)
~~~

Kết quả :
~~~text
Value 1
Value 2
Value 3
Value 4
Value 5
Value 6
~~~

### Merge

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/eus25a582r_10_merge.PNG)

Merge ghép nối các item của các Observables rồi trả về một Observable duy nhất. Khác với concat, nó không duy trì trật tự trong khi phát ra các item, xen kẽ các item phát ra của các Observable khác nhau.

Hãy cùng xem ví dụ bên dưới :

Trong ví dụ này, mình sẽ tạo ra 2 observable cứ 1 giây lại phát ra 1 giá trị A,B.

~~~java
Observable.merge(
    Observable.interval(1, TimeUnit.SECONDS).map { id -> "A $id" },
    Observable.interval(1, TimeUnit.SECONDS).map { id -> "B $id" })
    .subscribe(object : Observer<String> {
        override fun onComplete() {
        }
        override fun onSubscribe(d: Disposable) {
        }
        override fun onNext(t: String) {
            Log.e("TAG", "Value $t")
        }
        override fun onError(e: Throwable) {
        }
    }
)
~~~

Kết quả :
~~~
Value A 0
Value B 0
Value A 1
Value B 1
Value A 2
Value B 2
Value A 3
Value B 3
...
~~~

Các giá trị A,B sẽ lần lượt xen kẽ nhau chứ không theo thứ tự hết A rồi mới đến B.

Để thấy được sự khác nhau giữa concat và merge. Mình sẽ lấy một ví dụ :

Trong ví dụ này, mình sẽ tạo ra 2 Observable, mỗi observable cứ 1 giây lại phát ra 1 item sau đó nối lại với nhau bằng toán tử concat.

~~~java
Observable.concat(
    Observable.interval(1, TimeUnit.SECONDS).map { id -> "A $id" },
    Observable.interval(1, TimeUnit.SECONDS).map { id -> "B $id" })
    .subscribe(object : Observer<String> {
        override fun onComplete() {
        }
        override fun onSubscribe(d: Disposable) {
        }
        override fun onNext(t: String) {
            Log.e("TAG", "Value $t")
        }
        override fun onError(e: Throwable) {
        }
    }
)
~~~
Kết quả :
~~~file
A0
A1
A2
A3
A4
...
An
~~~

Như đã nói ở trên, chỉ khi phát hết item ở Observable A thì Observable B mới được phát. Ở đấy cứ 1s thì Observable A lại phát, vì vậy Observable B sẽ không bao giờ phát ra được.

### Conclusion

Hi vọng qua bài viết này, các bạn có thể hiểu được sự khác nhau giữa 2 thằng này :D
Happy coding !!!





