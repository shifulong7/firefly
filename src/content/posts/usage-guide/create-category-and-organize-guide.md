---
title: 新建"使用教程"分类并把文档归入其中（操作步骤）
published: 2026-08-20
description: "记录在 Firefly 博客中新建分类、归并文档并本地运行测试的完整步骤，含验证清单。"
tags: ["Firefly", "分类", "Git", "本地测试"]
category: 使用教程
---

> 本文记录一次完整的实操：在 Firefly 博客中新建 **使用教程** 分类、把《Firefly 仓库私有化指南》归入该分类，并在本地启动开发服务器进行测试。本文与《Firefly 仓库私有化指南》同属 **使用教程** 分类。

## 1. 本次任务目标

1. 新建一个分类，名称为 **使用教程**
2. 将《Firefly 仓库私有化指南》文档放入该分类
3. 本地运行开发服务器进行测试
4. 将以上步骤写成文档（即本文），同样放入该分类

## 2. 项目规定：Firefly 的分类机制（先了解再动手）

- 所有文章存放在 `src/content/posts/` 目录，**支持子目录**组织文章与资源（如 `posts/guide/`）
- **分类不是独立集合**：文章的 frontmatter 中通过 `category` 字段声明所属分类，如 `category: git使用`
- 分类页面 `/categories` 会**自动聚合**所有 `category` 相同（或为空）的文章，无需手动注册
- 内容集合的 schema 定义在 `src/content.config.ts`（`category` 为可选字符串字段，为空则不归类）

> 结论：**"新建分类" = 在文章的 frontmatter 里写 `category: 使用教程`**，不需要改任何配置文件或路由。

## 3. 操作步骤

### 3.1 新建分类目录（用于存放该类文章）

```powershell
New-Item -ItemType Directory -Force -Path "src\content\posts\usage-guide"
```

- 目录名 `usage-guide` 与分类名 `使用教程` 可以不同，目录仅用于文件组织，分类名以 frontmatter 为准
- 子目录中的图片等资源可直接放在同目录下

### 3.2 创建分类文章（归并文档）

文件：`src/content/posts/usage-guide/private-repo-key-access-guide.md`

frontmatter 示例：

```yaml
---
title: Firefly 仓库私有化指南：仅密钥电脑可上传/克隆，网站保持公开
published: 2026-08-20
description: "如何设置使项目只有拥有密钥的电脑才能上传和克隆，但不影响任意用户浏览网站。"
tags: ["Git", "GitHub", "SSH", "私有仓库", "权限管理"]
category: 使用教程
---
```

字段说明（与 `posts/guide/index.md` 中的 Front-matter 约定一致）：

| 字段 | 说明 |
| :--- | :--- |
| `title` | 文章标题，自动显示在文章页头部 |
| `published` | 发布日期（`YYYY-MM-DD`），用于排序 |
| `description` | 简短描述，显示在首页文章卡片上 |
| `tags` | 文章标签，用于标签页聚合 |
| `category` | **文章分类，本文属于 `使用教程`** |

正文即文档全文（Markdown 表格、代码块、标题锚点等均按博客渲染）。

### 3.3 编写操作步骤文章（即本文）

同样在 `src/content/posts/usage-guide/` 下创建本文，frontmatter 中的 `category: 使用教程`，实现"步骤文档也在该分类中"。

### 3.4 本地运行测试

```powershell
pnpm dev
```

浏览器打开 http://localhost:4321 进行验证。

## 4. 验证清单（本地测试要点）

| # | 验证项 | 预期结果 | 方法 |
| :---: | :--- | :--- | :--- |
| 1 | 启动无报错 | 终端无 error，提示 Local 地址 | 观察 `pnpm dev` 输出 |
| 2 | 首页文章列表 | 出现《Firefly 仓库私有化指南》与本文 | 访问 `/` |
| 3 | 分类页 | 出现 **使用教程** 分类，且所有文档都在其下 | 访问 `/categories` |
| 4 | 文章详情页 | 标题、表格、代码块、TOC 渲染正常 | 打开文章 URL |
| 5 | 分类聚合 | 点击"使用教程"分类只显示该类文章 | 分类页内点击分类 |

## 5. 注意事项

- **slug**：不设置 `slug` 时 URL 使用文件名（如 `/posts/git-use/private-repo-key-access-guide`），建议文件名用英文与连字符
- **draft**：文章默认 `draft: false` 才显示；写草稿可临时设为 `true`
- **分类名改动**：如需改名，只需批量修改文章的 `category` 字段，分类页自动更新
- **文档双备份**：原指南同时保留在 `docs/private-repo-key-access-guide.md`（仓库文档）与本文分类中（博客文章），内容保持一致

## 6. 一句话总结

> **Firefly 新建分类 = 写文章 + frontmatter 里声明 `category` 分类名，零配置文件改动；本地用 `pnpm dev` 即可验证。**
