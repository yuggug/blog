---
title: "零成本博客搭建"
date: 2026-08-03
tags: ["技术"]
summary: "记一次博客搭建"
---

# 博客搭建完整保姆级教程

> 方案组合：**GitHub 仓库（存文章）+ Cloudflare Pages（云端构建）+ Cloudflare CDN（全球加速）+ Spaceship 购买的 103254.xyz（Cloudflare 做解析）**
> 特点：**全程只用浏览器，不装 git、不装 Hugo、不敲命令行**。写好的 md 文件直接拖到 GitHub 网页上，剩下的全自动。
> 适用：Windows / macOS / 任何能上网的设备 · 2026 年 8 月更新

---

## 〇、先看全局：这套方案是怎么运转的

### 每个角色负责什么

| 角色 | 干什么 | 花钱吗 |
|---|---|---|
| **GitHub 仓库** | 仓库：存放你的博客配置和所有 md 文章 | 免费 |
| **Cloudflare Pages** | 组装厂：检测到 GitHub 有新文件，就在云端自动跑 Hugo 把 md 变成网页 | 免费 |
| **Cloudflare CDN** | 快递网：把生成的网页缓存到全球节点，谁访问都就近取货，快且带 HTTPS | 免费 |
| **103254.xyz 域名** | 门牌号：别人通过这个地址找到你的博客 | 仅域名年费 |
| **Cloudflare 解析** | 导航员：告诉全世界"103254.xyz 对应 Cloudflare 的哪些服务器" | 免费 |

### 数据流动图

```
你写好一篇 md 文章（记事本就行）
        │  拖到 GitHub 网页上传（1 分钟）
        ▼
GitHub 仓库（存 hugo.yaml 配置 + content 文章）
        │  Cloudflare Pages 自动检测到更新
        ▼
Cloudflare Pages 云端构建（自动下载主题 + 跑 Hugo，生成静态网页）
        │
        ▼
Cloudflare CDN（全球节点缓存 + 免费 HTTPS + 防攻击）
        │
        ▼
访客打开 https://103254.xyz 看到你的博客
```

### 总花费与总时间

- **花费**：只有域名年费（.xyz 首年通常几美元）。托管、构建、CDN、SSL 证书全部 0 元。
- **备案**：**不需要 ICP 备案**（域名在境外注册商 + 网站托管在 Cloudflare 海外节点）。
- **时间**：动手操作约 1 小时；之后等 NS 解析生效（通常 30 分钟 ~ 24 小时，最慢 48 小时），等待期间可以继续配置，互不耽误。
- **以后发文章**：每次只需 3 步、约 1 分钟。

### 操作顺序总览（照这个顺序做，不会返工）

```
① GitHub 注册 + 建仓库 + 网页创建 hugo.yaml + 传第一篇文章
② Cloudflare 注册 + 添加域名，拿到 2 条 NS 地址
③ 去 Spaceship 把 NS 改成 Cloudflare 的（关键一步）
④ 等待期间：Cloudflare Pages 连接 GitHub，配置构建，先让 blog.pages.dev 能访问
⑤ Pages 绑定 103254.xyz（含 www），SSL 自动签发
⑥ 确认 DNS 记录是橙色云朵（CDN 已开）
⑦ 最终验收，上线完成
```

---

# 第一部分：一次性初始化（只做一次）

## Step 1：注册 GitHub 账号（已有可跳过）

1. 打开 https://github.com → 右上角 **Sign up**。
2. 依次输入：邮箱（收得到验证邮件的）、密码、用户名（建议纯英文，比如 `my103254`，以后仓库地址会带这个名字）。
3. 完成人机验证 → 点 **Create account** → 去邮箱点验证链接。
4. **怎么确认做对了**：登录后右上角能看到你的头像，就说明账号好了。

> 小提醒：记住注册邮箱和密码，后面 Cloudflare 连接 GitHub 时还要登录一次。

## Step 2：创建博客仓库

1. 登录 GitHub 后，点右上角 **+** 号 → **New repository**（新建仓库）。
2. 按下面填：

```
Repository name（仓库名）:   blog        ← 必须英文，别用中文
Description（描述）:         我的博客     ← 可不填
Public 还是 Private:         选 Public（公开）
Add a README file:          不要勾
.gitignore / License:       都不选
```

