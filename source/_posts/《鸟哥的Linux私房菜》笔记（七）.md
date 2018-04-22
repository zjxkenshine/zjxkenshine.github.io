---
title: 《鸟哥的Linux私房菜》笔记（七）：正则表达式与shell script
date: 2018-03-25 21:11:01
tags: 
- Linux
- 正则表达式
- Shell
categories: 操作系统

---
## 0.学习之前
1. 包含章节
《鸟哥Linux习私房菜-基础学习篇》（第三版）
对应着知识点照着网上的内容整理的关于Centos7的操作（书本是Centos5.x）
第12章：正则表达式与文件格式化处理
第13章：学习shell script

2. 学习重点
	- 正则表达式
	- 基础的正则表达式
	- 扩展的正则表达式
	- 文件格式化处理
	- shell script的使用

---
## 1.认识正则表达式
1. 正则表达式对于系统管理员来说很重要（可惜我不是..）
2. 简单来说：
*正则表达式就是处理字符串的方法，以行为单位对字符串进行处理的程序。能通过一些特殊的符号来帮助用户更方便地查找、删除或替换某字符串。*
3. 正则表达式是一种表示法，只要工具程序支持这种表示法，改程序就可以被用来处理字符串。（cp,ls等命令不支持正则表达式）
4. 正则表达式和shell在Linux中的作用：任督二脉。
5. 分类：
正则表达式依照不同的严谨程度分为**基础正则表达式**和**扩展正则表达式**。
6. 与通配符的区别：
通配符是bush接口的一个功能
正则表达式则是一种字符串处理的方式

---
## 2.基础正则表达式
**1)语系对正则表达式的影响：**
1. 书上的练习都是以LANG=C来练习的，一般练习的时候就选择这个语系。因为它兼容POSIX接口。
2. 为了避免这个编码对数字和英文的选取造成影响，需要一些特殊符号的支持，如下图：
![](http://p5ki4lhmo.bkt.clouddn.com/00023%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A07-01.jpg)
几个特别重要的：
	- `[:alnum:]`： 0-9,A-Z,a-z
	- `[:alpha:]`：A-Z,a-z
	- `[:upper:]`：A-Z
	- `[:lower:]`：a-z
	- `[:digit:]`：0-9

**2)grep的高级用法：**
1. 语法：
		# grep [-A][-B] [--color=auto] '搜索字符串' filename
		-A:后面可加数字，after，列出该行后再列出该行后面的n行
		-B:后面可加数字，before，列出该行后再列出该行前面的n行
		--color=auto 将正确的那个选取数据列出颜色
grep是以整行为单位查找的。
2. 测试：
使用`# dmesg`列出内核信息并查询含有'etc'的行并显示前后三行：
		# dmesg | grep -n -A3 -B3 --color=auto 'eth'
测试结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00023%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A07-02.jpg)
3. 可以配置别名：`alias grep = 'grep --color=auto'`

