# 编译器与Makefile

[TOC]



## gcc/g++/clang

> 了解c/c++编译器的基本使用，能够在后续移植第三方框架进行交叉编译时，清楚的了解应该传递什么参数。

### clang

> clang 是一个`C、C++、Object-C`的轻量级编译器。基于`LLVM` （LLVM是以C++编写而成的构架编译器的框架系统，可以说是一个用于开发编译器相关的库）

### gcc

> `GNU C`编译器。原本只能处理`C语言`，很快扩展，变得可处理`C++`。(GNU计划，又称革奴计划。目标是创建一套完全自由的操作系统)

### g++

> `GNU c++`编译器



> gcc、g++、clang都是编译器。
>
> - gcc和g++都能够编译c/c++，但是编译时候行为不同。
>
> - clang也是一个编译器，对比gcc，它具有编译速度更快、编译产出更小等优点，但是某些软件在使用clang编译时候因为源码中内容的问题会出现错误。
>
> - clang++与clang就相当于gcc与g++。



对于gcc与g++：

1. 后缀为`.c`的源文件，gcc把它当作是C程序，而g++当作是C++程序；后缀为`.cpp`的，两者都会认为是c++程序

2. g++会自动链接c++标准库stl，gcc不会

3. gcc不会定义__cplusplus宏，而g++会

   

> `apt install build-essential` #安装gcc、g++与make

   

### 编译器过程

> 一个C/C++文件要经过预处理(preprocessing)、编译(compilation)、汇编(assembly)、和连接(linking)才能变成可执行文件。

1、**预处理**

​	 gcc -E main.c  -o main.i 

​	 `-E`的作用是让gcc在预处理结束后停止编译。

​	预处理阶段主要处理include和define等。它把#include包含进来的.h 文件插入到#include所在的位置，把源程序中使用到的用#define定义的宏用实际的字符串代替

2、**编译阶段**

​	gcc -S main.i -o main.s

​	 `-S`的作用是编译后结束，编译生成了汇编文件。

​	在这个阶段中，gcc首先要检查代码的规范性、是否有语法错误等，以确定代码的实际要做的工作，在检查无误后，gcc把代码翻译成汇编语言。

3、**汇编阶段**

​	gcc -c main.s -o main.o

​	汇编阶段把 .s文件翻译成二进制机器指令文件.o,这个阶段接收.c, .i, .s的文件都没有问题。

4、**链接阶段**

​	gcc -o main main.s

​	链接阶段，链接的是**函数库**。在main.c中并没有定义”printf”的函数实现，且在预编译中包含进的”stdio.h”中也只有该函数的声明。系统把这些函数实现都被做到名为`libc.so`的动态库。



> `/etc/ld.so.conf`：
> 	在没有特别指定时，gcc会到系统编译器只会使用/lib和/usr/lib这两个目录下的库文件。如果存在一个so不在这两个目录，在编译时候就会出现找不到的情况。
>
> `/etc/ld.so.conf`文件中可以指定而外的编译链接库路径。
>
> 输入：`cat /etc/ld.so.conf`：

```shell
include /etc/ld.so.conf.d/*.conf #引入其他的conf文件
/usr/local/lib	#增加库搜索目录

#编辑完成后 使用 ldconfig 更新
```



**函数库一般分为静态库和动态库两种**

- 静态库是指编译链接时，把库文件的代码全部加入到可执行文件中，因此生成的文件比较大，但在运行时也就不再需要库文件了。Linux中后缀名为”.a”。
- 动态库与之相反，在编译链接时并没有把库文件的代码加入到可执行文件中，而是在程序执行时由运行时链接文件加载库。Linux中后缀名为”.so”，如前面所述的libc.so就是动态库。gcc在编译时默认使用动态库。

> 静态库节省时间:不需要再进行动态链接，需要调用的代码直接就在代码内部
>
> 动态库节省空间:如果一个动态库被两个程序调用,那么这个动态库只需要在内存中
>
>  Java中在不经过封装的情况下只能直接使用动态库。