3. 点绿色按钮 **Create repository**。
4. **怎么确认做对了**：页面跳到 `https://github.com/你的用户名/blog`，能看到一个空仓库（提示你创建文件）。

## Step 3：网页上创建配置文件 hugo.yaml（复制粘贴即可）

Hugo 博客的全部"程序设置"都在这一个文件里。

1. 在仓库页面点绿色按钮 **Add file** → **Create new file**。
2. 顶部 "Name your file..." 输入框里输入：`hugo.yaml`
3. 下方大文本框里**完整粘贴**以下内容（一个字符都别少；缩进必须用空格，直接复制粘贴就对了，千万别手敲）：

```yaml
baseURL: "https://103254.xyz/"
languageCode: "zh-cn"
title: "103254 的博客"
theme: "papermod"
hasCJKLanguage: true
enableRobotsTXT: true
params:
  env: production
  defaultTheme: auto
  ShowReadingTime: true
  ShowPostNavLinks: true
  ShowBreadCrumbs: true
  ShowCodeCopyButtons: true
  homeInfoParams:
    Title: "你好，我是 103254"
    Content: "这是我的博客，记录生活与学习。"
```

4. 滚动到页面底部 **Commit changes** 区域 → 直接点绿色按钮 **Commit new file**（提交信息默认即可）。
5. **怎么确认做对了**：仓库文件列表里能看到 `hugo.yaml` 这个文件。

> 说明：`baseURL` 必须填最终域名 `https://103254.xyz/`，这样生成的所有页面链接才是你的域名。`theme: "papermod"` 指定使用 PaperMod 主题（主题文件本身不在你的仓库里，构建时由云端自动下载，见 Step 8）。

## Step 4：传第一篇文章（顺便建好文章目录）

文章必须放在 `content/post/` 目录下。我们用网页直接创建第一篇文章，目录就自动建好了。

1. 仓库页面 → **Add file** → **Create new file**。
2. 文件名输入框里输入（**输入斜杠 `/` 时会自动变成文件夹**）：

```
content/post/hello-world/index.md
```

3. 下方文本框粘贴以下内容：

```markdown
---
title: "你好，世界"
date: 2026-08-03
tags: ["随笔"]
summary: "这是我的第一篇文章"
---

这是我的第一篇文章。

Markdown 常用写法都支持：**加粗**、[链接](https://example.com)、列表：

- 第一点
- 第二点
```

4. 底部 **Commit new file** 提交。
5. **怎么确认做对了**：仓库里出现 `content` 文件夹，点进去有 `post` → `hello-world` → `index.md`。

> 文章开头的 `---` 包起来的部分叫 front matter（文章的"身份证"），每篇文章都必须有，其中 `title` 和 `date` 最重要。

## Step 5：注册 Cloudflare 并添加域名，拿到 2 条 NS

1. 打开 https://dash.cloudflare.com → **Sign up** 注册（免费）→ 验证邮箱后登录。
2. 登录后首页点 **Add a site**（添加站点）→ 输入 `103254.xyz` → **Continue**。
3. 选套餐：拉到最下面选 **Free（免费）** → **Continue**。
4. Cloudflare 会"扫描 DNS 记录"：因为域名还没指向 Cloudflare，这里可能什么都没有，**直接 Continue** 即可。
5. 页面会显示 **Cloudflare nameservers（名称服务器）**，就两条，长这样：

```
xxxx.ns.cloudflare.com
yyyy.ns.cloudflare.com
```

**把这两条完整复制下来，存到记事本，下一步马上要用。这个页面先别关。**

- **怎么确认做对了**：页面明确写着 "Cloudflare nameservers" 并列出两条地址。

## Step 6：去 Spaceship 把 NS 改成 Cloudflare 的（最关键一步）

这一步的作用：让 Cloudflare 全权接管 103254.xyz 的解析。

1. 打开 https://www.spaceship.com → 登录 → 进入 **Domains**（域名列表）→ 点你的 `103254.xyz`。
2. 找到 **DNS settings / Nameservers（名称服务器）** 区域 → 点 **Manage / Edit**。
3. 当前默认是 Spaceship 自己的 NS（类似 `ns1.spaceship.com`、`ns2.spaceship.com`）。把模式切换为 **Custom nameservers（自定义名称服务器）**：
   - 删掉原有的两条；
   - 把 Step 5 复制的 Cloudflare 两条 NS **逐条**填进去（一条一个输入框，别把两条填进同一框）。
