# 老旧 Mac mini (2011) 改造 Debian 13 无头服务器：网络冲突与自动化部署全记录

## 一、 环境背景与改造逻辑

*   **硬件设备**：Mac mini (Mid 2011) / Intel Core i5 2.3 GHz / 8GB DDR3 / Fusion Drive (500GB HDD + 24GB SSD)
*   **算力限制**：该二代酷睿 CPU **不支持 AVX2 指令集**，物理层面无法运行依赖该指令集的现代本地 AI 大模型（如 LLaMA、腾讯 Marvis 本地模式等）。
*   **选型结论**：放弃 macOS High Sierra 与 Windows 10，选择安装 **Debian 13 (Trixie) amd64**。为规避老旧 Intel HD 3000 集显性能瓶颈，并保障 RustDesk 等远控软件在 Linux 下的兼容性（规避 Wayland 协议黑屏问题），必须选择基于 X11 协议的 **XFCE 桌面环境**。

---

## 二、 核心认知盲区与避坑复盘

在本次部署中，出现过以下几个典型的 Linux/网络底层认知盲区，需重点记录：

### 1. 架构混淆：`amd64` vs `arm64`
*   **误区**：凭借对现代苹果 M 系芯片的惯性，误下载 `arm64` 架构的安装包。
*   **纠正**：老款 Intel Mac 刷入 Linux 后，其系统架构为标准的 **`amd64` (即 x86_64)**。在 GitHub 下载任何 Release 包（如 Clash Verge, RustDesk）时，必须认准 `amd64`。

### 2. 包管理机制：`dpkg` 的局限性
*   **误区**：使用 `dpkg -i xxx.deb` 安装本地包报错“缺少依赖”时，不知如何处理。
*   **纠正**：`dpkg` 仅为本地解包工具，不具备自动联网解决依赖的能力。规范的离线包安装工作流应为：
    1.  `sudo apt update`（更新源目录）
    2.  `sudo dpkg -i xxx.deb`（强行解包，报错无视）
    3.  **`sudo apt --fix-broken install -y`**（核心步骤：调用高级包管理器自动联网下载缺失项并完成配置）

### 3. CIDR 子网掩码计算：`/10` 的真实含义
*   **误区**：在为 Tailscale 配置代理绕过规则时，误以为 `100.64.0.0/10` 仅指代 `100.64.*.*`，质疑其无法覆盖 `100.113.*.*` 等分配 IP。
*   **纠正**：`/10` 为无类别域间路由（CIDR）表示法。IP 地址底层为 32 位二进制，`/10` 意为**锁定前 10 位，后 22 位自由分配**。
    *   二进制锁定前 10 位：`01100100 . 01` (即 100 . 64)
    *   剩余位全 0 到全 1 的实际十进制范围是：**`100.64.0.0` 至 `100.127.255.255`**。
    *   **结论**：一行 `100.64.0.0/10` 足以精准覆盖 Tailscale 所使用的整个 CGNAT 保留网段（419万个IP），无需额外添加。

### 4. 环境变量层级：Nano 转 Vim
*   **误区**：使用 `sudo update-alternatives --config editor` 全局替换 Vim 后，执行 `crontab -e` 依然唤起 Nano。
*   **纠正**：`crontab` 强依赖执行者当前用户的局部环境缓存 `~/.selected_editor`。修改 Root 用户的 `crontab` 编辑器，必须显式调用 **`sudo select-editor`** 重新分配优先级。

---

## 三、 网络冲突治理 (Clash TUN vs P2P)

当 Clash Verge 开启 TUN 模式（虚拟网卡接管全局流量）时，会劫持系统的 UDP 流量，导致远控与组网软件的 P2P 穿透失败。

### 1. RustDesk 远控冲突
*   **故障**：UDP 打洞流量被劫持至海外节点，NAT 丢包导致无法连接。
*   **方案**：搭建自建中继服务器 (Relay Server)。在主控端与被控端填写中继 IP 与 Key，并**勾选“强制走中继连接”**，将不可控的 UDP 转换为稳定的 TCP 长连接。

