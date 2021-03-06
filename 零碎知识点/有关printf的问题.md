# `printf()`函数学习

## 一.简介

​		在将结果输出到文件中修改成输出到控制台(`fwrite`→`printf`)，速度上慢了很多。在使用`printf`之后使用重定向输入到文件(> file),速度也慢。

## 二.问题分析

​		1.`printf()`如何实现变参的？

​		2.为什么`printf()`函数慢？

​		3.有什么办法加快输出？

​		解决思路：通过查询`MSDN`，源码，stack overflow。

## 三.问题解决

```
int printf(const char *fmt, ...)
{
	char printf_buf[1024];
	va_list args;
	int printed;
	va_start(args, fmt);
	printed = vsprintf(printf_buf, fmt, args);
	va_end(args);

	puts(printf_buf);

	return printed;
}
```

1.在我们实现可变参数之前，先得搞清楚函数是怎样传参的。

**C的函数参数入栈遵照`__stdcall`规则, 它是从右到左的，即函数中的参数入栈是从右到左的。**

例如：

```
#include<iostream>
#include<vector>

void test(char zNum1, int nNum2, double fNum3, char* pzNum4)
{
    printf("zNum1:%#p\n"
        "nNum2:%#p\n"
        "fNum3:%#p\n"
        "pzNum4:%#p",
        &zNum1, &nNum2, &fNum3, pzNum4
    );
}

int main()
{
    char zNum4;
    test('a', 12, 23.4, &zNum4);
    return 0;
}
输出结果
zNum1:006FF8E8
nNum2:006FF8EC
fNum3:006FF8F0
pzNum4:006FF9CF
```

从各个形参变量的地址可以看出它们地址大小确实是从右到左依次减小的，说明它们是从右到左压栈的。

对于固定参数列表的函数，每个参数的名称、类型都是直接可见的，他们的地址也都是可以直接得到的，比如：通过&`zNum1`就可以得到`zNum1`的地址，并通过函数原型声明了解到`zNum1`是char类型的。

但对于变长参数的函数该怎么办呢？思考一下函数传参的过程，无论...中有多少个参数,每个参数是什么类型的，它们都和固定参数的传参过程是一样的，简单来讲都是栈操作，同时**C标准的说明中，是支持变长参数的函数在原型声明中的，但须至少有一个最左固定参数.**再知道了某函数帧的栈上的一个固定参数的位置后，我们完全可以自己通过栈操作，推导出其他变长参数的位置，进而实现可变参数函数。

2.`printf`是行缓冲函数，先写到缓冲区，满足条件后，才刷新缓冲区(将输出从缓冲区发送到屏幕或文件称为刷新缓冲区).

`printf`刷新缓冲区条件：

​	1.缓冲区写满(缓冲区大小为1024`byte`).

​	2.`fflush`手动刷新缓冲区

​	3.执行`printf`的进程或者线程结束时会主动调用`fflush`刷新缓冲区

​	4.遇到换行符\n \r(不一定正确，输出设备是非交互式(磁盘，管道等)， 则输出不一定是行缓冲).

​	5.当有即将到来的输入(不一定正确，取决于输入输出是否交互).

3.可以扩大`stdout`缓冲区的大小，这样子再同样大小的数据量下，减少了程序刷新缓冲区的次数。

## 四.总结

​	在输出时，最好在每次将数据输出到文件时清空缓冲区，这样即使程序异常中断数据也不会丢失，但这样也会导致速度变慢。