**3)基础正则表达式的练习:**
1. 查找特定字符串：
`grep -n '字符串' 文件名`
反选：
`grep -vn '字符串' 文件名`
不分大小写：
`grep -in '字符串' 文件名`
测试：
![](http://p5ki4lhmo.bkt.clouddn.com/00023%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A07-03.jpg)
<br>
2. 使用`[]`来查找集合字符串：
查找含test或者tast的行：
`grep -n 't[ae]st' 文件名` --[ae]代表从a,e中选择一个字符
查找含有oo但是不以g开头的行：
`grep -n '[^g]oo' 文件名` --[^g]反向选择，代表不包括g
查找含oo但是前面不含小写字母的行：
`grep -n '[^a-z]oo' 文件名` --a-z的ASCII码是连续的，可以这样写
取得有数字的那一行：
`grep -n '[0-9]' 文件名`
还可以使用特殊符号来实现：
`grep -n '[^[:lower:]]oo' 文件名`
`grep -n '[[:digit:]]' 文件名`
测试：
![](http://p5ki4lhmo.bkt.clouddn.com/00023%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A07-04.jpg)
<br>
3. 行尾与行首字符`^$`:
找出以'the'开头的行：
`grep -n '^the' 文件名` 
找出开头是小写字母的行：
`grep -n '^[a-z]' 文件名`或者使用`'^[[:lower:]]'`
找出开头不是英文字母的行：
`gerp -n '^[^a-zA-Z]' 文件名`
找出结尾是小数点的行：
`grep -n '\.$' 文件名` --小数点是有意义的，需要使用\转义
找到空白行(打印时节省纸张可省略空白行)：
`grep -n '^$' 文件名` --Linux`^`代表开始，`$`代表结束
测试：
![](http://p5ki4lhmo.bkt.clouddn.com/00023%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A07-05.jpg)
<br>
4. 任意一个字符`.`与重复字符`*`:
小数点代表一定有一个任意字符；星号代表重复前一个字符任意次。
查找包含'g??d'的字符串：
`grep -n 'g..d' 文件名`
'o\*'代表包含0个或任意多个o,所以查找含有两个以上o的行需要这样写：
`grep -n 'ooo*' 文件名`
查找以含以g开头和结尾中间任意个(>1)o的字符串：
`grep -n 'goo*g' 文件名`
查找包含以g开头和结尾，中间可以是任意字符组合的字符串的行('g....g')：
`grep -n 'g.*g' 文件名`
查找含有数字字符串的行：
`grep -n '[0-9][0-9]*' 文件名`
测试：
![](http://p5ki4lhmo.bkt.clouddn.com/00023%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A07-06.jpg)
<br>
5. 限定连续的RE字符串范围`{}`：
使用`{}`规定重复多少次：相当于加了长度约束的星号。
但是{}在Linux中有特殊意义，需要转义。
查找含有两个o(及以上)的行：
`grep -n 'o\{2\}' 文件名`
查找含有以g开头和结尾并且只含有两个到五个o的字符串的行：
`grep -n 'go\{2,5\}g' 文件名`
查找包含以g开头和结尾并且有两个及以上o的行：
`grep -n 'go\{2,\}g' 文件名`
测试：
![](http://p5ki4lhmo.bkt.clouddn.com/00023%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A07-07.jpg)

**4)基础正则表达式字符和sed工具：**
1. 基础正则表达式字符：
![](http://p5ki4lhmo.bkt.clouddn.com/00023%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A07-08.jpg)
正则表达式也可以用于查找文件。
查找以a开头的文件，相当于`# ls -la*`:
		ls | grep -n '^a.*'
2. sed简介：
sed本身也是一个管道命令，可以分析stdin，还可以将数据进行替换，删除，新增，选取特定行等。
3. sed基本语法：
		# sed [-nefr] [动作]
		参数：
		-n：使用安静模式，只列出经过处理的行或者操作（默认列出全部）
		-e：直接在命令行模式下进行sed的动作编辑
		-f filename：后面可以接文件表示将sed动作写在该文件内
		-r：开启扩展正则表达式语法支持（默认是基础的）
		-i：直接修改文件内容，而不是由屏幕输出(最好别用，危险参数)
		动作说明：
			[n1[,n2]] function
		n1n2为起始和结束行
		function的可用值:
		a	：新增，后接字符串，在下一行新增
		c	：替换，后接字符串，替换n1到n2之间的值
		d	：删除，后面一般不加参数
		i	：插入，后接字符串，在上一行插入。
		p	：将某个选择的数据打印出来，通常和-n一起使用
		s	：替换，直接进行替换，可以配合正则表达式
4. sed测试：以行为单位的新增删除
列出测试文件`regular_express.txt`显示行号并删除第5-20行(d)：
`# nl regular_express.txt | sed '5-20d'`
再在第二行加上'this is the second line'(第一行后a）
`# nl regular_express.txt | sed '5-20d' | sed '1a this is the second line '`
测试结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00023%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A07-09.jpg)
发现确实添加了但是不会影响原来的行号输出。也不会改编源文件的内容。
5. sed测试：整行替换与指定显示行号
只显示文件`regular_express.txt`的第5-15行(p)：
`# nl regular_express.txt | sed -n '5-15p'`
不使用-n配合会输出很多没用的东西。
再将第789行替换为'three lines'(c):
`# nl regular_express.txt | sed -n '5-15p' | sed '7,9c three lines'`
测试：
![](http://p5ki4lhmo.bkt.clouddn.com/00023%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A07-10.jpg)
发现这些操作都是根据前面的标准输出来进行的（替换的根本不是源文件的第789行而是选择显示后的789行）
6. sed测试：部分数据的查找并替换(可配合正则表达式）：
语法：
		sed 's/要被替换的字符串/新的字符串/g'
例如将文件`regular_express.txt`中的任意数量的o全都替换为666：
`grep -n 'oo*' regular_express.txt | sed 's/oo*/666/g'`
测试结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00023%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A07-11.jpg)
如果新字符串为空，则相当于删除。

----
## 3.扩展的正则表达式
**1)扩展正则表达式介绍：**
1. 一般来说只用基本的正则表达式就已经完全够用了，扩展的正则表达式命令更多时候是为了简化命令操作。
2. 如想要去除以小写字母开头或以ASC开头的行：
基础正则表达式的写法：
`grep -v '^[[:lower:]]' 文件名 | grep -v '^[ABC]'`
扩展正则表达式的写法：
`egrep -v '^[a-z|ABC]' 文件名`
3. 注意需要使用`egrep`或者`grep -E`才能使用扩展正则表达式，前者是后者的别名。

**2)扩展正则表达式的语法**
1. `A+`：重复一个或一个以上的字符A
2. `A?`：有0或1个前面的字符A
3. `A|B`：用或的方式从A或B中选择一个
4. `()`：找出组字符串，如`g（oo|aa）d`表示可以是good也可以是gaad
5. `()+`：多个重复组的判断如`egrep 'A(xyz)+C'`表示查找以A开头C结束且中间有任意个xyz的行
6. ！和>等在正则表达式中并不是特殊符号，所以要查找只需要这样写：
		# grep -n '[!>]' 文件名

**3)扩展正则表达式的测试：**
1. 找出含有三个以上o的行：
`egrep -n 'ooo+' 文件名`
2. 找出含good或者god的行：
`egrep -n 'goo?d' 文件名`
3. 查找含gd或者god或者good的行：
`egrep -n 'good|god|gd' 文件名`
4. 同上，使用组：
`egrep -n 'g(|o|oo)d' 文件名`
5. 测试结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00023%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A07-12.jpg)

---
## 4.文件格式化与相关处理工具
**1)格式化打印工具：printf**
1. 很多时候需要将数据格式化输出，而printf可以帮助我们
2. 基本语法：
		# printf '打印格式' 实际内容
		关于格式的参数：
			\a：警告声音输出
			\b：退格键输出
			\f：清除屏幕
			\n：输出新的一行
			\r：ENTER按键
			\t：水平的tab按键
			\v：垂直的tab按键
			\xNN：将数值NN转换为字符(根据编码转换)
		C的常见变量格式：
			%ns:n是数字，代表多少字符
			%ni:代表多少整数字数
			%N.nf:N位整数与n位小数的浮点数
3. 注意grep不是管道命令，所以输入需要使用$(...)来代替|
4. 例子：
列出16进制数45代表的字符：
`# print '\x45\n'`
格式化列出某文件的值：
`# print '%ns %ni %N.nf \n' $(cat 文件名)`

**2)awk：好用的数据处理工具**
1. awk和sed都是Linux中很好用的数据处理工具，但是sed更倾向对一整行的处理，而awk则倾向于将一行分为部分（字段）来处理。awk和sed一样是管道命令。
2. 基本语法：
		awk '条件类型1{动作1} 条件类型2{动作2}...' filename
