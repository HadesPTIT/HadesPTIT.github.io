---
title: "[RxJava] Xử lý luồng trong Android"
date: 2019-07-08 21:15:03 +0700
categories: [Android]
tags: [android, rxjava, thread]
---

# Giới thiệu

Xử lý luồng luôn là một chủ đề khó trong lập trình. Và nó cực kì quan trọng trong Android, nhất là khi ta phải đưa tất cả luồng xử lý network, database, tính toán... xuống tác vụ nền để tránh block UI.

Vì vậy, không có cách nào để để thoát được việc phải học nó cả. Nhưng yên tâm, mọi thứ sẽ trở nên đơn giản hơn với RxJava.

## Rx Diagram

Trong bài viết trước, mình đã đề cập đến RxJava có thể được chia thành 3 phần đơn giản : Producer, Operators và Consumer.

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/2bz9kic96m_5_ystImfQS7wkeWuY3HmFclw.png)

Hãy tưởng tượng rằng, một số tác vụ tiêu tốn thời gian cần thực hiện tưới nền và chỉ `Consumer` chạy trên UI thread.

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/xwb673z5ev_6_RsRmerTl2mhwm1R8ZQhtnQ.png)

## observeOn(…)

Để xác định xem chuỗi operators được thực hiện trên luồng nào, ta chỉ cần đơn giản gắn thêm chuỗi **observeOn** ứng với thread mong muốn.
![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/uoapu98yoe_7_ZlzL7BOXR4hVuMjNsaVE3w.png)

Nó sẽ tương đương như sau:

~~~java
producer().observedOn(Schedulers.computational())
          .operation1()
          .operation2()
          .observedOn(Schedulers.io())
          .moreOperations()
          .observedOn(AndroidScheduler.mainThread())
          .subscribe(...)
~~~

Nhìn khá simple đúng không ? :D

## Có thể bạn sẽ đặt ra 1 câu hỏi...

Thế còn xử lý phía Producer thì sao ?

Giả sử hoạt động của producer rất chậm, và ta cũng muốn đưa nó vào background thì phải làm thế nào ?

~~~java
Observable.fromCallable { superSlowProcess() }.subscribe{ ... }
~~~

## subscribeOn(…)

Tương tự observeOn(...), sử dụng subscribeOn(...) nhưng nó khác ở điểm, bạn có thể subscribeOn(...) ở bất cứ đâu trong chuỗi ( sau Producer và trước Consumer). 
![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/pmtae8trri_7_bfXuF2uAWdCkbLNkiahAvQ.png)

Mã code sẽ trông như thế này:

~~~java
Observable.fromCallable { superSlowProcess() }
          .operators // or operators...
          .subscribeOn(Schedulers.io())          
          .observeOn(AndroidScheduler.mainThread())
          .subscribe{ ... }
~~~

Hoặc bạn cũng có thể viết như bên dưới đây, cùng trả về một kết quả, chỉ là ta có thể dễ dàng thấy nới đăng kí và quan sát gần nhau hơn. Đây cũng là cách mình thường xuyên dùng ::lol::

~~~java
Observable.fromCallable { superSlowProcess() }
          .operators // or operators...
          .subscribeOn(Schedulers.io())          
          .observeOn(AndroidScheduler.mainThread())
          .subscribe{ ... }
~~~

## Conclution

OK. Hi vọng qua bài viết này các bạn có thể hiểu qua về cách xử lý luồng trong Rx. 
Bài viết tới mình sẽ cùng tìm hiểu về HOT Observable và COLD Observable xem nó là gì và tác dụng từng loại nhé. Happy coding !!!

## Reference

* [Making threading easy in Android with RxJava](https://medium.com/@elye.project/rxjava-2-making-threading-easy-in-android-in-kotlin-603d8342d6c)


