---
title: 用 Docker、1Panel 和 Cloudflare 部署一个个人服务
published: 2026-08-14
description: 记录一次从本地项目到公网 HTTPS 服务的完整部署过程，包括 Docker Compose、1Panel 反向代理和 Cloudflare 配置。
image: api
tags: [Docker, VPS, 1Panel, Cloudflare]
category: 技术
draft: false
---

把项目在本地跑起来只是第一步。要让它长期稳定地运行在公网，还需要解决容器启动、域名解析、反向代理和 HTTPS 等问题。最近我完整走了一遍这套流程，这篇文章把关键步骤和容易踩坑的地方整理下来。

## 部署结构

这次使用的结构并不复杂：

```text
访客
  ↓
Cloudflare
  ↓
1Panel OpenResty（HTTPS 和反向代理）
  ↓
Docker Compose 服务（127.0.0.1:8787）
```

应用本身只需要监听一个本地端口，域名和证书交给 1Panel 与 Cloudflare 处理。这样以后更换应用或迁移容器时，不必改动外部访问地址。

## 一、确认服务器环境

部署前先确认系统架构、磁盘、内存和 Docker 是否可用：

```bash
cat /etc/os-release
uname -m
free -h
df -h /
docker --version
docker compose version
git --version
```

检查环境比直接执行安装脚本更重要。特别是内存较小的 VPS，应提前配置 Swap，并留意 Docker 构建时的内存占用。

## 二、拉取项目并配置环境变量

将项目放在统一目录下，后续维护会方便很多：

```bash
mkdir -p ~/apps
cd ~/apps
git clone <项目仓库地址>
cd <项目目录>
cp .env.example .env
```

编辑 `.env` 时，我习惯遵循三个原则：

- 所有默认密码和示例 Token 必须更换。
- 公网地址统一填写最终的 HTTPS 域名。
- `.env` 永远不提交到 Git 仓库。

配置完成后，可以用 `grep` 单独检查非敏感选项，但不要把完整 `.env` 输出到聊天记录或截图中。

## 三、启动 Docker Compose

```bash
docker compose up -d --build
docker compose ps
docker compose logs --tail=100
```

看到容器状态为 `healthy` 还不够，最好再从服务器本机请求一次健康检查：

```bash
curl http://127.0.0.1:8787/health/ready
```

如果这里无法访问，问题通常在应用或容器配置；如果本机访问正常而域名访问失败，问题才更可能出在反向代理、证书或防火墙。

## 四、配置 1Panel 反向代理

在 1Panel 中创建网站并设置反向代理，代理地址填写：

```text
127.0.0.1:8787
```

这里有一个很容易遇到的错误：代理目标输入框可能只接受“主机和端口”，如果填写成 `http://127.0.0.1:8787`，面板生成配置时可能再次拼接协议，最后触发 `invalid port in upstream`。

保存以后先用 HTTP 测试，确认代理链路正常，再申请和启用 HTTPS。

## 五、配置 Cloudflare 与 HTTPS

域名在 Cloudflare 中添加 A 记录指向 VPS 公网 IP。源站证书配置完成后，Cloudflare 的 SSL/TLS 模式建议使用 `Full (strict)`，避免 Cloudflare 到源站之间使用不安全连接。

最终检查：

```bash
curl -I https://example.com/health
```

应当看到 `200` 状态码，同时响应头中包含 HTTPS 和 Cloudflare 相关信息。

## 六、日常更新与排错

以后更新项目通常只需要：

```bash
cd ~/apps/<项目目录>
git pull --ff-only
docker compose up -d --build
docker compose ps
docker compose logs --tail=100
```

遇到问题时，我会按“应用本机访问 → Docker 日志 → 反向代理 → Cloudflare”的顺序排查。把链路拆开验证，比反复修改配置更容易找到真正原因。

## 写在最后

个人服务部署并不只是执行一条启动命令。真正重要的是让每一层的职责清晰：Docker 管理应用，1Panel 管理入口和证书，Cloudflare 管理 DNS 与边缘访问。理解这条链路以后，再部署其他项目也会轻松很多。
