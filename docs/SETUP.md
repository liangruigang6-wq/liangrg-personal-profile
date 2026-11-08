# 一次性配置清单

本文件列出「本地文件已经准备好、但必须在 GitHub 网页或授权命令行中完成」的步骤。当前环境没有 GitHub 凭据，因此这些步骤尚未执行。

> 约定：下文 `<用户名>` 需要替换为本人 GitHub 用户名。当前会话未获知该用户名，此处不作推测。

## 关于「连接失败」的处理（2026-11-08 核实）

本次尝试推荐安装 GitHub 连接器时返回 `on-demand install failed: fetch failed`，该连接器至今未出现在已安装列表中。本机诊断结果如下：

- 可验证的事实：本机设置了指向 `127.0.0.1` 的本地代理；`api.github.com`、`github.com`、`cli.github.com`、`registry.npmjs.org` 均返回 HTTP 200；`github.com` 的 DNS 解析正常；`winget` 可用。
- 由上述事实可以判断：到 GitHub 的网络链路正常，失败发生在 WorkBuddy 连接器插件包的按需拉取环节，不在 GitHub 侧。
- 无法查证的部分：插件市场服务的具体地址与失败根因，本次没有取得服务端日志，因此不作结论。

因此**不要继续重试连接器安装**。下面的方式 A 与方式 B 都不依赖该连接器。

---

## 步骤 1：创建公开仓库

两种方式任选其一。

### 方式 A：网页一键预填（推荐，无需安装工具）

登录 GitHub 后打开：

```text
https://github.com/new?name=liangrg-personal-profile&visibility=public&owner=@me&description=Cross-AI%20personal%20profile
```

核对 Owner 为本人账户、Visibility 为 **Public**，然后点 Create repository。

依据：GitHub 官方文档说明，`github.com/new` 支持 `name`、`description`、`visibility`、`owner` 等查询参数预填表单，登录后可用 `@me` 指代本人账户 (GitHub, "Creating a new repository")。仓库名规则为不超过 100 字符，且仅可包含 ASCII 字母、数字与 `.` `-` `_` (同上)。

**不要勾选** "Add a README file"、.gitignore 或 license，因为本地已经准备好这些内容；勾选会与后续推送产生分叉。该注意事项在官方文档中亦有说明：导入既有仓库时不要预填这些可选项 (同上)。

### 方式 B：GitHub CLI

本机目前没有 `gh`，但 `winget` 可用。安装系统软件需要本人确认，本次未执行：

```bash
winget install --id GitHub.cli
```

安装后重新打开终端，先执行 `gh auth login`，按提示在浏览器完成账户授权（这一步必须由本人操作，我无法代为登录），再创建并推送仓库：

```bash
gh auth login
gh repo create liangrg-personal-profile \
  --public \
  --description "Cross-AI personal profile" \
  --source=. \
  --remote=origin \
  --push
```

依据：`gh repo create` 支持 `--public`、`--description`、`--source`、`--remote`、`--push`；`--source` 指定本地仓库目录，`--push` 推送本地已有提交，`--remote` 指定远端名称 (GitHub CLI, "gh repo create")。

注意：CLI 手册提供 `--disable-issues`，但未提供 `--enable-issues` 或任何 Projects 相关参数 (同上)。也就是说，Issues 与 Projects 需要在步骤 2 单独确认。

---

## 步骤 2：启用待办跟踪

### 2.1 Issues（本地模板已备好）

仓库创建后进入 **Settings → General → Features**，确认 **Issues** 已勾选。推送本地 `.github/ISSUE_TEMPLATE/` 后，新建 Issue 时会出现「档案条目补充 / 待办跟踪 / 档案勘误」三个选项。

建立标签（用于模板中的 `labels` 字段）：

```bash
gh label create "profile"    --description "档案条目补充" --color "0E8A16" --repo <用户名>/liangrg-personal-profile
gh label create "todo"       --description "待办跟踪"     --color "FBCA04" --repo <用户名>/liangrg-personal-profile
gh label create "correction" --description "档案勘误"     --color "D93F0B" --repo <用户名>/liangrg-personal-profile
```

建立三条种子待办：

