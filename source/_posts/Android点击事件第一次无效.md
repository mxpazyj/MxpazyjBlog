---
title: Android点击事件，第一次无效
date: 2016-12-02 13:00:30
tags: Android
---
### Android点击事件，第一次无效

碰到的问题，设置点击事件之后，第一次无效，第二次才能触发

究其原因是因为之前测试selecter的时候加入了

`android:focusable="true"`

`android:focusableInTouchMode="true"`

将android:focusableInTouchMode改为false之后，问题解决。

`android:focusableInTouchMode`

是否通过touch来获取聚焦，若为true，第一次是获取焦点，第二次才相应click事件，为false，则直接响应。
