---

layout: post
title: nfs实现linux系统间文件共享
category: codings
tags: [Linux, Linux-basics, Linux-filesystem]
excerpt_separator: "<!--more-->"

---


NFS服务器的安装过程、配置文件和常用命令行工具，以及NFS客户端上如何安装常用工具，介绍如何挂载共享目录。

<!--more-->

# Quick Configuration

## 1. Server (the data storage machine)
 1. 安装NFS服务：
 `sudo apt install nfs-kernel-server `, apt自动安装nfs-common、rpcbind等13个软件包

 2. 编写配置文件：
    ```
    sudo vi /etc/exports ###编辑/etc/exports 文件
    #/etc/exports文件的内容如下：
    /home/whh/backup *(rw,sync,no_subtree_check,no_root_squash) 
    ```
 3. 创建共享目录
 `sudo mkdir /home/whh/backup`, 在服务器端创建/home/whh/backup共享目录

 4. 重启nfs服务：`sudo service nfs-kernel-server restart`
 5. 常用命令工具
  - 在安装NFS服务器时，已包含常用的命令行工具，无需额外安装。
  - 显示已经mount到本机nfs目录的客户端机器: `sudo showmount -e localhost ` or `showmount -e 192.168.1.52`
  - 将配置文件中的目录全部重新export一次！无需重启服务。`sudo exportfs -rv`
  - 查看NFS的运行状态sudo nfsstat 
  - 查看rpc执行信息，可以用于检测rpc运行情况sudo rpcinfo 
  - 查看网络端口，NFS默认是使用111端口。sudo netstat -tu -4  

## 2. Local (query machine)

showmount -e 192.168.1.52
 1. 安装客户端工具: `sudo apt install nfs-common `
 在需要连接到NFS服务器的客户端机器上，需要执行以下命令，安装nfs-common软件包。apt会自动安装nfs-common、rpcbind等12个软件包.
 2. 查看NFS服务器上的共享目录: `sudo showmount -e 192.168.3.167 ` #显示指定的（192.168.3.167）NFS服务器上export出来的目录
 3. 创建本地挂载目录: `sudo mkdir -p /mnt/backup`
 4. 挂载共享目录: `sudo mount -t nfs 192.168.3.167:/data /mnt/data`
 将NFS服务器192.168.3.167上的目录，挂载到本地的/mnt/目录下

>注：在没有安装nfs-common或者nfs-kernel-server软件包的机器上，
  - 直接执行showmount、exportfs、nfsstat、rpcinfo等命令时，
  - 系统会给出友好的提示，
  - 比如直接showmount会提示需要执行sudo apt install nfs-common命令，
  - 比如直接rpcinfo会提示需要执行sudo apt install rpcbind命令。


# Basic Knowledge

## NFS服务简介

　　NFS 是Network File System的缩写，即网络文件系统。一种使用于分散式文件系统的协定，由Sun公司开发，于1984年向外公布。功能是通过网络让不同的机器、不同的操作系统能够彼此分享个别的数据，让应用程序在客户端通过网络访问位于服务器磁盘中的数据，是在类Unix系统间实现磁盘文件共享的一种方法。

　　NFS 的基本原则是“容许不同的客户端及服务端通过一组RPC分享相同的文件系统”，它是独立于操作系统，容许不同硬件及操作系统的系统共同进行文件的分享。

　　NFS在文件传送或信息传送过程中依赖于RPC协议。RPC，远程过程调用 (Remote Procedure Call) 是能使客户端执行其他系统中程序的一种机制。NFS本身是没有提供信息传输的协议和功能的，但NFS却能让我们通过网络进行资料的分享，这是因为NFS使用了一些其它的传输协议。而这些传输协议用到这个RPC功能的。可以说NFS本身就是使用RPC的一个程序。或者说NFS也是一个RPC SERVER。所以只要用到NFS的地方都要启动RPC服务，不论是NFS SERVER或者NFS CLIENT。这样SERVER和CLIENT才能通过RPC来实现PROGRAM PORT的对应。可以这么理解RPC和NFS的关系：NFS是一个文件系统，而RPC是负责负责信息的传输。


## NFS系统守护进程

nfsd：它是基本的NFS守护进程，主要功能是管理客户端是否能够登录服务器；
mountd：它是RPC安装守护进程，主要功能是管理NFS的文件系统。当客户端顺利通过nfsd登录NFS服务器后，在使用NFS服务所提供的文件前，还必须通过文件使用权限的验证。它会读取NFS的配置文件/etc/exports来对比客户端权限。
portmap：主要功能是进行端口映射工作。当客户端尝试连接并使用RPC服务器提供的服务（如NFS服务）时，portmap会将所管理的与服务对应的端口提供给客户端，从而使客户可以通过该端口向服务器请求服务。

## NFS服务器的配置

NFS服务器的配置相对比较简单，只需要在相应的配置文件中进行设置，然后启动NFS服务器即可。

NFS的常用目录
```
/etc/exports            ## NFS服务的主要配置文件
/usr/sbin/exportfs          ## NFS服务的管理命令
/usr/sbin/showmount         ## 客户端的查看命令
/var/lib/nfs/etab           ## 记录NFS分享出来的目录的完整权限设定值
/var/lib/nfs/xtab           ## 记录曾经登录过的客户端信息
```


NFS服务的配置文件为 /etc/exports，这个文件是NFS的主要配置文件，不过系统并没有默认值，所以这个文件不一定会存在，可能要使用vim手动建立，然后在文件里面写入配置内容。

/etc/exports文件内容格式：

