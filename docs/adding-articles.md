# 如何向 jxc-w.com 添加新文章

> 站点结构约定见 `REQUIREMENTS.md`，本文是日常发文的操作手册。

## 一、发文前准备

确定三项信息：

1. **标题**：包含核心关键词，如"进销存报表 ECharts 大数据量渲染优化"
2. **所属栏目**（6 选 1）：

   | 栏目目录 | 栏目名 | 适合的内容 |
   |---------|--------|-----------|
   | `web-ui` | Web 界面与交互 | 表单、表格、打印、交互细节 |
   | `pwa` | PWA 与移动 Web | Service Worker、离线、推送、扫码 |
   | `api` | 前后端接口 | REST 设计、鉴权、幂等、OpenAPI |
   | `deploy` | 部署与运维 | Docker、nginx、HTTPS、备份 |
   | `data` | 报表与数据可视化 | 图表选型、ECharts、导出 |
   | `tech` | 技术架构 | 选型、模块划分、性能优化 |

3. **slug**：英文小写、连字符分隔、语义化，如 `echarts-large-data.html`。避免拼音和无意义缩写。

## 二、五个必改位置

新增一篇文章要改 5 个文件，缺一不可：

### 1. 新建文章页 `/articles/<slug>.html`

复制任意现有文章页（如 `articles/docker-deploy.html`）作为模板，替换以下内容：

| 位置 | 替换规则 |
|------|---------|
| `<title>` | `文章标题 - jxc-w.com`，≤ 60 字符 |
| `meta description` | ≤ 155 字，含关键词、自然成句，与首段呼应 |
| `canonical` / `og:url` | `https://jxc-w.com/articles/<slug>.html` |
| `og:title` | 同 title |
| `og:description` | 同 description |
| JSON-LD `Article` | `headline`、`description`、`datePublished`、`dateModified`（用发文当天日期） |
| JSON-LD `BreadcrumbList` | 第 2 项改为所属栏目页 URL 和栏目名，第 3 项为本文章 |
| 面包屑导航 | 与 BreadcrumbList 保持一致 |
| `<h1>` | 文章标题，全页唯一 |
| `.article-meta` | `发布时间：YYYY-MM-DD | 分类：栏目名` |
| 正文 | 见第三节要求 |

### 2. 所属栏目页 `/articles/<栏目>/index.html`

在 `<div class="page-group" id="page-1">` **顶部**插入卡片：

```html
<a class="card" href="/articles/<slug>.html">
  <h3>文章标题</h3>
  <p>一句话摘要（30 字左右）。</p>
  <span class="meta">栏目名 · YYYY-MM-DD</span>
</a>
```

同时更新 `.list-head` 里的"共 N 篇"计数。

### 3. 全部文章列表 `/articles/index.html`

在第 1 个 `page-group` 顶部插入同样卡片，更新"共 N 篇"计数。

**分页规则**：所有 page-group 同时展示，页码只是锚点滚动导航（不用 CSS 隐藏分组）。第 1 页满 6 张卡片时，把最旧的一张（组内最后一个）挪到新的 `<div class="page-group" id="page-2">`，第 2 页也满则级联到 `#page-3`。**每个 page-group 后紧跟一个 `.pager`**，页码覆盖全部组；紧跟第 K 组时，第 K 页标当前、其余为锚点链接：

```html
<div class="page-group" id="page-1">…6 张卡片…</div>
<nav class="pager" aria-label="分页">
  <span class="current">1</span>
  <a href="#page-2">2</a>
</nav>
<div class="page-group" id="page-2">…卡片…</div>
<nav class="pager" aria-label="分页">
  <a href="#page-1">1</a>
  <span class="current">2</span>
</nav>
```

### 4. 首页 `/index.html`

在「最新文章」区块的 `.grid` **顶部**插入同样卡片，保持 6 张；挤出的最旧一张直接删除（它仍在 `/articles/` 列表里可找到）。

注意：首页 6 张卡片的顺序 = 全站最新 6 篇文章的顺序。

### 5. 站点地图 `sitemap.xml`

在 `</urlset>` 前加一行（日期用发文当天）：

```xml
<url><loc>https://jxc-w.com/articles/<slug>.html</loc><lastmod>YYYY-MM-DD</lastmod><priority>0.7</priority></url>
```

## 三、正文写作要求

- **篇幅**：≥ 800 字（推荐 1000–1500 字）
- **首段**：100 字内点题，说明本文解决什么问题，与 meta description 呼应
- **结构**：3–5 个 `<h2>` 小节，小节下可用 `<h3>`、列表、表格、代码块（`<pre><code>`）
- **关键词**：标题、首段、至少一个 h2 中含核心关键词，自然分布不堆砌
- **内链**：正文中用 `<a href="/articles/xxx.html">` 引用同站相关文章 1–2 处（侧栏热门文章优先），文末"结语"后再带一句相关阅读
- **代码**：放 `<pre><code>`，行内代码用 `<code>`
- **文章导航**：保留 `.article-nav`，把上一篇/下一篇链接更新为本栏目相邻文章（新文章是栏目最新时，下一篇留 `<span></span>` 空占位）

## 四、可选：更新旧文章

如果新文章引用了某篇旧文章，可在旧文章正文合适位置加一句互链，权重双向流通。

## 五、发文后校验

```bash
# 1. 检查新页面 SEO 元素（title/desc/canonical/H1 都应为 1）
grep -c "<title>" articles/<slug>.html

# 2. 全站链接校验（确认 0 缺失）
node -e 'const fs=require("fs"),path=require("path");function w(d,o=[]){for(const e of fs.readdirSync(d,{withFileTypes:true})){if(e.name===".git")continue;const p=path.join(d,e.name);e.isDirectory()?w(p,o):e.name.endsWith(".html")&&o.push(p)}return o}const s=new Set();for(const f of w(".")){for(const m of fs.readFileSync(f,"utf-8").matchAll(/href="(\/[^"#]*)"/g))s.add(m[1])}let n=0;for(const l of s){let p="."+l;l.endsWith("/")&&(p+="index.html");fs.existsSync(p)||(n++,console.log("MISS "+l))}console.log("链接 "+s.size+" 个，缺失 "+n)'

# 3. 本地预览（可选）
npx serve .
```

## 六、发布

```bash
git add .
git commit -m "Add article: <slug>"
git push
```

GitHub Pages 约 1–2 分钟后自动部署。可 `curl -sI https://jxc-w.com/articles/<slug>.html` 确认返回 200。

## 七、加速收录

新文章上线后，到搜索引擎主动推送：

- **百度搜索资源平台** → 普通收录 → 手动提交新 URL
- **Google Search Console** → URL 检查 → 请求编入索引

## 偷懒方式

如果在本仓库里用 Claude Code，直接运行：

```
/add-article 文章标题 栏目
```

skill 会自动完成第二节的全部五个步骤和校验。
