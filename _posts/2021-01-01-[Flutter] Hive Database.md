---
title: "[Flutter] Hive Database"
date: 2020-08-15 21:15:03 +0700
categories: [Flutter]
tags: [dart, flutter, database]
---

Hive là một lightweight database xây dựng theo dạng key-value được viết bằng Dart thuần túy. 
Lấy cảm hứng từ [Bitcask](https://en.wikipedia.org/wiki/Bitcask)

## Introduction

https://docs.hivedb.dev/

- Hive là một cơ sở dữ liệu NoSQL đảm bảo đúng tiêu chí nhanh, gọn, nhẹ.
- Hive thực sự hữu ích nếu bạn bạn cần 1 CSDL dưới dạng key-value, ko có nhiều relationship và dễ dùng. 
- Hive lấy ý tưởng từ các hộp ( box - nơi chưa dữ liệu ) - mở ra trước khi sử dụng .
- Hive đã hỗ trợ lazy-loading và encrypt.

Trong phần I này, mình sẽ giới thiệu cơ bản về Hive và cách sử dụng nó.

Docs dựa trên Hive phiên bản sau:
~~~
dependencies:
  hive: ^1.4.4+1
  hive_flutter: ^0.3.1

dev_dependencies:
  hive_generator: ^0.8.2
  build_runner: ^1.11.1
~~~

## What are boxes ?

- Tất cả data được lưu trong Hive được tổ chức theo các hộp (box).
- Một box có thể được coi như 1 table trong SQL nhưng không có cấu trúc và có thể chứa mọi thứ.

## Open box ?

Để sử dụng được box, bạn cần mở nó trước.

~~~
var box = await Hive.openBox<E>('testBox');
~~~

Nếu `box` đã được mở, nó sẽ return ngay và các params truyền vào sẽ bị ignore.
Khi có được instance của box, bạn có thể đọc, ghi, xoá, sửa...

## Get open box ?

Hive lưu trữ 1 tham chiếu đến các box đã được mở. Nếu muốn mở một box đã được mở rồi :

~~~
var box = Hive.box('myBox');
~~~

Phương thức này rất hữu ích bởi bạn sẽ ko cần phải truyền qua lại box giữa các widgets.

## Close box ?

Nếu bạn ko cần sử dụng đến box nữa, hãy close nó. 
Tất cả key & value được lưu trong cache sẽ được xoá khỏi bộ nhớ.

~~~
var box = await Hive.openBox('myBox');
await box.put('hello', 'world');
await box.close();
~~~
**Note**: Trước khi đóng ứng dụng, bạn nên gọi `Hive.close()` để đóng toàn bộ các box lại.

## Type parameter: Box<E>

Khi gọi openBox, bạn có thể chỉ định rằng nó chỉ có thể chứa giá trị của 1 loại cụ thể. 

~~~
var box = await Hive.openBox<User>('users');

box.add(User());

box.add(5); // Compile time error
~~~
**Note**: 
* Box này có thể chứa subtype của User.
* Bạn cần cung cấp cùng kiểu params trong Hive.box(). Bạn ko thể mở cùng một box với các tham số kiểu khác nhau.

~~~
await Hive.openBox<User>('users');

Hive.box<User>('users'); // OK

Hive.box('users'); // ERROR
Hive.box<Dog>('users'); // ERROR
~~~

## Reading & Writing ?

### Reading 

- Việc đọc từ box rất đơn giản :

~~~
var box = Hive.box('myBox');

String name = box.get('name');

DateTime birthday = box.get('birthday');
~~~
Nếu như key không tồn tại, sẽ return `null`. 
Tuy nhiên, bạn cũng có thể set `defaultValue` trong trường hợp key không tồn tại.

~~~
double height = box.get('randomKey', defaultValue: 17.5);
~~~

Các list trả về bởi get() mặc định có dạng List<dynamic>. Sử dụng list.cast<SomeType>() để cast về kiểu cần dùng.

## Every object only exists once

Bạn sẽ luôn nhận được cùng 1 instance của một đối tượng với 1 key cụ thể.

## Writing

Việc ghi vào box gần giống như map. 
Tất cả các key phải là chuỗi string ASCII tối đa 255 kí tự hoặc số nguyên 32bit unsigned.

~~~
var box = Hive.box('myBox');

box.put('name', 'Paul');

box.put('friends', ['Dave', 'Simon', 'Lisa']);

box.put(123, 'test');

box.putAll({'key1': 'value1', 42: 'life'});
~~~

Bạn có thắc mắc tại sao việc write lại ko `async` ? Đây là 1 trong những điểm mạnh chính của Hive.

Các thay đổi được ghi vào disk dưới background nhưng tất cả listener sẽ được thông báo ngay lập tức. 

Nếu bạn muốn đảm bảo quá trình write thành công, chỉ cần await nó bởi Future.

=> Điều này khác với `lazyboxes` ở chỗ : nếu `Future` được trả về bởi function `put()` chưa hoàn thành thì khi gọi `get()` sẽ trả về giá trị cũ ( or null nếu chưa tồn tại).

~~~
var box = await Hive.openBox('box');

box.put('key', 'value');
print(box.get('key')); // value

var lazyBox = await Hive.openLazyBox('lazyBox');

var future = lazyBox.put('key', 'value');
print(lazyBox.get('key')); // null

await future;
print(lazyBox.get('key')); // value
~~~

## Delete 

Nếu bạn muốn xoá, xử dụng delete() 

**Note**: write `null` khác vơi delete();

~~~
var box = await Hive.openBox('writeNullBox');

  box.put('key', 'value');

  box.put('key', null);
  print(box.containsKey('key')); // TRUE

  box.delete('key');
  print(box.containsKey('key')); // FALSE
~~~

## Auto increment & indices

Hive hỗ trợ tự động tăng key => Điều này rất hữu ích để truy cập nhiều đối tượng, bạn có thể sử dụng box như một list.

~~~
  var friends = await Hive.openBox('friends');
  friends.clear();

  friends.add('Lisa');            // index 0, key 0
  friends.add('Dave');            // index 1, key 1
  friends.put(123, 'Marco');      // index 2, key 123
  friends.add('Paul');            // index 3, key 124

  print(friends.getAt(0));
  print(friends.get(0));
  
  print(friends.getAt(1));
  print(friends.get(1));
  
  print(friends.getAt(2));
  print(friends.get(123));
  
  print(friends.getAt(3));
  print(friends.get(124));
~~~

## Conclusion 

Chúc các bạn coding vui vẻ :D

## References

- [Hive Docs](https://docs.hivedb.dev/#/)