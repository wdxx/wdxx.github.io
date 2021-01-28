---
title: keepalived 安装与使用
date: 2021-01-12 10:35:15
tags: [Linux,keepalived]
categories: [linux]
---
&emsp;&emsp;Keepalived 是一个基于VRRP协议来实现的LVS服务高可用方案，可以利用其来避免单点故障。一个LVS服务会有2台服务器运行Keepalived，一台为主服务器（MASTER），一台为备份服务器（BACKUP），但是对外表现为一个虚拟IP（VIP），主服务器会发送特定的消息给备份服务器，当备份服务器收不到这个消息的时候，即主服务器宕机的时候，备份服务器就会接管虚拟IP，继续提供服务，从而保证了高可用性。说白了就是如果客户端在访问服务端时，如果服务端出了问题，可以及时自动的切换到另外一个服务器，而客户端感知不到，客户端通过VIP来访问服务端。
<!-- more -->
### keepalived 工作原理
下图是keepalived的工作原理图：
![keepalived工作原理](/images/keepalived-vrrp-network.png)。
```
1. LB1服务器：192.168.10.111 master
2. LB2服务器：192.168.10.112 backup
3. VIP: 192.168.10.121
```
1. keepalived在LB1与LB2上安装，一个为master一个为backup。图中LB1为master，所以刚开始的时候LB1服务器虚拟出VIP 192.168.10.121，当客户端访问192.168.10.121的时候实际访问的就是LB1服务器。
2. keepavlied是基于VRRP协议的，关于什么是VRRP协议可以自行[搜索](https://www.jianshu.com/p/7410507d57c3)。根据VRRP协议可以知道，由于当前master为LB1，所以LB1会做如下工作：
- 定时发送VRRP通告报文(时间间隔为Advertisement_Interval)
- 使用虚拟MAC地址响应VIP的ARP请求
- 转发目的MAC地址为虚拟MAC地址的IP报文
- 抢占模式下，如果收到比自己优先级大的VRRP报文，或者跟自己优先级相等，且本地接口IP地址小于源端接口IP地址时，则转变为Backup状态
- 收到Shutdown（关闭）消息后，则立即转变为Initialize状态
3. LB2当前为backup,backup机器会接受master发送的VRRP通告报文，判断master是否正常。如果一定时间间隔没有收到VRRP通告报文，即Master_Down_Interval（Master_Down_Interval = 3* Advertisement_Interval + Skew_time）超时，则判断为Master故障。backup的工作如下：
- 接收Master发送的VRRP通告报文，判断Master是否正常
- 对虚拟IP的ARP请求不做响应
- 丢弃目的MAC地址为虚拟路由器MAC地址的IP报文
- 丢弃目的IP地址为VIP地址的IP报文
- 如果收到优先级比自己高，或与自己相等的VRRP报文，则重置Master_Down_Interval定时器（不进一步比较IP地址）
- 如果收到优先级比自己小的VPPR报文，且优先级为0时，（表示原Master设备声明不参与该VRRP组了），定时器时间设置为Skew_time（偏移时间，Skew_time= (256 - priority)/256）
- 如果收到优先级比自己小的VPPR报文，且优先级不为0时，丢弃该报文，立即转变为Master状态
- Master_Down_Interval定时器超时，立即转变为Master状态
- 收到Shutdown（关闭）消息后，则立即转变为Initialize状态
4. 当LB1上的web1服务出现问题时，此时可以降低LB1的优先级，或者停止LB1上的keepalived进程，从而将master转移到LB2上。此时客户端访问192.168.10.121即访问的LB2服务器。

### keepalived 安装
测试环境为`CentOS Linux release 7.4.1708`,下面时安装命令。
```shell
yum install -y keepalived
```
配置文件路径为:/etc/keepalived/keepalived.conf
默认log就是syslog，/var/log/messages

### keepalived配置
1. master配置
```
! Configuration File for keepalived

global_defs {
    router_id keep_231
    script_user root
}

vrrp_script check_sample_alive {
    script "/check_sample.sh"
    interval 2
    weight -10
}

vrrp_instance VI_1 {
    state MASTER
    interface eth0
    virtual_router_id 51
    priority 101
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }

    track_script {
        check_sample_alive
    }

    # 单播模式
    unicast_src_ip 192.168.10.111
    unicast_peer {
        192.168.10.112
    }
    virtual_ipaddress {
        192.168.10.121
    }
}
```
关于配置文件的配置可以自行搜索，讲一下几个关键的。`script_user`表示执行`check_sample_alive`的用户。同时也可以在`global_defs`section定义`enable_script_security`来增加安全性，但是需要你的脚本所在位置所属为root的用户与用户组。也即是要`chown root:root /check_sample.sh`
下面时`check_sample.sh`脚本的内容：
```bash
active=`ps -ef | grep "python sample.py" | grep -v "grep" | wc -l`
if [ $active -eq 0 ];then
  sleep 3
  if [ `ps -ef | grep "python sample.py" | grep -v "grep" | wc -l` -eq 0 ];then
    echo "sample.py died"
    exit 1
  fi
fi
echo "sample.py is running..."
```
2. backup配置
```
! Configuration File for keepalived

global_defs {
    router_id keep_232
    script_user root
}

vrrp_script check_sample_alive {
    script "/check_sample.sh"
    interval 2
    weight 10
}

vrrp_instance VI_1 {
    state BACKUP
    interface eth0
    virtual_router_id 51
    priority 90
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }

    track_script {
        check_sample_alive
    }

    unicast_src_ip 192.168.10.112
    unicast_peer {
        192.168.10.111
    }
    virtual_ipaddress {
        192.168.10.121
    }
}
```

### 测试web服务器脚本
因为要测试master与backup是否能够正常切换，所以使用了python的wsgi脚本来简单的测试配置是否正确。下面是web服务器代码(sample.py)，它通过wsgi接口简单的返回当前主机的ip。
```python
#!/usr/bin/env python
from wsgiref.simple_server import make_server
import socket

def simple_app(environ, start_response):
    #setup_testing_defaults(environ)

    status = '200 OK'
    headers = [('Content-type', 'text/plain')]

    start_response(status, headers)

    ip = socket.gethostbyname(socket.gethostname())

    return ip

httpd = make_server('', 7000, simple_app)

httpd.serve_forever()
```

### 测试
先在LB1与LB2服务器上启动keepalived。
```bash
service keepalived start
```
然后在LB1与LB2服务器上以后台方式启动sample.py。
```bash
python sample.py &
```
通过在浏览器中访问: `http://192.168.10.121:7000`可以看到浏览器会返回当前master的IP:`192.168.10.111`。然后将192.168.10.111上的sample.py进程杀掉，再访问`http://192.168.10.121:7000`可以看到浏览器会返回当前master的IP:`192.168.10.112`。可见切换成功。

参考：
1. 入门：https://zhuanlan.zhihu.com/p/143295216
2. 入门：https://zhuanlan.zhihu.com/p/43971218
3. VRRP原理：https://www.jianshu.com/p/214f89298646
4. 优先级设置：https://www.cnblogs.com/arjenlee/p/9258188.html