```shell
#生成静态库
# -fPIC 产生与位置无关代码 
#可能会被不同的进程加载到不同的位置上，如果共享对象中的指令使用了绝对地址。那么在共享对象被加载时就必须根据相关模块的加载位置对这个地址做调整，也就是修改这些地址，让它在对应进程中能正确访问，那么就不能实现多进程共享一份物理内存(无法动态共享)
gcc -fPIC -c  Test.c -o Test.o
ar r libTest.a Test.o 

#生成动态库
gcc -fPIC -shared Test.c -o libTest.so
#或者
gcc -fPIC -c Test.c  #生成.o
gcc -shared Test.o -o libTest.so

#使用库
#默认优先使用动态库
gcc main.c -L. -lTest -o main
#强制使用静态库
#-Wl 表示传递给 ld 链接器的参数
#最后的 -Bdynamic 表示 默认仍然使用动态库 
gcc main.c -L. -Wl,-Bstatic  -lTest -Wl,-Bdynamic -o main
#使用动态库链接的程序，linux运行需要将动态库路径加入/etc/ld.so.conf
#mac(dylib)和windows(dll)可以将动态库和程序(main)放在一起
#mac 中gcc会被默认链接到xcode的llvm，不支持上面的指定链接动态库

#查看可执行文件符号
nm main

#打包 .a 到so
#--whole-archive: 将未使用的静态库符号(函数实现)也链接进动态库 
#--no-whole-archive : 默认，未使用不链接进入动态库
gcc -shared -o libMain.so -Wl,--whole-archive libMain.a -Wl,--no-whole-archive
```

头文件与库文件指定
```shell
--sysroot=XX
	使用xx作为这一次编译的头文件与库文件的查找目录，查找下面的 usr/include usr/lib目录
-isysroot XX
	头文件查找目录,覆盖--sysroot ，查找 XX/usr/include
-isystem XX
	指定头文件查找路径（直接查找根目录）
-IXX
	头文件查找目录
优先级：
	-I -> -isystem -> sysroot
	
-LXX
	指定库文件查找目录
-lxx.so
	指定需要链接的库名

#pie 位置无关的可执行程序
export CC="/root/android-ndk-r17b/toolchains/arm-linux-androideabi-4.9/prebuilt/linux-x86_64/bin/arm-linux-androideabi-gcc"
export CFLAGS="--sysroot=/root/android-ndk-r17b/platforms/android-21/arch-arm -isysroot /root/android-ndk-r17b/sysroot -isystem /root/android-ndk-r17b/sysroot/usr/include/arm-linux-androideabi -pie"	
$CC $CFLAGS -o main main.c
```



## Makefile

> android的Android.mk就是一段段Makefile单元，很多第三方库直接提供makefile，需要能够大致的读懂makefile文件,如增量更新的bspath库提供的makefile就有错误，需要修改。另外虽说现在google推荐使用cmake，但是如果遇见Android.mk还是需要能够读懂。

### 什么是Makefile	

​	无论是c、c++首先要把源文件编译成中间代码文件，在Windows下也就是 .obj 文件，UNIX下是 .o 文件，即 Object File，这个动作叫做``编译（compile）``，然后再把大量的Object File合成执行文件或者静动态库，这个动作叫作``链接（link）``。

一个工程中的源文件不计数，其按类型、功能、模块分别放在若干个目录中，makefile定义了一系列的规则来指定，哪些文件需要先编译，哪些文件需要后编译，如何进行链接等等操作。

makefile 就是“自动化编译”，告诉make命令如何编译和链接,即make工具的配置脚本。

默认的情况下，gnu make命令会在当前目录下按顺序找寻文件

`"GNUmakefile”、“makefile”、“Makefile”`
最好不要用“GNUmakefile”，这个文件是GNU的make识别的（Windows Nmake就不识别）
        

