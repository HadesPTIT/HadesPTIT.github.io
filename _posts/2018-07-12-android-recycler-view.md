---
title: "[Android] RecyclerView"
date: 2018-07-12 21:15:03 +0700
categories: [Android]
tags: [android, recyclerview]
---

# Giới thiệu

RecycleView là một viewGroup mới được giới thiệu trong Android L (API 21).
<br/>
Là một viewGroup có chức năng tương tự như ListView nhưng linh hoạt hơn rất nhiều. ListView chỉ hỗ trợ bạn scroll các item trong listView theo chiều dọc mà không hỗ trợ scroll theo chiều ngang. RecyclerView support được tất cả những thứ đó và hơn thế nữa. Trong bài viết này tôi sẽ giới thiệu cho các bạn về một ViewGroup này.

# So sánh với ListView

So với ListView thì RecyclerView có những điểm mạnh mẽ vượt trội hơn như sau :
* Yêu cầu sử dụng ViewHolder Pattern trong Adapter : Khi tạo ra một adapter sử dụng với RecyclerView bắt buộc phải sử dụng ViewHolder để cải thiện performance.

> Mục đích sử dụng ViewHolder để tái sử dụng view nhằm tránh việc tạo view mới và findViewById quá nhiều

* ListView chỉ support cho chúng ta danh sách dạng scroll dọc. Nhưng với recyclerView cung cấp cho chúng ta **RecyclerView.LayoutManager** cho phép layout các item trong listview theo các kiểu khác nhau ( ngang, dọc, grid, staggered grid).
* Với listView việc xử lý animation cho các item không hề dễ dàng. Nhưng đối với RecyclerView hỗ trợ **ItemAnimator**  giúp chúng ta có thể xử lý animation khi add hay remove 1 item ra khỏi list một cách dễ dàng. Mặc định RecyclerView sử dụng DefaultItemAnimator.
* Với listView việc sử sử dụng divider không được linh hoạt nhưng với recyclerView có hỗ trợ **ItemDecoration** cho phép chúng ta draw divider 1 cách tùy ý.
* ListView có support các phương thức setOnItemClickListener và setOnLongItemClickListener nhưng RecyclerView chỉ support 1 phương thức đó là **onItemTouchListener**.

# Các thành phần khi sử dụng RecyclerView

Khi làm việc với RecyclerView ta phải làm việc với những thành phần sau :

## RecyclerView.Adapter

Cũng giống ListView đây là thành phần xử lý data collection (dữ liệu kiểu danh sách) và bind (gắn) những dữ liệu này lên các **Item** của RecyclerView.

Khi tạo custom Adapter chúng ta phải **override** lại 2 phương thức chính đó là :
* **onCreateViewHolder** : phương thức này dùng để tạo view mới cho RecyclerView. Nếu RecyclerView  đã cached lại View thì phương thức này sẽ không gọi.
* **onBindViewHolder** : phương thức này dùng để gắn data và view.

## LayoutManager

Là thành phần có chức năng sắp xếp các item trong RecyclerView. Các item scroll dọc hay ngang phụ thuộc chúng ta setLayoutManager cho RecyclerView.
Các classs con của layoutManager.
**`LinearLayoutManager`**: Hỗ trợ scroll các item theo chiều ngang hay chiều dọc.

**`GridLayoutManager`**: Layout các item trong RecyclerView dưới dạng Grid giống như khi chúng ta sử dụng GridView.

**`StaggerdGridLayoutManager`** : Layout cá item trong ListView dưới dạng Grid so le.

**`ItemAnimator`** : Là thành phần hỗ trợ animation khi chúng ta add hay remove một item ra khỏi RecyclerView. Tôi sẽ hướng dẫn chi tiết về ItemAnimator trong bài viết sau.

# Sử dụng RecyclerView trong Android

**1. Đầu tiên bạn cần import thư viện bên dưới :**

~~~
compile 'com.android.support:recyclerview-v7:23.0.+'
~~~

**2. Tạo model để chứa data**

>Movie.java

~~~java
package com.example.hades.testrecyclerview;

/**
 * Created by Hades on 12/8/2016.
 */

public class Movie {

	private String title, genre, year;

	public Movie() {
	}

	public Movie(String title, String genre, String year) {
		this.title = title;
		this.genre = genre;
		this.year = year;
	}

	public String getTitle() {
		return title;
	}

	public void setTitle(String name) {
		this.title = name;
	}

	public String getYear() {
		return year;
	}

	public void setYear(String year) {
		this.year = year;
	}

	public String getGenre() {
		return genre;
	}

	public void setGenre(String genre) {
		this.genre = genre;
	}
}
~~~
**3. Thêm recyclerView vào main_activity.xml**

>activity_main.xml

~~~xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/activity_main"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context="com.example.hades.testrecyclerview.MainActivity">

    <android.support.v7.widget.RecyclerView
        android:id="@+id/recycler_view"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:scrollbars="vertical"/>

</RelativeLayout>
~~~
**4. Tạo giao diện cho 1 row**
>movie_list_row.xml

~~~xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:focusable="true"
    android:paddingLeft="16dp"
    android:paddingRight="16dp"
    android:paddingTop="10dp"
    android:paddingBottom="10dp"
    android:clickable="true"
    android:background="?android:attr/selectableItemBackground">

    <TextView
        android:id="@+id/m_title"
        android:textColor="#222222"
        android:textSize="16dp"
        android:textStyle="bold"
        android:layout_alignParentTop="true"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />

    <TextView
        android:id="@+id/m_genre"
        android:layout_below="@+id/m_title"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />

    <TextView
        android:id="@+id/m_year"
        android:textColor="#999999"
        android:layout_width="wrap_content"
        android:layout_alignParentRight="true"
        android:layout_height="wrap_content" />

