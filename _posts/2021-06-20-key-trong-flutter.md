---
title: "[Flutter] Key trong Flutter"
date: 2021-06-06 21:15:03 +0700
categories: [Flutter]
tags: [dart, flutter, key, localkey, uniqueKey]
---

## Giới thiệu

`Key` được nhìn thấy ở hầu hết các phương thức khởi tạo widget. Nhưng việc sử dụng chúng không được phổ biến lắm. Trong bài viết này, mình sẽ cùng tìm hiểu xem key là gì, có những loại nào, cách dùng của chúng ra sao và khi nào thì sử dụng Key nhé.

## Key là gì ?

Key là một đối tượng định danh, nó dùng để duy trì trạng thái của Widget khi các Widget di chuyển vị trí trong Widget Tree.

## Có những loại Key nào ?

`Key` có 2 subclass là `LocalKey` và `GlobalKey`. 

Trong đó `LocalKey` có 3 loại là `ValueKey`, `ObjectKey`, `UniqueKey`

* **ValueKey**

    Giả sử bạn đang xây dựng ứng dụng ToDo App, mỗi ToDo item đều có 1 ID duy nhất để phân biệt giữa các item. Trong trường hợp này, bạn nên sử dụng ValueKey. Nó sẽ giúp cho việc thêm, xoá, sửa 1 ToDo bất kỳ nhanh hơn, đặc biệt là với xoá.

* **ObjectKey**
  
    Giả sử rằng mỗi widget con lưu một tập dữ liệu phức tạp, chẳng hạn như một ứng dụng lưu trữ sổ địa chỉ cho thông tin người dùng. Bất kỳ trường thông tin nào, chẳng hạn như tên hoặc ngày sinh, có thể giống với một người khác, nhưng mỗi một tập dữ liệu là duy nhất. Trong những trường hợp này `ObjectKey` là phù hợp nhất.

* **UniqueKey**

   Nếu có nhiều widget có cùng loại hoặc nếu bạn cần đảm bảo rằng widget con không giống với các widget khác, bạn có thể sử dụng `UniqueKey`. Chúng ta đã sử dụng nó trong ví dụ trên. `UniqueKey`. Một điều cần chú ý rằng hãy cân nhắc việc sử dụng UniqueKey khi có những action như di chuyển, sửa đổi item chứa nhiều logic hay animation nặng bởi mỗi khi có thay đổi, state sẽ bị reset đồng nghĩa với việc widget sẽ bị rebuild lại.

* **GlobalKey**

    Chúng cho phép các Widget thay đổi parent của chúng ở bất kỳ vị trí nào trong ứng dụng mà không bị mất state hoặc chúng có thể được sử dụng để truy cập dữ liệu để lấy thông tin của một Widget bất kỳ.

    Trong trường hợp thứ hai, bạn có thể cần kiểm tra mật khẩu, tuy nhiên bạn không muốn chia sẻ status data với các widget khác nhau trong cây và bạn có thể sử dụng GlobalKey<FromState>, nó nắm giữ State của Form.

    Về cơ bản, GlobalKey giống như một biến toàn cục vậy. Có những cách khác tốt hơn để hoàn thành công việc của GlobalKey, chẳng hạn như inherited widget, Redux, or block pattern.


## Khi nào nên dùng Key ?

Lý thuyết xuông thì khó hiểu, giờ mình cùng đi vào 1 vài ví dụ nhé. Trong ví dụ dưới đây, mình sẽ cùng hoán đổi vị trí 2 Widget có 2 màu sắc khác nhau. Màu sắc của mỗi widget là Random nha.

### Swap 2 StatelessWidget ???

~~~dart
class Tile extends StatelessWidget {
  final Color color = generateRandomColor(); // mỗi object sẽ có 1 random Color không thể thay đổi

  @override
  Widget build(BuildContext context) {
    // tạm hiểu Container như cái thùng có màu sắc, kích thước => khá giống viên gạch =))
    return Container(
      color: color,
      width: 100,
      height: 100,
    );
  }
}

// hàm tạo ra một Color bất kỳ
Color generateRandomColor() {
  // biến random sẽ giúp ta tạo ra 1 số ngẫu nhiên
  final Random random = Random();
  
  // Màu sắc được tạo nên từ RGB, là một số ngẫu nhiên từ 0 -> 255 và opacity = 1
  return Color.fromRGBO(random.nextInt(255), random.nextInt(255), random.nextInt(255), 1);
}
~~~

~~~dart
class _MyHomePageState extends State<MyHomePage> {
  var listTile = <Widget>[Tile(), Tile()]; // tạo 1 list lưu trữ 2 Widget

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(
        child: Row(
          children: listTile, // Row nhận một list Widget nên mình truyền listTile vào
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: swapTwoTileWidget, // thực hiện hoán đổi vị trí 2 Widget trong listTile
        child: Icon(Icons.swap_horiz),
      ),
    );
  }

  // hàm thực hiện hoán đổi 2 Widget trong listTile
  void swapTwoTileWidget() {
    setState(() {
      listTile.insert(1, listTile.removeAt(0));
    });
  }
}
~~~

