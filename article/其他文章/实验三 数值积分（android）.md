#实验三 数值积分（android）
实验二博客地址：

实验一博客地址：



一、实验内容

分别写出变步长梯形法和romberge法计算定积分的算法，编写程序上机调试出结果，要求所编程序适用于任何类型的定积分，即能解决这一类问题，而不是某一个问题。

试验中以下列数据验证程序的正确性。

求  (sinx)/x的积分，积分区间为[0,1]



效果：**（源码在文章底部）**

<img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209911923450.png " alt="">  <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209911924521.png " alt="">



主要工作：

1、添加了ThreeFragment的xml界面

2、掌握理解变步长梯形法和龙贝格法



主要逻辑代码：

ThreeFragment:



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

/**
 * 项目名称：NumericCalculationTest
 * 创建人：Double2号
 * 创建时间：2016/4/13 21:41
 * 修改备注：
 */
public class ThreeFragment extends Fragment {

    private View views;
    private Spinner mSpinner;
    private final String[] spinnerChose = {"变步长梯形法", "龙贝格法"};
    private Button btnSure;
    private TextView tvResult;
    private String stringResult;

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
        views = inflater.inflate(R.layout.fra_three, null);
        initView();
        return views;
    }

    private void initView() {
        mSpinner = (Spinner) views.findViewById(R.id.sp_three_chose);
        btnSure = (Button) views.findViewById(R.id.btn_three_sure);
        tvResult = (TextView) views.findViewById(R.id.tv_three_result);

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
            TrapezoidCaculation();
        } else {
            RombergCaculation();
        }
    }

    //变步长梯形法求值
    private void TrapezoidCaculation() {
        Double a = 0.0, b = 1.0, h;
        int m = 1, n = 0;
        Double[] T = new Double[100];//固定只能计算100次

        T[n] = (b - a) * (f(a) + f(b)) / 2;
        do {
            h = (b - a) / m;
            double s = 0.0;
            for (int k = 0; k &lt; m; k++) {
                s += f(a + (k + 0.5) * h);
            }
            T[n + 1] = T[n] / 2 + h / 2 * s;
            m = 2 * m;
            n++;
        } while (Math.abs(T[n] - T[n - 1]) &gt;= 0.0000001);

        stringResult = "n= " + n +
                "\nS= " + T[n]
                + "\n误差为 " + Math.abs(T[n] - T[n - 1]);
        tvResult.setText(stringResult);
    }

    //龙贝格法求值
    private void RombergCaculation() {
        Double a = 0.0, b = 1.0, h;
        Double[][] T = new Double[100][100];//固定只能计算100次

        int i = 0;
        T[i][i] = (b - a) * (f(a) + f(b)) / 2;
        do {
            i++;
            double s = 0;
            for (int j = 0; j &lt;= Math.pow(2, i - 1) - 1; j++) {
                s += f(a + (2 * j + 1) * (b - a) / Math.pow(2, i));
            }
            s = s * (b - a) / Math.pow(2, i);
            s += T[i - 1][0] / 2;
            T[i][0] = s;
            for (int m = 1; m &lt;= i; m++) {
                T[i][m] = (Math.pow(4, m) * T[i][m - 1] - T[i - 1][m - 1]) / (Math.pow(4, m) - 1);
            }
        } while (Math.abs(T[i][i] - T[i - 1][i - 1]) &gt;= 0.0000001);
        stringResult = "n= " + i +
                "\nS= " + T[i][i]
                + "\n误差为 " + Math.abs(T[i][i] - T[i - 1][i - 1]);
        tvResult.setText(stringResult);

    }


    double f(double x) {
        double y;
        //如果x等于0，就直接返回1
        if (x == 0)
            y = 1;
        else y = Math.sin(x) / x;
        return (y);
    }
}

```

