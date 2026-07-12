---
author: "星陨"
pubDatetime: 2026-07-11T15:39:00
title: "约定式提交：让 Git 历史会说话"
tags:
  - "Git"
  - "开发规范"
description: "介绍 Conventional Commits 约定式提交规范，通过结构化的提交信息让 Git 历史清晰可读，实现 CHANGELOG 自动生成和语义化版本管理。"
---

> 写代码的时候，我们花大量时间讨论变量命名、代码格式、架构设计，却常常对提交信息敷衍了事。今天聊聊怎么用 **Conventional Commits** 把 Git 日志从"垃圾堆"变成"项目说明书"。

---

# 一、为什么你的 Commit Log 像天书

先来看几个真实的提交记录：

```
fix bug
update
tmp
WIP
refactor
some changes
```

这些记录可能在提交当时，你能明白它的含义，但是，在三个月后的你看来，这玩意其实和天书没什么区别。更糟的是，当团队需要排查问题、生成 CHANGELOG、或者做自动化发布时，这种混乱的日志几乎派不上任何用场。

**Conventional Commits（约定式提交）** 就是来解决这个问题的。它是一套轻量级的 Git 提交信息规范，由 Benjamin E. Coe 在 2017 年提出，灵感来自 Angular 团队的提交实践。核心思想很简单：**用结构化的格式，让每一次提交都清晰表达"做了什么"和"为什么做"**。

---

# 二、规范长什么样

一个标准的 Conventional Commit 长这样：

```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

拆开来逐个看：

## 1. type（类型）—— 必填

这是提交信息的"灵魂"，告诉机器和人这次提交的性质。官方规范只强制要求两个类型：

| 类型 | 含义 | 对应 SemVer |
|------|------|-------------|
| `feat` | 新功能 | MINOR（次版本） |
| `fix` | 修复 Bug | PATCH（补丁版本） |

除此之外，社区广泛采用 Angular 风格的扩展类型：

| 类型 | 什么时候用 |
|------|-----------|
| `docs` | 只改文档，不改代码 |
| `style` | 代码格式调整（空格、缩进、分号），不影响逻辑 |
| `refactor` | 重构代码，既不修 bug 也不加功能 |
| `perf` | 性能优化 |
| `test` | 增删测试代码 |
| `build` | 构建系统或外部依赖的变更 |
| `ci` | CI 配置和脚本调整 |
| `chore` | 杂项（改 `.gitignore`、更新依赖等） |
| `revert` | 回滚某个提交 |

## 2. scope（范围）—— 可选

用括号标注这次改动影响的模块或区域，帮助快速定位：

```
feat(auth): add JWT authentication
fix(ui): correct button alignment on mobile
```

scope 没有固定列表，由团队根据项目结构自行约定。比如一个博客系统可以有 `blog`、`theme`、`api`、`config` 等 scope。

## 3. description（描述）—— 必填

简短描述这次提交做了什么。有几个约定俗成的规则：

- **用祈使句**，现在时态。想象它在完成这句话："This commit will..."
- **小写开头**，不要大写
- **结尾不加句号**
- **不超过 72 个字符**

✅ 好的例子：
```
feat: add dark mode toggle
fix: resolve memory leak in event listener
```

❌ 不好的例子：
```
feat: Added dark mode toggle.    # 过去时 + 句号
fix: bug                         # 太模糊
```

## 4. body（正文）—— 可选

当标题说不清楚时，用正文补充**动机（Why）**和**具体改动（What）**。标题和正文之间必须空一行：

```
feat: support multi-language switching

Add English/Chinese toggle button with system language detection.
Default follows OS locale. Requires `i18n-utils` package.

- Add language context provider
- Update all hardcoded strings to translation keys
- Add `lang` query parameter override
```

## 5. footer（脚注）—— 可选

用于记录元信息，最常见两种用途：

**（1）破坏性变更（BREAKING CHANGE）**

当这次提交不兼容旧版本时，必须标注：

```
feat(api)!: replace getUser with fetchUser

BREAKING CHANGE: `getUser` method is removed. Use `fetchUser` instead.
Clients must update their integration code.
```

注意 `!` 符号可以放在 type/scope 后面，醒目地提示这是一个破坏性变更。这对应 SemVer 的 **MAJOR** 版本升级。

**（2）关联 Issue**

```
fix: resolve race condition in request handler

