# 功能：移动端阅读模式

## 背景

当前文档递归批注工具仅适配桌面端（左右分栏布局），手机浏览器打开时布局严重错乱，Monaco 编辑器在小屏上体验差。需要新增移动端阅读模式，让用户能在手机上正常查看文档和批注。

## 需求

### 布局与触发

- R1: 当视口宽度 ≤ 768px 时，自动进入移动端阅读模式（通过 CSS @media 触发）
- R2: 移动端布局将 Monaco 编辑器**完全隐藏**，改用已有的 simpleMarkdown 渲染文档内容
- R3: 移动端布局将桌面端右侧批注栏移到底部可折叠面板，默认展开约 50% 高度，可拖拽手柄展开到全屏或收起至仅标题条
- R4: 移动端侧边栏（目录树）改为汉堡菜单触发的全屏抽屉
- R5: 桌面端所有布局、样式、交互**零改动**，移动端样式全部限制在 @media (max-width: 768px) 作用域内

### 顶部工具栏

- R6: 移动端顶部工具栏简化为：汉堡按钮、品牌名、搜索框、设置按钮，隐藏不常用按钮（新建批注、批量导入导出等移到设置菜单内或省略）

### 批注查看与跳转

- R7: Markdown 渲染区域需在原文高亮标注已有批注位置（用黄色背景），点击高亮区域可跳转到对应批注卡片
- R8: 点击批注卡片上的锚点按钮可滚动文档到批注对应的原文位置

### 新建批注（触屏）

- R9: 支持触屏选中文档文字后弹出浮动"➕ 添加批注"按钮，点击后打开批注输入框
- R10: 触屏新建批注时，note 偏移量定位逻辑需适配从 Markdown 渲染层获取文本偏移（非 Monaco）

### 批注卡片

- R11: 底部面板内批注卡片样式适配窄屏（去掉 hover 效果、增大点击区域、字号微调）
- R12: 标签筛选、搜索功能在移动端保留，UI 简化为底部面板内的筛选条

### 功能取舍（移动端不支持）

- R13: 移动端**隐藏** Monaco 编辑器相关功能（文档编辑），仅支持查看
- R14: 移动端**隐藏**虚线指引连线功能（触屏复杂度过高）
- R15: 移动端**隐藏**拖拽分割条（桌面端的 col-resize splitter）

### 保留功能

- R16: GitHub 同步功能保留，通过设置弹窗访问
- R17: localStorage 自动保存、JSON 导入导出保留
- R18: 批注编辑（双击卡片）、子批注、标签修改保留

## 验收标准

- [ ] 视口 ≤ 768px 时自动切换移动端布局，≥ 769px 时恢复桌面布局，无闪烁
- [ ] 桌面端所有已有功能、样式、交互行为无回归（逐一回归测试）
- [ ] 移动端底部批注面板可正常拖拽展开/收起，拖拽流畅不卡顿
- [ ] 移动端汉堡菜单可正常打开/关闭全屏目录抽屉
- [ ] 移动端选中文档文字后出现浮动"➕ 添加批注"按钮，可完成新建批注流程
- [ ] 已有批注在文档渲染区以黄色背景高亮，点击可跳转对应卡片
- [ ] 虚拟键盘弹出时布局不被遮挡，不出现异常滚动（safe-area + viewport 适配）
- [ ] iPhone 刘海屏/底部 Home Indicator 区域内容不被遮挡（safe-area-inset）
- [ ] 虚线指引、分割条、Monaco 编辑功能在移动端确认隐藏

## 非目标

- 移动端文档编辑（Monaco）不在本期
- 移动端虚线指引连线（SVG 坐标 + 触屏交互复杂）不在本期
- 新增 CDN 依赖或构建工具
- App 打包（仍为纯网页，GitHub Pages 部署）

## 技术约束

- 纯 CSS @media (max-width: 768px) 隔离移动端样式，**不得污染全局**
- 所有改动限制在单个 index.html 文件内
- 复用已有 simpleMarkdown 渲染函数，不引入新的 Markdown 库
- 触屏新建批注的 note 偏移定位：从 Markdown 渲染 div 的 innerHTML 中查找锚点文本计算偏移
- 底部面板拖拽使用 touch 事件（touchstart/touchmove/touchend），不引入拖拽库
- viewport meta 设置：`<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">`
- safe-area 适配：`env(safe-area-inset-top)` / `env(safe-area-inset-bottom)` 用于顶部状态栏和底部 Home Indicator

## 相关

- 复用: simpleMarkdown 渲染函数（已存在于 index.html）
- 复用: 批注数据模型、持久化逻辑（localStorage）
- 复用: 标签、子批注、GitHub 同步等已有功能模块
- 非依赖: 不修改 specs/feature-annotation.md、feature-directory-tree.md 等已有 spec
