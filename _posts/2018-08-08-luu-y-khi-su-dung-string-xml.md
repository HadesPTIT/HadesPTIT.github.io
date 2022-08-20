---
title: "[Android] Một số lưu ý khi sử dụng strings.xml"
date: 2018-08-08 21:15:03 +0700
categories: [Android]
tags: [android]
---

Hi mọi người!<br/>
Trong bài viết này mình sẽ chia sẻ một số lưu ý khi làm việc với **strings.xml**.

## Đừng dùng lại
> Đừng sử dụng lại strings cho nhiều màn hình khác nhau.

**1**.	Tưởng tượng rằng bạn có một hộp thoại loading trên 2 màn hình Sign in và Sign up.<br/>Vì cả 2 đều có chung một hộp thoại nên bạn quyết định sử dụng chung string là **R.string.loading**

~~~xml
<string name="loading">Loading please wait...</string>
~~~

**2**.	 Sau đó nếu bạn quyết định sử dụng một cái khác, bạn sẽ phải tạo ra hai string mới và cần update lại file. Nếu từ đầu bạn sử dụng 2 string, bạn sẽ sửa duy nhất file strings.xml.

~~~xml
<string name="sign_in_loading">Signing in...</string>
<string name="sign_up_loading">Signing up...</string>
~~~

**3**.	Bạn không bao giờ biết ứng dụng của bạn hỗ trợ ngôn ngữ nào. Trong một ngôn ngữ - bạn sẽ có thể sử dụng một từ cho nhiều nội dung khác nhau, nhưng ở một ngôn ngữ khác - bạn sẽ phải sử dụng các từ khác nhau cho nội dung khác nhau.

###### *res/values/strings.xml*
~~~xml
<string name="download_file_yes">Yes</string>
<string name="terms_of_use_yes">Yes</string>
~~~

###### *res/values-UA/strings.xml*
~~~xml
<string name="download_file_yes">Гаразд</string>
<string name="terms_of_use_yes">Так</string>
~~~


=> Phiên bản tiếng Anh của `strings.xml` sử dụng cùng một từ `yes` cho cả hai string. Nhưng phiên bản tiếng Ukraina của strings.xml sử dụng hai từ khác nhau: “Гаразд” cho R.string.download_file_yes và “Так” cho R.string.terms_of_use_yes.

## Riêng biệt
Các strings riêng biệt thuộc chung một màn hình nên được phân biệt bởi tiền tố và comment

~~~xml
<!-- settings screen start -->
<string name="settings_log_out">Log out</string>
<string name="settings_notification_policy">Notification Policy</string>
<string name="settings_contact_us">Contact Us</string>
<!-- settings screen end -->

<!-- profile screen start -->
<string name="profile_name">Name</string>
<string name="profile_email">Email</string>
<string name="profile_work">Work</string>
<string name="profile_birthday">Birthday</string>
<!-- profile screen end -->
~~~

* Thêm tên màn hình như là tiền tố vào mỗi string giúp ta ngay lập tức nhận ra màn hình mà string hiện tại thuộc về.
* Tệp strings.xml rõ ràng giúp dễ dàng duy trì và chuyển đổi nhiều strings sang các ngôn ngữ khác nhau.


## Định dạng

Sử dụng `Resource#getString(int id, Object... formatArgs)` để định dạng strings.

Không bao giờ nối chuỗi bằng toán tử **+** bởi vì trong các ngôn ngữ khác thứ tự từ có thể thay đổi.
~~~xml
<string name="login_welcome_back"> - welcome back</string>
~~~
###### *res/values/strings.xml*
~~~java
String username =  "John Doe";
String result = username + " " + getString(R.string.login_welcome_back);
// EN result "John Doe - welcome back"
~~~
Cách đúng nên sử dụng **Resources#getString(int id, Object… formatArgs)**.
~~~xml
<string name="login_welcome_back"> %s - welcome back</string>
~~~
###### *res/values/strings.xml*
~~~xml
<string name="login_welcome_back"> з поверненням %s</string>
~~~
###### *res/values-UA/strings.xml*
Khi đó :
~~~java
String username = "John Doe";
String result = getString(R.string.login_welcome_back,username);
// EN result "John Doe - welcome back"
// UA result "з поверненням John Doe"
~~~

## Số nhiều 
Sử dụng `Resources#getQuantityString (int id, int quantity)` cho strings số lượng.
Không bao giờ giải quyết số nhiều trong code java , bởi vì các ngôn ngữ khác nhau có quy tắc khác nhau cho qui định về ngữ pháp với số lượng.
~~~xml
<plurals name="minutes">
		<item quantity="one">minute</item>
		<item quantity="other">minutes</item>
</plurals>
~~~
###### *res/values/strings.xml*
~~~java
int minute = Calendar.getInstance().get(Calendar.MINUTE);
String text = getResource().getQuantityString(R.plurals.minutes,minutes);
~~~

## Làm nổi bật một số từ
Sử dụng văn bản html để làm nổi bật những chữ cố định.

Nếu bạn muốn thay đổi màu sắc của một số từ trong TextView - **ForegroundColorSpan** không phải lúc nào cũng là sự lựa chọn tốt, bởi vì nếu nối bật thông qua các indexes, nó không an toàn trong các ứng dụng đa ngôn ngữ. Tốt hơn nên sử dụng *Html font color tags* bên trong file strings.xml.

Bạn có văn bản “Discover and play games.” Bạn muốn làm nổi bật từ “Discover” và “play” với màu xanh lam.
~~~xml
<string name="html_text" formatted="false">
	<![CDATA[
    <font color=\'#28b5f5\'>Discover</font> and <font color=\'#28b5f5\'>play</font> game.]]>
</string>
~~~
###### *res/values/strings.xml*
~~~java
TextView textView = (TextView) findViewById(R.id.tv);
textView.setText(Html.fromHtml(getString(R.string.html_text)));
~~~

## Kết bài

Chúc các bạn coding vui vẻ :D

