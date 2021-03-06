---
title: c/c++面经
date: 2021-9-4
top: 1
tags:
	面试
	c/c++
---
c/c++面经
<!--more-->

# 1. **在main执行之前和之后执行的代码可能是什么？**
main执行之前：主要是初始化系统相关资源
+ 设置栈指针；
+ 初始化静态static变量和global全局变量,即.data段的内容；
+ 将未初始化部分的全局变量赋初值：数值型short , int , long等为0 , bool为FALSE ,指针为NULL等等，即.bss段的内容；
+ 全局对象初始化,在main之前调用构造函数，这是可能会执行前的一些代码；
+ 将main函数的参数argc, argv等传递给main函数,然后才真正运行main函数；
+ ***ttribute***(constructor)。 

main函数执行之后
+ 全局对象的析构函数会在main函数之后执行；
+ 可以用atexit注册一个函数,它会在main之后执行；
+ ***attribute***(destructor)。

# 2. 结构体内存对齐问题？
- 结构体内成员按照声明顺序存储，第一个成员地址和整个结构体地址相同。
- 未特殊说明时,按结构体中size最大的成员对齐(若有double成员,按8字节对齐)。

# 3. 指针和引用的区别？
- 指针是一个变量，存储的是一个地址，引用跟原来的变量实质上是同一个东西,是原变量的别名；
- 指针可以有多级，引用只有一级；
- 指针可以为空，引用不能为NULL且在定义时必须初始化；
- 指针在初始化后可以改变指向，而引用在初始化之后不可再改变；
- sizeof指针得到的是本指针的大小，sizeof引用得到的是引用所指向变量的大小；
- 当把指针作为参数进行传递时，也是将实参的一个拷贝传递给形参，两者指向的地址相同,但不是同一个变量，在函数中改变这个变量的指向不影响实参，而引用却可以；
- 引用只是别名，不占用具体存储空间，只有声明没有定义；指针是具体变量,需要占用存储空间(但这也只是一般情况下,具体情况还要具体分析)；
- 引用在声明时必须初始化为另一变量，一旦出现必须为typename refname &varname形式；指针声明和定义可以分开，可以先只声明指针变量而不初始化，等用到时再指向具体变量；
- 引用一旦初始化之后就不可以再改变(变量可以被引用为多次，但引用只能作为一个变量引用)指针变量可以重新指向别的变量；
- 不存在指向空值的引用，必须有具体实体；但是存在指向空值的指针。

# 4. 堆和栈的区别？
- 申请方式不同
	1. 栈由系统自动分配释放；
	2. 堆由程序员手动申请释放。
- 申请大小限制不同
    1. 栈顶和栈底是之前预设好的，栈是向栈底扩展，大小固定，可以通过ulimit-a查看，由ulimit-s修改；
    2. 堆向高地址扩展，是不连续的内存区域，大小可以灵活调整。
- 申请效率不同
    1. 栈由系统分配，速度快，不会有碎片；
    2. 堆由程序员手动分配，速度慢，且会有碎片。
- 栈空间默认是4M；堆区一般是1-4G。

# 5. 指针？
- int *p[10]; ：数组指针，是一个数组变量，数组大小为10，组内每个元素都是指向int类型的指针变量；
- int (*)p[10];：数组指针，是一个指针类型变量，指向的是一个int类型的数组，这个数组的大小是10；
- int *p(int);：函数声明，函数名是p，参数是int类型的，返回值是int *类型的；
- int (*p)(int);：函数指针，指向的函数具有int类型参数，并且返回值是int类型的。

# 6. 基类的虚函数表存放在内存的什么区，虚表指针vptr的初始化时间？
首先整理一下虚函数表的特征：
- 虚函数表是全局共享的元素,即全局仅有一个,在编译时就构造完成；
- 虚函数表类似一个数组,类对象中存储vptr指针,指向虚函数表,即虚函数表不是函数,不是程序代码,不可能存储在代码段；
- 虚函数表存储虚函数的地址,即虚函数表的元素是指向类成员函数的指针,而类中虚函数的个数在编译时期可以确定,即虚函数表的大小可以确定,即大小是在编译时期确定的,不必动态分配内存空间存储虚函数表,所以不在堆中。
- 根据以上特征,虚函数表类似于类中静态成员变量静态成员变量也是全局共享,大小确定,因此最有可能存在全局数据区,测试结果显示：
1. 虚函数表vtable在Linux/Unix中存放在可执行文件的只读数据段中(rodata),这与微软的编译器将虚函数表存放在常量段存在一些差别；
2. 由于虚表指针yptr跟虚函数密不可分,对于有虚函数或者继承于拥有虚函数的基类,对该类进行实例化时,在构造函数执行时会对虚表指针进行初始化,并且存在对象内存布局的最前面。
一般分为五个区域：栈区、堆区、函数区(存放函数体等二进制代码)、全局静态区、常量区。
- C++中虚函数表位于只读数据段( .rodata)，也就是C++内存模型中的常量区;而虚函数则位于代码段(.text)，也就是C++内存模型中的代码区。

