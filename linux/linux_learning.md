# Linux技术笔记
## 1. 计算机概论

### CPU的种类
CPU内部有一些小指令集, 我们所使用的软件都要经过CPU内部的微指令集来完成, 指令集的设计主要被分为两种理念:
* 精简指令集RISC: 微指令集较为精简, 每个指令运行的时间都很短, 完成的动作也很单纯, 指令的执行性能较好, 但是如果要做复杂的事情的话就需要多个指令来完成(手机, 导航系统, 网络设备等)
* 复杂指令集CISC: CISC在微指令集的每一个小指令可以执行一些较为低阶的硬件操作, 指令数目多而且复杂, 每条指令的长度并不相同, 由于指令执行复杂, 每条指令执行时间较长, 但是可以处理的内容较为丰富. (个人计算机)

> x86架构CPU: 由于最早Intel的CPU代号为8086, 后来依此架构开发出别的CPU, 所以这种架构的CPU就称为x86架构, CPU之间的差异主要在于微指令集的不同, 新的x86CPU大多数含有比较先进的微指令集, 可以加速多媒体程序的运作, 也能够加强虚拟化的性能

### 容量单位
计算机只能通过有没有通电来记录信息, 所以它只认识0和1, 这个单位称之为bit, 但是这个能存储的东西太小, 所以一般每一份简单的数据都会用一个字节Byte来记录, 他们的关系为
```1byte = 8bit```
除此之外, 还有一些别的单位来表示容量大小:

![image-20190927163639197](/Users/nacht/Documents/Talking-To-The-Moon/tech_notes/linux/assets/image-20190927163639197.png)

### 速度单位
由于网络中使用的是bit作为单位, 所以网络中经常使用的单位是Mbps, 也就是Mbits per second, 如果转换成字节的单位, 有以下对应关系
`8Mbps = 1MBps`

> 小写b代表bit, 大写B代表byte

## 2. Linux的主机规划与磁盘分区
### 常见的硬盘机在linux文件系统中的命名

![image-20190927163648703](/Users/nacht/Documents/Talking-To-The-Moon/tech_notes/linux/assets/image-20190927163648703.png)
个人计算机常见的硬盘接口有两种, IDE和SATA, 现在主流是SATA接口

- IDE接口一般来说有两个接口, 一个称之为master, 一个称之为slave, 命名按照接口1master=>接口1slave=>接口2master=>接口2slave=>…顺序命名,如下图
![image-20190927163658539](/Users/nacht/Documents/Talking-To-The-Moon/tech_notes/linux/assets/image-20190927163658539.png)
- SATA接口按照系统读取到磁盘的顺序命名

### 磁盘分区
#### 磁盘分区的意义
- 保证数据安全, 对一个槽执行的数据清空操作不会影响到其他槽
- 系统的性能, 数据集中有助于数据读取的速度与性能
#### 磁盘分区的实现
磁盘第一个扇区记录了两个重要信息, 分别是主要启动记录区MBR(安装开机管理程序)和分区表(记录这个硬盘分区的状态, 64bytes)
分区表最多只能容纳四条分区的记录, 可以为主要分区和逻辑分区
![image-20190927163719334](/Users/nacht/Documents/Talking-To-The-Moon/tech_notes/linux/assets/image-20190927163719334.png)
延伸分区指的是是利用额外的扇区来记录扩展分区信息,如果是上图中的分区的话, Linux中的装置文件命名如下:

- P1: /dev/hda1
- P2: /dev/hda2
- L1: /dev/hda5
- L2: /dev/hda6
- …此处省略
L1从hda5开始是因为hda1-hda4是属于第一扇区的分割表中的分区, 所以如果有延伸分区, 必须从5开始命名
对于主要分区和延伸分区的特性做一个总结:
- 主要分区和逻辑分区最多只有四个
- 延伸分区最多只有一个(操作系统限制)
- 能够被格式化后作为数据存储的分割槽有主要分割槽(P1)和逻辑分割槽(L1-L5), P2无法格式化
- 逻辑分割槽的数量受操作系统和硬盘类型(IDE SATA)影响

