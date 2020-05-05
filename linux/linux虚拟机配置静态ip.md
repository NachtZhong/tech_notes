### 1. 查看宿主机vmnet8配置(MAC)

```shell
cat /Library/Preferences/VMware\ Fusion/vmnet8 && nat.conf
# VMware NAT configuration file
# Manual editing of this file is not recommended. Using UI is preferred.

[host]

# NAT gateway address
#这个是虚拟机中配置的网关
ip = 192.168.55.2
netmask = 255.255.255.0

# VMnet device if not specified on command line
device = vmnet8

# Allow PORT/EPRT FTP commands (they need incoming TCP stream ...)
activeFTP = 1
```

### 2. 配置虚拟机网卡

```shell
vi /etc/sysconfig/network-scripts/ifcfg-ens33
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
#协议改为static
BOOTPROTO=static
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=ens33
UUID=91b5ea8c-c165-40b3-8a37-9b2c00db1ffb
DEVICE=ens33
ONBOOT=yes
#你需要的静态ip地址
IPADDR=192.168.55.160
#默认网关(GATEWAY不要写错成GATEAWAY)
GATEWAY=192.168.55.2
#子网掩码
NETMASK=255.255.255.0
```

### 3. 重启网卡

```shell
systemctl restart network
```





### 4.配置DNS

#### 修改NetworkManager.conf 配置文件

在[main]中添加

dns=no



#### 修改resolv.conf配置文件

```shell
vim /etc/resolv.conf
```

添加

```shell
#主DNS服务器
nameserver 218.85.157.99
 
#备DNS服务器
nameserver 114.114.114.114
```



#### 重启NetworkManager

```shell
systemctl restart NetworkManager
```

