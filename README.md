# TVBox 点播接口

4 个 AppleCMS 点播站整合成一份 TVBox 配置。

## 填这个地址

```
https://cdn.jsdelivr.net/gh/run0102/tvbox@v4/config.json
```

备用（同一份内容）：

```
https://fastly.jsdelivr.net/gh/run0102/tvbox@v4/config.json
```

> 用**标签地址 `@v4`**，不是 `@main`。jsDelivr 对 `@main` 的新鲜度缓存很长，
> 改完配置后 `@main` 可能几小时还是旧版，连 `?v=时间戳` 都绕不过；
> 标签地址首次访问即回源，能立刻拿到当前内容。换配置时发新标签（v4、v5…），地址里的版本号跟着改。

## 包含什么

| 站名 | 接口 | 线路 |
|---|---|---|
| 360资源 | `360zy.com` | 单线路 m3u8 直链，最稳，点开即播 |
| 茅台资源 | `mtzy.me` | 单线路 m3u8 直链 |
| 极速资源 | `www.jisuzy.com` | 双线路：网页源 + m3u8 |
| 金鹰资源 | `jinyingzy.com` | 双线路：网页源 + m3u8 |

## 两个关键字段（改配置时必须守住）

**1. `type` 必须是 `1`**

| 值 | 解析器 | 适用接口 |
|---|---|---|
| 0 | XML | `api.php/provide/vod**/at/**`（带 `/at/` 的老接口） |
| 1 | JSON | `api.php/provide/vod`（本项目 4 个站都是这种） |

写成 0 的症状：站点名能显示，但**分类是空的、没有海报墙、搜索搜不出东西**。

**2. `categories` 白名单不能省**

AppleCMS 的 `class` 里，顶层分类（电影 / 连续剧 / 综艺 / 动漫）`type_pid=0`，
用 `?ac=detail&t=<顶层id>` 请求**返回 0 条**——数据全挂在子分类（动作片 / 喜剧片…）下。
TVBox 会把 `class` 里每个 id 都做成分类按钮，不配白名单的话，点开第一个分类就是空白页。

白名单由 `probe_categories.py` 实测生成，只保留真正有数据的分类名。

字段名还有一个坑：可筛选是 `filterable`，不是 `filter`。

## 播放线路说明

- **360资源 / 茅台资源**：单线路，点开即播。
- **极速资源 / 金鹰资源**：接口返回两条线路，第一条是网页播放页（需要第三方解析器，
  公共解析器大多已失效），第二条才是 m3u8 直链。点开报错就**切到第 2 条线路**。
  这两站的 m3u8 是 AES-128 加密，TVBox 内置播放器支持解密。

## 重新生成

```bash
python3 probe_categories.py                       # 站方改版后重探分类（写 cat_probe.json）
python3 build_tvbox_config.py --base <托管根地址>  # 生成 config.json / subscription.json
```

生成器带三道自检：`rules` 必须是数组、`type` 必须与 api 格式匹配、每站必须有分类白名单。
自检不过就不出文件，避免把错的推上去。

## 文件

| 文件 | 说明 |
|---|---|
| `config.json` | 完整配置（4 站） |
| `subscription.json` | 仅站点，给「订阅」入口 |
