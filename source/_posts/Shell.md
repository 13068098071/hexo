---
title: shell
date: 2021-12-22
description: shell笔记总结
cover: https://s2.loli.net/2021/12/23/G8z7X9LPuV1Mmhp.jpg
tags: linux
categories: linux
---
## Shell 

### 1 变量

#### 1.1 变量定义

```shell
name=mzc
name='mzc'
name="mzc"
```

#### 1.2 使用变量

```shell
name=mzc
echo ${name}
echo ${name}qwe
```

#### 1.3 设置只读变量

```shell
name=mzc
readonly name
```

#### 1.4 删除变量

```shell
name=mzc
unset name
```

#### 1.5 变量类型

全局变量（环境变量）

```shell
export name
password=qwe
declare -x password
```

#### 1.6 字符串

单引号与双引号的区别：

- 单引号中的内容会原样输出，不会执行、不会取变量；
- 双引号中的内容可以执行、可以取变量；

```shell
name=mzc
echo 'hello, $name \"hh\"'  # 单引号字符串，输出 hello, $name \"hh\"
echo "hello, $name \"hh\""  # 双引号字符串，输出 hello, yxc "hh"
```

获取字符串长度

```shell
name=mzc
echo ${#name} # 3
```

提取子串

```shell
echo ${name:0:2} # 取0-2字符
```

### 2 默认变量

#### 2.1 文件参数变量

在执行shell脚本时，可以向脚本传递参数。$1是第一个参数，$2是第二个参数，以此类推。

```shell
#! /bin/bash
echo "文件名："$0
echo "第一个参数："$1
echo "第二个参数："$2
echo "第三个参数："$3
echo "第四个参数："$4

acs@9e0ebfcd82d7:~$ ./test.sh 1 2 3 4
文件名：./test.sh
第一个参数：1
第二个参数：2
第三个参数：3
第四个参数：4
```

#### 2.2 其他参数

![](C:\Users\17561\Desktop\笔记\图片\shell命令.png)

### 3 数组

数组中可以存放多个不同类型的值，只支持一维数组，初始化时不需要指明数组大小。
数组下标从0开始。

#### 3.1 数组的定义

```shell
array=(1,abc,"asd",qwe)

array[0]=1
array[1]=zxc
```

#### 3.2 取值

```shell
${array[index]}

echo array[0]
echo array[1]

# 取出所有
echo ${array[*]}
echo ${array[@]}

# 数组长度
echo ${#array[*]}
echo ${#array[@]}
```

### 4 expr命令

expr命令用户求表达式的值，格式为：expr 表达式

表达式说明：

- 用空格隔开每一项
- 用反斜杠放在shell特定的字符前面（发现表达式运行错误时，可以试试转义）
- 对包含空格和其他特殊字符的字符串要用引号括起来
- expr会在stdout中输出结果。如果为逻辑关系表达式，则结果为真，stdout为1，否则为0。
- expr的exit code：如果为逻辑关系表达式，则结果为真，exit code为0，否则为1。

#### 4.1 字符串表达式

- length string 返回string长度
- index string charset    charset中任意单个字符在string中最前面的字符位置，下标从1开始。如果在string中完全不存在charset中的字符，则返回0。
- substr string position length    返回STRING字符串中从POSITION开始，长度最大为LENGTH的子串。如果POSITION或LENGTH为负数，0或非数值，则返回空字符串。

```shell
str="hello world!"
echo `expr length "$str"`  # 输出12
echo `expr` index "$str" awd`  # 输出 7
echo `expr substr "${str}" 2 3`  # 输出 ell
```

#### 4.2 数学表达式

expr支持普通的算术操作，算术表达式优先级低于字符串表达式，高于逻辑关系表达式。

- {+  -} 加减运算。两端参数会转换为整数，如果转换失败则报错。
- {* / %} 乘，除，取模运算。两端参数会转换为整数，如果转换失败则报错。
- () 可以该表优先级，但需要用反斜杠转义

```shell
a=3
b=4

