---
title: "[Android] DataBinding Basic"
date: 2019-03-21 21:15:03 +0700
categories: [Android]
tags: [android, data binding]
---

## 1. Introduction

Databinding là thư viện cho phép bạn liên kết các thành phần UI trong layout tới các nguồn dữ liệu trong ứng dụng bằng cách khai báo thay vì phải coding.

Databinding có chút khác biệt so với các layout bình thường. Nó bắt đầu bằng root tag **<layout>**, theo sau đó là **<data>** cuối cùng là root **<view>** 
    
Dưới dây là một ví dụ :
~~~xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android">
   <data>
       <variable name="user" type="com.example.User"/>
   </data>
   <LinearLayout
       android:orientation="vertical"
       android:layout_width="match_parent"
       android:layout_height="match_parent">
       <TextView android:layout_width="wrap_content"
           android:layout_height="wrap_content"
           android:text="@{user.firstName}"/>
       <TextView android:layout_width="wrap_content"
           android:layout_height="wrap_content"
           android:text="@{user.lastName}"/>
   </LinearLayout>
</layout>
~~~

Thông thường để khai báo 1 view thông thường sau đó setText cho nó ta thường
làm như sau :
~~~java
findViewById<TextView>(R.id.sample_text).apply {
    text = viewModel.userName
}
~~~
Đoạn mã bên dưới đây sẽ cho bạn thấy cách mà DataBinding gắn text vào UI trực tiếp. Điều này giúp loại bỏ việc phải gọi bất cứ dòng
code nào như ở bên trên :

~~~java
<TextView
    android:Text = "@{viewmodel.userName}" />
~~~

Vậy mục đích của nó là gì:
- Giúp giảm thiểu lời gọi trong activities, fragments..
- Đơn giản, dễ maintain hơn
- Tăng hiệu năng ứng dụng và tránh memory leak hay NPE.

## 2. Data Object
Giả sử bạn có một đối tượng User như sau :
~~~java
data class User(val firstName: String, val lastName: String)
~~~
Khi đó việc biểu diễn @{user.firstName} sử dụng với thuộc tính android:text sẽ truy xuất vào trường firstName của đối tượng.

## 3. Binding Data
Một lớp ràng buộc được tự động gen ra cho mỗi file layout. Mặc định, tên của class được dựa trên tên của layout đó bằng cách chuyển đổi nó sang Pascal case và thêm vào hậu tố Binding.

Ví dụ :
activity_main.xml => ActivityMainBinding

Lớp này giữ tất cả các ràng buộc từ các thuộc tính layout (VD: biến user) và cách gắn giá trị cho các biểu thức ràng buộc. 
~~~java
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    val binding: ActivityMainBinding = DataBindingUtil.setContentView(this, R.layout.activity_main)
    binding.user = User("Test", "User")
}
~~~
Tại runtime, ứng dụng sẽ hiển thị user Test trên màn hình. Ngoài ra, bạn cũng có thể lấy view sử dụng **LayoutInflater**, được biểu diễn như sau:
~~~java
val binding: ActivityMainBinding = ActivityMainBinding.inflate(getLayoutInflater())
~~~
Nếu bạn sử dụng data binding bên trong Fragment,Listview hay RecyclerView adapter, bạn có thể sử dụng phương thức **inflate()** của lớp binding hoặc lớp **DataBindingUtil**. 
~~~java
val listItemBinding = ListItemBinding.inflate(layoutInflater, viewGroup, false)
// or
val listItemBinding = DataBindingUtil.inflate(layoutInflater, R.layout.list_item, viewGroup, false)
~~~

## 4. Expression language

Để diễn đạt trong databinding có thể sử dụng 1 số cách diễn đạt sau. Nó cũng tương tự cách biểu diễn trong mã code :

- Mathematical + - / * %
- String concatenation +
- Logical && ||
- Binary & | ^
- Unary + - ! ~
- Shift >> >>> <<
- Comparison == > < >= <=
- instanceof
- Grouping ()
- Literals - character, String, numeric, null
- Cast
- Method calls
- Field access
- Array access []
- Ternary operator ?:

Ví dụ :
~~~java
android:text="@{String.valueOf(index+1)}"
android:visibility="@{age < 13 ? View.GONE : View.VISIBLE"
android:transitionName='@{"image_" + id}'
~~~

### 4.1. Missing Operators

Một số operation dưới bị thiếu từ cú pháp biểu diễn mà bạn có thể sử dụng trong các đoạn mã được quản lý :
- this
- super
- new
- Explicit generic invocation

### 4.2. Null coalescing operator (??)

Toán tử kết hợp null (??) này sẽ chọn toán hạng bên trái nếu nó không null còn không thì sẽ chọn bên phải
~~~java
android:text="@{user.displayName ?? user.lastName}"
~~~
Câu lệnh trên tương đương với :
~~~java
android:text="@{user.displayName != null ? user.displayName : user.lastName}"
~~~

### 4.3. Avoid null pointer exceptions

Mã code được gen với Databinding sẽ tự động check các giá trị null và tránh các Null Pointer Exceptions.

Ví dụ: Bạn tham chiếu đến @{user.name} mà user đang null thì user.name sẽ được gán giá trị mặc định là null.

### 4.4. Collections

Một số collection phổ biến có thể truy cập bởi toán tử [ ]  :
- arrays
- lists
- sparse lists
- map

~~~java
<data>
    <import type="android.util.SparseArray"/>
    <import type="java.util.Map"/>
    <import type="java.util.List"/>
    <variable name="list" type="List<String>"/>
    <variable name="sparse" type="SparseArray<String>"/>
    <variable name="map" type="Map<String, String>"/>
    <variable name="index" type="int"/>
    <variable name="key" type="String"/>
