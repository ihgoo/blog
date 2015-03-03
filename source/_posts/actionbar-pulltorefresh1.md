title: Actionbar-PullTorRefresh的使用
date: 2014-10-09 18:08:18
categories: Android
---
这次来玩一玩Actionbar-PullTorRefresh，[--地址点我--](https://github.com/chrisbanes/ActionBar-PullToRefresh)

在终端运行git命令讲工程pull下来：

``git pull git@github.com:chrisbanes/ActionBar-PullToRefresh.git``
<!--more-->
- pull下来观察了下，是Gradle的，搜了下，eclipse上需要装插件麻烦，直接复制源码吧。

- 先进library目录看看，ActionBar-PullToRefresh-master\library\src\main，里面是源码，把这个目录复制进对应的项目目录中去。

- 再进project.properties文件把target修改成高版本的，~~3.0以下根本木有actionbar嘛。~~哦，错了，3.0一下要支持的话需要导入actionbarsherlock这个类库。

- Clean一下项目还是报错，是在
`/main/src/uk/co/senab/actionbarpulltorefresh/library/DefaultHeaderTransformer.java中`
 SmoothProgressBar这个类没找到引起的错误，去github上又搜了一下。[--地址点我--](https://github.com/castorflex/SmoothProgressBar)下载下来 还是gradle的，和上一个一样。手动复制移动一下。修复好了改下res文件名称。导一下R文件的包。
 
- 新建一个工程叫PullToRefreshActionbarDemo，引用上面手动改好的库。贴上我的Demo的代码。

```
public class MainActivity extends ActionBarActivity implements OnRefreshListener {

	private PullToRefreshLayout mPullToRefreshLayout;
	private ListView mListView;
	private static String[] ITEMS = { "Abbaye de Belloc", "Abbaye du Mont des Cats", "Abertam", "Abondance", "Ackawi", "Acorn", "Adelost", "Affidelice au Chablis", "Afuega'l Pitu", "Airag", "Airedale", "Aisy Cendre", "Allgauer Emmentaler", "Abbaye de Belloc", "Abbaye du Mont des Cats", "Abertam", "Abondance", "Ackawi", "Acorn", "Adelost", "Affidelice au Chablis", "Afuega'l Pitu", "Airag", "Airedale", "Aisy Cendre", "Allgauer Emmentaler" };

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_ab_ptr);
		// Now find the PullToRefreshLayout to setup
		mPullToRefreshLayout = (PullToRefreshLayout) findViewById(R.id.ptr_layout);

		// Now setup the PullToRefreshLayout
		ActionBarPullToRefresh.from(this)
		// Mark All Children as pullable
				.allChildrenArePullable()
				// Set a OnRefreshListener
				.listener(this)
				// Finally commit the setup to our PullToRefreshLayout
				.setup(mPullToRefreshLayout);

		mListView = (ListView) findViewById(R.id.listview);
		mListView.setAdapter(new ArrayAdapter<String>(this, android.R.layout.simple_list_item_1, ITEMS));
	}

	@Override
	public void onRefreshStarted(View view) {
		// Hide the list
		// setListShown(false);

		/**
		 * Simulate Refresh with 4 seconds sleep
		 */
		new AsyncTask<Void, Void, Void>() {

			@Override
			protected Void doInBackground(Void... params) {
				try {
					Thread.sleep(5000);
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
				return null;
			}

			@Override
			protected void onPostExecute(Void result) {
				super.onPostExecute(result);

				// Notify PullToRefreshLayout that the refresh has finished
				mPullToRefreshLayout.setRefreshComplete();
			}
		}.execute();
	}

}
```

这是activity_ab_ptr.xml

```
<?xml version="1.0" encoding="utf-8"?>  
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"  
    android:layout_width="match_parent"  
    android:layout_height="match_parent" >  
      
    <uk.co.senab.actionbarpulltorefresh.library.PullToRefreshLayout
        xmlns:android="http://schemas.android.com/apk/res/android"  
        android:id="@+id/ptr_layout"  
        android:layout_width="match_parent"  
        android:layout_height="match_parent">  
      
        <!-- Your content, here we're using a ScrollView -->  
      
        <ListView  
            android:id="@+id/listview"  
            android:layout_width="match_parent"  
            android:layout_height="match_parent">  
      
        </ListView>  
      
    </uk.co.senab.actionbarpulltorefresh.library.PullToRefreshLayout>  
</RelativeLayout>  
```

运行起来效果图：

![](http://ihgoo.qiniudn.com/actionbar-pulltorefresh1%2Fpulltorefulshbar.gif)


###已经打成压缩包了，地址下载
[---嘤嘤嘤，点我可下载Demo和修改好的类库---](http://ihgoo.qiniudn.com/actionbar-pulltorefresh1%2FactionbarpullDemo.zip)


