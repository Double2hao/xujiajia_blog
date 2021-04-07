#实验一 直接法解线性方程组(android)
一、实验目的

掌握Guass列选主消去法，三角分解法解线性方程。



二、实验内容

分别写出Guass列选主元消去法，三角分解法的算法，编写程序上机调试出结果，要求所编程序适用于任何线性方程组问题，即能解决这一类问题，而不是某一个问题。

实验中以下列数据验证程序的正确性

1、Guass列选主元消去法

[2.5 2.3 -5.1][x1]  [3.7]

[5.3 9.6  1.5][x2]=[3.8]

[8.1 1.7 -4.3][x3]  [5.5]



2、Doolittle三角分解法

[ 2  10 0    -3 ]

[-3  -4 -12 13]

[ 1  2   3    -4 ]

[ 4  14 9   -13]



android效果：**（源码附在文章底部）**

<img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/2430.png" width="300" height="500" alt="">  <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/2431.png" width="300" height="500" alt="">  <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/2432.png" width="300" height="500" alt="">



**大致思路：**

1、一共有5次实验，采用比较方便的fragment，底部是5个button

2、每一个实验中的分功能使用spinner进行选择。

3、通过顶部的一个TextView来提示用户输入的内容。

4、让用户每次输入矩阵中的一个值，方便存储。



对fragment有兴趣的读者可以看一下笔者的另外几篇博客：











主要的运算逻辑就在OneFragment中处理了，此处就仅贴上OneFragment的代码。

