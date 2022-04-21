---
title: QButtonGroup简单使用
date: 2022-4-21
top: 1
tags:
	qt/c++
---
QButtonGroup简单使用
<!--more-->

```c++
    QButtonGroup *fileButton = new QButtonGroup(this);
    QPushButton *NewFileBtn = new QPushButton();
    QPushButton *UpdateFileBtn = new QPushButton();
    QPushButton *CopyFileBtn = new QPushButton();

    fileButton->addButton(ui->NewFileBtn,0);
    fileButton->addButton(ui->UpdateFileBtn,1);
    fileButton->addButton(ui->CopyFileBtn,2);

    connect(fileButton,SIGNAL(buttonClicked(int)),this,SLOT(btnSetSlot(int)));
	
    void btnSetSlot(int index)
    {
        /* --- */
    }
```