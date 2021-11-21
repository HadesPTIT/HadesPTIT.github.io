---
title: "[Flutter] ShrinkWrap vs Slivers | Decoding Flutter"
date: 2021-11-20 21:15:03 +0700
categories: [Flutter]
tags: [dart, flutter, flutter performance, listview]
---

## **Introduction**

Một trong những widget cơ bản mà các developer sử dụng thường xuyên đó là **ListView**.
<br/>
ListView rất dễ dùng, nhưng để tối ưu nó thì không hẳn ai cũng biết. Trong bài viết này, mình sẽ cùng đi vào một usecase rất phổ biến đó là lồng listview trong listview và bài toán performance.

Ví dụ bên dưới đây cho bạn thấy cách mà ta lồng các ListView con vào 1 ListView thông thường.

~~~dart

ListView(
  // Setting `shrinkWrap` to `true` here is both unnecessary and expensive.
  children: <Widget>[
    ListView.builder(
      itemCount: list1Children.length,
      itemBuilder: (BuildContext context, int index) {
        return list1Children[index];
      },
      // This forces the `ListView` to build all of its children up front,
      // negating much of the benefit of using `ListView.builder`.
      shrinkWrap: true,
    ),
    ListView.builder(
      itemCount: list2Children.length,
      itemBuilder: (BuildContext context, int index) {
        return list2Children[index];
      },
      // This forces the `ListView` to build all of its children up front,
      // negating much of the benefit of using `ListView.builder`.
      shrinkWrap: true,
    ),
    ...
  ],
)
~~~

Hãy cùng xem lại về định nghĩa của `shrinkWrap` ở [đây](https://api.flutter.dev/flutter/widgets/ScrollView/shrinkWrap.html)

> Việc set `shrinkWrap = true` sẽ force toàn bộ listview bên trong của bạn tính toán lại sao cho chiều cao của list là hữu hạn thay vì chiều cao thông thường của các listview là vô cùng. Điều này sẽ gây giảm hiệu năng hơn đáng kể bởi kích thước của scrollview luôn cần được tính toán lại bất cứ khi nào vị trí scroll thay đổi.

Giờ thì bạn đã hiểu tại sao phần code bên trên sẽ khá `tốn kém` rồi phải không :D

Có 2 vấn đề bạn cần phải để ý :

1. `ListView` bao ngoài cùng đang không set `shrinkWrap = true`, mà chỉ listview con cần set mà thôi
2. `ListView.Builder` hiệu quả trong việc build các thành phần con của nó, tránh việc phải build những widget không thuộc viewport. Nhưng việc set `shrinkWrap = true` sẽ overrides lại cách làm này, khi đó ListView.Builder sẽ không còn ý nghĩa nữa.

### Everything builds!

~~~dart
import 'package:flutter/material.dart';
import 'dart:math' as math;

void main() {
  runApp(ShrinkWrApp());
}

class ShrinkWrApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      title: 'ShrinkWrap vs Slivers',
      home: Scaffold(
        appBar: AppBar(
          title: const Text("ShrinkWrap, Street Rat, I don't, Buy that!"),
        ),
        body: const ShrinkWrapSlivers(),
      ),
    );
  }
}

class ShrinkWrapSlivers extends StatefulWidget {
  const ShrinkWrapSlivers({
    Key? key,
  }) : super(key: key);

  @override
  _ShrinkWrapSliversState createState() => _ShrinkWrapSliversState();
}

class _ShrinkWrapSliversState extends State<ShrinkWrapSlivers> {
  List<ListView> innerLists = [];
  final numLists = 15;
  final numberOfItemsPerList = 100;

  @override
  void initState() {
    super.initState();
    for (int i = 0; i < numLists; i++) {
      final _innerList = <ColorRow>[];
      for (int j = 0; j < numberOfItemsPerList; j++) {
        _innerList.add(const ColorRow());
      }
      innerLists.add(
        ListView.builder(
          itemCount: numberOfItemsPerList,
          itemBuilder: (BuildContext context, int index) => _innerList[index],
          shrinkWrap: true,
          physics: const NeverScrollableScrollPhysics(),
        ),
      );
    }
  }

  @override
  Widget build(BuildContext context) {
    return ListView.builder(
        itemCount: numLists,
        itemBuilder: (context, index) => innerLists[index]);
  }
}

@immutable
class ColorRow extends StatefulWidget {
  const ColorRow({Key? key}) : super(key: key);

  @override
  State createState() => ColorRowState();
}

class ColorRowState extends State<ColorRow> {
  Color? color;

  @override
  void initState() {
    super.initState();
    color = randomColor();
  }

  @override
  Widget build(BuildContext context) {
    print('Building ColorRowState');
    return Container(
      decoration: BoxDecoration(
        gradient: LinearGradient(
          begin: Alignment.topLeft,
          end: Alignment.bottomRight,
          colors: [
            randomColor(),
            randomColor(),
          ],
        ),
      ),
      child: Row(
        children: <Widget>[
          Padding(
            padding: const EdgeInsets.all(8.0),
            child: Container(height: 50, width: 50, color: Colors.white),
          ),
          Flexible(
            child: Column(
              children: const <Widget>[
                Padding(
                  padding: EdgeInsets.all(8),
                  child: Text('I\'m a widget!',
                      style: TextStyle(color: Colors.white)),
                ),
              ],
            ),
          ),
        ],
      ),
    );
  }
}

Color randomColor() =>
    Color((math.Random().nextDouble() * 0xFFFFFF).toInt()).withOpacity(1.0);
~~~

