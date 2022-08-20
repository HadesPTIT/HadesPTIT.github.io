---
title: "[Flutter] Package Dependencies"
date: 2020-08-08 21:15:03 +0700
categories: [Flutter, Dart]
tags: [flutter, dart, package dependencies, vs, pub-dev]
---

## Introduction

Hệ sinh thái Dart sử dụng các **`package`** để quản lý các phần mềm dùng chung như các libs hay tool.

Trong bài viết này, mình sẽ giới thiệu cách làm việc với các dependencies trong `pubspec.yaml`.

## Overview

Các dependency được chỉ định trong tệp `pubspec.yaml`. Để chỉ định 1 dependency:
~~~yaml
dependencies:
  transmogrify: ^1.0.0
~~~

Dependency với package `transmogrify` được thiết lập cho phép version nằm trong khoảng `[1.0.0 và < 2.0.0)`
Mình sẽ nói chi tiết hơn ở mục [version constraints]() bên dưới.

Nếu muốn chỉ định 1 **source** nhất định, syntax sẽ khác 1 chút :
~~~yaml
dependencies:
  transmogrify:
    hosted:
      name: transmogrify
      url: http://some-package-server.com
    version: ^1.0.0
~~~

Dependency theo package `transmogrify`  sử dụng `source` được định nghĩa bên trong `hosted`.
Trong đa số trường hợp, bạn chỉ sử dụng cấu trúc `packagename: version` mà thôi.

## Dependency sources

Pub sử dụng các source sau để xác định package:
* SDK
* Hosted packages
* Git packages
* Path packages

#### 1. SDK
SDK source được sử dụng cho bất kì SDK nào đã bao gồm các package, bản thân chúng cũng có thể là các dependencies.<br/>
Hiện tại, `Flutter` là SDK duy nhất được hỗ trợ.
~~~yaml
dependencies:
  flutter_driver:
    sdk: flutter
    version: ^0.0.1
~~~

#### 2. Hosted packages

Hosted package là nơi có thể download từ side [pub.dev](https://pub.dev/) ( hoặc một máy chủ HTTP khác sử dụng cùng 1 API).
~~~yaml
dependencies:
  transmogrify: ^1.4.0
~~~
Nếu bạn muốn sử dụng máy chủ riêng mình, bạn có thể tự chỉ định URL:
~~~yaml
dependencies:
  transmogrify:
    hosted:
      name: transmogrify
      url: http://your-package-server.com
    version: ^1.4.0
~~~

#### 3. Git packages

Đôi khi, bạn cần sử dụng 1 package nào đó mà nó chưa được release và vẫn đang trong quá trình phát triển chả hạn, bạn có thể depend trực tiếp vào Git repo luôn.
~~~yaml
dependencies:
  kittens:
    git: https://github.com/munificent/kittens.git
~~~
Nếu bạn muốn depend vào một commit nhất định, hay branch, hay tag. Hãy sử dụng **`ref`** argument.
~~~yaml
dependencies:
  kittens:
    git:
      url: git@github.com:munificent/kittens.git
      ref: some-branch
~~~
**`ref`** có thể là bất cứ thứ gì Git cho phép dùng để định danh 1 commit. Xem chi tiết ở [đây](https://mirrors.edge.kernel.org/pub/software/scm/git/docs/user-manual.html#naming-commits)

 Mặc định, Pub sẽ coi package nằm ở thư mục gốc của Git repo.
 Nếu muốn chỉ định 1 vị trí khác trong repo. Sử dụng **`path`** argument:
 ~~~yaml
 dependencies:
  kittens:
    git:
      url: git@github.com:munificent/cats.git
      path: path/to/kittens
~~~

#### 4. Path package

Đôi khi, bạn cần làm việc trên nhiều package liên quan cùng lúc và muốn thay đổi phụ thuộc trực tiếp vào local để debug chẳng hạn:
~~~yaml
dependencies:
  transmogrify:
    path: /Users/me/transmogrify
~~~

### Version constraints

Dart sử dụng semantic versioning, giúp bạn biết version nào sẽ hoạt động.

#### 1. Caret syntax

Caret syntax hay gọi là cú pháp dấu mũ là cách ngắn gọn để thể hiện ràng buộc version phổ biến.
**`^version`** có nghĩa là phạm vi của tất cả các phiên bản được đảm bảo tương thích ngược với phiên bản được chỉ định.

Ví dụ : **^1.2.3** tương đương với **">= 1.2.3 & < 2.0.0"**

**Note:** cú pháp mũ được giới thiệu từ Dart 1.8.3, hãy đảm bảo rằng sdk version của bạn >= nhé.
~~~yaml
environment:
  sdk: '>=1.8.3 <3.0.0'
~~~

#### 2. Traditional syntax

Sử dụng cú pháp truyền thống :

* **`any`** : Chuỗi string any cho phép bất kì version nào. Nó tương đương với việc bỏ trống version, nhưng rõ ràng hơn. Mặc dù any được cho phép, nhưng ta ko nên dùng nó.
* **`1.2.3`** : chỉ hoạt động với đúng version 1.2.3. 
* **`>=1.2.3`** : hoạt động với các phiên bản lớn hơn hoặc bằng 1.2.3.
* **`>1.2.3`** : hoạt động với các phiên bản lớn hơn 1.2.3
* **`<=1.2.3`** hoạt động với các phiên bản nhỏ hơn hoặc bằng 1.2.3
* **`<1.2.3`** hoạt động với các phiên bản nhỏ hơn 1.2.3
* **`>=1.2.3 <1.8.0`** : hoạt động với các phiên bản từ 1.2.3 đến dưới 1.8.0.

### Regular & Dev dependencies

Pub hỗ trợ 2 kiểu dependencies : 
* Regular dependencies ( phụ thuộc thông thường )
* Dev dependencies ( phụ thuộc phát triển ).

Dev dependencies khác với regular ở điểm các phụ thuộc này sẽ được **`ignore`**.

Ví dụ: 
~~~yaml
dev_dependencies:
  test: '>=0.5.0 <0.12.0'
~~~

### Dependency overrides

Sử dụng dependency_overrides  để tạm thời override tất cả các tham chiếu đến dependency này.
~~~yaml
name: my_app
dependencies:
  transmogrify: ^1.2.0
dependency_overrides:
  transmogrify: '3.2.1'
~~~

### Conclusion

Chúc các bạn coding vui vẻ :D

### References

* [https://dart.dev/tools/pub/dependencies#version-constraints](https://dart.dev/tools/pub/dependencies#version-constraints)
