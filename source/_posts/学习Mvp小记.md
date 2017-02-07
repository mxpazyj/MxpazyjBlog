---
title: 学习Mvp小记
date: 2017-01-05 09:39:30
tags: Android
---
## 学习Android MVP小记
简单来说MVP架构就是从一个activity中抽离出三个层级

1. Model
2. View
3. Presenter

MVP感觉最让初学者难以搞懂的就是Activity跟这几个层级的关系，下面我就来理一理这其中的关系

![MVP](https://www.processon.com/chart_image/584a4891e4b0d594ec6ae702.png)

有了这个图我们就很好说了




在google官方MVP demo里面基本上所有的view都是由fragment继承的,
那如果我们是一个单activity的界面呢?
有人说就由activity继承，但是个人觉得不太好，我感觉正确的做法应该是将activity的根View抽离出来，单独包装成一个View！

值得跟初学者一说的是，这里的View不是我们一般所理解的控件View，这里的View是作为一个控制UI展示的一个类。

下面上代码，实现一个登陆验证的功能
思考一下
1. View实现的功能，提供的接口
2. Presenter实现的功能，提供的接口

##### `MainContract` 存放View Presneter Model接口的类

Presenter需要提供给View输入接口 就是commit
View需要提供给Presenter 清除界面数据接口

这里还多了一个setPresenter()的方法
其实这个方法应该抽到View的基类里面去，但是为了字面上理解，先不抽基类

```java
public interface MainContract {
    interface View{
        void setPresenter(Presenter presenter);
        void clearEditText();
    }

    interface Presenter {
        boolean commit(String userName, String passWord);
    }
}
```



##### `MainActivity.java`

这里的Activity的工作变得非常简单只初始化了Presenter传入了根部View
```java
public class MainActivity extends AppCompatActivity {
    private MainPresenter mMainPresenter;
    private MainView mMainView;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        mMainPresenter = new MainPresenter(getWindow().getDecorView().findViewById(android.R.id.content));

    }
}
```
##### `MainPresenter.java`
这里的MainPresenter初始化了View继承了Main.Presenter实现了commit接口
```java
public class MainPresenter implements MainContract.Presenter {

    private MainView mMainView;

    public MainPresenter(View rootView){
        mMainView = new MainView(rootView);
        mMainView.setPresenter(this);
    }

    @Override
    public boolean commit(String userName, String passWord) {
        if(userName.equals("123456") && passWord.equals("123456")){
            return true;
        }else {
            return false;
        }
    }
}
```

##### `MainView.java`

```java
public class MainView implements MainContract.View, View.OnClickListener {
    private View rootView;
    private EditText mUserNameEt;
    private EditText mPassWordEt;
    private Button mCommitBtn;
    private MainContract.Presenter mainPresenter;

    public MainView(View rootView) {
        this.rootView = rootView;
        mUserNameEt = (EditText) rootView.findViewById(R.id.main_username_et);
        mPassWordEt = (EditText) rootView.findViewById(R.id.main_password_et);
        mCommitBtn = (Button) rootView.findViewById(R.id.main_commit_btn);
        mCommitBtn.setOnClickListener(this);
    }

    @Override
    public void setPresenter(MainContract.Presenter presenter) {
        this.mainPresenter = presenter;
    }

    @Override
    public void clearEditText() {
        mUserNameEt.setText("");
        mUserNameEt.setText("");
    }

    @Override
    public void onClick(View v) {
        if(TextUtils.isEmpty(mUserNameEt.getText().toString().trim()) ||
                TextUtils.isEmpty(mPassWordEt.getText().toString().trim())){
            Toast.makeText(rootView.getContext(), "不能为空", Toast.LENGTH_SHORT).show();
            return;
        }
        if(mainPresenter.commit(
                mUserNameEt.getText().toString().trim(),mUserNameEt.getText().toString().trim())){
            Toast.makeText(rootView.getContext(), "成功", Toast.LENGTH_SHORT).show();

        }else {
            Toast.makeText(rootView.getContext(), "失败", Toast.LENGTH_SHORT).show();

        }

    }
}
```
