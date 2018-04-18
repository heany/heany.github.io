---
title: Shell Script学习笔记
date: 2018-03-31 14:09:19
categories:
- Script
tags:
- Shell linux
---

## 一、 前言
最近开始使用Linux系统，经常操作terminal，经常使用Linux下的bash来进行一些重复的操作，因此想学习`shell script`来简化一些安装软件的步骤，实现自动化部署一些应用，节约时间。本文介绍了一些bash命令的一些技巧，然后详细地讲解了**正则表达式与文件格式化处理**，并且对于每一个操作都有相关的实例进行练习，加强理解;**`shell script`**，同样对于每一个功能都有相应的实例进行练习，可以说～非常适合新手来阅读学习了！


## 二、 基础知识
&nbsp;&nbsp;什么是 shell script (程序化脚本) 呢？就字面上的意义，我们将他分为两部份。 在『 shell 』部分，我们在bash当中已经提过了，那是一个文字介面底下让我们与系统沟通的一个工具介面。那么『 script 』是啥？ 字面上的意义， script 是『脚本、剧本』的意思。整句话是说， shell script 是针对 shell 所写的『剧本！』 
&nbsp;&nbsp;什么东西啊？其实， shell script 是利用 shell 的功能所写的一个『程序 (program)』，这个程序是使用纯文字档，将一些 shell 的语法与命令(含外部命令)写在里面， 搭配正规表示法、管线命令与数据流重导向等功能，以达到我们所想要的处理目的。
&nbsp;&nbsp;shell script 更提供阵列、回圈、条件与逻辑判断等重要功能，让使用者也可以直接以 shell 来撰写程序，而不必使用类似 C 程序语言等传统程序撰写的语法呢！


### 2.1 一些shell命令的基础知识及技巧
&nbsp;&nbsp;一些快捷键：
`^A` :鼠标光标移到命令最左端
`^E` :鼠标光标移到命令最右端
`^k` :删除鼠标光标后面的命令
`^u` :删除鼠标光标前面的命令
`^s` :锁定terminal
`^Q` :解锁
`^y` :撤销上一次操作
`^D` :输入结束（EOF），例如邮件结束的时候
`^Z` :暂停目前的命令

`cat`命令的一些技巧：以下代码将内容写入file
```
cat << EOF > file
> hello
> world
> ......
> EOF
```
>  管道及tee

管道
:    管道的作用是将前一条命令的输出变成管道后命令的输入
        ls /bin | wc -l   ##统计 ls /bin命令输出的行数
    系统中错误的输出是无法通过管道的
    用 `2>&1` 可以把错误的输出编号由2变成1

tee
:    `tee`复制输出到指定位置
        data | tee file |wc -l  ##tee命令复制date命令的输出到file中，并统计输出行数

### 2.2 脚本中调用其他的解释器(如python)执行的代码
&nbsp;&nbsp;shell script中第一行通常需要一行指令指定执行此脚本的解释器,如：
    #!/bin/bash
    #!/bin/python
    #!/bin/perl
以上分别指定了常用的三种脚本的解释器，

>  多条命令执行

1. 用`;`号分隔，这种方式各条命令之间没有逻辑性
    cd; ls; ll
2. 用逻辑连接符`&&` `||`来连接。
    ./confirgure && make && make install  ##必须前面的命令执行成功，才能执行后面的命令
```python
if con1
    com2
else com3
```
上面这段代码相当于
    com1 && com2 && com3

>  shell通配符

`*` :任意个
`?` :1个
`[]`:匹配括号里面的一个

>  `echo`输出颜色文本

    echo -e "\e[1;31m This is red text. \e[0m"

>  `printf`格式化输出
    printf "hello\n"

>  `read`命令变量键盘读取使用方法如下：

    read [-pt] variable
    paras:
    -p :后面接提示符
    -t :后面接等待的“秒数”
    example:
    read atest
    read -p "please a string:" -t 30 atest

>  `declare/typeset`命令：声明变量的类型：默认类型为字符串

    declare [-aixr] variable
    paras:
    -a :将后面的variable的变量定义为数组（array）类型
    -i :定义为integer类型
    -x :用法与export一样，将后面的变量变成环境变量
    -r :将变量设置成为readonly类型，该变量不可被更改，也不能重设
    example:
    sum=100+300+50
    echo $sum  # sum=100+300+50

    declare -i sum=100+300+50
    echo $sum    #450

>  `array`数组变量类型;读取时：${数组}

    var[index]=content
    example:
    var[1]="Tom"
    var[2]="tony"
    echo "${var[1]},${var[2]}"   #Tom,tony


>  `alias`、`unalias`命令别名设置：;
    alias lm='ls -l | more'  #lm执行的是ls -all more
    unalias lm   #去掉lm命令的别名


>  `history`查询历史命令

    history [n]
    history [-c]
    history [-raw]
    paras:
    n :数字，列出最近的n条命令行的意思
    -c ：将目前的shell中的所有history内容全部消除
    -a : 将新增的history命令新增进去histfiles,或默认写入~/.bash_history
    -r : 将histfiles的内容读到目前这个shell的history中
    -w : 将目前的history记忆内容写进histfiles中。
    另：
    ！number ：执行第几条命令的意思
    !command ：由最近的命令向前搜寻命令串开头command的那个命令，并执行
    !! : 执行上一个命令

### 2.3 数据流重定向(redirect)
>  **数据流重定向**就是将某个命令执行后应该要出现在屏幕上的数据传输到其他的地方。

+ 标准输入(stdin):代码为0，使用`<`(覆盖)或`<<`(追加)
+ 标准输出(stdout)：代码为1，使用`>`或`>>`
+ 标准错误输出(stderr):代码为2，使用`2>`或`2>>`

>  `/dev/null`垃圾桶黑洞设备

    find /home -name .bashrc 2> /dec/null  #将错误的数据丢弃，屏幕上显示正确的数据
    ## /home/heany/.bashrc

>  `standard input`:`<`与`<<`

    cat > catfile < ~/.bashrc #用stdin替代键盘的输入以创建新文件的简单流程

    cat > catfile << "eof"
    > This is a test.
    > OK now stop
    > eof    #输入这个关键字，立刻就结束不需要输入[ctrl+D]


### 2.4 管道命令(pipe)
>  **管道命令**使用`|`这个界定符号，仅能处理经由前面一个命令传来的正确的信息，也就是standard output的信息，对于standard error并没有直接处理的能力。管道后面接的第一个数据必定是”命令“，而且这个命令必须要能够接收standard input的数据才行，这样的命令才可以是“管道命令”，例如less, more,head, tail等。而ls, cp ,mv这些则不是管道命令，因为他们不会接收来自stdin的数据。

#### 2.4.1 选取命令：cut,grep
  *选取命令*就是将一段数据经过分析后，取出我们所想要的，或者是经由分析关键字，得我们所想要的哪一行，通常是针对"行"来分析的。

>  `cut`这个命令可以将一段信息的某一段“切出来”，处理信息以“行”为单位。

    cut -d '分隔字符' -f fields  #用于分隔字符
    cut -c 字符范围     #用于排列整齐的信息
    paras:
    -f ：依据-d的分隔字符将一段信息切割成数段，用-f取出第几段的意思。
    -c ：以字符的单位取出固定字符区间
    examples:
    echo $PATH | cut -d ':' -f 5   ## 将echo $PATH结果以":"分隔，，找到第5个

    export | cut -c 12-  ## 将export输出的信息取得第12字符后的所有字符串

>  `grep`命令是分析一行信息，若当中有我们需要的信息，就将该行拿出来。
    grep [-acinv] [--color=auto] '查找字符串' filename
    paras:
    -a:将binary文件以text文件的形式查找数据
    -c:计算找到"查找字符串"的次数
    -i:忽略大小写的不同，所以大小写视为相同
    -n:顺便输出行号
    -v: 反向选择，即显示没有"查找字符串"内容的那一行
    --color=auto : 可以将找到的关键字部分加上颜色显示
    examples:
    last | grep 'root'  #将last中有出现root的那一行就取出来
    last | grep -v 'root' # 与上面相反，只要没有root就取出
    last | grep 'root' | cut -d '' -f 1  #在取出root后，利用上一个命令cut的处理，就能取出第一列

#### 2.4.2 排序命令：sort, wc ,uniq

>  `sort`排序

    sort [-fbMnrtuk] [file or stdin]
    paras:
    -f : 忽略大小写的差异
    -b : 忽略最前面的空格符部分
    -M : 以月份的名字来排序
    -n : 使用”纯数字“进行排序
    -r : 反向排序
    -u : 就是uniq，相同的数据中，仅出现一行代表
    -t : 分隔符，默认是用[Tab]键来分隔
    -k : 以那个区间(field)来进行排序的意思
    examples:
    cat /etc/passwd | sort -t ':' -k 3  #以:来分隔，以第三列来排序

    last | cut -d '' -f1 | sort  #利用last将输出的数据仅取账号，并加以排序

>  `uniq`将重复的数字仅列出一个显示

    uniq [-ic]
    paras:
    -i : 忽略大小写字符的不同
    -c : 进行计数
    examples:
    last | cut -d '' -f1 | sort |uniq  # 使用last将账号列出，仅取出账号列，进行排序后取出最后一位

>  `wc`计算输出信息的整体数据

    wc [-lwm]
    paras:
    -l : 仅列出行
    -w : 仅列出多少字
    -m : 多少字符
    examples:
    cat /etc/man.config | wc  #列出config里面到底有多少相关行数、字数、字符数，分别输出三列

#### 2.4.3 双向重定向：tee
    *tee*会同时将数据流送与文件于屏幕，而输出到屏幕的就是standout,可以让下个命令继续处理。
    tee [-a] file
    paras:
    -a : 以累加(append)的方式，将数据加入file当中。
    examples:
    last | tee [-a] last.list | cut -d '' -f1   #将last的输出存一份last.list文件中 

#### 2.4.4 字符转换命令：tr, col, join, paste, expand 

*主要介绍这些字符转换命令在管道中的使用方法*

>  `tr`可以用来删除一段信息当中的文字，或者是进行文字信息的替换

    tr [-ds] SET1 ....
    paras:
    -d : 删除信息当中的SET1这个字符串，
    -s : 替换掉重复的字符串
    examples:
    last | tr [a-z] [A-Z]  #将last输出的信息中所有的小写字符变成大写字符

>  `col`

    col [-xb]
    paras:
    -x : 将Tab键转换成对等的空格键
    -b : 在文字内有反斜杠\时，仅保留反斜杠最后接的那个字符

>  `join`处理两个文件之间的数据，而且主要是将两个文件当中有相同数据的那一行加在一起。

    join [-ti12] file1 file2
    paras:
    -t : join默认以空格符分隔数据，并且对比”第一个字段“的数据，如果两个文件相同，则将两条数据连成一行，且第一个字段放在第一个。
    -i : 忽略大小写的差异
    -1 : 这个是数字的1,代表第一个文件要用哪个字段来分析的意思
    -2 : 代表第二个文件要用哪个字段来分析的意思。
    examples:
    join -t ':' /etc/passwd /etc/shadow #将两个文件第一个字段相同者整合成一行