awk主要是处理每一行字段内的数据，默认的字段分隔符是空格或【tab】键。
每行的字段都有名称(编号)：从左到右依次是$1,$2,$3...,还有$0，代表一整行。
3. 基本使用及awk的处理流程：
列出最后5行并只输出每行的第一和第三个字段：
		# last -n 5 | awk '{print $1 "\t" $3}'
awk处理流程：
	- 读入第一行并将第一行数据填入$0$2等变量当中
	- 根据条件判断是否要进行后面的操作
	- 完成所有动作与条件类型
	- 还有后续行则重复1-3步的操作直到所有的行都处理完毕
4. awk内置变量：
--NF：每一行拥有的字段总数
--NR：目前awk所处的是第几行
--FS：目前的分隔符，默认是空格
上面的例子如想要列出当前第几行以及每行共多少个字段可以这么写：
		# last -n 5 | awk '{print $1 "\t" lines: "NR" \t columes: "NF"}'
5. awk的逻辑运算符：
主要用在条件类别处：和java的完全相同
变量设置用=，判断相等用==；
6. 书上的例子：
![](http://p5ki4lhmo.bkt.clouddn.com/00023%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A07-13.jpg)
7. awk简单使用总结：
	- 所有的{}中的动作如果需要多个命令可以使用;分开或者使用ENTER键换行
	- 逻辑运算的等于是==
	- printf格式化输出时必须加上\n才能实现换行
	- awk中的变量可以直接使用而不用加上$.

**3)文件比较：diff,cmp，patch**
1. diff：
比较两文件的差别，一般用在ASCII纯文本文件的比较。
以行为比较单位，因此通常用在同一文件（软件）的新旧版本区别。
2. diff语法：
		# diff [-bBi] from-file to-file
		参数：
		from-file：欲比较文件的文件名
		to-file：目的比较文件的文件名（from和to的-代表stdin）
		-b：忽略长空格，一行中无论有几个空格都会被当成一个
		-B：忽略空白行
		-i：忽略大小写
