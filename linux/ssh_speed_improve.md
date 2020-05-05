# ssh连接慢优化措施

编辑配置文件

```
vim /etc/ssh/sshd_config
```

将UseDNS注释取消, 设置为no

重启ssh服务

```
service sshd restart
```