echo `expr ${a} + ${b}` # 7
echo `expr ${a} - ${b}` # -1
echo `expr ${a} \* ${b}` # 12
echo `expr ${a} / ${b}` # 0
echo `expr ${a} % ${b}` # 3
echo `expr \( $a + 1 \) \* \( $b + 1 \)` # 20
```

#### 4.3 逻辑关系表达式

- | 如果第一个参数非空且非0，则返回第一个参数的值，否则返回第二个参数的值，但要求第二个参数的值也是非空或非0，否则返回0。如果第一个参数是非空或非0时，不会计算第二个参数
-  & 如果两个参数都非空且非0，则返回第一个参数，否则返回0。如果第一个参为0或为空，则不会计算第二个参数。
- < <= = == != >= >  比较两端的参数，如果为true，则返回1，否则返回0。”==”是”=”的同义词。”expr”首先尝试将两端参数转换为整数，并做算术比较，如果转换失败，则按字符集排序规则做字符比较。
- () 可以该表优先级，但需要用反斜杠转义

```shell
a=3
b=4

echo `expr $a \> $b`  # 输出0，>需要转义
echo `expr $a '<' $b`  # 输出1，也可以将特殊字符用引号引起来
echo `expr $a '>=' $b`  # 输出0
echo `expr $a \<\= $b`  # 输出1

c=0
d=5

echo `expr $c \& $d`  # 输出0
echo `expr $a \& $b`  # 输出3
echo `expr $c \| $d`  # 输出5
echo `expr $a \| $b`  # 输出3
```

### 5 read

read命令用于从标准输入中读取单行数据。当读到文件结束符时，exit code为1，否则为0

参数说明

- -p: 后面可以接提示信息
- -t：后面跟秒数，定义输入字符的等待时间，超过等待时间后会自动忽略此命令

```shell
read name
mzc
echo $name 
mzc

read -p "Please input your name：" -t 5 name
Please input your name：mzc
echo $name
mzc
```

### 6 echo

输出格式：echo string



```shell
# 普通字符串
echo "Hello World"
echo Hello World

# 显示变量
name=mzc
echo "My name is $name"

# 显示换行
echo -e "Hi\n"
echo "acwing"

# 显示不换行
echo -e "Hi \c"
echo acwing

# 显示结果定向至文件
echo "Hello World" > out.txt

# 显示时间
echo `date`
```

### 7 print

printf命令用于格式化输出，类似于C/C++中的printf函数。

默认不会在字符串末尾添加换行符。

命令格式：printf format string

```shell
printf "%10d.\n" 123
printf "%-10.2f.\n" 123.123123
printf "My name is %s\n" "mzc"
printf "%d + %d = %d\n" 2 3 `expr 2 \* 3`

       123.
123.12    .
My name is yxc
2 * 3 = 6
```

### 8 test

#### 8.1 逻辑运算符&&和||

- && 表示与，|| 表示或
- 二者具有短路原则：
  expr1 && expr2：当expr1为假时，直接忽略expr2
  expr1 || expr2：当expr1为真时，直接忽略expr2
- 表达式的exit code为0，表示真；为非零，表示假。（与C/C++中的定义相反）

#### 8.2 test命令

在命令行中输入man test，可以查看test命令的用法。

test命令用于判断文件类型，以及对变量做比较。

test命令用exit code返回结果，而不是使用stdout。0表示真，非0表示假。

```shell
test 2 -lt 3 # 为真 0
echo $? # 输出0