如：
`# diff passwd.old passwd.new`
3. cmp：
的使用没有diff那么广，主要是以字节为单位进行比较，可以比较二进制文件。基本语法如下：
		# cmp [-s] file1 file2
		-s:将所有不同点的字节处都列出来。（默认仅会输出第一的不同点）
4. diff比较之后制作补丁文件（patch）:
		# diff -Naur file.old file.new > file.patch  <==制作补丁文件
		# patch -pN < file.patch  <==更新旧文件
		# patch -R -pN < file.path  <==还原
		参数:
		-p:N为数字，代表取消几层目录
		-R：将文件还原到旧版本

**4)文件打印准备：pr的简单使用**
1. 使用pr打印某一文件后会输出文件创建时间，文件名以及页码三大项
		# pr file
2. 简单测试：
![](http://p5ki4lhmo.bkt.clouddn.com/00023%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A07-14.jpg)
3. 更多用法可以`man pr`查看

---
## 5.shell script介绍
**1)什么是shell script:**
1. 程序化脚本语言，针对shell写的脚本。
2. shell script是利用shell的功能所写的一个程序，这个程序是使用纯文本文件将一些shell的语法和命令（含外部命令）写在里面，搭配正则表达式、管道命令与数据流重定向等功能，以达到我们要的处理目的。

**2)为什么学习shell script**
1. 自动化管理的重要依据（基础）
2. 能执行追踪与管理系统的重要工作
3. 有简单的入侵检测功能
4. 连续命令单一化：整合一些执行上连续的命令，方便新手使用
5. 简单的数据处理
6. 跨平台，简单易学

**3)shell script编写入门**
1. shell script在系统管理上是很好用的工具。但是用在处理大数值运算上就不好了，shell script速度慢，耗费资源多。不适合进行数值运算。
2. 编写时的注意事项：
	- 命令的执行顺序是上到下，左到右
	- 命令参数间的多个空白都会被忽略掉
	- 空白行也会被忽略，tab也会被视为空格键
	- 读取到一个Enter符号(CR),就开始尝试执行该行（串）命令
	- 一行的内容太多可以使用"\Enter"换到下一行
	- \#是批注，后面的所有数据都会被当成数据
