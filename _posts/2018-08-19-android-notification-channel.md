---
title: "[Android] Khám phá notification channel trên Android O"
date: 2018-08-19 21:15:03 +0700
categories: [Android]
tags: [android, notification channel]
---

Trong bài viết này chúng ta cùng tìm hiểu về Notification Channels và cách đưa chúng vào ứng dụng như thế nào nhé !

# Notification Channels là gì ?
Notification Channels cung cấp cho chúng ta khả năng nhóm các thông báo mà ứng dụng của ta gửi vào các nhóm để quản lý. Một khi những notifications nằm trong những kênh này, người dùng phải quản lý chúng.

Hãy nhìn vào các tùy chọn bên dưới :

![](https://images.viblo.asia/bfc711d9-73f6-4bff-b655-0678cbb7225d.png)

Bắt đầu từ bên trái, bạn có thể thấy trong màn hình đầu tiên cài đặt thông báo cho ứng dụng. Với các tùy chọn đó bạn có thể :
1. Chặn tất cả các kênh thông báo cho ứng dụng. Có nghĩa là tất cả thông báo từ ứng dụng của bạn sẽ không bao giờ được hiển thị trên thiết bị của người dùng.
2. Nếu được hỗ trợ, người dùng có thể xác định xem thông báo của ứng dụng có thể được hiển thị dưới dạng huy hiệu (badge) hay không.
3. Các danh mục thông báo tồn tại cho ứng dụng. Người dùng có thể bật hoặc tắt các tính năng này.

Tiếp đến là màn hình thứ hai, người dùng có thể truy cập vào nó khi chọn một danh mục từ mục Category ở màn hình đầu. Ở đây người dùng có thể :
1. Chặn tất cả thông báo của ứng dụng đến từ kênh này.
2. Hiển thị thông báo từ kênh này tại màn Home nếu được hỗ trợ.
3. Thiết đặt mực độ quan trọng của thông báo. Tùy chọn này thể hiện rõ khi nhận được một thông báo.
Các mức độ được đưa ra có thể thấy được tại màn hình cuối.

Một tính năng khá là hay nữa đó là chúng ta có thể nhóm các kênh thống báo lại thành các nhóm riêng biệt. Điều này giúp ta có thể có cùng một kênh thông báo trên nhiều chế độ ứng dụng.
![](https://images.viblo.asia/10ebc426-7b06-4bb3-b47a-40c2aa7c5ce0.png)


Lấy một ví dụ nào :

Ứng dụng của mình có thể hỗ trợ chế độ như Personal, Business, Child, Parent.. .
Điều này cho phép cung cấp các tùy chọn quản lý cài đặt thông báo trên nhiều nhóm.
Trên các phiên bản Android cũ trước Android O, các tính năng mới này bị bỏ qua hoàn toàn.

Đến đây thì chắc mọi người cũng biết thêm một chút về Notification Channels rồi, giờ thì bắt tay vào code thôi :D

# Tạo 1 Notification Channel

Khá đơn giản để tạo 1 kênh thông báo trong ứng dụng của mình :
```java
NotificationManager notificationManager = (NotificationManager) getSystemService(Context.NOTIFICATION_SERVICE);
String channelId = "some_channel_id";
CharSequence channelName = "Some Channel";
int importance = NotificationManager.IMPORTANCE_LOW;
NotificationChannel notificationChannel = new NotificationChannel(channelId, channelName, importance);
notificationChannel.enableLights(true);
notificationChannel.setLightColor(Color.RED);
notificationChannel.enableVibration(true);
notificationChannel.setVibrationPattern(new long[]{100, 200, 300, 400, 500, 400, 300, 200, 400});
notificationManager.createNotificationChannel(notificationChannel);
```
Khi thao tác với Notification Channel, bạn sẽ được cung cấp một số phương thức :

* **getId()** — Trả về Id của channel.
* **enableLights()** — Nếu thiết bị hỗ trợ đèn thông báo, nó cho phép kênh thông báo này có hiển thị ánh sáng hay không.
* **setLightColor()** — Xác định màu được sử dụng cho đèn thông báo.
* **enableVibration()** — Rung khi thông báo được hiển thị trên thiết bị.
* **setVibrationPattern()** — Kiểu mẫu rung được sử dụng, được định nghĩa bằng 1 mảng long[].
* **setImportance()** — Thiết đặt mức độ gián đoạn của kênh thông báo.
* **setSound()** — Cung cấp 1 Uri để phát âm thanh khi thông báo được đăng lên kênh này.
* **setGroup()** — Thiết đặt group mà thông báo được gán vào.
* **setBypassDnd()** — Có bỏ qua chế độ không làm phiền không (Not Disturb mode).
* **setLockScreenVisibility()** — Thông báo từ kênh này được hiển thị trên màn hình khóa không.
* **canShowBadge()** — Thông báo từ kênh này có thể xuất hiện dưới dạng huy hiệu (badge) trên launcher ứng dụng không.

Bạn có thể đọc thêm về Notification Channels [ở đây](https://developer.android.com/reference/android/app/NotificationChannel#enableLights%28boolean%29).

OK, sau khi tạo xong channel ta có thể sử dụng Id của channel vừa tạo để tạo một Notification hiển thị trên thiết bị người dùng.

# Tạo 1 Notification Channel Group
Như đã đề cập bên trên, ta có thể nhóm các kênh thông báo lại thành các nhóm có thể quản lý được. Điều này cho phép ta có các kênh thông báo có tên giống nhau được xác định bởi các phần quản lý khác nhau. Điều này rất hữu ích nếu ứng dụng hỗ trợ nhiều tài khoản (multi accounts) hay chế độ (mode).

Để tạo một nhóm kênh thông báo, ta gọi phương thức `createNotificationChannelGroup()` được cung cấp bởi Notification Manager. Khi đó, ta phải truyền vào 1 instance của `NotificationChannelGroup` bao gồm 2 tham số :

* **Group Id** : Id định danh cho group này.
* **Group name** : Tên của group được hiển thị với người dùng.

```java
String groupId = "some_group_id";
CharSequence groupName = "Some Group";
NotificationManager notificationManager = (NotificationManager) getSystemService(Context.NOTIFICATION_SERVICE);
notificationManager.createNotificationChannelGroup(new NotificationChannelGroup(groupId, groupName));
```

OK, sau khi tạo xong một channel group ta có thể gọi phương thức `setGroup()` khi tạo được một instance của Notification Channel bằng cách truyền vào Id của group mà muốn liên kết đến.

# Tạo 1 Notification

Kể từ Android O, khi bạn tạo 1 notification bắt buộc phải khai báo một channel bằng phương thức `setChannel()`. Điều này nhằm đảm bảo rằng thông báo thuộc về một kênh có thể được người dùng quản lý từ cài đặt thiết bị của họ.
```java

NotificationManager notificationManager = 
        (NotificationManager) getSystemService(Context.NOTIFICATION_SERVICE);
int notifyId = 1;
String channelId = "some_channel_id";

Notification notification = new Notification.Builder(MainActivity.this)
        .setContentTitle("Some Message")
        .setContentText("You've received new messages!")
        .setSmallIcon(R.drawable.ic_notification)
        .setChannel(channelId)
        .build();
        
notificationManager.notify(id, notification);
```

# Đọc thiết lập kênh thông báo
Để xem các thiết đặt mà người dùng đã áp dụng cho 1 kênh thông báo xác định :
```java
NotificationManager notificationManager =
        (NotificationManager) getSystemService(Context.NOTIFICATION_SERVICE);
NotificationChannel notificationChannel = notificationManager.getNotificationChannel("some_id");
```
Bạn cũng có thể truy xuất tất cả các kênh thông báo :
```java
NotificationManager notificationManager =
        (NotificationManager) getSystemService(Context.NOTIFICATION_SERVICE);
List<NotificationChannel> notificationChannels = notificationManager.getNotificationChannels();
```

# Cập nhật thiết lập kênh thông báo

Sau khi kênh thông báo được tạo, người dùng chịu trách nhiệm sửa đổi bất kì cài đặt nào của kênh đó. Mặc dù không thể cung cấp khả năng sửa đổi từ bên trong ứng dụng nhưng ta có thể hướng người dùng đến trang cài đặt thông báo thông qua Intent :

```java
Intent intent = new Intent(Settings.ACTION_CHANNEL_NOTIFICATION_SETTINGS);
intent.putExtra(Settings.EXTRA_CHANNEL_ID, notificationChannel.getId());
intent.putExtra(Settings.EXTRA_APP_PACKAGE, getPackageName());
startActivity(intent);
```

# Xóa Notification Channel

Nếu không cần sử dụng kênh thông báo nữa, ta có thể sử dụng phương thức `deleteNotificationChannel()` và truyền vào Id của channel muốn xóa.

```java
NotificationManager notificationManager =
        (NotificationManager) getSystemService(Context.NOTIFICATION_SERVICE);
String channelId = "some_channel_id";
NotificationChannel notificationChannel = 
        notificationManager.getNotificationChannel(channelId);
notificationManager.deleteNotificationChannel(notificationChannel);
```

# Kết luận
Trên đây là những giới thiệu cơ bản về `Notification Channel`. Hi vọng qua bài viết này bạn có thể hiểu được tổng quan về Notification Channel cũng như cách tạo channel, channel group hay cách quản lý chúng.

# Tham khảo
Nguồn tham khảo :

https://developer.android.com/guide/topics/ui/notifiers/notifications#ManageChannels

https://medium.com/exploring-android/exploring-android-o-notification-channels-94cd274f604c