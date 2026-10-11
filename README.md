# docbranch · 文档递归批注工具

> 一个**纯前端单 HTML 文件**的 Markdown 递归批注工具——批注里的关键词还能继续加批注，无限层级，锚点不随文档修改错位，支持 GitHub 双仓库同步。

![界面截图](docs/images/文档标注工具推荐.png)

---

## 为什么做

编写技术规则、产品文档时的常见痛点：

1. 专业术语需要解释，但**不想让注释嵌在原文里污染文档本体**
2. 解释里又出现新术语，需要**对批注本身继续追加批注（递归嵌套）**
3. 传统批注工具在文档增删后，**锚点位置错位导致批注失效**
4. 批注数据需要纳入 Git 版本管理，但手动同步流程繁琐

docbranch 就是为解决这些问题设计的。

## 核心特性

### ✅ 已实现

| 模块 | 说明 |
|------|------|
| 📝 **Markdown 编辑** | Monaco Editor（VS Code 同款），语法高亮、撤销重做、只读/编辑切换 |
| 🖼️ **递归批注** | 无限层级嵌套——选中原文加批注，批注里选中文字再加子批注 |
| 🎯 **锚点自动跟随** | 基于 Monaco Range + 字符偏移量，文档修改后高亮位置不丢 |
| 🏷️ **边界状态** | 原文删除 → 灰显失效；原文修改 → ⚠️ 变更提示 + 一键确认更新 |
| 📂 **多文件管理** | 文件索引 + 独立持久化，下拉切换、新建、重命名、删除 |
| ☁️ **GitHub 一键同步** | PAT 认证 + SHA 冲突处理 + 批量提交/拉取多文件 |
| 🌳 **目录树侧边栏** | 按 `docPath` 自动推导目录结构，220px 抽屉式，可折叠 |
| 💾 **持久化** | localStorage 自动保存（500ms 防抖）+ JSON 手动导入/导出 |
| 🔍 **辅助能力** | 搜索过滤、全局高亮开关、批注统计、子批注折叠、快捷键 |
| 🎨 **白色主题** | GitHub 风格，高亮色 `#0969da`，Monaco `vs` 主题 |
| 🏷️ **批注标签系统** | 4 种预设标签（定义/公式/规则/待确认），右栏按标签筛选（不持久化） |
| ✏️ **卡片编辑修改** | 双击批注说明进入编辑态，Ctrl+Enter 保存 / Esc 取消；父批注 note 变更后自动重算子批注偏移量 |
| 🎯 **SVG 红色虚线指引** | 顶层 SVG 悬浮层 + 贝塞尔虚线，默认智能模式（hover/聚焦触发），三态开关 |
| ↔️ **拖拽分割线** | 30%-80% 比例范围，双击重置 60:40，位置持久化 |
| 📱 **移动端阅读模式** | 768px 断点自动切换：内联 Markdown 渲染（不加载 Monaco）+ 底部三档拖拽批注面板 + 汉堡抽屉目录树；鸿蒙/安卓选区缓存防丢失；safe-area 刘海屏适配 |
| 🐛 **移动端调试彩蛋** | 手机端在「⚙️ GitHub 同步设置」弹窗标题上 1.5s 内连点 5 次 → 呼出调试浮窗，查看 console 日志；同样操作再连点 5 次隐藏 |

### 🗺️ 三期路线图（详见 `specs/`）

| 功能 | 说明 |
|------|------|
| Markdown 实时预览面板 | 三栏：编辑 ｜ 预览 ｜ 批注 |
| 导出带批注的 Markdown | 批注以脚注形式内嵌 |
| 文档版本对比 | 原文 + 批注变更差异 |
| 多人协作批注 | WebSocket 或 GitHub Issues 集成 |

## 快速开始

### 本地运行

推荐用 HTTP server 启动（Monaco Worker 需要 HTTP 环境，直接双击 `file://` 打开可能降级为只读基础模式）：

```bash
python -m http.server 8000
# 或
npx serve .
```

然后浏览器打开 `http://localhost:8000` 即可。

### GitHub Pages 部署

#### 方案 A：双仓库（GitHub Free 推荐，免费）

