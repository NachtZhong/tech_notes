### 1. 下载jdk

```shell
cd /usr/local/lib
wget https://download.java.net/java/GA/jdk14/076bab302c7b4508975440c56f6cc26a/36/GPL/openjdk-14_linux-x64_bin.tar.gz
tar -zxvf openjdk-14_linux-x64_bin.tar.gz
```



### 2. 添加环境变量

```shell
vi /etc/profile
#尾部添加以下内容
export JAVA_HOME=/usr/local/lib/jdk-14
export JRE_HOME=/${JAVA_HOME}
export CLASSPATH=.:${JAVA_HOME}/libss:${JRE_HOME}/lib
export PATH=${JAVA_HOME}/bin:$PATH
```



### 3. 测试是否安装成功

```shell
java -version
#openjdk version "14" 2020-03-17
#OpenJDK Runtime Environment (build 14+36-1461)
#OpenJDK 64-Bit Server VM (build 14+36-1461, mixed mode, sharing)
```

