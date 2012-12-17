---
layout: post
title: "Awk One liner Explain"
---

### 概述：
##Awk程序由一系列的patter-action语句（这种形式可以成为rules）构成（pattern {action statements}），
##在rules中pattern或者action可以被省略一个。
##如果pattern被省略了，那么action会对文件中的每行进行同等操作，如果action语句被省略了，那么如同
##action语句是{print}。

##Example 1: double-space a file
{% highlight bash %}
[root@web-db bash]# awk '1;{print ""}' input
[root@web-db bash]# awk '{print}{print ""}' input
[root@web-db bash]# awk 'BEGIN{ORS="\n\n"};1' input
{% endhighlight %}


解说：
{% highlight bash %}
[root@web-db bash]# awk '1;{print ""}' input 
{% endhighlight %}

中pattern1（1代笔永远为true）省略了action语句，所以被默认使用了{print}语句，所以与下面的
[root@web-db bash]# awk '{print}{print ""}' input相同。awk的每个print
语句后面都静默的跟了一个ORS（output record separator）变量，该变量的默认值是newline(\n)。
{print}语句没有参数的情况下，相当于print $0，即输出改行的内容。
对于第三种方法：通过使用BEGIN块，设置了ORS，在awk程序中有两个特殊的代码块BEING和END，BEGIN在awk程序处理正式内容之前执行，
END块在awk程序最后执行。
上述的方法虽然可以给每行后面添加一个空行，但是却不能忽略（如果一行的内容本来就是空的）这种情况，如果对于空行不需要处理，
可以使用下面的方法：
{% highlight bash %}
[root@web-db bash]# cat input
1
12

123
1234
12345
123456
1234567
12345678
123456789
1234567890
12345678901
123456789012
[root@web-db bash]# awk 'NF{print $0 "\n"}' input
1

12

123

1234

12345

123456

1234567

12345678

123456789

1234567890

12345678901

123456789012
{% endhighlight %}

解说：
通过使用NF（number of field of the record）变量，判断每行是否为空，如果为空行，则
NF=0，否则NF>=1,这样就可以省略了空行的操作，相当于过滤了空行。

### Example 2：Triple-space a file（每行后添加两个空行）
{% highlight bash %}
[root@web-db bash]# awk '1;{print "\n"}' input
[root@web-db bash]# awk '{print;print "\n"}' input
[root@web-db bash]# awk 'BEGIN{ORS="\n\n\n"};1' input
{% endhighlight %}

上面的三个命令通过Example1改编而来。

### Example 3：每行内容前附加行号：
{% highlight bash %}
[root@web-db bash]# cat input.txt
834608 0.50500 openssl-sp ampra2       r     04/18/2011 09:26:18 all.q@east-32.enterprisegrid.e     1
834609 0.55500 openssl-sp ampra2       qw    04/18/2011 14:43:03                                    1
834610 0.55500 openssl-sp ampra2       qw    04/18/2011 14:43:05                                    1
[root@web-db bash]# awk '{print NR "\t" $0}' input input.txt
1       1
2       12
3
4       123
5       1234
6       12345
7       123456
8       1234567
9       12345678
10      123456789
11      1234567890
12      12345678901
13      123456789012
14      834608 0.50500 openssl-sp ampra2       r     04/18/2011 09:26:18 all.q@east-32.enterprisegrid.e     1
15      834609 0.55500 openssl-sp ampra2       qw    04/18/2011 14:43:03                                    1
16      834610 0.55500 openssl-sp ampra2       qw    04/18/2011 14:43:05                                    1
[root@web-db bash]# awk '{print FNR "\t" $0}' input input.txt
1       1
2       12
3
4       123
5       1234
6       12345
7       123456
8       1234567
9       12345678
10      123456789
11      1234567890
12      12345678901
13      123456789012
1       834608 0.50500 openssl-sp ampra2       r     04/18/2011 09:26:18 all.q@east-32.enterprisegrid.e     1
2       834609 0.55500 openssl-sp ampra2       qw    04/18/2011 14:43:03                                    1
3       834610 0.55500 openssl-sp ampra2       qw    04/18/2011 14:43:05                                    1
{% endhighlight %}


