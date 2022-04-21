---
title: QLabel上画图片
date: 2022-4-15
top: 1
tags:
	qt/c++
---
QLabel上画图片
<!--more-->

```c++
QString picture = "../white.bmp";
QLabel *label = new QLabel();
QImage* image=new QImage();
if(!(image->load(picture))) //加载图像
{
    qDebug()<<"打开图片失败";
    delete image;
    return;
}
QSize LabelSize=ui->label->size();//要显示图片的label的大小
QImage imageNew = image->scaled(LabelSize,Qt::KeepAspectRatio);//重新调整图像大小以适应窗口
ui->label->setPixmap(QPixmap::fromImage(imageNew));
```