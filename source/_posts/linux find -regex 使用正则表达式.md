---
title: linux find -regex 使用正则表达式.md
date: 2024-06-10 06:44:39
tags:
---

https://www.cnblogs.com/jiangzhaowei/p/5451173.html

find之强大毋庸置疑，此处只是带领大家一窥find门径，更详细的说明见man  find和 info find。
整篇文章循序渐进，从最常用的文件名测试项开始步步深入，到第六节基本讲完find处理文件的规则，再之后的章节是一些常用表达式的说明。
（此篇中所有选项及例子基于GNU find version 4.2.28）


（一）Get Start
最简单的find用法莫过于如此：

$ find .
查找当前目录下的所有文件。
find命令的一般格式为：

find [-H] [-L] [-P] [path...] [expression]

其中，'-H' '-L' '-P'三个选项主要是用来处理符号连接，'-H'表示只跟随命令行中指定的符号连接，'-L'表示跟随所有的符号连接，'-P'是默认的选项，表示不跟随符号连接。
例如，在我的当前目录下有一个符号连接e1000，现在我想查找文件名中最后一个字母是数字的源文件，那么

$ find -H . -name "*[0-9].c" -print
./2234.c
像上面这样写只能查找出当前目录下符合要求的文件，却找不出e1000下的文件。因此可以这么写：

$ find -H e1000 . -name "*[0-9].c" -print
或者使用 '-L'选项

$ find -L . -name "*[0-9].c" -print

格式中的[path...]部分表示以此目录为根目录进行搜索。

格式中的[expression]是一个表达式。最基本的表达式分为三类：设置项(option)、测试项(test)、动作项(action)，这三类又可以通过逻辑运算符(operator)组合在一起形成更大更复杂的表达式。设置项（如-depth,-maxdepth等）针对这次查找任务，而不是仅仅针对某一个文件，设置项总是返回true；测试项(test)则不同，它针对具体的一个文件进行匹配测试，如-name,-num,-user等，返回true或者false；动作项(action)则是对某一个文件进行某种动作（最常见的如-print），返回true或者false。
正是[expression]部分的丰富，才使得find如此强大。此部分较复杂，后面慢慢说明。



（二）文件名
根据文件名来查找一个文件是大家经常遇到的事情，第一节中的'-name'正是解决此问题的。
-name属于表达式中的测试项(test)，它按照文件名模式来匹配文件，若匹配则返回true，否则返回false。最好用引号将文件名模式引起来，防止shell自己解析要匹配的字符串。（可以用单引号也可以用双引号，单引号和双引号在shell环境中的区别见后续部分）
例如，想要的当前目录及子目录中查找文件名以一个大写字母开头或者以小写a或b开头的文件，可以用：

$ find . -name "[A-Za-b]*" -print
./a_book_of_c.chm
./TMP1234
如果想在当前目录查找文件名不以大写字母开头，之后跟一个小写字母，再之后是两个数字，最后是.txt的文件，可以这么用：

$ find . -name "[^A-Z][a-z][0-9][0-9].txt" -print
./@y38.txt
注意：此处的模式匹配并不符合正则表达式。

-name对大小写字母敏感，如果想匹配时不考虑大小写可以使用-iname测试项。'i'可以加在许多选项前面，比如-ipath,-iregex,-iwholename等等，都是表示大小写不敏感。



（三）正则表达式
使用上面的-name测试项能解决许多问题，但是有些还是不太好办，比如：查找当前目录下名称全部为数字的c源代码文件，这时就该'-regex'出手了。正则表达式绝对值得你去好好研究一下，在unix系统下太有用了，这里不做过多说明，请读者自行学习。
-regex同样属于测试项。使用-regex时有一点要注意：-regex不是匹配文件名，而是匹配完整的文件名（包括路径）。例如，当前目录下有一个文件"abar9"，如果你用"ab.*9"来匹配，将查找不到任何结果，正确的方法是使用".*ab.*9"或者".*/ab.*9"来匹配。
针对上面的那个查找c代码的问题，可以这么写：

$ find . -regex ".*/[0-9]*/.c" -print
./2234.c

