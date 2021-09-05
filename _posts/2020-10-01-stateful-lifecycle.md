---
title: "[Flutter] StatefulWidget Lifecycle"
date: 2020-10-01 21:15:03 +0700
categories: [Flutter]
tags: [flutter, dart, statefulwidget]
---

Hi mọi người !
<br/>
Trong bài viết này mình sẽ cùng tìm hiểu về vòng đời của 1 `StatefulWidget` nhé.

## Introduction

Khi Flutter build một [StatefulWidget](https://api.flutter.dev/flutter/widgets/StatefulWidget-class.html), nó sẽ tạo ra một [State](https://api.flutter.dev/flutter/widgets/State-class.html) object. Object này là nơi chứa tất cả các trạng thái có thể thay đổi cho widget đó.

Về concept, state là nơi mà thông tin:

* Có thể đọc 1 cách đồng bộ khi widget đã được build.
* Có thể thay đổi trong suốt thời gian tồn tại (lifetime) của widget.

<center>
<img src="https://miro.medium.com/max/1400/1*9kFme2gBIl9HtYFidAZw0w.jpeg" width="70%"/>
<p>Image by Jelena Jovanoski.</p>
</center>

Vòng đời của 1 StatefulWidget bao gồm những bước sau

* `createState()`
* `mounted == true`
* `initState()`
* `didChangeDependencies()`
* `build()`
* `didUpdateWidget()`
* `setState()`
* `deactive()`
* `dispose()`
* `mounted == false`

Bên dưới là 1 ví dụ của StatefulWidget

~~~dart
class BasicWidget extends StatefulWidget {
  const BasicWidget({Key? key}) : super(key: key);

  @override
  _BasicWidgetState createState() => _BasicWidgetState();
}

class _BasicWidgetState extends State<BasicWidget> {
  @override
  Widget build(BuildContext context) {
    return Container();
  }
}
~~~

## Tại sao StatefulWidget vaf State lại phải tách ra làm 2 class ?

=> Hiệu năng

Nếu đọc chi tiết về State, bạn sẽ thấy State object tồn tại lâu dài nhưng `StatefulWiget` (hay rộng hơn các lớp con của `Widget`) đều bị rebuild bất cứ khi nào có config thay đổi. Điều này giúp nó tốn rất ít chi phí để rebuild lại widget.

Bởi state không bị remove mỗi lần rebuild, nên nó tránh được các việc tính toán tốn kém và nhận được các thuộc tính state, get/set... ngay khi có data mới được update.

### 1. createState()

Đây là phương thức đầu tiên khi một StatefulWidget được sinh ra.

### 2. mounted == true

Sau khi `createState()` tạo state class, một BuildContext được gắn với state đó. Bạn có thể hiểu đơn giản rằng, buildContext là một thành phần dùng để đánh dấu vị trí của Widget trong Widget tree.

Tất cả các Widget đều có 1 thuộc tính kiểu `bool` là `mounted`. Giá trị này sẽ trả về true khi buildContext được gắn vào widget.

Thuộc tính này hữu ích trong 1 số trường hợp bạn gọi `setState()` mà buildContext chưa được gắn vào widget.

### 3. initState()

Đây là phương thước được gọi đầu sau khi widget được khởi tạo. Một điều chú ý rằng `initState()` chỉ được gọi duy nhất 1 lần.

Thông thường, ta hay override lại method này để:

* Khởi tạo data dựa trên BuildContext đã được gắn vào.
* Khởi tạo các thuộc tính dựa trên widget cha.
* Subscribe các stream, change notifier hoặc bất cứ object nào có khả năng thay đổi dữ liệu trên widget này.

~~~dart
@override
initState() {
  super.initState();
  // Add listeners to this class
  cartItemStream.listen((data) {
    _updateWidget(data);
  });
}
~~~

### 4. didChangeDependencies()

Phương thức này được gọi ngay sau khi `initState()` được gọi.

Khác với `initState`, `didChangeDependencies()` được gọi bất cứ khi nào đối tượng mà widget này phụ thuộc vào dữ liệu được gọi.

Ví dụ, nếu nó dựa trên một InheritedWidget, nó sẽ updates.

`build()` method luôn luôn được gọi ngay sau khi `didChangeDependencies()`, bởi vậy phương thức này ít khi được sử dụng đến.

Trong docs flutter có suggest rằng phương thức này hữu dụng nếu bạn cần network call hay một vài action tiêu tốn tài khác khi InheritedWidget được update.

Mình có tìm đọc được 1 bài ví dụ khá chi tiết về usecase sử dụng thằng này ở [đây](https://medium.com/flutterworld/differentiate-between-didchangedependencies-and-initstate-f98a8ae43164). Mọi người có thể đọc qua nhé.

### 5. build()

Phương thức này được gọi thường xuyên mỗi khi widget cần update lại.

### 6. didUpdateWidget(Widget oldWidget)

Phương thức này được gọi khi widget cha của widget hiện tại thay đổi và cần rebuild lại widget hiện tại. 

Một điều cần chú ý rằng phương thức `build()` luôn được gọi sau phương thức này, vì vậy việc setState ở đây là không cần thiết.

`didUpdateWidget()` thường được dùng để kiểm tra input đầu vào của widget có cập nhật gì mới không để xử lý 1 tác vụ nào đó.

~~~dart
@override
void didUpdateWidget(Widget oldWidget) {
  if (oldWidget.importantProperty != widget.importantProperty) {
    _init();
  }
}
~~~

### 7. setState()

Phương thức setState thường được sử dụng ngay trong flutter framework lẫn từ các developer. Nó được sử dụng để thông báo rằng "đã có dữ liệu thay đổi". Khi đó, cả widget hiện tại sẽ được build lại.

### 8. deactive()

Phương thức này hiếm khi được sử dụng. Phương thức được gọi khi State bị remove khỏi tree, nhưng nó có thể được insert lại trước khi kết thúc quá trình thay đổi khung hiện tại.

### 9. dispose()

Phương thức được gọi khi đối tượng State bị remove khỏi tree. Đây cũng là nơi để unsubscribe hay cancel animation, streams, etc.

### 10. mounted = false

Khi đối tượng state không còn tồn tại nữa, thuộc tính mounted sẽ trả về false. Đây cũng là lý do lỗi xảy ra khi ta gọi setState() khi widget chưa được mount.

### Conclution

Chúc các bạn coding vui vẻ :D
