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
- 拥有一个 [Cloudflare](https://dash.cloudflare.com/) 账号。
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

### 4. （可选）配置 Docker Hub 账号变量
如果经常大量拉取镜像，建议配置账号以突破 Docker 官方对匿名 IP 的频次限制：
1. 在 Worker 页面点击 **设置 (Settings)** -> **变量和机密 (Variables and Secrets)** -> 点击 **添加**。
2. 可添加以下变量：

| 变量名 | 说明 |
| :--- | :--- |
| **`USERNAME`** | 选填：你的 Docker Hub 账号用户名。 |
| **`PASSWORD`** | 选填：你的 Docker Hub 密码或 Access Token。 |

> 💡 *小提示：安全防护（如仅限中国访问、拦截境外扫描器等）后续直接在 Cloudflare 域名的 **WAF (安全性 -> 规则)** 中配置更加灵活便捷。*

---

## 💻 使用方法

假设你绑定的域名是：`docker.yourdomain.com`

### 方法一：直接在命令行拉取（临时使用）
在原本的镜像名称前面加上你的域名即可：

```bash
# 官方常用镜像（如 nginx、ubuntu 等，需加上 library/ 前缀）
docker pull docker.yourdomain.com/library/nginx:latest

# 第三方用户镜像
docker pull docker.yourdomain.com/stilleshan/frpc:latest
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
