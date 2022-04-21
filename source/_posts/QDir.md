---
title: QDir使用笔记
date: 2022-4-21
top: 1
tags:
	qt/c++
---
QDir使用笔记
<!--more-->

```c++
QStringList GraphicsFileMap

startsWith()  //检测字符串是否以某个特定的串开始
endsWith()    //检测字符串是否以某个特定的串结尾

//遍历图片目录
void ATS_UI_Process::findPictList(const QString picPath)
{
    QDir dir(picPath);  //图片文件路径
    if (!dir.exists()) {
        return;
    }
    //取到所有的文件和文件名，但是去掉.和..的文件夹（这是QT默认有的）
    dir.setFilter(QDir::Dirs | QDir::Files | QDir::NoDotAndDotDot);
    //排序,文件夹优先
    dir.setSorting(QDir::DirsFirst);
    //遍历路径下目录
    QFileInfoList list = dir.entryInfoList();
    //遍历文件
    QStringList infolist = dir.entryList(QDir::Files | QDir::NoDotAndDotDot);
    int i = 0;
    int listSize = list.size();
    //递归
    do {
        QFileInfo fileInfo = list.at(i);
        //如果是文件夹，递归
        if (fileInfo.isDir()) {
            findPictList(fileInfo.filePath());
        }
        else {
            /* 检索 */
            int size = infolist.size();
            for (int cnt = 0; cnt < size; ++cnt) {
                QString it = infolist[cnt];
                if (it.endsWith(".BMP") || it.endsWith(".JPG") || it.endsWith(".GIF") || it.endsWith(".bmp") || it.endsWith(".jpg") || it.endsWith(".gif")) {
                    infolist[cnt] = picPath + "//" + it;  //添加图片路径
                }
                else {
                    infolist.removeAt(cnt);
                }
            }
            /* 去重 */
            for (auto sIt : GraphicsFileMap) {
                QFileInfo sInfo(sIt);
                for (auto tIt : infolist) {
                    QFileInfo tInfo(tIt);
                    if (sInfo.baseName() == tInfo.baseName()) {
                        infolist.removeOne(tIt);
                    }
                }
            }
            /* 追加图片列表 */
            GraphicsFileMap += infolist;
            break;  //遍历完当前路径文件后结束
        }
        i++;
    } while (i < listSize);
}
```