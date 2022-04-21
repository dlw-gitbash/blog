---
title: makefile
date: 2021-9-4
tags:
	makefile
---
makefile
<!--more-->

# 1. 什么是makefile？
Makefile 可以简单的认为是一个工程文件的编译规则，描述了整个工程的编译和链接等规则。
文件命名为"GNUmakefile" 、"makefile" 、"Makefile" 均可以。
规则主要是两个部分组成，分别是依赖的关系和执行的命令：
```jsx
targets:prerequisites
	command
or
targets : prerequisites; command
	command
```
targets：规则的目标，可以是 Object File（一般称它为中间文件），也可以是可执行文件，还可以是一个标签；
prerequisites：是我们的依赖文件，要生成 targets 需要的文件或者是目标。可以是多个，也可以是没有；
command：make 需要执行的命令（任意的 shell 命令）。可以有多条命令，每一条命令占一行。
注:我们的目标和依赖文件之间要使用冒号分隔开，命令的开始一定要使用Tab键。
```jsx
eg:
test:test.c
	gcc -o test test.c
```

# 2. makefile的组成
简单的概括一下Makefile 中的内容，它主要包含有五个部分，分别是：
1) 显式规则
显式规则说明了，如何生成一个或多的的目标文件。这是由 Makefile 的书写者明显指出，要生成的文件，文件的依赖文件，生成的命令。
2) 隐晦规则
由于我们的 make 命名有自动推导的功能，所以隐晦的规则可以让我们比较粗糙地简略地书写Makefile，这是由 make 命令所支持的。
3) 变量的定义
在 Makefile 中我们要定义一系列的变量，变量一般都是字符串，这个有点像C语言中的宏，当 Makefile 被执行时，其中的变量都会被扩展到相应的引用位置上。
4) 文件指示
其包括了三个部分，一个是在一个 Makefile 中引用另一个 Makefile，就像C语言中的 include 一样；
另一个是指根据某些情况指定 Makefile 中的有效部分，就像C语言中的预编译 #if 一样；还有就是定义一个多行的命令。
5) 注释
Makefile 中只有行注释，和 UNIX 的 Shell 脚本一样，其注释是用“#”字符，这个就像 C/C++ 的“//”一样。如果你要在你的 Makefile 中使用“#”字符，可以用反斜框进行转义，如：“\#”。

# 3.通配符
- *
匹配0个或者是任意个字符
```jsx
eg:
	.PHONY:clean
	clean:
		rm -rf *.o test*
```
- ？
*匹配任意一个字符*
- []
*我们可以指定匹配的字符放在 "[]" 中*
- %
*和**类似
```jsx
eg:
	test:test.o test1.o
		gcc -o $@ $^
	%.o:%.c
		gcc -o $@ $^
```
::"%.o" 把我们需要的所有的 ".o" 文件组合成为一个列表，从列表中挨个取出的每一个文件。

# 4. 变量
- 定义
变量的名称=值列表：VALUE_LIST = one two three
- 使用
"$(VALUE_LIST)" or "${VALUE_LIST}"
```jsx
eg:
	OBJ=main.o test.o test1.o test2.o
	test:$(OBJ)
		gcc -o test $(OBJ)
```
- 四种基本赋值方式
简单赋值 ( := ) 编程语言中常规理解的赋值方式，只对当前语句的变量有效。
```jsx
eg:
	x:=foo
	y:=$(x)b
	x:=new
	test：
		@echo "y=>$(y)"
		@echo "x=>$(x)"
在 shell 命令行执行make test:
	y=>foob
	x=>new	
```
递归赋值 (  = ) 赋值语句可能影响多个变量，所有目标变量相关的其他变量都受影响。
```jsx
eg:
	x=foo
	y=$(x)b
	x=new
	test：
		@echo "y=>$(y)"
		@echo "x=>$(x)"
在 shell 命令行执行make test:	
	y=>newb
	x=>new
```
条件赋值 ( ?= ) 如果变量未定义，则使用符号中的值定义变量。如果该变量已经赋值，则该赋值语句无效。
```jsx
eg:
	x:=foo
	y:=$(x)b
	x?=new
	test：
		@echo "y=>$(y)"
		@echo "x=>$(x)"
	在 shell 命令行执行make test:
		y=>foob
		x=>foo
```
追加赋值 ( += ) 原变量用空格隔开的方式追加一个新值。
```jsx
eg:
	x:=foo
	y:=$(x)b
	x+=$(y)
	test：
		@echo "y=>$(y)"
		@echo "x=>$(x)"
	在 shell 命令行执行make test:
		y=>foob
		x=>foo foob
```

# 5. VPATH和vpath
- VPATH
VPATH 是变量，更具体的说是环境变量，Makefile 中的一种特殊变量，使用时需要指定文件的路径。
```jsx
eg:
	VPATH := src
	当存在多个路径：
	VPATH := src car or VPATH := src:car
```
- vpath
vpath 是关键字，按照模式搜索，也可以说成是选择搜索。搜索的时候不仅需要加上文件的路径，还需要加上相应限制的条件。
1) vpath PATTERN DIRECTORIES
```jsx
eg:
	vpath test.c src	#在 src 路径下搜索文件 test.c
	当存在多个路径：
	vpath test.c src car or vpath test.c src : car
	#PATTERN：可以理解为要寻找的条件，DIRECTORIES：寻找的路径
```
2) vpath PATTERN
```jsx
eg:
	vpath test.c	#清除符合文件 test.c 的搜索目录
```
3) vpath
```jsx
eg:
	vpath
	#vpath 单独使的意思是清除所有已被设置的文件搜索路径。
```

# 6. 函数

调用格式：$(<function>,<arguments>) or ${<function>,<arguments>}
function 是函数名，arguments 是函数的参数，参数之间要用逗号分隔开。而参数和函数名之间使用空格分开。