解说：
FNR（File Line Number）：保存每个文件的当前行号。切换文件时，FNR会被重置，从1开始。
NR(Line Number)：切换文件时NR不会被重置，数字会继续增加，犹如同一个文件一样。

### Example 4：使用printf格式化输出
{% highlight bash %}
[root@web-db bash]# awk '{printf("%5d : %s\n",NR,$0)}' input.txt
    1 : 834608 0.50500 openssl-sp ampra2       r     04/18/2011 09:26:18 all.q@east-32.enterprisegrid.e     1
    2 : 834609 0.55500 openssl-sp ampra2       qw    04/18/2011 14:43:03                                    1
    3 : 834610 0.55500 openssl-sp ampra2       qw    04/18/2011 14:43:05                                    1
{% endhighlight %}


解说：
printf函数的输出不会自动添加"\n"，不同于print函数，所以需要自己在printf中自定义输出格式。

### Example 5：空行前不加行号
{% highlight bash %} 
[root@web-db bash]# awk 'NF{$0=++a " : " $0};{print}' input
1 : 1
2 : 12

3 : 123
4 : 1234
5 : 12345
6 : 123456
7 : 1234567
8 : 12345678
9 : 123456789
10 : 1234567890
11 : 12345678901
12 : 123456789012
[root@web-db bash]# awk 'NF{$0=NR " : " $0};{print}' input
1 : 1
2 : 12

4 : 123
5 : 1234
6 : 12345
7 : 123456
8 : 1234567
9 : 12345678
10 : 123456789
11 : 1234567890
12 : 12345678901
13 : 123456789012
{% endhighlight %}


解说：
NF：行中有内容非空则为true否则为false
{$0=NR " : " $0}：重新定义了行内的内容$0
{print}等同于{print $0}：输出行内内容。

### Example 6：计算文件行数
{% highlight bash %}
[root@web-db bash]# awk 'END{print NR}' input
13
{% endhighlight %}


### Example 7: 统计每行的字段数
{% highlight bash %}
[root@web-db bash]# awk '{print NR":"NF}' input.txt
1:9
2:8
3:8
{% endhighlight %}


### Example 8：统计文件的总字段数
{% highlight bash %}
[root@web-db bash]# awk '{total+=NF}END{print total}' input.txt
25
{% endhighlight %}


### Example 9：打印各字段值的绝对值：
{% highlight bash %}
[root@web-db bash]# cat 1
1 -23
0 -74 89 345 -123456
-1
-123 -1234 -456
-0.4 -98
-453 2 -3
[root@web-db bash]# awk '{for(i=1;i<=NF;i++)if($i<0)$i=-$i;print}' 1
1 23
0 74 89 345 123456
1
123 1234 456
0.4 98
453 2 3
{% endhighlight %}


### Example 10：打印包含单词“root”的行数
{% highlight bash %}
[root@web-db bash]# grep root /etc/passwd
root:x:0:0:root:/root:/bin/bash
operator:x:11:0:operator:/root:/sbin/nologin
[root@web-db bash]# awk '/root/{a++};END{print a+0}' /etc/passwd
2
{% endhighlight %}


### Example 11：打印第一个字段数值最大的行
{% highlight bash %}
[root@web-db bash]# cat 2
-1
-2
-3
-4
-5
-12

-123
-1234
[root@web-db bash]# awk 'NR==1{max=$1;maxline=$0;next;}$1>max{max=$1;maxline=$0};END{print max,maxline}' 2
-1 -1
{% endhighlight %}


### Example 12：打印每行的最后一个字段
{% highlight bash %}
[root@web-db bash]# awk '{print $NF}' input.txt
1
1
1
{% endhighlight %}


### Example 13：打印最后一行的最后一个字段
{% highlight bash %}
[root@web-db bash]# cat 2
-1
-2
-3
-4
-5
-12

-123
-1234 0
[root@web-db bash]# awk '{field = $NF};END{print field}' 2
0
[root@web-db bash]# awk 'END{print $NF}' 2
0
{% endhighlight %}


### Example 14：打印字段数大于2的行
{% highlight bash %}
[root@web-db bash]# cat 1
1 -23
0 -74 89 345 -123456
-1
-123 -1234 -456
-0.4 -98
-453 2 -3
23
325452
2342
345452
435634523
1234
00998
889004
124q
[root@web-db bash]# awk 'NF > 2' 1
0 -74 89 345 -123456
-123 -1234 -456
-453 2 -3
[root@web-db bash]#
{% endhighlight %}


