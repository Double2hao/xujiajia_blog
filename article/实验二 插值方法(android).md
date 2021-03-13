#实验二 插值方法(android)
实验一博客地址：



一、实验内容

分别写出拉格朗日插值法与牛顿插值法的算法，编写程序上机调试出结果，要求所编程序适用于在任何一组插值节点，即能解决这一类问题，而不是某一个问题。

试验中以下列数据验证程序的正确性。

已知下列函数表

X   0.56160   0.56280   0.56401   0.56521

Y   0.82741   0.82659   0.82577   0.82495

求X=0.5635时的函数值。



最终效果：**(源码在文章底部)**

<img src="https://img-blog.csdn.net/20160422093639010?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center" alt="">  <img src="https://img-blog.csdn.net/20160422093646307?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center" alt="">



主要工作：

1、理解拉格朗日插值法和牛顿插值法的公式。



2、由于采用的是fragment，直接复用的实验一的xml布局。



3、牛顿插值法的算法中采用了递归，为了提高效率直接在递归中判断是否要加入牛顿插值的结果。



具体算法就不多说了，主要逻辑写在TwoFragment中，若对此作业有兴趣可以自行拖到底部下载源码。



TwoFragment.java



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
public class TwoFragment extends Fragment {
    private View views;
    private TextView tvTitle;
    private TextView tvGuidance;
    private Button btnInPut;
    private EditText etInput;
    private Button btnClear;
    private TextView tvProgress;
    private TextView tvResult;
    private Spinner spChose;
    private final String[] spinnerChose = {"Lagrange插值", "Newton插值"};

