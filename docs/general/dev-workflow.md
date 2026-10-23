# 🛠 开发工作流 / Dev Workflow

> 让「改代码 → 提交 → 审查 → 合并」这条链路少一点鼠标、多一点可重复。
> 本节的判据是：能不能被**写进脚本或 CI**，而不只是让本地体验好看。

## 精选 / Top picks

- **[gh](https://github.com/cli/cli)** —— GitHub 官方 CLI，把 PR/issue/release/
  Actions 全部搬进终端，`gh api` 还能直接打任意接口。**代价**：⚠️ 需登录，
  且行为强绑定 GitHub（GitLab 用 `glab`）。
- **[lazygit](https://github.com/jesseduffield/lazygit)** —— Git 的 TUI，
  暂存单个 hunk、交互式 rebase、冲突解决比命令行直观得多。**代价**：TUI 无法
  脚本化，自动化场景仍要会原生 `git`。
- **[just](https://github.com/casey/just)** —— 命令运行器，比 `Makefile` 少了
  「文件时间戳」这套心智负担，适合当项目的「任务入口」。**代价**：
  `Makefile` 的依赖图能力它刻意不做。
- **[pre-commit](https://github.com/pre-commit/pre-commit)** —— 把 lint/format
  统一成提交前钩子，跨语言、可固定版本。**代价**：首次运行要建隔离环境，
  离线环境需要预拉取。

## 全部条目 / All entries

| Tool | 中文说明 | English | Tags | License |
| --- | --- | --- | --- | --- |
| [gh](https://github.com/cli/cli) | GitHub 官方 CLI：PR、issue、release、Actions、任意 API 调用 | Official GitHub CLI for PRs, issues, releases, Actions and raw API calls | `go`, `git`, `ci`, `github`, `requires-login` | MIT |
| [glab](https://gitlab.com/gitlab-org/cli) | GitLab 官方 CLI，与 `gh` 对应，覆盖 MR 与流水线 | Official GitLab CLI, the `gh` counterpart covering MRs and pipelines | `go`, `git`, `ci`, `gitlab`, `requires-login` | MIT |
| [lazygit](https://github.com/jesseduffield/lazygit) | Git 的终端 UI，按 hunk 暂存、交互式 rebase 与冲突处理 | Terminal UI for Git: hunk-level staging, interactive rebase and conflict resolution | `go`, `git`, `tui` | MIT |
| [gitui](https://github.com/gitui-org/gitui) | Rust 实现的 Git TUI，启动快，适合大仓库 | Rust Git TUI with fast startup, suited to large repositories | `rust`, `git`, `tui` | MIT |
| [tig](https://github.com/jonas/tig) | 以 ncurses 界面的方式浏览提交历史与 blame | ncurses interface for browsing commit history, blame and diffs | `c`, `git`, `tui` | GPL-2.0 |
| [difftastic](https://github.com/Wilfred/difftastic) | 语法感知的结构化 diff，能看出「换了变量名」而不是整行重写 | Syntax-aware structural diff that understands renames instead of whole-line rewrites | `rust`, `git`, `diff` | MIT |
| [just](https://github.com/casey/just) | 项目命令运行器，`justfile` 比 Makefile 更贴近日常脚本 | Project command runner with a recipe file friendlier than Makefile | `rust`, `task`, `automation` | CC0-1.0 |
| [Task](https://github.com/go-task/task) | 用 YAML 描述任务与依赖，跨平台、内置并行与文件指纹 | YAML-described tasks with dependencies, cross-platform and fingerprint-aware | `go`, `task`, `automation`, `ci` | MIT |
| [direnv](https://github.com/direnv/direnv) | 进入目录自动加载/卸载环境变量，`.envrc` 随仓库走 | Loads and unloads env vars per directory via versioned `.envrc` | `go`, `env`, `automation` | MIT |
| [pre-commit](https://github.com/pre-commit/pre-commit) | 语言无关的 Git 提交钩子管理，可锁定工具版本 | Language-agnostic Git hook manager with pinned tool versions | `python`, `git`, `hooks`, `ci` | MIT |
| [lefthook](https://github.com/evilmartians/lefthook) | 单二进制的 Git 钩子管理器，并行执行、配置集中 | Single-binary Git hook manager with parallel execution and one config | `go`, `git`, `hooks` | MIT |
| [act](https://github.com/nektos/act) | 在本机用容器运行 GitHub Actions 工作流，缩短反馈环 | Runs GitHub Actions workflows locally in containers to shorten feedback loops | `go`, `ci`, `containers` | MIT |
| [Flisk](https://github.com/flisk-sh/flisk) | 一条命令从空目录生成并启动整套技术栈（框架、数据库、配置一次到位） | One command from an empty folder to a running full-stack setup (framework, database, config) | `node`, `scaffold`, `automation`, `cross-platform` | MIT |

## 组合用法 / Combinations

```bash
# 1) 把常用动作固化成 justfile，新人照抄即可
# justfile:
#   test: ; pnpm test
#   lint: ; pnpm lint
just test

# 2) gh + jq：批量看等待我 review 的 PR
gh pr list --search 'review-requested:@me' --json number,title,author \
  | jq -r '.[] | "\(.number)\t\(.author.login)\t\(.title)"'

# 3) 变更很乱时，用语法感知 diff 看清语义变化
git config --global diff.external difft
git difftool -y HEAD~1

# 4) 钩子统一交给 pre-commit，避免每人装不同的工具
pre-commit install && pre-commit run --all-files

# 5) 改 CI 前先本地验一遍，减少 push 等待
act pull_request -j test
```

## 注意事项 / Notes

- ⚠️ `gh` / `glab` 需要登录并保存 token；在共享机器上注意凭据存储位置
  （见 [安全与加密](security.md)）。
- **别让 TUI 成为唯一技能**。`lazygit` 很好用，但 CI 与远程服务器上只有 `git`；
  出问题时要能用原生命令复现。
- `pre-commit` 的钩子会**修改**你的暂存区（如 formatter）。它改动后需要重新
  `git add`，这一点常让人以为「钩子没生效」。
- `act` 的行为与真实 GitHub 运行器**不完全一致**（镜像、权限、缓存）。
  它用来快速试错可以，别当成 CI 的完全替代。
- `direnv` 的 `.envrc` 是**可执行代码**。克隆陌生仓库时先读一遍再 `direnv allow`。

## 相关 / See also

- [查看与预览](viewers.md) —— `delta` 让 diff 更易读
- [云与容器](cloud-and-containers.md) —— `act` 依赖容器运行时
- [安全与加密](security.md) —— 提交前扫描密钥
