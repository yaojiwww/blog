# 仓颉 + Spire 博客后端

跟着 gitcode 上 `soulsoft/blog` 的教程**手工复现**（不 clone 仓库），边学边写。

## 目录约定

此项目专注后端，不涉及前端页面开发。

**仓库根 `/Users/yaojiwww/github/blog` 就是仓颉包的根**（`cjpm.toml` 所在处）。

教程里 `web-api/xxx` 的路径，在这里一律变成 `./xxx`。**没有 `web-api/` 或 `app-api/`

```
blog/                              ← 仓颉包的根，cjpm 命令在这里跑
├── CLAUDE.md
├── README.md
├── .gitignore
├── .vscode/
├── cjpm.toml                      # [package] 用 cjpm init 生成，依赖段从教程抄
├── src/
│   ├── abstractions/
│   ├── application/{behaviors,commands,models,queries,services}/
│   ├── controllers/
│   ├── domain/{entities,models,shared,views}/
│   ├── infrastructure/middlewares/
│   └── main.cj                    ← 入口必须在这里（放进 src/ 下）
├── blog.sql
└── appsettings.Development.json
```

## 开发环境

**在 macOS 上原生装仓颉 + spire框架 + mariadb**


| 东西 | 位置 |
|---|---|
| 仓颉 SDK 1.1.3（macOS arm64） | `~/cangjie`，`Target: aarch64-apple-darwin` |
| stdx（**单独下载**，SDK 不带） | `~/darwin_aarch64_cjnative/static/stdx` |
| OpenSSL | `/opt/homebrew/opt/openssl@3/lib`（**Homebrew 装的，见下面那条坑**） |

环境变量在 `~/.zshrc`：

```sh
export CANGJIE_STDX_PATH="$HOME/darwin_aarch64_cjnative/static/stdx"
export DYLD_LIBRARY_PATH="/opt/homebrew/opt/openssl@3/lib:$HOME/cangjie/runtime/lib/darwin_aarch64_cjnative:$HOME/cangjie/tools/lib"
```

⚠️ **`CANGJIE_STDX_PATH` 要指向 `static/stdx` 目录本身**（带 stdx 这一层），
不是它的父目录。指错了编译时找不到 stdx。

⚠️ 用户装了仓颉的 VS Code 插件（`.vscode/settings.json` 里是它写的配置）。

### 部署产物另外出

开发机 `aarch64-apple-darwin` ≠ 部署目标 `x86_64-unknown-linux-gnu`，
**产物完全不通用**。Mac 上只做"写代码 + 编译报错"的学习循环；出部署产物
必须在 Linux x86_64 上编（服务器本机 / amd64 容器 / CI）。

## 部署目标

**x86_64 的 Ubuntu 22.04**，而开发机是 arm64 —— **两边的编译产物不通用**。

仓颉是编译型语言，产物与 OS + CPU 架构强绑定。SDK 是工具（跟开发机架构走），
产物才是产品（跟服务器架构走），这两件事容易混淆。

如果之后又用回容器：宿主端口**不要用 5000**（macOS 的 ControlCenter /
隔空投送接收器默认占着它），数据库连接串要用服务名（如 `db:3306`）不能用 `localhost:3306`。

## 已经踩过的坑

- ⚠️ **cjpm 在 macOS 上会因为加载不到 OpenSSL 而 SIGABRT**：
  ```
  WARNING: .../cjpm is loading libcrypto in an unsafe way
  SIGABRT ... stdx.net.tls.TlsContext.createContext
  ```
  仓颉 SDK **不自带 OpenSSL**，系统 `/usr/lib` 下也没有，只有 Homebrew 装了。
  必须把 `/opt/homebrew/opt/openssl@3/lib` 加进 `DYLD_LIBRARY_PATH`，
  否则 `cjpm update` / `cjpm build` 只要走 HTTPS 就崩（已做对照实验确认）。
  另外 `envsetup.sh` 在变量为空时会留个**结尾冒号**，dyld 会把空项当成"当前目录"
  去搜动态库 —— 所以 `DYLD_LIBRARY_PATH` 要**整个重写**而不是追加。
- ⚠️ **教程的 `[target]` 没有 macOS 块**，只有 mingw32 和两个 linux-gnu。
  照抄的话在 Mac 上编译一块都不匹配 → stdx 找不到。项目里补了
  `[target.aarch64-apple-darwin]`，且**不加 `-ldl`**
  （macOS 的 dlopen/dlsym 在 libSystem 里，没有独立的 libdl）。
- **`cjpm.toml` 里 `mariadb` 是 Windows 反斜杠的本地依赖**
  （`"mariadb" = { path = ".\\library\\mariadb-driver" }`），Linux 上跑不通，
  要改成远程 git 依赖，或者把那个目录（388K / 59 文件）单独取过来。
- **仓颉官网的下载链接带鉴权参数**，在脚本里 `curl` 会失效，得手动下载。
- **`cjpm init` 在已有 `src/` 的目录里跑，只生成 `cjpm.toml`，不生成 `src/main.cj`。**
- **`cjpm update` 要求 `src/` 目录存在**，否则报 `path of src directory 'src' does not exist`。
- 那批 `soulsoft_*` / `sqlsharp` 依赖**都是源码包**（不是预编译二进制），
  所以 macOS arm64 上照样能拉、能编，不用担心没有 Mac 架构的产物。
  24 个直接依赖会自动带出 8 个传递依赖，共 32 个，全部正常解析。

## 网络 / 代理

用户机器上有代理软件（`MacPacket`，监听 `127.0.0.1:1082`），系统代理开着，
且 DNS 走 fake-ip（`github.com` 被解析成 `198.18.x.x`）。

**git 不会自动读 macOS 的系统代理设置**，所以连 GitHub 必须显式配置：

```bash
git config --global http.https://github.com.proxy http://127.0.0.1:1082
```

注意代理软件得开着，否则这条配置会让 git 连 GitHub 失败。**gitcode.com 直连正常，不需要代理。**

## 协作方式

用户正在边学边复现，要讲清"为什么"而不只是给命令。
不要重复推荐已经被否决的方案。
