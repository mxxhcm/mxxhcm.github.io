---
title: shadowsocks服务端以及客户端配置
date: 2019-03-04 13:03:57
tags: 
 - shadowsocks
 - linux
 - 工具
categories: linux
mathjax: false
---

## 服务器端配置
首先需要有一个VPS账号，vultr,digitalocean,搬瓦工等等都行。
### 启用BBR加速
~#:apt update
~#:apt upgrade
~#:echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf
~#:echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf 
~#:sysctl -p
上述命令就完成了BBR加速，执行以下命令验证：
~#:lsmod |grep bbr
看到输出包含tcp_bbr就说明已经成功了。

### 搭建shadowsocks server
#### 安装shadowsocks server
~#:apt install python-pip
~#:pip install shadowsocks
需要说一下的是，shadowsocks目前还不支持python3.5及以上版本，上次我把/usr/bin/python指向了python3.6，就是系统默认的python指向了python3.6，然后就gg了。一定要使用Python 2.6,2.7,3.3,3.4中的一个版本才能使用。。

#### 创建shadowsocks配置文件
如果你的VPS支持ipv6的话，那么可以开多进程分别运行ipv4和ipv6的shadowsocks server。本地只有ipv4的话，可以用本地ipv4访问ipv6，从而访问byr等网站，但是六维空间对此做了屏蔽。如果本地有ipv6的话，还可以用本地的ipv6访问ipv6实现校园网不走ipv4流量。
##### ipv4配置
~#:vim /etc/shadowsocks_v4.json
配置文件如下
{
"server":"0.0.0.0",
"server_port":8889,
"local_address":"127.0.0.1",
"local_port":1080,
"password":"你的密码",
"timeout":600,
"method":"aes-256-cfb"
}

##### ipv6配置
~#:vim /etc/shadowsocks_v6.json
配置文件如下
{
"server":"::",
"server_port":8888,
"local_address":"127.0.0.1",
"local_port":1080,
"password":"你的密码",
"timeout":600,
"method":"aes-256-cfb"
}
注意这两个文件的server_port一定要不同，以及双引号必须是英文引号。
##### 1.2.2.3.手动运行shadowsocks server
~#:ssserver -c /etc/shadowsock_v4.json -d start --pid-file ss1.pid
~#:ssserver -c /etc/shadowsock_v6.json -d start --pid-file ss2.pid
注意这里要给两条命令分配不同的进程号。

### 设置shadowsocks server开机自启
如果重启服务器的话，就需要重新手动执行上述命令，这里我们可以把它写成开机自启脚本。
~#:vim /etc/init.d/shadowsocks_v4
内容如下：
``` shell
#!/bin/sh
### BEGIN INIT INFO
# Provides:          apache2
# Required-Start:    $local_fs $remote_fs $network $syslog
# Required-Stop:     $local_fs $remote_fs $network $syslog
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: apache2 service
# Description:       apache2 service daemon
### END INIT INFO
start(){
  ssserver -c /etc/shadowsocks_v4.json -d start --pid-file ss2.pid
}
stop(){
  ssserver -c /etc/shadowsocks_v4.json -d stop --pid-file ss2.pid
}
case "$1" in
start)
  start
  ;;
stop)
  stop
  ;;
restart)
  stop
  start
  ;;
*)
  echo "Uasage: $0 {start|reload|stop}$"
  exit 1
  ;;
esac
```
                           
~#:vim /etc/init.d/shadowsocks_v6
内容如下：
``` shell
#!/bin/sh
### BEGIN INIT INFO
# Provides:          apache2
# Required-Start:    $local_fs $remote_fs $network $syslog
# Required-Stop:     $local_fs $remote_fs $network $syslog
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: apache2 service
# Description:       apache2 service daemon
### END INIT INFO
start(){
  ssserver -c /etc/shadowsocks_v6.json -d start --pid-file ss1.pid
}
stop(){
  ssserver -c /etc/shadowsocks_v6.json -d stop --pid-file ss1.pid

}
case "$1" in
start)
  start
  ;;
stop)
  stop
  ;;
restart)
  stop
  start
  ;;
*)
  echo "Uasage: $0 {start|reload|stop}$"
  exit 1
  ;;
esac
```
   