​	 当然，也可以使用别的文件名来书写Makefile，比如：“Make.Linux”，“Make.android”。这样在使用时候就需要 `` make -f XX 或者 make --file XX。``



### Makefile规则

```makefile
在Makefile中的命令，必须要以[Tab]键开始。

target : prerequisites ...(预备知识，先决条件)
	command（指令）
-----------------------------------------------------------------------------------------
      target也就是一个目标文件，可以是Object File，也可以是执行文件。还可以是一个标签。
      prerequisites 要生成那个target所需要的文件或是其他target。
      command也就是make需要执行的命令。（任意的Shell命令）
```

```makefile
# g++ -o  指定生成可执行文件的名称
# g++ -c 只编译不链接 
test:main.o T1.o
	g++ -o test main.o T1.o
#=============================================
#make会进行自动推导生成 main.o T1.o 可以不需要写这一部分
main.o:main.cpp T1.h
	g++ -c main.cpp
T1.o:T1.cpp T1.h
	g++ -c T1.cpp
#=============================================

#当然也可以一步到位 \ 是换行连接符 便于Makefile的易读,不用都挤在一行
test2:
	g++ -o test2 main.cpp \
T1.cpp
#=============================================

clean:
	rm test main.o T1.o
say:
	echo "112232"

# 读取到第一个targer test，则test是终极目标
# 生成目标test ，test需要main.o T1.0
# 查找到存在目标main.o T1.o 先生成 最后生成test

#clean和say是标签，并不生成“clean”这个文件，这样的target称之为 “伪目标”
#伪目标的名字不能和文件名重复
#如： 当前目录下有一个文件/文件夹 名字为clean,运行make clean则会：
# 		make: `clean' is up to date.
#为了避免这种情况,可以使用一个特殊的标记“.PHONY”来显示地指明一个目标是“伪目标”
.PHONY: clean
clean:
	rm test main.o T1.o
```



### 变量

```makefile
#如果比较复杂的情况，比如文件很多，target目标比较多，那么我们如果来修改，比如增加一个.cpp文件，
#那可能需要在很多地方都写一下，也容易出错。为了易于维护，可以在makefile中使用变量。
#相当于c中的宏

#声明变量
objects=main.o T1.o
#mac上自动编译 main.o
test:${objects}
	g++ -o test ${objects}

clean:
	rm test ${objects}
	
#=============================================
# *.c 表示所有后缀为c的文件。
# 让通配符在变量中(当前目录下所有 .c 文件)
objects = $(wildcard *.c)
test : ${objects}
        gcc -o test ${objects}
clean :
        rm test ${obects}
```



### include

```makefile
test:main.o T1.o
	g++ -o test main.o T1.o

#include make.clean
#或者
mk=make.clean
include ${mk}

#如果没有指定相对或者绝对路径，make会先在当前目录查找。
#然后查找 -I 或者 --include-dir 参数 {make -I include/ clean}
#如果存在 /usr/local/bin或/usr/include 目录 也会去这个目录找

#如果想让make不理那些无法读取的文件，而继续执行，你可以在include前加一个减号“-”
# -include ${mk}
#类似的 rm  g++ 等命令前都可以加-,表示不会因为这里命令执行错误而中断make。
```



### 文件搜索

在一些大的工程中，有大量的源文件存放在不同的目录中,最好的方法是把一个路径告诉make，让make在自动去找。
Makefile文件中的特殊变量``VPATH``就是完成这个功能的

```makefile
#默认先查找当前目录再查找当前目录下的a、b、c目录
VPATH = a:b:c
OBJ=a.o b.o c.o main.o

test : $(OBJ)
        gcc -o  test  $(OBJ)
clean :
        rm test $(OBJ)
        
#=============================================   
#小写的vpath是关键字  %.c表示所有.c文件
vpath %.c a
vpath %.c b:c
OBJ=a.o b.o c.o main.o

test : $(OBJ)
        gcc -o  test  $(OBJ)
clean :
        rm test $(OBJ)
