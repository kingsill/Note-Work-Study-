查看mysql服务状态

```shell
service mysql status
```

查看系统进行

```shell
ps aux #查看所有进程
ps -ef | grep xxx #查看xxx相关进程
```

```shell
netstat -aon | findstr :8080
```

查看现在使用的端口

```shell
netstat -ntlp
```

go env -w GOOS=linux
