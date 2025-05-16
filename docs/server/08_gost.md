# 服务器安装 gost

背景:服务器 socks 代理中转

---

- [gost 官网](https://gost.run/)

- 使用 GitHub 脚本进行一键部署 [脚本地址](https://github.com/KANIKIG/Multi-EasyGost)

- 登录天翼云控制台,对服务器进行重装,安装 Debian 11 系统,等待安装完成.
- 使用终端远程登录(刚刚的用户名和密码)
- 国内访问 GitHub 慢,所以需要使用代理,会解决很多网络问题.

```bash
export http_proxy="http://user:pass@ip:port"
export https_proxy="http://user:pass@ip:port" 
    ./your_script.sh # 你的脚本
unset http_proxy https_proxy  # 恢复原状
```

- 配置好代理后找到 GitHub 中的脚本 `wget --no-check-certificate -O gost.sh https://raw.githubusercontent.com/KANIKIG/Multi-EasyGost/master/gost.sh && chmod +x gost.sh && ./gost.sh`

- 第一步: 安装

    - 出现界面以后,选择 1 **安装 gost**
    - 因为设置了代理,所以让提示使用大陆镜像下载时选择 **n**

- 第二步: 恢复代理原状 unset http_proxy https_proxy

- 第三步: 执行脚本'./gost.sh' (刚刚的脚本会把这个.sh 文件放到当前目录下,所以直接执行即可)

    - 选择 7 **新增 gost 转发配置**
    - 选择 4 **一键安装 socks5 代理**
    - 选择 3 **http**
    - 输入密码,用户名和端口 (50000)
    - 成功以后会显示出相应的配置

- 第四步: 去防火墙开启上面输入的端口 (50000)

    - 天翼云进入机器详情,找到安全组,添加规则
    - 端口范围填写上面的端口 (50000)
    - 描述填写 gost, 确定

- 第五步: 测试代理

    - ```bash
        curl -x http://上面的用户名:上门的密码@你服务器的公网IP:上面的端口 ip.sb
        ```

    - 如果输出的结果与服务器 IP 地址相同,则验证成功!
