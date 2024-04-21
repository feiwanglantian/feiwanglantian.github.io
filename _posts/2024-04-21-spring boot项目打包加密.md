```
layout:     post
title:      spring boot项目打包加密
subtitle:   spring boot项目打包加密
date:       2024-04-21
author:     LIWC
header-img: img/post-bg-desk.jpg
catalog: true
tags:
    - spring boot
```



### 1、加密：

#### 1）将jar-project.jar和jar-project.security.properties文件放入到项目打好包的target目录下，如图所示

![img](https://cdn.nlark.com/yuque/0/2024/png/22150047/1713493509732-d914bca3-1a62-4fee-a76e-4e5e279affdd.png)

#### 2）修改jar-project.security.properties文件：

```properties
#过期时间,为空则不限制过期时间(默认到9999-01-01)
expireTime=2025-01-01
#加密密码,为空则随机生成动态密码
password=
#加密解密文件地址(加密java代码源码),为空则使用自带des加密
myEncryptCodeFile=
#加密方写入的版权信息声明,为空则无
myVersionInfo=请正规渠道获得版本授权文件,严禁进行反编译修改或破解,一经发现会追溯法律责任！
```

#### 3）然后使用如下命令，对已经编译好的jar包进行加密：

```java
java -jar jar-project.jar --fromJar "D:\项目名称\target\service_zhuyuan-1.0-SNAPSHOT.jar" --excludeClass "*ServiceZhuyuanApplication*" --includeJar "service_zhuyuan-*" --includeConfig "*.properties"
```

#### 4）当输出如下提示下，说明jar包已经加密成功了！

![img](https://cdn.nlark.com/yuque/0/2024/png/22150047/1713494205850-aec1d0cc-de21-4a69-9756-82394e848d4a.png)

![img](https://cdn.nlark.com/yuque/0/2024/png/22150047/1713494217425-35bb6be6-0f8e-4b6d-b4f1-a864ab082962.png)

#### 5）这是加密后的jar包和反编译后的代码

![img](https://cdn.nlark.com/yuque/0/2024/png/22150047/1713494373172-5e8df8b7-5c38-4d88-9eee-1cbfd03c0aff.png)

#### 6）执行java -jar命令，会发现报如下的错误，无法执行：

![img](https://cdn.nlark.com/yuque/0/2024/png/22150047/1713494498522-ad6459d0-3037-4600-a145-71e0bda961ea.png)

#### 所以得使用以下步骤进行运行解密

### 2、解密：

#### 1）将加密后的jar和配置文件一同上传到服务器上，请注意，一定要在用一个文件夹下，如图：

![img](https://cdn.nlark.com/yuque/0/2024/png/22150047/1713506079819-df670de9-77f5-4de7-a626-921044343b15.png)

#### 2）在部署文件夹下，使用如下命令，运行加密的jar包（必须使用此命令，否则启动报错！）

```powershell
java -javaagent:encrypt-service_gateway-1.0-SNAPSHOT.jar -jar encrypt-service_gateway-1.0-SNAPSHOT.jar
```

#### 3）发现可以正常运行了：

![img](https://cdn.nlark.com/yuque/0/2024/png/22150047/1713494785916-308d8fbd-0cea-4c58-bdfc-80c8a6bfefbe.png)

#### 至此，jar包已经解密了，可以正常使用了