4. 点 **Save / Save changes** 保存。

⚠️ 三个注意事项：

- 如果之前在 Spaceship 开过 **DNSSEC**，先把它**关掉**再改 NS，否则解析会中断。
- 两条 NS 一定要和 Cloudflare 给的**一字不差**。
- 保存后**不要**在 24 小时内反复修改。

- **怎么确认做对了**：Spaceship 页面显示的 NS 已经变成 `xxx.ns.cloudflare.com` 两条。

## Step 7：等待 NS 生效（期间直接去做 Step 8，不冲突）

1. 回到 Cloudflare 那个"改 NS"页面，点 **I've completed the changes / Check nameservers**。
2. NS 全球生效需要时间：**通常 30 分钟 ~ 24 小时，最慢 48 小时**。Cloudflare 生效后会给你的注册邮箱发一封 "Active" 邮件，控制台站点状态也会从 **Pending** 变成 **Active**。
3. 想随时查进度：打开 https://dnschecker.org → 输入 `103254.xyz` → 类型选 **NS** → Search。当世界各地的结果都变成 Cloudflare 那两条时，就是生效了。

**等待期间，继续做 Step 8～Step 9（Pages 构建不依赖 NS 生效），效率最高。**

## Step 8：Cloudflare Pages 连接 GitHub，配置云端构建（核心步骤）

1. Cloudflare 控制台 → 左侧菜单 **Workers & Pages** → 点上方 **Create** → 切到 **Pages** 标签 → 点 **Connect to Git**。
2. 点 **GitHub** → 跳转 GitHub 授权页 → **Install & Authorize** → 授权范围选 **Only select repositories** → 只勾你的 `blog` 仓库 → **Install**。
3. 回到 Cloudflare，选中 `blog` 仓库 → **Begin setup**。
4. 构建设置**照下面逐项填写（不要用默认值）**：

```
Project name（项目名）:        blog        → 会生成临时域名 blog.pages.dev
Production branch（生产分支）:  main        → 默认就是，不改
Framework preset（框架预设）:   选 Hugo
```

5. 点一下 **Environment variables（环境变量）** 展开，添加**一条**环境变量（重要，别漏）：

```
变量名：HUGO_VERSION
变量值：0.139.4
```

> 为什么要加这个：Cloudflare 默认的 Hugo 版本很老（0.54），而 PaperMod 主题要求新版 Hugo，不加这条构建大概率报错。

6. **Build command（构建命令）** 粘贴这一整行：

```
git clone --depth 1 https://github.com/adityatelange/hugo-papermod.git themes/papermod && hugo --gc --minify
```

> 这行命令的意思：构建时先从 GitHub 下载 PaperMod 主题（你的仓库里只放配置和文章，不装几百个主题文件），再运行 Hugo 生成网站。这就是"云端构建"——你的电脑什么都不用装。

7. **Build output directory（输出目录）** 填：

```
public
```

8. 点 **Save and Deploy** → 等 1~3 分钟。
9. **怎么确认做对了**：Deployments 列表最新一条变成绿色 ✅，页面顶部显示 `blog.pages.dev` 链接，点开能看到博客首页（有"你好，我是 103254"卡片和第一篇文章）。

**如果构建失败（红色 ❌）**：点那条失败记录 → **View build logs** 看日志：

- 日志里是 `clone` / 网络超时 → 点 **Retry deployment** 重试一次通常就好；
- 日志里出现 `yaml` / `config` 报错 → 回 Step 3 检查 hugo.yaml 是否粘贴完整、缩进是否被改动；
- 日志里出现 `hugo: command not found` 或主题兼容性报错 → 检查 HUGO_VERSION 环境变量是否加了。

## Step 9：绑定 103254.xyz 域名（SSL 证书全自动）

1. 进入 `blog` 项目 → 顶部 **Custom domains** 标签 → **Set up a custom domain**。
2. 输入 `103254.xyz` → **Continue** → **Activate domain**。
3. 再点一次 Set up a custom domain，输入 `www.103254.xyz`，同样激活（照顾习惯敲 www 的访客）。
4. Cloudflare 会**自动**在 DNS 里创建所需记录，不用你手动加任何记录。
5. **SSL 证书（HTTPS 小锁头）自动签发、自动续期，零操作**，一般 15 分钟 ~ 2 小时完成。

