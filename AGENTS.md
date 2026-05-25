# 开发规范

## 对话风格

- 回答简短精炼
- 提交信息、Issue、PR 评论和代码中不使用表情符号
- 不写冗余的客套话
- 只使用技术性语言，态度友善但直接（例如用 "谢谢 @user" 而不是 "非常感谢 @user！"）
- 用户提问时，先回答问题，再进行编辑或运行实现命令。

## 代码质量

- 在进行大范围修改之前，先完整阅读文件；在编辑尚未检查过的文件之前也要先完整阅读；在用户要求调查或审计时同样如此。不要仅依赖搜索片段进行大范围修改。
- 除非绝对必要，不使用 `any` 类型
- 禁止只有单一调用点的单行辅助函数；应直接内联。
- 检查 node_modules 中的外部 API 类型定义，而不是猜测
- **绝不使用内联导入** — 禁止 `await import("./foo.js")`，禁止在类型位置使用 `import("pkg").Type`，禁止为类型使用动态导入。始终使用标准的顶层导入。
- 绝不通过删除或降级代码来修复过时依赖的类型错误；应升级依赖。
- 仅使用与 Node strip-only 模式兼容的可擦除 TypeScript 语法，由根配置检查（`packages/*/src`、`packages/*/test` 和 `packages/coding-agent/examples`）。不要使用构造函数参数属性、`enum`、`namespace`/`module`、`import =`、`export =` 或其他需要 JavaScript 发射的 TypeScript 构造。使用显式字段和构造函数赋值代替参数属性。
- 在删除看似有意的功能或代码之前，必须先询问
- 除非用户明确要求，否则不保留向后兼容性
- 绝不硬编码按键检查，例如 `matchesKey(keyData, "ctrl+x")`。所有快捷键必须可配置。将默认值添加到对应的对象中（`DEFAULT_EDITOR_KEYBINDINGS` 或 `DEFAULT_APP_KEYBINDINGS`）
- 绝不直接修改 `packages/ai/src/models.generated.ts`。应改为更新 `packages/ai/scripts/generate-models.ts`。

## 命令

- 代码变更后（非文档变更）：运行 `npm run check`（获取完整输出，不要截断）。在提交前修复所有错误、警告和信息。
- 注意：`npm run check` 不运行测试。
- 绝不运行：`npm run build`、`npm test`
- 仅在用户指示时运行特定测试：`npx tsx ../../node_modules/vitest/dist/cli.js --run test/specific.test.ts`
- 从包的根目录运行测试，不是仓库根目录。
- 如果创建或修改了测试文件，必须运行该测试文件并迭代直到通过。
- 编写测试时，运行它们，识别测试或实现中的问题，并迭代直到修复。
- 对于 `packages/coding-agent/test/suite/`，使用 `test/suite/harness.ts` 加上 faux provider。不要使用真实 provider API、真实 API key 或付费 token。
- 将特定 issue 的回归测试放在 `packages/coding-agent/test/suite/regressions/` 下，命名为 `<issue-number>-<short-slug>.test.ts`。
- 对于临时脚本，将脚本写入临时文件（例如 `/tmp` 下），运行该文件，按需编辑，不再需要时删除。不要在 `bash` 命令中直接嵌入多行脚本。
- 除非用户要求，否则不提交

## 依赖和安装安全

- 将 npm 依赖和 lockfile 变更视为已审查的代码变更。直接外部依赖必须锁定到精确版本。
- 使用 `npm install --ignore-scripts` 在本地恢复/更新 `node_modules`。使用 `npm ci --ignore-scripts` 进行全新安装/CI 式验证。除非用户明确要求，否则不运行生命周期脚本。
- 如果依赖元数据变更，运行 `npm install --package-lock-only --ignore-scripts` 更新 `package-lock.json`，不安装也不运行脚本。
- 如果 `packages/coding-agent/npm-shrinkwrap.json` 需要重新生成，运行 `node scripts/generate-coding-agent-shrinkwrap.mjs`；使用 `node scripts/generate-coding-agent-shrinkwrap.mjs --check` 或 `npm run check` 验证。
- Pre-commit 钩子会阻止意外的 lockfile 提交，除非设置了 `PI_ALLOW_LOCKFILE_CHANGE=1`。除非用户明确要求提交 lockfile 变更，否则不要绕过。
- 带有生命周期脚本的新依赖需要审查并在 `scripts/generate-coding-agent-shrinkwrap.mjs` 中添加显式白名单条目；不要静默添加。

## 贡献门槛

- 来自新贡献者的新 Issue 会被 `.github/workflows/issue-gate.yml` 自动关闭
- 来自新贡献者（无 PR 权限）的新 PR 会被 `.github/workflows/pr-gate.yml` 自动关闭
- 维护者审批评论由 `.github/workflows/approve-contributor.yml` 处理
- 维护者每天审查自动关闭的 Issue
- 不符合 `CONTRIBUTING.md` 质量标准的 Issue 不会被重新打开，也不会收到回复
- `lgtmi` 批准未来的 Issue
- `lgtm` 批准未来的 Issue 和提交 PR 的权限

