# CLAUDE.md — tutududu 工作手册

> 给未来在这个仓库里开 session 的 Claude。项目的设计思路与迭代历史见仓库外的 `CONTEXT.md`。

---

## 项目概览

个人博客 · 静态站，无构建 · GitHub Pages 部署
- 部署：`https://tototutududu-bit.github.io/tutududu/`
- 技术栈：纯 HTML + CSS + JS，字体走 Google Fonts（Lora 衬线 + Caveat 手写）
- 页面：`index.html` / `writing.html` / `film.html` / `sound.html` / `guestbook.html`
- 数据：`articles.js`（文章数组，共享给首页+文字页）

---

## 发文章流程（最常用）

1. 用户把标题/日期/标签/正文贴过来
2. 编辑 `articles.js`，**在数组最前面**加一条：

```js
{
  id: <递增唯一>,           // 不要复用已有 id；顺序以数组位置为准
  tag: '读书',               // 自由标签，自动生成筛选按钮
  title: '《树犹如此》读后',
  date: '2026 · 03',          // 注意：年 + 空格 + 中圆点 + 空格 + 月
  // byline: 'by Claude',    // 仅 Claude 代写的内容才加；用户原创不加
  readtime: '5 min',
  excerpt: '...一两句引子...',
  illus: null,                // 或 { svgKey, caption, note, rot:'illus-r-n1' }
  body: `<p>...</p>`          // HTML 字符串，支持 <p> / <h3> / <blockquote> / pull-quote div / __ILLUS:...__
}
```

3. `git add articles.js && git commit -m "..." && git push`

### body 可用的结构
- `<p>` 段落
- `<h3>` 小标题（Lora serif）
- `<blockquote>` 长引用块（淡色左边线 + 斜体，适合引用书里的话）
- `<div class="pull-quote">...</div>` 便签式金句（Caveat 手写）——**默认不要用**，除非用户明确要求。用户偏好正文统一段落，不要自作主张挑句子做金句
- `__ILLUS:svgKey:caption:note:rotClass__` 内联插画占位符，渲染时会被 `writing.html` 的 `openArticle()` 替换为 SVG

> 发文章默认：纯 `<p>` 段落，原文一字不改。`h3`/`blockquote` 仅当原文本身有小标题或整段引用时才用；`pull-quote` 一律不主动用。

---

## 插画流程

SVG 定义集中在 `writing.html` 的 `ILL = {}` 对象里。新增：

```js
ILL.<key> = `<svg viewBox="0 0 200 200" ...>...</svg>`;
```

然后文章 `illus` 字段或 body 的 `__ILLUS:__` 占位符引用这个 key。

### 风格约束（严格遵守）
- viewBox 200×200 或类似小尺寸
- **4–6 种低饱和色**，不要纯色（不要 `#ff0000`）
- 以 `<rect>` / `<circle>` / `<line>` 为主，`<path>` 尽量简单
- 不画脸、不写字、只画氛围
- 详细色板和已有插画索引见 `CONTEXT.md` 第 212 行之后

### 旋转类
`illus-r-n2 / -n1 / -0 / -1 / -2`（对应 -2° 到 +2°），写在 `rot` 字段。

---

## 留言板架构（重要，容易忽略）

留言板 **不是** 静态的。后端是 **Google 表单 + Google Sheet**：

- 访客提交 → 前端 fetch POST 到 `FORM_POST_URL`（`mode:'no-cors'`）→ 进 Sheet
- 页面加载 → 前端 fetch `SHEET_CSV_URL` → 解析 CSV → 渲染留言
- 两个关键常量在 `guestbook.html` 顶部 `<script>` 里定义：`FORM_POST_URL` / `SHEET_CSV_URL` / `ENTRY_NAME` / `ENTRY_TEXT`

### 反垃圾 / 安全
- **蜜罐字段**：表单里有一个 off-screen 的 `#f-website`，真人看不到。提交时非空 → 判定为 bot，静默吞掉
- **XSS 转义**：`escHtml()` 把留言转义后才塞进 DOM，不要改成 innerHTML 直出
- **已知延迟**：Google 的 Publish-to-web CDN 有 1–10 分钟缓存，其他访客不会立刻看到新留言；本人提交的会通过 `pendingMessages` 立即显示

---

## Git 推送的沙盒坑（重要！）

Claude Code 的 bash 沙盒默认会把 `github.com` 路由到 `198.19.0.x`，导致 push 报 **"无法连接到远程服务器"**。

**解决**：push 命令一律加 `dangerouslyDisableSandbox: true`：

```
cd "D:/创造/小狗主页 202604/tutududu" && git add -A && git commit -m "..." && git push
```

用 Bash 工具时把参数 `dangerouslyDisableSandbox: true` 带上。

**首次 push 还需要**：弹浏览器登录 GitHub（Credential Manager），用户点两下授权后凭据存 Windows 凭据管理器，后续自动。

---

## 本地路径

- 仓库工作副本：`D:\创造\小狗主页 202604\tutududu\`
- 用户个人设计笔记（不入库）：`D:\创造\小狗主页 202604\CONTEXT.md`
- 以前的 playground 实验（不入库）：`D:\创造\小狗主页 202604\留言板方案*.html`

---

## 约定风格

### commit message
中文，简洁，动宾句。示例：
- `发文章《树犹如此》读后 + 加 h3/blockquote 样式`
- `重画《树犹如此》配图：仰望两棵柏，中间是晴空`
- `留言板接 Google 表单 + Sheet，加蜜罐反 bot，渲染做 XSS 转义`

末尾带：`Co-Authored-By: Claude Opus 4.7 (1M context) <noreply@anthropic.com>`

### 内容处理
- **用户原文一字不改**，只负责排版和 HTML 化
- 不主动帮用户"润色"或"结构化"
- Claude 代写的占位内容加 `byline: 'by Claude'`

### 审美偏好
- 克制、不对称、暖纸色调、手写体做点缀
- **减法优先于加法**；改风格时先问用户要"轻/中/重"哪档
- 不用 emoji（除非用户明确要）

---

## 颜色变量（各页已定义）

```css
--ink:  #1c1710   /* 主文字 */
--ink2: #3a3226 / #5a4838   /* 次文字 */
--line: #7a6e5c / #c8bfb0   /* 分割线 */
--rust: #b8592a / #b8541e   /* 强调 */
--paper: #f6efde / #f2ede4  /* 纸色背景 */
--cream: #faf5e8            /* 更亮的奶油色 */
--bg:   #ede6d2             /* 整页底 */
```
