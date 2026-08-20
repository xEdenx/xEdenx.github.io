---
title: 使用TabLayout实现底部自定义Tab布局
description: 在 Android 中实现底部自定义 TabLayout 的步骤。
publishDate: 2015-10-10 11:50:18
tags:
  - tablayout
  - tab
  - android
---

转自：[使用TabLayout实现底部Tab布局](http://www.aswifter.com/2015/08/09/implements-bottom-tab-with-tablayout/)

布局
下面我们开始实现底部Tab，layout布局比较简单，我们只用把TabLayout放置在底部即可

```xml

<LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout_below="@+id/appbar"
        android:orientation="vertical">


        <android.support.v4.view.ViewPager
            android:id="@+id/viewPager"
            android:layout_width="match_parent"
            android:layout_height="0dp"
            android:layout_weight="1.0"
            android:scrollbars="none" />

        <android.support.design.widget.TabLayout
            android:id="@+id/tabLayout"
            style="@style/MyCustomTabLayout"
            android:layout_width="match_parent"
            android:layout_height="wrap_content" />

</LinearLayout>

```

我定义了一个自定义的style,把tabIndicatorHeight设为0dp

```xml

<style name="MyCustomTabLayout" parent="Widget.Design.TabLayout">
        <item name="tabMaxWidth">@dimen/tab_max_width</item>
        <item name="tabIndicatorColor">?attr/colorAccent</item>
        <item name="tabIndicatorHeight">0dp</item>
        <item name="tabPaddingStart">12dp</item>
        <item name="tabPaddingEnd">12dp</item>
        <item name="tabBackground">@color/tab_bgcolor</item>
        <item name="tabSelectedTextColor">?android:textColorPrimary</item>
</style>

```

代码实现:
我们首先设置好ViewPager，然后设置TabLayout与ViewPager的对应关系，最后最关键的是使用TabLayout的setCustomView设置自定义的TAB View。

```java

viewPager = (ViewPager)findViewById(R.id.viewPager);
tabLayout = (TabLayout) findViewById(R.id.tabLayout);

SampleFragmentPagerAdapter pagerAdapter =
        new SampleFragmentPagerAdapter(getSupportFragmentManager(), this);
viewPager.setAdapter(pagerAdapter);

tabLayout.setupWithViewPager(viewPager);

for (int i = 0; i < tabLayout.getTabCount(); i++) {
    TabLayout.Tab tab = tabLayout.getTabAt(i);
    if (tab != null) {
        tab.setCustomView(pagerAdapter.getTabView(i));
    }
}

```

```java

viewPager.setCurrentItem(1);
public View getTabView(int position) {
            View v = LayoutInflater.from(context).inflate(R.layout.custom_tab, null);
            TextView tv = (TextView) v.findViewById(R.id.textView);
            tv.setText(tabTitles[position]);
            ImageView img = (ImageView) v.findViewById(R.id.imageView);
            //img.setImageResource(imageResId[position]);
            return v;
        }

```
