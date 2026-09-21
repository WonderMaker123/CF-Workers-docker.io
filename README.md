# 🐳 Docker 镜像加速代理（基于 Cloudflare Workers）

本项目基于 Cloudflare Workers 搭建，帮你解决国内访问和下载 Docker 镜像慢或无法连接的问题。

[![Deploy to Cloudflare Workers](https://deploy.workers.cloudflare.com/button)](https://deploy.workers.cloudflare.com/?url=https://github.com/WonderMaker123/CF-Workers-docker.io)

---

## 🚀 方式一：一键部署到 Cloudflare（小白极简推荐）

点击下方按钮，登录 Cloudflare 账号后授权 GitHub，即可自动克隆并一键部署此 Worker：

[![Deploy to Cloudflare Workers](https://deploy.workers.cloudflare.com/button)](https://deploy.workers.cloudflare.com/?url=https://github.com/WonderMaker123/CF-Workers-docker.io)

> **注意**：一键部署完成后，请按照下方 [绑定自定义域名](#3-绑定自定义域名必须) 绑定你自己的域名（因为官方默认的 `*.workers.dev` 域名在国内无法直连访问）。

---

## 🛠️ 方式二：手动部署指南

### 1. 准备工作
- 拥有一个 Cloudflare 账号。
- 一个托管在 Cloudflare 上的**自定义域名**。

### 2. 创建并部署 Worker
1. 登录 Cloudflare 控制台，点击左侧导航栏的 **Workers 和 Pages** -> **创建应用程序** -> **创建 Worker**。
2. 随意输入一个名称（如 `my-docker-proxy`），点击右下角 **部署**。
3. 部署成功后，点击 **编辑代码**：
   - 清空里面原有的所有代码。
   - 复制本项目中的 `_worker.js` 文件全部内容，粘贴进去。
   - 点击右上角 **部署** 即可。

### 3. 绑定自定义域名（必须）
1. 回到刚刚创建的 Worker 详情页面。
2. 点击上方的 **设置 (Settings)** -> 找到 **域和路由 (Domains & Routes)**。
3. 点击 **添加 (Add)** -> 选择 **自定义域 (Custom Domain)**。
4. 输入你的二级域名（例如：`docker.yourdomain.com`），点击添加，等待解析生效。

### 4. （可选）配置环境变量
如需个性化功能（如绑定账号防频控限制、伪装重定向、特定安全白名单等），可在 Cloudflare 页面轻松配置：
1. 在 Worker 页面点击 **设置 (Settings)** -> **变量和机密 (Variables and Secrets)** -> 点击 **添加**。
2. 支持的环境变量如下（**全部为选填**，无需改动代码）：

| 变量名 | 默认值 | 说明与示例 |
| :--- | :--- | :--- |
| **`USERNAME`** | 空 | Docker Hub 用户名（建议配置，可解除 Docker Hub 对匿名 IP 的拉取速率限制）。 |
| **`PASSWORD`** | 空 | Docker Hub 密码或访问令牌 (Access Token)。 |
| **`URL`** | `nginx` | 浏览器访问根目录时的行为。<br>• 默认展示 Nginx 静态伪装页（安全防扫描）<br>• 设置为其他网址（如 `https://www.baidu.com`）时反向代理该地址。 |
| **`URL302`** | 空 | 浏览器访问根目录时 302 重定向的目标网址。 |
| **`SHOW_DOCKER_PAGE`** | `false` | 是否开启 Web 搜索界面。设置为 `true` 时，在浏览器打开域名会显示 Docker 镜像搜索网页。 |
| **`PROXY_TOKEN`** | 空 | 私人访问密钥/Token。配置后拉取镜像需带上 `?token=你的密钥` 才能访问。 |
| **`UA`** | `netcraft` | 需要拦截的爬虫 User-Agent 关键词。 |

> 💡 **小白提示**：如果你只是自用拉取镜像，**不需要配置任何变量**即可直接使用！若遇到频繁限流，只需填入 `USERNAME` 和 `PASSWORD` 即可。安全规则建议直接在 Cloudflare 域名的 **安全性 -> WAF** 中点选配置。

---

## 💻 使用方法

假设你绑定的域名是：`docker.yourdomain.com`

### 方法一：直接在命令行拉取（临时使用）
在原本的镜像名称前面加上你的域名即可：

```bash
# 官方常用镜像（如 nginx、alpine 等，需加上 library/ 前缀）
docker pull docker.yourdomain.com/library/nginx:latest

# 第三方镜像
docker pull docker.yourdomain.com/username/imagename:latest
```

---

### 方法二：全局配置加速（一键配置，推荐）
让服务器后续使用常规 `docker pull` 命令时自动通过加速代理下载。

在 Linux / 群晖 / VPS 终端中运行以下命令（**注意将域名替换为你自己的**）：

```bash
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://docker.yourdomain.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

配置完成后，以后直接正常拉取即可享受加速：
```bash
docker pull nginx:latest
```

---

### 方法三：群晖 / 极空间 / 绿联等 NAS 用户配置
1. 打开 NAS 自带的 **Docker / Container Manager** 界面。
2. 找到 **镜像注册表 / 仓库设置 / Registry**。
3. 新增或编辑 Docker Hub 注册表，将镜像 URL / 加速地址填写为：
   ```text
   https://docker.yourdomain.com
   ```
4. 保存后，在 NAS 中搜索或下载镜像即可自动加速。
