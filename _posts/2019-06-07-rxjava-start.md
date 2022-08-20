---
title: "[RxJava] Khái niệm"
date: 2019-07-06 21:15:03 +0700
categories: [Android]
tags: [android, rxjava]
---

## Giới thiệu

Khi tôi mới học Android, ai đó đã nói với tôi rằng "Nếu bạn muốn học thứ gì khó khó, hãy học RxJava".<br/>
Trong series này, tôi sẽ cùng các bạn tìm hiểu nó từ những điều đơn giản nhất nhé.

## 3 thành phần trong Rx

Trong RxJava, chỉ có 3 phần để tạo nên một chuỗi Rx điển hình. Đó là:
* Producer
* Consumer
* Operator

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/8iw37rt7jd_1_3DX0debK9TfJjDsQyBzRFg.png)

### Producer

Producer về cơ bản chịu trách nhiệm sản xuất dữ liệu. Và dữ liệu được sản sinh ra này chỉ có thể được sử dụng bởi người connect với nó (Consumer)

Về cơ bản, Producer bao gồm 5 loại :
* Observable
* Single
* Maybe
* Flowable
* Completable

Ví dụ :
~~~java
Single.just(1)
~~~
Ở đây, Single đơn giản là 1 loại Producer chỉ có thể sinh ra 1 thứ. 
Ta sẽ cùng tìm hiểu và đưa ra ví dụ chi tiết với các loại producer khác sau nhé.

### Consumer

Bây giờ, khi `Producer` có data, và `Consumer` hay gọi là người tiêu thụ muốn sử dụng nó, họ chỉ cần kết nối bằng cách sử dụng function `Subscribe`

~~~java
Single.just(1).subscribe{ it -> print(it) }
~~~
Trong trường hợp này, Consumer chỉ muốn in ra kết quả mà Producer đã tạo.

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/pmbq5eyhyn_2_6nS1lO1l9Wc_Ww25jSrNBg.png)

Như ban đầu giới thiệu, vậy còn khúc `operators` thì sao ?

## Operators

Để dễ mình họa các toán tử (operators) trong Rx, hãy cùng xem diagram sau để rõ hơn nhé : https://rxmarbles.com/

Lấy một ví dụ đơn giản, ta muốn sinh ra một chuỗi số từ 0->9 và chỉ nhận các giá trị chẵn. Rất may mắn, trong Rx cung cấp operator `filter` cho phép lọc các phần tử thỏa mãn điều kiện cho trước.

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/9s59aoicsa_3_60-KO2eSudB75IoVVavo6A.png)

Và một điều tuyệt vời hơn nữa, bạn có thể sử dụng nhiều hơn 1 operator miễn sao trước khi nó đến được tay `Consumer`

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/fo4ece9cz1_4_ystImfQS7wkeWuY3HmFclw.png)

### Conclusion

Hi vọng qua bài viết này các bạn có thể hiểu cơ bản qua về Rx và các thành phần bên trong nó. 
Trong bài viết tiếp theo mình sẽ cùng tìm hiểu Rx mạnh mẽ thế nào khi xử lý luồng nhé.
Happy coding !!!

### References

* [Learning RxJava](https://medium.com/@elye.project/learning-rxjava-in-android-made-simple-in-kotlin-observable-range-filter-ec605bb273a1)