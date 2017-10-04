---

layout: post
title: linux环境变量学习整理
category: codings
tags: [Linux, Linux-basics, Linux-environment]
excerpt_separator: "<!--more-->"

---


今天上午玩服务器，把师姐的环境变量弄崩了，折腾了一下，不知怎的又恢复了，做一下记录供将来参考。查资料发现是提示符（prompt）出错了

<!--more-->

先留一份备忘
```
# .bashrc

# Source global definitions
if [ -f /etc/bashrc ]; then
	. /etc/bashrc
fi

# User specific aliases and functions

export PATH=$PATH:/pub7/program/R-3.1.2/lib64/R/bin
export PATH=$PATH:./
export PATH=$PATH:/pub7/wenyh2013/wanghh/programs/bowtie2-2.2.3

```
##环境变量的意义
环境变量，即全局变量（/etc/bashrc），相对于局部变量(~/.bashrc)，涉及范围更广。
自己的理解是方便调用。

>什么是环境变量呢？简要的说，就是指定一个目录，运行软件的时候，相关的程序将会按照该目录寻找相关文件。设置变量对于一般人最实用的功能就是：不用拷贝某些dll文件到系统目录中了，而path这一系统变量就是系统搜索dll文件的一系列路径； 
在linux系统下，如果你下载并安装了应用程序，很有可能在键入它的名称时出现“command not found”的提示内容。如果每次都到安装目标文件夹内，找到可执行文件来进行操作就太繁琐了。这涉及到环境变量 PATH 的 设置 问题，而 PATH 的 设置 也 是在linux下定制环境变量的一个组成部分。

Shell变量分为本地变量和环境变量。 
1. 本地变量 －－ 在用户现有运行的脚本中使用 
```
1) 定义本地变量 格式： variable-name=value 
例子：[root@jike1 /root]# LOCALTEST="test" 
[root@jike1 /root]# echo \$LOCALTEST test 
```
    2) 显示本地变量 格式： set 
     例子：[root@chinaitlab root]# set 
```
3) 清除本地变量 格式：unset variable-name 
例如：[root@jike1 /root]# unset LOCALTEST 
```
此时再执行echo \$LOCALTEST将看不到变量LOCALTEST的输出。
2. 环境变量 －－ 在所有的子进程中使用 
```
1) 定义环境变量 格式： export variable-name=value （与本地变量的定义相比，多了一个export关键字） 
例子：[root@chinaitlab /root]# export DOMAIN="chinaitlab.com" 
[root@ chinaitlab shell]# vi testenv.sh 
\#!/bin/bash 
echo $DOMAIN 
[root@chinaitlab shell]# chmod +x testenv.sh 
[root@chinaitlab shell]# ./testenv.sh 
chinaitlab.com 
```
```
2) 显示环境变量 格式： env （本地变量的显示使用set，环境变量的显示使用env） 
例子： [root@chinaitlab test]# env 
```
```
3) 清除环境变量 格式：unset variable-name （用法与本地变量相同，都使用unset） 
例子： [root@chinaitlab shell]# unset DOMAIN 
此时再执行./testenv.sh将看不到变量DOMAIN的输出。
```



##配置方法
>如想将一个路径加入到 \$PATH 中，可以像下面这样做
1. 控制台中,不赞成使用这种方法，因为换个shell，你的设置就无效了，因此这种方法仅仅是临时使用，以后要使用的时候又要重新设置，比较麻烦。 这个只针对特定的shell 
\$PATH="\$PATH:/my_new_path" （关闭shell，会还原PATH）
2. 修改/etc/profile文件,如果你的计算机仅仅作为开发使用时推荐使用这种方法，因为所有用户的shell都有权使用这些环境变量，可能会给系统带来安全性问题。 这里是针对所有的用户的,所有的shell; 
vi /etc/profile 
在里面加入:
     export PATH="$PATH:/my_new_path" 
3. 修改.bashrc文件,这种方法更为安全，它可以把使用这些环境变量的权限控制到用户级别,这里是针对某一个特定的用户，如果你需要给某个用户权限使用这些环境变量，你只需要修改其个人用户主目录下的.bashrc文件就可以了。
 vi /root/.bashrc 