# 7. **new/delete**与**malloc/free？**
- 相同点：
	1.都可用于内存的动态申请和释放。
- 不同点：
	1.new/delete是c++运算符，malloc/free是c/c++语言标准库函数；
	2.new自动计算要分配的空间大小，malloc需要手工计算；
	3.new是类型安全的，malloc不是；
    ```c++
    int *p = new float[2];  //编译错误
    int *p = (int *)malloc(2 * sizeof(double)); //编译无错误
    ```
	4.new调用名为operator new的标准库函数分配足够空间并调用相关对象的构造函数，delete堆指针所指对象运行适当的析构函数，然后提供调用名为operator delete的标准库函数释放该对象所用内存，malloc/free无相关调用；
	5.malloc/free需要库文件支持，new/delete不需要；
	6.new是封装了malloc，直接free不会报错，但只是释放内存，而不会析构对象。

# 8. new和delete的实现？
- new的实现过程是：首先调用名为operator new的标准库函数，分配足多大的原始为类型化的内存，以保存指定类型的一个对象；接下来运行该类型的一个构造函数，用指定初始化构造对象；最后返回指向新分配井构造后的的对象的指针；
- delete的实现过程：对指针指向的对象运行适当的析构函数；然后通过调用名为operator delete的标准库函数释放该对象所用内存。

# 9. malloc和new的区别?
- malloc和free是标准库函数，支持覆盖；new和delete是运算符，并且支持重载；
- malloc仅仅分配内存空间，free仅仅回收空间，不具备调用构造函数和析构函数功能，用malloc分配空间存储类的对象存在风险；new和delete除了分配回收功能外，还会调用构造函数和析构函数；
- mallac和free返回的是void类型指针(必须进行类型转换)， new和delete返回的是具体类型指针。
***delete和delete[]的区别？***
- delete只会调用一次析构函数；
- delete[]会调用数组中每个元素的析构函数。

# 10. 宏定义和函数？
- 宏在编译时完成替换，之后被替换的文本参与编译，相当于直接插入了代码，运行时不存在函数调用，执行起来更快；函数调用在运行时需要跳转到具体函数中调用函数；
- 宏定义属于在结构中插入代码，没有返回值；函数调用具有返回值；
- 宏定义参数没有类型，不进行类型检查；函数参数具有类型，需要进行类型检查；
- 宏定义不需要在最后加分号。

# 11. 宏定义和typedef？
- 宏定义主要用于定义常量及书写复杂的内容；typedef主要用于定义类型别名；
- 宏替换发生在编译阶段之前，属于文本插入替换；typedef时编译的一部分；
- 宏不进行类型检查；typedef会检查数据类型；
- 宏不是语句，不需要在最后加分号；typedef是语句，要加分号标识结束；
- 对于指针操作时：
	```c++
    #define p_char char *：
    p_char p1,p2; //p1类型为char *，p2类型为char。
    typedef char * p_char：
    p_char p1,p2; //p1,p2类型均为char *。
	```

# 12. 变量声明和定义？
- 声明仅仅是把变量的声明问位置及类型提供给编译器，并不分配内存空间；定义要在定义的地方为其分配存储空间；
- 相同变量可以在多处声明(外部变量extern)，但定义只能在一处定义。

# 13. 必须用到初始化成员列表的情况？
- 初始化一个reference成员；
- 初始化一个const成员；
- 调用一个基类的构造函数，而该函数有一组参数；
>通俗的就是，如果基类没有构造函数，派生类的初始化列表必须初始化基类，必须用基类的构造函数构造一个。因为派生类继承基类时，会先构造基类，然后在构造派生类，如果基类不是默认的构造函数，那么就不知道基类到底咋构造，因此……
- 调用一个数据成员对象的构造函数，而该函数有一组参数。
>列表初始化的顺序和书写顺序无关，只和变量的声明顺序有关。先声明哪个，就先初始化构造哪个。

