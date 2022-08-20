---
title: "[RxJava] Code gọn gàng hơn với doOnSubscribe, doFinally và doOnTerminate"
date: 2019-08-10 21:15:03 +0700
categories: [Android]
tags: [android, rxjava]
---

### Introduction

Là 1 fan của RxJava, khi sử dụng tôi nhận thấy rằng rất nhiều trường hợp sử dụng với RxJava tuân theo cùng một flow từ việc setup đến khi kết thúc logic. 
<br/>
Một usecase phổ biến là tải dữ liệu lên màn hình như sau:

1. Hiển thị trạng thái đang Loading.
2. Tải dữ liệu
3. Khi dữ liệu được tải thành công rồi thì ẩn trạng thái loading đi và cập nhật UI.
4. Hoặc khi có lỗi xảy ra, ẩn trạng thái loading đi và hiển thị lỗi.

OK, vậy khi thể hiện trong code nó sẽ như nào ?

~~~java
fun getDataFromServer() {
	
    view.showLoadingIndicator();

    loadFromServer()
            .compose(applySchedulers())
            .subscribe(data -> {
                view.hideLoadingIndicator();
                view.showData(data);
            }, error -> {
                view.hideLoadingIndicator();
                view.showError(error);
            });

    loadData()

}
~~~

Có lẽ bạn đã phát hiện ra điều gì đó rồi chứ, view.hideLoadingIndicator() bị gọi 2 lần. Có thể bạn thấy điều này không quá là vấn đề nhưng đây mới chỉ là ví dụ đơn giản mà việc hideLoading đã bị duplicate, với những trường hợp phức tạp hơn, việc hideLoading sẽ phải gọi nhiều lần trong chuỗi.
Có 2 lý do khiến mình không thích điều này :
* view.showLoadingIndicator() bị out khỏi chuỗi mặc dù nó được gắn với logic.
* Phần logic xử lý lỗi lại chứa code không xử lý lỗi.

### How to resolve ?

Trong phần này, mình muốn giới thiệu 3 toán tử tiện lợi giúp cho code trở lên clean hơn :
* **doOnSubscribe()** : sửa đổi nguồn source để nó gọi action được chỉ định **ngay** khi  được subscribe.
* **doOnTerminate()** : gọi action được chỉ định ngay **trước** khi Observable thông báo onError() hoặc onCompleted()
* **doAfterTerminate()** : gọi hành động được chỉ định ngay **sau** khi Observable rơi vào onError() hoặc onCompleted()
* **doFinally()** : gọi hành động được chỉ định **sau** khi Observable rơi vào onError() hoặc onCompleted() hoặc bị disposed bởi downstream.

Áp dụng những điều này cho ví dụ trên, kết quả sẽ như sau :

~~~java
fun getDataFromServer() {

    loadFromServer()
            .compose(applySchedulers())
            .doOnSubscribe(__ -> view.showLoadingIndicator())
            .doOnTerminate(() -> view.hideLoadingIndicator())
            .subscribe(data -> {
                view.showData(data);
            }, error -> {
                view.showError(error);
            });

    loadData();

}
~~~

Thay đổi nhỏ, nhưng nhìn đã clean hơn kha khá rồi phải không :D <br/>
Có bạn thắc mắc tại sao mình sử dụng **doOnTerminate()** thay vì **doFinally()** ?

=> Bởi vì trong trường hợp này, mình dispose observable sau khi screen bị destroy do đó mình không gọi các thao tác UI trong trường hợp này.

### Conclusion

Hi vọng bài viết này giúp bạn đánh giá cao lợi ích mà những thay đổi nho có thể mang lại. 
Happy coding !!!

### Reference 

* [Making rxjava code tidier with doOnSucsribe & doFinally](https://medium.com/@ValCanBuild/making-rxjava-code-tidier-with-doonsubscribe-and-dofinally-3748f223d32d)