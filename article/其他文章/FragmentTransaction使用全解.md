#FragmentTransaction使用全解
# 前言

之前已经写过用Fragment做主界面以及Fragment与Activity通信的文章。两篇文章如下：    

对于Fragment还剩FragmentTransaction没有具体讲到，此篇文章就讲一下FragmentTransaction的主要用法，也是对之前的回顾。

## （源码在文章结尾）

# 简单用法

<img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16210039262450.png" alt="这里写图片描述" title="">  如上是Fragment的简单使用演示，主要代码如下：

```
        btnMainOne =(Button)findViewById(R.id.btn_main_one);

        btnMainOne.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                mFragmentTransaction=getFragmentManager().beginTransaction();
                mFragmentTransaction.replace(R.id.fl_main,new FragmentOne());
                //设置简单的过度动画
                mFragmentTransaction.setTransition(FragmentTransaction.TRANSIT_FRAGMENT_OPEN);
                mFragmentTransaction.commit();
            }
        });

```

## FragmentTransaction使用注意

每次在使用FragmentTransaction的时候都需要重新获取，每一个FragmentTransaction只能够commit（）一次。  用代码表示下错误使用：

```
 btnMainOne =(Button)findViewById(R.id.btn_main_one);
 mFragmentTransaction=getFragmentManager().beginTransaction();

        btnMainOne.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {

                mFragmentTransaction.replace(R.id.fl_main,new FragmentOne());
                //设置简单的过度动画
                mFragmentTransaction.setTransition(FragmentTransaction.TRANSIT_FRAGMENT_OPEN);
                mFragmentTransaction.commit();
            }
        });
```

错误使用中把FragmentTransaction的获取挪到了点击事件之前，这样在点击第二次的时候就会出现错误。

# 添加到返回栈

在使用Fragment的时候，我们经常会有一个这样的需求，就是需要通过返回键让fragment回复到之前的一个状态。使用FragmentTransaction就能很容易做到，效果如下：  <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16210039262761.png" alt="这里写图片描述" title="">

其实主要就是通过addToBackStack（）这个方法，在commit（）之前使用，能够保留commit之前的状态，在使用返回键时，能够回到之前的状态。  具体代码如下：

这里写代码片

```
private void initViewTwo() {
        //主要逻辑在MainActivity的onClick中
        btnMainOne =(Button)findViewById(R.id.btn_main_one);
        btnMainOne.setOnClickListener(this);
        btnMainTwo =(Button)findViewById(R.id.btn_main_two);
        btnMainTwo.setOnClickListener(this);
        btnMainThree =(Button)findViewById(R.id.btn_main_three);
        btnMainThree.setOnClickListener(this);
    }


    @Override
    public void onClick(View v) {
        //每次点击事件都会重新获取FragmentTransaction
        mFragmentTransaction=getFragmentManager().beginTransaction();

        switch (v.getId()){
            case R.id.btn_main_one:
                if(fraOne==null){
                    fraOne=new FragmentOne();
                }
                mFragmentTransaction.replace(R.id.fl_main,fraOne);
                mFragmentTransaction.addToBackStack(null);//添加fragment到返回栈
                mFragmentTransaction.commit();
                break;
            case R.id.btn_main_two:
                if(fraTwo==null){
                    fraTwo=new FragmentTwo();
                }
                mFragmentTransaction.replace(R.id.fl_main,fraTwo);
                mFragmentTransaction.addToBackStack(null);//添加fragment到返回栈
                mFragmentTransaction.commit();
                break;
            case R.id.btn_main_three:
                if(fraThree==null){
                    fraThree=new FragmentThree();
                }
                mFragmentTransaction.replace(R.id.fl_main,fraThree);
                mFragmentTransaction.addToBackStack(null);//添加fragment到返回栈
                mFragmentTransaction.commit();
                break;
        }
    }
```

# 源码地址：

