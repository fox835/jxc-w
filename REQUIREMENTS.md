# jxc-w.com 站点建设要求（v2 · 定稿）

> 参考站：jxc.js.cn / psi.js.cn / ims.js.cn（GitHub Pages 托管的静态站群，同一套模板风格）
> 联系邮箱：webnic@qq.com

## 1. 定位与部署

- **主题**：进销存 Web 版技术站（与 jxc.js.cn 错位：那边偏软件技术积累，本站偏 Web 端实现）
- **语言**：简体中文，`lang="zh-CN"`
- **托管**：GitHub Pages（仓库根目录即站点根，`CNAME` 已有 `jxc-w.com`，强制 HTTPS，国外服务器，无 ICP 备案）
- **性质**：纯静态站，无后端、无数据库、无 JS 交互
- **联系邮箱**：`webnic@qq.com`（放 footer 和 about 页）

## 2. 技术约束

| 项目 | 要求 |
|------|------|
| 形态 | 纯静态 HTML + 单一样式表 `/css/style.css`，全站零 JavaScript |
| 性能 | 单页 < 50KB（不含 CSS），无外链字体/脚本/图片；目标 Lighthouse 性能 ≥ 95 |
| 响应式 | 桌面/平板/手机三端适配，移动用 checkbox 汉堡菜单 |
| SEO | 见第 5 章（本要求的核心章节） |

## 3. 页面结构

```
/                      首页
/articles/             全部文章列表（静态分页，每页 6 篇）
/articles/<栏目>/      栏目列表页（目录式 URL）
/articles/<slug>.html  文章详情页（平铺于 articles 下）
/about.html            关于本站
/404.html              自定义 404 页
/robots.txt            爬虫协议
/sitemap.xml           站点地图
/css/style.css         全站唯一样式表
```

## 4. 全站模板骨架（所有页面共用）

1. **header**：logo（JXC-W.com）+ checkbox 汉堡菜单 + 主导航
2. **layout**：`.sidebar`（栏目列表 / 热门文章 3 篇 / 关于本站）+ `.content`
3. **footer**：三栏（站点简介 / 栏目导航 / 相关站点互链 jxc.js.cn · psi.js.cn · ims.js.cn）+ 联系邮箱 `webnic@qq.com` + 版权（无备案号）

## 5. SEO 要求（重点）

### 5.1 页面级标签
- 每页**唯一** `<title>`：≤ 60 字符，核心关键词前置，格式 `文章标题 - jxc-w.com`
- 每页**唯一** `meta description`：≤ 155 字符，含关键词且自然成句
- 每页 `canonical` 指向自身 HTTPS 规范 URL
- Open Graph 全套：`og:title` / `og:description` / `og:type`（首页 `website`、文章 `article`）/ `og:url` / `og:locale=zh_CN`
- Twitter Card：`summary` 类型

### 5.2 结构化数据（JSON-LD）
- 全站：`WebSite` 类型；文章页：`Article`；面包屑：`BreadcrumbList`；列表页：`ItemList`

### 5.3 内容结构
- 语义化 HTML5，全页**唯一 H1**，正文 H2/H3 层级
- 每篇文章 800 字以上，首段 100 字内点题

### 5.4 URL 与内链
- 栏目页目录式 URL，文章语义化 slug；面包屑可见且与结构化数据一致
- 三向内链：侧栏热门文章、正文互引、footer 站群互链

### 5.5 爬虫与收录
- `robots.txt`：`User-agent: *` + `Allow: /` + `Sitemap: https://jxc-w.com/sitemap.xml`
- `sitemap.xml`：全量 URL 含 `<lastmod>`，随发文更新
- 上线后提交：百度搜索资源平台、Google Search Console、Bing Webmaster

### 5.6 体验与权重
- 纯静态秒开、零 JS 阻塞；每图必配 `alt`；404 页提供返回首页和全部文章链接

## 6. 内容栏目（初版 6 个）

| 栏目目录 | 名称 | 种子文章 slug |
|---------|------|--------------|
| web-ui/ | Web 界面与交互 | web-form-print.html |
| pwa/ | PWA 与移动 Web | pwa-offline.html |
| api/ | 前后端接口 | api-design.html |
| deploy/ | 部署与运维 | docker-deploy.html |
| data/ | 报表与数据可视化 | report-echarts.html |
| tech/ | 技术架构 | tech-stack.html |

## 7. 分页与卡片约定

- 卡片：`<a class="card">` = `<h3>` + 摘要 `<p>` + `栏目 · 日期` meta
- 静态分页：每页 6 篇，`<div class="page-group" id="page-N">`
- 文章页：`<article>` + `发布时间 | 分类` 元信息行 + H2 分节正文

## 8. 发文流程（新增文章步骤）

1. 在 `/articles/<slug>.html` 新建文章页（复制现有文章页改内容）
2. 在所属栏目页顶部插入卡片
3. 在 `/articles/` 第 1 页顶部插入卡片；若该页满 6 篇，最旧的挪入下一页并补页码链接
4. 在首页「最新文章」区顶部插入卡片，保持 6 张，挤出最旧的
5. 更新对应页面的 JSON-LD `dateModified` 与 sitemap.xml 的 `<lastmod>`