Nếu chạy thử, bạn sẽ thấy 1 vấn đề xảy ra, đó là ngay khi UI được load, 100 item đầu được render ngay mặc dù ta mới chỉ nhìn thấy vài item đầu... Và khi bạn scroll thêm một đoạn đủ 100 item, thì 100 item tiếp theo sẽ được load ngay sau đó tại 1 thời điểm. Đây như là một vụ bom nổ vậy :D

Giờ phải làm như nào ???

### Lazily building nested lists

Để cứu bạn, hãy cùng xay dựng lại giao diện với `Slivers` thay vì ListViews.

Để tìm hiểu thêm về sliver, hãy đọc ở [đây](https://docs.flutter.dev/development/ui/advanced/slivers)

#### Bước 1

Thay ListView wrap ngoài cùng bằng `SliverList`

~~~dart
// Before
@override
Widget build(BuildContext context) {
  return ListView.builder(
    itemCount: numberOfLists,
    itemBuilder: (context, index) => innerLists[index],
  );
}

// After
@override
Widget build(BuildContext context) {
  return CustomScrollView(slivers: innerLists);
}

~~~

#### Bước 2

Thay đổi data type của listview con từ List<ListView> => List<SliverList>

~~~dart
// Before
List<ListView> innerLists = [];

// After
List<SliverList> innerLists = [];

~~~

#### Bước 3

Giờ thì restructure đống listview con nào. SliverList thì hơi khác so với ListView, chủ yếu từ sự xuất hiện của 1 thằng là delegate.

~~~

// Before
@override
void initState() {
  super.initState();
  for (int i = 0; i < numberOfLists; i++) {
    final _innerList = <ColorRow>[];
    for (int j = 0; j < numberOfItemsPerList; j++) {
      _innerList.add(const ColorRow());
    }
    innerLists.add(
      ListView.builder(
        itemCount: numberOfItemsPerList,
        itemBuilder: (BuildContext context, int index) => _innerList[index],
        shrinkWrap: true,
        physics: const NeverScrollableScrollPhysics(),
      ),
    );
  }
}

// After
@override
void initState() {
  super.initState();
  for (int i = 0; i < numLists; i++) {
    final _innerList = <ColorRow>[];
    for (int j = 0; j < numberOfItemsPerList; j++) {
      _innerList.add(const ColorRow());
    }
    innerLists.add(
      SliverList(
        delegate: SliverChildBuilderDelegate(
          (BuildContext context, int index) => _innerList[index],
          childCount: numberOfItemsPerList,
        ),
      ),
    );
  }
}
~~~

Và đây là thành quả

~~~dart
import 'package:flutter/material.dart';
import 'dart:math' as math;

void main() {
  runApp(SliversApp());
}

class SliversApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      title: 'ShrinkWrap vs Slivers',
      home: Scaffold(
        appBar: AppBar(
          title: const Text("Revenge of the Slivers"),
        ),
        body: const ShrinkWrapSlivers(),
      ),
    );
  }
}

class ShrinkWrapSlivers extends StatefulWidget {
  const ShrinkWrapSlivers({
    Key? key,
  }) : super(key: key);

  @override
  _ShrinkWrapSliversState createState() => _ShrinkWrapSliversState();
}

class _ShrinkWrapSliversState extends State<ShrinkWrapSlivers> {
  List<SliverList> innerLists = [];
  final numLists = 15;
  final numberOfItemsPerList = 100;

  @override
  void initState() {
    super.initState();
    for (int i = 0; i < numLists; i++) {
      final _innerList = <ColorRow>[];
      for (int j = 0; j < numberOfItemsPerList; j++) {
        _innerList.add(const ColorRow());
      }
      innerLists.add(
        SliverList(
          delegate: SliverChildBuilderDelegate(
            (BuildContext context, int index) => _innerList[index],
            childCount: numberOfItemsPerList,
          ),
        ),
      );
    }
  }

  @override
  Widget build(BuildContext context) {
    return CustomScrollView(slivers: innerLists);
  }
}

@immutable
class ColorRow extends StatefulWidget {
  const ColorRow({Key? key}) : super(key: key);

  @override
  State createState() => ColorRowState();
}

class ColorRowState extends State<ColorRow> {
  Color? color;

  @override
  void initState() {
    super.initState();
    color = randomColor();
  }

  @override
  Widget build(BuildContext context) {
    print('Building ColorRowState');
    return Container(
      decoration: BoxDecoration(
        gradient: LinearGradient(
          begin: Alignment.topLeft,
          end: Alignment.bottomRight,
          colors: [
            randomColor(),
            randomColor(),
          ],
        ),
      ),
      child: Row(
        children: <Widget>[
          Padding(
            padding: const EdgeInsets.all(8.0),
            child: Container(height: 50, width: 50, color: Colors.white),
          ),
          Flexible(
            child: Column(
              children: const <Widget>[
                Padding(
                  padding: EdgeInsets.all(8),
                  child: Text('I\'m a widget!',
                      style: TextStyle(color: Colors.white)),
                ),
              ],
            ),
          ),
        ],
      ),
    );
  }
}

Color randomColor() =>
    Color((math.Random().nextDouble() * 0xFFFFFF).toInt()).withOpacity(1.0);
~~~

Hãy chạy thử đoạn code trên, bạn sẽ thấy rằng UI sẽ không còn render 100 item 1 lúc mà chỉ render 1 phần rất nhỏ. Còn gì tuyệt hơn khi scroll đến cuối danh sách mà không mất bất kì chi phí tốn kém nào.

## Conclusion

Cảm ơn các bạn đã đọc bài viết. Happy coding ^^

### References

* https://www.youtube.com/watch?v=LUqDNnv_dh0