```



### 其他

#### 预定义变量

| 命令变量 | 含义                              |
| -------- | --------------------------------- |
| AR       | 函数库的打包程序，默认为"ar"      |
| AS       | 汇编语言编译程序,默认为"as"       |
| CC       | C语言编译程序,默认命令是"cc"      |
| CXX      | C++语言编译程序,默认命令是"g++"   |
| RM       | 文件删除程序的名称,默认值为 rm –f |
| ARFLAGS  | 库文件维护程序的选项,无默认值     |
| ASFLAGS  | 汇编程序的选项,无默认值           |
| CFLAGS   | C 编译器的选项,无默认值           |
| CPPFLAGS | C 预编译的选项,无默认值           |
| CXXFLAGS | C++编译器的选项,无默认值          |

#### 自动变量

> $@  ``target的名字``  

```makefile
main.o:main.c
	gcc -c main.c -o main.o
#使用 $@ 代替 main.o
main.o:main.c
	gcc -c main.c -o $@
```

> $<  ``target依赖的第一个依赖文件名``

```makefile
main.o:main.c a.h b.h
	gcc -c main.c -o main.o
#使用 $< 代替 main.c
main.o:main.c a.h b.h
	gcc -c $< -o main.o
```

| 变量 | 说明                                                         |
| ---- | ------------------------------------------------------------ |
| $*   | 不包含扩展名的target文件名称                                 |
| $+   | 所有的依赖文件,以空格分开,并以出现的先后为序,可能包含 重复的依赖文件 |
| $?   | 所有时间戳比target文件晚的依赖文件,并以空格分开              |
| $^   | 所有不重复的依赖文件,以空格分开                              |

#### 条件语句

```makefile
#ifneq 
ifeq ($(CC),gcc)
        $(CC) -o foo $(objects) $(libs_for_gcc)
else
        $(CC) -o foo $(objects) $(normal_libs)