状态说明：

- NS 已生效 → 域名状态几分钟内变 **Active** ✅；
- NS 还没生效 → 显示 **Pending**，不用管，NS 生效后自动激活。

- **怎么确认做对了**：Custom domains 列表里两个域名最终都变成 Active。

## Step 10：确认 CDN 已开启（橙色云朵）

你的站点默认就走 CDN，这里亲眼确认一下：

1. Cloudflare 控制台 → 左侧选中域名 `103254.xyz` → **DNS** → **Records**。
2. 找到 Pages 自动创建的记录（名字为 `@` 或 `103254.xyz`，以及 `www`）。
3. 看每条记录上的云朵图标：**橙色云朵（Proxied）= CDN 已开启**。
4. 如果看到灰色云朵（DNS only），点它一下切换成橙色。

橙色云朵的含义：全球节点缓存加速 + 自动 HTTPS + 隐藏源站 + 基础 DDoS 防护，免费版不限流量，个人博客绰绰有余。

## Step 11：最终验收清单

等 NS 生效 + 域名 Active 后，逐条打勾：

```
□ https://103254.xyz 能打开，地址栏有小锁头
□ https://www.103254.xyz 也能打开
□ 首页能看到欢迎卡片和《你好，世界》这篇文章
□ 点开文章，正文正常显示
□ Cloudflare 站点状态为 Active
□ DNS 记录里 @ 和 www 都是橙色云朵
```

全部打勾 = **博客正式上线** 🎉

---

# 第二部分：日常发文章（每次约 1 分钟）

## 方式一（推荐）：本地写好，整个文件夹拖上去

**第 1 步：本地准备文章文件夹**

电脑上新建文件夹，名字用英文或拼音（别用中文、别用空格），比如 `my-second-post`。里面放：

- `index.md`：文章本体（记事本就能写），内容格式：

```markdown
---
title: "我的第二篇文章"
date: 2026-08-05
tags: ["生活"]
summary: "一句话简介，会显示在首页列表里"
---

正文写在这里。

插入同文件夹里的图片：
![图片说明](photo.jpg)
```

- 文章要用到的图片（如 `photo.jpg`）直接放在同一个文件夹里，正文中写文件名即可。

**第 2 步：上传到 GitHub**

1. 打开仓库：`https://github.com/你的用户名/blog`
2. 依次点进：**content** → **post**
3. 点 **Add file** → **Upload files**
4. 把 `my-second-post` **整个文件夹**拖进虚线框（网页支持拖文件夹，目录结构自动保留）
5. 底部 Commit changes → 点绿色 **Commit changes**

**第 3 步：等自动部署**

Cloudflare Pages 检测到更新 → 云端自动构建 → 约 2 分钟完成。刷新 103254.xyz 就能看到新文章。以后每篇都是这三步。

## 方式二：直接在 GitHub 网页里写（适合手机 / 临时快发）

1. 仓库 → 进入 **content/post** 目录 → **Add file** → **Create new file**
2. 文件名输入：`新文章名/index.md`（如 `summer-trip/index.md`，打斜杠自动建文件夹）
3. 粘贴文章模板，改好内容 → **Commit new file**
4. 同样等约 2 分钟自动部署

## 修改与删除文章

- **修改**：仓库里点进文章的 `index.md` → 右上角铅笔图标 ✏️ → 改完 → **Commit changes** → 自动重新构建。
- **删除**：打开文章的 `index.md` → 右上角垃圾桶图标 🗑 → **Commit changes** → 文章从网站消失。
- **给已有文章补图片**：进入该文章文件夹页面（如 `content/post/summer-trip/`）→ **Add file** → **Upload files** 拖入图片。

## 日常速查卡（贴在桌面）

```
发文：本地建英文文件夹（index.md + 图片）
      → GitHub：content/post → Upload files → 拖入 → Commit
      → 等 2 分钟 → 刷新 103254.xyz ✅
改文：点开 index.md → 铅笔图标 → 改 → Commit → 等 2 分钟
删文：点开 index.md → 垃圾桶图标 → Commit → 等 2 分钟
```

---

# 第三部分：常见问题排查表

