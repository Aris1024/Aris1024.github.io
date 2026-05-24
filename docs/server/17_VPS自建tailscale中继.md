# 腾讯云VPS自建 Tailscale 中继服务(derp)

## 背景
- 已经有域名 (*.yourdomain.com)
- 已有 acme.sh 自动管理ssl证书
- 使用 docker部署
## 闭坑
- 端口拦截（重点）：如果你的域名没有在国内备案，腾讯云会拦截 80 和 443 端口。因此我们需要使用自定义端口（如 3443）来替代标准的 443 端口。
- 防白嫖机制（防滥用）：默认情况下，DERP 服务器是公开的。如果被别人扫到，你的 VPS 流量会被瞬间耗尽。因此必须开启客户端验证（verify-clients）。开启此功能的前提是：你的腾讯云 VPS 本身也必须安装 Tailscale 客户端并登录你的账号。
- 安全组放行：必须在腾讯云的控制台/安全组中放行你设置的 TCP 端口（如 3443）和 UDP STUN 端口（默认 3478）。

## 登录控制台 配置 Auth keys

- https://login.tailscale.com/admin/settings/keys
- ![image-20260330184034255](assets/image-20260330184034255.png)
- ![image-20260330184119131](assets/image-20260330184119131.png)


## 第一步：在 VPS 本机安装 Tailscale 客户端（防白嫖必备）
为了开启 verify-clients（只允许你自己的 Tailscale 账号使用这个中继，防止被别人扫到刷爆流量），你的 VPS 宿主机必须运行 Tailscale。
在你的 VPS 上执行官方一键安装脚本：
```code
curl -fsSL https://tailscale.com/install.sh | sh
```

```shell
# 验证状态
tailscale status
# 退出登录
tailscale logout
# 使用 auth key 重新登录
sudo tailscale up --authkey tskey-auth-xxxxxxxxxxxx
```



## 第二步：提取 acme.sh 的证书供 DERP 使用

vps 已经使用acme.sh 管理证书,直接使用即可

```shell
# VPS 之
~/.acme.sh/acme.sh --list
```

**你会看到类似如下的输出：**



```
Main_Domain      KeyLength  SAN_Domains   CA               Created               Renew
yourdomain.com      "ec-256"   *.yourdomain.com ZeroSSL.com      2023-10-01T12:00:00Z  2023-11-30T12:00:00Z
```

**如何解读并提取参数：**

- 看 Main_Domain（主域名）这一列，比如是 yourdomain.com，这就是你的第一个 -d 参数。
- 看 SAN_Domains（备用域名）这一列，比如是 *.yourdomain.com，这就是你的第二个 -d 参数。
- 如果你有多个域名，就把它们都用 -d 连起来。
- 所以对应的参数就是：-d yourdomain.com -d *.yourdomain.com（请根据你实际的输出去替换，主域名写前面，备用域名写后面）。

```shell
cat ~/.acme.sh/yourdomain.com_ecc/yourdomain.com.conf | grep -E "Le_RealKeyPath|Le_RealFullChainPath|Le_ReloadCmd"
```

**请看一眼输出的结果，假设（只是假设）你看到的输出类似这样：**



- Le_RealKeyPath='/etc/nginx/ssl/yourdomain.com.key'

- Le_RealFullChainPath='/etc/nginx/ssl/fullchain.cer'

- *(你只需要记住这两个真实路径，后面要在 Docker 里用到。)*

    

## 第三步：编写 docker-compose.yml

```shell
# 创建 docker 配置目录
mkdir -p /app/docker_tailscale
# 创建 docker-compose.yml 配置文件
cd /app/docker_tailscale
sudo vi docker-compose.yml
```

配置如下内容:

