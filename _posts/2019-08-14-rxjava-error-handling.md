---
title: "[RxJava] Error Handing"
date: 2019-08-14 21:15:03 +0700
categories: [Android]
tags: [android, rxjava]
---

## Introduction

Hi ae, trong bài viết này, mình sẽ cùng các bạn tìm hiểu về cách xử lý lỗi trong RxJava.

Đầu tiên, hãy nhớ rằng `Observable` thường không ném ra ngoại lệ. Thay vào đó nó thông báo cho các observers rằng có lỗi đã xảy ra bằng cách bắn callback `onError()`sau đó sẽ thoát khỏi luồng mà không gọi thêm bất kì phương thức nào của observer.

Khi xảy ra lỗi, ta thường cần xử lý nó theo một cách nào đó. Ví dụ thay đổi trạng thái của các đối tượng liên quan, tiếp tục chuỗi với giá trị mặc định hoặc chỉ đơn giản là kệ cho lỗi xảy ra...

Hãy cùng đi qua các operator sau nhé :
* [doOnError](http://reactivex.io/documentation/operators/do.html)
* [onErrorComplete](http://reactivex.io/documentation/operators/catch.html)
* [onErrorResumeNext](http://reactivex.io/documentation/operators/catch.html)
* [onErrorReturn](http://reactivex.io/documentation/operators/catch.html)
* [onErrorReturnItem](http://reactivex.io/documentation/operators/catch.html)
* [retry](http://reactivex.io/documentation/operators/retry.html)

## doOnError

~~~java
    Observable.error<Any>(IOException("Something went wrong"))
        .doOnError { error: Throwable -> println("The error message is: " + error.message) }
        .subscribe({
            println("onNext should never be printed!")
        }, {
            println("onError: " + it.message)
        }, {
            println("onComplete should never be printed")
        })

// The error message is: Something went wrong
// onError Something went wrong
~~~

* Với `doOnError()` ta có thể gọi bất kì hành động nào cần thiết khi có lỗi.
* `onError()` vẫn được bắn ra.
* Hoạt động với : Flowable, Observable, Maybe, Single, Completable

## onErrorComplete
~~~java
    Completable.fromAction { throw IOException() }
        .onErrorComplete()
        .subscribe(
            { println("IOException was ignored, onComplete still being called") },
            { error : Throwable? ->
                println("onError should not be printed!")
            }
        )

// IOException was ignored, onComplete still being called
~~~

* Với `onErrorComplete()`, khi xảy ra lỗi nó sẽ chặn thông báo từ `onError()` và thay thế bằng một sự kiện hoàn thành.
* Hoạt động với : `Completable`

## onErrorResumeNext

~~~java
        Observable.just(1, 2, 3, 4, 5)
            .doOnNext {
                if (it == 3) throw RuntimeException("Exception on 3")
            }
            .onErrorResumeNext(Observable.just(-1, -2))
            .subscribe({
                println("Value : $it")
            }, { error : Throwable ->
                println("onError should not be called")
            })

// Value : 1
// Value : 2
// Value : -1
// Value : -2
~~~

* Với `onErrorResumeNext()`, nó sẽ xử lý item trước khi đi tới onError() bởi vậy onError sẽ không được gọi.
* `onErrorResumeNext()` sẽ replace stream hiện tại với stream mới tại thời điểm xảy ra lỗi, vì vậy các item 4,5 không được phát.
* Hoạt động với : `Flowable`, `Observable`, `Maybe`, `Single`, `Completable`

## onErrorReturn

~~~java
        Observable.just("1", "2", "x", "xxx")
            .map { v -> Integer.parseInt(v) }
            .onErrorReturn { error: Throwable? ->
                if (error is NumberFormatException) return@onErrorReturn -1 else throw IllegalArgumentException()
            }
            .subscribe({
                println("Number : $it")
            }, {
                println("onError should not be printed")
            })

// Number : 1
// Number : 2
// Number : -1
~~~
* **onErrorReturn()** bắn ra item được trả về bởi `io.reactivex.functions.Function` trong block khi gặp lỗi.
* Hoạt động với : `Flowable`, `Observable`, `Maybe`, `Single`

## onErrorReturnItem

~~~java
        Single.just("2A")
            .map { v: String -> v.toInt(10) }
            .onErrorReturnItem(0)
            .subscribe(
                { x: Int? -> println(x) }
            ) { error: Throwable? -> System.err.println("onError should not be printed!") }
~~~
* Tương tự `onErrorReturn()`, nhưng sẽ bắn ra item cụ thể ngay khi gặp lỗi.
* Hoạt động với : `Flowable`, `Observable`, `Maybe`, `Single`

## retry

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/l3l6vg8i9m_Screen%20Shot%202020-07-05%20at%2023.43.21.png)

~~~java
        Observable.just(1, 2, 3, 4)
            .doOnNext {
                if (it == 3) throw RuntimeException("Exception on 3")
            }
            .retry(2)
            .subscribe({
                println("Value : $it")
            }, { error : Throwable ->
                println("Errors can occur due to retry failed")
            })

// Value : 1
// Value : 2
// Value : 1
// Value : 2
// Value : 1
// Value : 2
// Errors can occur due to retry failed
~~~

* `retry()` là một trong những cách xử lý lỗi. Nó chỉ đơn giản là cố gắng đăng kí lại Observable trước.
* Hãy cẩn thận khi sử dụng toán tử này, nếu bạn không xác định số lần retry, có thể nó sẽ rơi vào vòng lặp vô thời hạn.
* Hoạt động với : `Flowable`, `Observable`, `Maybe`, `Single`, `Completable`

## Conclusion

Chúc các bạn coding vui vẻ :wink: