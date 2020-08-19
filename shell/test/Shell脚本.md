# Shell Script

[TOC]



[TOC]

`Shell`是一个用`C`语言编写的程序，它是用户使用`Linux`的桥梁。Shell既是一种命令语言，又是一种程序设计语言。Shell Script(Shell脚本)是一种为`Shell`编写的脚本程序。

Linux的Shell(程序)种类很多,常见的有：

- Bourne Shell (/usr/bin/sh或/bin/sh)

- Bourne Again Shell (/bin/bash)

- Shell for Root (/sbin/sh)
- ......

其中，bash由于易用和免费，被广泛使用。

下面是一段Shell脚本

```shell
#!/bin/bash
echo "Hello World!"
```

脚本第一行`#!/bin/bash` 表示告诉系统这个脚本需要哪种解释器来执行(属于哪种Shell程序)。

第二行使用 `echo` 输出一段字符串。Shell脚本很多情况下其实就是一段命令的集合。要完成一系列的操作，可能需要输入N次不同的指令。将这些指令放入一个Shell脚本中，执行Shell脚本就是执行这些命令。



执行新建的Shell脚本之前需要使该脚本权限为可执行。

```shell
# +x ： 增加可执行权限
chmod +x XX.sh 

#执行脚本
./XX.sh
#或
sh XX.sh
```

## 变量、字符串、数组与注释

```shell
#变量
#1、使用英文字母，数字和下划线，首个字符不能以数字开头
#2、中间不能有空格，可以使用下划线（_）。
#3、不能使用标点符号。
#4、不能使用bash里的关键字（help命令查看保留关键字）。
var="dongnao"
RUNOOB=1
LD_LIBRARY_PATH=2
_var=3
var2=4

#无效变量名
?var=123
user*name=runoob

#使用表达式/命令赋值
files=`ls .`
#使用变量
echo $files
echo ${files}

#只读变量
readonly files
#删除变量 不能删除只读变量
unset files

#字符串
#使用 ' 或者 " 来定义字符串，不同的
# ' 中不能使用变量
var="my dir has ${files}"

#字符串拼接
name="dongnao"
full1="hello, "$name" !"
full2="hello, ${name} !"
echo $full1 $full2
#获得长度
echo ${#name}

#从字符串第 2 个字符开始截取 3 个字符
echo ${name:1:3} # 输出 ong


#数组
array_name=(value0 value1 value2 value3)
echo ${array_name[0]}
echo ${array_name[1]}
#获得所有元素
echo ${array_name[@]}

# 取得数组元素的个数
length=${#array_name[@]}
length=${#array_name[*]}
```



## 传参

在执行 Shell 脚本时，向脚本传递参数，脚本内获取参数的格式为：**$n**。**n** 代表一个数字，1 为执行脚本的第一个参数，2 为执行脚本的第二个参数,......

```shell
echo "Shell 传递参数实例！";
echo "执行的文件名：$0";
echo "第一个参数为：$1";
echo "第二个参数为：$2";
echo "第三个参数为：$3";

#./test.sh 1 2 3
```

| 参数处理 | 说明                                                |
| -------- | --------------------------------------------------- |
| $#       | 传递到脚本的参数个数                                |
| $*       | 获得所有参数(不是数组), $1 $2 … $n 的形式           |
| $@       | 与$*相同，但是使用时加引号。"$1" "$2" … "$n" 的形式 |



## 运算符

```shell
a=10
b=20
#注意空格
val=`expr $a + $b`
#特别的 需要转义
val=`expr $a \* $b`

#关系运算符
# -eq 是否相等
# -ne 是否不想等
# -gt >
# -lt <
# -ge >=
# -le <=
if [ $a -eq $b ]
then
   echo "$a -eq $b : a 等于 b"
else
   echo "$a -eq $b: a 不等于 b"
fi
if(( $a==$b ))
then
	echo "yes"
else
	echo "no"
fi
#布尔
# ! 非
# -o = ||
# -a = &&

#字符串
# = 字符串相等
# != 字符串不相等
# -z 字符串长度为0   true
# -n 字符串长度不为0 true
str="123"
if [ $str ]
then
   echo "$str : 字符串不为空"
else
   echo "$str : 字符串为空"
fi

#文件
# -d 目录
# -f 普通文件
# -r 可读
# -w 可写
# -x 可执行
# -s 文件不为空(有内容) true
# -e 文件/目录存在 

```

## 流程控制

```shell
#for 循环

array=(1 2 3 4 5)
for var in ${array[@]}
do
    echo ${var}
done

for var in 1 2 3 4 5
do
done

for f in `ls`
do
done

#while 语句
a=1
while [ $a -le 5 ]
do
    echo $a
    let "a++"
done

b=1
while(( $b<=5 ))
do
    echo $b
    #自增
    let "b++"
done

#无限循环
for (( ; ; ))

while true
do
done


#until循环
until(( $a>=10 ))
do
	echo $a
	let "a++"
done


#case
a=1
case $a in
	1)  echo '1'
    ;;
    2)  echo '2'
    ;;
    *)  echo '其他'
    ;;
esac

case $a in
        1|2) echo "1/2"
        ;;
        *) echo "其他"
        ;;
esac

#break continue
```

## 函数与输出重定向

```shell
function func(){
    echo $0
    echo $1
    echo $2
}
func 1 2 


#输出重定向
#不会输出，而是将输出内容 放到一个a.txt文件，不存在则会创建
pwd > a.txt
#追加
echo "1212">>a.txt

#输入重定向
cat < a.txt
#其实可以省去 < 

#输入输出重定向
cat < a.txt > a1.txt

#将eof中内容交给wc -l
wc -l << EOF
	dongnao
	lance
	david
	jett
EOF

#/dev/null
#希望执行某个命令，但又不希望在屏幕上显示输出结果
#/dev/null 是一个特殊的文件，写入到它的内容都会被丢弃
wc -l a.txt > /dev/null


#引入其他shell
# 注意点号(.)和文件名中间有一空格
. filename   
source filename

#file1
#!/bin/bash
str=动脑

#file2
#!/bin/bash
#. t1.sh 或:
source t1.sh
echo $str

#file1不需要可执行权限
```