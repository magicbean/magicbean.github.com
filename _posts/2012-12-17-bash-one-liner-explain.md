---
layout: post
title: "Bash One liner Explain"
---

## 1：创建一个空文件
### 示例：
{% highlight bash %}
[root@web-db bash]# > empty
[root@web-db bash]# ll empty
-rw-r--r-- 1 root root 0 Dec 10 03:37 empty
[root@web-db bash]#
{% endhighlight %}

## 2：向文件中追加内容：
### 示例：
{% highlight bash %}
[root@web-db bash]# echo "hello world" >> empty
[root@web-db bash]# cat empty
hello world
[root@web-db bash]#
{% endhighlight %}

## 3：从文件读取一行并赋值给一个变量
### 示例：
{% highlight bash %}
[root@web-db bash]# read line < empty
[root@web-db bash]# echo $line
hello world
[root@web-db bash]#
{% endhighlight %}

### 默认情况下，read会去掉每行的行首和行尾的whitespaces，如果想要保留那些whitespaces
### 需要将变量IFS设置为空。
### 示例：
{% highlight bash %}
[root@web-db bash]# cat -A empty
  ^Ihello world$
[root@web-db bash]# read -r line < empty
[root@web-db bash]# echo $line
hello world
[root@web-db bash]#
[root@web-db bash]# OIFS=$IFS
[root@web-db bash]# IFS=
[root@web-db bash]# read line < empty
[root@web-db bash]# echo $line
        hello world
[root@web-db bash]#
{% endhighlight %}


## 4：一行行的读取文件
### 示例：
{% highlight bash %}
[root@web-db bash]# cat empty
hello world
yes no
[root@web-db bash]# while read line;do
> echo $line
> done < empty
hello world
yes no
[root@web-db bash]#
{% endhighlight %}

## 5：随机读取文件中的一行并赋值给一个变量
### 示例：
{% highlight bash %}
[root@web-db bash]# read -r random_line < <(shuf number)
[root@web-db bash]# echo $random_line
4
[root@web-db bash]# cat number
1
2
3
4
5
[root@web-db bash]#
[root@web-db bash]# read -r rand_line < <(/usr/local/bin/sort -R number)
[root@web-db bash]# echo $rand_line
2
[root@web-db bash]#
{% endhighlight %}

## 6：读取文件每行的前三个字段并赋值
### 示例：
{% highlight bash %}
[root@web-db bash]# cat number
1 2 3 4
5 6 7 8
9 10 11 12
[root@web-db bash]# while read -r f1 f2 f3 other;do echo $f1 $f2 $f3; done < number
1 2 3
5 6 7
9 10 11
[root@web-db bash]#
{% endhighlight %}

### 如果文件行中的字段数小于3，那么有几个就赋值几个变量，赋值的顺序从变量左到右。
### 如果文件中的字段数大于3，那么只去前三个，其余的都给other；如果没使用other变量的话，
### 那么从第三个字段开始到结束所有字段都给第三个变量。
### 还可以使用下边的写法：
{% highlight bash %}
[root@web-db bash]# while read -r f1 f2 f3 _;do echo $f1 $f2 $f3; done < number
1 2 3
5 6 7
9 10 11
[root@web-db bash]#
{% endhighlight %}

### 还可以使用文件描述符的形式读取文件（推荐）：
{% highlight bash %}
exec 3< input_file.txt  # open input_file.txt on fd 3
while read -u 3 -r line ; do
    # do stuff here
done
exec 3<&-  # close fd 3

[root@web-db bash]# string="20 packets in 10 seconds"
[root@web-db bash]# read packets _ _ time _ <<< "$string"
[root@web-db bash]# echo $packets $time
20 10
[root@web-db bash]#
{% endhighlight %}

### <<<：是一个here-string格式，可以让你直接向命令的standard input传递strings。

## 7：从绝对路径中提取文件名
### 示例：
{% highlight bash %}
[root@web-db bash]# path="/hello/world/test.txt"
[root@web-db bash]# echo ${path##*/}
test.txt
[root@web-db bash]# basename $path
test.txt
[root@web-db bash]#
{% endhighlight %}

## 8：从绝对路径中提取路径名
### 示例：
{% highlight bash %}
[root@web-db bash]# echo ${path%/*}
/hello/world
[root@web-db bash]# dirname $path
/hello/world
[root@web-db bash]#
{% endhighlight %}

## 9：打印一定范围内的字符
### 示例：
{% highlight bash %}
[root@web-db bash]# printf "%c" {a..z}
abcdefghijklmnopqrstuvwxyz[root@web-db bash]#
[root@web-db bash]# printf "%c" {a..z} $'\n'
abcdefghijklmnopqrstuvwxyz
[root@web-db bash]# echo $(printf "%c" {a..z})
abcdefghijklmnopqrstuvwxyz
[root@web-db bash]# echo {a..z}
a b c d e f g h i j k l m n o p q r s t u v w x y z
[root@web-db bash]# 
{% endhighlight %}