Ok, sau khi click swap, thì vị trí của 2 Widget được thay đổi đúng như những gì ta mong muốn.

1 lưu ý được các nhà phát triển Flutter có đề cập đến Key:

~~~
Remember, if the entire widget subtree in your collection is stateless, keys aren't needed.
~~~
> Có thể hiểu rằng Key sẽ không cần thiết nếu như các widget subtree là các StatelessWidget

#### Swap 2 StatefulWidget ???

Ok, hãy quay lại class `Tile` và convert nó thành StatefulWidget thay vì StatelessWidget.

~~~dart
class Tile extends StatefulWidget {
  @override
  State<Tile> createState() => _TileState();
}

class _TileState extends State<Tile> {
  final Color color = generateRandomColor(); 
  @override
  Widget build(BuildContext context) {
    // tạm hiểu Container như cái thùng có màu sắc, kích thước => khá giống viên gạch =))
    return Container(
      color: color,
      width: 100,
      height: 100,
    );
  }
}

// hàm tạo ra một Color bất kỳ
Color generateRandomColor() {
  // biến random sẽ giúp ta tạo ra 1 số ngẫu nhiên
  final Random random = Random();
  
  // Màu sắc được tạo nên từ RGB, là một số ngẫu nhiên từ 0 -> 255 và opacity = 1
  return Color.fromRGBO(random.nextInt(255), random.nextInt(255), random.nextInt(255), 1);
}
~~~

Giờ hãy click button swap xem...click mãi mà không thấy có gì xảy ra phải không :D nào thì giờ fixbug thôi.

Trong constructor của Tile, hãy truyền thêm tham số Key vào như sau:

~~~dart
Tile({Key key}) : super(key: key);
~~~

Giờ hãy tạo ra 1 UniqueKey rồi gắn vào nhé.

~~~dart
var listTile = <Widget>[Tile(key: UniqueKey()), Tile(key: UniqueKey())]; // UniqueKey() tạo ra 1 key
~~~

Ok, giờ thì swap lại hoạt động trở lại rồi đó :D

> Nếu Widget được khai báo Key thì Element không chỉ so sánh Widget Type mà còn so sánh cả Key của widget cũ mà nó đang giữ tham chiếu và widget mới

Hãy cùng làm một ví dụ tiếp nối ví dụ trên nhé. Vẫn sử dụng Tile là statefulWidget, giờ hãy wrap Tile vào 1 thằng `Padding` xem sao

~~~dart
var listTile = <Widget>[
    Padding(
      padding: const EdgeInsets.all(8.0),
      child: Tile(key: UniqueKey()),
    ),
    Padding(
      padding: const EdgeInsets.all(8.0),
      child: Tile(key: UniqueKey()),
    )
];
~~~

=> Giờ rebuild lại app và thử click button Swap, một điều kì diệu xảy ra là mỗi lần click thì màu của 2 ô vuông lại thay đổi, ảo ma canada =))

Màu của widget thay đổi liên tục đồng nghĩa với việc State của Widget đó bị khởi tạo lại, `createState()` được gọi và 1 màu mới được sinh ra.

Lý do bởi khi Local Key, thì quá trình matching up widget to elements chỉ diễn ra trong cùng một level cụ thể trên hệ thống tree, ở đây là 2 Padding view. Element bị deactive mà không tìm được widget để connect thì nó sẽ bị dispose.

=> LocalKey chỉ nên được đặt ở các Widget là top level, level cao nhất trong list children của một Row hoặc một Column. Trong 1 row, các widget con đều có cùng một level, Flutter sẽ tìm được.

Giờ cách fix đơn giản hơn rồi phải không, move key ở thằng Tile chuyển lên thằng Padding:

~~~dart
var listTile = <Widget>[
    Padding(
      key: UniqueKey(), // đưa key từ Tile lên Padding
      padding: const EdgeInsets.all(8.0),
      child: Tile(),
    ),
    Padding(
      key: UniqueKey(), // đưa key từ Tile lên Padding
      padding: const EdgeInsets.all(8.0),
      child: Tile(),
    )
  ];
~~~

### Tổng kết

Hi vọng qua bài viết này bạn có thể hiểu được cơ bản và cách dùng của Key. Happy coding :D

* [https://medium.com/flutter/keys-what-are-they-good-for-13cb51742e7d](https://medium.com/flutter/keys-what-are-they-good-for-13cb51742e7d)

* [https://app.slack.com/client/TBFDUP13L/C01CUJW4H18](https://app.slack.com/client/TBFDUP13L/C01CUJW4H18)








