# 配置记录与本地同步指南

本文件记录本仓库的一次性配置过程与实际结果。全部步骤已于 2026-11-08 完成，下方命令保留供日后复现或排查。

> 实际账户：`liangruigang6-wq`。下文复现命令仍用 `<用户名>` 占位，换账户时替换即可。

## 一、结果速查

| 项目 | 实际结果 |
| --- | --- |
| 仓库地址 | https://github.com/liangruigang6-wq/liangrg-personal-profile |
| 可见性 | Public |
| 默认分支 | `main` |
| Issues | 已开启 |
| 待办看板 | https://github.com/users/liangruigang6-wq/projects/1 |
| 看板视图 | `View 1`（表格）、`待办看板`（Board） |
| Status 字段 | Todo / In Progress / Done |
| 自定义标签 | `profile`、`todo`、`correction` |
| 种子待办 | Issue #1、#2、#3，三条均已关联看板 |
| 分支保护 | 禁止 force push、禁止删除、要求线性历史、要求会话解决、对管理员同样生效；**未启用** PR 审批 |
| 提交作者 | `liangruigang6-wq <354081825+liangruigang6-wq@users.noreply.github.com>` |

## 二、连接失败的处理（2026-11-08 核实）

尝试推荐安装 GitHub 连接器时返回 `on-demand install failed: fetch failed`，该连接器始终未出现在已安装列表中。本机诊断结果如下：

- 可验证的事实：本机设置了指向 `127.0.0.1` 的本地代理；`api.github.com`、`github.com`、`cli.github.com`、`registry.npmjs.org` 均返回 HTTP 200；`github.com` 的 DNS 解析正常；`winget` 可用。
- 由上述事实可以判断：到 GitHub 的网络链路正常，失败发生在 WorkBuddy 连接器插件包的按需拉取环节，不在 GitHub 侧。
- 无法查证的部分：插件市场服务的具体地址与失败根因，本次没有取得服务端日志，因此不作结论。

最终没有重试连接器，改走本机 GitHub CLI；后续创建、推送与配置全部成功。

## 三、实际执行的命令

### 1. 安装 CLI 与账户授权

```bash
winget install --id GitHub.cli --accept-package-agreements --accept-source-agreements --silent
GH_PROMPT_DISABLED=1 gh auth login --web --git-protocol https --hostname github.com --scopes "repo,project,read:org"
```

第一步安装到 `C:\Program Files\GitHub CLI\gh.exe`。第二步会打印一次性验证码与 `https://github.com/login/device`，**必须由账户本人在浏览器完成授权**，我无法代劳。授权后 `gh auth status` 显示令牌作用域为 `gist`、`project`、`read:org`、`repo`。

### 2. 推送前修正提交作者

本地前两次提交最初使用占位作者。因为尚未推送，改写本地历史是允许的，也符合 [CONTRIBUTING.md](../CONTRIBUTING.md) 的约定。 noreply 邮箱格式 `ID+USERNAME@users.noreply.github.com` 见 (GitHub, "Email Addresses Reference")。

```bash
git config user.name "liangruigang6-wq"
git config user.email "354081825+liangruigang6-wq@users.noreply.github.com"
git rebase --root --exec 'git commit --amend --reset-author --no-edit'
```

推送后用 `gh api repos/.../commits` 核实，两次提交的 `author.login` 均为 `liangruigang6-wq`。

### 3. 创建公开仓库并推送

```bash
gh repo create liangrg-personal-profile \
  --public \
  --description "Cross-AI personal profile" \
  --source=. \
  --remote=origin \
  --push
```

`--source` 指定本地仓库目录，`--push` 推送已有提交，`--remote` 指定远端名称 (GitHub CLI, "gh repo create")。手册提供 `--disable-issues`，但未提供 `--enable-issues` 或 Projects 相关参数 (同上)，因此创建后单独回读了仓库设置，确认 `hasIssuesEnabled` 与 `hasProjectsEnabled` 均为 `true`。

网页复现路径：`https://github.com/new?name=liangrg-personal-profile&visibility=public&owner=@me`。该页支持 `name`、`description`、`visibility`、`owner` 查询参数预填，登录后可用 `@me` 指代本人账户 (GitHub, "Creating a New Repository")。导入既有仓库时**不要**勾选 Add a README，否则会与本地推送产生分叉 (同上)。

### 4. Issues 标签与种子待办

```bash
R="liangruigang6-wq/liangrg-personal-profile"
gh label create "profile"    --description "档案条目补充" --color 0E8A16 --repo "$R"
gh label create "todo"       --description "待办跟踪"     --color FBCA04 --repo "$R"
gh label create "correction" --description "档案勘误"     --color D93F0B --repo "$R"

gh issue create --repo "$R" --project "个人档案维护" --label todo \
  --title "[待办] 填充第 1 章个人简介与背景" --body "..."
```

`gh label create` 的 `--repo`、`--description`、`--color` 与 `gh issue create` 的 `--project`、`--label`、`--title`、`--body` 均已核对官方手册 (GitHub CLI, "gh label create"; GitHub CLI, "gh issue create")。`--color` 需要 6 位十六进制值 (同上)。

推送的 `.github/ISSUE_TEMPLATE/` 提供「档案条目补充 / 待办跟踪 / 档案勘误」三个新建 Issue 选项。

