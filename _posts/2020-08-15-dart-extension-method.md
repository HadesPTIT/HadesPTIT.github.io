---
title: "[Flutter] Dart Extension Methods"
date: 2020-08-15 21:15:03 +0700
categories: [Flutter, Dart]
tags: [flutter, dart, extension-method]
---

### Introduction

Trước đây, có khá nhiều người chê `Dart` bởi nó thiếu các tiện ích mở rộng như **`Extensions`** - thứ mà các ae dev đã biết và sử dụng rất nhiều từ các ngôn ngữ như C#, Swift hay Kotlin...

Để chiều lòng các fan, tại [Google I/O 2019](.https://www.youtube.com/watch?t=1980&v=J5DQRPRBiFI&feature=youtu.be), đội ngũ Dart đã công cố 3 tính năng mới. Một trong số đó chính là [Extension Methods](https://dart.dev/guides/language/extension-methods).

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/hw3iq8lptu_dart_extension_functions.jpg)

Tính năng này đã được release từ phiên bản **`Dart 2.7`**. Trong bài viết này, mình sẽ cùng tìm hiểu xem có gì mới nhé.

### Enable Dart Extension Methods

Để bật tính năng này, chỉ cần phiên bản dart từ 2.7 trở lên:
~~~yaml
environment:
  sdk: '>=2.7.0 <3.0.0'
~~~

### How to create extension methods

Extension method được xây dựng theo cú pháp 
~~~dart
extension <'Tên Extension'> on <'Class cần extends'> {}
~~~
**Note:** Tên extension có thể có hoặc không đều được. Tốt nhất nên đặt để tránh conflicts khi khai báo.

Ví dụ :
~~~dart
// Import a library that contains an extension on String.
import 'int_x.dart';

extension on int {
  Duration seconds() => Duration(seconds: this);
}
~~~
~~~dart
final duration = 20.seconds();
~~~

### Extension method members

Extensions không chỉ xác định các method mà còn có thể xác định getter, setter hay các operators khác.

Ví dụ
~~~dart
extension NumberParsing on String {
  int parseInt() {
    return int.parse(this);
  }
  // ···
}
~~~
~~~dart
print('42'.parseInt()); // Use an extension method.
~~~

### Generic Extensions

Extension cũng có thể sử dụng generic type.

Bên dưới là 1 ví dụ sử dụng generic dựa trên List<T> 

~~~dart
extension MyFancyList<T> on List<T> {
  int get doubleLength => length * 2;
  List<T> operator -() => reversed.toList();
  List<List<T>> split(int at) => <List<T>>[sublist(0, at), sublist(at)];
}
~~~

### Conclusion

Chúc các bạn coding vui vẻ :D

### References

- [Extension methods](https://dart.dev/guides/language/extension-methods)