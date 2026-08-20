---
title: Firefly 仓库私有化指南：仅密钥电脑可上传/克隆，网站保持公开
published: 2026-08-20
description: "如何设置使项目只有拥有密钥的电脑才能上传和克隆，但不影响任意用户浏览网站。"
tags: ["Git", "GitHub", "SSH", "私有仓库", "权限管理"]
category: 使用教程
---

> **目标效果**
>
> - ✅ 只有配置了 SSH 密钥的电脑，才能 `git clone` / `git push` 本仓库
> - ✅ 任意访客无需任何凭据，都能用浏览器正常访问网站 https://shifulong7.github.io/firefly/
> - ✅ 自动部署（GitHub Actions）不受影响，推送后照常自动构建发布

---

## 1. 原理说明（为什么这样能实现）

GitHub Pages 站点是独立托管的**公开站点**，它和仓库的"可见性"是两个独立概念：

| 仓库可见性 | 任何人可 `git clone` | 任何人可浏览 Pages 网站 | 谁能 `git push` |
| :--- | :---: | :---: | :--- |
| **公开（Public）** | ✅ 可以（无法阻止） | ✅ | 仅协作者 |
| **私有（Private）** | ❌ 必须通过密钥/令牌认证 | ✅ | 仅协作者 |

- 把仓库改为**私有**后，`clone / pull / push` 全部要求身份认证，没有配置密钥的电脑一律被拒
- 但 **Pages 网站照常对所有人开放**，访客浏览器访问 URL 不需要登录、不需要密钥
- 免费账户限制：每个 GitHub 账户最多允许 **1 个私有仓库**启用 Pages（本项目正好占用这一个，无冲突）

---

## 2. 总体流程（只需 3 步）

```
第 1 步：每台要授权的电脑，生成 SSH 密钥并绑定到 GitHub 账户
第 2 步：把仓库 shifulong7/firefly 改为私有
第 3 步：验证（授权电脑可操作 / 未授权电脑被拒 / 访客可浏览网站）
```

---

## 3. 第 1 步：为每台授权电脑配置 SSH 密钥

> 每台电脑生成**独立的**密钥，把各自的公钥都添加到同一个 GitHub 账户。
> 这样"拥有密钥的电脑"就能通过账户认证访问仓库。

### 3.1 在电脑上生成密钥（Windows PowerShell）

```powershell
ssh-keygen -t ed25519 -C "firefly@电脑名" -f "$env:USERPROFILE\.ssh\firefly_ed25519"
```

- 一路回车即可（建议设置一个口令 passphrase，多一层保护）
- 生成两个文件：
  - `firefly_ed25519` —— **私钥**，绝不能泄露、绝不能提交到任何仓库
  - `firefly_ed25519.pub` —— **公钥**，下一步要添加到 GitHub

### 3.2 查看公钥内容

```powershell
Get-Content "$env:USERPROFILE\.ssh\firefly_ed25519.pub"
```

复制输出的整行内容（`ssh-ed25519 AAAA... firefly@电脑名`）。

### 3.3 把公钥添加到 GitHub 账户

1. 打开 https://github.com/settings/keys
2. 点击 **New SSH key**（新建 SSH 密钥）
3. **Title** 填一个易识别的名字（如 `Home-PC`、`Work-Laptop`，多台电脑靠这个区分）
4. **Key type** 选 `Authentication Key`（认证密钥）
5. 把公钥粘贴到 **Key** 输入框，点 **Add SSH key** 保存

### 3.4 把本地仓库远程地址切换为 SSH（推荐）

当前远程地址是 HTTPS 格式，建议改为 SSH（SSH 认证即密钥认证，最贴合本方案）：

```powershell
git remote set-url origin git@github.com:shifulong7/firefly.git
```

验证是否配置成功：

```powershell
ssh -T git@github.com
# 成功会显示：Hi shifulong7! You've successfully authenticated, but GitHub does not provide shell access.
```

