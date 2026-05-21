# ArcReel

AI 视频生成平台：将小说转化为短视频。本文件是领域术语表（ubiquitous language），只定义概念，不含实现细节。

## Language

### 供应商与后端

**provider（供应商）**：
一个媒体生成能力的提供方，由 provider id 标识（如 `gemini-aistudio`、`gemini-vertex`、`ark`、`custom-{id}`）。provider 是**身份**，不是连接对象。
_Avoid_: vendor、channel。

**backend（后端）**：
按某个 provider + model 构造出来的、真正调用其 API 的客户端对象。一个 provider 可派生出多个 backend。backend 是**构造物**，与 provider 身份是两件事——"选哪个 provider" 和 "造哪个 backend" 是两个独立决策。
_Avoid_: client（太泛）、adapter（另有架构含义）。

**规范 provider id（canonical provider id）**：
`PROVIDER_REGISTRY` 的 key 形式，是 provider 身份的唯一真相源与全系统唯一接受的写入形式。
_Avoid_: legacy provider 名。

**legacy provider 名**：
旧版本写入 `project.json` 的非规范别名（如 `gemini`、`aistudio`、`vertex`、`seedance`）。属于待清除的历史数据，**不是**有效身份；经一次性迁移转为规范 id 后即不再被接受（见 `docs/adr/0001`）。

### 解析

**provider 解析（resolution）**：
给定一个生成任务，决定它应使用哪个 **ProviderModel**。优先级自高而低：本次请求（payload）> 项目级（project.json）> 全局默认。这是"选身份"，不含 backend 构造。

**ProviderModel**：
provider 解析的结果——一对 `(provider_id, model_id)`（provider_id 为规范 id）。是"选了哪个 provider 及其 model"的值对象，**不是** backend（未构造任何客户端）。
_Avoid_: ResolvedBackend、BackendSelection（会与 backend 混淆）。

**capability（t2i / i2i）**：
图片任务的两种形态——t2i 文生图（无参考图）、i2i 图生图（带参考图）。一个镜头属于哪种，取决于"开画那一刻"是否拼出了参考图，**只有执行时才能确定**（见 `docs/adr/0001`）；入队与调度（worker claim）这两个执行前环节都无法获知。视频任务无 capability 维度。

## 示例对话

> **Dev**：worker 认领一个图片任务时，怎么知道用哪个 provider 限流？
> **Expert**：它做 provider 解析，但只到"选身份"为止——拿 provider 不拿 backend，更不真正生成。
> **Dev**：那它知道是 t2i 还是 i2i 吗？要是用户给两者配了不同 provider？
> **Expert**：不知道。capability 执行时才定，worker 只能按 t2i 取个代表性 provider 限流。真正用哪个，执行层会重新精确解析一次。
> **Dev**：那 project.json 里要是写着 `seedance` 呢？
> **Expert**：那是 legacy provider 名，迁移后不该再出现。系统只认规范 id `ark`。
