# Hugo 文章管理手册

> 面向 mosfet-notes 项目维护者的 CRUD 工作流文档。
> 本文件位于 `docs/` 目录，**不进 Hugo 构建**，纯本地维护参考。

---

## 目录

1. [项目结构](#项目结构)
2. [新增文章](#新增文章)
3. [修改文章](#修改文章)
4. [删除文章](#删除文章)
5. [查询文章](#查询文章)
6. [批量操作](#批量操作)
7. [Frontmatter 字段详解](#frontmatter-字段详解)
8. [特殊功能开关](#特殊功能开关)
9. [内容模板](#内容模板)
10. [常见问题](#常见问题)
11. [完整工作流示例](#完整工作流示例)

---

## 项目结构

```
mosfet-notes/
├── content/
│   ├── about.md                 # 关于页（单页面）
│   └── post/                    # 博客 section（mainSections）
│       └── mosfet-handbook-2026.md
├── archetypes/
│   ├── default.md               # hugo new 默认模板
│   └── post.md                  # hugo new post/... 模板
└── config.toml                  # mainSections = ["post"]
```

**关键约束**：
- `config.toml` 中 `mainSections = ["post"]` —— 只有 `content/post/` 下的文章显示在首页和 `/post/` 列表
- 任何文章文件路径改变，Hugo 会按文件内容重建 URL

---

## 新增文章

### 方法 1：用 Hugo 命令（推荐）

```bash
cd D:/Mosfet_Project/mosfet-notes

# 创建博客文章（用 archetypes/post.md 模板）
hugo new post/同步buck变换器驱动设计.md

# 创建关于类单页面
hugo new about/联系.md

# 创建其他 section（需先在 mainSections 加配置才会显示在首页）
hugo new docs/快速入门.md
```

**默认行为**：
- 文件创建在 `content/post/同步buck变换器驱动设计.md`
- 自动套用 `archetypes/post.md` 模板（含 categories/tags/description/math/lead 字段）
- `draft: true`（安全：避免未写完就发布）
- `date` 默认是当前时间

### 方法 2：手动创建文件

在 `content/post/` 下新建 `.md` 文件，复制下面这个模板：

```markdown
---
title: "你的文章标题"
date: 2025-12-20T10:00:00+08:00
draft: false
categories: ["器件"]
tags: ["MOSFET", "功率器件"]
description: "一两句话摘要，会显示在首页卡片和搜索引擎"
author: "Lance"
math: false
lead: ""
---

正文从这里开始写...
```

### 方法 3：复制旧文改新文

适合做"系列文章"或"同主题续作"：

```bash
cp content/post/mosfet-handbook-2026.md content/post/mosfet-handbook-part2.md
# 然后修改新文件的 title / date / draft
```

---

## 修改文章

### 编辑流程

```bash
# 1. 用任意编辑器打开文件
#    - VS Code: code content/post/xxx.md
#    - Cursor: cursor content/post/xxx.md
#    - Notepad++ / Sublime 等

# 2. 修改 frontmatter 或正文

# 3. 保存

# 4. dev server 会自动检测变更并重建（如果 hugo server 在跑）
#    http://localhost:1313/post/xxx/ 立即看到效果
```

### 修改建议

- **改标题**：直接改 `title` 字段即可，URL 不变（除非重命名文件）
- **改日期**：必须改成**过去时间**，否则 Hugo 不构建
- **改分类/标签**：直接改数组，下划线连接的多词会自动 `urlize`
- **改 draft 状态**：发布前改 `draft: false`

### 触发重建

dev server 默认开启 **Fast Render Mode**，修改内容保存后即时重建。

如果没自动重建，强制触发：

```bash
touch content/post/xxx.md
```

或重启 server：

```bash
# 停掉旧 server（Ctrl+C 或 task stop）
hugo server --disableFastRender   # 完全重建
```

---

## 删除文章

### 标准删除

```bash
cd D:/Mosfet_Project/mosfet-notes

# 永久删除文件
rm content/post/xxx.md

# 或 git rm（如果文件已被 git 跟踪）
git rm content/post/xxx.md
```

dev server 会自动检测到文件消失并重建。

### 软删除（保留文件但不显示）

```bash
# 在 frontmatter 改 draft: true
# 或
# 在 frontmatter 加： 
#   _build:
#     render: never
```

### ⚠️ 注意事项

- **没有回收站**，删除即丢失。建议先 `git status` 确认
- 如要保留 URL（避免 404），改 `draft: true` 比删文件好
- 删文件后 `public/post/xxx/index.html` 会自动消失

---

## 查询文章

### 列出所有文章

```bash
hugo list all                 # 全部内容（含 section / taxonomy 页）
hugo list published           # 只列出已发布
hugo list drafts              # 只列出草稿
```

### 按字段搜索

```bash
# 搜索标题含"MOSFET"的文章
hugo list all | grep -i mosfet

# 找所有含 "GaN" 的文章
grep -l "GaN" content/post/*.md

# 找所有 draft 文章
grep -l "^draft: true" content/post/*.md

# 找所有含 math: true 的文章
grep -l "^math: true" content/post/*.md

# 找所有带特定 tag 的文章
grep -l '"GaN"' content/post/*.md
```

### 按日期范围

```bash
# 列出 2025 年的所有文章
hugo list all --layout _default | awk -F, '$4 >= "2025-01-01" && $4 < "2026-01-01"'
```

### 统计

```bash
# 文章总数
ls content/post/*.md | wc -l

# 按分类统计
grep -h "^categories:" content/post/*.md | sort | uniq -c | sort -rn

# 按标签统计
grep -h "^tags:" content/post/*.md | sort | uniq -c | sort -rn
```

---

## 批量操作

### 批量改名 / 移动

```bash
# 改名（URL 会跟着变）
mv content/post/old-name.md content/post/new-name.md

# 移动到新 section（先在 config 加 mainSections）
mv content/post/xxx.md content/docs/xxx.md

# 批量重命名（用 for 循环）
cd content/post/
for f in *.md; do
  new=$(echo "$f" | sed 's/old/new/g')
  mv "$f" "$new"
done
```

### 批量改 frontmatter

```bash
# 批量改某字段（如全部改 draft 为 false）
sed -i 's/^draft: true/draft: false/' content/post/*.md

# 批量加 tags
# 用编辑器多文件替换更安全（VS Code / Cursor 支持）
```

### 批量删除

```bash
# 删除所有 draft 文章
find content/post -name "*.md" -exec grep -l "^draft: true" {} \; | xargs rm

# 删除某月之前的所有文章
find content/post -name "*.md" -newermt "2025-01-01" ! -newermt "2025-07-01" -exec rm {} \;
```

---

## Frontmatter 字段详解

| 字段 | 类型 | 必填 | 默认 | 说明 |
|------|------|------|------|------|
| `title` | string | ✅ | — | 文章标题 |
| `date` | datetime | ✅ | 当前时间 | 发布时间，**必须是过去时间** |
| `draft` | bool | ✅ | `true` | `true` = 草稿（默认 dev 看不到） |
| `categories` | array | ❌ | `[]` | 分类，单数个（影响 mainSections 归属） |
| `tags` | array | ❌ | `[]` | 标签，可多个 |
| `description` | string | ❌ | — | 摘要（首页卡片 + SEO） |
| `lead` | string | ❌ | — | 副标题（大号 italic 衬线） |
| `math` | bool | ❌ | `false` | `true` = 启用 KaTeX 公式 |
| `author` | string | ❌ | `Lance` | 显示在文章 meta |
| `comments` | bool | ❌ | `true` | `false` = 关闭单篇评论 |
| `thumbnail` | string | ❌ | — | 缩略图路径（如 `img/xxx.png`） |

### 完整字段示例

```markdown
---
title: "MOSFET 选型与驱动实战：从数据手册到 PCB 布局"
date: 2025-12-20T11:30:00+08:00
publishDate: 2025-12-20T11:30:00+08:00
draft: false
categories: ["器件"]
tags: ["MOSFET", "功率器件", "栅极驱动", "损耗", "PCB布局"]
author: "Lance"
math: true
lead: "做电源设计这些年，最让我头疼的不是拓扑，而是 MOSFET —— 它脾气大、敏感、却常常在你掌握规矩之后给你惊喜。"
description: "面向电源/电机驱动工程师的 MOSFET 全流程笔记：参数解读、损耗估算、栅极驱动、布局与实测。"
---
```

### 时间格式

```yaml
date: 2025-12-20T11:30:00+08:00       # RFC3339，最常用
date: 2025-12-20                      # 简写（UTC 00:00）
date: 2025-12-20T11:30:00Z            # UTC
```

时区建议用 `+08:00`（北京时间），保持一致。

---

## 特殊功能开关

### 数学公式

启用 KaTeX 渲染：

```markdown
---
math: true
---

正文里：

行内公式 $E = mc^2$ 自动渲染。

独立公式块：

$$
P_{\mathrm{loss}} = I_{\mathrm{rms}}^2 \cdot R_{\mathrm{DS(on)}}
$$
```

### Pull Quote（编辑型引文）

用 Hugo shortcode（不需配置）：

```markdown
{{< quote align="center" >}}
这句话作为酒红色大字引文居中显示。
{{< /quote >}}

{{< quote align="left" >}}
左浮动版，文字环绕。
{{< /quote >}}

{{< quote align="right" >}}
右浮动版。
{{< /quote >}}
```

### 单文章关闭评论

```markdown
---
comments: false
---
```

Twikoo 启用后此字段生效；当前项目评论系统未启用，配置保留待用。

### 草稿（dev 不可见）

```markdown
---
draft: true
---
```

dev server 默认不显示 draft 文章；加 `-D` 显示：

```bash
hugo server -D --port 1313
```

prod build（Netlify 部署）默认排除 draft。

---

## 内容模板

### archetypes/post.md（博客文章模板）

```markdown
+++
title = "{{ replace .File.ContentBaseName "-" " " | title }}"
date = {{ .Date }}
draft = true
categories = []
tags = []
description = ""
+++
```

实际项目用的是 TOML 格式（`+++`），但 Hugo 也支持 YAML（`---`）和 JSON。

### 自定义模板

修改 `archetypes/post.md` 可改变 `hugo new post/...` 的默认模板内容：

```markdown
+++
title = "{{ replace .File.ContentBaseName "-" " " | title }}"
date = {{ .Date }}
draft = true
categories = []
tags = []
description = ""
math = false
lead = ""
+++
```

---

## 常见问题

### Q1：为什么我新建的文章 404？

**A**：未来日期导致 Hugo 不构建。

```yaml
# ❌ 错
date: 2026-12-20T10:00:00+08:00

# ✅ 改成过去时间
date: 2025-12-20T10:00:00+08:00
```

### Q2：dev server 看不到我的新文章？

**A**：检查 `draft` 字段：

```bash
# dev 默认不显示 draft
hugo server --port 1313

# 显示 draft
hugo server -D --port 1313
```

或确认文件确实在 `content/post/` 下（Hugo 只处理此目录的文章）。

### Q3：改了文章没生效？

**A**：

1. 浏览器硬刷新（Ctrl+Shift+R）清除缓存
2. 检查 dev server 是否在跑（http://localhost:1313/）
3. 看 Hugo server 日志是否有 error
4. 强制 touch 触发重建：`touch content/post/xxx.md`

### Q4：文件名能用中文吗？

**A**：可以。Hugo 完全支持中文文件名：

```
content/post/同步buck变换器驱动设计.md
```

但 URL 会变成 URL-encoded 形式：

```
/post/%E5%90%8C%E6%AD%A5buck%E5%8F%98%E6%8D%A2%E5%99%A8%E9%A9%B1%E5%8A%A8%E8%AE%BE%E8%AE%A1/
```

**建议**：文件名用英文或拼音，避免 URL 过长：

```
content/post/sync-buck-driver-design.md
# → /post/sync-buck-driver-design/
```

### Q5：怎么让某篇文章排到最前？

**A**：改 `date` 为最近时间：

```yaml
date: 2025-12-31T23:59:59+08:00
```

文章按 date 倒序排列。但这是 hack，正经做法是改 `weight`：

```yaml
weight: -100  # 数字越小越靠前
```

### Q6：能放图片吗？

**A**：可以。三种方式：

1. **同目录 bundle**（推荐）：
   ```
   content/post/xxx/
   ├── index.md
   └── cover.png
   ```
   在 markdown 里引用：
   ```markdown
   ![封面](cover.png)
   ```

2. **static 目录**：
   ```
   static/img/xxx.png
   ```
   引用：
   ```markdown
   ![图片](/img/xxx.png)
   ```

3. **外链**：直接用图床 URL

### Q7：怎么引用站内其他文章？

**A**：用相对路径：

```markdown
参见 [MOSFET 实战笔记](/post/mosfet-handbook-2026/)。
```

或者用 Hugo 的 ref shortcode（更安全，重命名后链接不失效）：

```markdown
参见 [MOSFET 实战笔记]({{< ref "post/mosfet-handbook-2026.md" >}})。
```

### Q8：能加 TOC（目录）吗？

**A**：默认开启（`config.toml` 里 `showTOC = true`）。如需单篇关闭：

```markdown
---
showTOC: false
---
```

### Q9：怎么预览草稿？

**A**：

```bash
# 1. 文件 frontmatter 保持 draft: true

# 2. dev server 加 -D 参数
hugo server -D --port 1313

# 3. 文章会显示，URL 形如
#    http://localhost:1313/post/xxx/
```

注意 dev server 的草稿**不会**出现在 `/post/` 列表（除非把列表页也改成显示 draft）。

### Q10：删除文章后能不能恢复？

**A**：只要 git 跟踪过就能恢复：

```bash
# 查看删除历史
git log --diff-filter=D --summary

# 恢复
git checkout HEAD~1 -- content/post/xxx.md
```

如果文件从未 `git add`，删除即永久丢失。

---

## 完整工作流示例

### 场景：写一篇关于 GaN 器件的新文章

```bash
# 1. 创建草稿
cd D:/Mosfet_Project/mosfet-notes
hugo new post/gan-device-application.md

# 2. 用编辑器打开
code content/post/gan-device-application.md

# 3. 编辑 frontmatter
cat content/post/gan-device-application.md
```
```yaml
---
title: "GaN 功率器件在车载充电机中的应用"
date: 2025-12-22T14:00:00+08:00
draft: true                      # 草稿状态
categories: ["器件"]
tags: ["GaN", "功率器件", "OBC"]
description: "GaN HEMT 相比 Si MOSFET 在车载 OBC 上的优势、挑战与实测"
author: "Lance"
math: true                       # 有公式
lead: "GaN 给车载充电机带来更高频率、更小体积，但驱动与 EMC 设计的坑不少。"
---
```

```markdown
# 4. 写正文

## 引言

相比 Si MOSFET，GaN HEMT 的 $R_{\mathrm{DS(on)}}$ 在 100 V 以下显著更低。

{{< quote align="center" >}}
GaN 不是简单替代品，而是需要重新设计的系统。
{{< /quote >}}

## 主要优势

- 开关频率可达 MHz 级
- 反向恢复电荷 $Q_{\mathrm{rr}}$ 接近零

（继续写...）

# 5. 在浏览器预览
# dev server 已自动重建
# http://localhost:1313/post/gan-device-application/
```

```bash
# 6. 写完后改 draft: false
# （编辑器里改）

# 7. 确认发布
curl -s http://localhost:1313/post/gan-device-application/ | grep "<title>"

# 8. git 提交
git add content/post/gan-device-application.md
git commit -m "post: GaN 功率器件在车载充电机中的应用"
git push
# → Netlify 自动部署

# 9. （可选）删除旧文章
rm content/post/old-article.md
git add -A
git commit -m "post: 删除旧文章"
git push
```

---

## 附录：Hugo 常用命令速查

```bash
# 创建内容
hugo new post/xxx.md

# 本地预览（默认 1313 端口）
hugo server

# 显示草稿
hugo server -D

# 完整重建
hugo server --disableFastRender

# 生产构建（输出到 public/）
hugo --gc --minify

# 列出所有页面
hugo list all
hugo list published
hugo list drafts

# 调试渲染
hugo --templateMetrics
hugo --logLevel debug
```

---

## 附录：相关文件路径

```
D:/Mosfet_Project/mosfet-notes/
├── config.toml                  # mainSections, showReadingTime, showTOC 等配置
├── archetypes/
│   ├── default.md               # 默认模板
│   └── post.md                  # 博客文章模板
├── content/
│   ├── about.md                 # 关于页
│   └── post/                    # 博客 section
├── layouts/
│   ├── _default/
│   │   ├── single.html          # 文章详情页模板
│   │   └── list.html            # 列表页模板
│   └── partials/
│       ├── post-card.html       # 首页卡片
│       ├── post-meta.html       # 文章 meta
│       └── toc.html             # 目录
└── docs/
    └── article-management.md    # ← 你正在看的文档
```

---

**最后更新**：2026-07-05 · 适用于 mosfet-notes 项目