然后执行下列命令即可：
~#:chmod a+x /etc/init.d/shadowsocks_v4
~#:chmod a+x /etc/init.d/shadowsocks_v6
~#:update-rc.d shadowsocks_v4 defaults
~#:update-rc.d shadowsocks_v6 defaults

至此，服务器端配置完成。

## 客户端配置
### Windows客户端配置
#### 安装shadowsock客户端
到该网址 https://github.com/shadowsocks/shadowsocks-windows/releases 下载相应的windows客户端程序。
然后配置服务器即可～

### Linux客户端配置
#### 安装shadowsocks程序
~$:sudo pip install shadowsocks

#### 运行shadowsocks客户端程序

~$:sudo vim /etc/shadowsocks.json
填入以下配置文件
{
"server":"填上自己的shadowsocks server ip地址",
"server_port":"8888",//填上自己的shadowsocks server 端口"
"local_port":1080,
"password":"mxxhcm150929",
"timeout":600,
"method":"aes-256-cfb"
}

接下来可以执行以下命令运行shadowsocks客户端：
~$:sudo sslocal -c /etc/shadowsocks.json
然后报错：
> INFO: loading config from /etc/shadowsocks.json
2019-03-04 14:37:49 INFO     loading libcrypto from libcrypto.so.1.1
Traceback (most recent call last):
  File "/usr/local/bin/sslocal", line 11, in <module>
    load_entry_point('shadowsocks==2.8.2', 'console_scripts', 'sslocal')()
  File "/usr/local/lib/python2.7/dist-packages/shadowsocks/local.py", line 39, in main
    config = shell.get_config(True)
  File "/usr/local/lib/python2.7/dist-packages/shadowsocks/shell.py", line 262, in get_config
    check_config(config, is_local)
  File "/usr/local/lib/python2.7/dist-packages/shadowsocks/shell.py", line 124, in check_config
    encrypt.try_cipher(config['password'], config['method'])
  File "/usr/local/lib/python2.7/dist-packages/shadowsocks/encrypt.py", line 44, in try_cipher
    Encryptor(key, method)
  File "/usr/local/lib/python2.7/dist-packages/shadowsocks/encrypt.py", line 83, in __init__
    random_string(self._method_info[1]))
  File "/usr/local/lib/python2.7/dist-packages/shadowsocks/encrypt.py", line 109, in get_cipher
    return m[2](method, key, iv, op)
  File "/usr/local/lib/python2.7/dist-packages/shadowsocks/crypto/openssl.py", line 76, in __init__
    load_openssl()
  File "/usr/local/lib/python2.7/dist-packages/shadowsocks/crypto/openssl.py", line 52, in load_openssl
    libcrypto.EVP_CIPHER_CTX_cleanup.argtypes = (c_void_p,)
  File "/usr/lib/python2.7/ctypes/__init__.py", line 379, in __getattr__
    func = self.__getitem__(name)
  File "/usr/lib/python2.7/ctypes/__init__.py", line 384, in __getitem__
    func = self._FuncPtr((name_or_ordinal, self))
AttributeError: /usr/lib/x86_64-linux-gnu/libcrypto.so.1.1: undefined symbol: EVP_CIPHER_CTX_cleanup

按照参考文献4的做法，是在openssl 1.1.0版本中放弃了EVP_CIPHER_CTX_cleanup函数
> EVP_CIPHER_CTX was made opaque in OpenSSL 1.1.0. As a result, EVP_CIPHER_CTX_reset() appeared and EVP_CIPHER_CTX_cleanup() disappeared. 
EVP_CIPHER_CTX_init() remains as an alias for EVP_CIPHER_CTX_reset().