### 开机启动流程
1. BIOS识别作为启动盘的硬盘
2. BIOS到启动盘中读取第一个扇区的MBR
3. 根据MBR中的开机引导程序读取硬盘中的操作系统核心程序(此处有部分引导程序可以通过不同的loader选择启动不同的操作系统,这就是多重引导, 也可以说是多系统)
4. 启动系统核心程序

### 文件系统与目录树的关系(挂载)
挂载指的是利用一个目录当成进入点, 将磁盘分区槽的数据放在该目录下, 可以把目录理解为一个指针, 指向一个分区槽, 读写目录实质上是通过这个指针对磁盘文件进行操作
如果unmount之后, 指针不再指向对应的磁盘, 就无法读取到文件, 目录会变成空目录

## 3. Linux的文件与权限, 目录管理
### ls 控制台信息各部分解析
```shell
▶ ls -l
total 8
drwxr-xr-x  4 nacht  staff  128 Jun 24 10:13 Linux
```

第一栏表示文件的类型与权限
d : 文件类型, d代表目录, -代表普通文件, l代表快捷方式, b代表装置文件里面的存储设备, c代表串行端口设备(键盘,鼠标等)
第一个rwx: 该文件拥有者的权限
第二个r-x: 该文件拥有者用户组的权限
第三个r-x: 除了该组之外其他用户的权限
数字4: 如果是文件, 代表硬链接数, 如果是目录, 表示一级子目录数(包含两个隐藏目录)
Nacht: 该文件所属用户
Staff: 该文件所属用户组
数字128: 文件大小
日期: 文件创建日期/最近修改日期
Linux: 文件/目录名

### 改变文件的属性与权限
- 改变所属群组 chgrp
- 改变所有者  chown
- 改变权限 chmod

### Linux根目录以及下面各目录的含义
- /bin : /bin放置的是单人维护模式下还能被操作的指令, 在bin下面放置的指令可以被root用户和一般用户使用
- /boot : Linux的核心文件, 开机选项和开机所需文件, 如果Linux是使用grub这个开机管理程序的话, 该目录下面还会有一个grub目录
- /dev : 任何装置和接口设备都会存在于这个目录, 比较重要的有/dev/null, /dev/zero, /dev/tty, /dev/Ip*, /dev/sd*, /dev/hd*等
- /etc : 用来放置系统主要的配置文件, 例如人员的账号密码文件, 各种服务的启动脚本等, 一般来说这个目录下面的文件所有用户都能访问, 但是只有root用户才能修改
- /home : 系统默认的家目录, 每新增一个账号下面都会加一个子目录
- /lib : 系统开机会用到的依赖, 以及/bin和/sbin下面的程序运行所需要的依赖
- /media : 软盘, 光盘, DVD等可移除的装置都会挂载在这个目录
- /opt : 放置第三方软件的目录
- /root : 系统管理员账户的家目录
- /sbin : 用于放置一些修改系统的程序, 例如fdisk, fsck, ifconfig, init, mkfs等, 一般只有root用户才能执行它们来对系统做修改, 一般用户只能执行来做查询
- /srv : 一些网络服务启动后读取数据的目录, 常见的有ftp,WWW等
- /tmp : 让一般用户或者正在执行的程序临时放置数据的地方
- /lost+found : 使用ext2或ext3文件系统格式才会产生的一个目录, 用于在文件系统发生错误的时候将一些遗失的文件碎片放在这个目录下, 例如挂载一个磁盘到/disk,那么就会自动生成一个/disk/lost+found目录
- /proc : 这个文件夹本身的数据都放在内存中, 里面有系统核心, 进程信息, 外接硬件的状态, 网络状态等, 它本身不占用任何空间
- /sys : 和/proc类似, 记录已经加载的核心模块和硬件信息
- /usr : Unix Software Resource的缩写, 里面放的数据属于可分享的和不可变动的, 下面有如下常用的子目录:
/usr/bin : 绝大部分用户使用的指令都放在这里与/bin的不同之处在于/bin需要随开机过程加载, 而/usr/bin不是
/usr/include : c/c++等程序语言的档头和包含档放置处
/usr/lib : 应用软件的函数库, 目标文件, 和一些用户平时很少用到的可执行文件
/usr/local : 用户自己安装的程序
/usr/sbin : 非系统正常运作所需要的指令
/usr/share : 共享文件
/usr/src : 放置源码的地方
- /var : 放置一些经常性变动的文件, 由于这个目录经常读写, 最好不要与其他目录放在同一个分割槽, 这样当这个分割槽出现问题的时候不会影响到其他重要的目录
/var/cache : 放置缓存文件
/var/lib : 程序执行的过程中需要使用到的数据文件的目录
/var/lock : 锁
/var/log/ : 登录文件放置的目录
/var/mail : 放置个人电子邮箱的目录
/var/run : 某些程序或者服务启动之后会把它们的PID放到这个目录
/var/spool : 放置一些队列数据

