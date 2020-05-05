#### 1. 依赖冲突

解决办法 :

打开pom文件, 选择Diagrams =>show denpendencies

Command+F搜索冲突的类所在的jar包, 右键选择Exclude



#### 2. 打包jar包到本地仓库

```shell
mvn install:install-file -Dfile=/Users/nacht/Downloads/network-management-base-protocol-1.0.0-SNAPSHOT.jar -DgroupId=cn.gzsendi.itmp-cmdb -DartifactId=network-management-base-protocol -Dversion=1.0.0-SNAPSHOT -Dpackaging=jar
```