ls  # 列出当前目录下的所有文件
homework  output.txt  test.sh  tmp
test -e test.sh && echo "exist" || echo "Not exist"
exist  # test.sh 文件存在
test -e test2.sh && echo "exist" || echo "Not exist"
Not exist  # testh2.sh 文件不存在
```

#### 8.3 文件类型判断

```shell
test -e filename # 判断文件是否存在
```

-e 文件是否存在

-f 是否为文件

-d 是否为目录

#### 8.4 文件权限判断

```shell
test -r filename # 判断文件是否可读
```

-r 是否可读

-w 是否可写

-x 是否可执行

-s 是否为非null文件

#### 8.5 整数间的比较

```shell
test $a -eq $b # a 是否等于 b 
```

| 参数 | 代表意义 |
| ---- | -------- |
| -eq  | 等于     |
| -ne  | 不等于   |
| -gt  | 大于     |
| -lt  | 小于     |
| -ge  | 大于等于 |
| -le  | 小于等于 |

#### 8.6 字符串比较

| 参数              | 代表意义                       |
| ----------------- | ------------------------------ |
| test -z string    | 是否为null，为null，返回true   |
| test -n string    | 是否为非null，非null，返回true |
| test str1 == str2 | 是否相等                       |
| test str1 != str2 | 是否不相等                     |

#### 8.7 多重条件判定

```shell
test -r filename -a -x filename
```

| 参数 | 代表意义             |
| ---- | -------------------- |
| -a   | 两个条件同时成立     |
| -o   | 两个条件至少成立一个 |
| !    | 取反                 |

#### 8.8 判断符号[]

[]与test用法几乎一模一样，更常用于if语句中。另外[[]]是[]的加强版，支持的特性更多。

```shell
[ a -lt 3 ] # 为真，返回 0
echo $? # 输出0

ls
homework  output.txt  test.sh  tmp
[ -e test.sh ] && echo "exit" || echo "not exit" 
exit
[ -e test.sh ] && echo "exit" || echo "not exit" 
not exit
```

 

注意：

- []内的每一项都要用空格隔开
- 中括号内的变量，最好用双引号括起来
- 中括号内的常数，最好用单或双引号括起来

```shell
name="acwing yxc"
[ $name == "acwing yxc" ]  # 错误，等价于 [ acwing yxc == "acwing yxc" ]，参数太多
[ "$name" == "acwing yxc" ]  # 正确
```

### 9 判断语句

#### 9.1 单层if

格式

```shell
if condition
then 
	语句1
	语句2
	....
fi

a=3
b=4
if [ $a -lt $b ] 
then 
	echo $a
fi	
```

#### 9.2 单层if-else

格式

```shell
if condition
then
	语句1
	语句2
	...
else 
	语句1
	语句2
	...
fi

a=3
b=4
if ! [ "$a" -lt "$b" ]
then 
	echo $a
else 
	echo $b
fi	
```

#### 9.3 多层if-elif-elif-else

格式

```shell
if condition
then
	语句1
	语句2
	...
elif condition
then
	语句1
	语句2
	...
elif condition
then
	语句1
	语句2
	...
else
	语句1
	语句2
	...
fi

a=4
if [ $a -eq 1 ]
then 
	echo $a
elif [ $a -eq 2 ]	
then 
	echo $a
elif [ $a -eq 3 ]	
then
	echo $a
else 
	echo $a
fi	
```

#### 9.4 case…esac形式

```shell
case $变量名称 in
	值1)
		语句1
		语句2
		...
		;;
	值2)	
		语句1
		语句2
		...
		;;
	*)		
		语句1
		语句2
		...
		;;
esac		

a=4
case $a in
	1)
		echo $a
		;;
	2)
    	echo $a
    	;;
    3)	
    	echo $a
    	;;
    *)	
    	echo $a
    	;;
esac    	
```

### 10 循环语句

#### 10.1 for…in…do…done

```shell
for var in val1 val2 val3
do
	语句1
	语句2
	...
done

# 输出遍历元素
for i in a 22 cc
do
	echo $i
done

# 输出当前路劲下的所有文件名
for i in `ls`
do
	echo $i
done 

# 输出1-10
for i in $(seq 1 10)
do
	echo $i
done

# {1..10} {a..z}
for i in {a..z}
do
	echo $i
done
```

#### 10.2 for ((…;…;…)) do…done

```shell
for((expression;condition;expression))
do
	语句1
	语句2
done

for((i = 0; i <= 10; i++))
do
	echo $i