>  `paste`直接将两行贴在一起，且中间一[tab]键隔开。

    paste [-d] file1 file2
    paras:
    -d : 后面可以接分隔字符，默认是以[tab]来分隔的
    -  : 如果file部分写成 - ，表示来自standard input的数据的意思
    examples:
    paste /etc/passwd /etc/shadow  # 将两个文件的同一行贴在一起

#### 2.4.5 切割命令：split

*切割命令*将一个大文件依据文件大小或行数来切割成为小文件

    split [-bl] file PREFIX
    paras:
    -b : 后面可接欲切割成的文件大小，可加单位
    -l : 以行数来进行切割
    PREFIX : 代表前导符，可作为切割文件的前导文字。
    examples:
    cd /tmp; split -b 300k /etc/termcap termcap  # 将/etc/termcap分成300K一个文件
    ls -al / | split -l 10 -lsroot #将使用ls-al /输出的信息中，每10行记录成一个文件
    






## 三、 正则表达式与文件格式化处理
### 3.1、 正则表达式概念(Regular Expression)
  简单的说,正则表达式就是处理字串的方法,他是以行为单位来进行字串的处理行为, 正则
表达式通过一些特殊符号的辅助,可以让使用者轻易的达到“搜寻/删除/取代”某特定字串的处
理程序!<span style="color:#F00">强大的字符串处理能力</span>

### 3.2、 基础正则表达式
#### 3.2.1、 语系对正则表达式的影响
&nbsp;&nbsp;计算机文件其实记录的仅有0与1,我们看到的字符文字与数字都是通过编码表转换来的。由于不同语系的编码数据并不相同，所有就会造成数据提取结果的差异了。举例来说，在英文大小写的编码顺序中，zh_TW.big5及C这两种语系的输出结果分别如下:

+ LANG=C时：0 1 2 3 4 ... A B C D ... Z abcd ... z
+ LANG=zh_TW时: 0 1 2 3 4 ... a A b B C c ... z Z

上面的顺序是编码的顺序，我们可以很清楚的发现这两种语系明显就是不一样！如果你想要提取大写字符而使用[A-Z]时，会发现会发现LANG=C确实可以仅捉到大写字符 (因为是连续的),但是如果 LANG=zh_TW.big5时,就会发现到, 连同小写的b-z也会被撷取出来!因为就
编码的顺序来看,big5语系可以撷取到“ A b B c C ... z Z ”这一堆字符哩! 所以,使用正则表
达式时,需要特别留意当时环境的语系为何, 否则可能会发现与别人不相同的撷取结果喔!一般练习时采用`LANG=C`这个语系。

>  一些特殊符号的代表意义

<img src="https://wx4.sinaimg.cn/mw1024/e0db46edgy1fq7iwmdmuzj20og0fndj2.jpg"  alt="图片名称" align=center />

#### 3.2.2、 grep的一些进阶选项
<img src="https://wx1.sinaimg.cn/mw1024/e0db46edgy1fq7j5i0kq8j20zl0mtgul.jpg"  alt="图片名称" align=center />

>  grep是一个很常见也很常用的指令，它最重要的功能就是进行字符串数据的比对，然后将符合使用者需求的字串行印出来。**需要说明的是“grep 在数据中查寻一个字串时,是以 "整行" 为单位来进行数据的撷取的!”**也就是说,假如一个文件内有 10 行,其中有两行具有你所搜寻
的字串,则将那两行显示在屏幕上,其他的就丢弃了!

#### 3.2.3、 基础正则表达式练习
练习用文件下载如下：
```
wget http://linux.vbird.org/linux_basic/0330regularex/regular_express.txt

# content
"Open Source" is a good mechanism to develop programs.
apple is my favorite food.
Football game is not use feet only.
this dress doesn't fit me.
However, this dress is about $ 3183 dollars.^M
GNU is free air not free beer.^M
Her hair is very beauty.^M
I can't finish the test.^M
Oh! The soup taste good.^M
motorcycle is cheap than car.
This window is clear.
the symbol '*' is represented as start.
Oh!     My god!
The gd software is a library for drafting programs.^M
You are the best is mean you are the no. 1.
The world <Happy> is the same with "glad".
I like dog.
google is the best tools for search keyword.
goooooogle yes!
go! go! Let's go.
# I am VBird


```
+ 例题一、搜寻特定字串

从上面下载的文件中**取the这个特定字串**，最简单的方式就是这样：
```
grep -n 'the' regular_express.txt 

# output
8:I can't finish the test.
12:the symbol '*' is represented as start.
15:You are the best is mean you are the no. 1.
16:The world <Happy> is the same with "glad".
18:google is the best tools for search keyword.
```
如果要**反向选择**，也就是当该行没有'the'这个字串时才显示在屏幕上，那就直接使用：
```
grep -vn 'the' regular_express.txt 

# output
1:"Open Source" is a good mechanism to develop programs.
2:apple is my favorite food.
3:Football game is not use feet only.
4:this dress doesn't fit me.
5:However, this dress is about $ 3183 dollars.
6:GNU is free air not free beer.
7:Her hair is very beauty.
9:Oh! The soup taste good.
10:motorcycle is cheap than car.
11:This window is clear.
13:Oh!  My god!
14:The gd software is a library for drafting programs.
17:I like dog.
19:goooooogle yes!
20:go! go! Let's go.
21:# I am VBird
22:

```
如果想取得**不论大小写**的‘the’这个字串，则：
```
grep -in 'the' regular_express.txt 

# output
8:I can't finish the test.
9:Oh! The soup taste good.
12:the symbol '*' is represented as start.
14:The gd software is a library for drafting programs.
15:You are the best is mean you are the no. 1.
16:The world <Happy> is the same with "glad".
18:google is the best tools for search keyword.
```
+ 例题二、利用中括号[]来搜寻集合字符

如果我想要搜寻test或taste这两个单字时,可以发现到,其实她们有共通的't?st'存在~这
个时候,我可以这样来搜寻:
```
grep -n 't[ae]st' regular_express.txt

# output
8:I can't finish the test.
9:Oh! The soup taste good.
```

>  `[]`这个里面不论有几个字符，都仅匹配某一个字符，，如果想要搜寻到有`oo`的字符串时，则使用下面的命令：

```
grep -n 'oo' regular_express.txt 

# output
1:"Open Source" is a good mechanism to develop programs.
2:apple is my favorite food.
3:Football game is not use feet only.
9:Oh! The soup taste good.
18:google is the best tools for search keyword.
19:goooooogle yes!
```
但是，如果我不想要`oo`前面有g的话呢？此时，可以利用在集合字符的反向选择来达成：
```
grep -n '[^g]oo' regular_express.txt

# output
2:apple is my favorite food.
3:Football game is not use feet only.
18:google is the best tools for search keyword.
19:goooooogle yes!
```
如果我们不想要`oo`前面有小写字符，可以这样写：
```
grep -n '[^a-z]oo' regular_express.txt

# output
3:Football game is not use feet only.
```

>  **连续编码**可以使用减号`-`，也可以使用如下的方法来取得前面的两个测试的结果：

```
grep -n '[^[:lower:]]oo' regular_express.txt
# output
3:Football game is not use feet only.

# 只取含数字的那一行
grep -n '[[:digit:]]' regular_express.txt
# output
5:However, this dress is about $ 3183 dollars.
15:You are the best is mean you are the no. 1.
```
+ 例题三、行首与行尾字符^$

在上面例子中，可以查询到一行字串里面有`the`的，那如果我想要让`the`只在行首列出呢？这个时候就需要使用**定位字符**了！
```
grep -n '^the' regular_express.txt 

# output
12:the symbol '*' is represented as start.
```
如果我想**开头是小写字符**的那一行就列出呢？可以这样写：
```
grep -n '^[a-z]' regular_express.txt 
# output
2:apple is my favorite food.
4:this dress doesn't fit me.
10:motorcycle is cheap than car.
12:the symbol '*' is represented as start.
18:google is the best tools for search keyword.
19:goooooogle yes!
20:go! go! Let's go.

# 也可以使用下面的命令来实现
grep -n '^[[:lower:]]' regular_express.txt
```
如果我不想要开头是**英文字母**，则可以是这样：
```
grep -n '^[^a-zA-Z]' regular_express.txt 
# output
1:"Open Source" is a good mechanism to develop programs.
21:# I am VBird

# 也可以这样写
grep -n '^[^[:alpha:]]' regular_express.txt 
```

>  注意到了吧?那个`^`符号,在字符集合符号(括号[])之内与之外是不同的! 在 [] 内代表“反
向选择”,在[]之外则代表定位在行首的意义!要分清楚喔! 

反过来思考,那如果我想要找出
来,**行尾**结束为小数点 (.) 的那一行,该如何处理:
```
grep -n '\.$' regular_express.txt 
# output
1:"Open Source" is a good mechanism to develop programs.
2:apple is my favorite food.
3:Football game is not use feet only.
4:this dress doesn't fit me.
10:motorcycle is cheap than car.
11:This window is clear.
12:the symbol '*' is represented as start.
15:You are the best is mean you are the no. 1.
16:The world <Happy> is the same with "glad".
17:I like dog.
18:google is the best tools for search keyword.
20:go! go! Let's go.
```
>  因为小数点`.`具有其他意义，所以必须使用转义字符`(\)`来加以解除其特殊意义！但是第 5~9 行最后面也是 . 啊~怎么无法打 5~9  印出来? 这里就牵涉到 Windows 平台的软件对于断行字符的判断问题了!

我们使用`cat -A`前十行的倒数六行。
```
 cat -An  regular_express.txt| head -n 10  | tail -n 6
 # output
     5  However, this dress is about $ 3183 dollars.^M$
     6  GNU is free air not free beer.^M$
     7  Her hair is very beauty.^M$
     8  I can't finish the test.^M$
     9  Oh! The soup taste good.^M$
    10  motorcycle is cheap than car.$
```
>   在上面的输出中我们可以发现5~9行为Windows的断行字符`(^M$)`,而正常的Linux应该仅有第10行显示的那样`($)`。所以,那个`.`自然就不是紧接在`$`之前喔!也就捉不到5~9 行了!这样可以了解^与$的意义了吧。

如果我想找出哪一行是空白行，也就是该行没有输入任何数据，如何操作？
```
grep -n '^$' regular_express.txt 
# output
22:
```
>  因为只有行首和行尾`(^$)`，所以这样就可以找出空白行啦。

再来,假设你已经知道在一个程序脚本 (shell script) 或者是配置文件当中,空白行与开头为 # 的那一行是注解,因此如果你要将数据列出给别人参考时,可以将这些数据省略掉以节省保贵的纸张,那么你可以怎么做呢? 我们以 /etc/rsyslog.conf 这个文件来作范例,你可以自行参考一下输出的结果:
```
cat -n /etc/rsyslog.conf
# 在ubuntu 17.10中，输出了60行，很多空白行与#开头的注解行

grep -v '^$' /etc/rsyslog.conf | grep -v -n '^#'
# 结果仅有13行，其中第一个`-v '^$'` 代表不要空行
# 第二个“-v '^#'”， 代表不要开头是#的那行
# output
10:module(load="imuxsock") # provides support for local system logging
19:module(load="imklog" permitnonkernelfacility="on")
27:$ActionFileDefaultTemplate RSYSLOG_TraditionalFileFormat
29:$RepeatedMsgReduction on
33:$FileOwner syslog
34:$FileGroup adm
35:$FileCreateMode 0640
36:$DirCreateMode 0755
37:$Umask 0022
38:$PrivDropToUser syslog
39:$PrivDropToGroup syslog
43:$WorkDirectory /var/spool/rsyslog
47:$IncludeConfig /etc/rsyslog.d/*.conf


```
+ 例题四、任意一个字符`.`与重复字符`*`
我们知道通配符`*`可以用来代表任意（0或多个）字符，但是正则表达式并不是通用字符，两者之间是不同的！至于正则表达式中的`.`则代表“绝对有一个任意字符”的意思！这两个符号在正则表达式的意义如下：
>  `.`(小数点)：代表“一定有**一个**任意字符”的意思
>  
>  `*`(星号)：代表“**重复前一个字符**，0到无穷多次”的意思，为组合形态

