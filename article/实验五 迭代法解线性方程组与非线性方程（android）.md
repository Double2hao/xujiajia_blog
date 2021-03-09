#实验五 迭代法解线性方程组与非线性方程（android）
实验一博客地址：

实验二博客地址：

实验三博客地址：

实验四博客地址：





【实验内容】



分别写出高斯-赛德尔迭代法与牛顿迭代法的算法，编写程序上机调试出结果，要求所编程序适用于任何一个方程的求根，即能解决这一类问题，而不是某一个问题。



1、高斯-赛德尔迭代法求解线性方程组

[ 7  2   1 -2][x1]  [ 4]

[ 9 15  3 -2][x2]  [ 7]

[-2 -2 11  5][x3]=[-1]

[1  3  2  13][x4]  [ 0]



2、用牛顿迭代法求方程x^3-x-1=0的近似根，精确度&lt;=0.00001，牛顿法的初始值为1.



效果：**（源码在文章结尾）**

<img src="https://img-blog.csdn.net/20160524212756971?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center" alt=""> <img src="https://img-blog.csdn.net/20160524212812862?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center" alt="">





主要工作：

1、添加了FiveFragment的xml界面,直接采用文本读取的方式读取矩阵。（使用java的split）

输入的时候比较方便，可以直接复制。但是输入内容如果有错误就会导致程序崩溃，比较难控制。





2、掌握理解高斯-赛德尔迭代法和牛顿迭代法





主要逻辑代码

FiveFragment：



```
package com.example.double2.numericcalculationtest;

import android.app.Fragment;
import android.os.Bundle;
import android.util.Log;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.ArrayAdapter;
import android.widget.Button;
import android.widget.EditText;
import android.widget.Spinner;
import android.widget.TextView;
import android.widget.Toast;

import java.text.DecimalFormat;

/**
 * 项目名称：NumericCalculationTest
 * 创建人：Double2号
 * 创建时间：2016/4/13 21:41
 * 修改备注：
 */
public class FiveFragment extends Fragment {

    private View views;
    private Spinner spChose;
    private final String[] spinnerChose = {"高斯-赛德尔迭代法", "牛顿法"};
    private EditText etInputN;
    private EditText etInputMatrix;
    private Button btnCalculate;
    private Button btnClear;
    private TextView tvResult;
    private String resultShow;//用来展示结果
    private DecimalFormat mDecimalFormat = new DecimalFormat("0.000000");//保留六位小数

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
        views = inflater.inflate(R.layout.fra_five, null);
        initView();
        return views;
    }

    private void initView() {
        spChose = (Spinner) views.findViewById(R.id.sp_five_chose);
        etInputN = (EditText) views.findViewById(R.id.et_input_n);
        etInputMatrix = (EditText) views.findViewById(R.id.et_input_matrix);
        btnCalculate = (Button) views.findViewById(R.id.btn_five_calculate);
        btnClear = (Button) views.findViewById(R.id.btn_five_clear);
        tvResult = (TextView) views.findViewById(R.id.tv_five_result);

        spChose.setAdapter(new ArrayAdapter&lt;String&gt;(getActivity(),
                android.R.layout.simple_list_item_1, spinnerChose));

        btnCalculate.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                calculating();
            }
        });
        btnClear.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                clearData();
            }
        });
    }

    private void calculating() {
        //获取到EditText中的String
        String textInputN = etInputN.getText().toString();
        String textInputMatrix = etInputMatrix.getText().toString();

        Log.d("xujiajia", spChose.getSelectedItemPosition() + "");
        //判断为高斯-赛德尔迭代法还是牛顿法
        if (spChose.getSelectedItemPosition() == 0) {
            //如果为空就提示用户并且直接返回
            if (textInputN.equals("") || textInputMatrix.equals("")) {
                Toast.makeText(getActivity(), R.string.not_input_null, Toast.LENGTH_SHORT).show();
                return;
            } else {
                G_sCalculation(textInputN, textInputMatrix);
            }
        } else {
            NewtonCalculation();
            etInputMatrix.setText("牛顿法计算的为x^3-x-1=0的近似根\n精度&lt;=0.00001，牛顿法的初始值为1。");
        }
    }


    private void G_sCalculation(String textInputN, String textInputMatrix) {
        int n = Integer.parseInt(textInputN);
        final int iSize = 5;//迭代次数
        double[][] matrix = new double[n][n + 1];//用来存储矩阵
        double[][] result = new double[iSize + 1][n];//result用来存储迭代的数,规定迭代5次,第一组都为0

        String[] line = textInputMatrix.split("\n");
        Log.d("xujiajia5", line[0] + "\n");
        Log.d("xujiajia5", line[1] + "\n");
        for (int i = 0; i &lt; n; i++) {
            String[] num = line[i].split(" ");
            for (int j = 0; j &lt; n + 1; j++) {
                Log.d("xujiajia5", num[j]);
                matrix[i][j] = Double.parseDouble(num[j]);
            }
            result[0][i] = 0;//初始向量设置为0
        }

        for (int i = 1; i &lt; iSize + 1; i++) {
            for (int j = 0; j &lt; n; j++) {
                result[i][j] = matrix[j][n];
                for (int k = 0; k &lt; n; k++)
                    if (k &lt; j) {
                        result[i][j] -= matrix[j][k] * result[i][k];
                        Log.d("xujiajia555", matrix[j][k] + "");
                        Log.d("xujiajia555", result[i][k] + "");
                    } else if (k &gt; j) {
                        result[i][j] -= matrix[j][k] * result[i - 1][k];
                    }
                result[i][j] /= matrix[j][j];
            }
        }

        resultShow = "迭代次数为 " + iSize + "\n";
        resultShow += "最终的结果为：\n";
        for (int i = 0; i &lt; n; i++)
            resultShow += mDecimalFormat.format(result[iSize][i]) + "\n";
        tvResult.setText(resultShow);

    }

    private void NewtonCalculation() {
        double[] k = new double[3];
        k[0] = 0.5;
        k[1] = 1;

        while (Math.abs(k[1] - k[0]) &gt;= 0.00001) {
            k[2] = k[1] - f(k[1]) / (f(k[1]) - f(k[0])) * (k[1] - k[0]);
            k[0] = k[1];
            k[1] = k[2];
        }

        resultShow = "牛顿法的结果为：\n";
        resultShow += k[1] + "";
        tvResult.setText(resultShow);
    }

    private void clearData() {
        etInputN.setText("");
        etInputMatrix.setText("");
        tvResult.setText("");
        resultShow = "";
    }

    double f(double x) {
        return (Math.pow(x, 3) - x - 1);
    }

}

```



源码地址：
