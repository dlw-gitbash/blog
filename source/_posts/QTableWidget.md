---
title: QTableWidget简单应用
date: 2022-4-15
top: 1
tags:
	qt/c++
---
QTableWidget简单应用
<!--more-->

```c++
QTableWidget *FilePageHandle = new QTableWidget();
int lineCnt = 0;    //初始化行数,不算标题行
int columnCnt = 6;  //列数
QStringList columID = {"图形名称" , "时序" , "电源" , "锁秒(ms)" , "AutoLock", "功能"};  //列标题
ui->FilePageHandle->setColumnCount(columnCnt);  //设置列数
ui->FilePageHandle->setRowCount(lineCnt);  //设置行数
ui->FilePageHandle->setHorizontalHeaderLabels(columID);  //写入列标题
ui->FilePageHandle->verticalHeader()->setVisible(false); // 隐藏水平header
ui->FilePageHandle->horizontalHeader()->setVisible(false); //隐藏行表头
ui->FilePageHandle->setSelectionMode(QAbstractItemView::SingleSelection); //设置为选中单个目标
ui->FilePageHandle->setSelectionBehavior(QAbstractItemView::SelectRows); //整行选中的方式
ui->FilePageHandle->setEditTriggers(QAbstractItemView::DoubleClicked);  //修改表格数据方式
/* 表格样式定制 */
ui->FilePageHandle->setColumnWidth(0,200); //列宽
ui->FilePageHandle->setFont(QFont("Helvetica"));  //设置单元格字体
ui->FilePageHandle->horizontalHeader()->setStretchLastSection(true);  //最后一列设置自适应宽度
ui->FilePageHandle->horizontalHeader()->setFixedHeight(30);//设置表头的高度
ui->FilePageHandle->horizontalHeader()->setStyleSheet("QHeaderView::section{background:white;color:black;}");  //表头颜色
/* 滑块样式 */
ui->FilePageHandle->horizontalScrollBar()->setStyleSheet(
	 "QScrollBar:horizontal{background:#DADADA;height:25px;margin-left:25px;margin-right:25px }"
	 "QScrollBar::handle:horizontal{background:lightgray;min-height:80px;height:25px}"
	 "QScrollBar::sub-line:horizontal{width:25px;subcontrol-position:left;subcontrol-origin:margin;}"
	 "QScrollBar::add-line:horizontal{width:25px;subcontrol-position:right;subcontrol-origin:margin;}"
	 );
ui->FilePageHandle->verticalScrollBar()->setStyleSheet(
	 "QScrollBar{background:#DADADA;width:25px;margin-top:25px;margin-bottom:25px}"
	 "QScrollBar::handle:vertical{background:lightgray;min-height:80px;width:25px}"
	 "QScrollBar::sub-line:vertical{height:25px;subcontrol-position:top;subcontrol-origin:margin;}"
	 "QScrollBar::add-line:vertical{height:25px;subcontrol-position:bottom;subcontrol-origin:margin;}"
	 );
```

