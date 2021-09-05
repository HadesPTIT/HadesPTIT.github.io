---
title: "[Flutter] Dart’s null safety đã tối ưu ứng dụng của tôi như nào ?"
date: 2021-06-06 21:15:03 +0700
categories: [Flutter]
tags: [dart, flutter, null-safety]
---

## Giới thiệu

Chào mọi người ! <br/>
Hôm nay mình có đọc được 1 bài viết của một anh hiện đang là Google Developer Expert mảng Flutter. Bài viết chia sẻ về cách mà null safety đã tối ưu những dự án như thế nào. Hãy cùng tìm hiểu nhé.

> Nếu bạn chưa biết đến tính năng null-safety này của Flutter, hãy tìm hiểu trước ở [đây](https://dart.dev/null-safety/understanding-null-safety) nhé. 

Bài viết này bao gồm 2 chủ đề chính

* Migrate ứng dụng và các package lên null-safety
* Thay đổi cách code theo null-safety như nào

### Migrating ứng dụng và các package

Lần đầu nâng cấp Flutter lên phiên bản 2.0 (phiên bản hỗ trợ Null Safety) và cập nhật phiên bản dart SDK lên 2.12 trong tệp `pubspec.yaml` của ứng dụng Flutter, IDE đã cảnh báo tôi có rất nhiều lỗi trong project.

Ban đầu, tôi muốn thử làm thủ công bằng cách thêm `?` và `!` cho những phần code của mình mà không cần công cụ gì để migrate cả. Tôi cố ý thực hiện công việc này để để hiểu được những khó khăn mà Flutter team đã làm để tạo ra cái migration tool [này](https://dart.dev/null-safety/migration-guide#migration-tool). Nó giúp bạn chuyển đổi dự án của mình qua null-safety. Cuối cùng, tôi quyết định sử dụng công cụ tuyệt vời này để hoàn thành quá trình migrate cho ứng dụng của mình.

Ứng dụng này là ứng dụng thống kê các thông số có liên quan đến COVID-19 và nó hoàn toàn open source. Bạn có thể tìm thấy nó trên GitHub, mình để link ở cuối bài viết.

Thực sự thú vị khi thấy tất cả các code thay đổi mà công cụ chuyển đổi (migration tool) thực hiện trong dự án của tôi - những code thay đổi như thêm `?` trong giá trị Null (Nullable) và thêm `!` khi công cụ phát hiện ra rằng giá trị sẽ không bao giờ là giá trị rỗng.

![alt text](/resources/20210606/20210606_null_safe_img1.png)

Sau đây là một ví dụ về công cụ di chuyển tự động thêm `?` và `!`.

 `_homeCountry` là thuộc tính Nullable của một lớp có tên `HomeCountry` (cũng có thể nullable). Do đó, để bảo vệ quyền truy cập vào một trong các thuộc tính của `_homeCountry` (nếu class HomeCountry null thì bình thường nếu ta truy cập đến biến `_homeCountry` có thể gây crash ứng dụng), công cụ đã thêm toán tử `?`.‌‌
<br/>
<br/>

![alt text](/resources/20210606/20210606_null_safe_img2.png)

Sau khi thực hiện việc migrate, một số vấn đề trong code đã bắt đầu xuất hiện và rõ ràng hơn, đây là phần tuyệt nhất này.

![alt text](/resources/20210606/20210606_null_safe_img3.png)

Một trong những vấn đề xuất hiện đó là một số chuỗi string nullable đang được truyền dưới dạng danh sách trong plugin shared_preferences. Bởi vì các giá trị này có thể là Nullable, công cụ đã tạo toàn bộ danh sách kiểu <String?>[], lỗi xuất hiện vì nó chỉ chấp nhận kiểu <String>[].

Giải pháp đơn giản là loại bỏ `?` để kiểu dữ liệu truyền vào trùng với dữ liệu mong muốn. Sau khi tôi thực hiện điều này, bộ phân tích lại show ra một lỗi khác là `nullable type (String?) can’t be assigned to a non-nullable type (String)`

![alt text](/resources/20210606/20210606_null_safe_img5.png)

Để giải quyết vấn đề, tôi để các thuộc tính của class `HomeCountry` thành non-nullable và thêm keyword `required` vào. Điều này đồng nghĩa rằng các giá trị đó luôn được khởi tạo

Điều này giúp tôi không bị mắc sai lầm khi truyền các giá trị null tới `shared preferences`, điều này thật tuyệt.

Một issue khác mà null-safety giúp bạn tránh bị crash khi đang sử dụng ứng dụng. Hãy xem ví dụ bên dưới:

![alt text](/resources/20210606/20210606_null_safe_img6.png)

Bởi vì `list` có thể bị null vì vậy việc đọc giá trị của các phần tử dựa trên chỉ số của nó có thể dẫn tới crash. Sau khi migrate ứng dụng sang null-safety, bạn sẽ không biên dịch được ứng dụng nữa bởi chưa check null cho các giá trị này.

Ok, sau khi check null cho `list` thì đã compile được rồi. Thật ngạc nhiên khi việc migrate đã giúp tôi tìm ra một lỗi thực sự!

![alt text](/resources/20210606/20210606_null_safe_img7.png)

Tôi cũng đã thực hiện việc migrate cho một package rất nhỏ, bạn có thể tìm thấy package này trên [pub.dev](pub.dev), được gọi là `progress_indicators`. Tôi đã rất ngạc nhiên khi thấy cách công cụ migrate thêm các keyword `late` thay vì `?` khi nó kết luận rằng các biến đó đã được khởi tạo trước khi được sử dụng.

![alt text](/resources/20210606/20210606_null_safe_img8.png)


### Viết code như nào khi sử dụng null-safety ?

Giờ đây, Flutter đã có Null Safety, việc tạo ứng dụng mới là trải nghiệm tốt hơn cho lập trình viên. Viết code mới trong môi trường null-safe cũng giúp hiểu rõ hơn về các flow code, cùng với việc có thể viết mã crash-safe. 

Đoạn code dưới đây sẽ không thể biên dịch được:

~~~dart
class MyClass {
  String a;

  MyClass({this.a});
}
~~~

Đoạn code này gây ra lỗi compile ngay bởi biến a được đánh dấu là một biến nullable, đặt thêm từ khóa `required` trong hàm khởi tạo hoặc set giá trị default trong lúc khởi tạo. Lúc này, đoạn code mà bạn viết ra đảm bảo rằng biến a không bao giờ null. Vì vậy, tùy thuộc vào trường hợp sử dụng của bạn, bạn có thể thực hiện điều này:

~~~dart
class MyClass {
  String? a;

  MyClass({this.a});
}
~~~
hoặc như này
~~~
class MyClass {
  String a;

  MyClass({required this.a});
}
~~~
hoặc như này
~~~dart
class MyClass {
  String a;

  MyClass({this.a = ''});
}
~~~

> Chú ý rằng đừng thêm `@` vào trước keyword `required`, vì nó không có tác dụng từ Flutter2 (và Dart 2.12)

Bạn có thể tạo các class nullable tuỳ theo mục đích sử dụng

~~~dart
class MyClass {
  String? a;

  MyClass({this.a});
}
// somewhere in main code
MyClass? myClass;
~~~

Bởi vì myClass có kiểu nullable, vì vậy compiler sẽ báo lỗi nếu bạn viết như sau:

~~~dart
print(myClass.a);
~~~

> The property ‘a’ can’t be unconditionally accessed because the receiver can be ‘null’

Để sửa lỗi này, chỉ cần thêm `?` trước khi truy cập đảm bảo nó an toàn.

~~~dart
print(myClass?.a);
~~~

Tóm lại, `Sound null safety` của Dart là bước khởi đầu đáng tin cậy để xây dựng các ứng dụng an toàn hơn, nhanh hơn và đáng tin cậy hơn! Tôi khuyên bạn nên migrate các ứng dụng Dart cũ của mình sang null safety để hiểu cách hoạt động của nó. Có thể bạn sẽ đủ may mắn để tìm và sửa một số lỗi trong mã của mình!

Chúc các bạn coding vui vẻ :D

### Tham khảo

* https://github.com/wal33d006/novel_covid_19
* [How Dart’s null safety helped me augment my projects](https://medium.com/dartlang/how-darts-null-safety-helped-me-augment-my-projects-af58f8129cf)

