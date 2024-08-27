# 04_docker和服务器时间问题

### 1.国内镜像

配置时间: 2024-08-26

地址: https://dockerpull.com

```shell
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<EOF
{
    "registry-mirrors": ["https://dockerpull.com"],
    "log-opts": {
        "max-size": "100m"
    },
    "insecure-registries": [
        "10.10.200.0/24"
    ]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```



### 2.切换时区

```shell
# 切换时区
rm -rf /etc/localtime
ln -s /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
cat /etc/localtime
date
```

### 3.同步时间

```shell
ntpdate -u ntp.aliyun.com && hwclock -w 
yum install -y ntpdate
ntpdate -u ntp.aliyun.com && hwclock -w 
date
# echo "Asia/Shanghai" | sudo tee /etc/timezone
```

4.docker使用服务器时间

`docker-compose.yml`文件中 volumes 配置下添加如下

`/etc/localtime:/etc/localtime:ro # 同步本地时间`

在 `environment`添加 `- TZ=Asia/Shanghai`

参考如下

```
services:
  app:
    image: node:18  # 使用官方 Node.js 18 镜像
    volumes:
      - .:/app/5G  # 将当前目录挂载到容器的 /app/5G 目录
      - /app/5G/node_modules  # 防止本地覆盖 node_modules
      - /etc/localtime:/etc/localtime:ro # 同步本地时间
    working_dir: /app/5G  # 设置工作目录
    ports:
      - "3007:3007"  # 暴露端口
    command: sh -c "npm config set registry https://registry.npmmirror.com && npm install pm2 -g && npm install && pm2-runtime ecosystem.config.js"  # 设置 npm 源、安装 PM2、安装依赖并启动应用
    environment:
      - NODE_ENV=production
      - TZ=Asia/Shanghai
```

