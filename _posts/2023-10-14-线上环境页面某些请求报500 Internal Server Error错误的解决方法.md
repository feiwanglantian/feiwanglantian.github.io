#### 服务器后端项目日志报500 Internal Server Error错误，但是日志里面没有报错信息的解决方法

##### 接客户反馈，公司一个正在运行的项目中某些页面会有错误，但是没有错误信息，通过查看network发现，nginx报500 Internal Server Error错误，查看nginx日志发现，日志中含有大量的 socket() failed (24: Too many open files) while connecting to upstream错误信息，如下图所示：

![3](https://liwc-1309754561.cos.ap-beijing.myqcloud.com/3.png)

通过查询linux下文件句柄可知：

```shell
ulimit -n
1024
```

其系统的文件句柄为1024，过于小，所以通过一下方式修改

```shell
vi /etc/security/limits.conf
在文件末增加
# 配置nginx用户文件限制
nginx soft nofile 65535
nginx hard nofile 65535
#
# # 配置所有用户文件限制
soft nofile 65535
hard nofile 65535

同时vi /etc/sysctl.conf末尾添加
fs.file-max=65535

```

重新启动，在使用ulimit -n查看的数已经是65535，但由于需要重启服务器，故暂时在终端使用如下命令解决

```shell
ulimit -n 65535
```

需要同时在nginx的配置文件中配置

```shell
vi /etc/nginx/nginx.conf
worker_rlimit_nofile 65535;
events {
worker_connections 65535;
} 
```

即可解决此问题