### 关于执行文件路径的变量: $PATH
当我们在执行一个指令的时候, 系统会先根据PATH的设定去每个目录下寻找文件名为该指令名的可执行文件, 如果有多个同名文件, 那么第一个找到的可执行文件将会被执行

### 查看文件内容
- cat : 从第一行开始显示文件内容
- tac : 从最后一行开始显示文件内容, 注意他是cat的倒写
- nl : 显示的同时显示行号
- more : 一页一页的显示文件内容
- less : 与more相似, 但是可以向前翻页
- head : 只看头几行
- tail : 只看后几行
- od : 以二进制方式读取文件内容

### 查看文件类型: file
```shell
▶ file fuck.txt
fuck.txt: ASCII text
▶ file get-pip.py
get-pip.py: a /usr/bin/env python script text executable
```


## 4. Linux磁盘和文件系统管理

### Linux文件系统特性
Linux的文件系统可以分为下面三部分:
- superblock: 记录文件系统的整体信息, 例如inode/block的数量, 使用量, 剩余量, 以及文件系统的格式和相关信息等
- inode : 记录文件的权限和属性, 一个文件占用一个inode, 同时记录该文件所在的block编号
- block : 记录文件数据, 当文件比较大的时候会占用多个block

### 查看文件系统的相关信息
先用df指令查看当前机器上面挂载的文件系统信息, 再用dumpe2fs+设备名查看对应文件系统的详细信息

### 目录的inode和block作用
目录的inode记录了该目录对应的block编号, 目录的block记录了该目录下文件的文件名和该文件名对应的inode数据

> 所以, inode本身不会记录文件名, 文件名是记录在它所在的目录的block中, 所以要读取一个文件, 势必要通过它所在目录的inode和block, 进而获取到该文件的inode, 然后根据它inode里面记录的block编号到对应的block上面去读取文件

### filesystem大小和读取性能的问题

如果一个文件系统规划的比较大, 那么在频繁的读写之后文件可能会出现被写入到分散的block中的情况, 这种情况下磁盘读取头需要到不同的block中读取数据, 会降低读取数据的效率, 为了解决这个问题, 可以将该文件系统的文件复制出来, 格式化之后再重新写入, 但是在刚开始规划分区的时候应该考虑到实际使用情况, partition并不是越大越好

### 软连接和硬连接

- 硬链接：简单说，文件名就是文件的硬链接，硬链接就是给文件起了个别名，对应的 inode 与原文件一样
- 软链接：简单说，类似于快捷方式，它有自己单独的 inode，指向了被链接的文件（跟路径关联)
![image-20190927163733659](/Users/nacht/Documents/Talking-To-The-Moon/tech_notes/linux/assets/image-20190927163733659.png)

硬连接的特点: 
- 不能跨文件系统
- 不能对目录建立硬连接

## 5. 文件的压缩

### 常见的文件后缀
- *.Z : compress程序压缩的档案
- *.gz : gzip程序压缩的档案
- *.bz2 : bzip2程序压缩的档案
- *.tar : tar程序打包的档案, 没有被压缩过
- *.tar.gz : tar程序打包的档案, 并被gzip压缩过
- *.tar.bz2 : tar程序打包的档案, 并被bzip2压缩过

> 由于压缩只能对一个文件进行压缩, 所以在压缩目录的时候要先用tar进行打包再使用对应的压缩工具进行压缩

### 压缩指令
- compress : 比较老的压缩工具, 现在一般不使用, gzip可以解压用compress压缩的文件
- gzip : 目前使用最广泛的指令, 可以解压compres, zip , gzip压缩的文件, 可以用zcat查看被压缩的文件
- bzip2 : 提供比gzip更高的压缩比, 可以用bzcat查看被压缩后的文件