OneFragment：



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
public class OneFragment extends Fragment {
    private View views;
    private TextView tvGuidance;
    private Button btnInPut;
    private EditText etInput;
    private Button btnClear;
    private TextView tvProgress;
    private TextView tvResult;
    private Spinner spChose;
    private final String[] spinnerChose = {"Guass消去法", "Doolittle分解法"};
    private Double[][] aMatrix;//a矩阵
    private Double[] bMatrix;//b矩阵
    private int matrixSize;//矩阵大小
    private int linePosition;//行坐标
    private int listPosition;//列坐标
    private String progressShow;//用来展示过程
    private String resultShow;//用来展示结果
    private DecimalFormat mDecimalFormat = new DecimalFormat("0.00");//保留两位小数


    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
        views = inflater.inflate(R.layout.act_one, null);
        initView();
        return views;
    }

    private void initView() {
        tvGuidance = (TextView) views.findViewById(R.id.tv_one_guidance);
        btnInPut = (Button) views.findViewById(R.id.btn_input);
        etInput = (EditText) views.findViewById(R.id.et_input);
        btnClear = (Button) views.findViewById(R.id.btn_clear);
        tvProgress = (TextView) views.findViewById(R.id.tv_one_progress);
        tvResult = (TextView) views.findViewById(R.id.tv_one_result);
        spChose = (Spinner) views.findViewById(R.id.sp_one_chose);

        tvGuidance.setText(R.string.input_size);
        spChose.setAdapter(new ArrayAdapter&lt;String&gt;(getActivity(),
                android.R.layout.simple_list_item_1, spinnerChose));

        btnInPut.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                caculating();
            }
        });
        btnClear.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                clearData();
            }
        });
    }

    private void caculating() {
        //获取到EditText中的String
        //如果为空就提示用户并且直接返回
        String textInput = etInput.getText().toString();
        etInput.setText("");
        if (textInput.equals("")) {
            Toast.makeText(getActivity(), R.string.not_input_null, Toast.LENGTH_SHORT).show();
            return;
        }
        Log.d("xujiajia", spChose.getSelectedItemPosition() + "");
        //判断为GUASS消元法还是Doolittle分解法
        if (spChose.getSelectedItemPosition() == 0) {
            GuassCaculation(textInput);
        } else {
            DoolittleCaculation(textInput);
        }
    }

    private void DoolittleCaculation(String textInput) {
        String textGuidance = tvGuidance.getText().toString();
        //需要输入矩阵大小或者输入矩阵中的值
        Log.d("xujiajia", textGuidance);
        //通过提示的TextView的text来判断进行到哪一步
        if (textGuidance.equals(getString(R.string.input_size))) {
            matrixSize = Integer.parseInt(textInput);
            Log.d("xujiajia", matrixSize + "");
            aMatrix = new Double[matrixSize][matrixSize];
            progressShow = "a矩阵的大小被定义为 " + matrixSize + "如下\n" + "[ ";
            tvProgress.setText(progressShow);
            tvGuidance.setText("请依次输入a矩阵中的值");
            linePosition = 0;
            listPosition = 0;
        } else {
            aMatrix[listPosition][linePosition] = Double.parseDouble(textInput);
            linePosition++;
            progressShow += textInput + " ";
            if (linePosition == matrixSize) {
                progressShow += "]\n";
                if (listPosition != matrixSize - 1)//如果不为最后一行就加前中括号
                    progressShow += "[ ";
                linePosition = 0;
                listPosition++;
            }
            tvProgress.setText(progressShow);
            if (listPosition == matrixSize) {
                //如果列坐标与matrixSize相等，说明已经输入完毕，进行运算
                for (int i = 1; i &lt; matrixSize; i++) {
                    aMatrix[i][0] /= aMatrix[0][0];
                    for (int j = 1; j &lt; matrixSize; j++) {
                        if (i &gt; j) {
                            for (int k = 0; k &lt; j; k++) {
                                aMatrix[i][j] -= aMatrix[k][j] * aMatrix[i][k];
                            }
                            aMatrix[i][j] /= aMatrix[j][j];
                        } else if (i &lt;= j) {
                            for (int k = 0; k &lt; i; k++)
                                aMatrix[i][j] -= aMatrix[k][j] * aMatrix[i][k];
                        }

                    }
                }

                //运算完毕，创建L矩阵和U矩阵，赋值并显示
                Double[][] lMatrix = new Double[matrixSize][matrixSize];
                Double[][] uMatrix = new Double[matrixSize][matrixSize];
                for (int i = 0; i &lt; matrixSize; i++) {
                    for (int j = 0; j &lt; matrixSize; j++)
                        if (i == j) {
                            lMatrix[i][j] = 1.0;
                        } else if (i &lt; j) {
                            lMatrix[i][j] = 0.0;
                        } else if (i &gt; j) {
                            lMatrix[i][j] = aMatrix[i][j];
                        }
                }

                for (int i = 0; i &lt; matrixSize; i++) {
                    for (int j = 0; j &lt; matrixSize; j++)
                        if (i &lt;= j) {
                            uMatrix[i][j] = aMatrix[i][j];
                        } else if (i &gt; j) {
                            uMatrix[i][j] = 0.0;
                        }
                }

                resultShow = "最后算出的L如下\n";
                for (int i = 0; i &lt; matrixSize; i++) {
                    resultShow += "[ ";
                    for (int j = 0; j &lt; matrixSize; j++) {
                        resultShow += mDecimalFormat.format(lMatrix[i][j]) + " ";//保留两位小数输出
                    }
                    resultShow += "]\n";
                }

                resultShow += "最后算出的U如下\n";
                for (int i = 0; i &lt; matrixSize; i++) {
                    resultShow += "[ ";
                    for (int j = 0; j &lt; matrixSize; j++) {
                        resultShow += mDecimalFormat.format(uMatrix[i][j]) + " ";//保留两位小数输出
                    }
                    resultShow += "]\n";
                }

                tvResult.setText(resultShow);
                btnInPut.setClickable(false);//运算结束时，禁止用户点击输入，必须点击清空
            }
        }
    }

    private void GuassCaculation(String textInput) {
        String textGuidance = tvGuidance.getText().toString();
        //需要输入矩阵大小或者输入矩阵中的值
        Log.d("xujiajia", textGuidance);
        if (textGuidance.equals(getString(R.string.input_size))) {
            matrixSize = Integer.parseInt(textInput);
            aMatrix = new Double[matrixSize][matrixSize];
            progressShow = "a矩阵的大小被定义为 " + matrixSize + "如下\n" + "[ ";
            tvProgress.setText(progressShow);
            tvGuidance.setText(R.string.input_a_number);
            linePosition = 0;
            listPosition = 0;
        } else if (textGuidance.equals(getString(R.string.input_a_number))) {
            aMatrix[listPosition][linePosition] = Double.parseDouble(textInput);
            linePosition++;
            progressShow += textInput + " ";
            if (linePosition == matrixSize) {
                progressShow += "]\n";
                if (listPosition != matrixSize - 1)//如果不为最后一行就加前中括号
                    progressShow += "[ ";
                linePosition = 0;
                listPosition++;
                if (listPosition == matrixSize) {
                    //如果列坐标与size相等，说明a已经输入完毕，输入b
                    tvGuidance.setText(R.string.input_b_number);
                    bMatrix = new Double[matrixSize];
                    progressShow += "b矩阵如下\n[ ";
                }
            }
            tvProgress.setText(progressShow);
        } else if (textGuidance.equals(getString(R.string.input_b_number))) {
            bMatrix[linePosition] = Double.parseDouble(textInput);
            linePosition++;
            progressShow += textInput + " ";
            if (linePosition == matrixSize) {
                progressShow += "]\n";
            }
            tvProgress.setText(progressShow);

            if (linePosition == matrixSize) {
                caculateAB();
            }
        }
    }

    private void caculateAB() {
        //ab矩阵都输入完毕，接下来进入计算
        for (int i = 0; i &lt; matrixSize - 1; i++) {
            linePosition = 0;
            listPosition = 0;
            Double maxNum = aMatrix[i][0];
            for (int j = i; j &lt; matrixSize; j++) {
                if (Math.abs(aMatrix[j][i]) &gt; Math.abs(maxNum))//标记最大的数以及所在行
                {
                    maxNum = aMatrix[j][i];
                    listPosition = j;
                }
            }
            //交换行
            if (listPosition != i) {
                Double[] aTemp = aMatrix[listPosition];
                aMatrix[listPosition] = aMatrix[i];
                aMatrix[i] = aTemp;

                Double bTemp = bMatrix[listPosition];
                bMatrix[listPosition] = bMatrix[i];
                bMatrix[i] = bTemp;
            }
            //消元
            for (int j = i + 1; j &lt; matrixSize; j++) {
                Double factor = aMatrix[j][i] / maxNum;
                //a中循环计算
                for (int k = 0; k &lt; matrixSize; k++)
                    aMatrix[j][k] -= aMatrix[i][k] * factor;
                //b中循环计算
                bMatrix[j] -= bMatrix[i] * factor;
            }

        }

        //创建X并且计算
        resultShow = "最后算出的x如下\n[ ";
        Double[] xMatrix = new Double[matrixSize];
        for (int i = matrixSize - 1; i &gt;= 0; i--) {
            Double number = bMatrix[i];
            for (int j = matrixSize - 1; j &gt; i; j--)
                number -= aMatrix[i][j] * xMatrix[j];
            number /= aMatrix[i][i];
            xMatrix[i] = number;
        }
        for (int i = 0; i &lt; matrixSize; i++)
            resultShow += mDecimalFormat.format(xMatrix[i]) + " ";//保留两位小数输出
        resultShow += "]\n";
        tvResult.setText(resultShow);
        btnInPut.setClickable(false);//运算结束时，禁止用户点击输入，必须点击清空
    }

    //清空所有数据
    private void clearData() {
        tvGuidance.setText("请输入矩阵的大小");
        progressShow = "";
        resultShow = "";
        tvProgress.setText("");
        tvResult.setText("");
        listPosition = 0;
        linePosition = 0;
        aMatrix = null;
        bMatrix = null;
        btnInPut.setClickable(true);
    }
}

```