# 14. sizeof()和strlen()？
- sizeof()是运算符，并不是函数，结果在编译时就已经得到，strlen()是字符串处理的库函数；
- sizeof()参数可以是任何数据的类型或者数据(sizeof参数不退化)；strlen()的参数只能是字符指针且结尾是'\0'的字符串；
- sizeof()的值在编译时确定，所以不能用来得到动态分配(运行时分配)存储空间大小。
	```c++
	const char *str = "name";
	sizeof(str); //取的是指针str的长度，结果是4
	strlen(str); //取得是这个字符串的长度，不包含结尾的'\0',结果是4
	```

# 15. 常量指针和指针常量？
- 常量指针：指向常量的指针，顾名思义，就是指针指向的是常量，即，它不能指向变量，它指向的内容不能被改变，不能通过指针来修改它指向的内容，但是指针自身不是常量，它自身的值可以改变，从而可以指向另一个常量，如int const *p或const int *p；
- 指针常量：指针本身是常量。它指向的地址是不可改变的，但地址里的内容可以通过指针改变。它指向的地址将伴其一生，直到生命周期结束。有一点需要注意的是，指针常量在定义时必须同时赋初值，如int *const p。

# 16. a和&a？
```c++
int a[10];
int (*p)[10] = &a;
```
- a是数组名，是数组首元素的地址，a+1表示地址值加1个int类型的大小，如果a的地址是0x00000001,+1后操作后变为0x00000005，*(a+1) = a[1]；
- &a是数组的指针，其类型为int(*)10] (就是前面提到的数组指针) ，其加1时,系统会认为是数组首地址加上整个数组的偏移(10个int型变量)，值为数组a尾元素后一个元素的地址。
- 若(int *)p，此时输出*p时，其值为a[0]的值，因为被转为int *类型，解引用时按照int类型大小来读取。

# 17. 数组名和指针？
- 二者均可通过增减偏移量来访问数组中的元素；
- 数组名不是真正意义上的指针，可以理解为常指针，所以数组名没有自增、自减等操作；
- 当数组名当中形参传给调用函数后，就是去了原有的特性，退化成一般指针，多了自增、自检操作，但sizeof运算符不能再得到原数组的大小了。

# 18. 野指针和悬空指针？
都是指向无效内存区域(无效指"不安全不可控")的指针，访问行为将会导致未定义行为。
**野指针：**指的是没有被初始化或的指针
	```c++
	int *p; //未初始化
	std::cout<<*p<<endl; //访问野指针
	```
为了防止出错，对于指针初始化时都是赋值为NULL，这样在访问时编译器会直接报"非法内存访问"的错误；
**悬空指针：**指针最初指向的内存已被释放了的一种指针
	```c++
	int *p1 = NULL;
	int *p2 = new int;
	p1 = p2;
	delete p2;
	```
此时p1和p2就是悬空指针，指向的内存已被释放，继续使用这两个指针，行为不可预测，需要设置p1=p2=NULL。
**产生原因及解决办法：**
1. 野指针：指针变量未及时初始化⇒定义指针变量及时初始化；
2. 悬空指针：指针free或者delete之后没有及时置空⇒释放后立即置空。
3. c和c++？
    - C++中new和delete是对内存分配的运算符，取代了C中的malloc和free；
    - 标准C++中的字符串类取代了标准C函数库头文件中的字符数组处理函数(C中没有字符串类型)；
    - C++中用来做控制态输入输出的iostream类库替代了标准C中的stdio函教库；
    - C++中的try/catch/throw异常处理机制取代了标准C中的setjmp()和longimp()函数；
    - 在C++中，允许有相同的函数名，不过它们的参数类型不能完全相同，这样这些函数就可以相互区别开来。而这在C语言中是不允许的。也就是C++可以重载，C语言不允许；
    - C++语言中，允许变量定义语句在程序中的任何地方，只要在是使用它之前就可以；而C语言中,必须要在函数开头部分。而且C++允许重复定义变量，C语言也是做不到这一点的；
    - 在C++中，除了值和指针之外，新增了引用，引用型变量是其他变量的一个别名，我们可以认为他们只是名字不相同，其他都是相同的；
    - C++相对与C增加了一些关键字，如: bool, using, namespace等等。

