# linux常用指令



### 经常遇见的就是要端口被占用然后查看端口了

1.netstat  -anp  |grep  端口号

如下，我以3306为例，netstat  -anp  |grep  3306（此处备注下，我是以普通用户操作，故加上了sudo，如果是以root用户操作，不用加sudo即可查看），如下图1：

![img](https://images2017.cnblogs.com/blog/919962/201707/919962-20170728111009665-1483804266.png)





1.如何查看测试项目的日志

tail -f xx.out

如何查看最近100行日志

tail -1000  xx.out



还可以用 ss -s查看

![image-20210917112205271](D:\Typora图片\image-20210917112205271.png)



2.如何查看某个端口被占用



netstat -anp |grep 端口号

2.1查看当前所有已使用端口的情况

netstat -nultp



3.如何查找一个文件大小超过5MB的文件

find .-type  f -size +5M

4.如果知道一个文件名称，怎么查找这个文件在linux下的哪个目录 如：要查找tnsnames.ora文件

find / -name tnsnames.ora



5.查找文件

在



6.如何查看文件内容 

cat [选项] 要查看的文件

-n ：显示行号

cat -n /etc/profile

案例1:  /ect/profile 文件内容，并显示行号

  cat -n /etc/profile |  more

 cat **只能浏览文件，而不能修改文件**，为了浏览方便，一般会带上 管道命令 | more

cat xxx.txt | more (把cat xxx.txt 输出结果交给 more 处理)





more指令

more 要查看的文件

more指令是一个基于VI编辑器的文本过滤器，它以全屏幕的方式按页显示文本文件的内容。more指令中内置了若干快捷键，详见操作说明

less指令	

 less 要查看的文件



\> 指令 和 >> 指令

\> 输出重定向和 >> 追加



1.1.1 ln 指令 (link)

**案例****1:** **在****/home** **目录下创建一个软连接** **linkToRoot****，连接到** **/root** **目录**

ln -s /root/  linkToRoot 



如果希望设置某个服务自启动或关闭永久生效，要使用chkconfig指令

![img](file:///C:/Users/seajunnn/AppData/Local/Temp/msohtmlclip1/01/clip_image001.png)



![img](file:///C:/Users/seajunnn/AppData/Local/Temp/msohtmlclip1/01/clip_image001.png)



动态监控进程 查看cpu

top

![img](file:///C:/Users/seajunnn/AppData/Local/Temp/msohtmlclip1/01/clip_image002.jpg)



#### linux中怎么将文件合并

方法一：使用cat命令从文件中读入两个文件，然后将重定向到一个新的文件。这种方法可以一次性合并任意多个文件。

用法示例：

将file1.txt和file2.txt合并到file.txt

$ cat file1.txt file2.txt > file.txt





## linux只显示(只打印)文件中的某几行(中间几行)

从第3000行开始，显示1000行。即显示3000~3999行

cat filename | tail -n +3000 | head -n 1000

只显示第3000行

cat filename | tail -n +3000 | head -n 1



## linux查看文件的命令

cat more less

## 如果是大文件呢

less

## vi如何跳转到行尾

shift+g

## vi命令如何跳转到指定行

数字N +shift+g

## 实时追踪该文档的所有更新

tail -f 文件

## linux如何查看系统状态

vmstat 

![img](D:\Typora图片\1183448-20180117190014115-1020640097.png)

## **linux查看系统信息**

uname -a



## 如何查看哪个进程对cpu的占用最大

top指令







# 如何在日志文件中查找出访问最多的前10个IP地址





linux命令：

```linux
cat url.log | sort | uniq -c |sort -n -r -k 1 -t   ' ' | awk -F  '//'  '{print $2}' | head -10
```

现在来一一分析这些命令组合的含义。



### 1、原始访问日志

![log](https://img-blog.csdn.net/20150129020507265)

### 2、排序

```shell
cat t1.log | sort
```

![输入图片说明](https://img-blog.csdn.net/20150129020624754)

表示对data文件中的内容进行排序。sort命令是对于每一行的内容根据字典序（ASCII码）进行排序，这样可以保证重复的记录时相邻的。

### 3、合并相邻重复记录，统计重复数

```shell
cat t1.log | sort | uniq -c
```

`uniq -c `表示合并相邻的重复记录，并统计重复数。

### 4、对记录重新排序，需要用到sort命令的 -k 1 、-n、-r 三个命令参数

```shell
cat t1.log | sort | uniq -c | sort -k 1 -n -r
```

经过`uniq -c` 处理之后的数据格式形如"2 data"，第一个字段是数字，表示重复的记录数；第二个字段为记录的内容。

我们将对此内容进行排序，sort -k 1表示对于每行的第一个字段进行排序，这里即指代表重复记录数的那个字段，因为sort会按照ASCII,就会按照从小到大进行排序，数值2会排在数值11的前面，所以需要使用**-n** **参数指定sort命令按照数值大小进行排序**。**-r 表示逆序**，即按照从大到小的顺序进行排序。

### 5、使用awk -F命令分隔IP地址，并取数组的第二部分

```shell
cat t1.log | sort | uniq -c | sort -k 1 -n -r | awk -F '//' {print $2}
```

经过`sort data | uniq -c | sort -k 1 -n -r` 处理后的文本是 `http://192.168.1.100` 这样的格式,我们需要的是`192.168.1.100`这样的格式，需要去掉`http://`这些字段，采用`awk`才处理，`awk -F '//' `是将`http://192.168.1.100`分组成2部分 `http://` 和 `192.168.1.100`，`{print $2}`的作用是取数组的第二部分，即`192.168.1.100`

### 6、使用head -n命令选取前n行

```shell
cat t1.log | sort | uniq -c | sort -k 1 -n -r | awk -F '//' {print $2} | head -10
```

`head` 命令表示选取文本的前x行。通过`head -10` 就可以得到排序结果中前十行的内容。





## linux如何显示开机信息

**功能说明**

  demsg命令用于显示开机信息，内核会将开机信息存储在系统缓冲区（ring buffer）中，开机后可用dmesg命令查看，也可以在/var/log/目录中查看dmesg文件。用法如下：

**命令参数**

| **选项** | **含义**                            |
| -------- | ----------------------------------- |
| -c       | 显示开机信息后，清除ring buffer信息 |
| -s       | 设置缓冲区大小，默认设置为8192      |
| -n       | 设置记录信息的层级                  |



dmesg -c



## 查看消耗cpu过高的进程，并查看这个进程的运行目录的指令

先top指令 查找出 指令  查找出消耗CPU过高的PID

![img](https://img-blog.csdn.net/20180808151327700?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3lqY2xzeA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)



### [linux 查看进程所在目录](https://www.cnblogs.com/geloutingyu/p/9688003.html)

一下内容转自：https://blog.csdn.net/spring21st/article/details/50561550

通过 [ps](http://lovesoo.org/tag/ps) 及 [top](http://lovesoo.org/tag/top) 命令查看进程信息时，只能查到 [相对路径](http://lovesoo.org/tag/相对路径)，查不到的进程的详细信息，如 [绝对路径](http://lovesoo.org/tag/绝对路径) 等。这时，我们需要通过以下的方法来查看进程的详细信息：

[Linux](http://lovesoo.org/tag/linux) 在启动一个进程时，**系统会在 /[proc](http://lovesoo.org/tag/proc) 下创建一个以 PID 命名的文件夹，在该文件夹下会有我们的进程的信息，其中包括一个名为 exe 的文件即记录了[绝对路径](http://lovesoo.org/tag/绝对路径)，通过 [ll](http://lovesoo.org/tag/ll) 或 [ls](http://lovesoo.org/tag/ls) –l 命令即可查看**。

我们可以先通过 ps aux | grep process_name 找到对应 process 的 PID，再通过 ll /proc/PID 查到进程的绝对路径等信息







### 怎么杀死正在运行的[redis](https://www.nowcoder.com/jump/super-jump/word?word=redis)服务

ps-ef 



## 怎么从日志信息中找到一些敏感词

## 怎么查看linux的磁盘信息



## Linux文件权限相关信息



# [Linux系统查看有几块硬盘](https://www.cnblogs.com/oxspirt/p/6106055.html)



### df命令

![image-20211015094403291](E:\TyporaPic\image-20211015094403291.png)



完全虚拟化与半虚拟化



/ dev / sda是第一个检测到的IDE / SATA / SCSI类型的磁盘.在这种情况下,由管理程序模拟(完全虚拟化).

/ dev / vda是第一个检测到的半虚拟化磁盘驱动程序.如果两者都被引用到同一磁盘,则它比模拟的sdX设备更快,因为与模拟驱动器相比,其操作的开销更少.



可以看到有一块磁盘



![image-20211015095154388](E:\TyporaPic\image-20211015095154388.png)

![image-20211015095200937](E:\TyporaPic\image-20211015095200937.png)





# Linux下查找目录中所有文件中含有某个字符串，并且只打印出文件名

find . | xargs grep -ri "hello" -l





# 查询linux下有多少用户,Linux 查看系统现存所有用户命令

cat /etc/passwd



## linux如何配置一个网卡的IP？

ifconfig eth0 192.168.6.99 netmask 255.255.255.0 up





## 常用的linux指令

Top vmstat free df iostat ifstat



![image-20211104200530077](E:\TyporaPic\image-20211104200530077.png)





procs 

- r: 运行和等待CPU时间片的进程数，原则上1核CPU的运行队列不要超过2，整个系统的运行队列不能超过总核数的2倍

![image-20211104200654727](E:\TyporaPic\image-20211104200654727.png)

![image-20211104200745637](E:\TyporaPic\image-20211104200745637.png)





![image-20211104201640250](E:\TyporaPic\image-20211104201640250.png)

