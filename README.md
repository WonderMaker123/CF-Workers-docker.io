[**第三方 DockerHub 镜像服务列表**](https://github.com/cmliu/CF-Workers-docker.io?tab=readme-ov-file#%EF%B8%8F-%E7%AC%AC%E4%B8%89%E6%96%B9-dockerhub-%E9%95%9C%E5%83%8F%E6%9C%8D%E5%8A%A1)

![CF-Workers-docker.io](./img.png)

# 🐳 CF-Workers-docker.io：Docker仓库镜像代理工具

这个项目是一个基于 Cloudflare Workers 的 Docker 镜像代理工具。它能够中转对 Docker 官方镜像仓库的请求，解决一些访问限制和加速访问的问题。

> [!CAUTION]
> **docker.fxxk.dedyn.io 已被GFW污染，需自行部署使用。**

> [!WARNING]
> 根据 [Cloudflare 协议](https://www.cloudflare.com/zh-cn/terms/) 中，2.2.1 第 (j) use the Services to provide a virtual private network or other similar proxy services.
>
> 使用本服务可能存在被 Cloudflare 封号的潜在风险，请自行斟酌使用风险。
>
> 如果你选择了“根据主机名选择对应的上游地址”方式部署，你可能会:
> 
> 被 Netcraft 扫描到，收到警告邮件
>
> 被 Netcraft 同步到 Google Safe Browsing 标记为钓鱼网站
>
> 被 Netcraft 投诉到 Cloudflare 标记为钓鱼网站, 无法正常 pull 镜像
>
> 收到律师函

## 🚀 部署方式

- **Workers 部署（推荐）**：
  1. 登录 [Cloudflare Dashboard](https://dash.cloudflare.com/)，进入 **Workers & Pages** -> **Create application** -> **Create Worker**。
  2. 点击 **Deploy**，然后进入 **Edit code**，将本项目中的 [_worker.js](https://github.com/cmliu/CF-Workers-docker.io/blob/main/_worker.js) 内容全部替换粘贴进去，点击右上角 **Deploy**。
  3. 回到该 Worker 的管理页面，进入 **Settings** -> **Variables and Secrets**，按需添加环境变量（详见下文配置）。
  4. 进入 **Settings** -> **Domains & Routes** -> **Add** -> **Custom Domain**，绑定一个你自己的独立二级域名（如 `docker.yourdomain.com`）。
- **Pages 部署**：`Fork` 本仓库后 `连接 GitHub` 选择 Pages 一键部署即可。

## 🛡️ 个人自用安全防护与环境变量配置

为了规避 Netcraft、Google Safe Browsing 扫描导致的**钓鱼标记红屏、Cloudflare 封号警告**，并解决 Docker 官方**匿名拉取配额限制**，请在 Cloudflare 环境变量中添加如下设置：

### 1. 必选/推荐环境变量（在 Worker 的 Settings -> Variables 添加）

| 变量名 | 推荐值 | 作用说明 |
|:---|:---|:---|
| **`USERNAME`** | `你的DockerHub用户名` | 配置你的 Docker Hub 个人账号，解除 IP 共享的 100 次/6小时匿名限额。 |
| **`PASSWORD`** | `dckr_pat_xxxx` | 你的 Docker Hub 密码或访问令牌 (PAT)。 |
| **`REGION_WHITELIST`** | `CN` | **防境外扫描核心**：仅允许中国 IP 访问。国外所有 Netcraft、安全扫描器访问均只显示 Nginx 默认欢迎页，无法探测到任何 Docker 接口。 |
| **`UA_WHITELIST_REGEX`** | `^(docker\|containerd\|podman\|nerdctl\|curl\|synology)` | **客户端白名单核心**：仅放行 Docker 引擎拉取请求，浏览器访问一律伪装为 Nginx 页面。 |
| **`URL`** | `nginx` | 主页默认伪装为 Nginx 默认欢迎页。 |

> 💡 **提示**：以上变量也可以不走环境变量，直接在 `_worker.js` 开头的变量中修改默认值。

---

## ⚙️ 如何使用？ [视频教程](https://www.youtube.com/watch?v=l2jwq9CagNQ)

例如您的Workers项目域名为：`docker.fxxk.dedyn.io`；

### 1.官方镜像路径前面加域名

```shell
docker pull docker.fxxk.dedyn.io/stilleshan/frpc:latest
```

```shell
docker pull docker.fxxk.dedyn.io/library/nginx:stable-alpine3.19-perl
```

### 2.一键设置镜像加速

修改文件 `/etc/docker/daemon.json`（如果不存在则创建）

```shell
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://docker.fxxk.dedyn.io"]  # 请替换为您自己的Worker自定义域名
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

### 3. 配置常见仓库的镜像加速

#### 3.1 配置

`Containerd` 较简单，它支持任意 `registry` 的 `mirror`，只需要修改配置文件 `/etc/containerd/config.toml`，添加如下的配置：

```yaml
    [plugins."io.containerd.grpc.v1.cri".registry]
      [plugins."io.containerd.grpc.v1.cri".registry.mirrors]
        [plugins."io.containerd.grpc.v1.cri".registry.mirrors."docker.io"]
          endpoint = ["https://xxxx.xx.com"]
        [plugins."io.containerd.grpc.v1.cri".registry.mirrors."registry.k8s.io"]
          endpoint = ["https://xxxx.xx.com"]
        [plugins."io.containerd.grpc.v1.cri".registry.mirrors."k8s.gcr.io"]
          endpoint = ["https://xxxx.xx.com"]
        [plugins."io.containerd.grpc.v1.cri".registry.mirrors."gcr.io"]
          endpoint = ["https://xxxx.xx.com"]
        [plugins."io.containerd.grpc.v1.cri".registry.mirrors."ghcr.io"]
          endpoint = ["https://xxxx.xx.com"]
        [plugins."io.containerd.grpc.v1.cri".registry.mirrors."quay.io"]
          endpoint = ["https://xxxx.xx.com"]
```

`Podman` 同样支持任意 `registry` 的 `mirror`，修改配置文件 `/etc/containers/registries.conf`，添加配置：

```yaml
unqualified-search-registries = ['docker.io', 'k8s.gcr.io', 'gcr.io', 'ghcr.io', 'quay.io']

[[registry]]
prefix = "docker.io"
insecure = true
location = "registry-1.docker.io"

[[registry.mirror]]
location = "xxxx.xx.com"

[[registry]]
prefix = "registry.k8s.io"
insecure = true
location = "registry.k8s.io"

[[registry.mirror]]
location = "xxxx.xx.com"

[[registry]]
prefix = "k8s.gcr.io"
insecure = true
location = "k8s.gcr.io"

[[registry.mirror]]
location = "xxxx.xx.com"

[[registry]]
prefix = "gcr.io"
insecure = true
location = "gcr.io"

[[registry.mirror]]
location = "xxxx.xx.com"

[[registry]]
prefix = "ghcr.io"
insecure = true
location = "ghcr.io"

[[registry.mirror]]
location = "xxxx.xx.com"

[[registry]]
prefix = "quay.io"
insecure = true
location = "quay.io"

[[registry.mirror]]
location = "xxxx.xx.com"

```

#### 3.3 使用

对于以上配置，k8s 在使用的时候，就可以直接 `pull` 外部无法 pull 的镜像了。

```shell
# 手动可以直接pull配置了mirror的仓库
crictl pull registry.k8s.io/kube-proxy:v1.28.4
docker  pull nginx:1.21
```

## 🔧 变量说明

| 变量名 | 示例 | 必填 | 备注 |
|--|--|--|--|
| USERNAME | `your_dockerhub_username` |❌| Docker Hub 用户名（建议与 PASSWORD 一同配置以提升匿名限制） |
| PASSWORD | `dckr_pat_xxxx` |❌| Docker Hub 密码或个人访问令牌 (Personal Access Token) |
| REGION_WHITELIST | `CN` 或 `CN,HK,MO` |❌| **防境外扫描神器**：限制仅中国 IP 访问，境外节点/扫描器一律返回 Nginx 页面 |
| UA_WHITELIST_REGEX | `^(docker|containerd|podman|nerdctl|curl|synology)` |❌| **个人防扫必备**：仅放行常见容器引擎拉取，所有浏览器/爬虫访问均伪装为 Nginx |
| IP_WHITELIST_REGEX | `^(1\.2\.3\.4|123\.123\.)` |❌| **高安全性**：限制仅允许指定家庭/服务器公网 IP 地址访问 |
| PROXY_TOKEN | `my_secret_token_123` |❌| 访问令牌，设置后需在 URL 参数 `?token=xxx` 或请求头 `x-proxy-token` 携带 |
| URL302 | `https://t.me/CMLiussss` |❌| 主页302跳转 |
| URL | `https://www.baidu.com/` |❌| 主页伪装(设为`nginx`则伪装为nginx默认页面) |
| UA | `netcraft` |❌| 屏蔽爬虫UA，支持多元素, 元素之间使用空格或换行作间隔 |

# 🛠️ 第三方 DockerHub 镜像服务

**注意:**

- 以下内容仅做镜像服务的整理与搜集，未做任何安全性检测和验证。
- 使用前请自行斟酌，并根据实际需求进行必要的安全审查。
- 本列表中的任何服务都不做任何形式的安全承诺或保证。

| DockerHub 镜像仓库 | 镜像加地址 |
| ------------------ | ----------- |
| [bestcfipas 镜像服务](https://t.me/bestcfipas/4018) | `https://docker.registry.cyou` |
|  | `https://docker-cf.registry.cyou` |
|  | `https://registry.lfree.org` |
| [zero_free 镜像服务](https://t.me/zero_free/80) | `https://docker.jsdelivr.fyi` |
|  | `https://docker.aeko.cn` |
| [mingyu 镜像服务](https://github.com/ymyuuu/HubP) | `https://hubp.de` |
| [Docker 镜像加速站](https://docker.1panel.live)  | `https://docker.1panel.live` |
| [Hub Proxy](https://hub.rat.dev) | `https://hub.rat.dev` |
| [DaoCloud 镜像站](https://github.com/DaoCloud/public-image-mirror) | `https://docker.m.daocloud.io` |

# 🙏 鸣谢
### 💖 赞助支持 - 提供云服务器
- [![digitalvirt.com](https://digitalvirt.com/templates/BlueWhite/img/logo-dark.svg)](https://url.cmliussss.com/dv)

### 🛠 开源代码引用
- [muzihuaner](https://github.com/muzihuaner)
- [V2ex网友](https://global.v2ex.com/t/1007922)
- [ciiiii](https://github.com/ciiiii/cloudflare-docker-proxy)
- [ChatGPT](https://chatgpt.com/)
- [白嫖哥](https://t.me/bestcfipas/1900)
- [zero_free频道](https://t.me/zero_free/80)
- [dongyubin](https://github.com/cmliu/CF-Workers-docker.io/issues/8)
- [kiko923](https://github.com/cmliu/CF-Workers-docker.io/issues/5)
