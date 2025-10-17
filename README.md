# frp 一键部署脚本使用指南

本文档介绍如何使用仓库中的 `bin/install-frps.sh` 与 `bin/install-frpc.sh`，快速在 VPS 与家庭服务器之间搭建自建 frp 内网穿透服务。

## 前置条件

- 目标系统需为常见的 Linux 发行版，并使用 **systemd** 管理服务。
- 以 `root` 用户执行脚本，或使用 `sudo` 切换到 root。
- 系统中需预装 `curl`、`tar`、`install` 等常用命令（大多数发行版默认已安装）。

## 部署 frp 服务端（VPS）

1. 登录拥有公网 IP 的 VPS。
2. 下载脚本：

   ```bash
   curl -fsSL https://raw.githubusercontent.com/husibo16/frp-nps/main/bin/install-frps.sh -o install-frps.sh
   chmod +x install-frps.sh
   ```

3. 执行脚本：

   ```bash
   sudo ./install-frps.sh
   ```

   - 脚本会自动获取 GitHub 上 frp 的最新稳定版，下载后安装到 `/usr/local/bin/frps`。
   - 默认配置文件位于 `/etc/frp/frps.toml`，同时生成 `/etc/frp/frps.toml.default` 以供参考。
   - 服务文件 `/etc/systemd/system/frps.service` 会自动创建，并立即启用与启动。

4. 如需自定义端口或认证信息，可在运行脚本前通过环境变量覆盖：
   ```bash

   # 基本监听参数
   #======================

   # FRPS_BIND_PORT：frps 主服务监听端口（客户端 frpc 连接用）
   # 默认 7000，可通过环境变量覆盖，例如 FRPS_BIND_PORT=6000
   BIND_PORT=${FRPS_BIND_PORT:-7000}

   #======================
   # Web 管理面板配置（Dashboard）
   #======================

   # FRPS_WEB_ADDR：仪表盘监听地址（默认 0.0.0.0，即所有网卡）
   # 兼容旧变量 FRPS_DASHBOARD_ADDR
   WEB_ADDR=${FRPS_WEB_ADDR:-${FRPS_DASHBOARD_ADDR:-"0.0.0.0"}}

   # FRPS_WEB_PORT：仪表盘端口，默认 7500
   # 兼容旧变量 FRPS_DASHBOARD_PORT
   WEB_PORT=${FRPS_WEB_PORT:-${FRPS_DASHBOARD_PORT:-7500}}

   # FRPS_WEB_USER：仪表盘登录用户名，默认 admin
   # 兼容旧变量 FRPS_DASHBOARD_USER
   WEB_USER=${FRPS_WEB_USER:-${FRPS_DASHBOARD_USER:-admin}}

   # FRPS_WEB_PASS：仪表盘登录密码，默认 changeme
   # 兼容旧变量 FRPS_DASHBOARD_PASS
   WEB_PASS=${FRPS_WEB_PASS:-${FRPS_DASHBOARD_PASS:-changeme}}

   #======================
   # 认证配置
   #======================

   # FRPS_AUTH_METHOD：认证方式，默认 token
   # 目前 frp 支持 token、oidc 等模式
   AUTH_METHOD=${FRPS_AUTH_METHOD:-token}

   # FRPS_TOKEN：服务端认证口令，客户端必须一致
   # 默认 changeme，请务必修改！
   AUTH_TOKEN=${FRPS_TOKEN:-changeme}

   #======================
   # 日志配置
   #======================

   # FRPS_LOG_PATH：日志文件路径（默认 /var/log/frps.log）
   # 支持旧变量 FRPS_LOG_TO
   LOG_PATH=${FRPS_LOG_PATH:-${FRPS_LOG_TO:-/var/log/frps.log}}

   # FRPS_LOG_LEVEL：日志等级（默认为 info，可选 trace/debug/warn/error）
   LOG_LEVEL=${FRPS_LOG_LEVEL:-info}

   #======================
   # Prometheus 监控
   #======================

    # FRPS_ENABLE_PROMETHEUS：是否启用 /metrics 监控端点
    # 默认为 true，设置为 false 可关闭
   ENABLE_PROMETHEUS=${FRPS_ENABLE_PROMETHEUS:-true}

   #===================================
   # 后续脚本会根据以上变量生成 frps.toml
   #===================================
   ```