在里面加入： 
export PATH="\$PATH:/new_path" 
后两种方法一般需要重新注销系统才能生效，最后可以通过echo命令测试一下： 
 echo \$PATH 
输出已经是新路径了。

##相关设置文件
>用户登录后加载profile和bashrc的流程如下: 
1)/etc/profile----->/etc/profile.d/*.sh 
2)\$HOME/.bash_profile--->\$HOME/.bashrc--->/etc/bashrc 
说明: 
bash首先执行/etc/profile脚本,/etc/profile脚本先依次执行/etc/profile.d/*.sh 
随后bash会执行用户主目录下的.bash_profile脚本,.bash_profile脚本会执行用户主目录下的.bashrc脚本, 
而.bashrc脚本会执行/etc/bashrc脚本 
至此,所有的环境变量和初始化设定都已经加载完成. 
bash随后调用terminfo和inputrc，完成终端属性和键盘映射的设定. 
其中PATH这个变量特殊说明一下: 
如果是超级用户登录,在没有执行/etc/profile之前,PATH已经设定了下面的路径: 
/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin 
如果是普通用户,PATH在/etc/profile执行之前设定了以下的路径: 
/usr/local/bin:/bin:/usr/bin 
这里要注意的是:在用户切换并加载变量,例如su -,这时，如果用户自己切换自己,比如root用户再用su - root切换的话,加载的PATH和上面的不一样. 
准确的说，是不总是一样.所以，在/etc/profile脚本中，做了如下的配置: 
if [ `id -u` = 0 ]; then 
pathmunge /sbin 
pathmunge /usr/sbin 
pathmunge /usr/local/sbin 
fi 
如果是超级用户登录,在/etc/profile.d/krb5.sh脚本中,在PATH变量搜索路径的最前面增加/usr/kerberos/sbin:/usr/kerberos/bin 
如果是普通用户登录,在/etc/profile.d/krb5.sh脚本中,在PATH变量搜索路径的最前面增加/usr/kerberos/bin 
在/etc/profile脚本中,会在PATH变量的最后增加/usr/X11R6/bin目录 
在\$HOME/.bash_profile中,会在PATH变量的最后增加$HOME/bin目录 
以root用户为例,最终的PATH会是这样(没有其它自定义的基础上) 
/usr/kerberos/sbin:/usr/kerberos/bin:/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin:/usr/X11R6/bin:/root/bin 
以alice用户(普通用户)为例 
/usr/kerberos/bin:/usr/bin:/bin:/usr/X11R6/bin:/home/alice/bin
~/.bash_profile  用户登录时被读取，其中包含的命令被执行
~/.bashrc  启动新的shell时被读取，并执行
~/.bash_logout  shell 登录退出时被读取


Linux 中环境变量的文件

    当你进入系统的时候，linux就会为你读入系统的环境变量，这些环境变量存放在什么地方，那就是环境变量的文件中。Linux 中有很多记载环境变量的文件，它们被系统读入是按照一定的顺序的。

1./etc/profile 

此文件为系统的环境变量，它为每个用户设置环境信息，当用户第一次登录时，该文件被执行。并从/etc/profile.d 目录的配置文件中搜集shell 的设置。

    这个文件，是任何用户登陆操作系统以后都会读取的文件（如果用户的shell 是csh 、tcsh 、zsh ，则不会读取此文件），用于获取系统的环境变量，只在登陆的时候读取一次。
    
2./etc/bashrc

在执行完/etc/profile 内容之后，如果用户的SHELL 运行的是bash ，那么接着就会执行此文件。另外，当每次一个新的bash shell 被打开时, 该文件被读取。

    每个使用bash 的用户在登陆以后执行完/etc/profile 中内容以后都会执行此文件，在新开一个bash 的时候也会执行此文件。因此，如果你想让每个使用bash 的用户每新开一个bash 和每次登陆都执行某些操作，或者给他们定义一些新的环境变量，就可以在这个里面设置。

3.~/.bash_profile ：

每个用户都可使用该文件输入专用于自己使用的shell 信息。当用户登录时，该文件仅仅执行一次，默认情况下，它设置一些环境变量，执行用户的.bashrc 文件。

    单个用户此文件的修改只会影响到他以后的每一次登陆系统。因此，可以在这里设置单个用户的特殊的环境变量或者特殊的操作，那么它在每次登陆的时候都会去获取这些新的环境变量或者做某些特殊的操作，但是仅仅在登陆时。

4.~/.bashrc ：

该文件包含专用于单个人的bash shell 的bash 信息，当登录时以及每次打开一个新的shell 时, 该该文件被读取。

    单个用户此文件的修改会影响到他以后的每一次登陆系统和每一次新开一个bash 。因此，可以在这里设置单个用户的特殊的环境变量或者特殊的操作，那么每次它新登陆系统或者新开一个bash ，都会去获取相应的特殊的环境变量和特殊操作。

5.~/.bash_logout ：
当每次退出系统( 退出bash shell) 时, 执行该文件。

命令: env 和printenv
用于打印所有的环境 变量

set
   
     用于显示与设置当前本地 变量。单独一个set 就显示了当前环境的所有的变量，它肯定包括环境变量和一些非环境变量

unset

     用于清除变量。不管这个变量是环境变量还是本地变量，它都可以清除。

-- 下面是清除本地变量
```
[oracle@devdb1 oracle]$ set|grep myname
myname=ilonng
[oracle@devdb1 oracle]$ unset myname
[oracle@devdb1 oracle]$ set|grep myname
```
-- 下面是清除环境变量
```
[oracle@devdb1 oracle]$ env|grep myname
myname=ilonng
[oracle@devdb1 oracle]$ unset myname
[oracle@devdb1 oracle]$ env|grep myname
export
```

用于把变量变成当前shell 和其子shell 的环境变量，存活期是当前的shell 及其子shell，因此重新登陆以后，它所设定的环境变量就消失了。
当直接执行一个脚本的时候，其实是在一个子shell 环境运行的，即开启了一个子shell 来执行这个脚本，脚本执行完后该子shell 自动退出。
有没有办法在当前shell 中执行一个脚本呢？使用source 命令就可以让脚本在当前shell 中执行。如：
```
[oracle@dbamonitor NBU]$ cat test.sh    -- 查看脚本内容，显示变量内容
echo $myname
[oracle@dbamonitor NBU]$ echo $myname -- 变量存在，内容是ilonng
ilonng
[oracle@dbamonitor NBU]$ set |grep myname -- 变量是本地变量
myname=ilonng
[oracle@dbamonitor NBU]$ env |grep myname -- 变量不是环境变量
[oracle@dbamonitor NBU]$ sh test.sh -- 直接执行，新开子shell ，非环境变量的本地变量不具备继承性，在子shell 中不可见
```
如何将环境变量永久化？修改上面介绍的那几个环境变量的配置文件, then
$$source /.bashrc$$


##PROMPTING
__可以通过 $PS1 变量来设置提示符。__

参考（http://shunfengwei.blog.163.com/blog/static/17522511720122299241143/）
>命令echo \$PS1”, 将显示当前的设定。其中可用字符的含义在 man bash 的'PROMPTING'部分有说明。
如何才能完成理想的设置呢？对于健忘的初学者来讲，默认设定有些不友好，因为提示符只显示当前目录的最后一部分。如果你看到象这样的提示符
      [wsf@localhost bin]\$
能不能叫 shell 自动告诉你当前目录呢？
可以通过编辑各自home目录下的'.bash_profile'和'.bashrc'来改变设置。
在 man bash 中的'PROMPTING'部分，对这些参数(parameter)有详细说明。可以加入一些小玩意，如不同格式的当前时间，命令的历史记录号，甚至不同的颜色。
一种更适当的设定：
      PS1="[\u: \w]\\$ "
      这样，提示符就变成：
      [wsf: /usr/bin]\$
      你可以通过命令 export 来测试不同的设置（比如，export PS1="\u: \w\\$ "）。如果找到了适合的提示符，就将设置放到您的'.bashrc''中。这样，每次打开控制台或终端窗口时，都会生效。
     更详细可以参考http://blog.csdn.net/xiongmc/article/details/7801331
      

---

RefLinks
http://www.cnblogs.com/growup/archive/2011/07/02/2096142.html
http://blog.csdn.net/xifeijian/article/details/13355031