这样讲不好懂,我们直接做个练习吧!假设我需要找出 g??d 的字串,亦即共有四个字符,起头是 g 而结束是 d ,我可以这样做:
```
grep -n 'g..d' regular_express.txt 
# output
1:"Open Source" is a good mechanism to develop programs.
9:Oh! The soup taste good.
16:The world <Happy> is the same with "glad".
```
再来,如果我想要列出有 oo, ooo, oooo 等等的数据, 也就是说,至少要有两个(含)o以上,该如何是好?是 o 还是 oo 还是 ooo* 呢? 虽然你可以试看看结果, 不过结果太占版面了。

因为`*`代表的是**重复 0 个或多个前面的 RE 字符**”的意义, 因此,**`o*`代表的是具有空字符或一个o以上的字符**, 特别注意,因为允许空字符(就是有没有字符都可以的意思),因
此,`grep -n 'o*' regular_express.txt`将会把所有的数据都打印出来屏幕上!


那如果是`oo*`呢?则第一个o肯定必须要存在,第二个o则是可有可无的多个o, 所以,凡
是含有o,oo,ooo,oooo等等,都可以被列出来~

同理,当我们需要至少两个o以上的字串时,就需要`ooo*`,亦即是:
```
grep -n 'ooo*' regular_express.txt 
# output
1:"Open Source" is a good mechanism to develop programs.
2:apple is my favorite food.
3:Football game is not use feet only.
9:Oh! The soup taste good.
18:google is the best tools for search keyword.
19:goooooogle yes!
```
如果我们想要字符串开头和结尾都是g,但是两个g之间仅能存在至少一个o，亦即是`gog,goog,gooog..`等等，那该如何？
```
grep -n 'g.*g' regular_express.txt 
# . 代表任意一个字符
# .*代表任意个字符
```
如果我想找出含’任意数字‘的行列呢，所以就成为：
```
grep -n '[0-9][0-9]*' regular_express.txt 
# output
5:However, this dress is about $ 3183 dollars.
15:You are the best is mean you are the no. 1.
```
+ 例题五、限定连续RE字符范围

在上个例题当中,我们可以利用 . 与 RE 字符及 * 来设置 0 个到无限多个重复字符, 那如果
我想要限制一个范围区间内的重复字符数呢?举例来说,我想要找出两个到五个 o 的连续字
串,该如何做?这时候就得要使用到限定范围的字符 {} 了。 但因为 { 与 } 的符号在 shell 是有
特殊意义的,因此, 我们必须要使用转义字符 \ 来让他失去特殊意义才行。 至于 {} 的语法是
这样的,假设我要找到两个 o 的字串,可以是:
```
grep -n 'o\{2\}' regular_express.txt 
# output
1:"Open Source" is a good mechanism to develop programs.
2:apple is my favorite food.
3:Football game is not use feet only.
9:Oh! The soup taste good.
18:google is the best tools for search keyword.
19:goooooogle yes!

grep -n 'go\{2,5\}g' regular_express.txt 
# output
18:google is the best tools for search keyword.

# 没有了第19行，因为它有6个o
```


#### 3.2.4、 基础正则表达式字符汇整(characters)
经过上面的几个简单的练习，我们可以将基础的正则表达式特殊字符汇整如下表：

<img src="https://wx3.sinaimg.cn/mw1024/e0db46edgy1fq7ngc14chj20xj163nbj.jpg" align="center">

>  **正则表达式的特殊字符”与一般在命令行输入指令的“万用字符”并不相同**。
>  例如。在万用字符当中的 代表的是“ 0 ~ 无限多个字符”的意思,但是在正则表达式当中, 则是“重复0到无穷多个的前一个 RE 字符”的意思~使用的意义并不相同,不要搞混了!

举例来说,不支持正则表达式的 ls 这个工具中,若我们使用 “ls -l ” 代表的是任意文件名的文
件,而 “ls -l a ”代表的是以 a 为开头的任何文件名的文件, 但在正则表达式中,我们要找到
含有以 a 为开头的文件,则必须要这样:(需搭配支持正则表达式的工具)

    ls | grep -n '^a.*'
要想以`ls -l`配合grep找出`/etc/`下面文件类型为链接文件属性的文件名，该如何做呢？由于`ls -l`列出链接文件时标头会是`lrwxrwxrwx`,因此使用如下的指令即可找出结果：

    ls -l /etc \grep '^l'
    # 若仅想列出几个文件，再以`| wc -l`来累加处理即可。


#### 3.2.5、 sed工具
在了解了一些正则表达式的基础应用之后,再来呢?呵呵~两个东西可以玩一玩的,那就是
sed 跟下面会介绍的 awk 了! 这两个家伙可是相当的有用的啊!

我们先来谈一谈 sed 好了, sed 本身也是一个管线命令,可以分析 standard input 的啦! 而
且 sed 还可以将数据进行取代、删除、新增、撷取特定行等等的功能呢!很不错吧~ 我们先
来了解一下 sed 的用法,再来聊他的用途好了!

<img src="https://wx1.sinaimg.cn/mw1024/e0db46edgy1fq7o1zepd6j20zc0gmtem.jpg" align="center">

+ 以行为单位的新增/删除功能

sed光是用看的，是看不懂的啦！，又要开始练习了

<b><span style="color: red">范例一：</span></b>将`/etc/passwd`的内容拷贝`pd.txt`列出并且打印行号，同时，请将第2-5行删除！
```
nl pd.txt  | sed '2,5d'
# output
     1  root:x:0:0:root:/root:/bin/bash
     6  games:x:5:60:games:/usr/games:/usr/sbin/nologin
     7  man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
     8  lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
    ...........后面省略................

```
>  看到了吧?sed 的动作为 '2,5d' ,那个 d 就是删除!因为 2-5 行给他删除了,所以显示的数据
就没有 2-5 行啰~ 另外,注意一下,原本应该是要下达 sed -e 才对,没有 -e 也行啦!同时
也要注意的是, sed 后面接的动作,请务必以 '' 两个单引号括住喔!

如果只删除第2行，可以使用`nl pd.txt | sed '2d'`来实现，若要删除第3到最后一行，则可以使用`nl pd.txt |sed '3,$d'`,其中`$`代表最后一行！

<b><span style="color: red">范例二：</span></b>承上题，在第二行后（亦即是加在第三行）加上“drink tea?”字样！
```
nl pd.txt | sed '2a drink tea?'
# output
     1  root:x:0:0:root:/root:/bin/bash
     2  daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
drink tea?
     3  bin:x:2:2:bin:/bin:/usr/sbin/nologin
```
嘿嘿!在 a 后面加上的字串就已将出现在第二行后面啰!那如果是要在第二行前呢?`nl
pd.txt | sed '2i drink tea'`就对啦!就是将“ a ”变成“ i ”即可。 增加一行很简单,那如果
是要增将两行以上呢?

<b><span style="color: red">范例三：</span></b>在第二行后面加入两行字，例如“drink tea or ...”与"drink beer"字样！
```
nl pd.txt | sed '2a drink tea or ... \
 > drink beer?'
# output
     1  root:x:0:0:root:/root:/bin/bash
     2  daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
drink tea or ... 
drink beer?
     3  bin:x:2:2:bin:/bin:/usr/sbin/nologin
.............后面省略........
```
>  这个范例的重点是“我们可以新增不只一行喔!可以新增好几行”但是每一行之间都必须要以反
斜线“ \ ”来进行新行的增加喔!所以,上面的例子中,我们可以发现在第一行的最后面就有 \
存在啦!在多行新增的情况下, \ 是一定要的喔!

+ 以行为单位的取代与显示功能

刚刚只是介绍了如何新增与删除，那么如果要整行取代呢？

<b><span style="color: red">范例四：</span></b>我想将第2-5行的内容取代成为"No 2-5 number"呢？
```
nl pd.txt | sed '2,5c No 2-5 number'
# output
     1  root:x:0:0:root:/root:/bin/bash
No 2-5 number
     6  games:x:5:60:games:/usr/games:/usr/sbin/nologin

```
通过这个方法我们就能够将数据整行取代了!非常容易吧!sed 还有更好用的东东!我们以前
想要列出第 11~20 行, 得要通过`head -n 20 | tail -n 10`之类的方法来处理,很麻烦啦~ sed
则可以简单的直接取出你想要的那几行!是通过行号来捉的喔!看看下面的范例先:

<b><span style="color: red">范例五：</span></b>仅列出`pd.txt`文件内的第5-7行
```
nl pd.txt | sed -n '5,7p'
# output
     5  sync:x:4:65534:sync:/bin:/bin/sync
     6  games:x:5:60:games:/usr/games:/usr/sbin/nologin
     7  man:x:6:12:man:/var/cache/man:/usr/sbin/nologin

```
>  上述的指令中有个重要的选项``-n`,按照说明文档,这个 -n 代表的是“安静模式”! 那么为什
么要使用安静模式呢?你可以自行下达 `sed '5,7p'` 就知道了 (5-7 行会重复输出)! 有没有
加上 -n 的参数时,输出的数据可是差很多的喔!你可以通过这个 sed 的以行为单位的显示功
能, 就能够将某一个文件内的某些行号捉出来查阅!很棒的功能!不是吗?

+ 部分数据的搜寻并取代的功能

除了整行的处理模式之外，sed还可以用行为单位进行部分数据的查找并替换的功能，基本上sed的查找与替换与vi类似！

    sed 's/要被取代的字串/新的字串/g'

上表中特殊字体的部分为关键字,请记下来!至于三个斜线分成两栏就是新旧字串的替换
啦。我们使用下面这个取得IP数据的范例,一段一段的来处理,让你了解一下什么是咱们所谓的查找并替换吧!
```
# step1 : 先观察源信息，利用`sbin/ifconfig` 查询IP
    /sbin/ifconfig enp3s0
    # output
    enp3s0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
            inet 219.223.243.131  netmask 255.255.248.0  broadcast 219.223.247.255
            inet6 fe80::1234:4cee:444b:1d66  prefixlen 64  scopeid 0x20<link>
            inet6 2001:250:3c02:301:c9a3:6a1d:d5ce:ec5f  prefixlen 128  scopeid 0x0<global>
    # 重点在第二行，也就是ip地址那一行

# step2 :利用关键字配合grep选取出关键的一行数据
    ifconfig enp3s0 | grep 'inet '
    # output
        inet 219.223.243.131  netmask 255.255.248.0  broadcast 219.223.247.255

