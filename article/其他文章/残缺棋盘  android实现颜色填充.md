#残缺棋盘  android实现颜色填充
 原“残缺棋盘”题目：

  

 残缺棋盘是一个2^k*2^个方格的棋盘，其中恰有1个方格残缺。图中给出，其中残缺部分用阴影表示。

 <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/2120.png" alt="" style="border:none; max-width:100%"> 

 这样的棋盘称为"三格板"，残缺棋盘问题就是用这四种三格板覆盖更大的残缺棋盘。再次覆盖中要求:

 (1)两个三格板不能重复。

 (2)三格板不能覆盖残缺棋盘方格，但必须覆盖到其他所有的方格。

  

 **添加要求：**

 **1、使用图形化界面实现颜色填充残缺棋盘**

 **2、填充的色块，相邻之间的颜色不能相同**

  

 **最终效果：（源码在文章结尾）**

 <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/2121.png" alt=""> 

  

 **算法可以参考博客：**

  

  

 **android界面实现：**

 1、方格的view用gridview实现，item中放50px*50px的view。

 2、考虑到可能会超出屏幕用HorizontalScrollView包裹gridview。

 3、新建一个mColor的int[][]的二维数组存储每个view的背景颜色，在执行算法时候改变背景颜色。

 执行完毕，更新gridview的adapter。

  

  

 **碰到的问题：**

 HorizontalScrollView与gridview冲突，宽度上无法正常显示。 

  

 **解决办法：**（原理可以参考笔者博客：）

 使用一个LinearLayout包裹gridview，然后让HorizontalScrollView包裹LinearLayout。

 在gridview绑定了adapter之后，设置LinearLayout的宽度就可以了。

  

 如下图：

 <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/2122.png" alt="" width="800" height="150"> 

  

 <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/2123.png" alt="" width="1000" height="200"> 

  

 <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/2124.png" alt=""> 

  

 MainActivity：

 