将openssl库中的EVP_CIPHER_CTX_cleanup改为EVP_CIPHER_CTX_reset即可。
再次执行以下命令，查看shadowsocks安装位置
~#:pip install shadowsocks
Requirement already satisfied: shadowsocks in /usr/local/lib/python2.7/dist-packages
~#:cd /usr/local/lib/python2.7/dist-packages/shadowsocks
~#:vim crypto/openssl.py
搜索cleanup，将其替换为reset
具体位置在第52行libcrypto.EVP_CIPHER_CTX_cleanup.argtypes = (c_void_p,)和第111行libcrypto.EVP_CIPHER_CTX_cleanup(self._ctx) 

#### 手动运行后台挂起
将所有的log重定向到~/.log/sslocal.log文件中
~$:mkdir ~/.log
~$:touch ~/.log/ss-local.log
~$:nohup sslocal -c /etc/shadowsocks_v6.json \</dev/null &\>>~/.log/ss-local.log &

#### 开机自启shadowsocks client
但是这样子的话，每次开机都要重新运行上述命令，太麻烦了。可以写个开机自启脚本。执行以下命令：
~$:sudo vim /etc/init.d/shadowsocks
内容为以下shell脚本

``` shell
#!/bin/sh

### BEGIN INIT INFO
# Provides:          shadowsocks local
# Required-Start:    $local_fs $remote_fs $network $syslog
# Required-Stop:     $local_fs $remote_fs $network $syslog
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: shadowsocks service
# Description:       shadowsocks service daemon
### END INIT INFO
start(){
　　  sslocal -c /etc/shadowsocks.json -d start
}
stop(){
　　  sslocal -c /etc/shadowsocks.json -d stop
}
case “$1” in
start)
　　　start
　　　;;
stop)
　　　stop
　　　;;
reload)
　　　stop
　　　start
　　　;;
\*)
　　　echo “Usage: $0 {start|reload|stop}”
　　　exit 1
　　　;;
esac
```
然后执行以下命令即可：
~$:sudo chomod a+x /etc/init.d/shadowsocks
~$:sudo update_rc.d shadowsocks defaults
上述命令执行完成以后，进行测试
~$:sudo service shadosowcks start


#### 配置代理
上一步的目的是建立了shadowsocks服务的本地客户端，socks5流量会走该通道，但是浏览器的网页的流量是https的，我们需要配置相应的代理，将https流量转换为socks5流量，走ss客户端到达ss服务端。当然，也可以把其他各种流量，如tcp,udp等各种流量都转换为socks5流量，这个可以通过全局代理实现，也可以通过添加特定的代理规则实现。
##### 配置全局代理
如下图所示，添加ubuntu socks5系统代理：

然后就可以成功上网了。

##### 使用SwitchyOmega配置chrome代理
首先到 https://github.com/FelisCatus/SwitchyOmega/releases 下载SyitchyOmega.crx。然后在chrome的地址栏输入chrome://extensions，将刚才下载的插件拖进去。
然后在浏览器右上角就有了这个插件，接下来配置插件。如下图：
![mxx](https:)
直接配置proxy，添加如图所示的规则，这样chrome打开的所有网站都是走代理的。

Z#### 使用privoxy让terminal走socks5
~$:sudo apt install privoxy
~$:sudo vim /etc/privoxy/config
取消下列行的注释，或者添加相应条目
forward-socks5 / 127.0.0.1:1080 . # SOCKS5代理地址
listen-address 127.0.0.1:8118     # HTTP代理地址
forward 10.\*.\*.\*/ .               # 内网地址不走代理
forward .abc.com/ .             # 指定域名不走代理
重启privoxy服务
~$:sudo service privoxy restart
在bashrc中添加如下环境变量
export http_proxy="http://127.0.0.1:8118"
export https_proxy="http://127.0.0.1:8118"

~$:source ~/.bashrc
~$:curl.gs

## 参考文献
1. http://godjose.com/2017/06/14/new-article/
2. https://www.polarxiong.com/archives/搭建ipv6-VPN-让ipv4上ipv6-下载速度提升到100M.html
3. https://blog.csdn.net/li1914309758/article/details/86510127
4. https://blog.csdn.net/blackfrog_unique/article/details/60320737
5. https://blog.csdn.net/qq_31851531/article/details/78410146