### Example 15：转换windows/DOS newline(CRLF)到unix newline(LF)。
{% highlight bash %}
[root@web-db bash]# cat -A win.txt
hello^M$
world^M$
yes^M$
[root@web-db bash]# awk '{sub(/\r/,"");print}' win.txt | cat -A
hello$
world$
yes$
[root@web-db bash]#
{% endhighlight %}

awk的sub函数：
sub(regular expression,replace strings,Strings)
使用replace strings替换Strings中符合regular expression的部分。
如果省略了Strings那么，默认替换的位置是$0。 


### Example 16：翻转Example 15
{% highlight bash %}
[root@web-db bash]# cat -A aaa
hello$
world$
[root@web-db bash]# awk '{sub(/$/,"\r");print}' aaa | cat -A
hello^M$
world^M$
[root@web-db bash]#
{% endhighlight %}


### Example 17：删除每行开头的空格和tabs
{% highlight bash %}
[root@web-db bash]# cat -A whitespace
^Ihello$
 123$
 ^I345 345$
  ^Iworld$
[root@web-db bash]# awk '{sub(/^[ \t]+/,"");print}' whitespace | cat -A
hello$
123$
345 345$
world$
{% endhighlight %}


### Example 18：删除尾随的空格和tabs
{% highlight bash %}
[root@web-db bash]# cat -A whitespace
^Ihello^I ^I $
 123   $
 ^I345 345        ^I^I$
  ^Iworld^I^I  ^I$
[root@web-db bash]# awk '{sub(/[ \t]+$/,"");print}' whitespace | cat -A
^Ihello$
 123$
 ^I345 345$
  ^Iworld$
{% endhighlight %}

### Example 19：删除开头和尾随的空格和tabs
{% highlight bash %}
[root@web-db bash]# cat -A whitespace
^Ihello^I ^I $
 123   $
 ^I345 345        ^I^I$
  ^Iworld^I^I  ^I$
[root@web-db bash]# awk '{gsub(/^[ \t]+|[ \t]+$/,"");print}' whitespace | cat -A
hello$
123$
345 345$
world$
[root@web-db bash]#
{% endhighlight %}

awk中的gsub函数，gsub与sub工作原理一样，唯一的区别就是gsub=global sub全局操作，而sub
值操作一次。

### Example 20：移除字段间的whitespace
{% highlight bash %}
[root@web-db bash]# cat -A whitespace
^Ihello^I ^I $
 123   $
 ^I345 345        ^I^I$
  ^Iworld^I^I  ^I$
1 2^I3$
abc ^I345  hello$
hello world ^I^Ihello^Iworld$
[root@web-db bash]# awk '{$1=$1;print}' whitespace | cat -A
hello$
123$
345 345$
world$
1 2 3$
abc 345 hello$
hello world hello world$
[root@web-db bash]#
{% endhighlight %}

如果更改了一个awk中的field，那么awk会重新构筑$0变量，新的$0的各字段间会使用OFS（single space by default）
来分割。原先字段间的所有whitespace会被忽略。

### Example 21：在每行的开头添加两个空格
{% highlight bash %}
[root@web-db bash]# cat 2
-1
-2
-3
-4
-5
-12

-123
-1234 0
[root@web-db bash]# awk '{sub(/^/,"  ");print}' 2
  -1
  -2
  -3
  -4
  -5
  -12

  -123
  -1234 0
{% endhighlight %}

### Example 22：格式化文本的宽度（右对齐）
{% highlight bash %}
[root@web-db bash]# awk '{printf "%10s\n",$0}' 2
        -1
        -2
        -3
        -4
        -5
       -12

      -123
   -1234 0
{% endhighlight %}


设置宽度为10个字符，不足10个字符的从左边开始使用space来填充，直到满足10个字符为止。

### Example 23：格式化输出（左对齐）
{% highlight bash %}
[root@web-db bash]# awk '{printf "%-10s\n",$0}' 2
-1
-2
-3
-4
-5
-12

-123
-1234 0
{% endhighlight %}


