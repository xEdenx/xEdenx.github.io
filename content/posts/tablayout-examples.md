---
title: TabLayout实现底部自定义Tab布局2
description: TabLayout 文本、图标与自定义布局的实现示例。
publishDate: 2015-10-22 11:50:18
tags:
  - tablayout
  - tab
  - android
---

展示三种类型的 TabLayout，Text 类型，Icon 类型，和自定义的 Tab。

### TabLayout－Text 预览

![TabLayout－Text](http://i.imgur.com/goeTYhB.gif)

前提条件得满足。

```txt
compile 'com.android.support:design:22.2.1'
compile 'com.android.support:appcompat-v7:22.2.1'
```

### TabLayout

新建一个 xml 文件，声明 TabLayout & ViewPager。

```xml

<android.support.design.widget.TabLayout
   android:id="@+id/layout_tab"
   android:layout_height="wrap_content"
   android:layout_width="match_parent"
   style="@style/MyCustomTabLayout"
   />
<android.support.v4.view.ViewPager
   android:id="@+id/viewpager"
   android:layout_height="match_parent"
   android:layout_width="match_parent"
   />

```

###TabLayout Style

MyCustomTabLayout是自定义的 TabLayout 样式。各个选项的功能很容易从字面上理解。可以设置文字选择之后的颜色，指示线的颜色等。

```xml

<style name="MyCustomTabLayout" parent="Widget.Design.TabLayout">
   <item name="tabIndicatorColor">?attr/colorAccent</item>
   <item name="tabIndicatorHeight">2dp</item>
   <item name="tabPaddingStart">12dp</item>
   <item name="tabPaddingEnd">12dp</item>
   <item name="tabBackground">?android:colorPrimary</item>
   <item name="tabTextAppearance">@style/MyCustomTabTextAppearance</item>
   <item name="tabSelectedTextColor">?android:colorPrimary</item>
</style>
<style name="MyCustomTabTextAppearance" parent="TextAppearance.Design.Tab">
   <item name="android:textSize">14sp</item>
   <item name="android:textColor">?android:textColorSecondary</item>
   <item name="textAllCaps">false</item> <!--文本大写-->
</style>

```

### ViewPager

ViewPager 里面包含三个 Fragment 用来展示内容。

### Fragment

每个 Fragment 里面只有一个 TextView。

```java

public class ViewpaperFragment extends Fragment  {

    private TextView tvViewpager;

    private static final String INDEX = "index";

    private int mPage;

    public static ViewpaperFragment newInstance(int page) {
        Bundle args = new Bundle();
        args.putInt(INDEX, page);
        ViewpaperFragment fragment = new ViewpaperFragment();
        fragment.setArguments(args);
        return fragment;
    }

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        mPage = getArguments().getInt(INDEX);
    }

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                             Bundle savedInstanceState) {
        return inflater.inflate(R.layout.viewpaper, null);
    }

    @Override
    public void onViewCreated(View view, Bundle savedInstanceState) {
        super.onViewCreated(view, savedInstanceState);

        tvViewpager = (TextView) view.findViewById(R.id.tv_viewpager);
        tvViewpager.setText("Tab " + mPage);
    }

}

```

## FragmentPagerAdapter

在 FragmentPagerAdapter 里面添加 ViewPager 的页面，并添加 TabLayout 的 Tab 选项。

```java

public class ViewPagerAdapter extends FragmentPagerAdapter {

    private Context mContext;

    private String[] tabTitle = {"TAB1","TAB2","TAB3"};

    private int[] tabIcon = {
            R.drawable.selector_icon1,
            R.drawable.selector_icon2,
            R.drawable.selector_icon3};

    public ViewPagerAdapter(FragmentManager fm,Context context) {
        super(fm);
        this.mContext = context;
    }

    @Override
    public Fragment getItem(int position) {
        return ViewpaperFragment.newInstance(position+1);
    }

    @Override
    public int getCount() {
        return 3;
    }

    @Override
    public CharSequence getPageTitle(int position) {
        //return super.getPageTitle(position);
        return tabTitle[position];
    }
}

```

其实只要注意 getPageTitle 方法就好了，这里面设置了 Tab 的显示内容。

设置 TabLayout

```java

adapter = new ViewPagerAdapter(
      getActivity().getSupportFragmentManager(), getActivity());
mViewPager.setAdapter(adapter);
mTabLayout.setupWithViewPager(mViewPager);

```

setupWithViewPager 之后带文本的 Tab 就会跟随 ViewPager 的滑动而改变。

### TabLayout－Icon 预览

![TabLayout－Icon](http://i.imgur.com/2tOb3NP.gif)

Icon

使用图标类型的 Tab，只要更改 getPageTitle 方法。

```java

@Override
public CharSequence getPageTitle(int position) {

   Drawable image = mContext.getResources().getDrawable(tabIcon[position]);
   image.setBounds(0, 0, image.getIntrinsicWidth(), image.getIntrinsicHeight());
   SpannableString sb = new SpannableString(" ");
   ImageSpan imageSpan = new ImageSpan(image, ImageSpan.ALIGN_BOTTOM);
   sb.setSpan(imageSpan, 0, 1, Spannable.SPAN_EXCLUSIVE_EXCLUSIVE);
   return sb;
}

```

### TabLayout－自定义预览

![自定义预览](http://i.imgur.com/MiIxzfi.gif)

自定义的 Tab 布局

布局很常见，就是上面图标，下面文字。

```xml

<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical" android:layout_width="match_parent"
    android:gravity="center"
    android:layout_height="match_parent">
    <ImageView
        android:id="@+id/tab_icon"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:padding="2dp"
        android:src="@drawable/ic_action_filter_1"
        />
    <TextView
        android:id="@+id/tab_text"
        android:text="TAB 1"
        android:textSize="12sp"
        android:textColor="@color/selector_text"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content" />
</LinearLayout>

```

这里有个系统的 id，ImageView 的是 icon，TextView 的是 text1，直接使用这个 id 的话，直接可以使用 tab.setText() 和 tab.setIcon() 来设置图标和文字。

设置图标文字选中颜色

像往常一样，选中状态写的 seletor 就可以，类似这样。文字的更改一下颜色。

```xml

<?xml version="1.0" encoding="utf-8"?>
<selector xmlns:android="http://schemas.android.com/apk/res/android">
    <item
        android:left="2dp" android:right="4dp"
        android:state_focused="false"
        android:state_selected="false"
        android:state_pressed="false"
        android:drawable="@drawable/ic_action_filter_1" />
    <item
        android:left="2dp" android:right="4dp"
        android:state_focused="false"
        android:state_selected="true"
        android:state_pressed="false"
        android:drawable="@drawable/ic_action_filter_1_s" />
    <item
        android:left="2dp" android:right="4dp"
        android:state_focused="true"
        android:state_selected="false"
        android:state_pressed="false"
        android:drawable="@drawable/ic_action_filter_1_s" />
    <item
        android:left="2dp" android:right="4dp"
        android:state_focused="true"
        android:state_selected="true"
        android:state_pressed="false"
        android:drawable="@drawable/ic_action_filter_1_s" />
    <item
        android:left="2dp" android:right="4dp"
        android:state_pressed="true"
        android:drawable="@drawable/ic_action_filter_1_s" />
</selector>

```

### 设置 Tab

注释之前的 getPageTitle 方法，增加 getTabView 方法。

```java

public View getTabView(int position,TabLayout.Tab tab){

   View v = LayoutInflater.from(mContext).inflate(R.layout.layout_coutom_tab,null);
   ImageView img = (ImageView)v.findViewById(R.id.tab_icon);
   img.setImageResource(tabIcon[position]);
   TextView tv = (TextView)v.findViewById(R.id.tab_text);
   tv.setText(tabTitle[position]);
   if(position == 0){
       v.setSelected(true);
   }
   return v;
}

```

这里给每一个 tab 设置文字和图标。

### 设置 CustomView

给每一个 Tab 设置自定义的布局。

```java

for(int i=0;i<mTabLayout.getTabCount();i++){
    TabLayout.Tab tab = mTabLayout.getTabAt(i);
    tab.setCustomView(adapter.getTabView(i,tab));
 }

```

Tab 不显示问题

这个是之后补充的。在写后面文章的时候，更改了一下 xml 里面的显示布局，运行之后发现 tab 竟然不显示了。开始以为是被标题栏遮挡住了，试过 `android:fitsSystemWindows="true"` 和设置高度都不行。最后找到了解决方式。`

```java

if (ViewCompat.isLaidOut(mTabLayout)) {
  mTabLayout.setupWithViewPager(mViewPager);
} else {
  mTabLayout.addOnLayoutChangeListener(new View.OnLayoutChangeListener() {
      @Override
      public void onLayoutChange(...) {
          mTabLayout.setupWithViewPager(mViewPager);
          mTabLayout.removeOnLayoutChangeListener(this);
      }
  });
}

```