# 19. c++中struct和class？
**相同点：**
- 两者都拥有成员函数、公有和私有部分；
- 任何可以使用class完成的工作，同样可以用struct完成。
**不同点：**
- struct默认是公有的，class默认是私有的；
- struct默认是public继承，class默认是private继承；
- class可以作为模板类型，struct不行。
**c++和c的struct区别：**
- c语言中，struct是用户自定义数据类型(UDT)；c++中struct是抽象数据类型(ADT)，支持成员函数的定义，c++中的struct能继承，能实现多态；
- c中struct是没有设置权限的设置的，且struct中只能是一些变量的集合体，可以封装数据却不可以隐藏数据，而且成员不可以是函数；
- c++中，struct增加了访问权限，且可以和类一样有成员函数，成员默认访问说明符为public(为了与c兼容)；
- struct作为类的一种特例是用来自定义数据结构的。一个结构标记声明后，在c中必须在结构标记前加上struct才能做结构类型名(除：typedef struct class{};)；c++中结构体标记(结构体名)可以直接作为结构体类型名使用，此外结构体struct在c++中被当作类的一种特例。

# 20. define宏定义和const？
**编译阶段：**
- define是在编译阶段的预处理阶段起作用，而const是在编译、运行的时候起作用；
**安全性：**
- define只做替换，不做类型检查和计算，也不求解，容易产生错误，一般最好加上大括号包含住全部的内容，不然容易出错；
- const常量有数据类型，编译器可以对其进行类型安全检查；
**内存占用：**
- define只是将宏名称进行替换，在内存中会产生多份相同的备份；const在程序运行中只有一份备份，且可以执行常量折叠，能将复杂的表达式计算出结果放入常量表；
- 宏替换发生在编译阶段之前，属于文本插入替换；const作用发生于编译过程中；
- 宏不检查类型；const会检查数据类型；
- 宏定义的数据没有分配内存空间，只是插入替换掉；从上图定义的变量只是值不能改变，但要分配内存空间。

# 21. c++中static？
**不考虑类的情况**
1. 隐藏。所有不加static的全局变量和函数具有全局可见性，可以在其它文件中使用，加了之后只能在该文件所在的编译模块中使用；
2. 默认初始话为0，包括未初始化的全局静态变量与局部静态变量，都存在全局未初始化区；
3. 静态变量在函数内定义，始终存在，且只进行一次初始化，具有记忆性，其作用范围与局部变量相同，函数退出后仍然存在，但不能使用；
**考虑类的情况**
1. static成员变量：只与类关联，不与类的对象关联。定义时要分配空间，不能在类声明中初始化，必须在类定义体外部初始化，初始化时不需要标识为static；可以被非static成员函数任意访问；
2. static成员函数：不具有this指针，无法访问类对象的非static成员变量和非static成员函数；不能被声明为const，虚函数和volatile；可以被非static成员函数任意访问。

# 22. c++中const？
**不考虑类的情况**
1. const常量在定义时必须初始化，之后无法更改；
2. const形参可以接收const和非const类型的实参；
**考虑类的情况**
1. const成员变量：不能在类定义外部初始化，只能通过构造函数初始化列表进行初始化，并且必须有构造函数；不同类对其const数据成员的值可以不同，所以不能在类中声明时初始化；
2. const成员函数：const对象不可以调用非const成员函数；非const对象都可以调用；不可以改变非mutable(用该关键字声明的变量可以在const成员函数中被修改)数据的值。

# 23. 模板函数和模板类的特例话
**引入原因：**
编写单一的模板，它能适应多种类型的需求，使每种类型都具有相同的功能，但对于某种特定类型，如果要实现其特有的功能，单一模板就无法做到，这时就需要模板特例化。
**定义：**
对单一模板提供的一个特殊实例，它将一个或多个模板参数绑定到特定的类型或值上。
1. 模板函数特例化
    必须为原函数模板的每个模板参数都提供实参，且使用关键字template后跟一个空尖括号对<>，表明将原模板的所有模板参数提供实参，举例如下：
    ```c++
    template<typename T> //模板函数
    int compare(const T &vl, const T &v2)
    {
        if(v1 > v2) return -1;
        if(v2 > v1) return 1;
        return 0;
    }
    //模板特例化,满足针对字符申特定的比较,要提供所有实参,这里只有一个T
    template<>
    int compare(const char* const &v1, const char* const &v2) 
    {
        return strcmp(p1, p2);
    }
    ```
    **本质：**
    本质是实例化一个模板，而非重载它。特例化不影响参数匹配。参数匹配都以最佳匹配为原则，例如，此处如果是compare(3,5)，则调用普通的模板，若为compare(hi",haha")，则调用特例化版本(因为这个`cosnt char*`相对于T，更匹配实参类型)，注意二者函数体的语句不一样了，实现不同功能。
    **注意：**
    模板及其特例化版本应该声明在同一个头文件中，且所有同名模板的声明应该放在前面，后面放特例化版本。
