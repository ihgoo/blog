title: 上拉加载下拉刷新，xlistview详解（一）
categories: Android
date: 2014-10-08 12:52:58
---
刚才朋友问我XListView是怎么用的，我从github上pull一个看了一下，把使用方法记录到blog上。

XListView和pull-to-rerfesh差不太多，算是精简版吧。有一个ListView可以实现上拉加载和下拉刷新功能。

##Xlistview 快速入门<!--more-->

Xlistview对ListView进行了封装，用法和ListView没有太大的区别，下面看Demo

这是main.xml

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="fill_parent"
    android:layout_height="fill_parent"
    android:orientation="vertical" android:background="#f0f0f0">

    <TextView
        android:layout_width="fill_parent"
        android:layout_height="wrap_content"
        android:text="@string/hello" />

    <me.maxwin.view.XListView
        android:id="@+id/xListView"
        android:layout_width="fill_parent"
        android:layout_height="fill_parent"
        android:cacheColorHint="#00000000">
    </me.maxwin.view.XListView>

</LinearLayout>
```

声明了一个XListView在布局文件中，再来定义一个XListView条目里面的布局list_item.xml

```
<?xml version="1.0" encoding="utf-8"?>
<TextView xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="fill_parent"
    android:layout_height="fill_parent" android:id="@+id/list_item_textview" android:textSize="16sp" android:textColor="#000" android:padding="5dp">
</TextView>
```

下面是MainActivity

```
public class MainActivity extends Activity implements IXListViewListener {
	private XListView mListView;
	private ArrayAdapter<String> mAdapter;
	private ArrayList<String> items = new ArrayList<String>();
	private Handler mHandler;
	private int start = 0;
	private static int refreshCnt = 0;
	@Override
	public void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.main);
		geneItems();
		// 和listview一样，先findviewbyid找到它
		mListView = (XListView) findViewById(R.id.xListView);
		// 设置是否可以加载
		mListView.setPullLoadEnable(true);
		mAdapter = new ArrayAdapter<String>(this, R.layout.list_item, items);
		mListView.setAdapter(mAdapter);
		// 监听下拉刷新和上拉加载 要实现 IXListViewListener这个接口
		mListView.setXListViewListener(this);
		mHandler = new Handler();
	}

	private void geneItems() {
		for (int i = 0; i != 20; ++i) {
			items.add("refresh cnt " + (++start));
		}
	}

	private void onLoad() {
		mListView.stopRefresh();
		mListView.stopLoadMore();
		mListView.setRefreshTime("刚刚");
	}
	
	@Override
	public void onRefresh() {
		mHandler.postDelayed(new Runnable() {
			@Override
			public void run() {
				start = ++refreshCnt;
				items.clear();
				geneItems();
				// mAdapter.notifyDataSetChanged();
				mAdapter = new ArrayAdapter<String>(MainActivity.this, R.layout.list_item, items);
				mListView.setAdapter(mAdapter);
				onLoad();
			}
		}, 2000);
	}

	@Override
	public void onLoadMore() {
		mHandler.postDelayed(new Runnable() {
			@Override
			public void run() {
				geneItems();
				mAdapter.notifyDataSetChanged();
				onLoad();
			}
		}, 2000);
	}

}
```

好了，下拉刷新上拉加载已经写好了，上面代码中，在实现了IXListViewListener后，会有两个回调事件，一个是上拉回调，一个是下拉回调。