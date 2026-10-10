# 功能：文档递归批注工具（一期）

## 背景

编写技术规则文档时，需要对专业术语、关键语句添加解释说明，且批注内的词语仍需递归解释。传统批注工具存在注释嵌入原文污染、锚点易错位、无版本管理等问题。需要一个纯前端静态网页工具，支持无限层级递归批注、锚点自动跟随、localStorage 持久化，可部署至 GitHub Pages。

## 需求

### R1 双栏布局与主题
- R1: 左右双栏固定分割线布局，左栏 60%（Monaco Editor），右栏 40%（批注卡片区）
- R2: **白色主题**（GitHub 风格），页面背景 `#ffffff`，右栏面板背景 `#f6f8fa`，卡片背景 `#ffffff`，主文字 `#24292f`，主强调色 `#0969da`（v2: 从深色主题 `#1a1b26/#24283b/#2a2e42/#a9b1d6/#4a7aff` 改为白色主题）

### R2 左栏 Monaco Editor
- R3: CDN AMD loader 加载 Monaco Editor 0.45+，Markdown 语法高亮、撤销重做，主题使用 `vs`（白色），关闭 unicode 易混淆字符检测 `unicodeHighlight: { ambiguousCharacters: false }`（v2: 关闭全角标点黄框提示）
- R4: 右键菜单扩展「添加批注」，仅在选中文字时显示该菜单项
- R5: 批注高亮样式 `rgba(9,105,218,0.18)` 半透明底 + `border-bottom: 2px solid #0969da`（v2: 从深色主题适配的 `#4a7aff` 改为白色主题适配的 `#0969da`）
- R6: Monaco Range 自动跟随文档变更（内置能力），高亮位置实时刷新
- R7: 点击左栏高亮文字 → 右栏滚动定位到对应卡片
- R8: 顶部工具栏只读/编辑模式切换

### R3 右栏批注卡片（递归树形）
- R9: 批注卡片独立展示，包含原文片段（淡蓝高亮）+ 批注说明（内联 `simpleMarkdown()` 渲染，支持 **加粗**、`代码`、> 引用、列表、代码块）+ 操作按钮（v2: 从 marked.js + DOMPurify 改为内联 simpleMarkdown + escapeHtml，浏览器 ORB 拦截 CDN 资源）
- R10: 支持无限层级嵌套子批注，子批注左侧 `border-left: 2px solid #0969da` 缩进区分层级（v2: 颜色从 `#4a7aff` 改为 `#0969da`）
- R11: 右键菜单「添加子批注」在卡片内选中文字时触发，保存后缩进生成子卡片；自定义菜单通过常驻 click hide handler 管理，不使用 `setTimeout once` 反模式（v2: 修复 dismiss 后再次右键无反应的 bug）
- R12: 子批注定位使用 `noteOffsetStart/End`（父级 note 字符串字符偏移量）
- R13: 单条批注操作：🔗 定位原文（左栏滚动聚焦）、🗑️ 删除（级联删除全部子批注，含确认弹窗）
- R14: 右栏顶部工具条：🔍 搜索过滤（实时匹配 sourceText/note）、👁️ 全局高亮开关、📊 批注统计
- R26: **子批注可折叠**：每张有子批注的卡片右上角显示 toggle 按钮，展开态显示 `▾ N`、折叠态显示 `▸ N`（N = 直接子批注数量）；折叠时子批注区域隐藏并移除左侧缩进线视觉（`.children.collapsed { max-height:0; border-left-color:transparent }`）；折叠状态集合 `collapsedIds: Set<string>` 随 localStorage 持久化，刷新后保持

### R4 边界状态
- R15: orphaned（原文已删除）：Monaco Range 行号越界或为空，卡片 opacity 0.5 灰显，顶部标签「⚠️ 原文已删除」
- R16: stale（原文已修改）：Range 有效但 sourceText 不匹配，卡片显示黄色 ⚠️ 图标，hover tooltip 显示原/当前文本对比，提供「确认更新」按钮同步 sourceText

### R5 持久化
- R17: localStorage 自动保存，key `doc-branch-snapshot-v1`，文档变更 500ms 防抖 + 批注增删改立即保存
- R18: 页面启动时自动读取 localStorage 恢复文档内容和批注高亮
- R19: 手动导出：顶部按钮导出完整 JSON（含 version 字段 + docContent + annotations）
- R20: 手动导入：选择文件，弹窗选项合并或覆盖，导入后刷新高亮和卡片

### R6 快捷键
- R21: Ctrl/⌘+S 手动保存、Ctrl/⌘+Shift+E 导出、Ctrl/⌘+Shift+I 导入、Esc 关闭菜单/取消搜索

### R7 数据结构
- R22: 导出格式 `{version, exportedAt?, docContent, annotations}`
- R23: 一级批注节点 `{id, parentId:null, status, range:{startLineNumber,startColumn,endLineNumber,endColumn}, sourceText, note, children}`
- R24: 子批注节点 `{id, parentId, status, noteOffsetStart, noteOffsetEnd, sourceText, note, children}`
- R25: UUID v4 生成 id，status 取值 active/stale/orphaned

## 验收标准

- [ ] 打开 index.html（通过 HTTP server），Monaco Editor 正常加载并渲染 Markdown
- [ ] 左栏选中文字右键，菜单出现「添加批注」，弹窗输入后保存，左栏淡蓝高亮 + 右栏生成卡片
- [ ] 右栏卡片内选中文字右键，菜单出现「添加子批注」，保存后生成缩进子卡片
- [ ] 编辑左栏文档，Monaco Range 自动跟随，高亮位置不丢失
- [ ] 删除被标注原文片段，对应卡片灰显标记 orphaned
- [ ] 修改被标注原文片段，对应卡片显示 ⚠️ stale 提示
- [ ] localStorage 自动保存生效，刷新后数据恢复
- [ ] 导入导出 JSON 正常，version 字段存在
- [ ] 搜索过滤实时生效
- [ ] 快捷键全部响应
- [ ] 子批注可折叠：点击 `▾ N` 折叠 / `▸ N` 展开，折叠状态刷新后保持

## 非目标

- SVG 红色虚线指引连线（二期）
- GitHub REST API 一键提交（二期）
- 拖拽分割线（二期）
- 批注卡片编辑修改（二期）
- 批注标签系统（二期）
- Markdown 实时预览面板（三期）
- 多人协作（三期）

## 技术约束

- 单 HTML 文件，CDN 引入依赖，零构建
- Monaco Editor AMD loader 方式加载（CDNJS）
- **Markdown 渲染、HTML 转义、UUID 生成均为内联实现**（浏览器 ORB 拦截了 jsDelivr/CDNJS 的非 JS MIME 资源，marked.js / DOMPurify / uuid 全部改为代码内联，零额外 CDN 依赖）（v2: 从 CDN marked.js + DOMPurify + uuid 改为内联）
- 纯原生 ES2022+ JavaScript，不写 TypeScript
- 无后端，localStorage 为唯一自动持久化方案
- 离线打开（file://）Monaco Worker 可能降级，推荐 HTTP server

## 相关

- 需求: docs/需求.md
- 任务: tasks/annotation.md
- 技术约定: AGENTS.md 第 8 节
