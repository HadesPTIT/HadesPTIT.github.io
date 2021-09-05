---
title: "[Android] Clean architecture"
date: 2020-02-01 21:15:03 +0700
categories: [Android]
tags: [android, clean architecture, kotlin, mvvm]
---

# Giới thiệu

Việc xây dựng 1 kiến trúc nền tốt thật sự quan trọng cho ứng dụng để scale lên một cách đơn giản.
<br/>Trước đây thì mình cực kì không thích thằng clean này đơn giản vì nó phải code nhiều quá. Nhưng khi làm việc với các dự án cỡ vừa và lơn thì bạn sẽ thấy nó thật sự hữu ịch Trong bài viết này, mình sẽ tìm hiểu và "thích" nó nhé.

## Clean architecture ?

Clean architecture hay còn gọi là kiến trúc sạch được đề xuất vào năm 2012 bởi Robert C. Martin. Clean Architecture dựa trên các khái niệm, quy tắc, và mô hình nổi tiếng, giải thích làm thế nào để kết hợp chúng với nhau, để đề xuất cách xây dựng các ứng dụng chuẩn. Hãy đọc bài viết [này](http://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html) để hiểu hơn về, nếu không bạn có thể tẩu hỏa nhập ma đấy.

## Why ?

Tại sao lại tiếp cận theo hướng kiến trúc sạch ?

1. Tách biệt hoàn toàn các tầng (layer) với nhau với nhiệm vụ riêng biệt giúp cho việc chỉnh sửa dễ dàng hơn.
2. Tăng tính trừu tượng
3. Tránh ràng buộc
4. Dễ dàng kiểm thử

## Layers ?

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/nir3hvc7so_1_a5UQUjgYu5SZAbmkNELI_A.png)

* **Domain Layer** : chịu trách nhiệm thực thi các logic độc lập với các layer khác. Layer này chỉ bao gồm các mã kotlin thuần túy, không bao gồm sự phụ nào của Android Framework ở đây.
* **Data Layer** : chịu trách nhiệm phân chia data cần thiết cho app bằng cách implement các interface được phát ra từ tầng domain.
* **Presenter Layer** : chứa cả Domain và Data layer, chịu trách nhiệm thực thi các logic giao diện.

## What is Domain layer ?

- Domain là tầng chung nhất trong 3 tầng. Nó sẽ kết nối tầng presenter với tầng data. Đây là tầng nơi các business logic của ứng dụng được thực thi.
![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/rdtol6i413_2_m06XFPa5OTvOF6zGPC7Q0w.png )

### UseCases

Đóng gói và thực hiện các case của hệ thống theo đúng cái tên của nó. Mỗi usecase thường sẽ mô tả một chức năng riêng biệt.

~~~java
class GetNewsUseCase(private val transformer: FlowableRxTransformer<NewsSourcesEntity>,
                     private val repositories: NewsRepository): BaseFlowableUseCase<NewsSourcesEntity>(transformer){

    override fun createFlowable(data: Map<String, Any>?): Flowable<NewsSourcesEntity> {
        return repositories.getNews()
    }

    fun getNews(): Flowable<NewsSourcesEntity>{
        val data = HashMap<String, String>()
        return single(data)
    }
}
~~~

### Repositories

Repo là nơi lưu trữ, chỉ định các chức năng được yêu cầu tương ứng với các usecase chỉ định.

~~~java
interface NewsRepository {

    fun getNews(): Flowable<NewsSourcesEntity>
    fun getLocalNews(): Flowable<NewsSourcesEntity>
    fun getRemoteNews(): Flowable<NewsSourcesEntity>
}
~~~

## What is Data Layer ?

Tầng data có trách nhiệm cung cấp dữ liệu được yêu cầu bởi ứng dụng. Tầng data phải được thiết kế sao cho dữ liệu có thể được sử dụng lại ở bất kì ứng dụng nào mà không cần sửa đổi logic của tầng Presentation.

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/xab5yuqb1a_3_KbdhwDpsxspHEz7QInpbhA.png)

