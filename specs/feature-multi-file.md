# 功能：多文件支持（二期优先）

## 背景

一期工具默认"一页只配一份 Markdown 文档 + 一份批注树"。实际仓库中往往有多个 `.md` 文件需要批注（如 `rules.md`、`api.md`、`README.md`）。用户需要能在工具内切换、管理多文件，每个文件独立维护自己的批注数据。

## 需求

### R1 文件管理核心
- R1: 工具内维护**文件索引**，每个索引项包含：`id`（uuid）、`name`（显示名，如"筛选规则"）、`docPath`（GitHub 仓库内路径，如 `docs/rules.md`）、`isDirty`（未保存标记）
- R2: 顶部工具栏新增**文件切换器**（下拉 + 新建按钮 + 重命名 + 删除，布局为：`[📄 筛选规则 ▾] [+ 新建]`，位于 Monaco 工具栏左侧）
- R3: 默认文件：首次打开自动创建一份 "未命名文档" 作为初始文件，避免空白

### R2 文件切换行为
- R4: 切换文件时 **自动保存当前文件**（Storage.saveCurrent()），然后加载目标文件的 docContent → Monaco 重新 setValue + annotations 重新渲染 + 高亮刷新
- R5: 切换过程中显示**轻量 loading 指示**（Monaco 右上角显示"切换中..."），防止用户误操作
- R6: 如果当前文件有未保存的编辑（isDirty = true），切换时仍自动保存（不弹窗确认）

### R3 新建 / 重命名 / 删除
- R7: **新建文件**：弹窗输入名称 + GitHub 仓库内的目标路径（默认 `docs/未命名.md`，可改），创建空文件（DEFAULT_DOC 模板）并切换到新文件
- R8: **重命名文件**：修改 name（显示名）和 docPath（仓库路径），弹窗校验路径格式（必须以 `.md` 结尾）
- R9: **删除文件**：弹出确认框"确定删除此文件及其全部批注吗？"，删除后自动切换到剩余文件列表中的第一个；如果删完了则自动新建默认文件
- R10: 删除文件时，**GitHub 仓库内的对应 .md 和 .annotations.json 文件不删除**（需要后续 GitHub Sync 手动处理或二期扩展）

### R4 localStorage 存储架构（方案 A：索引 + 独立文件）
- R11: **两层 key 设计**，避免单 JSON 过大 + 文件隔离：
  - `doc-branch-index-v1` = `{ currentFileId, files: [{id, name, docPath}] }`
  - `doc-branch-files/{id}` = `{ docContent, annotations, collapsedIds, splitRatio, githubSha? }`
- R12: 切换文件时只读写对应文件的 key，不触碰其他文件数据
- R13: 新增 localStorage 容量监控：接近 5MB 上限时在工具栏提示"建议导出备份"

### R5 GitHub Sync 适配（与二期 GitHub Sync 联动）
- R14: 一键提交时，遍历所有文件，每个文件提交一对：
  - `docPath`（如 `docs/rules.md`）= 当前 Monaco 的 docContent
  - `annotations/{文件名}.annotations.json` = 当前文件的完整批注数据
- R15: 同步时也提交一份 `docbranch-index.json` 到仓库根目录，记录文件索引（方便工具下次加载时识别有哪些文件）
- R16: 从远端拉取时，先 GET `docbranch-index.json` 确定文件列表，再逐个 GET 每个文件的 md + annotations.json

### R6 导入导出适配
- R17: 手动导出时支持两种模式：**导出当前文件**（单个 annotations.json）和 **导出全部**（打包为 zip 或一个 index.json + 各文件 JSON 的嵌套结构）
- R18: 手动导入时自动识别文件类型：单个 annotations.json → 合并到当前文件；index.json → 批量导入全部文件

## 验收标准

- [ ] 首次打开工具，自动创建"未命名文档"
- [ ] 新建文件成功，切换到新文件，Monaco 显示 DEFAULT_DOC
- [ ] 切换到另一个文件再切回来，docContent 和 annotations 都完整保留
- [ ] 删除文件后自动切换到剩余文件
- [ ] 刷新页面，恢复上次的 currentFile + 所有文件的 docContent/annotations
- [ ] 新建 3 个文件后 localStorage 容量不超限（每文件 < 200KB 时总量 < 1MB）
- [ ] GitHub Sync 遍历所有文件正确提交

## 非目标

- 文件内容 diff / 版本对比（三期）
- 跨文件批注关联（如"候选号码"在 rules.md 和 api.md 同时出现，只批注一处自动同步另一处）
- 文件导入支持本地文件夹扫描（只支持手动逐个新建或导出/导入）
- localStorage IndexedDB 迁移（先保留简化方案，未来容量不够再升级）

## 技术约束

- 文件 id 用 UUID v4（复用一期内联的 uuidv4()）
- docPath 格式校验：必须匹配 `[\w\-/]+.md`，防止路径遍历攻击
- 文件切换时 Monaco `setValue()` 会触发大量 `didChangeModelContent` 事件 → 需要**切换期间暂时关闭 debounceSave**（加一个 `state.switchingFile = true` 标志位，debounceSave 检查这个标志位跳过）
- 工具栏宽度：文件切换器（下拉 + 新建按钮）总宽约 240px，现有工具栏布局需要微调

## 二期排序

**优先级：最高**（二期第一个实现，因为其他二期功能如 GitHub Sync、目录树、SVG 连线都依赖多文件架构）

1. **多文件支持（本 spec）** ← 二期第 1 个
2. GitHub Sync（feature-github-sync）← 二期第 2 个（依赖多文件的文件列表）
3. 目录树侧边栏（feature-directory-tree）← 二期第 3 个（依赖 docPath 字段，纯 UI 推导）
4. SVG 虚线指引连线（feature-guideline-lines）← 二期第 4 个（纯 UI，不依赖前两个）
5. 拖拽分割线（feature-split-resize）← 二期第 5 个（纯 UI）
6. 批注标签系统（feature-annotation-tags）← 二期第 6 个
7. 批注卡片编辑（feature-card-edit）← 二期第 7 个（最复杂，偏移重算风险高）

## 相关

- 一期依赖: specs/feature-annotation.md（R17 localStorage、R19/R20 导入导出）
- 联动: specs/feature-github-sync.md（R14-R16 适配多文件）
- PRD 章节: docs/需求.md 7.2 二期迭代