```
仓库 A（Public）: docbranch-host        ← 工具壳，托管 GitHub Pages
└── index.html                          ← 拖 GitHub 网页上传一次即可

仓库 B（Private）: docbranch-data       ← 文档 + 批注数据
├── docs/*.md                           ← 你的 Markdown 文档
├── annotations/*.annotations.json      ← 每个文件的批注（自动生成）
└── docbranch-index.json                ← 文件索引（自动生成）
```

1. 创建仓库 A（Public）→ Settings → Pages → Source `main` / 根目录 → Save
2. 创建仓库 B（Private）→ 空仓即可
3. 把本地 `index.html` 拖网页上传到仓库 A
4. 打开 `https://你的用户名.github.io/docbranch-host/` → ⚙️ 设置：
   - **PAT**：针对 **仓库 B** 生成的 fine-grained token，仅勾选 Contents（读写）+ Metadata（只读）
   - **仓库全名**：`用户名/docbranch-data`（填仓库 B，不是 A）
   - **分支名**：`main`
5. 之后一键同步/拉取全部自动走仓库 B

#### 方案 B：单仓库（GitHub Pro $4/月）

Private 仓库直接开 Pages，`index.html` + `docs/` + `annotations/` 全部在同一仓库，最干净。

#### 方案 C：Cloudflare Pages（国内更快）

免费计划支持 Private 仓库直连构建 + 自带密码保护，一个仓库搞定。

## 项目结构

```
docbranch/
├── index.html              ← 全部代码（HTML + CSS + JS，单文件）
├── README.md               ← 本文件
├── AGENTS.md               ← AI 协作元规则
├── docs/
│   ├── 需求.md              ← 完整 PRD
│   └── images/              ← 截图等静态资源
└── specs/                  ← 功能规格（单功能单文件）
    ├── feature-annotation.md       ← 一期已实现：批注核心
    ├── feature-multi-file.md       ← 二期已实现：多文件
    ├── feature-github-sync.md      ← 二期已实现：GitHub 同步
    ├── feature-directory-tree.md   ← 二期已实现：目录树
    ├── feature-guideline-lines.md  ← 二期已实现：SVG 虚线指引
    ├── feature-split-resize.md     ← 二期已实现：拖拽分割线
    ├── feature-annotation-tags.md  ← 二期已实现：标签系统
    └── feature-card-edit.md        ← 二期已实现：卡片编辑
```

## 技术栈

| 依赖 | 版本 | 引入方式 |
|------|------|----------|
| Monaco Editor | 0.45+ | CDN AMD loader（Markdown 编辑器核心） |
| simpleMarkdown / escapeHtml | — | **内联实现**（浏览器 ORB 拦截了外部 CDN 的非 JS MIME 资源） |
| uuidv4 | — | **内联实现** |

- **零构建**：无 npm、无 Webpack/Vite/esbuild，单 HTML 文件直接部署
- **纯前端**：无后端，持久化靠 localStorage + JSON 导入导出
- **GitHub Sync**：浏览器直连 GitHub REST API v3（CORS 允许），无需后端代理
- **语言**：原生 ES2022+ JavaScript，不写 TypeScript

## 快捷键

| 快捷键 | 功能 |
|--------|------|
| `Ctrl/⌘ + S` | 手动保存 |
| `Ctrl/⌘ + Shift + E` | 导出 JSON |
| `Ctrl/⌘ + Shift + I` | 导入 JSON |
| `Esc` | 关闭右键菜单 / 取消搜索过滤 |

## 数据安全

- **localStorage**：所有数据存储在浏览器本地，清除浏览器数据会丢失
- **GitHub PAT**：仅存储在本地 `doc-branch-github-config`，不会硬编码到页面中；公用电脑不要保存
- **公开壳地址**：公开部署的 `index.html` 壳地址**无用户数据泄露风险**——你的文档和批注都在 Private 仓库 B，PAT 也只存在你本地浏览器；壳地址的风险仅限于源码被克隆、搜索引擎索引（可在仓库加 `robots.txt` 禁止爬虫）

---

详细设计见 [docs/需求.md](docs/需求.md)。需求规格见 [specs/](specs/)。