### 2. Tailscale 组网冲突
*   **故障**：Tailscale 底层寻找节点的流量被劫持，导致全盘离线。
*   **方案**：在各设备的 Clash Verge `设置 -> 虚拟网卡模式 (⚙️) -> 排除自定义网段 (Bypass IP/CIDR)` 中双向写入以下路由白名单，并重启 TUN：
    *   `100.64.0.0/10` （Tailscale 专属网段）
    *   `192.168.0.0/16` （家庭局域网）
    *   `10.0.0.0/8` （企业/高级路由局域网）

---

## 四、 无头服务器 (Headless Server) 自动化部署

为了实现拔掉键盘、鼠标、显示器，且在遭遇意外断电后能够完全自恢复的服务器环境，需要打通底层硬件到图形界面的三道防线。

### 1. 主板寄存器级：断电恢复 (来电自启)
脱离 macOS 后，需使用 `setpci` 直接向主板 Intel LPC 电源管理芯片写入状态。
*   `00:1f.0` 为 2011 Mac mini LPC 控制器地址。
*   `0xa4.b=0` 代表设置状态为：通电即开机。

**执行与持久化写入：**
```bash
sudo crontab -e
```
在末尾追加定时任务，确保软关机不会重置该寄存器：
```text
@reboot /usr/bin/setpci -s 00:1f.0 0xa4.b=0
```
*(测试方法：机器运行时直接拔掉电源，10秒后重新插上，机器应自动点亮。软关机不会触发此机制。)*

### 2. 操作系统级：免密自动登录桌面
Clash Verge 等用户级 GUI 软件必须在桌面会话建立后才能随之启动，因此必须跳过登录界面的验证。
针对 `lightdm` 显示管理器写入自动登录规则：

```bash
# 1. 确保规则存放目录存在
sudo mkdir -p /etc/lightdm/lightdm.conf.d

# 2. 写入规则文件 (ai 替换为实际用户名)
echo -e "[Seat:*]\nautologin-user=ai\nautologin-user-timeout=0" | sudo tee /etc/lightdm/lightdm.conf.d/12-autologin.conf
```
*(逻辑说明：`[Seat:*]` 针对所有逻辑工作台。拔掉物理显示器后系统会生成虚拟 `seat0`，规则依然生效。)*

### 3. 物理安全级：开机延迟锁屏
自动登录会带来极大的物理窃密风险，需通过系统级锁屏组件在后台服务启动后切断物理屏幕交互。

**第一步：安装锁屏组件**
```bash
sudo apt update && sudo apt install light-locker -y
```

**第二步：写入延迟锁屏自启脚本**
将锁屏指令放入 XFCE 的自启动目录。设置为延迟 30 秒执行，确保网络组件与 Clash 渲染完成。
```bash
# 创建自启动目录
mkdir -p ~/.config/autostart

# 写入锁屏脚本（延时 30 秒，随后执行 xflock4）
echo -e "[Desktop Entry]\nType=Application\nName=AutoLock\nExec=sh -c 'sleep 30 && xflock4'\nRunHook=0\nStartupNotify=false\nTerminal=false\nHidden=false" > ~/.config/autostart/autolock.desktop
```
*(锁屏后，`light-locker` 会向显示器发送 DPMS 熄屏信号。物理显示器进入省电模式，但 SSH 与后台代理不受任何影响。)*

### 4. 补充说明：显卡欺骗器 (Dummy Plug)
Mac 拔掉所有显示器后，集显将关闭硬件加速，导致 RustDesk 远控时分辨率被锁死为 800x600 且极其卡顿。
**解决方案**：需在机身物理 HDMI 接口长期插一个 HDMI 显卡欺骗器（模拟 1080P/4K EDID 信号），骗过系统输出完整图形性能。