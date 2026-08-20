---
title: 音乐播放功能：如何使用与增加音乐
published: 2026-08-20
description: "Firefly 音乐播放器的使用指南：如何增加本地音乐、配置在线歌单（Meting）、添加封面与歌词，以及播放行为设置。"
tags: ["Firefly", "音乐", "配置", "使用教程"]
category: 使用教程
---

> 本文介绍 Firefly 音乐播放器的完整用法：**如何增加音乐**（本地文件或在线歌单）、配置封面与歌词，以及音量、播放模式等行为设置。所有修改只涉及配置文件与文件放置，无需改动组件代码。

## 1. 音乐播放器概览（先认识结构）

| 项目 | 说明 |
| :--- | :--- |
| 配置文件 | `src/config/musicConfig.ts`（`musicPlayerConfig`） |
| 音乐文件目录 | `public/assets/music/`（音频文件 + `cover/` 封面子目录） |
| 播放器入口 | 导航栏 + 侧边栏（可分别开关） |
| 数据来源模式 | `"local"` 本地音乐列表（默认） / `"meting"` 在线歌单 API |

当前配置为 **local 模式**，默认播放列表里有一首示例歌曲（`public/assets/music/使一颗心免于哀伤-哼唱.mp3`）。

## 2. 如何增加音乐（本地模式，推荐）

### 2.1 准备音乐文件

1. 把音频文件（推荐 `.mp3`，浏览器兼容性最好）复制到 `public/assets/music/` 目录
2. 可选：把封面图片（`.webp` / `.jpg` / `.png`）放到 `public/assets/music/cover/` 目录

```
public/assets/music/
├── 使一颗心免于哀伤-哼唱.mp3   ← 音频文件
└── cover/
    └── 109951169585655912.webp  ← 封面图片（可选）
```

### 2.2 在配置中添加歌曲

编辑 `src/config/musicConfig.ts`，在 `local.playlist` 数组中新增条目：

```ts
local: {
    playlist: [
        {
            name: "使一颗心免于哀伤",
            artist: "知更鸟 / HOYO-MiX / Chevy",
            url: "/assets/music/使一颗心免于哀伤-哼唱.mp3",
            cover: "/assets/music/cover/109951169585655912.webp",
            lrc: "",
        },
        // ↓ 新增的歌曲
        {
            name: "我的新歌",                    // 歌曲名（必填）
            artist: "演唱者",                     // 艺术家（必填）
            url: "/assets/music/我的新歌.mp3",     // 音频路径，以 / 开头（必填）
            cover: "/assets/music/cover/我的新歌封面.webp", // 封面（可选）
            lrc: "",                             // 歌词（可选，见 2.3）
        },
    ],
},
```

要点：

- `url` / `cover` 路径以 `/` 开头，相对于 `public` 目录
- 支持中文文件名（现有示例即为中文名），但建议使用英文文件名避免个别环境编码问题
- 想删除歌曲：直接删除对应条目即可；想调整顺序：调整数组元素顺序

### 2.3 添加歌词（可选）

`lrc` 字段支持两种写法：

```ts
// 写法一：歌词文件路径（把 .lrc 文件放到 public 目录任意位置）
lrc: "/assets/music/lrc/我的新歌.lrc",

// 写法二：直接填入 LRC 格式的歌词字符串
lrc: "[00:00.00]第一句歌词\n[00:05.00]第二句歌词",
```

歌词开关：`showLyrics: true` 时播放器显示歌词（默认关闭）。

## 3. 如何增加音乐（Meting 在线歌单模式）

不存放本地文件，直接播放在线平台歌单：

```ts
mode: "meting", // 切换为在线模式
meting: {
    api: "https://api.i-meto.com/meting/api?server=:server&type=:type&id=:id&r=:r",
    server: "netease",        // 平台：netease=网易云, tencent=QQ音乐, kugou=酷狗, baidu=百度
    type: "playlist",         // 类型：playlist=歌单, song=单曲, album=专辑, artist=艺术家, search=搜索
    id: "10046455237",        // 歌单/单曲 ID（网页端 URL 中的数字 ID）
    auth: "",                 // 认证 token（可选）
    fallbackApis: [           // 主 API 失败时的备用 API
        "https://api.injahow.cn/meting/?server=:server&type=:type&id=:id",
        "https://api.moeyao.cn/meting/?server=:server&type=:type&id=:id",
    ],
},
```

