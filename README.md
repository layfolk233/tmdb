<div align="center">

# 🎬 影觅 / FilmSeek

**基于 TMDB API 的影视发现平台**

[API端点](#api-端点) · [视图模式](#视图模式) · [关键函数](#关键函数) · [快速开始](#快速开始) · [布局](#布局结构) · [过滤器](#过滤器说明)

![HTML5](https://img.shields.io/badge/HTML5-E34F26?logo=html5&logoColor=white)
![CSS3](https://img.shields.io/badge/CSS3-1572C6?logo=css3&logoColor=white)
![JavaScript](https://img.shields.io/badge/JavaScript-F7DF1E?logo=javascript&logoColor=black)
![Vanilla](https://img.shields.io/badge/Vanilla-NoFramework-44cc11)
![TMDB](https://img.shields.io/badge/TMDB-API-01D277?logo=themoviedatabase&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-blue)

</div>

单文件纯前端应用（~2550行），通过 TMDB API 搜索/筛选/浏览电影和剧集。

```
index.html       ← 主应用（全部 HTML+CSS+JS，单文件）
test-netlify.html   Netlify 代理测试
_redirects          Netlify 代理规则 (/tmdb/* → api.themoviedb.org/:splat)
更新日志.txt        版本历史
```

## 状态

- **默认数据**：投票数 ≥ 100，排除音乐类型
- **API 密钥**：硬编码在 index.html:1079 (`TMDB_TOKEN`)
- **代理**：通过 `_redirects` 走 Netlify 代理，或直接在 `apiFetch()` 中直连

## API 端点

| 函数 | 端点 | 时机 |
|------|------|------|
| `fetchDiscover()` | `/discover/{movie,tv}` | 筛选/翻页/切换 |
| `fetchSearch()` | `/search/{movie,tv}` | 搜索 |
| `fetchGenres()` | `/genre/{movie,tv}/list` | 初始化（sessionStorage 缓存） |
| `fetchDetail()` | `/{movie,tv}/{id}?append_to_response=credits,recommendations` | 点击卡片 |
| `fetchPerson()` | `/person/{id}?append_to_response=combined_credits` | 点击演员头像 |

## AppState（核心状态对象）

```
mediaType, sortBy, withGenres[], withoutGenres[], voteAverageGte/Lte,
voteCountGte/Lte, dateGte/Lte, year, withOriginalLanguage, region,
watchProviders[], watchRegion, searchQuery, currentPage, totalPages,
totalResults, results[], genres{}, genreMap{}, viewMode(list/grid/detail),
isLoading, error, detailData, filtersOpen, hasStarted, displayLang,
theme(dark/light), respMode(desktop/phone), savedFilters{movie/tv}
```

## 视图模式

| 模式 | 渲染函数 | 说明 |
|------|---------|------|
| 概览 (list) | `renderListView()` | 宽屏双列，1行标题+1行元数据+简介2行 |
| 海报 (grid) | `renderGridView()` | 海报卡片网格 |
| 完整 (detail) | `renderDetailView()` | 单列，backdrop背景+全参数+完整简介 |

## 关键函数

**API (1309-1410):** `apiFetch` → `handleApiResponse` → `fetchGenres` / `fetchDiscover` / `fetchSearch` / `fetchDetail` / `fetchPerson`

**渲染 (1417-2112):** `renderSortOptions` / `renderGenreChips`（含排除类型）/ `renderListView` / `renderGridView` / `renderDetailView` / `renderDetailModal`(详情弹窗) / `openPerson`(人物卡片)

**数据 (2118-2180):** `loadData` / `applyFilters` / `resetFilters` / `scheduleFilterFetch`
- 搜索触发 `loadData`，通过 `fetchSearch` 或 `fetchDiscover`
- 筛选变更→保存到 AppState→点击"应用筛选"→`applyFilters()`

**事件 (2222-2500):** `setupEvents` 汇总所有事件绑定
- 媒体切换 → 保存/恢复 `savedFilters`
- 视图切换 → 切换渲染函数，不重请求
- 主题切换 → 切换 `data-theme` 属性

**初始化 (2500-2550):** `init()` → 初始化主题/事件/筛选渲染/加载类型列表

## 布局结构

```
Header (logo + 主题/响应式/关于按钮)
Toolbar (筛选🔍 | 电影|剧集 | 概览|海报|完整)
├─ Filter Sidebar (≥1024px 固定 / <1024px 抽屉)
│  ├─ 确定+重置（sticky 冻结）
│  ├─ 组1: 搜索+排序
│  ├─ 组2: 评分/票数/类型(多选chip)/排除类型(多选chip)/年份/日期
│  └─ 组3: 原始语言/上映地区/流媒体/显示语言
└─ Content Area
   ├─ Hero（首次访问显示）
   ├─ 结果标题+活跃筛选标签
   ├─ 卡片容器 (card-list / card-grid)
   └─ 分页导航
Modal: 详情弹窗 / 人物卡片 / 关于面板
```

## 主题

CSS 自定义属性系统，`:root` 为深色主题，`[data-theme="light"]` 覆盖。约 40+ 个变量（背景/文字/边框/强调色/评分色/间距/圆角/阴影）。主题切换通过 `applyTheme()` 切换 `data-theme` 属性 + localStorage 持久化。

## 过滤器说明

| 标签 | 类型 | 对应参数 |
|------|------|---------|
| 评分 | 双输入(min-max) | `vote_average.gte/.lte` |
| 票数 | 双输入 | `vote_count.gte/.lte` |
| 类型 | 多选chip | `with_genres` (逗号分隔) |
| 排除类型 | 多选chip | `without_genres` |
| 年份 | 数字输入 | `primary_release_year` / `first_air_date_year` |
| 日期 | 双日期 | `primary_release_date.gte/.lte` / `first_air_date.gte/.lte` |
| 原始语言 | dropdown | `with_original_language` |
| 上映地区 | dropdown | `region` |
| 流媒体 | dropdown+chip | `with_watch_providers` + `watch_region` |

注意：年份与日期范围互斥——输入年份自动填日期范围，修改日期范围自动清空年份。

## 升级要点

- 所有 label 在 `filter-row-inline` 中均加全角空格"　"对齐
- 语言切换电影/剧集时保存恢复筛选参数（`savedFilters.movie/tv`）
- 默认排除音乐类型（movie:10499, tv:10772）
- 请求记录 `requestLog[]` 保留5条，响应记录 `responseLog[]` 保留1条
- 动画: `shimmer` 骨架屏, `modalIn` 弹窗, 卡片 hover
- XSS: `escapeHtml()` 转义 & < > " '