还有一个设置项(option)'-regextype'，可以让你根据自己的喜好选择使用的正则表达式类型，大家可以试试。



（四）wholename与path
既然上一节提到了完整文件名（包括路径名），那么这里不妨说一下-wholename和-path。
-wholename和-path都属于测试项(test)，而且功能也一样。-path从字面上看给人一种错觉，好像只匹配路径名（或者目录名），其实它也可以匹配文件名，因此-wholename这个名字更贴切一些。
看看这个例子，当前目录下有一个phone目录，phone目录里有一个文件名称是puk.txt，使用-path：

$ find . -path '*phone/pu*'
./phone/puk.txt

另外要提一点：使用-path的一般格式是：find [path ...] -path pattern ...
它的意思是：在[path ...]部分指明的路径上，使用pattern匹配所有文件的完整文件名；而不是说在类似的pattern目录下查找文件。



（五）逻辑运算符
有了上面三个选项，你现在应该对文件名的相关匹配得心应手了，对于不是很复杂的查找应该也胜任了。但是看看这个例子，解释一下它在做什么？

$ find . -size +0c -wholename "*e*[0-9]*" -o ! /( -name "." -o -name "*phone" /) -prune  -name "*.c" -user xixi -o -name "*phone"
下面是当前目录下的所有文件：

$ ls -l
total 224
-rw-r--r-- 1 xixi  admin      0 2007-11-01 17:34 0dfe.c
-rw-r--r-- 1 abc admin      0 2007-10-30 15:56 0s8a.txt
-rw-r--r-- 1 abc admin      0 2007-11-04 01:00 0TMP123
-rw-r--r-- 1 abc admin     73 2007-11-05 15:33 2234.c
-rw-r--r-- 1 abc admin     72 2007-11-05 15:34 3e10.c
-rw------- 1 abc admin 224017 2006-03-16 12:16 a_book_of_c.chm
lrwxrwxrwx 1 abc admin     15 2007-11-04 11:48 e1000 -> ../e1000-7.6.9/
-rw-r--r-- 1 abc admin     70 2007-11-05 14:57 e100.dat
lrwxrwxrwx 1 abc admin     13 2007-11-05 14:59 e100puk.txt -> phone/puk.txt
-rw-r--r-- 1 abc admin      0 2007-11-06 22:21 e680phone
drwxr-xr-x 2 abc admin     37 2007-11-06 22:24 phone
drwxr-xr-x 2 abc admin     20 2007-11-07 01:07 phone1
drwxr-xr-x 2 abc admin      6 2007-11-05 15:37 phone2
-rw-r--r-- 1 abc admin     67 2007-11-04 12:23 @y38.txt

phone$ ls -l
total 4
-rw-r--r-- 1 abc admin  0 2007-11-06 22:24 e680gphone
-rw------- 1 abc admin 38 2007-11-05 14:58 puk.txt

phone1$ ls -l
total 0
-rw-r--r-- 1 xixi admin 0 2007-11-07 01:07 hello.c

phone2$ ls -l
total 0


要想解决上面的问题就得学习一下find中的逻辑运算符。逻辑运算符主要有以下几个，按照优先级从高到低的顺序如下：
 ( expr )

括号优先级最高，首先对括号内的求值
 ! expr

对expr表达式的值取反
 -not expr

同上，但是POSIX不支持
 expr1 expr2

不加任何运算符，相当于两个之间加and，即与运算，两个表达式值都为true整个才返回true。先对expr1表达式求值，若为false，则不对expr2求值。
 expr1 -a expr2

同上
 expr1 -and expr2

同上，但是POSIX不支持
 expr1 -o expr2

表示对expr1和expr2两个表达式的值求或，左右两个值只要有一个为ture，整个表达式就是true。先对expr1表达式求值，若为ture，则不对expr2求值。
 expr1 -or expr2

同上，但是POSIX不支持
 expr1 , expr2

逗号表达式。expr1和expr2都会求值，但是只返回expr2的值，expr1的值会被丢弃

正是因为有一个求值的顺序，所以你才有可能见到这样的写法：

