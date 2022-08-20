---
title: "[Flutter] Effective Dart (Part I)"
date: 2020-12-24 21:15:03 +0700
categories: [Flutter]
tags: [flutter, dart]
---

Bài viết giúp bạn làm thế nào để xây dựng mã code clean clear hơn. Chi tiết ở [đây](https://dart.dev/guides/language/effective-dart)

Để áp dụng vào dự án nhanh và dễ dàng hơn, dart analyzer cung cấp lint - một bộ quy tắc cần tuân theo khi viết code. Hãy tham khảo nó ở [đây](https://dart.dev/guides/language/analysis-options)

Linting là một công cụ cực kì hữu ích. Nó giúp bạn thay đổi hành vi về cách viết code và giúp bạn code một cách clean.

## Introduction

Bài viết bao gồm 4 phần :

* [Style Guide](https://dart.dev/guides/language/effective-dart/style) – Một số quy tắc bố trí và sắp xếp code, hoặc ít nhất là những gì mà dart format chưa handle cho bạn. Guide này cũng chỉ định cách format như nào cho hợp lý : camelCase, using_underscores, etc...

* [Documentation Guide](https://dart.dev/guides/language/effective-dart/documentation) – Hướng dẫn cách comment, viết docs... 

* [Usage Guide](https://dart.dev/guides/language/effective-dart/usage) – Làm cách nào để implement 1 đoạn mã sử dụng các statement hay expression hợp lý...

* [Design Guide](https://dart.dev/guides/language/effective-dart/design) – Đây là phần nhỏ nhưng scope rộng nhất. Cách khai báo, thiết kế consistent, usable APIs

Để xem 1 cách tổng quan, bạn có thể tham khảo [đây](https://dart.dev/guides/language/effective-dart#summary-of-all-rules).

Trong bài viết này, mình sẽ chia sẻ về 2 chủ đề đầu tiên nhé.

## 1. STYLE GUIDE

Một số quy tắc bố trí và sắp xếp code, hoặc ít nhất là những gì mà dart format chưa handle cho bạn. Guide này cũng chỉ định cách format như nào cho hợp lý : camelCase, using_underscores, etc…<br/>

### 1.1.1 UpperCamelCase

- Viết hoa chữ cái đầu tiên của mỗi từ
- Sử dụng với : Classes, enum types, typedefs, type parameters...

~~~dart
class SliderMenu { ... }

class HttpRequest { ... }

typedef Predicate<T> = bool Function(T value);
~~~

### 1.1.2 lowercase_with_underscores

- Chữ thường + gạch dưới.
- Sử dụng với : libraries, package, directories, source file, import as

~~~dart

library peg_parser.source_scanner;

import 'file_system.dart';
import 'slider_menu.dart';

// Extra

import 'dart:math' as math;
import 'package:angular_components/angular_components' as angular_components;
import 'package:js/js.dart' as js;
~~~

### 1.1.3 lowerCamelCase

- Tên viết hoa chữ cái đầu tiên của mỗi từ, ngoại trừ từ đầu tiên
- Áp dụng với: 
    * class members, top-level definitions, variables, parameters, and named parameters
    * const variables

~~~dart
var count = 3;

HttpRequest httpRequest;

void align(bool clearItems) {
  // ...
}
~~~

### 1.1.4 PREFER using _, __, etc. for unused callback parameters.

- Khi callback params không dùng đến, nên dùng gạch chân _ . Giúp việc đọc hiểu nhanh hơn.

### 1.2.1 DO place “dart:” imports before other imports.

- Import những thư viện liên quan đến dart trước tiên

~~~dart
import 'dart:async';
import 'dart:html';

import 'package:bar/bar.dart';
import 'package:foo/foo.dart';
~~~

### 1.2.2 DO place “package:” imports before relative imports.

- Sử dụng đường dẫn tương đối khi import

~~~dart
import 'package:bar/bar.dart';
import 'package:foo/foo.dart';

import 'util.dart';
~~~

### 1.2.3 DO specify exports in a separate section after all imports.

- Export và import nên phân cách nhau

~~~dart
import 'src/error.dart';
import 'src/foo_bar.dart';

export 'src/error.dart';
~~~

### 1.2.4 DO sort sections alphabetically.

- Sort thứ tự import/export... theo alphab

### 1.3.1 DO use curly braces for all flow control statements.

- Sử dụng {} hợp lý cho từng trường hợp
* if else nhiều : {}
* if chỉ có 1 mệnh đề => không dùng

~~~dart
if (isWeekDay) {
  print('Bike to work!');
} else {
  print('Go dancing or read a book!');
}

// 1 clause

if (arg == null) return defaultValue;

// 2+ clauses
if (overflowChars != other.overflowChars) {
  return overflowChars < other.overflowChars;
}

~~~

## 2. DOCUMENT GUIDE

Hướng dẫn cách comment, viết docs... <br/>

### 2.1.1 DO format comments like sentences.
~~~dart
// Not if there is nothing before it.
if (_chunks.isEmpty) return false;
~~~

### 2.1.2 DON’T use block comments for documentation.

~~~dart
GOOD
void greet(String name) {
  // Assume we have a valid name.
  print('Hi, $name!');
}

BAD
void greet(String name) {
  /* Assume we have a valid name. */
  print('Hi, $name!');
}
~~~

=> Không sử dụng comments theo bloc với documents

### 2.2.1 DO use /// doc comments to document members and types.

- Sử dụng /// với các comment mục đích định nghĩa, hướng dẫn 

~~~dart
GOOD
/// The number of characters in this chunk when unsplit.
int get length => ...

BAD
// The number of characters in this chunk when unsplit.
int get length => ...
~~~

### 2.2.2 DO start doc comments with a single-sentence summary.

- Bắt đầu viết doc bằng 1 dòng và ngăn cách với graph dưới nếu có chú thích thêm

~~~dart
/// Deletes the file at [path].
///
/// Throws an [IOError] if the file could not be found. Throws a
/// [PermissionError] if the file is present but could not be deleted.
void delete(String path) {
  ...
}
~~~

### 2.2.3 DO use square brackets in doc comments to refer to in-scope identifiers.

- Sử dụng ngoặc vuông để refer đến thuộc tính, nội dung đã được định danh

~~~dart
/// Throws a [StateError] if ...
/// similar to [anotherMethod()], but ...
~~~

### 2.2.4 Markdown

- Có thể sử dụng markdown khi comment. Dart có thư viện markdown support việc này.

~~~markdown
/// This is a paragraph of regular text.
///
/// This sentence has *two* _emphasized_ words (italics) and **two**
/// __strong__ ones (bold).
///
/// A blank line creates a separate paragraph. It has some `inline code`
/// delimited using backticks.
///
/// * Unordered lists.
/// * Look like ASCII bullet lists.
/// * You can also use `-` or `+`.
///
/// 1. Numbered lists.
/// 2. Are, well, numbered.
/// 1. But the values don't matter.
///
///     * You can nest lists too.
///     * They must be indented at least 4 spaces.
///     * (Well, 5 including the space after `///`.)
///
/// Code blocks are fenced in triple backticks:
///
/// ```dart
/// this.code
///     .will
///     .retain(its, formatting);
/// ```
///
/// The code language (for syntax highlighting) defaults to Dart. You can
/// specify it by putting the name of the language after the opening backticks:
///
/// ```html
/// <h1>HTML is magical!</h1>
/// ```
///
/// Links can be:
///
/// * https://www.just-a-bare-url.com
/// * [with the URL inline](https://google.com)
/// * [or separated out][ref link]
///
/// [ref link]: https://google.com
///
/// # A Header
///
/// ## A subheader
///
/// ### A subsubheader
///
/// #### If you need this many levels of headers, you're doing it wrong
~~~