| 现象 | 原因与解决办法 |
|---|---|
| Cloudflare 站点一直 Pending | NS 未生效。核对 Spaceship 里两条 NS 是否与 Cloudflare 给的一字不差；用 dnschecker.org 查 NS 类型。最慢 48 小时，超过就联系 Spaceship 客服 |
| 自定义域名一直 Pending | 同上，NS 生效后会自动变 Active，无需操作 |
| 构建失败（红叉） | 看 View build logs：主题下载超时 → Retry deployment；yaml 报错 → 检查 hugo.yaml 缩进和完整性；版本报错 → 确认加了 HUGO_VERSION=0.139.4 |
| 网站能打开但没有样式 | hugo.yaml 里 `baseURL` 不是 `https://103254.xyz/`。改完提交会自动重新构建 |
| 文章列表里没有新文章 | 确认文件在 `content/post/` 下；确认 index.md 顶部有完整的 `---` front matter；确认 Deployments 最新一条是绿色 |
| 打开提示"连接不安全" | SSL 还在签发中（Pending），等 15 分钟~2 小时；超过 24 小时仍有问题，在 Cloudflare → SSL/TLS 里把加密模式设为 **Full** |
| 改了文章网站没变化 | 构建需要 1~2 分钟；浏览器按 Ctrl+F5 强制刷新；CDN 缓存偶尔延迟，稍等即可 |
| 上传报错"文件太多/太大" | GitHub 单次最多 100 个文件、单文件最大 100MB。普通文章无压力；大量图片分几次传 |
| 访问 blog.pages.dev 很慢/打不开 | pages.dev 临时域名在国内不稳定，属正常现象。**用 103254.xyz 访问**（走 CDN）即可 |
| 想换主题 | Pages 项目 → Settings → Builds & deployments → Build configuration：改构建命令里的主题仓库地址（如 Stack 主题：`git clone --depth 1 https://github.com/CaiJimmy/hugo-theme-stack.git themes/stack && hugo --gc --minify`），同时 hugo.yaml 改 `theme: "stack"` |

---

# 第四部分：常见疑问

**Q：真的不用备案吗？**
不用。备案针对的是"网站托管在中国大陆服务器"的情况。你的域名在 Spaceship（境外注册商）、网站在 Cloudflare（海外节点），完全不需要 ICP 备案，域名买完当天就能上线。

**Q：以后换电脑怎么办？**
什么都不用迁移。文章、配置全在 GitHub 云端，构建在 Cloudflare 云端，任何设备打开浏览器就能继续发文。

**Q：GitHub 上的仓库要不要设 Private？**
建议保持 Public。Cloudflare Pages 免费版连接私有仓库有限制，公开仓库最省心。你的文章本来就是要公开发表的，没有隐私问题。

**Q：想备份怎么办？**
GitHub 仓库本身就是备份。想留本地副本：仓库页面 → 绿色 **Code** 按钮 → **Download ZIP**，整个仓库打包下载。

**Q：以后想加访客统计？**
免费方案：Cloudflare 控制台 → 左侧 **Analytics & Logs** → **Web Analytics** → Add site → 按提示获取一段 JS 代码或 token，再配合 PaperMod 的自定义头部参数注入即可。初期建议先不管，等内容多了再加。

**Q：域名到期了会怎样？**
到期后解析停止，网站打不开。Spaceship 会在到期前邮件提醒，记得续费（或直接在 Spaceship 开启自动续费）。

---

# 附：整体流程速览图

```
【初始化，只做一次】
GitHub 注册 → 建 blog 仓库 → 网页建 hugo.yaml → 传第一篇文章
→ Cloudflare 注册 → 添加 103254.xyz → 拿到 2 条 NS
→ Spaceship 把 NS 换成 Cloudflare 的（等 0.5~48h 生效）
→ 等待期间：Pages 连接 GitHub + 配置构建命令 + HUGO_VERSION 环境变量 → 部署
→ 验证 blog.pages.dev 能打开
→ Pages 绑定 103254.xyz 和 www.103254.xyz（SSL 自动）
→ 确认 DNS 是橙色云朵（CDN 开启）
→ ✅ 上线

【日常发文，每次 1 分钟】
本地写 index.md（+图片）放进英文文件夹
→ GitHub：content/post → Upload files → 拖入文件夹 → Commit
→ 等 2 分钟 → 刷新 103254.xyz → ✅ 发布完成
```

