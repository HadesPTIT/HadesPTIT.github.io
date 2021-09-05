---
title: "[Android] Performance  -  Rendering"
date: 2018-12-23 21:15:03 +0700
categories: [Android]
tags: [android, performance]
---

Hiệu năng ứng dụng luôn là vấn đề với mọi lập trình viên mobile. Bởi lẽ, nếu bạn nhận ra ứng dụng của mình chậm hoặc tệ hơn là ai đó nói với bạn rằng ứng dụng của bạn chậm quá thì thật đáng buồn phải không ? :D

Rendering là một trong những vấn đề hiệu suất phổ biến nhất. Trong bài viết này, mình sẽ cùng các bạn tìm hiểu về rendering trong android và một số vấn đề liên quan nhé.

## 1. Giới thiệu
Trong Android, hệ thống sẽ cố vẽ lại các hoạt động của bạn sau mỗi 16ms. Điều đó có nghĩa rằng ứng dụng của bạn cần xử lý các logic code để update màn hình trong khoảng 16ms, điều này giúp ứng dụng của bạn trở nên "mượt".

![](https://images.viblo.asia/32c8ef75-bbf0-4f4b-8867-21594bd883c3.png )

Nhưng điều gì xảy ra nếu bạn không thể hoàn thành logic code trong 16ms đó ?

Hiện tượng này gọi là "Dropped Frame" hay tụt khung hình. Giả sử logic của bạn chiếm 24ms vẽ trong khi hệ thống cố gắng làm mới khung hình mỗi 16ms. Bởi vậy phải sau 32ms thì khung hình mới được cập nhật.

![](https://images.viblo.asia/594eed06-e47d-43b4-9e56-c0ac58c292d5.png)

Nếu việc drop khung hình tới 5s thì bạn sẽ thấy thông báo lỗi ARN (Application Not Responding)

![](https://images.viblo.asia/2483c582-f1f8-49eb-88b2-204a9daadce0.png)

Để tìm hiểu kĩ hơn, hãy xem việc render được thực hiện như nào nhé!

## 2. Rendering
Về cơ bản, việc rendering được chia thành 2 phần : 
* CPU rendering
* GPU rendering

Về phía CPU, vấn đề hiệu suất thường đến từ việc sử dụng layout không cần thiết và invalidate() không đúng chỗ.

Về phía GPU, đó là Overdraw.

![](https://images.viblo.asia/5ecb3451-f597-42a9-b8cd-de4f69588b21.png)

Để những file .XML mà bạn design được chuyển thành những pixels mà người dùng có thể thấy và hiển thị, nó phải trải qua 1 quá trình được gọi là Rasterization. Đây là quá trình chuyển đổi các đối tượng cấp cao như String, Button, Path, Shape... thành những pixel trên màn hình.

Để rõ hơn hãy nhìn flow bên dưới:

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/b0bkqmixtr_1_2dt_c87fcQMFLTZNscr1pA.png)

1. Bất cứ khi nào các đối tượng cần được vẽ lên màn hình, đầu tiên nó được CPU xử lý thành các dạng nguyên thủy như polygons và texture và gửi tới GPU thông qua tập các OpenGLES API.
2. Các dữ liệu được chuyển tới GPU để Rasterization và được vẽ lên màn hình.

Một ví dụ về quá trình render 1 button lên màn hình :

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/675bksrpgo_1_np19-xORJ56n4F04zKLqPQ.png)

## 3. Overdraw

Overdraw là một thuật ngữ được sử dụng để mô tả một điểm ảnh trên màn hình được vẽ lại bao nhiêu lần trong 1 khung hình duy nhất.

Giả sử bạn có một giao diện hiển thị một stack các thẻ như bên dưới. Chỉ có thẻ trên cùng là hiển thị toàn bộ còn các thẻ bên dưới chỉ hiển thị 1 phần, nếu bạn vẽ tất cả các thẻ thì sẽ thật lãng phí khi GPU phải vẽ những thứ không hiển thị với người dùng.

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/8mcey93zrl_1_jUqD2SZ7HPIbG7eKG9gfwg.png)

Để xác định overdraw trên các device Android : **Setting -> Developer Options -> Show GPU Overdraw**

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/8o4af83w8v_1__lcAJ8_wOPNLzFNuOSevOA.png)

