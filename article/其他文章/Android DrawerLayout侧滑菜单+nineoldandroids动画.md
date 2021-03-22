#Android DrawerLayout侧滑菜单+nineoldandroids动画


参考博客：





DrawerLayout，鸿洋前辈的博客中其实写的已经很不错了，但是在**ViewHelper**的动画设置上，鸿洋前辈并未有过多的描述，对笔者此类没接触过**nineoldandroids**动画的新手来讲真的有点余言未尽的感觉，于是笔者下载了前辈的demo，然后自己又学习了一下。

 

学习内容：**（源码在文章结尾）**

1、简化了demo，仅仅留下了左边的抽屉。

2、添加了详细的注释。

3、学习了andorid studio添加library的方法。（添加nineoldandroids的library）

 

效果如图：

<img src="https://img-blog.csdn.net/20160108162824953?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center" width="400" height="700" alt="">  <img src="https://img-blog.csdn.net/20160108162828534?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center" width="400" height="700" alt=""> 

 

MainAcitivity：



```
package com.example.drawerlayouttest;

import android.os.Bundle;
import android.support.v4.app.FragmentActivity;
import android.support.v4.widget.DrawerLayout;
import android.support.v4.widget.DrawerLayout.DrawerListener;
import android.view.View;
import android.view.Window;

import com.nineoldandroids.view.ViewHelper;


public class MainActivity extends FragmentActivity
{

	private DrawerLayout mDrawerLayout;

	@Override
	protected void onCreate(Bundle savedInstanceState)
	{
		super.onCreate(savedInstanceState);
		requestWindowFeature(Window.FEATURE_NO_TITLE);
		setContentView(R.layout.activity_main);

		mDrawerLayout = (DrawerLayout) findViewById(R.id.id_drawerLayout);
		initEvents();

	}


	private void initEvents()
	{
		mDrawerLayout.setDrawerListener(new DrawerListener()
		{
			@Override
			public void onDrawerStateChanged(int newState)
			{
			}

			//当产生抽屉滑动时
			@Override
			public void onDrawerSlide(View drawerView, float slideOffset)
			{
				//获取mDrawerLayout中的第一个子布局，也就是布局中的Relativelayt
				//获取抽屉的view
				View mContent = mDrawerLayout.getChildAt(0);
				View mMenu = drawerView;
				float scale = 1 - slideOffset;
				float rightScale = 0.8f + scale * 0.2f;

				if (drawerView.getTag().equals("LEFT"))
				{

					float leftScale = 1 - 0.3f * scale;

					//设置左边菜单滑动后的占据屏幕大小
					ViewHelper.setScaleX(mMenu, leftScale);
					ViewHelper.setScaleY(mMenu, leftScale);
					//设置菜单透明度
					ViewHelper.setAlpha(mMenu, 0.6f + 0.4f * (1 - scale));

					//设置内容界面水平和垂直方向偏转量
					//在滑动时内容界面的宽度为 屏幕宽度减去菜单界面所占宽度
					ViewHelper.setTranslationX(mContent,
							mMenu.getMeasuredWidth() * (1 - scale));
					//设置内容界面操作无效（比如有button就会点击无效）
					mContent.invalidate();
					//设置右边菜单滑动后的占据屏幕大小
					ViewHelper.setScaleX(mContent, rightScale);
					ViewHelper.setScaleY(mContent, rightScale);
				}
			}

			@Override
			public void onDrawerOpened(View drawerView)
			{
			}

			@Override
			public void onDrawerClosed(View drawerView)
			{
			}
		});
	}



}

```



 

另外也是附上android studio中library的添加方法，**nineoldandroids的包需要按此方法手动添加**，如下图：

 

1、在file中找到 project stucture。

 

<img src="https://img-blog.csdn.net/20160108163415393?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center" alt=""> 

 

2、打开后，选择 app-&gt;dependencies，点击加号添加library

 

<img src="https://img-blog.csdn.net/20160108163443058?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center" alt=""> 

 

3、在 Choose Library Dependency的搜索框中搜索nineoldandroids，选择要添加的library。然后点击OK，添加完成。

 

<img src="https://img-blog.csdn.net/20160108163454051?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center" alt=""> 

 

 

源码地址：

 

 

