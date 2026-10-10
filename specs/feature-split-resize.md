# 功能：拖拽分割线（二期）

## 背景

一期左右栏比例固定为 60:40。不同用户场景下合适的比例不同：批注少时需要更多文档空间，批注多时需要看到更多右栏卡片。二期在两栏之间增加可拖拽分割线。

## 需求

### R1 基础拖拽
- R1: 左右栏之间 6px 宽的分割线区域，鼠标悬停时显示 `cursor: col-resize`；显示 2px 高亮竖线提示可拖拽
- R2: 按住左键拖拽时，实时改变两栏宽度比例（左栏百分比，右栏自动 100 - 百分比）
- R3: 比例范围：左栏 30% ~ 80%（防止某一栏被挤压到不可用）
- R4: 拖拽时 Monaco Editor 自动 `layout()` 重新计算布局（自动触发，因为监听了 container resize）

### R2 双击重置
- R5: 双击分割线，恢复默认比例 60:40

### R3 持久化
- R6: 分割线位置（百分比）随 localStorage 保存，key 复用 `doc-branch-snapshot-v1` 内新增 `splitRatio` 字段
- R7: 页面启动时恢复上次的分割线位置

### R4 无障碍
- R8: 拖拽过程中 `e.preventDefault()` 防止文字选中
- R9: 松开鼠标时用 `document.removeEventListener('mousemove', handler)` 和 `document.removeEventListener('mouseup', handler)` 清理监听器

## 验收标准

- [ ] 鼠标悬停分割线，光标变 col-resize，显示高亮提示线
- [ ] 拖拽分割线，两栏宽度实时变化，Monaco 不闪烁
- [ ] 左栏宽度不会小于 30% 或大于 80%
- [ ] 双击分割线恢复 60:40
- [ ] 刷新页面后分割线位置保持

## 非目标

- 记忆多组预设比例快捷切换
- 键盘方向键微调分割线位置
- 垂直分割线（上下栏调整）

## 技术约束

- 两栏容器用 `flex: 0 0 ${ratio}%` 控制宽度，或 CSS 变量 `--left-ratio` + `calc()` 计算
- 拖拽 handler 要 `stopPropagation` 不影响子元素（右键菜单、Monaco 本身）
- `ResizeObserver` 或 Monaco 的 `automaticLayout` 保证 Monaco 自动适配

## 相关

- 一期依赖: specs/feature-annotation.md（R1 双栏布局）
- PRD 章节: docs/需求.md 2 整体页面布局