# step3：将IP前面的部分予以删除
     ifconfig enp3s0 | grep 'inet ' | sed 's/^.*inet //g'
     # output
        219.223.243.131  netmask 255.255.248.0  broadcast 219.223.247.255

# step4：将IP后面的部分删除
    ifconfig enp3s0 | grep 'inet ' | sed 's/^.*inet//g' | sed 's/ netmask.*$//g'
    # output
        219.223.243.131 
```
>  通过这个范例的练习也建议您依据此一步骤来研究你的指令!就是先观察,然后再一层一层
的试做, 如果有做不对的地方,就先予以修改,改完之后测试,成功后再往下继续测试。

+ 直接修改文件内容（<b><span style="color: red">危险动作</span></b>）

你以为 sed 只有这样的能耐吗?那可不! sed 甚至可以直接修改文件的内容呢!而不必使用
管线命令或数据流重导向! 不过,由于这个动作会直接修改到原始的文件,所以请你千万不
要随便拿系统配置文件来测试喔! 我们还是使用你下载的 regular_express.txt 文件来测试看
看吧!

<b><span style="color: red">范例六：</span></b>利用sed将`regular_express.txt`内每一行结尾若为`.`则换成`！`。
```
sed -i 's/\.$/\!/g' regular_express.txt 
cat regular_express.txt 
# output
"Open Source" is a good mechanism to develop programs!
apple is my favorite food!
Football game is not use feet only!
this dress doesn't fit me!
However, this dress is about $ 3183 dollars.
GNU is free air not free beer.
Her hair is very beauty.
I can't finish the test.
Oh! The soup taste good.
motorcycle is cheap than car!
This window is clear!
the symbol '*' is represented as start!
Oh! My god!
The gd software is a library for drafting programs.
You are the best is mean you are the no. 1!
The world <Happy> is the same with "glad"!
I like dog!
google is the best tools for search keyword!
goooooogle yes!
go! go! Let's go!
# I am VBird

```

<b><span style="color: red">范例七：</span></b>利用sed直接在`regular_express.txt`最后一行加入`# This is a test`
```
sed -i '$a # This is a test' regular_express.txt
nl regular_express.txt| sed -n '20,25p'
# output
    20  go! go! Let's go!
    21  # I am VBird
       
    22  # This is a test

```
>  sed 的“ -i ”选项可以直接修改文件内容,这功能非常有帮助!举例来说,如果你有一个 100 万
行的文件,你要在第 100 行加某些文字,此时使用 vim 可能会疯掉!因为文件太大了!那怎
办?就利用 sed 啊!通过 sed 直接修改/取代的功能,你甚至不需要使用 vim 去修订!很棒
吧!


### 3.3、 扩展正则表达式
事实上,一般读者只要了解基础型的正则表达式大概就已经相当足够了,不过,某些时刻为
了要简化整个指令操作, 了解一下使用范围更广的延伸型正则表达式的表示式会更方便呢!
举个简单的例子好了,在上节的例题三的最后一个例子中,我们要去除空白行与行首为 # 的
行列,使用的是

    grep -v '^$' regular_express.txt | grep -v '^#'

需要使用到管线命令来搜寻两次！那么如果使用延伸型的正则表达式，我们可以简化为：

    egrep -v '^$|^#' regular_express.txt
>  延伸型正则表达式可以通过群组功能“ | ”来进行一次搜寻!那个在单引号内的管线意义为“或
or”啦! 是否变的更简单呢?此外,grep 默认仅支持基础正则表达式,如果要使用延伸型正则
表达式,你可以使用 grep -E , 不过更建议直接使用 egrep !直接区分指令比较好记忆!其
实 egrep 与 grep -E 是类似命令别名的关系。

熟悉了正则表达式之后,到这个延伸型的正则表达式,你应该也会想到,不就是多几个重要
的特殊符号吗?  是的~所以,我们就直接来说明一下,延伸型正则表达式的特殊符如下表所示：

<img src="https://wx2.sinaimg.cn/mw1024/e0db46edgy1fq7wpghpsdj20ue0jdjxc.jpg" align="center">

**例子测试结果如下图所示：**

<img src="https://wx2.sinaimg.cn/mw1024/e0db46edgy1fq7wx5abqmj20mz0aljti.jpg" align="center">

>  `!`不是特殊字符，如果想要查出来文件中含有！与>的字行时，可以这样操作：
>      grep -n '[!>]' regular_express.txt


### 3.4、 文件的格式化与相关处理
接下来让我们来将文件进行一些简单的编排吧!下面这些动作可以将你的讯息进行排版的动
作, 不需要重新以 vim 去编辑,通过数据流重导向配合下面介绍的 printf 功能,以及 awk 指
令, 就可以让你的讯息以你想要的模样来输出了!试看看吧!

#### 3.4.1、 格式化打印：printf

<img src="https://wx1.sinaimg.cn/mw690/e0db46edgy1fq7x8zjkhij20j908c75p.jpg" align="center">


#### 3.4.2、 `awk`：好用的数据处理工具
awk 也是一个非常棒的数据处理工具!相较于 sed 常常作用于一整个行的处理, awk 则比较
倾向于一行当中分成数个“字段”来处理。因此,awk 相当的适合处理小型的数据数据处理!`awk` 通常运行的模式是这样的:

    awk '条件类型1{动作1} 条件类型2{动作2} .... ' filename

awk 后面接两个单引号并加上大括号 {} 来设置想要对数据进行的处理动作。 awk 可以处理后
续接的文件,也可以读取来自前个指令的 standard output 。 但如前面说的, awk 主要是处
理“每一行的字段内的数据”,而默认的“字段的分隔符号为 "空白键" 或 "[tab]键" ”!举例来说,
我们用 last 可以将登陆者的数据取出来,结果如下所示:（下面命令是在我购买的远程主机上进行的）

```
last -n 5
# output
root     pts/1        58.60.1.102      Tue Apr 10 22:29   still logged in   
root     pts/0        58.60.1.76       Tue Apr 10 22:21   still logged in   
root     pts/0        58.60.1.76       Sun Apr  8 20:58 - 21:36  (00:37)    
root     pts/0        58.60.1.76       Fri Mar 23 08:19 - 08:19  (00:00)    
reboot   system boot  4.13.9-1.el6.elr Tue Jan 23 22:26 - 22:30 (76+23:03)  
```
若我想取出帐号与登录者的IP，且帐号与IP之间一`[Tab]`隔开，可以进行如下操作：
```
last -n 5 | awk '{print $1 "\t" $3}'
# output
root    58.60.1.102
root    58.60.1.76
root    58.60.1.76
root    58.60.1.76
reboot  boot
```
上表是 awk 最常使用的动作!通过 print 的功能将字段数据列出来!字段的分隔则以空白键或
[tab] 按键来隔开。 因为不论哪一行我都要处理,因此,就不需要有 "条件类型" 的限制!我所
想要的是第一栏以及第三栏, 但是,第五行的内容怪怪的~这是因为数据格式的问题啊!所以~使用 awk 的时候,请先确认一下你的数据当中,如果是连续性的数据,请不要有空格或 [tab]在内,否则,就会像这个例子这样,会发生误判!

另外,由上面这个例子你也会知道,在awk的括号内,每一行的每个字段都是有变量名称
的,那就是 $1, $2... 等变量名称。以上面的例子来说, `root`是$1,因为他是第一栏!
至于`58.60.1.102`是第三栏, 所以他就是`$3`!后面以此类推~呵呵!还有个变量!那
就是 $0 ,$0 代表“一整列数据”的意思~以上面的例子来说,第一行的 $0 代表的就是一整行数据的意思。 由此可知,刚刚上面五行当中,整个 awk 的处理流程是:
```
step1：读入第一行，并将第一行的数据填入 $0, $1, $2.... 等变量当中;
step2：依据”条件类型“的限制，判断是否需要进行后面的”动作“;
step3：做完所有的动作与条件类型
step4：若还有后续的‘行’的数据，则重复上面1-3的步骤，知道所有的数据都读完为止。
```
经过这样的步骤,你会晓得, awk 是“以行为一次处理的单位”, 而“以字段为最小的处理单
位”。好了,那么 awk 怎么知道我到底这个数据有几行?有几栏呢?这就需要 awk 的内置变
量的帮忙啦~

| 变量名称 | 代表意义 |
| :----  | :----:  |
| NF      | 每一行($0)拥有的字段总数 |
| NR      | 目前awk所处理的是“第几行”数据 |
| FS      | 目前的分隔字符，默认是空白键 |

>  继续上面的例子，如果我想实现以下效果：<br>
>    &emsp;列出每一行的帐号(就是 $1);<br>
>    &emsp;列出目前处理的行数(就是 awk 内的 NR 变量)<br>
>    &emsp;并且说明,该行有多少字段(就是 awk 内的 NF 变量)

```
last -n 5 | awk '{print $1 "\t lines: " NR "\t columns: " NF}'
# output
root     lines: 1    columns: 10
root     lines: 2    columns: 10
root     lines: 3    columns: 10
root     lines: 4    columns: 10
reboot   lines: 5    columns: 11
```

+ `awk`的逻辑运算字符

既然有需要用到“条件”的类别，自然就需要一些逻辑运算咯～例如下面这些，如下表所示：

| 运算单元 | 代表意义 |
| :-----: | :----: |
| >  |  大于|
| <  |  小于|
| >= |  大于等于 |
| <= |  小于等于 |
| == |  等于 |
| != |  不等于 |

好了,我们实际来运用一下逻辑判断吧!举例来说,在 `pd.txt` 当中是以冒号 ":" 来作为
字段的分隔, 该文件中第一字段为帐号,第三字段则是 UID。那假设我要查阅,第三栏小于
10 以下的数据,并且仅列出帐号与第三栏, 那么可以这样做:
```
cat pd.txt  | \
> awk '{FS=":"} $3 < 10 {print $1 "\t" $3}'
# output
root:x:0:0:root:/root:/bin/bash 
daemon  1
bin 2
sys 3
sync    4
games   5
man 6
lp  7
mail    8
news    9
```
有趣吧!不过,怎么第一行没有正确的显示出来呢?这是因为我们读入第一行的时候,那些
变量 $1, $2... 默认还是以空白键为分隔的,所以虽然我们定义了 FS=":" 了, 但是却仅能在第
二行后才开始生效。那么怎么办呢?我们可以预先设置 awk 的变量啊! 利用 BEGIN 这个关
键字喔!这样做:
```
cat pd.txt | \
> awk 'BEGIN {FS=":"} $3 < 10 {print $1 "\t" $3}'
# output
root    0
daemon  1
bin     2
sys     3
sync    4
games   5
```
我有一个薪资数据表文件名为`pay.txt`，内容是这样的：
```
Name    1st     2nd     3th
Vd      23000   24000   25000
Di      21000   20000   23000
Bd      43000   42000   41000
```
如果我想计算每个人的总额呢？而且我还想要格式化输出，我们可以这样考虑：
>  第一行只是说明，所以第一行不要进行加总（NR==1时处理）<br>
>  第二行以后就会有加总的情况出现（NR >=2 以后处理 ）

