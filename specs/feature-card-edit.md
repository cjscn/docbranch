# 功能：批注卡片编辑修改（二期）

## 背景

一期中批注的 note 字段一旦保存就无法修改，只能删除后重建。当用户输入有误或需要完善批注说明时，需要重新走整个"选中 → 右键 → 输入"流程。二期支持双击批注说明直接进入编辑态，原地修改 Markdown 内容。

## 需求

### R1 双击进入编辑
- R1: 双击卡片内的批注说明区域（`.note-content`）进入编辑态；编辑态使用可编辑 `<textarea>` 或 Monaco mini editor（markdown 模式，只读 Markdown 语法高亮）
- R2: 编辑时显示 `Ctrl+Enter` 保存、`Esc` 取消 的提示
- R3: 编辑期间不允许再次双击其他卡片；`Esc` 或点击外部空白处取消编辑

### R2 保存与校验
- R4: 保存时自动重新渲染 Markdown（调用现有 `simpleMarkdown()`），刷新卡片显示
- R5: 保存后触发 Storage.save()，更新 localStorage
- R6: 如果修改了一级批注的 note，其下所有子批注的 `noteOffsetStart/End` **需要重新计算**（因为父 note 文本长度变了）——**这是核心难度**
- R7: 重新计算方案：修改后遍历 `parent.children`，用 `child.sourceText` 在新 note 中 `indexOf` 查找新偏移；如果某子批注的 sourceText 在新 note 中找不到，该子批注标记为 `stale` 并在卡片上提示"父批注文本已变更，需手动调整偏移"

### R3 行内高亮标记更新
- R8: 父批注 note 修改后，子批注在 note 中的高亮 `<mark class="sub-anchor">` 位置需要同步重建（调用 `_applySubAnnotationMarks`）

## 验收标准

- [ ] 双击批注说明区域，进入编辑态
- [ ] 编辑后 Ctrl+Enter 保存，显示更新后的 Markdown 渲染
- [ ] Esc 取消编辑，恢复原内容
- [ ] 修改父批注 note 后，子批注的 noteOffset 自动更新（sourceText 在新 note 中能找到的情况）
- [ ] 修改后 localStorage 数据更新

## 非目标

- 编辑 sourceText（原文片段）—— sourceText 只能由「确认更新」按钮更新
- 实时预览（只在保存后重新渲染）
- 多人并发编辑冲突解决

## 技术约束

- 编辑控件用原生 `<textarea>`（Monaco mini editor 虽然更优雅但增加复杂度，二期先用 textarea）
- 偏移重算逻辑封装为 `Annotations.recalcChildOffsets(parentId)` 独立方法
- 编辑态的 CSS 过渡（圆角、边框高亮）

## 相关

- 一期依赖: specs/feature-annotation.md（R9 卡片渲染、R12 noteOffset 定位、R26 折叠）
- 数据结构: docs/需求.md 4.2 节点结构
