---
title: "[CI/CD] Tích hợp CI/CD với Gitlab"
date: 2019-06-19 21:15:03 +0700
categories: [CI/CD]
tags: [cicd, gitlab]
---

### 1.Introduction

![](https://images.viblo.asia/d3358d65-f8ad-4d94-b371-9bbcc6afe3f0.png)

Chắc hẳn các bạn đã từng nghe đến hệ thống CI/CD. Mục tiêu của nó là tự động hóa các khâu trong quy trình phát triển phần mềm hiện nay. Nếu chưa biết nó là gì, hãy tham khảo bài viết [này](https://viblo.asia/p/ci-cd-va-devops-07LKXYXDZV4) của anh @nghiadd để hiểu cơ bản về CI/CD.

Hiện nay có rất nhiều hệ thống như **Travis**, **Jenkin**, **Circle**, **Gitlab** có thể giúp bạn làm được điều đó. 
Trong bài viết này, mình sẽ giới thiệu với các bạn tổng quan `Gitlab CI` và cách để xây dựng một hệ thống cơ bản.

### 2. Git Flow

![](https://images.viblo.asia/703279f5-241c-472d-8adc-deb324758551.png)

Developer hoàn thành một task nào đó và push commit lên Gitlab để mọi người review. 

Khi đó `Gitlab CI` cũng bắt đầu thực hiện công việc mà nó được giao. Nó sử dụng file `.gitlab-ci.ym` nằm trong thư mục gốc của repo để cấu hình project sử dụng các `Runner`. Một `Pipeline` CI sinh ra và report sẽ được hiển thị trên giao diện.

![](https://images.viblo.asia/a50f0294-80d3-4a49-8e89-5a95f113ef66.PNG)

Vậy `Pipelines` là gì ?

### 3. Pipelines 

Là thành phần cấp cao nhất của tích hợp, phân phối và triển khai liên tục.

~~~
Pipelines are the top-level component of continuous integration, delivery, and deployment.
~~~
**Pipeline** bao gồm :
* **Jobs** : Các công việc được giao thực thi. ( Ví dụ : biên dịch mã hoặc chạy test )
* **Stage** : Xác định các thời điểm và cách thực hiện. ( Ví dụ : test chỉ chạy sau khi biên dịch thành công )

**Pipeline** hoạt động theo nguyên tắc sau :
* Tất cả các công việc trong cùng một `stage` được `Runner` thực hiện song song, nếu có đủ số lượng `Runner` đồng thời.
* Nếu Success, pipeline chuyển sang `stage` tiếp theo.
* Nếu Failed, pipeline sẽ dừng lại. Có một ngoại lệ là nếu job được đánh dấu làm thủ công, thì dù bị fail thì pipeline vẫn tiếp tục.

Bên dưới là ví dụ về đồ thị Pipeline thông thường :

![](https://images.viblo.asia/af6e91ef-0fd1-41e9-9efb-5127a7b239e9.png)

Tóm lại, các bước để **Gitlab CI** hoạt động như sau :
* Thêm` .gitlab-ci.yml` vào thư mục gốc của repo.
* Cấu hình gitlab `Runner`

### 4. Config gitlab-ci.yml

#### 4.1 Create .gilab-ci.yml

 `.gitlab-ci.yml` được viết theo định dạng YAML. Bạn có thể tìm hiểu thêm tại [đây](https://yaml.org/).

Như đã đề cập ở trên, `.gitlab-ci.yml` cho `Runner` biết những công việc cần phải làm. Mặc định, nó sẽ chạy một pipeline với 3 stage :
* build
* test
* deploy

Tuy nhiên, bạn không cần phải sử dụng cả 3 stage, các stage không được giao việc sẽ được bỏ qua. 

Dưới đây là ví dụ cấu hình cho một dự án Ruby on Rails :
~~~yaml
image: "ruby:2.5"

before_script:
  - apt-get update -qq && apt-get install -y -qq sqlite3 libsqlite3-dev nodejs
  - ruby -v
  - which ruby
  - gem install bundler --no-document
  - bundle install --jobs $(nproc)  "${FLAGS[@]}"

rspec:
  script:
    - bundle exec rspec

rubocop:
  script:
    - bundle exec rubocop
~~~

Đây là cấu hình đơn giản nhất có thể hoạt động với hầu hết các ứng dụng Ruby
1. Xác định 2 job `rspec` và `rubocop` với các lệnh thực thi khác nhau.
2. Trước mỗi job, các lệnh được xác định bởi **before_script** được thực thi.

Nếu bạn muốn kiểm tra xem file` .gitlab-ci.yml` có hợp lệ hay không, có một công cụ lint trong page **/-/ci/lint** trong namespace project. Bạn cũng có thể thấy nút `CI Lint` để đến trang này trong mục **CI/CD** -> **Pipelines** và **Pipelines** -> **Jobs** trong page Project.

#### 4.2. Push .gitlab-ci.yml to GitLab

Sau khi tạo file `.gitlab-ci.yml`, thêm nó vào git repos và push nó lên gitlab.
~~~
git add .gitlab-ci.yml
git commit -m "Add .gitlab-ci.yml"
git push origin master
~~~
Trở về `Pipeline` page, bạn sẽ thấy pipeline đang ở trạng thái `pending`. Bạn cũng có thể tới page `Commits`, để ý icon tạm dừng nhỏ bên cạnh commit SHA.

![](https://images.viblo.asia/70eec0ec-a424-41bc-915a-130746df5e25.png)

Click vào nó sẽ chuyển hướng đến page `Jobs` của commit tương ứng.

![](https://images.viblo.asia/a0bb93f8-f5ac-42df-9e79-3cc2e49495c2.png)

Thông báo **`stuck`** chỉ rằng chưa có `Gitlab runner` nào được cấu hình để thực thi Job này.

Bước tiếp theo mình sẽ cấu hình Runner để nó thực thi công việc này.

### 5. Gitlab Runner

Trong gitlab, các `Runner` thực thi các jobs được định nghĩa trong file `.gitlab-ci.yml`.

Một **Runner** có thể là một **máy ảo** (VM), một **VPS**, một **bare-metal**, một **docker container** hay thậm chí là một **cluster container**. Gitlab và Runners giao tiếp với nhau thông qua API, vì vậy yêu cầu duy nhất là máy chạy Runner có quyền truy cập Gitlab server.

Một `Runner` có thể xác định cụ thể cho một dự án nhất định hoặc phục vụ cho nhiều dự án trong Gitlab. Nếu nó phục vụ cho tất cả project thì được gọi là `Shared Runner`.

Để xác định xem **Runner** nào được chỉ định cho project của bạn. Vào **Settings** -> **CI/CD**. 

Để Runner hoạt động được, bạn cần thực hiện 2 bước :
* [Cài đặt Runner](https://docs.gitlab.com/runner/install/)
* [Cấu hình Runner](https://docs.gitlab.com/ee/ci/runners/README.html#registering-a-specific-runner)

Trong bài viết này, mình sẽ chỉ các bạn các bước để thiết lập trên môi trường Windows :

Đầu tiên, truy cập vào project Gitlab. Vào **Settings** -> **CI/CD** -> **Runners** (expand). 

![](https://images.viblo.asia/22c2585f-85d1-4594-a28f-c799c2631835.png)


1. Tạo một folder bất kì trên hệ thống ( ví dụ : `C:\GitLab-Runner`)
2. Tùy thuộc vào phiên bản hệ điều hành mà bạn download bản [x86](https://gitlab-runner-downloads.s3.amazonaws.com/latest/binaries/gitlab-runner-windows-386.exe) hay [amd64](https://gitlab-runner-downloads.s3.amazonaws.com/latest/binaries/gitlab-runner-windows-amd64.exe) . Sau đó di chuyển vào thư mục vừa tạo và đổi tên thành `gitlab-runner.exe`.
3. Bật CMD với quyền admin, cd tới thư mục vừa tạo và chạy lệnh :

    ~~~
    gitlab-runner register
    ~~~
4. Nhập gitlab url của bạn :
    ~~~
     Please enter the gitlab-ci coordinator URL (e.g. https://gitlab.com )
     https://gitlab.com
     ~~~
 5. Nhập Token nhận được từ giao diện trên.
    ~~~
    Please enter the gitlab-ci token for this runner
    xxx
    ~~~
6. Thêm mô tả cho Runner, phần này có thể thay đổi sau trong giao diện người dùng Gitlab :

    ~~~
     Please enter the gitlab-ci description for this runner
     [hostname] my-runner
     ~~~
 7. Thêm các thẻ Tags được liên kết với Runners, phần này có thể thay đổi sau :
     ~~~
      Please enter the gitlab-ci tags for this runner (comma separated):
      my-tag,another-tag
     ~~~
8. Chọn **Executor** cho Runner
    ~~~
     Please enter the executor: ssh, docker+machine, docker-ssh+machine, kubernetes, docker, parallels, virtualbox, docker-ssh, shell:
     docker
    ~~~
9. Nếu chọn Docker, bạn sẽ được yêu cầu thêm image mặc định nếu image không được định nghĩa trong `.gitlab-ci.yml`.

    ~~~
     Please enter the Docker image (eg. ruby:2.1):
     alpine:latest
     ~~~

OK, đến đây là đã thiết lập thành công Gitlab CI rồi đó :+1: 

### 6. Seeing status of your pipeline & jobs

Sau khi cấu hình thành công, bạn có thể thấy trạng thái của commit cuối cùng đã chuyển từ `pending` thành `running` or `success` or `failed`...

Bạn có thể theo dõi tất cả `pipeline` trong project :

![](https://images.viblo.asia/86a31214-0aac-447c-9927-7a13d46f02d4.png)

Hoặc có thể xem các jobs bằng cách truy cập **Pipelines** -> **Jobs**.

![](https://images.viblo.asia/8b74282b-54e8-418f-8076-ba5f93391062.png)

Bằng cách click vào trạng thái job, bạn có thể xem log của nó. Điều này rất quan trọng để chuẩn đoán tại sao job lại thất bại hay hành động khác với mong đợi.

![](https://images.viblo.asia/dab770ee-f62b-40b6-8c2a-4f44e9b9f0fd.png)

### Summary

Hi vọng bài biết giúp các bạn hiểu được các khái niệm cơ bản về CI/CD cũng như cách xây dựng hệ thống cơ bản với Gilab CI. Chúc các bạn coding vui vẻ :+1::+1:

Tham khảo : [https://docs.gitlab.com/](https://docs.gitlab.com/)