2. 类模板特例化
    原理类似函数模板，不过在类中，我们可以对模板进行特例化，也可以对类进行部分特例化。对类进行特例化时，仍然用template表示是一个特例化版本，例如：
    ```c++
    template<>
    class hash<sales_data>
    {
        size_t operator()(sales_data& s);
        //里面所有T都换成特例化类型版本sales_data
        /* 按照最佳匹配原则,若T != sales_data,就用普通类模板,否则,就使用含有特定功能的
        特例化版本。 */
    }
    ```
**类模板的部分特例化**
不必为所有模板参教提供实参，可以指定一部分而非所有模板参数，一个类模板的部分特例化本身仍是一个模板，使用它时还必须为其特例化版本中未指定的模板参数提供实参(特例化时类名一定要和原来的模板相同，只是参数类型不同，按最佳匹配原则，哪个最匹配，就用相应的模板)
**特例化类中的部分成员**
可以特例话类中的部分成员函数而不是整个类，举个例子：
	```c++
	template<typename T> 
	class Foo 
	{
		void BarO();
		void Barst(T a) ();
	};
	template<>
	void Foocint::Bar()
	{
		//1进行int类型的特例化处理
		cout <<"我是int型特例化"<<end1;
	}
	Foo<string> fs;
	Foo<int> fi;//使用特例化
	fs. BarO);//使用的是普通模板,即Foo<string>::BarO
	fi.BarO);//特例化版本,执行Foo(int : :BarO)
	//Foo<string>:: Bar()和Foocintx::Bar()功能不同
	```

# 24. c和c++的类型安全？
**什么是类型安全？**
类型安全很大程度上可以等价于内存安全，类型安全的代码不会试图访问自己没被授权的内存区域。"类型安全"常被用来形容编程语言，其根据在于该门编程语言是否提供保障类型安全的机制；有的时候也用"类型安全"形容某个程序，判别的标准在于该程序是否隐含类型错误，类型安全的编程语言与类型安全的程序之间没有必然联系。好的程序员可以使用类型不那么安全的语言写出类型相当安全的程序，相反的，差一点儿的程序员可能使用类型相当安全的语言写出类型不太安全的程序，绝对类型安全的编程语言暂时还没有。
1. c的类型安全
    c只在局部上下文中表现出类型安全，比如试图从一种结构体的指针转换为另外一种结构体的指针时，编译器将会报错，除非使用显示类型转换。
    - printf格式输出
    ```c++
	printf("整型数:%d\n",10);
	printf("浮点型数:%f\n",10);
	```
    上述代码中，使用%d控制整型数字的输出，没有问题，但是改成%时，明显输出错误，再改成%s时，运行直接报segmentation fault错误。
    - malloc函数的返回值
    malloc是C中进行内存分配的函数，它的返回类型是void *，即空类型指针,常常有这样的用法char *p = (char *)malloc(100 * sizeof(char));这里明显做了显式的类型转换。类型匹配尚且没有问题,但是一旦出现int* p=(int*)malloc(100*sizeof(char));就很可能带来一些问题,而这样的转换C并不会提示错误。。
2. c++的类型安全
    如果c++使用得当，它将远比c更有类型安全性。相比于c语言， c++提供了一些新的机制保障类型安全：
    - 操作符new返回的指针类型严格与对象匹配，而不是void *；
    - c中很多以*void **为参数的函数可以改写为c++模板函数，而模板是支持类型检查的；
    - 引入const关键字代替#define constants，它是有类型、有作用域的，而#define constants只是简单的文本替换；
    - 一些#define宏可被改写为inline函数，结合函数的重载，可在类型安全的前提下支持多种类型，当然改写为模板也能保证类型安全；
    - c++提供了dynamic_cast关键字，使得转换过程更加安全，因为dynamic_cast比static_cast涉及更多具体的类型检查。

# 25. 大小端存储？
大端存储：字数据的高字节存储在低地址中；
小端存储：字数据的低字节存储在低地址中。
**两种判断大小端存储的方法**
1. 使用强制类型转换
    ```c++
    #include <iostream> 
    using namespace std;

    int main()
    {
        int a = 0x1234;
        //亩于int和char的长度不同,借助int型转换成char型,只会留下低地址的部分
        char c = (char)a;
        if (C == 0x12)
            cout << "big endian" << endl;
        else if(C == 0x34)
            cout << "little endian" << endl;

        return 0;
    }
    ```
2. 巧用union联合体
    ```c++
    #include <iostream>
    using namespace std;
    //union联合体的重叠式存储, endian联合体占用内存的空间为每个成员字节长度的最大值
    union endian
    {
        int a;
        char ch;
    };

    int main()
    {
        endian value;
        value.a =0x1234;
        //a和ch共用4字节的内存空间
        if (value.ch ==0x12)
            cout << "big endian" << endl;
        else if (value.ch == 0x34)
            cout << "Tittle endian" << endl;
        
        return 0;
    }
    ```

# 26. c++内存分区？
**堆**
在执行函数时，函数内局部变量的存储单元都可以在栈上创建，函数执行结束时这些存储单元自动被释放。栈内存分配运算内置于处理器的指令集中，效率很高，但是分配的内存容量有限；
**栈**
就是那些由new分配的内存块，他们的释放编译器不去管，由我们的应用程序去控制，一般一个new就要对应一个delete。如果程序员没有释放掉，那么在程序结束后，操作系统会自动回收；
**自由存储区**
就是那些由malloc等分配的内存块，它和堆是十分相似的，不过它是用free来结束自己的生命的；
**全局/静态存储区**
全局变量和静态变量被分配到同一块内存中，在以前的C语言中，全局变量和静态变量又分为初始化的和未初始化的，在c++里面没有这个区分了，它们共同占用同一块内存区，在该区定义的变量若没有初始化，则会被自动初始化，例如int型变量自动初始为0；
**常量存储区**
这是一块比较特殊的存储区,，这里面存放的是常量，不允许修改；
**代码段**
存放函数体的二进制代码

# 27. 形参和实参？
- 形参变量只有在被调用时才分配内存单元，在调用结束时，即刻释放所分配的内存单元。因此，形参只有在函数内部有效。函数调用结束返回主调函数后则不能再使用该形参变量；
- 实参可以是常量、变量、表达式、函数等，无论实参是何种类型的量，在进行函数调用时，它们都必须具有确定的值，以便把这些值传送给形参。因此应预先用赋值，输入等办法使实参获得确定值，会产生一个临时变量；
- 实参和形参在数量上、类型上、顺序上应严格一致，否则会发生"类型不匹配"的错误；
- 函数调用中发生的数据传送是单向的。即只能把实参的值传送给形参，而不能把形参的值反向地传送给实参。因此在函数调用过程中，形参的值发生改变，而实参中的值不会变化；
- 当形参和实参不是指针类型时，在该函数运行时，形参和实参是不同的变量，他们在内存中位于不同的位置，形参将实参的内容复制一份，在该函数运行结束的时候形参被释放，而实参内容不会改变。

# 28. 值传递、指针传递、引用传递的区别和效率？
- 值传递：有一个形参向函数所属的栈拷贝数据的过程，如果值传递的对象是类对象或是大的结构体对象，将耗费一定的时间和空间(传值)。
- 指针传递：同样有一个形参向函数所属的栈拷贝数据的过程，但拷贝的数据是一个固定为4字节的地址(传值,传递的是地址值)；
- 引用传递：同样有上述的数据拷贝过程，但其是针对地址的，相当于为该数据所在的地址起了一个别名(传地址)；
- 效率上讲，指针传递和引用传递比值传递效率高。一般主张使用引用传递，代码逻辑上更加紧凑、清晰。

# 29. 深拷贝与浅拷贝？
- 浅复制：只是拷贝了基本类型的数据,，而引用类型数据，复制后也是会发生引用，我们把这种拷贝叫做" (浅复制)浅拷贝" ，换句话说，浅复制仅仅是指向被复制的内存地址，如果原地址中对象被改变了，那么浅复制出来的对象也会相应改变；
    深复制：在计算机中开辟了一块新的内存地址用于存放复制的对象；
- 在某些状况下，类内成员变量需要动态开辟堆内存，如果实行位拷贝，也就是把对象里的值完全复制给另一个对象，如A-B，这时，如果B中有一个成员变量指针已经申请了内存，那A中的那个成员变量也指向同一块内存。这就出现了问题：当B把内存释放了(如：析构) ，这时A内的指针就是野指针了出现运行错误。

