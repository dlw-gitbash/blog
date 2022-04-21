---
title: QTableWidget单元格插入控件
date: 2022-4-15
top: 1
tags:
	qt/c++
---
QTableWidget单元格插入控件
<!--more-->

以插入QCheckBox为例:
```c++
QTableWidget *FilePageHandle = new QTableWidget();
QCheckBox *lockCheck = new QCheckBox(); 
//加入布局
QWidget *widget = new QWidget();;
QHBoxLayout *hLayout = new QHBoxLayout();;
hLayout->addWidget(lockCheck);
hLayout->setMargin(0); // 必须添加, 否则CheckBox不能正常显示
hLayout->setAlignment(lockCheck, Qt::AlignCenter);
widget->setLayout(hLayout);  
ui->FilePageHandle->setCellWidget(0,1-1,widget);  //0行1列插入QCheckBox
```