$ find . -name "*.txt" -o -print
表示，如果表达式-name "*.txt"为真，就不再执行另一个表达式-print，即查找所有不是以.txt结尾的文件。

再有，要查找当前目录下，文件名中包括字母'e'，在'e'之后又有数字的不是目录文件的所有文件，可以这么写：

$ find . -name "*e*[0-9]*" ! -type d -print
./e1000
./e100.dat
./e100puk.txt
./3e10.c
大家可以自己多举几个例子试一下。



（六）-prune
-prune是一个动作项，它表示当文件是一个目录文件时，不进入此目录进行搜索。
要理解-prune动作，首先得理解find命令的搜索规则（也可以说find命令的算法）。
find命令递归遍历所指定的目录树，针对每个文件依次执行find命令中的表达式，表达式首先根据逻辑运算符进行结合，然后依次从左至右对表达式求值。以下面代码为例，进行说明

find PATHP1 OPT1 TEST1 ACT1 ( TEST2 or TEST3 ) ACT2
(1) 根据OPT1设置项进行find命令的整体设置，若没有-depth设置项，依次进行下面的步骤
(2) 令文件变量File = PATHP1
(3) 对File文件进行TEST1测试，若执行结果为false，转(8)
(4) 对File文件进行ACT1动作，若执行结果为false，转(8)
(5) 对File文件进行TEST2测试，若执行结果为true，转(7)
(6) 对File文件进行TEST3测试，若执行结果为false，转(8)
(7) 对File文件进行ACT2动作
(8) 若File文件是一个目录，并且没有被执行过-prune动作，则进入此目录
(9) 当前目录下是否还有文件，若有依次取一个文件，令File指向此文件，转(3)；
(10) 判断当前目录是否是PATHP1，若是则程序退出；若不是，则返回上一层目录，转(9)

理解了上面的流程，那么不难理解下面的代码为什么只输出一个'.'

$ find . -prune
.
再有，当前目录下大于4090字节的文件有两个，而大于4096字节的文件只有一个，如下：

$ find . -size +4090c -print
.
./a_book_of_c.chm

$ find . -size +4096c -print
./a_book_of_c.chm

那么，将上面两个-print都替换为-prune，这两条命令分别输出什么？

$ find . -size +4090c -prune
.
$ find . -size +4096c -prune
./a_book_of_c.chm
这就是答案，如果你答对了，恭喜你，你已经掌握了find命令！

-prune经常和-path或-wholename一起使用，以避开某个目录，常见的形式是：

