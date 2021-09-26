---
typora-copy-images-to: ./
typora-root-url: ./
---

# Makefile入门到精通

> 文档中例程下载地址：https://github.com/i-wanglei/makefile_pratice.git

## 1 什么是makefile

​		关于什么是makefile，大家可以查看百度，具体内容不在此过多描述，本篇博客主要带大家逐步完成makefile 的编写，从简到繁



## 2 单文件夹下makefile编写

### 2.1 目录结构与文件

​		单文件夹下的makefile是指需要编译的文件均在一个目录下，目录结构如下图所示

![image-20210926110955202](/image-20210926110955202.png)

​	各文件内的内容如下：

func1.c :

```c
#include <stdio.h>
#include <unistd.h>
#include "func1.h"

void fun1(void)
{
        printf("In func1: hello!\n");
}
```

func1.h :

```c
#ifndef __FUNC1_H_
#define __FUNC1_H_

void fun1(void);

#endif /* __FUNC1_H_ */
```

func2.c

```c
#include <stdio.h>
#include <unistd.h>
#include "func2.h"

void fun2(void)
{
        printf("In func2: hello!\n");
}
```

func2.h

```c
#ifndef __FUNC2_H_
#define __FUNC2_H_

void fun2(void);

#endif /* __FUNC2_H_ */
```

main.c

```c
#include <stdio.h>
#include <unistd.h>
#include "func1.h"
#include "func2.h"

int main()
{
        printf("hello word\n");
        fun1();
        fun2();

        return 0;
}
```



### 2.2 makefile编写

#### Version 1.0

```makefile
main : main.c func1.c func2.c
	gcc main.c func1.c func2.c -o main
```

​		采用直接编译的方式，直接将.c文件编译生成可执行文件，此方法特点是makefile编写简单，但是每一次改动其中任何一个文件都需要全部重新编译一次，当工程文件较大时，此方法明显不合理。



#### Version 2.0

```makefile
main : main.o func1.o func2.o
	gcc main.o func1.o func2.o -o main

main.o : main.c
	gcc -c main.c

func1.o : func1.c
	gcc -c func1.c

func2.o : func2.c
	gcc -c func2.c
```

​		将.c文件先编译为.o文件，之后将.o文件链接生成可执行文件，由于**makefile编译的原理**是比较目标与依赖之间的最后更新时间，如果目标文件的最后更新时间比依赖文件的最后更新时间早，则会执行相应的命令更新目标文件；反之，如果目标文件的最后更新时间比依赖文件的最后更新时间晚，则认为此文件不需要编译。

​		使用此方法修改编写makefile时，如果只修改了 func1.c 的内容，则执行make时只需要重新编译func.1文件，以及重新执行链接，而main.c和func2.c则不用重新编译，当工程文件比较大的时候，这将大大提高开发效率，有的工程全部编译一次就得半小时了，如果继续使用Verision 1.0的那种方式，就不用写程序了～

​		但是使用此方法编写makefile的时候，每增加一个.c文件，都需要增加一个对应的编译规则，文件少的时候还好，当文件多的时候，就不能接受了，作为一个程序猿，对于此类重复的工作你肯定是没有办法接受的！



#### Version 3.0

```makefile
obj := main.o func1.o func2.o
target := main
CC = gcc

$(target) : $(obj)
	$(CC) $^ -o $@

%.o : %.c
	$(CC) -c $< -o $@

clean:
	rm *.o $(target) -f
```

​		针对Version 2.0存在的问题，进行改进，加入变量以及自动化变量的使用（makefile变量怎么用查下百度吧）

注：自动化变量：

> $* 　　不包含扩展名的目标文件名称。
>
> $+ 　　所有的依赖文件，以空格分开，并以出现的先后为序，可能包含重复的依赖文件。
>
> $< 　　第一个依赖文件的名称。
>
> $? 　　 所有的依赖文件，以空格分开，这些依赖文件的修改日期比目标的创建日期晚。
>
> $@ 　   目标的完整名称。
>
> $^ 　    所有的依赖文件，以空格分开，不包含重复的依赖文件。
>
> $%       如果目标是归档成员，则该变量表示目标的归档成员名称。