* **api** cung cấp các network interface.
* **db** cung cấp các local database interface. 
* **repository** nhiệm vụ implement dữ liệu từ local, remote.

~~~java
class NewsRepositoryImpl(private val remote: NewsRemoteImpl,
                         private val cache: NewsCacheImpl) : NewsRepository {

    override fun getLocalNews(): Flowable<NewsSourcesEntity> {
        return cache.getNews()
    }

    override fun getRemoteNews(): Flowable<NewsSourcesEntity> {
        return remote.getNews()
    }

    override fun getNews(): Flowable<NewsSourcesEntity> {
        val updateNewsFlowable = remote.getNews()
        return cache.getNews()
                .mergeWith(updateNewsFlowable.doOnNext{
                    remoteNews -> cache.saveArticles(remoteNews)
                })
    }
}
~~~
Repositories hoạt động như 1 điểm truy cập duy nhất tới data layer.

## What is the Presentation layer?

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/14jkrlp3dp_1_4UH3LeLcGg8tjp1BmPm1jw.png)

Presentation layer cung cấp giao diện người dùng của ứng dụng.<br/>
Đây là layer `câm` chỉ thực hiện lệnh mà không bao gồm logic trong đó, kết nối mọi thứ với nhau.<br/>
Presentation có thể triển khai theo các pattern khác nhau như MVC, MVP, MVVM, MVI...

Folder `di` cung cấp tất cả các phụ thuộc từ khi bắt đàu khởi động ứng dụng cho đến khi kết thúc. Trong android, DI có thể implement bằng dagger, kodein, koin hay tự xây dựng theo service locator pattern.

### Tại sao nên dùng ViewModels ?

Trong tài liệu mô tả android có đề cập đến [ViewModel](https://developer.android.com/topic/libraries/architecture/viewmodel) như sau:

> Store and manage UI-related data in a lifecycle conscious way. It allows data to survive configuration changes such as screen rotations.

~~~java
class NewsViewModel(private val getNewsUseCase: GetNewsUseCase,
                    private val mapper: Mapper<NewsSourcesEntity, NewsSources>) : BaseViewModel() {

    companion object {
        private val TAG = "viewmodel"
    }

    var mNews = MutableLiveData<Data<NewsSources>>()

    fun fetchNews() {
        val disposable = getNewsUseCase.getNews()
                .flatMap { mapper.Flowable(it) }
                .subscribe({ response ->
                    Log.d(TAG, "On Next Called")
                    mNews.value = Data(responseType = Status.SUCCESSFUL, data = response)
                }, { error ->
                    Log.d(TAG, "On Error Called")
                    mNews.value = Data(responseType = Status.ERROR, error = Error(error.message))
                }, {
                    Log.d(TAG, "On Complete Called")
                })

        addDisposable(disposable)
    }

    fun getNewsLiveData() = mNews
}
~~~

* Việc chuyển đổi data từ usecase sang UI thông qua ViewModel có thể giữ lại dữ liệu ngay cả khi có configuration change.
* ViewModel được tích hợp sẵn life aware components

## How all the layers are connected?

Sau khi đã hiểu nhiệm vụ của các layer, giờ thì mình sẽ ghép nối lại xem các layer connect với nhau như nào nhé.
* Mỗi layer có các `entities` riêng được xác định riêng.
* Mapper được sử dụng để chuyển đổi các `entities` từ layer này qua layer khác.
* Việc xây dựng các entities riêng biệt cho các layer giúp nó trở nên hoàn toàn độc lập và chỉ có dữ liệu cần thiết mới chuyển sang layer tiếp theo.

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/zhgz22cddn_5_a-AUcEVdyRJhIepo9JyJBw.png)

## Conclusion

Hi vọng qua bài viết các bạn có thể hiểu được cơ bản về clean architecture cũng như flow của nó. 
Mọi người có thể xem qua sample tại [đây](https://github.com/HadesPTIT/news-clean-architecture). Happy coding !!!

## References

* [Kotlin clean architecture](https://proandroiddev.com/kotlin-clean-architecture-1ad42fcd97fa)
* [Clean coder blog](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html) 

