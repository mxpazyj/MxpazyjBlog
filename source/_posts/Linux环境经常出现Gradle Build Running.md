---
title: 在Linux环境下经常出现Gradle Build Running
date: 2016-12-01 12:39:30
tags: Android
---
### Android Studio 在Linux环境下经常出现Gradle Build Running

解决办法:

系统缺少某些依赖

`sudo apt-get install lib32z1`

`sudo apt-get install zlib1g`

`sudo apt-get install libc6-i386 lib32stdc++6 lib32gcc1 lib32ncurses5`
