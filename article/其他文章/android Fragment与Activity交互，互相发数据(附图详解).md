#android Fragment与Activity交互，互相发数据(附图详解)
笔者近期看官方training，发现了很多有用又好玩的知识。其中，fragment与Activity通信就是一个。

fragment与Activity通信主要是两点： 

1、fragment传递信息给Activity

此点是通过在fragment中定义接口与Activity共享数据。

2、Activity传递信息给fragment

此点主要是通过fragment的getArgument()和setArgument()两个函数传递bundle来传递。

 

效果：**(最后附上源码)**

<img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16210039198600.png" width="300" height="500" alt=""> 

 

主要流程：

1、在MainActivity中设置一个fragment容器，在MainActivity初始化的时候替换成OneFragment。

 

2、OneFragment中由于定义了接口，在MainActivity中实现了此接口，当点击button的时候会实现此接口中的函数，并且和Activity共享此数据。（也就是EditText中的内容）

 

3、MainActivity中获取到数据后，创建一个新的TwoFragment，并且使用Fragment的setArgument()方法把获取到的数据传递到TwoFragment，TwoFragment使用getArgument()接收。

 

代码：

<img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16210039200161.png" alt=""> 

 

MainActivity:



```
package com.example.trainingfragment;

import android.app.Activity;
import android.app.FragmentManager;
import android.app.FragmentTransaction;
import android.os.Bundle;

public class MainActivity extends Activity
        implements OneFragment.OnOneFragmentClickListener{

    private FragmentManager fm;
    private FragmentTransaction ft;

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        fm=getFragmentManager();
        ft=fm.beginTransaction();
        ft.replace(R.id.fragment_container,new OneFragment());
        //使用FragmentTransaction一定不要忘记提交
        ft.commit();
    }

    @Override
    public void onArticleClick(String text) {
        //此处定义一个新的TwoFragment
        //并且把在OneFragment获取到的text传入TwoFragment
        TwoFragment newFragment = new TwoFragment();
        Bundle args=new Bundle();
        args.putString("text", text);
        newFragment.setArguments(args);

        //由于之前的一个FragmentTransaction已经提交过，所以此处需要重新定义
        ft=fm.beginTransaction();
        ft.replace(R.id.fragment_container, newFragment);
        //使用FragmentTransaction一定不要忘记提交
        ft.commit();

    }
}
```





```
package com.example.trainingfragment;

import android.app.Activity;
import android.app.Fragment;
import android.os.Bundle;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.Button;
import android.widget.EditText;

/**
 * 项目名称：TrainingFragment
 * 类描述：
 * 创建人：佳佳
 * 创建时间：2016/3/25 17:57
 * 修改人：佳佳
 * 修改时间：2016/3/25 17:57
 * 修改备注：
 */
public class OneFragment extends Fragment {

    OnOneFragmentClickListener mCallback;
    private EditText et;
    private Button btn;

    //此处定义接口
    public interface OnOneFragmentClickListener {
        //在接口中定义函数
        void onArticleClick(String text);
    }

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
        View view=inflater.inflate(R.layout.fragment_one,container,false);
        et=(EditText)view.findViewById(R.id.et_one);
        btn=(Button)view.findViewById(R.id.btn_one);
        btn.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                mCallback.onArticleClick(et.getText().toString());
            }
        });

        return view;
    }

    @Override
    public void onAttach(Activity activity) {
        super.onAttach(activity);
        try {
            mCallback = (OnOneFragmentClickListener) activity;
        } catch (ClassCastException e) {
            throw new ClassCastException(activity.toString()
                    + " must implement OnHeadlineSelectedListener");
        }
    }
}

```





```
package com.example.trainingfragment;

import android.app.Fragment;
import android.os.Bundle;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.TextView;

/**
 * 项目名称：TrainingFragment
 * 类描述：
 * 创建人：佳佳
 * 创建时间：2016/3/25 17:57
 * 修改人：佳佳
 * 修改时间：2016/3/25 17:57
 * 修改备注：
 */
public class TwoFragment extends Fragment {


    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {

        return inflater.inflate(R.layout.fragment_two, container, false);
    }

    @Override
    public void onStart() {
        super.onStart();
        //获取Activity传递过来的数据并且显示到
        Bundle args=getArguments();
        setMyText(args.getString("text"));
    }

    public void setMyText(String text) {
        TextView tv=(TextView)getActivity().findViewById(R.id.tv_two);
        tv.setText(text);
    }
}

```





```
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/fragment_container"
    android:layout_width="match_parent"
    android:layout_height="match_parent" /&gt;
```





```
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="10dp"&gt;

    &lt;EditText
        android:id="@+id/et_one"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="请输入要传递的内容" /&gt;

    &lt;Button
        android:id="@+id/btn_one"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="跳转并传递"
        android:textSize="20sp" /&gt;
&lt;/LinearLayout&gt;
```





```
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="10dp"&gt;

    &lt;EditText
        android:id="@+id/et_one"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="请输入要传递的内容" /&gt;

    &lt;Button
        android:id="@+id/btn_one"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="跳转并传递"
        android:textSize="20sp" /&gt;
&lt;/LinearLayout&gt;
```

源码地址：