`<输出目录> [客户端1 选项（访问权限,用户映射,其他）] [客户端2 选项（访问权限,用户映射,其他）]`


### a. 输出目录：
输出目录是指NFS系统中需要共享给客户机使用的目录；
### b. 客户端：
客户端是指网络中可以访问这个NFS输出目录的计算机
客户端常用的指定方式
指定ip地址的主机：192.168.0.200
指定子网中的所有主机：192.168.0.0/24 192.168.0.0/255.255.255.0
指定域名的主机：david.bsmart.cn
指定域中的所有主机：*.bsmart.cn
所有主机：*

### c. 选项：
选项用来设置输出目录的访问权限、用户映射等。
NFS主要有3类选项：

1. 访问权限选项
 - 设置输出目录只读：ro
 - 设置输出目录读写：rw

2. 用户映射选项
 - all_squash：将远程访问的所有普通用户及所属组都映射为匿名用户或用户组（nfsnobody）；
 - no_all_squash：与all_squash取反（默认设置）；
 - root_squash：将root用户及所属组都映射为匿名用户或用户组（默认设置）；
 - no_root_squash：与rootsquash取反；
 - anonuid=xxx：将远程访问的所有用户都映射为匿名用户，并指定该用户为本地用户（UID=xxx）；
 - anongid=xxx：将远程访问的所有用户组都映射为匿名用户组账户，并指定该匿名用户组账户为本地用户组账户（GID=xxx）；

3. 其它选项
 - secure：限制客户端只能从小于1024的tcp/ip端口连接nfs服务器（默认设置）；
 - insecure：允许客户端从大于1024的tcp/ip端口连接服务器；
 - sync：将数据同步写入内存缓冲区与磁盘中，效率低，但可以保证数据的一致性；
 - async：将数据先保存在内存缓冲区中，必要时才写入磁盘；
 - wdelay：检查是否有相关的写操作，如果有则将这些写操作一起执行，这样可以提高效率（默认设置）；
 - no_wdelay：若有写操作则立即执行，应与sync配合使用；
 - subtree：若输出目录是一个子目录，则nfs服务器将检查其父目录的权限(默认设置)；
 - no_subtree：即使输出目录是一个子目录，nfs服务器也不检查其父目录的权限，这样可以提高效率；

## NFS服务器的启动与停止

在对exports文件进行了正确的配置后，就可以启动NFS服务器了。

```
1、启动NFS服务器
为了使NFS服务器能正常工作，需要启动portmap和nfs两个服务，并且portmap一定要先于nfs启动。
# service portmap start


2、查询NFS服务器状态

# service portmap status
# service nfs status  

3、停止NFS服务器

要停止NFS运行时，需要先停止nfs服务再停止portmap服务，对于系统中有其他服务(如NIS)需要使用时，不需要停止portmap服务

# service nfs stop
# service portmap stop
```


4、设置NFS服务器的自动启动状态

对于实际的应用系统，每次启动LINUX系统后都手工启动nfs服务器是不现实的，需要设置系统在指定的运行级别自动启动portmap和nfs服务。
```
chkconfig --list portmap
chkconfig --list nfs

```
![](https://images0.cnblogs.com/blog/370046/201301/03140300-180710ca746b49eb914e8ace20add0e9.jpg)

设置portmap和nfs服务在系统运行级别3和5自动启动。
```
chkconfig --level 35 portmap on
chkconfig --level 35 nfs on
```

![](https://images0.cnblogs.com/blog/370046/201301/03140345-54a300b91d8f41f2828273b18a4e710f.jpg)



## 启动自动挂载nfs文件系统: `vi /etc/fstab`

格式：
<server>:</remote/export> </local/directory> nfs < options> 0 0


## 相关命令

1. exportfs

如果我们在启动了NFS之后又修改了/etc/exports，是不是还要重新启动nfs呢？这个时候我们就可以用exportfs 命令来使改动立刻生效，该命令格式如下：
```
　　# exportfs [-aruv]

　　-a 全部挂载或卸载 /etc/exports中的内容 
　　-r 重新读取/etc/exports 中的信息 ，并同步更新/etc/exports、/var/lib/nfs/xtab
　　-u 卸载单一目录（和-a一起使用为卸载所有/etc/exports文件中的目录）
　　-v 在export的时候，将详细的信息输出到屏幕上。

具体例子： 
　　# exportfs -au 卸载所有共享目录
　　# exportfs -rv 重新共享所有目录并输出详细信息
````

2. nfsstat

查看NFS的运行状态，对于调整NFS的运行有很大帮助。

3. rpcinfo
查看rpc执行信息，可以用于检测rpc运行情况的工具，利用rpcinfo -p 可以查看出RPC开启的端口所提供的程序有哪些。

4. showmount
```
　　-a 显示已经于客户端连接上的目录信息
　　-e IP或者hostname 显示此IP地址分享出来的目录
```

5. netstat

可以查看出nfs服务开启的端口，其中nfs 开启的是2049，portmap 开启的是111，其余则是rpc开启的。

最后注意两点，虽然通过权限设置可以让普通用户访问，但是挂载的时候默认情况下只有root可以去挂载，普通用户可以执行sudo。

NFS server 关机的时候一点要确保NFS服务关闭，没有客户端处于连接状态！通过showmount -a 可以查看，如果有的话用kill killall pkill 来结束，（-9 强制结束）



# 参考
1. [Ubuntu 16.04系统上NFS的安装与使用](https://blog.csdn.net/CSDN_duomaomao/article/details/77822883)
2. [Linux NFS服务器的安装与配置](http://www.cnblogs.com/mchina/archive/2013/01/03/2840040.html)