```yml
version: '3.8'
services:
  derper:
    image: fredliang/derper:latest
    container_name: tailscale-derper
    restart: always
    network_mode: "host" 
    environment:
      - DERP_DOMAIN=derp.yourdomain.com     
      - DERP_CERT_MODE=manual            
      - DERP_ADDR=:3443                  
      - DERP_HTTP_PORT=-1                
      - DERP_STUN_PORT=3478              
      - DERP_VERIFY_CLIENTS=true         
    volumes:
      # 【修改这里】：把冒号左边的路径，替换为你刚才在第一步查到的真实 FullChain 路径
      - /此处替换为你的/RealFullChainPath.cer:/app/certs/derp.yourdomain.com.crt:ro
      
      # 【修改这里】：把冒号左边的路径，替换为你刚才在第一步查到的真实 Key 路径
      - /此处替换为你的/RealKeyPath.key:/app/certs/derp.yourdomain.com.key:ro
      
      # 下面这个是鉴权用的 tailscale socket，保持不变
      - /var/run/tailscale/tailscaled.sock:/var/run/tailscale/tailscaled.sock
```

*注：结尾的* *:ro* *表示只读模式，保护你的原证书不被意外修改。*

### 解决自动续期后的重启问题

当 acme.sh 自动为你续期证书后，Nginx 会自动重新加载，但我们的 DERP 容器也需要重启才能读取到新证书。

最简单且绝对不破坏你现有 Nginx 配置的方法，是添加一个**定时任务（Cron Job）**，让它每天凌晨自动重启一次 DERP 容器（重启耗时不到 1 秒，完全无感）。

1. 输入命令编辑定时任务：

    ```shell
    crontab -e
    ```

    *(如果提示选择编辑器，选 1 也就是 nano 即可)*

2. 在文件最末尾添加这一行：

    ```shell
    0 4 * * * docker restart tailscale-derper >/dev/null 2>&1
    ```

    *(这代表每天凌晨 4 点重启一次 DERP 容器，确保续期后能立即加载新证书)*

3. 保存并退出。

## 第四步：在腾讯云控制台放行端口 (关键!)

- 登录腾讯云控制台 -> 进入你的云服务器（CVM/轻量应用服务器）详情页 -> 找到 “防火墙” 或 “安全组”：
- 添加以下两条入站规则：
    - TCP 端口：3443 （必须放行，这是 DERP 核心数据传输通道）
    - UDP 端口：3478 （必须放行，这是 STUN 协助 P2P 打洞的通道）
## 第五步：启动 DERP 中继服务

### 启动容器并测试

1. 在 /app/docker_tailscale 目录下启动：

    ```shell
    docker compose up -d
    ```

2. 看一下日志，确认证书有没有加载成功：

    ```sh
    docker logs -f tailscale-derper
    ```

## 第六步：去 Tailscale 控制台配置 ACL 规则

https://login.tailscale.com/admin/acls/file

```json
"derpMap": {
    "OmitDefaultRegions": true, # false:保留官方还未服务器备用; true: 只用当前VPS节点
    "Regions": {
      "900": {
        "RegionID":   900,
        "RegionCode": "Tencent",
        "RegionName": "Tencent Cloud Derper",
        "Nodes":[
          {
            "Name":     "1",
            "RegionID": 900,
            "HostName": "derp.yourdomain.com",
            "DERPPort": 3443,
            "STUNPort": 3478
          }
        ]
      }
    }
  }
```

*(注：**OmitDefaultRegions: false* *意思是保留官方的海外中继作为备用。如果你想***只***用你自己的腾讯云测试一下，可以暂时把它改成* *true**)*

![image-20260330190027665](assets/image-20260330190027665.png)

## 第七步：最终测试！

拿出你的**个人电脑**（或者是手机等其它设备，**不能是这台腾讯云 VPS**），打开命令行终端，输入：

```sh
tailscale netcheck
```

稍等几秒钟，看输出结果：

- 向下寻找 Report: 区域，你会看到列表里多了一个 **Tencent Cloud Derper**！
- 看看它的延迟是不是绿色的，并且数字极低（通常国内互联只有 10ms ~ 30ms）。

![image-20260330190352037](assets/image-20260330190352037.png)

---

## mac 客户端配置

- 下载地址: https://tailscale.com/download
- 用网页登录即可 (也可以使用 auth key 登录)
- auth key 配置地址: https://login.tailscale.com/admin/settings/keys

## Windows 客户端配置

- 下载地址: https://tailscale.com/download
- 安装提示错误,需要开启 iphlpsvc 服务

