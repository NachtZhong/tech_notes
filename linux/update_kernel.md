# 升级centos内核

Centos 7最小化安装的环境下进行

检查已安装的内核版本 
```uname -sr```

CentOS下使用 ELRepo第三方的仓库，可以将内核升级到最新版本噢。

ELRepo 仓库官方网站：http://elrepo.org/tiki/

导入公钥后安装ELRepo的rpm就好了

```rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org```
`rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-3.el7.elrepo.noarch.rpm
`
ELRepo官网说，可以用fastestmirror的插件，让yum在更新时先根据ping值进行判断，然后从最快响应的地址下载。 
`sudo yum -y install install yum-plugin-fastestmirror`

仓库启用后，你可以使用下面的命令列出可用的内核相关包： 
`yum --disablerepo="*" --enablerepo="elrepo-kernel" list available `

接下来，安装最新的主线稳定内核 
`yum -y --enablerepo=elrepo-kernel install kernel-ml`

为了让新安装的内核成为默认启动选项，你需要如下修改 GRUB 配置： 
打开并编辑 

`/etc/default/grub `
注释掉原来的

`GRUB_DEFAULT=saved `
添加一行

`GRUB_DEFAULT=0 `
意思是 GRUB 初始化页面的第一个内核将作为默认内核。

接下来运行下面的命令来重新创建内核配置。 
`grub2-mkconfig -o /boot/grub2/grub.cfg`	

最后，重启机器并应用最新内核，接着运行下面的命令检查最新内核版本 
`uname -sr `

删除旧的内核
先查询一下系统已安装的内核 
`rpm -qa | grep kernel `

`sudo yum remove -y 旧内核的名字就好了`