Closes #234
Refs: #198
```

---

# 三、用一个项目实战演练

假设我们在开发一个开源的配置管理平台，叫 **Perch**，支持多环境配置下发和版本控制。下面模拟一段真实的开发历程，看看 Conventional Commits 怎么贯穿其中。

## 项目结构

```
perch/
├── cmd/
│   └── server/          # 服务端入口
├── internal/
│   ├── api/             # HTTP API 层
│   ├── config/          # 配置解析逻辑
│   ├── store/           # 数据存储层
│   └── auth/            # 认证模块
├── web/
│   ├── ui/              # 前端界面
│   └── dashboard/       # 管理后台
├── docs/                # 文档
├── Makefile
└── README.md
```

## 场景 1：初始化项目骨架

```bash
git commit -m "feat: initialize perch project structure

Bootstrap Go module with standard layout.
Add Makefile for common tasks (build, test, lint).
Include basic README with architecture overview.

Closes #1"
```

## 场景 2：给 API 层加上 JWT 认证

```bash
git commit -m "feat(api): add JWT authentication middleware

Implement token-based auth using HS256.
Middleware extracts Bearer token from Authorization header
and validates against configured secret.

- Add `auth` package with JWT utilities
- Add `PERCH_JWT_SECRET` env variable
- Update API routes to use middleware chain

Closes #15"
```

## 场景 3：修复配置解析的边界问题

```bash
git commit -m "fix(config): handle empty environment variables

Previously, empty env vars caused the parser to panic
with index out of range. Now returns empty string safely
with a warning log.

- Add nil check in env resolver
- Add regression test for empty env var case

Closes #42"
```

## 场景 4：重构存储层，不改功能

```bash
git commit -m "refactor(store): extract database connection pool

Move connection management from inline to dedicated
`PoolManager` to support future read-replica support.
No functional changes.

Refs: #38"
```

## 场景 5：前端加了个新页面，纯 UI

```bash
git commit -m "feat(ui): add environment comparison page

Side-by-side diff view for comparing config values
across environments (dev/staging/prod).

- Add `DiffViewer` component
- Add route `/compare/:envA/:envB`
- Update navigation menu

Closes #56"
```

## 场景 6：CI 配置调整

```bash
git commit -m "ci: add GitHub Actions workflow for PR lint

Run golangci-lint and commitlint on every pull request.
Uses Go 1.22 and Node 20 runners.

- Add `.github/workflows/pr.yml`
- Add `.golangci.yml` configuration
- Add commitlint config with conventional preset"
```

## 场景 7：发了个大版本，有不兼容变更

```bash
git commit -m "feat(api)!: redesign configuration endpoint paths

BREAKING CHANGE: All API endpoints now prefixed with `/v1/`.
Previous paths (`/configs`, `/environments`) are removed.

Migration guide:
- `/configs` → `/v1/configs`
- `/environments` → `/v1/environments`
- `/auth/login` → `/v1/auth/login`

Clients must update their base URL or use the provided
migration helper script at `scripts/migrate-api.sh`.

Closes #78, #79"
```

---

# 四、它能带来什么好处

用了一段时间后，你会发现几个明显的变化：

## 1. 自动生成 CHANGELOG

工具比如 `conventional-changelog` 或 `standard-version` 可以直接扫描提交历史，按类型分类生成漂亮的发布日志：

```markdown
## [1.2.0] - 2026-07-10

### Features
- **api**: add JWT authentication middleware
- **ui**: add environment comparison page

### Bug Fixes
- **config**: handle empty environment variables

### BREAKING CHANGES
- **api**: redesign configuration endpoint paths
```

再也不用手动维护 CHANGELOG.md 了。

## 2. 语义化版本自动推断

结合 `semantic-release` 这类工具，版本号升级完全自动化：

- `fix:` → PATCH（1.0.0 → 1.0.1）
- `feat:` → MINOR（1.0.0 → 1.1.0）
- `BREAKING CHANGE` → MAJOR（1.0.0 → 2.0.0）

## 3. 快速筛选和排查

想快速找到所有破坏性变更？一行命令：

```bash
git log --oneline --grep="BREAKING CHANGE"
```

想看 auth 模块最近修了哪些 bug？

```bash
git log --oneline --grep="^fix(auth)"
```

## 4. 团队协作更顺畅

Code Review 时，扫一眼提交类型就知道这次改动的性质，不用逐行猜意图。新成员也能通过清晰的提交历史快速理解项目演进。

---

# 五、怎么在团队里落地

规范再好，推不下去也是白搭。这里分享一个渐进式的落地策略：

## 第一步：先手动写，不急着上工具

别一上来就装 `commitlint` + `husky`，容易让人反感。先让核心成员带头按规范写，大家看到好处自然就跟上了。

## 第二步：在 CONTRIBUTING.md 里写清楚约定

```markdown
## Commit Message Convention