### 完整备份指令dump
Dump在进行文件系统的备份时可以指定等级, 比如对某个文件系统进行dump备份,那么在第二次备份, 如果指定level为1, 那么只会记录和第一次备份有差异的文件
![image-20190927163745355](/Users/nacht/Documents/Talking-To-The-Moon/tech_notes/linux/assets/image-20190927163745355.png)

> dump的level功能只能在备份对象时文件系统的时候才有用, 当备份对象为目录的时候只能选择level0来作为备份等级, 也就是完整备份

## 6. vi和vim
### vi的三种模式
- 一般模式: vi刚打开的时候就是这个模式, 可以进行删除字符, 删除整行, 复制粘贴等操作
- 编辑模式: 一般模式输入i, I, o, O, a, A, r, R 之后进入的编辑模式, 这个就是正常的文本编辑模式
- 指令命令模式: 输入:/?三个中的一个可以进入这个模式, 搜索, 保存编辑, 大量替换字符, 离开vi等操作都是在这个模式中进行的

### vi常用操作
- n<space> : 先输入光标移动的字符数, 再按space , 光标向后跳动n个字符
- 0 : 移动到这一行最前面一个字符处
- $ : 移动到这一行最后面一个字符处
- n<G> : 移动到文档的第n行, 例如20G会调到第20行
- gg: : 移动到文档的第一行, 相当于1G
- n<Enter> : 向下移动n行
- /word : 向下搜索某个关键词
- ?word : 向上搜索某个关键词
- n : 重复前一个搜索动作
- N : 反向执行前一个搜索动作
- x : 向后删除一个字符
- X : 向前删除一个字符
- nx : 向后删除n个字符
- dd : 删除光标所在的一行
- yy : 复制光标所在的一行
- p : 粘贴在下一行
- P : 粘贴在上一行
- u : 撤销前一个操作
- ctrl + r : 复制前一个操作

## 7. bash
### 为命令设置别名
```shell
alias lm = 'ls -al'
```


### bash编程
#### 变量
1. 变量设置语法为a=5, 等号两边不能有空格
2. 等号右边不能直接接空格符, 例如a=fuck you这个语句是非法的
3. 变量名字只能是字母和数字, 但是不能以数字开头
4. 如果变量的值里面带有空格可以用单引号或者双引号包起来, 单引号和双引号的区别在于双引号里面的$变量名会被替换为对应的变量值, 而单引号不会
5. 可用\将特殊字符转义为一般字符
6. 如果需要获取一个指令的返回值, 可以用反单引号`或者$()将指令包起来
7. 如果一个变量由另外一个变量扩展, 可以用”$变量名”或者${变量名},例如PATH=“$PATH”:/home/bin
8. 如果变量需要在其他子程序中用到, 可以用export来生成环境变量:export PATH
9. 取消变量设置的指令为unset 变量名

#### 读取键盘输入
read指令用于读取键盘输入,-p可以加上提示语句, -t可以指定等待输入的时间
```shell
read -p 'input your name :' name
echo "your name is $name"
输出: 
input your name :fuck
your name is fuck
```


#### 声明变量类型 declare
```shell
root@nacht:~
▶ sum=300+300

root@nacht:~
▶ echo $sum
300+300

root@nacht:~
▶ declare -i sum=300+300

root@nacht:~
▶ echo $sum
600
```

#### 数组
```shell
root@nacht:~
▶ var[1]=0

root@nacht:~
▶ var[2]=1

root@nacht:~
▶ echo $var
0 1