Android sử dụng những bản màu khác nhau để làm nổi các mức overdraw trên màn hình. 

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/u4wn1wg4sz_1_C42FN45vW3MQ2KNNEFuwWg.png)

* Đúng màu : không có overdraw
* Xanh biển : overdraw 1 lần
* Xanh lá : overdraw 2 lần
* Hồng : overdraw 3 lần
* Đỏ : overdraw từ 4 lần trở lên


![](https://images.viblo.asia/6fd91805-3dad-41bc-a2b7-9ab369f09cd5.png)

Trên thực tế thì xanh biển là chấp nhận được. Overdraw càng nhiều, hiệu năng càng giảm. Để tránh điều này, có thể dùng những giải pháp sau:

### 3.1. Loại bỏ background không cần thiết

Đơn giản nhất, loại bỏ những background mà người dùng không bao giờ thấy được.

~~~xml
<ImageView
    android:layout_width=”match_parent”
    android:layout_height=”match_parent”
    android:src=”@drawable/beach”
    android:background=”@android:color/white”>
</ImageView>

<ImageView
    android:layout_width=”match_parent”
    android:layout_height=”match_parent”
    android:src=”@drawable/beach” >
</ImageView>
~~~

### 3.2. Hạn chế sử dụng Transparency

Việc render những pixel trong suốt trên màn hình được gọi là Alpha rendering. Quá trình này  cũng gây ra overdraw, bởi hệ thống sẽ phải hiển thị cả pixel trong suốt và điểm ảnh bên dưới pixel, sau đó kết hợp cả 2 chúng lại.
Các hiệu ứng hình ảnh như trong suốt, mờ dần, đổ bóng...đều liên quan đến transparency. Để cải thiện overdraw, hãy thử giảm bớt số lượng transparent object bạn cần.
Ví dụ : xem xét đến việc tạo text với solid color và sử dụng transitions mà không cần sử dụng đến alpha chẳng hạn.
Video bên dưới sẽ cho bạn thấy chi phí của transparency

{@youtube: https://www.youtube.com/watch?v=wIy8g8yNhNk}

### 3.3. Sử dụng clipping khi xây dựng custom view

Đối với custom view, khi bạn ghi đè phương thức *onDraw()*, hệ thống cơ bản không có thông tin gì về nội dung mà bạn đang vẽ, điều này dễ dẫn tới overdraw.
Ví dụ như ảnh 1 stack các thẻ bên trên, bạn rất có thể sẽ vẽ toàn bộ các thẻ ngay cả khi những thẻ bên dưới chỉ hiện thị 1 phần nhỏ :D

Khi đó, clipping ( cắt, xén bớt) là cách để loại trừ các vùng không cần thiết khỏi việc bị vẽ. Clipping chỉ vẽ các phần của người dùng nhìn thấy, giảm bớt việc rendering của hệ thống và tăng hiệu suất của ứng dụng.
Một vài method bạn có thể tìm hiểu :
* [Canvas.clipRect()](https://developer.android.com/reference/android/graphics/Canvas.html#clipRect(float,%20float,%20float,%20float,%20android.graphics.Region.Op))
* [Canvas.quickReject()](https://developer.android.com/reference/android/graphics/Canvas.html#quickReject(float,%20float,%20float,%20float,%20android.graphics.Canvas.EdgeType))

## 4. Mở rộng

Android cung cấp một số bộ công cụ giúp phân tích layout. Tùy vào yêu cầu mà bạn có thể sử dụng một trong các tools dưới đây :
* Profile GPU Rendering
* Show GPU view updates
* Layout Inspector
* Lint
* Systrace and dumpsys


## 5. Tham khảo

1. https://developer.android.com/topic/performance/rendering/

2. https://medium.com/@elifbon/android-application-performance-step-1-rendering-ba820653ad3