</data>
…
android:text="@{list[index]}"
…
android:text="@{sparse[index]}"
…
android:text="@{map[key]}"
~~~

### 4.5. String Literals

Có thể sử dụng nháy đơn hoặc nháy kép để bao quanh giá trị thuộc tính :
~~~java
android:text='@{map["firstName"]}'
~~~
hoặc
~~~java
android:text="@{map[`firstName`]}"
~~~

### 4.6. Resources

Có thể sử dụng các giá trị của resource bằng cách khai báo như sau:
~~~java
android:padding="@{large? @dimen/largePadding : @dimen/smallPadding}"
~~~
Để format strings và plurals cần sử dụng theo format :
~~~java
android:text="@{@string/nameFormat(firstName, lastName)}"
android:text="@{@plurals/banana(bananaCount)}"
~~~

Một số resource yêu cầu rõ ràng loại :

<!-- Type |	Normal reference	| Expression reference -->
<!-- String[]|	@array|	@stringArray -->
<!-- int[]	@array	@intArray -->
<!-- TypedArray	@array	@typedArray -->
<!-- Animator	@animator	@animator -->
<!-- StateListAnimator	@animator	@stateListAnimator -->
<!-- color int	@color	@color -->
<!-- ColorStateList	@color	@colorStateList -->

## 5. Event Handling

Databinding cho phép bạn diễn đạt xử lý sự kiện. Tên thuộc tính sự kiện được xác định bằng tên của phương thức listener với 1 vài ngoại lệ. 

Ví dụ View.OnClickListener có method onClick(), nên thuộc tính cho sự kiện này là android:onClick. Một số thuộc tính khác như :
~~~java
android:onClick
android:onSearchClick
android:onZoomIn
android:onZoomOut
...
~~~
Trong Databinding có 2 cơ chế xử lý sự kiện :

- Method references :
- Listener bindings :

#### Method References

Sự khác biệt giữa method references và listener binding là thực tế thì
listener implementation được tạo khi data được gắn vào, chứ ko phải khi
sự kiện được kích hoạt.
=> Nếu bạn muốn evaluate the expresstion khi sự kiện xảy ra. Bạn nên sử
dụng listener binding.

Gỉa sử, để gán một sự kiện cho handler của nó, sử dụng cách binding bình
thưởng. ví dụ :
~~~java
public class MyHandlers {
    public void onClickFriend(View view) {
        //TODO
    }
}
~~~
Binding expression có thể gán click listener cho view thông qua method
onClickFriend() như sau :
~~~xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android">
   <data>
       <variable name="handlers" type="com.example.MyHandlers"/>
       <variable name="user" type="com.example.User"/>
   </data>
   <LinearLayout
       android:orientation="vertical"
       android:layout_width="match_parent"
       android:layout_height="match_parent">
       <TextView android:layout_width="wrap_content"
           android:layout_height="wrap_content"
           android:text="@{user.firstName}"
           android:onClick="@{handlers::onClickFriend}"/>
   </LinearLayout>
</layout>
~~~

### Listener binding

Listener binding là kiểu binding được chạy khi có sự kiện xảy ra. Nó
tương tự method refrenence nhưng nó cho phép bạn chạy các biểu thưc ràng
buộc dữ liệu tùy ý.

~~~java
public class Presenter {
    public void onSaveClick(Task task){}
}
~~~
Khi đó bạn có thể bind sự kiện tới onSaveClick(), như sau :
~~~xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android">
    <data>
        <variable name="task" type="com.android.example.Task" />
        <variable name="presenter" type="com.android.example.Presenter" />
    </data>
    <LinearLayout android:layout_width="match_parent" android:layout_height="match_parent">
        <Button android:layout_width="wrap_content" android:layout_height="wrap_content"
        android:onClick="@{() -> presenter.onSaveClick(task)}" />
    </LinearLayout>
</layout>
~~~

### Import, variables, include

Databinding cung cấp 1 số feature như import, variable, includes.
Import giúp việc tham chiếu đến class bên trong layout của bạn đơn giản hơn.
Variable cho phép bạn miêu tả đối tượng


### Type Aliases

Khi có xung đột về tên lớp, một trong các lớp có thể được đổi thành bí danh. (alias)
Ví dụ sau đây sẽ rename View class trong package com.example.real.estate
thành Vista :
~~~java
<import type="android.view.View"/>
<import type="com.example.real.estate.View"
        alias="Vista"/>
~~~

### Imported other class

~~~xml
<data>
    <import type="com.example.User"/>
    <import type="java.util.List"/>
    <variable name="user" type="User"/>
    <variable name="userList" type="List<User>"/>
</data>
~~~
Imported types có thể sử dụng khi tham chiếu các trường hay method static.
Đoạn code sau imports lớp MyStringUtils và tham chiếu đến lớp capitalize của nó :
~~~xml
<data>
    <import type="com.example.MyStringUtils"/>
    <variable name="user" type="com.example.User"/>
</data>
…
<TextView
   android:text="@{MyStringUtils.capitalize(user.lastName)}"
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"/>
~~~
    
## 6. Conclusion
Trên đây là phần mở đầu về data binding, hi vọng bài viết có thể giúp các bạn có cái nhìn tổng quan về databinding và cách dùng cơ bản. Trong bài viết sau mình sẽ cùng tìm hiểu về cách sử dụng data binding với LiveData, binding adapter... 
    
Cảm ơn các bạn đã đọc bài viết. Happy coding ^^