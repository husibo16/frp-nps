# 🧭 FRP 一键部署脚本使用指南

本指南介绍如何使用仓库中的 `bin/install-frps.sh` 与 `bin/install-frpc.sh`，在 VPS + 家庭服务器 间快速搭建安全稳定的 FRP 内网穿透服务。

## 前置条件
- 系统：常见 Linux 发行版（Ubuntu / Debian / CentOS 等）
- 目标系统需为常见的 Linux 发行版，并使用 **systemd** 管理服务。
- 权限：以 `root` 运行（或 `sudo -i` 切换）
- 依赖：需预装 `curl、tar、install`（通常系统自带）

# 🚀 一、部署 FRP 服务端（VPS）

1. 登录拥有公网 IP 的 VPS。
2. 下载并安装：

   ```bash
   curl -fsSL https://raw.githubusercontent.com/husibo16/frp-nps/main/bin/install-frps.sh -o install-frps.sh
   chmod +x install-frps.sh
   sudo ./install-frps.sh
   ```

3. 执行后：
 - 安装位置：/usr/local/bin/frps
 - 默认配置：/etc/frp/frps.toml
 - 服务文件：/etc/systemd/system/frps.service
 - 自动启动并开机自启

4. 如需自定义端口或认证信息，可在运行脚本前通过环境变量覆盖：
 - 后续改动可直接编辑 /etc/frp/frps.toml 并重启服务。
   ```bash
   #======================
   # 基础监听端口
   #======================
   FRPS_BIND_PORT=7000      
   # 👉 客户端连接的主端口，需开放防火墙（默认7000）
   # ✅ 生产推荐：6000~8999 间任意空闲端口，如 7000、7100 等。

   #======================
   # Web 管理面板（Dashboard）
   #======================
   FRPS_WEB_ADDR=0.0.0.0    
   # 👉 监听地址（0.0.0.0 表示允许所有来源访问）
   # ✅ 生产推荐：保留默认；若启用防火墙，可仅放行管理 IP。

   FRPS_WEB_PORT=7500       
   # 👉 Web 面板端口：http://VPS_IP:7500
   # ✅ 生产推荐：非必须暴露公网，可用 Cloudflare Tunnel 反代访问。

   FRPS_WEB_USER=admin      
   # 👉 登录用户名
   # ✅ 生产推荐：改成自定义用户名。

   FRPS_WEB_PASS='StrongFrpWeb@2025'
   # 👉 登录密码
   # ✅ 必改！推荐复杂度高的强密码。

   #======================
   # 认证配置
   #======================
   FRPS_AUTH_METHOD=token   
   # 👉 认证方式，常用 token 模式（默认）
   # ✅ 推荐保留 token，OIDC 仅企业场景使用。

   FRPS_TOKEN='frpSecureToken@2025'
   # 👉 通信口令，客户端必须一致
   # ✅ 必改！长度≥12 且混合大小写符号。

   #======================
   # 日志配置
   #======================
   FRPS_LOG_PATH=/var/log/frps.log
   # 👉 日志文件路径
   # ✅ 推荐路径：/var/log/frps.log（独立于系统日志）

   FRPS_LOG_LEVEL=info
   # 👉 日志等级(trace/debug/info/warn/error)
   # ✅ 生产推荐：info（必要时可改为 warn）

   #======================
   # Prometheus 监控
   #======================
   FRPS_ENABLE_PROMETHEUS=true
   # 👉 是否启用 /metrics 端点
   # ✅ 推荐：true（方便监控流量/连接）

   ```

3️⃣ 修改配置与重启服务

   ```bash
sudo nano /etc/frp/frps.toml
sudo systemctl daemon-reload
sudo systemctl restart frps
sudo systemctl status frps
   ```
🔐 建议：首次安装后务必修改默认 token 和面板密码。

## 🖥️ 二、部署 FRP 客户端（家庭服务器）

1. 在需要穿透的家庭服务器上执行以下步骤。
2. 下载并安装：

   ```bash
   curl -fsSL https://raw.githubusercontent.com/husibo16/frp-nps/main/bin/install-frpc.sh -o install-frpc.sh
   chmod +x install-frpc.sh
   sudo ./install-frpc.sh
   ```

3. 自定义配置变量：

   ```bash
   #======================
   # 服务端连接信息
   #======================
   FRPC_SERVER_ADDR=1.2.3.4
   # 👉 服务端公网 IP 或域名（VPS）
   # ✅ 生产推荐：绑定 DDNS 域名，方便动态 IP。

   FRPC_SERVER_PORT=7000
   # 👉 服务端监听端口，需与 frps.toml 的 bindPort 一致。
   # ✅ 默认 7000，除非服务端改过。

   #======================
   # 认证信息
   #======================
   FRPC_AUTH_METHOD=token
   FRPC_TOKEN='frpSecureToken@2025'
   # 👉 必须与服务端一致，否则无法连接。
   # ✅ 建议统一放在 .env 文件中管理。

   #======================
   # 代理隧道设置
   #======================
   FRPC_TUNNEL_NAME=xboard
   # 👉 隧道名称，用于标识（多条隧道可用不同名称）
   # ✅ 推荐：与应用名一致，如 xboard、nginx、ssh

   FRPC_LOCAL_PORT=7001
   # 👉 本地服务端口（要穿透的服务）
   # ✅ 示例：Xboard 面板监听端口（宝塔默认 7001）

   FRPC_REMOTE_PORT=9000
   # 👉 VPS 端对外暴露的端口
   # ✅ 推荐：9000~9999 段；避免与 bindPort 冲突。


4. 查看与验证。
   ```
   sudo systemctl status frpc
   sudo journalctl -u frpc -f
   ```
6. 日志示例（成功）：：

   ```bash
   [I] [client/service.go:317] login to server success
   [I] [proxy/proxy_manager.go:177] proxy added: [xboard]

   ```

## 常用命令

```bash
# 查看运行状态
sudo systemctl status frps
sudo systemctl status frpc

# 实时日志（调试连接问题）
sudo journalctl -u frps -f
sudo journalctl -u frpc -f

# 修改配置后重启
sudo systemctl daemon-reload
sudo systemctl restart frps
sudo systemctl restart frpc

```

## 卸载/清理

若需卸载，可执行：

```bash
sudo systemctl disable --now frps frpc
sudo rm -f /usr/local/bin/frps /usr/local/bin/frpc
sudo rm -f /etc/systemd/system/frps.service /etc/systemd/system/frpc.service
sudo rm -rf /etc/frp
sudo systemctl daemon-reload
sudo systemctl reset-failed
```
> ⚠️ 卸载脚本未集成于仓库中，上述步骤需按需手动执行。
🧩 五、生产环境建议汇总表

| 配置项                 | 推荐值         | 说明         |
| ------------------- | ----------- | ---------- |
| `bindPort`          | 6000~8999   | 主连接端口，防扫描  |
| `auth.token`        | 长度≥12 强随机   | 必改，客户端需一致  |
| `webServer.port`    | 7500（仅内网访问） | 管理面板端口     |
| `FRPC_REMOTE_PORT`  | 9000~9999   | VPS 对外暴露端口 |
| `log.level`         | info        | 推荐生产等级     |
| `enablePrometheus`  | true        | 可供监控采集     |
| `systemctl restart` | 每次改配置后执行    | 确保生效       |




