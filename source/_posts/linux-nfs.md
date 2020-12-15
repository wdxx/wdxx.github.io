---
title: linux nfs 挂载
date: 2020-09-03 09:46:36
tags:
---
在Linux服务器上使用nfs挂载需要先安装nfs server。下面是Ubuntu上安装nfs server的命令。
```bash
sudo apt-get install nfs-kernel-server
## nfs 配置目录在/etc/exports
```
客户端需要安装`nfs-common`,安装方法如下：
```bash
sudo apt-get install nfs-common
```
<!--more-->
### nfs server配置
nfs server的配置文件在`/etc/exports`文件中，需要有`root`权限才能编辑。例如`/work/nfs_share 192.168.2.55(rw,sync,no_subtree_check)`表示允许192.168.2.55使用nfs挂载当前服务器的`/work/nfs_share`目录。下面列举下nfs配置的各个参数：
* ro 该主机对该共享目录有只读权限
* rw 该主机对该共享目录有读写权限
* root_squash 客户机用root用户访问该共享文件夹时，将root用户映射成匿名用户
* no_root_squash 客户机用root访问该共享文件夹时，不映射root用户
* all_squash 客户机上的任何用户访问该共享目录时都映射成匿名用户
* anonuid 将客户机上的用户映射成指定的本地用户ID的用户
* anongid 将客户机上的用户映射成属于指定的本地用户组ID
* sync 资料同步写入到内存与硬盘中
* async 资料会先暂存于内存中，而非直接写入硬盘
* insecure 允许从这台机器过来的非授权访问 
* subtree_check 如果共享/usr/bin之类的子目录时，强制NFS检查父目录的权限（默认）
* no_subtree_check 和上面相对，不检查父目录权限
* wdelay 如果多个用户要写入NFS目录，则归组写入（默认）
* no_wdelay 如果多个用户要写入NFS目录，则立即写入，当使用async时，无需此设置。
* hide 在NFS共享目录中不共享其子目录
* no_hide 共享NFS目录的子目录
* secure NFS通过1024以下的安全TCP/IP端口发送
* insecure NFS通过1024以上的端口发送
例子：
```
/ user01(rw) user02(rw,no_root_squash) 表示共享服务器上的根目录(/)只有user01和user02两台主机可以访问，且有读写权限；user01主机用root用户身份访问时，将客户机的root用户映射成服务器上的匿名用户(root_squash,该参数为缺省参数)，相当于在服务器使用nobody用户访问目录；user02主机用root用户身份访问该共享目录时，不映射root用户(no_root_squash),即相当于在服务器上用root身份访问该目录

　　/root/share/ 192.168.1.2(rw,insecure,sync,all_squash) 表示共享服务器上的/root/share/目录只有192.168.1.2主机可以访问，且有读写权限；此主机用任何身份访问时，将客户机的用户都映射成服务器上的匿名用户(all_squash),相当于在服务器上用nobody用户访问该目录（若客户机要在该共享目录上保存文件（即写操作），则服务器上的nobody用户对该目录必须有写的权限）

　　/home/ylw/ .test.com (rw,insecure,sync,all_squash) 表示共享/home/ylw/目录，.test.com域中所有的主机都可以访问该目录，且有读写权限

　　/home/share/ .test.com (ro,sync,all_squash,anonuid=zh3,anongid=wa4) 表示共享目录/home/share/，*.test.com域中的所有主机都可以访问，但只有只读的权限，所有用户都映射成服务器上的uid为zh3、gid为wa4的用户
```

## 参考
https://blog.csdn.net/qq_36357820/article/details/78488077