root@nacht:~
▶ echo ${var[1]}
0
```

#### 变量字符串处理
假设有变量PATH=/usr/kerberos/sbin:/usr/kerberos/bin:/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin:/root/bin
- 从前面开始删除符合表达式的最短字符串
`${PATH#/*:}`=>删除/usr/kerberos/sbin: ,因为从左边开始符合`/*:`的最短字符串为/usr/kerberos/sbin:
- 从前面开始删除符合表达式的最长字符串
`${PATH##/*:}`=>删除/usr/kerberos/sbin:/usr/kerberos/bin:/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin: , 因为从左边开始符合`/*:`的最长字符串为/usr/kerberos/sbin:/usr/kerberos/bin:/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin: 
- 从后往前删除符号为%,其他和从前面删除一样(%最短,%%最长)
- 从前面开始替换符合表达式的第一个字符串`${PATH/sbin/SBIN}`=>将第一个sbin替换为SBIN
- 从前面开始替换符合表达式的所有字符串`${PATH//sbin/SBIN}`=>将所有的sbin替换为SBIN
[image:E9FB56F6-1C11-4F9C-B5A5-A164D9E4FB55-454-00028BC0520350E4/FC2817C6-82F5-496B-8DDB-D1F5B325FAF8.png]

#### 变量的测试和内容变换

- 判断某个变量是否存在, 如果不存在则赋予默认值:
```shell
root@nacht:~
▶ str='this'

root@nacht:~
▶ echo ${str-fuccck}
this

root@nacht:~
▶ echo ${strr-fuccck}
fuccck
```

> 注意空字符串也会被认为是存在, 不会被赋予-后面的值


- 判断某个变量是否存在/是否为空字符串, 如果不存在/为空字符串则赋予默认值
```shell
root@nacht:~
▶ str=''

root@nacht:~
▶ echo ${str:-haha}
haha

root@nacht:~
▶ echo ${strr:-haha}
haha
```

> 变量为空字符串也会被赋予默认值

其他一些赋值用法
<img src='http://ww1.sinaimg.cn/large/006tNc79gy1g5hx6ay8hoj30my0cdq53.jpg'/>


### 别名设置
```shell
root@nacht:~                                                                                                     ⍉
▶ alias ff='df -h'

root@nacht:~
▶ ff
Filesystem      Size  Used Avail Use% Mounted on
/dev/vda1        40G  1.8G   36G   5% /
tmpfs           1.9G     0  1.9G   0% /dev/shm
```


> unalias可以用来清除已经定义的别名

### 通配符和特殊符号
![image-20190927163759784](/Users/nacht/Documents/Talking-To-The-Moon/tech_notes/linux/assets/image-20190927163759784.png)

```shell
root@nacht:~
▶ ls
get-pip.py  haha.sh  python  softwares  test

root@nacht:~
▶ ls *.py
get-pip.py

root@nacht:~
▶ ls ??????
python

root@nacht:~
▶ ls [opq]ython
python

root@nacht:~
▶ ls [o-q]ython
python

root@nacht:~
▶ ls [^t]est
zsh: no matches found: [^t]est
```

### 输入输出重定向

- 标准输入: 代码为0 , 用<或<<
- 标准输出: 代码为1 , 用>(重写文件)或>>(追加到文件)
- 标准错误输出: 代码为2 , 用2>(重写文件或)2>>(追加到文件)

> 如果想让屏幕不输出任何信息的同时又不写入文件, 可以把输出重定向到/dev/null中

> 如果想要把正确和错误的输出信息都写到同一个文件, 假设这个文件叫out.txt,应该用> out.txt 2> &1而不能用> out.xtx 2> out.txt因为后者会造成正确和错误输出的次序混乱

### 执行多个指令

```shell
#多个指令之间没有相关性,cmd1,cmd2和cmd3会串行执行
cmd1;cmd2;cmd3
#cmd2只有当cmd1成功执行后才会被执行
cmd1 && cmd2
#cmd2只有当cmd1执行失败后才会被执行
cmd1 || cmd2

root@nacht:~
▶ ls && ls
get-pip.py  haha.sh  python  softwares  test
get-pip.py  haha.sh  python  softwares  test

root@nacht:~
▶ ls || ls
get-pip.py  haha.sh  python  softwares  test

root@nacht:~
▶ ls ; ls
get-pip.py  haha.sh  python  softwares  test
get-pip.py  haha.sh  python  softwares  test
```

### 管道操作符

支持管道操作符的常用命令: 

- cut: 截取字符串
```shell
root@nacht:~                                                                                                        ⍉
▶ cat >fuck.txt
a|b|c|d|e|f

root@nacht:~
▶ cat fuck.txt
a|b|c|d|e|f

root@nacht:~
▶ cat fuck.txt|cut -d '|' -f 1
a
root@nacht:~                                                                                                        ⍉
▶ cat fuck.txt|cut -d '|' -f 2
b

root@nacht:~
▶ cat fuck.txt|cut -d '|' -f 3
c
```

- grep: 查找字符串
```shell
root@nacht:~
▶ cat > fuck.txt
this is a string
this is an apple
this is your dad

root@nacht:~
▶ cat fuck.txt| grep this
this is a string
this is an apple
this is your dad

root@nacht:~
▶ cat fuck.txt| grep your
this is your dad
```

- sort: 排序
```shell
root@nacht:~                                                                                                                                                                                                                                
▶ cat > fuck.txt
c
b
a
d
g
e
r
p

root@nacht:~
▶ cat fuck.txt| sort
a
b
c
d
e
g
p
r
```

- uniq: 去重
```shell
root@nacht:~
▶ cat > fuck.txt
a
a
a
b
b
b

root@nacht:~
▶ cat fuck.txt | uniq
a
b
```

- wc: 统计字数/字符数/行数
```shell
root@nacht:~
▶ cat > fuck.txt
a
b
c
^C

root@nacht:~                                                                                                                                                                                                                                ▶ cat fuck.txt | wc -l
3
```

- tee: 双重重定向流
```shell
root@nacht:~
#同时把ls的结果输出到屏幕和文件ls.txt中
▶ ls | tee ls.txt
fuck.txt
get-pip.py
haha.sh
ls.txt
python
softwares
test

root@nacht:~
▶ cat ls.txt
fuck.txt
get-pip.py
haha.sh
ls.txt
python
softwares
test
```

- tr: 进行字符的删除/替换
```shell
root@nacht:~
▶ cat ls.txt
fuck.txt
get-pip.py
haha.sh
ls.txt
python
softwares
test

root@nacht:~
▶ cat ls.txt | tr -d .
fucktxt
get-pippy
hahash
lstxt
python
softwares
test

root@nacht:~
▶ cat ls.txt | tr [a-z] [A-Z]
FUCK.TXT
GET-PIP.PY
HAHA.SH
LS.TXT
PYTHON
SOFTWARES
TEST
```

- col: 将tab转成空格
- xargs: 将输出结果作为后面接的指令的参数
```shell
root@nacht:~
▶ cat >fuck.txt
/root

root@nacht:~
▶ cat fuck.txt | xargs ls
fuck.txt  haha.sh  ls.txt  python  softwares  test
```



### sed

**sed** 是一种流编辑器，它是文本处理中非常中的工具，能够完美的配合正则表达式使用，功能不同凡响。处理时，把当前处理的行存储在临时缓冲区中，称为“模式空间”（pattern space），接着用sed命令处理缓冲区中的内容，处理完成后，把缓冲区的内容送往屏幕。接着处理下一行，这样不断重复，直到文件末尾。文件内容并没有 改变，除非你使用重定向存储输出。Sed主要用来自动编辑一个或多个文件；简化对文件的反复操作；编写转换程序等。

### awk

**awk** 是一种编程语言，用于在linux/unix下对文本和数据进行处理。数据可以来自标准输入(stdin)、一个或多个文件，或其它命令的输出。它支持用户自定义函数和动态正则表达式等先进功能，是linux/unix下的一个强大编程工具。它在命令行中使用，但更多是作为脚本来使用。awk有很多内建的功能，比如数组、函数等，这是它和C语言的相同之处，灵活性是awk最大的优势。



## 8. Shell Script

### 四则运算

shell中可以采用$((计算表达式)) 的形式来进行四则运算, 这样可以不用进行变量的显式声明, 否则如果要进行证书的加减乘除需要用declare关键字声明变量

```shell
root@nacht:~                                                                                                                                                                                                                                ⍉
▶ cat haha.sh
read -p "input a number" a
read -p "input a number" b
echo "the sum of this two number is $((a+b))"

root@nacht:~
▶ ./haha.sh
input a number6
input a number6
the sum of this two number is 12
```



### shell执行的差异

- 利用直接执行的方式执行脚本

  直接用./或者bash xxx.sh的方式执行脚本的时候, 会在bash主程序下面新起一个子环境来执行脚本内的指令

  ```shell
  root@nacht:~
  ▶ cat haha.sh
  a=5
  
  root@nacht:~
  ▶ ./haha.sh
  
  root@nacht:~
  ▶ echo ${a}
  
  
  root@nacht:~
  ▶ source haha.sh
  
  root@nacht:~
  ▶ echo ${a}
  5
  ```

  ![image-20190927163813147](/Users/nacht/Documents/Talking-To-The-Moon/tech_notes/linux/assets/image-20190927163813147.png)

- 用source执行脚本

  用source执行脚本是直接在父程序bash中执行, 所以上面的示例中设置的a变量可以在父bash中获取到数值



### 利用test进行条件判断

可以用test和&&以及||配合使用来测试系统的某些文件或者属性

例如

```shell
root@nacht:~
▶ ls
fuck.txt  haha.sh  ls.txt  python  softwares  test

root@nacht:~
▶ test -e haha.sh && echo "haha.sh exists"||echo "not exist"
haha.sh exists

root@nacht:~
▶ test -e haha1.sh && echo "haha1.sh exists"||echo "not exist"
not exist
```



### shell的默认参数

调用shell脚本的时候会默认将调用时候的参数以```$1```,``` $2```...等等的方式传入到脚本中

![image-20190927164031009](/Users/nacht/Documents/Talking-To-The-Moon/tech_notes/linux/assets/image-20190927164031009.png)

```shell
root@nacht:~
▶ cat haha.sh
echo "first para : $1"
echo "first para : $2"
echo "first para : $3"

root@nacht:~
▶ ./haha.sh hello to you
first para : hello
first para : to
first para : you
```



### if then条件判断表达式

```shell
root@nacht:~
▶ cat haha.sh
a=5
#注意中括号中的空格, 这些空格都是不可省略的
if [ $a == 5 ]; then
    echo "the value of a is 5"
fi


root@nacht:~
▶ ./haha.sh
the value of a is 5
```



多重条件判断

![image-20190927164046907](/Users/nacht/Documents/Talking-To-The-Moon/tech_notes/linux/assets/image-20190927164046907.png)



### function功能

shell中定义函数的语法:

```shell
function fname(){
	#Todo
}
```



```shell
root@nacht:~
▶ cat haha.sh
function sayHello(){
    echo "hello son";
}
sayHello;

root@nacht:~
▶ ./haha.sh
hello son
```



### 循环loop

![image-20190927164117953](/Users/nacht/Documents/Talking-To-The-Moon/tech_notes/linux/assets/image-20190927164117953.png)



输出9*9乘法表:

```bash
root@nacht:~
▶ cat haha.sh
a=1
while [ $a -le 9 ]
do
    b=1
    while [ $b -le $a ]
    do
     echo -n "$b*$a=$((b*a)) "
     b=$((b+1))
    done
    a=$((a+1))
    echo ""
done

root@nacht:~
▶ ./haha.sh
1*1=1
1*2=2 2*2=4
1*3=3 2*3=6 3*3=9
1*4=4 2*4=8 3*4=12 4*4=16
1*5=5 2*5=10 3*5=15 4*5=20 5*5=25
1*6=6 2*6=12 3*6=18 4*6=24 5*6=30 6*6=36
1*7=7 2*7=14 3*7=21 4*7=28 5*7=35 6*7=42 7*7=49
1*8=8 2*8=16 3*8=24 4*8=32 5*8=40 6*8=48 7*8=56 8*8=64
1*9=9 2*9=18 3*9=27 4*9=36 5*9=45 6*9=54 7*9=63 8*9=72 9*9=81
```



for in do done循环

```shell
root@nacht:~
▶ cat dd.sh
for a in $1 $2 $3
do
  echo "para===>$a"
done

root@nacht:~
▶ ./dd.sh a b hello
para===>a
para===>b
para===>hello
```



for(( ;; )) do done 循环

```shell
root@nacht:~
▶ cat dd.sh
for((i=0;$i<10;i++))
do
  echo $i
done

root@nacht:~
▶ ./dd.sh
0
1
2
3
4
5
6
7
8
9
```



### shell的追踪和debug

```shell
#不执行脚本, 只检查语法错误
sh -n xxx.sh
#执行脚本之前打印出脚本的内容
sh -v xxx.sh
#详细列出脚本的执行过程
sh -x xxx.sh
```













































































