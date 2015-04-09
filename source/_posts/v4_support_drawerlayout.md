title: 抽屉效果DrawerLayout~
date: 2015-01-03 22:33:51
categories: Android
---
####创建一个导航抽屉
关于抽屉的组件层出不穷，有Slidingmenu，MenuDrawer，FoldingNavigationDrawer(滑动并以折叠方式打开菜单)等等，谷歌也是出了几种抽屉：
- [SlidingDrawer](http://developer.android.com/reference/android/widget/SlidingDrawer.html)（在api17的时候过时了）
- [DrawerLayout](http://developer.android.com/reference/android/support/v4/widget/DrawerLayout.html) (V4 support包中的控件)<!--more-->



	在决定使用抽屉之前，应该明白导航抽屉的设计原则

这其实是一个梗，在产品经理都不知道抽屉是什么的情况下居然也能在axure上设计出来抽屉出来，只有呵呵两字能表示我此刻的心情了。
####创建抽屉的布局
DrawerLayout有两个子View，一个FrameLayout作为主内容区域，另一部分用ListView作为导航菜单。Layout如下所示：

    <android.support.v4.widget.DrawerLayout
        xmlns:android="http://schemas.android.com/apk/res/android"
        android:id="@+id/drawer_layout"
        android:layout_width="match_parent"
        android:layout_height="match_parent">
        <!-- The main content view -->
        <FrameLayout
            android:id="@+id/content_frame"
            android:layout_width="match_parent"
            android:layout_height="match_parent" />
        <!-- The navigation drawer -->
        <ListView android:id="@+id/left_drawer"
            android:layout_width="240dp"
            android:layout_height="match_parent"
            android:layout_gravity="start"
            android:choiceMode="singleChoice"
            android:divider="@android:color/transparent"
            android:dividerHeight="0dp"
            android:background="#111"/>
    </android.support.v4.widget.DrawerLayout>

这里展示了DrawerLayout一些要注意的地方

- 主要视图(上面的FrameLayout)，必须作为DrawerLayout第一个孩子，因为在xml中是在布局的最上面的。
- 抽屉视图，可以设置layout_gravity：为start或者end(在左边或者右边)。

####初始化抽屉列表
在这儿，activity中，第一件事情就是初始化导航抽屉的item：

        public class MainActivity extends Activity {
            private String[] mPlanetTitles;
            private DrawerLayout mDrawerLayout;
            private ListView mDrawerList;
            ...

            @Override
            public void onCreate(Bundle savedInstanceState) {
                super.onCreate(savedInstanceState);
                setContentView(R.layout.activity_main);

                mPlanetTitles = getResources().getStringArray(R.array.planets_array);
                mDrawerLayout = (DrawerLayout) findViewById(R.id.drawer_layout);
                mDrawerList = (ListView) findViewById(R.id.left_drawer);

                // Set the adapter for the list view
                mDrawerList.setAdapter(new ArrayAdapter<String>(this,
                        R.layout.drawer_list_item, mPlanetTitles));
                // Set the list's click listener
                mDrawerList.setOnItemClickListener(new DrawerItemClickListener());

                ...
            }
        }

####处理导航的点击事件

    private class DrawerItemClickListener implements ListView.OnItemClickListener {
        @Override
        public void onItemClick(AdapterView parent, View view, int position, long id) {
            selectItem(position);
        }
    }

    /** Swaps fragments in the main content view */
    private void selectItem(int position) {
        // Create a new fragment and specify the planet to show based on position
        Fragment fragment = new PlanetFragment();
        Bundle args = new Bundle();
        args.putInt(PlanetFragment.ARG_PLANET_NUMBER, position);
        fragment.setArguments(args);

        // Insert the fragment by replacing any existing fragment
        FragmentManager fragmentManager = getFragmentManager();
        fragmentManager.beginTransaction()
                       .replace(R.id.content_frame, fragment)
                       .commit();

        // Highlight the selected item, update the title, and close the drawer
        mDrawerList.setItemChecked(position, true);
        setTitle(mPlanetTitles[position]);
        mDrawerLayout.closeDrawer(mDrawerList);
    }

    @Override
    public void setTitle(CharSequence title) {
        mTitle = title;
        getActionBar().setTitle(mTitle);
    }

####监听抽屉的开关事件
可以调用setDrawerListener()来监听抽屉的开和关。

####TRY IT OUT
[Download the sample app](http://developer.android.com/shareables/training/NavigationDrawer.zip)