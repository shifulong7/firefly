---
title: 打赏功能：如何关闭与替换为自己的内容
published: 2026-08-20
description: "Firefly 打赏功能的完整配置指南：如何彻底关闭打赏、关闭文章内打赏按钮，以及如何替换为支付宝/微信收款码、赞助链接等自己的内容。"
tags: ["Firefly", "打赏", "配置", "使用教程"]
category: 使用教程
---

> 本文介绍 Firefly 打赏（Sponsor）功能的两个核心操作：**如何关闭打赏**，以及**如何把打赏替换为自己的内容**。所有修改只涉及配置文件，无需改动组件代码。

## 1. 打赏功能涉及的位置（先认识结构）

| 位置 | 文件 | 作用 |
| :--- | :--- | :--- |
| 打赏页面 | `/sponsor`（`src/pages/sponsor.astro`） | 展示打赏方式、打赏者列表、评论区 |
| 打赏配置 | `src/config/sponsorConfig.ts` | 所有打赏内容（标题、方式、收款码、打赏者） |
| 页面开关 | `src/config/siteConfig.ts` → `pages.sponsor` | 是否启用打赏页面（false 时页面返回 404 并自动隐藏导航菜单项） |
| 文章内按钮 | `src/pages/posts/[...slug].astro` | 文章详情页底部的"打赏 & 分享"按钮区 |

## 2. 如何关闭打赏

按"关闭程度"从彻底到局部，共 4 个开关：

### 2.1 彻底关闭（推荐：不想用打赏功能时）

修改 `src/config/siteConfig.ts`：

```ts
pages: {
    // 打赏页面开关
    sponsor: false,
    // ...其他页面开关保持不变
},
```

关闭后效果（联动）：

- `/sponsor` 页面直接返回 404
- 导航栏的"打赏"菜单项自动隐藏
- 文章详情页底部的打赏按钮自动隐藏（该区域降级为纯分享按钮）

### 2.2 只关闭文章详情页底部的打赏按钮（保留打赏页面）

修改 `src/config/sponsorConfig.ts`：

```ts
// 是否在文章详情页底部显示打赏按钮
showButtonInPost: false,
```

### 2.3 关闭单个打赏方式

`methods` 数组中每一项都有 `enabled` 开关，改为 `false` 即不在页面显示：

```ts
{
    name: "支付宝",
    qrCode: "/assets/images/sponsor/alipay.png",
    enabled: false, // false = 该方式不在打赏页显示
},
```

### 2.4 关闭打赏者列表 / 评论区

```ts
// 是否显示打赏者列表
showSponsorsList: false,
// 是否显示评论区（需评论系统已启用，见 commentConfig.ts）
showComment: false,
```

## 3. 如何替换为自己的内容

### 3.1 页面标题与描述（默认使用 i18n 翻译）

```ts
title: "赞助支持",          // 留空 "" 则使用 i18n 默认文案
description: "你的支持是我持续创作的最大动力！",
usage: "您的打赏将用于服务器维护、内容创作和功能开发。",
```

- `usage` 会显示在打赏页的提示框中
- `title` / `description` 留空时使用 `src/i18n/languages/zh_CN.ts` 中的 `sponsorTitle` / `sponsorDescription` 翻译

### 3.2 替换打赏方式（收款码或链接）

`methods` 数组每项支持两种展示形式，**二选一**：

```ts
methods: [
    // 形式一：扫码收款（二维码图片）
    {
        name: "支付宝",
        icon: "fa7-brands:alipay",           // 图标（Iconify 名称）
        qrCode: "/assets/images/sponsor/alipay.png", // 收款码图片，放在 public 目录下
        link: "",                             // 留空则只显示二维码
        description: "使用 支付宝 扫码打赏",
        enabled: true,
    },
    // 形式二：跳转链接（如 Ko-fi、爱发电、PayPal）
    {
        name: "Ko-fi",
        icon: "simple-icons:kofi",
        qrCode: "",                            // 留空则只显示跳转按钮
        link: "https://ko-fi.com/你的用户名",
        description: "Buy a Coffee",
        enabled: true,
    },
],
```

替换自己的收款码步骤：

1. 把自己的支付宝/微信收款码图片放到 `public/assets/images/sponsor/` 目录
2. 修改对应 `qrCode` 路径（如 `/assets/images/sponsor/alipay.png`）
3. 也可直接删除不需要的方式（删掉整个对象）

### 3.3 替换打赏者列表

```ts
sponsors: [
    { name: "张三", avatar: "https://.../头像.png", amount: "¥50", date: "2026-08-01" },
    { name: "匿名用户", amount: "¥20", date: "2026-08-02" }, // 无头像则显示首字母
],
```

- `avatar` 支持网络图片或 `public` 目录图片
- 不需要该列表时设 `showSponsorsList: false`

## 4. 修改后的验证

```powershell
# 方式一：本地运行查看效果
pnpm dev
# 访问 http://localhost:4321/firefly/sponsor/ 和任意文章页底部

# 方式二：类型与内容检查
pnpm check
```

验证要点：

| 检查项 | 预期 |
| :--- | :--- |
| 关闭后访问 `/sponsor` | 404 |
| 关闭后导航栏 | 无"打赏"菜单 |
| 替换收款码后 | 打赏页显示新二维码图片 |
| 文章底部 | 按配置显示打赏按钮或分享按钮 |

## 5. 注意事项

- **收款码图片必须放在 `public` 目录**（如 `public/assets/images/sponsor/`），路径以 `/` 开头
- `pages.sponsor: false` 是"总开关"，同时控制页面、导航和文章内按钮；`showButtonInPost` 只影响文章内按钮
- 修改配置后 dev 服务器热更新即可生效，无需重启
- 若同时想改按钮/页面的默认文案（如"打赏支持"），可编辑 `src/i18n/languages/zh_CN.ts` 中 `sponsor*` 开头的翻译项

## 6. 一句话总结

> **关闭打赏：`siteConfig.pages.sponsor = false`（彻底）或 `sponsorConfig` 各开关（局部）；替换内容：改 `sponsorConfig.ts` 的标题、收款码、链接与打赏者列表即可。**