```bash
gh issue create --repo <用户名>/liangrg-personal-profile \
  --title "[待办] 填充第 1 章个人简介与背景" --label todo \
  --body "删除 [示例] 行，补入可公开的稳定事实，并标记 [已确认]。"

gh issue create --repo <用户名>/liangrg-personal-profile \
  --title "[待办] 确认沟通风格偏好并统一到各 AI 助手" --label todo \
  --body "核对第 3 章，把确认后的条目同步到各 AI 助手的自定义指令。"

gh issue create --repo <用户名>/liangrg-personal-profile \
  --title "[待办] 补充常用工具与技术栈" --label todo \
  --body "核对第 4 章，列出工具名称与用途。"
```

### 2.2 Projects（可选，需网页操作）

GitHub Projects 属于**用户或组织层级**，不是仓库内的文件，因此无法靠推送本地文件创建；`gh repo create` 也没有对应参数。官方文档将 project 描述为"在用户或组织层级与 issues 和 pull requests 集成"的计划跟踪工具 (GitHub, "About Projects")。

创建路径：GitHub 右上角头像 → **Your projects** → **New project** → 选 Board 看板 → 命名 `个人档案维护` → 建立 `Todo / In Progress / Done` 三列 → 用 "Add item from repository" 关联本仓库的 Issues。

若只需最小可用方案，用 2.1 的 Issues + 标签即可满足待办跟踪，Projects 可以之后再加。

---

## 步骤 3：保护版本历史

进入 **Settings → Branches → Add branch protection rule**，Branch name pattern 填 `main`，建议勾选：

- **Require a pull request before merging** → Required approvals: 1
- **Require linear history**
- **Do not allow bypassing the above settings**

依据：GitHub 官方文档说明，分支保护规则默认即禁止 force push 并阻止删除受保护分支；`Require linear history` 会禁止 merge commit，并指出线性历史"有助于更容易地回退变更" (GitHub, "About protected branches")。

配合 [CONTRIBUTING.md](../CONTRIBUTING.md) 的提交规范，即可做到每次修改都有独立提交、可 `git log` 追溯、可 `git revert` 回退。

---

## 步骤 4：本地同步

### 情形 A：使用本次已准备好的本地仓库

```bash
cd D:/2026-11-08-19-46-13/liangrg-personal-profile

# 先改成自己的 GitHub 身份（当前是占位身份，见下方说明）
git config user.name "你的 GitHub 用户名"
git config user.email "你的 GitHub 绑定邮箱"
git commit --amend --reset-author --no-edit

git remote add origin https://github.com/<用户名>/liangrg-personal-profile.git
git branch -M main
git push -u origin main
```

> 本地初始提交目前使用占位作者 `liangrg <local-setup@example.com>`，因为本机 `git config` 中没有配置身份。上面第一条命令与 `--amend --reset-author` 会在首次推送前把它改成你自己的身份。此操作仅改写**尚未推送**的本地提交，不影响远端历史。

### 情形 B：在另一台电脑上从零开始

```bash
git clone https://github.com/<用户名>/liangrg-personal-profile.git
cd liangrg-personal-profile
git config user.name "你的 GitHub 用户名"
git config user.email "你的 GitHub 绑定邮箱"
```

之后的日常循环：

```bash
git pull --rebase origin main     # 开改前先同步
# 编辑 README.md 的某一章节，并追加 CHANGELOG.md
git add -A
git commit -m "docs(1-个人简介与背景): 补充语言与时区"
git push origin main
```

### 情形 C：只把档案内容给 AI 读取

无需克隆。把 README 的 raw 地址交给 AI 助手即可：

```text
https://raw.githubusercontent.com/<用户名>/liangrg-personal-profile/main/README.md
```

---

## 未完成事项确认表

| 项目 | 状态 | 说明 |
| --- | --- | --- |
| 本地仓库初始化 + 初始提交 | 已完成 | 见 `git log` |
| README 六章节与示例条目 | 已完成 | 见 `README.md` |
| Issues 模板文件 | 已完成（本地） | 推送后生效 |
| 远端公开仓库创建 | **未完成** | 需要本人授权，步骤 1 |
| Issues / Projects 启用 | **未完成** | 需要本人授权，步骤 2 |
| 分支保护规则 | **未完成** | 需要本人授权，步骤 3 |
| 首次推送 | **未完成** | 需要本人授权，步骤 4 |
