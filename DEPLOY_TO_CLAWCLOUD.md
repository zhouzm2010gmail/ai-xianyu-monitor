# ClawCloud 部署指南

本指南将帮助你将 `ai-xianyu-monitor` 部署到 ClawCloud (或任何标准 Linux VPS) 上。

## 1. 准备工作

### 1.1 获取服务器
在 ClawCloud 控制台购买一台 Linux 服务器，推荐配置：
- **操作系统**: Ubuntu 22.04 LTS 或 Debian 11/12 (推荐使用 Ubuntu)
- **CPU/内存**: 建议至少 2核心 2GB 内存 (Playwright 浏览器较消耗资源)
- **磁盘**: 建议 20GB 以上

### 1.2 连接服务器
使用 SSH 连接到你的服务器：
```bash
ssh root@<你的服务器IP>
```

## 2. 环境安装

在服务器终端执行以下命令，一键安装 Docker 和 Docker Compose：

```bash
# 更新系统
apt-get update && apt-get upgrade -y

# 安装 Docker (使用官方便捷脚本)
curl -fsSL https://get.docker.com -o get-docker.sh
sh get-docker.sh

# 验证安装
docker --version
docker compose version
```

## 3. 部署代码

### 3.1 上传代码
你可以选择 **Git Clone** (如果代码在 GitHub/GitLab) 或 **本地上传**。

**方式 A: 本地直接上传 (推荐，如果你有本地修改)**
在**你的本地电脑**上，使用 `scp` 命令将当前项目文件夹上传到服务器：

```bash
# 在项目根目录下执行
# 将 <服务器IP> 替换为实际 IP
scp -r . root@<服务器IP>:/root/ai-xianyu-monitor
```

**方式 B: Git Clone**
```bash
# 在服务器上执行
git clone https://github.com/Usagi-org/ai-goofish-monitor.git
cd ai-goofish-monitor
```

### 3.2 配置环境
进入项目目录并配置 `.env` 文件：

```bash
cd /root/ai-xianyu-monitor

# 复制示例配置
cp .env.example .env

# 编辑配置 (填入你的 OpenAI Key 等信息)
nano .env
```
*按 `Ctrl+O` 再按 `Enter` 保存，按 `Ctrl+X` 退出编辑器。*

**⚠️ 注意：**
- 确保 `.env` 中的 `OPENAI_API_KEY` 和 `OPENAI_MODEL_NAME` 已正确填写。
- 建议修改 `WEB_USERNAME` 和 `WEB_PASSWORD` 以保护后台安全。

## 4. 启动服务

使用我们生成的 `docker-compose.prod.yaml` 进行构建并启动：

```bash
# -f 指定使用生产环境配置
# --build 强制构建本地代码
# -d 后台运行
docker compose -f docker-compose.prod.yaml up --build -d
```

## 5. 验证与维护

### 查看服务状态
```bash
docker compose -f docker-compose.prod.yaml ps
```

### 查看实时日志
```bash
docker compose -f docker-compose.prod.yaml logs -f
```

### 访问 Web 界面
在浏览器输入：`http://<你的服务器IP>:8000`
默认账号/密码：`admin` / `admin123` (或者你在 .env 中设置的值)

### 停止服务
```bash
docker compose -f docker-compose.prod.yaml down
```

### 更新代码
如果你在本地修改了代码，需要重新上传覆盖，然后执行：
```bash
# 重新构建并重启
docker compose -f docker-compose.prod.yaml up --build -d
```

## 常见问题
1. **端口无法访问**：请检查 ClawCloud 的安全组/防火墙设置，确保 **8000** 端口已放行 TCP 流量。
2. **内存不足**：如果构建过程中报错退出，可能是内存不足，尝试增加 Swap 分区。
