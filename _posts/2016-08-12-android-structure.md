---
title: "[Android] Cách tổ chức thư mục cho 1 project Android"
date: 2016-08-12 21:15:03 +0700
categories: [Android]
tags: [android, structure]
---

Hi mọi người!
<br/>
Dạo gần đây mình có xây dựng 1 dự án android nhỏ. Dưới đây là một cách tổ chức thư mục mà mình cho là khá hợp lý. Hi vọng mọi người có thể tham khảo và góp ý.

~~~html
├─ com.abc
│  ├─ activities
│  ├─ adapters
│  ├─ fragments
│  ├─ example
│  ├─ interfaces
│  ├─ models
│  ├─ navigates
│  ├─ networks
│  ├─ notifications
│  ├─ utils
│  └─ views
~~~

Trong đó mỗi thành phần sẽ có 1 ý nghĩa riếng như sau :
* **`com`** :  Tên công ty ( đồng thời nằm trong thư mục root package
* **`abc`** : Tên viết tắt của team
* **`activities`** : Các activity sẽ được đưa vào đây
* **`adapters`** :  Dành cho các custom adapter
* **`fragments`** : Toàn bộ các fragment
* **`example`** : Tên của project. Trong này sẽ chứa tất cả những gì liên quán đến project bao gồm file Config.java, Application.java.. .
* **`models`** : Làm việc với preference, làm việc với SQLite,.. .
* **`navigates`** : Các phương thức điều khiển vào ra của fragment sẽ được đưa vào đây
* **`networks`** : Xử lý network, picasso, Volley, OKHttp.. .
* **`notifications`** : Tất cả mọi hoạt động liên quan đến notification, GCM
* **`utils`** : Các lớp hỗ trợ trong quá trình sử dụng như StorageUtil.java, ImageUtil.java, TextUtil.java.. 
* **`views`** : Khai báo các custom view.