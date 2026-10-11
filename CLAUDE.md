# 仓颉 + Spire 博客后端

跟着 gitcode 上 `soulsoft/blog` 的教程**手工复现**（不 clone 仓库），边学边写。

## 目录约定

**项目根目录 `/Users/yaojiwww/github/blog` 就是仓颉包的根，不分 `web-api/`和 `web-ui` 那一层。**

教程里所有 `web-api/xxx` 的路径，在这里一律变成 `./xxx`。别再建议加 `web-api/`。

```
blog/
├── Dockerfile                  # 开发容器的定义
├── docker-compose.yml          # dev + db 两个服务
├── .devcontainer/              # 让 VSCode 连进容器
├── vendor/                     # 手动下载的仓颉 SDK / stdx 安装包（不入版本库）
├── cjpm.toml                   # [package] 用 cjpm init 生成，依赖段从教程抄
├── src/                        # controllers/ application/ domain/ infrastructure/
├── blog.sql                    # 建库建表脚本
└── appsettings.Development.json
```

## 范围

- **只做后端，不写 web-ui。** 教程里 web-ui / wwwroot / npm 相关的章节全部跳过。
- 接口用 Postman 测。

## 开发环境

全部跑在 Docker 里，**Mac 上不装任何仓颉工具链**（cjc / cjpm / stdx 都在容器内）。

- `dev`：Ubuntu 22.04 **arm64**（Apple Silicon 原生速度），仓颉 **1.1.3**，Target `aarch64-unknown-linux-gnu`
- `db`：`mariadb:11`，root 密码 `root123`，库名 `blog`
- 常用命令：`docker compose up -d` / `docker compose exec dev bash`

### 两个必须记住的坑

1. **宿主端口是 5001，不是 5000。**
   macOS 的 ControlCenter（隔空投送接收器）默认占着 5000，映射 `5000:5000` 会
   `bind: address already in use`，容器直接起不来。所以映射写的是 `5001:5000`。
   容器内部仍然是 5000，但 **Postman 要用 `http://localhost:5001`**。

2. **数据库连接串用 `db:3306`，不是 `localhost:3306`。**
   容器里的 `localhost` 指 dev 容器自己。密码是 `root123`（教程里写的是 `1024`）。

### 构建相关的坑

- 仓颉官网下载链接带鉴权参数，**Dockerfile 里 curl 会失效**，所以 SDK 和 stdx 要
  手动下载放进 `vendor/`。Dockerfile 用通配符匹配，换版本不用改文件，
  但**同一时间只能放一种架构的包**。
- `cjpm.toml` 里 `mariadb` 原本是 Windows 反斜杠的**本地依赖**（`.\library\mariadb-driver`），
  Linux 上跑不通，要改成远程 git 依赖。

## 部署目标

**x86_64 的 Ubuntu 22.04**，而开发机是 arm64 —— 两边的编译产物不通用。

出部署产物时要**另外起一个 amd64 容器**在里面编译（模拟运行，慢，但只在发布时跑一次）。
**日常开发容器必须保持 arm64**，改成 amd64 会慢到没法用。

## 协作方式

用户是 Docker 新手，正在边学边复现。讲清"为什么"，不要只丢命令。
不要重复推荐已经被否决的方案。