### Example 24：格式化输出（中央对齐）
{% highlight bash %}
[root@web-db bash]# awk '{long=length();spaces=int((79-1)/2);printf "%"(spaces+long)"s\n",$0}' 2
                                       -1
                                       -2
                                       -3
                                       -4
                                       -5
                                       -12

                                       -123
                                       -1234 0
{% endhighlight %}

解说：
length函数：length(var)：计算var的长度，如果省略了var那么会计算$0的长度。
计算完$0的长度然后将值赋值给变量long，然后通过固定宽度79-long在平分得出左右
两边需要的空格数，最后通过左对齐输出long+spaces的字符串数的宽度输出$0，即输出改行的内容。

### Example 25：符合判断条件下进行替换
{% highlight bash %}
[root@web-db bash]# cat input.txt
834608 0.50500 openssl-sp ampra2       r     04/18/2011 09:26:18 all.q@east-32.enterprisegrid.e     1
834609 0.55500 openssl-sp ampra2       qw    04/18/2011 14:43:03                                    1
834610 0.55500 openssl-sp ampra2       qw    04/18/2011 14:43:05                                    1
[root@web-db bash]# awk '$5 == "r"{sub(/openssl-sp/,"hello");print}' input.txt
834608 0.50500 hello ampra2       r     04/18/2011 09:26:18 all.q@east-32.enterprisegrid.e     1
[root@web-db bash]# awk '$5 != "r"{sub(/openssl-sp/,"hello");print}' input.txt
834609 0.55500 hello ampra2       qw    04/18/2011 14:43:03                                    1
834610 0.55500 hello ampra2       qw    04/18/2011 14:43:05 
                                   1
[root@web-db bash]# cat mysql
+--------------+-------------+---------+------+------------+
| name | owner | species | sex | birth |
+--------------+-------------+---------+------+------------+
| Mickey Mouse | Walt Disney | mouse | m | NULL |
| Tom | Walt Disney | cat | m | 1960-03-03 |
| Jery | Walt Disney | mouse | m | NULL |
| Spoky | Trey Parker | fish | NULL | 2001-06-13 |
| Fluffy | Harold | Hamster | f | 1993-02-04 |
[root@web-db bash]# awk '{gsub(/Tom|cat|fish/,"red");print}' mysql
+--------------+-------------+---------+------+------------+
| name | owner | species | sex | birth |
+--------------+-------------+---------+------+------------+
| Mickey Mouse | Walt Disney | mouse | m | NULL |
| red | Walt Disney | red | m | 1960-03-03 |
| Jery | Walt Disney | mouse | m | NULL |
| Spoky | Trey Parker | red | NULL | 2001-06-13 |
| Fluffy | Harold | Hamster | f | 1993-02-04 |
{% endhighlight %}

### Example 26：awk翻转文件输出（类似tac命令）
{% highlight bash %}
[root@web-db bash]# awk '{a[i++] = $0}END{for(j=i-1;j>=0;)print a[j--]}' input.txt
834610 0.55500 openssl-sp ampra2       qw    04/18/2011 14:43:05                                    1
834609 0.55500 openssl-sp ampra2       qw    04/18/2011 14:43:03                                    1
834608 0.50500 openssl-sp ampra2       r     04/18/2011 09:26:18 all.q@east-32.enterprisegrid.e     1
[root@web-db bash]# awk '{a[i++] = $0}END{for(j=i-1;j>=0;j--)print a[j]}' input.txt
834610 0.55500 openssl-sp ampra2       qw    04/18/2011 14:43:05                                    1
834609 0.55500 openssl-sp ampra2       qw    04/18/2011 14:43:03                                    1
834608 0.50500 openssl-sp ampra2       r     04/18/2011 09:26:18 all.q@east-32.enterprisegrid.e     1
[root@web-db bash]# tac input.txt
834610 0.55500 openssl-sp ampra2       qw    04/18/2011 14:43:05                                    1
834609 0.55500 openssl-sp ampra2       qw    04/18/2011 14:43:03                                    1
834608 0.50500 openssl-sp ampra2       r     04/18/2011 09:26:18 all.q@east-32.enterprisegrid.e     1
{% endhighlight %}

