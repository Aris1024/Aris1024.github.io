# shell指令备忘

### 1. 查看目录占用空间 (最大1层目录)

```shell
# 查看目录占用空间 (最大1层目录)
du -h --max-depth=1
```

> #输出:
> [bhjr@VM_15_11_centos app]$ du -h --max-depth=1
> 196M    ./log
> 365M    ./jdk1.8.0_131
> 27M     ./exporter
> 84K     ./temp
> 258M    ./logs
> 1.1G    ./jar
> 165M    ./apache-tomcat-8.5.14-saas
> 1.5G    ./bak
> 8.0K    ./package-backup
> 8.0K    ./private
> 69M     ./package
> 16K     ./lost+found
> 233M    ./logstash-5.0.0
> 37M     ./rsync
> 187M    ./tools
> 32K     ./data
> 153M    ./supervisor
> 303M    ./apache-tomcat-8.5.14
> 4.5G    .

### 2. 查看磁盘空间

```shell
df -h .
```

> #输出:
>
> [bhjr@VM_15_11_centos ~]$ df -h .
> Filesystem      Size  Used Avail Use% Mounted on
> /dev/vda1        50G  3.2G   44G   7% /



### 3. tar打包

```shell
打包单个
tar -zcvf app.tar.gz app
打包多个
tar -zcvf app.tar.gz redis rvm  tools  vf_api_data  xusa_data
打包全部
tar -zcvf app.tar.gz *
打包全部并排除logs,log,temp目录
tar --exclude=logs --exclude=log --exclude=temp -zcvf app.tar.gz ./*
```



