# 运行docker时碰到的因为系统内核导致容器无法启动的问题

因为linux系统内核过低, 运行docker时无法启动,提示 OCI runtime create failed: container_linux.go:344
解决办法:

1. 升级linux系统内核(参见update_kernel)
2. 安装最新版本docker(参见docker_install)