    private String xShow;//用来展示过程
    private String yShow;//用来展示过程
    private int dataSize;
    private Double[][] data;
    private Double myX;
    private int linePosition;//行坐标
    private Double resultNewton = 0.0;
    private boolean[] hasAdded; //记录该均差是否已经计算过，加入过了结构
    private DecimalFormat mDecimalFormat = new DecimalFormat("0.00000000");//保留八位小数


    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
        views = inflater.inflate(R.layout.fra_one_and_two, null);
        initView();
        return views;
    }

    private void initView() {
        tvTitle = (TextView) views.findViewById(R.id.tv_title);
        tvGuidance = (TextView) views.findViewById(R.id.tv_one_guidance);
        btnInPut = (Button) views.findViewById(R.id.btn_input);
        etInput = (EditText) views.findViewById(R.id.et_input);
        btnClear = (Button) views.findViewById(R.id.btn_clear);
        tvProgress = (TextView) views.findViewById(R.id.tv_one_progress);
        tvResult = (TextView) views.findViewById(R.id.tv_one_result);
        spChose = (Spinner) views.findViewById(R.id.sp_one_chose);

        tvTitle.setText("插值方法");
        tvGuidance.setText(R.string.input_data_size);
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

    private void clearData() {
        tvGuidance.setText(R.string.input_data_size);
        xShow = "";
        yShow = "";
        tvProgress.setText("");
        tvResult.setText("");
        linePosition = 0;
        dataSize = 0;
        data = null;
        hasAdded = null;
        btnInPut.setClickable(true);
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

        //Lagrange插值和Newton插值在计算之前的读取数据过程一样
        readData(textInput);

    }

    private void readData(String textInput) {
        String textGuidance = tvGuidance.getText().toString();
        //通过提示的TextView的text来判断进行到哪一步
        if (textGuidance.equals(getString(R.string.input_data_size))) {
            dataSize = Integer.parseInt(textInput);
            data = new Double[2][dataSize];
            xShow = "x分别为:\n";
            yShow = "y分别为:\n";
            tvProgress.setText(xShow + yShow);
            tvGuidance.setText(R.string.input_x);
            linePosition = 0;

        } else if (textGuidance.equals(getString(R.string.input_x))) {
            data[0][linePosition] = Double.parseDouble(textInput);
            Log.d("xujiajia1", data[0][linePosition] + "");
            xShow += data[0][linePosition] + "\n";
            tvProgress.setText(xShow + yShow);
            tvGuidance.setText(R.string.input_y);
        } else if (textGuidance.equals(getString(R.string.input_y))) {
            data[1][linePosition] = Double.parseDouble(textInput);
            yShow += data[1][linePosition] + "\n";
            tvProgress.setText(xShow + yShow);
            linePosition++;

            //倘若还没有输入完
            if (linePosition != dataSize) {
                tvGuidance.setText(R.string.input_x);
            } else {
                tvGuidance.setText(R.string.input_my_x);
                yShow += "你需要测的x为:\n";
                tvProgress.setText(xShow + yShow);
            }

        } else if (textGuidance.equals(getString(R.string.input_my_x))) {
            myX = Double.parseDouble(textInput);
            yShow += myX + "\n";
            tvProgress.setText(xShow + yShow);
            tvGuidance.setText(R.string.input_end);

            if (spChose.getSelectedItemPosition() == 0) {
                LagrangeCaculation();
            } else {
                NewtonCaculation();
            }

        }
    }

    private void LagrangeCaculation() {
        Double result = 0.0;
        Double aloneResule;
        for (int i = 0; i &lt; dataSize; i++) {
            aloneResule = data[1][i];
            for (int j = 0; j &lt; dataSize; j++) {
                if (i != j) {
                    aloneResule *= (myX - data[0][j]) / (data[0][i] - data[0][j]);
                }
            }
            result += aloneResule;
        }
        tvResult.setText(mDecimalFormat.format(result) + "");
        btnInPut.setClickable(false);
    }

    private void NewtonCaculation() {
        hasAdded = new boolean[dataSize];
        for (int i = 0; i &lt; dataSize; i++) {
            hasAdded[i] = false;
        }
        int[] dataIndex = new int[dataSize];
        for (int i = 0; i &lt; dataSize; i++)
            dataIndex[i] = i;

        resultNewton = data[1][0];//初始值为
        hasAdded[0] = true;
        meanDeviation(dataIndex);//开始递归
        tvResult.setText(mDecimalFormat.format(resultNewton) + "");
        btnInPut.setClickable(false);
    }

    //计算均差的递归，并且在每次递归的时候判断是否要加入到结果里
    private Double meanDeviation(int[] dataIndex) {
        int size = dataIndex.length;
        Double deviation;

        if (size == 1) {
            deviation = data[1][0];
        } else if (size == 2) {
            deviation = (data[1][dataIndex[1]] - data[1][dataIndex[0]]) /
                    (data[0][dataIndex[1]] - data[0][dataIndex[0]]);
        } else {
            //如果大于二就进入递归
            int[] dataIndex_one = new int[size - 1];
            int[] dataIndex_two = new int[size - 1];
            for (int i = 0; i &lt; size - 2; i++) {
                dataIndex_one[i] = dataIndex[i];
                dataIndex_two[i] = dataIndex[i];
            }
            dataIndex_one[size - 2] = dataIndex[size - 1];
            dataIndex_two[size - 2] = dataIndex[size - 2];
            deviation = (meanDeviation(dataIndex_one) - meanDeviation(dataIndex_two)) /
                    (data[0][dataIndex[size - 1]] - data[0][dataIndex[size - 2]]);
        }

        //判断这个值是否要加到newton插值的结果中
        boolean isNeedAdd = true;
        if (dataIndex[0] == 0) {
            for (int i = 0; i &lt; size - 1; i++) {
                if (dataIndex[i] != dataIndex[i + 1] - 1) {
                    isNeedAdd = false;
                    break;
                }
            }
        } else {
            isNeedAdd = false;
        }

        //如果是满足要求的均差并且没有添加过，那么久添加
        if (isNeedAdd &amp;&amp; !hasAdded[size - 1]) {
            hasAdded[size - 1] = true;
            double item = deviation;
            for (int j = 0; j &lt; size - 1; j++) {
                Log.d("xujiajia4", size + "  " + data[0][j]);
                item *= (myX - data[0][j]);
                Log.d("xujiajia4", item + "");
            }
            resultNewton += item;
        }
        return deviation;
    }


}

```