### 将输出赋值给一个变量：
{% highlight bash %}
[root@web-db bash]# printf -v chars "%c" {a..z}
[root@web-db bash]# echo $chars
abcdefghijklmnopqrstuvwxyz
{% endhighlight %}

## 10：打印数字
### 示例：
{% highlight bash %}
[root@web-db bash]# printf "%02d\n" {0..9}
00
01
02
03
04
05
06
07
08
09
[root@web-db bash]#
[root@web-db bash]# echo $(printf "%02d\n" {0..9})
00 01 02 03 04 05 06 07 08 09
[root@web-db bash]#
[root@web-db bash]# printf "%02d\n" {0..9} | tr '\n' ' '|sed 's/$/\n/'
00 01 02 03 04 05 06 07 08 09
[root@web-db bash]#
[root@web-db bash]# echo 1{,,,}
1 1 1 1
[root@web-db bash]# clear
{% endhighlight %}

## 11：字符替换
### 示例：
{% highlight bash %}
[root@web-db bash]# string=foobarfoo
[root@web-db bash]# echo ${string/foo/hello}
hellobarfoo
[root@web-db bash]#
[root@web-db bash]# echo ${string//foo/hello}
hellobarhello
[root@web-db bash]#
{% endhighlight %}

## 12：检测字符串匹配：
### 示例：
{% highlight bash %}
[root@web-db bash]# cat match.sh
#!/bin/bash
string="Yellow"
if [[ $string = [Yy]* ]]; then
        echo "yes"
fi
string="123.456"
if [[ $string =~ [0-9]+\.[0-9]+ ]]; then
        echo "yes"
fi
[root@web-db bash]# bash match.sh
yes
yes
[root@web-db bash]#
{% endhighlight %}

## 13：计算字符串的长度及提取特定长度的字符串
### 示例：
{% highlight bash %}
${var:offset:length}
[root@web-db bash]# echo $string
foobarfoo
[root@web-db bash]# echo ${#string}
9
[root@web-db bash]# echo ${string:6}
foo
[root@web-db bash]# echo ${string:3:5}
barfo
[root@web-db bash]# echo ${string:3:3}
bar
[root@web-db bash]#
{% endhighlight %}

## 14：$@与$*的差异
### 示例：
{% highlight bash %}
[root@web-db bash]# cat argu.sh
#!/bin/bash
for str in "$@"
do
echo $str
done
for str in "$*"
do
echo $str
done
[root@web-db bash]# bash argu.sh hello world yes
hello
world
yes
hello world yes
[root@web-db bash]#
{% endhighlight %}

## 15：使用bash访问网站
### 示例：
{% highlight bash %}
exec 3<>/dev/tcp/www.baidu.com/80
echo -e "GET / HTTP/1.0\n\n" >&3
cat <&3
{% endhighlight %}

## 16：禁止重定向时覆盖原有文件内容
### 示例：
{% highlight bash %}
[root@web-db bash]# echo hello > ori
[root@web-db bash]# cat ori
hello
[root@web-db bash]# echo world > ori
[root@web-db bash]# cat ori
world
[root@web-db bash]# set -o noclobber
[root@web-db bash]# echo hello again > ori
-bash: ori: cannot overwrite existing file
[root@web-db bash]# echo hello again >| ori
[root@web-db bash]# cat ori
hello again
[root@web-db bash]#
{% endhighlight %}

## 17：tee命令既能保证标准输出又能将标准输出输出到另一个文件
## tee是通过copy的方式将stdout到另一个文件中，而不是通过重定向
## 的方法。
### 示例：
{% highlight bash %}
[root@web-db bash]# echo hello | tee ori
hello
[root@web-db bash]# cat ori
hello
[root@web-db bash]#
[root@web-db bash]# echo 123 > ori
-bash: ori: cannot overwrite existing file
[root@web-db bash]#
{% endhighlight %}

## 18：重定向的顺序
### 示例：
{% highlight bash %}
[root@web-db bash]# ls /tmp/hello
ls: cannot access /tmp/hello: No such file or directory
[root@web-db bash]# echo hello > /tmp/hello
[root@web-db bash]# cat /tmp/hello
hello
[root@web-db bash]# echo > /tmp/hello hello world
[root@web-db bash]# cat /tmp/hello
hello world
[root@web-db bash]# > /tmp/hello echo hello again
[root@web-db bash]# cat /tmp/hello
hello again
[root@web-db bash]#
{% endhighlight %}

## 19：交换stdout和stderr
### 示例：
command 3>&1 1>&2 2>&3 3>&-

