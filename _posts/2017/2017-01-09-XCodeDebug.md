---
layout: post

title: Xcode调试技巧

date: 2017-01-09 15:32:24.000000000 +09:00

---


## 打印或者打印内存地址的真实数据

### 打印
```C
int *retIntArray(void){
    int *array = (int *)malloc(sizeof(int)*4);
    array[0] = 0;
    array[1] = 1;
    array[2] = 2;
    array[3] = 3;
    printf("array : %p \n",array);// array : 0x1004002e0
    return array;
}

Consloe：
(lldb) x/4du array
0x1003002e0: 0
0x1003002e4: 1
0x1003002e8: 2
0x1003002ec: 3

```
[在 Xcode 调试时查看内存中的数据](http://www.samirchen.com/xcode-debug-memory-data/)

### 查看
直接在Xcode中查看
![](/assets/images/WX20170408-181942@2x.png)

### 碰见问题

```C
void testPoint()
{
    int *p = retIntArray();
    printf("p : %p \n",p);
    for (int i = 0 ; i < 4; i++) {
        printf("%d\n",p[i]);
    }
}
consloe
array : 0x100300330 
p : 0x300330 
```
retIntArray里面打印的要返回的指针和接受的 `*p` 居然不一样。
原因由于 int *retIntArray(void) 仅在.c文件中实现了，并没有放到.h头文件中声明。而C语言默认返回int, 相当于本来返回的是指针，但被默认转化成了int，失去了精度，再转换回来也就错了。


