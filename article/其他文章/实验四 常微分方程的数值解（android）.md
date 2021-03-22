#实验四 常微分方程的数值解（android）
实验一博客地址：

实验二博客地址：

实验三博客地址：



【实验内容】

分别写出改进**欧拉法**与**四阶龙格库塔**求解的算法，编写成熟上机调试出结果，要求所编程序适用于任何一阶常微分方程的数值解问题，即能解决这一类问题，而不是某一个问题。

试验中以下列数据验证程序的正确性。

求

y'=-xy^2

y(0)=2

(0&lt;=x&lt;=5)  步长h=0.25



效果：**（源码在文章结尾）**

**<img src="https://img-blog.csdn.net/20160511111013394?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center" alt="">  <img src="https://img-blog.csdn.net/20160511111103281?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center" alt="">**

****

****



主要工作：

1、添加了FourFragment的xml界面



2、掌握理解改进欧拉法与四阶龙格库塔法





主要逻辑代码

FourFragment:



```
package com.example.double2.numericcalculationtest;

import android.app.Fragment;
import android.os.Bundle;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.ArrayAdapter;
import android.widget.Button;
import android.widget.Spinner;
import android.widget.TextView;

import java.text.DecimalFormat;

/**
 * 项目名称：NumericCalculationTest
 * 创建人：Double2号
 * 创建时间：2016/4/13 21:41
 * 修改备注：
 */
public class FourFragment extends Fragment {

    private View views;
    private Spinner mSpinner;
    private final String[] spinnerChose = {"改进欧拉法", "四阶龙格库塔法"};
    private Button btnSure;
    private TextView tvResult;
    private String stringResult;
    final double h=0.25;
    private DecimalFormat mDecimalFormat = new DecimalFormat("0.00");//保留两位小数

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
        views = inflater.inflate(R.layout.fra_four, null);
        initView();
        return views;
    }

    private void initView() {
        mSpinner = (Spinner) views.findViewById(R.id.sp_four_chose);
        btnSure = (Button) views.findViewById(R.id.btn_four_sure);
        tvResult = (TextView) views.findViewById(R.id.tv_four_result);

        mSpinner.setAdapter(new ArrayAdapter&lt;String&gt;(getActivity(),
                android.R.layout.simple_list_item_1, spinnerChose));
        btnSure.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                caculating();
            }
        });
    }

    private void caculating() {
        if (mSpinner.getSelectedItemPosition() == 0) {
            EulerCaculation();
        } else {
            RungeKuttaCaculation();
        }
    }

    private void EulerCaculation() {
        double[] x=new double[21];
        double[] y=new double[21];
        double[] y_next=new double[21];

        x[0]=0;
        for(int i=1;i&lt;21;i++) {
            x[i]=x[i-1]+h;
        }
        y[0]=2;
        stringResult="x= "+mDecimalFormat.format(x[0])+" y= "+y[0]+"\n";

        for(int i=0;i&lt;20;i++){
            y_next[i]=y[i]+h*f(x[i],y[i]);
            y[i+1]=y[i]+(f(x[i],y[i])+f(x[i+1],y_next[i]))*h/2;
            stringResult+="x= "+mDecimalFormat.format(x[i+1])+" y= "+y[i+1]+"\n";
        }
        tvResult.setText(stringResult);
    }

    private void RungeKuttaCaculation() {
        double[] x=new double[21];
        double[] y=new double[21];
        double[] k=new double[4];

        x[0]=0;
        for(int i=1;i&lt;21;i++) {
            x[i]=x[i-1]+h;
        }
        y[0]=2;
        stringResult="x= "+mDecimalFormat.format(x[0])+" y= "+y[0]+"\n";

        for(int i=0;i&lt;20;i++){
            k[0]=f(x[i],y[i]);
            k[1]=f(x[i]+h/2,y[i]+k[0]*h/2);
            k[2]=f(x[i]+h/2,y[i]+k[1]*h/2);
            k[3]=f(x[i]+h,y[i]+h*k[2]);
            y[i+1]=y[i]+(k[0]+2*k[1]+2*k[2]+k[3])*h/6;
            stringResult+="x= "+mDecimalFormat.format(x[i+1])+" y= "+y[i+1]+"\n";
        }
        tvResult.setText(stringResult);

    }

    double f(double x,double y) {
        return -(x*Math.pow(y,2));
    }
}

```

源码地址：