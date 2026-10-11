# vendor/ —— 手动放置的安装包

仓颉官网的下载链接带鉴权参数，**没法在 Dockerfile 里直接 curl**（会失效）。
所以这两个包要你手动下载，放到这个目录，Dockerfile 再用 `COPY` 装进镜像。

好处：构建稳定、可离线、重建容器不用重新下载。

## 要下载什么

去 [仓颉下载中心](https://cangjie-lang.cn/download) 下载两个包：

| 包 | 说明 |
|---|---|
| `cangjie-sdk-linux-aarch64-<版本>.tar.gz` | 仓颉 SDK（编译器 cjc + 包管理器 cjpm） |
| `cangjie-stdx-linux-aarch64-<版本>.zip` | 仓颉标准扩展库 stdx（SDK 里不含，必须单独装） |

版本按项目的 `web-api/cjpm.toml` 里 `cjc-version` 指定的来。

## 注意

- **架构要选 aarch64**（因为开发容器是 arm64）。将来出 x86_64 部署镜像时，
  换成 `x64` 的包，但**不要和 aarch64 的包同时放在这里** ——
  Dockerfile 用的是通配符 `cangjie-sdk-linux-*.tar.gz`，两个都在会匹配错。
- stdx 的版本号是四位（如 `1.1.0.1`），和 SDK 的版本编号规则不一样。