### Example 27：交换field 1 和field 2的位置
{% highlight bash %}
[root@web-db bash]# awk '{temp = $1;$1 = $2; $2 = temp;print}' input.txt
0.50500 834608 openssl-sp ampra2 r 04/18/2011 09:26:18 all.q@east-32.enterprisegrid.e 1
0.55500 834609 openssl-sp ampra2 qw 04/18/2011 14:43:03 1
0.55500 834610 openssl-sp ampra2 qw 04/18/2011 14:43:05 1
{% endhighlight %}

### Example 28：删除指定的字段
{% highlight bash %}
[root@web-db bash]# awk '{$2 = "";print}' input.txt
834608  openssl-sp ampra2 r 04/18/2011 09:26:18 all.q@east-32.enterprisegrid.e 1
834609  openssl-sp ampra2 qw 04/18/2011 14:43:03 1
834610  openssl-sp ampra2 qw 04/18/2011 14:43:05 1
{% endhighlight %}

### Example 29：行内各字段翻转
{% highlight bash %}
[root@web-db bash]# awk '{for(i=NF;i>0;i--)printf("%s ",$i);printf("\n")}' whitespace
hello
123
345 345
world
3 2 1
hello 345 abc
world hello world hello
{% endhighlight %}

### Example 30：去除连贯的重复行
{% highlight bash %}
[root@web-db bash]# cat copy
hello world
world hello
hello
hello
hello
red yellow pure
hello
[root@web-db bash]# awk ' a != $0;{a = $0}' copy
hello world
world hello
hello
red yellow pure
hello
{% endhighlight %}

### Example 31：移除非连贯的重复行
{% highlight bash %}
[root@web-db bash]# awk '!a[$0]++' copy
hello world
world hello
hello
red yellow pure
{% endhighlight %}

This one-liner is very idiomatic. It registers the lines seen in the associative-array "a"
 (arrays are always associative in Awk) and at the same time tests if it had seen the line before. 
 If it had seen the line before, then a[line] > 0 and !a[line] == 0. 
 Any expression that evaluates to false is a no-op, 
 and any expression that evals to true is equal to "{ print }".

 When Awk sees the first "foo", it evaluates the expression "!a["foo"]++". 
 "a["foo"]" is false, but "!a["foo"]" is true - Awk prints out "foo". 
 Then it increments "a["foo"]" by one with "++" post-increment operator. 
 Array "a" now contains one value "a["foo"] == 1".

Next Awk sees "bar", it does exactly the same what it did to "foo" and prints out "bar". 
Array "a" now contains two values "a["foo"] == 1" and "a["bar"] == 1".

Now Awk sees the second "foo". This time "a["foo"]" is true, 
"!a["foo"]" is false and Awk does not print anything! Array "a" 
still contains two values "a["foo"] == 2" and "a["bar"] == 1".

Finally Awk sees "baz" and prints it out because "!a["baz"]" is true. 
Array "a" now contains three values "a["foo"] == 2" and "a["bar"] == 1" and "a["baz"] == 1".

{% highlight bash %}
[root@web-db bash]# awk '!($0 in a){a[$0];print}' copy
hello world
world hello
hello
red yellow pure
{% endhighlight %}

### Example 32：使用逗号连接相邻的2行
{% highlight bash %}
[root@web-db bash]# awk 'ORS=NR%2?",":"\n"' copy
hello world,world hello
hello,hello
hello,red yellow pure
hello,[root@web-db bash]#
{% endhighlight %}

### Example 33：输出文件的前10行
{% highlight bash %}
[root@web-db bash]# seq 11 | awk 'NR < 11'
1
2
3
4
5
6
7
8
9
10
{% endhighlight %}

如果文件的行数远远大于了10行，那么使用下面的命令效率会更高，因为对于上述的命令，
当行数大于等于11时，awk默认什么都不操作，但是会继续浏览完文件。

{% highlight bash %}
[root@web-db bash]# seq 11 | awk '1;NR == 10 {exit}'
1
2
3
4
5
6
7
8
9
10
{% endhighlight %}

上述命令当文件走到第十行的时候输出内容然后退出。
如果只打印文件的第一行可以使用如下命令：

{% highlight bash %}
[root@web-db bash]# seq 5 | awk 'NR > 1 {exit};1'
1
{% endhighlight %}

