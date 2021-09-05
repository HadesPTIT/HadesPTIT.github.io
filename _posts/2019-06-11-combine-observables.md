---
title: "[RxJava] Combining Observables"
date: 2019-08-11 21:15:03 +0700
categories: [Android]
tags: [android, rxjava]
---


## Combining Observables

Trong bài viết này mình sẽ cùng đi qua các toán tử kết hợp trong Rx thường được sử dụng để xem cách chúng hoạt động nhé.

* [startWith](https://github.com/ReactiveX/RxJava/blob/3.x/docs/Combining-Observables.md#startwith)
* [merge](https://github.com/ReactiveX/RxJava/blob/3.x/docs/Combining-Observables.md#merge)
* [mergeDelayError](https://github.com/ReactiveX/RxJava/blob/3.x/docs/Combining-Observables.md#mergedelayerror)
* [combineLastest](http://reactivex.io/documentation/operators/combinelatest.html)
* [zip](https://github.com/ReactiveX/RxJava/blob/3.x/docs/Combining-Observables.md#zip)

## startWith

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/ocur5iewn9_startwith.png)

* Toán tử **startWith** phát ra item được chỉ định trước khi bắt đầu phát các item từ Observable.

* Hoạt động với : `Flowable`, `Observable`

~~~java
Observable<String> names = Observable.just("Spock", "McCoy");
names.startWith("Kirk").subscribe(item -> System.out.println(item));

// Output : Kirk, Spock, McCoy
~~~

## merge

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/i9k344dy4w_merge.png)

* Toán tử **merge** Kết hợp các Observable lại với nhau thành một bằng cách hợp các emit của chúng lại.

* Hoạt động với : `Flowable`, `Observable`, `Maybe`, `Single`, `Completable`

* Nếu bất kì source nào trong chuỗi xảy ra lỗi, `onError()` sẽ được bắn ra ngay và observable sẽ bị terminate.

~~~java
Observable.just(1, 2, 3)
    .mergeWith(Observable.just(4, 5, 6))
    .subscribe(item -> System.out.println(item));

// Output : 1, 2, 3, 4, 5, 6
~~~

## mergeDelayError

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/seoas5z2nl_merge_delay_error.png)

* Tương tự với **merge**, **mergeDelayError** cũng kết hợp các Observable lại với nhau thành một bằng cách hợp các emit của chúng lại.
* Nếu bất kì source nào trong chuỗi xảy ra lỗi, `onError()` sẽ được giữ lại cho đến khi quá trình merge hoàn thành. Đây chính là điểm khác biệt so với toán tử **merge**.
* Hoạt động với : `Flowable`, `Observable`, `Maybe`, `Single`, `Completable`

~~~java
Observable<String> observable1 = Observable.error(new IllegalArgumentException(""));
Observable<String> observable2 = Observable.just("Four", "Five", "Six");
Observable.mergeDelayError(observable1, observable2)
        .subscribe(item -> System.out.println(item));

// emits 4, 5, 6 and then the IllegalArgumentException (in this specific
// example, this throws an `OnErrorNotImplementedException`).
~~~

## zip

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/tsg5dzb3oo_zip.png)

* Toán tử **zip** kết hợp tập các item được phát ra bởi 2 hay nhiều Observable lại thông qua một function chỉ định và phát ra các item dựa trên kết quả của function này.

* **zip** phát lượng item bằng lượng item được phát ra bởi source phát ít item nhất.
* Hoạt động với : `Flowable`, `Observable`, `Maybe`, `Single`

~~~java
Observable<String> firstNames = Observable.just("A", "B", "C");
Observable<String> lastNames = Observable.just("1", "2", "3");
firstNames.zipWith(lastNames, (first, last) -> first + " " + last)
    .subscribe(item -> System.out.println(item));

// Output: A 1, B 2, C 3
~~~

## combineLatest

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/9ncj2t083l_combineLatest.png)

* Khi một item được phát ra bởi một trong 2 source, nó sẽ kết hợp với item cuối cùng được phát bởi source còn lại thông qua function chỉ định và sẽ phát ra item dựa trên kết quả của function này.

* Hoạt động với : `Flowable`, `Observable`

~~~java
Observable<Long> newsRefreshes = Observable.interval(100, TimeUnit.MILLISECONDS);
Observable<Long> weatherRefreshes = Observable.interval(50, TimeUnit.MILLISECONDS);
Observable.combineLatest(newsRefreshes, weatherRefreshes,
    (newsRefreshTimes, weatherRefreshTimes) ->
        "Refreshed news " + newsRefreshTimes + " times and weather " + weatherRefreshTimes)
    .subscribe(item -> System.out.println(item));

// prints:
// Refreshed news 0 times and weather 0
// Refreshed news 0 times and weather 1
// Refreshed news 0 times and weather 2
// Refreshed news 1 times and weather 2
// Refreshed news 1 times and weather 3
// Refreshed news 1 times and weather 4
// Refreshed news 2 times and weather 4
// Refreshed news 2 times and weather 5
// ...
~~~

## Conclusion

Chúc ae coding vui vẻ :trollface: :trollface: :trollface: