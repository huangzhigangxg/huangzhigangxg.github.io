---
layout: post

title: 目标文件里有什么

date: 2017-02-25 15:32:24.000000000 +09:00

---

### 1 目标文件的格式
目标文件就是源代码编译后但未经过连接的中间文件  *Linux下的Func.o* 
目标文件距离可执行文件只是还没有经过连接过程，其中可能有写符号和地址还没有被调整。
但是目标文件和可执行文件它们格式几乎是一样的，甚至是动态链接库。统称为ELF文件。

### 2 目标文件是什么样的
头文件File Header  描述是否可执行，是静态连接还是动态链接，入口地址，目标硬件，操作系统
段表 Section Table 描述了各个段的属性 *(.data .text .bss)* 和偏移量 

![](/assets/images/WX20170225-173002@2x.png){: height="200px"}

将段表分为 代码段 *只读* 和 数据段 *读写* 的好处是 

+ 可以防止被无意更改
+ CPU处理指令缓存时命中率更高
+ 多个程序时共享代码段

### 3 挖掘 SimpleSection.o
![](/assets/images/WX20170226-113257@2x.png){: height="200px"}
##### 3.1 代码段
代码段保存的是程序代码。
![](/assets/images/WX20170226-114529@2x.png){: height="200px"}
##### 3.2 数据段和只读段
.data 段保存的是 *已经初始化* 的全局静态变量和局部变量。
.rodata 段 *read only data*  存放只读数据 *const* 和 常量字符串。
![](/assets/images/WX20170226-114959@2x.png){: height="200px"}
.data段前面的4个字节 0x54 0x00 0x00 0x00 这正是global_init_varabal 十进制的84.
##### 3.3 BSS段
.bss  段存放的是 *未初始化*的全局变量和局部静态变量
##### 3.4 其他段
![](/assets/images/WX20170226-115624@2x.png){: height="200px"}
### 4 ELF文件结构描述
这个ELF大概可以看成这样
![](/assets/images/WX20170226-115836@2x.png){: height="200px"}
其中两个重要的结构就是 ELF Header 和 Section Header Table 它们分别描述里整个ELF的组成信息
##### 4.1 头文件
ELF Header 描述里ELF文件的基本属性，目标机器型号，程序入口地址。
![](/assets/images/WX20170226-120708@2x.png){: height="200px"}
![](/assets/images/WX20170226-121402@2x.png){: height="200px"}
![](/assets/images/WX20170226-121038@2x.png){: height="200px"}
##### 4.2 段表
Section Header Table 描述了 ELF中所有段的 名字，长度，偏移，读写权限等。编译器，连接器，装载器都是依靠段表来定位和访问段的属性的。
打印出结果如下;
![](/assets/images/WX20170226-122621@2x.png){: height="200px"}
这其实是一个数组，其中的元素为叫 *Elf32_Shdr* 的结构体。
![](/assets/images/WX20170226-121945@2x.png){: height="200px"}
分析这个数组就能得到全部的ELF文件信息。
##### 4.3 重定位表
段表中有.rel.text 或者 .rel.data 的表示 引用了尚未确定的函数地址 *printf* 等 需要重定位。
Relocation Text。
##### 4.4 字符串表
这里记录了将用到的字符串都组织了起来。用数字下标引用即可
### 5 链接的接口----符号
所有的函数和变量名在链接的时候都会转化成符号。
##### 5.1 ELF符号表结构
![](/assets/images/WX20170226-123303@2x.png){: height="200px"}
##### 5.2 特殊符号
__executable_start  程序起始地址
__etext 代码段结束地址
_end 程序结束地址
##### 5.3 符号修饰与函数签名
未了避免不同模块引用相同的全局变量名或者全局函数名发生冲突，编译器在符号化这些变量和函数时需要做函数符号的修饰和签名，函数签名的目的是为了识别不同的函数。
C语言将 *foo* 函数 简单转化为 *_foo*
C++则引入更强大的命名空间 ：
```
 int func(int);  
 float func(float);
 class C {
 	int func(int)
 	class C2 {
 		int func(int);
 	}
 }
 namespace N{
 	class C {
 		int func(int);
 	}
 }
```
![](/assets/images/WX20170226-124417@2x.png){: height="200px"}

##### 5.4 extern“C”
C++和C语言的符号化的方式差异如此巨大，那么如何使编译器实现C和C++混编呢？
extern "C" 告诉编译器 这里的函数用C语言的符号签名方法。因为如果编译器在编译C++的代码时，里面引用了C语言的函数库 string.h 里的函数 void *memset（void *，int，size_t）;  他会按着C++
的符号化修饰方法对memset函数进行签名，当最后链接C语言标准库时，就找不到了对应的方法。所以要将C++代码中的C语言函数和变量都要用extern“C”来保护起来。
![](/assets/images/WX20170226-125726@2x.png){: height="200px"}
##### 5.5 弱符号与强符号
当相同的全局变量函数或者变量名编译时发生冲突时，就是它们都互为强符号。可以通过修饰 __attribute__((week)) 来修饰避免冲突。
### 6 调试信息
在我们生产的目标文件中还可能保存调试信息，我们可以设置断点，监控变量的变化的前提都是这些调试信息说明类源代码和目标代码之间的关系，比如目标代码中的函数地址对应源代码中的哪一行等。
现在的ELF文件采用一个叫做 DWARF(Debug With Arbitrary Record Format)的标准的调试信息格式。有了这个基础 ，再加上Xcode的entitlements 里面声明向ios系统内核申请查看其他进程的权限标签。就能用Xcode调试自己的程序进程运行时的信息了。








参考：[程序员的自我修养]