</RelativeLayout>
~~~
**5. Tạo Custom Adapter và bind dữ liệu cho từng row trong Adapter**

>MoviesAdapter.java

~~~java
package com.example.hades.testrecyclerview;

import android.content.Context;
import android.support.v7.widget.RecyclerView;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.TextView;
import android.widget.Toast;

import java.util.List;
/**
 * Created by Hades on 12/8/2016.
 */
public class MoviesAdapter extends RecyclerView.Adapter<MoviesAdapter.MyViewHolder> {

	private List<Movie> movieList;
	private LayoutInflater mLayoutInflater;
	private Context mContext;

	public MoviesAdapter(Context context,List<Movie> movieList) {
		this.movieList = movieList;
		this.mContext = context;
		mLayoutInflater = LayoutInflater.from(context);
	}

	public class MyViewHolder extends RecyclerView.ViewHolder {

		public TextView title, year, genre;

		public MyViewHolder(View itemView) {
			super(itemView);
			title = (TextView) itemView.findViewById(R.id.m_title);
			genre = (TextView) itemView.findViewById(R.id.m_genre);
			year = (TextView) itemView.findViewById(R.id.m_year);

			itemView.setOnClickListener(new View.OnClickListener() {
				@Override
				public void onClick(View v) {
					Toast.makeText(mContext,title.getText(),Toast.LENGTH_SHORT).show();
				}
			});
            itemView.setOnLongClickListener(new View.OnLongClickListener() {
				@Override
				public boolean onLongClick(View v) {
					Toast.makeText(mContext,"Long item clicked " + title.getText() + ,Toast.LENGTH_SHORT).show();
					return true;
				}
			});
		}
	}


	@Override
	public MyViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
		View item = mLayoutInflater.inflate(R.layout.movie_list_row,parent,false);
		return new MyViewHolder(item);
	}

	@Override
	public void onBindViewHolder(MoviesAdapter.MyViewHolder holder, int position) {
		Movie movie = movieList.get(position);
		holder.title.setText(movie.getTitle());
		holder.genre.setText(movie.getGenre());
		holder.year.setText(movie.getYear());
	}

	@Override
	public int getItemCount() {
		return movieList.size();
	}

}

~~~
Chúng ta có thể select item trong RecyclerView bằng cách setOnClickListenner cho itemView trong constructor của ViewHolder.

**6. Setup RecyclerView trong MainActivity.java**

Không giống như ListView chúng ta chỉ cần setAdapter cho ListView là đủ nhưng đối với RecyclerView chúng ta phải set 2 thành phần bắt buộc dưới đấy :
* **setAdapter** cho RecyclerView.
* **setLayoutManager** chỉ ra các item được hiển thị như theo cách nào.

> MainActivity.java

~~~java
package com.example.hades.testrecyclerview;

import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.support.v7.widget.DefaultItemAnimator;
import android.support.v7.widget.LinearLayoutManager;
import android.support.v7.widget.RecyclerView;

import java.util.ArrayList;
import java.util.List;

public class MainActivity extends AppCompatActivity {

	private List<Movie> mListMovie = new ArrayList<>();
	private RecyclerView recyclerView;
	private MoviesAdapter mAdapter;

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
		prepareMovieData();
		recyclerView = (RecyclerView) findViewById(R.id.recycler_view);
		mAdapter = new MoviesAdapter(this,mListMovie);

		RecyclerView.LayoutManager mLayoutManager = new LinearLayoutManager(getApplicationContext());
		recyclerView.setLayoutManager(mLayoutManager);
		recyclerView.setItemAnimator(new DefaultItemAnimator());
		recyclerView.setAdapter(mAdapter);

	}

	private void prepareMovieData() {
		Movie movie = new Movie("Mad Max: Fury Road", "Action & Adventure", "2015");
		mListMovie.add(movie);

		movie = new Movie("Inside Out", "Animation, Kids & Family", "2015");
		mListMovie.add(movie);

		movie = new Movie("Star Wars: Episode VII - The Force Awakens", "Action", "2015");
		mListMovie.add(movie);

		movie = new Movie("Shaun the Sheep", "Animation", "2015");
		mListMovie.add(movie);

		movie = new Movie("The Martian", "Science Fiction & Fantasy", "2015");
		mListMovie.add(movie);

		movie = new Movie("Mission: Impossible Rogue Nation", "Action", "2015");
		mListMovie.add(movie);

		movie = new Movie("Up", "Animation", "2009");
		mListMovie.add(movie);

		movie = new Movie("Star Trek", "Science Fiction", "2009");
		mListMovie.add(movie);

		movie = new Movie("The LEGO Movie", "Animation", "2014");
		mListMovie.add(movie);

		movie = new Movie("Iron Man", "Action & Adventure", "2008");
		mListMovie.add(movie);

		movie = new Movie("Aliens", "Science Fiction", "1986");
		mListMovie.add(movie);
	}
}

~~~

Dưới đây là video demo : [https://www.youtube.com/watch?v=XrFG4BMIFpo](https://www.youtube.com/watch?v=XrFG4BMIFpo)