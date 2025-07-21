# SSH 禁用密码认证(登录)

**背景: vps仅使用秘钥登录,禁用密码登录**

### 第一步:获取本机公钥

```sh
cat ~/.ssh/id_ed25519.pub
# ssh-ed25519 AAAA******H5 Aris
# 创建: ssh-keygen -t ed25519 -C "your_email@example.com"
```

### 第二步:vps 添加公钥并验证

```sh
vi ~/.ssh/authorized_keys
# 添加上面的id_ed25519.pub内容,保存退出!
```

- 客户端添加

```sh
vi ~/.ssh/config
```

- 添加如下:

```sh
Host vps101 # 名字
   User root
   Hostname your.VPS.IP.address #你 vps IP 地址
   Port 22
   IdentityFile ~/.ssh/id_ed25519 #私钥地址
```

- 测试连接

```sh
ssh vps101
# 登录成功即可
```

### 第三步:vps 禁用密码登录

```sh
cat > /etc/ssh/sshd_config.d/custom.conf << 'EOF'

# 全局禁用密码认证
PasswordAuthentication no

# 禁用键盘交互认证
ChallengeResponseAuthentication no
KbdInteractiveAuthentication no

# 禁用PAM认证
UsePAM no

# 明确要求使用公钥认证
PubkeyAuthentication yes

# 禁用root密码登录
PermitRootLogin prohibit-password

# 禁用X11转发
X11Forwarding no
EOF
```

重启 sshd

```sh
systemctl restart sshd
```

### 第四步: 在另外一台机器上验证

```sh
ssh root@your.VPS.IP.address
```

提示 **Permission denied (publickey).**即表示当前机器禁止密码登录!

成功!😄

---

## 在另外一台机器上验证密钥对登录

### 第一步: 拷贝本机 ~/.ssh 目录下的 id_ed25519 和 id_ed25519.pub

```sh
cat id_ed25519
# -----BEGIN OPENSSH PRIVATE KEY----- ******	-----END OPENSSH PRIVATE KEY-----
cat id_ed25519.pub
# ssh-ed25519 AAAA******H5 Aris
```



### 第二步: 在另外一台机器 ~/.ssh 创建 id_ed25519 和 id_ed25519.pub并对应拷贝输入密钥对内容

```sh
vi id_ed25519.pub
#输入 ssh-ed25519 AAAA******H5 Aris
```

```sh
vi id_ed25519
# 输入私钥内容
```

**注意: 私钥id_ed25519 私钥必须是这种格式,注意回车换行,注意首尾去空格**

```sh
# 注意: 私钥id_ed25519 私钥必须是这种格式,注意回车换行,注意首尾去空格
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnN******tzc2gtZW
QyNTUxO******3UNE/91D
RAAAAA******tlAzk92nB
AAAEDj******u+8XcM3Pq
T3acH4******cH5AAAABE
-----END OPENSSH PRIVATE KEY-----
```



### 第三步: 在另外一台机器 ~/.ssh目录修改 私钥 id_ed25519权限

```sh
# 修改 私钥权限,必须仅对所有者可读写!!!只有 -rw,否则提示 too open 错误
chmod 600 id_ed25519
ls -ll
# -rw------- 1 root root 387 Jul 21 10:04 id_ed25519
```

> 说明: SSH 对私钥文件（如 id_ed25519）的权限有严格要求：
>     私钥文件（如 id_ed25519）必须仅对所有者可读写，即**权限应为 600（-rw-------）**。
>     公钥文件（如 id_ed25519.pub）、config、known_hosts 等可以设置为 644（-rw-r--r--）。
>     authorized_keys 文件权限通常也应为 600 或 644，但更严格的做法是 600。

### 第四步: 在另外一台机器配置 config 文件

```sh
vi config
# 输入一下:
Host vps101
   User root
   Hostname your.VPS.IP.address
   Port 22
   IdentityFile ~/.ssh/id_ed25519
```

第五步: 在另外一台机器测试登录

```sh
ssh vps101
```

登录成功即可!

---

秘钥备份

- 自己本机 ssh 目录一份
- 1password 一份
- GitHub 私有仓库一份

enjoy! 😄

