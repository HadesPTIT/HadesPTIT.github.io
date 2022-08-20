---
title: "[Android] Architecture Components : Paging Library"
date: 2019-05-20 21:15:03 +0700
categories: [Android]
tags: [android, paging]
---

## Introduction

Hiện nay việc tải và hiển thị dữ liệu dưới dạng danh sách rất phổ biến và hầu như ứng dụng nào cũng có. Với những tập dữ liệu đầu vào vừa và nhỏ ta thường tải xong sau đó mới hiển thị lên.

Tuy nhiên nếu tập dữ liệu đầu vào rất lớn thì sao ? Nếu không xử lý cẩn thận, việc request dữ liệu một cách thừa thãi chỉ tổ lãng phí bộ nhớ, tốn pin và băng thông thậm chí gây ra đơ lag nếu phải chờ quá lâu cho việc truy vấn CSDL. Paging Library ra đời sẽ giúp bạn xử lý những vấn đề này.

Trong phần này mình sẽ trình bày qua concept của Paging cũng như cách thức hoạt động của nó.

## Library architecture
### 1. PagedList

Đây là thành phần quan chính Paging, nó có nhiệm vụ tải các đoạn dữ liệu hoặc trang của ứng dụng của bạn.  Khi cần thêm dữ liệu, nó sẽ được phân trang vào đối tượng PagedList hiện có. Nếu bất cứ dữ liệu dữ liệu được tải nào thay đổi, một instance mới của PagedList sẽ được emit tới observable data holder từ một đối tượng dựa trên LiveData hay Rx. 

Ví dụ bên dưới cho thấy cách cấu hình ViewModel để tải và hiển thị dữ liệu sử dụng LiveData với PagedList :

~~~java
public class ConcertViewModel extends ViewModel {
    private ConcertDao concertDao;
    public final LiveData<PagedList<Concert>> concertList;

    // Creates a PagedList object with 50 items per page.
    public ConcertViewModel(ConcertDao concertDao) {
        this.concertDao = concertDao;
        concertList = new LivePagedListBuilder<>(
                concertDao.concertsByDate(), 50).build();
    }
}
~~~
~~~kotlin
class ConcertViewModel(concertDao: ConcertDao) : ViewModel() {
    val concertList: LiveData<PagedList<Concert>> =
            concertDao.concertsByDate().toLiveData(pageSize = 50)
}
~~~

### 2. Data
Mỗi instance của PagedList sẽ tải một snapshot cập nhật dữ liệu từ đối tượng DataSource tương ứng. Dữ liệu sẽ được chuyển từ phía backend or database vào PagedList.

Ví dụ bên dưới sử dụng Room để lấy dữ liệu, nhưng nếu bạn muốn lưu trữ dữ liệu của mình bằng cách khác, bạn cũng có thể cung cấp một data source factory riêng.
~~~java
@Dao
public interface ConcertDao {
    // The Integer type parameter tells Room to use a PositionalDataSource object
    @Query("SELECT * FROM concerts ORDER BY date DESC")
    DataSource.Factory<Integer, Concert> concertsByDate();
}
~~~
~~~kotlin
@Dao
interface ConcertDao {
    // The Int type parameter tells Room to use a PositionalDataSource object.
    @Query("SELECT * FROM concerts ORDER BY date DESC")
    fun concertsByDate(): DataSource.Factory<Int, Concert>
}
~~~

Để tìm hiểu thêm về cách tải data vào PagedList, hãy xem guide sau : [Load paged data](https://developer.android.com/topic/libraries/architecture/paging/data)

### 3. UI
PagedList class hoạt động với PagedListAdapter để tải các items vào RecyclerView. Những lớp này hoạt đồng cùng với nhau để nạp và hiển thị nội dung mà nó đã tải, tìm và tải trước nội dung không thấy được, xử lý animation khi thay đổi.

Để tìm hiểu thêm, hãy đọc guide bên sau : [Display paged lists](https://developer.android.com/topic/libraries/architecture/paging/ui)


## Support diferent data architecture

Paging Library hỗ trợ những kiểu kiến trúc dữ liệu sau:
* Chỉ lấy dữ liệu từ backend server.
* Chỉ được lưu trữ trong database thiết bị.
* Kết hợp của các nguồn khác nhau, sử dụng database thiết bị làm cache.

![](https://images.viblo.asia/56de9a04-c112-4221-8c65-2dc958535fca.PNG)

Vậy, với mỗi case trên thì nên handle như nào cho hợp lý ?

### 1. Network Only
Với dữ liệu từ backend server, nên sử dụng Retrofit để load dữ liệu vào DataSource.

### 2. Database Only
Sử dụng recyclerview với Room. Theo cách đó, bất cứ khi nào data được insert hay modified trong database, những thay đổi đó sẽ tự động cập nhật lên RecyclerView.

### 3. Network & Database

Observable database, lắng nghe khi nào database hết dữ liệu bằng cách dùng PagedList.BoundaryCallback. Sau đó bạn có thể tải thêm dữ liệu từ network và insert vào database.

## Handle network errors

Khi sử dụng mạng để tải hay phân trang dữ liệu sử dụng Paging, một điều quan trọng là bạn không nên xử lý bằng cách kiểm tra xem mạng có khả dụng hay không tại mọi thời điểm. Bởi có nhiều kết nối không liên tục hay không ổn định.

Thay vào đó, bạn nên xử lý chúng bằng cách kiểm tra mỗi request lỗi và cơ chế cho phép retry khi mạng không khả dụng.

## Conclusion

Cảm ơn các bạn đã đọc bài viết. Happy coding ^^