# 30. 智能指针？
**原理**
智能指针是一个类，用来存储指向动态分配对象的指针，负责自动释放动态分配的对象，防止堆内存泄露。交给一个类对象区管理，当类对象生命周期结束时，自动调用析构函数释放资源。
**常用的智能指针**
1. **shared_ptr**
    实现原理：**采用引用计数器的方法**，允许多个智能指针指向同一个对象，每当多一个指针指向该对象时，指向该对象的所有智能指针内部的引用计数加1，每当减少一个智能指针指向对象时，引用计数会减1，当计数为0的时候会自动释放动态分配的资源。
    - 智能指针将一个计数器与类指向的对象相关联，引用计数器跟踪共有多少个类对象共享同一指针；
    - 每次创建类的新对象时，初始化指针并将引用计数置为1；
    - 当对象作为另一对象的副本而创建时，拷贝构造函数拷贝指针并增加与之相应的引用计数；
    - 对一个对象进行赋值时，赋值操作符减少左操作数所指对象的引用计数（如果引用计数为减至0，则删除对象），并增加右操作数所指对象的引用计数；
    - 调用析构函数时，构造函数减少引用计数（如果引用计数减至0，则删除基础对象）。
2. **unique_pt**
    unique_ptr采用的是独享所有权语义，一个非空的unique_ptr总是拥有它所指向的资源。转移一个unique_ptr将会把所有权全部从源指针转移给目标指针，源指针被置空；所以**unique_ptr不支持普通的拷贝和赋值操作**，不能用在STL标准容器中；局部变量的返回值除外（因为编译器知道要返回的对象将要被销毁）；如果你拷贝一个unique_ptr，那么拷贝结束后，这两个unique_ptr都会指向相同的资源，造成在结束时对同一内存指针多次释放而导致程序崩溃。
3. **weak_ptr**
    weak_ptr：弱引用。 引用计数有一个问题就是互相引用形成环（环形引用），这样两个指针指向的内存都无法释放。需要使用weak_ptr打破环形引用。**weak_ptr是一个弱引用，它是为了配合shared_ptr而引入的一种智能指针**，它指向一个由shared_ptr管理的对象而不影响所指对象的生命周期，也就是说，它只引用，不计数。如果一块内存被shared_ptr和weak_ptr同时引用，当所有shared_ptr析构了之后，不管还有没有weak_ptr引用该内存，内存也会被释放。所以weak_ptr不保证它指向的内存一定是有效的，在使用之前使用函数lock()检查weak_ptr是否为空指针。
4. **auto_ptr**
    **auto_ptr不支持拷贝和赋值操作**，不能用在STL标准容器中。STL容器中的元素经常要支持拷贝、赋值操作，在这过程中auto_ptr会传递所有权，auto_ptr采用的是独享所有权语义，一个非空的unique_ptr总是拥有它所指向的资源。转移一个auto_ptr将会把所有权全部从源指针转移给目标指针，源指针被置空。
**智能指针代码实现：** 用两个类来实现智能指针的功能，一个是引用计数类，另一个则是指针类。
**引用计数类**
	```c++
	//引用计数器类用于存储指向同一对象的指针数
	template<typename T>
	class Counter
	{
	private:
		//数据成员
		T *ptr;//对象指针
		int cnt; // 引用计数器1
		//友元类声明
		template<typename T>
		friend class SmartPtr;
		//构造函数
		Counter(T *p)
		{
			ptr =p;
			cnt = 1;
		}
		//折构函数
		~Counter()
		{
			delete ptr;
		}
	}
	```
**指针类**
	```c++
	template<typename T>
	class SmartPtr
	{
	private:
		//数据成员
		Counter<T> ptr_cnt;
	public:
		//普通构造函数 初始化计数类
		SmartPtr(T *p)
		{
			ptr_cnt = new Counter<T>(p);
		}
		//拷贝构造函数
		SmartPtr(const SmartPtr &other)
		{
			ptr_cnt = other.ptr_cnt;
			ptr_cnt->cnt++;
		}
		//赋值运算符重载函数
		SmartPtr &operator=(const SmartPtr &rhs)
		{
			ptr_cnt = rhs->ptr_cnt;
			rhs.ptr cnt->cnt++;
			ptr_cnt->cnt--;
			if (ptr_cnt->cnt == 0)
				delete ptr_cnt;
			return *this;
		}
		//解引用运算符重载函数
		T &operator*()
		{
			return *(ptr_cnt->cnt);
		}
		//析构函数
		~SmartPtr()
		{
			ptr_cnt->cnt--;
			if(ptr_cnt->cnt == 0){
				delete ptr_cnt;
			}else{
				cout << "还有" << ptr_cnt->cnt << "个指针指向基础对象" << endl;
			}
		}
	}
	```