创建 Issue 时：

- 添加 `pkg:*` 标签以指示 Issue 影响的包
  - 可用标签：`pkg:agent`、`pkg:ai`、`pkg:coding-agent`、`pkg:tui`
- 如果 Issue 跨越多个包，添加所有相关标签

发布 Issue/PR 评论时：

- 将完整评论写入临时文件，使用 `gh issue comment --body-file` 或 `gh pr comment --body-file`
- 绝不在 shell 命令中通过 `--body` 直接传递多行 markdown
- 在发布前预览确切的评论文本
- 只发布一条最终评论，除非用户明确要求多条评论
- 如果评论格式错误，立即删除，然后发布一条修正后的评论
- 保持评论简洁、技术化，使用用户的语气

通过提交关闭 Issue 时：

- 在提交信息中包含 `fixes #<number>` 或 `closes #<number>`
- 这会在提交合并时自动关闭 Issue

## PR 工作流

- 先分析 PR，不要先拉取到本地
- 如果用户批准：创建功能分支、拉取 PR、变基到 main、应用调整、提交、合并到 main、推送、关闭 PR 并以用户的语气留下评论
- 你不自行创建 PR。我们在功能分支上工作直到一切符合用户要求，然后合并到 main 并推送。

## 使用 tmux 测试 pi 交互模式

在受控终端环境中测试 pi 的 TUI：

```bash
# 创建指定尺寸的 tmux 会话
tmux new-session -d -s pi-test -x 80 -y 24

# 从源码启动 pi
tmux send-keys -t pi-test "cd /Users/badlogic/workspaces/pi-mono && ./pi-test.sh" Enter

# 等待启动，然后捕获输出
sleep 3 && tmux capture-pane -t pi-test -p

# 发送输入
tmux send-keys -t pi-test "your prompt here" Enter

# 发送特殊按键
tmux send-keys -t pi-test Escape
tmux send-keys -t pi-test C-o  # ctrl+o

# 清理
tmux kill-session -t pi-test
```

## 变更日志

位置：`packages/*/CHANGELOG.md`（每个包有自己的变更日志）

### 格式

在 `## [Unreleased]` 下使用以下章节：

- `### Breaking Changes` — 需要迁移的 API 变更
- `### Added` — 新功能
- `### Changed` — 现有功能的变更
- `### Fixed` — 缺陷修复
- `### Removed` — 移除的功能

### 规则

- 在添加条目之前，先阅读完整的 `[Unreleased]` 章节以查看已有的子章节
- 新条目始终添加到 `## [Unreleased]` 章节下
- 追加到已有的子章节（如 `### Fixed`），不要创建重复项
- 绝不修改已发布的版本章节（如 `## [0.12.2]`）
- 每个版本章节一旦发布即不可变

### 归属标注

- **内部变更（来自 Issue）**：`Fixed foo bar ([#123](https://github.com/earendil-works/pi-mono/issues/123))`
- **外部贡献**：`Added feature X ([#456](https://github.com/earendil-works/pi-mono/pull/456) by [@username](https://github.com/username))`

## 添加新的 LLM Provider（packages/ai）

添加新 provider 需要修改多个文件：

### 1. 核心类型（`packages/ai/src/types.ts`）

- 将 API 标识符添加到 `Api` 类型联合中（如 `"bedrock-converse-stream"`）
- 创建继承 `StreamOptions` 的选项接口
- 添加到 `ApiOptionsMap` 的映射
- 将 provider 名称添加到 `KnownProvider` 类型联合中

### 2. Provider 实现（`packages/ai/src/providers/`）

创建 provider 文件，导出：

- `stream<Provider>()` 函数，返回 `AssistantMessageEventStream`
- `streamSimple<Provider>()` 用于 `SimpleStreamOptions` 映射
- Provider 特定的选项接口
- 消息/工具转换函数
- 响应解析，发出标准化事件（`text`、`tool_call`、`thinking`、`usage`、`stop`）

### 3. Provider 导出和延迟注册

- 在 `packages/ai/package.json` 中添加包子路径导出，指向 `./dist/providers/<provider>.js`
- 在 `packages/ai/src/index.ts` 中添加 `export type` 重导出，用于应从根入口可用的 provider 选项类型
- 在 `packages/ai/src/providers/register-builtins.ts` 中通过延迟加载包装器注册 provider，不要在那里静态导入 provider 实现模块
- 在 `packages/ai/src/env-api-keys.ts` 中添加凭证检测

### 4. 模型生成（`packages/ai/scripts/generate-models.ts`）

- 添加从 provider 源获取/解析模型的逻辑
- 映射到标准化的 `Model` 接口

### 5. 测试（`packages/ai/test/`）