如果只打印文件的后两行，使用如下命令：

{% highlight bash %}
[root@web-db bash]# seq 5 | awk '{y = x "\n" $0;x = $0};END{print y}'
4
5
{% endhighlight %}

模仿毕竟是模仿，如果文件内容是几十MB，设置更大，没有必要玩模仿，直接tail -n 2即可。
如果要打印文件的最后一行：

{% highlight bash %}
[root@web-db bash]# seq 7 | awk 'END{print}'
7
{% endhighlight %}

### Example 34：打印匹配模式的上一行
{% highlight bash %}
[root@web-db bash]# awk '/root/ && NR != 1{print x};{x=$0}' /etc/passwd
uucp:x:10:14:uucp:/var/spool/uucp:/sbin/nologin
[root@web-db bash]# awk '/root/{print (x==""?"match on line1":x)};{x=$0}' /etc/passwd
match on line1
uucp:x:10:14:uucp:/var/spool/uucp:/sbin/nologin
[root@web-db bash]# awk '/root/{print (x==""?$0:x)};{x=$0}' /etc/passwd
root:x:0:0:root:/root:/bin/bash
uucp:x:10:14:uucp:/var/spool/uucp:/sbin/nologin
{% endhighlight %}

### Example 35：打印匹配模式行的下一行
{% highlight bash %}
[root@web-db bash]# awk '/root/{getline;print}' /etc/passwd
bin:x:1:1:bin:/bin:/sbin/nologin
games:x:12:100:games:/usr/games:/sbin/nologin
[root@web-db bash]# grep -C 1 root /etc/passwd
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
--
uucp:x:10:14:uucp:/var/spool/uucp:/sbin/nologin
operator:x:11:0:operator:/root:/sbin/nologin
games:x:12:100:games:/usr/games:/sbin/nologin
{% endhighlight %}

### Example 36：打印行的内容长度大于某个数字的行
{% highlight bash %}
[root@web-db bash]# awk 'length > 60' /etc/passwd
sshd:x:74:74:Privilege-separated SSH:/var/empty/sshd:/sbin/nologin
nfsnobody:x:4294967294:4294967294:Anonymous NFS User:/var/lib/nfs:/sbin/nologin
avahi-autoipd:x:100:101:avahi-autoipd:/var/lib/avahi-autoipd:/sbin/nologin
{% endhighlight %}

### Example 37：打印从匹配模式行到文件结尾
{% highlight bash %}
[root@web-db bash]# seq 7 | awk '/3/,0'
3
4
5
6
7
{% endhighlight %}

0：代表false，所以永远不会匹配成功，awk会一直读取文件并print。

### Example 38：打印行内的行
{% highlight bash %}
[root@web-db bash]# seq 7 | awk 'NR==3,NR==6'
3
4
5
6
{% endhighlight %}

### Example 39：打印特定的行
{% highlight bash %}
[root@web-db bash]# seq 17 | awk 'NR==7{print;exit}'
7
{% endhighlight %}

### Example 40：打印匹配模式之间的行
{% highlight bash %}
[root@web-db bash]# cat 3
hello
red
yellow
apple
pear
beer
bear
monkey
donkey
[root@web-db bash]# awk '/apple/,/monkey/' 3
apple
pear
beer
bear
monkey
{% endhighlight %}

### Example 41：删除文件中所有的空行
{% highlight bash %}
[root@web-db bash]# cat 2
-1


-4

-5
-12

-123
-1234 0
[root@web-db bash]# awk NF 2
-1
-4
-5
-12
-123
-1234 0
[root@web-db bash]# awk '/./' 2
-1
-4
-5
-12
-123
-1234 0
[root@web-db bash]#
{% endhighlight %}

### Example 42：（处理多个文件）

awk tip1 ：
awk 'NR==FNR { # some actions; next} # other condition {# other actions}'

