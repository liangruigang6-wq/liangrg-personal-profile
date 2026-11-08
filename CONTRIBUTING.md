# 修改规范：可追溯、可回退

本仓库把「每一次修改都能查到、都能退回」作为硬性要求。以下约定只依赖 Git 自身能力，不需要额外服务。

## 一、提交粒度

1. 一次提交只改一个章节或一个主题，不把多处无关改动混在一起。
2. 提交信息采用固定格式：

   ```text
   <类型>(<章节>): <一句话说明>

   # 示例
   docs(3-沟通风格偏好): 补充引用标注要求
   chore(issue-template): 新增待办模板
   ```

   常用类型：`feat`（新增结构）、`docs`（档案内容）、`fix`（纠错）、`chore`（模板与配置）。

3. 提交信息写「改了什么、为什么改」，不写「更新」「修改」这类无法追溯的说明。

## 二、禁止破坏历史

1. 不对 `main` 执行 `git push --force` 或 `git push --force-with-lease`。
2. 不执行改写已推送历史的 `git rebase` 与 `git commit --amend`（仅限尚未推送的本地提交可整理）。
3. 远端 `main` 建议设置分支保护，见 [docs/SETUP.md](docs/SETUP.md)。GitHub 官方文档说明，分支保护规则默认即禁止 force push 并阻止删除受保护分支（GitHub, "About protected branches"）。

## 三、追溯与回退的常用命令

```bash
# 查看完整变更历史（每次修改的作者、时间、说明）
git log --oneline --decorate --all

# 查看某次提交改了哪些行
git show <commit-id>

# 查看某个文件每一行的来源
git blame README.md

# 回退方式 A（推荐，用于已推送的历史）：新增一次「反向提交」，原历史完整保留
git revert <commit-id>
git push origin main

# 回退方式 B（仅限尚未推送的本地提交）：移动分支指针，会丢弃后续提交
git reset --hard <commit-id>

# 临时查看旧版本，不改当前分支
git checkout <commit-id> -- README.md
```

选择依据：`git revert` 不改写历史，适合公开仓库与多人协作；`git reset --hard` 会丢弃提交，只应作用于本地未推送的内容。

## 四、打标签固化版本

每次完成一轮较大填充后打一个标签，便于 AI 或本人直接引用某一版档案：

```bash
git tag -a v0.2 -m "完成六个章节首轮填充"
git push origin v0.2
```

## 五、纠错流程

发现档案条目写错时，不要直接静默改掉。请开一个「档案勘误」Issue，说明错误条目、正确内容与依据，再按上面的提交规范修改，并在提交信息中引用 Issue 编号。