### 5. Projects 看板

GitHub Projects 属于**用户或组织层级**，不是仓库内文件，因此无法靠推送本地文件创建；官方文档将 project 描述为在用户或组织层级与 issues、pull requests 集成的计划跟踪工具 (GitHub, "About Projects")。本次通过 GraphQL 创建：

```bash
NODE_ID=$(gh api user --jq .node_id)
gh api graphql -f query="mutation { createProjectV2(input: {ownerId: \"$NODE_ID\", title: \"个人档案维护\"}) { projectV2 { url } } }"
gh api graphql -f query='mutation { createProjectV2View(input: {projectId: "<看板ID>", name: "待办看板", layout: BOARD_LAYOUT}) { projectV2View { id name layout } } }'
```

新建看板自带 `Status` 字段的 Todo / In Progress / Done 三个选项，另有一个表格视图；本次追加了 Board 布局的「待办看板」。

### 6. main 分支保护

```bash
printf '%s' '{"required_status_checks":null,"enforce_admins":true,"required_pull_request_reviews":null,"restrictions":null,"required_linear_history":true,"allow_force_pushes":false,"allow_deletions":false,"required_conversation_resolution":true}' \
  | gh api -X PUT "repos/$R/branches/main/protection" --input -
```

回读结果：`allow_force_pushes=false`、`allow_deletions=false`、`required_linear_history.enabled=true`、`enforce_admins.enabled=true`、`required_pull_request_reviews=null`。

官方文档说明，分支保护规则默认即禁止 force push 并阻止删除受保护分支；`Require linear history` 会禁止 merge commit，并指出线性历史有助于更容易地回退变更 (GitHub, "About Protected Branches")。

**关于未启用 PR 审批**：这是本次针对单人维护场景的配置取舍，不是 GitHub 官方要求。要求审批会让所有改动必须走 PR 流程，而单人仓库没有其他审阅人，容易把日常维护卡住。当前四项设置已能保证历史不被改写；若以后有协作者，再到 Settings → Branches 补上审批要求即可。

## 四、本地同步

### 本机（已配置好，直接日常使用）

`D:/2026-11-08-19-46-13/liangrg-personal-profile` 已绑定 `origin`，`main` 跟踪 `origin/main`，git 身份已设为账户 noreply 邮箱。

```bash
git pull --rebase origin main     # 开改前先同步
# 编辑 README.md 的某一章节，并追加 CHANGELOG.md
git add -A
git commit -m "docs(1-个人简介与背景): 补充语言与时区"
git push origin main
```

需要回退时用 `git revert <commit-id>` 生成反向提交，原历史完整保留；完整命令见 [CONTRIBUTING.md](../CONTRIBUTING.md)。

### 另一台电脑

```bash
git clone https://github.com/liangruigang6-wq/liangrg-personal-profile.git
cd liangrg-personal-profile
git config user.name "liangruigang6-wq"
git config user.email "354081825+liangruigang6-wq@users.noreply.github.com"
```

### 只把档案内容交给 AI 读取

无需克隆，直接使用：

```text
https://raw.githubusercontent.com/liangruigang6-wq/liangrg-personal-profile/main/README.md
```

## 五、执行结果确认表

| 项目 | 状态 | 核实方式 |
| --- | --- | --- |
| 本地仓库初始化 + 提交 | 已完成 | `git log` |
| README 六章节与示例条目 | 已完成 | `README.md` |
| Issues 模板文件 | 已完成并生效 | 仓库 `.github/ISSUE_TEMPLATE/` |
| 远端公开仓库创建 | 已完成 | `gh repo view` 显示 `visibility=PUBLIC` |
| 首次推送 | 已完成 | `main` 跟踪 `origin/main` |
| 提交作者归属 | 已完成 | `gh api commits` 显示 `author.login` |
| Issues 标签与种子待办 | 已完成 | `gh label list`、`gh issue list` |
| Projects 看板与视图 | 已完成 | GraphQL 回读，3 条 Issue 已关联 |
| 分支保护 | 已完成 | `gh api branches/main/protection` 回读 |

## 引用出处（MLA）

GitHub. "About Projects." *GitHub Docs*, docs.github.com/en/issues/planning-and-tracking-with-projects/learning-about-projects/about-projects. Accessed 8 Nov. 2026.

GitHub. "About Protected Branches." *GitHub Docs*, docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/about-protected-branches. Accessed 8 Nov. 2026.

GitHub. "Creating a New Repository." *GitHub Docs*, docs.github.com/en/repositories/creating-and-managing-repositories/creating-a-new-repository. Accessed 8 Nov. 2026.

GitHub. "Email Addresses Reference." *GitHub Docs*, docs.github.com/en/account-and-profile/reference/email-addresses-reference. Accessed 8 Nov. 2026.

GitHub. "gh issue create." *GitHub CLI Manual*, cli.github.com/manual/gh_issue_create. Accessed 8 Nov. 2026.

GitHub. "gh label create." *GitHub CLI Manual*, cli.github.com/manual/gh_label_create. Accessed 8 Nov. 2026.

GitHub. "gh repo create." *GitHub CLI Manual*, cli.github.com/manual/gh_repo_create. Accessed 8 Nov. 2026.
