# issh-plugin-registry

issh 桌面客户端（Tauri 版）的官方插件索引仓库。客户端「设置 → 插件商城」从此仓库拉取 `index.json` 浏览与安装插件。

## 索引格式

`index.json`（schema 1）：

```json
{
    "schema": 1,
    "updated": "2026-08-27T00:00:00Z",
    "plugins": [
        {
            "id": "issh-plugin-llm",
            "name": "AI 命令补全",
            "version": "0.3.0",
            "description": "…",
            "kind": "feature",
            "audience": "user",
            "permissions": ["terminal:decorate", "settings:tab"],
            "minAppVersion": "0.0.1",
            "downloadUrl": "https://gitee.com/YangAlvin/issh-plugin-registry/raw/main/packages/issh-plugin-llm/0.3.0/issh-plugin-llm-0.3.0.tgz",
            "sha256": "…",
            "signature": "…",
            "homepage": "…",
            "repository": "…"
        }
    ]
}
```

### 字段说明

| 字段 | 说明 |
|---|---|
| `id` | 插件唯一标识（`issh-plugin-` 前缀） |
| `name` / `version` / `description` | 展示信息 |
| `kind` | 分类：`feature`（功能）/ `appearance`（外观）/ `integration`（集成） |
| `audience` | 面向对象：`user`（普通用户，默认展示）/ `developer`（开发者示例，需手动开启） |
| `permissions` | 权限声明：`settings:tab` / `home:card` / `panel:register` / `terminal:decorate` / `profiles:read` / `profiles:write` |
| `minAppVersion` | 最低客户端版本门槛 |
| `downloadUrl` | Gitee 索引仓库 `packages/<id>/<version>/<id>-<version>.tgz` 下的原始签名包 |
| `sha256` | tgz 包哈希，安装时 Rust 端强制校验 |
| `signature` | ed25519 签名（base64），对 `id\nversion\nsha256` UTF-8 字节签名；客户端内置公钥验签 |

## 签名机制

- 客户端 Rust 端内置 ed25519 公钥（`plugin_market.rs` 的 `SIGNING_PUBLIC_KEY`）
- 拉取索引与安装下载时双重强制验签：签名不匹配 → 拒绝加载/安装
- 私钥由维护者本地保管（`~/.psacowork/issh-plugin-signing.key`，勿提交）

## 发布流程

monorepo 中开发（`plugins/<name>/`），用发布脚本完成 build → 打包 tgz + sha256 → ed25519 签名 → GitHub 备份发布 → Gitee 镜像原包与索引 → 更新 GitHub 索引。商城优先访问 Gitee；旧 GitHub 官方源设置也会优先使用国内索引和安装包。

```bash
node scripts/publish-plugin.mjs <pluginDirName>
```

要求：gh CLI 已认证、Gitee SSH 对 `YangAlvin/issh-plugin-registry` 有写入权限、签名私钥已生成（`node scripts/gen-signing-key.mjs`）。缺失签名或摘要不一致会终止发布。

已有版本迁移或补齐国内镜像无需重构建、重签名：

```bash
node scripts/sync-plugin-gitee.mjs           # 本地校验原始签名包与索引
node scripts/sync-plugin-gitee.mjs --publish # 单次提交原包、索引与说明到 Gitee
```

镜像工具逐包核对固定发布者签名及 SHA-256，保留已有版本，不强推远端历史；本地缺少旧包时复用已克隆国内仓库里的原包。客户端只允许官方 Gitee 索引跳转到 `raw.giteeusercontent.com` 的同一路径，自定义索引仍禁止跳转。安装包请求失败或返回非成功 HTTP 状态时可尝试 GitHub 备用源；包大小超限或摘要失败立即停止。错误提示保留网络/TLS/超时原因并去除 URL 查询参数。

## 当前收录插件

| 插件 | 版本 | 分类 | 说明 |
|---|---|---|---|
| issh-plugin-ai-assistant | 0.2.2 | feature | AI 助手 |
| issh-plugin-linkifier | 0.1.1 | feature | 终端链接识别 |
| issh-plugin-llm | 0.3.0 | feature | AI 命令补全 |
| issh-plugin-config-sync | 0.2.0 | integration | 主机配置同步 |
| issh-plugin-serial | 0.1.0 | integration | 串口终端 |
| issh-plugin-sandbox-demo | 0.4.2 | feature | 沙箱面板演示（开发者示例） |
| issh-plugin-theme-field | 0.1.3 | appearance | Field 主题 |
| issh-plugin-theme-foundry | 0.1.3 | appearance | Foundry 主题 |
| issh-plugin-theme-phosphor | 0.1.3 | appearance | Phosphor 主题 |
| issh-plugin-theme-void | 0.1.3 | appearance | Void 主题 |

> 已内置进程序、不再作为插件分发：sudo 密码自动填充、保险库（vault）。客户端会拒绝安装同名插件。