5. 首次安装后，脚本会提醒默认 token `changeme`，请手动编辑 `/etc/frp/frps.toml` 并重启服务：

   ```bash
   sudo nano /etc/frp/frps.toml
   sudo systemctl restart frps
   ```

## 部署 frp 客户端（家庭服务器）

1. 在需要穿透的家庭服务器上执行以下步骤。
2. 下载脚本：

   ```bash
   curl -fsSL https://raw.githubusercontent.com/husibo16/frp-nps/main/bin/install-frpc.sh -o install-frpc.sh
   chmod +x install-frpc.sh
   ```

3. 配置并运行脚本：

   ```bash
   # 服务端连接信息
   #========================

   # FRPC_SERVER_ADDR：frps 服务端地址（公网 IP 或域名）
   # 默认 example.com（仅占位），必须改成你 VPS 的公网地址
   SERVER_ADDR=${FRPC_SERVER_ADDR:-example.com}

   # FRPC_SERVER_PORT：frps 服务端监听端口
   # 默认 7000（对应 frps.toml 中的 bindPort）
   SERVER_PORT=${FRPC_SERVER_PORT:-7000}

   #========================
   # 认证信息
   #========================

   # FRPC_AUTH_METHOD：认证方式，默认为 token
   # 当前常用方式是 token，未来版本可能支持 OIDC 等。
   AUTH_METHOD=${FRPC_AUTH_METHOD:-token}

   # FRPC_TOKEN：客户端连接口令（必须与 frps.toml 的 token 一致）
   # 默认 changeme，建议改成强随机字符串。
   AUTH_TOKEN=${FRPC_TOKEN:-changeme}

   #========================
   # 代理隧道设置
   #========================

   # FRPC_TUNNEL_NAME：隧道名称，用于标识该代理。
   # 默认 "web"，可自定义多个不同名称以建立多条隧道。
   TUNNEL_NAME=${FRPC_TUNNEL_NAME:-web}

   # FRPC_LOCAL_PORT：本地服务端口（被映射的目标）
   # 例如：Xboard 面板在本地 7001 端口，则这里填 7001。
   LOCAL_PORT=${FRPC_LOCAL_PORT:-80}

   # FRPC_REMOTE_PORT：远程服务端口（在 frps 上暴露的公网端口）
   # 外部用户访问 VPS:REMOTE_PORT 时，会自动转发到本地 LOCAL_PORT。
   # 默认 6000，可以根据需要修改。
   REMOTE_PORT=${FRPC_REMOTE_PORT:-6000}

   #===================================
   # 后续脚本会基于以上变量生成 frpc.toml
   #===================================

4. 脚本会生成 `/etc/frp/frpc.toml` 并创建 `frpc.service` systemd 单元。首次运行若仍使用示例地址或默认 token，脚本会给出提示信息。

5. 修改配置后，可通过以下命令重启：

   ```bash
   sudo systemctl restart frpc
   ```

## 常用命令

```bash
# 查看服务状态
sudo systemctl status frps
sudo systemctl status frpc

# 查看日志
sudo journalctl -u frps -f
sudo journalctl -u frpc -f
```

## 卸载/清理

若需卸载，可执行：

```bash
sudo systemctl disable --now frps
sudo rm -f /etc/systemd/system/frps.service /usr/local/bin/frps

sudo systemctl disable --now frpc
sudo rm -f /etc/systemd/system/frpc.service /usr/local/bin/frpc

sudo rm -rf /etc/frp
```

> ⚠️ 卸载脚本未集成于仓库中，上述步骤需按需手动执行。