This is used when processing two files. When processing more than one file, 
awk reads each file sequentially, one after another, in the order they are specified on the command line. 
The special variable NR stores the total number of input records read so far, 
regardless of how many files have been read. The value of NR starts at 1 and always 
increases until the program terminates. Another variable, FNR, 
stores the number of records read from the current file being processed. 
The value of FNR starts from 1, increases until the end of the current file, 
starts again from 1 as soon as the first line of the next file is read, and so on. 
So, the condition "NR==FNR" is only true while awk is reading the first file. Thus, 
in the program above, the actions indicated by "# some actions" are executed when awk is reading the first file; 
the actions indicated by "# other actions" are executed when awk is reading the second file, 
if the condition in "# other condition" is met. The "next" at the end of the first action block 
is needed to prevent the condition in "# other condition" from being evaluated, 
and the actions in "# other actions" from being executed while awk is reading the first file.

### 示例：prints lines that are both in file1 and file2
{% highlight bash %}
[root@web-db bash]# cat file1
1
2
3
4
hello
world
7
[root@web-db bash]# cat file2
hello
world
7
8
[root@web-db bash]# awk 'NR==FNR{a[$0];next} $0 in a' file1 file2
hello
world
7
{% endhighlight %}

### awk tip2：将两个文件间接关联：
{% highlight bash %}
[root@web-db bash]# cat 1.1
20081010 1123 xxx
20081011 1234 def
20081012 0933 xyz
20081013 0512 abc
20081013 0717 def
[root@web-db bash]# cat 1.2
abc withdrawal
def payment
xyz deposit
xxx balance
{% endhighlight %}

将1.1文件中的最后一列换成1.2中的第二列内容。

{% highlight bash %}
[root@web-db bash]# awk 'NR==FNR{a[$1]=$2;next}{$3=a[$3]}1' 1.2 1.1
20081010 1123 balance
20081011 1234 payment
20081012 0933 deposit
20081013 0512 withdrawal
20081013 0717 payment
{% endhighlight %}

最后那个1必须的，否则没有任何输出，因为{$3=a[$3]}只是一个省略了pattern的action，
默认对文件1.1的每行都进行操作，然后后面的1作为pattern省略了action默认是{print $0}。
所以如果1存在就会输出1.1的内容。

### awk tip3：计算文件中最大值与其他值之间的差距：
{% highlight bash %}
[root@web-db bash]# cat input
1
12

123
1234
12345
123456
1234567
12345678
123456789
1234567890
12345678901
123456789012
[root@web-db bash]# awk 'NR==FNR{if($0>max)max=$0;next}{$0=max-$0}1' input input
123456789011
123456789000
123456789012
123456788889
123456787778
123456776667
123456665556
123455554445
123444443334
123333332223
122222221122
111111110111
0
{% endhighlight %}

Caveat: all the programs that use the two-files idiom will not work correctly 
if the first file is empty (in that case, awk will execute the actions associated to 
NR==FNR while reading the second file). To correct that, 
you can reinforce the NR==FNR condition by adding a test that checks that also FILENAME equals ARGV[1].

### awk tip4：

# prints lines from /beginpat/ to /endpat/, not inclusive

awk '/beginpat/,/endpat/{if (!/beginpat/&&!/endpat/)print}'

# prints lines from /beginpat/ to /endpat/, not including /beginpat/

awk '/beginpat/,/endpat/{if (!/beginpat/)print}'

Example 1: Parenthesize first character of each word

### awk tip5：awk 中的flag：
prints lines from /beginpat/ to /endpat/, not inclusive

awk '/endpat/{p=0};p;/beginpat/{p=1}'

prints lines from /beginpat/ to /endpat/, excluding /endpat/

awk '/endpat/{p=0} /beginpat/{p=1} p'

prints lines from /beginpat/ to /endpat/, excluding /beginpat/

awk 'p; /endpat/{p=0} /beginpat/{p=1}'

示例：

{% highlight bash %}
[root@web-db bash]# seq 7 | awk '/3/{p=1};p;/6/{p=0}'
3
4
5
6
[root@web-db bash]# seq 7 | awk '/6/{p=0};p;/3/{p=1}'
4
5
[root@web-db bash]# seq 7 | awk '/6/{p=1};p;/3/{p=0}'
6
7
[root@web-db bash]# seq 7 | awk '/3/{p=0};p;/6/{p=1}'
7
[root@web-db bash]# seq 7 | awk '/6/{p=0}/3/{p=1}p'
3
4
5
[root@web-db bash]# seq 7 | awk 'p;/6/{p=0}/3/{p=1}'
4
5
6
{% endhighlight %}