done	
```

#### 10.3 until…do…done循环

```shell
until condition 
do
	语句1
	语句2
	..
done

until [ "${word}" == "yes ] || [ "${word}" == "YES"]
do
	read -p "Please input yes/YES to stop this program" word
done
```

### 11 函数

bash中的函数类似于C/C++中的函数，但return的返回值与C/C++不同，返回的是exit code，取值为0-255，0表示正常结束。

如果想获取函数的输出结果，可以通过echo输出到stdout中，然后通过$(function_name)来获取stdout中的结果。

函数的return值可以通过$?来获取。

格式：

```shell
[function] function_name(){ # function可以省略
	语句1
	语句2
}

func(){
	name=mzc
	echo "Hello $name" 
}

func

func(){
	name=mzc
	echo "Hello $name"
	return 123
}

output=$(func)
ret=$?

echo	"output=$(output)"
echo	"return=$(ret)"
```

#### 11.1 函数的输入参数

在函数内，$1表示第一个输入参数，$2表示第二个输入参数，依次类推

注意：$0表示文件名

```shell
func(){
	word=""
	while [ "$word" != 'y' ] && [ "$word" != 'n' ]
	do
		read -p ""要进入func($1)函数吗？请输入y/n：" word
	done
	
	if [ "word" == 'n' ]
	then
		echo 0
		return 0
	fi
	
	if [ $1 -le 0 ]
	then 
		echo 0
		return 0
	fi
	
	sum=$(func $(expr $1 - 1))
	echo $(expr $sum + $1)
}

echo $(func 10)
```



### 11 练习

```shell
#! /bin/bash

# ***************  homework_0  *****************
dir0=/home/acs/homework/lesson_1/homework_0

homework 1 create 0

for i in dir_a dir_b dir_c
do
    mkdir "${dir0}/$i"
done


# ***************  homework_1  *****************
dir1=/home/acs/homework/lesson_1/homework_1

homework 1 create 1

for i in a.txt b.txt c.txt 
do
    cp "${dir1}/${i}" "${dir1}/${i}.bak"
done




# ***************  homework_2  *****************
dir2=/home/acs/homework/lesson_1/homework_2

homework 1 create 2

for i in a b c
do
    mv "${dir2}/${i}.txt" "${dir2}/${i}_new.txt"
done

# ***************  homework_3  *****************
dir3=/home/acs/homework/lesson_1/homework_3

homework 1 create 3

for i in a.txt b.txt c.txt
do
    mv "${dir3}/dir_a/${i}" "${dir3}/dir_b"
done

# ***************  homework_4  *****************

dir4=/home/acs/homework/lesson_1/homework_4

homework 1 create 4

rm ${dir4}/*



# ***************  homework_5  *****************

dir5=/home/acs/homework/lesson_1/homework_5
homework 1 create 5

rm -rf ${dir5}/*


# ***************  homework_6  *****************

dir6=/home/acs/homework/lesson_1/homework_6

homework 1 create 6

mkdir "${dir6}/dir_a"

mv "${dir6}/task.txt" "${dir6}/dir_a/done.txt"


# ***************  homework_7  *****************
dir7=/home/acs/homework/lesson_1/homework_7

homework 1 create 7

for((i = 0; i < 3; i++))
do
    mkdir ${dir7}/dir_${i}
    for j in a b c
    do
        cp ${dir7}/${j}.txt ${dir7}/dir_${i}/${j}${i}.txt
    done
done

# ***************  homework_8  *****************
dir8=/home/acs/homework/lesson_1/homework_8

homework 1 create 8
rm ${dir8}/dir_a/a.txt
mv ${dir8}/dir_b/b.txt ${dir8}/dir_b/b_new.txt
cp ${dir8}/dir_c/c.txt ${dir8}/dir_c/c.txt.bak


# ***************  homework_9  *****************

dir9=/home/acs/homework/lesson_1/homework_9

homework 1 create 9

rm ${dir9}/*.txt
homework 1 test
```