## 20:打印所有管道命令的退出码
### 示例：
{% highlight bash %}
[root@web-db bash]# echo 'pants are cool' | grep 'moo' | sed 's/0/x/' | awk '{print $1}'
[root@web-db bash]# echo ${PIPESTATUS[@]}
0 1 0 0
[root@web-db bash]#
{% endhighlight %}

### 显然grep命令失败了。
### 各管道命令的退出码都保存在数组PIPESTATUS中。

## 21：histroy设置
### 示例：
取消记录历史命令：

unset HISTFILE

HISTFILE变量定义了历史命令的存储地。

或者可以使用如下方法：

HISTFILE=/dev/null

设置历史命令时间格式：

HISTTIMEFORMAT="%Y-%m-%d %H:%M:%S "

忽略重复行：

HISTTIMEFORMAT="%Y-%m-%d %H:%M:%S"

export HISTTIMEFORMAT="%F %T `whoami` "

计算history命令使用最多的前10个：

{% highlight bash %}
[root@web-db bash]# history
    1  2012-12-10 10:09:48 root > ~/.bash_history
    2  2012-12-10 10:09:48 root clear
    3  2012-12-10 10:09:48 root vim ~/.bashrc
    4  2012-12-10 10:09:48 root bash ~/.bashrc
    5  2012-12-10 10:09:48 root clear
    6  2012-12-10 10:09:48 root exit
    7  2012-12-10 10:09:50 root cd bash/
    8  2012-12-10 10:09:51 root clear
    9  2012-12-10 10:10:01 root cat ~/.bash_history
    .......
[root@web-db bash]# history | sed 's/^ *//' | cut -d' ' -f6-|awk '{count[$0]++}END{for(i in count)print count[i],i}'|sort -nr|head
11 clear
2 vim ~/.bashrc
2 sed 's/^ *//' his| cut -d' ' -f6-
2 cat his
2 bash ~/.bashrc
1 vim ~/.bashrc
1 sed 's/^ *//' his| cut -f5-
1 sed 's/^ *//' his| cut -f1
1 sed 's/^ *//' his| cut -d' ' -f6-|awk '{count[$0]++}END{for(i in count)print count[i],i}'|sort -nr|head
1 sed 's/^ *//' his| cut -d' ' -f6-|awk '{count[$0]++}END{for(i in count)print count[i],i}'|sort -nr
[root@web-db bash]#
{% endhighlight %}

## 执行上一个历史命令：
### !!

## 22：在text editor中打开上一个命令并编辑：
### fc

## 23：浏览ASCII table
###　man 7 ascii 

## 24：ssh跳板
### 示例：
ssh -t reachable_host ssh unreachable_host

解说：

该命令通过reachable_host创建了一个到unreachable_host的ssh连接。通过在reachable_host上

执行ssh unreachable_host，-t强制ssh分配一个pseudo-tty（伪终端），该终端是ssh到

unreachable_host进行交互所必须的。

跳板可以有多个：

ssh -t host1 ssh -t host2 ssh -t host ssh -t host4 ...

## 25:ssh创建tunnel
### ssh -N -L2001:localhost:80 somemachine

This one-liner creates a tunnel from your computer's port 2001 to somemachine's port 80. 
Each time you connect to port 2001 on your machine, your connection gets tunneled to somemachine:80.

The -L option can be summarized as -L port:host:hostport. Whenever a connection is made to localhost:port, 
the connection is forwarded over the secure channel, and a connection is made to host:hostport from the remote machine.

The -N option makes sure you don't run shell as you connect to somemachine.

To make things more concrete, here is another example:

ssh -f -N -L2001:www.google.com:80 somemachine

This one-liner creates a tunnel from your computer's port 2001 to www.google.com:80 via somemachine. 
Each time you connect to localhost:2001, ssh tunnels your request via somemachine, 
where it tries to open a connection to www.google.com.

Notice the additional -f flag - it makes ssh daemonize (go into background) so it didn't consume a terminal.

