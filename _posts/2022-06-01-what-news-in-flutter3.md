---
title: "[Flutter] Flutter 3.0 có gì mới ?"
date: 2022-06-01 21:15:03 +0700
categories: [Flutter]
tags: [dart, flutter, flutter3, flutter, window]
---

## **Giới thiệu**

11/05/2022, Google I/O chính thức phát hành phiên bản Flutter 3 với rất nhiều cải tiến đáng kể. Đặc biệt dành cho các thiết bị sử dụng window. Trong bài viết này mình sẽ cùng tìm hiểu xem có gì trong bản cập nhật này nhé.

* Flutter hiện đã stable cho cả 3 nền tảng MacOS, Linux và Windows.

* 5248 pull requests được merged.

* Một số thông tin thêm về Flutter 3 bạn có thể tìm hiểu thêm ở đây:

* [ Introducing Flutter 3](https://medium.com/flutter/introducing-flutter-3-5eb69151622f)
* [What’s new in Flutter 3](https://medium.com/flutter/whats-new-in-flutter-3-8c74a5bc32d0)
* [Dart 2.17: Productivity and integration](https://medium.com/dartlang/dart-2-17-b216bfc80c5d)
* [Flutter 3 release notes](https://docs.flutter.dev/development/tools/sdk/release-notes/release-notes-3.0.0)

## I. Thay đổi trên các thiết bị Desktop

Linux và macOS đã chạy ổn định và bao gồm những tính năng sau:

#### 1. Cascading menus and support for the macOS system menu bar

Giờ bạn có thể tạo được thanh menu trên macOS sử dụng widget `PlatformMenuBar`, hỗ trợ chèn các platform menus và điều khiển những gì xuất hiện trong ứng dụng.

#### 2. Full support for international text input on all desktop platforms

Text input bao gồm các ngôn ngữ như tiếng Trung, Nhật, Hàn đã được hỗ trợ đầy đủ trên các platform desktop, bao gồm cả các phương thức nhập của bên thứ ba như Google Japan Input, Sogou.

#### 3. Accessibility on all desktop platforms

Flutter chạy trên Windows, macOS, và Linux được hỗ trợ các dịch vụ trợ năng như trình đọc màn hình (screen-readers), trình điều hướng (accessible navigation) và đảo ngược màu (inverted colors)

#### 4. Universal binaries by default on macOS

Từ phiên bản Flutter 3, các ứng dụng chạy trên macOS được xây dựng dưới dạng nhị phân hợp (universal binaries), cùng với việc hỗ trợ riêng cho máy Mac chạy intel và các thiết bị Apple Silicon mới nhất.

#### 5. Deprecating Windows 7/8 for development

Flutter team khuyến nghị nâng cấp phát triển lên phiên bản Window10. Mặc dù không chặn việc chạy trên các phiên bản cũ (window7,8...) nhưng các phiên bản trên không được Microsoft hỗ trợ nữa và team Flutter cũng chỉ có thẻ cung cấp các phiên bản thử nghiệm giới hạn.

    Chú ý: Việc thông báo trên chỉ ảnh hưởng đến môi trường phát triển khuyến nghị. Team Flutter vẫn tiếp tục cung cấp các hỗ trựo cho các ứng dụng chạy trên Window 7,8.

## II. Thay đổi trên các thiết bị Mobile

Phiên bản cập nhật này bao gồm các thay đổi sau trên nền tảng di động:

#### 1. Foldable phone support

Fluter 3 hỗ trợ các thiết bị gập (foldable phone).

- `MediaQuery` giờ đã bổ sung thêm `DisplayFeatures`, hỗ trợ mô tả các giới hạn (bounds) và các trạng thái của thiết bị như bản lề (linges), gấp (folds) và các đoạn cắt (cutofs). Ngoài ra, tính năng `DisplayFeatureSubScreen` cho phép hiển thị tính năng con mà không bị chồng chéo lên vùng hiển thị `DisplayFeatures` 

‌‌Rất cảm ơn nhóm Microsoft, và đặt biệt là @[andreidiaconu](https://github.com/andreidiaconu), vì những đóng góp của họ!

Bạn có thể thử trải nghiệm [Surface Duo emulator sample](https://docs.microsoft.com/en-us/dual-screen/flutter/samples) , bao gồm một nhánh (branch) đặc biệt của Flutter Gallery, để xem các màn hình kép của Flutter đang hoạt động.

#### 2. iOS variable refresh rate support

Flutter hỗ trựo tốc độ quét màn hình trên các thiết bị IOS có màn hình ProMotion, bao gồm iPhone13 Pro và iPad Pro. Ứng dụng trên các thiết bị trên có thể đạt tốc độ quét 120Hz thay vì bị giới hạn 60Hz như trước đây.

Để biết thêm chi tiết, tham khảo ở [đây](http://flutter.dev/go/variable-refresh-rate)

#### 3 Simplified iOS releases

Team Flutter đã cải tiến thêm 1 số option trong việc build ipa nhằm đơn giản hoá việc phát hành ứng dụng IOS của bạn.

Khi ứng dụng đã sẵn sàng phân phối tới TestFlight hoặc Appstore, chạy câu lệnh sau để tạo Xcode archive file (.xcarchive) và bundle file (.ipa)
~~~dart
flutter build ipa
~~~
Một số params tuỳ chọn thêm:
~~~dart
—-export-method ad-hoc, 
—-export-method development
—-export-method enterprise
~~~

Sau khi app bundle được tạo, hãy upload ứng dụng thông qua [Apple Transport macOS](https://apps.apple.com/us/app/transporter/id1450874784) hoặc sử dụng command lines sử dụng `xcrun altool` bằng cách chạy:

~~~dart
man altool
~~~

Sau khi upload thành công, ứng dụng của bạn sẽ available trên TestFlight hoặc AppStore. Sau khi thiết lập cài đặt ban đầu dự án, bạn không cần phải mở XCode để phát hành ứng dụng nữa.

#### 4. Gradle version update

Trên Flutter 3, nếu bạn tạo một dự án mới bằng công cụ Flutter tool, bạn sẽ thấy các file gen sử dụng phiên bản mới nhất của Gradle plugin và Android gradle. Với các dự án hiện tại, bạn cần migrate lên Gradle 7.4 và 7.1.2 cho Android gradle plugin thủ công.

#### 5. Sunsetting 32-bit iOS/iOS 9/iOS 10

Như đã thông báo vào tháng 2/2022 với bản phát hành 2.10. Flutter sẽ dừng hỗ trợ cho các thiết bị ios32 bit và các phiên bản ios9, ios10. Thay đổi này sẽ ảnh hưởng đến các thiết bị iPhone 4S,5,5C và các thiết bị iPad thế hệ 2,3,4. Flutter 3 sẽ là bản phát hành cuối cùng hộ trợ các thiết bị trên.

Để tìm hiểu thêm về thay đổi này, hãy xem ở đây [RFC: End of support for 32-bit iOS devices](http://flutter.dev/go/rfc-32-bit-ios-unsupported)

## III. Thay đổi trên Web

#### 1. Image decoding

Flutter web hiện nay tự động phát hiện và sử dụng API ImageDecoder trong các trình duyệt được hỗ trợ. Cho đến hôm nay, hầu hết các trình duyệt dựa trên Chromium (Chrome, Edge, Opera, Samsung Browser, v.v.) đã thêm API này.

API mới này decode hình ảnh bất đồng bộ khỏi main thread mà bằng cách sử dụng image codec tích hợp của trình duyệt. Điều này giúp tăng tốc độ giải mã hình ảnh lên gấp 2 lần và main thread sẽ không bao giờ bị chặn nữa, loại bỏ tất cả các lỗi jank do hình ảnh gây ra trước đó.

#### 2. Web app lifecycles

Vòng đời (lifecycle) API mới của các ứng dụng web Flutter cung cấp cho bạn sự linh hoạt để kiểm soát quá trình khởi động của ứng dụng Flutter từ trang HTML hosting và giúp Lighthouse phân tích hiệu suất của ứng dụng của bạn. Điều này áp dụng cho nhiều trường hợp sử dụng, bao gồm các trường hợp được yêu cầu thường xuyên như sau:

* Màn hình splash khi khởi động.
* Hiệu ứng loading indicator.
* Các trang Html thuần.

Để biết thêm chi tiết, xem ở [đây](https://docs.flutter.dev/development/platform-integration/web/initialization)

## IV. Tooling updates

Một số cập nhật cho Flutter và Dart bao gồm:

#### 1. Lint package

Lint 2.0 đã release trên cả 2 bản Flutter và Dart

* Flutter: [https://pub.dev/packages/flutter_lints/versions/2.0.0](https://pub.dev/packages/flutter_lints/versions/2.0.0)
* Dart: [https://pub.dev/packages/lints/versions/2.0.0](https://pub.dev/packages/lints/versions/2.0.0)

Các ứng dụng được tạo từ Flutter 3 bằng lệnh `flutter create` sẽ được tự động sử dụng bộ lints 2.0. Các plugin & package được khuyến nghị chuyển sang phiên bản 2.0 để cập nhật.
~~~dart
flutter pub upgrade --major-versions flutter_lints
~~~

#### 2. Performance improvements

* Tính năng repaint partial được bật trên các thiết bị android được hỗ trợ. Cảm ơn @[knopp](https://github.com/knopp)
* Cải thiện hiệu suất của các OpacityAnimation.
* Cải thiện mức độ ưu tiên của raster & ui thread trên Android và IOS cao hơn các thread khác. Giúp tăng ~20% tốc độ frame build. Cảm ơn @[JsouLiang](https://github.com/JsouLiang)
* Cải thiện frame dropped trên ios. Cảm ơn @[ColdPaleLight ](https://github.com/ColdPaleLight)

#### 3. Impeller

Flutter đã làm việc chăm chỉ để tìm ra giải phát giải quyết tình trạng Jank ở những frame đầu trên trên IOS và các nền tảng khác. Trong bản Flutter 3, bạn có thể xem trước thử nghiệm này trên IOS có tên là Impeller.

- Impeller sẽ biên dịch trước một bộ đổ bóng nhỏ và đơn giản hơn để chúng khong biên dịch lại khi ứng dụng đang chạy. Đây là nguyên nhân của Jank trong Flutter.

- Impeller vẫn trong giai đoạn phát triển và còn khá lâu mới hoàn thành. Không phải tất cả tính năng của Flutter đều được triển khai, nhưng Flutter team khá hài lòng với độ trung thực và hiệu suất của nó trên ứng dụng [Flutter Gallery](https://github.com/flutter/gallery) mà team đang phát triển và chia sẻ về tiến trình của mình. Một điều đặc biệt là frame kém nhất trong animation chuyển tiếp của ứng dụng này đã nhanh hơn khoảng 20 lần so với trước đây.

- Impeller có thể được kích hoạt dưới dạng flag trên IOS. 
  - Cách 1: Bản có thể thêm nó khi gọi `flutter run` vơi flag `—-enable-impeller`
  - Cách 2: Trong `Info.plist`, set `FLTEnableImpellerflag` = true


#### 4. Inline ads on android

* Tăng hiệu suất package [google_mobile_ads](https://pub.dev/packages/google_mobile_ads)
  
## V. Các cập nhật khác

#### 1. Material 3

* Flutter 3 hỗ trợ [Material Design 3](https://m3.material.io/), thế hệ tiếp theo của Material Design
* Flutter 3 cung cấp hỗ trựo chọn cho M3 bao gồm các tính năng như dynamic color, color system, text style...

Bạn có thể thử các tính năng mới của Material 3 trong codelab sau : [Take your Flutter app from Boring to Beautiful](https://codelabs.developers.google.com/codelabs/flutter-boring-to-beautiful)


#### 2. Theme extensions

Flutter giờ cho phép thêm bất kỳ thứ gì vào `ThemeData` với concept gọi là Theme Extensions. Triển khai copyWith, lerf,...

#### 3. Ads

Team Flutter biết rằng điều quan trọng là các publishers phải yêu cầu sự đồng ý đối với quảng cáo được cá nhân hóa và xử lý các yêu cầu về tính minh bạch theo dõi ứng dụng (ATT) của Apple.

Để biết thêm chi tiết, tham khảo Google mobiles ads package [google_mobile_ads](https://pub.dev/packages/google_mobile_ads)

## VI. Breaking changes

Với mong muốn phát triển và cải thiện Flutter, team mong muốn số lượng breaking changes là tối thiểu. Tại bản phát hành Flutter 3, chúng ta có một số breaking change sau:

* [Deprecated API removed after v2.10](https://docs.flutter.dev/release/breaking-changes/2-10-deprecations)
* [Page transitions replaced by ZoomPageTransitionsBuilder](https://docs.flutter.dev/release/breaking-changes/page-transition-replaced-by-ZoomPageTransitionBuilder)
* [Migrate useDeleteButtonTooltip to deleteButtonTooltipMessage of Chips](https://docs.flutter.dev/release/breaking-changes/chip-usedeletebuttontooltip-migration)

Nếu bạn đang sử dụng bất kì bộ API nào trong số kể trên, hãy tham khảo [migration guide](https://docs.flutter.dev/release/breaking-changes).

## Conclusion

Cảm ơn các bạn đã đọc bài viết. Happy coding ^^

### References

* [https://medium.com/flutter/whats-new-in-flutter-3-8c74a5bc32d0](https://medium.com/flutter/whats-new-in-flutter-3-8c74a5bc32d0)