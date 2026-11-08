# 变更记录

本文件是便于人阅读的摘要。具备法律意义与完整追溯能力的记录以 Git 提交历史为准（`git log`）。

格式：按版本倒序，每条写明日期、改动章节、改动说明。

## v0.1 — 2026-11-08

- `README.md`：初始化档案入口，建立六个章节与 `[示例]` / `[待补充]` / `[已确认]` 三种状态标记。
- `CONTRIBUTING.md`：建立提交粒度、禁止 force push、revert 回退与标签规范。
- `.github/ISSUE_TEMPLATE/`：建立档案补充、待办、勘误三类 Issue 模板。
- `docs/SETUP.md`：整理一次性配置清单（Issues、Projects、分支保护、本地同步）。
- `.gitignore`：排除本地编辑器与系统临时文件。

说明：本版全部为结构与示例内容，未包含任何真实个人档案条目。

## v0.1.1 — 2026-11-08

- `docs/SETUP.md`：记录 GitHub 连接器按需安装失败的诊断结论；补充 `winget install --id GitHub.cli` 路径；明确改走不依赖连接器的方式 A 与方式 B。

## v0.2 — 2026-11-08

- 远端公开仓库创建并推送：`https://github.com/liangruigang6-wq/liangrg-personal-profile`，默认分支 `main`。
- `README.md`：写入实际仓库地址、待办看板与 raw 地址；`owner` 由占位值更正为账户名 `liangruigang6-wq`。
- `docs/SETUP.md`：由待办清单改为配置记录，补入实际执行命令、执行结果，以及「未启用 PR 审批」的取舍说明。
- 两次本地提交的作者由占位身份更正为账户 noreply 身份，远端提交已核实归属到 `liangruigang6-wq`。
- 远端配置：Issues 开启；新增 `profile` / `todo` / `correction` 标签；建立 Issue #1~#3 并关联 Projects 看板「个人档案维护」（表格视图 + 看板视图，Status 为 Todo / In Progress / Done）；`main` 分支保护生效。