endif
```

#### 输出信息

```makefile
AAA=111
#输出变量AAA
$(warning $(AAA))
$(info $(AAA))
```



### Android.mk

微小 GNU makefile 片段。

将源文件分组为*模块*。 模块是静态库、共享库或独立可执行文件。 可在每个 `Android.mk` 文件中定义一个或多个模块，也可在多个模块中使用同一个源文件。 

```makefile
#源文件在的位置。宏函数 my-dir 返回当前目录（包含 Android.mk 文件本身的目录）的路径。
LOCAL_PATH := $(call my-dir)
#引入其他makefile文件。CLEAR_VARS 变量指向特殊 GNU Makefile，可为您清除许多 LOCAL_XXX 变量
#不会清理 LOCAL_PATH 变量
include $(CLEAR_VARS)
#存储您要构建的模块的名称 每个模块名称必须唯一，且不含任何空格
#如果模块名称的开头已是 lib，则构建系统不会附加额外的前缀 lib；而是按原样采用模块名称，并添加 .so 扩展名。
LOCAL_MODULE := hello-jni
#包含要构建到模块中的 C 和/或 C++ 源文件列表 以空格分开
LOCAL_SRC_FILES := hello-jni.c
#构建动态库
include $(BUILD_SHARED_LIBRARY)
```



**变量和宏**

定义自己的任意变量。在定义变量时请注意，NDK 构建系统会预留以下变量名称：

- 以 `LOCAL_` 开头的名称，例如 `LOCAL_MODULE`。
- 以 `PRIVATE_`、`NDK_` 或 `APP` 开头的名称。构建系统在内部使用这些变量。
- 小写名称，例如 `my-dir`。构建系统也是在内部使用这些变量。

如果为了方便而需要在 `Android.mk` 文件中定义自己的变量，建议在名称前附加 `MY_`。



**常用内置变量**

| 变量名                  | 含义                       | 示例                               |
| ----------------------- | -------------------------- | ---------------------------------- |
| BUILD_STATIC_LIBRARY    | 构建静态库的Makefile脚本   | include $(BUILD_STATIC_LIBRARY)    |
| PREBUILT_SHARED_LIBRARY | 预编译共享库的Makeifle脚本 | include $(PREBUILT_SHARED_LIBRARY) |
| PREBUILT_STATIC_LIBRARY | 预编译静态库的Makeifle脚本 | include $(PREBUILT_STATIC_LIBRARY) |
| TARGET_PLATFORM         | Android API 级别号         | TARGET_PLATFORM := android-22      |
| TARGET_ARCH             | CUP架构                    | arm arm64 x86 x86_64               |
| TARGET_ARCH_ABI         | CPU架构                    | armeabi  armeabi-v7a  arm64-v8a    |



**模块描述变量**

| 变量名                       | 描述                                   | 例                                                     |
| ---------------------------- | -------------------------------------- | ------------------------------------------------------ |
| LOCAL_MODULE_FILENAME        | 覆盖构建系统默认用于其生成的文件的名称 | LOCAL_MODULE := foo LOCAL_MODULE_FILENAME := libnewfoo |
| LOCAL_CPP_FEATURES           | 特定 C++ 功能                          | 支持异常:LOCAL_CPP_FEATURES := exceptions              |
| LOCAL_C_INCLUDES             | 头文件目录查找路径                     | LOCAL_C_INCLUDES := $(LOCAL_PATH)/include              |
| LOCAL_CFLAGS                 | 构建 C *和* C++ 的编译参数             |                                                        |
| LOCAL_CPPFLAGS               | c++                                    |                                                        |
| LOCAL_STATIC_LIBRARIES       | 当前模块依赖的静态库模块列表           |                                                        |
| LOCAL_SHARED_LIBRARIES       |                                        |                                                        |
| LOCAL_WHOLE_STATIC_LIBRARIES | --whole-archive                        | 将未使用的函数符号也加入编译进入这个模块               |
| LOCAL_LDLIBS                 | 依赖 系统库                            | LOCAL_LDLIBS := -lz                                    |

导出给引入模块的模块使用：

LOCAL_EXPORT_CFLAGS

LOCAL_EXPORT_CPPFLAGS

LOCAL_EXPORT_C_INCLUDES

LOCAL_EXPORT_LDLIBS

------

**引入其他模块**

```makefile
#将一个新的路径加入NDK_MODULE_PATH变量
#NDK_MODULE_PATH 变量是系统环境变量

$(call import-add-path,$(LOCAL_PATH)/platform/third_party/android/prebuilt)
#包含CocosDenshion/android目录下的mk文件
$(call import-module,CocosDenshion/android)