## 26：使mount的输出更漂亮点
### 示例：
{% highlight bash %}
[root@web-db bash]# mount
/dev/mapper/VolGroup00-LogVol00 on / type ext3 (rw)
proc on /proc type proc (rw)
sysfs on /sys type sysfs (rw)
devpts on /dev/pts type devpts (rw,gid=5,mode=620)
/dev/sda1 on /boot type ext3 (rw)
tmpfs on /dev/shm type tmpfs (rw)
none on /proc/sys/fs/binfmt_misc type binfmt_misc (rw)
sunrpc on /var/lib/nfs/rpc_pipefs type rpc_pipefs (rw)
[root@web-db bash]# mount | column -t
/dev/mapper/VolGroup00-LogVol00  on  /                         type  ext3         (rw)
proc                             on  /proc                     type  proc         (rw)
sysfs                            on  /sys                      type  sysfs        (rw)
devpts                           on  /dev/pts                  type  devpts       (rw,gid=5,mode=620)
/dev/sda1                        on  /boot                     type  ext3         (rw)
tmpfs                            on  /dev/shm                  type  tmpfs        (rw)
none                             on  /proc/sys/fs/binfmt_misc  type  binfmt_misc  (rw)
sunrpc                           on  /var/lib/nfs/rpc_pipefs   type  rpc_pipefs   (rw)
[root@web-db bash]#
[root@web-db bash]# (echo "DEVICE PATH TYPE FLAGS" && mount|awk '$2=$4="";1')|column -t
DEVICE                           PATH                      TYPE         FLAGS
/dev/mapper/VolGroup00-LogVol00  /                         ext3         (rw)
proc                             /proc                     proc         (rw)
sysfs                            /sys                      sysfs        (rw)
devpts                           /dev/pts                  devpts       (rw,gid=5,mode=620)
/dev/sda1                        /boot                     ext3         (rw)
tmpfs                            /dev/shm                  tmpfs        (rw)
none                             /proc/sys/fs/binfmt_misc  binfmt_misc  (rw)
sunrpc                           /var/lib/nfs/rpc_pipefs   rpc_pipefs   (rw)
[root@web-db bash]#
{% endhighlight %}

## 27：使用wget下载网站
### wget --random-wait -r -p -e robots=off -U Mozilla www.baidu.com

--random-wait - wait between 0.5 to 1.5 seconds between requests.

-r - turn on recursive retrieving.

-e robots=off - ignore robots.txt.

-U Mozilla - set the "User-Agent" header to "Mozilla". 

Though a better choice is a real User-Agent like "Mozilla/4.0 

(compatible; MSIE 7.0; Windows NT 6.1; .NET CLR 2.0.50727; .NET CLR 3.5.30729; .NET CLR 3.0.30729)".

--limit-rate=20k - limits download speed to 20kbps.

-o logfile.txt - log the downloads.

-l 0 - remove recursion depth (which is 5 by default).

--wait=1h - be sneaky, download one file every hour.

## 28：创建并挂载一个临时的ram分区
### 示例：
{% highlight bash %}
mount -t tmpfs -o size=1024m tmpfs /mnt/tmpfs
[root@web-db bash]# mount
/dev/mapper/VolGroup00-LogVol00 on / type ext3 (rw)
proc on /proc type proc (rw)
sysfs on /sys type sysfs (rw)
devpts on /dev/pts type devpts (rw,gid=5,mode=620)
/dev/sda1 on /boot type ext3 (rw)
tmpfs on /dev/shm type tmpfs (rw)
none on /proc/sys/fs/binfmt_misc type binfmt_misc (rw)
sunrpc on /var/lib/nfs/rpc_pipefs type rpc_pipefs (rw)
[root@web-db bash]# mkdir /mnt/tmpfs;mount -t tmpfs -o size=1024m tmpfs /mnt/tmpfs
[root@web-db bash]# mount
/dev/mapper/VolGroup00-LogVol00 on / type ext3 (rw)
proc on /proc type proc (rw)
sysfs on /sys type sysfs (rw)
devpts on /dev/pts type devpts (rw,gid=5,mode=620)
/dev/sda1 on /boot type ext3 (rw)
tmpfs on /dev/shm type tmpfs (rw)
none on /proc/sys/fs/binfmt_misc type binfmt_misc (rw)
sunrpc on /var/lib/nfs/rpc_pipefs type rpc_pipefs (rw)
tmpfs on /mnt/tmpfs type tmpfs (rw,size=1024m)
[root@web-db bash]#
{% endhighlight %}

### 解说：
This command creates a temporary RAM filesystem of 1GB (1024m) and mounts it at /mnt. 
The -t flag to mount specifies the filesystem type and the -o size=1024m passes the size sets the filesystem size.

If it doesn't work, make sure your kernel was compiled to support the tmpfs. If tmpfs was compiled as a module, 
make sure to load it via modprobe tmpfs. If it still doesn't work, you'll have to recompile your kernel.

To unmount the ram disk, use the umount /mnt command (as root). 
But remember that mounting at /mnt is not the best practice. Better mount your drive to /mnt/tmpfs or a similar path.

If you wish your filesystem to grow dynamically, use ramfs filesystem type instead of tmpfs. 
Another note: tmpfs may use swap, while ramfs won't.

## 29：比较远程与本地文件的差异
### 示例：
ssh user@remote-host cat /path/file | diff /path/localfile -

## 原文链接：
原文链接：<a href="http://www.catonmat.net/blog/bash-one-liners-explained-part-one/">http://www.catonmat.net/blog/bash-one-liners-explained-part-one/</a>

