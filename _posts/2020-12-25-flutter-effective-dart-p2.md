---
title: "[Flutter] Effective Dart (Part II)"
date: 2020-12-26 21:15:03 +0700
categories: [Flutter]
tags: [flutter, dart]
---

Tiếp tục phần trước, hôm nay mình sẽ chia sẻ 2 chủ đề còn lại là Usage Guide và Design Guide.

* [Usage Guide](https://dart.dev/guides/language/effective-dart/usage) – Làm cách nào để implement 1 đoạn mã sử dụng các statement hay expression hợp lý...

* [Design Guide](https://dart.dev/guides/language/effective-dart/design) – Đây là phần nhỏ nhưng scope rộng nhất. Cách khai báo, thiết kế consistent, usable APIs

## 3. USAGE GUIDE

Làm cách nào để implement 1 đoạn mã sử dụng các statement hay expression hợp lý...<br/>

### 3.1.1 DO use collection literals when possible.

- Dart có 3 kiểu collection chính : List, Map, Set. Nên khởi tạo như sau:

~~~dart

GOOD
var points = <Point>[];
var addresses = <String, Address>{};
var counts = <int>{};

BAD

var points = List<Point>();
var addresses = Map<String, Address>();
var counts = Set<int>();
~~~

### 3.1.2 DON’T use List.from() unless you intend to change the type of the result.

- Không nên sử dụng List.from() trừ khi bạn muốn thay đổi kiểu của kết quả trả về.

Ví dụ, có 1 iterable, có 2 cách để tạo ra 1 list mới chứa các phần tử tương tự

~~~dart
var copy1 = iterable.toList();
var copy2 = List.from(iterable);
~~
Dễ thấy, cách copy1 ngắn hơn. và quan trọng hơn là cách 1 giữ lại được kiểu (type) của đối tượng gốc

~~~dart
// Creates a List<int>:
var iterable = [1, 2, 3];

// Prints "List<int>":
print(iterable.toList().runtimeType);

/// WITH

// Creates a List<int>:
var iterable = [1, 2, 3];

// Prints "List<dynamic>":
print(List.from(iterable).runtimeType);
~~~

Còn nếu muốn thay đổi kiểu, sử dụng List.from() sẽ hợp lý với trường hợp sau:

~~~dart
var numbers = [1, 2.3, 4]; // List<num>.
numbers.removeAt(1); // Now it only contains integers.
var ints = List<int>.from(numbers);
~~~

### 3.1.3 DO use whereType() to filter a collection by type.

- Sử dụng whereType để lọc collection theo kiểu xác định

~~~dart
GOOD
var objects = [1, 'a', 2, 'b', 3];
var ints = objects.whereType<int>();

BAD
var objects = [1, 'a', 2, 'b', 3];
var ints = objects.where((e) => e is int);
~~~

### 3.2.1 Prefer function declarations over variables

- Trong Dart, thậm chí các functions cũng là object. Bên dưới là 1 số cách sử dụng liên quan đến function
- Dart cho phép khai báo local functions.

~~~dart

GOOD
void main() {
  localFunction() {
    ...
  }
}

BAD
void main() {
  var localFunction = () {
    ...
  };
}
~~~

### 3.2.2 DO remove unnecessary lambdas

- Nếu function invoke method với cùng argument truyền vào thì không cần wrap nó lại với lambdas

~~~dart
GOOD
names.forEach(print);

BAD
names.forEach((name) {
  print(name);
});
~~~

### 3.2.3 DO use = to separate a named parameter from its default value.

- Dart hỗ trợ gán default value bằng cả 2 cách "=" và ":".
= Để nhất quán, sử dụng "=" để set default value cho named params thay vì ":"

~~~
void insert(Object item, {int at = 0}) { ... }
~~~

### 3.3.1 DO follow a consistent rule for var and final on local variables.

- Tuân theo các quy tắc sau khi sử dụng var và final

* Sử dụng final cho biến không được assign lại giá trị và var cho biến được assign lại giá trị

### 3.3.2 AVOID storing what you can calculate.

- Không lưu lại các giá trị có thể tính toán.

Bên dưới là 1 ví dụ về sai lầm này...

~~~
BAD
class Circle {
  double radius;
  double area;
  double circumference;

  Circle(double radius)
      : radius = radius,
        area = pi * radius * radius,
        circumference = pi * 2.0 * radius;
}
~~~

* Thứ nhất, tốn tài nguyên, memory.
* Thứ hai, area và circumference là 2 giá trị có thể tự tính toán được mà không cần phải lưu lại.
* Thứ ba, radius trong trường hợp này là mutable, nếu thay đổi giá trị thì area và cir vẫn giữ giá trị cũ => sai

Vì vậy, hãy xây dựng lại theo format sau :
~~~dart
class Circle {
  double radius;

  Circle(this.radius);

  double get area => pi * radius * radius;
  double get circumference => pi * 2.0 * radius;
}
~~~

* Ngắn hơn, sử dụng ít bộ nhớ hơn và ít xảy ra lỗi hơn, không sợ việc mất đồng bộ dữ liệu vì chỉ có 1 source duy nhất.
* Trong 1 số trường hợp, bạn có thể phải cache lại kết quả nếu việc get tính toán mất nhiều thời gian, nhưng chỉ nên làm nếu performance đang gặp vấn đề.

### 3.4.1 PREFER using a final field to make a read-only property.

- Để đảm bảo dữ liệu được wrap lại và được trích xuất đúng nơi, nên set final

- Tất nhiên, nếu cần assign lại phạm vi internal, thì nên set private field và chỉ public getter.

~~~dart
class Box {
  final contents = [];

  double _data;

  double get data => _data;

  set data(double value) {
    _data = value;
  }

}
~~~

### 3.5.1 DO use initializing formals when possible.

- Nên init giá trị nếu có thể

~~~dart
GOOD
class Point {
  double x, y;
  Point(this.x, this.y);
}

BAD
class Point {
  double x, y;
  Point(double x, double y)
      : x = x,
        y = y;
}
~~~
=> this.syntax trong constructor được gọi là “initializing formal” (khởi tạo chính thức)

### 3.5.2 DO use ; instead of {} for empty constructor bodies.

- Dùng ; thay vì {} với constructor rỗng.

~~~dart
GOOD
class Point {
  double x, y;
  Point(this.x, this.y);
}
BAD
class Point {
  double x, y;
  Point(this.x, this.y) {}
}
~~~

### 3.5.3 DON’T use new.

- Từ Dart 2 trở đi, không cần sử dụng từ khoá "new"

### 3.6.1 AVOID catches without on clauses.

- Try/catch nên xác định loại error bằng "on" clauses.

## 4. DESIGN GUIDE

Duới đây là 1 số hướng dẫn để viết code tốt hơn
- Sử dụng cùng một pattern cho những mục đích giống nhau trong suốt các phần code của dự án.
- Việc quy ước này giúp việc đọc hiểu nhanh hơn, dễ tìm hơn

### 4.1.1 DO use terms consistently.

~~~dart
GOOD
pageCount         // A field.
updatePageCount() // Consistent with pageCount.
toSomething()     // Consistent with Iterable's toList().
asSomething()     // Consistent with List's asMap().
Point             // A familiar concept.
~~~

~~~dart
BAD
renumberPages()      // Confusingly different from pageCount.
convertToSomething() // Inconsistent with toX() precedent.
wrappedAsSomething() // Inconsistent with asX() precedent.
Cartesian       
~~~

### 4.1.2 AVOID abbreviations.

- Tránh viết tắt. Trừ khi những từ viết tắt phổ biến hơn full.

~~~dart
GOOD
pageCount
buildRectangles
IOStream
HttpRequest

BAD
numPages    // "Num" is an abbreviation of "number (of)".
buildRects
InputOutputStream
HypertextTransferProtocolRequest
~~~

### 4.1.3 PREFER a non-imperative verb phrase for a boolean property or variable.

- Sử dụng các cụm động từ cho thuộc tính hoặc biến kiểu boolean

~~~dart
GOOD
isEmpty
hasElements
hasShownPopup
canClose
canShowPopup
shouldConsume
wasShown
willFire

BAD
empty         // Adjective or verb?
withElements  // Sounds like it might hold elements.
closeable     // Sounds like an interface.
              // "canClose" reads better as a sentence.
closingWindow // Returns a bool or a window?
showPopup     // Sounds like it shows the popup.
~~~

### 4.1.4 CONSIDER omitting the verb for a named boolean parameter.

- Cân nhắc việc bỏ qua động từ cho 1 số named param kiểu boolean

~~~dart
GOOD
Isolate.spawn(entryPoint, message, paused: false);
var copy = List.from(elements, growable: true);
var regExp = RegExp(pattern, caseSensitive: false);
~~~

### 4.1.5 PREFER the “positive” name for a boolean property or variable.

- Ưu tiên đặt tên kiểu tích cực với boolean 

~~~dart
GOOD
if (socket.isConnected && database.hasData) {
  socket.write(database.read());
}
BAD
if (!socket.isDisconnected && !database.isEmpty) {
  socket.write(database.read());
}
~~~

### 4.1.6 DO follow existing mnemonic conventions when naming type parameters.

- Nên tuân theo các quy ước sau khi đặt tên cho các tham số kiểu

* Sử dụng E cho các phần tử trong collection
~~~dart
GOOD

class IterableBase<E> {}
class List<E> {}
class HashSet<E> {}
class RedBlackTree<E> {}

~~~
* Sử dụng K và V ( key & values) cho kiểu tập hợp
~~~dart
class Map<K, V> {}
class Multimap<K, V> {}
class MapEntry<K, V> {}
~~~
* Sử dụng R cho kiểu được sử dụng là giá trị trả về của phương thức

~~~dart
abstract class ExpressionVisitor<R> {
  R visitBinary(BinaryExpression node);
  R visitLiteral(LiteralExpression node);
  R visitUnary(UnaryExpression node);
}
~~~

### 4.2.1 AVOID defining a one-member abstract class when a simple function will do.
- Dart là một ngôn ngữ hướng đối tượng thuần tuý trong đó các đối tượng đều là instance của một lớp nào đó.
- Nhưng dart không yêu cầu tất cả code phải được định nghĩa bên trong class. Bạn có thể định nghĩa top-level variables, constants, functions...
- Tránh define abstract class với 1 member bên trong

~~~dart
GOOD
typedef Predicate<E> = bool Function(E element);

BAD
abstract class Predicate<E> {
  bool test(E element);
}
~~~

### 4.3.1 CONSIDER making your constructor const if the class supports it.

- Cân nhắc việc đưa constructor const nếu có thể

- Nếu bạn xây dựng 1 class mà tất cả các field là "final", và constructor không làm gì ngoài việc khởi tạo giá trị,
bạn nên để constructor const, điều này cho phép tạo các instance của class ở những nơi bắt buộc phải có yêu cầu const
như : bên trong const lớn hơn, switch case, default params values...

### 4.4.1 DON’T specify a return type for a setter.

- Setter luôn trả về kiểu void trong dart, vì vậy thêm vào rất vô nghĩa

~~~dart
GOOD
set foo(Foo value) { ... }

BAD
void set foo(Foo value) { ... }
~~~