#这里即为 我需要引入 CocosDenshion/android 下面的Android.mk
#CocosDenshion/android 的路径会从 $(LOCAL_PATH)/platform/third_party/android/prebuilt 去查找
```



### Application.mk

同样是GNU Makefile 片段,在Application.mk中定义一些全局(整个项目)的配置



**APP_OPTIM**

将此可选变量定义为 `release` 或 `debug`。在构建应用的模块时可使用它来更改优化级别。发行模式是默认模式，可生成高度优化的二进制文件。调试模式会生成未优化的二进制文件，更容易调试。



**APP_CFLAGS**

为任何模块编译任何 C 或 C++ 源代码时传递到编译器的一组 C 编译器标志



**APP_CPPFLAGS**

构建 C++ 源文件时传递到编译器的一组 C++ 编译器标志。



**APP_ABI**

需要生成的cpu架构(ndk r17 只支持：armeabi-v7a, arm64-v8a, x86, x86_64)

| 指令集                             | 值                       |
| ---------------------------------- | ------------------------ |
| 基于 ARMv7 的设备上的硬件 FPU 指令 | `APP_ABI := armeabi-v7a` |
| ARMv8 AArch64                      | `APP_ABI := arm64-v8a`   |
| IA-32                              | `APP_ABI := x86`         |
| Intel64                            | `APP_ABI := x86_64`      |
| MIPS32                             | `APP_ABI := mips`        |
| MIPS64 (r6)                        | `APP_ABI := mips64`      |
| 所有支持的指令集                   | `APP_ABI := all`         |

不同 Android 手机使用不同的 CPU，因此支持不同的指令集。

**armeabi**

此 ABI 适用于基于 ARM、至少支持 ARMv5TE 指令集的 CPU。此 ABI 不支持硬件辅助的浮点计算。 相反，所有浮点运算都使用编译器 `libgcc.a` 静态库中的软件帮助程序函数。

**armeabi-v7a**

`armeabi-v7a` ABI 使用 `-mfloat-abi=softfp` 开关强制实施规则，要求编译器在函数调用时必须传递核心寄存器对中的所有双精度值，而不是专用浮点值。 系统可以使用 FP 寄存器执行所有内部计算。 这样可极大地加速计算。

如果要以 armeabi-v7a ABI 为目标，则必须设置下列标志：

```
CFLAGS= -march=armv7-a -mfloat-abi=softfp -mfpu=vfpv3-d16
```

**arm64-v8a**

此 ABI 适用于基于 ARMv8、支持 AArch64 的 CPU。它还包含 NEON 和 VFPv4 指令集。

**x86**

此 ABI 适用于支持通常称为“x86”或“IA-32”的指令集的 CPU。设置的标志如：

```
-march=i686 -mtune=intel -mssse3 -mfpmath=sse -m32
```

**x86_64**

```
-march=x86-64 -msse4.2 -mpopcnt -m64 -mtune=intel
```



现在手机主要是armeabi-v7a。查看手机cpu：

```
adb shell cat /proc/cpuinfo
adb shell getprop ro.product.cpu.abi
```



apk在安装的时候，如果手机是armeabi-v7a的，则会首先查看apk中是否存在armeabi-v7a目录，如果没有就会查找armeabi。

**保证cpu目录下so数量一致。**

​	如果目标是armeabi-v7a，但是拥有一个armeabi的，也可以把它放到armeabi-v7a目录下。但是反过来不行

| ABI(横 so)/CPU(竖 手机) | armeabi | armeabi-v7a | arm64-v8a | x86  | x86_64 |
| ----------------------- | ------- | ----------- | --------- | ---- | ------ |
| ARMV5                   | 支持    |             |           |      |        |
| ARMV7                   | 支持    | 支持        |           |      |        |
| ARMV8                   | 支持    | 支持        | 支持      |      |        |
| X86                     |         |             |           | 支持 |        |
| X86_64                  |         |             |           | 支持 | 支持   |



**APP_PLATFORM**

此变量包含目标 Android 平台的名称。例如，`android-3` 指定 Android 1.5 系统映像



**APP_STL**

默认情况下，NDK 构建系统为 Android 系统提供的最小 C++ 运行时库 (`system/lib/libstdc++.so`) 提供 C++ 功能。 

| 名称              | 说明>                        | 功能                    |
| ----------------- | ---------------------------- | ----------------------- |
| libstdc++（默认） | 默认最小系统 C++ 运行时库。  | 不适用                  |
| gabi++_static     | GAbi++ 运行时（静态）。      | C++ 异常和 RTTI         |
| gabi++_shared     | GAbi++ 运行时（共享）。      | C++ 异常和 RTTI         |
| stlport_static    | STLport 运行时（静态）。     | C++ 异常和 RTTI；标准库 |
| stlport_shared    | STLport 运行时（共享）。     | C++ 异常和 RTTI；标准库 |
| gnustl_static     | GNU STL（静态）。            | C++ 异常和 RTTI；标准库 |
| gnustl_shared     | GNU STL（共享）。            | C++ 异常和 RTTI；标准库 |
| c++_static        | LLVM libc++ 运行时（静态）。 | C++ 异常和 RTTI；标准库 |
| c++_shared        | LLVM libc++ 运行时（共享）。 | C++ 异常和 RTTI；标准库 |