![d3123f4a127c88524696c364c723c3ad](assets/54ff45f9a70b41ab64c5fa5867f68c96.png)

解决办法

![d3123f4a127c88524696c364c723c3ad](assets/d3123f4a127c88524696c364c723c3ad-4869629.png)

然后再安装就可以了;

建议使用 auth key 登录 (我不喜欢在哪里都用网页登录,不方便~)

auth key 配置地址: https://login.tailscale.com/admin/settings/keys


```sh
# Windows 的操作 auth key 登录
PS C:\Users\Administrator> tailscale up --authkey tskey-auth-kN******E8GmiRdXnG1H1
# tailscale 状态
PS C:\Users\Administrator> tailscale status
100.116.137.78   desk**t51  w***@  windows  -
100.114.217.106  al**             w***@  macOS    -
100.86.77.5      vm***bian   w***@  linux    -

# Windows 检查服务状态
PS C:\Users\Administrator> Get-Service Tailscale

Status   Name               DisplayName
------   ----               -----------
Running  Tailscale          Tailscale

# 确保启动类型为自动
PS C:\Users\Administrator> Set-Service -Name "Tailscale" -StartupType Automatic
PS C:\Users\Administrator> Start-Service Tailscale
```



### 禁止被其他设备访问:

执行 `tailscale up --shields-up --accept-routes`

#### 查看状态

```bash
tailscale status
```

如果显示 `-` 或 `shields-up` 标记，说明已生效。




---



## 配置域名访问内网服务 (已放弃该方案,受限于腾讯云网络,如果后续配置可以参考!)

以deerflow为例, 在组网机器中,访问 http://100.96.95.15:2026 能正常访问

现在要改成 https://deerflow.yourdomain.com 方式!

第一步：修改 DNS 解析（最关键的一步）
    既然你有泛域名解析，为了让 deerflow 这个子域名走内网，你需要单独为它添加一条解析记录，去覆盖泛域名解析。
    登录你的域名解析服务商（如腾讯云 DNSPod / 阿里云等）。
    添加一条新的解析记录：
    主机记录（Host）：deerflow
    记录类型：A 记录
    记录值：填写你 VPS 的 Tailscale IP（即以 100.x.x.x 开头的那个 IP）。
    原理解释：100.64.0.0/10 是 Tailscale 的专属局域网 IP 段，公网是无法路由的。当公网上的陌生人 Ping deerflow.yourdomain.com 时，会得到一个 100.x.x.x 的局域网 IP，根本连不上；而你朋友的电脑因为开启了 Tailscale，它的虚拟网卡知道如何将流量加密并发送给你的 VPS。

1. 查看服务机器的组网ip

    - ```
        tailscale status
        ```

    - 找到目标机器的ip地址(100.96.95.15)

2. 配置 VPS nginx

    - ```sh
        nginx -t # 查询nginx配置文件位置
        ```

    - ```nginx
        # Nginx 反向代理（在 VPS 上）
        server {
            listen 443 ssl;
            server_name deerflow.yourdomain.com;
        
            ssl_certificate /root/.acme.sh/yourdomain.com_ecc/fullchain.cer;
            ssl_certificate_key /root/.acme.sh/yourdomain.com_ecc/yourdomain.com.key;

            # 【核心安全配置】：只允许 Tailscale 网段访问，拒绝其他所有公网访问
            allow 100.64.0.0/10;  # Tailscale 分配的专属 IP 段
            allow 127.0.0.1;      # 允许本地访问（排错用）
            deny all;             # 拒绝除此之外的所有访问！
        
            location / {
                proxy_pass http://100.96.95.15:2026;
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        
                # 如果你的服务包含 WebSocket，请保留下面两行
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection "upgrade";
            }
        }
        ```

3. 重启nginx

    - ```sh
        # 测试配置是否正确
        sudo nginx -t
        
        # 重新加载配置（推荐，不中断连接）
        sudo systemctl reload nginx
        
        # 或完全重启
        sudo systemctl restart nginx
        
        # 查看状态
        sudo systemctl status nginx
        ```

    - 

4. 在组网的其他机器上访问 

    - 机器上连接tailscale
    - 能访问 https://deerflow.yourdomain.com 即可
