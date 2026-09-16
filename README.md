# Slite for LazyCat (slite-lzcapp)

[Slite](https://github.com/miantiao-me/Slite) 打包为懒猫微服 LPK v2 应用：自托管短链接服务，带实时访问统计（3D 地球）、二维码、AI 辅助 slug 与导入导出。

## 包信息

- 包名：`community.lazycat.app.slite`（新应用，默认 community 前缀）
- 上游镜像：`ghcr.io/miantiao-me/slite`（GHCR，AGPL-3.0，作者 miantiao-me）
- 目标架构：amd64；`min_os_version: 1.6.0`（使用 service 级 `run_as: 5483` 映射持久目录属主，镜像为 distroless 非 root 运行）

## 转换要点

| Compose | LPK |
|---|---|
| `ports: 5483:5483` | `upstreams: / → http://slite:5483/`，`public_path: /`（短链对匿名访客开放，控制台由站点令牌保护） |
| `volumes: slite-data:/data` | `binds: /lzcapp/var/data:/data`（SQLite 链接库、DuckDB 统计、上传图片、备份） |
| `NUXT_SITE_TOKEN` | 部署参数 `site_token`（secret，`$random(len=20)`）+ `/dashboard/login` 免密自动填充 |
| `NUXT_TRUST_PROXY=false` | `NUXT_TRUST_PROXY=true`（懒猫网关为唯一可信入口，保证访客 IP/地域统计正确） |
| `init: true` / `stop_grace_period` | 由 lzcinit 接管进程生命周期，无需等价字段 |
| （无 healthcheck） | 不添加（遵循源 Compose） |
| 导入/导出、图片上传 | 接入懒猫网盘文件选择器拦截（商店强制要求） |

## 部署参数

| 参数 | 类型 | 默认 | 说明 |
|---|---|---|---|
| `site_token` | secret | `$random(len=20)` | 控制台/API 令牌（≥8 位无空格），登录页自动填充 |
| `ai_base_url` | string | 空 | 可选，OpenAI 兼容接口地址 |
| `ai_model` | string | 空 | 可选，模型标识 |
| `ai_api_key` | secret | 空 | 可选，AI 服务商密钥 |

## 本地构建与验证

```sh
lzc-cli project release          # 产出 dist/*.lpk
lzc-cli lpk info dist/*.lpk      # 查看包信息
```

## 自动发布（双商店）

`.github/workflows/lazycat.yml` 每日定时（或手动触发）调用
[ca-x/lazycat-github-action](https://github.com/ca-x/lazycat-github-action)@v1：

1. 检查 `ghcr.io/miantiao-me/slite` 新的 `vX.Y.Z` tag（`tag_regex: ^v\d+\.\d+\.\d+$`，stable + semver）
2. 远程复制镜像到 `registry.lazycat.cloud`（delivery: `lazycat`，官方商店必需），更新 manifest 与 `package.yml` 版本
3. 构建 LPK → 提交 → 打 tag → 创建 GitHub Release（附件 `community.lazycat.app.slite-v<version>.lpk`）
4. 发布到**官方商店**（`LZC_API_TOKEN`）与**私有商店**（`APPSTORE_URL`/`APPSTORE_TOKEN`）

所需 Secrets（组织或仓库级）：

| Secret | 用途 |
|---|---|
| `LZC_API_TOKEN` | 官方商店 PAT（镜像复制 + 官方发布） |
| `LZC_API_HOST` | 可选，PAT API 覆盖 |
| `APPSTORE_URL` / `APPSTORE_TOKEN` | 私有商店（喵喵商店） |
| `APP_ID` | 可选，私有商店应用 ID |
| `PRIVATE_STORE_GROUP_CODES` | 可选，私有分组 |

## License

上游 Slite 为 AGPL-3.0；本仓库仅为打包配置。
