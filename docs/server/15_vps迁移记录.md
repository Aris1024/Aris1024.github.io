# vps迁移记录

2025-10-16

##### 背景: 腾讯云 VPS 即将到期,续费较贵,更换账户抢购优惠VPS机器,需将原有服务迁移到新VPS上.

旧机器: 我的账号, 北京六区, Centos
新机器: 新的账号, 广州六区, Dibian


1. 镜像共享
    - 我的北京机器打包镜像,共享给新的广州账号(ID)
    - 因地域不同,需将镜像拷贝到广州区,然后再共享.
    - 新的机器重装系统,选择广州区,就会显示共享镜像
    PS: 最后未成功,因两个系统盘大小不一致,无法重装."当前实例系统盘大小40GB不满足该镜像的最小系统盘配置50GB要求。您可以升级套餐后再选择此镜像。"

2. 旧VPS 打包, 新VPS解压
    - 进入root,打包: tar -cvzf archive.tar.gz *
    - 发送至新机器: scp ./archive.tar.gz root@新机器IP:/root/
        注意: 关闭窗口会中断,可以使用
        nohup scp -r /path/to/file root@B机器IP:/root/
        进程会在后台运行
        查看进度:jobs 或 ps aux | grep scp
    - 新机器解压 tar -xvzf archive.tar.gz

3. 安装 nginx
    命令:
        apt update
        apt install nginx -y
    配置:
        cd /etc/nginx
        把旧机器的nginx配置复制到新机器的 /etc/nginx/nginx.conf 文件中
    启动(重启)
    systemctl start nginx
    systemctl status nginx

4. 安装docker
    绕了很多弯路,直接说结果: [使用腾讯云的脚本](https://cloud.tencent.com/document/product/213/46000)

    1. 执行以下命令，添加 Docker 软件源。
        ```shell
        sudo apt-get update
        sudo apt-get install ca-certificates curl -y
        sudo install -m 0755 -d /etc/apt/keyrings
        sudo curl -fsSL https://mirrors.cloud.tencent.com/docker-ce/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
        sudo chmod a+r /etc/apt/keyrings/docker.asc
        echo   "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://mirrors.cloud.tencent.com/docker-ce/linux/debian/ \
        $(. /etc/os-release && echo "$VERSION_CODENAME") stable" |   sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
        sudo apt-get update
        ```
        
    2. 执行以下命令，安装 Docker。
        ``` shell
        sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin 
        ```
    3. 执行以下命令，运行 Docker。
        ```shell
        sudo systemctl start docker 
        ```
    4. 执行以下命令，检查安装结果。
        ```shell
        sudo docker info
        ```
    5. 其他命令
        ```shell
        sudo systemctl stop docker
        sudo systemctl restart docker
        sudo docker pull nginx 
        docker ps -a
        ```

5. 配置docker镜像源
    编辑(没有就创建) /etc/docker/daemon.json


    - ```json
        {
            "registry-mirrors": [
                "https://mirror.ccs.tencentyun.com"
            ]
        }
        ```


6. 重启

    ```shell
    # 配置完重启
        sudo systemctl restart docker
    # 查看docker信息
        docker info
    ```

7. 启动项目
    进入docker 项目中,在 docker-compose.yml文件目录中
    执行命令 (没有中间横杠)
        docker compose up -d
    查看
        docker ps -a

8. 域名和网站
    - 查询配置:因为新旧网站内容一样,需要区分,在nginx配置中 'root /app/public/docs;'
    - 修改内容:进入 /app/public/docs 修改 index.html文件的title字段,改成 '技术博客' 以区分!
    - 修改解析: 在我的账户中,找到域名解析,把IP 改成新机器IP地址

9. 其他可能问题
    ssl证书到期更新? 需配置 acme.sh 自动更新证书