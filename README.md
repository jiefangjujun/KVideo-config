# KVideo 精选源配置（个人版）

基于 [puppet680/KVideo-config](https://github.com/puppet680/KVideo-config) 的自动化巡检框架，源列表已按**实测码率 + 起播速度 + 广告洁净度**重新筛选，仅保留 6 个常规源，无成人内容。

## 订阅地址

| 用途 | 地址 |
| :--- | :--- |
| **KVideo 订阅（推荐）** | `https://raw.githubusercontent.com/jiefangjujun/KVideo-config/main/lite.json` |
| 国内加速镜像 | `https://cdn.jsdelivr.net/gh/jiefangjujun/KVideo-config@main/lite.json` |
| 原始全量 | `https://raw.githubusercontent.com/jiefangjujun/KVideo-config/main/KVideo-config.json` |

在 KVideo 的「设置 → 订阅管理 → 添加订阅」里填入上面第一条即可，仓库每天自动巡检更新，订阅内容会跟着变。

> 注意：订阅链接必须是返回 **JSON 数组**的地址，`baseUrl`（如 `https://ikunzyapi.com/api.php/provide/vod`）不能当订阅链接用，那是单个视频源。

## 精选源清单

实测方法：每个源用 2 个片名搜索测延迟，抓 m3u8 主清单读声明分辨率，下载首分片按「码率 = 体积 ÷ 时长」反推实际画质，首分片加载耗时记为起播速度。

| # | 源 | 实测码率 | 分辨率 | 搜索延迟 | 起播耗时 | 插播广告 | 定位 |
| :---: | :--- | :---: | :--- | :---: | :---: | :---: | :--- |
| 1 | iKun 资源 | 1405 kbps | 1920×1080 | 0.52s | 2.0s | 无 | 唯一横屏 1080P，主力 |
| 2 | 猫眼资源 | 644 kbps | 1280×720 | 0.54s | 1.5s | 无 | 起播最快，秒开备用 |
| 3 | 极速资源 | 920 kbps | 未声明（≈720P） | 0.43s | 2.0s | 无 | 覆盖最广 |
| 4 | 红牛资源 | 471 kbps | 未声明（≈480P） | 0.71s | 1.6s | 无 | 响应快，画质一般 |
| 5 | 光速资源 | 471 kbps | 未声明（≈480P） | 0.31s | 2.0s | 无 | 搜索最快，热门片名易超时 |
| 6 | 魔都资源 | 1648 kbps | 720×1280 竖屏 | 0.42s | 2.3s | 无 | 码率最高，短剧居多 |

> 这批采集源整体码率偏低：1.4 Mbps 的 1080P 属于压缩版（正常 1080P 在 3–5 Mbps），所以「最优」不等于「真高清」。

### 首轮巡检结果

GitHub Actions 首次运行（关键字「斗罗大陆」）后，5 个源通过，**光速资源被自动剔除**：

| 源 | 结果 | 原因 |
| :--- | :--- | :--- |
| iKun / 猫眼 / 极速 / 红牛 / 魔都 | ✅ 100% | 正常返回结果 |
| 光速资源 | ❌ 0% | 返回 `{"code":1,"msg":"服务器繁忙，请稍后搜索"}` |

复测确认这不是偶发：同一接口搜《庆余年》《甘将》时，「庆余年」正常、《甘将》报繁忙——**它的搜索后端在高热度片名上会超时**。所以 `lite.json` 当前只有 5 个源。

`KVideo-config.json` 里仍保留了光速，巡检每天重新测试，恢复后会自动回到 `lite.json`，无需手动干预。

## 被剔除的源及原因

| 源 | 原因 |
| :--- | :--- |
| 无尽资源 | 搜索接口返回 403，不可用 |
| 如意资源 | 仅 608P，分片数异常（疑似预告片） |
| 爱奇艺 | 分片伪装成 `.jpeg` 后缀，有 2 处插播断点，起播 4.0s |
| 360 资源 | 917kbps 但起播 4.1s |
| 电影天堂 / 量子 / 暴风 / 非凡 / U酷 | m3u8 全部 403，播放链路无法验证 |
| 豪华 / 金鹰 / 新浪 / 速播 | 与光速同一上游（分片路径同为 `947/20260915/3709113`、码率同为 471kbps），重复冗余 |

原始 58 条（含 30 条成人源）保留在 `sources-archive.json`，仅作备份，不参与巡检。

## 关于广告

- **元数据层**：用 30 个广告词扫描全部源的前 60 条记录，只有 2 条命中「广告」，逐条核对是剧情简介里的文字，属误报，接口本身没有推广文案。
- **播放层**：解析所有 m3u8，没有 `#EXT-X-DISCONTINUITY` 插播断点，也没有 `/ad/`、`/gg/`、`preroll` 命名的分片，说明没有整段贴片。
- **播放器兜底**：在 KVideo 环境变量里配置，自动跳过命中的广告分片：

```
AD_KEYWORDS=ad,ads,gg,guanggao,preroll,sponsor,promo,juliang
```

画面内嵌的角标水印无法过滤，只能换源。

## 仓库结构

| 文件 | 说明 |
| :--- | :--- |
| `KVideo-config.json` | 源列表录入入口，改这里即可增删源 |
| `lite.json` | 巡检后自动生成，剔除 premium 组与失效源，供 KVideo 订阅 |
| `adult.json` | 巡检后自动生成，包含全部源（含失效源备注） |
| `check_api.js` | 巡检核心：测连通性、算 priority、生成报告 |
| `.github/workflows/build-check-encode.yml` | 每天北京时间 01:00 自动运行 |
| `CORSAPI/_worker.js` | Cloudflare Worker 中转脚本 |
| `report.md` | 巡检简报（源站状态、7 天趋势） |
| `sources-archive.json` | 原始全量配置备份 |

## 自动巡检

推送到 `KVideo-config.json` 或每天 01:00 会触发 Actions：

1. 依次请求每个 `baseUrl?ac=detail&wd=斗罗大陆`，超时 10s、重试 2 次。
2. 按 30 天历史成功率计算 `priority`：100% → 1，≥90% → 5，其余 → 10，连断 3 天 → 200+，手动禁用 → 999。
3. 重写 `lite.json` / `adult.json` / `report.md` 并提交。

**Fork 仓库的 Actions 默认关闭**，需要在 `Actions` 页面点一次「I understand my workflows, go ahead and enable them」。

## 可选：Cloudflare Worker 中转

源站被墙或服务端直连失败时使用：

1. 把 `CORSAPI/_worker.js` 部署到 Cloudflare Workers。
2. 绑定 KV 命名空间，变量名必须是 `KV`（600s 缓存）。
3. 订阅链接换成 `https://你的域名.workers.dev/?format=1&source=lite`。
   `format=1` 会把每个 `baseUrl` 重写为 Worker 中转地址；`format=0` 为原始直链。

KVideo 的搜索由服务端发起，本身不受浏览器跨域限制，多数情况下不需要中转。

## 致谢

巡检框架、Worker 脚本与配置结构来自 [puppet680/KVideo-config](https://github.com/puppet680/KVideo-config)，本仓库仅替换源列表并补充实测数据。所有源均为第三方采集接口，可用性与内容合规性由各源站负责。