We follow [Conventional Commits](https://www.conventionalcommits.org/).

### Types we use

- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation only
- `style`: Code style (formatting, no logic change)
- `refactor`: Code refactoring
- `perf`: Performance improvement
- `test`: Test related
- `build`: Build system or dependencies
- `ci`: CI/CD configuration
- `chore`: Other changes

### Scopes

- `api`: HTTP API layer
- `config`: Configuration parsing
- `store`: Database/storage layer
- `auth`: Authentication module
- `ui`: Frontend UI
- `ci`: CI/CD
- *(omit scope for global changes)*

### Example

feat(auth): add OAuth2 GitHub login

Implement GitHub OAuth2 flow as alternative to JWT.
Users can now login via GitHub account.

- Add `github` provider in auth package
- Add `PERCH_GITHUB_CLIENT_ID` env var
- Update login page with GitHub button

Closes #88

```

## 第三步：加 `commitlint` 做校验

等团队习惯了，再加硬约束。用 `@commitlint/config-conventional` 配合 Husky 在提交时自动检查：

```bash
npm install --save-dev @commitlint/config-conventional @commitlint/cli husky
npx husky install
npx husky add .husky/commit-msg 'npx --no -- commitlint --edit ${1}'
```

`.commitlintrc.json`：

```json
{
  "extends": ["@commitlint/config-conventional"],
  "rules": {
    "scope-enum": [2, "always", ["api", "config", "store", "auth", "ui", "ci"]]
  }
}
```

这样不符合规范的提交会被直接拦下来，从源头保证历史干净。

## 第四步：接入自动化发布

最后可以上 `semantic-release`，实现提交即发布：

```bash
npm install --save-dev semantic-release
```

配置 `.releaserc.json`：

```json
{
  "branches": ["main"],
  "plugins": [
    "@semantic-release/commit-analyzer",
    "@semantic-release/release-notes-generator",
    "@semantic-release/changelog",
    "@semantic-release/github"
  ]
}
```

配合 GitHub Actions，每次合并到 main 分支就自动打 tag、生成 release notes、发布版本。

---

# 六、一些常见误区

**Q: 类型选错了怎么办？**

还没合并到主分支的话，用 `git rebase -i` 修改。已经发布了就别纠结，下次注意就行。

**Q: 一个提交里既有 feat 又有 fix，怎么写？**

Conventional Commits 鼓励**原子提交**——一个提交只做一件事。尽量拆成两个提交。如果实在拆不开，选最主要那个类型，在 body 里说明其他改动。

**Q: 团队有人不想用怎么办？**

如果团队用 squash merge 工作流，维护者可以在合并时重写提交信息，贡献者不需要额外负担。但最好还是培养大家主动遵守的习惯。

**Q: 所有项目都必须用吗？**

个人小项目随意，但只要是多人协作、需要维护一段时间的项目，强烈建议引入。投入成本极低，长期收益巨大。

---

# 七、写在最后

Conventional Commits 不是银弹，它不会让你的代码质量自动提升，但它能让你的 **Git 历史从"负担"变成"资产"**。当一年后你回头看提交记录，能清晰知道每个版本发生了什么、为什么发生；当新同事加入，能通过日志快速理解项目脉络；当发布日来临，能一键生成 CHANGELOG 而不是熬夜手动整理。

说到底，写好的提交信息是一种**对未来的自己和他人的尊重**。花那几十秒多想一下 type 和 scope，换来的是整个项目生命周期的清晰和高效。

---

**参考**

- [Conventional Commits 1.0.0 官方规范](https://www.conventionalcommits.org/en/v1.0.0/)
- [Semantic Versioning 2.0.0](https://semver.org/)
- [commitlint](https://commitlint.js.org/)
- [semantic-release](https://semantic-release.gitbook.io/)