```
cat pay.txt | \
> awk 'NR == 1 {printf "%10s %10s %10s %10s %10s\n",$1,$2,$3,$4,"Total"}
> NR >= 2 {Total = $2+$3+$4
> printf "%10s %10s %10s %10s %10.2f\n",$1,$2,$3,$4,Total}'
# output
      Name        1st        2nd        3th      Total
        Vd      23000      24000      25000   72000.00
        Di      21000      20000      23000   64000.00
        Bd      43000      42000      41000  126000.00

```
上面的例子有几个重要事项应该先说明的：

+ `awk`的指令间隔:所有awk的动作,亦即在{}内的动作,如果有需要多个指令辅助时,可利用分号“;”间隔, 或者直接以 [Enter] 按键来隔开每个指令,例如上面的范例中我就使用了三次 [enter] 喔!
+ 逻辑运算当中，如果是**等于**的情况，则务必使用两个等号“==”！
+ 格式化输出时，在printf的格式设置当中，务必加上`\n`，才能进行分行
+ 与bash shell的变量不同，在`awk`当中，变量可以直接使用，不需加上`$`符号

利用 awk 这个玩意儿,就可以帮我们处理很多日常工作了呢!真是好用的很~ 此外, awk
的输出格式当中,常常会以 printf 来辅助,所以, 最好你对 printf 也稍微熟悉一下比较好啦!
另外, awk 的动作内 {} 也是支持 if (条件) 的喔! 举例来说,上面的指令可以修改成为这
样:
```
cat pay.txt | \
> awk '{if (NR==1) printf "%10s %10s %10s %10s %10s\n",$1,$2,$3,$4,"Total"}
> NR >= 2 {Total = $2+$3+$4
> printf "%10s %10s %10s %10s %10.2f\n",$1,$2,$3,$4,Total}'
# output
      Name        1st        2nd        3th      Total
        Vd      23000      24000      25000   72000.00
        Di      21000      20000      23000   64000.00
        Bd      43000      42000      41000  126000.00

```



#### 3.4.3、 文件比较工具
什么时候会用到文件的比对啊?通常是**同一个软件的不同版本之间,比较配置文件与原始文件的差别**。很多时候所谓的文件比对,通常是用在ASCII纯文本文件的比对上的!那么比对文件的指令有哪些?最常见的就是diff啰!另外,除了diff比对之外,我们还可以借由cmp来比对非纯文本文件!同时,也能够借由 diff 创建的分析档,以处理补丁(patch)功能的文件呢!就来玩玩先!

+ <b style="color: red">diff</b>

`diff`就是用在比对两个文件之间的差异的,并且是**以行为单位来比对的**!一般是用在ASCII纯文本文件的比对上。由于是以行为比对的单位,因此**`diff`通常是用在同一的文件(或软件)的新旧版本区别上**! 举例来说,假如我们要将`/etc/passwd`处理成为一个新的版本`pd`,处理方式为:将第四行删除,第六行则取代成为“no six line”,新的文件放置到 `/shell_learning/test` 里面,那么应该怎么做?

```
cd ~/Desktop/shell_learning
mkdir test
cp /etc/passwd pd
cat pd | sed -e '4d' -e '6c no six line' > pd.new
# output
cat -n pd.new | head -n 6
     1  root:x:0:0:root:/root:/bin/bash
     2  daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
     3  bin:x:2:2:bin:/bin:/usr/sbin/nologin
     4  sync:x:4:65534:sync:/bin:/bin/sync
     5  no six line
     6  man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
```
接下来讨论一下关于`diff`的用法吧！
```
diff [-bBi] from-file to-file
# paras：
    from-file : 一个文件名,作为原始比对文件的文件名;
    to-file : 一个文件名,作为目的比对文件的文件名;
    注意,from-file 或 to-file 可以 - 取代,那个 - 代表“Standard input”之意。
    -b  :忽略一行当中,仅有多个空白的差异(例如 "about me" 与 "about       me" 视为相同
    -B  :忽略空白行的差异。
    -i  :忽略大小写的不同。
```
<span style="color:red"><b>范例一:</b></span>比对`pd`与`pd.new`的差异
```
diff pd pd.new 
# output
4d3      <==左边第四行被删除(d)了,基准是右边第三行
< sys:x:3:3:sys:/dev:/usr/sbin/nologin   <===这边列出左边(<)文件被删除的那一行内容
6c5      <==左边文件的第六行被替换(c)成右边文件的第五行
< games:x:5:60:games:/usr/games:/usr/sbin/nologin   <===左边(<)文件的第六行内容
---
> no six line     <==右边(>)文件第五行内容
```
>  很聪明吧！用`diff`把我们刚刚的处理给比对完毕了！<br>
>  `diff`也可以比较整个目录下的区别<br>
>  `diff`还可以比较不同目录下的相同文件名的内容

+ <b style="color: red">cmp</b>

相对于`diff`的广泛用途,`cmp`似乎就用的没有这么多了~`cmp`主要也是在比对两个文件,他
主要利用“字节”单位去比对, 因此,当然也可以比对`binary file`啰~(还是要再提醒喔,`diff`主要是以“行”为单位比对,`cmp`则是以“字节”为单位去比对,这并不相同!)
```
cmp [-l] file1 file2
# paras:
    -l :将所有的不同点的字节处都列出来。因为 cmp 默认仅会输出第一个发现的不同点。
# examples:
    cmp pd pd.new 
    # output
    pd pd.new differ: byte 120, line 4
```
+ <b style="color: red">patch</b>

patch 这个指令与 diff 可是有密不可分的关系啊!我们前面提到,diff 可以用来分辨两个版本
之间的差异, 举例来说,刚刚我们所创建的`pd`及`pd.new`之间就是两个不同版本的文件。那么,如果要“升级”呢?就是“将旧的文件升级成为新的文件”时,应该要怎么做呢?其实也不难啦!就是“先比较先旧版本的差异,并将差异档制作成为补丁文件,再由补丁文件更新旧文件”即可。举例来说,我们可以这样做测试:

<b style="color: red">范例一：</b>以`pd`与`pd.new`制作补丁文件
```
diff -Naur pd pd.new  > pd.patch
cat pd.patch

# output
--- pd  2018-04-11 13:03:58.165695684 +0800  <== 新旧文件的信息
+++ pd.new  2018-04-11 13:08:07.710232163 +0800
@@ -1,9 +1,8 @@           <== 新旧文件要修改数据的界定范围，旧文件1-9行，新文件1-8行
 root:x:0:0:root:/root:/bin/bash
 daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
 bin:x:2:2:bin:/bin:/usr/sbin/nologin
-sys:x:3:3:sys:/dev:/usr/sbin/nologin   <== 左侧文件被删除
 sync:x:4:65534:sync:/bin:/bin/sync     <== 左侧文件被删除
-games:x:5:60:games:/usr/games:/usr/sbin/nologin
+no six line                            <== 右侧新文件加入
 man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
 lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
 mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
```
一般来说,使用`diff`制作出来的比较文件通常使用扩展名为`.patch`啰。至于内容就如同上面介绍的样子。基本上就是以行为单位,看看哪边有一样与不一样的,找到一样的地方,然后将不一样的地方取代掉! 以上面表格为例,新文件看到-会删除,看到+会加入!好了,那么如何将**旧的文件更新成为新的内容**呢? 就是将`pd`改成与`pd.new`相同!可以这样做:
```
patch -pN < patch_file <==更新
patch -R -pN < patch_file <==还原
# paras:
-p :后面可以接“取消几层目录”的意思。
-R :代表还原,将新的文件还原成原来旧的版本。


```
<b style="color: red">范例二：</b>将刚才制作出来的`patch file`用来更新旧版数据
```
patch -p0 < pd.patch
# output
patching file pd

heany@heany:~/Desktop/shell-learning$ ll pd*
-rw-r--r-- 1 heany heany 2253 4月  11 13:43 pd
-rw-r--r-- 1 heany heany 2253 4月  11 13:08 pd.new

## 两个文件大小一模一样
```
<b style="color: red">范例三：</b>将刚才制作出来的`patch file`恢复旧文件的内容
```
patch -R -p0 < pd.patch
# output
patching file pd

heany@heany:~/Desktop/shell-learning$ ll pd*
-rw-r--r-- 1 heany heany 2326 4月  11 13:47 pd
-rw-r--r-- 1 heany heany 2253 4月  11 13:08 pd.new

## 文件恢复成旧的版本了
```
>  为什么这里会使用`-p0`呢?因为我们在比对新旧版的数据时是在同一个目录下,因此不需要减去目录啦!如果是使用整体目录比对(diff旧目录 新目录)时, 就得要依据创建`patch`文件件所在目录来进行目录的删减啰!



#### 3.4.4、 文件打印准备：`pr`
如果你曾经使用过一些图形接口的文书处理软件的话,那么很容易发现,当我们在打印的时
候, 可以同时选择与设置每一页打印时的标头吧!也可以设置页码呢!那么,如果我是在
Linux 下面打印纯文本文件呢 可不可以具有标题啊?可不可以加入页码啊?呵呵!当然可以
啊!使用 pr 就能够达到这个功能了。不过, pr 的参数实在太多了,说不完,一般来
说,仅使用最简单的方式来处理就行。举例来说,如果想要打印`pd`呢?
```
heany@heany:~/Desktop/shell-learning$ pr pd

2018-04-11 13:47                        pd                        Page 1

root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
....................下面省略..........

```



## 四、 学习<b style="color: red">Shell Scripts</b>
### 4.1、什么是shell script
`shell script`可以简单的被看成是批处理文件,也可以被说成是一个程序语言,且这个程序语言由于都是利用`shell`与相关工具指令, 所以不需要编译即可执行,且拥有不错的除错(debug)工具,所以,他可以帮助系统管理员快速的管理好主机。

`shell script`其实就是纯文本文件,我们可以编辑这个文件,然后让这个文件来帮我们一次执行多个指令,或者是利用一些运算与逻辑判断来帮我们达成某些功能。所以啦,要编辑这个文件的内容时,当然就需要具备有 bash 指令下达的相关认识。下达指令需要注意的事项在第四章的开始下达指令小节内已经提过,有疑问请自行回去翻阅。 在`shell script`的撰写中还需要用到下面的注意事项:

    1. 指令的执行是从上而下、从左到右的分析与执行;
    2. 指令的下达：指令、选项与参数间的多个空白都会被忽略掉;
    3. 空白行也将被忽略掉,并且 [tab] 按键所得的空白同样视为空格键;
    4. 如果读取到一个Enter符号（CR），就尝试开始执行该行（该串）命令;
    5. 至于如果一行的内容太多，则可以使用"[Enter]"来延伸至下一行;
    6. “ # ”可做为注解!任何加在 # 后面的数据将全部被视为注解文字而被忽略!
如此一来,我们在`script`内所撰写的程序,就会被一行一行的执行。现在我们假设你写的这个程序文件名是`/home/heany/Desktop/shell-learning/shell.sh`好了,那如何执行这个文件?很简单,可以有下面几个方法:

+ 直接下达指令：`shell.sh`文件必须要具备可读与可执行（rx）的权限，然后：

    绝对路径：使用`/home/heany/Desktop/shell-learning/shell.sh`来下达指令;<br>
    相对路径：假设工作目录在`/home/heany/Desktop/shell-learning`，则使用`./shell.sh`来执行;<br>
    变量`PATH`功能：将`shell.sh`放在PATH指定的目录内，例如：`~/bin/`