3. 如何执行一个shell script文件(需要拥有rx权限)：
假设为/XXXX/XXX/shell.sh文件
直接命令执行：可以使用绝对路径，相对路径或者PATH路径
bash命令执行：
`bash shell.sh`或`sh shell.sh`
4. 创建scripts目录，编写第一个script程序：
![](http://p5ki4lhmo.bkt.clouddn.com/00023%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A07-15-1.jpg)
第一行：`!/bin/bash`声明了这个script所使用的shell的名称。
后面的注释：程序内容说明及相关信息介绍
PATH那一行：主要环境变量的声明，配置重要的环境变量，最重要的是LANG和PATH。
echo -e那部分：主要的程序部分
告知程序结果：
--exit 0代表强制中断程序，并传回一个值给系统
--程序回传值：?,可以使用$?来查看具体的值
测试结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00023%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A07-16.jpg)
5. 程序开头的注释应该尽量记录以下内容：
script的功能
script的版本信息
script的作者与联系方式
script的版权声明方式
script的历史记录（History）
script内较特殊的命令，用绝对路径来执行
script执行时需要的环境变量的预先声明和配置

---
## 6. 简单的shell script练习
**1)例子测试：**
1. 交互式脚本，变量内容由用户决定
![](http://p5ki4lhmo.bkt.clouddn.com/00023%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A07-17.jpg)
可以让用户输入姓和名字并输出姓名。
2. 利用日期进行文件创建的脚本：
执行脚本后会自动创建型如前缀_日期时间这样的文件：
![](http://p5ki4lhmo.bkt.clouddn.com/00023%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A07-18.jpg)
（前天与今天的时间创建空文件）
3. 数值运算，实现简单的乘法：
![](http://p5ki4lhmo.bkt.clouddn.com/00023%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A07-19.jpg)
![](http://p5ki4lhmo.bkt.clouddn.com/00023%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A07-20.jpg)

**2)不同执行方式的区别：**
1. 利用bash进程执行：
程序在子进程执行，而父进程会休眠。无法在程序运行完得到文件中的变量
2. 利用source执行：
`# source 程序文件名`
在本个bash中执行。可以得到文件中的变量。

---
## 7.判断式的使用
可以通过$$和||判断回传码并决定是否要继续执行后面的代码。
但是可以通过test命令实现更加简单的条件判断
**1)利用test命令进行测试：**
1. 简单应用：判断某一文件是否存在
`# test -e 文件名 `
2. test的参数（可以判断的类型）
![](http://p5ki4lhmo.bkt.clouddn.com/00023%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A07-21.jpg)
![](http://p5ki4lhmo.bkt.clouddn.com/00023%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A07-22.jpg)
可以看出test的主要可以进行以下测试：
	- 判断某个文件名的文件类型及是否存在等
	- 判断文件的权限，对文件权限进行监测
	- 两个文件的比较
	- 两个整数之间的判断
	- 判断字符串的数据
	- 多条件判断

**2)利用判断符号[]**
1. 除了test之外还可以使用中括号"[]"来进行数据的判断。
如果想要知道$HOME这个变量是否为空，可以这样做：
`# [ -z "$HOME" ];echo $?`
2. 使用中括号的规则：
	- 中括号内的每个组件都要有空格来分隔
	- 中括号内的变量最好都以双引号括起来
	- 中括号内的常量最好都以单引号括起来
3. 变量不加双引号的错误例子：
`# name='kenshine 666'`
`# [ $name == "kenshine" ]`
这样子会报错说参数太多，因name有两个部分组成，最后会变成：
`[ keshien 666 = "kenshine" ]`而不是我们所希望的`[ "keshien 666" = "kenshine" ]`,所以需要用双引号括起来。
4. 中括号的用法几乎和test一模一样（参数也相同），但是中括号较常用在分支判断"if...then..."中。
5. []测试，实现输入y确认，输入n取消：
![](http://p5ki4lhmo.bkt.clouddn.com/00023%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A07-23-1.jpg)

**3)shell script的默认变量：$0,$1...**
1. 在执行shell script文件时，后面可以带默认的参数，就不用使用read每一次执行都输入了（$1-$4分别代表后面的第几个参数）：
		# sh /path/to/程序名 opt1 opt2 opt3 opt4
			   $0           $1    $2   $3   $4
2. 若执行了上述代码，则在程序体内可以使用用下列符号取值：
$0：程序名
$1-$4：opt1-opt4
$#：后面的参数个数（这里是4，可以多个）
$@：代表"$1","$2","$3","$4"，每个是独立的
$*：代表"$1分隔符$2分隔符$3分隔符$4",分隔符默认是空格。
3. shift变量偏移：
在程序体中使用`shift n`表示偏移多少个变量（原来的$1-$n消失，后面的依次补上）
4. shift 测试：
一个程序，6个参数，先偏移一个变量,后偏移3个变量，最后只剩下原来的第5和第6个参数：
![](http://p5ki4lhmo.bkt.clouddn.com/00023%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A07-24.jpg)
测试方法：
![](http://p5ki4lhmo.bkt.clouddn.com/00023%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A07-25.jpg)

---
## 8.条件判断式
**1)if...then...条件判断：**
1. 单层简单的条件判断式：
		if [ 条件判断式 ]；then
				满足条件的程序段
		if  <==将if反过来，代表if结束
如果条件判断式有多个条件需要判断，可以使用多个中括号加&&，||来实现：
`if([ 条件1 == 值1 ]&&/||[ 条件2 == 值2 ]...)`
2. 多重复杂的条件判断式：
不那么复杂的语法：
		if [ 条件判断式 ]；then
		//满足条件的代码
		else
		//不满足条件的代码
		fi
多条件的复杂一点的语法：
		if [ 条件判断式一 ]；then
		//满足条件1的代码
		elseif [ 条件判断式二 ]；then
		//满足条件2的代码
		....
		else
		//所有条件都不满足时的代码
		fi
3. 简单测试：
![](http://p5ki4lhmo.bkt.clouddn.com/00023%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A07-26.jpg)

**2)case...esac条件判断：**
1. 基本语法：
		case $变量名称 in
			"满足条件1的值"）
				//满足条件1的程序
			；；
			"满足条件2的值"）
				//满足条件2的程序
			；；
			*）
			//以上条件都不满足时的程序
			；；
		esac  <==反过来写代表结束
2. 可以将上面例子中的程序内容修改如下：
		case $1 in
			"1")
			echo "hello"
			;;
			"")
			echo "input a param"
			;;
			*）
			echo "you can only use param 1"
		esac
效果和使用if...elseif..相同。

**3)function结合条件判断式及默认参数：**
1. function:函数，在shell script内定义的能自动执行的程序段。
2. 基本语法：
		function 变量名（）{
		//程序段
		}
变量名后面的括号应该是不加参数的，参数可以直接在函数内部通过$n来得到。
3. function有参数的例子（无参的直接调用就行）
sh执行程序后输入参数，将输入的参数打印出来：
![](http://p5ki4lhmo.bkt.clouddn.com/00023%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A07-27.jpg)

---
## 9.循环(loop)
**1)不固定循环的两种常见形式：**
1. while循环：（条件满足则循环，不满足则退出循环）
		while [condition]
		do		<==循环开始的标志
			//程序段落
		done	<==循环结束的标志
2. until循环：（条件满足则退出循环）
		until [condition]
		do
			//循环体
		done
3. 测试：
让用户输入Y或者N，只有当输入Y或N时才退出循环，否则会一直提示用户输入:
![](http://p5ki4lhmo.bkt.clouddn.com/00023%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A07-28.jpg)

**2)for...do...done固定循环：**
1. 基本语法：
		for var in con1 con2 con3 ...
		do
			程序段
		done
第一次循环：$var的内容是con1
第二次循环：$var的内容是con2
以此类推...
2. 简单例子：
依次输出1234
![](http://p5ki4lhmo.bkt.clouddn.com/00023%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A07-29.jpg)
3. 书上一个输出passwd文件所有用户名的例子：
![](http://p5ki4lhmo.bkt.clouddn.com/00023%E9%B8%9F%E5%93%A5Linux%E5%AD%A6%E4%B9%A07-30.jpg)

**3)for循环的另外一种方式：**
1. for循环的另一种写法(和java的差不多)：
		for ((初始值;限制值；步长))
		do
			循环体程序段
		done
2. 初始值：直接以类似i=1方式设置就行
限制值：在这个范围内就循环，如i<100
执行步长：i=i+1

---
## 10.shell script的追踪和测试
1. 不需要执行就可以在执行前判断scripts.sh有无错误--`sh`的用法。
2. 语法：
		sh [-nvx] scripts.sh
		-n：不执行scripts.sh，仅查询语法
		-v：执行scripts前先将script中的内容输出到屏幕上
		-x：很有用的参数，将使用到的script内容显示到屏幕上

---
## 11.重要命令总结
`# grep [-A][-B] [--color=auto] '搜索字符串' filename` :查询整行数据

sed数据处理：
		# sed [-nefr] [动作]
		参数：
		-n：使用安静模式，只列出经过处理的行或者操作（默认列出全部）
		-e：直接在命令行模式下进行sed的动作编辑
		-f filename：后面可以接文件表示将sed动作写在该文件内
		-r：开启扩展正则表达式语法支持（默认是基础的）
		-i：直接修改文件内容，而不是由屏幕输出(最好别用，危险参数)
		动作说明：
			[n1[,n2]] function
		n1n2为起始和结束行
		function的可用值:
		a	：新增，后接字符串，在下一行新增
		c	：替换，后接字符串，替换n1到n2之间的值
		d	：删除，后面一般不加参数
		i	：插入，后接字符串，在上一行插入。
		p	：将某个选择的数据打印出来，通常和-n一起使用
		s	：替换，直接进行替换，可以配合正则表达式

`# printf '打印格式' 实际内容`：格式化打印
`awk '条件类型1{动作1} 条件类型2{动作2}...' filename`：awk数据处理

扩展正则表达式的使用（egrep）:
		1. 找出含有三个以上o的行：
		`egrep -n 'ooo+' 文件名`
		2. 找出含good或者god的行：
		`egrep -n 'goo?d' 文件名`
		3. 查找含gd或者god或者good的行：
		`egrep -n 'good|god|gd' 文件名`
		4. 同上，使用组：
		`egrep -n 'g(|o|oo)d' 文件名`

`# diff [-bBi] from-file to-file` ：文件比较
`# cmp [-s] file1 file2` ：文件比较

diff比较后制作补丁：
		# diff -Naur file.old file.new > file.patch  <==制作补丁文件
		# patch -pN < file.patch  <==更新旧文件
		# patch -R -pN < file.path  <==还原
`# source 程序文件名`
`# bash 程序文件名`
`# sh 程序文件名`  执行程序文件
`# pr file`：打印准备
`# test -e 文件名 `：test测试是否为空
`# [ -z "$HOME" ];echo $?`：中括号测试是否为空
		

		//shell script默认变量
		# sh /path/to/程序名 opt1 opt2 opt3 opt4
			   $0           $1    $2   $3   $4
在程序体中使用`shift n`表示偏移多少个变量

if选择语法：
			
				if [ 条件判断式一 ]；then
				//满足条件1的代码
				elseif [ 条件判断式二 ]；then
				//满足条件2的代码
				....
				else
				//所有条件都不满足时的代码
				fi

case选择语句：
		
		case $变量名称 in
			"满足条件1的值"）
				//满足条件1的程序
			；；
			"满足条件2的值"）
				//满足条件2的程序
			；；
			*）
			//以上条件都不满足时的程序
			；；
		esac  <==反过来写代表结束

function函数的使用：

		function 变量名（）{
		//程序段
		}

不固定循环：

	 while循环：（条件满足则循环，不满足则退出循环）
			while [condition]
			do		<==循环开始的标志
				//程序段落
			done	<==循环结束的标志
	 until循环：（条件满足则退出循环）
			until [condition]
			do
				//循环体
			done

for循环形式1：

		for var in con1 con2 con3 ...
		do
			程序段
		done

for循环形式2：

		for ((初始值;限制值；步长))
		do
			循环体程序段
		done

sh的用法（执行，调试，追踪shell script）:

		sh [-nvx] scripts.sh
		-n：不执行scripts.sh，仅查询语法
		-v：执行scripts前先将script中的内容输出到屏幕上
		-x：很有用的参数，将使用到的script内容显示到屏幕上

---