```
package com.example.double2.gridviewtest;

import android.content.Context;
import android.graphics.Color;
import android.os.AsyncTask;
import android.os.Bundle;
import android.support.v7.app.AppCompatActivity;
import android.view.View;
import android.view.ViewGroup;
import android.widget.AbsListView;
import android.widget.BaseAdapter;
import android.widget.Button;
import android.widget.EditText;
import android.widget.GridView;
import android.widget.LinearLayout;
import android.widget.Toast;

public class MainActivity extends AppCompatActivity {

    private GridView mGridView;
    private int mSize = 1;
    private int mY;
    private int mX;
    final int[] twoColor = {Color.GREEN, Color.YELLOW};
    int colorIndex = 0;//颜色的下标，0的时候表示绿色，1的时候表示黄色
    int colorItem = 1;//填充颜色的时候，一个颜色填两次，然后换一个颜色，colorItem用来记录填充次数
    private int[][] mColor;
    private EditText etSize;
    private EditText etY;
    private EditText etX;
    private Button btnCreate;
    private Button btnStart;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        initView();
    }

    private void initView() {
        mGridView = (GridView) findViewById(R.id.gv_main);
        etSize = (EditText) findViewById(R.id.et_main_size);
        etY = (EditText) findViewById(R.id.et_main_y);
        etX = (EditText) findViewById(R.id.et_main_x);
        btnCreate = (Button) findViewById(R.id.btn_main_create);
        btnStart = (Button) findViewById(R.id.et_main_start);

        btnCreate.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                int k = Integer.parseInt(etSize.getText().toString());
                mSize = (int) Math.pow(2, k);//获取到mSize

                mGridView.setNumColumns(mSize);
                mGridView.setAdapter(new GridViewAdapter(MainActivity.this));
                setGridViewWidth(mGridView);
                mColor = new int[mSize][mSize];
                for (int i = 0; i &lt; mSize; i++) {
                    for (int j = 0; j &lt; mSize; j++)
                        mColor[i][j] = Color.WHITE;
                }

                //重置棋盘的一些信息
                colorIndex = 0;
                colorItem = 1;
            }
        });

        btnStart.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                //添上try catch，防止X和Y的EditText值为空的情况
                try {
                    mY = Integer.parseInt(etY.getText().toString());
                    mX = Integer.parseInt(etX.getText().toString());
                } catch (Exception e) {
                }
                if (mSize == 1) {
                    Toast.makeText(MainActivity.this, "请先创建视图！", Toast.LENGTH_SHORT).show();
                } else if (etY.getText().toString().equals("") || etX.getText().toString().equals("")) {
                    Toast.makeText(MainActivity.this, "请输入X和Y的值！", Toast.LENGTH_SHORT).show();
                } else if (mY &gt;= mSize || mX &gt;= mSize) {
                    Toast.makeText(MainActivity.this, "X或Y的值越界！", Toast.LENGTH_SHORT).show();
                } else {
                    //考虑到可能多次计算同一个size的情况，要让第一个残缺的位置为白色
                    mColor[mX][mY] = Color.WHITE;
                    //使用AsyncTask在线程中计算
                    new Caculation().execute();
                }
            }
        });
    }

    public class Caculation extends AsyncTask&lt;Void, Void, Void&gt; {


        @Override
        protected Void doInBackground(Void... params) {
            //在线程中进行三角板的递归
            cover(0, 0, mX, mY, mSize);
            return null;
        }


        @Override
        protected void onPostExecute(Void aVoid) {
            super.onPostExecute(aVoid);
            //在计算结束之后刷新gridView的adapter
            mGridView.setAdapter(new GridViewAdapter(MainActivity.this));
            setGridViewWidth(mGridView);
        }

        void cover(int tr, int tc, int dr, int dc, int size) {
            int s;
            s = size / 2;

            if (size == 2) {
                if (dr &lt; tr + s &amp;&amp; dc &lt; tc + s) {
                    mColor[tr + s - 1][tc + s] = twoColor[colorIndex];
                    mColor[tr + s][tc + s - 1] = twoColor[colorIndex];
                    mColor[tr + s][tc + s] = twoColor[colorIndex];
                } else if (dr &lt; tr + s &amp;&amp; dc &gt;= tc + s) {
                    mColor[tr + s - 1][tc + s - 1] = twoColor[colorIndex];
                    mColor[tr + s][tc + s - 1] = twoColor[colorIndex];
                    mColor[tr + s][tc + s] = twoColor[colorIndex];
                } else if (dr &gt;= tr + s &amp;&amp; dc &lt; tc + s) {
                    mColor[tr + s - 1][tc + s - 1] = twoColor[colorIndex];
                    mColor[tr + s - 1][tc + s] = twoColor[colorIndex];
                    mColor[tr + s][tc + s] = twoColor[colorIndex];
                } else if (dr &gt;= tr + s &amp;&amp; dc &gt;= tc + s) {
                    mColor[tr + s - 1][tc + s - 1] = twoColor[colorIndex];
                    mColor[tr + s - 1][tc + s] = twoColor[colorIndex];
                    mColor[tr + s][tc + s - 1] = twoColor[colorIndex];
                }
                if (colorItem == 1) {
                    colorItem = 0;
                    if (colorIndex == 1)
                        colorIndex = 0;
                    else
                        colorIndex = 1;
                } else {
                    colorItem++;
                }
            } else {
                if (dr &lt; tr + s &amp;&amp; dc &lt; tc + s) {           //左上
                    mColor[tr + s - 1][tc + s] = Color.RED;
                    mColor[tr + s][tc + s - 1] = Color.RED;
                    mColor[tr + s][tc + s] = Color.RED;
                    cover(tr, tc, dr, dc, s);
                    cover(tr, tc + s, tr + s - 1, tc + s, s);
                    cover(tr + s, tc, tr + s, tc + s - 1, s);
                    cover(tr + s, tc + s, tr + s, tc + s, s);
                } else if (dr &lt; tr + s &amp;&amp; dc &gt;= tc + s) {       //右上
                    mColor[tr + s - 1][tc + s - 1] = Color.RED;
                    mColor[tr + s][tc + s - 1] = Color.RED;
                    mColor[tr + s][tc + s] = Color.RED;
                    cover(tr, tc, tr + s - 1, tc + s - 1, s);
                    cover(tr, tc + s, dr, dc, s);
                    cover(tr + s, tc, tr + s, tc + s - 1, s);
                    cover(tr + s, tc + s, tr + s, tc + s, s);
                } else if (dr &gt;= tr + s &amp;&amp; dc &lt; tc + s) {       //左下
                    mColor[tr + s - 1][tc + s - 1] = Color.RED;
                    mColor[tr + s - 1][tc + s] = Color.RED;
                    mColor[tr + s][tc + s] = Color.RED;
                    cover(tr, tc, tr + s - 1, tc + s - 1, s);
                    cover(tr, tc + s, tr + s - 1, tc + s, s);
                    cover(tr + s, tc, dr, dc, s);
                    cover(tr + s, tc + s, tr + s, tc + s, s);
                } else if (dr &gt;= tr + s &amp;&amp; dc &gt;= tc + s) {      //右上
                    mColor[tr + s - 1][tc + s - 1] = Color.RED;
                    mColor[tr + s - 1][tc + s] = Color.RED;
                    mColor[tr + s][tc + s - 1] = Color.RED;
                    cover(tr, tc, tr + s - 1, tc + s - 1, s);
                    cover(tr, tc + s, tr + s - 1, tc + s, s);
                    cover(tr + s, tc, tr + s, tc + s - 1, s);
                    cover(tr + s, tc + s, dr, dc, s);
                }
            }
        }
    }

    private class GridViewAdapter extends BaseAdapter {

        private Context context;

        public GridViewAdapter(Context context) {
            this.context = context;
        }

        int count = mSize * mSize;

        @Override
        public int getCount() {
            return count;
        }

        @Override
        public Object getItem(int position) {
            return position;
        }

        @Override
        public long getItemId(int position) {
            return position;
        }

        @Override
        public View getView(int position, View convertView, ViewGroup parent) {
            View result = new View(context);
            //设置固定宽高50px
            result.setLayoutParams(new AbsListView.LayoutParams(50, 50));
            result.setBackgroundColor(mColor[position / mSize][position % mSize]); //设置背景颜色
            return result;
        }

    }

    public void setGridViewWidth(GridView gView) {
        //由于HorizontalScrollView和GridView共同使用的时候宽度上会有问题
        //所以此处重置GridView的宽度来解决这个问题
        //56=50+6,单位为px，50为view的宽度，6为预留的子部局间隔
        gView.setLayoutParams((new LinearLayout.LayoutParams(56 * mSize, ViewGroup.LayoutParams.WRAP_CONTENT)));
    }
}

```



 

