---
title: qt窗口屏蔽
date: 2022-4-15
top: 1
tags:
	qt/c++
---
qt窗口屏蔽
<!--more-->

```c++
//置顶：
Qt::WindowFlags flag = this->windowFlags();
this->setWindowFlags(flag | Qt::CustomizeWindowHint | Qt::WindowStaysOnTopHint);
//禁止其他窗口响应事件
this->setWindowModality(Qt::ApplicationModal);
```