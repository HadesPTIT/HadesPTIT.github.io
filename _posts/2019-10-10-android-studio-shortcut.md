---
title: "[Tips & Trick] Android Studio shortcuts"
date: 2019-10-10 21:15:03 +0700
categories: [Android]
tags: [android, tips, as, android studio]
---

## Introduction

Hi ae, hôm nay mình sẽ chia sẻ một vài phím tắt hữu ích khi lập trình với Android Studio.<br/>
Nếu muốn tự đọc về đống shortcuts thì bạn chỉ cần check trong ***Preferences > Keymap***

### 1. Clipboard history

Để mở lịch sử clipboard của bạn : **CMD + SHIFT + V**

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/w4rltnyt7u_cmd-shift-v.png)

Popup sẽ bao gồm danh sách lịch sử mà bạn clipboard lại, đỡ mất công phải paste vào đâu đó phải không ? :D

### 2. Last used files

Để mở popup danh sách những file bạn đã xem gần nhất : **CMD + E**

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/vjeslbnsyo_cmd-e.png)

### 3. Last used files with preview

=> **CMD + SHIFT + E**

Thay vì chỉ hiển thị danh sách các file đã mở gần nhất như **#2** thì sẽ hiển thị cùng với preview vùng mà bạn đã trỏ vào cuối cùng ở các file. Preview hiển thị được tối đa 5 dòng cho mỗi file.

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/3dpc9jqwtm_cmd-shift-e.png)

### 4. Search

 Phím tắt khá phổ biến phải không :D
 
 Để mở popup **Search All** chỉ cần nháy đúp : **SHIFT x2**
 
Chọn **Tab** sau đó để di chuyển sang các tab khác nếu bạn không muốn nhớ phím tắt cho đống còn lại. Nếu cần thì bên dưới đã có :

* Search class : **CMD + O**
* Search File : **CMD + SHIFT + O**
* Search symbol : **CMD + ALT + O**
* Search action : **CMD + SHIFT + A**
![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/1nmj8695lj_search.png)

### 5. Find Everywhere

=> **CMD + SHIFT + F**

Cũng là tìm kiếm nhưng theo dạng tìm kiếm văn bản đơn thuần. Bạn có thể thu hẹp phạm vi tìm theo bộ Filter.

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/yrj8ej4ih_cmd-shift-f.png)

### 6. Run

Để chạy project với config đã chọn : **CTRL + R**

### 7. Jump to first / last file

Để nhảy tới vị trí đầu hoặc cuối file : **CMD + ⌥ + < / >**

### 8. Jump to last Edit

Để di chuyển đến vị trí chỉnh sửa cuối cùng : **CMD + SHIFT + BACKSPACE**

### 9. Check cursor history

=> **CMD + {**

Di chuyển đến những nơi con trỏ đã trỏ đến gần nhất. Phím tắt khá hữu ích khi bạn đang dò debug mà muốn back lại chẳng hạn :D

### 10. Fast selection

=> **⌥ + ARROW_UP**

Phím tắt mình rất hay dùng để select một vùng code. Nó sẽ chọn vùng bao con trỏ theo các block lớn dần.

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/y2z76boabt_fast-select.gif)

### 11. Move functions

Để di chuyển các function tới trước / sau các khối khác : **CMD + SHIFT + ARROW UP/DOWN**

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/9gd8i4w0om_move-function.gif)

### 12. Move line by line

Nếu chỉ cần di chuyển vị trí code theo dòng : **⌥ + SHIFT + ARROW UP/DOWN**

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/uj5a03c09p_line-by-line.gif)

### 13. Duplicate code

Để tạo bản sao của line đang trỏ đến hoặc cả khối code đang chọn : **CMD + D**

### 14. Multi Selection

Tính năng ưa thích của mình mỗi khi phải sửa một đống line liền kề nhau với cùng một nội dung. Có 2 cách để làm :

* Giữ phím **⌥**  + giữ chuột trái để chọn vùng cần sửa.
* Click duplicate **⌥** .  Lần click thứ 2 giữ phím ⌥ + dùng phím mũi tên để chọn vùng sửa.

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/qakbj1dm6c_multi-select.gif)

### 15. Extract

Tách khối code đã chọn thành các function, variable, params...

2 thằng hay được dùng nhất :

* Extract variable : **⌥ + CMD + V**
![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/cn236r2xy7_extract-variable.gif)

* Extract function : **⌥ + CMD + M**

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/aoxm39z5mo_extract-method.gif)

### 16. Refactor

Nếu bạn muốn mở menu refactor file hay code : **Control ⌃ + T**

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/dzw65udiye_refactor.png)

### 17. Surround

=> **⌥ + CMD + T**

Phím tắt giúp tạo bao ngoài cho vùng select. Ví dụ như try/catch, if/else, region...

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/sq253utdvu_surround.png)


### 18. Parameters

=> **CMD + P**

Khi sử dụng 1 function mà quên mất nó cần params gì, thứ tự ra sao... 

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/dhvxdd2oev_params.gif)

### Conclution

Hi vọng đống shortcut trên sẽ giúp ae dev nhanh hơn nhiều. Happy coding :D

### References

* [Android studio shortcuts ](https://proandroiddev.com/android-studio-shortcuts-how-to-manage-the-ide-without-a-mouse-b3203459e2ea)