$ find PATH (-path <don't want this path #1> -o -path <don't want this path #2>) -prune -o -path <global expression for what I do want>

注意:如果同时使用-depth设置项，那么-prune将被find命令忽略。man手册页中这么说："If -depth is given, false; no effect."
说到这里，又得说说-depth设置项。网上好多资料说-depth设置项的功能是“在查找文件时，首先查找当前目录中的文件，然后再在其子目录中查找”，这明显是错误的，man手册页中如是说："-depth Process each directory's  contents before the directory itself."。这有点像树的后序遍历，先遍历当前节点的所有子节点，然后再访问当前节点...
考考你：
下面的命令输出什么？为什么？

$ find . -depth -prune -name "*.c" -print



（七）时间戳
理解了上面几节，你已经掌握了find命令的“道” ^_^ ，下面这几节只是介绍一些常用、好用的“招式”。这一节介绍时间戳。

文件有三个时间属性：创建时间、最近修改时间、最近访问时间。
最近修改时间又包括两种，一是文件的状态（也即权限如rwx等）最近被修改时间，一是文件的数据（也即内容）最近被修改时间。touch命令改变的即是文件数据最近被修改时间。
最近访问时间，指的是最近一次文件数据（内容）被访问的时间。因此，使用ls命令输出文件的相关信息并不会修改文件的最近访问时间。

find命令提供了针对文件的最近访问时间、文件状态最近被修改时间、文件数据最近被修改时间进行匹配的测试项，分别是-amin, -cmin, -mmin和-atime, -ctime, -mtime两组，第一组基于分钟，第二组基于天。
以-amin为例，假设当前时间tnow="2007-11-12 14:42:10"、t1="2007-11-12 14:39:10"、t2="2007-11-12 14:40:10"，那么要查找最近访问时间属于[t1,t2]时间段的文件，可以这么写：

$ find . -amin 3

若测试项参数是数字，则基本上都可以在数字参数前加"+"或者"-"号，表示“大于”或“小于”的意思，因此，要查找最近访问时间属于[t1,tnow]时间段的文件，可以这么写：

$ find . -amin -3

"-amin n"和"-atime n"的处理方法都是：根据当前时间和文件的相应时间属性求n值，然后比较n值和参数n，看是否符合要求。但是这个求n值的过程却有很大不同，他们的不同也代表了两组（基于分钟和基于天）的不同：
"-amin n"

1、求Δt，用当前时间减去文件对应属性的时间值即得到Δt，Δt = tnow - tfile;
2、求浮点数f，用Δt除以1分钟，f = Δt / 1min;
3、将f的小数部分入到整数部分，得到n。即，不管f是6.0102还是6.8901，n都等于7
"-atime n"

1、求Δt，用当前时间减去文件对应属性的时间值即得到Δt，Δt = tnow - tfile;
2、求浮点数f，用Δt处以24小时，f = Δt / 24hours;
3、将f的小数部分都舍掉，得到n。即，不管f是6.0102还是6.8901，n都等于6
大家可以多做实验，试一下。



（八）权限位
很多人都在用windows，从windows系统拷过来的文件经常被加上了可执行权限，比如我现在想把主目录下所有的后缀名为.txt .pdf .rm并且具有可执行权限位的文件查找出来，该怎么写呢？
这里就不得不说一说权限位测试项：-perm。-perm支持符号权限位表示法也支持绝对（八进制）权限位表示法，但是最好使用八进制的权限表示法（这只是个建议  ^_^ ）。
-perm基本上有下面这几中形式：

-perm mode       File's  permission  bits  are  exactly mode.
-perm -mode     All  of the permission bits mode are set for the file.
-perm /mode     Any of the permission bits mode are set for the file.
-perm +mode (此形式已经不推荐使用，功能与/mode相同)
好好理解上面蓝色部分，理解了，-perm测试项也就掌握了。
考考你：
看看下面这句话是什么意思？

find . -perm -444 -perm /222 ! -perm /111

现在再来解决本节最开始提出的问题：查找主目录下所有的后缀名为.txt .pdf .rm并且具有可执行权限位的文件。

$ find ~ /( -name "*.txt" -o -name "*.pdf" -o -name "*.rm" /)  /
   -not /( -type d -o -type l /)  /
   -perm /111 -print



（九）文件类型
有一个问题：我只想查找符号连接文件，可是查找结果中却包括了普通文件、目录文件等等，不相关的东西太多了，怎么把不是符号连接文件的查找结果去掉？
-type测试项刚好可以满足你的要求，-type c即可，其中c表示文件类型，find中支持如下类型：

              b      block (buffered) special
              c      character (unbuffered) special
              d      directory
              p      named pipe (FIFO)
              f       regular file
              l       symbolic link;
              s      socket
              D     door (Solaris)
针对上面的问题，可以这么写：

$ find . -name "e100*" -type l -print
./e1000
./e100puk.txt
但是，不要这么写：

$ find -L . -name "e100*" -type l -print
加上'-L'选项之后，你将查不到需要的东西，除非符号连接已经失效了。



（十）文件大小
前面一再使用-size测试项，这里简单介绍一下。
-size测试项根据文件的大小查找文件，文件大小既可以用块（block）来计量，也可以用字节来计量。默认情况下以块计量文件大小，若想使用字节来计量只需要在数字参数后加c即可。find支持的其他计量方式有：
-size n[cwbkMG]，分别表示

              ‘b’    for 512-byte blocks (this is the default if no suffix  is used)
              ‘c’    for bytes
              ‘w’    for two-byte words
              ‘k’    for Kilobytes (units of 1024 bytes)
              ‘M’    for Megabytes (units of 1048576 bytes)
              ‘G’    for Gigabytes (units of 1073741824 bytes)




（十一）用户、用户组
根据用户、用户组来查找文件，这个没有太多要说的，记住命令格式即可：

-uid n
-user username or uid
-nouser
-gid n
-group gname or gid
-nogroup



（十二）输出格式
如果你不想查找到你想要的文件事单调的输出文件名，你可以使用-printf动作项输出你想要的格式，下面举几个-printf动作的参数：

%p    输出文件名，包括路径名
%f     输出文件名，不包括路径名
%m    以8进制方式输出文件的权限
%g    输出文件所属的组
%h    输出文件所在的目录名
%u    输出文件的属主名
...
例如：

$ find . -user xixi -printf "%m %p //n"
644 ./phone1/hello.c 
644 ./0dfe.c 


其余的，看man手册页吧。



（十三）执行外部命令
这又是一个很容易出彩的地方。find真是强大，对查找到的文件竟然可以调用外部命令进行处理。-exec动作项就是来完成这个功能的，格式是：

find . EXPR1 -exec command {} /;
注意：后一个花括号'}'和'/'之间有一个空格。
例如，查找当前目录下的所有普通文件，并用ls命令输出：

find . -type f -exec ls -l {} /;
有些操作系统中出于安全考虑只允许-exec选项执行诸如l s或ls -l这样的命令。
也可以使用-exec动作项的安全模式：-ok动作项。它的功能和语法都跟-exec一样，只不过它以更安全的模式运行，当要删除文件时，它会给出提示，让你选择到底删除还是不删。
例如：

$ find logs -name "*abc*" -ok rm {} /;

使用-exec动作项处理匹配到的文件时，find命令会将所有匹配到的文件一起传递给exec执行。但有些系统对能够传递给exec的命令长度有限制，这样在find命令运行几分钟之后，就会出现溢出错误。错误信息通常是“参数列太长”或“参数列溢出”。这就是xargs命令的用处所在，特别是与find命令一起使用。
xargs的使用格式是：

find PATH EXPR1 EXPR2 | xargs command
利用管道，把find命令匹配到的文件名传递给xargs命令，而xargs命令每次只获取一部分文件而不是全部。这样它可以先处理最先获取的一部分文件，然后是下一批，并如此继续下去。
在有些系统中，使用-exec动作项会为处理每一个匹配到的文件而发起一个相应的进程，并非将匹配到的文件全部作为参数一次执行；这样在有些情况下就会出现进程过多，系统性能下降的问题，因而效率不高；而使用xargs命令则只有一个进程。另外，在使用xargs命令时，究竟是一次获取所有的参数，还是分批取得参数，以及每一次获取参数的数目都会根据该命令的选项及系统内核中相应的可调参数来确定。
例如，要在普通文件中查找文件内容中包含"io"的文件，可以这么写：

$ find . -type f | xargs grep "io"
Binary file ./a_book_of_c.chm matches
./2234.c:#include <stdio.h>
./3e10.c:#include <stdio.h>

find命令配合exec和xargs可以对所匹配到的文件执行几乎所有的命令。


（十四）总结
理解并运用find，关键是掌握find命令的处理规则（见第五节）：递归遍历所指定的目录树，针对每个文件依次执行find命令中的表达式，表达式首先根据逻辑运算符进行结合，然后依次从左至右对表达式求值。把这个理解了，需要什么功能查一下man就可以了。

find命令还有好多功能这里没有涉及到，具体的大家看man手册页吧。在任何时候，man都是一个极好的帮助工具。   ^_^


（十五）附
第五节提出的问题，答案如下:

$ find . -size +0c -wholename "*e*[0-9]*" -o   /
         !  /( -name "." -o -name "*phone" /) -prune  -name "*.c" -user xixi   /
         -o -name "*phone"
./e1000
./e100.dat
./phone
./phone/e680gphone
./e100puk.txt
./3e10.c
./phone1
./phone1/hello.c
./phone2
./0dfe.c
./e680phone



这篇文章断断续续写了好久，今天终于基本完工。参考了man手册页以及一些网上的资料。
要把自己心中所想有条理的写出来感觉真是不易，希望对大家有所帮助。
欢迎批评指正。