# 31. 函数指针？
**定义：**函数指针是指向函数的指针变量。 因此“函数指针”本身首先应是指针变量，只不过该指针变量指向函数。这正如用指针变量可指向整型变量、字符型、数组一样，这里是指向函数。
**有两个用途：**调用函数和做函数的参数。
**声明方法：**
返回值类型 (* 指针变量名) (形参列表);
typedef int (*fun)(int, int);  // 声明一个指向同样参数、返回值的函数指针类型
typedef int(fun)(int, int);  //声明一种函数类型
**注1：**“返回值类型”说明函数的返回类型，“(指针变量名 )”中的括号不能省，括号改变了运算符的优先级。若省略整体则成为一个函数说明，说明了一个返回的数据类型是指针的函数，后面的“形参列表”表示指针变量指向的函数所带的参数列表。                         **注2：**函数括号中的形参可有可无，视情况而定。
**指向函数的指针变量没有++和--运算。**

# 32. c/c++语言编译四步骤？
**预编译**
主要处理源代码文件中的以“#”开头的预编译指令。处理规则见下：
1. 删除所有的#define，展开所有的宏定义；
2. 处理所有的条件预编译指令，如“#if”、“#endif”、“#ifdef”、“#elif”和“#else”；
3. 处理“#include”预编译指令，将文件内容替换到它的位置，这个过程是递归进行的，文件中包含其他文件；
4. 删除所有的注释，“//”和“/**/”；
5. 保留所有的#pragma 编译器指令，编译器需要用到他们，如：#pragma once 是为了防止有文件被重复引用；
6. 添加行号和文件标识，便于编译时编译器产生调试用的行号信息，和编译时产生编译错误或警告是能够显示行号。
**编译**
把预编译之后生成的.i或.ii文件，进行一系列词法分析、语法分析、语义分析及优化后，生成相应的汇编代码文件。
1. 词法分析：利用类似于“有限状态机”的算法，将源代码程序输入到扫描机中，将其中的字符序列分割成一系列的记号；
2. 语法分析：语法分析器对由扫描器产生的记号，进行语法分析，产生语法树。由语法分析器输出的语法树是一种以表达式为节点的树；
3. 语义分析：语法分析器只是完成了对表达式语法层面的分析，语义分析器则对表达式是否有意义进行判断，其分析的语义是静态语义——在编译期能分期的语义，相对应的动态语义是在运行期才能确定的语义；
4. 优化：源代码级别的一个优化过程；
5. 目标代码生成：由代码生成器将中间代码转换成目标机器代码，生成一系列的代码序列——汇编语言表示；
6. 目标代码优化：目标代码优化器对上述的目标机器代码进行优化：寻找合适的寻址方式、使用位移来替代乘法运算、删除多余的指令等。
**汇编**
将汇编代码转变成机器可以执行的指令(机器码文件)。 汇编器的汇编过程相对于编译器来说更简单，没有复杂的语法，也没有语义，更不需要做指令优化，只是根据汇编指令和机器指令的对照表一一翻译过来，汇编过程由汇编器as完成。经汇编之后，产生目标文件(与可执行文件格式几乎一样).o(Windows下)、.obj(Linux下)。
**链接**
将不同的源文件产生的目标文件进行链接，从而形成一个可以执行的程序。链接分为静态链接和动态链接。
1. 静态链接
    函数和数据被编译进一个二进制文件。在使用静态库的情况下，在编译链接可执行文件时，链接器从库中复制这些函数和数据并把它们和应用程序的其它模块组合起来创建最终的可执行文件。
    空间浪费：因为每个可执行程序中对所有需要的目标文件都要有一份副本，所以如果多个程序对同一个目标文件都有依赖，会出现同一个目标文件都在内存存在多个副本；
    更新困难：每当库函数的代码修改了，这个时候就需要重新进行编译链接形成可执行程序。
    运行速度快：但是静态链接的优点就是，在可执行程序中已经具备了所有执行程序所需要的任何东西，在执行的时候运行速度快。
2. 动态链接
    动态链接的基本思想是把程序按照模块拆分成各个相对独立部分，在程序运行时才将它们链接在一起形成一个完整的程序，而不是像静态链接一样把所有程序模块都链接成一个单独的可执行文件。
    共享库：就是即使需要每个程序都依赖同一个库，但是该库不会像静态链接那样在内存中存在多分，副本，而是这多个程序在执行时共享同一份副本；
    更新方便：更新时只需要替换原来的目标文件，而无需将所有的程序再重新链接一遍。当程序下一次运行时，新版本的目标文件会被自动加载到内存并且链接起来，程序就完成了升级的目标。
    性能损耗：因为把链接推迟到了程序运行时，所以每次执行程序都需要进行链接，所以性能会有一定损失。