+ 以`bash`程序来执行：通过"bash shell.sh"或`sh shell.sh`来执行。
+ 编写第一个`script`
``` shell
#!/bin/bash
# Program:
#    This program shows "Hello World!" in your screen.
# History:
# 2018/04/11  VBird   First release
PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin
export PATH
echo -e "Hello World! \a \n"
exit 0

```
程序说明如下：

+ **第一行`#!/bin/bash`在宣告这个script使用的shell名称**: 因为我们使用的是bash ,所以,必须要以`#!/bin/bash`来宣告这个文件内的语法使用bash语法!那么当这个程序被执行时,他就能够载入`bash`的相关环境配置文件 (一般来说就是`non-login shell`的 ~/.bashrc), 并且执行`bash`来使我们下面的指令能够执行!这很重要的!(在很多状况中,如果没有设置好这一行,那么该程序很可能会无法执行,因为系统可能无法判断该程序需要使用什么 shell 来执行啊!)
+ **程序内容的说明**: 整个 script 当中,除了第一行的`#!`是用来宣告shell的之外,其他的`#`都是“注解”用途! 所以上面的程序当中,第二行以下就是用来说明整个程序的基本数据。一般来说, 建议你一定要养成说明该 script 的:1. 内容与功能; 2. 版本信息; 3.作者与联络方式; 4. 创建日期;5. 历史纪录 等等。这将有助于未来程序的改写与 debug呢!
+ **主要环境变量的声明**: 建议务必要将一些重要的环境变量设置好,个人认为,`PATH`与`LANG` (如果有使用到输出相关的信息时) 是当中最重要的! 如此一来,则可让我们这支程序在进行时,可以直接下达一些外部指令,而不必写绝对路径呢!比较方便啦！
+ **主要程序部分**就将主要的程序写好即可！在这个例子当中，就是echo那一行。
+ **告知执行结果**：一个命令的执行成功与否，可以使用`$?`这个变量来观察～那么我们也可以利用`exit`这个指令来让程序中断，并且回传一个数值给系统。在我们这个例子当中,我使用`exit 0`,这代表离开`script`并且回传一个`0`给系统, 所以我执行完这个`script`后,若接着下达`echo $?` 则可得到`0`的值喔! 更聪明的读者应该也知道了!利用这个`exit n`(n 是数字) 的功能,我
们还可以自订错误讯息, 让这支程序变得更加的`smart`呢!
```
chmod a+x hello.sh
./hello.sh
# output
Hello World!  
```

+ <b style="color: red;font-size: 24px">编写`shell script`的良好习惯</b>

一个良好习惯的养成是很重要的~大家在刚开始撰写程序的时候,最容易忽略这部分, 认为
程序写出来就好了,其他的不重要。其实,如果程序的说明能够更清楚,那么对你自己是有
很大的帮助的。<b style="color: red">要学会比较仔细的将程序的设计过程给它记录下来，而且记录一些历史记录</b>。在每个`script`的文件开始处记录好：

    1. `script`的功能
    2. `script`的版本信息
    3. `script`的作者与联系方式
    4. `script`的版权声明方式
    5. `script`的`History`（历史记录）
    6. `script`内比较特殊的指令，使用“绝对路径”的方式下达
    7. `script`运行时需要的环境变量预先宣告与设置
    8. 在较为特殊程序代码部分，加上适当的注释说明

>  程序代码的编写最好使用嵌套方式，**最好能以`[tab]`按键的空格缩排**这样代码看起来非常漂亮与有条理。


### 4.2、 简单的`shell`练习
#### 4.2.1、 简单范例
+ 交互式脚本：变量内容由用户决定

请你以`read`命令的用途,编写一个`script`,他可以让使用者输入first name与last name,最后并且在屏幕上显示:“Your full name is: ”的内容:
```
# showname.sh
#!/bin/bash
# paragram:
#       User inputs his first name and last name. Program shows his full name.
# History
# 2018/04/11   heany  First release
PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin
export PATH
# 提示使用者输入
read -p "Please input your first name: " firstname
# 提示使用者输入
read -p "Please input your last name: " lastname
# 结果由屏幕输出
echo -e "\nYour full name is: ${firstname} ${lastname}" 

## 执行脚本
 .  showname.sh
Please input your first name: heany
Please input your last name: boy

Your full name is: heany boy
```
+ 随日期变化：利用`data`进行文件的创建

想像一个状况,假设我的服务器内有数据库,数据库每天的数据都不太一样,因此当我备份时,希望将每天的数据都备份成不同的文件名,这样才能够让旧的数据也能够保存下来不被覆盖。哇!不同文件名呢!这真困扰啊?难道要我每天去修改`script`?

不需要啊!考虑每天的“日期”并不相同,所以我可以将文件名取成类似:`backup.2015-07-16.data`, 不就可以每天一个不同文件名了吗?呵呵!确实如此。那个`2015-07-16`怎么来的?那就是重点啦!接下来出个相关的例子: 假设我想要创建三个空的文件 (通过 touch)文件名最开头由使用者输入决定,假设使用者输入`filename`好了,那今天的日期是`2015/07/16`, 我想要以前天、昨天、今天的日期来创建这些文件,亦即filename_20150714,filename_20150715, filename_20150716 ,该如何是好?
```
#!/bin/bash
# program:
#       Program creates three files, which named by user's input and date command.
# History:
# 2018/04/11    heany   First release
PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin
export PATH
# 1. 让使用者输入文件名称,并取得 fileuser 这个变量;
echo -e "I will use 'touch' command to create 3 files." # 纯粹显示信息
read -p "Please input your filename: " fileuser         # 提示使用者输入

# 2. 为了避免使用者随意按Enter,利用变量功能分析文件名是否有设置
filename=${fileuser:-"filename"}                # 开始判断有否配置文件名

# 3. 开始利用 date 指令来取得所需要的文件名了;
date1=$(date --date='2 days ago' +%Y%m%d)       # 前两天的日期
date2=$(date --date='1 days ago' +%Y%m%d)       # 前一天的日期
date3=$(date +%Y%m%d)                           # 今天的日期
file1=${filename}${date1}                       # 下面三行在配置文件名
file2=${filename}${date2}
file3=${filename}${date3}
```
>  在一串命令中如果还需要通过其他的命令提供的信息，可以使用反单引号\`命令\`或`$`(命令)。`filename=${fileuser:-"filename"}  `，这条命令用来判断`fileuser`是否已经赋值，`:-`是一起的，`fileuser`如果有值的话，就用所拥有的值赋给`filename`变量，如果没有值的话，就把"filename"赋给`fileuser`，再赋给`filename`变量。

```
v=$(uname -r)
echo $v
# output
4.13.0-38-generic

vv=`uname -r`
echo $vv
# output
4.13.0-38-generic
```
+ 数值运算：简单的加减乘除

我们可以使用`declare`来定义变量的类型吧? 当变量定义成为整数后才能够进行加减运算啊!此外,我们也可以利用**`$((计算式))`**来进行数值运算的。可惜的是,`bash shell`里头默认仅支持到整数的数据而已。OK!那我们来玩玩看,如果我们要使用者输入两个变量, 然后将两个变量的内容相乘,最后输出相乘的结果,那可以怎么做?
``` shell
#!/bin/bash
# Program:
#       User inputs 2 integer numbers; program will cross thest two numbers.
# History:
# 2018/04/11    heany   First release

PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin
export PATH

echo -e "You SHOULD input 2 numbers, I will multiplying them! \n"
read -p "first number: " firstnu
read -p "second number: " secnu
total=$((${firstnu}*${secnu}))
echo -e "\nThe result of $firstnu x $secnu is ==> $total"
                                                                       
```
>  在数值计算上，我们还可以使用**`declare -i total=$fistnu*$secnu`**,不推荐使用这种方式，因为比较繁琐，不容易记。使用上面程序中的那种方式来计算数值。

如果想要计算含有小数点的数据时，其实可以通过`bc`这个指令的协助，例如：

    echo "123.123*55.9" | bc
    # output
    6882.575



### 4.3、 善用判断式
`$?`这个变量代表的是上一条命令执行的情况，若没问题则回传码为`0`,若存在问题，回传错误代码(非0), 此外,也通过`&&`及`||`来作为前一个指令执行回传值对于后一个指令是否要进行的依据。我们知道想要判断一个目录是否存在, 之前我们使用的是ls这个指令搭配数据流重导向,最后配合`$?`来决定后续的指令进行与否。但是否有更简单的方式可以来进行“条件判断”呢?有的~那就是`test`这个指令。

#### 4.3.1、 利用`test`指令的测试功能
当我们要检测系统上面某些文件或者是相关的属性时，利用`test`这个指令来工作真是好用得炸了，举例来说，我要检查`/heany`是否存在时，使用：

    test -e /heany
执行结果不会显示任何信息，但最后我们可以通过`$?`或`&&及||`来展现整个结果！例如我们在将上面的例子改写成这样：

    test -e /heany && echo "exist" || echo "No exist"
    # output
    No exist  ==> 结果显示不存在
最终的结果可以告知我们是`exist`还是`Not exist`呢!那我知道`-e`是测试一个“东西”在不在,
如果还想要测试一下该文件名是啥玩意儿时,还有哪些标志可以来判断的呢?呵呵!有下面这些东西喔!

1. 关于某个文件名的"文件类型"判断，如`test -e filename`表示存在与否

| 测试的标志 |   代表意义     |
| :-----: | :-----: |
| -e |  该文件名是否存在 |
| -f |  该文件名是否存在且问文件 |
| -d |  该文件名是否存在且为目录  |
| -b |  该文件名是否存在且为一个`block device`设备  |
| -c |  该文件名是否存在且为一个`character device`设备  |
| -S |  该文件名是否存在且为一个Socket文件  |
| -p |  该文件名是否存在且为一个FIFO（pipe）文件  |
| -L |  该文件名是否存在且为一个链接文件  |

2. 关于文件的权限侦测，如`test -r filename`表示可读否（但root权限常有例外）

<img src="https://wx4.sinaimg.cn/mw1024/e0db46edgy1fq90rrorbrj20m607p0u0.jpg" align="center">

3. 两个文件之间的比较，如：`test file1 -nt file2`

| 测试的标志 |   代表意义     |
| :-----: | :----: |
| -nt | (newer than)判断`file1`是否比`file2`新 |
| -ot | (older than)判断`file1`是否比`file2`旧|
| -ef | 判断 file1 与 file2 是否为同一文件,<br>可用在判断`hard link`的判定上。<br>主要意义在判定,两个文件是否均指向同一个`inode` |

4. 关于两个整数之间的判定，例如：`test n1 -eq n2`

<img src="https://wx4.sinaimg.cn/mw1024/e0db46edgy1fq90rrpaszj20mf06n74s.jpg" align="center">

5. 判定字串的数据

<img src="https://wx3.sinaimg.cn/mw1024/e0db46edgy1fq90rrp8zij20mc050mxw.jpg" align="center">

6. 多重条件判定，例如：`test -r filename -a -x filename`

| 测试的标志 |   代表意义     |
| :-----: | :----: |
| -a | 两个条件同时成立！<br>例如 `test -r file -a -x file`,则file同时具有r与x权限时。才回传true|
| -o | (or)两状况任何一个成立!例如 test -r file -o -x file,则 file 具有 r 或 x 权限时,就可回传 true。|
| ! | 反相状态,如`test ! -x file`,当file不具有x时,回传`true`|

现在我们利用`test`来帮我们写几个简单的例子。首先，判断一下，让使用者输入一个文件名，我们判断：

1. 这个文件是否存在，若不存在则给予一个"Filename does not exist"的信息，并中断程序; 
2. 若这个文件存在，则判断他是个文件或目录，结果输出“Filename is regular file”或"Filename is directory"
3. 判断一下，执行者的身份对这个文件或目录所拥有的权限，并输出权限数据！

``` shell
#!/bin/bash
# Program:
#       User input a filename, program will check flowing:
#       1. exist?  2. file/directory?  3. file permisions
# History:
# 2018/04/11    heany   First release

PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin
export PATH

# 1. 让使用者输入文件名,并且判断使用者是否真的有输入字串?
echo -e "Please input a filename, I will check the filename's type and permission. \n\n"
read -p "Input a filename : " filename
test -z ${filename} && echo "You MUST input a filename." && exit 0

# 2. 判断文件是否存在?若不存在则显示信息并结束脚本
test ! -e ${filename} && echo "The filename '${filename}' DO NOT exist" && exit 0

# 3. 开始判断文件类型与属性
test -f ${filename} && filetype="regulare file"
test -d ${filename} && filetype="directory"
test -r ${filename} && perm="readable"
test -w ${filename} && perm="${perm} writable"

# 4. 开始输出信息!
echo "The filename: ${filename} is a ${filetype}"
echo "And the permissions for you are : ${perm}"
```


#### 4.3.2、 利用判断符号`[]`
除了我们很喜欢使用的`test`之外,其实,我们还可以利用判断符号`[ ]`(就是中括号啦)来进行数据的判断呢! 举例来说,如果我想要知道`${HOME}`这个变量是否为空的,可以这样做:

    [ -z "$HOME"] ; echo $?
>  使用中括号必须要特别注意,因为中括号用在很多地方,包括万用字符与正则表达式等等,所以如果要在 bash 的语法当中使用中括号作为`shell`的判断式时,必须要注意<b style="color: red">中括号的两端需要有空白字符来分隔!</b> 假设我空白键使用“□”符号来表示,那么,在这些地方你都需要有空白键:

    [ "$HOME" == "$MAIL"]
    [□"$HOME"□==□"$MAIL"□]
可是差很多的喔!另外,**中括号的使用方法与`test`几乎一模一样啊~ **只是中括号比较常用在条件判断式`if ..... then ..... fi`的情况中就是了。好,那我们也使用中括号的判断来做一个小案例好了,案例设置如下:

    1. 当执行一个程序的时候,这个程序会让使用者选择 Y 或 N ,
    2. 如果使用者输入Y或y时，就显示"OK， continue"
    3. 如果使用者输入 n 或 N 时,就显示“ Oh, interrupt !”
    4. 如果不是 Y/y/N/n 之内的其他字符,就显示“ I don't know what your choice is ”

利用中括号、&&与||来继续吧！
``` shell
#!/bin/bash
# Program:
#       This program shows the user's choice.
# History:
# 2018/04/11    heany   First release

PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin
export PATH

read -p "Please input (Y/N): " yn
[ "${yn}" == "Y" -o "${yn}" == "y" ] && echo "OK, continue" && exit 0
[ "${yn}" == "N" -o "${yn}" == "n" ] && echo "Oh, interrupt!" && exit 0
echo "I don't know what your choice is" && exit 0
```
>  这里使用`-o`来链接两个判断


#### 4.3.3、 **`Shell Script`**的默认变量（$0,$1...）
我们知道指令可以带有选项与参数,例如`ls -la`可以察看包含隐藏文件的所有属性与权限。那么`shell script`能不能在脚本文件名后面带有参数呢?很有趣喔!举例来说,如果你想要重新启动系统的网络,可以这样做:

    file /etc/init.d/network
    /etc/init.d/network restart   # 重启

restart 是重新启动的意思,上面的指令可以“重新启动 /etc/init.d/network 这支程序”的意思!唔!那么如果你在 /etc/init.d/network 后面加上 stop 呢?没错!就可以直接关闭该服务了!这么神奇啊? 错啊!如果你要依据程序的执行给予一些变量去进行不同的任务时,本章一开始是使用 read 的功能!但 read 功能的问题是你得要手动由键盘输入一些判断式。如果通过指令后面接参数, 那么一个指令就能够处理完毕而不需要手动再次输入一些变量行为!这样下达指令会比较简单方便啦!

script 是怎么达成这个功能的呢?其实 script 针对参数已经有设置好一些变量名称了!对应如下:
```
/path/to/scriptname     opt1    opt2    opt3    opt4
        $0               $1      $2      $3      $4
```
这样够清楚了吧?执行的脚本文件名为`$0`这个变量,第一个接的参数就是`$1`啊~ 所以,只要我们在 script 里面善用`$1`的话,就可以很简单的立即下达某些指令功能了!除了这些数字的变量之外, 我们还有一些较为特殊的变量可以在`script`内使用来调用这些参数喔!

    $# :代表后接的参数“个数”,以上表为例这里显示为“ 4 ”;
    $@ :代表“ "$1" "$2" "$3" "$4" ”之意,每个变量是独立的(用双引号括起来);
    $* :代表“ "$1<u>c</u>$2<u>c</u>$3<u>c</u>$4" ”,<br>
        其中 <u>c</u> 为分隔字符,默认为空白键, 所以本例中代表“ "$1 $2 $3 $4" ”之意。
来练习：假设我们要执行一个可以携带参数的`script`，执行该脚本后屏幕会显示如下数据：

    程序的文件名为何？
    共有几个参数？
    若参数的个数小于2则告知使用者参数数量太少
    全部的参数内容为何？
    第一个参数为何？
    第二个参数为何？

``` shell
#!/bin/bash
# Program:
#       Program shows the script name, parameters and so on....
# History:
# 2018/04/12    heany   First release

PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin
export PATH

echo "The script name is ===>  $0"
echo "Total parameter number is ===> $#"
[ "$#" -lt 2 ] && echo "The number of parameter is less than 2.  Stop here."\
        && exit 0
echo "Your whole parameter is ==> '$@'"
echo "The 1st parameter   ===> $1"
echo "The 2nd parameter   ===> $2"

# 脚本执行
sh how_paras.sh I am a boy
# output
The script name is ===>  how_paras.sh
Total parameter number is ===> 4
Your whole parameter is ==> 'I am a boy'
The 1st parameter   ===> I
The 2nd parameter   ===> am


```

+ `shift`：造成参数变量号码偏移

什么是偏移（shift）呢？我们用下面的范例来说明下，我们将`how_paras.sh`的内容稍作变化，用来显示每次偏移后参数的变化情况：
``` shell
#!/bin/bash
# Program:
#       Program shows the effect of shift function....
# History:
# 2018/04/12    heany   First release

PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin
export PATH

echo "Total parameter number is ===>  $#"
echo "Your whole parameter is   ===> '$@'"

shift   # 进行第一次"一个变量的 shift"
echo "Total parameter number is ===>  $#"
echo "Your whole parameter is   ===> '$@'"

shift 3 #进行第二次"三个变量的 shift"
echo "Total parameter number is ===>  $#"
echo "Your whole parameter is   ===> '$@'"

# execute script
sh paras_shift.sh one two three four five six 
# output
Total parameter number is ===>  6
Your whole parameter is   ===> 'one two three four five six'
Total parameter number is ===>  5
Your whole parameter is   ===> 'two three four five six'
Total parameter number is ===>  2
Your whole parameter is   ===> 'five six'
```
光看结果你就可以知道啦,那个`shift`会移动变量,而且`shift`后面可以接数字,代表拿掉最前面的几个参数的意思。 上面的执行结果中,第一次进行`shift` 后他的显示情况是`one two three four five six`,所以就剩下五个啦!第二次直接拿掉三个,就变成`two three four five six`啦!这样这个案例可以了解了吗?理解了`shift`的功能了吗?


### 4.4、 条件判断式
#### 4.4.1、 利用`if .... then`

+ 单层、简单条件判断

如果你只有一个判断式要进行，那么我们可以简单的这样看：

    if [ 条件判断式 ]; then
        当条件判断式成立时，可以进行命令工作内容
    fi <=== 将`if`反过来写，就成为`fi`，结束`if`的意思。

条件判断式有多个条件时，可以将多个条件写入一个中括号里面，也可以有多个中括号来隔开。括号之间，则用`&&与||`来隔开。举上面的`ans_yn.sh`例子来说明，修改里面的判断式：

    [ "$yn" == "Y" -o "$yn" == "y" ] 上式可替换为 [ "$yn" == "Y" ] || [ "$yn" == "y" ]

>  这样符合人们的习惯问题，我们都喜欢一个中括号仅有一个判别式。接下来，我们将`ans_yn.sh`这个脚本修改成为`if ... then`的样式：

```
#!/bin/bash
# Program:
#       This program shows the user's choice.
# History:
# 2018/04/11    heany   First release

PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin
export PATH

read -p "Please input (Y/N): " yn

if [ "$yn" == "Y" ] || [ "$yn" == "y" ]; then
        echo "OK, continue."
        exit 0
fi

if [ "$yn" == "N" ] || [ "$yn" == "n" ]; then
        echo "Oh, interrupt!"
        exit 0
fi

echo "I don't know what your choice is" && exit 0