- 始终将 provider 添加到 `stream.test.ts`，至少包含一个代表性模型，即使它复用了现有的 API 实现（如 `openai-completions`）
- 在适用的情况下将 provider 添加到更广泛的 provider 矩阵：`tokens.test.ts`、`abort.test.ts`、`empty.test.ts`、`context-overflow.test.ts`、`unicode-surrogate.test.ts`、`tool-call-without-result.test.ts`、`image-tool-result.test.ts`、`total-tokens.test.ts`、`cross-provider-handoff.test.ts`
- 对于 `cross-provider-handoff.test.ts`，至少添加一个 provider/model 对。如果 provider 暴露多个模型族（例如 GPT 和 Claude），每个族至少添加一对。
- 对于非标准认证，创建工具（如 `bedrock-utils.ts`）用于凭证检测。

### 6. Coding Agent（`packages/coding-agent/`）

- `src/core/model-resolver.ts`：将默认模型 ID 添加到 `defaultModelPerProvider`
- `src/core/provider-display-names.ts`：添加 API-key 登录显示名称，使 `/login` 和相关 UI 能显示该 provider 的内置 API-key 认证
- `src/cli/args.ts`：添加环境变量文档
- `README.md`：添加 provider 设置说明
- `docs/providers.md`：添加设置说明、环境变量和 `auth.json` 键

### 7. 文档

- `packages/ai/README.md`：添加到 providers 表，文档化选项/认证，添加环境变量
- `packages/ai/CHANGELOG.md`：在 `## [Unreleased]` 下添加条目

## 发布

**锁步版本控制**：所有包始终共享相同的版本号。每次发布同时更新所有包。

**版本语义**（无 major 发布）：

- `patch`：缺陷修复和新功能
- `minor`：API 破坏性变更

### 步骤

1. **更新 CHANGELOG**：确保上次发布以来的所有变更都记录在每个受影响包的 CHANGELOG.md 的 `[Unreleased]` 章节中

2. **软门槛：本地发布冒烟测试**：在运行真正的发布脚本之前，构建一个未发布的本地版本并在仓库外部手动冒烟测试，使其无法意外解析工作区文件：
   ```bash
   npm run release:local -- --out /tmp/pi-local-release --force
   cd /tmp
   /tmp/pi-local-release/node/pi --help
   /tmp/pi-local-release/node/pi --version
   /tmp/pi-local-release/node/pi
   /tmp/pi-local-release/bun/pi --help
   /tmp/pi-local-release/bun/pi --version
   ```
   在交互式冒烟测试中，验证启动、模型/账户列表，以及至少一个使用目标默认 provider 的真实提示。将失败视为发布阻断项，除非用户明确接受风险。

3. **运行发布脚本直到 npm publish**：
   ```bash
   npm run release:patch    # 修复和新增
   npm run release:minor    # API 破坏性变更
   ```

   npm 发布需要维护者的 npm WebAuthn/安全密钥 2FA，无法由 agent 单独完成。如果发布脚本在 `npm publish` 处停止并显示 npm 浏览器认证 URL，维护者必须在本地运行或批准 `npm run publish`。不要重新运行版本升级。

4. **发布成功后，完成发布收尾工作**：
   - 在包变更日志中添加新的 `## [Unreleased]` 章节。
   - 使用 `Add [Unreleased] section for next cycle` 提交。
   - 推送 `main` 和发布标签。

当 npm publish 认证已满足时，发布脚本处理完整流程。如果 npm 在发布期间需要 WebAuthn，使用上述步骤从现有的发布提交/标签手动继续。

## **关键** 并行 Agent 的 Git 规则 **关键**

多个 agent 可能同时在同一工作树中处理不同文件。你必须遵循以下规则：

### 提交

- **只提交你在本次会话中修改的文件**
- 始终在提交信息中包含 `fixes #<number>` 或 `closes #<number>`（当有关联的 Issue 或 PR 时）
- 绝不使用 `git add -A` 或 `git add .` — 这些会批量添加其他 agent 的变更
- 始终使用 `git add <具体文件路径>` 只列出你修改的文件
- 提交前运行 `git status` 验证你只暂存了自己的文件
- 跟踪你在会话中创建/修改/删除的文件
- 在提交中包含 `packages/ai/src/models.generated.ts` 始终是可以的（与你要提交的实际文件一起）

### 禁止的 Git 操作

以下命令可能破坏其他 agent 的工作：

- `git reset --hard` — 破坏未提交的变更
- `git checkout .` — 破坏未提交的变更
- `git clean -fd` — 删除未跟踪的文件
- `git stash` — 暂存所有变更，包括其他 agent 的工作
- `git add -A` / `git add .` — 暂存其他 agent 未提交的工作
- `git commit --no-verify` — 绕过必要的检查，永远不允许

### 安全工作流

```bash
# 1. 先检查状态
git status

# 2. 只添加你的特定文件
git add packages/ai/src/providers/transform-messages.ts
git add packages/ai/CHANGELOG.md

# 3. 提交
git commit -m "fix(ai): description"

# 4. 推送（如需要先 pull --rebase，但绝不 reset/checkout）
git pull --rebase && git push
```

### 如果变基冲突发生

- 只解决你自己文件中的冲突
- 如果冲突在你未修改的文件中，中止并询问用户
- 绝不 force push

### 用户覆盖

如果用户的指示与此处的规则冲突，先确认他们是否要覆盖规则。确认后再执行他们的指示。