```
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="10dp"&gt;

    &lt;LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal"&gt;

        &lt;TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="请输入2^k中k的值:"
            android:textColor="#000"
            android:textSize="20sp"/&gt;

        &lt;EditText
            android:id="@+id/et_main_size"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:digits="123456"
            android:hint="请输入1~6的数"
            android:maxLength="1"
            /&gt;
    &lt;/LinearLayout&gt;

    &lt;Button
        android:id="@+id/btn_main_create"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="创 建 视 图"
        android:textSize="20sp"/&gt;

    &lt;LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal"&gt;

        &lt;TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="请输入初始点的位置"
            android:textColor="#000"
            android:textSize="20sp"/&gt;

        &lt;EditText
            android:id="@+id/et_main_x"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:digits="0123456789"
            android:hint="x=?"/&gt;

        &lt;EditText
            android:id="@+id/et_main_y"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:digits="0123456789"
            android:hint="y=?"/&gt;
    &lt;/LinearLayout&gt;

    &lt;Button
        android:id="@+id/et_main_start"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="开 始 运 行"
        android:textSize="20sp"/&gt;

    &lt;TextView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="10dp"
        android:text="结果如下："
        android:textSize="20sp"/&gt;

    &lt;HorizontalScrollView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        &gt;

        &lt;LinearLayout
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:orientation="horizontal"&gt;

            &lt;GridView
                android:id="@+id/gv_main"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:background="#DCDCDC"
                android:horizontalSpacing="4dp"
                android:padding="5dp"
                android:verticalSpacing="4dp"
                /&gt;
        &lt;/LinearLayout&gt;
    &lt;/HorizontalScrollView&gt;
&lt;/LinearLayout&gt;

```



  

 源码地址：