​		除了引入变量以外，此处还针对Version 2.0需要重复编写对应.c文件的编译规则采用通配符匹配的处理方式，解决重复编写的苦恼

```makefile
%.o : %.c
	$(CC) -c $< -o $@
```

​		如当%为main时，对应为

%.o : %.c => main.o : main.c

$(CC) => gcc

$< => main.c

$@ => main.o

转化之后为

```makefile
main.o : main.c
    	gcc -c main.c -o main.o
```

​		至此，我们的makefile看上去是不是高级了许多，但是我们使用的时候发现，当我的工程每增加一个.c文件时，都需要修改一次makefile，在obj变量中增加对应.c文件的.o文件，这样每次都需要修改看上去还是不太智能，那么makefile能不能自己帮我们查找呢？答案时肯定的



#### Version 4.0

```makefile
src := $(wildcard ./*.c)
obj := $(patsubst %.c, %.o, $(src))
target := main
CC := gcc

$(target) : $(obj)
	$(CC) -o $@ $^
	
%.o : %.c
	$(CC) -c $^

debug :
	@echo $(src)
	@echo $(obj)

clean:
	rm *.o $(target) -f
```

​		此处我们引入wildcard和patsubst函数帮助我们解决Version3.0存在问题，让makefile实现自动查找目录下的.c文件，这样我们就不需要每次增减文件时都来修改makefile了

​		wildcard函数的作用如下：在Makefile规则中，通配符会被自动展开。但在变量的定义和函数引用时，通配符将失效。这种情况下如果需要通配符有效，就需要使用函数“wildcard”，在Makefile中，它被展开为已经存在的、使用空格分开的、匹配此模式的所有文件列表。

​		patsubst函数的作用如下：格式$(patsubst pattern,replacement,text)，查找text中的单词（单词以“空格”、“Tab”或“回车”“换行”分隔）是否符合模式pattern，如果匹配的话，则以replacement替换，这里，pattern可以包括通配符“%”，表示任意长度的字串。如果replacement中也包含“%”，那么，replacement中的这个“%”将是pattern中的那个“%”所代表的字串。

​		通过wildcard函数获取当前目录下的所有.c文件赋值给src变量，通过patsubst命令将src变量中所有.c文件转化为.o文件，再结合通配符、变量、自动化变量完成此makefile的编写，此方案可编译工程目录中的.c文件，当工程目录中的.c文件增加或删改时无需修改makefile，此外可通过输入make debug查看src和obj的值；通过输入make clean清除编译文件。

​		至此，针对单文件夹下的makefile已初步成熟，但是在实际开发过程中，我们往往会将文件按照类型进行分类，不会将所有文件存放在一个文件夹下，那么针对多文件夹的makefile又如何被编写呢？



## 3 多文件夹下makefile编写

​		多文件夹工程目录结构如下所示：

![image-20210926135557239](/image-20210926135557239.png)

​		./inc目录下存放所有.h头文件，./src目录下存放所有.c文件，./obj目录下存放所有编译生成的文件

​		针对此类工程我所知道的主要有两种形势的makefile，第一种是一个makefile解决整个工程的编译，第二种是每级目录下均设有一个makefile，负责完成所在目录下文件的编译，上层makefile通过调用下层makefile完成整个工程的编译。

​		此处展示第一种形势的makefile，通过一个makefile解决整个工程编译。此类工程的makefile如下所示：

```makefile
DIR_INC = ./inc
DIR_SRC = ./src
DIR_OBJ = ./obj

SRC := $(wildcard $(DIR_SRC)/*.c)
OBJ := $(patsubst %.c, ${DIR_OBJ}/%.o, $(notdir $(SRC)))
TARGET := main
CC := gcc
CFLAGS := -I $(DIR_INC)

$(TARGET) : $(OBJ)
	$(CC) $(OBJ) -o $@

$(DIR_OBJ)/%.o : $(DIR_SRC)/%.c
	$(CC) -c $< $(CFLAGS) -o $@

debug :
	@echo $(SRC)
	@echo $(OBJ)
	@echo $(CC) $(OBJ) -o $(TARGET)

clean :
	-rm $(TARGET) $(OBJ)
```

​		大家仔细阅读后可以发现，此版本与单文件夹makefile Version 4.0版本相似，只是在Version4.0的基础上将各文件增加了一级目录

