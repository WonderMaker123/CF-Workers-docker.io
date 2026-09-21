# 🐳 Docker 镜像加速代理（基于 Cloudflare Workers）

本项目基于 Cloudflare Workers 搭建，帮你解决国内访问和下载 Docker 镜像慢或无法连接的问题。

---

## 🛠️ 部署指南（推荐：极简 3 步网页部署）

> ⚠️ **说明**：Cloudflare 官方的一键部署按钮经常受 GitHub API 限流影响导致报错 `无法获取存储库内容`。**推荐直接使用官方控制台部署，1 分钟即可完成，最稳妥、不需要任何工具！**

### 第一步：创建 Worker
1. 登录 [Cloudflare 控制台](https://dash.cloudflare.com/)。
2. 点击左侧导航栏的 **Workers 和 Pages** -> **创建应用程序** -> 点击 **创建 Worker**。
3. 随意输入一个名称（例如：`docker-proxy`），直接点击右下角的 **部署**。

### 第二步：粘贴代码
1. 部署成功后，点击页面中间的 **编辑代码** 按钮。
2. 将左侧/中间编辑器原有的内容全部清空。
3. 打开本项目中的 [`_worker.js`](./_worker.js) 文件，**全选复制全部内容**，粘贴进 Cloudflare 编辑器中。
4. 点击右上角 **部署**（Deploy）即可生效。

### 第三步：绑定自定义域名（必须）
> 💡 Cloudflare 默认提供的 `*.workers.dev` 域名在国内已被阻断，必须绑定自己的域名才能正常拉取镜像。

1. 回到刚刚创建的 Worker 详情页面。
2. 点击顶部的 **设置 (Settings)** -> 找到 **域和路由 (Domains & Routes)**。
3. 点击 **添加 (Add)** -> 选择 **自定义域 (Custom Domain)**。
4. 输入你的二级域名（例如：`docker.yourdomain.com`，需已托管在 Cloudflare），点击添加，等待状态变为生效即可。

---

## ⚙️ （可选）配置环境变量
如需个性化功能（如绑定账号防频控限制、伪装重定向、开启搜索页等），可在 Cloudflare 页面轻松配置：
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

> 💡 **小白提示**：如果你只是自用拉取镜像，**不需要配置任何变量**即可直接使用！若遇到频繁限流，只需填入 `USERNAME` 和 `PASSWORD` 即可。区域白名单/防刷建议直接在 Cloudflare 域名的 **安全性 -> WAF** 中设置。

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
