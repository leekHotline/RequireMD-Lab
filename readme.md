**RequirementsDoc Lab** 展示：产品理解、工程落地、AI 编排、数据建模、可观测性。

头脑风暴方案：从定位 → 核心功能 → 数据模型 → API/页面 → AI 流程 → 指标与测试。

---

# RequirementsDoc Lab（需求文档实验室）— 项目概念

一个面向产品/研发的工具：把“零散需求”变成**可执行的工程规格**，并能追踪质量与效果。

输入可以是：

* 一段自然语言需求（微信/会议纪要/老板一句话）
* PRD 草稿
* 用户反馈集合
* issue 列表

输出是：

* 标准化需求文档（PRD / 用户故事 / 验收标准）
* 技术拆分（模块、接口、数据表、权限）
* 风险清单（边界条件、灰度、性能、风控）
* 测试用例（Vitest / e2e 提示）
* 变更对比（版本 diff）

---

# 能做的 3 个版本（从简单到硬核）

## V1：Doc Generator（最小可用）

* 新建项目
* 粘贴需求 → 生成：

  * 用户故事（As a… I want…）
  * 验收标准（Given/When/Then）
  * 需求边界（不做什么）
* 保存版本，支持历史查看

✅ 价值：能上线、能演示、能放作品集

## V2：Engineering Spec（工程化）

在 V1 基础上加：

* 生成接口草案（API contracts）
* 生成数据模型草案（ERD/表字段建议）
* 权限模型（RBAC/多租户）
* 生成任务拆分（Epic/Story/Task）

✅ 价值：你开始体现“全栈产品工程”

## V3：Quality Lab（AI 产品工程）

在 V2 基础上加：

* 需求质量评分（完整性/一致性/可测性/风险）
* 需求冲突检测（互斥/模糊/遗漏）
* 生成测试计划 + mock 数据
* 指标追踪：生成后用户修改了多少？哪些模块经常被推翻？

✅ 价值：这就是“AI 产品工程”，不是文本生成

---

# 技术栈怎么用（对齐你列的栈）

## UI（Next + Tailwind + shadcn + Motion）

* 文档编辑器页面：左边输入，右边生成预览
* 版本对比页面：diff viewer（可以用开源 diff 组件）
* Dashboard 页面：质量评分趋势、用户修改率等
* Motion 做：生成结果逐段出现、版本切换动效

## API（route.ts 或 Hono）

* `/api/projects`
* `/api/docs`
* `/api/docs/:id/generate`
* `/api/docs/:id/versions`
* `/api/metrics`

Hono 的价值点：路由分组 + 全局中间件（auth/log/rate limit）更顺。

## 数据库（Supabase Postgres）

最关键：你要把“需求文档”当结构化资产，而不是一坨字符串。

---

# 数据建模（建议的表）

最小一套（V1 就够）：

* `users`
* `projects`
* `requirements_docs`

  * id, project_id, title, raw_input, created_by
* `doc_versions`

  * doc_id, version, generated_markdown, model, prompt_hash, created_at
* `quality_scores`（V3）

  * version_id, completeness, clarity, testability, risk, overall
* `events`（埋点）

  * user_id, event_name, payload(jsonb), created_at

你这套表一出来，面试官一看就知道你“懂产品工程”。

---

# AI 流程设计（这项目的灵魂）

你不要“一次生成一整篇”，而是**分阶段生成 + 可解释**：

### 生成流水线（pipeline）

1. **Normalize**：把输入需求清洗成结构化要点（目标、用户、场景、约束）
2. **Spec**：生成用户故事 + 验收标准 + 边界条件
3. **Engineering**：生成 API / 数据表 / 权限建议
4. **QA**：生成测试点、异常场景、回归清单
5. **Score**：输出质量评分 + 改进建议

每一步都可以：

* 单独 rerun
* 单独保存版本
* 单独对比

这就体现：
✅ 你懂“AI 产品工程 = 编排系统”，不是 call API

---

# Middleware（你能在这项目里体现得很自然）

**Web middleware：**

* Auth：只有登录用户能创建项目/文档
* Rate limit：防止狂点生成
* Logging：记录耗时、错误、token 使用
* Validation：校验输入（项目id、文档id）
* Error handler：统一返回错误结构

**系统中间件（可选）：**

* Redis：缓存生成结果 / 限流计数
* Queue：生成任务异步化（长文档）
* S3/R2：存附件（会议纪要文件）

---

# Dashboard 指标（这项目会非常像真实产品）

你可以追踪：

* 生成次数 / 日活
* 平均生成耗时
* 用户对生成结果的“修改率”（改得越多说明质量差）
* 需求质量评分趋势（overall score）
* 哪类需求最容易“模糊”（比如边界缺失最多）

这种指标是 JD 里特别喜欢的。

---

# Vitest 测试点（别只测 API）

你要测“业务逻辑”：

* quality score 的计算逻辑（如果你做加权）
* pipeline 每一步的输入输出 schema
* auth middleware 是否拦截
* 版本号递增是否正确
* prompt_hash 相同是否复用缓存（可选）

---

# 你可以选一个“杀手级差异化功能”

让这个项目一下变得很有辨识度：

### 1) 版本 diff

用户手改后保存新版本，系统给出：

* 改动摘要（AI 生成）
* 哪些验收标准被改了
* 哪些风险被新增/删除

### 2) “模糊点雷达”

自动标红：

* 时间/数量/边界不明确
* 角色权限不明确
* 失败兜底不明确

### 3) 一键生成 Jira/GitHub Issues

把需求拆成 tasks，导出 JSON/CSV。

---

# 最后：我给你一个落地建议（不绕）

你就做 **RequirementsDoc Lab V1 + 一个差异化功能（版本 diff）**。

这个组合最稳：

* 一周能做完
* 演示效果强
* 面试讲得出来（分层、数据、AI pipeline、指标）

---

细化成一份**PRD/Requirements Doc**（项目本身就是需求文档实验室嘛）：

* 用户故事
* 页面信息架构
* API 列表
* 表结构 SQL
* 里程碑拆分（MVP→V2→V3）

用户是谁更偏向？

1. AI创业者（偏 PRD 输出）
2. 工程团队（偏技术规格、任务拆分）
3. 两者都要（我会做成多模式）