```

+ 多重、复杂条件判断式

在同一个数据的判断中,如果该数据需要进行多种不同的判断时,应该怎么作?举例来说,上面的 ans_yn.sh 脚本中,我们只要进行一次 ${yn} 的判断就好 (仅进行一次 if ),不想要作多次 if 的判断。 此时你就得要知道下面的语法了:

    # 一个条件判断，分成功进行与失败进行(else)
    if [ 条件判断式一 ]; then
        当条件判断式成立时，可以进行的指令工作内容：
    elif [ 条件判断式二 ]; then
        当条件判断式二成立时，可以进行的指令工作内容
    else
        当条件判断式一和二都不成立时，可以进行的指令工作内容;
    fi 
一般来说，如果你不希望使用者由键盘输入额外的数据时，可以使用上面提到的参数功能（$1），让使用者在下达命令时就将参数带进去！现在我们想让使用者输入`hello`这个关键字时，利用参数的方法可以这样依序设计：

    1. 判断`$1`是否为hello，如果是的话，就显示“hello，how are you？”
    2. 如果没有加任何参数,就提示使用者必须要使用的参数下达法;
    3. 而如果加入的参数不是 hello ,就提醒使用者仅能使用 hello 为参数。

之前我们已经学会了`grep`这个好用的玩意儿，那么多学一个叫做`netstat`的命令，这个指令可以查询到目前主机有打开的网络服务端口（service ports）,我们可以利用`netstat -tuln`来取得目前主机有启动的服务，而且取得的信息有点像这样：
```
netstat -tuln
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State      
tcp        0      0 0.0.0.0:5355            0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN     
tcp        0      0 127.0.0.1:631           0.0.0.0:*               LISTEN     
tcp        0      0 127.0.0.1:1080          0.0.0.0:*               LISTEN    
udp        0      0 0.0.0.0:5353            0.0.0.0:*                          
udp        0      0 0.0.0.0:5353            0.0.0.0:*                      
#封包格式           本地IP：端口              远程IP：端口              是否监听  
```
上面的重点是“Local Address (本地主机的IP与端口对应)”那个字段,他代表的是本机所启动的网络服务! IP的部分说明的是该服务位于那个接口上,若为 127.0.0.1 则是仅针对本机开放,若是 0.0.0.0 或 ::: 则代表对整个`Internet`开放。 每个端口 (port) 都有其特定的网络服务,几个常见的 port 与相关网络服务的关系是:

    80：www
    22:ssh
    21:ftp
    25:mail
    111:RPC(远端程序调用)
    631:CUPS（打印服务功能）
假设我的主机有兴趣要侦测的是比较常见的 port 21, 22, 25及 80 时,那我如何通过`netstat`去侦测我的主机是否有打开这四个主要的网络服务端口呢?由于每个服务的关键字都是接在冒号`:`后面, 所以可以借由提取类似`:80`来侦测的!那我就可以简单的这样去写这个程序喔:

#### 4.4.2、 利用`case ... esac`
语法如下：

    case $变量名称 in      <=== 关键字为case。还有变量前有$
        "第一个变量内容")   <=== 每个变量内容建议用双引号括起来，关键字则为小括号）
            程序段
            ;;            <=== 每个类别结尾i使用两个连续的分号来处理
        "第二个变量内容"）
            程序段
            ;;
        *)                <=== 最后一个变量内容都会用*来代表所有其他值
            不包括第一个变量内容与第二个变量内容的其他程序执行段
            exit 1
            ;;
    esac                  <=== 最终的case结尾，“反过来写”
这么说或许你的感受性还不高,好,我们直接写个程序来玩玩:让使用者能够输入 one, two,three , 并且将使用者的变量显示到屏幕上,如果不是 one, two, three 时,就告知使用者仅有这三种选择。
```
#!/bin/bash
# Program:
#        This script only accepts the flowing parameter: one, two or three.
# History:
# 2018/04/12    heany   Fist release
PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin
export PATH

echo "This program will print your selection !"
# read -p "Input your choice: " choice          # 暂时取消,可以替换!
# case ${choice} in                     # 暂时取消,可以替换!

case ${1} in            # 现在使用,可以用上面两行替换!
        "one")
                echo "Your choice is ONE"
                ;;
        "two")
                echo "Your choice is TWO"
                ;;
        "three")
                echo "Your choice is THREE"
                ;;
        *)
                echo "Usage ${0} {one|two|three}"
                ;;
esac

```

#### 4.4.3、 利用`function`功能
什么是“函数 (function)”功能啊?简单的说,其实, 函数可以在 shell script 当中做出一个类似自订执行指令的东西,最大的功能是, 可以简化我们很多的程序码~举例来说,上面的show123.sh 当中,每个输入结果 one, two, three 其实输出的内容都一样啊~那么我就可以使用 function 来简化了! function 的语法是这样的:

    function fname(){
        程序段
    }

>   由于`shell script`的执行方式是由上而下,由左而右, 因此在`shell script`当中的<b style="color: red">`function`的设置一定要在程序的最前面</b>, 这样才能够在执行时被找到可用的程序段喔(这一点与传统程序语言差异相当大!初次接触的朋友要小心!)! 好~我们将上面的脚本改写
一下,自定义一个名为`printit`的函数来使用:

```
# History:
# 2018/04/12    heany   First release

PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin
export PATH

function printit(){
        echo -n "Your choice is "     # 加上 -n 可以不换行继续在同一行显示
}

echo "This program will print your selection !"
case ${1} in
        "one")
                printit; echo $1 | tr 'a-z' 'A-Z'  # 将参数做大小写转换!
                ;;
        "two")
                printit; echo $1 | tr 'a-z' 'A-Z'
                ;;
        "three")
                printit; echo $1 | tr 'a-z' 'A-Z'
                ;;
        *)      
                echo "Usage ${0} {one|two|three}"
"show123_function.sh" 29L, 591C written                       11,53-55      50%

```




### 4.5、 循环（`loop`）
除了 if...then...fi 这种条件判断式之外,循环可能是程序当中最重要的一环了~ 循环可以不断的执行某个程序段落,直到使用者设置的条件达成为止。所以,重点是那个“条件的达成”是什么。除了这种依据判断式达成与否的不定循环之外,循环形态,可称为固定循环的形态呢!下面我们就来谈一谈:

#### 4.5.1、 `while do done`,`until do done`（不定循环）
一般来说，不定循环最常见的就是下面这两种状态了：

    while [ condition ] <=== 中括号内的状态就是判别式
    do
        代码段
    done
`while`的中文是“当...时”，所以，这种方式说的是当`condition`条件成立时，就进行循环，直到`condition`的条件不成立才停止的意思。还有另为一种不定循环的方式：

    until [ condition ]
    do
        代码段
    done
这种方式恰恰与 while 相反,它说的是“当 condition 条件成立时,就终止循环, 否则就持续
进行循环的程序段。”是否刚好相反啊~我们以 while 来做个简单的练习好了。 假设我要让使
用者输入 yes 或者是 YES 才结束程序的执行,否则就一直进行告知使用者输入字串。
``` shell
#!/bin/bash
# Program:
#       Repeat question until user input correct answer.
# History:
# 2018/04/12    leagle  First release

# while
while [ "$yn" != "yes" -a "$yn" != "YES" ] 
do
        read -p "Please input yes/YES to stop this program: " yn
done    
echo "OK! you input the correct answer."

# until
until [ "$yn" == "yes" -o "$yn" == "YES" ] 
do
        read -p "Please input yes/YES to stop this program: " yn
done
echo "OK! you input the correct answer."
```


#### 4.5.2、 `for ... do ...done`(固定循环)
`for`循环的语法如下所示：

    for var in con1 con2 con3 ...
    do
        code
    done
简单练习。
```
#!/bin/bash
# Program:
#       Using for ... loop to print 3 animal.
# History:
# 2018/04/12    leagle  First release

for animal in dog cat elephant
do      
        echo "There are ${animal}s ...."
done 
```
让我们想像另外一种状况,由于系统上面的各种帐号都是写在 /etc/passwd 内的第一个字段,你能不能通过管线命令的 cut 捉出单纯的帐号名称后,以`id`分别检查使用者的识别码与特殊参数呢?由于不同的 Linux 系统上面的帐号都不一样!此时实际去捉`/etc/passwd`并使用循环处理,就是一个可行的方案了!程序可以如下:
```
#!/bin/bash
# Program:
#       Use id, finger command to check system account's information.
# History:
# 2018/04/12    leagle  First release

users=$(cut -d ':' -f1 ./pd.txt)         # 提获帐号名称
for username in ${users}        # 开始循环进行!
do
        id ${username}
done
```
执行上面的脚本后,你的系统帐号就会被捉出来检查啦!这个动作还可以用在每个帐号的删除、重整上面呢! 换个角度来看,如果我现在需要一连串的数字来进行循环呢?举例来说,我想要利用 ping 这个可以判断网络状态的指令, 来进行网络状态的实际侦测时,我想要侦测的网域是本机所在的 192.168.1.1~192.168.1.100,由于有 100 台主机, 总不会要我在 for 后面输入 1 到 100 吧?此时你可以这样做喔!
```
#!/bin/bash
# Program:
#       Use ping command to check the network's PC state.
# History:
# 2018/04/12    leagle  First release

network="192.168.1"     # 先定义一个网域的前面部分!

for sitenu in $(seq 1 100)        # seq 为 sequence(连续) 的缩写之意
do
        # 下面的程序在取得 ping 的回传值是正确的还是失败的!
        ping -c 1 -w 1 ${network}.${sitenu} &> /dev/null && result=0 || result=1

        # 开始显示结果是正确的启动 (UP) 还是错误的没有连通 (DOWN)

        if [ "${result}" == 0 ]; then
                echo "Server ${network}.${sitenu} is UP."
        else
                echo "Server ${network}.${sitenu} is DOWN."
        fi
done
```
>  注意那个`$(seq 1 100)`那个位置，那个是连续产生1-100个数。

最后,让我们来玩判断式加上循环的功能!我想要让使用者输入某个目录文件名,然后我找出某目录内的文件名的权限,该如何是好?可以这样做啦~
```
#!/bin/bash
# Program:
#       Use input dir name, I find the permission of files.
# History:
# 2018/04/12    leagle  First release

# 1. 先看看这个目录是否存在啊?
read -p "Please input a directory: " dir
if [ "${dir}" == "" -o ! -d "${dir}" ]; then
        echo "The ${dir} is NOT exist in your system."
        exit 1
fi

# 2. 开始测试文件~
filelist=$(ls ${dir})   # 列出所有在该目录下的文件名称

for filename in ${filelist}
do
        perm=""
        test -r "${dir}/${filename}" && perm="${perm} readable"
        test -w "${dir}/${filename}" && perm="${perm} writable"
        test -x "${dir}/${filename}" && perm="${perm} executable"
        echo "The file ${dir}/${filename}'s permission is ${perm} "
done
```


#### 4.5.3 `for ... do ...done`的数值处理
除了上述的方法之外，`for`循环还有另外一种写法，类似于`C语言`中的`for循环`！语法如下所示：

    for ((初始值;限制值;执行步长))
    do
        code
    done

这种语法很适合用于数值方式的运算当中，值得注意的是,在“执行步阶”的设置上,如果每次增加 1 ,则可以使用类似`i++`的方式,亦即是 i 每次循环都会增加一的意思。好,我们以这种方式来进行 1 累加到使用者输入的循环吧!
```
#!/bin/bash
# Program:
#       Try to calculate "1+2+3+...+${yourinput}".
# History:
# 2018/04/12    leagle  First release

read -p "Please input a number, I will count for 1+2+...+your_input: " nu

s=0
for (( i=1; i<=$nu; i=i+1))
do      
        s=$(($s+$i))
done    
echo "The result of '1+2+...+$nu' is ===> $s"
```



### 4.6、 `Shell Script`的追踪与`Debug`
`scripts`在执行之前,最怕的就是出现语法错误的问题了!那么我们如何`debug` 呢?有没有办法不需要通过直接执行该`scripts`就可以来判断是否有问题呢?呵呵!当然是有的!我们就直接以 bash 的相关参数来进行判断吧!

    # 语法如下：
    sh [-nvx] scripts.sh
    # paras:
    -n ：不要执行script，仅查询语法的问题;
    -v ：在执行script之前，先将scripts的内容输出到屏幕上;
    -x ：将使用到的script内容显示到屏幕上，这是很有用的参数！
一些范例测试：
```
bash -x ans_yn-2.sh 
+ PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:/home/heany/bin
+ export PATH
+ read -p 'Please input (Y/N): ' yn
Please input (Y/N): y
+ '[' y == Y ']'
+ '[' y == y ']'
+ echo 'OK, continue.'
OK, continue.
+ exit 0
```


## 五、 结束语
&emsp;&emsp;这个系列终于结束了，码字调bug很辛苦哇～但是从中学到了很多有用的知识，解决了之前很多搞不清楚的问题，总之很棒啦！本文转载自<b>鸟哥的Linux私房菜：基础学习篇 第四版</b>。总之这本书非常适合入门学习～强烈推荐！！！