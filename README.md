# Web AI Workspace

## 中文说明

Web AI Workspace 是一个用于网页端多 AI 交叉协作的 Codex Skill。

它把浏览器中的 ChatGPT、Grok 或其他 AI 工作线程，与远程 Git、云盘和本地 Codex courier 连接起来，同时严格区分“谁负责思考”和“谁负责传递”。

### 核心职责

- AI worker：负责审阅、研究、编辑、语义整合和最终结论。
- Courier/controller：负责发送、路由、状态监控、精确传输、版本记录和机械工作区操作。
- Workspace adapter：负责 Git 或云盘的读取、写入、版本和发布操作。
- Finalizer：作为独立 worker，负责最终语义综合。

### 关键边界

Courier/controller 不得代替 worker：

- 审阅或总结内容；
- 改写、翻译或润色内容；
- 进行代码评审；
- 将自然语言修改要求转换为代码；
- 判断语义冲突或决定哪个结论正确；
- 把“无文本冲突”判断为“语义安全”；
- 把不完整、失败或迟到的结果伪装成成功。

### 支持的协作模式

| 模式 | 说明 |
| --- | --- |
| Remote-web Git reviewer | 网页端 AI 审阅远程仓库，可输出 review findings 或评论，但默认不能修改仓库 |
| Remote-web Git editor | 网页端 AI 在授权范围内创建分支、提交、推送或 PR |
| Local Codex courier | 本地 Codex 只接收精确 commit、patch、完整文件或二进制，不根据 prose 自行写代码 |
| Cloud-drive collaboration | 默认每个 worker 发布独立 artifact，避免多个 worker 同时覆盖同一正式文件 |
| Finalizer worker | 读取冻结的依赖快照，完成最终语义综合并发布最终 artifact |

### 版本与恢复

每次 handoff 使用精确版本、`task_id`、`attempt_id` 和 `handoff_id`。Artifact 需要保留 immutable identity、digest、publication evidence 和 provenance。

Skill 明确区分：

```text
TURN_COMPLETE       != TASK_SUCCEEDED
TASK_SUCCEEDED      != PUBLISHED
PUBLISHED           != DEPENDENCY_READY
NO_TEXT_CONFLICT    != SEMANTIC_SAFE
SUPERSEDED          != INVALIDATED
```

推送、上传、消息发送或云盘写入出现不确定结果时，使用 `UNKNOWN_OUTCOME`，先重新观察目标状态，再决定是否重试。

### 调用

在 Codex 中使用：

`$web-ai-workspace`

## English

Web AI Workspace is a Codex Skill for browser-based multi-AI cross-review and workflow collaboration.

It connects browser AI threads such as ChatGPT and Grok with remote Git, cloud-drive artifacts, and local Codex courier execution while preserving a strict separation between semantic work and mechanical transport.

### Core roles

- AI worker: performs review, research, editing, semantic integration, and conclusions.
- Courier/controller: sends messages, routes work, monitors state, transfers exact outputs, records versions, and performs mechanical workspace operations.
- Workspace adapter: reads, writes, versions, publishes, and verifies Git or cloud-drive artifacts.
- Finalizer: an independent worker responsible for final semantic synthesis.

### Boundary rules

The courier/controller must not replace a worker by:

- reviewing or summarizing content;
- rewriting, translating, or polishing content;
- performing code review;
- turning prose change requests into code;
- resolving semantic conflicts or choosing between conclusions;
- treating a clean textual merge as semantic approval;
- converting partial, failed, blocked, ambiguous, or late output into success.

### Supported modes

| Mode | Responsibility |
| --- | --- |
| Remote-web Git reviewer | Reviews a remote repository through the web and may write findings or comments, but has no implicit editor authority |
| Remote-web Git editor | Creates authorized branches, commits, pushes, or pull requests through the web |
| Local Codex courier | Handles only exact commits, patches, complete files, or binaries; it does not implement prose instructions |
| Cloud-drive collaboration | Publishes per-worker artifacts by default and avoids unsafe concurrent edits to one authoritative file |
| Finalizer worker | Performs semantic synthesis from a frozen dependency snapshot and publishes the final artifact |

### Versioning and recovery

Every handoff carries exact input versions, `task_id`, `attempt_id`, and `handoff_id`. Artifacts preserve immutable identity, digest, publication evidence, and provenance.

The Skill explicitly distinguishes:

```text
TURN_COMPLETE       != TASK_SUCCEEDED
TASK_SUCCEEDED      != PUBLISHED
PUBLISHED           != DEPENDENCY_READY
NO_TEXT_CONFLICT    != SEMANTIC_SAFE
SUPERSEDED          != INVALIDATED
```

When a push, upload, message send, or cloud write has an uncertain outcome, use `UNKNOWN_OUTCOME`, re-observe the destination, and retry only when safe.

### Invocation

Use the Skill in Codex with:

`$web-ai-workspace`

## Files

- `SKILL.md` — full orchestration contract.
- `agents/openai.yaml` — Codex display name and invocation metadata.
