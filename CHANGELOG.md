# Changelog

历史 tag 与 commit 内版本号的对照放在文末「历史发布 Tag / 版本错位」一节。版本发布明细以 [GitHub Releases](https://github.com/CarlJia/agent/releases) 为准。

## v1.1.0 - 2026-09-21

### 破坏性变更

- **上报协议改 msgpack**：上行从 JSON 文本帧改为 msgpack 二进制帧,约省 50% 流量。envelope 新增 `version` 首字段(当前 `1`),版本不匹配即断开重连——**agent 与 hub 必须同步升级**,旧 JSON agent 连不上新 hub,反之亦然。
- **不变量按需上报**:`mem_total`/`swap_total`/`disk_total` 不再每帧发,只在 hello 时发;检测到漂移(热插盘/内存变更)时主动重发 hello,与新的 `used` 值在同一帧配对。热插盘后新盘最多 30 秒(mounts 缓存 TTL)进入 totals。

### 优化

- 热路径去分配:`proc_count` 改读 `/proc/loadavg` 的 total(不再每秒遍历 `/proc` 全目录);`meminfo` 去掉 HashMap 改固定结构;`/proc/self/mounts` 加 30 秒 TTL 缓存 + O(n) 解析;网卡前缀判定数组常量化。

## 历史发布 Tag / 版本错位（只读，不重建）

下列 tag 在发布时**没有同步更新** `Cargo.toml` 里的版本号。重建需要 `push --force`，会影响远端已引用的 tag；本节只标记、保留 tag 不动。

### 类型 A：发布漏改（`Cargo.toml` 没跟着 tag 升级）

| Tag | 指向 commit 的 `version` | 期望 | 状态 |
|-----|--------------------------|------|------|
| `v1.0.1` | `1.0.0` | `1.0.1` | 漏改 |
| `v1.0.2` | `1.0.0` | `1.0.2` | 漏改 |
| `v1.0.3` | `1.0.0` | `1.0.3` | 漏改 |
| `v1.0.4` | `1.0.0` | `1.0.4` | 漏改 |
| `v1.0.5` | `1.0.5` | `1.0.5` | ✓（最近一次发版） |

### 重建映射（按 `Cargo.toml` 反查 tag）

- `Cargo.toml = "1.0.0"` → 多个 tag 都指向版本号仍为 `1.0.0` 的 commit：`v1.0.1` / `v1.0.2` / `v1.0.3` / `v1.0.4`——发布序号与 commit 内版本号脱钩，类型 A。
- `Cargo.toml = "1.0.5"` → tag `v1.0.5`（最近一次正确发版）。

如需引用「Cargo.toml 版本号 = tag 名」的对应关系，请使用 `v1.0.5` 及后续 tag。
