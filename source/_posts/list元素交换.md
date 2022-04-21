---
title: std::list元素交换
date: 2022-4-15
top: 1
tags:
	c/c++
---
std::list元素交换
<!--more-->

```c++
template<typename T>
void listSwap(std::list<T> &list,int select,int target) 
{
    //定义第三方变量
    std::list<T>::iterator sIt = list.begin();
    for(int j =0;j<select;j++,sIt++){}
    std::list<T>::iterator tIt = list.begin();
    for(int j =0;j<target;j++,tIt++){}
     //交换值
    std::iter_swap(sIt,tIt);
}
```