### awk tip6：分割文件：
{% highlight bash %}
[root@web-db bash]# cat tfile
line1
line2
line3
line4
FOO1
line5
line6
FOO2
line7
line8
FOO3
line9
line10
line11
FOO4
line12
FOO5
line13
[root@web-db bash]# awk -v n=1 '/^FOO[0-9]*/{close("out"n);n++;next}{print > "out"n}' tfile
[root@web-db bash]# ls
1    2           copy   input      mysql.awk  out3  out6         test.awk  whitespace
1.1  3           file1  input.txt  out1       out4  process.awk  test.in
1.2  apache.log  file2  mysql      out2       out5  test         tfile
[root@web-db bash]# cat out1
line1
line2
line3
line4
[root@web-db bash]# cat out2
line5
line6
[root@web-db bash]# cat out3
line7
line8
[root@web-db bash]# cat out4
line9
line10
line11
[root@web-db bash]# cat out6
line13
[root@web-db bash]# cat out5
line12
[root@web-db bash]# LC_ALL=C gawk -v RS='FOO[0-9]*\n' -v ORS= '{print > "out"NR}' tfile
{% endhighlight %}

This sed example prints the first character of every word in paranthesis.
 [root@web-db bash]# echo "Welcome To Our School" | sed  's/\(\b[A-Z]\)/\(\1\)/g'
(W)elcome (T)o (O)ur (S)chool

### 群内问题： split strings in particular format
{% highlight bash %}
[root@web-db bash]# cat input
1
12
123
1234
12345
123456
1234567
12345678
123456789
1234567890
12345678901
123456789012
{% endhighlight %}

### 群内：我为SUN疯狂.乐看的方法
{% highlight bash %}
[root@web-db bash]# sed -r ':a;s/([0-9])([0-9]{3})(,|$)/\1,\2\3/;ta' input
1
12
123
1,234
12,345
123,456
1,234,567
12,345,678
123,456,789
1,234,567,890
12,345,678,901
123,456,789,012
{% endhighlight %}

### 群内：昨夜星辰的方法
{% highlight bash %}
[root@web-db bash]# sed -r ':1;s/(.*[0-9])([0-9]{3})/\1,\2/;t1' input
1
12
123
1,234
12,345
123,456
1,234,567
12,345,678
123,456,789
1,234,567,890
12,345,678,901
123,456,789,012
[root@web-db bash]#
{% endhighlight %}

### 群内：潜艇.c的方法
{% highlight bash %}
[root@web-db bash]# awk '{for(i=1;i<length($1);i++)printf("%s",(length($1)-i)%3==0?substr($1,i,1)",":substr($1,i,1));print substr($1,length($1),1)}' input
1
12
123
1,234
12,345
123,456
1,234,567
12,345,678
123,456,789
1,234,567,890
12,345,678,901
123,456,789,012
{% endhighlight %}

### 群内：灰灰的方法
{% highlight bash %}
[root@web-db bash]# perl -ne 's/(?<=\d)(?=(\d{3})+$)/,/g;print' input
1
12
123
1,234
12,345
123,456
1,234,567
12,345,678
123,456,789
1,234,567,890
12,345,678,901
123,456,789,012
{% endhighlight %}

### 群内：rrFeng的rev思想
{% highlight bash %}
[root@web-db bash]# cat input|rev|sed 's/[^\n]\{3\}/&,/g'|rev|sed 's/^,//g'
1
12

123
1,234
12,345
123,456
1,234,567
12,345,678
123,456,789
1,234,567,890
12,345,678,901
123,456,789,012
{% endhighlight %}

### 群内天朝.帝都june方法
{% highlight bash %}
[root@web-db bash]# cat printf.sh
#!/bin/bash

exec 3<input
while read -u 3 -r line;do
        printf "%'d\n" $line
done
exec 3<&-
[root@web-db bash]# bash printf.sh
1
12
0
123
1,234
12,345
123,456
1,234,567
12,345,678
123,456,789
1,234,567,890
12,345,678,901
123,456,789,012
[root@web-db bash]#
{% endhighlight %}

### 原文链接：

原文链接: <a href="http://www.catonmat.net/blog/awk-one-liners-explained-part-one/">http://www.catonmat.net/blog/awk-one-liners-explained-part-one/</a>




