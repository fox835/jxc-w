---
description: 向 jxc-w.com 站点新增一篇文章：创建文章页并同步更新栏目页、全部文章列表、首页和 sitemap
---

在 jxc-w.com 站点（当前仓库）中新增一篇文章。完整规范见 `docs/adding-articles.md`。

## 输入

用户输入：$ARGUMENTS（格式：`文章标题 [栏目]`，栏目为 web-ui / pwa / api / deploy / data / tech 之一）

如果标题或栏目缺失、栏目名不是上述六个之一，先向用户询问确认，不要自行猜测栏目归属。

## 分页设计约定（执行前必读）

本站列表页（栏目页与 `/articles/`）采用**锚点分页**，规则如下，务必严格遵守：

1. 同一 HTML 内用多个 `<div class="page-group" id="page-N">` 分组（N 从 1 开始连续编号），页码链接用 `#page-N` 锚点
2. **所有分组同时展示**，页码只是滚动导航；**不要**用 CSS `:target` 隐藏分组，保持与现有页面行为一致
3. 每组最多 6 张卡片；新卡插组顶，插入后若组内有 7 张，把**该组最后一张**（即最旧）移入下一组
4. 下一组不存在则新建，放在**前一组的 pager 之后**；若移入后下一组也变成 7 张，按同规则级联处理 #page-3、#page-4……
5. **每个 page-group 紧跟一个 `.pager`**，页码覆盖当前页全部组：紧跟第 K 组时，第 K 页用 `<span class="current">K</span>`，其余页用 `<a href="#page-N">N</a>`
6. 首页不分页，「最新文章」固定 6 张

## 执行步骤

### 第 1 步：生成文章元信息

- slug：根据标题提炼 2–4 个英文语义词，小写连字符分隔（如 `echarts-large-data.html`）；先检查 `/articles/` 下不存在同名文件
- 日期：使用今天的日期（YYYY-MM-DD）
- 栏目名映射：web-ui→Web 界面与交互，pwa→PWA 与移动 Web，api→前后端接口，deploy→部署与运维，data→报表与数据可视化，tech→技术架构

### 第 2 步：撰写并创建文章页

以 `articles/docker-deploy.html` 为结构模板，创建 `/articles/<slug>.html`，要求：

1. **正文 ≥ 800 字**，围绕标题主题撰写真实、具体的技术内容（可结合进销存 Web 版场景），禁止空话凑字
2. **SEO 元素**：title `文章标题 - jxc-w.com`（≤60 字符）、description ≤155 字、canonical 与 og:url 指向本页、og:type=article、Twitter summary
3. **JSON-LD**：`Article`（headline/description/datePublished/dateModified 均为当天）+ `BreadcrumbList`（首页→栏目页→本文章）
4. **结构**：全页唯一 h1；首段 100 字内点题并与 description 呼应；3–5 个 h2 小节；正文含 1–2 处指向站内已有文章的 `<a href="/articles/xxx.html">` 内链（优先同栏目文章和侧栏热门文章）
5. **页面骨架**：header 导航、面包屑（与 BreadcrumbList 一致）、sidebar（栏目/热门文章/关于本站，与现有页面完全一致）、article-meta 行（`发布时间：日期 | 分类：栏目名`）、文末 `.article-nav` 更新为本栏目相邻文章的上一篇/下一篇链接（本篇为栏目最新时"下一篇"用 `<span></span>` 占位）、footer 与其他页面逐字一致

### 第 3 步：更新所属栏目页 `/articles/<栏目>/index.html`

- 在 `#page-1` 顶部插入标准卡片（h3 标题 + 一句话摘要 p + `栏目名 · 日期` meta）
- 更新 `.list-head` 的"共 N 篇"计数（N+1）
- 按「分页设计约定」第 3–5 条处理满组：数 `#page-1` 卡片数，为 7 张时将最后一张移入 `#page-2`（不存在则新建，位于 #page-1 的 pager 之后），级联检查后续组；确保每个 page-group 后紧跟一个页码完整的 `.pager`

### 第 4 步：更新全部文章列表 `/articles/index.html`

- 第 1 个 page-group 顶部插入同样卡片，"共 N 篇"计数 +1
- 满组处理规则与第 3 步完全相同

### 第 5 步：更新首页 `/index.html`

- 「最新文章」区 `.grid` 顶部插入同样卡片，保持 6 张，删去最旧一张
- 首页卡片顺序必须等于全站最新 6 篇的顺序（本篇在最前）

### 第 6 步：更新 `sitemap.xml`

在 `</urlset>` 前插入：

```xml
<url><loc>https://jxc-w.com/articles/<slug>.html</loc><lastmod>当天日期</lastmod><priority>0.7</priority></url>
```

### 第 7 步：校验（全部通过才算完成）

用 Node.js 校验（本环境 shell 变量展开异常，必须用 node -e 方式）：

1. 新页面 title/description/canonical/h1 各恰好 1 个；JSON-LD 可被 JSON.parse 解析
2. 全站站内链接 0 缺失（遍历所有 .html 提取 `href="/..."`，目录链接补 `index.html` 后检查文件存在）
3. 分页结构：每个含卡片的 page-group 后紧跟一个 `.pager`；每组卡片数 ≤ 6；组编号从 1 连续；pager 页码覆盖所有组且无多余页码

### 第 8 步：汇报

向用户报告：新文章 URL、改动的 5 个文件、校验结果，并给出建议的 commit 命令（不自动执行 git 提交，除非用户要求）。