> 提示：Meting 模式依赖第三方 API 的可用性，若播放失败可尝试更换 `api` 或备用 API。本地模式不受网络影响，稳定性更佳。

## 4. 播放行为配置

```ts
// 默认音量 (0-1)
volume: 0.7,

// 播放模式：'list'=列表循环, 'one'=单曲循环, 'random'=随机播放
playMode: "list",

// 是否显示歌词
showLyrics: false,

// 是否在导航栏显示音乐播放器入口
showInNavbar: true,

// 是否在侧边栏显示音乐播放器组件
showInSidebar: true,
```

不需要播放器时：将 `showInNavbar` 和 `showInSidebar` 都设为 `false` 即可隐藏（不播放不加载音频，不影响性能）。

## 5. 修改后的验证

```powershell
# 本地运行查看效果
pnpm dev
# 打开 http://localhost:4321/firefly/ ，观察导航栏/侧边栏播放器，播放新加的歌曲

# 类型与内容检查
pnpm check
```

验证要点：

| 检查项 | 预期 |
| :--- | :--- |
| 新歌曲出现在播放列表 | 播放器列表可见新增条目 |
| 点击播放 | 音频正常出声，切歌正常 |
| 封面 | 显示新封面（未配置则显示默认占位） |
| 歌词（开启后） | 随播放进度滚动显示 |

## 6. 注意事项

- 音频文件**必须放在 `public` 目录**（如 `public/assets/music/`），路径以 `/` 开头
- 浏览器播放支持格式：`mp3`、`ogg`、`wav`（`flac` 支持不稳定），建议统一使用 `mp3`
- 音频文件建议压缩优化（如 320kbps 及以下），过大的文件会拖慢站点加载
- `mode` 切换为 `"meting"` 后，`local.playlist` 配置不再生效（反之亦然），两种配置可同时保留随时切换
- 播放器为全局组件，任意页面刷新后按配置恢复默认音量与播放模式

## 7. mflac 等加密格式如何处理（FAQ）

### 7.1 能直接部署吗？

**不能。** `.mflac` 是 QQ音乐客户端的加密音频格式（内部为加密的 FLAC），浏览器无法解码；`.ncm`（网易云）、`.kgm`（酷狗）、`.qmc*`（QQ音乐旧版）同理。必须先用解密工具转为标准格式才能放入播放列表。

### 7.2 操作步骤（解密 → 转码 → 部署）

```
第 1 步：解密 .mflac → .flac
第 2 步：转码 .flac → .mp3（推荐）
第 3 步：放入 public/assets/music/ 并配置（见第 2 节）
```

**第 1 步：解密** —— 使用开源工具 Unlock Music（浏览器端本地解密，文件不上传服务器）：

1. 打开 Unlock Music（在线版或自建，项目地址 `github.com/unlock-music/unlock-music`）
2. 把 `.mflac` 文件拖入页面
3. 工具自动识别并解密 → 下载得到标准 `.flac` 文件

**第 2 步：转码为 mp3**（FFmpeg）：

```powershell
ffmpeg -i 歌曲.flac -b:a 320k 歌曲.mp3
```

> 为什么不直接用解密后的 FLAC？解密产物是 FLAC，而 **Safari 对 FLAC 支持不稳定**；mp3 兼容性最好且体积更小。

**第 3 步：部署** —— 把 mp3 复制到 `public/assets/music/`，按[第 2 节](#2-如何增加音乐本地模式推荐)在 `local.playlist` 添加条目，刷新页面即可播放。

### 7.3 合规提醒

仅解密和部署**你拥有版权或已获授权**的音乐文件。

## 8. 一句话总结

> **增加音乐：文件放入 `public/assets/music/` + 在 `musicConfig.ts` 的 `local.playlist` 添加条目（或切到 `meting` 模式填歌单 ID）；全部只改配置和放文件，无需动代码。**
