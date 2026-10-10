# 功能：GitHub API 一键提交（二期）

## 背景

一期的持久化方案是 localStorage + 手动导出 JSON。用户的批注数据要纳入 Git 版本管理，需要手动"导出 JSON → 复制到仓库 → commit → push"，流程繁琐。二期通过 GitHub REST API 实现一键将 `annotations-tree.json` 和 `doc.md` commit 回仓库。

## 需求

### R1 认证配置
- R1: 首次使用时弹出设置弹窗，用户输入 **GitHub Personal Access Token（PAT）** + **仓库全名**（格式 `owner/repo`）+ **分支名**（默认 `main`）+ **目标文件路径**（默认 `annotations-tree.json` 和 `doc.md`，可自定义）
- R2: PAT 仅需 `repo` scope；设置保存在 localStorage（key `doc-branch-github-config`），**不暴露到页面源码中**
- R3: 提供"测试连接"按钮，调用 `GET /repos/{owner}/{repo}` 验证 PAT 和仓库有效性

### R2 提交能力
- R4: 顶部工具栏新增「☁️ 一键提交」按钮；点击后：
  1. 遍历 `doc-branch-index-v1` 中的所有文件
  2. 对每个文件，调用 GitHub API 获取两个文件的当前 SHA：`docPath` 和 `annotations/{文件名}.annotations.json`
  3. 构造 commit 信息，**批量提交**所有文件（用一个 commit 包含所有变更，GitHub REST API v3 不支持一次改多文件，所以需要多次 PUT，或用 Git Data API 的 refs + trees 方案做真正的批量）
  4. 同时提交一份 `docbranch-index.json` 到仓库根目录，记录文件索引
- R5: commit message 自动生成为 `docbranch: 同步 {N} 个文件批注 {时间戳}`，可在弹窗中编辑
- R6: 提交成功后显示 ✅ + 提供 GitHub commit 链接；失败显示具体哪个文件的错误（PAT 无效 / SHA 冲突 / 网络错误）

### R3 冲突处理
- R7: 如果 `409 Conflict`（SHA 冲突，说明远端文件已被他人修改）：
  - 弹窗提示「远端有更新」，提供三个选项：**强制覆盖**、**先拉取再覆盖**（GET 最新 SHA 后再 PUT）、**取消**
- R8: 如果本地配置里记录的 SHA 与远端一致（无他人改动），直接提交（乐观更新）

### R4 拉取能力（可选）
- R9: 顶部工具栏新增「📥 从远端拉取」按钮；调用 GET contents 接口获取最新 annotations + docContent，与本地合并或覆盖（弹窗选择）

## 验收标准

- [ ] 首次使用弹出设置弹窗，输入 PAT + repo 后「测试连接」按钮验证通过
- [ ] 点击「一键提交」，两个文件成功 push 到 GitHub 仓库
- [ ] commit message 自动生成，可编辑
- [ ] PAT 无效时显示明确错误信息
- [ ] 刷新页面后配置仍在，无需重新输入

## 非目标

- OAuth 授权流程（只用 PAT，简单直接）
- 自动定时提交（只支持用户手动触发）
- 多人协作合并冲突解决（只处理简单 SHA 冲突）
- GitHub Enterprise 支持（只支持 github.com）
- 真正的批量 commit（GitHub Contents API 每次 PUT 一个文件；Git Data API 可批量但复杂度高，二期先用多次 PUT）

## 技术约束

- 使用 GitHub REST API v3：`https://api.github.com`
- 浏览器直连 API（CORS 允许），无需后端代理
- PAT 保存在 localStorage，用户需自行注意安全（public 电脑不要保存）
- base64 编码用 `btoa(unescape(encodeURIComponent(str)))` 兼容 UTF-8
- **仓库目录结构约定**（与多文件 spec 配合）：
  ```
  数据仓库（Private，GitHub Sync 读写目标）/
  ├── docs/*.md                    ← 用户原文档（docPath 指定）
  ├── annotations/*.annotations.json ← 每个文件的批注（自动生成）
  └── docbranch-index.json         ← 文件索引（工具识别）

  壳仓库（Public 或 Private，GitHub Pages 源）/
  └── index.html                   ← 工具页面（零数据，纯前端代码）
  ```
- **双仓库架构**：工具壳（index.html）和数据（docs/ + annotations/）分两个仓库存放。壳仓库用于 GitHub Pages 托管公网访问；数据仓库设为 Private，GitHub Sync 通过 PAT + Contents API 单独读写。代码层面无硬编码仓库名，完全由用户在 ⚙️ 设置里配置。

## 部署方案

### 方案 A：GitHub Pro（单仓库）
最干净，$4/月。Private 仓库直接开 Pages，index.html + docs + annotations 全部在同一个仓库。

### 方案 B：GitHub Free（双仓库，推荐）
Free 账号 Pages 只能从 Public 仓库构建 → 壳 Public + 数据 Private 分离。步骤：

1. 创建 **仓库 A（Public）** 命名如 `docbranch-host` → Settings → Pages → Source main 根目录 → Save
2. 创建 **仓库 B（Private）** 命名如 `docbranch-data` → 空仓即可
3. 把本地 `index.html` 拖 GitHub 网页上传到仓库 A（一次就行）
4. 打开 `https://用户名.github.io/docbranch-host/` → ⚙️ 设置：
   - PAT：针对 **仓库 B** 生成的 fine-grained token（Contents Read and Write + Metadata Read）
   - 仓库全名：`用户名/docbranch-data`（**仓库 B，不是 A！**）
   - 分支名：`main`
5. 之后所有一键同步/拉取操作自动走仓库 B，完全不感知双仓库架构

### 方案 C：Cloudflare Pages（双仓库，国内更快）
Cloudflare Pages 免费计划支持从 GitHub Private 仓库构建，一个仓库搞定，且自带 Password Protection。

## 相关

- 一期依赖: specs/feature-annotation.md（R17 localStorage 持久化格式）
- **强依赖**: specs/feature-multi-file.md（文件列表来自 `doc-branch-index-v1`，切换文件保存不同 key 下的数据）
- 二期排序: 多文件（第 1 个）→ GitHub Sync（第 2 个）
- PRD 章节: docs/需求.md 6.2 二期预留