> 不想切换也可以：HTTPS 方式在私有仓库下需要"用户名 + 个人访问令牌（PAT）"，见 [第 6 节](#6-htps-方式的替代方案) 。

---

## 4. 第 2 步：把仓库改为私有

1. 打开仓库设置页：https://github.com/shifulong7/firefly/settings
2. 滚动到页面最底部的 **Danger Zone（危险区域）**
3. 点击 **Change repository visibility（更改仓库可见性）** → **Change to private（改为私有）**
4. 按提示输入仓库名 `shifulong7/firefly` 确认

改私有之后：

- ✅ 网站 https://shifulong7.github.io/firefly/ **继续正常访问**，不受任何影响
- ✅ GitHub Actions 自动构建部署**照常工作**（部署走 GitHub 内部通道，无需密钥）
- ❌ 未配置密钥的电脑，`clone / pull / push` 全部失败
- ⚠️ 已配置密钥的电脑不受影响；HTTPS 方式下若系统缓存了凭据，也能继续推送

---

## 5. 第 3 步：验证效果

### 5.1 授权电脑（已配置密钥）—— 应成功

```powershell
git clone git@github.com:shifulong7/firefly.git   # 克隆成功
git push origin master                            # 推送成功
```

### 5.2 未授权电脑 —— 应被拒绝

```powershell
git clone https://github.com/shifulong7/firefly.git
# 报错：Repository not found（或要求输入用户名/密码，且无凭据无法通过）
```

GitHub 出于安全考虑，对无权限访问的私有仓库统一显示 `Repository not found`，不会暴露仓库是否存在。✅ 达到目的。

### 5.3 任意访客浏览器 —— 应正常浏览

直接打开 **https://shifulong7.github.io/firefly/**，无需登录即可浏览全部页面。✅

---

## 6. HTTPS 方式的替代方案（不想用 SSH 时）

私有仓库用 HTTPS 克隆/推送时，密码已不再支持，必须使用**个人访问令牌（PAT）**：

1. 打开 https://github.com/settings/tokens 生成令牌（勾选 `repo` 权限，即"完整控制私有仓库"）
2. 克隆时按提示输入：用户名为 GitHub 用户名，密码粘贴该令牌
3. Windows 凭据管理器会记住令牌，之后无需重复输入

> 注意：PAT 就是"密码"，泄露等同密钥泄露，用完可随时在 GitHub 上吊销。

---

## 7. 日常管理与注意事项

| 场景 | 操作 |
| :--- | :--- |
| 新电脑要授权 | 在新电脑重复 [第 3 节](#3-第-1-步为每台授权电脑配置-ssh-密钥)，把新公钥加到账户 |
| 某台电脑不再授权 | https://github.com/settings/keys → 删除对应的 SSH 密钥，该电脑立即失效 |
| 怀疑密钥泄露 | 立即删除对应密钥并重新生成；私钥不要发给任何人 |
| 想限制某台电脑只读 | 也可改用 **Deploy Key**（仓库级密钥）：Settings → Deploy keys，只勾读权限 |
| 更换 GitHub 账户/电脑 | 旧密钥删掉，新电脑重新配置 |

其他说明：

- **免费额度**：私有仓库的 Actions 免费额度为 **2000 分钟/月**，本项目一次完整构建约 3~5 分钟，绰绰有余
- **搜索引擎收录**：私有仓库的 Pages 站点**不会被搜索引擎收录**（GitHub 平台策略）。如果将来需要 SEO，需保持仓库公开
- **公开时期被克隆过的副本**：那些本地副本仍能查看旧内容，但无法再拉取更新、也无法推送（无权限）
- **密钥（私钥）红线**：`id_ed25519` / `firefly_ed25519` 等私钥文件绝不能提交进仓库（本项目 `.gitignore` 已默认忽略，注意不要手动强制添加）

---

## 8. 如何配置"指定电脑白名单"（只有规定的电脑能上传）

### 8.1 核心机制：白名单 = 账户里的 SSH 密钥列表

- GitHub 认证的最小单位是**密钥**，没有"电脑"这个概念
- 你把哪些电脑的公钥添加到 GitHub 账户，哪些电脑就进入了白名单：
  - 公钥在账户里 → 该电脑可以 clone / push
  - 公钥不在账户里 → 该电脑一律被拒（`Permission denied` / `Repository not found`）
- **一台电脑 = 一把独立密钥**，添加时用 Title 标注电脑名，白名单一目了然

### 8.2 白名单管理操作表

| 操作 | 步骤 | 生效时间 |
| :--- | :--- | :--- |
| **加入白名单**（新电脑） | 该电脑按[第 3 节](#3-第-1-步为每台授权电脑配置-ssh-密钥)生成密钥 → 公钥添加到 https://github.com/settings/keys | 立即生效 |
| **查看白名单** | 打开 https://github.com/settings/keys ，列表里每一条密钥 = 一台授权电脑（Title 即电脑名） | — |
| **踢出白名单**（某电脑） | 在 https://github.com/settings/keys 删除该电脑对应的密钥 | 该电脑立即失去上传/克隆权限 |
| **更换电脑** | 新电脑生成新密钥加入白名单，旧电脑的密钥删除 | — |

> 要点：**删除密钥就是踢出白名单**。只要把密钥列表控制好，上传权限就完全由你规定，与 GitHub 仓库页面无关。

### 8.3 一台电脑有多把密钥时指定密钥测试（可选）

默认 `git`/`ssh` 会使用 `~/.ssh/id_ed25519`（或 `ssh-agent` 中第一个密钥）。要针对某把特定密钥测试：

```powershell
ssh -i "$env:USERPROFILE\.ssh\firefly_ed25519" -T git@github.com
```

也可以在 `~/.ssh/config` 中为 `github.com` 固定使用哪把密钥：

```
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/firefly_ed25519
    IdentitiesOnly yes
```

---

## 9. 如何检测某台电脑能否上传

### 9.1 三步检测法（全部在目标电脑上执行）

```powershell
# 第 1 步：检测这台电脑的 SSH 认证是否有效（密钥是否在白名单）
ssh -T git@github.com
# ✅ 显示：Hi shifulong7! You've successfully authenticated, but GitHub does not provide shell access.
# ❌ 显示：Permission denied (publickey).

# 第 2 步：检测对该仓库的访问权（私有仓库未授权会直接拒绝）
git ls-remote git@github.com:shifulong7/firefly.git
# ✅ 输出类似：33173be...  refs/heads/master  → 有仓库读权限
# ❌ 报错：Repository not found. 或 Permission denied  → 不在白名单

# 第 3 步：模拟推送（不真正上传任何内容），检测上传权限
cd 任意已克隆的仓库目录
 git push --dry-run origin master
# ✅ 显示：Everything up-to-date（或列出将推送的提交）→ 可以上传
# ❌ 报错：Permission to shifulong7/firefly.git denied → 不能上传
```

### 9.2 检测结果判断表

| 第 1 步 `ssh -T` | 第 2 步 `ls-remote` | 第 3 步 `push --dry-run` | 结论 |
| :---: | :---: | :---: | :--- |
| ✅ | ✅ | ✅ | **该电脑可以上传** 🎉 |
| ✅ | ❌ | — | 密钥有效，但不是本仓库协作者（账户无仓库权限） |
| ❌ | — | — | 该电脑**不在白名单**，完全无权限 |

> 快捷方式：第 1、2 步足以判断"能不能访问"；第 3 步是上传权限的最终确认。日常只需跑第 1 步即可。

### 9.3 上传记录审计（谁在什么时候推了什么）

- 网页端：仓库 **Commits** 页面可查看每次提交的作者、时间、内容
- 本地：`git log --pretty="%h | %an | %cn | %s"` 查看提交者信息
- 注意：commit 里填写的作者名可以伪造，严格审计需使用 GitHub 组织版的 Audit log（审计日志）

---

## 10. 一句话总结

> **仓库设为私有 + 每台电脑配置 SSH 密钥 = 只有密钥电脑能上传/克隆；GitHub Pages 站点天然公开，访客浏览不受影响。**
